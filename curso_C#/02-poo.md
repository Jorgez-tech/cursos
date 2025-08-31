# 02. Programación Orientada a Objetos

## 🏗️ Fundamentos de POO en C#

### Conceptos fundamentales
La Programación Orientada a Objetos (POO) en C# se basa en cuatro pilares fundamentales:

1. **Encapsulamiento**: Ocultar detalles internos y exponer solo lo necesario
2. **Herencia**: Crear nuevas clases basadas en clases existentes
3. **Polimorfismo**: Un mismo método puede comportarse de diferentes maneras
4. **Abstracción**: Simplificar la complejidad mediante modelos conceptuales

## 📦 Clases y Objetos

### Definición básica de una clase
```csharp
// Clase básica que representa una cuenta bancaria
public class CuentaBancaria
{
    // Campos privados (encapsulamiento)
    private string _numeroCuenta;
    private decimal _saldo;
    private string _titular;
    private DateTime _fechaApertura;
    
    // Constructor
    public CuentaBancaria(string numeroCuenta, string titular, decimal saldoInicial = 0)
    {
        _numeroCuenta = numeroCuenta;
        _titular = titular;
        _saldo = saldoInicial;
        _fechaApertura = DateTime.Now;
    }
    
    // Propiedades (encapsulamiento controlado)
    public string NumeroCuenta 
    { 
        get => _numeroCuenta; 
        private set => _numeroCuenta = value; 
    }
    
    public decimal Saldo 
    { 
        get => _saldo; 
        private set => _saldo = value; 
    }
    
    public string Titular 
    { 
        get => _titular; 
        set => _titular = value ?? throw new ArgumentNullException(nameof(value)); 
    }
    
    public DateTime FechaApertura => _fechaApertura;
    
    // Métodos públicos
    public void Depositar(decimal cantidad)
    {
        if (cantidad <= 0)
            throw new ArgumentException("La cantidad debe ser positiva");
        
        _saldo += cantidad;
        Console.WriteLine($"Depósito exitoso. Nuevo saldo: {_saldo:C}");
    }
    
    public bool Retirar(decimal cantidad)
    {
        if (cantidad <= 0)
            throw new ArgumentException("La cantidad debe ser positiva");
        
        if (cantidad > _saldo)
        {
            Console.WriteLine("Fondos insuficientes");
            return false;
        }
        
        _saldo -= cantidad;
        Console.WriteLine($"Retiro exitoso. Nuevo saldo: {_saldo:C}");
        return true;
    }
    
    public void MostrarInformacion()
    {
        Console.WriteLine($"Cuenta: {_numeroCuenta}");
        Console.WriteLine($"Titular: {_titular}");
        Console.WriteLine($"Saldo: {_saldo:C}");
        Console.WriteLine($"Fecha apertura: {_fechaApertura:dd/MM/yyyy}");
    }
}
```

### Uso de la clase
```csharp
class Program
{
    static void Main()
    {
        // Crear objetos (instancias de la clase)
        var cuenta1 = new CuentaBancaria("001-123456", "Juan Pérez", 1000);
        var cuenta2 = new CuentaBancaria("001-654321", "María García");
        
        // Usar métodos del objeto
        cuenta1.MostrarInformacion();
        cuenta1.Depositar(500);
        cuenta1.Retirar(200);
        
        Console.WriteLine("\n" + new string('-', 40));
        
        cuenta2.MostrarInformacion();
        cuenta2.Depositar(750);
        cuenta2.Retirar(1000); // Intentar retirar más de lo disponible
    }
}
```

