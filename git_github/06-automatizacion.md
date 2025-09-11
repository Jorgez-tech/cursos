# 06. Automatización y DevOps con GitHub Actions

## 🤖 Introducción a GitHub Actions

GitHub Actions es una plataforma de CI/CD nativa que permite automatizar workflows directamente desde tu repositorio. Transforma GitHub de un simple sistema de control de versiones a una plataforma completa de DevOps.

### ¿Por qué GitHub Actions?
- **Integración nativa**: Directamente en GitHub, sin herramientas externas
- **Eventos ricos**: Triggers en pushes, PRs, issues, schedules, etc.
- **Marketplace**: Miles de actions reutilizables
- **Matrix builds**: Tests en múltiples versiones/OS simultáneamente
- **Gratuito**: Generoso plan gratuito para repositorios públicos
- **Escalable**: De proyectos simples a organizaciones enterprise

### Conceptos fundamentales
- **Workflow**: Proceso automatizado (archivo .yml)
- **Job**: Conjunto de steps que se ejecutan en el mismo runner
- **Step**: Tarea individual (ejecutar comando o usar action)
- **Runner**: Servidor que ejecuta los workflows
- **Action**: Aplicación reutilizable para realizar tareas complejas

## 📝 Estructura de Workflows

### Anatomía de un workflow
```yaml
# .github/workflows/ci.yml
name: Continuous Integration  # Nombre del workflow

# Eventos que activan el workflow
on:
  push:
    branches: [main, develop]
    paths-ignore: ['**.md']
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 2 * * *'  # Daily at 2 AM UTC

# Variables de entorno globales
env:
  NODE_VERSION: '18'
  CACHE_PATHS: |
    ~/.npm
    ~/.cache

# Definición de jobs
jobs:
  # Job de tests
  test:
    name: Run Tests
    runs-on: ubuntu-latest
    timeout-minutes: 10
    
    # Estrategia de matriz para múltiples versiones
    strategy:
      matrix:
        node-version: [16, 18, 20]
        os: [ubuntu-latest, windows-latest, macos-latest]
      fail-fast: false
    
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        
      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Run tests
        run: npm run test:ci
        env:
          CI: true
          
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        if: matrix.node-version == '18' && matrix.os == 'ubuntu-latest'
        with:
          file: ./coverage/lcov.info
          flags: unittests

  # Job de build
  build:
    name: Build Application
    runs-on: ubuntu-latest
    needs: test  # Espera a que termine el job test
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Build application
        run: npm run build
        
      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: build-files
          path: dist/
          retention-days: 7
```

### Triggers avanzados
```yaml
on:
  # Múltiples eventos
  push:
    branches: [main, develop, 'release/**']
    paths:
      - 'src/**'
      - 'package.json'
      - '.github/workflows/**'
    tags:
      - 'v*.*.*'
  
  pull_request:
    types: [opened, synchronize, reopened, ready_for_review]
    branches: [main]
  
  # Eventos de issues
  issues:
    types: [opened, labeled]
  
  # Workflow manual
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to deploy'
        required: true
        default: 'staging'
        type: choice
        options:
          - staging
          - production
      debug:
        description: 'Enable debug mode'
        type: boolean
        default: false
  
  # Scheduled
  schedule:
    - cron: '0 8 * * 1-5'  # Weekdays at 8 AM
    - cron: '0 0 1 * *'    # First day of month
  
  # Repository dispatch (external triggers)
  repository_dispatch:
    types: [deploy-staging, run-tests]
```

## 🧪 Workflows de Testing

### Testing completo con múltiples estrategias
```yaml
# .github/workflows/test-suite.yml
name: Test Suite

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  # Tests unitarios
  unit-tests:
    name: Unit Tests
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
      
      - run: npm ci
      - run: npm run test:unit
      
      - name: Upload coverage reports
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
          flags: unit
  
  # Tests de integración
  integration-tests:
    name: Integration Tests
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: testpass
          POSTGRES_DB: testdb
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
      
      - run: npm ci
      
      - name: Run database migrations
        run: npm run db:migrate
        env:
          DATABASE_URL: postgresql://postgres:testpass@localhost:5432/testdb
      
      - name: Run integration tests
        run: npm run test:integration
        env:
          DATABASE_URL: postgresql://postgres:testpass@localhost:5432/testdb
  
  # Tests E2E con Playwright
  e2e-tests:
    name: E2E Tests
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
      
      - run: npm ci
      
      - name: Install Playwright
        run: npx playwright install --with-deps
      
      - name: Start application
        run: |
          npm run build
          npm run start &
          sleep 10
        
      - name: Run E2E tests
        run: npx playwright test
        
      - name: Upload test results
        uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 30

  # Security tests
  security-tests:
    name: Security Scan
    runs-on: ubuntu-latest
    
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
        with:
          sarif_file: 'trivy-results.sarif'
      
      - name: Audit npm dependencies
        run: npm audit --audit-level=high
```

