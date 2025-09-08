# 17. Seguridad Avanzada en Smart Contracts

## 🛡️ Amenazas avanzadas y mitigaciones

Los contratos inteligentes pueden ser vulnerables a ataques sofisticados. Es clave aplicar técnicas avanzadas de seguridad y auditoría.

### Principales amenazas

- Ataques de front-running
- Manipulación de oráculos
- Phishing y suplantación

## ⚡ Ejemplo: Uso de oráculos seguros

```solidity
pragma solidity ^0.8.0;

interface IOracle {
	function getPrecio() external view returns (uint);
}

contract ConsultaOracle {
	IOracle public oracle;

	constructor(address oracleAddress) {
		oracle = IOracle(oracleAddress);
	}

	function obtenerPrecioSeguro() public view returns (uint) {
		return oracle.getPrecio();
	}
}
```

## 📝 Ejercicio Práctico

1. Implementa validaciones para evitar front-running en una subasta.
2. Simula un ataque de oráculo y propón una solución.

## 🎯 Buenas Prácticas

- Audita el código con herramientas automáticas y manuales
- Limita la confianza en fuentes externas
- Mantén actualizadas las dependencias

---

**Anterior**: [16. Gas y Optimización](./16-gas-optimizacion.md) | **Siguiente**: [18. Documentación y Escalabilidad](./18-documentacion.md) | **Inicio**: [README](../README.md)
