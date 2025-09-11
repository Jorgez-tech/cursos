# 05. Seguridad y Buenas Prácticas

## 🔐 Fundamentos de Seguridad en Git y GitHub

La seguridad en el desarrollo no es opcional. Git y GitHub manejan tu código fuente, credenciales y propiedad intelectual, por lo que implementar prácticas de seguridad robustas desde el inicio es fundamental para proteger tus proyectos y tu equipo.

### Principios de seguridad
- **Defense in depth**: Múltiples capas de seguridad
- **Least privilege**: Solo los permisos necesarios
- **Zero trust**: Verificar siempre, nunca asumir
- **Security by design**: Integrar desde el diseño
- **Continuous monitoring**: Supervisión constante

## 🔑 Autenticación Robusta

### SSH Keys: La Forma Más Segura

#### Generar llaves SSH modernas
```bash
# Ed25519 - Recomendado (más seguro y rápido)
ssh-keygen -t ed25519 -C "tu-email@empresa.com" -f ~/.ssh/id_ed25519_github

# RSA 4096 - Si tu sistema no soporta Ed25519
ssh-keygen -t rsa -b 4096 -C "tu-email@empresa.com" -f ~/.ssh/id_rsa_github

# Configurar passphrase fuerte
# ¡SIEMPRE usa passphrase para llaves SSH!
```

#### Configuración SSH avanzada
```bash
# ~/.ssh/config
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_github
    IdentitiesOnly yes
    AddKeysToAgent yes
    UseKeychain yes  # macOS only

Host github-work
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_work
    IdentitiesOnly yes

# Múltiples cuentas de GitHub
Host github-personal
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_personal
```

#### Gestión segura de SSH Agent
```bash
# Iniciar SSH agent
eval "$(ssh-agent -s)"

# Agregar llave con timeout (8 horas)
ssh-add -t 28800 ~/.ssh/id_ed25519_github

# Verificar llaves cargadas
ssh-add -l

# Probar conexión
ssh -T git@github.com
```

### Personal Access Tokens (PAT)

#### Crear tokens con scope mínimo
```markdown
## Token Scopes Recomendados

### Para desarrollo personal:
- [x] repo (si necesitas repositorios privados)
- [x] public_repo (solo repositorios públicos)
- [x] write:packages (si usas GitHub Packages)

### Para CI/CD:
- [x] repo (solo si es necesario)
- [x] write:packages
- [x] read:packages

### ¡NUNCA marques estos a menos que sea absolutamente necesario!:
- [ ] admin:org
- [ ] delete_repo
- [ ] admin:public_key
```

#### Gestión segura de tokens
```bash
# Usar GitHub CLI para autenticación automática
gh auth login

# Almacenar token en keychain (macOS)
security add-generic-password -a "github-token" -s "github-api" -w "tu_token_aqui"

# Recuperar token (en scripts)
TOKEN=$(security find-generic-password -a "github-token" -s "github-api" -w)
```

### Two-Factor Authentication (2FA)

#### Configuración recomendada
1. **Habilita 2FA** en tu cuenta de GitHub
2. **Usa app authenticator** (Authy, 1Password) no SMS
3. **Descarga recovery codes** y guárdalos seguros
4. **Configura múltiples métodos** (app + hardware key)

```bash
# Para operaciones Git con 2FA habilitado
# Debes usar PAT en lugar de contraseña
git clone https://tu-token@github.com/usuario/repo.git
```

## 🛡️ Protección de Repositorios

### Branch Protection Rules

#### Configuración básica de protección
```markdown
## Branch Protection Settings para 'main'

### General:
- [x] Restrict pushes that create files larger than 100MB
- [x] Restrict pushes to matching files
- [x] Require branches to be up to date before merging

### Branch protection rules:
- [x] Require pull request reviews before merging
  - Required approving reviews: 2
  - [x] Dismiss stale reviews when new commits are pushed
  - [x] Require review from code owners
  - [x] Restrict reviews to users with write access

- [x] Require status checks to pass before merging
  - [x] Require branches to be up to date
  - Required status checks: CI, Security Scan

- [x] Require conversation resolution before merging
- [x] Include administrators
- [x] Restrict pushes that create files larger than 100MB
```

