# 06. Documentación y Calidad

## 📚 Blazor Server y WebAssembly

### Configuración de Blazor Server
```csharp
// Program.cs para Blazor Server
var builder = WebApplication.CreateBuilder(args);

// Servicios para Blazor Server
builder.Services.AddRazorPages();
builder.Services.AddServerSideBlazor();

// Configuración de SignalR para mejor performance
builder.Services.AddServerSideBlazor(options =>
{
    options.DetailedErrors = builder.Environment.IsDevelopment();
    options.DisconnectedCircuitMaxRetained = 100;
    options.DisconnectedCircuitRetentionPeriod = TimeSpan.FromMinutes(3);
    options.JSInteropDefaultCallTimeout = TimeSpan.FromMinutes(1);
    options.MaxBufferedUnacknowledgedRenderBatches = 10;
});

// Configurar Entity Framework
builder.Services.AddDbContext<TiendaDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

// Registrar servicios de aplicación
builder.Services.AddScoped<IProductoService, ProductoService>();
builder.Services.AddScoped<ICategoriaService, CategoriaService>();
builder.Services.AddScoped<ICarritoService, CarritoService>();

// Servicios de autenticación
builder.Services.AddDefaultIdentity<IdentityUser>(options => 
{
    options.SignIn.RequireConfirmedAccount = false;
    options.Password.RequireDigit = true;
    options.Password.RequiredLength = 6;
})
.AddEntityFrameworkStores<TiendaDbContext>();

var app = builder.Build();

// Pipeline de middleware
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();

app.UseAuthentication();
app.UseAuthorization();

app.MapRazorPages();
app.MapBlazorHub();
app.MapFallbackToPage("/_Host");

app.Run();
```

