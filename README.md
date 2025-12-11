# Anchor Mint-Vault-Swap

A Solana smart contract system for minting, vaulting, and swapping Metaplex Core NFT collections and assets. This project demonstrates advanced Solana development patterns including cross-program invocations (CPI), program-derived addresses (PDA), and secure NFT management.

## Overview

This project provides a complete NFT lifecycle management system on Solana, automating the creation and management of Metaplex Core Collections and assets through three integrated smart contracts:

1. **Minting**: Create collections and mint individual assets using the Metaplex Core standard
2. **Vaulting**: Securely lock NFTs in a program-controlled vault with access control
3. **Swapping**: Exchange vaulted collections for SOL tokens seamlessly

## Features

- Create Metaplex Core NFT collections with customizable metadata
- Mint individual assets from collections
- Secure vault storage with program-controlled authority
- Atomic swap functionality for converting NFTs to SOL
- Protocol-level fee management
- Treasury account for fee collection
- Admin-controlled initialization

## Fee Structure

- **Locking Fee**: 1 SOL (paid when securing assets in the vault)
- **Purchase/Swap Fee**: 2 SOL (paid when exchanging vaulted assets for SOL)

## Architecture

The project consists of two primary Anchor programs:

### 1. Mint-Vault Program

The core program responsible for NFT lifecycle management.

**Program ID**: `6VVXJ3hHsXn8kFqCWRPT6VeigbGkcHkUZhhopritHdMi`

**Instructions:**
- `init`: Initialize protocol configuration and asset manager accounts
- `create_collection`: Create a new Metaplex Core collection
- `mint_asset`: Mint an asset from an existing collection
- `lock_in_vault`: Transfer and lock an asset in the program vault
- `purchase`: Purchase and unlock an asset from the vault

### 2. Swap Program

Facilitates atomic swaps between vaulted NFTs and SOL.

**Program ID**: `xnrMV3UCFqDefZW3oEY4QGVX8fFmopJGETwWDSfCiUd`

**Instructions:**
- `swap`: Execute a CPI call to purchase a vaulted asset using SOL

## Technical Requirements

### Development Environment

```bash
# Solana CLI
solana-install init 1.18.8
# Version: solana-cli 1.18.8 (src:e2d34d37; feat:3469865029, client:SolanaLabs)

# Anchor Framework
avm use 0.30.1
# Version: anchor-cli 0.30.1

# Rust Toolchain (use stable or 1.75.0 for compatibility)
rustup default stable
```

**Note**: Due to Rust toolchain compatibility with Anchor 0.30.1, you may encounter IDL build issues with `anchor build` on newer Rust versions. If this occurs, the programs can still be built successfully using `cargo build --release`, and tests can be run with the test scripts.

### Dependencies

**Rust Dependencies:**
- `anchor-lang`: 0.30.1
- `mpl-core`: 0.7.2
- `solana-program`: 1.18.17

**TypeScript/JavaScript:**
- `@coral-xyz/anchor`: 0.30.1
- `@metaplex-foundation/umi`: ^0.9.2
- `@solana/web3.js`: ^1.98.4

## Installation

1. Clone the repository:
```bash
git clone <repository-url>
cd anchor-mint-vault-swap
```

2. Install dependencies:
```bash
yarn install
```

3. Build the programs:
```bash
anchor build
```

## Testing

The test suite is split into two files to comply with Solana RPC rate limits:

### Mint-Vault Tests
Tests collection creation, asset minting, and vault locking:
```bash
yarn test:mint-vault
```

### Swap Tests
Tests the asset purchase and swap functionality:
```bash
yarn test:swap
```

### Running All Tests
```bash
anchor test
```

**Important**: Tests must be run individually to avoid RPC rate limiting. Update the `uploadAssetFiles` method in `tests/utils.ts` with your local file path for asset images before running tests.

## Usage

### 1. Initialize the Protocol

**Note**: Only run this once when deploying a new program.

```typescript
await program.methods
  .init()
  .accounts({
    payer: admin.publicKey,
    assetManager,
    protocol,
    treasury: treasuryPubkey,
    coreProgram: MPL_CORE_PROGRAM_ID,
    systemProgram: SystemProgram.programId,
  })
  .signers([admin])
  .rpc();
```

### 2. Create a Collection

```typescript
await program.methods
  .createCollection({
    name: "My NFT Collection",
    uri: "https://arweave.net/metadata.json",
    items: 100, // Maximum number of assets
  })
  .accounts({
    payer: payer.publicKey,
    collection: collectionKeypair.publicKey,
    collectionData,
    coreProgram: MPL_CORE_PROGRAM_ID,
    systemProgram: SystemProgram.programId,
  })
  .signers([payer, collectionKeypair])
  .rpc();
```

