# 03. Setup y Entorno de Desarrollo

## 🛠️ IDEs recomendados para Java

### IntelliJ IDEA Community Edition (Recomendado)

#### Instalación
```bash
# Descargar desde: https://www.jetbrains.com/idea/download/
# O usando package managers:

# Windows (Chocolatey)
choco install intellijidea-community

# macOS (Homebrew)
brew install --cask intellij-idea-ce

# Linux (Snap)
sudo snap install intellij-idea-community --classic
```

#### Configuración inicial
1. **Configurar JDK**:
   - File → Project Structure → Project Settings → Project
   - Seleccionar Project SDK: Java 17 o superior

2. **Configurar Maven**:
   - File → Settings → Build Tools → Maven
   - Verificar Maven home directory

3. **Plugins esenciales**:
   - SonarLint (análisis de código)
   - GitToolBox (mejoras para Git)
   - Rainbow Brackets (colores para brackets)

### Eclipse IDE

#### Instalación y configuración
```bash
# Descargar Eclipse IDE for Java Developers
# https://www.eclipse.org/downloads/

# Configurar workspace encoding
Window → Preferences → General → Workspace → Text file encoding: UTF-8

# Configurar JRE
Window → Preferences → Java → Installed JREs → Add...
```

### Visual Studio Code

#### Extensiones para Java
```bash
# Extension Pack for Java (incluye todo lo necesario)
code --install-extension vscjava.vscode-java-pack

# Extensiones individuales incluidas:
# - Language Support for Java (redhat.java)
# - Debugger for Java (vscjava.vscode-java-debug)
# - Test Runner for Java (vscjava.vscode-java-test)
# - Maven for Java (vscjava.vscode-maven)
# - Project Manager for Java (vscjava.vscode-java-dependency)
# - Visual Studio IntelliCode (visualstudioexptteam.vscodeintellicode)
```

## 📦 Gestión de dependencias con Maven

### Estructura de proyecto Maven
```
mi-proyecto/
├── pom.xml                 # Archivo de configuración Maven
├── src/
│   ├── main/
│   │   ├── java/          # Código fuente
│   │   │   └── com/
│   │   │       └── empresa/
│   │   │           └── App.java
│   │   └── resources/     # Recursos (archivos de configuración)
│   └── test/
│       ├── java/          # Tests unitarios
│       └── resources/     # Recursos para tests
├── target/                # Archivos compilados (generado)
└── .gitignore
```

### Archivo pom.xml básico
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
    <modelVersion>4.0.0</modelVersion>
    
    <!-- Información del proyecto -->
    <groupId>com.empresa</groupId>
    <artifactId>mi-proyecto</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>
    
    <name>Mi Proyecto Java</name>
    <description>Descripción del proyecto</description>
    
    <!-- Propiedades -->
    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <junit.version>5.9.2</junit.version>
    </properties>
    
    <!-- Dependencias -->
    <dependencies>
        <!-- JUnit 5 para testing -->
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>
        
        <!-- Apache Commons Lang -->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
            <version>3.12.0</version>
        </dependency>
        
        <!-- Jackson para JSON -->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.15.2</version>
        </dependency>
    </dependencies>
    
    <!-- Plugins de build -->
    <build>
        <plugins>
            <!-- Plugin del compilador -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.11.0</version>
                <configuration>
                    <source>17</source>
                    <target>17</target>
                </configuration>
            </plugin>
            
            <!-- Plugin de Surefire para tests -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.0.0</version>
            </plugin>
            
            <!-- Plugin para crear JAR ejecutable -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>3.4.1</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <transformers>
                                <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <mainClass>com.empresa.App</mainClass>
                                </transformer>
                            </transformers>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

### Comandos Maven esenciales
```bash
# Crear nuevo proyecto Maven
mvn archetype:generate -DgroupId=com.empresa -DartifactId=mi-proyecto -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false

# Compilar el proyecto
mvn compile

# Ejecutar tests
mvn test

# Crear JAR
mvn package

# Limpiar archivos generados
mvn clean

# Comando completo (limpiar, compilar, testear, empaquetar)
mvn clean package

# Ejecutar la aplicación
mvn exec:java -Dexec.mainClass="com.empresa.App"

# Instalar en repositorio local
mvn install

# Mostrar dependencias
mvn dependency:tree
```

