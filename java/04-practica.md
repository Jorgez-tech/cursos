# 04. Programación Orientada a Objetos

## 🏗️ Fundamentos de POO en Java

### Principios básicos de la POO
1. **Encapsulamiento**: Ocultar detalles internos y exponer interfaz controlada
2. **Herencia**: Reutilizar código mediante relaciones "es-un"
3. **Polimorfismo**: Un objeto puede tomar múltiples formas
4. **Abstracción**: Representar conceptos esenciales ocultando complejidad

## 📦 Clases y Objetos

### Definición de una clase
```java
public class Persona {
    // Atributos (variables de instancia)
    private String nombre;
    private int edad;
    private String email;
    
    // Constructor por defecto
    public Persona() {
        this.nombre = "Sin nombre";
        this.edad = 0;
        this.email = "sin@email.com";
    }
    
    // Constructor parametrizado
    public Persona(String nombre, int edad, String email) {
        this.nombre = nombre;
        this.edad = edad;
        this.email = email;
    }
    
    // Métodos getter
    public String getNombre() {
        return nombre;
    }
    
    public int getEdad() {
        return edad;
    }
    
    public String getEmail() {
        return email;
    }
    
    // Métodos setter
    public void setNombre(String nombre) {
        if (nombre != null && !nombre.trim().isEmpty()) {
            this.nombre = nombre;
        }
    }
    
    public void setEdad(int edad) {
        if (edad >= 0 && edad <= 150) {
            this.edad = edad;
        }
    }
    
    public void setEmail(String email) {
        if (email != null && email.contains("@")) {
            this.email = email;
        }
    }
    
    // Métodos de comportamiento
    public void saludar() {
        System.out.println("Hola, soy " + nombre + " y tengo " + edad + " años");
    }
    
    public boolean esMayorDeEdad() {
        return edad >= 18;
    }
    
    // Método toString
    @Override
    public String toString() {
        return String.format("Persona{nombre='%s', edad=%d, email='%s'}", 
                           nombre, edad, email);
    }
    
    // Método equals
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        
        Persona persona = (Persona) obj;
        return edad == persona.edad &&
               nombre.equals(persona.nombre) &&
               email.equals(persona.email);
    }
    
    // Método hashCode
    @Override
    public int hashCode() {
        return Objects.hash(nombre, edad, email);
    }
}
```

### Uso de la clase
```java
public class TestPersona {
    public static void main(String[] args) {
        // Crear objetos
        Persona persona1 = new Persona();
        Persona persona2 = new Persona("Ana García", 25, "ana@email.com");
        
        // Usar métodos
        persona1.setNombre("Carlos López");
        persona1.setEdad(30);
        persona1.setEmail("carlos@email.com");
        
        // Mostrar información
        System.out.println(persona1);
        System.out.println(persona2);
        
        persona1.saludar();
        persona2.saludar();
        
        // Verificar mayor de edad
        System.out.println(persona1.getNombre() + " es mayor de edad: " + 
                         persona1.esMayorDeEdad());
    }
}
```

## 🧬 Herencia

### Clase padre (superclase)
```java
public class Vehiculo {
    protected String marca;
    protected String modelo;
    protected int año;
    protected double precio;
    
    public Vehiculo(String marca, String modelo, int año, double precio) {
        this.marca = marca;
        this.modelo = modelo;
        this.año = año;
        this.precio = precio;
    }
    
    public void arrancar() {
        System.out.println("El vehículo está arrancando...");
    }
    
    public void detener() {
        System.out.println("El vehículo se está deteniendo...");
    }
    
    public void mostrarInfo() {
        System.out.printf("%s %s (%d) - $%.2f%n", 
                         marca, modelo, año, precio);
    }
    
    // Getters y setters
    public String getMarca() { return marca; }
    public String getModelo() { return modelo; }
    public int getAño() { return año; }
    public double getPrecio() { return precio; }
}
```

