# 03. Colecciones y Programación Funcional

## 📚 Arrays y Colecciones

### Arrays básicos
```csharp
// Declaración e inicialización de arrays
int[] numeros = new int[5];                    // Array de 5 elementos
string[] nombres = { "Ana", "Carlos", "Luis" }; // Inicialización directa
double[] precios = new double[] { 19.99, 25.50, 12.75 };

// Arrays multidimensionales
int[,] matriz = new int[3, 3];                 // Matriz 3x3
int[,] tabla = { {1, 2, 3}, {4, 5, 6} };      // Inicialización directa

// Arrays jagged (arrays de arrays)
int[][] grupos = new int[3][];
grupos[0] = new int[] { 1, 2, 3, 4 };
grupos[1] = new int[] { 5, 6 };
grupos[2] = new int[] { 7, 8, 9 };

// Ejemplo práctico: sistema de calificaciones
string[] materias = { "Matemáticas", "Historia", "Ciencias", "Literatura" };
double[,] calificaciones = new double[4, 3]; // 4 materias, 3 estudiantes

// Llenar calificaciones
Random random = new Random();
for (int materia = 0; materia < 4; materia++)
{
    for (int estudiante = 0; estudiante < 3; estudiante++)
    {
        calificaciones[materia, estudiante] = random.Next(60, 101); // 60-100
    }
}

// Mostrar calificaciones
Console.WriteLine("Calificaciones por materia:");
for (int i = 0; i < materias.Length; i++)
{
    Console.Write($"{materias[i]}: ");
    for (int j = 0; j < 3; j++)
    {
        Console.Write($"{calificaciones[i, j]:F1} ");
    }
    Console.WriteLine();
}
```

### Colecciones genéricas: List<T>
```csharp
// List<T> - La colección más utilizada
var productos = new List<Producto>
{
    new Producto("Laptop", 1200, "Tecnología"),
    new Producto("Mouse", 25, "Tecnología"),
    new Producto("Escritorio", 300, "Muebles"),
    new Producto("Silla", 150, "Muebles")
};

// Operaciones básicas
productos.Add(new Producto("Monitor", 250, "Tecnología"));
productos.Insert(0, new Producto("Teclado", 80, "Tecnología"));

// Búsqueda y filtrado
var productoEncontrado = productos.Find(p => p.Nombre == "Laptop");
var tecnologia = productos.FindAll(p => p.Categoria == "Tecnología");
var caros = productos.Where(p => p.Precio > 200).ToList();

// Ordenamiento
productos.Sort((p1, p2) => p1.Precio.CompareTo(p2.Precio)); // Por precio
var ordenadosPorNombre = productos.OrderBy(p => p.Nombre).ToList();

// Manipulación
productos.RemoveAll(p => p.Precio < 50);
productos.ForEach(p => Console.WriteLine(p));

// Ejemplo práctico: gestión de inventario
public class GestorInventario
{
    private List<Producto> inventario = new List<Producto>();
    
    public void AgregarProducto(Producto producto)
    {
        var existente = inventario.Find(p => p.Nombre.Equals(producto.Nombre, 
                                             StringComparison.OrdinalIgnoreCase));
        if (existente != null)
        {
            existente.Stock += producto.Stock;
            Console.WriteLine($"Stock actualizado para {producto.Nombre}");
        }
        else
        {
            inventario.Add(producto);
            Console.WriteLine($"Producto {producto.Nombre} agregado al inventario");
        }
    }
    
    public List<Producto> BuscarPorCategoria(string categoria)
    {
        return inventario.Where(p => p.Categoria.Equals(categoria, 
                                    StringComparison.OrdinalIgnoreCase)).ToList();
    }
    
    public void MostrarResumen()
    {
        var resumen = inventario
            .GroupBy(p => p.Categoria)
            .Select(g => new 
            {
                Categoria = g.Key,
                Cantidad = g.Count(),
                ValorTotal = g.Sum(p => p.Precio * p.Stock),
                PromedioPrecios = g.Average(p => p.Precio)
            });
        
        Console.WriteLine("\n=== RESUMEN DE INVENTARIO ===");
        foreach (var item in resumen)
        {
            Console.WriteLine($"Categoría: {item.Categoria}");
            Console.WriteLine($"  Productos: {item.Cantidad}");
            Console.WriteLine($"  Valor total: {item.ValorTotal:C}");
            Console.WriteLine($"  Precio promedio: {item.PromedioPrecios:C}");
            Console.WriteLine();
        }
    }
}
```

