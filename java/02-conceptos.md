# 02. Conceptos Fundamentales

## 🏗️ Sintaxis básica de Java

### Estructura de un programa Java
```java
// Comentario de una línea
/* Comentario de 
   múltiples líneas */
/**
 * Comentario de documentación (JavaDoc)
 * @author Tu nombre
 * @version 1.0
 */
package com.empresa.proyecto;

import java.util.Scanner;

public class ConceptosFundamentales {
    public static void main(String[] args) {
        // Código del programa
    }
}
```

### Convenciones de nomenclatura
```java
// Clases: PascalCase
public class MiClase { }

// Métodos y variables: camelCase
public void miMetodo() { }
int miVariable;

// Constantes: UPPER_SNAKE_CASE
public static final int MI_CONSTANTE = 100;

// Paquetes: lowercase con puntos
package com.empresa.proyecto.modulo;
```

## 📊 Tipos de datos primitivos

### Tipos numéricos enteros
```java
byte edad = 25;           // 8 bits: -128 a 127
short año = 2025;         // 16 bits: -32,768 a 32,767
int poblacion = 1000000;  // 32 bits: -2^31 a 2^31-1
long distancia = 150000000000L; // 64 bits: -2^63 a 2^63-1
```

### Tipos numéricos decimales
```java
float precio = 19.99f;    // 32 bits, precisión simple
double pi = 3.14159265359; // 64 bits, precisión doble (recomendado)
```

### Otros tipos primitivos
```java
boolean activo = true;    // true o false
char inicial = 'J';       // 16 bits, caracteres Unicode
char unicode = '\u0041';  // Carácter 'A' en Unicode
```

### Ejemplos prácticos
```java
public class TiposDatos {
    public static void main(String[] args) {
        // Declaración e inicialización
        int edad = 25;
        double salario = 50000.50;
        boolean empleado = true;
        char categoria = 'A';
        
        // Mostrar valores
        System.out.println("Edad: " + edad);
        System.out.println("Salario: $" + salario);
        System.out.println("¿Es empleado?: " + empleado);
        System.out.println("Categoría: " + categoria);
        
        // Mostrar rangos de tipos
        System.out.println("Rango byte: " + Byte.MIN_VALUE + " a " + Byte.MAX_VALUE);
        System.out.println("Rango int: " + Integer.MIN_VALUE + " a " + Integer.MAX_VALUE);
        System.out.println("Rango double: " + Double.MIN_VALUE + " a " + Double.MAX_VALUE);
    }
}
```

## 🔧 Variables y constantes

### Declaración de variables
```java
// Declaración sin inicialización
int numero;
String nombre;

// Declaración con inicialización
int contador = 0;
String mensaje = "Hola Java";

// Múltiples variables del mismo tipo
int a = 1, b = 2, c = 3;
```

### Constantes
```java
// Constante de clase (static final)
public static final double PI = 3.14159;
public static final String EMPRESA = "Mi Empresa";

// Constante local (final)
public void metodo() {
    final int LIMITE = 100;
    // LIMITE = 200; // Error: no se puede reasignar
}
```

### Scope (Alcance) de variables
```java
public class ScopeEjemplo {
    // Variable de clase (static)
    static int variableClase = 1;
    
    // Variable de instancia
    int variableInstancia = 2;
    
    public void metodo() {
        // Variable local
        int variableLocal = 3;
        
        // Variable de bloque
        if (true) {
            int variableBloque = 4;
            System.out.println(variableBloque); // OK
        }
        // System.out.println(variableBloque); // Error: fuera de scope
    }
}
```

## ⚙️ Operadores

### Operadores aritméticos
```java
public class OperadoresAritmeticos {
    public static void main(String[] args) {
        int a = 10, b = 3;
        
        System.out.println("a + b = " + (a + b)); // 13
        System.out.println("a - b = " + (a - b)); // 7
        System.out.println("a * b = " + (a * b)); // 30
        System.out.println("a / b = " + (a / b)); // 3 (división entera)
        System.out.println("a % b = " + (a % b)); // 1 (módulo/resto)
        
        // Para división decimal
        double division = (double) a / b;
        System.out.println("División decimal: " + division); // 3.333...
    }
}
```

### Operadores de asignación
```java
int x = 10;
x += 5;  // x = x + 5  → x = 15
x -= 3;  // x = x - 3  → x = 12
x *= 2;  // x = x * 2  → x = 24
x /= 4;  // x = x / 4  → x = 6
x %= 5;  // x = x % 5  → x = 1
```

