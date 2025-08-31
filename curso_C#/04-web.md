# 04. Desarrollo Web con ASP.NET Core

## 🌐 Introducción a ASP.NET Core

### Arquitectura y conceptos fundamentales
ASP.NET Core es un framework web multiplataforma, de alto rendimiento y open source para construir aplicaciones web modernas y servicios conectados a la nube.

**Características principales:**
- **Multiplataforma**: Windows, macOS, Linux
- **Alto rendimiento**: Uno de los frameworks web más rápidos
- **Open source**: Código abierto y desarrollo en GitHub
- **Modular**: Sistema de middleware configurable
- **Dependency Injection**: Inyección de dependencias nativa
- **Hosting flexible**: IIS, Nginx, Apache, Docker, Cloud

### Creación del primer proyecto
```bash
# Crear nuevo proyecto Web API
dotnet new webapi -n MiPrimeraAPI

# Crear proyecto MVC
dotnet new mvc -n MiPrimerMVC

# Crear proyecto Blazor Server
dotnet new blazorserver -n MiPrimerBlazor

# Ejecutar la aplicación
cd MiPrimeraAPI
dotnet run
```

### Estructura básica del proyecto
```
MiPrimeraAPI/
├── Controllers/           # Controladores de la API
├── Models/               # Modelos de datos
├── Services/             # Servicios de negocio
├── Data/                 # Contexto de base de datos
├── Properties/           # Configuración del proyecto
├── wwwroot/             # Archivos estáticos
├── appsettings.json     # Configuración de la aplicación
├── Program.cs           # Punto de entrada y configuración
└── MiPrimeraAPI.csproj  # Archivo del proyecto
```

## ⚙️ Configuración y Startup

### Program.cs moderno (ASP.NET Core 6+)
```csharp
using Microsoft.EntityFrameworkCore;
using MiPrimeraAPI.Data;
using MiPrimeraAPI.Services;

var builder = WebApplication.CreateBuilder(args);

// Configurar servicios
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Configurar Entity Framework
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

// Configurar servicios personalizados
builder.Services.AddScoped<IProductoService, ProductoService>();
builder.Services.AddScoped<IEmailService, EmailService>();

// Configurar CORS
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowSpecificOrigin", policy =>
    {
        policy.WithOrigins("https://localhost:3000", "https://miapp.com")
              .AllowAnyHeader()
              .AllowAnyMethod();
    });
});

// Configurar autenticación JWT
builder.Services.AddAuthentication("Bearer")
    .AddJwtBearer("Bearer", options =>
    {
        options.Authority = "https://localhost:5001";
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateAudience = false
        };
    });

var app = builder.Build();

// Configurar pipeline de middleware
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
    app.UseDeveloperExceptionPage();
}
else
{
    app.UseExceptionHandler("/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseCors("AllowSpecificOrigin");
app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();

// Seed de datos inicial
using (var scope = app.Services.CreateScope())
{
    var context = scope.ServiceProvider.GetRequiredService<ApplicationDbContext>();
    await SeedData.Initialize(context);
}

app.Run();
```

### Configuración con appsettings.json
```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=MiAplicacionDB;Trusted_Connection=true;MultipleActiveResultSets=true",
    "Redis": "localhost:6379"
  },
  "JwtSettings": {
    "SecretKey": "mi-clave-super-secreta-que-debe-ser-larga",
    "Issuer": "https://miapi.com",
    "Audience": "https://miapp.com",
    "ExpirationInMinutes": 60
  },
  "EmailSettings": {
    "SmtpServer": "smtp.gmail.com",
    "SmtpPort": 587,
    "SenderEmail": "noreply@miapp.com",
    "SenderName": "Mi Aplicación"
  },
  "AllowedHosts": "*"
}
```

## 🎮 Controllers y Actions

