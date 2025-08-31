# 02. Conceptos Fundamentales

## 🏗️ Pirámide de Testing

```
       🔺 E2E Tests
      ────────────
     🔺🔺 Integration Tests  
    ─────────────────────
   🔺🔺🔺 Unit Tests
  ──────────────────────
```

### Unit Tests (Base)
- **Propósito**: Validar funciones/métodos individuales
- **Velocidad**: Muy rápida (<1s)
- **Cobertura ideal**: 70-80%
- **Herramientas**: Jest, JUnit, xUnit, pytest

```javascript
// Ejemplo: Test unitario en Jest
describe('Calculator', () => {
  test('should add two numbers correctly', () => {
    expect(add(2, 3)).toBe(5);
  });
});
```

### Integration Tests (Medio)
- **Propósito**: Validar interacción entre componentes
- **Velocidad**: Moderada (1-10s)
- **Cobertura ideal**: 15-25%
- **Herramientas**: TestContainers, Postman, Supertest

```javascript
// Ejemplo: Test de integración API
describe('User API', () => {
  test('should create user and store in database', async () => {
    const response = await request(app)
      .post('/users')
      .send({ name: 'John', email: 'john@test.com' });
    
    expect(response.status).toBe(201);
    // Verificar en base de datos
    const user = await User.findOne({ email: 'john@test.com' });
    expect(user).toBeTruthy();
  });
});
```

### E2E Tests (Cima)
- **Propósito**: Validar flujos completos de usuario
- **Velocidad**: Lenta (10s-5min)
- **Cobertura ideal**: 5-15%
- **Herramientas**: Cypress, Playwright, Selenium

```javascript
// Ejemplo: Test E2E con Cypress
describe('E-commerce checkout', () => {
  it('should complete purchase flow', () => {
    cy.visit('/products');
    cy.get('[data-testid="add-to-cart"]').first().click();
    cy.get('[data-testid="checkout-button"]').click();
    cy.get('[data-testid="payment-form"]').within(() => {
      cy.get('#card-number').type('4242424242424242');
      cy.get('#expiry').type('12/25');
      cy.get('#cvc').type('123');
    });
    cy.get('[data-testid="confirm-purchase"]').click();
    cy.contains('Purchase successful').should('be.visible');
  });
});
```

## 🔧 Tipos de Testing por categoría

### Por alcance
| Tipo | Descripción | Ejemplo de herramienta |
|------|-------------|----------------------|
| **Unit** | Función individual | Jest, JUnit |
| **Integration** | Múltiples componentes | TestContainers |
| **System** | Sistema completo | Postman |
| **Acceptance** | Criterios de negocio | Cucumber |

### Por propósito
| Tipo | Descripción | Ejemplo de herramienta |
|------|-------------|----------------------|
| **Functional** | ¿Hace lo que debe? | Selenium |
| **Performance** | ¿Qué tan rápido? | K6, JMeter |
| **Security** | ¿Es seguro? | OWASP ZAP |
| **Usability** | ¿Es fácil de usar? | Manual/A/B testing |

### Por momento de ejecución
| Tipo | Descripción | Cuándo ejecutar |
|------|-------------|----------------|
| **Smoke** | Funcionalidad básica | Cada deploy |
| **Regression** | No romper lo existente | Cada release |
| **Exploratory** | Descubrir problemas | Semanalmente |
| **Load** | Bajo estrés | Pre-producción |

## 🏆 Comparativa de herramientas principales

### Frameworks de Unit Testing

| Herramienta | Lenguaje | Pros | Contras |
|-------------|----------|------|---------|
| **Jest** | JavaScript | Configuración mínima, snapshot testing | Principalmente para JS/TS |
| **JUnit 5** | Java | Maduro, extensible, anotaciones | Curva de aprendizaje |
| **pytest** | Python | Sintaxis simple, fixtures potentes | Debugging complejo |
| **xUnit** | .NET | Integrado con ecosystem .NET | Solo .NET |

### Frameworks de E2E Testing

| Herramienta | Pros | Contras | Mejor para |
|-------------|------|---------|-----------|
| **Cypress** | Developer experience excelente, debugging visual | Solo Chrome-based browsers | SPAs modernas |
| **Playwright** | Multi-browser, paralelo, rápido | Curva de aprendizaje | Testing cross-browser |
| **Selenium** | Maduro, multi-lenguaje, gran comunidad | Lento, frágil, setup complejo | Legacy systems |

## 🎯 Estrategias de selección

### Matriz de decisión
```
Alto impacto, bajo esfuerzo → Implementar primero
Alto impacto, alto esfuerzo → Planificar cuidadosamente  
Bajo impacto, bajo esfuerzo → Implementar si sobra tiempo
Bajo impacto, alto esfuerzo → Evitar
```

### Preguntas clave
1. **¿Cuál es el costo de un bug en producción?**
2. **¿Qué tan frecuentemente se despliega?**
3. **¿Cuál es la experiencia del equipo con testing?**
4. **¿Qué restricciones tecnológicas existen?**

---

**Anterior**: [01. Introducción](./01-introduccion.md) | **Siguiente**: [03. Setup y Configuración](./03-setup.md)