### Propiedades automáticas y init-only (C# 9.0+)
```csharp
// Clase moderna con propiedades automáticas
public class Producto
{
    // Propiedades automáticas
    public int Id { get; set; }
    public string Nombre { get; set; } = string.Empty;
    public decimal Precio { get; set; }
    public int Stock { get; set; }
    
    // Propiedades init-only (solo se pueden asignar en inicialización)
    public string Categoria { get; init; } = "General";
    public DateTime FechaCreacion { get; init; } = DateTime.Now;
    
    // Propiedad calculada
    public decimal ValorInventario => Precio * Stock;
    
    // Validación en setter
    private string _codigo = string.Empty;
    public string Codigo 
    { 
        get => _codigo;
        set => _codigo = value?.ToUpper() ?? throw new ArgumentNullException(nameof(value));
    }
    
    // Constructor
    public Producto(string nombre, decimal precio, string categoria = "General")
    {
        Nombre = nombre;
        Precio = precio;
        Categoria = categoria;
        Codigo = GenerarCodigo();
    }
    
    private string GenerarCodigo()
    {
        return $"{Categoria[..3].ToUpper()}-{DateTime.Now.Ticks % 10000:D4}";
    }
    
    public override string ToString()
    {
        return $"{Codigo}: {Nombre} - {Precio:C} (Stock: {Stock})";
    }
}

// Uso con object initializer
var producto = new Producto("Laptop Dell", 1200)
{
    Stock = 5,
    Categoria = "Tecnología"
};
```

## 🧬 Herencia

### Clase base y clases derivadas
```csharp
// Clase base
public abstract class Vehiculo
{
    // Propiedades protegidas (accesibles por clases derivadas)
    protected string marca;
    protected string modelo;
    protected int año;
    protected decimal precio;
    
    // Constructor de la clase base
    public Vehiculo(string marca, string modelo, int año, decimal precio)
    {
        this.marca = marca;
        this.modelo = modelo;
        this.año = año;
        this.precio = precio;
    }
    
    // Propiedades públicas
    public string Marca => marca;
    public string Modelo => modelo;
    public int Año => año;
    public virtual decimal Precio => precio;
    
    // Método virtual (puede ser sobrescrito)
    public virtual void MostrarInformacion()
    {
        Console.WriteLine($"Vehículo: {marca} {modelo}");
        Console.WriteLine($"Año: {año}");
        Console.WriteLine($"Precio: {precio:C}");
    }
    
    // Método abstracto (debe ser implementado por clases derivadas)
    public abstract decimal CalcularSeguro();
    
    // Método virtual con implementación por defecto
    public virtual string ObtenerTipoVehiculo()
    {
        return "Vehículo genérico";
    }
}

// Clase derivada: Automóvil
public class Automovil : Vehiculo
{
    private int numeroPuertas;
    private TipoCombustible combustible;
    
    public Automovil(string marca, string modelo, int año, decimal precio, 
                    int numeroPuertas, TipoCombustible combustible)
        : base(marca, modelo, año, precio) // Llamar al constructor base
    {
        this.numeroPuertas = numeroPuertas;
        this.combustible = combustible;
    }
    
    public int NumeroPuertas => numeroPuertas;
    public TipoCombustible Combustible => combustible;
    
    // Sobrescribir método virtual
    public override void MostrarInformacion()
    {
        base.MostrarInformacion(); // Llamar al método base
        Console.WriteLine($"Puertas: {numeroPuertas}");
        Console.WriteLine($"Combustible: {combustible}");
    }
    
    // Implementar método abstracto
    public override decimal CalcularSeguro()
    {
        decimal baseSeguro = precio * 0.05m; // 5% del precio
        
        // Ajustar por edad del vehículo
        int antiguedad = DateTime.Now.Year - año;
        if (antiguedad > 10)
            baseSeguro *= 0.8m; // 20% descuento para vehículos antiguos
        
        // Ajustar por tipo de combustible
        if (combustible == TipoCombustible.Electrico)
            baseSeguro *= 0.9m; // 10% descuento para eléctricos
        
        return baseSeguro;
    }
    
    public override string ObtenerTipoVehiculo()
    {
        return $"Automóvil {combustible}";
    }
}

// Clase derivada: Motocicleta
public class Motocicleta : Vehiculo
{
    private int cilindrada;
    private bool tieneSidecar;
    
    public Motocicleta(string marca, string modelo, int año, decimal precio, 
                      int cilindrada, bool tieneSidecar = false)
        : base(marca, modelo, año, precio)
    {
        this.cilindrada = cilindrada;
        this.tieneSidecar = tieneSidecar;
    }
    
    public int Cilindrada => cilindrada;
    public bool TieneSidecar => tieneSidecar;
    
    public override void MostrarInformacion()
    {
        base.MostrarInformacion();
        Console.WriteLine($"Cilindrada: {cilindrada}cc");
        Console.WriteLine($"Sidecar: {(tieneSidecar ? "Sí" : "No")}");
    }
    
    public override decimal CalcularSeguro()
    {
        decimal baseSeguro = precio * 0.08m; // 8% del precio (más riesgo)
        
        // Ajustar por cilindrada
        if (cilindrada > 600)
            baseSeguro *= 1.2m; // 20% más para motos de alta cilindrada
        
        if (tieneSidecar)
            baseSeguro *= 0.95m; // 5% descuento por mayor estabilidad
        
        return baseSeguro;
    }
    
    public override string ObtenerTipoVehiculo()
    {
        return $"Motocicleta {cilindrada}cc";
    }
}

public enum TipoCombustible
{
    Gasolina,
    Diesel,
    Electrico,
    Hibrido
}
```

