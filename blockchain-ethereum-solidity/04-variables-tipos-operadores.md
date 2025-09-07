# 04. Variables, Tipos y Operadores en Solidity

## 🌟 Tipos de Variables y Datos

Solidity soporta tipos de datos similares a otros lenguajes, pero con particularidades para blockchain.


### Tipos principales

- **uint/int**: Enteros sin/sin signo
- **address**: Dirección de cuenta
- **bool**: Booleano
- **string/bytes**: Texto y datos binarios
- **arrays/structs**: Colecciones y estructuras

## 🛠️ Ejemplo de Variables y Operadores

```solidity
// Operadores.sol
pragma solidity ^0.8.0;

contract Operadores {
	uint public resultado;

	function sumar(uint a, uint b) public {
		resultado = a + b;
	}

	function comparar(uint a, uint b) public pure returns (bool) {
		return a > b;
	}
}
```

## 📊 Ejercicio Práctico

1. Modifica el contrato para agregar funciones de resta, multiplicación y división.
2. Implementa una función que compare dos direcciones y retorne si son iguales.

## 🎯 Buenas Prácticas

- Usa tipos específicos para ahorrar gas
- Valida entradas antes de operar
- Documenta cada función

---

**Anterior**: [03. Fundamentos de Solidity](./03-fundamentos-solidity.md) | **Siguiente**: [05. Funciones y Estructuras de Control](./05-funciones-estructuras-control.md)