### Componentes Blazor avanzados
```razor
@* Componente de lista de productos con funcionalidades avanzadas *@
@page "/productos"
@using System.Timers
@inject IProductoService ProductoService
@inject IJSRuntime JSRuntime
@inject IToastService ToastService
@implements IDisposable

<PageTitle>Gestión de Productos</PageTitle>

<div class="container-fluid">
    <div class="row">
        <div class="col-12">
            <div class="card">
                <div class="card-header d-flex justify-content-between align-items-center">
                    <h3 class="mb-0">
                        <i class="fas fa-box"></i> Productos
                        <span class="badge badge-primary">@totalProductos</span>
                    </h3>
                    <button class="btn btn-primary" @onclick="AbrirModalNuevoProducto">
                        <i class="fas fa-plus"></i> Nuevo Producto
                    </button>
                </div>

                <div class="card-body">
                    @* Filtros y búsqueda *@
                    <div class="row mb-3">
                        <div class="col-md-4">
                            <div class="input-group">
                                <input type="text" class="form-control" 
                                       placeholder="Buscar productos..." 
                                       @bind="filtroNombre" 
                                       @bind:event="oninput"
                                       @onkeyup="BuscarConDelay" />
                                <div class="input-group-append">
                                    <span class="input-group-text">
                                        <i class="fas fa-search"></i>
                                    </span>
                                </div>
                            </div>
                        </div>
                        <div class="col-md-3">
                            <select class="form-control" @bind="categoriaSeleccionada" @onchange="FiltrarPorCategoria">
                                <option value="">Todas las categorías</option>
                                @foreach (var categoria in categorias)
                                {
                                    <option value="@categoria.Id">@categoria.Nombre</option>
                                }
                            </select>
                        </div>
                        <div class="col-md-2">
                            <select class="form-control" @bind="ordenPor" @onchange="OrdenarProductos">
                                <option value="nombre">Nombre</option>
                                <option value="precio">Precio</option>
                                <option value="stock">Stock</option>
                                <option value="fecha">Fecha</option>
                            </select>
                        </div>
                        <div class="col-md-2">
                            <div class="form-check mt-2">
                                <input class="form-check-input" type="checkbox" 
                                       @bind="soloConStock" @onchange="FiltrarConStock" />
                                <label class="form-check-label">
                                    Solo con stock
                                </label>
                            </div>
                        </div>
                        <div class="col-md-1">
                            <button class="btn btn-outline-secondary" @onclick="LimpiarFiltros">
                                <i class="fas fa-undo"></i>
                            </button>
                        </div>
                    </div>

                    @* Loading indicator *@
                    @if (cargando)
                    {
                        <div class="text-center py-5">
                            <div class="spinner-border text-primary" role="status">
                                <span class="sr-only">Cargando...</span>
                            </div>
                            <p class="mt-2">Cargando productos...</p>
                        </div>
                    }
                    else
                    {
                        @* Tabla de productos *@
                        <div class="table-responsive">
                            <table class="table table-hover">
                                <thead class="thead-dark">
                                    <tr>
                                        <th>
                                            <button class="btn btn-link text-white p-0" @onclick="() => CambiarOrden('nombre')">
                                                Nombre
                                                @if (ordenPor == "nombre")
                                                {
                                                    <i class="fas fa-sort-@(ordenDescendente ? "down" : "up")"></i>
                                                }
                                            </button>
                                        </th>
                                        <th>Categoría</th>
                                        <th>
                                            <button class="btn btn-link text-white p-0" @onclick="() => CambiarOrden('precio')">
                                                Precio
                                                @if (ordenPor == "precio")
                                                {
                                                    <i class="fas fa-sort-@(ordenDescendente ? "down" : "up")"></i>
                                                }
                                            </button>
                                        </th>
                                        <th>
                                            <button class="btn btn-link text-white p-0" @onclick="() => CambiarOrden('stock')">
                                                Stock
                                                @if (ordenPor == "stock")
                                                {
                                                    <i class="fas fa-sort-@(ordenDescendente ? "down" : "up")"></i>
                                                }
                                            </button>
                                        </th>
                                        <th>Estado</th>
                                        <th>Acciones</th>
                                    </tr>
                                </thead>
                                <tbody>
                                    <Virtualize Items="productosMostrados" Context="producto">
                                        <tr class="@(producto.TieneBajoStock ? "table-warning" : "")">
                                            <td>
                                                <div class="d-flex align-items-center">
                                                    <div class="avatar-sm me-2">
                                                        <img src="@GetImagenProducto(producto.Id)" 
                                                             class="rounded" width="40" height="40" 
                                                             alt="@producto.Nombre" />
                                                    </div>
                                                    <div>
                                                        <strong>@producto.Nombre</strong>
                                                        @if (!string.IsNullOrEmpty(producto.Descripcion))
                                                        {
                                                            <br><small class="text-muted">@TruncarTexto(producto.Descripcion, 50)</small>
                                                        }
                                                    </div>
                                                </div>
                                            </td>
                                            <td>
                                                <span class="badge badge-info">@producto.NombreCategoria</span>
                                            </td>
                                            <td>
                                                <strong>@producto.Precio.ToString("C")</strong>
                                            </td>
                                            <td>
                                                <span class="badge @GetClasseStock(producto.Stock)">
                                                    @producto.Stock
                                                </span>
                                                @if (producto.TieneBajoStock)
                                                {
                                                    <i class="fas fa-exclamation-triangle text-warning ms-1" 
                                                       title="Stock bajo"></i>
                                                }
                                            </td>
                                            <td>
                                                <div class="form-check form-switch">
                                                    <input class="form-check-input" type="checkbox" 
                                                           checked="@producto.Activo"
                                                           @onchange="(e) => CambiarEstadoProducto(producto.Id, (bool)e.Value!)" />
                                                </div>
                                            </td>
                                            <td>
                                                <div class="btn-group" role="group">
                                                    <button class="btn btn-sm btn-outline-primary" 
                                                            @onclick="() => EditarProducto(producto.Id)"
                                                            title="Editar">
                                                        <i class="fas fa-edit"></i>
                                                    </button>
                                                    <button class="btn btn-sm btn-outline-info" 
                                                            @onclick="() => VerDetalleProducto(producto.Id)"
                                                            title="Ver detalle">
                                                        <i class="fas fa-eye"></i>
                                                    </button>
                                                    <button class="btn btn-sm btn-outline-warning" 
                                                            @onclick="() => AjustarStock(producto.Id)"
                                                            title="Ajustar stock">
                                                        <i class="fas fa-warehouse"></i>
                                                    </button>
                                                    <button class="btn btn-sm btn-outline-danger" 
                                                            @onclick="() => ConfirmarEliminar(producto.Id)"
                                                            title="Eliminar">
                                                        <i class="fas fa-trash"></i>
                                                    </button>
                                                </div>
                                            </td>
                                        </tr>
                                    </Virtualize>
                                </tbody>
                            </table>
                        </div>

                        @* Paginación *@
                        <nav aria-label="Paginación de productos">
                            <ul class="pagination justify-content-center">
                                <li class="page-item @(paginaActual == 1 ? "disabled" : "")">
                                    <button class="page-link" @onclick="() => CambiarPagina(paginaActual - 1)">
                                        <i class="fas fa-chevron-left"></i>
                                    </button>
                                </li>
                                
                                @for (int i = Math.Max(1, paginaActual - 2); i <= Math.Min(totalPaginas, paginaActual + 2); i++)
                                {
                                    var pagina = i;
                                    <li class="page-item @(paginaActual == pagina ? "active" : "")">
                                        <button class="page-link" @onclick="() => CambiarPagina(pagina)">
                                            @pagina
                                        </button>
                                    </li>
                                }
                                
                                <li class="page-item @(paginaActual == totalPaginas ? "disabled" : "")">
                                    <button class="page-link" @onclick="() => CambiarPagina(paginaActual + 1)">
                                        <i class="fas fa-chevron-right"></i>
                                    </button>
                                </li>
                            </ul>
                        </nav>

                        @* Información de paginación *@
                        <div class="d-flex justify-content-between align-items-center mt-3">
                            <div>
                                <small class="text-muted">
                                    Mostrando @((paginaActual - 1) * tamañoPagina + 1) a 
                                    @Math.Min(paginaActual * tamañoPagina, totalProductos) 
                                    de @totalProductos productos
                                </small>
                            </div>
                            <div>
                                <select class="form-control form-control-sm" style="width: auto; display: inline-block;"
                                        @bind="tamañoPagina" @onchange="CambiarTamañoPagina">
                                    <option value="10">10 por página</option>
                                    <option value="25">25 por página</option>
                                    <option value="50">50 por página</option>
                                    <option value="100">100 por página</option>
                                </select>
                            </div>
                        </div>
                    }
                </div>
            </div>
        </div>
    </div>
</div>

@* Modal para nuevo/editar producto *@
<ProductoModal @ref="productoModal" 
               OnProductoGuardado="OnProductoGuardado" 
               Categorias="categorias" />

@* Modal de confirmación de eliminación *@
<ConfirmModal @ref="confirmModal" 
              Titulo="Confirmar eliminación"
              Mensaje="¿Está seguro de que desea eliminar este producto?"
              OnConfirmado="EliminarProducto" />

@* Modal de ajuste de stock *@
<StockModal @ref="stockModal" 
            OnStockActualizado="OnStockActualizado" />

@code {
    // Estado del componente
    private List<ProductoListaDto> productos = new();
    private List<ProductoListaDto> productosMostrados = new();
    private List<CategoriaDto> categorias = new();
    
    // Filtros y búsqueda
    private string filtroNombre = "";
    private string categoriaSeleccionada = "";
    private string ordenPor = "nombre";
    private bool ordenDescendente = false;
    private bool soloConStock = false;
    private Timer? searchTimer;
    
    // Paginación
    private int paginaActual = 1;
    private int tamañoPagina = 25;
    private int totalProductos = 0;
    private int totalPaginas = 0;
    
    // Estado de UI
    private bool cargando = true;
    private int? productoAEliminar;
    
    // Referencias a modals
    private ProductoModal productoModal = default!;
    private ConfirmModal confirmModal = default!;
    private StockModal stockModal = default!;

    protected override async Task OnInitializedAsync()
    {
        await CargarCategorias();
        await CargarProductos();
    }

    private async Task CargarCategorias()
    {
        categorias = (await ProductoService.ObtenerCategoriasAsync()).ToList();
    }

    private async Task CargarProductos()
    {
        cargando = true;
        StateHasChanged();

        try
        {
            var filtro = new FiltroBusquedaProducto
            {
                Nombre = filtroNombre,
                CategoriaId = string.IsNullOrEmpty(categoriaSeleccionada) ? null : int.Parse(categoriaSeleccionada),
                OrdenarPor = ordenPor,
                OrdenDescendente = ordenDescendente,
                Pagina = paginaActual,
                TamañoPagina = tamañoPagina
            };

            var resultado = await ProductoService.BuscarProductosAsync(filtro);
            
            productosMostrados = resultado.Datos.ToList();
            totalProductos = resultado.TotalRegistros;
            totalPaginas = resultado.TotalPaginas;
        }
        catch (Exception ex)
        {
            ToastService.ShowError($"Error al cargar productos: {ex.Message}");
        }
        finally
        {
            cargando = false;
            StateHasChanged();
        }
    }

    private void BuscarConDelay()
    {
        searchTimer?.Dispose();
        searchTimer = new Timer(500); // 500ms delay
        searchTimer.Elapsed += async (sender, e) =>
        {
            searchTimer?.Dispose();
            await InvokeAsync(async () =>
            {
                paginaActual = 1;
                await CargarProductos();
            });
        };
        searchTimer.Start();
    }

    private async Task FiltrarPorCategoria(ChangeEventArgs e)
    {
        categoriaSeleccionada = e.Value?.ToString() ?? "";
        paginaActual = 1;
        await CargarProductos();
    }

    private async Task OrdenarProductos(ChangeEventArgs e)
    {
        ordenPor = e.Value?.ToString() ?? "nombre";
        await CargarProductos();
    }

    private async Task CambiarOrden(string campo)
    {
        if (ordenPor == campo)
        {
            ordenDescendente = !ordenDescendente;
        }
        else
        {
            ordenPor = campo;
            ordenDescendente = false;
        }
        
        await CargarProductos();
    }

    private async Task FiltrarConStock(ChangeEventArgs e)
    {
        soloConStock = (bool)(e.Value ?? false);
        paginaActual = 1;
        await CargarProductos();
    }

    private async Task LimpiarFiltros()
    {
        filtroNombre = "";
        categoriaSeleccionada = "";
        ordenPor = "nombre";
        ordenDescendente = false;
        soloConStock = false;
        paginaActual = 1;
        await CargarProductos();
    }

    private async Task CambiarPagina(int nuevaPagina)
    {
        if (nuevaPagina >= 1 && nuevaPagina <= totalPaginas)
        {
            paginaActual = nuevaPagina;
            await CargarProductos();
        }
    }

    private async Task CambiarTamañoPagina(ChangeEventArgs e)
    {
        tamañoPagina = int.Parse(e.Value?.ToString() ?? "25");
        paginaActual = 1;
        await CargarProductos();
    }

    // Métodos de UI helpers
    private string GetImagenProducto(int productoId)
    {
        return $"/images/productos/{productoId}.jpg";
    }

    private string TruncarTexto(string texto, int longitud)
    {
        return texto.Length <= longitud ? texto : texto.Substring(0, longitud) + "...";
    }

    private string GetClasseStock(int stock)
    {
        return stock switch
        {
            <= 0 => "badge-danger",
            <= 10 => "badge-warning",
            _ => "badge-success"
        };
    }

    // Métodos de acciones
    private async Task AbrirModalNuevoProducto()
    {
        await productoModal.AbrirParaNuevo();
    }

    private async Task EditarProducto(int productoId)
    {
        await productoModal.AbrirParaEditar(productoId);
    }

    private async Task VerDetalleProducto(int productoId)
    {
        // Implementar modal de detalle o navegar a página de detalle
        await JSRuntime.InvokeVoidAsync("window.open", $"/productos/{productoId}", "_blank");
    }

    private async Task AjustarStock(int productoId)
    {
        await stockModal.Abrir(productoId);
    }

    private async Task ConfirmarEliminar(int productoId)
    {
        productoAEliminar = productoId;
        await confirmModal.Mostrar();
    }

    private async Task EliminarProducto()
    {
        if (productoAEliminar.HasValue)
        {
            try
            {
                await ProductoService.EliminarAsync(productoAEliminar.Value);
                ToastService.ShowSuccess("Producto eliminado exitosamente");
                await CargarProductos();
            }
            catch (Exception ex)
            {
                ToastService.ShowError($"Error al eliminar producto: {ex.Message}");
            }
            finally
            {
                productoAEliminar = null;
            }
        }
    }

    private async Task CambiarEstadoProducto(int productoId, bool nuevoEstado)
    {
        try
        {
            await ProductoService.CambiarEstadoAsync(productoId, nuevoEstado);
            var producto = productosMostrados.FirstOrDefault(p => p.Id == productoId);
            if (producto != null)
            {
                producto.Activo = nuevoEstado;
                StateHasChanged();
            }
            
            ToastService.ShowSuccess($"Producto {(nuevoEstado ? "activado" : "desactivado")} exitosamente");
        }
        catch (Exception ex)
        {
            ToastService.ShowError($"Error al cambiar estado: {ex.Message}");
            await CargarProductos(); // Recargar para revertir cambio visual
        }
    }

    // Event handlers
    private async Task OnProductoGuardado()
    {
        await CargarProductos();
    }

    private async Task OnStockActualizado()
    {
        await CargarProductos();
    }

    public void Dispose()
    {
        searchTimer?.Dispose();
    }
}
```

