# 06. Desarrollo de Aplicaciones

## 🧵 Concurrencia y Multithreading

### Conceptos básicos de hilos
```java
// Método 1: Extender Thread
class MiHilo extends Thread {
    private String nombre;
    
    public MiHilo(String nombre) {
        this.nombre = nombre;
    }
    
    @Override
    public void run() {
        for (int i = 1; i <= 5; i++) {
            System.out.println(nombre + " - Iteración: " + i);
            try {
                Thread.sleep(1000); // Pausa 1 segundo
            } catch (InterruptedException e) {
                System.out.println(nombre + " fue interrumpido");
                return;
            }
        }
        System.out.println(nombre + " terminó");
    }
}

// Método 2: Implementar Runnable
class MiTarea implements Runnable {
    private String nombre;
    
    public MiTarea(String nombre) {
        this.nombre = nombre;
    }
    
    @Override
    public void run() {
        for (int i = 1; i <= 3; i++) {
            System.out.println(nombre + " ejecutando tarea " + i);
            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                return;
            }
        }
    }
}

public class EjemploHilos {
    public static void main(String[] args) throws InterruptedException {
        System.out.println("=== Ejemplo de Hilos ===");
        
        // Crear y ejecutar hilos
        MiHilo hilo1 = new MiHilo("Hilo-1");
        MiHilo hilo2 = new MiHilo("Hilo-2");
        
        Thread hilo3 = new Thread(new MiTarea("Tarea-1"));
        Thread hilo4 = new Thread(new MiTarea("Tarea-2"));
        
        // Iniciar hilos
        hilo1.start();
        hilo2.start();
        hilo3.start();
        hilo4.start();
        
        // Esperar que terminen
        hilo1.join();
        hilo2.join();
        hilo3.join();
        hilo4.join();
        
        System.out.println("Todos los hilos han terminado");
    }
}
```

### Sincronización y thread safety
```java
class Contador {
    private int valor = 0;
    
    // Método sincronizado
    public synchronized void incrementar() {
        valor++;
    }
    
    // Bloque sincronizado
    public void decrementar() {
        synchronized(this) {
            valor--;
        }
    }
    
    public synchronized int getValor() {
        return valor;
    }
}

class TareaContador implements Runnable {
    private Contador contador;
    private boolean incrementar;
    
    public TareaContador(Contador contador, boolean incrementar) {
        this.contador = contador;
        this.incrementar = incrementar;
    }
    
    @Override
    public void run() {
        for (int i = 0; i < 1000; i++) {
            if (incrementar) {
                contador.incrementar();
            } else {
                contador.decrementar();
            }
        }
    }
}

public class EjemploSincronizacion {
    public static void main(String[] args) throws InterruptedException {
        Contador contador = new Contador();
        
        Thread t1 = new Thread(new TareaContador(contador, true));
        Thread t2 = new Thread(new TareaContador(contador, false));
        Thread t3 = new Thread(new TareaContador(contador, true));
        
        t1.start();
        t2.start();
        t3.start();
        
        t1.join();
        t2.join();
        t3.join();
        
        System.out.println("Valor final del contador: " + contador.getValor());
    }
}
```

### ExecutorService y thread pools
```java
import java.util.concurrent.*;
import java.util.List;
import java.util.ArrayList;

class TareaCalculadora implements Callable<Integer> {
    private int numero;
    
    public TareaCalculadora(int numero) {
        this.numero = numero;
    }
    
    @Override
    public Integer call() throws Exception {
        // Simular trabajo pesado
        Thread.sleep(1000);
        int resultado = numero * numero;
        System.out.println("Calculando " + numero + "² = " + resultado + 
                          " en hilo: " + Thread.currentThread().getName());
        return resultado;
    }
}

public class EjemploExecutorService {
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        // Crear pool de hilos
        ExecutorService executor = Executors.newFixedThreadPool(3);
        
        // Enviar tareas
        List<Future<Integer>> futures = new ArrayList<>();
        for (int i = 1; i <= 5; i++) {
            Future<Integer> future = executor.submit(new TareaCalculadora(i));
            futures.add(future);
        }
        
        // Recoger resultados
        System.out.println("=== Resultados ===");
        for (int i = 0; i < futures.size(); i++) {
            Integer resultado = futures.get(i).get(); // Bloquea hasta obtener resultado
            System.out.println("Resultado " + (i + 1) + ": " + resultado);
        }
        
        // Cerrar executor
        executor.shutdown();
        executor.awaitTermination(5, TimeUnit.SECONDS);
        System.out.println("Todas las tareas completadas");
    }
}
```

