# 05. CI/CD Moderno: Integración y Despliegue Continuo

## ⚡ Introducción a CI/CD

La Integración Continua (CI) y el Despliegue Continuo (CD) son prácticas fundamentales del desarrollo moderno que automatizan la integración, validación y entrega de código, permitiendo equipos más ágiles y software de mayor calidad.

### Historia y evolución

- **2000-2005**: Build servers manuales, procesos ad-hoc
- **2005-2010**: Jenkins, Hudson - Primera generación de CI
- **2010-2015**: Travis CI, CircleCI - CI como servicio
- **2015-2020**: GitLab CI, GitHub Actions - Integración nativa
- **Actualidad**: GitOps, Infrastructure as Code, pipelines declarativos

### Diferencia entre CI, CD y CD
- **CI (Continuous Integration)**: Integrar cambios frecuentemente con validación automática
- **CD (Continuous Delivery)**: Entregar software listo para producción en cualquier momento
- **CD (Continuous Deployment)**: Desplegar automáticamente cada cambio que pase validaciones

## 🏗️ Componentes de un Pipeline CI/CD

### Fases del Pipeline
1. **Source**: Control de versiones (Git)
2. **Build**: Compilación y empaquetado
3. **Test**: Pruebas automatizadas (unit, integration, e2e)
4. **Security**: Análisis de vulnerabilidades
5. **Package**: Creación de artefactos deployables
6. **Deploy**: Despliegue a diferentes entornos
7. **Monitor**: Observabilidad y alertas

### Tipos de Testing en CI/CD
- **Unit Tests**: Validación de componentes individuales
- **Integration Tests**: Interacción entre módulos
- **Contract Tests**: APIs y interfaces
- **E2E Tests**: Flujos completos de usuario
- **Performance Tests**: Carga y rendimiento
- **Security Tests**: Vulnerabilidades y compliance

## ⚡ Implementación con GitHub Actions

