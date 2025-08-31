# 04. Práctica Guiada

## 🎯 Proyecto ejemplo: API de E-commerce

Desarrollaremos una estrategia de testing completa para una API REST de e-commerce con los siguientes endpoints:

- `GET /products` - Listar productos
- `POST /products` - Crear producto  
- `GET /products/:id` - Obtener producto
- `POST /orders` - Crear orden
- `GET /orders/:id` - Obtener orden

## 🧪 Ejercicio 1: Unit Tests

### Servicio a testear
```javascript
// src/services/ProductService.js
class ProductService {
  constructor(database) {
    this.db = database;
  }

  async createProduct(productData) {
    // Validación
    if (!productData.name || !productData.price) {
      throw new Error('Name and price are required');
    }
    
    if (productData.price <= 0) {
      throw new Error('Price must be positive');
    }

    // Calcular precio con descuento
    const finalPrice = this.calculateDiscountedPrice(
      productData.price, 
      productData.discountPercentage || 0
    );

    // Guardar en base de datos
    const product = await this.db.products.create({
      ...productData,
      finalPrice,
      createdAt: new Date()
    });

    return product;
  }

  calculateDiscountedPrice(price, discountPercentage) {
    if (discountPercentage < 0 || discountPercentage > 100) {
      throw new Error('Discount percentage must be between 0 and 100');
    }
    return price * (1 - discountPercentage / 100);
  }
}

module.exports = ProductService;
```

### Unit Tests correspondientes
```javascript
// tests/unit/services/ProductService.test.js
const ProductService = require('../../../src/services/ProductService');

describe('ProductService', () => {
  let productService;
  let mockDatabase;

  beforeEach(() => {
    // Mock de la base de datos
    mockDatabase = {
      products: {
        create: jest.fn()
      }
    };
    productService = new ProductService(mockDatabase);
  });

  describe('createProduct', () => {
    it('should create product with valid data', async () => {
      // Arrange
      const productData = {
        name: 'Laptop',
        price: 1000,
        discountPercentage: 10
      };
      const expectedProduct = { id: 1, ...productData, finalPrice: 900 };
      mockDatabase.products.create.mockResolvedValue(expectedProduct);

      // Act
      const result = await productService.createProduct(productData);

      // Assert
      expect(result).toEqual(expectedProduct);
      expect(mockDatabase.products.create).toHaveBeenCalledWith({
        name: 'Laptop',
        price: 1000,
        discountPercentage: 10,
        finalPrice: 900,
        createdAt: expect.any(Date)
      });
    });

    it('should throw error when name is missing', async () => {
      // Arrange
      const productData = { price: 1000 };

      // Act & Assert
      await expect(productService.createProduct(productData))
        .rejects.toThrow('Name and price are required');
    });

    it('should throw error when price is zero or negative', async () => {
      // Arrange
      const productData = { name: 'Laptop', price: 0 };

      // Act & Assert
      await expect(productService.createProduct(productData))
        .rejects.toThrow('Price must be positive');
    });
  });

  describe('calculateDiscountedPrice', () => {
    it('should calculate discounted price correctly', () => {
      // Test cases con diferentes escenarios
      const testCases = [
        { price: 100, discount: 0, expected: 100 },
        { price: 100, discount: 10, expected: 90 },
        { price: 100, discount: 50, expected: 50 },
        { price: 100, discount: 100, expected: 0 },
      ];

      testCases.forEach(({ price, discount, expected }) => {
        const result = productService.calculateDiscountedPrice(price, discount);
        expect(result).toBe(expected);
      });
    });

    it('should throw error for invalid discount percentage', () => {
      expect(() => productService.calculateDiscountedPrice(100, -5))
        .toThrow('Discount percentage must be between 0 and 100');
      
      expect(() => productService.calculateDiscountedPrice(100, 105))
        .toThrow('Discount percentage must be between 0 and 100');
    });
  });
});
```

## 🔗 Ejercicio 2: Integration Tests

