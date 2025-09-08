# 06. Seguridad Básica en Smart Contracts

## 🔒 Principios de Seguridad en Solidity

La seguridad es fundamental en contratos inteligentes, ya que los errores pueden causar pérdidas irreversibles.


### Riesgos comunes

- Reentrancy
- Desbordamiento de enteros
- Acceso no autorizado

## 🛡️ Ejemplo de Protección contra Reentrancy

```solidity
pragma solidity ^0.8.0;

contract SeguroReentrancy {
	bool private locked;

	modifier noReentrancy() {
		require(!locked, "No reentrancy");
		locked = true;
		_;
		locked = false;
	}

	function retirar() public noReentrancy {
		// lógica de retiro
	}
}
```

## 📝 Ejercicio Práctico

1. Implementa un modifier que solo permita al owner ejecutar una función.
2. Simula un ataque de reentrancy y corrígelo.

## 🎯 Buenas Prácticas

- Usa modifers para controlar acceso
- Valida entradas y salidas
- Audita el código antes de desplegar

---

**Anterior**: [10. Desarrollo y Pruebas de Smart Contracts](./10-desarrollo-pruebas-smart-contracts.md) | **Siguiente**: [12. Integración y Seguridad Avanzada](./12-integracion.md) | **Inicio**: [README](../README.md)