## 📦 Gestión de dependencias con Gradle (Alternativa)

### Archivo build.gradle
```gradle
plugins {
    id 'java'
    id 'application'
}

group = 'com.empresa'
version = '1.0.0'
sourceCompatibility = '17'

repositories {
    mavenCentral()
}

dependencies {
    // JUnit 5
    testImplementation 'org.junit.jupiter:junit-jupiter:5.9.2'
    
    // Apache Commons
    implementation 'org.apache.commons:commons-lang3:3.12.0'
    
    // Jackson JSON
    implementation 'com.fasterxml.jackson.core:jackson-databind:2.15.2'
}

application {
    mainClass = 'com.empresa.App'
}

test {
    useJUnitPlatform()
}

jar {
    manifest {
        attributes 'Main-Class': 'com.empresa.App'
    }
}
```

### Comandos Gradle esenciales
```bash
# Compilar
./gradlew build

# Ejecutar tests
./gradlew test

# Ejecutar aplicación
./gradlew run

# Crear JAR
./gradlew jar

# Limpiar
./gradlew clean

# Mostrar dependencias
./gradlew dependencies
```

## 🐛 Debugging y profiling

### Debugging en IntelliJ IDEA
```java
public class DebuggingExample {
    public static void main(String[] args) {
        int[] numeros = {1, 2, 3, 4, 5};
        int suma = 0;
        
        for (int i = 0; i < numeros.length; i++) {
            suma += numeros[i]; // Colocar breakpoint aquí
            System.out.println("Suma parcial: " + suma);
        }
        
        System.out.println("Suma total: " + suma);
    }
}
```

#### Técnicas de debugging:
1. **Breakpoints**: Click en margen izquierdo del editor
2. **Step Over (F8)**: Ejecutar línea actual
3. **Step Into (F7)**: Entrar en métodos
4. **Step Out (Shift+F8)**: Salir del método actual
5. **Evaluate Expression (Alt+F8)**: Evaluar expresiones en tiempo real

### Profiling básico
```java
public class ProfilingExample {
    public static void main(String[] args) {
        // Medir tiempo de ejecución
        long startTime = System.currentTimeMillis();
        
        // Operación a medir
        for (int i = 0; i < 1000000; i++) {
            Math.sqrt(i);
        }
        
        long endTime = System.currentTimeMillis();
        System.out.println("Tiempo de ejecución: " + (endTime - startTime) + " ms");
        
        // Información de memoria
        Runtime runtime = Runtime.getRuntime();
        long usedMemory = runtime.totalMemory() - runtime.freeMemory();
        System.out.println("Memoria usada: " + usedMemory / 1024 / 1024 + " MB");
    }
}
```

## 📁 Estructura de proyecto recomendada

### Proyecto simple
```
mi-aplicacion/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/
│   │   │       └── empresa/
│   │   │           ├── App.java              # Clase principal
│   │   │           ├── model/                # Modelos de datos
│   │   │           │   ├── Usuario.java
│   │   │           │   └── Producto.java
│   │   │           ├── service/              # Lógica de negocio
│   │   │           │   ├── UsuarioService.java
│   │   │           │   └── ProductoService.java
│   │   │           ├── util/                 # Utilidades
│   │   │           │   ├── DateUtil.java
│   │   │           │   └── StringUtil.java
│   │   │           └── config/               # Configuraciones
│   │   │               └── DatabaseConfig.java
│   │   └── resources/
│   │       ├── application.properties        # Configuración
│   │       └── logback.xml                   # Configuración de logs
│   └── test/
│       ├── java/
│       │   └── com/
│       │       └── empresa/
│       │           ├── service/
│       │           │   ├── UsuarioServiceTest.java
│       │           │   └── ProductoServiceTest.java
│       │           └── util/
│       │               ├── DateUtilTest.java
│       │               └── StringUtilTest.java
│       └── resources/
│           └── test.properties
├── pom.xml                                   # Maven configuration
├── .gitignore
└── README.md
```

