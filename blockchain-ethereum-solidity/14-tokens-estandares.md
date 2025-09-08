# 10. Tokens y Estándares en Blockchain

## 🪙 ¿Qué son los tokens?

Los tokens son activos digitales que representan valor, derechos o acceso dentro de una blockchain. Los estándares definen cómo deben comportarse para ser interoperables.

### Principales estándares

- **ERC-20**: Tokens fungibles (criptomonedas, puntos).
- **ERC-721**: Tokens no fungibles (NFTs).
- **ERC-1155**: Multi-tokens (fungibles y no fungibles).

## ⚡ Ejemplo: Token ERC-20 básico

```solidity
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract MiToken is ERC20 {
	constructor() ERC20("MiToken", "MTK") {
		_mint(msg.sender, 1000 * 10 ** decimals());
	}
}
```

## 📝 Ejercicio Práctico

1. Implementa un token ERC-721 básico usando OpenZeppelin.
2. Crea un script para transferir tokens entre dos cuentas.

## 🎯 Buenas Prácticas

- Usa librerías auditadas (OpenZeppelin)
- Documenta el propósito del token
- Limita permisos de mint y transfer

---

**Anterior**: [13. Interacción con Blockchain](./13-interaccion-blockchain.md) | **Siguiente**: [15. Descentralización y dApps](./15-descentralizacion-dapps.md) | **Inicio**: [README](../README.md)
