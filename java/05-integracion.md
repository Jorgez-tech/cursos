# 05. Librerías y APIs Avanzadas

## 📦 Collections Framework

### Jerarquía de Collections
```java
import java.util.*;

public class CollectionsDemo {
    public static void main(String[] args) {
        // Diferentes tipos de Collections
        demostrarListas();
        demostrarSets();
        demostrarMaps();
        demostrarQueues();
    }
    
    public static void demostrarListas() {
        System.out.println("=== LISTAS ===");
        
        // ArrayList - Rápido acceso por índice
        List<String> arrayList = new ArrayList<>();
        arrayList.add("Java");
        arrayList.add("Python");
        arrayList.add("JavaScript");
        arrayList.add("Java"); // Permite duplicados
        
        System.out.println("ArrayList: " + arrayList);
        System.out.println("Elemento en índice 1: " + arrayList.get(1));
        
        // LinkedList - Rápida inserción/eliminación
        List<String> linkedList = new LinkedList<>();
        linkedList.addAll(arrayList);
        linkedList.add(0, "C++"); // Insertar al inicio
        
        System.out.println("LinkedList: " + linkedList);
        
        // Vector - Thread-safe (sincronizado)
        Vector<Integer> vector = new Vector<>();
        for (int i = 1; i <= 5; i++) {
            vector.add(i * 10);
        }
        System.out.println("Vector: " + vector);
    }
    
    public static void demostrarSets() {
        System.out.println("\n=== SETS ===");
        
        // HashSet - Sin orden, no duplicados
        Set<String> hashSet = new HashSet<>();
        hashSet.add("Manzana");
        hashSet.add("Banana");
        hashSet.add("Cereza");
        hashSet.add("Manzana"); // No se agrega (duplicado)
        
        System.out.println("HashSet: " + hashSet);
        
        // LinkedHashSet - Mantiene orden de inserción
        Set<String> linkedHashSet = new LinkedHashSet<>();
        linkedHashSet.add("Primero");
        linkedHashSet.add("Segundo");
        linkedHashSet.add("Tercero");
        
        System.out.println("LinkedHashSet: " + linkedHashSet);
        
        // TreeSet - Orden natural (ordenado)
        Set<Integer> treeSet = new TreeSet<>();
        treeSet.add(50);
        treeSet.add(20);
        treeSet.add(80);
        treeSet.add(10);
        
        System.out.println("TreeSet: " + treeSet); // Automáticamente ordenado
    }
    
    public static void demostrarMaps() {
        System.out.println("\n=== MAPS ===");
        
        // HashMap - Clave-valor sin orden
        Map<String, Integer> hashMap = new HashMap<>();
        hashMap.put("Java", 1995);
        hashMap.put("Python", 1991);
        hashMap.put("JavaScript", 1995);
        hashMap.put("C++", 1985);
        
        System.out.println("HashMap: " + hashMap);
        System.out.println("Java creado en: " + hashMap.get("Java"));
        
        // TreeMap - Claves ordenadas
        Map<String, Integer> treeMap = new TreeMap<>(hashMap);
        System.out.println("TreeMap (ordenado): " + treeMap);
        
        // Iterar sobre Map
        System.out.println("\nIterando sobre el Map:");
        for (Map.Entry<String, Integer> entry : hashMap.entrySet()) {
            System.out.printf("%s: %d%n", entry.getKey(), entry.getValue());
        }
    }
    
    public static void demostrarQueues() {
        System.out.println("\n=== QUEUES ===");
        
        // LinkedList como Queue
        Queue<String> queue = new LinkedList<>();
        queue.offer("Primero en llegar");
        queue.offer("Segundo en llegar");
        queue.offer("Tercero en llegar");
        
        System.out.println("Queue: " + queue);
        System.out.println("Poll (sacar): " + queue.poll()); // FIFO
        System.out.println("Queue después: " + queue);
        
        // PriorityQueue - Cola con prioridad
        PriorityQueue<Integer> priorityQueue = new PriorityQueue<>();
        priorityQueue.offer(30);
        priorityQueue.offer(10);
        priorityQueue.offer(20);
        
        System.out.println("PriorityQueue: " + priorityQueue);
        System.out.println("Poll con prioridad: " + priorityQueue.poll()); // Menor primero
    }
}
```