## 🖥️ Interfaces gráficas con Swing

### Ventana básica
```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;

public class VentanaBasica extends JFrame {
    
    public VentanaBasica() {
        configurarVentana();
        crearComponentes();
    }
    
    private void configurarVentana() {
        setTitle("Mi Primera Aplicación Swing");
        setSize(400, 300);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLocationRelativeTo(null); // Centrar ventana
        setLayout(new BorderLayout());
    }
    
    private void crearComponentes() {
        // Panel superior con etiqueta
        JPanel panelSuperior = new JPanel();
        JLabel etiqueta = new JLabel("¡Bienvenido a Java Swing!");
        etiqueta.setFont(new Font("Arial", Font.BOLD, 16));
        panelSuperior.add(etiqueta);
        
        // Panel central con botones
        JPanel panelCentral = new JPanel(new GridLayout(2, 2, 10, 10));
        
        JButton botonSaludo = new JButton("Saludar");
        JButton botonHora = new JButton("Mostrar Hora");
        JButton botonColor = new JButton("Cambiar Color");
        JButton botonLimpiar = new JButton("Limpiar");
        
        panelCentral.add(botonSaludo);
        panelCentral.add(botonHora);
        panelCentral.add(botonColor);
        panelCentral.add(botonLimpiar);
        
        // Panel inferior con área de texto
        JTextArea areaTexto = new JTextArea(5, 30);
        areaTexto.setEditable(false);
        areaTexto.setBorder(BorderFactory.createTitledBorder("Output"));
        JScrollPane scrollPane = new JScrollPane(areaTexto);
        
        // Eventos de botones
        botonSaludo.addActionListener(e -> 
            areaTexto.append("¡Hola desde Java!\n"));
        
        botonHora.addActionListener(e -> 
            areaTexto.append("Hora actual: " + new java.util.Date() + "\n"));
        
        botonColor.addActionListener(e -> {
            Color colorAleatorio = new Color(
                (int)(Math.random() * 256),
                (int)(Math.random() * 256),
                (int)(Math.random() * 256)
            );
            panelCentral.setBackground(colorAleatorio);
            areaTexto.append("Color cambiado\n");
        });
        
        botonLimpiar.addActionListener(e -> areaTexto.setText(""));
        
        // Agregar componentes a la ventana
        add(panelSuperior, BorderLayout.NORTH);
        add(panelCentral, BorderLayout.CENTER);
        add(scrollPane, BorderLayout.SOUTH);
    }
    
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            try {
                UIManager.setLookAndFeel(UIManager.getSystemLookAndFeel());
            } catch (Exception e) {
                e.printStackTrace();
            }
            
            new VentanaBasica().setVisible(true);
        });
    }
}
```

