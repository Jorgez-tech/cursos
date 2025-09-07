# 05. Funciones y Estructuras de Control en Solidity

## 🧩 Funciones en Solidity

Las funciones permiten modularizar la lógica y controlar el acceso a los datos.


### Ejemplo básico

```solidity
pragma solidity ^0.8.0;

contract EjemploFunciones {
	function suma(uint a, uint b) public pure returns (uint) {
		return a + b;
	}
}
```

## 🔄 Estructuras de Control

Solidity soporta estructuras como `if`, `else`, `while`, `for` y `switch` (limitado).


### Ejemplo de control de flujo

```solidity
pragma solidity ^0.8.0;

contract ControlFlujo {
	function evaluar(uint valor) public pure returns (string memory) {
		if (valor > 10) {
			return "Mayor que 10";
		} else {
			return "10 o menos";
		}
	}
}
```

## 📝 Ejercicio Práctico

1. Crea una función que reciba un número y retorne si es par o impar.
2. Implementa un bucle `for` que sume los primeros N números naturales.

## 🎯 Buenas Prácticas

- Limita la complejidad de las funciones
- Usa nombres descriptivos
- Documenta entradas y salidas

---

**Anterior**: [04. Variables, Tipos y Operadores](./04-variables-tipos-operadores.md) | **Siguiente**: [06. Seguridad Básica en Smart Contracts](./06-seguridad-basica-smart-contracts.md)
