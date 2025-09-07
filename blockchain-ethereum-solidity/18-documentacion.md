# 06. Documentación y Escalabilidad

## 📚 Documentación Técnica de Contratos

### NatSpec Documentation

Solidity incluye un sistema de documentación llamado NatSpec (Natural Specification Format) que permite generar documentación automáticamente.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

/**
 * @title Sistema de Votación Avanzado
 * @author Tu Nombre
 * @notice Este contrato permite crear y gestionar votaciones descentralizadas
 * @dev Implementa funciones de votación con tokens y delegación
 */
contract VotacionAvanzada {
    
    /**
     * @notice Estructura que define una propuesta de votación
     * @dev Incluye mapping interno para tracking de votantes
     */
    struct Propuesta {
        string descripcion;
        uint256 votosFavor;
        uint256 votosContra;
        uint256 fechaInicio;
        uint256 fechaFin;
        bool activa;
        mapping(address => bool) haVotado;
        mapping(address => uint256) pesoVoto; // Para voting power
    }
    
    /// @notice Mapping de ID de propuesta a datos de la propuesta
    mapping(uint256 => Propuesta) public propuestas;
    
    /// @notice Número total de propuestas creadas
    uint256 public totalPropuestas;
    
    /// @notice Dirección del administrador del contrato
    address public admin;
    
    /**
     * @notice Evento emitido cuando se crea una nueva propuesta
     * @param id ID único de la propuesta
     * @param descripcion Descripción de la propuesta
     * @param fechaInicio Timestamp de inicio de votación
     * @param fechaFin Timestamp de fin de votación
     */
    event PropuestaCreada(
        uint256 indexed id,
        string descripcion,
        uint256 fechaInicio,
        uint256 fechaFin
    );
    
    /**
     * @notice Evento emitido cuando se emite un voto
     * @param propuestaId ID de la propuesta votada
     * @param votante Dirección del votante
     * @param voto true para voto a favor, false para voto en contra
     * @param peso Peso del voto (en tokens)
     */
    event VotoEmitido(
        uint256 indexed propuestaId,
        address indexed votante,
        bool voto,
        uint256 peso
    );
    
    /**
     * @notice Constructor del contrato
     * @dev Establece el deployer como administrador inicial
     */
    constructor() {
        admin = msg.sender;
        totalPropuestas = 0;
    }
    
    /**
     * @notice Crea una nueva propuesta de votación
     * @dev Solo el administrador puede crear propuestas
     * @param _descripcion Descripción de la propuesta
     * @param _duracionDias Duración de la votación en días
     * @return propuestaId ID de la propuesta creada
     */
    function crearPropuesta(
        string memory _descripcion,
        uint256 _duracionDias
    ) public returns (uint256 propuestaId) {
        require(msg.sender == admin, "Solo el admin puede crear propuestas");
        require(bytes(_descripcion).length > 0, "La descripcion no puede estar vacia");
        require(_duracionDias > 0, "La duracion debe ser mayor a 0");
        
        propuestaId = totalPropuestas;
        
        Propuesta storage nuevaPropuesta = propuestas[propuestaId];
        nuevaPropuesta.descripcion = _descripcion;
        nuevaPropuesta.fechaInicio = block.timestamp;
        nuevaPropuesta.fechaFin = block.timestamp + (_duracionDias * 1 days);
        nuevaPropuesta.activa = true;
        
        totalPropuestas++;
        
        emit PropuestaCreada(
            propuestaId,
            _descripcion,
            nuevaPropuesta.fechaInicio,
            nuevaPropuesta.fechaFin
        );
    }
    
    /**
     * @notice Emite un voto para una propuesta específica
     * @dev El votante debe tener tokens y no haber votado previamente
     * @param _propuestaId ID de la propuesta
     * @param _voto true para votar a favor, false para votar en contra
     */
    function votar(uint256 _propuestaId, bool _voto) public {
        require(_propuestaId < totalPropuestas, "Propuesta inexistente");
        require(propuestas[_propuestaId].activa, "Propuesta no activa");
        require(
            block.timestamp >= propuestas[_propuestaId].fechaInicio,
            "Votacion aun no ha comenzado"
        );
        require(
            block.timestamp <= propuestas[_propuestaId].fechaFin,
            "Votacion ha finalizado"
        );
        require(
            !propuestas[_propuestaId].haVotado[msg.sender],
            "Ya has votado en esta propuesta"
        );
        
        // Aquí asumiríamos integración con un token ERC20 para peso de voto
        uint256 pesoVoto = 1; // Por simplicidad, cada voto vale 1
        
        propuestas[_propuestaId].haVotado[msg.sender] = true;
        propuestas[_propuestaId].pesoVoto[msg.sender] = pesoVoto;
        
        if (_voto) {
            propuestas[_propuestaId].votosFavor += pesoVoto;
        } else {
            propuestas[_propuestaId].votosContra += pesoVoto;
        }
        
        emit VotoEmitido(_propuestaId, msg.sender, _voto, pesoVoto);
    }
    
    /**
     * @notice Obtiene la información completa de una propuesta
     * @param _propuestaId ID de la propuesta
     * @return descripcion Descripción de la propuesta
     * @return votosFavor Número de votos a favor
     * @return votosContra Número de votos en contra
     * @return fechaInicio Timestamp de inicio
     * @return fechaFin Timestamp de fin
     * @return activa Estado de la propuesta
     */
    function obtenerPropuesta(uint256 _propuestaId)
        public
        view
        returns (
            string memory descripcion,
            uint256 votosFavor,
            uint256 votosContra,
            uint256 fechaInicio,
            uint256 fechaFin,
            bool activa
        )
    {
        require(_propuestaId < totalPropuestas, "Propuesta inexistente");
        
        Propuesta storage prop = propuestas[_propuestaId];
        return (
            prop.descripcion,
            prop.votosFavor,
            prop.votosContra,
            prop.fechaInicio,
            prop.fechaFin,
            prop.activa
        );
    }
}
```

### Generar Documentación

```bash
# Instalar solc y dodoc
npm install -g solc
npm install --save-dev @primitivefi/hardhat-dodoc

