# 05. Integración de Datos

## 🗄️ Entity Framework Core

### Configuración inicial
```csharp
// Instalación de paquetes NuGet
// dotnet add package Microsoft.EntityFrameworkCore.SqlServer
// dotnet add package Microsoft.EntityFrameworkCore.Tools
// dotnet add package Microsoft.EntityFrameworkCore.Design

// DbContext principal
public class TiendaDbContext : DbContext
{
    public TiendaDbContext(DbContextOptions<TiendaDbContext> options) : base(options)
    {
    }

    // DbSets (tablas)
    public DbSet<Producto> Productos { get; set; }
    public DbSet<Categoria> Categorias { get; set; }
    public DbSet<Cliente> Clientes { get; set; }
    public DbSet<Pedido> Pedidos { get; set; }
    public DbSet<DetallePedido> DetallesPedido { get; set; }
    public DbSet<Proveedor> Proveedores { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        // Configurar entidades
        ConfigurarProducto(modelBuilder);
        ConfigurarCategoria(modelBuilder);
        ConfigurarCliente(modelBuilder);
        ConfigurarPedido(modelBuilder);
        ConfigurarDetallePedido(modelBuilder);
        ConfigurarProveedor(modelBuilder);

        // Seed data
        SeedData(modelBuilder);
    }

    private void ConfigurarProducto(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Producto>(entity =>
        {
            entity.HasKey(p => p.Id);
            
            entity.Property(p => p.Nombre)
                  .IsRequired()
                  .HasMaxLength(200);
            
            entity.Property(p => p.Descripcion)
                  .HasMaxLength(1000);
            
            entity.Property(p => p.Precio)
                  .IsRequired()
                  .HasPrecision(18, 2);
            
            entity.Property(p => p.Stock)
                  .IsRequired()
                  .HasDefaultValue(0);
            
            entity.Property(p => p.FechaCreacion)
                  .IsRequired()
                  .HasDefaultValueSql("GETUTCDATE()");

            // Relación con Categoria
            entity.HasOne(p => p.Categoria)
                  .WithMany(c => c.Productos)
                  .HasForeignKey(p => p.CategoriaId)
                  .OnDelete(DeleteBehavior.Restrict);

            // Relación con Proveedor
            entity.HasOne(p => p.Proveedor)
                  .WithMany(pr => pr.Productos)
                  .HasForeignKey(p => p.ProveedorId)
                  .OnDelete(DeleteBehavior.Restrict);

            // Índices
            entity.HasIndex(p => p.Nombre);
            entity.HasIndex(p => p.CategoriaId);
            entity.HasIndex(p => new { p.Precio, p.Stock });
        });
    }

    private void ConfigurarCategoria(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Categoria>(entity =>
        {
            entity.HasKey(c => c.Id);
            
            entity.Property(c => c.Nombre)
                  .IsRequired()
                  .HasMaxLength(100);
            
            entity.Property(c => c.Descripcion)
                  .HasMaxLength(500);

            entity.HasIndex(c => c.Nombre)
                  .IsUnique();
        });
    }

    private void ConfigurarCliente(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Cliente>(entity =>
        {
            entity.HasKey(c => c.Id);
            
            entity.Property(c => c.Nombre)
                  .IsRequired()
                  .HasMaxLength(200);
            
            entity.Property(c => c.Email)
                  .IsRequired()
                  .HasMaxLength(255);
            
            entity.Property(c => c.Telefono)
                  .HasMaxLength(20);

            entity.HasIndex(c => c.Email)
                  .IsUnique();
        });
    }

    private void ConfigurarPedido(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Pedido>(entity =>
        {
            entity.HasKey(p => p.Id);
            
            entity.Property(p => p.FechaPedido)
                  .IsRequired()
                  .HasDefaultValueSql("GETUTCDATE()");
            
            entity.Property(p => p.Total)
                  .HasPrecision(18, 2);
            
            entity.Property(p => p.Estado)
                  .IsRequired()
                  .HasConversion<string>()
                  .HasMaxLength(50);

            // Relación con Cliente
            entity.HasOne(p => p.Cliente)
                  .WithMany(c => c.Pedidos)
                  .HasForeignKey(p => p.ClienteId)
                  .OnDelete(DeleteBehavior.Restrict);

            entity.HasIndex(p => p.FechaPedido);
            entity.HasIndex(p => p.Estado);
        });
    }

    private void ConfigurarDetallePedido(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<DetallePedido>(entity =>
        {
            entity.HasKey(d => d.Id);
            
            entity.Property(d => d.Cantidad)
                  .IsRequired();
            
            entity.Property(d => d.PrecioUnitario)
                  .IsRequired()
                  .HasPrecision(18, 2);

            // Relación con Pedido
            entity.HasOne(d => d.Pedido)
                  .WithMany(p => p.Detalles)
                  .HasForeignKey(d => d.PedidoId)
                  .OnDelete(DeleteBehavior.Cascade);

            // Relación con Producto
            entity.HasOne(d => d.Producto)
                  .WithMany()
                  .HasForeignKey(d => d.ProductoId)
                  .OnDelete(DeleteBehavior.Restrict);

            // Clave compuesta alternativa
            entity.HasIndex(d => new { d.PedidoId, d.ProductoId })
                  .IsUnique();
        });
    }

    private void SeedData(ModelBuilder modelBuilder)
    {
        // Seed Categorias
        modelBuilder.Entity<Categoria>().HasData(
            new Categoria { Id = 1, Nombre = "Electrónicos", Descripcion = "Dispositivos electrónicos" },
            new Categoria { Id = 2, Nombre = "Ropa", Descripcion = "Vestimenta y accesorios" },
            new Categoria { Id = 3, Nombre = "Hogar", Descripcion = "Artículos para el hogar" }
        );

        // Seed Proveedores
        modelBuilder.Entity<Proveedor>().HasData(
            new Proveedor { Id = 1, Nombre = "TechCorp", Email = "contacto@techcorp.com", Telefono = "555-0001" },
            new Proveedor { Id = 2, Nombre = "Fashion Inc", Email = "ventas@fashion.com", Telefono = "555-0002" }
        );

        // Seed Productos
        modelBuilder.Entity<Producto>().HasData(
            new Producto 
            { 
                Id = 1, 
                Nombre = "Laptop Gaming", 
                Descripcion = "Laptop para gaming de alta gama",
                Precio = 1299.99m,
                Stock = 10,
                CategoriaId = 1,
                ProveedorId = 1,
                FechaCreacion = DateTime.UtcNow
            },
            new Producto 
            { 
                Id = 2, 
                Nombre = "Camiseta Premium", 
                Descripcion = "Camiseta de algodón premium",
                Precio = 29.99m,
                Stock = 50,
                CategoriaId = 2,
                ProveedorId = 2,
                FechaCreacion = DateTime.UtcNow
            }
        );
    }
}
```

