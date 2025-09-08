# 16. Gas y Optimización en Smart Contracts

## ⛽ ¿Qué es el gas?

El gas es la unidad de medida para el costo computacional de ejecutar operaciones en la blockchain. Optimizar el uso de gas reduce costos y mejora la eficiencia.

### Factores que afectan el gas

- Complejidad de las funciones
- Uso de almacenamiento
- Bucles y llamadas externas

## ⚡ Ejemplo: Optimización de gas

```solidity
pragma solidity ^0.8.0;

contract GasOptimizado {
	uint[] public datos;

	function agregar(uint valor) public {
		datos.push(valor);
	}

	// Optimización: usar memoria en vez de almacenamiento
	function suma(uint[] memory valores) public pure returns (uint total) {
		for (uint i = 0; i < valores.length; i++) {
			total += valores[i];
		}
	}
}
```

## 📝 Ejercicio Práctico

1. Identifica funciones costosas en gas y propón mejoras.
2. Compara el gas usado por diferentes implementaciones de una función.

## 🎯 Buenas Prácticas

- Minimiza escrituras en almacenamiento
- Evita bucles innecesarios
- Audita el consumo de gas antes de desplegar

---

**Anterior**: [15. Descentralización y dApps](./15-descentralizacion-dapps.md) | **Siguiente**: [17. Seguridad Avanzada](./17-seguridad-avanzada-smart-contracts.md) | **Inicio**: [README](../README.md)