### Controlador básico
```csharp
using Microsoft.AspNetCore.Mvc;
using MiPrimeraAPI.Models;
using MiPrimeraAPI.Services;

[ApiController]
[Route("api/[controller]")]
public class ProductosController : ControllerBase
{
    private readonly IProductoService _productoService;
    private readonly ILogger<ProductosController> _logger;

    public ProductosController(IProductoService productoService, ILogger<ProductosController> logger)
    {
        _productoService = productoService;
        _logger = logger;
    }

    /// <summary>
    /// Obtiene todos los productos
    /// </summary>
    /// <returns>Lista de productos</returns>
    [HttpGet]
    [ProducesResponseType(typeof(IEnumerable<ProductoDto>), StatusCodes.Status200OK)]
    public async Task<ActionResult<IEnumerable<ProductoDto>>> GetProductos()
    {
        _logger.LogInformation("Obteniendo todos los productos");
        
        var productos = await _productoService.ObtenerTodosAsync();
        return Ok(productos);
    }

    /// <summary>
    /// Obtiene un producto específico por ID
    /// </summary>
    /// <param name="id">ID del producto</param>
    /// <returns>Producto encontrado</returns>
    [HttpGet("{id}")]
    [ProducesResponseType(typeof(ProductoDto), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<ActionResult<ProductoDto>> GetProducto(int id)
    {
        _logger.LogInformation("Obteniendo producto con ID: {ProductoId}", id);

        var producto = await _productoService.ObtenerPorIdAsync(id);
        
        if (producto == null)
        {
            _logger.LogWarning("Producto con ID {ProductoId} no encontrado", id);
            return NotFound($"Producto con ID {id} no encontrado");
        }

        return Ok(producto);
    }

    /// <summary>
    /// Crea un nuevo producto
    /// </summary>
    /// <param name="productoDto">Datos del producto a crear</param>
    /// <returns>Producto creado</returns>
    [HttpPost]
    [ProducesResponseType(typeof(ProductoDto), StatusCodes.Status201Created)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    public async Task<ActionResult<ProductoDto>> CrearProducto(CrearProductoDto productoDto)
    {
        _logger.LogInformation("Creando nuevo producto: {ProductoNombre}", productoDto.Nombre);

        if (!ModelState.IsValid)
        {
            return BadRequest(ModelState);
        }

        var productoCreado = await _productoService.CrearAsync(productoDto);
        
        return CreatedAtAction(
            nameof(GetProducto), 
            new { id = productoCreado.Id }, 
            productoCreado);
    }

    /// <summary>
    /// Actualiza un producto existente
    /// </summary>
    /// <param name="id">ID del producto</param>
    /// <param name="productoDto">Nuevos datos del producto</param>
    /// <returns>Producto actualizado</returns>
    [HttpPut("{id}")]
    [ProducesResponseType(typeof(ProductoDto), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<ActionResult<ProductoDto>> ActualizarProducto(int id, ActualizarProductoDto productoDto)
    {
        if (id != productoDto.Id)
        {
            return BadRequest("El ID del producto no coincide");
        }

        _logger.LogInformation("Actualizando producto con ID: {ProductoId}", id);

        var productoActualizado = await _productoService.ActualizarAsync(productoDto);
        
        if (productoActualizado == null)
        {
            return NotFound($"Producto con ID {id} no encontrado");
        }

        return Ok(productoActualizado);
    }

    /// <summary>
    /// Elimina un producto
    /// </summary>
    /// <param name="id">ID del producto a eliminar</param>
    /// <returns>Confirmación de eliminación</returns>
    [HttpDelete("{id}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<ActionResult> EliminarProducto(int id)
    {
        _logger.LogInformation("Eliminando producto con ID: {ProductoId}", id);

        var eliminado = await _productoService.EliminarAsync(id);
        
        if (!eliminado)
        {
            return NotFound($"Producto con ID {id} no encontrado");
        }

        return NoContent();
    }

    /// <summary>
    /// Busca productos por criterio
    /// </summary>
    /// <param name="termino">Término de búsqueda</param>
    /// <param name="categoria">Filtro por categoría</param>
    /// <param name="precioMin">Precio mínimo</param>
    /// <param name="precioMax">Precio máximo</param>
    /// <returns>Productos que coinciden con el criterio</returns>
    [HttpGet("buscar")]
    [ProducesResponseType(typeof(IEnumerable<ProductoDto>), StatusCodes.Status200OK)]
    public async Task<ActionResult<IEnumerable<ProductoDto>>> BuscarProductos(
        [FromQuery] string? termino = null,
        [FromQuery] string? categoria = null,
        [FromQuery] decimal? precioMin = null,
        [FromQuery] decimal? precioMax = null)
    {
        _logger.LogInformation("Buscando productos con término: {Termino}", termino);

        var criterio = new BusquedaProductoDto
        {
            Termino = termino,
            Categoria = categoria,
            PrecioMinimo = precioMin,
            PrecioMaximo = precioMax
        };

        var productos = await _productoService.BuscarAsync(criterio);
        return Ok(productos);
    }
}
```

