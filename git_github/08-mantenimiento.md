# 08. Mantenimiento, Auditoría y Eficiencia

## 🧹 Limpieza y Optimización de Repositorios

### ¿Por qué es importante el mantenimiento?
Los repositorios acumulan "deuda técnica" con el tiempo: ramas obsoletas, archivos innecesarios, historial complejo y referencias rotas. El mantenimiento regular mejora rendimiento, claridad y eficiencia del equipo.

### Beneficios del mantenimiento proactivo
- **Performance**: Repositorios más rápidos y eficientes
- **Claridad**: Estructura limpia y navegable
- **Seguridad**: Elimina código y credenciales obsoletas
- **Collaboration**: Facilita trabajo en equipo
- **Storage**: Reduce uso de espacio en disco

## 🗂️ Limpieza de Ramas

### Identificar ramas obsoletas
```bash
# Ver todas las ramas locales
git branch -v

# Ver ramas ya fusionadas a main
git branch --merged main

# Ver ramas remotas
git branch -r

# Ver últimas actividades por rama
git for-each-ref --format='%(committerdate) %09 %(authorname) %09 %(refname)' | sort -k5n -k2M -k3n -k4n
```

### Limpieza sistemática de ramas
```bash
# 1. Actualizar referencias remotas
git fetch --all --prune

# 2. Ver ramas fusionadas (candidatas para eliminación)
git branch --merged main | grep -v "main\|master\|develop"

# 3. Eliminar ramas locales fusionadas
git branch --merged main | grep -v "main\|master\|develop" | xargs -n 1 git branch -d

# 4. Eliminar referencias de ramas remotas borradas
git remote prune origin

# 5. Ver ramas no fusionadas (revisar manualmente)
git branch --no-merged main
```

### Script de limpieza automatizada
```bash
#!/bin/bash
# cleanup-repo.sh

echo "🧹 Iniciando limpieza del repositorio..."

# Actualizar desde remote
echo "📡 Actualizando referencias remotas..."
git fetch --all --prune

# Cambiar a rama principal
git checkout main
git pull origin main

# Eliminar ramas locales fusionadas
echo "🗑️ Eliminando ramas locales fusionadas..."
MERGED_BRANCHES=$(git branch --merged main | grep -v "main\|master\|develop" | tr -d ' ')
if [ ! -z "$MERGED_BRANCHES" ]; then
    echo "$MERGED_BRANCHES" | xargs -n 1 git branch -d
    echo "✅ Ramas eliminadas: $MERGED_BRANCHES"
else
    echo "ℹ️ No hay ramas fusionadas para eliminar"
fi

# Limpiar referencias remotas
echo "🧽 Limpiando referencias remotas obsoletas..."
git remote prune origin

# Optimizar repositorio
echo "⚡ Optimizando repositorio..."
git gc --aggressive --prune=now

echo "✨ Limpieza completada!"
```

### Política de ramas para equipos
```markdown
## Branch Lifecycle Policy

### Naming Convention
- `feature/JIRA-123-user-authentication`
- `bugfix/fix-login-validation`  
- `hotfix/security-patch-v2.1.1`
- `release/v2.1.0`

### Lifetime Rules
- **Feature branches**: Eliminar después del merge
- **Release branches**: Mantener por 6 meses después de release
- **Hotfix branches**: Eliminar después del merge
- **Experimental**: Eliminar si no hay actividad en 30 días

### Automated Cleanup
- CI/CD elimina ramas después de merge exitoso
- Bot semanal identifica ramas obsoletas
- Notificación a owner antes de eliminación automática
```

## 📁 Gestión de Archivos y .gitignore

### Limpiar archivos no deseados
```bash
# Ver archivos no trackeados
git clean -n  # Dry run (ver qué se eliminaría)

# Eliminar archivos no trackeados
git clean -f

# Eliminar archivos y directorios no trackeados
git clean -fd

# Eliminar incluyendo archivos ignorados
git clean -fxd
```

### Optimizar .gitignore
```gitignore
# === .gitignore optimizado ===

# Dependencies
node_modules/
vendor/
.pnp
.pnp.js

# Production builds
/build
/dist
/out
/.next/
/.nuxt/

# Environment variables
.env
.env.local
.env.*.local

# Logs
logs
*.log
npm-debug.log*
yarn-debug.log*

# Runtime data
pids
*.pid
*.seed
*.pid.lock

# IDE/Editor files
.vscode/
.idea/
*.swp
*.swo
*~

# OS generated files
.DS_Store
.DS_Store?
._*
.Spotlight-V100
.Trashes
ehthumbs.db
Thumbs.db

# Temporary files
*.tmp
*.temp
/tmp/

# Cache directories
.cache/
.parcel-cache/
.eslintcache

# Coverage reports
coverage/
*.lcov
.nyc_output

# Dependency directories
node_modules/
jspm_packages/

# Package managers
package-lock.json  # Si usas yarn
yarn.lock         # Si usas npm

# Secrets (¡NUNCA commitear!)
*.key
*.pem
*.p12
*.cert
secrets.txt
```