### Aplicación completa: Calculadora
```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;

public class Calculadora extends JFrame implements ActionListener {
    private JTextField display;
    private double num1 = 0, num2 = 0;
    private String operador = "";
    private boolean nuevaOperacion = true;
    
    public Calculadora() {
        configurarVentana();
        crearInterfaz();
    }
    
    private void configurarVentana() {
        setTitle("Calculadora Java");
        setSize(300, 400);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLocationRelativeTo(null);
        setResizable(false);
    }
    
    private void crearInterfaz() {
        setLayout(new BorderLayout());
        
        // Display
        display = new JTextField("0");
        display.setFont(new Font("Arial", Font.BOLD, 24));
        display.setHorizontalAlignment(JTextField.RIGHT);
        display.setEditable(false);
        display.setBackground(Color.WHITE);
        add(display, BorderLayout.NORTH);
        
        // Panel de botones
        JPanel panelBotones = new JPanel(new GridLayout(5, 4, 5, 5));
        panelBotones.setBorder(BorderFactory.createEmptyBorder(10, 10, 10, 10));
        
        String[] botones = {
            "C", "CE", "⌫", "/",
            "7", "8", "9", "*",
            "4", "5", "6", "-",
            "1", "2", "3", "+",
            "±", "0", ".", "="
        };
        
        for (String texto : botones) {
            JButton boton = new JButton(texto);
            boton.setFont(new Font("Arial", Font.BOLD, 18));
            boton.addActionListener(this);
            
            // Colores diferentes para tipos de botones
            if ("+-*/=".contains(texto)) {
                boton.setBackground(new Color(255, 149, 0));
                boton.setForeground(Color.WHITE);
            } else if ("C CE ⌫".contains(texto)) {
                boton.setBackground(new Color(160, 160, 160));
                boton.setForeground(Color.BLACK);
            } else {
                boton.setBackground(new Color(230, 230, 230));
                boton.setForeground(Color.BLACK);
            }
            
            panelBotones.add(boton);
        }
        
        add(panelBotones, BorderLayout.CENTER);
    }
    
    @Override
    public void actionPerformed(ActionEvent e) {
        String comando = e.getActionCommand();
        
        try {
            if ("0123456789".contains(comando)) {
                manejarNumero(comando);
            } else if ("+-*/".contains(comando)) {
                manejarOperador(comando);
            } else if ("=".equals(comando)) {
                calcular();
            } else if ("C".equals(comando)) {
                limpiarTodo();
            } else if ("CE".equals(comando)) {
                limpiarEntrada();
            } else if ("⌫".equals(comando)) {
                borrarUltimoDigito();
            } else if (".".equals(comando)) {
                agregarDecimal();
            } else if ("±".equals(comando)) {
                cambiarSigno();
            }
        } catch (Exception ex) {
            display.setText("Error");
            limpiarTodo();
        }
    }
    
    private void manejarNumero(String numero) {
        if (nuevaOperacion) {
            display.setText(numero);
            nuevaOperacion = false;
        } else {
            String textoActual = display.getText();
            if (!textoActual.equals("0")) {
                display.setText(textoActual + numero);
            } else {
                display.setText(numero);
            }
        }
    }
    
    private void manejarOperador(String op) {
        if (!operador.isEmpty() && !nuevaOperacion) {
            calcular();
        }
        
        num1 = Double.parseDouble(display.getText());
        operador = op;
        nuevaOperacion = true;
    }
    
    private void calcular() {
        if (operador.isEmpty()) return;
        
        num2 = Double.parseDouble(display.getText());
        double resultado = 0;
        
        switch (operador) {
            case "+": resultado = num1 + num2; break;
            case "-": resultado = num1 - num2; break;
            case "*": resultado = num1 * num2; break;
            case "/": 
                if (num2 != 0) {
                    resultado = num1 / num2;
                } else {
                    throw new ArithmeticException("División por cero");
                }
                break;
        }
        
        display.setText(formatearResultado(resultado));
        operador = "";
        nuevaOperacion = true;
    }
    
    private String formatearResultado(double numero) {
        if (numero == (long) numero) {
            return String.valueOf((long) numero);
        } else {
            return String.valueOf(numero);
        }
    }
    
    private void limpiarTodo() {
        display.setText("0");
        num1 = 0;
        num2 = 0;
        operador = "";
        nuevaOperacion = true;
    }
    
    private void limpiarEntrada() {
        display.setText("0");
        nuevaOperacion = true;
    }
    
    private void borrarUltimoDigito() {
        String texto = display.getText();
        if (texto.length() > 1) {
            display.setText(texto.substring(0, texto.length() - 1));
        } else {
            display.setText("0");
        }
    }
    
    private void agregarDecimal() {
        String texto = display.getText();
        if (!texto.contains(".")) {
            display.setText(texto + ".");
            nuevaOperacion = false;
        }
    }
    
    private void cambiarSigno() {
        double numero = Double.parseDouble(display.getText());
        numero = -numero;
        display.setText(formatearResultado(numero));
    }
    
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            try {
                UIManager.setLookAndFeel(UIManager.getSystemLookAndFeel());
            } catch (Exception e) {
                e.printStackTrace();
            }
            new Calculadora().setVisible(true);
        });
    }
}
```

