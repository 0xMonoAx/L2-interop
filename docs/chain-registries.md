# Chain Registry

## Context and Motivation

Historically, the Ethereum ecosystem has relied on off-chain registries, such as [`ethereum-lists/chains`](https://github.com/ethereum-lists/chains) or [`chainlist.org`](https://chainlist.org), to maintain the correspondence between chain names and their identifiers (chainId).

This approach, while practical, has two fundamental limitations:
1.  It requires trust in external repositories or servers.
2.  The information cannot be queried natively by smart contracts.

## The Objective of the Chain Registry

The Chain Registry addresses these limitations through a verifiable and standardized on-chain registry implemented in the `ChainResolver.sol` contract.

This system:
-   Stores chain data directly on the blockchain.
-   Allows native queries from smart contracts.
-   Establishes a source of truth that is resistant to censorship and spoofing.
-   Is governed by an open model, initially controlled by a multisig and designed to evolve toward community consensus.

## Architecture

The `ChainResolver` contract is based on the previous `L2Resolver` implementation, developed by Wonderland and Unruggable Labs.

| Concept         | Description                                                                                                                              |
| --------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| **L2Resolver**  | A minimalist resolver that associates `labelhashes` (keccak256 of the chain name) with their identifier and name, allowing for direct and reverse lookups. |
| **ChainResolver** | Extends `L2Resolver`, adapting it to a global multichain registry and aligning it with the ERC-7930 standard.                               |

Both share a consistent mapping structure:
-   **Forward mapping**: `labelhash` → `ChainData(chainId, chainName)`
-   **Reverse mapping**: `chainId` → `labelhash`
-   **Ownership mapping**: `labelhash` → `owner`

## Ownership and Permissions

The global owner of the contract (by default, a multisig address) can register new chains via:
`register(chainName, owner, chainId)`

Each registration has its own owner (`labelOwner`), who is responsible for managing the data for that chain. Owners can delegate permissions to operators (`setOperator`) to allow for authorized updates. Verifications are performed with `_authenticateCaller`, which ensures that only the owner or their operators can modify a record.

## Identifier Format: ERC-7930

The Chain Registry uses the Chain Identifier defined in **ERC-7930**, a compact format that encapsulates two components:
-   Chain type
-   Short reference

This format is designed to be interoperable with the CAIP-2 standard, allowing for uniform identification for both EVM and non-EVM chains. In practice, the identifiers are short (≈40 bytes), but the full ERC-7930 specification supports up to 263 bytes for more complex cases.

## Core Operations

| Function                             | Description                                            |
| ------------------------------------ | ------------------------------------------------------ |
| `register(chainName, owner, chainId)` | Creates a new chain registration (only callable by the global owner). |
| `setRecord(labelHash, chainId, chainName)` | Updates a chain's data (owner or its operator).       |
| `chainId(labelHash)`                 | Returns a chain's identifier (direct lookup).          |
| `chainName(chainId)`                 | Returns the chain's name (reverse lookup).             |
| `setLabelOwner(labelHash, owner)`    | Transfers the ownership of a label.                    |
| `setOperator(operator, isOperator)`  | Manages delegated permissions.                         |

## Conceptually: An ENS Layer for Chains

The Chain Registry functions as a specialized ENS (Ethereum Name Service) for chains. Instead of resolving names to addresses, it resolves chain names to standardized identifiers (ERC-7930).

This offers:
-   Censorship resistance
-   On-chain verifiability
-   Multichain interoperability
-   Compatibility with existing ENS resolvers

## Benefits

-   Smart contracts can check the validity of a chain without relying on external sources.
-   Adopts CAIP-2 and ERC-7930, ensuring compatibility between EVM and non-EVM ecosystems.
-   Starts under a multisig and evolves toward community control.