### Clases hijas (subclases)
```java
public class Automovil extends Vehiculo {
    private int numeroPuertas;
    private String tipoTransmision;
    
    public Automovil(String marca, String modelo, int año, double precio,
                     int numeroPuertas, String tipoTransmision) {
        super(marca, modelo, año, precio); // Llamar constructor padre
        this.numeroPuertas = numeroPuertas;
        this.tipoTransmision = tipoTransmision;
    }
    
    @Override
    public void arrancar() {
        System.out.println("El automóvil " + marca + " " + modelo + " está arrancando...");
    }
    
    public void activarAireAcondicionado() {
        System.out.println("Aire acondicionado activado");
    }
    
    @Override
    public void mostrarInfo() {
        super.mostrarInfo();
        System.out.printf("Puertas: %d, Transmisión: %s%n", 
                         numeroPuertas, tipoTransmision);
    }
}

public class Motocicleta extends Vehiculo {
    private int cilindrada;
    private boolean tieneCasco;
    
    public Motocicleta(String marca, String modelo, int año, double precio,
                       int cilindrada) {
        super(marca, modelo, año, precio);
        this.cilindrada = cilindrada;
        this.tieneCasco = false;
    }
    
    @Override
    public void arrancar() {
        if (tieneCasco) {
            System.out.println("La motocicleta está arrancando con seguridad...");
        } else {
            System.out.println("¡PELIGRO! Póngase el casco antes de arrancar");
        }
    }
    
    public void ponerseCasco() {
        this.tieneCasco = true;
        System.out.println("Casco puesto. Listo para conducir con seguridad.");
    }
    
    @Override
    public void mostrarInfo() {
        super.mostrarInfo();
        System.out.printf("Cilindrada: %d cc, Casco: %s%n", 
                         cilindrada, tieneCasco ? "Puesto" : "No puesto");
    }
}
```

## 🎭 Polimorfismo

### Ejemplo de polimorfismo
```java
public class ConcesionarioDemo {
    public static void main(String[] args) {
        // Polimorfismo: referencias del tipo padre apuntando a objetos hijos
        Vehiculo[] vehiculos = {
            new Automovil("Toyota", "Corolla", 2023, 25000.0, 4, "Automática"),
            new Motocicleta("Honda", "CBR600", 2023, 12000.0, 600),
            new Automovil("Ford", "Mustang", 2023, 45000.0, 2, "Manual")
        };
        
        System.out.println("=== INVENTARIO DEL CONCESIONARIO ===");
        
        // Polimorfismo en acción
        for (Vehiculo vehiculo : vehiculos) {
            vehiculo.mostrarInfo(); // Llama al método sobrescrito correspondiente
            vehiculo.arrancar();    // Llama al método sobrescrito correspondiente
            
            // Verificar tipo específico para funcionalidades únicas
            if (vehiculo instanceof Automovil) {
                Automovil auto = (Automovil) vehiculo;
                auto.activarAireAcondicionado();
            } else if (vehiculo instanceof Motocicleta) {
                Motocicleta moto = (Motocicleta) vehiculo;
                moto.ponerseCasco();
            }
            
            vehiculo.detener();
            System.out.println("--------------------");
        }
        
        // Calcular precio promedio
        double promedioPrecios = calcularPrecioPromedio(vehiculos);
        System.out.printf("Precio promedio: $%.2f%n", promedioPrecios);
    }
    
    // Método que acepta cualquier tipo de Vehículo (polimorfismo)
    public static double calcularPrecioPromedio(Vehiculo[] vehiculos) {
        double suma = 0;
        for (Vehiculo vehiculo : vehiculos) {
            suma += vehiculo.getPrecio();
        }
        return suma / vehiculos.length;
    }
}
```

## 📜 Interfaces

### Definición de interfaces
```java
public interface Volador {
    // Constantes (public static final implícito)
    double VELOCIDAD_MAXIMA_CRUCERO = 900.0;
    
    // Métodos abstractos (public abstract implícito)
    void despegar();
    void aterrizar();
    void volar();
    
    // Método por defecto (Java 8+)
    default void mostrarInstrucciones() {
        System.out.println("Instrucciones generales de vuelo:");
        System.out.println("1. Verificar condiciones météorológicas");
        System.out.println("2. Realizar chequeos prevuelo");
        System.out.println("3. Solicitar autorización de despegue");
    }
    
    // Método estático
    static void mostrarRegulaciones() {
        System.out.println("Regulaciones internacionales de aviación...");
    }
}

public interface Acuatico {
    void navegar();
    void anclar();
    double profundidadMaxima();
}
```

