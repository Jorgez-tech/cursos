# 06. CI/CD y Automatización

## ⚡ Introducción a CI/CD

La Integración Continua (CI) y el Despliegue Continuo (CD) son prácticas clave para entregar software de calidad de forma rápida y confiable. Automatizan pruebas, builds y despliegues.

### Historia y evolución

- **2000s**: Herramientas como Jenkins y Travis CI
- **2010s**: GitHub Actions, GitLab CI, Azure DevOps
- **Actualidad**: Pipelines declarativos, automatización avanzada, DevOps

## 🏗️ Componentes de CI/CD

- Repositorio de código fuente
- Pipeline de integración y despliegue
- Automatización de pruebas
- Entornos de staging y producción

## ⚡ Ejemplo: Pipeline básico con GitHub Actions

```yaml
name: CI/CD
on:
	push:
		branches: [main]
jobs:
	build:
		runs-on: ubuntu-latest
		steps:
			- uses: actions/checkout@v3
			- name: Instalar dependencias
				run: npm install
			- name: Ejecutar pruebas
				run: npm test
			- name: Desplegar
				run: echo "Despliegue exitoso"
```

## 📝 Ejercicio Práctico

1. Crea un pipeline de CI/CD para una app Node.js o Python.
2. Automatiza pruebas y despliegue en un entorno de staging.

## 🎯 Buenas Prácticas

- Automatiza la validación de código y pruebas
- Usa variables de entorno seguras
- Monitorea y revisa los pipelines periódicamente

---

**Siguiente**: [07. Contenedores y Virtualización](./07-contenedores.md)
