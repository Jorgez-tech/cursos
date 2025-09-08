# 05. Integración y Seguridad en Desarrollo Blockchain

## 🔗 Integración con Servicios Externos

### APIs y Oráculos

Los contratos inteligentes pueden necesitar datos del mundo real. Los oráculos son servicios que proporcionan esta información de manera segura.

#### Chainlink Oracles

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@chainlink/contracts/src/v0.8/interfaces/AggregatorV3Interface.sol";

contract PrecioCriptomoneda {
    AggregatorV3Interface internal priceFeed;
    
    constructor() {
        // ETH/USD en Goerli
        priceFeed = AggregatorV3Interface(0xD4a33860578De61DBAbDc8BFdb98FD742fA7028e);
    }
    
    function obtenerPrecioETH() public view returns (int) {
        (, int price, , , ) = priceFeed.latestRoundData();
        return price;
    }
}
```

### Integración con IPFS

Para almacenar archivos grandes descentralizadamente:

```javascript
// Frontend: Subir a IPFS
const IPFS = require('ipfs-http-client');
const ipfs = IPFS.create({ host: 'ipfs.infura.io', port: 5001, protocol: 'https' });

async function subirAIPFS(archivo) {
    const result = await ipfs.add(archivo);
    return result.path; // Hash IPFS
}
```

```solidity
// Contrato: Referenciar archivo IPFS
contract DocumentoDescentralizado {
    mapping(uint256 => string) public documentos; // ID => IPFS hash
    uint256 public contadorDocumentos;
    
    function agregarDocumento(string memory ipfsHash) public {
        documentos[contadorDocumentos] = ipfsHash;
        contadorDocumentos++;
    }
}
```

## 🛡️ Seguridad en Smart Contracts

### Vulnerabilidades Comunes

#### 1. Reentrancy Attack

```solidity
// ❌ VULNERABLE
contract VulnerableBank {
    mapping(address => uint) public balances;
    
    function withdraw() public {
        uint balance = balances[msg.sender];
        require(balance > 0);
        
        (bool success, ) = msg.sender.call{value: balance}("");
        require(success);
        
        balances[msg.sender] = 0; // ⚠️ Después del call!
    }
}

// ✅ SEGURO - ReentrancyGuard
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

contract SecureBank is ReentrancyGuard {
    mapping(address => uint) public balances;
    
    function withdraw() public nonReentrant {
        uint balance = balances[msg.sender];
        require(balance > 0);
        
        balances[msg.sender] = 0; // ✅ Antes del call!
        
        (bool success, ) = msg.sender.call{value: balance}("");
        require(success);
    }
}
```

#### 2. Integer Overflow/Underflow

```solidity
// ✅ SEGURO en Solidity 0.8+
contract SecureCounter {
    uint256 public counter;
    
    function increment() public {
        counter++; // Auto-revierte en overflow
    }
    
    function decrement() public {
        counter--; // Auto-revierte en underflow
    }
}
```

#### 3. Access Control

```solidity
// ✅ CONTROL DE ACCESO ROBUSTO
import "@openzeppelin/contracts/access/AccessControl.sol";

contract AdminContract is AccessControl {
    bytes32 public constant ADMIN_ROLE = keccak256("ADMIN_ROLE");
    bytes32 public constant MODERATOR_ROLE = keccak256("MODERATOR_ROLE");
    
    constructor() {
        _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
        _grantRole(ADMIN_ROLE, msg.sender);
    }
    
    function funcionAdmin() public onlyRole(ADMIN_ROLE) {
        // Solo admins pueden ejecutar
    }
    
    function funcionModerador() public onlyRole(MODERATOR_ROLE) {
        // Solo moderadores pueden ejecutar
    }
}
```

## 🧪 Testing de Integración

### Setup de Testing con Hardhat

```javascript
// test/integration/FullSystem.test.js
const { expect } = require("chai");
const { ethers } = require("hardhat");