#### Configuración avanzada de protección
```yaml
# .github/branch-protection.yml (usando GitHub API)
protection:
  required_status_checks:
    strict: true
    contexts:
      - "CI Tests"
      - "Security Scan" 
      - "Code Quality"
  enforce_admins: true
  required_pull_request_reviews:
    required_approving_review_count: 2
    dismiss_stale_reviews: true
    require_code_owner_reviews: true
    dismissal_restrictions:
      users: []
      teams: ["core-team"]
  restrictions:
    users: []
    teams: ["core-team"]
    apps: []
```

### CODEOWNERS para revisiones obligatorias
```text
# .github/CODEOWNERS

# Global owners
* @team-leads @security-team

# Frontend code
/frontend/ @frontend-team @ui-lead
*.js @frontend-team
*.ts @frontend-team
*.vue @frontend-team

# Backend API
/api/ @backend-team @architect
*.py @backend-team
*.go @backend-team

# Infrastructure
/infrastructure/ @devops-team @sre-team
/docker/ @devops-team
/.github/ @devops-team @security-team
Dockerfile @devops-team

# Security-sensitive files
/auth/ @security-team @backend-lead
*.key @security-team
*.pem @security-team
secrets.* @security-team

# Database migrations
/migrations/ @backend-team @dba-team @architect

# Documentation
/docs/ @docs-team
*.md @docs-team @product-owner
```

## 🚫 Prevención de Secretos

### .gitignore Inteligente
```gitignore
# === SECURITY-FIRST .gitignore ===

# Credentials and secrets
.env
.env.*
!.env.example
*.key
*.pem
*.p12
*.pfx
*.cert
*.crt
secret*
*secret*
*password*
*token*
credentials*
auth.json
service-account*.json

# API Keys patterns
**/*apikey*
**/*api-key*
**/*api_key*
**/secret*
**/secrets*

# Database
*.db
*.sqlite
*.sqlite3
database.yml
!database.yml.example

# Configuration with secrets
config/production.yml
config/staging.yml
!config/production.yml.example

# Cloud provider configs
.aws/
.gcp/
.azure/

# Container secrets
docker-compose.prod.yml
docker-compose.staging.yml
!docker-compose.prod.yml.example

# IDE secret files
.vscode/settings.json
.idea/workspace.xml

# OS files (potential info leak)
.DS_Store
Thumbs.db
desktop.ini
```

### pre-commit hooks para prevención
```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.4.0
    hooks:
      - id: check-added-large-files
        args: ['--maxkb=500']
      - id: check-yaml
      - id: end-of-file-fixer
      - id: trailing-whitespace

  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.4.0
    hooks:
      - id: detect-secrets
        args: ['--baseline', '.secrets.baseline']
        
  - repo: https://github.com/gitguardian/ggshield
    rev: v1.18.0
    hooks:
      - id: ggshield
        language: python
        stages: [commit]
```

### Scan de secretos en historial
```bash
# Herramientas para detectar secretos
pip install detect-secrets
pip install git-secrets

# Scan inicial del repositorio
detect-secrets scan --all-files --baseline .secrets.baseline

# Auditar baseline
detect-secrets audit .secrets.baseline

# Configurar git-secrets
git secrets --register-aws
git secrets --install
git secrets --scan
```

## 👥 Gestión de Permisos y Roles

### Modelo de permisos en GitHub

#### Niveles de acceso por repositorio
| Nivel | Permisos |
|-------|----------|
| **Read** | Clonar, crear issues/PRs |
| **Triage** | Gestionar issues/PRs, no push |
| **Write** | Push, merge PRs, gestionar issues |
| **Maintain** | Write + manage settings |
| **Admin** | Control total del repositorio |

