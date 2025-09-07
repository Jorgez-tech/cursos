# 09. Interacción con la Blockchain

## 🔗 ¿Cómo interactúan los contratos con la blockchain?

Los contratos inteligentes se comunican con la blockchain mediante transacciones, llamadas y eventos. Los usuarios y aplicaciones pueden interactuar usando wallets, scripts o aplicaciones web.

### Métodos de interacción

- **Transacciones**: Modifican el estado y requieren gas.
- **Llamadas de solo lectura**: Consultan datos sin gastar gas.
- **Eventos**: Permiten notificar acciones a aplicaciones externas.

## ⚡ Ejemplo: Interacción desde JavaScript

```javascript
// interact.js
const { ethers } = require("ethers");
const provider = new ethers.JsonRpcProvider("https://sepolia.infura.io/v3/TU_API_KEY");
const contrato = new ethers.Contract("DIRECCION_CONTRATO", ABI, provider);

async function leerValor() {
				const valor = await contrato.valor();
				console.log("Valor actual:", valor);
}

leerValor();
```

## 📝 Ejercicio Práctico

1. Realiza una llamada de solo lectura a un contrato desplegado en testnet.
2. Envía una transacción para modificar el estado del contrato.

## 🎯 Buenas Prácticas

- Protege las claves privadas y credenciales
- Usa proveedores seguros (Infura, Alchemy)
- Maneja errores y confirma transacciones

---

**Anterior**: [08. Desarrollo y Pruebas de Smart Contracts](./08-desarrollo-pruebas-smart-contracts.md) | **Siguiente**: [10. Tokens y Estándares](./10-tokens-estandares.md)