### Modelos de entidades
```csharp
// Entidad base para auditoría
public abstract class EntidadBase
{
    public int Id { get; set; }
    public DateTime FechaCreacion { get; set; }
    public DateTime? FechaModificacion { get; set; }
    public string? UsuarioCreacion { get; set; }
    public string? UsuarioModificacion { get; set; }
}

public class Producto : EntidadBase
{
    public string Nombre { get; set; } = string.Empty;
    public string? Descripcion { get; set; }
    public decimal Precio { get; set; }
    public int Stock { get; set; }
    public bool Activo { get; set; } = true;
    
    // Foreign Keys
    public int CategoriaId { get; set; }
    public int ProveedorId { get; set; }
    
    // Navigation Properties
    public virtual Categoria Categoria { get; set; } = null!;
    public virtual Proveedor Proveedor { get; set; } = null!;
    public virtual ICollection<DetallePedido> DetallesPedido { get; set; } = new List<DetallePedido>();
    
    // Propiedades calculadas
    public decimal ValorInventario => Precio * Stock;
    public bool TieneBajoStock => Stock < 10;
}

public class Categoria : EntidadBase
{
    public string Nombre { get; set; } = string.Empty;
    public string? Descripcion { get; set; }
    public bool Activa { get; set; } = true;
    
    // Navigation Properties
    public virtual ICollection<Producto> Productos { get; set; } = new List<Producto>();
    
    // Propiedades calculadas
    public int CantidadProductos => Productos?.Count ?? 0;
}

public class Cliente : EntidadBase
{
    public string Nombre { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
    public string? Telefono { get; set; }
    public string? Direccion { get; set; }
    public DateTime? FechaNacimiento { get; set; }
    public bool Activo { get; set; } = true;
    
    // Navigation Properties
    public virtual ICollection<Pedido> Pedidos { get; set; } = new List<Pedido>();
    
    // Propiedades calculadas
    public int CantidadPedidos => Pedidos?.Count ?? 0;
    public decimal TotalComprado => Pedidos?.Sum(p => p.Total) ?? 0;
}

public class Pedido : EntidadBase
{
    public DateTime FechaPedido { get; set; }
    public EstadoPedido Estado { get; set; }
    public decimal Total { get; set; }
    public string? DireccionEnvio { get; set; }
    public string? Observaciones { get; set; }
    
    // Foreign Key
    public int ClienteId { get; set; }
    
    // Navigation Properties
    public virtual Cliente Cliente { get; set; } = null!;
    public virtual ICollection<DetallePedido> Detalles { get; set; } = new List<DetallePedido>();
    
    // Propiedades calculadas
    public int CantidadArticulos => Detalles?.Sum(d => d.Cantidad) ?? 0;
    public bool EsReciente => FechaPedido >= DateTime.UtcNow.AddDays(-30);
}

public class DetallePedido
{
    public int Id { get; set; }
    public int Cantidad { get; set; }
    public decimal PrecioUnitario { get; set; }
    
    // Foreign Keys
    public int PedidoId { get; set; }
    public int ProductoId { get; set; }
    
    // Navigation Properties
    public virtual Pedido Pedido { get; set; } = null!;
    public virtual Producto Producto { get; set; } = null!;
    
    // Propiedades calculadas
    public decimal Subtotal => Cantidad * PrecioUnitario;
}

public class Proveedor : EntidadBase
{
    public string Nombre { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
    public string? Telefono { get; set; }
    public string? Direccion { get; set; }
    public bool Activo { get; set; } = true;
    
    // Navigation Properties
    public virtual ICollection<Producto> Productos { get; set; } = new List<Producto>();
}

public enum EstadoPedido
{
    Pendiente,
    Confirmado,
    Preparando,
    Enviado,
    Entregado,
    Cancelado
}
```

