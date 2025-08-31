# 07. Proyecto Final y Evaluación

## 🎯 Proyecto Final: Sistema de Gestión Empresarial

### Descripción del proyecto
Desarrollar un **Sistema de Gestión Empresarial** completo que integre todos los conceptos aprendidos en el curso:

**Funcionalidades principales:**
- 👥 Gestión de empleados y departamentos
- 📦 Control de inventario y productos  
- 💰 Facturación y ventas
- 📊 Reportes y estadísticas
- 🔐 Sistema de autenticación y permisos
- 🗄️ Persistencia de datos con base de datos
- 🖥️ Interfaz gráfica intuitiva

### Arquitectura del sistema
```
src/
├── main/
│   ├── java/
│   │   ├── com/empresa/gestion/
│   │   │   ├── Main.java
│   │   │   ├── modelos/
│   │   │   │   ├── Usuario.java
│   │   │   │   ├── Empleado.java
│   │   │   │   ├── Departamento.java
│   │   │   │   ├── Producto.java
│   │   │   │   └── Venta.java
│   │   │   ├── dao/
│   │   │   │   ├── ConexionBD.java
│   │   │   │   ├── UsuarioDAO.java
│   │   │   │   ├── EmpleadoDAO.java
│   │   │   │   └── ProductoDAO.java
│   │   │   ├── servicios/
│   │   │   │   ├── AutenticacionService.java
│   │   │   │   ├── EmpleadoService.java
│   │   │   │   └── InventarioService.java
│   │   │   ├── gui/
│   │   │   │   ├── LoginFrame.java
│   │   │   │   ├── MainFrame.java
│   │   │   │   ├── EmpleadosPanel.java
│   │   │   │   └── InventarioPanel.java
│   │   │   └── utils/
│   │   │       ├── Validaciones.java
│   │   │       └── ReporteGenerator.java
│   │   └── resources/
│   │       ├── database/
│   │       │   └── schema.sql
│   │       └── images/
└── test/
    └── java/
        └── com/empresa/gestion/
            ├── dao/
            ├── servicios/
            └── utils/
```

### Implementación paso a paso

#### 1. Modelos de datos
```java
// Usuario.java
public class Usuario {
    private int id;
    private String nombreUsuario;
    private String password;
    private String email;
    private TipoUsuario tipo;
    private boolean activo;
    private LocalDateTime fechaCreacion;
    
    // Constructor, getters, setters, equals, hashCode, toString
    
    public enum TipoUsuario {
        ADMINISTRADOR, SUPERVISOR, EMPLEADO
    }
}

// Empleado.java
public class Empleado {
    private int id;
    private String nombre;
    private String apellido;
    private String cedula;
    private String email;
    private String telefono;
    private Departamento departamento;
    private double salario;
    private LocalDate fechaIngreso;
    private boolean activo;
    
    // Constructor completo
    public Empleado(int id, String nombre, String apellido, String cedula, 
                   String email, String telefono, Departamento departamento, 
                   double salario, LocalDate fechaIngreso) {
        this.id = id;
        this.nombre = nombre;
        this.apellido = apellido;
        this.cedula = cedula;
        this.email = email;
        this.telefono = telefono;
        this.departamento = departamento;
        this.salario = salario;
        this.fechaIngreso = fechaIngreso;
        this.activo = true;
    }
    
    public String getNombreCompleto() {
        return nombre + " " + apellido;
    }
    
    public int getAntiguedad() {
        return Period.between(fechaIngreso, LocalDate.now()).getYears();
    }
    
    // Getters, setters, equals, hashCode, toString
}

// Departamento.java
public class Departamento {
    private int id;
    private String nombre;
    private String descripcion;
    private Empleado supervisor;
    private double presupuesto;
    private List<Empleado> empleados;
    
    public Departamento() {
        this.empleados = new ArrayList<>();
    }
    
    public void agregarEmpleado(Empleado empleado) {
        if (!empleados.contains(empleado)) {
            empleados.add(empleado);
            empleado.setDepartamento(this);
        }
    }
    
    public double getSalariosTotales() {
        return empleados.stream()
                       .mapToDouble(Empleado::getSalario)
                       .sum();
    }
    
    // Getters, setters, toString
}
```