### Modelos y DTOs
```csharp
// Modelo de entidad
public class Producto
{
    public int Id { get; set; }
    public string Nombre { get; set; } = string.Empty;
    public string Descripcion { get; set; } = string.Empty;
    public decimal Precio { get; set; }
    public int Stock { get; set; }
    public string Categoria { get; set; } = string.Empty;
    public DateTime FechaCreacion { get; set; }
    public DateTime? FechaActualizacion { get; set; }
    public bool Activo { get; set; } = true;
}

// DTO para transferencia de datos
public class ProductoDto
{
    public int Id { get; set; }
    public string Nombre { get; set; } = string.Empty;
    public string Descripcion { get; set; } = string.Empty;
    public decimal Precio { get; set; }
    public int Stock { get; set; }
    public string Categoria { get; set; } = string.Empty;
    public DateTime FechaCreacion { get; set; }
    public bool DisponibleEnStock => Stock > 0;
}

// DTO para crear producto
public class CrearProductoDto
{
    [Required(ErrorMessage = "El nombre es requerido")]
    [StringLength(100, ErrorMessage = "El nombre no puede exceder 100 caracteres")]
    public string Nombre { get; set; } = string.Empty;

    [StringLength(500, ErrorMessage = "La descripción no puede exceder 500 caracteres")]
    public string Descripcion { get; set; } = string.Empty;

    [Required(ErrorMessage = "El precio es requerido")]
    [Range(0.01, double.MaxValue, ErrorMessage = "El precio debe ser mayor que cero")]
    public decimal Precio { get; set; }

    [Required(ErrorMessage = "El stock es requerido")]
    [Range(0, int.MaxValue, ErrorMessage = "El stock no puede ser negativo")]
    public int Stock { get; set; }

    [Required(ErrorMessage = "La categoría es requerida")]
    [StringLength(50, ErrorMessage = "La categoría no puede exceder 50 caracteres")]
    public string Categoria { get; set; } = string.Empty;
}

// DTO para actualizar producto
public class ActualizarProductoDto : CrearProductoDto
{
    [Required]
    public int Id { get; set; }
}

// DTO para búsqueda
public class BusquedaProductoDto
{
    public string? Termino { get; set; }
    public string? Categoria { get; set; }
    public decimal? PrecioMinimo { get; set; }
    public decimal? PrecioMaximo { get; set; }
}
```

## 🔄 Routing y Middleware

### Configuración de rutas
```csharp
// Rutas convencionales en un controlador MVC
[Route("tienda")]
public class TiendaController : Controller
{
    // GET: /tienda
    [Route("")]
    [Route("inicio")]
    public IActionResult Index()
    {
        return View();
    }

    // GET: /tienda/productos
    // GET: /tienda/productos/categoria/electronicos
    [Route("productos")]
    [Route("productos/categoria/{categoria}")]
    public IActionResult Productos(string? categoria = null)
    {
        // Lógica para mostrar productos
        return View();
    }

    // GET: /tienda/producto/123
    [Route("producto/{id:int}")]
    public IActionResult DetalleProducto(int id)
    {
        // Lógica para mostrar detalle
        return View();
    }

    // POST: /tienda/producto/123/comentario
    [HttpPost]
    [Route("producto/{productId:int}/comentario")]
    public IActionResult AgregarComentario(int productId, [FromBody] ComentarioDto comentario)
    {
        // Lógica para agregar comentario
        return Json(new { success = true });
    }
}
```

