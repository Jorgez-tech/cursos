# 06. Documentación y Mantenimiento

## 📖 Documentar estrategias de testing

### README para el proyecto de testing
```markdown
# Testing Strategy - [Proyecto Name]

## 🎯 Objetivos de Testing
- Detectar errores antes de producción
- Garantizar funcionalidad según especificaciones
- Mantener cobertura mínima del 80%
- Automatizar validaciones en CI/CD

## 📊 Métricas actuales
- **Cobertura de código**: 85%
- **Tests unitarios**: 340 tests
- **Tests de integración**: 45 tests  
- **Tests E2E**: 25 scenarios
- **Tiempo de ejecución total**: 8 minutos

## 🏗️ Arquitectura de Testing

### Estrategia por capas
```
🔺 E2E (5%)     - Flujos críticos de usuario
🔺🔺 Integration (20%) - APIs y servicios
🔺🔺🔺 Unit (75%)      - Lógica de negocio
```

### Herramientas utilizadas
| Tipo          | Herramienta | Propósito                        |
|---------------|-------------|----------------------------------|
| Unit          | Jest        | Testing de funciones y clases    |
| Integration   | Supertest   | Testing de APIs                  |
| E2E           | Cypress     | Flujos de usuario                |
| Mocking       | MSW         | Simulación de APIs externas      |
| Coverage      | Istanbul    | Medición de cobertura            |

## 🚀 Ejecutar tests

```bash
# Todos los tests
npm run test:all

# Por tipo
npm run test:unit
npm run test:integration  
npm run test:e2e

# Con cobertura
npm run test:coverage

# Modo watch para desarrollo
npm run test:watch

# Solo tests relacionados con cambios
npm run test:changed
```

## 📋 Checklist de calidad
- [ ] Cobertura > 80%
- [ ] Todos los tests pasan
- [ ] Tests E2E críticos funcionan
- [ ] Performance tests estables
- [ ] Security scans limpios
```

### Documentación de casos de test
```markdown
# Test Cases Documentation

## TC001: User Registration
**Objetivo**: Verificar que un usuario puede registrarse exitosamente

**Precondiciones**:
- Base de datos limpia
- Email no existe en sistema

**Pasos**:
1. Navegar a /register
2. Completar formulario con datos válidos
3. Hacer clic en "Register"

**Resultado esperado**:
- Usuario creado en base de datos
- Email de confirmación enviado
- Redirección a página de bienvenida

**Criterios de aceptación**:
- [ ] Email es validado (formato correcto)
- [ ] Password cumple política de seguridad
- [ ] Confirmación de email es requerida
- [ ] No permite emails duplicados

**Datos de prueba**:
```json
{
  "valid_user": {
    "email": "test@example.com",
    "password": "SecurePass123!",
    "firstName": "John",
    "lastName": "Doe"
  },
  "invalid_email": {
    "email": "invalid-email",
    "password": "SecurePass123!"
  }
}
```
```

## 🔄 Mantenimiento de test suites

### Estrategia de refactoring
```javascript
// scripts/test-maintenance.js
const fs = require('fs');
const glob = require('glob');

class TestMaintenanceAnalyzer {
  async analyzeTestSuite() {
    const testFiles = glob.sync('tests/**/*.test.js');
    const analysis = {
      slowTests: [],
      duplicatedLogic: [],
      flappyTests: [],
      outdatedTests: []
    };

    for (const file of testFiles) {
      const content = fs.readFileSync(file, 'utf8');
      
      // Detectar tests lentos
      if (content.includes('timeout(') && content.includes('10000')) {
        analysis.slowTests.push(file);
      }

      // Detectar lógica duplicada
      const setupBlocks = content.match(/beforeEach\([\s\S]*?\)/g) || [];
      if (setupBlocks.length > 3) {
        analysis.duplicatedLogic.push(file);
      }

      // Detectar tests que saltan frecuentemente
      if (content.includes('.skip') || content.includes('xit(')) {
        analysis.flappyTests.push(file);
      }
    }

    return analysis;
  }

  generateMaintenanceReport(analysis) {
    const report = `
