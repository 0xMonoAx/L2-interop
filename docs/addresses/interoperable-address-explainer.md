# Interoperable Addresses Explained

*The formal specification can be found in [ERC-7930](https://eips.ethereum.org/EIPS/eip-7930).*

## The Problem: Hundreds of Chains, One Address Format

With hundreds of L2s in the Ethereum ecosystem, an address like `0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045` is no longer sufficient. Does it belong to mainnet, Arbitrum, Optimism, or another chain? Sending funds to the wrong chain, especially to smart contracts whose code can vary between chains, can result in their permanent loss.

To solve this, **ERC-7930** creates a unified address format that explicitly includes chain information.

## The Solution: A Standard for Two Audiences

The development of ERC-7930 was guided by a key insight: on-chain infrastructure developers (bridges, messaging protocols) and wallet developers have different needs. The former require a binary, canonical, and compact format, while the latter prioritize human readability.

Therefore, ERC-7930 was designed with a "binary-first" approach, establishing a solid foundation for infrastructure that also supports a readable format for users.

## How It Works: The Two Formats

### 1. The Binary Format (for Contracts and Infrastructure)

ERC-7930 defines an efficient and unambiguous binary structure, ideal for processing by smart contracts.

```
┌─────────┬───────────┬──────────────────────┬────────────────┬───────────────┬─────────┐
│ Version │ ChainType │ ChainReferenceLength │ ChainReference │ AddressLength │ Address │
└─────────┴───────────┴──────────────────────┴────────────────┴───────────────┴─────────┘
```
- **Version**: A 2-byte identifier that allows for future upgrades.
- **ChainType**: A 2-byte value defining the blockchain family (e.g., `eip155` for EVM).
- **ChainReference/Length**: The chain's identifier (e.g., `1` for Ethereum) and its length.
- **Address/Length**: The account address in binary format and its length.

### 2. The Interoperable Name (for Humans)

For users, the binary format is represented as an easy-to-read text string with a checksum to prevent errors.

**Syntax:** `<address>@<chain>#<checksum>`

**Example (EVM):**
`0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045@eip155:1#4CA88C9C`

**Example (Non-EVM):**
`MJKqp326RZCHnAAbew9MDdui3iCKWco7fsK9sVuZTX2@solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdpKuc147dw2N9d#88835C11`

## Extending Readability with ERC-7828

**[ERC-7828](https://ethereum-magicians.org/t/erc-7828-chain-specific-addresses-using-ens/21930)** extends ERC-7930 to replace addresses and chain identifiers with human-readable names resolved through ENS.

**Example:** `vitalik.eth@eth#5966be0f`
- `vitalik.eth` resolves to an address.
- `eth` resolves to a chain identifier (`eip155:1`).
- The `checksum` ensures the integrity of the resolved data.

## Compatibility with Existing Standards

### CAIP-10
ERC-7930 is **fully compatible** with CAIP-10. Both share the same data (`address` and `chainId`). However, ERC-7930 introduces key improvements:
- **Binary Format:** Optimized for smart contracts, unlike the text-only CAIP-10.
- **No Length Limit:** CAIP-10 limits chain identifiers to 32 characters, while ERC-7930 has no such limit, making it future-proof.
- **Security and Extensibility:** It includes a checksum and a version field.

### CAIP-50
They are **incompatible**. CAIP-50 uses a monolithic format (multicodec), whereas ERC-7930 is modular, separating the "container" (binary format) from its "content" (defined by CAIP-350).

### Decentralized Identifiers (DID)
ERC-7930 is compatible with the principles of the W3C DID standard (globally unique, verifiable, and no central registry), but it still needs a "DID Method" specification to be fully compliant.

## The Role of CAIP-350: The Instruction Manual for Universal Support

We've seen that ERC-7930 provides a flexible binary format, a kind of universal "box" or "container" for addresses. But if the format is generic, how does a wallet or application know what the bytes in the `ChainReference` and `Address` fields mean for a specific blockchain? An Ethereum address is structured differently from a Bitcoin or Solana address.

This is where **CAIP-350** comes in. It acts as a companion standard, an "instruction manual" that defines the precise serialization rules for different chain families.

- **ERC-7930 defines the *container***: It specifies a generic binary structure: `Version | ChainType | ChainReferenceLength | ChainReference | AddressLength | Address`. It provides the universal package but is agnostic about its content.
- **CAIP-350 defines the *content***: It indicates *what* the data inside the container means for a specific chain type (`namespace`). For each ecosystem (EVM, Solana, Bitcoin, etc.), a "CAIP-350 profile" is created that details:
  - The unique 2-byte `ChainType` that identifies that chain family.
  - The exact rules for converting text addresses (e.g., Base58) to their canonical binary representation.
  - The rules for converting chain identifiers to their binary format.

In short, **ERC-7930 is the box, and CAIP-350 is the set of instructions on what to put in the box and how to pack it**. This modularity allows ERC-7930 to support any new blockchain without needing to modify the ERC-7930 standard itself. A new CAIP-350 profile simply needs to be created for that new chain, making the system truly extensible and future-proof.