### Middleware personalizado
```csharp
// Middleware para logging de requests
public class RequestLoggingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<RequestLoggingMiddleware> _logger;

    public RequestLoggingMiddleware(RequestDelegate next, ILogger<RequestLoggingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        var startTime = DateTime.UtcNow;
        var requestId = Guid.NewGuid().ToString();

        // Log request
        _logger.LogInformation("Request {RequestId} started: {Method} {Path}",
            requestId, context.Request.Method, context.Request.Path);

        // Agregar RequestId a los headers de respuesta
        context.Response.Headers.Add("X-Request-ID", requestId);

        try
        {
            await _next(context);
        }
        finally
        {
            var duration = DateTime.UtcNow - startTime;
            _logger.LogInformation("Request {RequestId} completed in {Duration}ms with status {StatusCode}",
                requestId, duration.TotalMilliseconds, context.Response.StatusCode);
        }
    }
}

// Middleware para manejo de errores globales
public class GlobalExceptionMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<GlobalExceptionMiddleware> _logger;

    public GlobalExceptionMiddleware(RequestDelegate next, ILogger<GlobalExceptionMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            await _next(context);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "An unhandled exception occurred");
            await HandleExceptionAsync(context, ex);
        }
    }

    private static async Task HandleExceptionAsync(HttpContext context, Exception exception)
    {
        context.Response.ContentType = "application/json";
        
        var response = new
        {
            error = new
            {
                message = "An error occurred while processing your request",
                type = exception.GetType().Name,
                timestamp = DateTime.UtcNow
            }
        };

        context.Response.StatusCode = exception switch
        {
            ArgumentException => StatusCodes.Status400BadRequest,
            UnauthorizedAccessException => StatusCodes.Status401Unauthorized,
            NotImplementedException => StatusCodes.Status501NotImplemented,
            _ => StatusCodes.Status500InternalServerError
        };

        var jsonResponse = JsonSerializer.Serialize(response);
        await context.Response.WriteAsync(jsonResponse);
    }
}

// Registrar middleware en Program.cs
app.UseMiddleware<RequestLoggingMiddleware>();
app.UseMiddleware<GlobalExceptionMiddleware>();
```

## 💉 Dependency Injection

