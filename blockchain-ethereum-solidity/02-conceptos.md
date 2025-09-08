# 02. Conceptos Fundamentales de Blockchain

## 🌟 ¿Qué es Blockchain?

Blockchain es una tecnología de registro distribuido que mantiene una lista de registros (bloques) enlazados y asegurados mediante criptografía. Cada bloque contiene un hash criptográfico del bloque anterior, una marca de tiempo y datos de transacciones.

### Características clave:
- **Descentralización**: No existe una autoridad central
- **Inmutabilidad**: Los datos no pueden ser alterados fácilmente
- **Transparencia**: Todas las transacciones son visibles
- **Consenso**: La red debe ponerse de acuerdo sobre el estado

## 🔗 Componentes de un Bloque

```
┌─────────────────────────────────────┐
│ HEADER DEL BLOQUE                   │
├─────────────────────────────────────┤
│ • Hash del bloque anterior          │
│ • Merkle root                       │
│ • Timestamp                         │
│ • Nonce                             │
├─────────────────────────────────────┤
│ TRANSACCIONES                       │
│ • Transacción 1                     │
│ • Transacción 2                     │
│ • ...                               │
│ • Transacción N                     │
└─────────────────────────────────────┘
```

## 🤝 Mecanismos de Consenso

### Proof of Work (PoW)
- Los mineros compiten resolviendo puzzles criptográficos
- Requiere poder computacional significativo
- Usado por Bitcoin

### Proof of Stake (PoS)
- Los validadores son elegidos basándose en su participación
- Más eficiente energéticamente
- Usado por Ethereum 2.0

## 🔐 Criptografía en Blockchain

### Hash Functions
```javascript
// Ejemplo conceptual
SHA256("Hola Blockchain") = "a1b2c3d4e5f6..."
SHA256("Hola blockchain") = "x9y8z7w6v5u4..." // Completamente diferente
```

### Firmas Digitales
- Clave privada para firmar
- Clave pública para verificar
- Garantiza autenticidad y no repudio

## 🌐 Tipos de Blockchain

### Públicas
- Abiertas a todos
- Completamente descentralizadas
- Ejemplos: Bitcoin, Ethereum

### Privadas
- Controladas por una organización
- Acceso restringido
- Ejemplos: Hyperledger Fabric

### Consorte
- Controladas por un grupo de organizaciones
- Semi-descentralizadas
- Ejemplos: R3 Corda

## 💰 Tokens y Criptomonedas

### Monedas Nativas
- Tienen su propia blockchain
- Ejemplos: Bitcoin (BTC), Ether (ETH)

### Tokens
- Construidos sobre blockchains existentes
- Ejemplos: USDC, LINK (en Ethereum)

## ⚙️ Ejercicio Práctico

1. **Investigación**: Busca un explorador de blockchain (etherscan.io) y explora:
   - Un bloque reciente
   - Una transacción específica
   - El hash de un bloque

2. **Reflexión**: Escribe un párrafo explicando cómo la inmutabilidad de blockchain podría beneficiar a un sector específico (salud, educación, logística, etc.)

## 🎯 Buenas Prácticas

- Comprende las limitaciones de cada tipo de blockchain
- Considera los costos de transacción y velocidad
- Evalúa la seguridad vs. escalabilidad
- Mantente actualizado sobre nuevos consensos

## 🔗 Recursos Adicionales

- [Bitcoin Whitepaper](https://bitcoin.org/bitcoin.pdf)
- [Ethereum Whitepaper](https://ethereum.org/whitepaper/)
- [Blockchain Explorer - Etherscan](https://etherscan.io)

---

**Anterior**: [01. Introducción](./01-introduccion.md) | **Siguiente**: [03. Ethereum y Contratos Inteligentes](./03-ethereum-smart-contracts.md) | **Inicio**: [README](../README.md)