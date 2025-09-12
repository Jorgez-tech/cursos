# 04. Dependencias y Entornos de Desarrollo

## 🧩 Gestión de Dependencias

Las dependencias son librerías, frameworks y herramientas externas que utiliza un proyecto. Una gestión adecuada evita conflictos, facilita actualizaciones y mejora la seguridad.

### Historia y evolución

- **Década de 2000**: Uso de archivos de configuración manuales (DLLs, JARs)
- **2010s**: Gestores automáticos (NuGet, npm, Maven, pip)
- **Actualidad**: Entornos virtuales, contenedores y automatización

## 🏗️ Tipos de Entornos

- Entorno local (desarrollo)
- Entorno de pruebas (QA, staging)
- Entorno de producción
- Entornos virtuales y contenedores (Docker, venv)

## ⚡ Ejemplo: Configuración de dependencias en Node.js y Python

```bash
# Node.js
npm init -y
npm install express mongoose

# Python
python -m venv venv
source venv/bin/activate
pip install flask sqlalchemy
```

## 📝 Ejercicio Práctico

1. Crea un entorno virtual y gestiona dependencias para un proyecto Python.
2. Configura un archivo Dockerfile básico para una aplicación web.

## 🎯 Buenas Prácticas

- Documenta las dependencias y versiones
- Usa entornos aislados para cada proyecto
- Automatiza la instalación y actualización de dependencias

---

**Siguiente**: [05. CI/CD y Automatización](./05-cicd.md)