#### 2. Capa de acceso a datos (DAO)
```java
// ConexionBD.java
public class ConexionBD {
    private static final String URL = "jdbc:sqlite:empresa.db";
    private static final String DRIVER = "org.sqlite.JDBC";
    
    static {
        try {
            Class.forName(DRIVER);
        } catch (ClassNotFoundException e) {
            throw new RuntimeException("Error al cargar driver SQLite", e);
        }
    }
    
    public static Connection obtenerConexion() throws SQLException {
        return DriverManager.getConnection(URL);
    }
    
    public static void inicializarBD() {
        try (Connection conn = obtenerConexion();
             Statement stmt = conn.createStatement()) {
            
            // Crear tablas
            String crearDepartamentos = """
                CREATE TABLE IF NOT EXISTS departamentos (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    nombre TEXT NOT NULL UNIQUE,
                    descripcion TEXT,
                    supervisor_id INTEGER,
                    presupuesto REAL DEFAULT 0
                )
            """;
            
            String crearEmpleados = """
                CREATE TABLE IF NOT EXISTS empleados (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    nombre TEXT NOT NULL,
                    apellido TEXT NOT NULL,
                    cedula TEXT NOT NULL UNIQUE,
                    email TEXT NOT NULL,
                    telefono TEXT,
                    departamento_id INTEGER,
                    salario REAL NOT NULL,
                    fecha_ingreso DATE NOT NULL,
                    activo BOOLEAN DEFAULT 1,
                    FOREIGN KEY (departamento_id) REFERENCES departamentos(id)
                )
            """;
            
            String crearUsuarios = """
                CREATE TABLE IF NOT EXISTS usuarios (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    nombre_usuario TEXT NOT NULL UNIQUE,
                    password TEXT NOT NULL,
                    email TEXT NOT NULL,
                    tipo TEXT NOT NULL DEFAULT 'EMPLEADO',
                    activo BOOLEAN DEFAULT 1,
                    fecha_creacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP
                )
            """;
            
            stmt.execute(crearDepartamentos);
            stmt.execute(crearEmpleados);
            stmt.execute(crearUsuarios);
            
            // Insertar datos iniciales
            insertarDatosIniciales(conn);
            
        } catch (SQLException e) {
            throw new RuntimeException("Error al inicializar base de datos", e);
        }
    }
    
    private static void insertarDatosIniciales(Connection conn) throws SQLException {
        // Insertar usuario administrador por defecto
        String insertarAdmin = """
            INSERT OR IGNORE INTO usuarios (nombre_usuario, password, email, tipo) 
            VALUES ('admin', 'admin123', 'admin@empresa.com', 'ADMINISTRADOR')
        """;
        
        try (PreparedStatement pstmt = conn.prepareStatement(insertarAdmin)) {
            pstmt.executeUpdate();
        }
    }
}

// EmpleadoDAO.java
public class EmpleadoDAO {
    
    public boolean insertar(Empleado empleado) {
        String sql = """
            INSERT INTO empleados (nombre, apellido, cedula, email, telefono, 
                                 departamento_id, salario, fecha_ingreso) 
            VALUES (?, ?, ?, ?, ?, ?, ?, ?)
        """;
        
        try (Connection conn = ConexionBD.obtenerConexion();
             PreparedStatement pstmt = conn.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS)) {
            
            pstmt.setString(1, empleado.getNombre());
            pstmt.setString(2, empleado.getApellido());
            pstmt.setString(3, empleado.getCedula());
            pstmt.setString(4, empleado.getEmail());
            pstmt.setString(5, empleado.getTelefono());
            pstmt.setInt(6, empleado.getDepartamento().getId());
            pstmt.setDouble(7, empleado.getSalario());
            pstmt.setDate(8, Date.valueOf(empleado.getFechaIngreso()));
            
            int filasAfectadas = pstmt.executeUpdate();
            
            if (filasAfectadas > 0) {
                try (ResultSet rs = pstmt.getGeneratedKeys()) {
                    if (rs.next()) {
                        empleado.setId(rs.getInt(1));
                    }
                }
                return true;
            }
            
        } catch (SQLException e) {
            System.err.println("Error al insertar empleado: " + e.getMessage());
        }
        
        return false;
    }
    
    public List<Empleado> obtenerTodos() {
        List<Empleado> empleados = new ArrayList<>();
        String sql = """
            SELECT e.*, d.nombre as departamento_nombre 
            FROM empleados e 
            LEFT JOIN departamentos d ON e.departamento_id = d.id 
            WHERE e.activo = 1
            ORDER BY e.apellido, e.nombre
        """;
        
        try (Connection conn = ConexionBD.obtenerConexion();
             Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery(sql)) {
            
            while (rs.next()) {
                Departamento dept = new Departamento();
                dept.setId(rs.getInt("departamento_id"));
                dept.setNombre(rs.getString("departamento_nombre"));
                
                Empleado empleado = new Empleado(
                    rs.getInt("id"),
                    rs.getString("nombre"),
                    rs.getString("apellido"),
                    rs.getString("cedula"),
                    rs.getString("email"),
                    rs.getString("telefono"),
                    dept,
                    rs.getDouble("salario"),
                    rs.getDate("fecha_ingreso").toLocalDate()
                );
                
                empleados.add(empleado);
            }
            
        } catch (SQLException e) {
            System.err.println("Error al obtener empleados: " + e.getMessage());
        }
        
        return empleados;
    }
    
    public boolean actualizar(Empleado empleado) {
        String sql = """
            UPDATE empleados 
            SET nombre = ?, apellido = ?, email = ?, telefono = ?, 
                departamento_id = ?, salario = ? 
            WHERE id = ?
        """;
        
        try (Connection conn = ConexionBD.obtenerConexion();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            pstmt.setString(1, empleado.getNombre());
            pstmt.setString(2, empleado.getApellido());
            pstmt.setString(3, empleado.getEmail());
            pstmt.setString(4, empleado.getTelefono());
            pstmt.setInt(5, empleado.getDepartamento().getId());
            pstmt.setDouble(6, empleado.getSalario());
            pstmt.setInt(7, empleado.getId());
            
            return pstmt.executeUpdate() > 0;
            
        } catch (SQLException e) {
            System.err.println("Error al actualizar empleado: " + e.getMessage());
            return false;
        }
    }
    
    public boolean eliminar(int id) {
        String sql = "UPDATE empleados SET activo = 0 WHERE id = ?";
        
        try (Connection conn = ConexionBD.obtenerConexion();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            pstmt.setInt(1, id);
            return pstmt.executeUpdate() > 0;
            
        } catch (SQLException e) {
            System.err.println("Error al eliminar empleado: " + e.getMessage());
            return false;
        }
    }
}
```