## 🧬 Generics

### Uso básico de Generics
```java
// Clase genérica personalizada
public class Caja<T> {
    private T contenido;
    
    public Caja(T contenido) {
        this.contenido = contenido;
    }
    
    public T getContenido() {
        return contenido;
    }
    
    public void setContenido(T contenido) {
        this.contenido = contenido;
    }
    
    public boolean estaVacia() {
        return contenido == null;
    }
    
    @Override
    public String toString() {
        return "Caja{contenido=" + contenido + "}";
    }
}

// Métodos genéricos
public class UtilGenerico {
    // Método genérico para intercambiar elementos de un array
    public static <T> void intercambiar(T[] array, int i, int j) {
        if (i >= 0 && i < array.length && j >= 0 && j < array.length) {
            T temp = array[i];
            array[i] = array[j];
            array[j] = temp;
        }
    }
    
    // Método genérico con bounded type parameter
    public static <T extends Number> double sumar(T[] numeros) {
        double suma = 0.0;
        for (T numero : numeros) {
            suma += numero.doubleValue();
        }
        return suma;
    }
    
    // Método con wildcards
    public static void imprimirLista(List<?> lista) {
        for (Object elemento : lista) {
            System.out.println(elemento);
        }
    }
    
    // Wildcard con upper bound
    public static double sumarNumeros(List<? extends Number> numeros) {
        double suma = 0.0;
        for (Number numero : numeros) {
            suma += numero.doubleValue();
        }
        return suma;
    }
    
    public static void main(String[] args) {
        // Usar clase genérica
        Caja<String> cajaTexto = new Caja<>("Hola Mundo");
        Caja<Integer> cajaNumero = new Caja<>(42);
        
        System.out.println(cajaTexto);
        System.out.println(cajaNumero);
        
        // Usar método genérico
        String[] palabras = {"Java", "Python", "C++"};
        System.out.println("Antes: " + Arrays.toString(palabras));
        intercambiar(palabras, 0, 2);
        System.out.println("Después: " + Arrays.toString(palabras));
        
        // Usar bounded type parameter
        Integer[] enteros = {1, 2, 3, 4, 5};
        Double[] decimales = {1.5, 2.5, 3.5};
        
        System.out.println("Suma enteros: " + sumar(enteros));
        System.out.println("Suma decimales: " + sumar(decimales));
        
        // Usar wildcards
        List<String> listaTexto = Arrays.asList("a", "b", "c");
        List<Integer> listaNumeros = Arrays.asList(1, 2, 3);
        
        imprimirLista(listaTexto);
        imprimirLista(listaNumeros);
        
        System.out.println("Suma con wildcard: " + sumarNumeros(listaNumeros));
    }
}
```

## 🌊 Stream API