# Test Maintenance Report

## 🐌 Tests Lentos (${analysis.slowTests.length})
${analysis.slowTests.map(f => `- ${f}`).join('\n')}

**Acción recomendada**: Optimizar o mover a test de integración

## 🔄 Lógica Duplicada (${analysis.duplicatedLogic.length})
${analysis.duplicatedLogic.map(f => `- ${f}`).join('\n')}

**Acción recomendada**: Extraer a test helpers

## ⚠️ Tests Inestables (${analysis.flappyTests.length})
${analysis.flappyTests.map(f => `- ${f}`).join('\n')}

**Acción recomendada**: Revisar y corregir inmediatamente
    `;

    fs.writeFileSync('test-maintenance-report.md', report);
  }
}

// Uso
const analyzer = new TestMaintenanceAnalyzer();
analyzer.analyzeTestSuite()
  .then(analysis => analyzer.generateMaintenanceReport(analysis));
```

### Automatización de limpieza
```yaml
# .github/workflows/test-maintenance.yml
name: Test Maintenance

on:
  schedule:
    - cron: '0 6 * * 1'  # Lunes a las 6 AM

jobs:
  analyze-tests:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '18'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run test analysis
      run: node scripts/test-maintenance.js
    
    - name: Create maintenance PR
      uses: peter-evans/create-pull-request@v5
      with:
        title: '🧹 Test Maintenance Report'
        body: |
          Reporte automático de mantenimiento de tests.
          Ver archivo `test-maintenance-report.md` para detalles.
        branch: test-maintenance
        delete-branch: true
```

## 📈 Escalabilidad de testing

### Paralelización de tests
```javascript
// jest.config.js para proyectos grandes
module.exports = {
  // Ejecutar tests en paralelo
  maxWorkers: '50%',
  
  // Configuración por proyecto
  projects: [
    {
      displayName: 'unit',
      testMatch: ['<rootDir>/tests/unit/**/*.test.js'],
      setupFilesAfterEnv: ['<rootDir>/tests/setup/unit.js']
    },
    {
      displayName: 'integration',
      testMatch: ['<rootDir>/tests/integration/**/*.test.js'],
      setupFilesAfterEnv: ['<rootDir>/tests/setup/integration.js'],
      // Tests de integración en serie para evitar conflictos
      runner: '@jest/runner-serial'
    }
  ],
  
  // Cache para mejorar velocidad
  cache: true,
  cacheDirectory: '<rootDir>/.jest-cache',
  
  // Solo ejecutar tests afectados por cambios
  changedFilesWithAncestor: true
};
```

### Sharding para E2E tests
```javascript
// cypress.config.js con sharding
import { defineConfig } from 'cypress'

export default defineConfig({
  e2e: {
    setupNodeEvents(on, config) {
      // Plugin para sharding
      const { cloudPlugin } = require('cypress-cloud/plugin')
      return cloudPlugin(on, config)
    },
  },
  // Configuración para CI paralelo
  env: {
    currents_projectId: 'your-project-id',
    currents_recordKey: 'your-record-key'
  }
})
```

### Test data management
```javascript
// tests/helpers/test-data-factory.js
class TestDataFactory {
  static createUser(overrides = {}) {
    return {
      id: this.generateId(),
      email: this.generateEmail(),
      firstName: 'Test',
      lastName: 'User',
      createdAt: new Date(),
      ...overrides
    };
  }

  static createProduct(overrides = {}) {
    return {
      id: this.generateId(),
      name: `Product ${this.generateId()}`,
      price: Math.random() * 100,
      stock: Math.floor(Math.random() * 50),
      ...overrides
    };
  }

  static createOrder(user, products = [], overrides = {}) {
    return {
      id: this.generateId(),
      userId: user.id,
      items: products.map(p => ({
        productId: p.id,
        quantity: 1,
        price: p.price
      })),
      total: products.reduce((sum, p) => sum + p.price, 0),
      status: 'pending',
      ...overrides
    };
  }

  static generateId() {
    return Math.random().toString(36).substr(2, 9);
  }

  static generateEmail() {
    return `test-${this.generateId()}@example.com`;
  }
}

module.exports = TestDataFactory;
```