#### 3. Capa de servicios
```java
// AutenticacionService.java
public class AutenticacionService {
    private UsuarioDAO usuarioDAO;
    private Usuario usuarioActual;
    
    public AutenticacionService() {
        this.usuarioDAO = new UsuarioDAO();
    }
    
    public boolean autenticar(String nombreUsuario, String password) {
        try {
            Usuario usuario = usuarioDAO.buscarPorNombreUsuario(nombreUsuario);
            
            if (usuario != null && usuario.isActivo()) {
                // En un entorno real, la password estaría hasheada
                if (usuario.getPassword().equals(password)) {
                    this.usuarioActual = usuario;
                    return true;
                }
            }
            
        } catch (Exception e) {
            System.err.println("Error en autenticación: " + e.getMessage());
        }
        
        return false;
    }
    
    public boolean tienePermiso(String accion) {
        if (usuarioActual == null) return false;
        
        return switch (usuarioActual.getTipo()) {
            case ADMINISTRADOR -> true; // Acceso total
            case SUPERVISOR -> accion.equals("LEER") || accion.equals("MODIFICAR");
            case EMPLEADO -> accion.equals("LEER");
        };
    }
    
    public Usuario getUsuarioActual() {
        return usuarioActual;
    }
    
    public void cerrarSesion() {
        this.usuarioActual = null;
    }
}

// EmpleadoService.java
public class EmpleadoService {
    private EmpleadoDAO empleadoDAO;
    private DepartamentoDAO departamentoDAO;
    
    public EmpleadoService() {
        this.empleadoDAO = new EmpleadoDAO();
        this.departamentoDAO = new DepartamentoDAO();
    }
    
    public ResultadoOperacion<Empleado> crearEmpleado(Empleado empleado) {
        // Validaciones de negocio
        if (!Validaciones.esEmailValido(empleado.getEmail())) {
            return new ResultadoOperacion<>(false, "Email inválido");
        }
        
        if (!Validaciones.esCedulaValida(empleado.getCedula())) {
            return new ResultadoOperacion<>(false, "Cédula inválida");
        }
        
        if (empleado.getSalario() <= 0) {
            return new ResultadoOperacion<>(false, "El salario debe ser mayor a 0");
        }
        
        // Verificar que el departamento existe
        Departamento dept = departamentoDAO.buscarPorId(empleado.getDepartamento().getId());
        if (dept == null) {
            return new ResultadoOperacion<>(false, "Departamento no encontrado");
        }
        
        boolean exito = empleadoDAO.insertar(empleado);
        return new ResultadoOperacion<>(exito, 
            exito ? "Empleado creado exitosamente" : "Error al crear empleado", 
            empleado);
    }
    
    public List<Empleado> buscarEmpleados(String criterio) {
        return empleadoDAO.buscarPorCriterio(criterio);
    }
    
    public Map<String, Object> obtenerEstadisticas() {
        Map<String, Object> stats = new HashMap<>();
        
        List<Empleado> empleados = empleadoDAO.obtenerTodos();
        
        stats.put("totalEmpleados", empleados.size());
        stats.put("salarioPromedio", empleados.stream()
                                             .mapToDouble(Empleado::getSalario)
                                             .average().orElse(0.0));
        
        stats.put("porDepartamento", empleados.stream()
                                             .collect(Collectors.groupingBy(
                                                 e -> e.getDepartamento().getNombre(),
                                                 Collectors.counting())));
        
        return stats;
    }
}
```