## 🗄️ Conexión a bases de datos con JDBC

### Configuración y conexión
```java
import java.sql.*;
import java.util.ArrayList;
import java.util.List;

public class ConexionBD {
    private static final String URL = "jdbc:sqlite:productos.db";
    
    public static Connection obtenerConexion() throws SQLException {
        return DriverManager.getConnection(URL);
    }
    
    public static void crearTablas() {
        String sqlProductos = """
            CREATE TABLE IF NOT EXISTS productos (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                nombre TEXT NOT NULL,
                categoria TEXT NOT NULL,
                precio REAL NOT NULL,
                stock INTEGER NOT NULL,
                fecha_creacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            )
        """;
        
        try (Connection conn = obtenerConexion();
             Statement stmt = conn.createStatement()) {
            
            stmt.execute(sqlProductos);
            System.out.println("Tabla productos creada exitosamente");
            
        } catch (SQLException e) {
            System.err.println("Error al crear tablas: " + e.getMessage());
        }
    }
}
```

### Operaciones CRUD
```java
class ProductoDAO {
    
    public boolean insertarProducto(Producto producto) {
        String sql = "INSERT INTO productos (nombre, categoria, precio, stock) VALUES (?, ?, ?, ?)";
        
        try (Connection conn = ConexionBD.obtenerConexion();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            pstmt.setString(1, producto.getNombre());
            pstmt.setString(2, producto.getCategoria());
            pstmt.setDouble(3, producto.getPrecio());
            pstmt.setInt(4, producto.getStock());
            
            int filasAfectadas = pstmt.executeUpdate();
            return filasAfectadas > 0;
            
        } catch (SQLException e) {
            System.err.println("Error al insertar producto: " + e.getMessage());
            return false;
        }
    }
    
    public List<Producto> obtenerTodosLosProductos() {
        List<Producto> productos = new ArrayList<>();
        String sql = "SELECT * FROM productos ORDER BY nombre";
        
        try (Connection conn = ConexionBD.obtenerConexion();
             Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery(sql)) {
            
            while (rs.next()) {
                Producto producto = new Producto(
                    rs.getInt("id"),
                    rs.getString("nombre"),
                    rs.getString("categoria"),
                    rs.getDouble("precio"),
                    rs.getInt("stock")
                );
                productos.add(producto);
            }
            
        } catch (SQLException e) {
            System.err.println("Error al obtener productos: " + e.getMessage());
        }
        
        return productos;
    }
    
    public boolean actualizarStock(int productorId, int nuevoStock) {
        String sql = "UPDATE productos SET stock = ? WHERE id = ?";
        
        try (Connection conn = ConexionBD.obtenerConexion();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            pstmt.setInt(1, nuevoStock);
            pstmt.setInt(2, productorId);
            
            int filasAfectadas = pstmt.executeUpdate();
            return filasAfectadas > 0;
            
        } catch (SQLException e) {
            System.err.println("Error al actualizar stock: " + e.getMessage());
            return false;
        }
    }
    
    public boolean eliminarProducto(int id) {
        String sql = "DELETE FROM productos WHERE id = ?";
        
        try (Connection conn = ConexionBD.obtenerConexion();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            pstmt.setInt(1, id);
            
            int filasAfectadas = pstmt.executeUpdate();
            return filasAfectadas > 0;
            
        } catch (SQLException e) {
            System.err.println("Error al eliminar producto: " + e.getMessage());
            return false;
        }
    }
}
```

## 🧪 Testing con JUnit 5

### Dependencias Maven
```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>5.9.2</version>
    <scope>test</scope>
</dependency>
```