### Configuración de servicios
```csharp
// Interface del servicio
public interface IProductoService
{
    Task<IEnumerable<ProductoDto>> ObtenerTodosAsync();
    Task<ProductoDto?> ObtenerPorIdAsync(int id);
    Task<ProductoDto> CrearAsync(CrearProductoDto producto);
    Task<ProductoDto?> ActualizarAsync(ActualizarProductoDto producto);
    Task<bool> EliminarAsync(int id);
    Task<IEnumerable<ProductoDto>> BuscarAsync(BusquedaProductoDto criterio);
}

// Implementación del servicio
public class ProductoService : IProductoService
{
    private readonly ApplicationDbContext _context;
    private readonly IMapper _mapper;
    private readonly ILogger<ProductoService> _logger;
    private readonly ICacheService _cache;

    public ProductoService(
        ApplicationDbContext context,
        IMapper mapper,
        ILogger<ProductoService> logger,
        ICacheService cache)
    {
        _context = context;
        _mapper = mapper;
        _logger = logger;
        _cache = cache;
    }

    public async Task<IEnumerable<ProductoDto>> ObtenerTodosAsync()
    {
        const string cacheKey = "todos_productos";
        
        var productosCache = await _cache.GetAsync<IEnumerable<ProductoDto>>(cacheKey);
        if (productosCache != null)
        {
            _logger.LogInformation("Productos obtenidos desde cache");
            return productosCache;
        }

        var productos = await _context.Productos
            .Where(p => p.Activo)
            .OrderBy(p => p.Nombre)
            .ToListAsync();

        var productosDto = _mapper.Map<IEnumerable<ProductoDto>>(productos);
        
        await _cache.SetAsync(cacheKey, productosDto, TimeSpan.FromMinutes(10));
        
        _logger.LogInformation("Obtenidos {Count} productos desde base de datos", productos.Count);
        return productosDto;
    }

    public async Task<ProductoDto?> ObtenerPorIdAsync(int id)
    {
        var cacheKey = $"producto_{id}";
        
        var productoCache = await _cache.GetAsync<ProductoDto>(cacheKey);
        if (productoCache != null)
        {
            return productoCache;
        }

        var producto = await _context.Productos
            .FirstOrDefaultAsync(p => p.Id == id && p.Activo);

        if (producto == null)
        {
            return null;
        }

        var productoDto = _mapper.Map<ProductoDto>(producto);
        await _cache.SetAsync(cacheKey, productoDto, TimeSpan.FromMinutes(5));
        
        return productoDto;
    }

    public async Task<ProductoDto> CrearAsync(CrearProductoDto crearProductoDto)
    {
        var producto = _mapper.Map<Producto>(crearProductoDto);
        producto.FechaCreacion = DateTime.UtcNow;

        _context.Productos.Add(producto);
        await _context.SaveChangesAsync();

        // Invalidar cache
        await _cache.RemoveAsync("todos_productos");

        _logger.LogInformation("Producto creado con ID: {ProductoId}", producto.Id);
        
        return _mapper.Map<ProductoDto>(producto);
    }

    public async Task<ProductoDto?> ActualizarAsync(ActualizarProductoDto actualizarProductoDto)
    {
        var producto = await _context.Productos
            .FirstOrDefaultAsync(p => p.Id == actualizarProductoDto.Id && p.Activo);

        if (producto == null)
        {
            return null;
        }

        _mapper.Map(actualizarProductoDto, producto);
        producto.FechaActualizacion = DateTime.UtcNow;

        await _context.SaveChangesAsync();

        // Invalidar cache
        await _cache.RemoveAsync("todos_productos");
        await _cache.RemoveAsync($"producto_{producto.Id}");

        _logger.LogInformation("Producto actualizado: {ProductoId}", producto.Id);
        
        return _mapper.Map<ProductoDto>(producto);
    }

    public async Task<bool> EliminarAsync(int id)
    {
        var producto = await _context.Productos
            .FirstOrDefaultAsync(p => p.Id == id && p.Activo);

        if (producto == null)
        {
            return false;
        }

        producto.Activo = false; // Soft delete
        producto.FechaActualizacion = DateTime.UtcNow;

        await _context.SaveChangesAsync();

        // Invalidar cache
        await _cache.RemoveAsync("todos_productos");
        await _cache.RemoveAsync($"producto_{id}");

        _logger.LogInformation("Producto eliminado (soft delete): {ProductoId}", id);
        
        return true;
    }

    public async Task<IEnumerable<ProductoDto>> BuscarAsync(BusquedaProductoDto criterio)
    {
        var query = _context.Productos.Where(p => p.Activo);

        if (!string.IsNullOrWhiteSpace(criterio.Termino))
        {
            query = query.Where(p => p.Nombre.Contains(criterio.Termino) || 
                                   p.Descripcion.Contains(criterio.Termino));
        }

        if (!string.IsNullOrWhiteSpace(criterio.Categoria))
        {
            query = query.Where(p => p.Categoria == criterio.Categoria);
        }

        if (criterio.PrecioMinimo.HasValue)
        {
            query = query.Where(p => p.Precio >= criterio.PrecioMinimo.Value);
        }

        if (criterio.PrecioMaximo.HasValue)
        {
            query = query.Where(p => p.Precio <= criterio.PrecioMaximo.Value);
        }

        var productos = await query
            .OrderBy(p => p.Nombre)
            .ToListAsync();

        _logger.LogInformation("Búsqueda realizada, encontrados {Count} productos", productos.Count);
        
        return _mapper.Map<IEnumerable<ProductoDto>>(productos);
    }
}

// Registrar servicios en Program.cs
builder.Services.AddScoped<IProductoService, ProductoService>();
builder.Services.AddScoped<ICacheService, RedisCacheService>();
builder.Services.AddAutoMapper(typeof(Program));
```

