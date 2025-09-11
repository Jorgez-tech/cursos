# 02. Repositorios Remotos y GitHub

## 🌐 ¿Qué es un repositorio remoto?
Un repositorio remoto es una copia de tu proyecto alojada en un servidor externo (GitHub, GitLab, Bitbucket) que permite la colaboración entre desarrolladores, el respaldo automático y la sincronización de cambios entre diferentes máquinas.

### Ventajas de los repositorios remotos
- **Colaboración**: Múltiples desarrolladores pueden trabajar en el mismo proyecto
- **Respaldo**: Tu código está seguro en la nube
- **Accesibilidad**: Puedes acceder desde cualquier dispositivo
- **Historial centralizado**: Un solo punto de verdad para el proyecto
- **Integración**: CI/CD, issues, documentación, etc.

## 🔗 Conexión con GitHub

### Crear un repositorio en GitHub
1. Ve a [github.com](https://github.com/) e inicia sesión
2. Haz clic en "New repository" (botón verde)
3. Completa los datos:
   - **Repository name**: nombre-del-proyecto
   - **Description**: Descripción breve del proyecto
   - **Visibility**: Public (público) o Private (privado)
   - **Initialize**: README, .gitignore, License (opcional)

### Conectar tu repositorio local existente
```bash
# Agregar el remoto origin
git remote add origin https://github.com/usuario/mi-proyecto.git

# Verificar la configuración
git remote -v

# Primera subida con tracking
git push -u origin main
```

### Clonar un repositorio existente
```bash
# HTTPS (requiere usuario/token cada vez)
git clone https://github.com/usuario/repositorio.git

# SSH (configuración previa necesaria)
git clone git@github.com:usuario/repositorio.git

# Clonar en carpeta específica
git clone https://github.com/usuario/repo.git mi-proyecto
```

## 🛠️ Comandos Remotos Fundamentales

### Gestión de remotos
```bash
# Ver remotos configurados
git remote -v

# Agregar nuevo remoto
git remote add upstream https://github.com/original/repo.git

# Cambiar URL de remoto
git remote set-url origin https://github.com/nuevo/repo.git

# Eliminar remoto
git remote remove upstream
```

### Sincronización básica
```bash
# Descargar cambios (fetch + merge)
git pull origin main

# Solo descargar sin mezclar
git fetch origin

# Subir cambios locales
git push origin main

# Subir todas las ramas
git push --all origin
```

### Comandos avanzados de sincronización
```bash
# Pull con rebase (historial limpio)
git pull --rebase origin main

# Push forzado (¡CUIDADO!)
git push --force-with-lease origin main

# Ver diferencias con remoto
git diff origin/main

# Ver commits que no están en remoto
git log origin/main..HEAD
```

## 🍴 Forks y Upstream

### ¿Qué es un Fork?
Un **fork** es una copia personal de un repositorio de otro usuario. Te permite:
- Experimentar libremente sin afectar el proyecto original
- Proponer cambios mediante Pull Requests
- Mantener tu propia versión personalizada

### Flujo de trabajo con Forks
```bash
# 1. Fork del repositorio en GitHub (botón Fork)
# 2. Clonar tu fork
git clone https://github.com/tu-usuario/proyecto-forkeado.git
cd proyecto-forkeado

# 3. Agregar upstream (repositorio original)
git remote add upstream https://github.com/propietario-original/proyecto.git

# 4. Verificar remotos
git remote -v
# origin    https://github.com/tu-usuario/proyecto-forkeado.git (fetch)
# origin    https://github.com/tu-usuario/proyecto-forkeado.git (push)
# upstream  https://github.com/propietario-original/proyecto.git (fetch)
# upstream  https://github.com/propietario-original/proyecto.git (push)
```

### Mantener tu Fork actualizado
```bash
# Descargar cambios del upstream
git fetch upstream

# Cambiar a rama principal
git checkout main

# Fusionar cambios de upstream
git merge upstream/main

# Subir cambios actualizados a tu fork
git push origin main
```

## 🔐 Autenticación y Seguridad

### Personal Access Tokens (PAT)
GitHub ya no acepta contraseñas para operaciones Git. Usa tokens:

1. GitHub → Settings → Developer settings → Personal access tokens
2. Generate new token (classic)
3. Selecciona permisos necesarios
4. Copia el token y guárdalo seguro

```bash
# Uso del token
git clone https://token@github.com/usuario/repo.git
# O cuando te lo pida: usuario = tu-username, contraseña = token
```

### Configuración SSH (Recomendado)
```bash
# Generar clave SSH
ssh-keygen -t ed25519 -C "tu-email@ejemplo.com"

# Agregar a SSH agent
ssh-add ~/.ssh/id_ed25519

# Copiar clave pública
cat ~/.ssh/id_ed25519.pub
# Pegar en GitHub → Settings → SSH and GPG keys
```

## 🏋️ Ejercicios Prácticos

### Ejercicio 1: Repositorio desde cero
1. Crea un repositorio local con algunos archivos
2. Crea un repositorio en GitHub
3. Conecta ambos y sube el contenido
4. Haz cambios locales y súbelos
5. Haz cambios en GitHub y bájalos

### Ejercicio 2: Fork y contribución
1. Haz fork de un repositorio público pequeño
2. Clona tu fork localmente
3. Crea una rama para tu contribución
4. Haz cambios y súbelos a tu fork
5. Crea un Pull Request al repositorio original

### Ejercicio 3: Colaboración
1. Invita a un compañero a tu repositorio
2. Que ambos hagan cambios simultáneos
3. Practica resolver conflictos de merge
4. Experimenta con diferentes estrategias de pull

## 🎯 Buenas Prácticas

### Para repositorios personales
- Usa nombres descriptivos y consistentes
- Incluye siempre un README.md claro
- Configura .gitignore apropiado para tu tecnología
- Usa releases/tags para versiones importantes

### Para colaboración
- Mantén tu fork actualizado con upstream
- Usa ramas descriptivas para features
- Haz commits pequeños y frecuentes
- Sincroniza regularmente para evitar conflictos grandes

### Para seguridad
- **Nunca** hagas commit de credenciales o API keys
- Usa SSH keys en lugar de HTTPS cuando sea posible
- Revisa permisos de tokens periódicamente
- Mantén repositorios privados para código sensible

## 🔧 Solución de Problemas Comunes

### Error: "remote origin already exists"
```bash
git remote remove origin
git remote add origin https://github.com/usuario/repo.git
```

### Error: "non-fast-forward" al hacer push
```bash
git pull origin main  # Descargar cambios primero
git push origin main  # Luego subir
```

### Ver historial de un repositorio remoto
```bash
git log origin/main --oneline
git show origin/main:archivo.txt  # Ver archivo específico
```

---
**Anterior**: [01. Introducción](./01-introduccion.md) | **Siguiente**: [03. Ramas y Flujos de Trabajo](./03-ramas.md)