### WPF con MVVM moderno
```csharp
// MainWindow.xaml.cs
public partial class MainWindow : Window
{
    public MainWindow()
    {
        InitializeComponent();
        DataContext = App.ServiceProvider.GetRequiredService<MainViewModel>();
    }
}

// ViewModels/MainViewModel.cs
public class MainViewModel : ViewModelBase
{
    private readonly IProductoService _productoService;
    private readonly IDialogService _dialogService;
    private readonly IEventAggregator _eventAggregator;

    public MainViewModel(
        IProductoService productoService,
        IDialogService dialogService,
        IEventAggregator eventAggregator)
    {
        _productoService = productoService;
        _dialogService = dialogService;
        _eventAggregator = eventAggregator;

        // Comandos
        CargarProductosCommand = new AsyncRelayCommand(CargarProductosAsync);
        AgregarProductoCommand = new AsyncRelayCommand(AgregarProductoAsync);
        EditarProductoCommand = new AsyncRelayCommand<ProductoViewModel>(EditarProductoAsync);
        EliminarProductoCommand = new AsyncRelayCommand<ProductoViewModel>(EliminarProductoAsync);
        BuscarCommand = new AsyncRelayCommand(BuscarProductosAsync);

        // Inicialización
        _ = Task.Run(InicializarAsync);
    }

    // Propiedades
    private ObservableCollection<ProductoViewModel> _productos = new();
    public ObservableCollection<ProductoViewModel> Productos
    {
        get => _productos;
        set => SetProperty(ref _productos, value);
    }

    private ProductoViewModel? _productoSeleccionado;
    public ProductoViewModel? ProductoSeleccionado
    {
        get => _productoSeleccionado;
        set => SetProperty(ref _productoSeleccionado, value);
    }

    private string _filtroNombre = "";
    public string FiltroNombre
    {
        get => _filtroNombre;
        set
        {
            if (SetProperty(ref _filtroNombre, value))
            {
                // Búsqueda con delay
                _busquedaTimer?.Stop();
                _busquedaTimer = new DispatcherTimer { Interval = TimeSpan.FromMilliseconds(500) };
                _busquedaTimer.Tick += async (s, e) =>
                {
                    _busquedaTimer.Stop();
                    await BuscarProductosAsync();
                };
                _busquedaTimer.Start();
            }
        }
    }

    private bool _cargando;
    public bool Cargando
    {
        get => _cargando;
        set => SetProperty(ref _cargando, value);
    }

    private DispatcherTimer? _busquedaTimer;

    // Comandos
    public IAsyncRelayCommand CargarProductosCommand { get; }
    public IAsyncRelayCommand AgregarProductoCommand { get; }
    public IAsyncRelayCommand<ProductoViewModel> EditarProductoCommand { get; }
    public IAsyncRelayCommand<ProductoViewModel> EliminarProductoCommand { get; }
    public IAsyncRelayCommand BuscarCommand { get; }

    // Métodos
    private async Task InicializarAsync()
    {
        await CargarProductosAsync();
    }

    private async Task CargarProductosAsync()
    {
        try
        {
            Cargando = true;

            var productos = await _productoService.ObtenerTodosAsync();
            var productosViewModel = productos.Select(p => new ProductoViewModel(p, _productoService)).ToList();

            await Application.Current.Dispatcher.InvokeAsync(() =>
            {
                Productos.Clear();
                foreach (var producto in productosViewModel)
                {
                    Productos.Add(producto);
                }
            });
        }
        catch (Exception ex)
        {
            await _dialogService.ShowErrorAsync("Error", $"Error al cargar productos: {ex.Message}");
        }
        finally
        {
            Cargando = false;
        }
    }

    private async Task BuscarProductosAsync()
    {
        if (string.IsNullOrWhiteSpace(FiltroNombre))
        {
            await CargarProductosAsync();
            return;
        }

        try
        {
            Cargando = true;

            var productos = await _productoService.BuscarPorNombreAsync(FiltroNombre);
            var productosViewModel = productos.Select(p => new ProductoViewModel(p, _productoService)).ToList();

            await Application.Current.Dispatcher.InvokeAsync(() =>
            {
                Productos.Clear();
                foreach (var producto in productosViewModel)
                {
                    Productos.Add(producto);
                }
            });
        }
        catch (Exception ex)
        {
            await _dialogService.ShowErrorAsync("Error", $"Error en la búsqueda: {ex.Message}");
        }
        finally
        {
            Cargando = false;
        }
    }

    private async Task AgregarProductoAsync()
    {
        var nuevoProducto = new Producto();
        var resultado = await _dialogService.ShowProductoDialogAsync(nuevoProducto);

        if (resultado == true)
        {
            try
            {
                await _productoService.AgregarAsync(nuevoProducto);
                await CargarProductosAsync();
                
                _eventAggregator.PublishOnUIThread(new ProductoAgregadoEvent(nuevoProducto));
            }
            catch (Exception ex)
            {
                await _dialogService.ShowErrorAsync("Error", $"Error al agregar producto: {ex.Message}");
            }
        }
    }

    private async Task EditarProductoAsync(ProductoViewModel? productoViewModel)
    {
        if (productoViewModel?.Producto == null) return;

        var productoOriginal = await _productoService.ObtenerPorIdAsync(productoViewModel.Producto.Id);
        if (productoOriginal == null) return;

        var resultado = await _dialogService.ShowProductoDialogAsync(productoOriginal);

        if (resultado == true)
        {
            try
            {
                await _productoService.ActualizarAsync(productoOriginal);
                await CargarProductosAsync();
                
                _eventAggregator.PublishOnUIThread(new ProductoActualizadoEvent(productoOriginal));
            }
            catch (Exception ex)
            {
                await _dialogService.ShowErrorAsync("Error", $"Error al actualizar producto: {ex.Message}");
            }
        }
    }

    private async Task EliminarProductoAsync(ProductoViewModel? productoViewModel)
    {
        if (productoViewModel?.Producto == null) return;

        var confirmacion = await _dialogService.ShowConfirmationAsync(
            "Confirmar eliminación",
            $"¿Está seguro de que desea eliminar el producto '{productoViewModel.Producto.Nombre}'?");

        if (confirmacion == true)
        {
            try
            {
                await _productoService.EliminarAsync(productoViewModel.Producto.Id);
                await CargarProductosAsync();
                
                _eventAggregator.PublishOnUIThread(new ProductoEliminadoEvent(productoViewModel.Producto.Id));
            }
            catch (Exception ex)
            {
                await _dialogService.ShowErrorAsync("Error", $"Error al eliminar producto: {ex.Message}");
            }
        }
    }
}

// ViewModels/ProductoViewModel.cs
public class ProductoViewModel : ViewModelBase
{
    private readonly IProductoService _productoService;

    public ProductoViewModel(Producto producto, IProductoService productoService)
    {
        Producto = producto;
        _productoService = productoService;
        
        CambiarEstadoCommand = new AsyncRelayCommand(CambiarEstadoAsync);
        ActualizarStockCommand = new AsyncRelayCommand(ActualizarStockAsync);
    }

    public Producto Producto { get; }

    // Propiedades binding
    public string Nombre => Producto.Nombre;
    public decimal Precio => Producto.Precio;
    public int Stock => Producto.Stock;
    public string NombreCategoria => Producto.Categoria?.Nombre ?? "";
    public bool Activo => Producto.Activo;
    public bool TieneBajoStock => Producto.TieneBajoStock;

    // Propiedades de estilo
    public Brush ColorFondo => TieneBajoStock ? Brushes.LightYellow : Brushes.White;
    public Brush ColorTexto => Activo ? Brushes.Black : Brushes.Gray;
    public string IconoEstado => Activo ? "✓" : "✗";

    // Comandos
    public IAsyncRelayCommand CambiarEstadoCommand { get; }
    public IAsyncRelayCommand ActualizarStockCommand { get; }

    private async Task CambiarEstadoAsync()
    {
        try
        {
            Producto.Activo = !Producto.Activo;
            await _productoService.ActualizarAsync(Producto);
            
            // Notificar cambios
            OnPropertyChanged(nameof(Activo));
            OnPropertyChanged(nameof(ColorTexto));
            OnPropertyChanged(nameof(IconoEstado));
        }
        catch (Exception ex)
        {
            // Revertir cambio
            Producto.Activo = !Producto.Activo;
            throw new InvalidOperationException($"Error al cambiar estado: {ex.Message}");
        }
    }

    private async Task ActualizarStockAsync()
    {
        // Implementar lógica para actualizar stock
        // Por ejemplo, mostrar un dialog para ingresar nuevo stock
    }
}

// Views/MainWindow.xaml
<Window x:Class="TiendaWPF.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        mc:Ignorable="d"
        Title="Gestión de Productos" Height="800" Width="1200">
    
    <Window.Resources>
        <BooleanToVisibilityConverter x:Key="BoolToVisConverter"/>
    </Window.Resources>

    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
            <RowDefinition Height="Auto"/>
        </Grid.RowDefinitions>

        <!-- Toolbar -->
        <ToolBar Grid.Row="0" ToolBarTray.IsLocked="True">
            <Button Command="{Binding CargarProductosCommand}" ToolTip="Recargar">
                <StackPanel Orientation="Horizontal">
                    <TextBlock Text="🔄" FontSize="16" Margin="0,0,5,0"/>
                    <TextBlock Text="Recargar"/>
                </StackPanel>
            </Button>
            <Separator/>
            <Button Command="{Binding AgregarProductoCommand}" ToolTip="Agregar producto">
                <StackPanel Orientation="Horizontal">
                    <TextBlock Text="➕" FontSize="16" Margin="0,0,5,0"/>
                    <TextBlock Text="Agregar"/>
                </StackPanel>
            </Button>
            <Button Command="{Binding EditarProductoCommand}" 
                    CommandParameter="{Binding ProductoSeleccionado}"
                    ToolTip="Editar producto seleccionado">
                <StackPanel Orientation="Horizontal">
                    <TextBlock Text="✏️" FontSize="16" Margin="0,0,5,0"/>
                    <TextBlock Text="Editar"/>
                </StackPanel>
            </Button>
            <Button Command="{Binding EliminarProductoCommand}"
                    CommandParameter="{Binding ProductoSeleccionado}"
                    ToolTip="Eliminar producto seleccionado">
                <StackPanel Orientation="Horizontal">
                    <TextBlock Text="🗑️" FontSize="16" Margin="0,0,5,0"/>
                    <TextBlock Text="Eliminar"/>
                </StackPanel>
            </Button>
        </ToolBar>

        <!-- Filtros -->
        <Grid Grid.Row="1" Margin="10">
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="Auto"/>
                <ColumnDefinition Width="*"/>
                <ColumnDefinition Width="Auto"/>
            </Grid.ColumnDefinitions>

            <TextBlock Grid.Column="0" Text="Buscar:" VerticalAlignment="Center" Margin="0,0,10,0"/>
            <TextBox Grid.Column="1" Text="{Binding FiltroNombre, UpdateSourceTrigger=PropertyChanged}" 
                     Margin="0,0,10,0" Padding="5"/>
            <Button Grid.Column="2" Content="🔍" Command="{Binding BuscarCommand}" Padding="10,5"/>
        </Grid>

        <!-- DataGrid -->
        <DataGrid Grid.Row="2" 
                  ItemsSource="{Binding Productos}"
                  SelectedItem="{Binding ProductoSeleccionado}"
                  AutoGenerateColumns="False"
                  IsReadOnly="True"
                  GridLinesVisibility="Horizontal"
                  AlternatingRowBackground="LightGray"
                  Margin="10">
            
            <DataGrid.Columns>
                <DataGridTextColumn Header="ID" Binding="{Binding Producto.Id}" Width="60"/>
                
                <DataGridTemplateColumn Header="Nombre" Width="*">
                    <DataGridTemplateColumn.CellTemplate>
                        <DataTemplate>
                            <StackPanel Orientation="Vertical">
                                <TextBlock Text="{Binding Nombre}" 
                                           FontWeight="Bold"
                                           Foreground="{Binding ColorTexto}"/>
                                <TextBlock Text="{Binding Producto.Descripcion}" 
                                           FontSize="10"
                                           Foreground="Gray"
                                           TextTrimming="CharacterEllipsis"/>
                            </StackPanel>
                        </DataTemplate>
                    </DataGridTemplateColumn.CellTemplate>
                </DataGridTemplateColumn>

                <DataGridTextColumn Header="Categoría" Binding="{Binding NombreCategoria}" Width="120"/>
                
                <DataGridTemplateColumn Header="Precio" Width="100">
                    <DataGridTemplateColumn.CellTemplate>
                        <DataTemplate>
                            <TextBlock Text="{Binding Precio, StringFormat=C}" 
                                       HorizontalAlignment="Right"
                                       FontWeight="Bold"/>
                        </DataTemplate>
                    </DataGridTemplateColumn.CellTemplate>
                </DataGridTemplateColumn>

                <DataGridTemplateColumn Header="Stock" Width="80">
                    <DataGridTemplateColumn.CellTemplate>
                        <DataTemplate>
                            <Border Background="{Binding ColorFondo}" 
                                    CornerRadius="3" Padding="5,2">
                                <TextBlock Text="{Binding Stock}" 
                                           HorizontalAlignment="Center"
                                           FontWeight="Bold"/>
                            </Border>
                        </DataTemplate>
                    </DataGridTemplateColumn.CellTemplate>
                </DataGridTemplateColumn>

                <DataGridTemplateColumn Header="Estado" Width="80">
                    <DataGridTemplateColumn.CellTemplate>
                        <DataTemplate>
                            <CheckBox IsChecked="{Binding Activo}" 
                                      Command="{Binding CambiarEstadoCommand}"
                                      HorizontalAlignment="Center"/>
                        </DataTemplate>
                    </DataGridTemplateColumn.CellTemplate>
                </DataGridTemplateColumn>

                <DataGridTemplateColumn Header="Acciones" Width="120">
                    <DataGridTemplateColumn.CellTemplate>
                        <DataTemplate>
                            <StackPanel Orientation="Horizontal" HorizontalAlignment="Center">
                                <Button Content="📝" 
                                        Command="{Binding DataContext.EditarProductoCommand, RelativeSource={RelativeSource AncestorType=DataGrid}}"
                                        CommandParameter="{Binding}"
                                        Margin="2" Padding="5"/>
                                <Button Content="📦" 
                                        Command="{Binding ActualizarStockCommand}"
                                        Margin="2" Padding="5"/>
                            </StackPanel>
                        </DataTemplate>
                    </DataGridTemplateColumn.CellTemplate>
                </DataGridTemplateColumn>
            </DataGrid.Columns>
        </DataGrid>

        <!-- Loading indicator -->
        <Border Grid.Row="2" 
                Background="White" 
                Opacity="0.8"
                Visibility="{Binding Cargando, Converter={StaticResource BoolToVisConverter}}">
            <StackPanel HorizontalAlignment="Center" VerticalAlignment="Center">
                <ProgressBar IsIndeterminate="True" Width="200" Height="20"/>
                <TextBlock Text="Cargando..." Margin="0,10,0,0" HorizontalAlignment="Center"/>
            </StackPanel>
        </Border>

        <!-- Status bar -->
        <StatusBar Grid.Row="3">
            <StatusBarItem>
                <TextBlock Text="{Binding Productos.Count, StringFormat='Total productos: {0}'}"/>
            </StatusBarItem>
            <Separator/>
            <StatusBarItem>
                <TextBlock Text="{Binding ProductoSeleccionado.Nombre, StringFormat='Seleccionado: {0}'}"/>
            </StatusBarItem>
        </StatusBar>
    </Grid>
</Window>
```