### Pipeline completo para aplicación Node.js
```yaml
# .github/workflows/cicd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
  
env:
  NODE_VERSION: '18'
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # Job de Continuous Integration
  ci:
    name: Continuous Integration
    runs-on: ubuntu-latest
    
    outputs:
      version: ${{ steps.version.outputs.version }}
      
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Para semantic versioning
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Code quality checks
        run: |
          npm run lint
          npm run format:check
          npm run type-check
      
      - name: Unit tests
        run: npm run test:unit -- --coverage
      
      - name: Upload coverage reports
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
          flags: unittests
          
      - name: Integration tests
        run: npm run test:integration
        env:
          DATABASE_URL: postgres://test:test@localhost:5432/testdb
      
      - name: Security audit
        run: npm audit --audit-level=high
      
      - name: Build application
        run: npm run build
      
      - name: Archive build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: build-artifacts
          path: |
            dist/
            package.json
            package-lock.json
          retention-days: 7
      
      - name: Generate version
        id: version
        run: |
          if [[ "${{ github.ref }}" == "refs/heads/main" ]]; then
            VERSION=$(npm version patch --no-git-tag-version --no-commit-hooks)
            echo "version=$VERSION" >> $GITHUB_OUTPUT
          else
            VERSION="develop-$(git rev-parse --short HEAD)"
            echo "version=$VERSION" >> $GITHUB_OUTPUT
          fi

  # Job de Security Scanning
  security:
    name: Security Analysis
    runs-on: ubuntu-latest
    needs: ci
    if: github.event_name != 'pull_request'
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          format: 'sarif'
          output: 'trivy-results.sarif'
      
      - name: Upload Trivy scan results
        uses: github/codeql-action/upload-sarif@v2
        if: always()
        with:
          sarif_file: 'trivy-results.sarif'
      
      - name: OWASP Dependency Check
        uses: dependency-check/Dependency-Check_Action@main
        with:
          project: 'MyProject'
          path: '.'
          format: 'ALL'
          
      - name: Upload OWASP results
        uses: actions/upload-artifact@v4
        with:
          name: dependency-check-report
          path: reports/

  # Build Docker image
  docker:
    name: Build Docker Image
    runs-on: ubuntu-latest
    needs: [ci, security]
    if: github.ref == 'refs/heads/main' || github.ref == 'refs/heads/develop'
    
    permissions:
      contents: read
      packages: write
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Download build artifacts
        uses: actions/download-artifact@v4
        with:
          name: build-artifacts
          path: ./
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Log in to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=semver,pattern={{version}},value=${{ needs.ci.outputs.version }}
            type=sha,prefix={{branch}}-
      
      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          file: ./Dockerfile
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          platforms: linux/amd64,linux/arm64

  # Deploy to Staging
  deploy-staging:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    needs: [ci, docker]
    if: github.ref == 'refs/heads/develop'
    
    environment:
      name: staging
      url: https://staging.myapp.com
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy to staging server
        uses: appleboy/ssh-action@v0.1.6
        with:
          host: ${{ secrets.STAGING_HOST }}
          username: ${{ secrets.STAGING_USER }}
          key: ${{ secrets.STAGING_SSH_KEY }}
          script: |
            cd /opt/myapp
            docker-compose pull
            docker-compose up -d --force-recreate
            docker system prune -f
      
      - name: Run smoke tests
        run: |
          sleep 30  # Wait for deployment
          curl -f https://staging.myapp.com/health
          curl -f https://staging.myapp.com/api/status
      
      - name: Run E2E tests against staging
        run: |
          npm ci
          npm run test:e2e -- --baseURL https://staging.myapp.com

  # Deploy to Production
  deploy-production:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: [ci, docker, deploy-staging]
    if: github.ref == 'refs/heads/main'
    
    environment:
      name: production
      url: https://myapp.com
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Blue/Green Deployment
        run: |
          # Logic for blue/green deployment
          echo "Deploying to production with zero downtime"
      
      - name: Deploy to production
        uses: azure/webapps-deploy@v2
        with:
          app-name: 'myapp-production'
          publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ needs.ci.outputs.version }}
      
      - name: Production smoke tests
        run: |
          sleep 60  # Wait for deployment
          curl -f https://myapp.com/health
          curl -f https://myapp.com/api/status
      
      - name: Create GitHub Release
        uses: softprops/action-gh-release@v1
        with:
          tag_name: ${{ needs.ci.outputs.version }}
          name: Release ${{ needs.ci.outputs.version }}
          generate_release_notes: true
          draft: false
      
      - name: Notify team
        uses: 8398a7/action-slack@v3
        with:
          status: success
          text: "🚀 Production deployment successful: ${{ needs.ci.outputs.version }}"
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}

  # Performance Testing
  performance:
    name: Performance Tests
    runs-on: ubuntu-latest
    needs: deploy-staging
    if: github.event_name != 'pull_request'
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Install k6
        run: |
          sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys C5AD17C747E3415A3642D57D77C6C491D6AC1D69
          echo "deb https://dl.k6.io/deb stable main" | sudo tee /etc/apt/sources.list.d/k6.list
          sudo apt-get update
          sudo apt-get install k6
      
      - name: Run load tests
        run: k6 run --out json=results.json performance/load-test.js
        env:
          BASE_URL: https://staging.myapp.com
      
      - name: Upload performance results
        uses: actions/upload-artifact@v4
        with:
          name: performance-results
          path: results.json
```

### Pipeline para diferentes tecnologías

#### Python con Django
```yaml
# Python CI/CD example
jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:13
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
          cache: 'pip'
      
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install -r requirements-dev.txt
      
      - name: Run tests
        run: |
          python manage.py test
          coverage run --source='.' manage.py test
          coverage report
        env:
          DATABASE_URL: postgres://postgres:postgres@localhost:5432/testdb
      
      - name: Django system check
        run: python manage.py check --deploy
```

## 🔧 Herramientas y Plataformas CI/CD

