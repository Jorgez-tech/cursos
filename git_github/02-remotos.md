# 02. Repositorios Remotos y GitHub

## ?? ¿Qué es un repositorio remoto?
Un repositorio remoto es una copia de tu proyecto alojada en un servidor (por ejemplo, GitHub) que permite la colaboración y el respaldo.

## ?? Conexión con GitHub

### Crear un repositorio en GitHub
1. Ve a [github.com](https://github.com/) y crea un nuevo repositorio
2. Copia la URL (HTTPS o SSH)

### Conectar tu repositorio local
```bash
git remote add origin https://github.com/usuario/mi-proyecto.git
git push -u origin main
```

## ?? Comandos Clave
- **Clonar**: `git clone URL`
- **Push**: `git push origin main` (sube cambios)
- **Pull**: `git pull origin main` (descarga cambios)
- **Fetch**: `git fetch origin` (descarga sin mezclar)

## ?? Forks y Upstream
- **Fork**: Copia de un repositorio para trabajar de forma independiente
- **Upstream**: Repositorio original del que se hizo fork

## ?? Ejercicio Práctico
1. Crea un repositorio en GitHub
2. Conéctalo con tu repositorio local
3. Haz push y pull de cambios

---
**Anterior**: [01. Introducción](./01-introduccion.md) | **Siguiente**: [03. Ramas y Flujos de Trabajo](./03-ramas.md)