### Implementación de interfaces
```java
public class Avion extends Vehiculo implements Volador {
    private String aerolinea;
    private int capacidadPasajeros;
    private double altitudMaxima;
    
    public Avion(String marca, String modelo, int año, double precio,
                 String aerolinea, int capacidadPasajeros, double altitudMaxima) {
        super(marca, modelo, año, precio);
        this.aerolinea = aerolinea;
        this.capacidadPasajeros = capacidadPasajeros;
        this.altitudMaxima = altitudMaxima;
    }
    
    @Override
    public void despegar() {
        System.out.println("Avión " + modelo + " despegando...");
        System.out.println("Velocidad de despegue alcanzada");
    }
    
    @Override
    public void aterrizar() {
        System.out.println("Avión " + modelo + " aterrizando...");
        System.out.println("Tren de aterrizaje desplegado");
    }
    
    @Override
    public void volar() {
        System.out.printf("Volando a %.0f metros de altitud%n", altitudMaxima);
    }
    
    @Override
    public void arrancar() {
        System.out.println("Encendiendo motores del avión...");
    }
}

// Clase que implementa múltiples interfaces
public class Hidroavion extends Vehiculo implements Volador, Acuatico {
    private boolean enAgua;
    
    public Hidroavion(String marca, String modelo, int año, double precio) {
        super(marca, modelo, año, precio);
        this.enAgua = false;
    }
    
    @Override
    public void despegar() {
        if (enAgua) {
            System.out.println("Hidroavión despegando desde el agua...");
        } else {
            System.out.println("Hidroavión despegando desde tierra...");
        }
    }
    
    @Override
    public void aterrizar() {
        System.out.println("Hidroavión aterrizando en el agua...");
        enAgua = true;
    }
    
    @Override
    public void volar() {
        enAgua = false;
        System.out.println("Hidroavión volando sobre el agua...");
    }
    
    @Override
    public void navegar() {
        enAgua = true;
        System.out.println("Navegando como embarcación...");
    }
    
    @Override
    public void anclar() {
        if (enAgua) {
            System.out.println("Anclando en el agua...");
        } else {
            System.out.println("No se puede anclar en tierra");
        }
    }
    
    @Override
    public double profundidadMaxima() {
        return 0.5; // Solo superficie
    }
}
```

## 🔐 Encapsulamiento Avanzado

### Clase con encapsulamiento robusto
```java
public class CuentaBancaria {
    // Atributos privados
    private final String numeroCuenta; // Inmutable
    private String titular;
    private double saldo;
    private boolean activa;
    private static int contadorCuentas = 0; // Variable de clase
    
    // Constructor
    public CuentaBancaria(String titular, double saldoInicial) {
        this.numeroCuenta = generarNumeroCuenta();
        this.titular = titular;
        this.saldo = saldoInicial >= 0 ? saldoInicial : 0;
        this.activa = true;
    }
    
    // Método estático privado
    private static String generarNumeroCuenta() {
        return String.format("CUENTA-%06d", ++contadorCuentas);
    }
    
    // Métodos públicos con validación
    public boolean depositar(double monto) {
        if (!activa) {
            System.out.println("Cuenta inactiva. No se puede depositar.");
            return false;
        }
        
        if (monto <= 0) {
            System.out.println("El monto debe ser positivo.");
            return false;
        }
        
        saldo += monto;
        System.out.printf("Depositado: $%.2f. Saldo actual: $%.2f%n", 
                         monto, saldo);
        return true;
    }
    
    public boolean retirar(double monto) {
        if (!activa) {
            System.out.println("Cuenta inactiva. No se puede retirar.");
            return false;
        }
        
        if (monto <= 0) {
            System.out.println("El monto debe ser positivo.");
            return false;
        }
        
        if (monto > saldo) {
            System.out.println("Saldo insuficiente.");
            return false;
        }
        
        saldo -= monto;
        System.out.printf("Retirado: $%.2f. Saldo actual: $%.2f%n", 
                         monto, saldo);
        return true;
    }
    
    public boolean transferir(CuentaBancaria cuentaDestino, double monto) {
        if (this.retirar(monto)) {
            if (cuentaDestino.depositar(monto)) {
                System.out.printf("Transferencia exitosa a %s por $%.2f%n", 
                                 cuentaDestino.getNumeroCuenta(), monto);
                return true;
            } else {
                // Revertir la transacción
                this.depositar(monto);
                System.out.println("Error en transferencia. Transacción revertida.");
            }
        }
        return false;
    }
    
    // Getters (solo lectura para datos sensibles)
    public String getNumeroCuenta() {
        return numeroCuenta;
    }
    
    public String getTitular() {
        return titular;
    }
    
    public double getSaldo() {
        return saldo;
    }
    
    public boolean isActiva() {
        return activa;
    }
    
    public static int getTotalCuentas() {
        return contadorCuentas;
    }
    
    // Método para desactivar cuenta
    public void desactivar() {
        this.activa = false;
        System.out.println("Cuenta desactivada.");
    }
    
    @Override
    public String toString() {
        return String.format("CuentaBancaria{numero='%s', titular='%s', saldo=%.2f, activa=%s}", 
                           numeroCuenta, titular, saldo, activa);
    }
}
```

