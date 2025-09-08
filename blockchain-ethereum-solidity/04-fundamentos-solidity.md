# 03. Fundamentos de Solidity

## 🌟 Historia y Conceptos Clave

Solidity es el lenguaje principal para escribir contratos inteligentes en Ethereum. Fue creado en 2014 por Christian Reitwiessner y su sintaxis se inspira en JavaScript y C++.

### Cronología clave:
- **2014**: Nace Solidity
- **2015**: Primeros contratos en Ethereum
- **2017+**: Evolución del lenguaje y seguridad

## 🛠️ Sintaxis y Estructura Básica

```solidity
// HolaSolidity.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract HolaSolidity {
	string public mensaje = "¡Hola, Solidity!";

	function actualizarMensaje(string memory nuevoMensaje) public {
		mensaje = nuevoMensaje;
	}
}
```

### Elementos principales:
- Tipos de datos: uint, int, address, bool, string
- Variables de estado y locales
- Funciones públicas y privadas
- Modificadores de acceso

## 📊 Ejemplo Práctico

```solidity
// Contador.sol
pragma solidity ^0.8.0;

contract Contador {
	uint public valor;

	function incrementar() public {
		valor += 1;
	}

	function decrementar() public {
		require(valor > 0, "No puede ser negativo");
		valor -= 1;
	}
}
```

## ⚙️ Ejercicio Guiado

1. Crea y despliega el contrato `Contador` en Remix IDE.
2. Prueba las funciones `incrementar` y `decrementar`.
3. Modifica el contrato para permitir solo al creador modificar el valor.

## 🎯 Buenas Prácticas

- Usa versiones recientes de Solidity
- Documenta tus contratos
- Valida entradas y restringe acceso
- Prueba en testnet antes de mainnet

---

**Anterior**: [03. Ethereum y Contratos Inteligentes](./03-ethereum-smart-contracts.md) | **Siguiente**: [05. Setup y Configuración del Entorno](./05-setup.md) | **Inicio**: [README](../README.md)