## 🔧 Filtros y Atributos

### Filtros de acción personalizados
```csharp
// Filtro para validación de modelo
public class ValidateModelAttribute : ActionFilterAttribute
{
    public override void OnActionExecuting(ActionExecutingContext context)
    {
        if (!context.ModelState.IsValid)
        {
            var errors = context.ModelState
                .Where(x => x.Value?.Errors.Count > 0)
                .ToDictionary(
                    kvp => kvp.Key,
                    kvp => kvp.Value?.Errors.Select(e => e.ErrorMessage).ToArray()
                );

            var result = new
            {
                message = "Validation failed",
                errors = errors
            };

            context.Result = new BadRequestObjectResult(result);
        }
    }
}

// Filtro para logging de performance
public class PerformanceLoggingAttribute : ActionFilterAttribute
{
    private readonly ILogger<PerformanceLoggingAttribute> _logger;
    private readonly int _warningThresholdMs;

    public PerformanceLoggingAttribute(int warningThresholdMs = 1000)
    {
        _warningThresholdMs = warningThresholdMs;
    }

    public override void OnActionExecuting(ActionExecutingContext context)
    {
        context.HttpContext.Items["ActionStartTime"] = DateTime.UtcNow;
    }

    public override void OnActionExecuted(ActionExecutedContext context)
    {
        if (context.HttpContext.Items["ActionStartTime"] is DateTime startTime)
        {
            var duration = DateTime.UtcNow - startTime;
            var actionName = $"{context.Controller.GetType().Name}.{context.ActionDescriptor.DisplayName}";

            if (duration.TotalMilliseconds > _warningThresholdMs)
            {
                var logger = context.HttpContext.RequestServices
                    .GetRequiredService<ILogger<PerformanceLoggingAttribute>>();
                    
                logger.LogWarning("Slow action detected: {ActionName} took {Duration}ms",
                    actionName, duration.TotalMilliseconds);
            }
        }
    }
}

// Filtro para cache de respuestas
public class CacheResponseAttribute : ActionFilterAttribute
{
    private readonly int _durationInSeconds;

    public CacheResponseAttribute(int durationInSeconds = 300) // 5 minutos por defecto
    {
        _durationInSeconds = durationInSeconds;
    }

    public override void OnActionExecuted(ActionExecutedContext context)
    {
        if (context.Result is ObjectResult result && result.StatusCode == 200)
        {
            context.HttpContext.Response.Headers.Add("Cache-Control", 
                $"public, max-age={_durationInSeconds}");
        }
    }
}

// Uso de filtros en controladores
[ApiController]
[Route("api/[controller]")]
[ValidateModel]
[PerformanceLogging(500)] // Warning si toma más de 500ms
public class ProductosOptimizadoController : ControllerBase
{
    private readonly IProductoService _productoService;

    public ProductosOptimizadoController(IProductoService productoService)
    {
        _productoService = productoService;
    }

    [HttpGet]
    [CacheResponse(600)] // Cache por 10 minutos
    public async Task<ActionResult<IEnumerable<ProductoDto>>> GetProductos()
    {
        var productos = await _productoService.ObtenerTodosAsync();
        return Ok(productos);
    }

    [HttpPost]
    // ValidateModel se aplica automáticamente por el filtro global
    public async Task<ActionResult<ProductoDto>> CrearProducto(CrearProductoDto producto)
    {
        var productoCreado = await _productoService.CrearAsync(producto);
        return CreatedAtAction(nameof(GetProducto), new { id = productoCreado.Id }, productoCreado);
    }
}
```

## 📋 Proyecto Integrador: API de E-commerce