#### 4. Interfaz gráfica principal
```java
// MainFrame.java
public class MainFrame extends JFrame {
    private AutenticacionService authService;
    private JTabbedPane tabbedPane;
    private EmpleadosPanel empleadosPanel;
    private InventarioPanel inventarioPanel;
    
    public MainFrame(AutenticacionService authService) {
        this.authService = authService;
        configurarVentana();
        crearMenuBar();
        crearInterfaz();
    }
    
    private void configurarVentana() {
        setTitle("Sistema de Gestión Empresarial - " + 
                authService.getUsuarioActual().getNombreUsuario());
        setSize(1200, 800);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLocationRelativeTo(null);
        setExtendedState(JFrame.MAXIMIZED_BOTH);
    }
    
    private void crearMenuBar() {
        JMenuBar menuBar = new JMenuBar();
        
        // Menú Archivo
        JMenu menuArchivo = new JMenu("Archivo");
        JMenuItem itemSalir = new JMenuItem("Salir");
        itemSalir.addActionListener(e -> salir());
        menuArchivo.add(itemSalir);
        
        // Menú Herramientas
        JMenu menuHerramientas = new JMenu("Herramientas");
        JMenuItem itemReportes = new JMenuItem("Generar Reportes");
        JMenuItem itemBackup = new JMenuItem("Backup Base de Datos");
        
        if (authService.tienePermiso("ADMINISTRAR")) {
            menuHerramientas.add(itemReportes);
            menuHerramientas.add(itemBackup);
        }
        
        // Menú Ayuda
        JMenu menuAyuda = new JMenu("Ayuda");
        JMenuItem itemAcerca = new JMenuItem("Acerca de");
        itemAcerca.addActionListener(e -> mostrarAcercaDe());
        menuAyuda.add(itemAcerca);
        
        menuBar.add(menuArchivo);
        menuBar.add(menuHerramientas);
        menuBar.add(menuAyuda);
        
        setJMenuBar(menuBar);
    }
    
    private void crearInterfaz() {
        tabbedPane = new JTabbedPane();
        
        // Panel de empleados
        empleadosPanel = new EmpleadosPanel(authService);
        tabbedPane.addTab("👥 Empleados", empleadosPanel);
        
        // Panel de inventario
        inventarioPanel = new InventarioPanel(authService);
        tabbedPane.addTab("📦 Inventario", inventarioPanel);
        
        // Panel de reportes (solo para admin/supervisor)
        if (authService.tienePermiso("MODIFICAR")) {
            ReportesPanel reportesPanel = new ReportesPanel(authService);
            tabbedPane.addTab("📊 Reportes", reportesPanel);
        }
        
        add(tabbedPane, BorderLayout.CENTER);
        
        // Barra de estado
        JPanel statusBar = new JPanel(new FlowLayout(FlowLayout.LEFT));
        JLabel statusLabel = new JLabel("Usuario: " + 
            authService.getUsuarioActual().getNombreUsuario() + 
            " | Fecha: " + LocalDate.now().format(DateTimeFormatter.ofPattern("dd/MM/yyyy")));
        statusBar.add(statusLabel);
        
        add(statusBar, BorderLayout.SOUTH);
    }
    
    private void salir() {
        int opcion = JOptionPane.showConfirmDialog(
            this,
            "¿Está seguro que desea salir del sistema?",
            "Confirmar salida",
            JOptionPane.YES_NO_OPTION
        );
        
        if (opcion == JOptionPane.YES_OPTION) {
            authService.cerrarSesion();
            System.exit(0);
        }
    }
    
    private void mostrarAcercaDe() {
        String mensaje = """
            Sistema de Gestión Empresarial
            Versión 1.0
            
            Desarrollado como proyecto final del curso de Java
            
            Funcionalidades:
            • Gestión de empleados y departamentos
            • Control de inventario
            • Sistema de autenticación
            • Generación de reportes
            
            © 2024 - Curso Java Avanzado
        """;
        
        JOptionPane.showMessageDialog(this, mensaje, "Acerca del Sistema", 
                                    JOptionPane.INFORMATION_MESSAGE);
    }
}
```