## 🎯 Ejercicios prácticos

### Ejercicio 1: Sistema de biblioteca
Crea un sistema de biblioteca con las siguientes clases:
- `Libro` (título, autor, ISBN, disponible)
- `Usuario` (nombre, ID, libros prestados)
- `Biblioteca` (catálogo de libros, usuarios registrados)

### Ejercicio 2: Formas geométricas
Implementa una jerarquía de formas:
- Interface `Figura` con métodos `calcularArea()` y `calcularPerimetro()`
- Clases `Circulo`, `Rectangulo`, `Triangulo`
- Clase `CalculadoraGeometrica` que trabaje con cualquier figura

### Ejercicio 3: Sistema de empleados
Diseña un sistema con:
- Clase base `Empleado`
- Subclases `EmpleadoTiempoCompleto`, `EmpleadoMedioTiempo`, `Contratista`
- Diferentes formas de calcular salario según el tipo

<details>
<summary>💡 Ver solución del Ejercicio 2</summary>

```java
// Interface Figura
public interface Figura {
    double calcularArea();
    double calcularPerimetro();
    void mostrarInfo();
}

// Clase Circulo
public class Circulo implements Figura {
    private double radio;
    
    public Circulo(double radio) {
        this.radio = radio > 0 ? radio : 1.0;
    }
    
    @Override
    public double calcularArea() {
        return Math.PI * radio * radio;
    }
    
    @Override
    public double calcularPerimetro() {
        return 2 * Math.PI * radio;
    }
    
    @Override
    public void mostrarInfo() {
        System.out.printf("Círculo - Radio: %.2f, Área: %.2f, Perímetro: %.2f%n",
                         radio, calcularArea(), calcularPerimetro());
    }
}

// Clase Rectangulo
public class Rectangulo implements Figura {
    private double largo;
    private double ancho;
    
    public Rectangulo(double largo, double ancho) {
        this.largo = largo > 0 ? largo : 1.0;
        this.ancho = ancho > 0 ? ancho : 1.0;
    }
    
    @Override
    public double calcularArea() {
        return largo * ancho;
    }
    
    @Override
    public double calcularPerimetro() {
        return 2 * (largo + ancho);
    }
    
    @Override
    public void mostrarInfo() {
        System.out.printf("Rectángulo - Largo: %.2f, Ancho: %.2f, Área: %.2f, Perímetro: %.2f%n",
                         largo, ancho, calcularArea(), calcularPerimetro());
    }
}

// Clase CalculadoraGeometrica
public class CalculadoraGeometrica {
    public static void analizarFiguras(Figura[] figuras) {
        double areaTotal = 0;
        double perimetroTotal = 0;
        
        System.out.println("=== ANÁLISIS DE FIGURAS ===");
        
        for (Figura figura : figuras) {
            figura.mostrarInfo();
            areaTotal += figura.calcularArea();
            perimetroTotal += figura.calcularPerimetro();
        }
        
        System.out.printf("Área total: %.2f%n", areaTotal);
        System.out.printf("Perímetro total: %.2f%n", perimetroTotal);
    }
    
    public static void main(String[] args) {
        Figura[] figuras = {
            new Circulo(5.0),
            new Rectangulo(10.0, 6.0),
            new Circulo(3.0)
        };
        
        analizarFiguras(figuras);
    }
}
```
</details>

---

**Anterior**: [03. Setup y Entorno](./03-setup.md) | **Siguiente**: [05. Librerías y APIs](./05-integracion.md) | **Inicio**: [README](../README.md)