## 🔄 Migraciones y Code First

### Comandos de migración
```bash
# Crear primera migración
dotnet ef migrations add InicialCreate

# Aplicar migraciones a la base de datos
dotnet ef database update

# Crear migración para nuevos cambios
dotnet ef migrations add AgregarCamposTelefono

# Ver el SQL que se ejecutará
dotnet ef migrations script

# Revertir a una migración específica
dotnet ef database update NombreMigracionAnterior

# Eliminar la última migración (si no se ha aplicado)
dotnet ef migrations remove

# Generar script SQL para producción
dotnet ef migrations script --output migracion.sql
```

### Configuración de migración personalizada
```csharp
// Migración personalizada con datos específicos
public partial class AgregarDatosIniciales : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        // Agregar categorías adicionales
        migrationBuilder.InsertData(
            table: "Categorias",
            columns: new[] { "Id", "Nombre", "Descripcion", "FechaCreacion", "Activa" },
            values: new object[,]
            {
                { 4, "Deportes", "Artículos deportivos y fitness", DateTime.UtcNow, true },
                { 5, "Libros", "Literatura y material educativo", DateTime.UtcNow, true }
            });

        // Crear índices adicionales para performance
        migrationBuilder.CreateIndex(
            name: "IX_Productos_Precio_Stock",
            table: "Productos",
            columns: new[] { "Precio", "Stock" });

        migrationBuilder.CreateIndex(
            name: "IX_Pedidos_FechaPedido_Estado",
            table: "Pedidos",
            columns: new[] { "FechaPedido", "Estado" });

        // Agregar constraint personalizada
        migrationBuilder.Sql(@"
            ALTER TABLE Productos 
            ADD CONSTRAINT CK_Productos_PrecioPositivo 
            CHECK (Precio > 0)");

        migrationBuilder.Sql(@"
            ALTER TABLE Productos 
            ADD CONSTRAINT CK_Productos_StockNoNegativo 
            CHECK (Stock >= 0)");
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.Sql("ALTER TABLE Productos DROP CONSTRAINT CK_Productos_PrecioPositivo");
        migrationBuilder.Sql("ALTER TABLE Productos DROP CONSTRAINT CK_Productos_StockNoNegativo");
        
        migrationBuilder.DropIndex(
            name: "IX_Productos_Precio_Stock",
            table: "Productos");

        migrationBuilder.DropIndex(
            name: "IX_Pedidos_FechaPedido_Estado",
            table: "Pedidos");

        migrationBuilder.DeleteData(
            table: "Categorias",
            keyColumn: "Id",
            keyValues: new object[] { 4, 5 });
    }
}
```

## 🔍 Consultas LINQ Avanzadas