### Operadores de incremento/decremento
```java
int contador = 5;

// Pre-incremento: incrementa y luego usa el valor
int a = ++contador; // contador = 6, a = 6

// Post-incremento: usa el valor y luego incrementa
int b = contador++; // b = 6, contador = 7

// Pre-decremento: decrementa y luego usa el valor
int c = --contador; // contador = 6, c = 6

// Post-decremento: usa el valor y luego decrementa
int d = contador--; // d = 6, contador = 5
```

### Operadores de comparación
```java
int x = 10, y = 20;

System.out.println(x == y);  // false (igual a)
System.out.println(x != y);  // true (diferente de)
System.out.println(x < y);   // true (menor que)
System.out.println(x > y);   // false (mayor que)
System.out.println(x <= y);  // true (menor o igual)
System.out.println(x >= y);  // false (mayor o igual)
```

### Operadores lógicos
```java
boolean a = true, b = false;

System.out.println(a && b);  // false (AND lógico)
System.out.println(a || b);  // true (OR lógico)
System.out.println(!a);      // false (NOT lógico)

// Evaluación perezosa (short-circuit)
boolean resultado = (x != 0) && (100 / x > 5);
// Si x == 0, no evalúa la segunda condición
```

## 🔄 Control de flujo

### Condicionales: if-else
```java
public class Condicionales {
    public static void main(String[] args) {
        int edad = 18;
        
        // if simple
        if (edad >= 18) {
            System.out.println("Eres mayor de edad");
        }
        
        // if-else
        if (edad >= 18) {
            System.out.println("Puedes votar");
        } else {
            System.out.println("No puedes votar aún");
        }
        
        // if-else if-else
        if (edad < 13) {
            System.out.println("Niño");
        } else if (edad < 18) {
            System.out.println("Adolescente");
        } else if (edad < 65) {
            System.out.println("Adulto");
        } else {
            System.out.println("Adulto mayor");
        }
    }
}
```

### Operador ternario
```java
int edad = 20;
String categoria = (edad >= 18) ? "Adulto" : "Menor";
System.out.println("Categoría: " + categoria);

// Anidado (no recomendado para legibilidad)
String tipo = (edad < 13) ? "Niño" : (edad < 18) ? "Adolescente" : "Adulto";
```

### Switch
```java
public class Switch {
    public static void main(String[] args) {
        int dia = 3;
        String nombreDia;
        
        // Switch tradicional
        switch (dia) {
            case 1:
                nombreDia = "Lunes";
                break;
            case 2:
                nombreDia = "Martes";
                break;
            case 3:
                nombreDia = "Miércoles";
                break;
            case 4:
                nombreDia = "Jueves";
                break;
            case 5:
                nombreDia = "Viernes";
                break;
            case 6:
            case 7:
                nombreDia = "Fin de semana";
                break;
            default:
                nombreDia = "Día inválido";
        }
        
        System.out.println("Día: " + nombreDia);
        
        // Switch expression (Java 14+)
        String tipoJornada = switch (dia) {
            case 1, 2, 3, 4, 5 -> "Día laboral";
            case 6, 7 -> "Fin de semana";
            default -> "Día inválido";
        };
        
        System.out.println("Tipo: " + tipoJornada);
    }
}
```

### Bucles: for
```java
public class BucleFor {
    public static void main(String[] args) {
        // for tradicional
        System.out.println("Números del 1 al 5:");
        for (int i = 1; i <= 5; i++) {
            System.out.println("Número: " + i);
        }
        
        // for anidado - tabla de multiplicar
        System.out.println("\nTabla del 5:");
        for (int i = 1; i <= 10; i++) {
            int resultado = 5 * i;
            System.out.printf("5 x %d = %d%n", i, resultado);
        }
        
        // for con arrays
        int[] numeros = {10, 20, 30, 40, 50};
        System.out.println("\nArray con for tradicional:");
        for (int i = 0; i < numeros.length; i++) {
            System.out.println("Índice " + i + ": " + numeros[i]);
        }
        
        // for-each (enhanced for)
        System.out.println("\nArray con for-each:");
        for (int numero : numeros) {
            System.out.println("Valor: " + numero);
        }
    }
}
```