### Ejemplo de uso con herencia
```csharp
class ProgramaVehiculos
{
    static void Main()
    {
        // Crear diferentes tipos de vehículos
        var vehiculos = new List<Vehiculo>
        {
            new Automovil("Toyota", "Corolla", 2020, 25000, 4, TipoCombustible.Gasolina),
            new Automovil("Tesla", "Model 3", 2023, 45000, 4, TipoCombustible.Electrico),
            new Motocicleta("Honda", "CBR600RR", 2022, 12000, 600),
            new Motocicleta("Harley-Davidson", "Street Glide", 2021, 28000, 1745, true)
        };
        
        // Polimorfismo en acción
        foreach (var vehiculo in vehiculos)
        {
            Console.WriteLine(new string('=', 50));
            vehiculo.MostrarInformacion();
            Console.WriteLine($"Tipo: {vehiculo.ObtenerTipoVehiculo()}");
            Console.WriteLine($"Seguro anual: {vehiculo.CalcularSeguro():C}");
            Console.WriteLine();
        }
        
        // Trabajar con tipos específicos
        MostrarEstadisticasAutomoviles(vehiculos.OfType<Automovil>());
    }
    
    static void MostrarEstadisticasAutomoviles(IEnumerable<Automovil> automoviles)
    {
        Console.WriteLine("\n=== ESTADÍSTICAS DE AUTOMÓVILES ===");
        
        var grupos = automoviles.GroupBy(a => a.Combustible);
        
        foreach (var grupo in grupos)
        {
            Console.WriteLine($"\n{grupo.Key}:");
            Console.WriteLine($"  Cantidad: {grupo.Count()}");
            Console.WriteLine($"  Precio promedio: {grupo.Average(a => a.Precio):C}");
            Console.WriteLine($"  Seguro promedio: {grupo.Average(a => a.CalcularSeguro()):C}");
        }
    }
}
```

## 🔄 Polimorfismo

