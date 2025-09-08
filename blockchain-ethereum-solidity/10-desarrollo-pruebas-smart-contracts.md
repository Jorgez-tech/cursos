# 10. Desarrollo y Pruebas de Smart Contracts

## 🧪 Ciclo de Desarrollo

El desarrollo profesional de smart contracts incluye diseño, codificación, pruebas y despliegue.

### Fases principales

- Diseño y modelado
- Implementación en Solidity
- Pruebas unitarias y de integración
- Despliegue en testnet/mainnet

## ⚡ Ejemplo: Prueba Unitaria con Hardhat

```javascript
// test/miContratoTest.js
const { expect } = require("chai");

describe("MiContrato", function () {
		it("debería inicializar correctamente", async function () {
			const MiContrato = await ethers.getContractFactory("MiContrato");
			const contrato = await MiContrato.deploy();
			expect(await contrato.valor()).to.equal(0);
		});
});
```

## 📝 Ejercicio Práctico

1. Escribe una prueba unitaria para una función de suma en Solidity.
2. Despliega el contrato en una testnet usando Hardhat.

## 🎯 Buenas Prácticas

- Automatiza pruebas con scripts
- Usa mocks para simular dependencias
- Revisa cobertura de pruebas

---

**Anterior**: [09. Herramientas de Desarrollo](./09-herramientas-desarrollo.md) | **Siguiente**: [11. Seguridad Básica en Smart Contracts](./11-seguridad-basica-smart-contracts.md) | **Inicio**: [README](../README.md)
