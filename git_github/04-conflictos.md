# 04. Resolución de Conflictos y Comandos Avanzados

## ⚔️ ¿Qué son los conflictos de merge?
Los conflictos ocurren cuando Git no puede fusionar automáticamente dos ramas porque ambas han modificado las mismas líneas de un archivo. Es una situación normal en desarrollo colaborativo que requiere intervención manual.

### Cuándo ocurren conflictos
- **Modificaciones simultáneas**: Mismas líneas cambiadas en diferentes ramas
- **Archivo renombrado**: Una rama renombra, otra modifica
- **Eliminación vs modificación**: Una rama elimina archivo, otra lo modifica
- **Merge vs rebase**: Durante operaciones de reorganización

### Tipos de conflictos
1. **Content conflict**: Cambios en el mismo contenido
2. **Rename conflict**: Conflictos de nombres de archivo
3. **Delete/modify conflict**: Uno elimina, otro modifica
4. **Mode conflict**: Cambios en permisos de archivo

## 🔍 Identificación de Conflictos

### Síntomas de conflicto durante merge
```bash
git merge feature/nueva-funcionalidad
# Auto-merging archivo.txt
# CONFLICT (content): Merge conflict in archivo.txt
# Automatic merge failed; fix conflicts and then commit the result.

git status
# On branch main
# You have unmerged paths.
#   (fix conflicts and run "git commit")
#   (use "git merge --abort" to abort the merge)
#
# Unmerged paths:
#   (use "git add <file>..." to mark resolution)
#         both modified:   archivo.txt
```

### Anatomy de un conflicto
```text
<<<<<<< HEAD (rama actual)
Contenido de la rama actual (main)
=======
Contenido de la rama entrante (feature)
>>>>>>> feature/nueva-funcionalidad
```

## 🛠️ Resolución Manual de Conflictos

### Paso a paso para resolver conflictos
1. **Identificar archivos en conflicto**
   ```bash
   git status
   git diff  # Ver detalles del conflicto
   ```

2. **Abrir archivo y resolver**
   ```bash
   code archivo-en-conflicto.txt
   # O cualquier editor de tu preferencia
   ```

3. **Decidir qué contenido mantener**
   - Mantener solo la versión actual (HEAD)
   - Mantener solo la versión entrante
   - Combinar ambas versiones
   - Crear solución completamente nueva

4. **Limpiar marcadores de conflicto**
   - Eliminar `<<<<<<<`, `=======`, `>>>>>>>`
   - Verificar que el código funcione correctamente

5. **Marcar como resuelto**
   ```bash
   git add archivo-resuelto.txt
   ```

6. **Completar el merge**
   ```bash
   git commit  # Git crea mensaje automático
   # O personalizar: git commit -m "Resuelve conflicto en archivo.txt"
   ```

### Ejemplo práctico de resolución
**Archivo original:**
```javascript
function saludar(nombre) {
    console.log("Hola " + nombre);
}
```

**Conflicto:**
```javascript
function saludar(nombre) {
<<<<<<< HEAD
    console.log("Hola " + nombre + "!");
    return "Saludo enviado";
=======
    console.log(`¡Hola ${nombre}! ¿Cómo estás?`);
>>>>>>> feature/mejorar-saludo
}
```

**Resolución combinada:**
```javascript
function saludar(nombre) {
    console.log(`¡Hola ${nombre}! ¿Cómo estás?`);
    return "Saludo enviado";
}
```

## 🔧 Herramientas para Resolución de Conflictos

### Configurar herramienta de merge
```bash
# VS Code
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait $MERGED'

# Meld (Linux)
git config --global merge.tool meld

# KDiff3 (Multiplataforma)
git config --global merge.tool kdiff3

# P4Merge (Visual)
git config --global merge.tool p4merge
```

### Usar herramienta de merge
```bash
git mergetool
# Abre la herramienta configurada para resolver conflictos visualmente
```

### Comandos de resolución rápida
```bash
# Aceptar versión actual (HEAD)
git checkout --ours archivo.txt

# Aceptar versión entrante
git checkout --theirs archivo.txt

# Ver diferencias durante conflicto
git diff --name-only --diff-filter=U  # Archivos no fusionados
git show :1:archivo.txt  # Versión ancestro común
git show :2:archivo.txt  # Versión HEAD
git show :3:archivo.txt  # Versión merge
```

## 🔄 Rebase: Alternativa al Merge

### ¿Qué es rebase?
Rebase reescribe el historial aplicando commits de una rama sobre otra, creando una historia lineal más limpia.

```bash
# Rebase básico
git checkout feature/nueva-funcionalidad
git rebase main

# Rebase interactivo
git rebase -i HEAD~3
```

### Rebase vs Merge
| Rebase | Merge |
|--------|-------|
| Historia lineal | Historia ramificada |
| Reescribe commits | Preserva historial original |
| Ideal para features | Ideal para integraciones importantes |
| Más limpio | Más seguro |

### Resolución de conflictos en rebase
```bash
# Durante rebase con conflicto
git status  # Ver archivos en conflicto
# Resolver conflictos manualmente
git add archivo-resuelto.txt
git rebase --continue

# Abortar rebase si es necesario
git rebase --abort

# Saltar commit problemático (¡cuidado!)
git rebase --skip
```