### Eliminar archivos del historial (¡CUIDADO!)
```bash
# Para archivos grandes accidentalmente comiteados
git filter-branch --tree-filter 'rm -f large-file.zip' HEAD

# Método más moderno con git-filter-repo
pip install git-filter-repo
git filter-repo --path large-file.zip --invert-paths

# Para credenciales comiteadas por error
git filter-repo --replace-text <(echo 'password123==>***REMOVED***')
```

## 🔍 Auditoría y Análisis

### Git blame y arqueología de código
```bash
# Ver quien modificó cada línea
git blame archivo.js

# Ver cambios en rango específico
git blame -L 10,20 archivo.js

# Ignorar cambios cosméticos (whitespace, etc.)
git blame -w archivo.js

# Ver historial de un archivo
git log --follow -p archivo.js

# Ver cuando se introdujo una línea específica
git log -S "function login" --source --all
```

### Análisis de contribuciones
```bash
# Estadísticas de contribuciones
git shortlog -s -n

# Contribuciones por período
git shortlog -s -n --since="2023-01-01" --until="2023-12-31"

# Archivos más modificados
git log --format=format: --name-only | egrep -v '^$' | sort | uniq -c | sort -rn | head -10

# Autores por archivo
git log --format='%aN' -- archivo.js | sort | uniq -c | sort -rn
```

### Identificar código problemático
```bash
# Archivos con más conflictos de merge
git log --grep="Merge conflict" --format='%s' | grep -o 'archivo[^:]*' | sort | uniq -c | sort -rn

# Commits más grandes (posibles refactors masivos)
git log --format='%h %s' --shortstat | grep -A1 -B1 "files changed" | head -20

# Archivos que cambian juntos frecuentemente
git log --format=format: --name-only | egrep -v '^$' | sort | uniq -c | sort -rn
```

### Scripts de análisis personalizado
```python
#!/usr/bin/env python3
# analyze-repo.py

import subprocess
import json
from collections import defaultdict, Counter

def get_commits_by_author():
    """Obtener commits por autor"""
    result = subprocess.run(['git', 'shortlog', '-s', '-n'], 
                          capture_output=True, text=True)
    authors = {}
    for line in result.stdout.strip().split('\n'):
        if line.strip():
            count, author = line.strip().split('\t', 1)
            authors[author] = int(count)
    return authors

def get_file_changes():
    """Obtener archivos más modificados"""
    result = subprocess.run(['git', 'log', '--format=format:', '--name-only'], 
                          capture_output=True, text=True)
    files = [f for f in result.stdout.split('\n') if f.strip()]
    return Counter(files)

def get_commit_sizes():
    """Obtener tamaños de commits"""
    result = subprocess.run(['git', 'log', '--format=%h,%s', '--shortstat'], 
                          capture_output=True, text=True)
    # Parsear output y analizar tamaños
    return result.stdout

if __name__ == "__main__":
    print("📊 Análisis del repositorio\n")
    
    authors = get_commits_by_author()
    print("🏆 Top contribuidores:")
    for author, count in list(authors.items())[:5]:
        print(f"  {author}: {count} commits")
    
    files = get_file_changes()
    print("\n📁 Archivos más modificados:")
    for file, count in files.most_common(5):
        print(f"  {file}: {count} cambios")
```

## 🐛 Git Bisect: Encontrar Bugs

### ¿Qué es git bisect?
Herramienta de búsqueda binaria para encontrar el commit que introdujo un bug.

### Proceso de bisect
```bash
# 1. Iniciar bisect
git bisect start

# 2. Marcar commit malo (actual)
git bisect bad

# 3. Marcar commit bueno (funcionaba antes)
git bisect good v2.0.0

# 4. Git checkouts commit medio, tu pruebas y marcas:
git bisect good    # Si funciona
# O
git bisect bad     # Si está roto

# 5. Repetir hasta encontrar el commit problemático
# 6. Terminar bisect
git bisect reset
```

### Bisect automatizado
```bash
# Script que prueba automáticamente
#!/bin/bash
# test-script.sh
npm test
exit $?

# Usar el script
git bisect run ./test-script.sh
```