## 🧪 Pruebas Unitarias y de Integración

### Configuración de pruebas con xUnit
```csharp
// Pruebas unitarias para servicios
public class ProductoServiceTests : IClassFixture<ProductoServiceTestFixture>
{
    private readonly ProductoServiceTestFixture _fixture;
    private readonly IProductoService _productoService;
    private readonly Mock<IRepositorio<Producto>> _mockRepositorio;
    private readonly Mock<ILogger<ProductoService>> _mockLogger;

    public ProductoServiceTests(ProductoServiceTestFixture fixture)
    {
        _fixture = fixture;
        _mockRepositorio = new Mock<IRepositorio<Producto>>();
        _mockLogger = new Mock<ILogger<ProductoService>>();
        _productoService = new ProductoService(_mockRepositorio.Object, _mockLogger.Object);
    }

    [Fact]
    public async Task ObtenerPorId_ProductoExistente_DeberiaRetornarProducto()
    {
        // Arrange
        var productoId = 1;
        var productoEsperado = _fixture.CrearProducto(productoId, "Test Producto");
        
        _mockRepositorio
            .Setup(r => r.ObtenerPorIdAsync(productoId))
            .ReturnsAsync(productoEsperado);

        // Act
        var resultado = await _productoService.ObtenerPorIdAsync(productoId);

        // Assert
        Assert.NotNull(resultado);
        Assert.Equal(productoEsperado.Id, resultado.Id);
        Assert.Equal(productoEsperado.Nombre, resultado.Nombre);
        
        _mockRepositorio.Verify(r => r.ObtenerPorIdAsync(productoId), Times.Once);
    }

    [Fact]
    public async Task ObtenerPorId_ProductoNoExistente_DeberiaRetornarNull()
    {
        // Arrange
        var productoId = 999;
        
        _mockRepositorio
            .Setup(r => r.ObtenerPorIdAsync(productoId))
            .ReturnsAsync((Producto?)null);

        // Act
        var resultado = await _productoService.ObtenerPorIdAsync(productoId);

        // Assert
        Assert.Null(resultado);
        
        _mockRepositorio.Verify(r => r.ObtenerPorIdAsync(productoId), Times.Once);
    }

    [Theory]
    [InlineData("")]
    [InlineData("   ")]
    [InlineData(null)]
    public async Task Agregar_NombreInvalido_DeberiaLanzarExcepcion(string nombreInvalido)
    {
        // Arrange
        var producto = _fixture.CrearProducto(0, nombreInvalido);

        // Act & Assert
        var excepcion = await Assert.ThrowsAsync<ArgumentException>(
            () => _productoService.AgregarAsync(producto));
        
        Assert.Contains("nombre", excepcion.Message.ToLower());
        
        _mockRepositorio.Verify(r => r.AgregarAsync(It.IsAny<Producto>()), Times.Never);
    }

    [Fact]
    public async Task Agregar_ProductoValido_DeberiaAgregarYRetornarProducto()
    {
        // Arrange
        var producto = _fixture.CrearProducto(0, "Nuevo Producto");
        var productoAgregado = _fixture.CrearProducto(1, "Nuevo Producto");
        
        _mockRepositorio
            .Setup(r => r.AgregarAsync(It.IsAny<Producto>()))
            .ReturnsAsync(productoAgregado);

        // Act
        var resultado = await _productoService.AgregarAsync(producto);

        // Assert
        Assert.NotNull(resultado);
        Assert.Equal(productoAgregado.Id, resultado.Id);
        Assert.Equal(productoAgregado.Nombre, resultado.Nombre);
        
        _mockRepositorio.Verify(r => r.AgregarAsync(producto), Times.Once);
    }

    [Fact]
    public async Task BuscarPorNombre_ConTermino_DeberiaRetornarProductosFiltrados()
    {
        // Arrange
        var termino = "test";
        var productosEsperados = new List<Producto>
        {
            _fixture.CrearProducto(1, "Test Producto 1"),
            _fixture.CrearProducto(2, "Test Producto 2")
        };
        
        _mockRepositorio
            .Setup(r => r.ObtenerPorCriterioAsync(It.IsAny<Expression<Func<Producto, bool>>>()))
            .ReturnsAsync(productosEsperados);

        // Act
        var resultado = await _productoService.BuscarPorNombreAsync(termino);

        // Assert
        Assert.NotNull(resultado);
        Assert.Equal(2, resultado.Count());
        Assert.All(resultado, p => Assert.Contains(termino, p.Nombre, StringComparison.OrdinalIgnoreCase));
        
        _mockRepositorio.Verify(
            r => r.ObtenerPorCriterioAsync(It.IsAny<Expression<Func<Producto, bool>>>()), 
            Times.Once);
    }

    [Fact]
    public async Task ActualizarStock_ProductoExistente_DeberiaActualizarStock()
    {
        // Arrange
        var productoId = 1;
        var nuevoStock = 50;
        var producto = _fixture.CrearProducto(productoId, "Test Producto");
        producto.Stock = 10;
        
        _mockRepositorio
            .Setup(r => r.ObtenerPorIdAsync(productoId))
            .ReturnsAsync(producto);
        
        _mockRepositorio
            .Setup(r => r.ActualizarAsync(It.IsAny<Producto>()))
            .ReturnsAsync(producto);

        // Act
        var resultado = await _productoService.ActualizarStockAsync(productoId, nuevoStock);

        // Assert
        Assert.True(resultado);
        Assert.Equal(nuevoStock, producto.Stock);
        
        _mockRepositorio.Verify(r => r.ObtenerPorIdAsync(productoId), Times.Once);
        _mockRepositorio.Verify(r => r.ActualizarAsync(producto), Times.Once);
    }
}

// Fixture para pruebas
public class ProductoServiceTestFixture
{
    public Producto CrearProducto(int id, string nombre)
    {
        return new Producto
        {
            Id = id,
            Nombre = nombre,
            Descripcion = $"Descripción de {nombre}",
            Precio = 100.00m,
            Stock = 10,
            Activo = true,
            CategoriaId = 1,
            ProveedorId = 1,
            FechaCreacion = DateTime.UtcNow
        };
    }

    public Categoria CrearCategoria(int id, string nombre)
    {
        return new Categoria
        {
            Id = id,
            Nombre = nombre,
            Descripcion = $"Descripción de {nombre}",
            Activa = true,
            FechaCreacion = DateTime.UtcNow
        };
    }
}

// Pruebas de integración
public class ProductoIntegrationTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly WebApplicationFactory<Program> _factory;
    private readonly HttpClient _client;

    public ProductoIntegrationTests(WebApplicationFactory<Program> factory)
    {
        _factory = factory;
        _client = _factory.CreateClient();
    }

    [Fact]
    public async Task Get_Productos_DeberiaRetornarListaDeProductos()
    {
        // Act
        var response = await _client.GetAsync("/api/productos");

        // Assert
        response.EnsureSuccessStatusCode();
        
        var content = await response.Content.ReadAsStringAsync();
        var productos = JsonSerializer.Deserialize<List<ProductoDto>>(content, new JsonSerializerOptions
        {
            PropertyNameCaseInsensitive = true
        });
        
        Assert.NotNull(productos);
        Assert.NotEmpty(productos);
    }

    [Fact]
    public async Task Post_Producto_DeberiaCrearNuevoProducto()
    {
        // Arrange
        var nuevoProducto = new
        {
            Nombre = "Producto de Prueba",
            Descripcion = "Descripción de prueba",
            Precio = 99.99m,
            Stock = 20,
            CategoriaId = 1,
            ProveedorId = 1
        };
        
        var json = JsonSerializer.Serialize(nuevoProducto);
        var content = new StringContent(json, Encoding.UTF8, "application/json");

        // Act
        var response = await _client.PostAsync("/api/productos", content);

        // Assert
        response.EnsureSuccessStatusCode();
        
        var responseContent = await response.Content.ReadAsStringAsync();
        var productoCreado = JsonSerializer.Deserialize<ProductoDto>(responseContent, new JsonSerializerOptions
        {
            PropertyNameCaseInsensitive = true
        });
        
        Assert.NotNull(productoCreado);
        Assert.Equal(nuevoProducto.Nombre, productoCreado.Nombre);
        Assert.True(productoCreado.Id > 0);
    }

    [Theory]
    [InlineData("/api/productos/1")]
    [InlineData("/api/productos/search?nombre=test")]
    [InlineData("/api/categorias")]
    public async Task Get_Endpoints_DeberianRetornarOk(string url)
    {
        // Act
        var response = await _client.GetAsync(url);

        // Assert
        response.EnsureSuccessStatusCode();
        Assert.Equal("application/json", response.Content.Headers.ContentType?.MediaType);
    }
}

// Pruebas de performance
public class ProductoPerformanceTests
{
    private readonly ITestOutputHelper _output;
    private readonly TiendaDbContext _context;

    public ProductoPerformanceTests(ITestOutputHelper output)
    {
        _output = output;
        
        var options = new DbContextOptionsBuilder<TiendaDbContext>()
            .UseInMemoryDatabase(databaseName: Guid.NewGuid().ToString())
            .Options;
        
        _context = new TiendaDbContext(options);
        SeedDatabase();
    }

    [Fact]
    public async Task BuscarProductos_ConGranVolumenDatos_DeberiaSerRapido()
    {
        // Arrange
        var stopwatch = Stopwatch.StartNew();
        const int maxTiempoMs = 1000; // 1 segundo máximo

        // Act
        var productos = await _context.Productos
            .Include(p => p.Categoria)
            .Where(p => p.Activo)
            .OrderBy(p => p.Nombre)
            .Take(100)
            .ToListAsync();

        stopwatch.Stop();

        // Assert
        Assert.NotEmpty(productos);
        Assert.True(stopwatch.ElapsedMilliseconds < maxTiempoMs, 
            $"La consulta tardó {stopwatch.ElapsedMilliseconds}ms, esperado menos de {maxTiempoMs}ms");
        
        _output.WriteLine($"Consulta completada en {stopwatch.ElapsedMilliseconds}ms");
    }

    [Fact]
    public async Task ConsultaCompleja_ConJoinsMultiples_DeberiaOptimizarse()
    {
        // Arrange
        var stopwatch = Stopwatch.StartNew();

        // Act
        var estadisticas = await _context.Productos
            .Where(p => p.Activo)
            .GroupBy(p => p.Categoria.Nombre)
            .Select(g => new
            {
                Categoria = g.Key,
                CantidadProductos = g.Count(),
                PrecioPromedio = g.Average(p => p.Precio),
                StockTotal = g.Sum(p => p.Stock)
            })
            .ToListAsync();

        stopwatch.Stop();

        // Assert
        Assert.NotEmpty(estadisticas);
        _output.WriteLine($"Consulta de estadísticas completada en {stopwatch.ElapsedMilliseconds}ms");
        
        foreach (var stat in estadisticas)
        {
            _output.WriteLine($"Categoría: {stat.Categoria}, Productos: {stat.CantidadProductos}, " +
                             $"Precio promedio: {stat.PrecioPromedio:C}, Stock total: {stat.StockTotal}");
        }
    }

    private void SeedDatabase()
    {
        // Crear categorías
        var categorias = new List<Categoria>();
        for (int i = 1; i <= 10; i++)
        {
            categorias.Add(new Categoria
            {
                Id = i,
                Nombre = $"Categoría {i}",
                Descripcion = $"Descripción de categoría {i}",
                Activa = true,
                FechaCreacion = DateTime.UtcNow
            });
        }
        
        _context.Categorias.AddRange(categorias);

        // Crear productos
        var random = new Random();
        var productos = new List<Producto>();
        
        for (int i = 1; i <= 1000; i++)
        {
            productos.Add(new Producto
            {
                Id = i,
                Nombre = $"Producto {i}",
                Descripcion = $"Descripción detallada del producto {i}",
                Precio = (decimal)(random.NextDouble() * 1000 + 10),
                Stock = random.Next(0, 100),
                Activo = random.NextDouble() > 0.1, // 90% activos
                CategoriaId = random.Next(1, 11),
                ProveedorId = 1,
                FechaCreacion = DateTime.UtcNow.AddDays(-random.Next(0, 365))
            });
        }
        
        _context.Productos.AddRange(productos);
        _context.SaveChanges();
    }
}
```

---

**Anterior**: [05. Integración de Datos](./05-integracion.md) | **Siguiente**: [07. Evaluación Final](./07-evaluacion.md) | **Inicio**: [README](../README.md)