### Testing en múltiples entornos
```yaml
# Matrix strategy avanzada
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest, macos-latest]
    node-version: [16, 18, 20]
    include:
      # Configuraciones especiales
      - os: ubuntu-latest
        node-version: 18
        coverage: true
      - os: windows-latest
        node-version: 18
        windows-specific: true
    exclude:
      # Excluir combinaciones problemáticas
      - os: macos-latest
        node-version: 16
  
  # No fallar todo si una combinación falla
  fail-fast: false
  
  # Máximo de jobs paralelos
  max-parallel: 4

jobs:
  test-matrix:
    name: Test Node ${{ matrix.node-version }} on ${{ matrix.os }}
    runs-on: ${{ matrix.os }}
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm test
        
      - name: Coverage (Ubuntu + Node 18 only)
        if: matrix.coverage
        run: npm run coverage
```

## 🚀 Deployment Workflows

### Deployment automático por entorno
```yaml
# .github/workflows/deploy.yml
name: Deploy Application

on:
  push:
    branches:
      - main        # Deploy to production
      - develop     # Deploy to staging
  
  # Manual deployment
  workflow_dispatch:
    inputs:
      environment:
        description: 'Deployment environment'
        required: true
        type: choice
        options:
          - staging
          - production
      version:
        description: 'Version tag (optional)'
        required: false

jobs:
  # Job de build común
  build:
    name: Build Application
    runs-on: ubuntu-latest
    
    outputs:
      build-version: ${{ steps.version.outputs.version }}
    
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Para generar version correcta
      
      - name: Generate version
        id: version
        run: |
          if [[ "${{ github.ref }}" == "refs/heads/main" ]]; then
            VERSION=$(git describe --tags --always)
          else
            VERSION="develop-$(git rev-parse --short HEAD)"
          fi
          echo "version=$VERSION" >> $GITHUB_OUTPUT
          echo "Generated version: $VERSION"
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
      
      - run: npm ci
      - run: npm run build
      
      - name: Create deployment package
        run: |
          tar -czf app-${{ steps.version.outputs.version }}.tar.gz \
            dist/ package.json package-lock.json
      
      - name: Upload build artifact
        uses: actions/upload-artifact@v4
        with:
          name: deployment-package
          path: app-${{ steps.version.outputs.version }}.tar.gz
          retention-days: 30

  # Deploy to staging
  deploy-staging:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/develop' || (github.event_name == 'workflow_dispatch' && github.event.inputs.environment == 'staging')
    
    environment:
      name: staging
      url: https://staging.miapp.com
    
    steps:
      - name: Download build artifact
        uses: actions/download-artifact@v4
        with:
          name: deployment-package
      
      - name: Deploy to staging server
        uses: appleboy/ssh-action@v0.1.6
        with:
          host: ${{ secrets.STAGING_HOST }}
          username: ${{ secrets.STAGING_USER }}
          key: ${{ secrets.STAGING_SSH_KEY }}
          script: |
            cd /opt/myapp
            sudo systemctl stop myapp
            tar -xzf ~/app-${{ needs.build.outputs.build-version }}.tar.gz
            sudo systemctl start myapp
            sudo systemctl status myapp
      
      - name: Run smoke tests
        run: |
          curl -f https://staging.miapp.com/health || exit 1
          echo "✅ Staging deployment successful!"

  # Deploy to production
  deploy-production:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/main' || (github.event_name == 'workflow_dispatch' && github.event.inputs.environment == 'production')
    
    environment:
      name: production
      url: https://miapp.com
    
    steps:
      - name: Download build artifact
        uses: actions/download-artifact@v4
        with:
          name: deployment-package
      
      - name: Deploy to production
        uses: azure/webapps-deploy@v2
        with:
          app-name: 'myapp-production'
          publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
          package: app-${{ needs.build.outputs.build-version }}.tar.gz
      
      - name: Create GitHub Release
        if: github.ref == 'refs/heads/main'
        uses: softprops/action-gh-release@v1
        with:
          tag_name: ${{ needs.build.outputs.build-version }}
          name: Release ${{ needs.build.outputs.build-version }}
          generate_release_notes: true
          files: app-${{ needs.build.outputs.build-version }}.tar.gz
```