### Ejemplo práctico de bisect
```bash
# Escenario: el login dejó de funcionar entre v1.0 y actual
git bisect start
git bisect bad HEAD           # HEAD está roto
git bisect good v1.0.0        # v1.0.0 funcionaba

# Git checkout automático a commit medio
# Probar: npm start -> ir a /login -> probar
git bisect bad    # Si no funciona

# Git hace siguiente checkout
# Probar de nuevo
git bisect good   # Si funciona

# Continuar hasta que git identifique el commit exacto
# Output: "abc1234 is the first bad commit"

git bisect reset  # Volver a HEAD
git show abc1234  # Ver qué cambió
```

## ⚡ Optimización de Performance

### Garbage collection y optimización
```bash
# Garbage collection básico
git gc

# Garbage collection agresivo (repositorios grandes)
git gc --aggressive --prune=now

# Ver estadísticas del repositorio
git count-objects -v

# Optimizar pack files
git repack -a -d --depth=250 --window=250

# Verificar integridad
git fsck --full
```

### Configuraciones de performance
```bash
# Optimizar para repositorios grandes
git config core.preloadindex true
git config core.fscache true
git config gc.auto 256

# Habilitar parallel processing
git config submodule.fetchJobs 8
git config fetch.parallel 8

# Optimizar diff performance
git config diff.algorithm patience
git config merge.renameLimit 999999
```

### Monitorear performance
```bash
# Medir tiempo de operaciones
time git status
time git log --oneline | head -100

# Ver tamaño de objetos grandes
git rev-list --objects --all | sort -k 2 | cut -f 2 -d' ' | uniq | while read filename; do
    echo "$(git rev-list --all --objects | grep $filename | wc -l) $filename"
done | sort -rn | head -10

# Identificar archivos grandes en historial
git rev-list --all --objects | cut -f 2 -d' ' | sort -u | while read filename; do
    if [ -n "$filename" ]; then
        size=$(git cat-file -s $(git rev-list --all --objects | grep $filename | cut -f 1 -d' ' | head -1) 2>/dev/null || echo 0)
        echo "$size $filename"
    fi
done | sort -rn | head -10
```

## 📊 Reporting y Métricas

### Generar reportes de actividad
```bash
# Reporte semanal de commits
git log --since="1 week ago" --format="%an: %s" | sort

# Líneas de código por autor
git log --author="Juan" --pretty=tformat: --numstat | awk '{ add += $1; subs += $2; loc += $1 - $2 } END { printf "added lines: %s, removed lines: %s, total lines: %s\n", add, subs, loc }'

# Commits por día de la semana
git log --format="%ad" --date=format:"%u" | sort | uniq -c

# Actividad por mes
git log --format="%ad" --date=format:"%Y-%m" | sort | uniq -c
```

### Dashboard de métricas (script Python)
```python
#!/usr/bin/env python3
# git-dashboard.py

import subprocess
import matplotlib.pyplot as plt
from datetime import datetime, timedelta
import pandas as pd

def get_commit_history(days=30):
    """Obtener historial de commits de últimos N días"""
    since = (datetime.now() - timedelta(days=days)).strftime('%Y-%m-%d')
    
    result = subprocess.run([
        'git', 'log', '--since', since, 
        '--format=%ad|%an|%s', '--date=short'
    ], capture_output=True, text=True)
    
    commits = []
    for line in result.stdout.strip().split('\n'):
        if '|' in line:
            date, author, message = line.split('|', 2)
            commits.append({
                'date': datetime.strptime(date, '%Y-%m-%d'),
                'author': author,
                'message': message
            })
    
    return pd.DataFrame(commits)

def generate_dashboard():
    df = get_commit_history(30)
    
    if df.empty:
        print("No hay commits en los últimos 30 días")
        return
    
    fig, ((ax1, ax2), (ax3, ax4)) = plt.subplots(2, 2, figsize=(15, 10))
    fig.suptitle('Git Repository Dashboard - Últimos 30 días', fontsize=16)
    
    # Commits por día
    daily_commits = df.groupby(df['date'].dt.date).size()
    ax1.plot(daily_commits.index, daily_commits.values)
    ax1.set_title('Commits por día')
    ax1.tick_params(axis='x', rotation=45)
    
    # Commits por autor
    author_commits = df['author'].value_counts()
    ax2.bar(author_commits.index[:10], author_commits.values[:10])
    ax2.set_title('Top 10 contribuidores')
    ax2.tick_params(axis='x', rotation=45)
    
    # Actividad por hora (si tienes fecha/hora completa)
    # commits_by_hour = df['date'].dt.hour.value_counts().sort_index()
    # ax3.bar(commits_by_hour.index, commits_by_hour.values)
    ax3.set_title('Patrones de actividad')
    
    # Longitud de mensajes de commit
    message_lengths = df['message'].str.len()
    ax4.hist(message_lengths, bins=20)
    ax4.set_title('Distribución de longitud de mensajes')
    
    plt.tight_layout()
    plt.savefig('git-dashboard.png', dpi=300, bbox_inches='tight')
    print("📊 Dashboard guardado como git-dashboard.png")

if __name__ == "__main__":
    generate_dashboard()
```