## 🔧 Herramientas de optimización

### Test impact analysis
```javascript
// scripts/test-impact-analysis.js
const { execSync } = require('child_process');
const fs = require('fs');

class TestImpactAnalyzer {
  getChangedFiles() {
    const output = execSync('git diff --name-only HEAD~1 HEAD', { encoding: 'utf8' });
    return output.trim().split('\n').filter(Boolean);
  }

  findRelatedTests(changedFiles) {
    const relatedTests = new Set();

    changedFiles.forEach(file => {
      // Tests directos
      const directTest = file.replace('src/', 'tests/').replace('.js', '.test.js');
      if (fs.existsSync(directTest)) {
        relatedTests.add(directTest);
      }

      // Tests que importan este archivo
      const grepOutput = execSync(
        `grep -r "from.*${file}" tests/ || true`, 
        { encoding: 'utf8' }
      );
      
      grepOutput.split('\n').forEach(line => {
        const match = line.match(/^([^:]+):/);
        if (match) {
          relatedTests.add(match[1]);
        }
      });
    });

    return Array.from(relatedTests);
  }

  generateTestCommand(relatedTests) {
    if (relatedTests.length === 0) {
      return 'npm run test:quick'; // Suite mínima
    }

    const testPattern = relatedTests.join('|');
    return `npm test -- --testPathPattern="${testPattern}"`;
  }
}

// Uso en CI
const analyzer = new TestImpactAnalyzer();
const changedFiles = analyzer.getChangedFiles();
const relatedTests = analyzer.findRelatedTests(changedFiles);
const command = analyzer.generateTestCommand(relatedTests);

console.log(`Running: ${command}`);
execSync(command, { stdio: 'inherit' });
```

### Performance monitoring
```javascript
// tests/helpers/performance-monitor.js
class TestPerformanceMonitor {
  constructor() {
    this.metrics = {};
  }

  startTimer(testName) {
    this.metrics[testName] = {
      start: process.hrtime.bigint(),
      memory: process.memoryUsage()
    };
  }

  endTimer(testName) {
    if (!this.metrics[testName]) return;

    const end = process.hrtime.bigint();
    const duration = Number(end - this.metrics[testName].start) / 1000000; // ms

    this.metrics[testName].duration = duration;
    this.metrics[testName].endMemory = process.memoryUsage();
  }

  generateReport() {
    const slowTests = Object.entries(this.metrics)
      .filter(([_, data]) => data.duration > 1000) // > 1 segundo
      .sort(([,a], [,b]) => b.duration - a.duration);

    const report = {
      totalTests: Object.keys(this.metrics).length,
      slowTests: slowTests.length,
      slowestTests: slowTests.slice(0, 10),
      averageDuration: Object.values(this.metrics)
        .reduce((sum, data) => sum + data.duration, 0) / Object.keys(this.metrics).length
    };

    fs.writeFileSync('test-performance-report.json', JSON.stringify(report, null, 2));
    return report;
  }
}

module.exports = TestPerformanceMonitor;
```

## 📋 Documentación como código

### ADR (Architecture Decision Records) para testing
```markdown
# ADR-001: Adopción de Cypress para E2E Testing

## Estado
Aceptado

## Contexto
Necesitamos una herramienta confiable para testing E2E que:
- Sea fácil de usar para el equipo
- Tenga buen debugging
- Integre bien con nuestro stack (React + Node.js)

## Decisión
Adoptar Cypress como herramienta principal para E2E testing.

## Consecuencias

### Positivas
- Developer experience excelente
- Time-travel debugging
- Real-time reloading
- Screenshots y videos automáticos

### Negativas
- Solo funciona en navegadores Chromium
- Limitaciones con múltiples tabs
- Tests pueden ser más lentos que Playwright

## Fecha
2025-08-24

## Revisión programada
2026-02-24
```

---

**Anterior**: [05. Integración](./05-integracion.md) | **Siguiente**: [07. Evaluación](./07-evaluacion.md)
