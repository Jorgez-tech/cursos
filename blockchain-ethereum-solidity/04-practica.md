# 04. Práctica Guiada - Desarrollando Tu Primer dApp

## 🎯 Objetivo del Módulo

En esta práctica desarrollaremos una dApp completa que incluye contratos inteligentes, interfaz web y despliegue en una red de prueba. Crearemos un sistema de votación descentralizado.

## 🗳️ Proyecto: Sistema de Votación Descentralizado

### Características del Proyecto

- Creación de propuestas de votación
- Votación con restricciones (un voto por dirección)
- Consulta de resultados en tiempo real
- Interfaz web para interactuar

## 📋 Prerequisitos

- Entorno de desarrollo configurado (Módulo 03)
- Conocimientos básicos de Solidity
- MetaMask configurado con ETH de prueba
- Node.js y npm instalados

## 🔨 Paso 1: Crear el Contrato de Votación

```solidity
// contracts/Votacion.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Votacion {
    struct Propuesta {
        string descripcion;
        uint256 votosFavor;
        uint256 votosContra;
        bool activa;
        mapping(address => bool) haVotado;
    }
    
    mapping(uint256 => Propuesta) public propuestas;
    uint256 public totalPropuestas;
    address public owner;
    
    event PropuestaCreada(uint256 indexed id, string descripcion);
    event VotoEmitido(uint256 indexed propuestaId, address indexed votante, bool voto);
    
    modifier soloOwner() {
        require(msg.sender == owner, "Solo el owner puede ejecutar esta funcion");
        _;
    }
    
    constructor() {
        owner = msg.sender;
        totalPropuestas = 0;
    }
    
    function crearPropuesta(string memory _descripcion) public soloOwner {
        uint256 propuestaId = totalPropuestas;
        
        Propuesta storage nuevaPropuesta = propuestas[propuestaId];
        nuevaPropuesta.descripcion = _descripcion;
        nuevaPropuesta.votosFavor = 0;
        nuevaPropuesta.votosContra = 0;
        nuevaPropuesta.activa = true;
        
        totalPropuestas++;
        
        emit PropuestaCreada(propuestaId, _descripcion);
    }
    
    function votar(uint256 _propuestaId, bool _voto) public {
        require(_propuestaId < totalPropuestas, "Propuesta no existe");
        require(propuestas[_propuestaId].activa, "Propuesta no activa");
        require(!propuestas[_propuestaId].haVotado[msg.sender], "Ya has votado");
        
        propuestas[_propuestaId].haVotado[msg.sender] = true;
        
        if (_voto) {
            propuestas[_propuestaId].votosFavor++;
        } else {
            propuestas[_propuestaId].votosContra++;
        }
        
        emit VotoEmitido(_propuestaId, msg.sender, _voto);
    }
    
    function cerrarPropuesta(uint256 _propuestaId) public soloOwner {
        require(_propuestaId < totalPropuestas, "Propuesta no existe");
        propuestas[_propuestaId].activa = false;
    }
    
    function obtenerPropuesta(uint256 _propuestaId) public view returns (
        string memory descripcion,
        uint256 votosFavor,
        uint256 votosContra,
        bool activa
    ) {
        require(_propuestaId < totalPropuestas, "Propuesta no existe");
        
        Propuesta storage propuesta = propuestas[_propuestaId];
        return (
            propuesta.descripcion,
            propuesta.votosFavor,
            propuesta.votosContra,
            propuesta.activa
        );
    }
    
    function haVotado(uint256 _propuestaId, address _votante) public view returns (bool) {
        require(_propuestaId < totalPropuestas, "Propuesta no existe");
        return propuestas[_propuestaId].haVotado[_votante];
    }
}
```

## 🔨 Paso 2: Script de Despliegue

```javascript
// scripts/deploy-votacion.js
async function main() {
  const [deployer] = await ethers.getSigners();
  
  console.log("Desplegando contratos con la cuenta:", deployer.address);
  console.log("Balance de la cuenta:", (await deployer.getBalance()).toString());
  
  const Votacion = await ethers.getContractFactory("Votacion");
  const votacion = await Votacion.deploy();
  
  await votacion.deployed();
  
  console.log("Contrato Votacion desplegado en:", votacion.address);
  
  // Crear una propuesta de ejemplo
  await votacion.crearPropuesta("¿Deberíamos adoptar un nuevo protocolo de consenso?");
  console.log("Propuesta de ejemplo creada");
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

## 🔨 Paso 3: Tests del Contrato

```javascript
// test/Votacion.test.js
const { expect } = require("chai");
const { ethers } = require("hardhat");