### Repository Pattern con Entity Framework
```csharp
// Interface genérica del repositorio
public interface IRepositorio<T> where T : EntidadBase
{
    Task<T?> ObtenerPorIdAsync(int id);
    Task<IEnumerable<T>> ObtenerTodosAsync();
    Task<IEnumerable<T>> ObtenerPorCriterioAsync(Expression<Func<T, bool>> criterio);
    Task<T> AgregarAsync(T entidad);
    Task<T> ActualizarAsync(T entidad);
    Task<bool> EliminarAsync(int id);
    Task<bool> ExisteAsync(int id);
    Task<int> ContarAsync();
    Task<int> ContarPorCriterioAsync(Expression<Func<T, bool>> criterio);
}

// Implementación genérica del repositorio
public class Repositorio<T> : IRepositorio<T> where T : EntidadBase
{
    protected readonly TiendaDbContext _context;
    protected readonly DbSet<T> _dbSet;
    protected readonly ILogger<Repositorio<T>> _logger;

    public Repositorio(TiendaDbContext context, ILogger<Repositorio<T>> logger)
    {
        _context = context;
        _dbSet = context.Set<T>();
        _logger = logger;
    }

    public virtual async Task<T?> ObtenerPorIdAsync(int id)
    {
        return await _dbSet.FindAsync(id);
    }

    public virtual async Task<IEnumerable<T>> ObtenerTodosAsync()
    {
        return await _dbSet.ToListAsync();
    }

    public virtual async Task<IEnumerable<T>> ObtenerPorCriterioAsync(Expression<Func<T, bool>> criterio)
    {
        return await _dbSet.Where(criterio).ToListAsync();
    }

    public virtual async Task<T> AgregarAsync(T entidad)
    {
        entidad.FechaCreacion = DateTime.UtcNow;
        _dbSet.Add(entidad);
        await _context.SaveChangesAsync();
        
        _logger.LogInformation("Entidad {TipoEntidad} con ID {Id} agregada", typeof(T).Name, entidad.Id);
        return entidad;
    }

    public virtual async Task<T> ActualizarAsync(T entidad)
    {
        entidad.FechaModificacion = DateTime.UtcNow;
        _dbSet.Update(entidad);
        await _context.SaveChangesAsync();
        
        _logger.LogInformation("Entidad {TipoEntidad} con ID {Id} actualizada", typeof(T).Name, entidad.Id);
        return entidad;
    }

    public virtual async Task<bool> EliminarAsync(int id)
    {
        var entidad = await _dbSet.FindAsync(id);
        if (entidad == null) return false;

        _dbSet.Remove(entidad);
        await _context.SaveChangesAsync();
        
        _logger.LogInformation("Entidad {TipoEntidad} con ID {Id} eliminada", typeof(T).Name, id);
        return true;
    }

    public virtual async Task<bool> ExisteAsync(int id)
    {
        return await _dbSet.AnyAsync(e => e.Id == id);
    }

    public virtual async Task<int> ContarAsync()
    {
        return await _dbSet.CountAsync();
    }

    public virtual async Task<int> ContarPorCriterioAsync(Expression<Func<T, bool>> criterio)
    {
        return await _dbSet.CountAsync(criterio);
    }
}

// Repositorio específico para Productos con consultas especializadas
public interface IProductoRepositorio : IRepositorio<Producto>
{
    Task<IEnumerable<Producto>> ObtenerPorCategoriaAsync(int categoriaId);
    Task<IEnumerable<Producto>> ObtenerConBajoStockAsync(int limite = 10);
    Task<IEnumerable<Producto>> BuscarPorNombreAsync(string termino);
    Task<IEnumerable<Producto>> ObtenerMasVendidosAsync(int cantidad = 10);
    Task<IEnumerable<ProductoEstadistica>> ObtenerEstadisticasVentasAsync();
    Task<bool> ActualizarStockAsync(int productoId, int nuevoStock);
}

public class ProductoRepositorio : Repositorio<Producto>, IProductoRepositorio
{
    public ProductoRepositorio(TiendaDbContext context, ILogger<ProductoRepositorio> logger) 
        : base(context, logger)
    {
    }

    public override async Task<Producto?> ObtenerPorIdAsync(int id)
    {
        return await _dbSet
            .Include(p => p.Categoria)
            .Include(p => p.Proveedor)
            .FirstOrDefaultAsync(p => p.Id == id);
    }

    public override async Task<IEnumerable<Producto>> ObtenerTodosAsync()
    {
        return await _dbSet
            .Include(p => p.Categoria)
            .Include(p => p.Proveedor)
            .Where(p => p.Activo)
            .OrderBy(p => p.Nombre)
            .ToListAsync();
    }

    public async Task<IEnumerable<Producto>> ObtenerPorCategoriaAsync(int categoriaId)
    {
        return await _dbSet
            .Include(p => p.Categoria)
            .Include(p => p.Proveedor)
            .Where(p => p.CategoriaId == categoriaId && p.Activo)
            .OrderBy(p => p.Nombre)
            .ToListAsync();
    }

    public async Task<IEnumerable<Producto>> ObtenerConBajoStockAsync(int limite = 10)
    {
        return await _dbSet
            .Include(p => p.Categoria)
            .Where(p => p.Stock <= limite && p.Activo)
            .OrderBy(p => p.Stock)
            .ToListAsync();
    }

    public async Task<IEnumerable<Producto>> BuscarPorNombreAsync(string termino)
    {
        return await _dbSet
            .Include(p => p.Categoria)
            .Include(p => p.Proveedor)
            .Where(p => p.Nombre.Contains(termino) && p.Activo)
            .OrderBy(p => p.Nombre)
            .ToListAsync();
    }

    public async Task<IEnumerable<Producto>> ObtenerMasVendidosAsync(int cantidad = 10)
    {
        return await _dbSet
            .Include(p => p.Categoria)
            .Include(p => p.DetallesPedido)
            .Where(p => p.Activo)
            .OrderByDescending(p => p.DetallesPedido.Sum(d => d.Cantidad))
            .Take(cantidad)
            .ToListAsync();
    }

    public async Task<IEnumerable<ProductoEstadistica>> ObtenerEstadisticasVentasAsync()
    {
        return await _context.Productos
            .Where(p => p.Activo)
            .Select(p => new ProductoEstadistica
            {
                ProductoId = p.Id,
                NombreProducto = p.Nombre,
                CantidadVendida = p.DetallesPedido.Sum(d => d.Cantidad),
                TotalVentas = p.DetallesPedido.Sum(d => d.Cantidad * d.PrecioUnitario),
                UltimaVenta = p.DetallesPedido
                    .OrderByDescending(d => d.Pedido.FechaPedido)
                    .Select(d => d.Pedido.FechaPedido)
                    .FirstOrDefault()
            })
            .OrderByDescending(e => e.CantidadVendida)
            .ToListAsync();
    }

    public async Task<bool> ActualizarStockAsync(int productoId, int nuevoStock)
    {
        var producto = await _dbSet.FindAsync(productoId);
        if (producto == null) return false;

        producto.Stock = nuevoStock;
        producto.FechaModificacion = DateTime.UtcNow;
        
        await _context.SaveChangesAsync();
        return true;
    }
}

// DTO para estadísticas
public class ProductoEstadistica
{
    public int ProductoId { get; set; }
    public string NombreProducto { get; set; } = string.Empty;
    public int CantidadVendida { get; set; }
    public decimal TotalVentas { get; set; }
    public DateTime? UltimaVenta { get; set; }
}
```