### Métodos virtuales y override
```csharp
public abstract class Empleado
{
    public string Nombre { get; set; }
    public int Id { get; set; }
    public DateTime FechaIngreso { get; set; }
    
    protected Empleado(string nombre, int id)
    {
        Nombre = nombre;
        Id = id;
        FechaIngreso = DateTime.Now;
    }
    
    // Método virtual con implementación base
    public virtual decimal CalcularSalario()
    {
        return 0; // Salario base por defecto
    }
    
    // Método abstracto que debe ser implementado
    public abstract string ObtenerTipoEmpleado();
    
    // Método virtual para bonificaciones
    public virtual decimal CalcularBonificacion()
    {
        int antiguedad = DateTime.Now.Year - FechaIngreso.Year;
        return antiguedad * 100; // $100 por año de antigüedad
    }
    
    public virtual void MostrarInformacion()
    {
        Console.WriteLine($"ID: {Id}");
        Console.WriteLine($"Nombre: {Nombre}");
        Console.WriteLine($"Tipo: {ObtenerTipoEmpleado()}");
        Console.WriteLine($"Fecha ingreso: {FechaIngreso:dd/MM/yyyy}");
        Console.WriteLine($"Salario: {CalcularSalario():C}");
        Console.WriteLine($"Bonificación: {CalcularBonificacion():C}");
    }
}

public class EmpleadoTiempoCompleto : Empleado
{
    public decimal SalarioMensual { get; set; }
    public decimal BonoAnual { get; set; }
    
    public EmpleadoTiempoCompleto(string nombre, int id, decimal salarioMensual, decimal bonoAnual = 0)
        : base(nombre, id)
    {
        SalarioMensual = salarioMensual;
        BonoAnual = bonoAnual;
    }
    
    public override decimal CalcularSalario()
    {
        return SalarioMensual + (BonoAnual / 12); // Salario mensual + bono prorrateado
    }
    
    public override string ObtenerTipoEmpleado()
    {
        return "Tiempo Completo";
    }
    
    public override decimal CalcularBonificacion()
    {
        var bonificacionBase = base.CalcularBonificacion();
        return bonificacionBase + (SalarioMensual * 0.1m); // 10% adicional del salario
    }
}

public class EmpleadoPorHoras : Empleado
{
    public decimal TarifaPorHora { get; set; }
    public int HorasTrabajadasMes { get; set; }
    
    public EmpleadoPorHoras(string nombre, int id, decimal tarifaPorHora)
        : base(nombre, id)
    {
        TarifaPorHora = tarifaPorHora;
    }
    
    public override decimal CalcularSalario()
    {
        decimal salarioBase = TarifaPorHora * HorasTrabajadasMes;
        
        // Tiempo extra (más de 160 horas al mes)
        if (HorasTrabajadasMes > 160)
        {
            int horasExtra = HorasTrabajadasMes - 160;
            salarioBase += horasExtra * TarifaPorHora * 1.5m; // 50% extra
        }
        
        return salarioBase;
    }
    
    public override string ObtenerTipoEmpleado()
    {
        return "Por Horas";
    }
    
    public void RegistrarHoras(int horas)
    {
        HorasTrabajadasMes = horas;
        Console.WriteLine($"Registradas {horas} horas para {Nombre}");
    }
}

public class Consultor : Empleado
{
    public decimal TarifaPorProyecto { get; set; }
    public int ProyectosCompletados { get; set; }
    
    public Consultor(string nombre, int id, decimal tarifaPorProyecto)
        : base(nombre, id)
    {
        TarifaPorProyecto = tarifaPorProyecto;
    }
    
    public override decimal CalcularSalario()
    {
        return TarifaPorProyecto * ProyectosCompletados;
    }
    
    public override string ObtenerTipoEmpleado()
    {
        return "Consultor";
    }
    
    public override decimal CalcularBonificacion()
    {
        // Los consultores tienen bonificación por volumen de proyectos
        if (ProyectosCompletados >= 5)
            return TarifaPorProyecto * 0.5m; // 50% de un proyecto como bono
        
        return base.CalcularBonificacion();
    }
    
    public void CompletarProyecto()
    {
        ProyectosCompletados++;
        Console.WriteLine($"{Nombre} completó un proyecto. Total: {ProyectosCompletados}");
    }
}
```

## 🔗 Interfaces

