# 01. Introducción a Java

## 🎯 Historia y evolución de Java

Java fue creado por **James Gosling** en Sun Microsystems en 1995. Inicialmente llamado "Oak", fue diseñado para dispositivos embebidos, pero su verdadero potencial se descubrió en el desarrollo web y empresarial.

### Hitos importantes
- **1995**: Lanzamiento público de Java 1.0
- **1998**: Java 2 (J2SE, J2EE, J2ME)
- **2006**: Java se vuelve open source
- **2010**: Oracle adquiere Sun Microsystems
- **2017**: Java 9 introduce modularidad
- **2018+**: Ciclo de lanzamiento cada 6 meses

## 🚀 Características principales de Java

### Write Once, Run Anywhere (WORA)
```
Código fuente (.java) → Compilador (javac) → Bytecode (.class) → JVM → Ejecución
```

### Características clave
- **Orientado a objetos**: Todo es un objeto (excepto primitivos)
- **Independiente de plataforma**: JVM actúa como capa de abstracción
- **Robusto**: Manejo automático de memoria, verificación de tipos
- **Seguro**: Sandbox de seguridad, verificación de bytecode
- **Multithreaded**: Soporte nativo para concurrencia
- **Alto rendimiento**: JIT compilation, optimizaciones en tiempo de ejecución

## 🛠️ Instalación y configuración del entorno

### Paso 1: Descargar e instalar JDK
```bash
# Verificar si Java está instalado
java -version
javac -version

# Si no está instalado, descargar desde:
# https://www.oracle.com/java/technologies/downloads/
# o usar OpenJDK: https://openjdk.org/
```

### Paso 2: Configurar variables de entorno
**Windows:**
```cmd
set JAVA_HOME=C:\Program Files\Java\jdk-17
set PATH=%JAVA_HOME%\bin;%PATH%
```

**Linux/Mac:**
```bash
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk
export PATH=$JAVA_HOME/bin:$PATH

# Agregar al .bashrc o .zshrc para persistencia
echo 'export JAVA_HOME=/usr/lib/jvm/java-17-openjdk' >> ~/.bashrc
echo 'export PATH=$JAVA_HOME/bin:$PATH' >> ~/.bashrc
```

### Paso 3: Verificar instalación
```bash
java -version
# Debería mostrar: openjdk version "17.0.x" ...

javac -version
# Debería mostrar: javac 17.0.x
```

## 👋 Primer programa: "Hola Mundo"

### Archivo: HolaMundo.java
```java
/**
 * Mi primer programa en Java
 * Autor: [Tu nombre]
 * Fecha: Agosto 2025
 */
public class HolaMundo {
    public static void main(String[] args) {
        System.out.println("¡Hola Mundo desde Java!");
        System.out.println("Java versión: " + System.getProperty("java.version"));
    }
}
```

### Compilación y ejecución
```bash
# 1. Compilar el archivo
javac HolaMundo.java

# 2. Verificar que se creó el archivo .class
ls -la
# Debería aparecer: HolaMundo.class

# 3. Ejecutar el programa
java HolaMundo

# Salida esperada:
# ¡Hola Mundo desde Java!
# Java versión: 17.0.x
```

## 🏗️ Anatomía de un programa Java

### Estructura básica
```java
// 1. Declaración del paquete (opcional)
package com.miempresa.miapp;

// 2. Importación de clases (opcional)
import java.util.Scanner;
import java.time.LocalDate;

// 3. Declaración de la clase (obligatorio)
public class MiPrograma {
    
    // 4. Método main (punto de entrada)
    public static void main(String[] args) {
        // 5. Código del programa
        System.out.println("Mi programa Java");
    }
}
```

### Explicación de componentes

#### Clase
```java
public class MiPrograma {
    // El nombre del archivo debe ser MiPrograma.java
    // Una clase public por archivo
    // El nombre debe empezar con mayúscula (convención)
}
```