# En hardhat.config.js
require("@primitivefi/hardhat-dodoc");

module.exports = {
  dodoc: {
    runOnCompile: true,
    debugMode: true,
    outputDir: "./docs",
    include: ["contracts"]
  }
};

# Generar documentación
npx hardhat dodoc
```

## 📊 Escalabilidad y Optimización

### Layer 2 Solutions

#### Polygon (MATIC)

```javascript
// hardhat.config.js - Configuración para Polygon
module.exports = {
  networks: {
    polygon: {
      url: "https://polygon-rpc.com",
      accounts: [process.env.PRIVATE_KEY],
      gasPrice: 20000000000, // 20 gwei
    },
    mumbai: {
      url: "https://rpc-mumbai.maticvigil.com",
      accounts: [process.env.PRIVATE_KEY],
      gasPrice: 20000000000,
    }
  }
};
```

#### Arbitrum

```javascript
// Configuración para Arbitrum
arbitrum: {
  url: "https://arb1.arbitrum.io/rpc",
  accounts: [process.env.PRIVATE_KEY],
  gasPrice: 1000000000, // 1 gwei
}
```

### Optimización de Gas

#### 1. Packed Structs

```solidity
// ❌ No optimizado (cada variable usa 32 bytes)
struct PersonaNoOptimizada {
    uint256 id;        // 32 bytes
    bool activo;       // 32 bytes (desperdicia 31 bytes)
    uint8 edad;        // 32 bytes (desperdicia 31 bytes)
    address direccion; // 32 bytes (usa 20, desperdicia 12)
}