## 🏋️ Ejercicios Prácticos

### Ejercicio 1: Limpieza completa de repositorio
1. Clona un repositorio con múltiples ramas
2. Identifica ramas obsoletas con `git branch --merged`
3. Elimina ramas locales fusionadas
4. Actualiza .gitignore para tu stack tecnológico
5. Ejecuta `git gc --aggressive`
6. Documenta el proceso y ahorro de espacio

### Ejercicio 2: Investigación con git blame y bisect
1. Encuentra un bug en un proyecto existente
2. Usa `git blame` para identificar cuando se introdujo
3. Usa `git bisect` para encontrar el commit exacto
4. Analiza qué cambió en ese commit
5. Propón una solución

### Ejercicio 3: Análisis de contribuciones
1. Genera reporte de actividad del último mes
2. Identifica los archivos más modificados
3. Encuentra patrones de trabajo del equipo
4. Crea visualización de métricas con script
5. Presenta insights al equipo

### Ejercicio 4: Optimización de performance
1. Mide tiempo de `git status` en repo grande
2. Ejecuta `git gc --aggressive`
3. Configura opciones de performance
4. Vuelve a medir y compara resultados
5. Documenta mejoras obtenidas

## 🎯 Mejores Prácticas de Mantenimiento

### Rutinas regulares
- **Diaria**: Review de PRs pendientes, merge ramas completadas
- **Semanal**: Limpieza de ramas locales, actualización de dependencies
- **Mensual**: Análisis de métricas, optimización de performance
- **Trimestral**: Auditoría de seguridad, limpieza profunda de historial

### Automatización recomendada
- **CI/CD**: Auto-eliminación de ramas después de merge
- **Bots**: Notificaciones de ramas obsoletas
- **Scripts**: Limpieza regular automatizada
- **Monitoring**: Alertas de repositorios grandes o lentos

### Documentación de procesos
```markdown
## Runbook: Mantenimiento de Repositorio

### Checklist Semanal
- [ ] Actualizar main: `git checkout main && git pull`
- [ ] Limpiar ramas: ejecutar `cleanup-repo.sh`
- [ ] Revisar .gitignore: agregar patrones nuevos
- [ ] Verificar tamaño: `git count-objects -v`
- [ ] Generar métricas: ejecutar `git-dashboard.py`

### Escalation
- **Repo > 1GB**: Investigar archivos grandes
- **Performance lenta**: Ejecutar `git gc --aggressive`  
- **Muchas ramas**: Comunicar política de limpieza
- **Historial complejo**: Considerar squash de features antiguas
```

### Herramientas de monitoreo
- **GitHub Insights**: Pulse, contributors, traffic
- **git-sizer**: Análisis de tamaño de repositorio
- **git-standup**: ¿Qué trabajé ayer?
- **git-extras**: Suite de herramientas adicionales

## 🔧 Troubleshooting Avanzado

### Problemas comunes y soluciones
```bash
# Repositorio corrupto
git fsck --full --unreachable
git gc --aggressive --prune=now

# Performance muy lenta
git config --global core.preloadindex true
git config --global core.fscache true

# Historial muy complejo
git log --graph --pretty=oneline --abbrev-commit | head -50

# Objetos huérfanos
git reflog expire --expire=now --all
git gc --prune=now --aggressive
```

### Recovery scenarios
```bash
# Recuperar branch eliminado accidentalmente
git reflog  # Encontrar hash del último commit
git checkout -b recovered-branch <hash>

# Recuperar de disco lleno durante operación
git reset --hard HEAD
git clean -fd
git gc

# Reparar repositorio después de crash
git fsck --full
# Si hay objetos corruptos, re-clone es más seguro
```

---
**Anterior**: [07. Colaboración Avanzada y Gestión de Proyectos](./07-colaboracion.md) | **🏠 Inicio**: [README del Curso](./README.md)