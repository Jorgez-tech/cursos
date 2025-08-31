# 05. Integración CI/CD

## 🔄 GitHub Actions para testing automatizado

### Workflow completo de testing
```yaml
# .github/workflows/test.yml
name: Test Suite

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  NODE_VERSION: '18'
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # 1. Unit y Integration Tests
  test-backend:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: testdb
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432
      
      redis:
        image: redis:7
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 6379:6379

    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: ${{ env.NODE_VERSION }}
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run unit tests
      run: npm run test:unit
      env:
        DATABASE_URL: postgres://postgres:postgres@localhost:5432/testdb
        REDIS_URL: redis://localhost:6379
    
    - name: Run integration tests
      run: npm run test:integration
      env:
        DATABASE_URL: postgres://postgres:postgres@localhost:5432/testdb
        REDIS_URL: redis://localhost:6379
    
    - name: Generate coverage report
      run: npm run test:coverage
    
    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
      with:
        token: ${{ secrets.CODECOV_TOKEN }}
        files: ./coverage/lcov.info

  # 2. E2E Tests
  test-e2e:
    runs-on: ubuntu-latest
    needs: test-backend
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: ${{ env.NODE_VERSION }}
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Build application
      run: npm run build
    
    - name: Start application with Docker
      run: |
        docker-compose -f docker-compose.test.yml up -d
        ./scripts/wait-for-services.sh
    
    - name: Run Cypress E2E tests
      uses: cypress-io/github-action@v6
      with:
        wait-on: 'http://localhost:3000'
        wait-on-timeout: 120
        record: true
        parallel: true
      env:
        CYPRESS_RECORD_KEY: ${{ secrets.CYPRESS_RECORD_KEY }}
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    
    - name: Upload E2E artifacts
      uses: actions/upload-artifact@v4
      if: failure()
      with:
        name: cypress-screenshots
        path: cypress/screenshots
    
    - name: Stop application
      if: always()
      run: docker-compose -f docker-compose.test.yml down -v

  # 3. Security Testing
  security-scan:
    runs-on: ubuntu-latest
    needs: test-backend
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Run npm audit
      run: npm audit --audit-level high
    
    - name: Run Snyk security scan
      uses: snyk/actions/node@master
      env:
        SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
      with:
        args: --severity-threshold=medium
    
    - name: OWASP ZAP Baseline Scan
      uses: zaproxy/action-baseline@v0.7.0
      with:
        target: 'http://localhost:3000'

  # 4. Performance Testing
  performance-test:
    runs-on: ubuntu-latest
    needs: test-backend
    if: github.ref == 'refs/heads/main'
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: ${{ env.NODE_VERSION }}
    
    - name: Install k6
      run: |
        sudo gpg -k
        sudo gpg --no-default-keyring --keyring /usr/share/keyrings/k6-archive-keyring.gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys C5AD17C747E3415A3642D57D77C6C491D6AC1D69
        echo "deb [signed-by=/usr/share/keyrings/k6-archive-keyring.gpg] https://dl.k6.io/deb stable main" | sudo tee /etc/apt/sources.list.d/k6.list
        sudo apt-get update
        sudo apt-get install k6
    
    - name: Start application
      run: |
        docker-compose -f docker-compose.test.yml up -d
        ./scripts/wait-for-services.sh
    
    - name: Run performance tests
      run: k6 run tests/performance/load-test.js
    
    - name: Upload performance results
      uses: actions/upload-artifact@v4
      with:
        name: performance-results
        path: performance-results.json

  # 5. Build y Deploy (solo en main)
  build-and-deploy:
    runs-on: ubuntu-latest
    needs: [test-backend, test-e2e, security-scan]
    if: github.ref == 'refs/heads/main'
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Log in to Container Registry
      uses: docker/login-action@v3
      with:
        registry: ${{ env.REGISTRY }}
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}
    
    - name: Build and push Docker image
      uses: docker/build-push-action@v5
      with:
        context: .
        push: true
        tags: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest
    
    - name: Deploy to staging
      run: |
        echo "Deploying to staging environment..."
        # Aquí va tu lógica de deploy
```

## 🧪 Testing en diferentes entornos

### Environment-specific configurations
```yaml
# .github/workflows/test-environments.yml
name: Multi-Environment Testing

on:
  schedule:
    - cron: '0 2 * * *'  # Daily at 2 AM
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to test'
        required: true
        default: 'staging'
        type: choice
        options:
        - staging
        - production

jobs:
  test-environment:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        environment: [staging, production]
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '18'
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run smoke tests
      run: npm run test:smoke
      env:
        TEST_ENV: ${{ matrix.environment }}
        BASE_URL: ${{ secrets[format('{0}_BASE_URL', matrix.environment)] }}
        API_KEY: ${{ secrets[format('{0}_API_KEY', matrix.environment)] }}
    
    - name: Run health checks
      run: npm run test:health
      env:
        TEST_ENV: ${{ matrix.environment }}
        BASE_URL: ${{ secrets[format('{0}_BASE_URL', matrix.environment)] }}
    
    - name: Notify on failure
      if: failure()
      uses: 8398a7/action-slack@v3
      with:
        status: failure
        text: "❌ Tests failed in ${{ matrix.environment }} environment"
      env:
        SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
```

## 📊 Reportes y métricas

### Allure Reports integration
```yaml
# En el job de testing
- name: Generate Allure Report
  run: |
    npm install -g allure-commandline
    allure generate allure-results --clean -o allure-report

- name: Deploy Allure Report to GitHub Pages
  uses: peaceiris/actions-gh-pages@v3
  if: github.ref == 'refs/heads/main'
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}
    publish_dir: ./allure-report
    destination_dir: test-reports
```