### Dictionary<TKey, TValue>
```csharp
// Diccionarios para relaciones clave-valor
var estudiantes = new Dictionary<string, Estudiante>
{
    ["EST001"] = new Estudiante("Ana García", 20, 85.5),
    ["EST002"] = new Estudiante("Carlos López", 22, 92.0),
    ["EST003"] = new Estudiante("María Rodríguez", 21, 78.3)
};

// Operaciones con diccionarios
estudiantes.Add("EST004", new Estudiante("Juan Pérez", 23, 88.7));

// Verificar existencia de clave
if (estudiantes.ContainsKey("EST001"))
{
    Console.WriteLine($"Estudiante encontrado: {estudiantes["EST001"].Nombre}");
}

// TryGetValue para operaciones seguras
if (estudiantes.TryGetValue("EST005", out Estudiante? estudiante))
{
    Console.WriteLine($"Encontrado: {estudiante.Nombre}");
}
else
{
    Console.WriteLine("Estudiante no encontrado");
}

// Iterar sobre diccionario
foreach (var kvp in estudiantes)
{
    Console.WriteLine($"ID: {kvp.Key}, Estudiante: {kvp.Value.Nombre}");
}

// Solo claves o valores
var ids = estudiantes.Keys.ToList();
var listaEstudiantes = estudiantes.Values.ToList();

// Ejemplo práctico: cache de resultados
public class CacheResultados<TKey, TValue> where TKey : notnull
{
    private readonly Dictionary<TKey, (TValue Value, DateTime Timestamp)> _cache 
        = new Dictionary<TKey, (TValue, DateTime)>();
    private readonly TimeSpan _tiempoExpiracion;
    
    public CacheResultados(TimeSpan tiempoExpiracion)
    {
        _tiempoExpiracion = tiempoExpiracion;
    }
    
    public void Agregar(TKey clave, TValue valor)
    {
        _cache[clave] = (valor, DateTime.Now);
    }
    
    public bool TryGetValue(TKey clave, out TValue? valor)
    {
        valor = default(TValue);
        
        if (_cache.TryGetValue(clave, out var item))
        {
            if (DateTime.Now - item.Timestamp <= _tiempoExpiracion)
            {
                valor = item.Value;
                return true;
            }
            else
            {
                _cache.Remove(clave); // Eliminar elemento expirado
            }
        }
        
        return false;
    }
    
    public void LimpiarExpirados()
    {
        var expirados = _cache
            .Where(kvp => DateTime.Now - kvp.Value.Timestamp > _tiempoExpiracion)
            .Select(kvp => kvp.Key)
            .ToList();
        
        foreach (var clave in expirados)
        {
            _cache.Remove(clave);
        }
        
        Console.WriteLine($"Eliminados {expirados.Count} elementos expirados");
    }
}
```

