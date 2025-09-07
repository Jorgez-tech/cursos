# 03. Setup y Configuración del Entorno de Desarrollo

## 🛠️ Herramientas Necesarias

Para desarrollar en blockchain y Solidity necesitarás un entorno completo que incluya editores, compiladores, redes de prueba y herramientas de despliegue.

### Herramientas Esenciales

1. **Node.js y npm**: Para gestión de paquetes y herramientas JavaScript
2. **IDE/Editor**: Visual Studio Code con extensiones de Solidity
3. **Hardhat o Truffle**: Frameworks de desarrollo para Ethereum
4. **MetaMask**: Wallet para interactuar con dApps
5. **Git**: Control de versiones

## 🚀 Instalación Paso a Paso

### 1. Instalar Node.js

```bash
# Verificar si Node.js está instalado
node --version
npm --version

# Si no está instalado, descarga desde: https://nodejs.org/
# Versión recomendada: LTS (18.x o superior)
```

### 2. Configurar Visual Studio Code

```bash
# Instalar Visual Studio Code desde: https://code.visualstudio.com/

# Extensiones recomendadas:
# - Solidity (Juan Blanco)
# - Hardhat for VS Code
# - GitLens
# - Prettier
```

### 3. Crear Proyecto con Hardhat

```bash
# Crear directorio del proyecto
mkdir mi-proyecto-blockchain
cd mi-proyecto-blockchain

# Inicializar proyecto de Node.js
npm init -y

# Instalar Hardhat
npm install --save-dev hardhat

# Inicializar proyecto de Hardhat
npx hardhat init

# Selecciona: "Create a JavaScript project"
```

### 4. Instalar Dependencias Adicionales

```bash
# Instalar OpenZeppelin (contratos seguros)
npm install @openzeppelin/contracts

# Instalar dotenv para variables de entorno
npm install --save-dev dotenv

# Instalar ethers.js para interactuar con Ethereum
npm install ethers
```

## 🌐 Configuración de Redes

### Configurar Hardhat para Redes de Prueba

Edita el archivo `hardhat.config.js`:

```javascript
require("@nomicfoundation/hardhat-toolbox");
require("dotenv").config();

module.exports = {
  solidity: "0.8.19",
  networks: {
    hardhat: {
      // Red local de Hardhat
    },
    goerli: {
      url: process.env.GOERLI_URL || "",
      accounts: [process.env.PRIVATE_KEY || ""]
    },
    sepolia: {
      url: process.env.SEPOLIA_URL || "",
      accounts: [process.env.PRIVATE_KEY || ""]
    }
  },
  etherscan: {
    apiKey: process.env.ETHERSCAN_API_KEY
  }
};
```

### Archivo .env (Variables de Entorno)

```bash
# Crear archivo .env en la raíz del proyecto
touch .env

# Agregar contenido (NUNCA compartir estas claves):
GOERLI_URL=https://eth-goerli.g.alchemy.com/v2/YOUR_API_KEY
SEPOLIA_URL=https://eth-sepolia.g.alchemy.com/v2/YOUR_API_KEY
PRIVATE_KEY=your_private_key_here
ETHERSCAN_API_KEY=your_etherscan_api_key
```

## 🦊 Configurar MetaMask

### Instalación y Configuración

1. **Instalar extensión**: Descarga MetaMask desde [metamask.io](https://metamask.io)
2. **Crear wallet**: Sigue las instrucciones para crear una nueva wallet
3. **Guardar seed phrase**: ⚠️ CRÍTICO - Guarda tu frase de recuperación de forma segura
4. **Agregar redes de prueba**:
   - Configuración → Redes → Agregar red
   - Goerli: RPC URL, Chain ID, Symbol
   - Sepolia: RPC URL, Chain ID, Symbol

### Obtener ETH de Prueba

```bash
# Faucets para obtener ETH de prueba:
# Goerli: https://faucets.chain.link/goerli
# Sepolia: https://faucets.chain.link/sepolia

# Copia tu dirección de MetaMask y solicita fondos
```

## 🧪 Proyecto Base de Prueba

### Crear tu Primer Contrato

```solidity
// contracts/HolaMundo.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract HolaMundo {
    string public mensaje = "¡Hola, Blockchain!";
    
    function actualizarMensaje(string memory _nuevoMensaje) public {
        mensaje = _nuevoMensaje;
    }
    
    function obtenerMensaje() public view returns (string memory) {
        return mensaje;
    }
}
```

### Script de Despliegue

```javascript
// scripts/deploy.js
async function main() {
  const HolaMundo = await ethers.getContractFactory("HolaMundo");
  const holaMundo = await HolaMundo.deploy();
  
  await holaMundo.deployed();
  
  console.log("HolaMundo desplegado en:", holaMundo.address);
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
```

### Compilar y Desplegar

```bash
# Compilar contratos
npx hardhat compile

# Desplegar en red local
npx hardhat run scripts/deploy.js

# Desplegar en red de prueba
npx hardhat run scripts/deploy.js --network goerli
```

## ⚙️ Ejercicio Práctico

1. **Setup completo**: Sigue todos los pasos de instalación
2. **Proyecto base**: Crea y despliega el contrato HolaMundo
3. **Interacción**: Usa MetaMask para interactuar con tu contrato
4. **Verificación**: Verifica tu contrato en Etherscan

## 🎯 Buenas Prácticas

- **Nunca expongas claves privadas** en código o repositorios públicos
- **Usa redes de prueba** antes de desplegar en mainnet
- **Mantén actualizadas** las dependencias y herramientas
- **Realiza backups** de tus wallets y proyectos

## 🔧 Troubleshooting Común

### Error de Gas Insuficiente
```bash
# Aumentar gas limit en hardhat.config.js
gas: 3000000
```

### Error de Versión de Solidity
```bash
# Verificar versión en hardhat.config.js coincida con contratos
solidity: "0.8.19"
```

### Error de Red
```bash
# Verificar URLs de RPC y conectividad
# Revisar variables de entorno en .env
```

## 🔗 Recursos Adicionales

- [Hardhat Documentation](https://hardhat.org/docs)
- [MetaMask Documentation](https://docs.metamask.io/)
- [Solidity Documentation](https://docs.soliditylang.org/)
- [OpenZeppelin Documentation](https://docs.openzeppelin.com/)

---

**Anterior**: [02. Conceptos](./02-conceptos.md) | **Siguiente**: [04. Variables, Tipos y Operadores](./04-variables-tipos-operadores.md) | **Inicio**: [README](../README.md)