## ⚡ Comandos Avanzados de Git

### Cherry-pick
Aplica commits específicos de una rama a otra:
```bash
# Cherry-pick commit específico
git cherry-pick abc1234

# Cherry-pick rango de commits
git cherry-pick abc1234^..def5678

# Cherry-pick con conflicto
git cherry-pick abc1234
# Resolver conflictos
git add .
git cherry-pick --continue
```

### Revert
Deshace cambios creando un nuevo commit:
```bash
# Revertir último commit
git revert HEAD

# Revertir commit específico
git revert abc1234

# Revertir merge commit
git revert -m 1 abc1234
```

### Reset (¡CUIDADO!)
Mueve HEAD y opcionalmente modifica staging y working directory:
```bash
# Reset suave (mantiene cambios)
git reset --soft HEAD~1

# Reset mixto (default, mantiene working directory)
git reset HEAD~1

# Reset duro (¡PIERDE CAMBIOS!)
git reset --hard HEAD~1

# Reset a commit específico
git reset --hard abc1234
```

### Stash avanzado
```bash
# Stash con mensaje
git stash push -m "Trabajo en progreso en login"

# Stash archivos específicos
git stash push -m "Solo cambios CSS" styles.css

# Stash con archivos no trackeados
git stash -u

# Lista de stashes
git stash list

# Aplicar stash específico
git stash apply stash@{2}

# Ver contenido de stash
git stash show -p stash@{0}

# Crear rama desde stash
git stash branch nueva-rama stash@{0}
```

## 🧪 Estrategias de Merge

### 1. Merge commit (--no-ff)
```bash
git merge --no-ff feature/nueva-funcionalidad
```
- **Ventajas**: Preserva contexto e historial completo
- **Desventajas**: Historial más complejo visualmente

### 2. Fast-forward merge (default)
```bash
git merge feature/nueva-funcionalidad  # Si es posible
```
- **Ventajas**: Historial lineal y simple
- **Desventajas**: Pierde información de que fue una rama

### 3. Squash merge
```bash
git merge --squash feature/nueva-funcionalidad
git commit -m "Implementa nueva funcionalidad"
```
- **Ventajas**: Un solo commit por feature
- **Desventajas**: Pierde historial detallado de desarrollo

### 4. Rebase y merge
```bash
git checkout feature/nueva-funcionalidad
git rebase main
git checkout main
git merge feature/nueva-funcionalidad  # Fast-forward
```
- **Ventajas**: Historial limpio y lineal
- **Desventajas**: Reescribe historial

## 🏋️ Ejercicios Prácticos

### Ejercicio 1: Conflicto básico
1. Crea repositorio con archivo README
2. Crea dos ramas que modifiquen la misma línea
3. Intenta merge y resuelve el conflicto
4. Verifica el resultado

### Ejercicio 2: Conflicto complejo
1. Crea archivo con múltiples funciones
2. En rama A: modifica función 1 y 3
3. En rama B: modifica función 1 y 2
4. Genera conflicto y resuelve inteligentemente

### Ejercicio 3: Rebase interactivo
1. Crea rama con 5 commits pequeños
2. Usa `git rebase -i` para:
   - Combinar 2 commits (squash)
   - Cambiar mensaje de 1 commit (reword)
   - Reordenar commits
3. Compara historial antes y después

### Ejercicio 4: Cherry-pick selectivo
1. Crea rama experimental con 3 commits
2. Solo quieres 1 commit específico en main
3. Usa cherry-pick para traer solo ese commit
4. Maneja cualquier conflicto que surja

## 🎯 Buenas Prácticas para Conflictos

### Prevención
- **Comunicación**: Coordina cambios en archivos importantes
- **Pulls frecuentes**: Mantén tu rama actualizada
- **Commits pequeños**: Facilita resolución de conflictos
- **Modularidad**: Separa funcionalidades en archivos diferentes

### Durante resolución
- **Entiende el contexto**: Lee ambas versiones completamente
- **Prueba el resultado**: Verifica que el código funcione
- **Conserva funcionalidad**: No rompas features existentes
- **Documenta decisiones**: Explica resoluciones complejas

### Después de resolver
- **Tests**: Ejecuta pruebas completas
- **Review**: Pide revisión adicional si es complejo
- **Documenta**: Actualiza documentación si es necesario

## 🔧 Solución de Problemas Avanzados

### "Detached HEAD state"
```bash
git status  # Verifica el estado
git checkout main  # Regresa a rama
# O crear rama desde HEAD actual:
git checkout -b nueva-rama
```

### Recuperar commits perdidos
```bash
git reflog  # Ver historial de movimientos de HEAD
git checkout abc1234  # Ir al commit perdido
git checkout -b rama-recuperada  # Crear rama desde ahí
```

### Deshacer merge problemático
```bash
# Si no has hecho push aún
git reset --hard HEAD~1

# Si ya hiciste push
git revert -m 1 abc1234  # abc1234 = hash del merge commit
```

### Limpiar repositorio después de conflictos
```bash
git clean -fd  # Elimina archivos no trackeados
git reset --hard HEAD  # Descarta cambios locales
```

---
**Anterior**: [03. Ramas y Flujos de Trabajo](./03-ramas.md) | **Siguiente**: [05. Seguridad y Buenas Prácticas](./05-seguridad.md)