### Definición e implementación
```csharp
// Interface para objetos que pueden ser notificados
public interface INotificable
{
    string Email { get; set; }
    bool RecibirNotificaciones { get; set; }
    
    void EnviarNotificacion(string mensaje);
    Task EnviarNotificacionAsync(string mensaje);
}

// Interface para cálculos de impuestos
public interface ICalculadoraImpuestos
{
    decimal CalcularImpuesto(decimal monto);
    decimal CalcularDescuentos(decimal monto);
}

// Interface para auditoría
public interface IAuditable
{
    DateTime FechaCreacion { get; }
    DateTime? FechaModificacion { get; }
    string UsuarioCreacion { get; }
    string? UsuarioModificacion { get; }
    
    void RegistrarModificacion(string usuario);
}

// Clase que implementa múltiples interfaces
public class Cliente : INotificable, ICalculadoraImpuestos, IAuditable
{
    // Propiedades básicas
    public int Id { get; set; }
    public string Nombre { get; set; } = string.Empty;
    public string Telefono { get; set; } = string.Empty;
    public TipoCliente Tipo { get; set; }
    
    // Implementación de INotificable
    public string Email { get; set; } = string.Empty;
    public bool RecibirNotificaciones { get; set; } = true;
    
    // Implementación de IAuditable
    public DateTime FechaCreacion { get; private set; }
    public DateTime? FechaModificacion { get; private set; }
    public string UsuarioCreacion { get; private set; } = string.Empty;
    public string? UsuarioModificacion { get; private set; }
    
    // Constructor
    public Cliente(string nombre, string email, string usuarioCreacion)
    {
        Nombre = nombre;
        Email = email;
        UsuarioCreacion = usuarioCreacion;
        FechaCreacion = DateTime.Now;
    }
    
    // Implementación de INotificable
    public void EnviarNotificacion(string mensaje)
    {
        if (!RecibirNotificaciones || string.IsNullOrEmpty(Email))
        {
            Console.WriteLine($"Notificación no enviada a {Nombre}");
            return;
        }
        
        Console.WriteLine($"📧 Email enviado a {Email}: {mensaje}");
    }
    
    public async Task EnviarNotificacionAsync(string mensaje)
    {
        if (!RecibirNotificaciones || string.IsNullOrEmpty(Email))
            return;
        
        // Simular envío asíncrono
        await Task.Delay(100);
        Console.WriteLine($"📧 Email async enviado a {Email}: {mensaje}");
    }
    
    // Implementación de ICalculadoraImpuestos
    public decimal CalcularImpuesto(decimal monto)
    {
        return Tipo switch
        {
            TipoCliente.Regular => monto * 0.15m,      // 15%
            TipoCliente.Premium => monto * 0.12m,      // 12%
            TipoCliente.VIP => monto * 0.10m,          // 10%
            TipoCliente.Corporativo => monto * 0.18m,  // 18%
            _ => monto * 0.15m
        };
    }
    
    public decimal CalcularDescuentos(decimal monto)
    {
        return Tipo switch
        {
            TipoCliente.Premium => monto * 0.05m,      // 5%
            TipoCliente.VIP => monto * 0.10m,          // 10%
            TipoCliente.Corporativo => monto * 0.08m,  // 8%
            _ => 0m
        };
    }
    
    // Implementación de IAuditable
    public void RegistrarModificacion(string usuario)
    {
        UsuarioModificacion = usuario;
        FechaModificacion = DateTime.Now;
    }
    
    public override string ToString()
    {
        return $"{Nombre} ({Email}) - {Tipo}";
    }
}

public enum TipoCliente
{
    Regular,
    Premium,
    VIP,
    Corporativo
}
```

## 🛡️ Principios SOLID