### HashSet<T> y SortedSet<T>
```csharp
// HashSet para elementos únicos
var tags = new HashSet<string> { "programación", "C#", ".NET" };
tags.Add("desarrollo"); // Se agrega
tags.Add("C#");         // No se agrega (ya existe)

Console.WriteLine($"Tags únicos: {tags.Count}"); // 4

// Operaciones de conjuntos
var tagsBackend = new HashSet<string> { "C#", "SQL", "API", "backend" };
var tagsFrontend = new HashSet<string> { "JavaScript", "HTML", "CSS", "frontend" };
var tagsFullstack = new HashSet<string> { "C#", "JavaScript", "SQL" };

// Unión
var todosTags = new HashSet<string>(tagsBackend);
todosTags.UnionWith(tagsFrontend);
Console.WriteLine($"Todos los tags: {string.Join(", ", todosTags)}");

// Intersección
var tagsComunes = new HashSet<string>(tagsBackend);
tagsComunes.IntersectWith(tagsFullstack);
Console.WriteLine($"Tags comunes: {string.Join(", ", tagsComunes)}");

// SortedSet para elementos únicos ordenados
var prioridades = new SortedSet<int> { 3, 1, 4, 1, 5, 9, 2, 6 };
Console.WriteLine($"Prioridades ordenadas: {string.Join(", ", prioridades)}"); // 1, 2, 3, 4, 5, 6, 9

// Ejemplo práctico: sistema de etiquetas
public class SistemaEtiquetas
{
    private readonly Dictionary<string, HashSet<string>> _etiquetasPorElemento;
    private readonly Dictionary<string, HashSet<string>> _elementosPorEtiqueta;
    
    public SistemaEtiquetas()
    {
        _etiquetasPorElemento = new Dictionary<string, HashSet<string>>();
        _elementosPorEtiqueta = new Dictionary<string, HashSet<string>>();
    }
    
    public void AgregarEtiqueta(string elemento, string etiqueta)
    {
        // Agregar etiqueta al elemento
        if (!_etiquetasPorElemento.ContainsKey(elemento))
            _etiquetasPorElemento[elemento] = new HashSet<string>();
        
        _etiquetasPorElemento[elemento].Add(etiqueta);
        
        // Agregar elemento a la etiqueta
        if (!_elementosPorEtiqueta.ContainsKey(etiqueta))
            _elementosPorEtiqueta[etiqueta] = new HashSet<string>();
        
        _elementosPorEtiqueta[etiqueta].Add(elemento);
    }
    
    public HashSet<string> ObtenerElementosConEtiqueta(string etiqueta)
    {
        return _elementosPorEtiqueta.GetValueOrDefault(etiqueta, new HashSet<string>());
    }
    
    public HashSet<string> ObtenerEtiquetasDeElemento(string elemento)
    {
        return _etiquetasPorElemento.GetValueOrDefault(elemento, new HashSet<string>());
    }
    
    public HashSet<string> BuscarElementosConTodasLasEtiquetas(params string[] etiquetas)
    {
        if (etiquetas.Length == 0) return new HashSet<string>();
        
        var resultado = new HashSet<string>(ObtenerElementosConEtiqueta(etiquetas[0]));
        
        for (int i = 1; i < etiquetas.Length; i++)
        {
            resultado.IntersectWith(ObtenerElementosConEtiqueta(etiquetas[i]));
        }
        
        return resultado;
    }
}
```

## 🔍 LINQ (Language Integrated Query)

### Consultas básicas
```csharp
var numeros = new[] { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

// Sintaxis de método (más común)
var pares = numeros.Where(n => n % 2 == 0);
var cuadrados = numeros.Select(n => n * n);
var primerosCinco = numeros.Take(5);
var despuesDeCinco = numeros.Skip(5);

// Sintaxis de consulta (similar a SQL)
var paresQuery = from n in numeros
                 where n % 2 == 0
                 select n;

var cuadradosQuery = from n in numeros
                     select n * n;

// Operaciones de agregación
int suma = numeros.Sum();
double promedio = numeros.Average();
int maximo = numeros.Max();
int minimo = numeros.Min();
int cantidad = numeros.Count();
bool todosPares = numeros.All(n => n % 2 == 0);
bool algunPar = numeros.Any(n => n % 2 == 0);

Console.WriteLine($"Suma: {suma}, Promedio: {promedio:F2}");
Console.WriteLine($"Máximo: {maximo}, Mínimo: {minimo}");
Console.WriteLine($"Cantidad: {cantidad}");
Console.WriteLine($"Todos pares: {todosPares}, Algún par: {algunPar}");
```