### Blue/Green Deployment
```yaml
# Blue/Green deployment strategy
deploy-blue-green:
  name: Blue/Green Deploy
  runs-on: ubuntu-latest
  needs: [build, test]
  
  environment:
    name: production
    url: https://miapp.com
  
  steps:
    - name: Download artifact
      uses: actions/download-artifact@v4
      with:
        name: deployment-package
    
    - name: Deploy to Blue environment
      id: deploy-blue
      uses: azure/webapps-deploy@v2
      with:
        app-name: 'myapp-blue'
        publish-profile: ${{ secrets.AZURE_WEBAPP_BLUE_PROFILE }}
        package: deployment-package.zip
    
    - name: Warm up Blue environment
      run: |
        for i in {1..5}; do
          curl -f https://myapp-blue.azurewebsites.net/health
          sleep 10
        done
    
    - name: Run production smoke tests
      run: |
        npm run test:smoke -- --baseURL=https://myapp-blue.azurewebsites.net
    
    - name: Switch traffic to Blue (Green -> Blue)
      uses: azure/cli@v1
      with:
        azcliversion: 2.30.0
        inlineScript: |
          az webapp traffic-routing set \
            --resource-group myapp-rg \
            --name myapp-production \
            --distribution myapp-blue=100
    
    - name: Monitor new deployment
      run: |
        sleep 60  # Wait for traffic switch
        curl -f https://miapp.com/health
        echo "✅ Blue/Green deployment successful!"
```

## 📦 Docker y Containerización

### Build y push de imágenes Docker
```yaml
# .github/workflows/docker.yml
name: Docker Build & Push

on:
  push:
    branches: [main, develop]
    tags: ['v*.*.*']
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  docker:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Log in to Container Registry
        if: github.event_name != 'pull_request'
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
            type=ref,event=pr
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=sha,prefix={{branch}}-
      
      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          file: ./Dockerfile
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          platforms: linux/amd64,linux/arm64
      
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ steps.meta.outputs.version }}
          format: 'sarif'
          output: 'trivy-results.sarif'
      
      - name: Upload Trivy scan results
        uses: github/codeql-action/upload-sarif@v2
        if: always()
        with:
          sarif_file: 'trivy-results.sarif'
```

### Deployment a Kubernetes
```yaml
# Deploy to Kubernetes
deploy-k8s:
  name: Deploy to Kubernetes
  runs-on: ubuntu-latest
  needs: docker
  if: github.ref == 'refs/heads/main'
  
  environment:
    name: production-k8s
    url: https://api.miapp.com
  
  steps:
    - name: Checkout
      uses: actions/checkout@v4
    
    - name: Configure kubectl
      uses: azure/k8s-set-context@v1
      with:
        method: kubeconfig
        kubeconfig: ${{ secrets.KUBE_CONFIG_DATA }}
    
    - name: Update deployment image
      run: |
        kubectl set image deployment/myapp-deployment \
          myapp-container=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
        
        kubectl rollout status deployment/myapp-deployment --timeout=300s
    
    - name: Verify deployment
      run: |
        kubectl get services
        kubectl get pods
        
        # Test service endpoint
        kubectl port-forward svc/myapp-service 8080:80 &
        sleep 10
        curl -f http://localhost:8080/health
```

## 🔄 Workflows Avanzados

