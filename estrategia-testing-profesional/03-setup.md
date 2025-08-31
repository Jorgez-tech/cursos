# 03. Setup y Configuración

## 🛠️ Entorno replicable con Docker

### Docker Compose para testing
```yaml
# docker-compose.test.yml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=test
      - DATABASE_URL=postgres://test:test@db:5432/testdb
    depends_on:
      - db
      - redis
    
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: testdb
      POSTGRES_USER: test
      POSTGRES_PASSWORD: test
    ports:
      - "5432:5432"
    
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
      
  test-runner:
    build: .
    command: npm run test:e2e
    depends_on:
      - app
    volumes:
      - ./tests:/app/tests
      - ./reports:/app/reports
```

### Script de inicialización
```bash
#!/bin/bash
# setup-test-env.sh

echo "🚀 Configurando entorno de testing..."

# Limpiar contenedores anteriores
docker-compose -f docker-compose.test.yml down -v

# Construir y levantar servicios
docker-compose -f docker-compose.test.yml up -d --build

# Esperar que los servicios estén listos
echo "⏳ Esperando servicios..."
./scripts/wait-for-services.sh

# Ejecutar migraciones
docker-compose -f docker-compose.test.yml exec app npm run db:migrate

# Sembrar datos de prueba
docker-compose -f docker-compose.test.yml exec app npm run db:seed

echo "✅ Entorno listo para testing"
```

## 🔧 Instalación de herramientas esenciales

### 1. Node.js + Jest (JavaScript/TypeScript)
```bash
# Instalación
npm init -y
npm install --save-dev jest @types/jest ts-jest typescript

# Configuración jest.config.js
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  collectCoverageFrom: [
    'src/**/*.{ts,js}',
    '!src/**/*.d.ts',
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

### 2. Cypress para E2E
```bash
# Instalación
npm install --save-dev cypress

# Configuración cypress.config.js
import { defineConfig } from 'cypress'

export default defineConfig({
  e2e: {
    baseUrl: 'http://localhost:3000',
    viewportWidth: 1280,
    viewportHeight: 720,
    video: true,
    screenshotOnRunFailure: true,
    defaultCommandTimeout: 10000,
    requestTimeout: 10000,
    responseTimeout: 10000,
    setupNodeEvents(on, config) {
      // Plugins aquí
    },
  },
})
```

### 3. Playwright (alternativa moderna)
```bash
# Instalación
npm init playwright@latest

# Configuración playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
    { name: 'webkit', use: { ...devices['Desktop Safari'] } },
  ],
  webServer: {
    command: 'npm run start',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

## 🔐 Variables de entorno y secrets

### Archivo .env.test
```bash
# .env.test - NO commitear secrets reales
NODE_ENV=test
DATABASE_URL=postgres://test:test@localhost:5432/testdb
REDIS_URL=redis://localhost:6379
API_KEY=test-api-key-not-real
JWT_SECRET=test-jwt-secret-not-real
EXTERNAL_API_URL=http://localhost:8080/mock-api

# Feature flags para testing
ENABLE_FEATURE_X=true
ENABLE_SLOW_TESTS=false
```

### Manejo seguro de secrets
```javascript
// config/test.js
const config = {
  database: {
    url: process.env.DATABASE_URL || 'postgres://test:test@localhost:5432/testdb',
  },
  api: {
    key: process.env.API_KEY || 'fallback-test-key',
    timeout: parseInt(process.env.API_TIMEOUT) || 5000,
  },
  features: {
    enableSlowTests: process.env.ENABLE_SLOW_TESTS === 'true',
  }
};

module.exports = config;
```

## 📦 Package.json optimizado para testing

```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "test:unit": "jest --testPathPattern=unit",
    "test:integration": "jest --testPathPattern=integration",
    "test:e2e": "cypress run",
    "test:e2e:open": "cypress open",
    "test:all": "npm run test:unit && npm run test:integration && npm run test:e2e",
    "test:ci": "npm run test:coverage && npm run test:e2e -- --record --key $CYPRESS_RECORD_KEY",
    "setup:test": "./scripts/setup-test-env.sh",
    "teardown:test": "docker-compose -f docker-compose.test.yml down -v"
  },
  "jest": {
    "projects": [
      {
        "displayName": "unit",
        "testMatch": ["<rootDir>/tests/unit/**/*.test.js"]
      },
      {
        "displayName": "integration", 
        "testMatch": ["<rootDir>/tests/integration/**/*.test.js"],
        "setupFilesAfterEnv": ["<rootDir>/tests/setup/database.js"]
      }
    ]
  }
}
```

## 🏗️ Estructura de directorios recomendada

```
proyecto/
├── src/                          # Código fuente
├── tests/
│   ├── unit/                     # Tests unitarios
│   │   ├── services/
│   │   ├── utils/
│   │   └── models/
│   ├── integration/              # Tests de integración
│   │   ├── api/
│   │   ├── database/
│   │   └── external-services/
│   ├── e2e/                      # Tests end-to-end
│   │   ├── specs/
│   │   ├── pages/                # Page Object Model
│   │   └── fixtures/
│   ├── setup/                    # Configuración de tests
│   │   ├── database.js
│   │   ├── mocks.js
│   │   └── global-setup.js
│   └── helpers/                  # Utilities para tests
├── scripts/
│   ├── setup-test-env.sh
│   ├── wait-for-services.sh
│   └── generate-test-data.js
├── docker-compose.test.yml
├── .env.test
└── jest.config.js
```

## ✅ Checklist de configuración

- [ ] Docker y Docker Compose instalados
- [ ] Framework de testing configurado (Jest/Cypress/Playwright)
- [ ] Variables de entorno para testing definidas
- [ ] Base de datos de testing separada
- [ ] Scripts de setup/teardown funcionando
- [ ] CI/CD pipeline básico configurado
- [ ] Reportes de cobertura generándose
- [ ] Estructura de directorios organizada

---

**Anterior**: [02. Conceptos](./02-conceptos.md) | **Siguiente**: [04. Práctica Guiada](./04-practica.md)