#### Método main
```java
public static void main(String[] args) {
    // public: accesible desde cualquier parte
    // static: se puede llamar sin crear objeto
    // void: no retorna valor
    // main: nombre especial reconocido por JVM
    // String[] args: parámetros de línea de comandos
}
```

#### Sentencias
```java
System.out.println("Texto");  // Imprimir con salto de línea
System.out.print("Texto");    // Imprimir sin salto de línea
System.out.printf("Formato %s", variable); // Formato tipo C
```

## 💡 Ejemplo práctico: Calculadora básica

```java
import java.util.Scanner;

public class CalculadoraBasica {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        
        System.out.println("=== Calculadora Básica ===");
        
        // Solicitar números
        System.out.print("Ingrese el primer número: ");
        double num1 = scanner.nextDouble();
        
        System.out.print("Ingrese el segundo número: ");
        double num2 = scanner.nextDouble();
        
        // Realizar operaciones
        double suma = num1 + num2;
        double resta = num1 - num2;
        double multiplicacion = num1 * num2;
        double division = (num2 != 0) ? num1 / num2 : 0;
        
        // Mostrar resultados
        System.out.println("\n=== Resultados ===");
        System.out.printf("%.2f + %.2f = %.2f%n", num1, num2, suma);
        System.out.printf("%.2f - %.2f = %.2f%n", num1, num2, resta);
        System.out.printf("%.2f * %.2f = %.2f%n", num1, num2, multiplicacion);
        
        if (num2 != 0) {
            System.out.printf("%.2f / %.2f = %.2f%n", num1, num2, division);
        } else {
            System.out.println("No se puede dividir por cero");
        }
        
        scanner.close();
    }
}
```

## 🎯 Ejercicios prácticos

### Ejercicio 1: Información personal
Crea un programa que solicite al usuario su nombre, edad y ciudad, y muestre un mensaje personalizado.

### Ejercicio 2: Conversor de temperaturas
Desarrolla un programa que convierta temperaturas entre Celsius, Fahrenheit y Kelvin.

### Ejercicio 3: Tabla de multiplicar
Crea un programa que genere la tabla de multiplicar de un número ingresado por el usuario.

<details>
<summary>💡 Ver solución del Ejercicio 1</summary>

```java
import java.util.Scanner;

public class InformacionPersonal {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        
        System.out.print("¿Cuál es tu nombre? ");
        String nombre = scanner.nextLine();
        
        System.out.print("¿Cuántos años tienes? ");
        int edad = scanner.nextInt();
        scanner.nextLine(); // Limpiar buffer
        
        System.out.print("¿En qué ciudad vives? ");
        String ciudad = scanner.nextLine();
        
        System.out.println("\n=== Información Personal ===");
        System.out.printf("Hola %s!%n", nombre);
        System.out.printf("Tienes %d años%n", edad);
        System.out.printf("Vives en %s%n", ciudad);
        
        // Categorizar por edad
        if (edad < 18) {
            System.out.println("Eres menor de edad");
        } else if (edad < 65) {
            System.out.println("Eres adulto");
        } else {
            System.out.println("Eres adulto mayor");
        }
        
        scanner.close();
    }
}
```
</details>

## 🔧 Herramientas recomendadas

### IDEs principales
1. **IntelliJ IDEA Community** (Gratuito)
   - Excelente autocompletado y refactoring
   - Debugging avanzado
   - Integración con Git

2. **Eclipse IDE** (Gratuito)
   - IDE maduro y estable
   - Gran ecosistema de plugins
   - Bueno para proyectos empresariales

3. **Visual Studio Code** (Gratuito)
   - Ligero y rápido
   - Extension Pack for Java
   - Ideal para proyectos pequeños

### Extensiones útiles para VS Code
```bash
# Instalar Extension Pack for Java
code --install-extension vscjava.vscode-java-pack

# Incluye:
# - Language Support for Java
# - Debugger for Java
# - Test Runner for Java
# - Maven for Java
# - Project Manager for Java
```

---

**Anterior**: [README](../README.md) | **Siguiente**: [02. Conceptos Fundamentales](./02-conceptos.md) | **Inicio**: [README](../README.md)