#### 5. Tests del sistema
```java
// EmpleadoServiceTest.java
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class EmpleadoServiceTest {
    private EmpleadoService empleadoService;
    private Departamento departamentoPrueba;
    
    @BeforeAll
    void configurarPruebas() {
        // Configurar BD de pruebas
        System.setProperty("test.db", "true");
        ConexionBD.inicializarBD();
        
        empleadoService = new EmpleadoService();
        
        // Crear departamento de prueba
        departamentoPrueba = new Departamento();
        departamentoPrueba.setNombre("TI");
        departamentoPrueba.setDescripcion("Tecnología de la Información");
    }
    
    @Test
    @DisplayName("Crear empleado con datos válidos")
    void testCrearEmpleadoValido() {
        Empleado empleado = new Empleado(
            0, "Juan", "Pérez", "1234567890",
            "juan.perez@empresa.com", "555-1234",
            departamentoPrueba, 50000.0, LocalDate.now()
        );
        
        ResultadoOperacion<Empleado> resultado = empleadoService.crearEmpleado(empleado);
        
        assertTrue(resultado.isExito());
        assertNotNull(resultado.getDatos());
        assertTrue(resultado.getDatos().getId() > 0);
    }
    
    @Test
    @DisplayName("Crear empleado con email inválido debe fallar")
    void testCrearEmpleadoEmailInvalido() {
        Empleado empleado = new Empleado(
            0, "Ana", "García", "9876543210",
            "email-invalido", "555-5678",
            departamentoPrueba, 45000.0, LocalDate.now()
        );
        
        ResultadoOperacion<Empleado> resultado = empleadoService.crearEmpleado(empleado);
        
        assertFalse(resultado.isExito());
        assertEquals("Email inválido", resultado.getMensaje());
    }
    
    @ParameterizedTest
    @ValueSource(doubles = {-1000.0, 0.0, -50.5})
    @DisplayName("Salarios inválidos deben ser rechazados")
    void testSalariosInvalidos(double salario) {
        Empleado empleado = new Empleado(
            0, "Test", "Usuario", "1111111111",
            "test@empresa.com", "555-0000",
            departamentoPrueba, salario, LocalDate.now()
        );
        
        ResultadoOperacion<Empleado> resultado = empleadoService.crearEmpleado(empleado);
        
        assertFalse(resultado.isExito());
        assertTrue(resultado.getMensaje().contains("salario"));
    }
    
    @Test
    @DisplayName("Buscar empleados por criterio")
    void testBuscarEmpleados() {
        // Crear empleados de prueba
        crearEmpleadosPrueba();
        
        List<Empleado> resultados = empleadoService.buscarEmpleados("Juan");
        
        assertFalse(resultados.isEmpty());
        assertTrue(resultados.stream()
                  .anyMatch(e -> e.getNombre().contains("Juan")));
    }
    
    @Test
    @DisplayName("Obtener estadísticas del sistema")
    void testObtenerEstadisticas() {
        Map<String, Object> stats = empleadoService.obtenerEstadisticas();
        
        assertNotNull(stats);
        assertTrue(stats.containsKey("totalEmpleados"));
        assertTrue(stats.containsKey("salarioPromedio"));
        assertTrue(stats.containsKey("porDepartamento"));
    }
    
    private void crearEmpleadosPrueba() {
        List<Empleado> empleados = Arrays.asList(
            new Empleado(0, "Juan", "Pérez", "1111", "juan@test.com", "555-1111", 
                        departamentoPrueba, 50000, LocalDate.now()),
            new Empleado(0, "María", "González", "2222", "maria@test.com", "555-2222",
                        departamentoPrueba, 55000, LocalDate.now()),
            new Empleado(0, "Carlos", "Rodríguez", "3333", "carlos@test.com", "555-3333",
                        departamentoPrueba, 48000, LocalDate.now())
        );
        
        empleados.forEach(empleadoService::crearEmpleado);
    }
}
```