### 1. Single Responsibility Principle (SRP)
```csharp
// ❌ Clase que viola SRP (múltiples responsabilidades)
public class UsuarioBad
{
    public string Nombre { get; set; }
    public string Email { get; set; }
    
    // Responsabilidad 1: Validación
    public bool ValidarEmail()
    {
        return Email.Contains("@");
    }
    
    // Responsabilidad 2: Persistencia
    public void GuardarEnBaseDatos()
    {
        // Código para guardar en BD
    }
    
    // Responsabilidad 3: Notificación
    public void EnviarEmailBienvenida()
    {
        // Código para enviar email
    }
}

// ✅ Separando responsabilidades
public class Usuario
{
    public string Nombre { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
}

public class ValidadorEmail
{
    public bool EsEmailValido(string email)
    {
        return !string.IsNullOrEmpty(email) && 
               email.Contains("@") && 
               email.Contains(".");
    }
}

public class RepositorioUsuario
{
    public void Guardar(Usuario usuario)
    {
        // Lógica de persistencia
        Console.WriteLine($"Usuario {usuario.Nombre} guardado en BD");
    }
}

public class ServicioNotificaciones
{
    public void EnviarBienvenida(Usuario usuario)
    {
        // Lógica de notificación
        Console.WriteLine($"Email de bienvenida enviado a {usuario.Email}");
    }
}
```

### 2. Open/Closed Principle (OCP)
```csharp
// Abstracción para cálculo de descuentos
public abstract class CalculadorDescuento
{
    public abstract decimal Calcular(decimal monto);
}

// Implementaciones específicas (cerradas para modificación, abiertas para extensión)
public class DescuentoRegular : CalculadorDescuento
{
    public override decimal Calcular(decimal monto)
    {
        return monto * 0.05m; // 5%
    }
}

public class DescuentoPremium : CalculadorDescuento
{
    public override decimal Calcular(decimal monto)
    {
        return monto * 0.15m; // 15%
    }
}

public class DescuentoVIP : CalculadorDescuento
{
    public override decimal Calcular(decimal monto)
    {
        return monto * 0.25m; // 25%
    }
}

// Nueva implementación sin modificar código existente
public class DescuentoEmpleado : CalculadorDescuento
{
    public override decimal Calcular(decimal monto)
    {
        return monto * 0.30m; // 30%
    }
}

public class ProcesadorPedido
{
    public decimal ProcesarPedido(decimal monto, CalculadorDescuento calculador)
    {
        decimal descuento = calculador.Calcular(monto);
        return monto - descuento;
    }
}
```

### 3. Liskov Substitution Principle (LSP)
```csharp
// Clase base que define comportamiento común
public abstract class Ave
{
    public string Nombre { get; set; } = string.Empty;
    public abstract void Mover();
    public virtual void Comer() => Console.WriteLine($"{Nombre} está comiendo");
}

// Subclases que mantienen el contrato de la clase base
public class Aguila : Ave
{
    public override void Mover()
    {
        Console.WriteLine($"{Nombre} está volando alto");
    }
}

public class Pinguino : Ave
{
    public override void Mover()
    {
        Console.WriteLine($"{Nombre} está nadando"); // No viola LSP
    }
}

// Interface separada para aves que vuelan
public interface IVolador
{
    void Volar();
    int AltitudMaxima { get; }
}

public class AguilaVoladora : Ave, IVolador
{
    public int AltitudMaxima => 3000;
    
    public override void Mover()
    {
        Volar();
    }
    
    public void Volar()
    {
        Console.WriteLine($"{Nombre} está volando a {AltitudMaxima}m de altura");
    }
}
```