### Operaciones básicas con Streams
```java
import java.util.*;
import java.util.stream.*;

public class StreamDemo {
    public static void main(String[] args) {
        List<Persona> personas = crearPersonas();
        
        demostrarOperacionesBasicas(personas);
        demostrarFiltradoYMapeo(personas);
        demostrarAgrupacion(personas);
        demostrarReducciones(personas);
    }
    
    public static List<Persona> crearPersonas() {
        return Arrays.asList(
            new Persona("Ana", 25, "Madrid", 50000),
            new Persona("Carlos", 30, "Barcelona", 60000),
            new Persona("Elena", 28, "Valencia", 55000),
            new Persona("David", 35, "Madrid", 70000),
            new Persona("Sofia", 27, "Barcelona", 58000),
            new Persona("Miguel", 32, "Valencia", 62000)
        );
    }
    
    public static void demostrarOperacionesBasicas(List<Persona> personas) {
        System.out.println("=== OPERACIONES BÁSICAS ===");
        
        // Filtrar y mostrar
        System.out.println("Personas mayores de 30:");
        personas.stream()
                .filter(p -> p.getEdad() > 30)
                .forEach(System.out::println);
        
        // Contar elementos
        long mayoresDe30 = personas.stream()
                                  .filter(p -> p.getEdad() > 30)
                                  .count();
        System.out.println("Cantidad de personas mayores de 30: " + mayoresDe30);
        
        // Encontrar primer elemento
        Optional<Persona> primeraMadrid = personas.stream()
                                                 .filter(p -> "Madrid".equals(p.getCiudad()))
                                                 .findFirst();
        primeraMadrid.ifPresent(p -> System.out.println("Primera persona de Madrid: " + p));
    }
    
    public static void demostrarFiltradoYMapeo(List<Persona> personas) {
        System.out.println("\n=== FILTRADO Y MAPEO ===");
        
        // Mapear a nombres y convertir a lista
        List<String> nombres = personas.stream()
                                      .map(Persona::getNombre)
                                      .collect(Collectors.toList());
        System.out.println("Nombres: " + nombres);
        
        // Mapear salarios con aumento del 10%
        List<Double> salariosAumentados = personas.stream()
                                                 .mapToDouble(Persona::getSalario)
                                                 .map(salario -> salario * 1.10)
                                                 .boxed()
                                                 .collect(Collectors.toList());
        System.out.println("Salarios con aumento del 10%: " + salariosAumentados);
        
        // Filtrar, mapear y ordenar
        List<String> nombresJovenes = personas.stream()
                                            .filter(p -> p.getEdad() < 30)
                                            .map(Persona::getNombre)
                                            .sorted()
                                            .collect(Collectors.toList());
        System.out.println("Nombres de jóvenes (ordenados): " + nombresJovenes);
    }
    
    public static void demostrarAgrupacion(List<Persona> personas) {
        System.out.println("\n=== AGRUPACIÓN ===");
        
        // Agrupar por ciudad
        Map<String, List<Persona>> porCiudad = personas.stream()
                                                      .collect(Collectors.groupingBy(Persona::getCiudad));
        System.out.println("Agrupación por ciudad:");
        porCiudad.forEach((ciudad, lista) -> {
            System.out.println(ciudad + ": " + lista.size() + " personas");
        });
        
        // Agrupar por rango de edad
        Map<String, List<Persona>> porEdad = personas.stream()
                                                    .collect(Collectors.groupingBy(p -> {
                                                        if (p.getEdad() < 30) return "Jóvenes";
                                                        else if (p.getEdad() < 35) return "Adultos";
                                                        else return "Mayores";
                                                    }));
        System.out.println("\nAgrupación por edad:");
        porEdad.forEach((grupo, lista) -> {
            System.out.println(grupo + ": " + lista.size() + " personas");
        });
        
        // Contar por ciudad
        Map<String, Long> contarPorCiudad = personas.stream()
                                                   .collect(Collectors.groupingBy(
                                                       Persona::getCiudad,
                                                       Collectors.counting()
                                                   ));
        System.out.println("\nConteo por ciudad: " + contarPorCiudad);
    }
    
    public static void demostrarReducciones(List<Persona> personas) {
        System.out.println("\n=== REDUCCIONES ===");
        
        // Suma de salarios
        double totalSalarios = personas.stream()
                                      .mapToDouble(Persona::getSalario)
                                      .sum();
        System.out.println("Total de salarios: " + totalSalarios);
        
        // Promedio de edad
        OptionalDouble promedioEdad = personas.stream()
                                             .mapToInt(Persona::getEdad)
                                             .average();
        promedioEdad.ifPresent(promedio -> 
            System.out.println("Promedio de edad: " + promedio));
        
        // Persona con mayor salario
        Optional<Persona> mayorSalario = personas.stream()
                                                .max(Comparator.comparing(Persona::getSalario));
        mayorSalario.ifPresent(p -> 
            System.out.println("Mayor salario: " + p));
        
        // Reducción personalizada: concatenar nombres
        String nombresConcat = personas.stream()
                                      .map(Persona::getNombre)
                                      .reduce("", (a, b) -> a.isEmpty() ? b : a + ", " + b);
        System.out.println("Nombres concatenados: " + nombresConcat);
        
        // Estadísticas de salarios
        DoubleSummaryStatistics estadisticas = personas.stream()
                                                      .mapToDouble(Persona::getSalario)
                                                      .summaryStatistics();
        System.out.println("Estadísticas de salarios:");
        System.out.println("  Mínimo: " + estadisticas.getMin());
        System.out.println("  Máximo: " + estadisticas.getMax());
        System.out.println("  Promedio: " + estadisticas.getAverage());
        System.out.println("  Suma: " + estadisticas.getSum());
        System.out.println("  Cantidad: " + estadisticas.getCount());
    }
}

// Clase auxiliar
class Persona {
    private String nombre;
    private int edad;
    private String ciudad;
    private double salario;
    
    public Persona(String nombre, int edad, String ciudad, double salario) {
        this.nombre = nombre;
        this.edad = edad;
        this.ciudad = ciudad;
        this.salario = salario;
    }
    
    // Getters
    public String getNombre() { return nombre; }
    public int getEdad() { return edad; }
    public String getCiudad() { return ciudad; }
    public double getSalario() { return salario; }
    
    @Override
    public String toString() {
        return String.format("Persona{nombre='%s', edad=%d, ciudad='%s', salario=%.0f}", 
                           nombre, edad, ciudad, salario);
    }
}
```