### Criterios de evaluación

#### Funcionalidad (40%)
- ✅ **CRUD completo** de entidades principales
- ✅ **Sistema de autenticación** y permisos
- ✅ **Interfaz gráfica** funcional e intuitiva
- ✅ **Persistencia** correcta en base de datos
- ✅ **Manejo de errores** y validaciones

#### Código (30%)
- ✅ **Arquitectura** bien estructurada (MVC/capas)
- ✅ **Principios SOLID** aplicados
- ✅ **Nomenclatura** consistente y clara
- ✅ **Comentarios** y documentación
- ✅ **Reutilización** de código

#### Testing (20%)
- ✅ **Cobertura de tests** mínima 70%
- ✅ **Tests unitarios** para lógica de negocio
- ✅ **Tests de integración** para DAO
- ✅ **Casos edge** y manejo de errores
- ✅ **Tests parametrizados** donde aplique

#### Presentación (10%)
- ✅ **Demostración** funcionando del sistema
- ✅ **Explicación** de arquitectura y decisiones
- ✅ **Documentación** de usuario
- ✅ **Manual** de instalación y configuración

### Entrega

#### Documentos requeridos:
1. **Código fuente** completo del proyecto
2. **Manual de usuario** con screenshots
3. **Documentación técnica** (arquitectura, BD, APIs)
4. **Instrucciones de instalación** y configuración
5. **Reporte de tests** con cobertura
6. **Video demo** (5-10 minutos) del sistema funcionando

#### Formato de entrega:
```
proyecto-final-java.zip
├── src/                    # Código fuente
├── docs/                   # Documentación
│   ├── manual-usuario.pdf
│   ├── documentacion-tecnica.pdf
│   └── arquitectura.png
├── tests/                  # Tests y reportes
├── database/              # Scripts SQL
├── README.md              # Instrucciones principales
├── pom.xml               # Dependencias Maven
└── demo-video.mp4        # Video demostración
```

#### Cronograma:
- **Semana 1-2**: Diseño y arquitectura del sistema
- **Semana 3-4**: Implementación del backend (modelos, DAO, servicios)
- **Semana 5-6**: Desarrollo de la interfaz gráfica
- **Semana 7**: Testing y corrección de errores
- **Semana 8**: Documentación y preparación de entrega

## 🏆 Casos de éxito y mejores prácticas

### Patrones implementados exitosamente:
1. **DAO Pattern** para acceso a datos
2. **Service Layer** para lógica de negocio
3. **Observer Pattern** para actualizaciones de UI
4. **Factory Pattern** para creación de objetos
5. **Singleton Pattern** para conexión BD

### Tecnologías integradas:
- **Java 17+** con características modernas
- **SQLite** para persistencia
- **Swing** para interfaz gráfica
- **JUnit 5** para testing
- **Maven** para gestión de dependencias

¡Felicitaciones por completar el curso Java de básico a avanzado! 🎉

---

**Anterior**: [06. Desarrollo de Aplicaciones](./06-documentacion.md) | **Inicio**: [README](../README.md)