### .gitignore para proyectos Java
```gitignore
# Compiled class files
*.class

# Log files
*.log

# BlueJ files
*.ctxt

# Mobile Tools for Java (J2ME)
.mtj.tmp/

# Package Files
*.jar
*.war
*.nar
*.ear
*.zip
*.tar.gz
*.rar

# Virtual machine crash logs
hs_err_pid*

# Maven
target/
pom.xml.tag
pom.xml.releaseBackup
pom.xml.versionsBackup
pom.xml.next
release.properties
dependency-reduced-pom.xml
buildNumber.properties
.mvn/timing.properties
.mvn/wrapper/maven-wrapper.jar

# Gradle
.gradle
build/
!gradle/wrapper/gradle-wrapper.jar
!**/src/main/**/build/
!**/src/test/**/build/

# IntelliJ IDEA
.idea/
*.iws
*.iml
*.ipr
out/
!**/src/main/**/out/
!**/src/test/**/out/

# Eclipse
.apt_generated
.classpath
.factorypath
.project
.settings
.springBeans
.sts4-cache
bin/
!**/src/main/**/bin/
!**/src/test/**/bin/

# VS Code
.vscode/

# Mac
.DS_Store

# Windows
Thumbs.db
ehthumbs.db
Desktop.ini
```

## 🎯 Proyecto práctico: Configuración completa

### Crear proyecto desde cero
```bash
# 1. Crear proyecto con Maven
mvn archetype:generate \
  -DgroupId=com.empresa.gestiontareas \
  -DartifactId=gestor-tareas \
  -DarchetypeArtifactId=maven-archetype-quickstart \
  -DinteractiveMode=false

# 2. Entrar al directorio
cd gestor-tareas

# 3. Abrir en IDE
idea .  # IntelliJ IDEA
# o
code .  # VS Code
```

### Ejemplo de aplicación: Gestor de Tareas
```java
// src/main/java/com/empresa/gestiontareas/App.java
package com.empresa.gestiontareas;

import com.empresa.gestiontareas.model.Tarea;
import com.empresa.gestiontareas.service.TareaService;

public class App {
    public static void main(String[] args) {
        TareaService tareaService = new TareaService();
        
        // Crear algunas tareas
        tareaService.crearTarea("Estudiar Java", "Completar módulo 3");
        tareaService.crearTarea("Hacer ejercicio", "Correr 30 minutos");
        
        // Mostrar todas las tareas
        System.out.println("=== Lista de Tareas ===");
        tareaService.listarTareas().forEach(System.out::println);
    }
}
```

```java
// src/main/java/com/empresa/gestiontareas/model/Tarea.java
package com.empresa.gestiontareas.model;

import java.time.LocalDateTime;

public class Tarea {
    private static int contador = 1;
    
    private final int id;
    private String titulo;
    private String descripcion;
    private boolean completada;
    private final LocalDateTime fechaCreacion;
    
    public Tarea(String titulo, String descripcion) {
        this.id = contador++;
        this.titulo = titulo;
        this.descripcion = descripcion;
        this.completada = false;
        this.fechaCreacion = LocalDateTime.now();
    }
    
    // Getters y setters
    public int getId() { return id; }
    public String getTitulo() { return titulo; }
    public void setTitulo(String titulo) { this.titulo = titulo; }
    public String getDescripcion() { return descripcion; }
    public void setDescripcion(String descripcion) { this.descripcion = descripcion; }
    public boolean isCompletada() { return completada; }
    public void setCompletada(boolean completada) { this.completada = completada; }
    public LocalDateTime getFechaCreacion() { return fechaCreacion; }
    
    @Override
    public String toString() {
        return String.format("[%d] %s - %s (%s)", 
            id, titulo, descripcion, completada ? "✓" : "○");
    }
}
```

### Comandos para el proyecto
```bash
# Compilar
mvn compile

# Ejecutar
mvn exec:java -Dexec.mainClass="com.empresa.gestiontareas.App"

# Crear JAR ejecutable
mvn package

# Ejecutar JAR
java -jar target/gestor-tareas-1.0-SNAPSHOT.jar
```

## ✅ Checklist de configuración

- [ ] JDK 17+ instalado y configurado
- [ ] IDE configurado (IntelliJ/Eclipse/VS Code)
- [ ] Maven o Gradle funcionando
- [ ] Proyecto de prueba creado y ejecutándose
- [ ] Debugging funcional
- [ ] Git configurado para el proyecto
- [ ] .gitignore configurado apropiadamente
- [ ] Estructura de directorios organizada

---

**Anterior**: [02. Conceptos Fundamentales](./02-conceptos.md) | **Siguiente**: [04. Programación Orientada a Objetos](./04-practica.md)