### Bucles: while y do-while
```java
public class BuclesWhile {
    public static void main(String[] args) {
        // while - puede no ejecutarse nunca
        int contador = 1;
        System.out.println("Bucle while:");
        while (contador <= 3) {
            System.out.println("Iteración: " + contador);
            contador++;
        }
        
        // do-while - se ejecuta al menos una vez
        int numero = 10;
        System.out.println("\nBucle do-while:");
        do {
            System.out.println("Número: " + numero);
            numero++;
        } while (numero <= 12);
        
        // Ejemplo práctico: validación de entrada
        Scanner scanner = new Scanner(System.in);
        int edad;
        
        do {
            System.out.print("Ingrese una edad válida (1-120): ");
            edad = scanner.nextInt();
            
            if (edad < 1 || edad > 120) {
                System.out.println("Edad inválida. Intente nuevamente.");
            }
        } while (edad < 1 || edad > 120);
        
        System.out.println("Edad válida ingresada: " + edad);
        scanner.close();
    }
}
```

### Control de bucles: break y continue
```java
public class ControlBucles {
    public static void main(String[] args) {
        // break - sale del bucle
        System.out.println("Ejemplo con break:");
        for (int i = 1; i <= 10; i++) {
            if (i == 6) {
                break; // Sale del bucle cuando i = 6
            }
            System.out.println("i = " + i);
        }
        
        // continue - salta a la siguiente iteración
        System.out.println("\nEjemplo con continue:");
        for (int i = 1; i <= 10; i++) {
            if (i % 2 == 0) {
                continue; // Salta números pares
            }
            System.out.println("Número impar: " + i);
        }
        
        // Etiquetas con break y continue
        System.out.println("\nEjemplo con etiquetas:");
        externo:
        for (int i = 1; i <= 3; i++) {
            for (int j = 1; j <= 3; j++) {
                if (i == 2 && j == 2) {
                    break externo; // Sale de ambos bucles
                }
                System.out.printf("i=%d, j=%d%n", i, j);
            }
        }
    }
}
```

## 🎯 Ejercicio práctico: Sistema de calificaciones

```java
import java.util.Scanner;

public class SistemaCalificaciones {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        
        System.out.println("=== Sistema de Calificaciones ===");
        
        // Solicitar número de estudiantes
        System.out.print("¿Cuántos estudiantes hay? ");
        int numEstudiantes = scanner.nextInt();
        
        double sumaNotas = 0;
        int aprobados = 0;
        int reprobados = 0;
        
        // Procesar cada estudiante
        for (int i = 1; i <= numEstudiantes; i++) {
            System.out.printf("\nEstudiante %d:%n", i);
            System.out.print("Ingrese la nota (0-100): ");
            double nota = scanner.nextDouble();
            
            // Validar nota
            while (nota < 0 || nota > 100) {
                System.out.print("Nota inválida. Ingrese nuevamente (0-100): ");
                nota = scanner.nextDouble();
            }
            
            // Calcular letra y estado
            char letra;
            String estado;
            
            if (nota >= 90) {
                letra = 'A';
                estado = "Excelente";
            } else if (nota >= 80) {
                letra = 'B';
                estado = "Bueno";
            } else if (nota >= 70) {
                letra = 'C';
                estado = "Regular";
            } else if (nota >= 60) {
                letra = 'D';
                estado = "Suficiente";
            } else {
                letra = 'F';
                estado = "Reprobado";
            }
            
            // Mostrar resultado
            System.out.printf("Nota: %.1f - Letra: %c - Estado: %s%n", 
                            nota, letra, estado);
            
            // Acumular estadísticas
            sumaNotas += nota;
            if (nota >= 60) {
                aprobados++;
            } else {
                reprobados++;
            }
        }
        
        // Mostrar estadísticas finales
        double promedio = sumaNotas / numEstudiantes;
        double porcentajeAprobados = (double) aprobados / numEstudiantes * 100;
        
        System.out.println("\n=== Estadísticas Finales ===");
        System.out.printf("Promedio del grupo: %.2f%n", promedio);
        System.out.printf("Estudiantes aprobados: %d (%.1f%%)%n", 
                         aprobados, porcentajeAprobados);
        System.out.printf("Estudiantes reprobados: %d (%.1f%%)%n", 
                         reprobados, 100 - porcentajeAprobados);
        
        scanner.close();
    }
}
```

## 🎯 Ejercicios para práctica

### Ejercicio 1: Calculadora de IMC
Crea un programa que calcule el Índice de Masa Corporal y determine la categoría.

### Ejercicio 2: Generador de números primos
Desarrolla un programa que genere todos los números primos hasta un límite dado.

### Ejercicio 3: Juego de adivinanza
Implementa un juego donde el usuario debe adivinar un número aleatorio.

### Ejercicio 4: Conversión de bases numéricas
Crea un programa que convierta números entre diferentes bases (binario, octal, decimal, hexadecimal).

---

**Anterior**: [01. Introducción](./01-introduccion.md) | **Siguiente**: [03. Setup y Entorno](./03-setup.md)