### Test de integración API + Base de datos
```javascript
// tests/integration/api/products.test.js
const request = require('supertest');
const app = require('../../../src/app');
const { setupTestDatabase, cleanupTestDatabase } = require('../../setup/database');

describe('Products API Integration', () => {
  beforeAll(async () => {
    await setupTestDatabase();
  });

  afterAll(async () => {
    await cleanupTestDatabase();
  });

  beforeEach(async () => {
    // Limpiar datos entre tests
    await global.testDb.products.deleteMany({});
  });

  describe('POST /products', () => {
    it('should create product and store in database', async () => {
      // Arrange
      const productData = {
        name: 'Gaming Mouse',
        price: 50,
        category: 'Electronics',
        discountPercentage: 15
      };

      // Act
      const response = await request(app)
        .post('/products')
        .send(productData)
        .expect(201);

      // Assert - Verificar respuesta
      expect(response.body).toMatchObject({
        id: expect.any(Number),
        name: 'Gaming Mouse',
        price: 50,
        finalPrice: 42.5, // 50 - 15%
        category: 'Electronics'
      });

      // Assert - Verificar en base de datos
      const savedProduct = await global.testDb.products.findById(response.body.id);
      expect(savedProduct).toBeTruthy();
      expect(savedProduct.name).toBe('Gaming Mouse');
    });

    it('should return 400 for invalid product data', async () => {
      // Arrange
      const invalidData = { name: 'Product without price' };

      // Act & Assert
      const response = await request(app)
        .post('/products')
        .send(invalidData)
        .expect(400);

      expect(response.body).toMatchObject({
        error: 'Name and price are required'
      });
    });
  });

  describe('GET /products', () => {
    it('should return paginated products list', async () => {
      // Arrange - Crear productos de prueba
      const products = [
        { name: 'Product 1', price: 10 },
        { name: 'Product 2', price: 20 },
        { name: 'Product 3', price: 30 }
      ];
      
      for (const product of products) {
        await global.testDb.products.create(product);
      }

      // Act
      const response = await request(app)
        .get('/products?page=1&limit=2')
        .expect(200);

      // Assert
      expect(response.body).toMatchObject({
        products: expect.arrayContaining([
          expect.objectContaining({ name: 'Product 1' }),
          expect.objectContaining({ name: 'Product 2' })
        ]),
        pagination: {
          page: 1,
          limit: 2,
          total: 3,
          totalPages: 2
        }
      });
    });
  });
});
```

## 🌐 Ejercicio 3: E2E Tests con Cypress

### Page Object Model
```javascript
// tests/e2e/pages/ProductsPage.js
class ProductsPage {
  visit() {
    cy.visit('/products');
  }

  getProductCard(productName) {
    return cy.get(`[data-testid="product-card"]`)
      .contains(productName)
      .closest('[data-testid="product-card"]');
  }

  addToCart(productName) {
    this.getProductCard(productName)
      .find('[data-testid="add-to-cart-btn"]')
      .click();
  }

  verifyCartCount(expectedCount) {
    cy.get('[data-testid="cart-count"]')
      .should('contain.text', expectedCount);
  }

  searchProduct(productName) {
    cy.get('[data-testid="search-input"]').type(productName);
    cy.get('[data-testid="search-btn"]').click();
  }

  verifyProductExists(productName) {
    this.getProductCard(productName).should('be.visible');
  }

  verifyProductNotExists(productName) {
    cy.get('[data-testid="product-card"]')
      .contains(productName)
      .should('not.exist');
  }
}

module.exports = new ProductsPage();
```