### Estructura del proyecto completo
```csharp
// Models/Entities
public class Usuario
{
    public int Id { get; set; }
    public string Email { get; set; } = string.Empty;
    public string NombreCompleto { get; set; } = string.Empty;
    public string PasswordHash { get; set; } = string.Empty;
    public DateTime FechaRegistro { get; set; }
    public bool Activo { get; set; } = true;
    
    // Relaciones
    public virtual ICollection<Pedido> Pedidos { get; set; } = new List<Pedido>();
}

public class Pedido
{
    public int Id { get; set; }
    public int UsuarioId { get; set; }
    public DateTime FechaPedido { get; set; }
    public EstadoPedido Estado { get; set; }
    public decimal Total { get; set; }
    public string DireccionEnvio { get; set; } = string.Empty;
    
    // Relaciones
    public virtual Usuario Usuario { get; set; } = null!;
    public virtual ICollection<DetallePedido> Detalles { get; set; } = new List<DetallePedido>();
}

public class DetallePedido
{
    public int Id { get; set; }
    public int PedidoId { get; set; }
    public int ProductoId { get; set; }
    public int Cantidad { get; set; }
    public decimal PrecioUnitario { get; set; }
    
    // Relaciones
    public virtual Pedido Pedido { get; set; } = null!;
    public virtual Producto Producto { get; set; } = null!;
}

public enum EstadoPedido
{
    Pendiente,
    Confirmado,
    Enviado,
    Entregado,
    Cancelado
}

// DbContext
public class EcommerceDbContext : DbContext
{
    public EcommerceDbContext(DbContextOptions<EcommerceDbContext> options) : base(options) { }

    public DbSet<Usuario> Usuarios { get; set; }
    public DbSet<Producto> Productos { get; set; }
    public DbSet<Pedido> Pedidos { get; set; }
    public DbSet<DetallePedido> DetallesPedido { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Configurar Usuario
        modelBuilder.Entity<Usuario>(entity =>
        {
            entity.HasKey(e => e.Id);
            entity.Property(e => e.Email).HasMaxLength(255).IsRequired();
            entity.Property(e => e.NombreCompleto).HasMaxLength(200).IsRequired();
            entity.HasIndex(e => e.Email).IsUnique();
        });

        // Configurar Producto
        modelBuilder.Entity<Producto>(entity =>
        {
            entity.HasKey(e => e.Id);
            entity.Property(e => e.Nombre).HasMaxLength(200).IsRequired();
            entity.Property(e => e.Precio).HasPrecision(18, 2);
        });

        // Configurar Pedido
        modelBuilder.Entity<Pedido>(entity =>
        {
            entity.HasKey(e => e.Id);
            entity.Property(e => e.Total).HasPrecision(18, 2);
            entity.HasOne(p => p.Usuario)
                  .WithMany(u => u.Pedidos)
                  .HasForeignKey(p => p.UsuarioId);
        });

        // Configurar DetallePedido
        modelBuilder.Entity<DetallePedido>(entity =>
        {
            entity.HasKey(e => e.Id);
            entity.Property(e => e.PrecioUnitario).HasPrecision(18, 2);
            entity.HasOne(d => d.Pedido)
                  .WithMany(p => p.Detalles)
                  .HasForeignKey(d => d.PedidoId);
            entity.HasOne(d => d.Producto)
                  .WithMany()
                  .HasForeignKey(d => d.ProductoId);
        });
    }
}

// Servicio de Pedidos
public interface IPedidoService
{
    Task<PedidoDto> CrearPedidoAsync(CrearPedidoDto pedido);
    Task<IEnumerable<PedidoDto>> ObtenerPedidosUsuarioAsync(int usuarioId);
    Task<PedidoDto?> ObtenerPedidoPorIdAsync(int pedidoId);
    Task<bool> ActualizarEstadoPedidoAsync(int pedidoId, EstadoPedido nuevoEstado);
}

public class PedidoService : IPedidoService
{
    private readonly EcommerceDbContext _context;
    private readonly IProductoService _productoService;
    private readonly IEmailService _emailService;
    private readonly ILogger<PedidoService> _logger;

    public PedidoService(
        EcommerceDbContext context,
        IProductoService productoService,
        IEmailService emailService,
        ILogger<PedidoService> logger)
    {
        _context = context;
        _productoService = productoService;
        _emailService = emailService;
        _logger = logger;
    }

    public async Task<PedidoDto> CrearPedidoAsync(CrearPedidoDto crearPedidoDto)
    {
        using var transaction = await _context.Database.BeginTransactionAsync();
        
        try
        {
            // Validar stock de productos
            var detallesValidados = new List<DetallePedido>();
            decimal total = 0;

            foreach (var detalle in crearPedidoDto.Detalles)
            {
                var producto = await _context.Productos
                    .FirstOrDefaultAsync(p => p.Id == detalle.ProductoId);

                if (producto == null)
                    throw new ArgumentException($"Producto {detalle.ProductoId} no encontrado");

                if (producto.Stock < detalle.Cantidad)
                    throw new InvalidOperationException($"Stock insuficiente para producto {producto.Nombre}");

                // Actualizar stock
                producto.Stock -= detalle.Cantidad;

                var detallePedido = new DetallePedido
                {
                    ProductoId = detalle.ProductoId,
                    Cantidad = detalle.Cantidad,
                    PrecioUnitario = producto.Precio
                };

                detallesValidados.Add(detallePedido);
                total += detallePedido.Cantidad * detallePedido.PrecioUnitario;
            }

            // Crear pedido
            var pedido = new Pedido
            {
                UsuarioId = crearPedidoDto.UsuarioId,
                FechaPedido = DateTime.UtcNow,
                Estado = EstadoPedido.Pendiente,
                Total = total,
                DireccionEnvio = crearPedidoDto.DireccionEnvio,
                Detalles = detallesValidados
            };

            _context.Pedidos.Add(pedido);
            await _context.SaveChangesAsync();
            await transaction.CommitAsync();

            // Enviar email de confirmación
            await _emailService.EnviarConfirmacionPedidoAsync(pedido.Id);

            _logger.LogInformation("Pedido {PedidoId} creado exitosamente para usuario {UsuarioId}", 
                pedido.Id, pedido.UsuarioId);

            return new PedidoDto
            {
                Id = pedido.Id,
                FechaPedido = pedido.FechaPedido,
                Estado = pedido.Estado,
                Total = pedido.Total,
                DireccionEnvio = pedido.DireccionEnvio
            };
        }
        catch
        {
            await transaction.RollbackAsync();
            throw;
        }
    }

    // Implementar otros métodos...
}

// Controller de Pedidos
[ApiController]
[Route("api/[controller]")]
[Authorize] // Requiere autenticación
public class PedidosController : ControllerBase
{
    private readonly IPedidoService _pedidoService;

    public PedidosController(IPedidoService pedidoService)
    {
        _pedidoService = pedidoService;
    }

    [HttpPost]
    public async Task<ActionResult<PedidoDto>> CrearPedido(CrearPedidoDto pedido)
    {
        var pedidoCreado = await _pedidoService.CrearPedidoAsync(pedido);
        return CreatedAtAction(nameof(ObtenerPedido), new { id = pedidoCreado.Id }, pedidoCreado);
    }

    [HttpGet("{id}")]
    public async Task<ActionResult<PedidoDto>> ObtenerPedido(int id)
    {
        var pedido = await _pedidoService.ObtenerPedidoPorIdAsync(id);
        
        if (pedido == null)
            return NotFound();

        return Ok(pedido);
    }

    [HttpGet("usuario/{usuarioId}")]
    public async Task<ActionResult<IEnumerable<PedidoDto>>> ObtenerPedidosUsuario(int usuarioId)
    {
        var pedidos = await _pedidoService.ObtenerPedidosUsuarioAsync(usuarioId);
        return Ok(pedidos);
    }

    [HttpPatch("{id}/estado")]
    [Authorize(Roles = "Admin")] // Solo administradores
    public async Task<ActionResult> ActualizarEstado(int id, [FromBody] EstadoPedido nuevoEstado)
    {
        var actualizado = await _pedidoService.ActualizarEstadoPedidoAsync(id, nuevoEstado);
        
        if (!actualizado)
            return NotFound();

        return NoContent();
    }
}
```

---

**Anterior**: [03. Colecciones y Programación Funcional](./03-colecciones.md) | **Siguiente**: [05. Integración de Datos](./05-integracion.md) | **Inicio**: [README](../README.md)