## ⚠️ Manejo Avanzado de Excepciones

### Jerarquía de excepciones personalizada
```java
// Excepciones personalizadas
class SistemaException extends Exception {
    public SistemaException(String mensaje) {
        super(mensaje);
    }
    
    public SistemaException(String mensaje, Throwable causa) {
        super(mensaje, causa);
    }
}

class ValidacionException extends SistemaException {
    public ValidacionException(String mensaje) {
        super("Error de validación: " + mensaje);
    }
}

class AccesoException extends SistemaException {
    public AccesoException(String mensaje) {
        super("Error de acceso: " + mensaje);
    }
}

// Gestor de excepciones robusto
public class GestorCuentaBancaria {
    private Map<String, Double> cuentas;
    private Set<String> cuentasBloqueadas;
    
    public GestorCuentaBancaria() {
        this.cuentas = new HashMap<>();
        this.cuentasBloqueadas = new HashSet<>();
    }
    
    public void crearCuenta(String numeroCuenta, double saldoInicial) 
            throws ValidacionException {
        // Validaciones con excepciones específicas
        if (numeroCuenta == null || numeroCuenta.trim().isEmpty()) {
            throw new ValidacionException("Número de cuenta no puede estar vacío");
        }
        
        if (cuentas.containsKey(numeroCuenta)) {
            throw new ValidacionException("La cuenta ya existe: " + numeroCuenta);
        }
        
        if (saldoInicial < 0) {
            throw new ValidacionException("Saldo inicial no puede ser negativo");
        }
        
        cuentas.put(numeroCuenta, saldoInicial);
        System.out.println("Cuenta creada: " + numeroCuenta + " con saldo: " + saldoInicial);
    }
    
    public void transferir(String cuentaOrigen, String cuentaDestino, double monto) 
            throws SistemaException {
        try {
            // Validar cuentas existen
            validarCuentaExiste(cuentaOrigen);
            validarCuentaExiste(cuentaDestino);
            
            // Validar acceso
            validarAcceso(cuentaOrigen);
            validarAcceso(cuentaDestino);
            
            // Validar monto
            if (monto <= 0) {
                throw new ValidacionException("Monto debe ser positivo");
            }
            
            // Validar saldo suficiente
            double saldoOrigen = cuentas.get(cuentaOrigen);
            if (saldoOrigen < monto) {
                throw new ValidacionException("Saldo insuficiente en cuenta " + cuentaOrigen);
            }
            
            // Realizar transferencia
            cuentas.put(cuentaOrigen, saldoOrigen - monto);
            cuentas.put(cuentaDestino, cuentas.get(cuentaDestino) + monto);
            
            System.out.printf("Transferencia exitosa: $%.2f de %s a %s%n", 
                             monto, cuentaOrigen, cuentaDestino);
            
        } catch (ValidacionException | AccesoException e) {
            // Re-lanzar excepciones conocidas
            throw e;
        } catch (Exception e) {
            // Envolver excepciones inesperadas
            throw new SistemaException("Error inesperado en transferencia", e);
        }
    }
    
    private void validarCuentaExiste(String numeroCuenta) throws ValidacionException {
        if (!cuentas.containsKey(numeroCuenta)) {
            throw new ValidacionException("Cuenta no existe: " + numeroCuenta);
        }
    }
    
    private void validarAcceso(String numeroCuenta) throws AccesoException {
        if (cuentasBloqueadas.contains(numeroCuenta)) {
            throw new AccesoException("Cuenta bloqueada: " + numeroCuenta);
        }
    }
    
    public void bloquearCuenta(String numeroCuenta) throws ValidacionException {
        validarCuentaExiste(numeroCuenta);
        cuentasBloqueadas.add(numeroCuenta);
        System.out.println("Cuenta bloqueada: " + numeroCuenta);
    }
    
    public double consultarSaldo(String numeroCuenta) throws SistemaException {
        validarCuentaExiste(numeroCuenta);
        validarAcceso(numeroCuenta);
        return cuentas.get(numeroCuenta);
    }
    
    // Método con try-with-resources para recursos automáticos
    public void exportarEstadoCuentas(String archivo) throws SistemaException {
        try (FileWriter writer = new FileWriter(archivo);
             PrintWriter printWriter = new PrintWriter(writer)) {
            
            printWriter.println("=== ESTADO DE CUENTAS ===");
            printWriter.println("Fecha: " + java.time.LocalDateTime.now());
            printWriter.println();
            
            for (Map.Entry<String, Double> entry : cuentas.entrySet()) {
                String estado = cuentasBloqueadas.contains(entry.getKey()) ? "BLOQUEADA" : "ACTIVA";
                printWriter.printf("%s: $%.2f [%s]%n", 
                                 entry.getKey(), entry.getValue(), estado);
            }
            
            System.out.println("Estado exportado a: " + archivo);
            
        } catch (IOException e) {
            throw new SistemaException("Error al exportar estado de cuentas", e);
        }
    }
    
    public static void main(String[] args) {
        GestorCuentaBancaria gestor = new GestorCuentaBancaria();
        
        try {
            // Crear cuentas
            gestor.crearCuenta("001", 1000.0);
            gestor.crearCuenta("002", 500.0);
            
            // Transferencia exitosa
            gestor.transferir("001", "002", 200.0);
            
            // Bloquear cuenta
            gestor.bloquearCuenta("002");
            
            // Intentar transferir a cuenta bloqueada (genera excepción)
            gestor.transferir("001", "002", 100.0);
            
        } catch (ValidacionException e) {
            System.err.println("Error de validación: " + e.getMessage());
        } catch (AccesoException e) {
            System.err.println("Error de acceso: " + e.getMessage());
        } catch (SistemaException e) {
            System.err.println("Error del sistema: " + e.getMessage());
            if (e.getCause() != null) {
                System.err.println("Causa: " + e.getCause().getMessage());
            }
        }
        
        // Exportar estado final
        try {
            gestor.exportarEstadoCuentas("estado_cuentas.txt");
        } catch (SistemaException e) {
            System.err.println("Error al exportar: " + e.getMessage());
        }
    }
}
```

