# 05. Seguridad y Buenas Pr�cticas

## ?? Seguridad en Git y GitHub

### Llaves SSH y credenciales
- Genera una llave SSH: `ssh-keygen -t ed25519 -C "tu@email.com"`
- Agrega la llave p�blica a GitHub
- Usa HTTPS o SSH para autenticaci�n

### Protecci�n de ramas
- Configura reglas en GitHub para evitar pushes directos a main
- Requiere revisiones antes de merge

### Gesti�n de permisos y roles
- Usa equipos y roles en GitHub (admin, maintainer, contributor)
- Limita acceso a repositorios privados

## ??? Buenas pr�cticas
- No subas archivos sensibles (usa `.gitignore`)
- Haz commits descriptivos y frecuentes
- Revisa el historial antes de hacer merge

## ?? Ejercicio Pr�ctico
1. Configura una llave SSH y �sala en GitHub
2. Protege la rama principal de tu repositorio
3. Crea un archivo `.gitignore` y agrega archivos a ignorar

---
**Anterior**: [04. Conflictos y Avanzado](./04-conflictos.md) | **Siguiente**: [06. Automatizaci�n y DevOps](./06-automatizacion.md)