### Test configuration para Allure
```javascript
// jest.config.js con Allure
module.exports = {
  testEnvironment: 'node',
  setupFilesAfterEnv: ['<rootDir>/tests/setup/jest.setup.js'],
  reporters: [
    'default',
    ['jest-allure-reporter', {
      resultsDir: 'allure-results',
      categories: [
        {
          name: 'Critical Features',
          matchedStatuses: ['failed'],
          messageRegex: '.*critical.*'
        }
      ]
    }]
  ],
  collectCoverageFrom: [
    'src/**/*.{js,ts}',
    '!src/**/*.d.ts',
    '!src/config/**',
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    }
  }
};
```

## 🔒 Seguridad en CI/CD

### Manejo seguro de secrets
```yaml
# .github/workflows/secure-testing.yml
name: Secure Testing Pipeline

on:
  push:
    branches: [ main ]

jobs:
  test-with-secrets:
    runs-on: ubuntu-latest
    environment: testing  # GitHub Environment con protecciones
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: us-east-1
    
    - name: Get secrets from AWS Secrets Manager
      run: |
        DB_PASSWORD=$(aws secretsmanager get-secret-value \
          --secret-id test-db-password \
          --query SecretString --output text)
        echo "::add-mask::$DB_PASSWORD"
        echo "DB_PASSWORD=$DB_PASSWORD" >> $GITHUB_ENV
    
    - name: Run tests with secrets
      run: npm test
      env:
        DATABASE_URL: postgres://user:${{ env.DB_PASSWORD }}@localhost:5432/testdb
        API_KEY: ${{ secrets.TEST_API_KEY }}
        JWT_SECRET: ${{ secrets.JWT_SECRET }}
```

### SAST (Static Application Security Testing)
```yaml
# Integración con CodeQL
- name: Initialize CodeQL
  uses: github/codeql-action/init@v2
  with:
    languages: javascript

- name: Run CodeQL Analysis
  uses: github/codeql-action/analyze@v2
```

## 🐳 Testing con Docker

### Multi-stage Dockerfile para testing
```dockerfile
# Dockerfile.test
FROM node:18-alpine AS base
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM base AS test
RUN npm ci  # Incluye dev dependencies
COPY . .
RUN npm run test:unit
RUN npm run test:integration

FROM base AS production
COPY --from=test /app/dist ./dist
COPY --from=test /app/public ./public
EXPOSE 3000
CMD ["npm", "start"]
```

### Docker Compose para testing completo
```yaml
# docker-compose.ci.yml
version: '3.8'
services:
  app-test:
    build:
      context: .
      dockerfile: Dockerfile.test
      target: test
    environment:
      - NODE_ENV=test
      - DATABASE_URL=postgres://test:test@db:5432/testdb
      - REDIS_URL=redis://redis:6379
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy
    volumes:
      - ./tests:/app/tests
      - ./coverage:/app/coverage
    command: npm run test:ci

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: testdb
      POSTGRES_USER: test
      POSTGRES_PASSWORD: test
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U test"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  e2e-tests:
    build:
      context: .
      dockerfile: Dockerfile.test
    environment:
      - CYPRESS_baseUrl=http://app:3000
    depends_on:
      - app
    volumes:
      - ./tests/e2e:/app/tests/e2e
      - ./cypress/screenshots:/app/cypress/screenshots
      - ./cypress/videos:/app/cypress/videos
    command: npm run test:e2e:ci
```

## ✅ Quality Gates

### SonarQube integration
```yaml
- name: SonarQube Scan
  uses: sonarqube-quality-gate-action@master
  env:
    SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
  with:
    scanMetadataReportFile: target/sonar/report-task.txt

- name: Quality Gate check
  run: |
    if [ "${{ steps.sonarqube-quality-gate-check.outputs.quality-gate-status }}" != "PASSED" ]; then
      echo "Quality gate failed"
      exit 1
    fi
```

### Custom quality checks
```javascript
// scripts/quality-gate.js
const fs = require('fs');
const path = require('path');

async function checkQualityGate() {
  // 1. Check test coverage
  const coverage = JSON.parse(
    fs.readFileSync(path.join(__dirname, '../coverage/coverage-summary.json'))
  );
  
  const minCoverage = 80;
  const actualCoverage = coverage.total.lines.pct;
  
  if (actualCoverage < minCoverage) {
    console.error(`❌ Coverage ${actualCoverage}% is below threshold ${minCoverage}%`);
    process.exit(1);
  }

  // 2. Check for TODO/FIXME comments
  const { execSync } = require('child_process');
  const todoCount = execSync('grep -r "TODO\\|FIXME" src/ | wc -l', { encoding: 'utf8' });
  
  if (parseInt(todoCount) > 10) {
    console.error(`❌ Too many TODO/FIXME comments: ${todoCount}`);
    process.exit(1);
  }

  // 3. Check bundle size
  const bundleStats = JSON.parse(
    fs.readFileSync(path.join(__dirname, '../dist/bundle-stats.json'))
  );
  
  const maxBundleSize = 1024 * 1024; // 1MB
  if (bundleStats.size > maxBundleSize) {
    console.error(`❌ Bundle size ${bundleStats.size} exceeds limit`);
    process.exit(1);
  }

  console.log('✅ All quality gates passed');
}

checkQualityGate().catch(console.error);
```

---

**Anterior**: [04. Práctica](./04-practica.md) | **Siguiente**: [06. Documentación](./06-documentacion.md)