### E2E Test Spec
```javascript
// tests/e2e/specs/product-purchase-flow.cy.js
const ProductsPage = require('../pages/ProductsPage');
const CartPage = require('../pages/CartPage');
const CheckoutPage = require('../pages/CheckoutPage');

describe('Product Purchase Flow', () => {
  beforeEach(() => {
    // Seed data y autenticación
    cy.task('seedDatabase');
    cy.login('testuser@example.com', 'password123');
  });

  it('should complete full purchase journey', () => {
    // 1. Buscar producto
    ProductsPage.visit();
    ProductsPage.searchProduct('Gaming Laptop');
    ProductsPage.verifyProductExists('Gaming Laptop');

    // 2. Agregar al carrito
    ProductsPage.addToCart('Gaming Laptop');
    ProductsPage.verifyCartCount('1');

    // 3. Ir al carrito
    CartPage.visit();
    CartPage.verifyProductInCart('Gaming Laptop');
    CartPage.updateQuantity('Gaming Laptop', 2);
    CartPage.proceedToCheckout();

    // 4. Checkout
    CheckoutPage.fillShippingInfo({
      address: '123 Test Street',
      city: 'Test City',
      zipCode: '12345'
    });
    
    CheckoutPage.selectPaymentMethod('credit-card');
    CheckoutPage.fillPaymentInfo({
      cardNumber: '4242424242424242',
      expiryDate: '12/25',
      cvv: '123'
    });

    CheckoutPage.submitOrder();

    // 5. Verificar confirmación
    cy.url().should('include', '/order-confirmation');
    cy.get('[data-testid="order-success"]')
      .should('contain.text', 'Order placed successfully');
    
    cy.get('[data-testid="order-number"]')
      .should('match', /ORD-\d{6}/);
  });

  it('should handle out of stock products', () => {
    ProductsPage.visit();
    ProductsPage.searchProduct('Out of Stock Item');
    
    cy.get('[data-testid="product-card"]')
      .contains('Out of Stock Item')
      .closest('[data-testid="product-card"]')
      .within(() => {
        cy.get('[data-testid="add-to-cart-btn"]')
          .should('be.disabled');
        cy.get('[data-testid="stock-status"]')
          .should('contain.text', 'Out of Stock');
      });
  });

  it('should validate payment form', () => {
    // Setup carrito con producto
    cy.task('addProductToCart', { 
      userId: 'testuser@example.com', 
      productId: 1 
    });

    CheckoutPage.visit();
    CheckoutPage.fillShippingInfo({
      address: '123 Test Street',
      city: 'Test City',
      zipCode: '12345'
    });

    // Intentar enviar con información de pago inválida
    CheckoutPage.selectPaymentMethod('credit-card');
    CheckoutPage.fillPaymentInfo({
      cardNumber: '1234', // Número inválido
      expiryDate: '01/20', // Fecha expirada
      cvv: '12' // CVV inválido
    });

    CheckoutPage.submitOrder();

    // Verificar errores de validación
    cy.get('[data-testid="card-number-error"]')
      .should('contain.text', 'Invalid card number');
    cy.get('[data-testid="expiry-error"]')
      .should('contain.text', 'Card expired');
    cy.get('[data-testid="cvv-error"]')
      .should('contain.text', 'CVV must be 3 digits');
  });
});
```

## 🔧 Utilidades y helpers

### Database Helper
```javascript
// tests/setup/database.js
const { MongoMemoryServer } = require('mongodb-memory-server');
const mongoose = require('mongoose');

let mongoServer;

async function setupTestDatabase() {
  mongoServer = await MongoMemoryServer.create();
  const mongoUri = mongoServer.getUri();
  
  await mongoose.connect(mongoUri);
  global.testDb = mongoose.connection.db;
}

async function cleanupTestDatabase() {
  await mongoose.disconnect();
  await mongoServer.stop();
}

async function seedTestData() {
  const products = [
    { name: 'Gaming Laptop', price: 1200, stock: 10 },
    { name: 'Gaming Mouse', price: 50, stock: 25 },
    { name: 'Out of Stock Item', price: 100, stock: 0 }
  ];

  await global.testDb.products.insertMany(products);
}

module.exports = {
  setupTestDatabase,
  cleanupTestDatabase,
  seedTestData
};
```

### Cypress Custom Commands
```javascript
// tests/e2e/support/commands.js
Cypress.Commands.add('login', (email, password) => {
  cy.request({
    method: 'POST',
    url: '/api/auth/login',
    body: { email, password }
  }).then((response) => {
    window.localStorage.setItem('authToken', response.body.token);
  });
});

Cypress.Commands.add('addProductToCart', (productId, quantity = 1) => {
  cy.request({
    method: 'POST',
    url: '/api/cart/items',
    headers: {
      'Authorization': `Bearer ${window.localStorage.getItem('authToken')}`
    },
    body: { productId, quantity }
  });
});
```

## 📊 Troubleshooting común

### Problema: Tests lentos
```javascript
// ❌ Malo: Crear datos en cada test
beforeEach(async () => {
  await createUser();
  await createProducts();
  await createOrders();
});

// ✅ Bueno: Datos compartidos + limpieza específica
beforeAll(async () => {
  await createSharedTestData();
});

beforeEach(async () => {
  await cleanupOnlyModifiedData();
});
```

### Problema: Tests frágiles
```javascript
// ❌ Malo: Dependiente de timing
cy.wait(3000);
cy.get('#submit-btn').click();

// ✅ Bueno: Esperas explícitas
cy.get('#submit-btn').should('be.enabled').click();
cy.get('#success-message').should('be.visible');
```

---

**Anterior**: [03. Setup](./03-setup.md) | **Siguiente**: [05. Integración CI/CD](./05-integracion.md)