### Consultas complejas con LINQ
```csharp
public class ConsultasAvanzadasService
{
    private readonly TiendaDbContext _context;

    public ConsultasAvanzadasService(TiendaDbContext context)
    {
        _context = context;
    }

    // Consulta con múltiples joins y agrupaciones
    public async Task<IEnumerable<ReporteVentasPorCategoria>> ObtenerVentasPorCategoriaAsync(
        DateTime fechaInicio, DateTime fechaFin)
    {
        return await _context.Categorias
            .Select(c => new ReporteVentasPorCategoria
            {
                CategoriaId = c.Id,
                NombreCategoria = c.Nombre,
                CantidadProductos = c.Productos.Count(p => p.Activo),
                TotalVentas = c.Productos
                    .SelectMany(p => p.DetallesPedido)
                    .Where(d => d.Pedido.FechaPedido >= fechaInicio && 
                               d.Pedido.FechaPedido <= fechaFin &&
                               d.Pedido.Estado != EstadoPedido.Cancelado)
                    .Sum(d => d.Cantidad * d.PrecioUnitario),
                CantidadUnidadesVendidas = c.Productos
                    .SelectMany(p => p.DetallesPedido)
                    .Where(d => d.Pedido.FechaPedido >= fechaInicio && 
                               d.Pedido.FechaPedido <= fechaFin &&
                               d.Pedido.Estado != EstadoPedido.Cancelado)
                    .Sum(d => d.Cantidad)
            })
            .Where(r => r.TotalVentas > 0)
            .OrderByDescending(r => r.TotalVentas)
            .ToListAsync();
    }

    // Consulta con subconsultas y CTEs simuladas
    public async Task<IEnumerable<ClienteTopVentas>> ObtenerTopClientesAsync(int cantidad = 10)
    {
        var fechaLimite = DateTime.UtcNow.AddMonths(-12);

        return await _context.Clientes
            .Where(c => c.Activo && c.Pedidos.Any(p => p.FechaPedido >= fechaLimite))
            .Select(c => new ClienteTopVentas
            {
                ClienteId = c.Id,
                NombreCliente = c.Nombre,
                Email = c.Email,
                CantidadPedidos = c.Pedidos.Count(p => p.FechaPedido >= fechaLimite),
                TotalGastado = c.Pedidos
                    .Where(p => p.FechaPedido >= fechaLimite && p.Estado != EstadoPedido.Cancelado)
                    .Sum(p => p.Total),
                PromedioCompra = c.Pedidos
                    .Where(p => p.FechaPedido >= fechaLimite && p.Estado != EstadoPedido.Cancelado)
                    .Average(p => p.Total),
                UltimaCompra = c.Pedidos
                    .Where(p => p.Estado != EstadoPedido.Cancelado)
                    .Max(p => p.FechaPedido),
                CategoriasCompradas = c.Pedidos
                    .Where(p => p.FechaPedido >= fechaLimite)
                    .SelectMany(p => p.Detalles)
                    .Select(d => d.Producto.Categoria.Nombre)
                    .Distinct()
                    .Count()
            })
            .OrderByDescending(c => c.TotalGastado)
            .Take(cantidad)
            .ToListAsync();
    }

    // Consulta con window functions simuladas
    public async Task<IEnumerable<TendenciaVentas>> ObtenerTendenciaVentasAsync()
    {
        var ventasPorMes = await _context.Pedidos
            .Where(p => p.Estado != EstadoPedido.Cancelado)
            .GroupBy(p => new { 
                Año = p.FechaPedido.Year, 
                Mes = p.FechaPedido.Month 
            })
            .Select(g => new VentaMensual
            {
                Año = g.Key.Año,
                Mes = g.Key.Mes,
                TotalVentas = g.Sum(p => p.Total),
                CantidadPedidos = g.Count(),
                PromedioTicket = g.Average(p => p.Total)
            })
            .OrderBy(v => v.Año)
            .ThenBy(v => v.Mes)
            .ToListAsync();

        // Calcular tendencias en memoria (simulando window functions)
        var tendencias = new List<TendenciaVentas>();
        
        for (int i = 0; i < ventasPorMes.Count; i++)
        {
            var actual = ventasPorMes[i];
            var anterior = i > 0 ? ventasPorMes[i - 1] : null;
            
            tendencias.Add(new TendenciaVentas
            {
                Año = actual.Año,
                Mes = actual.Mes,
                TotalVentas = actual.TotalVentas,
                CantidadPedidos = actual.CantidadPedidos,
                PromedioTicket = actual.PromedioTicket,
                CrecimientoVentas = anterior != null 
                    ? ((actual.TotalVentas - anterior.TotalVentas) / anterior.TotalVentas) * 100 
                    : 0,
                CrecimientoPedidos = anterior != null
                    ? ((actual.CantidadPedidos - anterior.CantidadPedidos) / (decimal)anterior.CantidadPedidos) * 100
                    : 0
            });
        }

        return tendencias;
    }

    // Consulta con filtros dinámicos
    public async Task<IEnumerable<Producto>> BuscarProductosAvanzadoAsync(FiltroBusquedaProducto filtro)
    {
        var query = _context.Productos
            .Include(p => p.Categoria)
            .Include(p => p.Proveedor)
            .Where(p => p.Activo);

        // Aplicar filtros dinámicamente
        if (!string.IsNullOrWhiteSpace(filtro.Nombre))
        {
            query = query.Where(p => p.Nombre.Contains(filtro.Nombre));
        }

        if (filtro.CategoriaId.HasValue)
        {
            query = query.Where(p => p.CategoriaId == filtro.CategoriaId.Value);
        }

        if (filtro.PrecioMinimo.HasValue)
        {
            query = query.Where(p => p.Precio >= filtro.PrecioMinimo.Value);
        }

        if (filtro.PrecioMaximo.HasValue)
        {
            query = query.Where(p => p.Precio <= filtro.PrecioMaximo.Value);
        }

        if (filtro.StockMinimo.HasValue)
        {
            query = query.Where(p => p.Stock >= filtro.StockMinimo.Value);
        }

        if (filtro.ProveedorId.HasValue)
        {
            query = query.Where(p => p.ProveedorId == filtro.ProveedorId.Value);
        }

        // Aplicar ordenamiento
        query = filtro.OrdenarPor switch
        {
            "nombre" => filtro.OrdenDescendente 
                ? query.OrderByDescending(p => p.Nombre)
                : query.OrderBy(p => p.Nombre),
            "precio" => filtro.OrdenDescendente
                ? query.OrderByDescending(p => p.Precio)
                : query.OrderBy(p => p.Precio),
            "stock" => filtro.OrdenDescendente
                ? query.OrderByDescending(p => p.Stock)
                : query.OrderBy(p => p.Stock),
            "fecha" => filtro.OrdenDescendente
                ? query.OrderByDescending(p => p.FechaCreacion)
                : query.OrderBy(p => p.FechaCreacion),
            _ => query.OrderBy(p => p.Nombre)
        };

        // Aplicar paginación
        if (filtro.Pagina > 0 && filtro.TamañoPagina > 0)
        {
            query = query
                .Skip((filtro.Pagina - 1) * filtro.TamañoPagina)
                .Take(filtro.TamañoPagina);
        }

        return await query.ToListAsync();
    }
}

// DTOs para consultas
public class ReporteVentasPorCategoria
{
    public int CategoriaId { get; set; }
    public string NombreCategoria { get; set; } = string.Empty;
    public int CantidadProductos { get; set; }
    public decimal TotalVentas { get; set; }
    public int CantidadUnidadesVendidas { get; set; }
}

public class ClienteTopVentas
{
    public int ClienteId { get; set; }
    public string NombreCliente { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
    public int CantidadPedidos { get; set; }
    public decimal TotalGastado { get; set; }
    public decimal PromedioCompra { get; set; }
    public DateTime? UltimaCompra { get; set; }
    public int CategoriasCompradas { get; set; }
}

public class VentaMensual
{
    public int Año { get; set; }
    public int Mes { get; set; }
    public decimal TotalVentas { get; set; }
    public int CantidadPedidos { get; set; }
    public decimal PromedioTicket { get; set; }
}

public class TendenciaVentas : VentaMensual
{
    public decimal CrecimientoVentas { get; set; }
    public decimal CrecimientoPedidos { get; set; }
}

public class FiltroBusquedaProducto
{
    public string? Nombre { get; set; }
    public int? CategoriaId { get; set; }
    public decimal? PrecioMinimo { get; set; }
    public decimal? PrecioMaximo { get; set; }
    public int? StockMinimo { get; set; }
    public int? ProveedorId { get; set; }
    public string OrdenarPor { get; set; } = "nombre";
    public bool OrdenDescendente { get; set; } = false;
    public int Pagina { get; set; } = 1;
    public int TamañoPagina { get; set; } = 20;
}
```