### LINQ con objetos complejos
```csharp
public record Empleado(int Id, string Nombre, string Departamento, decimal Salario, DateTime FechaIngreso);

var empleados = new List<Empleado>
{
    new(1, "Ana García", "IT", 75000, new DateTime(2020, 1, 15)),
    new(2, "Carlos López", "Ventas", 60000, new DateTime(2019, 3, 10)),
    new(3, "María Rodríguez", "IT", 80000, new DateTime(2021, 6, 1)),
    new(4, "Juan Pérez", "Marketing", 55000, new DateTime(2018, 11, 20)),
    new(5, "Luis González", "IT", 90000, new DateTime(2017, 8, 5)),
    new(6, "Carmen Ruiz", "Ventas", 65000, new DateTime(2020, 9, 12))
};

// Filtrado y proyección
var empleadosIT = empleados
    .Where(e => e.Departamento == "IT")
    .Select(e => new { e.Nombre, e.Salario })
    .ToList();

// Ordenamiento
var empleadosOrdenados = empleados
    .OrderByDescending(e => e.Salario)
    .ThenBy(e => e.Nombre)
    .ToList();

// Agrupación
var porDepartamento = empleados
    .GroupBy(e => e.Departamento)
    .Select(g => new
    {
        Departamento = g.Key,
        Cantidad = g.Count(),
        SalarioPromedio = g.Average(e => e.Salario),
        SalarioTotal = g.Sum(e => e.Salario)
    })
    .ToList();

Console.WriteLine("Empleados por departamento:");
foreach (var dept in porDepartamento)
{
    Console.WriteLine($"  {dept.Departamento}: {dept.Cantidad} empleados");
    Console.WriteLine($"    Salario promedio: {dept.SalarioPromedio:C}");
    Console.WriteLine($"    Salario total: {dept.SalarioTotal:C}");
}

// Consultas complejas con múltiples operaciones
var analisisDetallado = empleados
    .Where(e => e.FechaIngreso >= new DateTime(2019, 1, 1)) // Empleados recientes
    .GroupBy(e => e.Departamento)
    .Where(g => g.Count() >= 2) // Departamentos con al menos 2 empleados
    .Select(g => new
    {
        Departamento = g.Key,
        EmpleadosRecientes = g.Count(),
        SalarioMasAlto = g.Max(e => e.Salario),
        EmpleadoMejorPagado = g.OrderByDescending(e => e.Salario).First().Nombre,
        AntiguedadPromedio = g.Average(e => (DateTime.Now - e.FechaIngreso).Days / 365.25)
    })
    .OrderByDescending(d => d.SalarioMasAlto)
    .ToList();

Console.WriteLine("\nAnálisis de empleados recientes:");
foreach (var analisis in analisisDetallado)
{
    Console.WriteLine($"Departamento: {analisis.Departamento}");
    Console.WriteLine($"  Empleados recientes: {analisis.EmpleadosRecientes}");
    Console.WriteLine($"  Mejor pagado: {analisis.EmpleadoMejorPagado} ({analisis.SalarioMasAlto:C})");
    Console.WriteLine($"  Antigüedad promedio: {analisis.AntiguedadPromedio:F1} años");
}
```

