# 01. Introducción a Git y GitHub

## 📚 ¿Qué es Git?

Git es un **sistema de control de versiones distribuido** (DVCS) que revolucionó la forma en que los desarrolladores gestionan el código fuente. Creado por Linus Torvalds en 2005 para el desarrollo del kernel de Linux, Git se ha convertido en el estándar de facto para el control de versiones en la industria del software.

### Historia y contexto
- **2005**: Linus Torvalds crea Git debido a la necesidad de una mejor herramienta de control de versiones
- **Problema**: Los sistemas existentes (SVN, CVS) eran centralizados y limitantes
- **Solución**: Un sistema distribuido donde cada desarrollador tiene una copia completa del historial
- **Impacto**: Cambió fundamentalmente cómo los equipos colaboran en código

### ¿Por qué Git es diferente?
**Control de versiones tradicional (centralizado)**:
```
Servidor Central
      |
   Desarrolladores (copias de trabajo)
```

**Git (distribuido)**:
```
Repo Completo ← → Repo Completo ← → Repo Completo
  (Dev A)           (Servidor)         (Dev B)
```

### Ventajas fundamentales de Git
- **🔄 Trabajo offline**: Historial completo disponible localmente
- **👥 Colaboración sin conflictos**: Múltiples desarrolladores pueden trabajar simultáneamente
- **📈 Historial inmutable**: Registro completo y verificable de todos los cambios
- **⏪ Recuperación total**: Vuelta a cualquier punto en el tiempo
- **🚀 Rendimiento superior**: Operaciones locales extremadamente rápidas
- **🌐 Flexibilidad de flujos**: Múltiples estrategias de trabajo colaborativo
- **🔒 Integridad de datos**: Verificación criptográfica (SHA-1) de todo el contenido

## ⚙️ Instalación y Configuración

### Instalación de Git

#### Windows
```bash
# Opción 1: Winget (recomendado)
winget install --id Git.Git

# Opción 2: Chocolatey
choco install git

# Opción 3: Descarga directa
# Ir a https://git-scm.com/downloads y descargar el instalador
```

#### macOS
```bash
# Opción 1: Homebrew (recomendado)
brew install git

# Opción 2: MacPorts
sudo port install git

# Opción 3: Xcode Command Line Tools
xcode-select --install
```

#### Linux
```bash
# Ubuntu/Debian
sudo apt-get update
sudo apt-get install git

# CentOS/RHEL/Fedora
sudo yum install git        # CentOS 7
sudo dnf install git        # Fedora/CentOS 8+

# Arch Linux
sudo pacman -S git

# Verificar instalación
git --version
```

### Configuración esencial

#### Identidad del usuario (OBLIGATORIO)
```bash
# Configuración global (para todos los repositorios)
git config --global user.name "Tu Nombre Completo"
git config --global user.email "tu.email@ejemplo.com"

# Verificar configuración
git config --global user.name
git config --global user.email
```

#### Editor predeterminado
```bash
# Visual Studio Code
git config --global core.editor "code --wait"

# Vim
git config --global core.editor "vim"

# Nano
git config --global core.editor "nano"

# Notepad++ (Windows)
git config --global core.editor "'C:/Program Files/Notepad++/notepad++.exe' -multiInst -notabbar -nosession -noPlugin"
```

#### Configuraciones adicionales recomendadas
```bash
# Configurar salto de línea (importante para colaboración multiplataforma)
git config --global core.autocrlf true    # Windows
git config --global core.autocrlf input   # macOS/Linux

# Mejorar visualización de diffs
git config --global color.ui auto

# Configurar herramienta de merge
git config --global merge.tool vimdiff    # o tu herramienta preferida

# Ver toda la configuración actual
git config --list
git config --list --global
```

## 🏗️ Conceptos Fundamentales

### Anatomía de un repositorio Git

```
mi-proyecto/
├── .git/           ← Base de datos de Git (historial, configuración)
├── archivo1.txt    ← Archivos del proyecto (working directory)
├── archivo2.py
└── README.md
```

### Los tres estados de Git

Git maneja tres áreas principales donde pueden estar tus archivos:

```
Working Directory  →  Staging Area  →  Repository (.git)
(Directorio de      (Área de         (Historial 
 trabajo)           preparación)      permanente)
     |                    |                |
  Modificar           git add         git commit
  archivos            archivos        cambios
```

#### 1. Working Directory (Directorio de trabajo)
- Archivos actuales en tu carpeta de proyecto
- Incluye cambios sin confirmar
- Es donde editas y modificas archivos

