Introducing: Interoperable Addresses
====

*Formal format specifications are written [here](/specs/addresses/cross-chain-interoperable-addresses-spec.md)*.

## The Problem: Hundreds of Chains, One Address Format

With hundreds of L2s in the Ethereum ecosystem, an address like `0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045` is no longer sufficient. Does it belong to mainnet, Arbitrum, Optimism, or another chain? Sending funds to the wrong chain can result in their loss.

To solve this, **ERC-7930** creates a unified address format that explicitly includes chain information.

## The Solution: A Standard for Two Audiences

The development of ERC-7930 was guided by a key insight: on-chain infrastructure developers (bridges, messaging protocols) and wallet developers have different needs. The former require a binary, canonical, and compact format, while the latter prioritize human readability.

Therefore, ERC-7930 was designed with a "binary-first" approach, establishing a foundation for infrastructure that also supports a readable format for users. 

## How It Works: The Two Formats

### 1. The Binary Format (for Contracts and Infrastructure)

ERC-7930 defines an efficient and unambiguous binary structure, ideal for processing by smart contracts.

```
┌─────────┬───────────┬──────────────────────┬────────────────┬───────────────┬─────────┐
│ Version │ ChainType │ ChainReferenceLength │ ChainReference │ AddressLength │ Address │
└─────────┴───────────┴──────────────────────┴────────────────┴───────────────┴─────────┘
```
- **Version**: A 2-byte identifier that allows for future upgrades to the standard.
- **ChainType**: A 2-byte value defining the blockchain family (e.g., `eip155` for EVM chains).
- **ChainReferenceLength**: A 1-byte integer specifying the length of the `ChainReference` field.
- **ChainReference**: The chain's identifier within its family (e.g., `1` for Ethereum Mainnet).
- **AddressLength**: A 1-byte integer specifying the length of the `Address` field.
- **Address**: The account address in its native binary format.

### 2. The Interoperable Name (for Humans)

For users, the binary format is represented as a human-readable string that is easy to share, with a checksum to prevent errors.

**Syntax:** `<address>@<chain>#<checksum>`

**Example (base ERC-7930):**
`0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045@eip155:1#4CA88C9C`

- `0xd8dA...`: The account address.
- `eip155:1`: The chain identifier, which follows the **CAIP-2** standard (`namespace:reference`). In this case, `eip155` is the namespace for EVM chains, and `1` is the ID for Ethereum Mainnet.
- `#4CA88C9C`: A 4-byte checksum that guarantees data integrity.

## Extending Human Readability with ERC-7828

**[ERC-7828](https://ethereum-magicians.org/t/erc-7828-chain-specific-addresses-using-ens/21930)** extends ERC-7930 to replace raw addresses and chain identifiers with human-readable names resolved via ENS.

**Example (with ERC-7828):**
`vitalik.eth@eth#5966be0f`

Here, each part serves a clear purpose:
- `vitalik.eth`: The account name, which resolves to an address (`0xd8dA...`) via ENS.
- `eth`: The chain name, which resolves to a chain identifier (`eip155:1`) via ENS.
- `#5966be0f`: The checksum, which ensures that the resolved data is correct.

## Key Advantages Over CAIP-10

While CAIP-10 was a pioneering standard, ERC-7930 was designed to overcome its limitations:
- **Efficient Binary Format:** Unlike the text-only CAIP-10, ERC-7930 provides a compact binary format optimized for use in smart contracts.
- **Future-Proof:** CAIP-10 has a 32-character limit for chain identifiers, whereas ERC-7930 has no such limit.
- **Extensible and Secure:** ERC-7930 includes a version field for future upgrades and a checksum (`#checksum`) to prevent errors.

## The Role of CAIP-350: Universal Support

ERC-7930 defines a binary "container," but it is **CAIP-350** that specifies how to serialize the addresses of different blockchains (like Bitcoin or Solana) within it. Each ecosystem defines a "CAIP-350 profile" that sets the encoding rules, allowing Interoperable Addresses to be truly universal.
