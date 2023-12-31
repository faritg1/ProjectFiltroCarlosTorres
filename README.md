# Carlos farit

Devuelve un listado con el código de pedido, código de cliente, fecha 
esperada y fecha de entrega de los pedidos cuya fecha de entrega ha sido al 
menos dos días antes de la fecha esperada

http://localhost:5004/Consultas/Primera

```[HttpGet("Primera")]
        [ProducesResponseType(StatusCodes.Status200OK)]
        [ProducesResponseType(StatusCodes.Status400BadRequest)]
        public Task<IQueryable<PrimeraDto>> Getprimera()
        {
            var client = from cli in _context.Clientes 
                        join pe in _context.Pedidos on cli.Id equals pe.IdCliente 
                        join de in _context.DetallePedidos on cli.Id equals de.IdPedido
            where (pe.FechaEntrega > pe.FechaEsperada || pe.FechaEntrega.Day == 0)
            select new  PrimeraDto{    
                IdCliente = cli.Id,
                FechaEntrega = pe.FechaEntrega,
                FechaEsperada = pe.FechaEsperada

            };
            return  Task.FromResult(client);
        }```

Devuelve las oficinas donde no trabajan ninguno de los empleados que 
hayan sido los representantes de ventas de algún cliente que haya realizado 
la compra de algún producto de la gama Frutales.

http://localhost:5004/Consultas/Tercera


```[HttpGet("Tercera")]
        [ProducesResponseType(StatusCodes.Status200OK)]
        [ProducesResponseType(StatusCodes.Status400BadRequest)]
        [ProducesResponseType(StatusCodes.Status404NotFound)]
        public Task<IQueryable<TercerDto>> GetOfficeNotContent(){
            var office = from emp in _context.Empleados
                        join off in _context.Oficinas on emp.IdOficina equals off.Id
                        join cli in _context.Clientes on emp.Id equals cli.IdEmpleado
            where cli.IdEmpleado != emp.Id
            select new TercerDto{
                NombreOficina = off.LineaDireccion1,
                NombreEmpleado = emp.Nombre
            };
            return Task.FromResult(office);
        }```

Devuelve un listado de los 20 productos más vendidos y el número total de 
unidades que se han vendido de cada uno. El listado deberá estar ordenado 
por el número total de unidades vendidas.

http://localhost:5004/Consultas/Cuarta

```[HttpGet("Cuarta")] // revisar
        [ProducesResponseType(StatusCodes.Status200OK)]
        [ProducesResponseType(StatusCodes.Status400BadRequest)]
        [ProducesResponseType(StatusCodes.Status404NotFound)]
        public async Task<IEnumerable<CuartaDto>> GetTopSoldProducts()
        {
            var groupedResults = await (
            from Detalle in _context.DetallePedidos
            join product in _context.Productos on Detalle.IdProducto equals product.Id
            group new { Detalle, product } by new { product.Id, product.Nombre } into groupedDetalles
            orderby groupedDetalles.Sum(od => od.Detalle.Cantidad) descending
            select new
            {
                ProductId = groupedDetalles.Key.Id,
                ProductName = groupedDetalles.Key.Nombre,
                TotalUnitsSold = groupedDetalles.Sum(od => od.Detalle.Cantidad)
            })
            .Take(20)
            .ToListAsync();

            var topSoldProducts = groupedResults
            .Select(result => new CuartaDto
            {
                ProductId = result.ProductId,
                ProductName = result.ProductName,
                TotalUnitsSold = result.TotalUnitsSold
            })
            .ToList();

            return topSoldProducts;
        }
```

Lista las ventas totales de los productos que hayan facturado más de 3000 
euros. Se mostrará el nombre, unidades vendidas, total facturado y total 
facturado con impuestos (21% IVA)

http://localhost:5004/Consultas/Quinta

```
[HttpGet("Quinta")]
        [ProducesResponseType(StatusCodes.Status200OK)]
        [ProducesResponseType(StatusCodes.Status400BadRequest)]
        [ProducesResponseType(StatusCodes.Status404NotFound)]
        public async Task<IEnumerable<ProductSalesDto>> GetProductsSalesOver3000()
        {
            var productsSales = await (from DetallePedidoDto in _context.DetallePedidos
            join product in _context.Productos on DetallePedidoDto.IdProducto equals product.Id
            group new { DetallePedidoDto, product } by new { product.Id, product.Nombre } into grouped
            where grouped.Sum(g => g.DetallePedidoDto.Cantidad * g.product.CantidadEnStock) > 3000
            select new ProductSalesDto
            {
                ProductName = grouped.Key.Nombre,
                UnitsSold = grouped.Sum(g => g.DetallePedidoDto.Cantidad),
                TotalSales = grouped.Sum(g => g.DetallePedidoDto.Cantidad * g.product.CantidadEnStock),
                TotalSalesWithTax = grouped.Sum(g => g.DetallePedidoDto.Cantidad * g.product.CantidadEnStock) * 0.21
            })
            .ToListAsync();

            return productsSales;
        }
```


Devuelve un listado con los datos de los empleados que no tienen clientes 
asociados y el nombre de su jefe asociado

http://localhost:5004/Consultas/Novena
```
[HttpGet("Novena")]
        [ProducesResponseType(StatusCodes.Status200OK)]
        [ProducesResponseType(StatusCodes.Status400BadRequest)]
        [ProducesResponseType(StatusCodes.Status404NotFound)]
        public Task<IQueryable<NovenaDto>> GetEmployeeNotClient(){
            var emplo = from emp in _context.Empleados
                        join cli in _context.Clientes on emp.Id equals cli.IdEmpleado
                        into emplClient from empCli in emplClient.DefaultIfEmpty()
            where emp.Id != empCli.IdEmpleado
            select new NovenaDto{
                Id = emp.Id,
                Nombre = emp.Nombre,
                Apellido1 = emp.Apellido1,
                Apellido2 = emp.Apellido2
            };
            return Task.FromResult(emplo);
        }
```

Devuelve el nombre del producto del que se han vendido más unidades. 
(Tenga en cuenta que tendrá que calcular cuál es el número total de 
unidades que se han vendido de cada producto a partir de los datos de la 
tabla detalle_pedido)

http://localhost:5004/Consultas/Decima

```
[HttpGet("Decima")]
        [ProducesResponseType(StatusCodes.Status200OK)]
        [ProducesResponseType(StatusCodes.Status400BadRequest)]
        [ProducesResponseType(StatusCodes.Status404NotFound)]
        public Task<IQueryable<DecimaDto>> GetProductOrderDetail(){
            var product = from pro in _context.Productos // CUANDO SE VALIA QUE NO HAYA APARECIDO NINGUN PRODUCTO DEBE SALIR EL PRODUCTO Y SE TENDRA QUE COMENZAR POR LA TABLA DEL PRODUCTO.
                        join orderDetail in _context.DetallePedidos on pro.Id equals orderDetail.IdProducto
                        into orderDetaProd from ordPro in orderDetaProd.DefaultIfEmpty()
                        join ranPro in _context.GamaProductos on pro.IdGama equals ranPro.Id
                        into rangProdu from rangerPro in rangProdu.DefaultIfEmpty()
            where pro.Id != ordPro.IdProducto
            select new DecimaDto{
                Nombre = pro.Nombre,
                Des = rangerPro.DescripcionTexto,
                Img = rangerPro.Imagen
            };
            return Task.FromResult(product);
        }

```