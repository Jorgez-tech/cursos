# 04. Resolución de Conflictos y Comandos Avanzados

## ? ¿Qué es un conflicto?
Un conflicto ocurre cuando dos ramas modifican la misma línea de un archivo y Git no puede fusionarlas automáticamente.

## ??? Resolución de conflictos
- Al hacer `git merge` o `git pull`, Git marca los archivos en conflicto
- Edita el archivo y elige qué cambios conservar
- Marca el conflicto como resuelto:
```bash
git add archivo-en-conflicto
git commit
```

## ?? Comandos avanzados
- **Rebase**: `git rebase rama-base` (reaplica commits sobre otra rama)
- **Cherry-pick**: `git cherry-pick <hash>` (aplica un commit específico)
- **Revert**: `git revert <hash>` (crea un commit que deshace otro)

## ?? Estrategias de merge
- Fast-forward, recursive, octopus
- Usar `git merge --no-ff` para mantener historial

## ?? Ejercicio Práctico
1. Simula un conflicto editando la misma línea en dos ramas
2. Resuelve el conflicto y haz commit
3. Prueba rebase y cherry-pick en tu repositorio

---
**Anterior**: [03. Ramas y Flujos](./03-ramas.md) | **Siguiente**: [05. Seguridad y Buenas Prácticas](./05-seguridad.md)