### Plataformas populares
| Herramienta | Pros | Cons | Mejor para |
|-------------|------|------|------------|
| **GitHub Actions** | Integrado, gratis para público | Limitado en runners | Proyectos en GitHub |
| **GitLab CI** | GitOps nativo, self-hosted | Curva de aprendizaje | Enterprise DevOps |
| **Jenkins** | Flexible, plugins infinitos | Mantenimiento complejo | Legacy systems |
| **CircleCI** | Rápido, configuración simple | Costo en proyectos privados | Startups |
| **Azure DevOps** | Integración Microsoft | Ecosistema cerrado | .NET projects |

### Conceptos avanzados

#### Pipeline as Code
```yaml
# Definir pipeline en el repositorio
# Versionado junto con el código
# Review process para cambios en CI/CD
# Rollback de configuraciones
```

#### GitOps
```yaml
# Repositorio separado para configuraciones
# Argo CD, Flux para Kubernetes
# Declarative deployments
# Git como única fuente de verdad
```

#### Feature Flags en CI/CD
```javascript
// Deployment separado de release
if (featureFlag.isEnabled('NEW_CHECKOUT')) {
  return newCheckoutProcess();
} else {
  return legacyCheckoutProcess();
}
```

## 📝 Ejercicios Prácticos

### Ejercicio 1: Pipeline básico
1. Crea aplicación web simple (Node.js/Python/etc.)
2. Configura pipeline con build, test, deploy
3. Agrega quality gates (coverage, linting)
4. Implementa deployment automático a staging
5. Documenta el proceso completo

### Ejercicio 2: Pipeline multi-ambiente
1. Configura 3 ambientes: dev, staging, production
2. Implementa promotion entre ambientes
3. Agrega approval gates para producción
4. Configura rollback automático en fallos
5. Integra monitoring y alertas

### Ejercicio 3: Security-first pipeline
1. Integra SAST scanning (CodeQL, SonarQube)
2. Agrega dependency vulnerability checking
3. Implementa container image scanning
4. Configura compliance checking
5. Crea security dashboard

## 🎯 Mejores Prácticas

### Diseño de Pipeline
- **Fast feedback**: Tests rápidos primero, lentos después
- **Fail fast**: Fallar temprano para feedback rápido
- **Idempotent**: Mismo resultado en múltiples ejecuciones
- **Immutable artifacts**: Un build, múltiples deploys
- **Traceability**: Links claros entre commits, builds y deploys

### Testing Strategy
- **Test pyramid**: Muchos unit, algunos integration, pocos E2E
- **Shift left**: Testing lo más temprano posible
- **Parallel execution**: Paralelizar para velocidad
- **Test data management**: Datos consistentes y aislados
- **Flaky test management**: Identificar y eliminar tests inestables

### Security e Compliance
- **Secrets management**: Nunca hardcodear credenciales
- **Least privilege**: Permisos mínimos necesarios
- **Audit trail**: Log completo de todas las acciones
- **Compliance as code**: Automatizar verificaciones regulatorias
- **Security scanning**: Integrado en cada stage del pipeline

### Monitoreo y Observabilidad
- **Pipeline metrics**: Tiempo, tasa de éxito, frecuencia
- **Deployment tracking**: Correlación entre deploys y issues
- **Performance monitoring**: Impact de cambios en performance
- **Alert fatigue**: Alertas accionables, no ruido
- **Incident response**: Playbooks automatizados para rollbacks

## 🚀 Tendencias Futuras

### Emerging Practices
- **AI-assisted CI/CD**: Predictive failures, auto-optimization
- **Progressive delivery**: Canary, blue/green automation
- **Policy as Code**: Open Policy Agent (OPA) integration  
- **Serverless CI/CD**: Function-based pipelines
- **Multi-cloud strategies**: Avoid vendor lock-in

### Herramientas emergentes
- **Tekton**: Cloud-native pipeline building blocks
- **Argo Workflows**: Kubernetes-native workflows
- **Dagger**: Portable devkit for CI/CD pipelines
- **Earthly**: Build automation with containers

---

**Siguiente**: [06. Contenedores: Docker y Kubernetes](./06-contenedores.md)