### LINQ con joins y operaciones avanzadas
```csharp
public record Proyecto(int Id, string Nombre, int IdDepartamento, DateTime FechaInicio, DateTime? FechaFin);
public record Departamento(int Id, string Nombre, string Ubicacion);

var departamentos = new List<Departamento>
{
    new(1, "IT", "Edificio A"),
    new(2, "Ventas", "Edificio B"),
    new(3, "Marketing", "Edificio C")
};

var proyectos = new List<Proyecto>
{
    new(1, "Sistema CRM", 1, new DateTime(2023, 1, 1), new DateTime(2023, 6, 30)),
    new(2, "Campaña Q2", 3, new DateTime(2023, 4, 1), new DateTime(2023, 6, 30)),
    new(3, "App Mobile", 1, new DateTime(2023, 7, 1), null),
    new(4, "Análisis Ventas", 2, new DateTime(2023, 3, 15), new DateTime(2023, 5, 15))
};

// Join entre tablas
var proyectosConDepartamento = from p in proyectos
                               join d in departamentos on p.IdDepartamento equals d.Id
                               select new
                               {
                                   ProyectoNombre = p.Nombre,
                                   DepartamentoNombre = d.Nombre,
                                   Ubicacion = d.Ubicacion,
                                   FechaInicio = p.FechaInicio,
                                   Activo = p.FechaFin == null
                               };

// Left join (usando GroupJoin)
var departamentosConProyectos = from d in departamentos
                                join p in proyectos on d.Id equals p.IdDepartamento into proyectosDept
                                select new
                                {
                                    Departamento = d.Nombre,
                                    CantidadProyectos = proyectosDept.Count(),
                                    ProyectosActivos = proyectosDept.Count(p => p.FechaFin == null),
                                    Proyectos = proyectosDept.Select(p => p.Nombre).ToList()
                                };

Console.WriteLine("Departamentos y sus proyectos:");
foreach (var dept in departamentosConProyectos)
{
    Console.WriteLine($"{dept.Departamento}:");
    Console.WriteLine($"  Total proyectos: {dept.CantidadProyectos}");
    Console.WriteLine($"  Proyectos activos: {dept.ProyectosActivos}");
    Console.WriteLine($"  Proyectos: {string.Join(", ", dept.Proyectos)}");
}

// Consultas con múltiples fuentes de datos
var reporteCompleto = from e in empleados
                      join d in departamentos on e.Departamento equals d.Nombre
                      join p in proyectos on d.Id equals p.IdDepartamento into proyectosDept
                      where e.Salario > 60000
                      select new
                      {
                          Empleado = e.Nombre,
                          Departamento = e.Departamento,
                          Salario = e.Salario,
                          Ubicacion = d.Ubicacion,
                          ProyectosEnDepartamento = proyectosDept.Count()
                      };
```

## ⚡ Programación Asíncrona

### async/await básico
```csharp
// Método asíncrono básico
public async Task<string> ObtenerDatosAsync(string url)
{
    using var client = new HttpClient();
    
    try
    {
        Console.WriteLine($"Iniciando descarga de {url}...");
        string contenido = await client.GetStringAsync(url);
        Console.WriteLine($"Descarga completada. Tamaño: {contenido.Length} caracteres");
        return contenido;
    }
    catch (HttpRequestException ex)
    {
        Console.WriteLine($"Error al descargar: {ex.Message}");
        return string.Empty;
    }
}

// Procesamiento asíncrono en paralelo
public async Task<List<string>> ProcesarMultiplesUrlsAsync(List<string> urls)
{
    var tareas = urls.Select(url => ObtenerDatosAsync(url)).ToList();
    
    Console.WriteLine($"Procesando {urls.Count} URLs en paralelo...");
    var resultados = await Task.WhenAll(tareas);
    
    Console.WriteLine("Todas las descargas completadas");
    return resultados.ToList();
}

// Uso del código asíncrono
public async Task EjemploUsoAsync()
{
    var urls = new List<string>
    {
        "https://jsonplaceholder.typicode.com/posts/1",
        "https://jsonplaceholder.typicode.com/posts/2",
        "https://jsonplaceholder.typicode.com/posts/3"
    };
    
    var stopwatch = System.Diagnostics.Stopwatch.StartNew();
    
    // Procesamiento secuencial
    Console.WriteLine("=== Procesamiento Secuencial ===");
    foreach (var url in urls)
    {
        await ObtenerDatosAsync(url);
    }
    Console.WriteLine($"Tiempo secuencial: {stopwatch.ElapsedMilliseconds}ms\n");
    
    // Procesamiento en paralelo
    stopwatch.Restart();
    Console.WriteLine("=== Procesamiento Paralelo ===");
    await ProcesarMultiplesUrlsAsync(urls);
    Console.WriteLine($"Tiempo paralelo: {stopwatch.ElapsedMilliseconds}ms");
}
```