#### 2. Staging Area (Área de preparación)
- Archivos marcados para el próximo commit
- Permite seleccionar qué cambios incluir
- Se gestiona con `git add`

#### 3. Repository (Repositorio Git)
- Base de datos con el historial completo
- Commits confirmados permanentemente
- Se actualiza con `git commit`

### Estados de archivos en Git

```
Untracked  →  Unmodified  →  Modified  →  Staged
    |           |              |           |
git add     (sin cambios)   git add      git commit
    ↓           ↓              ↓           ↓
  Staged    Unmodified     Staged    Unmodified
```

- **Untracked**: Archivo nuevo, Git no lo conoce
- **Unmodified**: Archivo sin cambios desde último commit
- **Modified**: Archivo modificado pero no añadido al staging
- **Staged**: Archivo preparado para commit

## 🚀 Primer Repositorio y Commit

### Crear un nuevo repositorio desde cero

#### Método 1: Repositorio local
```bash
# Crear y entrar en directorio del proyecto
mkdir mi-primer-proyecto
cd mi-primer-proyecto

# Inicializar repositorio Git
git init

# Verificar que se creó .git/
ls -la
# Deberías ver: .git/ (directorio oculto)

# Verificar estado
git status
```

#### Método 2: Clonar repositorio existente
```bash
# Clonar desde GitHub (ejemplo)
git clone https://github.com/usuario/proyecto.git

# Entrar en el directorio clonado
cd proyecto

# Ya está listo para trabajar
git status
```

### Flujo básico de trabajo (tu primer commit)

#### 1. Crear contenido
```bash
# Crear archivo README
echo "# Mi Primer Proyecto" > README.md

# Crear archivo de código (ejemplo)
echo 'print("Hola, Git!")' > main.py

# Ver archivos creados
ls -la
```

#### 2. Verificar estado
```bash
git status
# Output:
# On branch main
# No commits yet
# Untracked files:
#   README.md
#   main.py
```

#### 3. Añadir archivos al staging area
```bash
# Añadir archivo específico
git add README.md

# Añadir todos los archivos
git add .

# Añadir archivos por extensión
git add *.py

# Verificar qué está en staging
git status
```

#### 4. Hacer el commit
```bash
# Commit con mensaje
git commit -m "Primer commit: agrega README y archivo principal"

# Commit con editor (mensaje más detallado)
git commit
# Se abrirá tu editor configurado para escribir el mensaje

# Ver que el commit se realizó
git status
```

### Explorando el historial

#### Ver historial de commits
```bash
# Historial completo
git log

# Historial compacto (una línea por commit)
git log --oneline

# Historial con gráfico de ramas
git log --oneline --graph

# Historial con estadísticas
git log --stat

# Últimos N commits
git log -3
```

#### Ver detalles de un commit específico
```bash
# Ver último commit
git show

# Ver commit específico por hash
git show abc1234

# Ver solo los archivos que cambiaron
git show --name-only abc1234
```

### Workflow típico después del primer commit

```bash
# 1. Modificar archivos
echo 'print("Versión 2!")' >> main.py

# 2. Ver cambios
git status
git diff

# 3. Añadir cambios
git add main.py

# 4. Commit
git commit -m "Actualiza mensaje en main.py"

# 5. Ver historial
git log --oneline
```

## 🧠 Conceptos Clave de Git

### Terminología esencial

#### Repositorio (Repository)
- **Definición**: Carpeta que contiene tu proyecto y toda su historia de versiones
- **Ubicación**: Local (en tu computadora) o remoto (GitHub, GitLab)
- **Contenido**: Archivos del proyecto + directorio .git con metadatos

#### Commit
- **Definición**: "Fotografía" de tu proyecto en un momento específico
- **Contenido**: Cambios, metadatos (autor, fecha, mensaje), hash único
- **Característica**: Inmutable una vez creado

```bash
# Anatomía de un commit
commit 2f8a7bc91e4d5a6b3c8f9e1d2a3b4c5e6f7g8h9i
Author: Tu Nombre <tu@email.com>
Date:   Mon Jan 15 10:30:25 2024 -0300
    
    Agrega funcionalidad de login de usuarios
```

#### Branch (Rama)
- **Definición**: Línea de desarrollo independiente
- **Propósito**: Aislar trabajo en funcionalidades o experimentos
- **Rama principal**: `main` (antes `master`)

#### Merge
- **Definición**: Unir cambios de una rama a otra
- **Tipos**: Fast-forward, recursive, squash
- **Resultado**: Combinar historial de diferentes líneas de desarrollo

#### Remote (Remoto)
- **Definición**: Versión del repositorio alojada en servidor externo
- **Propósito**: Colaboración, respaldo, sincronización
- **Ejemplos**: GitHub, GitLab, Bitbucket