describe("Sistema Completo", function () {
    let votacion, token, owner, addr1, addr2;
    
    beforeEach(async function () {
        [owner, addr1, addr2] = await ethers.getSigners();
        
        // Deploy Token
        const Token = await ethers.getContractFactory("VotingToken");
        token = await Token.deploy("VoteToken", "VTK");
        
        // Deploy Votacion con dependencia de Token
        const Votacion = await ethers.getContractFactory("TokenVoting");
        votacion = await Votacion.deploy(token.address);
        
        // Distribuir tokens
        await token.transfer(addr1.address, ethers.utils.parseEther("100"));
        await token.transfer(addr2.address, ethers.utils.parseEther("100"));
    });
    
    it("Integración completa: crear propuesta y votar con tokens", async function () {
        // Crear propuesta
        await votacion.crearPropuesta("Propuesta de integración");
        
        // Aprobar tokens para el contrato de votación
        await token.connect(addr1).approve(votacion.address, ethers.utils.parseEther("10"));
        
        // Votar usando tokens
        await votacion.connect(addr1).votarConTokens(0, true, ethers.utils.parseEther("10"));
        
        const propuesta = await votacion.obtenerPropuesta(0);
        expect(propuesta.votosFavor).to.equal(ethers.utils.parseEther("10"));
    });
});
```

## 🔄 CI/CD para Blockchain

### GitHub Actions Workflow

```yaml
# .github/workflows/blockchain-ci.yml
name: Blockchain CI/CD

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Compile contracts
      run: npx hardhat compile
    
    - name: Run tests
      run: npx hardhat test
    
    - name: Coverage
      run: npx hardhat coverage
    
    - name: Upload coverage
      uses: codecov/codecov-action@v3
```

## 🌐 Integración Frontend/Backend

### React + Ethers.js

```javascript
// hooks/useContract.js
import { useState, useEffect } from 'react';
import { ethers } from 'ethers';

export function useContract(contractAddress, abi) {
    const [contract, setContract] = useState(null);
    const [loading, setLoading] = useState(true);
    
    useEffect(() => {
        async function initContract() {
            if (window.ethereum) {
                const provider = new ethers.providers.Web3Provider(window.ethereum);
                const signer = provider.getSigner();
                const contractInstance = new ethers.Contract(contractAddress, abi, signer);
                setContract(contractInstance);
            }
            setLoading(false);
        }
        
        initContract();
    }, [contractAddress, abi]);
    
    return { contract, loading };
}
```

## ⚙️ Ejercicios Prácticos

1. **Integrar Chainlink**: Modifica el contrato de votación para incluir precio de ETH
2. **Implementar IPFS**: Almacena descripciones de propuestas en IPFS
3. **Testing de seguridad**: Ejecuta análisis con Slither y Mythril
4. **Frontend completo**: Desarrolla una interfaz React completa

## 🎯 Checklist de Seguridad

- ✅ Usar contratos auditados de OpenZeppelin
- ✅ Implementar ReentrancyGuard cuando sea necesario
- ✅ Validar todas las entradas de funciones
- ✅ Usar eventos para logging importante
- ✅ Implementar pausabilidad para emergencias
- ✅ Testing exhaustivo incluyendo edge cases
- ✅ Auditorías de código antes de mainnet

## 🔗 Recursos Adicionales

- [OpenZeppelin Security Guidelines](https://docs.openzeppelin.com/contracts/4.x/security)
- [Consensys Smart Contract Best Practices](https://consensys.github.io/smart-contract-best-practices/)
- [Chainlink Documentation](https://docs.chain.link/)
- [IPFS Documentation](https://docs.ipfs.io/)

---

**Anterior**: [11. Seguridad Básica en Smart Contracts](./11-seguridad-basica-smart-contracts.md) | **Siguiente**: [13. Interacción con Blockchain](./13-interaccion-blockchain.md) | **Inicio**: [README](../README.md)