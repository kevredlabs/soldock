# Skelz CLI

[![Crates.io](https://img.shields.io/crates/v/skelz.svg)](https://crates.io/crates/skelz)
[![Documentation](https://docs.rs/skelz/badge.svg)](https://docs.rs/skelz)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

A decentralized container image registry CLI for Solana blockchain. Sign and verify Docker images using Solana signatures and upload proofs to OCI registries like GitHub Container Registry (GHCR).

## 🚀 Features

- **🔐 Image Signing**: Sign Docker images with Solana blockchain signatures
- **✅ Signature Verification**: Verify image signatures against Solana blockchain
- **📦 OCI Registry Integration**: Upload and retrieve signature proofs from GHCR
- **⚙️ Configuration Management**: Flexible configuration with environment variable overrides
- **🌐 Multi-cluster Support**: Support for devnet, testnet, mainnet-beta, and localnet
- **🔧 CLI & Library**: Use as command-line tool or integrate as Rust library

## 📦 Installation

### From crates.io (recommended)
```bash
cargo install skelz
```

### From source
```bash
git clone https://github.com/kevredlabs/skelz-hackathon
cd skelz-hackathon/cli
cargo install --path .
```

## 🏃 Quick Start

### 1. Initialize configuration
```bash
skelz config init
```

### 2. Set up GitHub Container Registry credentials
```bash
export GHCR_USER=your-github-username
export GHCR_TOKEN=ghp_your_github_token
```

### 3. Sign a Docker image
```bash
skelz sign ghcr.io/username/repo@sha256:abc123def456...
```

### 4. Verify an image signature
```bash
skelz verify ghcr.io/username/repo@sha256:abc123def456... --signer <signer-public-key>
```

## 📖 Usage

### Global flags
- `-v` / `-vv`: Increase verbosity (uses `tracing` under the hood)

### Commands

#### `config` - Configuration management
```bash
# Initialize configuration
skelz config init [--output PATH] [--force] [--cluster CLUSTER] [--rpc-url URL] [--keypair PATH]

# Get configuration values
skelz config get [KEY]

# Set configuration values
skelz config set KEY VALUE
```

**Configuration keys:**
- `cluster`: Solana cluster (devnet, testnet, mainnet-beta, localnet)
- `rpc_url`: Solana RPC endpoint URL
- `keypair_path`: Path to Solana keypair file
- `commitment`: Solana commitment level (processed, confirmed, finalized)
- `ghcr_user`: GitHub Container Registry username (optional)
- `ghcr_token`: GitHub Container Registry token (optional)

**Examples:**
```bash
# Initialize with devnet
skelz config init --cluster devnet

# Set custom RPC URL
skelz config set rpc_url https://api.devnet.solana.com

# Get current configuration
skelz config get
```

#### `sign` - Sign Docker images
```bash
skelz sign IMAGE_REFERENCE [--rpc-url URL] [--keypair PATH]
```

**Requirements:**
- Image reference must be canonical with digest: `ghcr.io/username/repo@sha256:abc123...`
- Only GitHub Container Registry (GHCR) is supported
- Requires valid Solana keypair and GHCR credentials

**Examples:**
```bash
# Sign an image
skelz sign ghcr.io/username/repo@sha256:abc123def456...

# Sign with custom RPC and keypair
skelz sign ghcr.io/username/repo@sha256:abc123def456... \
  --rpc-url https://api.devnet.solana.com \
  --keypair ~/.config/solana/id.json
```

#### `verify` - Verify image signatures
```bash
skelz verify IMAGE_REFERENCE --signer SIGNER_PUBLIC_KEY [--rpc-url URL]
```

**Examples:**
```bash
# Verify an image signature
skelz verify ghcr.io/username/repo@sha256:abc123def456... \
  --signer 4uw8DwTRdUMwGmbNrK5GZ5kgdVtco4aUaTGDnEUBrYKt
```

#### `registry` - Registry operations
```bash
# Login to GitHub Container Registry
skelz registry login [--registry REGISTRY] [--username USERNAME]
```

**Examples:**
```bash
# Login to GHCR
skelz registry login

# Login to custom registry
skelz registry login --registry ghcr.io --username my-username
```

## ⚙️ Configuration

### Configuration file location
- **Default**: `~/.config/skelz/config.toml` (XDG-compliant)
- **Override**: Use `--output` flag with `config init`

### Environment variables
- `SOLANA_RPC_URL`: Solana RPC endpoint
- `SOLANA_KEYPAIR`: Path to Solana keypair file
- `GHCR_USER`: GitHub Container Registry username
- `GHCR_TOKEN`: GitHub Container Registry token
- `XDG_CONFIG_HOME`: Custom config directory

### Configuration resolution order
1. Command-line arguments (highest priority)
2. Environment variables
3. Configuration file
4. Default values (lowest priority)

### Example configuration file
```toml
cluster = "devnet"
rpc_url = "https://api.devnet.solana.com"
keypair_path = "~/.config/skelz/id.json"
commitment = "confirmed"
ghcr_user = "my-github-username"  # Optional
ghcr_token = "ghp_xxxxxxxxxxxx"   # Optional (use env vars instead)
```

## 🔧 As a Rust Library

Add to your `Cargo.toml`:
```toml
[dependencies]
skelz = "0.1.0"
```

### Example usage
```rust
use skelz::{SkelzConfig, sign_image_with_oci, verify_image_signature};
use anyhow::Result;

#[tokio::main]
async fn main() -> Result<()> {
    let config = SkelzConfig::default();
    
    // Sign an image
    let signature = sign_image_with_oci(
        "ghcr.io/username/repo@sha256:abc123...",
        &config,
        "github-username",
        "github-token"
    ).await?;
    
    println!("Image signed: {}", signature);
    
    // Verify an image
    verify_image_signature(
        "ghcr.io/username/repo@sha256:abc123...",
        "expected-signer-pubkey",
        &config,
        "github-username",
        "github-token"
    ).await?;
    
    println!("Image verified successfully!");
    Ok(())
}
```

## 🧪 Development

### Prerequisites
- Rust 1.70+ (stable)
- Solana CLI tools
- Docker (for testing with images)

### Build and test
```bash
# Build
cargo build --release

# Run tests
cargo test

# Run integration tests
cargo test --test integration

# Lint
cargo clippy

# Format
cargo fmt
```

### Using Makefile
```bash
# Build, test, and lint
make build
make test
make lint
```

## 🔒 Security

### Credentials management
- **Never commit** private keys, tokens, or passwords to the repository
- Use environment variables for sensitive data
- Configuration files with credentials should be in `.gitignore`

### Keypair security
- Solana keypairs are stored locally and never transmitted
- Use `~/.config/skelz/id.json` for default keypair location
- Consider using hardware wallets for production use

### Network security
- All Solana RPC calls use HTTPS in production
- Program IDs are public identifiers (not sensitive)
- Transaction hashes are public blockchain data

## 🌐 Supported Networks

- **Devnet**: `https://api.devnet.solana.com` (default)
- **Testnet**: `https://api.testnet.solana.com`
- **Mainnet**: `https://api.mainnet-beta.solana.com`
- **Localnet**: `http://127.0.0.1:8899`

## 📚 Documentation

- **API Documentation**: [docs.rs/skelz](https://docs.rs/skelz)
- **Source Code**: [GitHub](https://github.com/kevredlabs/skelz-hackathon)
- **Issues**: [GitHub Issues](https://github.com/kevredlabs/skelz-hackathon/issues)

## 🤝 Contributing

Contributions are welcome! Please read our contributing guidelines and submit pull requests to our [GitHub repository](https://github.com/kevredlabs/skelz-hackathon).

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- [Solana](https://solana.com/) for the blockchain infrastructure
- [Anchor](https://www.anchor-lang.com/) for the Solana program framework
- [GitHub Container Registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry) for OCI registry support

---

**Skelz** - Decentralized container image registry for the Solana blockchain 🚀