### Monorepo con Matrix Conditional
```yaml
# Para repositorios monorepo
name: Monorepo CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  # Detectar cambios
  detect-changes:
    runs-on: ubuntu-latest
    outputs:
      frontend: ${{ steps.changes.outputs.frontend }}
      backend: ${{ steps.changes.outputs.backend }}
      shared: ${{ steps.changes.outputs.shared }}
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Detect changes
        uses: dorny/paths-filter@v2
        id: changes
        with:
          filters: |
            frontend:
              - 'packages/frontend/**'
              - 'packages/shared/**'
            backend:
              - 'packages/backend/**'
              - 'packages/shared/**'
            shared:
              - 'packages/shared/**'
  
  # Test frontend solo si hay cambios
  test-frontend:
    needs: detect-changes
    if: needs.detect-changes.outputs.frontend == 'true'
    runs-on: ubuntu-latest
    
    defaults:
      run:
        working-directory: packages/frontend
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
          cache-dependency-path: packages/frontend/package-lock.json
      
      - run: npm ci
      - run: npm test
      - run: npm run build

  # Test backend solo si hay cambios
  test-backend:
    needs: detect-changes
    if: needs.detect-changes.outputs.backend == 'true'
    runs-on: ubuntu-latest
    
    defaults:
      run:
        working-directory: packages/backend
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v4
        with:
          python-version: '3.11'
          cache: 'pip'
      
      - run: pip install -r requirements.txt
      - run: pytest
      - run: python -m mypy src/
```

### Workflow con Artifacts y Dependencies
```yaml
# Pipeline completo con dependencies
name: Full Pipeline

on:
  push:
    branches: [main]

jobs:
  # Stage 1: Build and Test
  build-and-test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
      
      - run: npm ci
      - run: npm run build
      - run: npm test
      
      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: dist-files
          path: dist/
          retention-days: 1
      
      - name: Upload test results
        uses: actions/upload-artifact@v4
        with:
          name: test-results
          path: coverage/
          retention-days: 7

  # Stage 2: Security Analysis
  security-scan:
    runs-on: ubuntu-latest
    needs: build-and-test
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Download build artifacts
        uses: actions/download-artifact@v4
        with:
          name: dist-files
          path: dist/
      
      - name: Run security scan
        run: |
          # Scan built files
          docker run --rm -v $PWD:/app \
            securecodewarrior/docker-security-scanner /app/dist
      
      - name: Upload SARIF results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: security-results.sarif

  # Stage 3: Deploy
  deploy:
    runs-on: ubuntu-latest
    needs: [build-and-test, security-scan]
    environment: production
    
    steps:
      - name: Download artifacts
        uses: actions/download-artifact@v4
        with:
          name: dist-files
          path: dist/
      
      - name: Deploy to production
        run: |
          # Deploy logic here
          echo "Deploying to production..."
```

## 🏋️ Ejercicios Prácticos

### Ejercicio 1: CI/CD básico
1. Crea repositorio con aplicación Node.js simple
2. Configura workflow que:
   - Instale dependencias
   - Ejecute tests
   - Haga build
   - Suba artifacts
3. Prueba con diferentes branches
4. Agrega badge de status al README

### Ejercicio 2: Testing matrix
1. Configura matriz para múltiples versiones de Node
2. Agrega múltiples sistemas operativos
3. Incluye steps condicionales para coverage
4. Excluye combinaciones problemáticas
5. Optimiza con cache y paralelización

### Ejercicio 3: Docker workflow completo
1. Crea Dockerfile para tu aplicación  
2. Configura workflow para build multi-arch
3. Push a GitHub Container Registry
4. Agrega scanning de vulnerabilidades
5. Implementa deployment a staging

### Ejercicio 4: Deployment con environments
1. Configura environment de staging y production
2. Implementa approval gates para production
3. Agrega smoke tests post-deployment
4. Configura rollback automático en fallos
5. Integra con servicio de monitoring

## 🎯 Mejores Prácticas

### Optimización de performance
- **Cache**: Usa cache para dependencias y builds
- **Parallelization**: Jobs independientes en paralelo
- **Artifacts**: Comparte builds entre jobs
- **Matrix strategy**: Para múltiples configuraciones
- **Self-hosted runners**: Para loads intensivos

### Seguridad en workflows
- **Secrets management**: Nunca hardcodees credenciales
- **Least privilege**: Permisos mínimos necesarios
- **Pin action versions**: Usa SHA commits, no tags
- **Environment protection**: Approval para production
- **SARIF uploads**: Integra security scanning

### Mantenibilidad
- **Naming conventions**: Nombres descriptivos
- **Reusable workflows**: DRY principle
- **Documentation**: Comenta workflows complejos
- **Monitoring**: Alertas en fallos críticos
- **Regular updates**: Mantén actions actualizadas

---
**Anterior**: [05. Seguridad y Buenas Prácticas](./05-seguridad.md) | **Siguiente**: [07. Colaboración Avanzada y Gestión de Proyectos](./07-colaboracion.md)