## 📁 Manejo de Archivos y NIO.2

### Operaciones modernas con archivos
```java
import java.nio.file.*;
import java.nio.charset.StandardCharsets;
import java.io.*;
import java.util.*;
import java.util.stream.Stream;

public class ManejoArchivosModerno {
    
    public static void main(String[] args) {
        Path directorioTrabajo = Paths.get("datos");
        
        try {
            demostrarOperacionesBasicas(directorioTrabajo);
            demostrarLecturaEscritura(directorioTrabajo);
            demostrarBusquedaArchivos(directorioTrabajo);
            demostrarWatchService(directorioTrabajo);
        } catch (IOException e) {
            System.err.println("Error: " + e.getMessage());
        }
    }
    
    public static void demostrarOperacionesBasicas(Path directorio) throws IOException {
        System.out.println("=== OPERACIONES BÁSICAS ===");
        
        // Crear directorio si no existe
        if (!Files.exists(directorio)) {
            Files.createDirectories(directorio);
            System.out.println("Directorio creado: " + directorio);
        }
        
        // Crear archivo de ejemplo
        Path archivo = directorio.resolve("ejemplo.txt");
        if (!Files.exists(archivo)) {
            Files.createFile(archivo);
            System.out.println("Archivo creado: " + archivo);
        }
        
        // Información del archivo
        System.out.println("Información del archivo:");
        System.out.println("  Existe: " + Files.exists(archivo));
        System.out.println("  Es archivo: " + Files.isRegularFile(archivo));
        System.out.println("  Es directorio: " + Files.isDirectory(archivo));
        System.out.println("  Tamaño: " + Files.size(archivo) + " bytes");
        System.out.println("  Última modificación: " + Files.getLastModifiedTime(archivo));
        
        // Atributos del archivo
        try {
            System.out.println("  Legible: " + Files.isReadable(archivo));
            System.out.println("  Escribible: " + Files.isWritable(archivo));
            System.out.println("  Ejecutable: " + Files.isExecutable(archivo));
        } catch (Exception e) {
            System.out.println("  No se pudieron obtener todos los atributos");
        }
    }
    
    public static void demostrarLecturaEscritura(Path directorio) throws IOException {
        System.out.println("\n=== LECTURA Y ESCRITURA ===");
        
        Path archivo = directorio.resolve("datos.txt");
        
        // Escribir datos
        List<String> lineas = Arrays.asList(
            "Primera línea",
            "Segunda línea con números: 123",
            "Tercera línea con caracteres especiales: ñáéíóú",
            "Cuarta línea: " + java.time.LocalDateTime.now()
        );
        
        Files.write(archivo, lineas, StandardCharsets.UTF_8,
                   StandardOpenOption.CREATE, StandardOpenOption.TRUNCATE_EXISTING);
        System.out.println("Datos escritos en: " + archivo);
        
        // Leer todas las líneas
        List<String> lineasLeidas = Files.readAllLines(archivo, StandardCharsets.UTF_8);
        System.out.println("\nContenido leído:");
        lineasLeidas.forEach(linea -> System.out.println("  " + linea));
        
        // Leer como Stream (eficiente para archivos grandes)
        System.out.println("\nLeyendo con Stream:");
        try (Stream<String> stream = Files.lines(archivo)) {
            stream.filter(linea -> linea.contains("línea"))
                  .forEach(linea -> System.out.println("  Filtrada: " + linea));
        }
        
        // Agregar datos al archivo
        String nuevaLinea = "Línea agregada: " + System.currentTimeMillis();
        Files.write(archivo, Arrays.asList(nuevaLinea), StandardCharsets.UTF_8,
                   StandardOpenOption.APPEND);
        
        // Leer solo el contenido agregado
        List<String> todasLineas = Files.readAllLines(archivo);
        System.out.println("Nueva línea agregada: " + todasLineas.get(todasLineas.size() - 1));
    }
    
    public static void demostrarBusquedaArchivos(Path directorio) throws IOException {
        System.out.println("\n=== BÚSQUEDA DE ARCHIVOS ===");
        
        // Crear algunos archivos de prueba
        crearArchivosPrueba(directorio);
        
        // Listar contenido del directorio
        System.out.println("Contenido del directorio:");
        try (Stream<Path> paths = Files.list(directorio)) {
            paths.forEach(path -> {
                try {
                    String tipo = Files.isDirectory(path) ? "[DIR]" : "[FILE]";
                    long tamaño = Files.isDirectory(path) ? 0 : Files.size(path);
                    System.out.printf("  %s %s (%d bytes)%n", 
                                     tipo, path.getFileName(), tamaño);
                } catch (IOException e) {
                    System.out.println("  Error al leer: " + path.getFileName());
                }
            });
        }
        
        // Buscar archivos por extensión
        System.out.println("\nArchivos .txt:");
        try (Stream<Path> paths = Files.walk(directorio)) {
            paths.filter(Files::isRegularFile)
                 .filter(path -> path.toString().endsWith(".txt"))
                 .forEach(path -> System.out.println("  " + path.getFileName()));
        }
        
        // Buscar archivos por tamaño
        System.out.println("\nArchivos mayores a 10 bytes:");
        try (Stream<Path> paths = Files.walk(directorio)) {
            paths.filter(Files::isRegularFile)
                 .filter(path -> {
                     try {
                         return Files.size(path) > 10;
                     } catch (IOException e) {
                         return false;
                     }
                 })
                 .forEach(path -> {
                     try {
                         System.out.printf("  %s (%d bytes)%n", 
                                          path.getFileName(), Files.size(path));
                     } catch (IOException e) {
                         System.out.println("  Error: " + path.getFileName());
                     }
                 });
        }
    }
    
    private static void crearArchivosPrueba(Path directorio) throws IOException {
        // Crear subdirectorio
        Path subdir = directorio.resolve("subdir");
        Files.createDirectories(subdir);
        
        // Crear archivos de diferentes tipos
        Files.write(directorio.resolve("archivo1.txt"), 
                   "Contenido del archivo 1".getBytes());
        Files.write(directorio.resolve("archivo2.log"), 
                   "Log de prueba".getBytes());
        Files.write(subdir.resolve("archivo3.txt"), 
                   "Contenido en subdirectorio".getBytes());
    }
    
    public static void demostrarWatchService(Path directorio) throws IOException {
        System.out.println("\n=== WATCH SERVICE ===");
        System.out.println("Monitoreando cambios en: " + directorio);
        System.out.println("(El programa terminará automáticamente después de 10 segundos)");
        
        try (WatchService watchService = FileSystems.getDefault().newWatchService()) {
            
            // Registrar el directorio para monitoreo
            directorio.register(watchService, 
                               StandardWatchEventKinds.ENTRY_CREATE,
                               StandardWatchEventKinds.ENTRY_DELETE,
                               StandardWatchEventKinds.ENTRY_MODIFY);
            
            // Crear un archivo para generar un evento
            new Thread(() -> {
                try {
                    Thread.sleep(2000);
                    Path archivoNuevo = directorio.resolve("archivo_nuevo.txt");
                    Files.write(archivoNuevo, "Archivo creado automáticamente".getBytes());
                    System.out.println("Archivo creado automáticamente: " + archivoNuevo.getFileName());
                    
                    Thread.sleep(2000);
                    Files.delete(archivoNuevo);
                    System.out.println("Archivo eliminado automáticamente: " + archivoNuevo.getFileName());
                } catch (Exception e) {
                    System.err.println("Error en hilo de prueba: " + e.getMessage());
                }
            }).start();
            
            // Monitorear eventos por tiempo limitado
            long tiempoInicio = System.currentTimeMillis();
            while (System.currentTimeMillis() - tiempoInicio < 10000) { // 10 segundos
                try {
                    WatchKey key = watchService.poll(1, java.util.concurrent.TimeUnit.SECONDS);
                    if (key != null) {
                        for (WatchEvent<?> event : key.pollEvents()) {
                            WatchEvent.Kind<?> kind = event.kind();
                            Path archivo = (Path) event.context();
                            
                            System.out.printf("Evento: %s - Archivo: %s%n", 
                                             kind.name(), archivo);
                        }
                        
                        if (!key.reset()) {
                            break;
                        }
                    }
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    break;
                }
            }
        }
        
        System.out.println("Monitoreo terminado.");
    }
}
```

---

**Anterior**: [04. Programación Orientada a Objetos](./04-practica.md) | **Siguiente**: [06. Desarrollo de Aplicaciones](./06-documentacion.md) | **Inicio**: [README](../README.md)