// ✅ Optimizado (packed en menos slots)
struct PersonaOptimizada {
    uint256 id;        // Slot 1: 32 bytes
    address direccion; // Slot 2: 20 bytes
    uint8 edad;        // Slot 2: 1 byte (total 21 bytes)
    bool activo;       // Slot 2: 1 byte (total 22 bytes)
    // Slot 2 desperdicia solo 10 bytes en lugar de 94
}
```

#### 2. Batch Operations

```solidity
// ✅ Procesar múltiples operaciones en una sola transacción
function votarMultiple(
    uint256[] calldata propuestaIds,
    bool[] calldata votos
) external {
    require(propuestaIds.length == votos.length, "Arrays must have same length");
    
    for (uint256 i = 0; i < propuestaIds.length; i++) {
        _votar(propuestaIds[i], votos[i]);
    }
}

function _votar(uint256 propuestaId, bool voto) internal {
    // Lógica de votación
}
```

#### 3. Storage vs Memory vs Calldata

```solidity
// Gas costs comparison
function procesarDatos(
    uint256[] calldata datos // ✅ Más barato para read-only
) external pure returns (uint256) {
    uint256 suma = 0;
    for (uint256 i = 0; i < datos.length; i++) {
        suma += datos[i];
    }
    return suma;
}

function procesarDatosMemory(
    uint256[] memory datos // ⚠️ Más caro, pero permite modificación
) external pure returns (uint256[] memory) {
    for (uint256 i = 0; i < datos.length; i++) {
        datos[i] = datos[i] * 2;
    }
    return datos;
}
```

### State Channels

```solidity
// Simplified Payment Channel
contract PaymentChannel {
    address public sender;
    address public recipient;
    uint256 public expiration;
    
    constructor(address _recipient, uint256 _expiration) payable {
        sender = msg.sender;
        recipient = _recipient;
        expiration = block.timestamp + _expiration;
    }
    
    function close(uint256 amount, bytes memory signature) external {
        require(block.timestamp >= expiration || msg.sender == sender);
        
        bytes32 message = keccak256(abi.encodePacked(amount));
        require(recoverSigner(message, signature) == sender);
        
        payable(recipient).transfer(amount);
        payable(sender).transfer(address(this).balance);
    }
    
    function recoverSigner(bytes32 message, bytes memory sig)
        internal
        pure
        returns (address)
    {
        uint8 v;
        bytes32 r;
        bytes32 s;
        
        (v, r, s) = splitSignature(sig);
        
        return ecrecover(message, v, r, s);
    }
    
    function splitSignature(bytes memory sig)
        internal
        pure
        returns (uint8, bytes32, bytes32)
    {
        require(sig.length == 65);
        
        bytes32 r;
        bytes32 s;
        uint8 v;
        
        assembly {
            r := mload(add(sig, 32))
            s := mload(add(sig, 64))
            v := byte(0, mload(add(sig, 96)))
        }
        
        return (v, r, s);
    }
}
```

## 🔄 Upgradeable Contracts

### OpenZeppelin Proxy Pattern

```solidity
// contracts/VotingV1.sol
import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";

contract VotingV1 is Initializable, OwnableUpgradeable {
    mapping(uint256 => string) public proposals;
    uint256 public proposalCount;
    
    function initialize() public initializer {
        __Ownable_init();
        proposalCount = 0;
    }
    
    function createProposal(string memory description) external onlyOwner {
        proposals[proposalCount] = description;
        proposalCount++;
    }
}
```

```solidity
// contracts/VotingV2.sol - Versión actualizada
contract VotingV2 is VotingV1 {
    mapping(uint256 => uint256) public voteCounts; // Nueva funcionalidad
    
    function vote(uint256 proposalId) external {
        require(proposalId < proposalCount, "Invalid proposal");
        voteCounts[proposalId]++;
    }
    
    function getVoteCount(uint256 proposalId) external view returns (uint256) {
        return voteCounts[proposalId];
    }
}
```

```javascript
// scripts/deploy-proxy.js
const { ethers, upgrades } = require("hardhat");