### 3. Mint an Asset

```typescript
await program.methods
  .mintAsset({
    name: "Asset #1",
    uri: "https://arweave.net/asset-metadata.json",
  })
  .accounts({
    payer: payer.publicKey,
    asset: assetKeypair.publicKey,
    collection: collectionKeypair.publicKey,
    collectionData,
    coreProgram: MPL_CORE_PROGRAM_ID,
    systemProgram: SystemProgram.programId,
  })
  .signers([payer, assetKeypair])
  .rpc();
```

### 4. Lock Asset in Vault

Requires 1 SOL payment:

```typescript
await program.methods
  .lockInVault()
  .accounts({
    payer: payer.publicKey,
    treasury: treasuryPubkey,
    asset: assetKeypair.publicKey,
    collection: collectionKeypair.publicKey,
    assetManager,
    protocol,
    coreProgram: MPL_CORE_PROGRAM_ID,
    systemProgram: SystemProgram.programId,
  })
  .signers([payer])
  .rpc();
```

### 5. Purchase/Swap Asset

Requires 2 SOL payment:

```typescript
await program.methods
  .purchase()
  .accounts({
    payer: buyer.publicKey,
    buyer: buyer.publicKey,
    previousOwner: originalOwner.publicKey,
    asset: assetKeypair.publicKey,
    collection: collectionKeypair.publicKey,
    assetManager,
    protocol,
    coreProgram: MPL_CORE_PROGRAM_ID,
    systemProgram: SystemProgram.programId,
  })
  .signers([buyer])
  .rpc();
```

## Program Accounts

### Protocol
Stores global protocol configuration:
- `treasury`: Treasury account for fee collection
- `rent`: Fixed rental fee (1 SOL)

### AssetManager
Program-controlled escrow account that holds vaulted assets:
- `bump`: PDA bump seed for signing authority

### CollectionData
Tracks collection metadata:
- `bump`: PDA bump seed
- `items_available`: Remaining mintable assets
- `authority`: Collection creator
- `collection`: Collection public key

## Security Features

- **Admin-only initialization**: Only the configured admin can initialize the protocol
- **PDA-controlled vaults**: Assets are held by program-derived addresses
- **Balance validation**: Pre-transaction checks ensure sufficient funds
- **Atomic operations**: All operations are atomic and fail-safe
- **Access control**: Proper authority checks on all sensitive operations

## Development

### Project Structure
```
anchor-mint-vault-swap/
├── programs/
│   ├── mint-vault/          # Main NFT management program
│   │   └── src/
│   │       ├── instructions/ # Instruction handlers
│   │       ├── state/        # Account structures
│   │       ├── constants.rs  # Program constants
│   │       ├── error.rs      # Custom errors
│   │       └── lib.rs        # Program entry point
│   └── swap/                 # Swap facilitation program
│       └── src/
│           ├── instructions/
│           └── lib.rs
├── tests/                    # Integration tests
│   ├── mint-vault.ts
│   ├── swap.ts
│   └── utils.ts
├── Anchor.toml              # Anchor configuration
└── package.json             # Node dependencies
```

### Linting and Formatting

Format code:
```bash
yarn lint:fix
```

Check formatting:
```bash
yarn lint
```

## Deployment

1. Update `Anchor.toml` with your target cluster (devnet/mainnet-beta)
2. Ensure your wallet has sufficient SOL for deployment
3. Deploy programs:
```bash
anchor deploy
```

4. Verify deployment:
```bash
anchor idl init <PROGRAM_ID> -f target/idl/<program_name>.json
```

## Network Configuration

Current configuration (Anchor.toml):
- **Cluster**: Devnet
- **Wallet**: `~/.config/solana/id.json`
- **RPC**: https://api.apr.dev

## Error Codes

| Code | Error | Description |
|------|-------|-------------|
| 6000 | CollectionMintedOut | No more assets can be minted from collection |
| 6001 | PubkeyMismatch | Received key doesn't match expected key |
| 6002 | InsufficientLamportsForRent | Not enough SOL to cover 1 SOL locking fee |
| 6003 | InsufficientLamportsForPurchase | Not enough SOL to cover 2 SOL purchase fee |

## Contributing

Contributions are welcome! Please follow these guidelines:
1. Fork the repository
2. Create a feature branch
3. Write tests for new functionality
4. Ensure all tests pass
5. Submit a pull request

## License

MIT License - see LICENSE file for details

## Resources

- [Anchor Framework Documentation](https://www.anchor-lang.com/)
- [Metaplex Core Documentation](https://developers.metaplex.com/)
- [Solana Documentation](https://docs.solana.com/)

## Contact

For questions or support, please open an issue in the GitHub repository.