describe("Votacion", function () {
  let Votacion;
  let votacion;
  let owner;
  let addr1;
  let addr2;

  beforeEach(async function () {
    Votacion = await ethers.getContractFactory("Votacion");
    [owner, addr1, addr2] = await ethers.getSigners();
    votacion = await Votacion.deploy();
  });

  describe("Deployment", function () {
    it("Debería establecer el owner correcto", async function () {
      expect(await votacion.owner()).to.equal(owner.address);
    });

    it("Debería inicializar totalPropuestas en 0", async function () {
      expect(await votacion.totalPropuestas()).to.equal(0);
    });
  });

  describe("Crear Propuestas", function () {
    it("Debería crear una propuesta correctamente", async function () {
      await votacion.crearPropuesta("Propuesta de prueba");
      expect(await votacion.totalPropuestas()).to.equal(1);
      
      const propuesta = await votacion.obtenerPropuesta(0);
      expect(propuesta.descripcion).to.equal("Propuesta de prueba");
      expect(propuesta.activa).to.be.true;
    });

    it("Solo el owner debería poder crear propuestas", async function () {
      await expect(
        votacion.connect(addr1).crearPropuesta("Propuesta no autorizada")
      ).to.be.revertedWith("Solo el owner puede ejecutar esta funcion");
    });
  });

  describe("Votar", function () {
    beforeEach(async function () {
      await votacion.crearPropuesta("Propuesta para votar");
    });

    it("Debería permitir votar a favor", async function () {
      await votacion.connect(addr1).votar(0, true);
      
      const propuesta = await votacion.obtenerPropuesta(0);
      expect(propuesta.votosFavor).to.equal(1);
      expect(propuesta.votosContra).to.equal(0);
    });

    it("Debería permitir votar en contra", async function () {
      await votacion.connect(addr1).votar(0, false);
      
      const propuesta = await votacion.obtenerPropuesta(0);
      expect(propuesta.votosFavor).to.equal(0);
      expect(propuesta.votosContra).to.equal(1);
    });

    it("No debería permitir votar dos veces", async function () {
      await votacion.connect(addr1).votar(0, true);
      
      await expect(
        votacion.connect(addr1).votar(0, false)
      ).to.be.revertedWith("Ya has votado");
    });
  });
});
```

## 🚀 Paso 5: Despliegue y Pruebas

### 1. Compilar y Testear

```bash
# Compilar contratos
npx hardhat compile

# Ejecutar tests
npx hardhat test

# Verificar cobertura
npx hardhat coverage
```

### 2. Desplegar en Red de Prueba

```bash
# Desplegar en Goerli
npx hardhat run scripts/deploy-votacion.js --network goerli

# Verificar contrato (opcional)
npx hardhat verify --network goerli DIRECCION_DEL_CONTRATO
```

## ⚙️ Ejercicios Adicionales

1. **Mejora el contrato**: Agrega funcionalidad para establecer fecha límite de votación
2. **Mejora la interfaz**: Agrega estilos CSS más avanzados o usa un framework como React
3. **Agrega más funciones**: Implementa la capacidad de editar propuestas
4. **Eventos en tiempo real**: Usa eventos de Ethereum para actualizar la interfaz automáticamente

## 🎯 Buenas Prácticas Aplicadas

- ✅ **Modificadores**: Uso de `modifier` para control de acceso
- ✅ **Eventos**: Emisión de eventos para logging
- ✅ **Validaciones**: Checks antes de ejecutar lógica crítica
- ✅ **Testing**: Casos de prueba completos
- ✅ **Documentación**: Comentarios claros en el código

## 🔗 Recursos para Continuar

- [Hardhat Tasks](https://hardhat.org/hardhat-runner/docs/advanced/building-plugins)
- [React + Ethers.js Tutorial](https://docs.ethers.io/v5/getting-started/)
- [OpenZeppelin Upgradeable Contracts](https://docs.openzeppelin.com/upgrades-plugins/1.x/)

---

**Anterior**: [03. Setup](./03-setup.md) | **Siguiente**: [05. Funciones y Estructuras de Control](./05-funciones-estructuras-control.md) | **Inicio**: [README](../README.md)