## ⚡ Optimización de Performance

### Estrategias de optimización
```csharp
public class OptimizacionService
{
    private readonly TiendaDbContext _context;
    private readonly IMemoryCache _cache;
    private readonly ILogger<OptimizacionService> _logger;

    public OptimizacionService(
        TiendaDbContext context, 
        IMemoryCache cache, 
        ILogger<OptimizacionService> logger)
    {
        _context = context;
        _cache = cache;
        _logger = logger;
    }

    // Optimización 1: Lazy Loading vs Eager Loading
    public async Task<IEnumerable<Producto>> ObtenerProductosConCategoriaEagerAsync()
    {
        // Eager Loading: Carga todo de una vez
        return await _context.Productos
            .Include(p => p.Categoria)
            .Include(p => p.Proveedor)
            .Where(p => p.Activo)
            .ToListAsync();
    }

    public async Task<IEnumerable<Producto>> ObtenerProductosConCategoriaSelectiveAsync()
    {
        // Selective Loading: Solo los campos necesarios
        return await _context.Productos
            .Where(p => p.Activo)
            .Select(p => new Producto
            {
                Id = p.Id,
                Nombre = p.Nombre,
                Precio = p.Precio,
                Stock = p.Stock,
                Categoria = new Categoria { Nombre = p.Categoria.Nombre }
            })
            .ToListAsync();
    }

    // Optimización 2: Proyección a DTOs
    public async Task<IEnumerable<ProductoListaDto>> ObtenerProductosParaListaAsync()
    {
        return await _context.Productos
            .Where(p => p.Activo)
            .Select(p => new ProductoListaDto
            {
                Id = p.Id,
                Nombre = p.Nombre,
                Precio = p.Precio,
                Stock = p.Stock,
                NombreCategoria = p.Categoria.Nombre,
                TieneBajoStock = p.Stock < 10
            })
            .OrderBy(p => p.Nombre)
            .ToListAsync();
    }

    // Optimización 3: Paginación eficiente
    public async Task<ResultadoPaginado<ProductoListaDto>> ObtenerProductosPaginadosAsync(
        int pagina, int tamañoPagina, string? filtroNombre = null)
    {
        var query = _context.Productos
            .Where(p => p.Activo);

        if (!string.IsNullOrWhiteSpace(filtroNombre))
        {
            query = query.Where(p => p.Nombre.Contains(filtroNombre));
        }

        var totalRegistros = await query.CountAsync();

        var productos = await query
            .Select(p => new ProductoListaDto
            {
                Id = p.Id,
                Nombre = p.Nombre,
                Precio = p.Precio,
                Stock = p.Stock,
                NombreCategoria = p.Categoria.Nombre
            })
            .OrderBy(p => p.Nombre)
            .Skip((pagina - 1) * tamañoPagina)
            .Take(tamañoPagina)
            .ToListAsync();

        return new ResultadoPaginado<ProductoListaDto>
        {
            Datos = productos,
            PaginaActual = pagina,
            TamañoPagina = tamañoPagina,
            TotalRegistros = totalRegistros,
            TotalPaginas = (int)Math.Ceiling((double)totalRegistros / tamañoPagina)
        };
    }

    // Optimización 4: Cache de consultas frecuentes
    public async Task<IEnumerable<CategoriaDto>> ObtenerCategoriasConCacheAsync()
    {
        const string cacheKey = "categorias_activas";
        
        if (_cache.TryGetValue(cacheKey, out IEnumerable<CategoriaDto>? categoriasCache))
        {
            _logger.LogInformation("Categorías obtenidas desde cache");
            return categoriasCache!;
        }

        var categorias = await _context.Categorias
            .Where(c => c.Activa)
            .Select(c => new CategoriaDto
            {
                Id = c.Id,
                Nombre = c.Nombre,
                CantidadProductos = c.Productos.Count(p => p.Activo)
            })
            .OrderBy(c => c.Nombre)
            .ToListAsync();

        var cacheOptions = new MemoryCacheEntryOptions
        {
            AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(30),
            SlidingExpiration = TimeSpan.FromMinutes(5)
        };

        _cache.Set(cacheKey, categorias, cacheOptions);
        _logger.LogInformation("Categorías cargadas en cache");

        return categorias;
    }

    // Optimización 5: Bulk operations
    public async Task ActualizarPreciosEnLoteAsync(
        Dictionary<int, decimal> actualizacionesPrecios)
    {
        var productosIds = actualizacionesPrecios.Keys.ToList();
        
        var productos = await _context.Productos
            .Where(p => productosIds.Contains(p.Id))
            .ToListAsync();

        foreach (var producto in productos)
        {
            if (actualizacionesPrecios.TryGetValue(producto.Id, out decimal nuevoPrecio))
            {
                producto.Precio = nuevoPrecio;
                producto.FechaModificacion = DateTime.UtcNow;
            }
        }

        await _context.SaveChangesAsync();
        _logger.LogInformation("Actualizados precios de {Cantidad} productos", productos.Count);
    }

    // Optimización 6: Raw SQL para consultas complejas
    public async Task<IEnumerable<EstadisticaVentasCompleja>> ObtenerEstadisticasComplejasSQLAsync()
    {
        var sql = @"
            SELECT 
                c.Id as CategoriaId,
                c.Nombre as NombreCategoria,
                COUNT(DISTINCT p.Id) as CantidadProductos,
                COUNT(DISTINCT pe.Id) as CantidadPedidos,
                SUM(dp.Cantidad * dp.PrecioUnitario) as TotalVentas,
                AVG(dp.Cantidad * dp.PrecioUnitario) as PromedioVenta,
                MAX(pe.FechaPedido) as UltimaVenta
            FROM Categorias c
            LEFT JOIN Productos p ON c.Id = p.CategoriaId AND p.Activo = 1
            LEFT JOIN DetallesPedido dp ON p.Id = dp.ProductoId
            LEFT JOIN Pedidos pe ON dp.PedidoId = pe.Id AND pe.Estado != 'Cancelado'
            WHERE c.Activa = 1
            GROUP BY c.Id, c.Nombre
            HAVING SUM(dp.Cantidad * dp.PrecioUnitario) > 0
            ORDER BY TotalVentas DESC";

        return await _context.Database
            .SqlQueryRaw<EstadisticaVentasCompleja>(sql)
            .ToListAsync();
    }

    // Optimización 7: Transacciones optimizadas
    public async Task ProcesarPedidoOptimizadoAsync(PedidoCompleto pedidoCompleto)
    {
        using var transaction = await _context.Database.BeginTransactionAsync();
        
        try
        {
            // 1. Verificar stock en una sola consulta
            var productosIds = pedidoCompleto.Detalles.Select(d => d.ProductoId).ToList();
            var productos = await _context.Productos
                .Where(p => productosIds.Contains(p.Id))
                .Select(p => new { p.Id, p.Stock, p.Precio })
                .ToListAsync();

            // 2. Validar stock
            foreach (var detalle in pedidoCompleto.Detalles)
            {
                var producto = productos.FirstOrDefault(p => p.Id == detalle.ProductoId);
                if (producto == null || producto.Stock < detalle.Cantidad)
                {
                    throw new InvalidOperationException($"Stock insuficiente para producto {detalle.ProductoId}");
                }
            }

            // 3. Actualizar stock en lote
            await _context.Database.ExecuteSqlRawAsync(@"
                UPDATE Productos 
                SET Stock = Stock - @cantidad,
                    FechaModificacion = GETUTCDATE()
                WHERE Id = @productoId",
                pedidoCompleto.Detalles.Select(d => new { cantidad = d.Cantidad, productoId = d.ProductoId }));

            // 4. Crear pedido
            var pedido = new Pedido
            {
                ClienteId = pedidoCompleto.ClienteId,
                FechaPedido = DateTime.UtcNow,
                Estado = EstadoPedido.Pendiente,
                Total = pedidoCompleto.Detalles.Sum(d => d.Cantidad * productos.First(p => p.Id == d.ProductoId).Precio)
            };

            _context.Pedidos.Add(pedido);
            await _context.SaveChangesAsync();

            // 5. Crear detalles en lote
            var detalles = pedidoCompleto.Detalles.Select(d => new DetallePedido
            {
                PedidoId = pedido.Id,
                ProductoId = d.ProductoId,
                Cantidad = d.Cantidad,
                PrecioUnitario = productos.First(p => p.Id == d.ProductoId).Precio
            }).ToList();

            _context.DetallesPedido.AddRange(detalles);
            await _context.SaveChangesAsync();

            await transaction.CommitAsync();
            
            _logger.LogInformation("Pedido {PedidoId} procesado exitosamente", pedido.Id);
        }
        catch
        {
            await transaction.RollbackAsync();
            throw;
        }
    }
}

// DTOs para optimización
public class ProductoListaDto
{
    public int Id { get; set; }
    public string Nombre { get; set; } = string.Empty;
    public decimal Precio { get; set; }
    public int Stock { get; set; }
    public string NombreCategoria { get; set; } = string.Empty;
    public bool TieneBajoStock { get; set; }
}

public class CategoriaDto
{
    public int Id { get; set; }
    public string Nombre { get; set; } = string.Empty;
    public int CantidadProductos { get; set; }
}

public class ResultadoPaginado<T>
{
    public IEnumerable<T> Datos { get; set; } = new List<T>();
    public int PaginaActual { get; set; }
    public int TamañoPagina { get; set; }
    public int TotalRegistros { get; set; }
    public int TotalPaginas { get; set; }
    public bool TienePaginaAnterior => PaginaActual > 1;
    public bool TienePaginaSiguiente => PaginaActual < TotalPaginas;
}

public class EstadisticaVentasCompleja
{
    public int CategoriaId { get; set; }
    public string NombreCategoria { get; set; } = string.Empty;
    public int CantidadProductos { get; set; }
    public int CantidadPedidos { get; set; }
    public decimal TotalVentas { get; set; }
    public decimal PromedioVenta { get; set; }
    public DateTime? UltimaVenta { get; set; }
}

public class PedidoCompleto
{
    public int ClienteId { get; set; }
    public List<DetallePedidoDto> Detalles { get; set; } = new();
}

public class DetallePedidoDto
{
    public int ProductoId { get; set; }
    public int Cantidad { get; set; }
}
```

---

**Anterior**: [04. Desarrollo Web con ASP.NET Core](./04-web.md) | **Siguiente**: [06. Documentación y Calidad](./06-documentacion.md) | **Inicio**: [README](../README.md)
