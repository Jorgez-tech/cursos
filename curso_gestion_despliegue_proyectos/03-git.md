# 03. Git y Control de Versiones

## 🗂️ Introducción a Git

Git es el sistema de control de versiones más utilizado en desarrollo de software. Permite gestionar cambios, colaborar y mantener la trazabilidad de los proyectos.

### Historia y evolución

- **2005**: Creación por Linus Torvalds para el kernel de Linux
- **2008**: GitHub populariza el uso colaborativo
- **Actualidad**: Integración con CI/CD, DevOps y gestión ágil

## 🏗️ Conceptos Clave

- Repositorio local y remoto
- Commits y ramas
- Merge y resolución de conflictos
- Etiquetas y versiones
- Flujos de trabajo (feature branch, git flow)

## ⚡ Ejemplo: Flujo básico con Git

```bash
# Inicializar repositorio
git init

# Agregar archivos y hacer commit
git add .
git commit -m "Primer commit"

# Crear y cambiar de rama
git branch desarrollo
git checkout desarrollo

# Fusionar cambios
git merge main

# Subir a remoto
git remote add origin https://github.com/usuario/repositorio.git
git push -u origin main
```

## 📝 Ejercicio Práctico

1. Crea un repositorio local y realiza al menos 3 commits.
2. Simula un conflicto de merge y resuélvelo.

## 🎯 Buenas Prácticas

- Escribe mensajes de commit claros y descriptivos
- Realiza commits pequeños y frecuentes
- Usa ramas para nuevas funcionalidades

---

**Siguiente**: [04. Documentación de Proyectos](./04-documentacion.md)