### Task.Run para operaciones intensivas en CPU
```csharp
// Operación intensiva en CPU
public async Task<long> CalcularFibonacciAsync(int n)
{
    return await Task.Run(() => CalcularFibonacci(n));
}

private long CalcularFibonacci(int n)
{
    Console.WriteLine($"Calculando Fibonacci({n}) en hilo: {Thread.CurrentThread.ManagedThreadId}");
    
    if (n <= 1) return n;
    
    long a = 0, b = 1;
    for (int i = 2; i <= n; i++)
    {
        long temp = a + b;
        a = b;
        b = temp;
    }
    return b;
}

// Procesamiento de múltiples cálculos
public async Task EjemploCalculosParalelosAsync()
{
    var numeros = new[] { 40, 41, 42, 43, 44 };
    
    Console.WriteLine($"Hilo principal: {Thread.CurrentThread.ManagedThreadId}");
    
    var tareas = numeros.Select(n => CalcularFibonacciAsync(n)).ToArray();
    var resultados = await Task.WhenAll(tareas);
    
    Console.WriteLine("\nResultados:");
    for (int i = 0; i < numeros.Length; i++)
    {
        Console.WriteLine($"Fibonacci({numeros[i]}) = {resultados[i]}");
    }
}
```

### Configuración y manejo de errores en async/await
```csharp
public class ServicioAsync
{
    private readonly HttpClient _httpClient;
    
    public ServicioAsync()
    {
        _httpClient = new HttpClient { Timeout = TimeSpan.FromSeconds(30) };
    }
    
    // Método con timeout personalizado
    public async Task<string> ObtenerConTimeoutAsync(string url, int timeoutSegundos)
    {
        using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(timeoutSegundos));
        
        try
        {
            var response = await _httpClient.GetAsync(url, cts.Token);
            response.EnsureSuccessStatusCode();
            return await response.Content.ReadAsStringAsync();
        }
        catch (OperationCanceledException) when (cts.Token.IsCancellationRequested)
        {
            throw new TimeoutException($"La operación excedió el timeout de {timeoutSegundos} segundos");
        }
    }
    
    // Retry automático con backoff exponencial
    public async Task<T> EjecutarConRetryAsync<T>(Func<Task<T>> operacion, int maxIntentos = 3)
    {
        var delay = TimeSpan.FromSeconds(1);
        
        for (int intento = 1; intento <= maxIntentos; intento++)
        {
            try
            {
                return await operacion();
            }
            catch (Exception ex) when (intento < maxIntentos)
            {
                Console.WriteLine($"Intento {intento} falló: {ex.Message}");
                Console.WriteLine($"Reintentando en {delay.TotalSeconds} segundos...");
                
                await Task.Delay(delay);
                delay = TimeSpan.FromMilliseconds(delay.TotalMilliseconds * 2); // Backoff exponencial
            }
        }
        
        // Si llegamos aquí, todos los intentos fallaron
        return await operacion(); // Lanzará la excepción final
    }
    
    // Manejo de múltiples operaciones con diferentes estrategias
    public async Task<Dictionary<string, object>> ProcesarDatosComplejoAsync()
    {
        var resultado = new Dictionary<string, object>();
        
        // Operación 1: Obtener datos de API (crítica)
        try
        {
            var datosAPI = await EjecutarConRetryAsync(
                () => ObtenerConTimeoutAsync("https://api.ejemplo.com/datos", 10)
            );
            resultado["datos_api"] = datosAPI;
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Error crítico en API: {ex.Message}");
            throw; // Re-lanzar porque es crítica
        }
        
        // Operación 2: Cálculo intensivo (opcional)
        try
        {
            var calculo = await CalcularFibonacciAsync(35);
            resultado["calculo"] = calculo;
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Error en cálculo (no crítico): {ex.Message}");
            resultado["calculo"] = "Error en cálculo";
        }
        
        // Operación 3: Múltiples llamadas en paralelo con timeout individual
        var urlsOpcionales = new[]
        {
            "https://api1.ejemplo.com",
            "https://api2.ejemplo.com",
            "https://api3.ejemplo.com"
        };
        
        var tareasOpcionales = urlsOpcionales.Select(async url =>
        {
            try
            {
                return await ObtenerConTimeoutAsync(url, 5);
            }
            catch
            {
                return $"Error al obtener {url}";
            }
        });
        
        var resultadosOpcionales = await Task.WhenAll(tareasOpcionales);
        resultado["datos_opcionales"] = resultadosOpcionales;
        
        return resultado;
    }
}
```