#### Mejores prácticas de permisos
```markdown
## Matriz de Permisos Recomendada

### Desarrolladores Junior:
- Repositorios de desarrollo: Write
- Repositorio principal: Triage (solo PRs)
- Repositorios críticos: Read

### Desarrolladores Senior:
- Repositorios de desarrollo: Write
- Repositorio principal: Write (con branch protection)
- Repositorios críticos: Write

### Tech Leads:
- Mayoría de repositorios: Maintain
- Repositorios críticos: Admin (limitado)

### DevOps/SRE:
- Repositorios de infraestructura: Admin
- Repositorios de aplicación: Maintain
```

### Teams y organizaciones
```markdown
## Estructura de Teams Recomendada

### Por área técnica:
- @frontend-team
- @backend-team  
- @mobile-team
- @devops-team
- @qa-team

### Por responsabilidad:
- @security-reviewers
- @code-owners
- @release-managers
- @incident-responders

### Por seniority:
- @senior-developers
- @tech-leads
- @architects

### Permisos en cascada:
- Core team: Write en repositorios principales
- Security team: Admin en repositorios críticos
- All developers: Read en documentación
```

## 🔍 Auditoría y Monitoring

### Git hooks para auditoría
```bash
#!/bin/bash
# .git/hooks/pre-push

echo "🔒 Ejecutando verificaciones de seguridad..."

# Verificar que no hay secretos
if command -v detect-secrets >/dev/null; then
    detect-secrets-hook --baseline .secrets.baseline || exit 1
fi

# Verificar tamaño de archivos
git diff --cached --name-only | while read file; do
    if [ -f "$file" ]; then
        size=$(wc -c < "$file" | tr -d ' ')
        if [ $size -gt 5242880 ]; then  # 5MB
            echo "❌ Archivo demasiado grande: $file ($size bytes)"
            exit 1
        fi
    fi
done

# Log de auditoría
echo "$(date): Push by $(git config user.name) <$(git config user.email)>" >> .git/audit.log

echo "✅ Verificaciones completadas"
```

### Monitoring con GitHub Actions
```yaml
# .github/workflows/security-audit.yml
name: Security Audit

on:
  schedule:
    - cron: '0 2 * * *'  # Daily at 2 AM
  push:
    branches: [main]
  pull_request:

jobs:
  security-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Full history for secret scanning
          
      - name: Secret Detection
        uses: GitGuardian/ggshield-action@v1
        env:
          GITHUB_PUSH_BEFORE_SHA: ${{ github.event.before }}
          GITHUB_PUSH_BASE_SHA: ${{ github.event.base }}
          GITHUB_PULL_BASE_SHA: ${{ github.event.pull_request.base.sha }}
          GITHUB_DEFAULT_BRANCH: ${{ github.event.repository.default_branch }}
          GITGUARDIAN_API_KEY: ${{ secrets.GITGUARDIAN_API_KEY }}

      - name: Dependency Vulnerability Scan
        uses: github/super-linter/slim@v4
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          
      - name: SAST Scan
        uses: github/codeql-action/analyze@v2
        with:
          languages: javascript, python
```

### Alertas y notificaciones
```yaml
# .github/workflows/security-alerts.yml
name: Security Alerts

on:
  schedule:
    - cron: '0 9 * * 1'  # Monday mornings

jobs:
  security-report:
    runs-on: ubuntu-latest
    steps:
      - name: Check Dependabot Alerts
        uses: actions/github-script@v6
        with:
          script: |
            const alerts = await github.rest.dependabot.listAlertsForRepo({
              owner: context.repo.owner,
              repo: context.repo.repo,
              state: 'open'
            });
            
            if (alerts.data.length > 0) {
              // Send to Slack, Teams, or email
              console.log(`🚨 ${alerts.data.length} security alerts found!`);
            }
```