### 4. Interface Segregation Principle (ISP)
```csharp
// ❌ Interface muy grande que viola ISP
public interface ITrabajadorBad
{
    void Trabajar();
    void Comer();
    void Dormir();
    void ProgramarComputadoras();
    void VenderProductos();
}

// ✅ Interfaces segregadas y específicas
public interface ITrabajador
{
    void Trabajar();
}

public interface IPersona
{
    void Comer();
    void Dormir();
}

public interface IProgramador
{
    void ProgramarComputadoras();
    void DepurarCodigo();
}

public interface IVendedor
{
    void VenderProductos();
    void AtenderClientes();
}

// Implementaciones específicas
public class Programador : ITrabajador, IPersona, IProgramador
{
    public string Nombre { get; set; } = string.Empty;
    
    public void Trabajar() => Console.WriteLine($"{Nombre} está trabajando");
    public void Comer() => Console.WriteLine($"{Nombre} está comiendo");
    public void Dormir() => Console.WriteLine($"{Nombre} está durmiendo");
    public void ProgramarComputadoras() => Console.WriteLine($"{Nombre} está programando");
    public void DepurarCodigo() => Console.WriteLine($"{Nombre} está depurando código");
}

public class Vendedor : ITrabajador, IPersona, IVendedor
{
    public string Nombre { get; set; } = string.Empty;
    
    public void Trabajar() => Console.WriteLine($"{Nombre} está trabajando");
    public void Comer() => Console.WriteLine($"{Nombre} está comiendo");
    public void Dormir() => Console.WriteLine($"{Nombre} está durmiendo");
    public void VenderProductos() => Console.WriteLine($"{Nombre} está vendiendo");
    public void AtenderClientes() => Console.WriteLine($"{Nombre} está atendiendo clientes");
}
```

### 5. Dependency Inversion Principle (DIP)
```csharp
// Abstracción para persistencia
public interface IRepositorio<T>
{
    Task<T?> ObtenerPorIdAsync(int id);
    Task<IEnumerable<T>> ObtenerTodosAsync();
    Task GuardarAsync(T entidad);
    Task EliminarAsync(int id);
}

// Implementación específica
public class RepositorioSQL<T> : IRepositorio<T> where T : class
{
    private readonly string _connectionString;
    
    public RepositorioSQL(string connectionString)
    {
        _connectionString = connectionString;
    }
    
    public Task<T?> ObtenerPorIdAsync(int id)
    {
        Console.WriteLine($"Obteniendo {typeof(T).Name} con ID {id} desde SQL");
        // Implementación SQL
        return Task.FromResult<T?>(null);
    }
    
    public Task<IEnumerable<T>> ObtenerTodosAsync()
    {
        Console.WriteLine($"Obteniendo todos los {typeof(T).Name} desde SQL");
        return Task.FromResult(Enumerable.Empty<T>());
    }
    
    public Task GuardarAsync(T entidad)
    {
        Console.WriteLine($"Guardando {typeof(T).Name} en SQL");
        return Task.CompletedTask;
    }
    
    public Task EliminarAsync(int id)
    {
        Console.WriteLine($"Eliminando {typeof(T).Name} con ID {id} de SQL");
        return Task.CompletedTask;
    }
}

// Servicio que depende de la abstracción, no de implementación concreta
public class ServicioUsuario
{
    private readonly IRepositorio<Usuario> _repositorio;
    private readonly ValidadorEmail _validador;
    
    // Dependency Injection
    public ServicioUsuario(IRepositorio<Usuario> repositorio, ValidadorEmail validador)
    {
        _repositorio = repositorio;
        _validador = validador;
    }
    
    public async Task<bool> CrearUsuarioAsync(string nombre, string email)
    {
        if (!_validador.EsEmailValido(email))
        {
            Console.WriteLine("Email inválido");
            return false;
        }
        
        var usuario = new Usuario { Nombre = nombre, Email = email };
        await _repositorio.GuardarAsync(usuario);
        
        Console.WriteLine($"Usuario {nombre} creado exitosamente");
        return true;
    }
}

// Uso con inyección de dependencias
class ProgramSOLID
{
    static async Task Main()
    {
        // Configurar dependencias
        IRepositorio<Usuario> repositorio = new RepositorioSQL<Usuario>("connectionString");
        var validador = new ValidadorEmail();
        
        // Crear servicio con dependencias inyectadas
        var servicioUsuario = new ServicioUsuario(repositorio, validador);
        
        // Usar el servicio
        await servicioUsuario.CrearUsuarioAsync("Juan Pérez", "juan@email.com");
    }
}
```

---

**Anterior**: [01. Introducción](./01-introduccion.md) | **Siguiente**: [03. Colecciones y Programación Funcional](./03-colecciones.md) | **Inicio**: [README](../README.md)