## 📁 Manipulación de Archivos

### Operaciones básicas con archivos
```csharp
// Leer archivo completo
string contenido = await File.ReadAllTextAsync("archivo.txt");
string[] lineas = await File.ReadAllLinesAsync("archivo.txt");

// Escribir archivo
await File.WriteAllTextAsync("salida.txt", "Contenido del archivo");
await File.WriteAllLinesAsync("lista.txt", new[] { "Línea 1", "Línea 2", "Línea 3" });

// Verificar existencia
if (File.Exists("archivo.txt"))
{
    var info = new FileInfo("archivo.txt");
    Console.WriteLine($"Tamaño: {info.Length} bytes");
    Console.WriteLine($"Última modificación: {info.LastWriteTime}");
}

// Trabajar con directorios
string directorioTrabajo = @"C:\MiAplicacion\Datos";
Directory.CreateDirectory(directorioTrabajo);

var archivos = Directory.GetFiles(directorioTrabajo, "*.txt");
var subdirectorios = Directory.GetDirectories(directorioTrabajo);

Console.WriteLine($"Archivos .txt encontrados: {archivos.Length}");
foreach (var archivo in archivos)
{
    Console.WriteLine($"  {Path.GetFileName(archivo)}");
}
```

### Procesamiento de archivos grandes con StreamReader/StreamWriter
```csharp
public class ProcesadorArchivos
{
    // Leer archivo línea por línea (eficiente para archivos grandes)
    public async Task ProcesarArchivoGrandeAsync(string rutaArchivo)
    {
        using var reader = new StreamReader(rutaArchivo);
        int numeroLinea = 0;
        
        while (!reader.EndOfStream)
        {
            var linea = await reader.ReadLineAsync();
            numeroLinea++;
            
            // Procesar línea
            if (!string.IsNullOrWhiteSpace(linea))
            {
                await ProcesarLineaAsync(linea, numeroLinea);
            }
            
            // Reportar progreso cada 1000 líneas
            if (numeroLinea % 1000 == 0)
            {
                Console.WriteLine($"Procesadas {numeroLinea} líneas...");
            }
        }
        
        Console.WriteLine($"Archivo procesado completamente. Total líneas: {numeroLinea}");
    }
    
    private async Task ProcesarLineaAsync(string linea, int numeroLinea)
    {
        // Simular procesamiento asíncrono
        await Task.Delay(1);
        
        // Aquí iría la lógica específica de procesamiento
        if (linea.Contains("ERROR"))
        {
            Console.WriteLine($"Línea {numeroLinea}: Encontrado error - {linea}");
        }
    }
    
    // Escribir archivo de forma eficiente
    public async Task GenerarReporteAsync(string rutaSalida, IEnumerable<string> datos)
    {
        using var writer = new StreamWriter(rutaSalida);
        
        await writer.WriteLineAsync("=== REPORTE GENERADO ===");
        await writer.WriteLineAsync($"Fecha: {DateTime.Now:yyyy-MM-dd HH:mm:ss}");
        await writer.WriteLineAsync();
        
        int contador = 0;
        foreach (var dato in datos)
        {
            await writer.WriteLineAsync($"{++contador:D4}: {dato}");
        }
        
        await writer.WriteLineAsync();
        await writer.WriteLineAsync($"Total registros: {contador}");
    }
}

// Ejemplo de uso
var procesador = new ProcesadorArchivos();

// Generar datos de prueba
var datosPrueba = Enumerable.Range(1, 10000)
    .Select(i => $"Registro {i} - {DateTime.Now.AddMinutes(-i):HH:mm:ss}");

await procesador.GenerarReporteAsync("reporte.txt", datosPrueba);
await procesador.ProcesarArchivoGrandeAsync("reporte.txt");
```

