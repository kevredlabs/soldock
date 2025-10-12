# Skelz – Bringing trust to the software supply chain: sign, verify, and secure container images on Solana

[![Crates.io](https://img.shields.io/crates/v/skelz.svg)](https://crates.io/crates/skelz)
[![Documentation](https://docs.rs/skelz/badge.svg)](https://docs.rs/skelz)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## Pitch

- **Problem**: The software supply chain is fragile (compromised dependencies/images, centralized registries that can be corrupted or censored). How can we guarantee that the deployed image is exactly the one that was built and approved?
- **Solution**: An on-chain (Solana) registry of image digests and signatures; attestations (SBOM, provenance) stored on IPFS/Arweave; immutable policies; automatic verification in Kubernetes via an Admission Controller.
- **Workflow**: CI builds → Cosign signs → On-chain publication (digest, signatures, IPFS CIDs) → Admission Controller compares digest/signatures/policies → ALLOW/DENY.
- **Benefits**: Security, transparency, compliance, interoperability (Sigstore/Cosign), censorship resistance.

## MVP scope (hackathon)

- ✅ **Solana smart contract** (registry of digests, signatures, CIDs, policies)
- ✅ **CLI `sign`/`verify`** (Rust) - **Published on crates.io**
- 🚧 **K8s Admission Controller** (Rust) for ALLOW/DENY
- 🚧 **E2E demo** (KinD): build → sign → publish → deploy

## Current status

### ✅ **Completed**
- **CLI Tool**: Fully functional with `config`, `sign`, `verify`, and `registry` commands
- **Solana Integration**: Anchor-based smart contract for signature storage
- **OCI Registry Support**: GitHub Container Registry (GHCR) integration
- **Configuration Management**: TOML-based config with environment variable overrides
- **Multi-cluster Support**: Devnet, testnet, mainnet-beta, and localnet
- **Documentation**: Complete API documentation and user guides
- **Testing**: Unit tests and integration tests
- **CI/CD**: Automated publishing to crates.io

### 🚧 **In Progress**
- **Admission Controller**: Kubernetes validating webhook for automatic verification
- **E2E Demo**: Complete workflow demonstration with KinD

### 📋 **Planned**
- **SDK**: Shared Rust library for common functionality
- **Infrastructure**: K8s manifests, Helm charts, KinD setup
- **Examples**: Complete E2E flows and use cases

## Project structure

```text
contracts/              # ✅ Solana program (Anchor/Rust)
├── programs/skelz/     # Smart contract for signature storage
└── tests/              # Contract integration tests

cli/                    # ✅ CLI sign/verify (Rust) - Published on crates.io
├── src/                # Source code (lib.rs + main.rs)
├── tests/              # Integration tests
├── examples/           # Usage examples
└── README.md           # Complete CLI documentation

admission-controller/   # 🚧 K8s webhook (Rust) - In development
sdk/                    # 📋 Shared clients (Rust) - Planned
infra/                  # 📋 K8s manifests, Helm, KinD - Planned
docs/                   # ✅ Documentation and ADRs
├── diagram.md          # Architecture diagrams
└── adrs/               # Architecture Decision Records

test/                   # ✅ Integration/E2E tests
└── cypherpunk-demo-image/  # Demo image and signatures
```

## Quick start

### Install the CLI
```bash
# From crates.io (recommended)
cargo install skelz

# Or from source
git clone https://github.com/kevredlabs/skelz-hackathon
cd skelz-hackathon/cli
cargo install --path .
```

### Basic usage
```bash
# Initialize configuration
skelz config init

# Set up credentials
export GHCR_USER=your-github-username
export GHCR_TOKEN=ghp_your_github_token

# Sign a Docker image
skelz sign ghcr.io/username/repo@sha256:abc123def456...

# Verify an image signature
skelz verify ghcr.io/username/repo@sha256:abc123def456... --signer <signer-public-key>
```

### Development setup
```bash
# Clone and build
git clone https://github.com/kevredlabs/skelz-hackathon
cd skelz-hackathon

# Build CLI
cd cli
make setup
make build
make test

# Build contracts
cd ../contracts
make build
make test
```

## Components

### 🔧 **CLI Tool** (`cli/`)
- **Status**: ✅ Complete and published on crates.io
- **Features**: Image signing, verification, configuration management, GHCR integration
- **Documentation**: [cli/README.md](cli/README.md)
- **Installation**: `cargo install skelz`

### 📜 **Solana Smart Contract** (`contracts/`)
- **Status**: ✅ Complete
- **Features**: PDA-based signature storage, Anchor framework
- **Program ID**: `4uw8DwTRdUMwGmbNrK5GZ5kgdVtco4aUaTGDnEUBrYKt`
- **Documentation**: [contracts/README.md](contracts/README.md)

### 🎛️ **Admission Controller** (`admission-controller/`)
- **Status**: 🚧 In development
- **Features**: Kubernetes validating webhook for automatic image verification
- **Technology**: Rust with kube-rs

### 📚 **Documentation** (`docs/`)
- **Status**: ✅ Complete
- **Contents**: Architecture diagrams, ADRs, technical specifications
- **Files**: [diagram.md](docs/diagram.md), [adrs/](docs/adrs/)

## CI/CD and image signing

The project includes GitHub Actions workflows for container image signing:

### Legacy Signing (OIDC-based)
- `legacy-signing-and-verifying.yml`: Signs images using GitHub OIDC tokens
- Automatically builds the demo image on each push
- Signs with Cosign using GitHub's OIDC infrastructure
- Publishes to GitHub Container Registry (ghcr.io)

### Private Key Signing
- `private-key-signing-and-verifying.yml`: Signs images using private keys
- Provides more control over the signing process
- Requires configuration of GitHub secrets (`COSIGN_PRIVATE_KEY`, `COSIGN_PASSWORD`)

Both workflows support the amd64 architecture and the image is available at: `ghcr.io/kevredlabs/skelz:latest`

## Conventions

- Short-lived branches `feat/*`, `fix/*`, merges into `main` (protected)
- Commits: Conventional Commits (`feat:`, `fix:`, `chore:` …)
- Formatting/lints: Rust (`rustfmt` / `clippy`)
- Hooks: `pre-commit`

## Prerequisites (dev)

- Docker, Cosign, Rust/Cargo, Solana CLI, Kind/kubectl
- GitHub Container Registry access (for testing)

## Architecture

The system follows a decentralized approach:

1. **Build Phase**: CI builds container images
2. **Sign Phase**: Images are signed with Solana signatures using the CLI
3. **Storage Phase**: Signatures and metadata are stored on Solana blockchain
4. **Verify Phase**: Admission controller verifies signatures before deployment
5. **Deploy Phase**: Only verified images are allowed in Kubernetes

See [docs/diagram.md](docs/diagram.md) for detailed architecture diagrams.

## Contributing

Contributions are welcome! Please read our contributing guidelines and submit pull requests to our [GitHub repository](https://github.com/kevredlabs/skelz-hackathon).

## License

MIT — see [LICENSE](LICENSE).

## Acknowledgments

- [Solana](https://solana.com/) for the blockchain infrastructure
- [Anchor](https://www.anchor-lang.com/) for the Solana program framework
- [GitHub Container Registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry) for OCI registry support
- [Cosign](https://github.com/sigstore/cosign) for container signing standards

---

**Skelz** - Decentralized container image registry for the Solana blockchain 🚀