### Tests básicos
```java
import org.junit.jupiter.api.*;
import static org.junit.jupiter.api.Assertions.*;

class CalculadoraTest {
    private Calculadora calc;
    
    @BeforeEach
    void setUp() {
        calc = new Calculadora();
    }
    
    @Test
    @DisplayName("Suma de números positivos")
    void testSumaPositivos() {
        assertEquals(8, calc.sumar(3, 5));
        assertEquals(10, calc.sumar(4, 6));
    }
    
    @Test
    @DisplayName("División por cero debe lanzar excepción")
    void testDivisionPorCero() {
        assertThrows(ArithmeticException.class, () -> {
            calc.dividir(10, 0);
        });
    }
    
    @ParameterizedTest
    @ValueSource(ints = {-1, 0, 1, 5, 100})
    void testNumeroAlCuadrado(int numero) {
        assertTrue(calc.alCuadrado(numero) >= 0);
    }
    
    @Test
    @Timeout(value = 2, unit = TimeUnit.SECONDS)
    void testOperacionRapida() {
        // Test que debe completarse en menos de 2 segundos
        calc.operacionCompleja();
    }
}
```

## 🎯 Proyecto integrador: Sistema de gestión completo

```java
// Aplicación principal que integra GUI + BD + Threading
public class SistemaGestionApp extends JFrame {
    private ProductoDAO productoDAO;
    private JTable tablaProductos;
    private DefaultTableModel modeloTabla;
    private ExecutorService executor;
    
    public SistemaGestionApp() {
        this.productoDAO = new ProductoDAO();
        this.executor = Executors.newFixedThreadPool(2);
        
        ConexionBD.crearTablas();
        configurarVentana();
        crearInterfaz();
        cargarDatos();
    }
    
    private void configurarVentana() {
        setTitle("Sistema de Gestión de Productos");
        setSize(800, 600);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLocationRelativeTo(null);
    }
    
    private void crearInterfaz() {
        setLayout(new BorderLayout());
        
        // Toolbar
        JToolBar toolbar = new JToolBar();
        JButton btnAgregar = new JButton("Agregar");
        JButton btnActualizar = new JButton("Actualizar");
        JButton btnEliminar = new JButton("Eliminar");
        JButton btnReporte = new JButton("Generar Reporte");
        
        toolbar.add(btnAgregar);
        toolbar.add(btnActualizar);
        toolbar.add(btnEliminar);
        toolbar.addSeparator();
        toolbar.add(btnReporte);
        
        // Tabla
        String[] columnas = {"ID", "Nombre", "Categoría", "Precio", "Stock"};
        modeloTabla = new DefaultTableModel(columnas, 0) {
            @Override
            public boolean isCellEditable(int row, int column) {
                return false; // Solo lectura
            }
        };
        
        tablaProductos = new JTable(modeloTabla);
        tablaProductos.setSelectionMode(ListSelectionModel.SINGLE_SELECTION);
        
        JScrollPane scrollPane = new JScrollPane(tablaProductos);
        
        // Eventos
        btnAgregar.addActionListener(e -> mostrarDialogoAgregar());
        btnReporte.addActionListener(e -> generarReporteAsync());
        
        add(toolbar, BorderLayout.NORTH);
        add(scrollPane, BorderLayout.CENTER);
    }
    
    private void cargarDatos() {
        // Cargar datos en hilo separado
        executor.submit(() -> {
            List<Producto> productos = productoDAO.obtenerTodosLosProductos();
            
            SwingUtilities.invokeLater(() -> {
                modeloTabla.setRowCount(0);
                for (Producto p : productos) {
                    modeloTabla.addRow(new Object[]{
                        p.getId(), p.getNombre(), p.getCategoria(),
                        String.format("$%.2f", p.getPrecio()), p.getStock()
                    });
                }
            });
        });
    }
    
    private void mostrarDialogoAgregar() {
        JDialog dialogo = new JDialog(this, "Agregar Producto", true);
        dialogo.setSize(300, 200);
        dialogo.setLocationRelativeTo(this);
        
        JPanel panel = new JPanel(new GridBagLayout());
        GridBagConstraints gbc = new GridBagConstraints();
        
        JTextField txtNombre = new JTextField(15);
        JTextField txtCategoria = new JTextField(15);
        JTextField txtPrecio = new JTextField(15);
        JTextField txtStock = new JTextField(15);
        
        gbc.insets = new Insets(5, 5, 5, 5);
        
        gbc.gridx = 0; gbc.gridy = 0;
        panel.add(new JLabel("Nombre:"), gbc);
        gbc.gridx = 1;
        panel.add(txtNombre, gbc);
        
        gbc.gridx = 0; gbc.gridy = 1;
        panel.add(new JLabel("Categoría:"), gbc);
        gbc.gridx = 1;
        panel.add(txtCategoria, gbc);
        
        gbc.gridx = 0; gbc.gridy = 2;
        panel.add(new JLabel("Precio:"), gbc);
        gbc.gridx = 1;
        panel.add(txtPrecio, gbc);
        
        gbc.gridx = 0; gbc.gridy = 3;
        panel.add(new JLabel("Stock:"), gbc);
        gbc.gridx = 1;
        panel.add(txtStock, gbc);
        
        JButton btnGuardar = new JButton("Guardar");
        gbc.gridx = 0; gbc.gridy = 4;
        gbc.gridwidth = 2;
        panel.add(btnGuardar, gbc);
        
        btnGuardar.addActionListener(e -> {
            try {
                Producto producto = new Producto(
                    0, // ID se genera automáticamente
                    txtNombre.getText(),
                    txtCategoria.getText(),
                    Double.parseDouble(txtPrecio.getText()),
                    Integer.parseInt(txtStock.getText())
                );
                
                if (productoDAO.insertarProducto(producto)) {
                    JOptionPane.showMessageDialog(dialogo, "Producto agregado exitosamente");
                    cargarDatos();
                    dialogo.dispose();
                } else {
                    JOptionPane.showMessageDialog(dialogo, "Error al agregar producto");
                }
            } catch (NumberFormatException ex) {
                JOptionPane.showMessageDialog(dialogo, "Por favor, ingrese valores válidos");
            }
        });
        
        dialogo.add(panel);
        dialogo.setVisible(true);
    }
    
    private void generarReporteAsync() {
        JProgressBar progressBar = new JProgressBar();
        progressBar.setIndeterminate(true);
        progressBar.setString("Generando reporte...");
        progressBar.setStringPainted(true);
        
        JDialog dialogoProgreso = new JDialog(this, "Procesando", true);
        dialogoProgreso.add(progressBar);
        dialogoProgreso.setSize(300, 100);
        dialogoProgreso.setLocationRelativeTo(this);
        
        executor.submit(() -> {
            try {
                // Simular trabajo pesado
                Thread.sleep(3000);
                
                List<Producto> productos = productoDAO.obtenerTodosLosProductos();
                StringBuilder reporte = new StringBuilder();
                reporte.append("=== REPORTE DE PRODUCTOS ===\n\n");
                
                for (Producto p : productos) {
                    reporte.append(p.toString()).append("\n");
                }
                
                SwingUtilities.invokeLater(() -> {
                    dialogoProgreso.dispose();
                    
                    JTextArea areaTexto = new JTextArea(reporte.toString());
                    areaTexto.setEditable(false);
                    
                    JScrollPane scroll = new JScrollPane(areaTexto);
                    scroll.setPreferredSize(new Dimension(500, 400));
                    
                    JOptionPane.showMessageDialog(this, scroll, "Reporte de Productos", 
                                                JOptionPane.INFORMATION_MESSAGE);
                });
                
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        
        dialogoProgreso.setVisible(true);
    }
    
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            try {
                UIManager.setLookAndFeel(UIManager.getSystemLookAndFeel());
            } catch (Exception e) {
                e.printStackTrace();
            }
            new SistemaGestionApp().setVisible(true);
        });
    }
}
```

---

**Anterior**: [05. Librerías y APIs Avanzadas](./05-integracion.md) | **Siguiente**: [07. Proyecto Final y Evaluación](./07-evaluacion.md)