### Hash de commit (SHA-1)
Cada commit tiene un identificador único de 40 caracteres:
```
2f8a7bc91e4d5a6b3c8f9e1d2a3b4c5e6f7g8h9i
```

**Características**:
- Generado criptográficamente
- Garantiza integridad de datos
- Se puede abreviar (primeros 7-8 caracteres)
- Permite referenciar commits específicos

### Conceptos de flujo de trabajo

#### HEAD
- **Definición**: Puntero al commit actual en el que estás trabajando
- **Ubicación**: Normalmente apunta al último commit de la rama activa
- **Simbolo**: `HEAD`, `HEAD~1` (commit anterior), `HEAD~2` (dos commits atrás)

#### Working Directory vs Repository
```
Working Directory    Repository
     (Actual)         (Historial)
        |                 |
   Archivos que      Commits con
   estás editando    versiones guardadas
```

#### Staging Area (Index)
```bash
# Ver qué está en staging area
git status

# Ver diferencias entre working directory y staging
git diff

# Ver diferencias entre staging y último commit
git diff --cached
```

## 🎯 Ejercicios Prácticos

### Ejercicio 1: Configuración inicial
**Objetivo**: Configurar Git por primera vez

1. **Instala Git** en tu sistema operativo
2. **Verifica la instalación**:
   ```bash
   git --version
   ```
3. **Configura tu identidad**:
   ```bash
   git config --global user.name "Tu Nombre Real"
   git config --global user.email "tu-email@ejemplo.com"
   ```
4. **Configura tu editor** (ejemplo con VS Code):
   ```bash
   git config --global core.editor "code --wait"
   ```
5. **Verifica tu configuración**:
   ```bash
   git config --list --global
   ```

### Ejercicio 2: Primer repositorio local
**Objetivo**: Crear y gestionar tu primer repositorio

1. **Crea un nuevo directorio**:
   ```bash
   mkdir mi-portafolio
   cd mi-portafolio
   ```

2. **Inicializa el repositorio**:
   ```bash
   git init
   ```

3. **Crea archivos iniciales**:
   ```bash
   echo "# Mi Portafolio Personal" > README.md
   echo "Este es mi primer proyecto con Git" >> README.md
   echo "<h1>Bienvenido</h1>" > index.html
   ```

4. **Verifica el estado**:
   ```bash
   git status
   ```

5. **Añade archivos al staging**:
   ```bash
   git add README.md
   git add index.html
   # O alternativamente: git add .
   ```

6. **Haz tu primer commit**:
   ```bash
   git commit -m "Primer commit: agrega README e index inicial"
   ```

7. **Visualiza el historial**:
   ```bash
   git log
   git log --oneline
   ```

### Ejercicio 3: Simulando desarrollo iterativo
**Objetivo**: Practicar el flujo básico de Git

1. **Modifica el archivo index.html**:
   ```bash
   echo "<p>Soy desarrollador en aprendizaje</p>" >> index.html
   ```

2. **Crea un nuevo archivo**:
   ```bash
   echo "body { font-family: Arial; }" > styles.css
   ```

3. **Verifica cambios**:
   ```bash
   git status
   git diff index.html
   ```

4. **Añade y confirma cambios**:
   ```bash
   git add index.html styles.css
   git commit -m "Agrega descripción personal y estilos CSS"
   ```

5. **Realiza más cambios**:
   ```bash
   echo "<link rel='stylesheet' href='styles.css'>" >> index.html
   echo "## Mis Proyectos" >> README.md
   echo "- Portafolio personal (este proyecto)" >> README.md
   ```

6. **Commit por separado**:
   ```bash
   git add index.html
   git commit -m "Vincula hoja de estilos CSS"
   
   git add README.md
   git commit -m "Agrega sección de proyectos al README"
   ```

7. **Visualiza tu progreso**:
   ```bash
   git log --oneline --graph
   ```

### Ejercicio 4: Explorando el historial
**Objetivo**: Navegar y entender el historial de commits

1. **Ver detalles del último commit**:
   ```bash
   git show
   ```

2. **Ver cambios entre commits**:
   ```bash
   git log --stat
   ```

3. **Ver un commit específico** (usa el hash de uno de tus commits):
   ```bash
   git show [hash-de-commit]
   ```

4. **Ver solo los nombres de archivos modificados**:
   ```bash
   git log --name-only
   ```

### ✅ Checklist de verificación
Al completar estos ejercicios, deberías tener:

- [ ] Git instalado y configurado
- [ ] Un repositorio local funcional
- [ ] Al menos 3-4 commits en tu historial
- [ ] Experiencia con `git add`, `git commit`, `git status`
- [ ] Conocimiento de `git log` en diferentes formatos
- [ ] Comprensión del flujo básico: modificar → add → commit

## 🔧 Solución de Problemas Comunes

### Error: "git: command not found"
**Problema**: Git no está instalado o no está en el PATH
```bash
# Verificar instalación
git --version

# Si no funciona, reinstalar Git y reiniciar terminal
```

### Error: "Please tell me who you are"
**Problema**: Falta configuración de identidad
```bash
# Solución
git config --global user.name "Tu Nombre"
git config --global user.email "tu@email.com"
```

### Error: "fatal: not a git repository"
**Problema**: Intentas usar Git en un directorio que no es repositorio
```bash
# Verificar si estás en un repositorio
ls -la .git

# Si no existe, inicializar o cambiar de directorio
git init  # Crear nuevo repositorio
# o
cd /ruta/a/tu/repositorio  # Ir a repositorio existente
```

### Error: "nothing added to commit but untracked files present"
**Problema**: Archivos creados pero no añadidos al staging
```bash
# Ver archivos no trackeados
git status

# Añadir archivos específicos
git add archivo.txt

# O añadir todos
git add .
```

### Error: "Your branch is ahead of 'origin/main'"
**Problema**: Tienes commits locales que no están en el servidor
```bash
# Solución (veremos más en módulo 2)
git push origin main
```

### Revertir cambios no confirmados
```bash
# Revertir un archivo específico
git checkout -- archivo.txt

# Revertir todos los cambios no confirmados
git checkout -- .

# Quitar archivo del staging area
git reset archivo.txt
```

## 📋 Buenas Prácticas para Principiantes

### Mensajes de commit efectivos

#### ✅ Buenos mensajes
```bash
git commit -m "Agrega validación de formulario de login"
git commit -m "Corrige error en cálculo de precios"
git commit -m "Actualiza documentación de instalación"
```

#### ❌ Malos mensajes
```bash
git commit -m "fix"
git commit -m "changes"
git commit -m "wip"
git commit -m "asdasd"
```

#### Estructura recomendada
```
Tipo: Descripción concisa (50 caracteres max)

Explicación detallada si es necesario (72 caracteres por línea).
Puede incluir:
- Por qué se hizo el cambio
- Qué problema resuelve
- Referencias a issues (#123)
```

### Frecuencia de commits

#### ✅ Hacer commit cuando:
- Completas una funcionalidad pequeña
- Solucionas un bug específico
- Refactorizas código exitosamente
- Agregas documentación importante

#### ❌ Evitar:
- Commits demasiado grandes (muchos archivos/cambios)
- Commits con código que no funciona
- Commits con cambios no relacionados mezclados

### Organización de archivos

#### Usar .gitignore desde el inicio
```bash
# Crear .gitignore
touch .gitignore

# Contenido típico para proyectos generales
echo "*.log" >> .gitignore
echo "*.tmp" >> .gitignore
echo ".DS_Store" >> .gitignore
echo "node_modules/" >> .gitignore
echo ".env" >> .gitignore
```

#### Estructura de proyecto recomendada
```
mi-proyecto/
├── .gitignore
├── README.md
├── LICENSE.md
├── src/          # Código fuente
├── docs/         # Documentación
├── tests/        # Pruebas
└── assets/       # Recursos (imágenes, etc.)
```

### Comandos que debes memorizar

#### Nivel principiante (esencial)
```bash
git init          # Crear repositorio
git add .         # Añadir archivos
git commit -m ""  # Confirmar cambios
git status        # Ver estado
git log           # Ver historial
```

#### Nivel intermedio (próximos módulos)
```bash
git clone         # Copiar repositorio
git push          # Subir cambios
git pull          # Descargar cambios
git branch        # Gestionar ramas
git merge         # Unir ramas
```

### Mentalidad de control de versiones

#### ✅ Filosofía correcta
- Git es tu "máquina del tiempo" para código
- Cada commit es un punto de restauración
- El historial es valioso, mantenlo limpio
- Commitea temprano y frecuentemente

#### ❌ Conceptos erróneos comunes
- "Git es solo para backup" (es mucho más)
- "Solo commito cuando está perfecto" (commita progreso)
- "No necesito mensajes descriptivos" (tu yo futuro te agradecerá)
- "Git es complicado" (con práctica se vuelve natural)

---
**Siguiente**: [02. Repositorios Remotos y GitHub](./02-remotos.md) | **Inicio**: [README](../README.md)