async function main() {
  // Deploy V1
  const VotingV1 = await ethers.getContractFactory("VotingV1");
  const proxy = await upgrades.deployProxy(VotingV1, [], {
    initializer: "initialize"
  });
  
  console.log("Proxy deployed to:", proxy.address);
}

// scripts/upgrade.js
async function main() {
  const VotingV2 = await ethers.getContractFactory("VotingV2");
  const upgraded = await upgrades.upgradeProxy(PROXY_ADDRESS, VotingV2);
  
  console.log("Contract upgraded");
}
```

## 📈 Métricas y Monitoreo

### Dashboard de Métricas

```javascript
// utils/metrics.js
const { ethers } = require("ethers");

class VotingMetrics {
    constructor(contractAddress, abi, provider) {
        this.contract = new ethers.Contract(contractAddress, abi, provider);
    }
    
    async getProposalMetrics() {
        const totalProposals = await this.contract.totalPropuestas();
        const metrics = [];
        
        for (let i = 0; i < totalProposals; i++) {
            const proposal = await this.contract.obtenerPropuesta(i);
            metrics.push({
                id: i,
                description: proposal.descripcion,
                votesFor: proposal.votosFavor.toNumber(),
                votesAgainst: proposal.votosContra.toNumber(),
                totalVotes: proposal.votosFavor.add(proposal.votosContra).toNumber(),
                participationRate: this.calculateParticipationRate(proposal),
                active: proposal.activa
            });
        }
        
        return metrics;
    }
    
    async getEventMetrics(fromBlock = 0) {
        const voteEvents = await this.contract.queryFilter(
            this.contract.filters.VotoEmitido(),
            fromBlock
        );
        
        const dailyVotes = {};
        
        voteEvents.forEach(event => {
            const date = new Date(event.args.timestamp * 1000).toDateString();
            dailyVotes[date] = (dailyVotes[date] || 0) + 1;
        });
        
        return {
            totalVotes: voteEvents.length,
            dailyVotes,
            uniqueVoters: new Set(voteEvents.map(e => e.args.votante)).size
        };
    }
    
    calculateParticipationRate(proposal) {
        // Lógica para calcular tasa de participación
        const totalVotes = proposal.votosFavor.add(proposal.votosContra);
        // Assumiendo 1000 votantes elegibles como ejemplo
        return (totalVotes.toNumber() / 1000) * 100;
    }
}

module.exports = VotingMetrics;
```

## ⚙️ Ejercicios Prácticos

1. **Documentar contrato**: Agrega documentación NatSpec completa a un contrato existente
2. **Optimizar gas**: Identifica y optimiza un contrato para reducir costos de gas
3. **Implementar proxy**: Crea un contrato upgradeable usando OpenZeppelin
4. **Métricas dashboard**: Desarrolla un dashboard para visualizar métricas del contrato

## 🎯 Checklist de Buenas Prácticas

### Documentación
- ✅ NatSpec comments para todas las funciones públicas
- ✅ README con instrucciones de despliegue
- ✅ Diagramas de arquitectura
- ✅ Ejemplos de uso
- ✅ Guías de troubleshooting

### Escalabilidad
- ✅ Optimización de estructuras de datos
- ✅ Batch operations cuando sea posible
- ✅ Considerar Layer 2 solutions
- ✅ Implementar upgradeability si es necesario
- ✅ Monitoreo de métricas de performance

## 🔗 Recursos Adicionales

- [Solidity NatSpec Documentation](https://docs.soliditylang.org/en/latest/natspec-format.html)
- [OpenZeppelin Upgrades](https://docs.openzeppelin.com/upgrades-plugins/1.x/)
- [Polygon Documentation](https://docs.polygon.technology/)
- [Arbitrum Documentation](https://developer.arbitrum.io/)
- [Gas Optimization Techniques](https://gist.github.com/hrkrshnn/ee8fabd532058307229d65dcd5836ddc)

---

**Anterior**: [05. Integración](./05-integracion.md) | **Siguiente**: [07. Evaluación](./07-evaluacion.md) | **Inicio**: [README](../README.md)