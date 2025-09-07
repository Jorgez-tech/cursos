# 06. Automatizaci�n y DevOps con GitHub Actions

## ?? �Qu� es GitHub Actions?
GitHub Actions permite automatizar flujos de trabajo como pruebas, despliegues y an�lisis de c�digo directamente en GitHub.

## ??? Workflows y pipelines CI/CD
- Los workflows se definen en `.github/workflows/*.yml`
- Ejemplo b�sico:
```yaml
name: CI
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Instalar dependencias
        run: npm install
      - name: Ejecutar pruebas
        run: npm test
```

## ?? Ejecuci�n de pruebas autom�ticas
- Configura jobs para ejecutar tests en cada push o pull request
- Usa matrices para probar en m�ltiples versiones de Node, Python, etc.

## ?? Despliegue continuo
- Integra con servicios como Azure, AWS, Vercel
- Usa secrets para credenciales seguras

## ?? Ejercicio Pr�ctico
1. Crea un workflow b�sico que ejecute pruebas al hacer push
2. Agrega un job de despliegue (simulado)
3. Revisa el historial de ejecuciones en GitHub

---
**Anterior**: [05. Seguridad](./05-seguridad.md) | **Siguiente**: [07. Colaboraci�n Avanzada](./07-colaboracion.md)