## 🏋️ Ejercicios Prácticos

### Ejercicio 1: Configuración SSH completa
1. Genera nueva llave SSH Ed25519 con passphrase fuerte
2. Configura ~/.ssh/config para múltiples identidades
3. Agrega llave a GitHub y prueba conexión
4. Configura SSH agent con timeout
5. Documenta el proceso para tu equipo

### Ejercicio 2: Protección de repositorio crítico
1. Crea repositorio nuevo para proyecto "crítico"
2. Configura branch protection con todos los controles
3. Crea CODEOWNERS file con diferentes áreas
4. Invita colaborador y prueba flujo de PR
5. Verifica que las protecciones funcionan

### Ejercicio 3: Detección y remediación de secretos
1. Instala detect-secrets y git-secrets
2. Crea archivo con "secreto" falso (API key simulada)
3. Configura baseline y pre-commit hooks
4. Intenta commit y verifica que se bloquea
5. Simula limpieza de historial si hay secreto real

### Ejercicio 4: Auditoría de seguridad
1. Revisa permisos actuales en tus repositorios
2. Implementa estructura de teams si usas organización
3. Configura workflow de security scanning
4. Habilita Dependabot alerts
5. Crea proceso de respuesta a incidentes

## 🎯 Mejores Prácticas de Seguridad

### Para desarrolladores individuales
- **Multi-factor authentication**: Siempre habilitado
- **SSH keys**: Usa Ed25519 con passphrase
- **Tokens**: Scope mínimo necesario, rotación regular
- **Secrets**: Nunca en código, usa variables de entorno
- **Updates**: Mantén dependencias actualizadas

### Para equipos y organizaciones
- **Principio de menor privilegio**: Solo permisos necesarios
- **Revisiones obligatorias**: Branch protection en ramas importantes
- **Auditoría continua**: Logs, monitoring y alertas
- **Respuesta a incidentes**: Plan documentado y probado
- **Educación**: Training regular sobre security

### Checklist de seguridad por repositorio
```markdown
## Security Checklist

### Autenticación:
- [ ] 2FA habilitado para todos los collaboradores
- [ ] SSH keys o PATs configurados (no passwords)
- [ ] Tokens con scope mínimo necesario

### Protección de código:
- [ ] Branch protection rules configuradas
- [ ] CODEOWNERS file definido
- [ ] Required status checks habilitados
- [ ] Secrets scanner en pre-commit hooks

### Monitoring:
- [ ] Dependabot alerts habilitado
- [ ] Security scanning en CI/CD
- [ ] Audit logs monitoreados
- [ ] Incident response plan definido

### Compliance:
- [ ] .gitignore incluye patrones de secretos
- [ ] Documentación de seguridad actualizada
- [ ] Training completado por el equipo
```

## 🔧 Herramientas de Seguridad

### Escáners de secretos
- **GitGuardian**: Detección en tiempo real
- **detect-secrets**: Pre-commit hooks
- **git-secrets**: AWS patterns
- **TruffleHog**: Entropia y patrones
- **GitHub Secret Scanning**: Nativo de GitHub

### Análisis de vulnerabilidades
- **Dependabot**: Dependencias vulnerables
- **Snyk**: Análisis integral de vulnerabilidades
- **OWASP Dependency Check**: Scanner OWASP
- **GitHub CodeQL**: SAST nativo
- **SonarQube**: Quality y security gates

### Herramientas de compliance
- **git-blame**: Auditoría de cambios
- **git-log**: Tracking de actividad  
- **GitHub Insights**: Métricas de colaboración
- **Audit logs**: Logs de organización
- **Branch protection**: Compliance automático

---
**Anterior**: [04. Resolución de Conflictos y Comandos Avanzados](./04-conflictos.md) | **Siguiente**: [06. Automatización y DevOps con GitHub Actions](./06-automatizacion.md)