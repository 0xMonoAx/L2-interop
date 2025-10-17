# Chain-specific addresses

A core challenge in Ethereum's multichain landscape is disambiguating addresses across L2s, L3s, Lns, and other EVM ecosystems. Since a given Ethereum address can exist on any EVM chain, applications and users need a way to also specify a target chain, to prevent errors and confusion, especially in the case of smart contracts, where the code underlying an address can differ entirely between chains.

> [!NOTE]
> To address this need, the community is pushing for the adoption of **[ERC-7930: Interoperable Addresses](https://eips.ethereum.org/EIPS/eip-7930)**. This standard focuses on solving the problem from a fundamental perspective, separating the needs of on-chain infrastructure developers (who require a binary, canonical, and compact format) from those of wallet developers (who prioritize human readability).
>
> ERC-7930 proposes a binary-first format that establishes a foundation for infrastructure, while also defining a human-readable format ("Interoperable Names") for user interfaces. This dual approach allows for unifying address identification in the multichain ecosystem.
>

# Existing efforts: Context and Related Standards

The standardization of multichain addresses is the result of a joint community effort, built upon the lessons of previous proposals. To understand the current landscape, it is useful to group related standards by their scope and purpose.

## Chain Agnostic Improvement Proposals (CAIPs)
CAIPs (Chain Agnostic Improvement Proposals) are standards designed to work on any blockchain, promoting interoperability beyond the Ethereum ecosystem.

### CAIP-2: Blockchain ID Specification
[CAIP-2](https://github.com/ChainAgnostic/CAIPs/blob/main/CAIPs/caip-2.md) defines a method for identifying a blockchain in a human- and developer-readable format. It uses a `namespace:reference` format (e.g., `eip155:1` for Ethereum mainnet), allowing any blockchain to be uniquely referenced, not just those in the EVM ecosystem.

### CAIP-10: Account ID Specification
[CAIP-10](https://github.com/ChainAgnostic/CAIPs/blob/main/CAIPs/caip-10.md) is a chain-agnostic standard that defines a way to identify an account on any blockchain that complies with [CAIP-2](https://github.com/ChainAgnostic/CAIPs/blob/main/CAIPs/caip-2.md), such as `eip155:1:0xAbC...123` for Ethereum mainnet. It is not necessarily intended to be human-friendly but serves as a universal reference for multi-chain tools like Wallet Connect.

### CAIP-50: Multi-Chain Account ID Specification
[CAIP-50](./caip-50.md) proposes an alternative to CAIP-10 for identifying accounts on multiple blockchains. It uses a [multicodec](https://github.com/multiformats/multicodec/)-based format to create a compact binary identifier encoded as a `base58btc` string. The main goal is to offer a more byte-efficient representation than CAIP-10, while maintaining the ability to programmatically verify and decode the address and chain.

### CAIP-350: Interoperable Address Profiles
[CAIP-350](https://github.com/ChainAgnostic/CAIPs/blob/main/CAIPs/caip-350.md) is a companion standard to ERC-7930 that specifies profiles for different chain types. It defines how to encode and decode addresses for various chain families (e.g., EVM, Bitcoin, Cosmos), ensuring that the ERC-7930 binary format can be applied consistently across the ecosystem.

## Ethereum Improvement Proposals (EIPs/ERCs)
EIPs (Ethereum Improvement Proposals) and ERCs (Ethereum Request for Comments) are the standards that define new features and protocols within the Ethereum ecosystem.

### EIP-155: Simple Replay Attack Protection
[EIP-155](https://eips.ethereum.org/EIPS/eip-155) introduces the `CHAIN_ID` into the transaction signing process to prevent replay attacks between different EVM chains. While not an address format standard itself, it is fundamental because it establishes the numeric chain identifier used by many other standards to differentiate networks.

### ERC-3770: Chain-specific Addresses
[ERC-3770](https://eips.ethereum.org/EIPS/eip-3770) prepends a human-readable chain short name as a prefix to addresses (e.g., `eth:0xAbC...123`). A `chainID` could also be used, but it is not intended to be preferred over readable strings, as this proposal is designed primarily for UX improvements.

### ERC-7828: Chain-specific Addresses using ENS
[ERC-7828](https://ethereum-magicians.org/t/erc-7828-chain-specific-addresses-using-ens/21930) extends ERC-7930 to improve the usability of human-readable addresses. This standard focuses on integrating with the Ethereum Name System (ENS) for resolving human-readable chain and account names across different blockchains, including non-EVM ones. This allows for aliases like `alice@rollup` or `alice.rollup.eth`, which wallets can resolve on-chain to the corresponding interoperable address.

> **A Note on ERC-7785**: It is worth noting that the community explored proposals like ERC-7785, which aimed to create an on-chain registry for chain metadata. Although ERC-7785 was eventually deprecated, the discussions around it influenced the direction of current standards, leading to the adoption of ERC-7930.
>

## Ethereum Name Service Improvement Proposals (ENSIPs)
ENSIPs (ENS Improvement Proposals) focus on improving and standardizing the Ethereum Name Service (ENS) infrastructure.

### ENSIP-9: Multichain Address Resolution
[ENSIP-9](https://github.com/ensdomains/ensips/blob/master/ensips/9.md) introduces a unified way for ENS resolvers to store and return addresses for different blockchains by overloading the `addr` function. Instead of proposing a textual format like `chain:address`, ENSIP-9 leverages coin types following [SLIP-0044](https://github.com/satoshilabs/slips/blob/master/slip-0044.md). This allows resolvers to identify each blockchain by its unique coin type and store the address in its native binary form, such as a 20-byte hex for Ethereum or base58-decoded bytes for Bitcoin.

### ENSIP-11: EVM-compatible Chain Address Resolution
[ENSIP-11](https://github.com/ensdomains/ensips/blob/master/ensips/11.md) extends ENSIP-9 by introducing a dedicated range of coin types for EVM chains. This range prevents collisions with existing [SLIP-0044](https://github.com/satoshilabs/slips/blob/master/slip-0044.md) coin types.

## Comparison

The existing approaches to chain-specific addresses represent an evolution in thinking about cross-chain identification. Rather than being competing solutions, many of these standards build upon the lessons learned from previous implementations or address different aspects of the problem.

| **Feature** | ERC-3770 | CAIP-10 | CAIP-50 | ERC-7828 | ENSIP-9/ENSIP-11 | CAIP-2 | ERC-7930 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| **Scope** | Primarily a UI/UX layer standard for human-readable prefixes | A universal, machine-readable account identifier format for all blockchains | Compact, binary account ID format for all blockchains | On-chain naming integration with ENS for human-readable chain and address names | ENS resolver-level standards for storing/retrieving multichain addresses | Universal blockchain identifier (machine-readable) | Universal and extensible binary and text format for chain-specific addresses |
| **Status** | Draft (Fully defined) | Final (Fully defined) | Draft | Draft (Incomplete) | Final/Draft | Final | Last Call |
| **Format Example** | `chain:address` | `chain_namespace:chain_reference:address` | `zUJWDx...` (base58btc) | `address:chain.eth` or `address@chain.eth` | Still uses typical `.eth` format | `namespace:reference` | `<address>@<chain>#<checksum>` |
| **Human Readability (_best-case_)** | Medium | Medium | Low | High | High (from typical ENS format) | High | High |
| **Technical Compatibility** | EVM only (but extensible) | All chains | All chains | All chains (via ENS) | Blockchains part of [SLIP-0044](https://github.com/satoshilabs/slips/blob/master/slip-0044.md) or following EVM `chainId` specs | All chains | All chains (via CAIP-350 profiles) |
| **ENS Integration** | Not Required | Not Required | Not Required | Required | Required | Not Required | Supported |
| **DID Compatibility** | | | Supported | | | Not direct | Compatible |
| **Checksum Support** | Incomplete (ERC-55 only) | No | Yes (parity byte) | Yes | Yes | No | Yes |
| **Use of lists** | Yes (referencing github.com/ethereum-lists/chains) | No | No | Yes (requires on-chain registry) | Based on [SLIP-0044](https://github.com/satoshilabs/slips/blob/master/slip-0044.md) | Not direct | Indirect (via CAIP-350 profiles) |

# Additional Considerations

Unique chain identifiers rely on social consensus to avoid collisions that could break integrations. The transition from off-chain registries to on-chain methods in the Ethereum ecosystem has been widely discussed and documented [here](chain-registries.md).