### Serialización JSON
```csharp
using System.Text.Json;
using System.Text.Json.Serialization;

// Modelo para serialización
public class ConfiguracionApp
{
    public string NombreApp { get; set; } = string.Empty;
    public string Version { get; set; } = "1.0.0";
    public DatabaseConfig Database { get; set; } = new();
    public List<string> Modulos { get; set; } = new();
    
    [JsonIgnore]
    public string ClaveSecreta { get; set; } = string.Empty;
    
    [JsonPropertyName("fecha_creacion")]
    public DateTime FechaCreacion { get; set; } = DateTime.Now;
}

public class DatabaseConfig
{
    public string Servidor { get; set; } = "localhost";
    public int Puerto { get; set; } = 5432;
    public string BaseDatos { get; set; } = "miapp";
    public bool UsarSSL { get; set; } = true;
}

public class GestorConfiguracion
{
    private readonly JsonSerializerOptions _opcionesJson;
    
    public GestorConfiguracion()
    {
        _opcionesJson = new JsonSerializerOptions
        {
            WriteIndented = true,
            PropertyNamingPolicy = JsonNamingPolicy.CamelCase,
            DefaultIgnoreCondition = JsonIgnoreCondition.WhenWritingNull
        };
    }
    
    // Guardar configuración
    public async Task GuardarConfiguracionAsync(ConfiguracionApp config, string rutaArchivo)
    {
        var json = JsonSerializer.Serialize(config, _opcionesJson);
        await File.WriteAllTextAsync(rutaArchivo, json);
        Console.WriteLine($"Configuración guardada en {rutaArchivo}");
    }
    
    // Cargar configuración
    public async Task<ConfiguracionApp?> CargarConfiguracionAsync(string rutaArchivo)
    {
        if (!File.Exists(rutaArchivo))
        {
            Console.WriteLine($"Archivo {rutaArchivo} no encontrado");
            return null;
        }
        
        var json = await File.ReadAllTextAsync(rutaArchivo);
        var config = JsonSerializer.Deserialize<ConfiguracionApp>(json, _opcionesJson);
        
        Console.WriteLine($"Configuración cargada desde {rutaArchivo}");
        return config;
    }
    
    // Crear configuración por defecto
    public ConfiguracionApp CrearConfiguracionPorDefecto()
    {
        return new ConfiguracionApp
        {
            NombreApp = "Mi Aplicación",
            Version = "1.0.0",
            Database = new DatabaseConfig
            {
                Servidor = "localhost",
                Puerto = 5432,
                BaseDatos = "produccion"
            },
            Modulos = new List<string> { "Usuarios", "Productos", "Ventas" },
            FechaCreacion = DateTime.Now
        };
    }
}

// Ejemplo de uso
var gestor = new GestorConfiguracion();

// Crear y guardar configuración
var config = gestor.CrearConfiguracionPorDefecto();
await gestor.GuardarConfiguracionAsync(config, "config.json");

// Cargar configuración
var configCargada = await gestor.CargarConfiguracionAsync("config.json");
if (configCargada != null)
{
    Console.WriteLine($"App: {configCargada.NombreApp} v{configCargada.Version}");
    Console.WriteLine($"BD: {configCargada.Database.Servidor}:{configCargada.Database.Puerto}");
    Console.WriteLine($"Módulos: {string.Join(", ", configCargada.Modulos)}");
}
```

---

**Anterior**: [02. Programación Orientada a Objetos](./02-poo.md) | **Siguiente**: [04. Desarrollo Web con ASP.NET Core](./04-web.md) | **Inicio**: [README](../README.md)
