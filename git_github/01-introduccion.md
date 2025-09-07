# 01. Introducci�n a Git y GitHub

## ?? �Qu� es Git?
Git es un sistema de control de versiones distribuido que permite gestionar el historial de cambios en proyectos de software. Fue creado por Linus Torvalds en 2005 y es la base de la colaboraci�n moderna en desarrollo.

### Ventajas de Git
- Permite trabajar en equipo sin sobrescribir el trabajo de otros
- Guarda el historial completo de cambios
- Facilita la recuperaci�n de versiones anteriores
- Es r�pido, seguro y ampliamente adoptado

## ??? Instalaci�n y Configuraci�n

### Instalaci�n de Git
- **Windows**: `winget install --id Git.Git`
- **macOS**: `brew install git`
- **Linux**: `sudo apt-get install git`

### Configuraci�n inicial
```bash
git config --global user.name "Tu Nombre"
git config --global user.email "tu@email.com"
git config --global core.editor "code --wait" # VS Code como editor
```

## ?? Primer Repositorio y Commit

### Crear un nuevo repositorio
```bash
mkdir mi-proyecto
cd mi-proyecto
git init
```

### Flujo b�sico de trabajo
```bash
touch README.md
git add README.md
git commit -m "Primer commit: agrega README"
```

### Ver historial de cambios
```bash
git log --oneline
```

## ?? Conceptos Clave
- **Repositorio**: Carpeta gestionada por Git
- **Commit**: Registro de cambios
- **Branch (rama)**: L�nea de desarrollo paralela
- **Merge**: Unir ramas
- **Remote**: Repositorio alojado en la nube (GitHub)

## ?? Ejercicio Pr�ctico
1. Instala Git y configura tu usuario
2. Crea un repositorio local y haz tu primer commit
3. Visualiza el historial con `git log`

---
**Siguiente**: [02. Repositorios Remotos y GitHub](./02-remotos.md) | **Inicio**: [README](../README.md)