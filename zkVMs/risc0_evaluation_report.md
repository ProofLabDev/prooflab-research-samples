# ZKVM Evaluation Report: Risc0 (v1.0.0-731-g2384a9589)

## Basic Information
- **Name** - Risc0
- **Team Affiliation** - RiscZero Inc.
- **Repositories Considered:**
  - **risc0** - Core Risc0 zkVM implementation
    - **Version Number** - v1.0.0-731-g2384a9589
    - **Repository URL** - https://github.com/risc0/risc0
    - **Commit** - 2384a958948366f9181032d4f9791e295df4e98d
  - **risc0-solana** - Solana integration and tooling  
    - **Version Number** - 0af1d39
    - **Repository URL** - https://github.com/risc0/risc0-solana
    - **Commit** - 0af1d39ecde3f9a12d34bc78084a31f1cc6c59ba
- **License Type** - Apache 2.0[^1][^2]

[^1]: [risc0 LICENSE file](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/LICENSE)
[^2]: [risc0-solana LICENSE file](https://github.com/risc0/risc0-solana/blob/0af1d39ecde3f9a12d34bc78084a31f1cc6c59ba/LICENSE)

## Technical Overview

### How It Works

Risc0 is a zero-knowledge virtual machine (zkVM) that executes RISC-V programs and produces verifiable cryptographic proofs of their correct execution. The system implements a three-layer recursive architecture that transforms execution traces into increasingly compact proof formats, culminating in blockchain-verifiable receipts.

**Design Philosophy and Architecture**

Risc0's architecture prioritizes practical deployment through a modular, performance-optimized design. The system separates execution from proving, allowing programs to run at near-native speeds during development while generating cryptographic proofs for production deployment[^9]. The design emphasizes developer familiarity by supporting standard RISC-V toolchains and Rust development workflows, enabling existing code to run in the zkVM with minimal modifications.

**Execution Model**

Programs compile to standard RISC-V RV32IM binaries and execute within a 32-bit memory space organized in 1KB pages[^10]. The VM implements a complete RISC-V execution environment including registers, program counter, and memory management, with specialized system calls for guest-host communication[^11]. The execution model supports continuations, allowing long-running programs to be segmented across multiple proving sessions while maintaining cryptographic integrity of the complete execution trace.

**Trace Generation and Witness Construction** 

During execution, the VM captures a complete execution trace recording all register updates, memory accesses, and instruction executions[^12]. This trace is structured as a matrix where each row represents one execution cycle, and columns represent different aspects of the machine state. The witness construction process transforms this raw execution data into polynomial constraints suitable for STARK proving, with specialized handling for precompiled operations that bypass expensive constraint generation.

**Constraint System and Arithmetization**

Risc0 uses a custom STARK-based constraint system operating over the Baby Bear field (2^31 - 2^27 + 1)[^13]. The constraint system is generated using the Zirgen domain-specific language, which produces optimized constraint polynomials for RISC-V instruction semantics[^14]. The system supports lookup tables for efficient range checks and incorporates specialized circuits for cryptographic operations, significantly reducing the constraint complexity for common operations like hashing and digital signatures.

**Proving Process**

The proving pipeline consists of three stages: segment proving generates STARK proofs for individual execution segments, recursion circuits compress these proofs into succinct constant-size receipts, and optionally, Groth16 circuits produce ultra-compact blockchain receipts[^15]. Hardware acceleration via CUDA and Metal enables efficient parallel computation of number-theoretic transforms and field operations, dramatically reducing proving times from hours to minutes for typical applications.

**Verification and Deployment**

Verification occurs through smart contracts deployed on Ethereum and Solana networks[^16]. The Groth16 verifier contracts provide constant-time verification regardless of original program complexity, enabling practical on-chain verification with gas costs under 300,000 units. The system supports proof aggregation and composition, allowing multiple program executions to be batched into single verification operations.

**Key Differentiators**

Risc0 distinguishes itself through production-ready blockchain integration, comprehensive hardware acceleration with fully open-source GPU kernels, and extensive precompile support for cryptographic operations[^17]. The three-tier proving architecture provides flexibility between proof size and generation time, while the RISC-V ISA enables broad language support beyond just Rust, including C, C++, and other compiled languages.

## Architecture

### Core Architecture
- **Instruction Set Architecture** - RISC-V
- **ISA Extensions/Profile** - RV32IM[^3]
  - *Context: RISC-V 32-bit base integer instruction set (RV32I) with multiply/divide extension (M)*

### Optimization Features
- **Precompiles/Built-ins** - SHA256 | Keccak | Poseidon | Blake2 | Blake3 | ECDSA | Ed25519 | Secp256k1 | BLS12-381 pairing | BN254 pairing | Range checks | Bitwise ops | Poseidon2 | P-256 / secp256r1 | KZG verification | RSA | Modular arithmetic[^4]

### Trace Generation Model
- **Continuations Support** - Full[^5]
  - *Context: VM can pause execution and resume later with state commitment, supporting segmented proving*
- **Segmentation Model** - Configurable execution segments with proof composition[^6]
- **Default Segment Size** - 2^13 to 2^24 cycles (8K - 16M instructions)[^7]
- **Checkpoint State Components** - Program Counter | Stack Pointer | Registers | Memory State[^8]

[^3]: [RV32IM circuit implementation](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/risc0/circuit/rv32im/src/execute/rv32im.rs#L225-L290)
[^4]: [Precompiles documentation](https://dev.risczero.com/api/zkvm/precompiles)
[^5]: [Continuations support in zkVM core](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/risc0/zkvm/src/lib.rs)
[^6]: [Segmentation in circuit implementation](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/risc0/circuit/rv32im/src/lib.rs#L44-L48)
[^7]: [ZKP framework cycle range](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/risc0/zkp/src/lib.rs)
[^8]: [Binary format and memory image](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/risc0/binfmt/src/lib.rs#L33-L46)

## Proof System
- **Backend Model** - Configurable
  - *Context: Can switch between hash functions and hardware backends via HAL*

### Proof System Framework
- **Framework Name** - Custom STARK implementation built on risc0-zkp
- **Framework Version** - 3.0.2[^18]
- **Constraint Language** - Zirgen-generated constraints[^19]

### Backend Configurations

#### Default STARK Backend
- **Backend/Config Name** - STARK (default)
- **Proof System Type** - STARK
- **Arithmetization** - AIR (Algebraic Intermediate Representation)
- **Field/Curve** - Baby Bear
- **Field Size** - 31 bits
- **Field Modulus** - 2013265921 (15 * 2^27 + 1)[^20]
- **Commitment Scheme** - FRI
- **Hash Function** - Poseidon2 (default)[^21]
- **Trusted Setup Required** - None
- **Default Backend** - Yes

#### Alternative Hash Functions
- **Backend/Config Name** - STARK-SHA256
- **Hash Function** - SHA256[^22]
- **Backend/Config Name** - STARK-Blake2b  
- **Hash Function** - Blake2b[^23]

#### Groth16 Compression Backend
- **Backend/Config Name** - Groth16
- **Proof System Type** - SNARK
- **Arithmetization** - R1CS
- **Field/Curve** - BN254
- **Field Size** - 254 bits
- **Commitment Scheme** - KZG
- **Hash Function** - Inherited from STARK layer
- **Trusted Setup Required** - Universal (Powers of Tau)[^24]
- **Default Backend** - No (optional compression layer)

### Post-Quantum Security
- **Core Proof System Post-Quantum Security** - Secure[^25]
  - *Core STARK system relies only on hash functions and symmetric primitives*
- **Quantum-Vulnerable Components** - None (in default STARK mode)
  - *Groth16 compression layer uses elliptic curve cryptography but is optional*
- **Post-Quantum Alternative Available** - Yes
  - *Default STARK backend provides post-quantum security*
- **Post-Quantum Configuration** - Use STARK receipts instead of Groth16 compression
- **Quantum Security Notes** - The core zkVM provides post-quantum security through pure STARK proofs. Groth16 compression is optional and only used for ultra-compact blockchain receipts.

## Acceleration Support

### Hardware Support
- **Hardware Acceleration** - GPU-CUDA | GPU-Metal | CPU-AVX2[^26]
- **GPU Vendor Support** - NVIDIA | Apple[^27]

### GPU Code Availability and Licensing
- **GPU Code in Repository** - Yes[^28]
- **GPU Code License** - Apache 2.0[^29]
- **GPU Implementation Method** - Custom Kernels[^30]
- **GPU Code Transparency** - Full[^31]
  - *All CUDA (.cu) and Metal (.metal) kernels are publicly available*

### General Acceleration Support
- **Acceleration Open Source** - Yes[^32]
- **Acceleration License** - Apache 2.0

### Execution Distribution
- **Prover Distribution** - Multi-threaded CPU | Single GPU | Multi-GPU (single machine) | Hybrid CPU+GPU[^33]
- **Memory Requirements per Process** - 8-32 GB (varies by circuit size)
- **Recommended Process/Thread Configuration** - Auto-detected based on available hardware

### Performance Optimization
- **Assembly Backend Available** - Yes (via sppark library)[^34]
- **Shared Memory Optimization** - Yes
- **Platform-Specific Optimizations** - Linux x86_64 | Windows x86_64 | macOS x86_64 | macOS ARM64[^35]

## Verification

### Verification Targets
- **Verification Target** - On-chain EVM | On-chain Solana[^36]

### Cost Analysis  
- **EVM Verification Gas Cost** - ~270,000 (estimated for Groth16)[^37]
- **SVM Verification Compute Units** - Available via Solana verifier program[^38]
- **Cost Model Available** - No (requires benchmarking)

### Verifier Availability
- **Solidity Verifier Available** - Yes[^39]
- **Solana Verifier Available** - Yes[^40] 
- **Verifier Generation Tool** - Yes (automatic via receipt generation)[^41]

### Advanced Verification Features
- **Proof Aggregation Support** - Advanced[^42]
  - *Can aggregate multiple program executions via recursion circuits*
- **Recursion Support** - Full IVC[^43]
  - *Full incremental verifiable computation via lift/join/resolve programs*
- **Proof Composition** - Recursive[^44]
  - *Three-tier architecture: Execution → STARK → Groth16 for optimal size/cost tradeoffs*

## Development and Production Status

### Maturity
- **Development Status** - Production[^45]
- **Security Audit Status** - Multiple[^46]
  - *Multiple security audits available through rz-security repository*
- **Vendor Suggests Production Use** - Yes[^47]
- **API Stability** - Stable[^48]
- **First Commit Date** - 2022-02-19 (based on repository metadata)
- **Last Commit Date** - 2025-08-22[^49]
- **Total Commits** - 3,143[^50]
- **Commit Frequency** - 2.46 commits/day (active development)

### Platform Support
- **Supported Operating Systems** - Linux | Windows | macOS[^51]
- **Supported Architectures** - x86_64 | ARM64[^52]
- **Build Requirements** - Rust 1.88 (2024 edition)[^53]
- **External Dependencies** - CUDA toolkit (optional), sppark library

### Toolchain Integration
- **Compiler Support** - Rust | C/C++[^54]
- **Compiler Version** - Rust 1.88 with risc0-specific cross-compilation support[^55]
- **IDE Integration** - VS Code | CLI[^56]
- **Debugging Support** - Yes[^57]
  - *Supports GDB debugging and profiling with pprof integration*
- **Profiling Tools** - Yes[^58]
  - *Built-in profiling with flame graph generation and performance analysis*

## Documentation Quality

### Overall Assessment
The Risc0 zkVM documentation demonstrates **high quality** with comprehensive coverage across multiple domains. The documentation is well-structured, professionally maintained, and effectively serves both newcomers and experienced developers. With extensive coverage including tutorials, API references, examples, and advanced topics, the documentation ecosystem is mature and well-organized[^59]. The documentation strikes an excellent balance between accessibility for beginners and comprehensive technical depth for advanced users.

### Strengths
- **Comprehensive Tutorial System**: Excellent step-by-step tutorials including detailed Hello World guide with working code examples[^60]
- **Rich Example Ecosystem**: Robust collection of 25+ practical examples covering digital signatures, machine learning, blockchain integration, and more[^61]  
- **Excellent Terminology Coverage**: Thorough definitions of 35+ key concepts from basic to advanced zkVM terminology[^62]
- **Advanced Developer Tools**: Strong coverage of profiling, optimization, and debugging tools with industry-standard integration[^63]
- **Security Documentation**: Comprehensive security development lifecycle documentation with detailed threat modeling processes[^64]

### Areas for Improvement
- **API Reference Integration**: More integrated API examples within main documentation would improve developer experience
- **Advanced Troubleshooting**: More extensive troubleshooting guides for common development issues beyond the current FAQ[^65]
- **Enterprise Architecture Guidance**: Limited coverage of large-scale enterprise implementation patterns

### Coverage Analysis
- **Well-documented areas**: Getting started, core concepts, security practices, performance optimization, examples
- **Partially documented areas**: Integration patterns, deployment guidance, version migration
- **Missing documentation**: Advanced debugging techniques, enterprise architecture patterns

### Documentation Metrics
- **Total documentation pages**: 395+ markdown files
- **Code examples count**: 25+ complete working examples
- **Tutorial/guide count**: Multiple tutorials with step-by-step instructions
- **Last documentation update**: Recent (based on version 3.0 release)

## Notes

### Security Considerations
- **Security Warnings** - Precompiles do not provide constant-time execution guarantees; use caution with private data[^66]
- **Known Limitations** - Some precompiles require "unstable" feature flag and are undergoing review[^67]
- **Recommended Use Cases** - Blockchain applications, verifiable computation, privacy-preserving protocols, compliance verification

### Additional Information  
- **Performance Benchmarks** - Comprehensive benchmark suite available with detailed metrics[^68]
- **Notable Features** - Three-tier proving architecture, extensive precompile library, production-ready blockchain integration, fully open-source GPU acceleration
- **Community/Ecosystem** - Active Discord community, comprehensive examples, regular security audits, bug bounty program[^69]

### Note to User
Risc0 represents a **production-ready zkVM** with exceptional engineering quality and comprehensive ecosystem support. The system's three-tier proving architecture (STARK → Recursion → Groth16) provides optimal flexibility between proof size and verification cost, making it suitable for diverse deployment scenarios from development to production blockchain applications.

**Key Technical Strengths:**
- **Complete RISC-V compatibility** enables seamless migration of existing code
- **Hardware acceleration with full transparency** - all GPU kernels are open source under Apache 2.0
- **Extensive cryptographic precompiles** significantly improve performance for common operations
- **Production blockchain integration** with deployed verifiers on Ethereum and Solana
- **Post-quantum security** in default STARK mode

**Practical Considerations:**
- **Licensing**: Fully permissive Apache 2.0 license across all components including GPU acceleration
- **Development Experience**: Excellent tooling with standard Rust workflows, comprehensive debugging support
- **Performance**: Hardware acceleration provides practical proving times for real-world applications
- **Security**: Multiple professional audits, active bug bounty program, comprehensive security documentation
- **Community**: Strong ecosystem with active development, extensive examples, responsive support

**Recommendation**: Risc0 is highly recommended for production use, particularly for applications requiring blockchain verification, extensive cryptographic operations, or migration of existing RISC-V/Rust codebases. The combination of technical maturity, comprehensive tooling, full open-source transparency, and active ecosystem makes it an excellent choice for serious zero-knowledge application development.

---

### Citations

[^9]: [zkVM host-guest separation](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/risc0/zkvm/src/lib.rs#L20-L25)
[^10]: [Memory organization with 1KB pages](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/risc0/binfmt/src/lib.rs#L44-L46)
[^11]: [RISC-V execution environment](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/risc0/circuit/rv32im/src/execute/rv32im.rs#L21-L63)
[^12]: [Execution trace capture](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/risc0/circuit/rv32im/src/trace.rs)
[^13]: [Baby Bear field definition](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/risc0/core/src/field/baby_bear.rs#L84)
[^14]: [Zirgen constraint generation](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/risc0/circuit/rv32im/src/zirgen/mod.rs)
[^15]: [Three-stage proving pipeline](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/risc0/zkvm/src/receipt.rs)
[^16]: [Blockchain verifier contracts](https://github.com/risc0/risc0-solana/blob/0af1d39ecde3f9a12d34bc78084a31f1cc6c59ba/solana-verifier/programs/groth_16_verifier/src/lib.rs)
[^17]: [Precompiles and acceleration features](https://dev.risczero.com/api/zkvm/precompiles)
[^18]: [risc0-zkp version in workspace](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/Cargo.toml#L67)
[^19]: [Zirgen constraint language](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/risc0/circuit/rv32im/src/zirgen/circuit.rs)
[^20]: [Baby Bear field modulus](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/risc0/core/src/field/baby_bear.rs#L84)
[^21]: [Poseidon2 hash implementation](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/risc0/zkp/src/core/hash/poseidon2/mod.rs)
[^22]: [SHA256 hash option](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/risc0/zkp/src/core/hash/sha.rs)
[^23]: [Blake2b hash implementation](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/risc0/zkp/src/core/hash/blake2b.rs)
[^24]: [Groth16 trusted setup](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/groth16_proof/groth16/risc0.circom)
[^25]: [STARK post-quantum security](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/risc0/zkp/src/lib.rs)
[^26]: [Hardware acceleration features](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/risc0/zkvm/src/lib.rs#L55-L57)
[^27]: [GPU vendor support](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/risc0/sys/kernels/zkp/)
[^28]: [CUDA kernels in repository](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/risc0/sys/kernels/zkp/cuda/)
[^29]: [GPU code under Apache 2.0](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/LICENSE)
[^30]: [Custom CUDA kernel implementations](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/risc0/sys/kernels/zkp/cuda/ntt.cu)
[^31]: [Metal kernel transparency](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/risc0/sys/kernels/zkp/metal/ntt.metal)
[^32]: [Open source acceleration](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/risc0/sys/)
[^33]: [Multi-GPU support in HAL](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/risc0/zkp/src/hal/)
[^34]: [sppark assembly backend](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/Cargo.toml#L71)
[^35]: [Platform-specific optimizations](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/risc0/sys/kernels/zkp/metal/)
[^36]: [Ethereum and Solana verification](https://github.com/risc0/risc0-solana/blob/0af1d39ecde3f9a12d34bc78084a31f1cc6c59ba/solana-verifier/)
[^37]: [Groth16 verifier contract](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/groth16_proof/groth16/verifier.sol)
[^38]: [Solana verifier program](https://github.com/risc0/risc0-solana/blob/0af1d39ecde3f9a12d34bc78084a31f1cc6c59ba/solana-verifier/programs/groth_16_verifier/src/lib.rs)
[^39]: [Solidity verifier availability](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/groth16_proof/groth16/verifier.sol)
[^40]: [Solana verifier program deployment](https://github.com/risc0/risc0-solana/blob/0af1d39ecde3f9a12d34bc78084a31f1cc6c59ba/solana-verifier/Anchor.toml)
[^41]: [Automatic verifier generation](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/risc0/groth16/src/lib.rs)
[^42]: [Recursion and aggregation](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/risc0/circuit/recursion/)
[^43]: [IVC implementation](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/risc0/recursion/)
[^44]: [Three-tier proof composition](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/risc0/zkvm/src/receipt.rs)
[^45]: [Production status from version 3.0](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/Cargo.toml#L40)
[^46]: [Security audit reports](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/SECURITY.md#L7)
[^47]: [Production use recommendation](https://dev.risczero.com/api/getting-started)
[^48]: [API stability](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/Cargo.toml#L40)
[^49]: [Latest commit date](https://github.com/risc0/risc0/commit/2384a958948366f9181032d4f9791e295df4e98d)
[^50]: [Total commit count from git log](https://github.com/risc0/risc0)
[^51]: [Multi-platform support](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/.github/workflows/)
[^52]: [Architecture support](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/rust-toolchain.toml)
[^53]: [Rust toolchain version](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/rust-toolchain.toml)
[^54]: [Language support](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/examples/c-guest/)
[^55]: [risc0 Rust toolchain](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/rust-toolchain.toml)
[^56]: [IDE integration via cargo-risczero](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/risc0/cargo-risczero/)
[^57]: [Debugging support](https://dev.risczero.com/api/zkvm/debugging-with-gdb)
[^58]: [Profiling tools](https://dev.risczero.com/api/zkvm/profiling)
[^59]: [Documentation structure](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/website/)
[^60]: [Hello World tutorial](https://dev.risczero.com/api/zkvm/quickstart)
[^61]: [Examples collection](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/examples/)
[^62]: [Terminology documentation](https://dev.risczero.com/terminology)
[^63]: [Profiling documentation](https://dev.risczero.com/api/zkvm/profiling)
[^64]: [Security development lifecycle](https://dev.risczero.com/api/secure-sdlc)
[^65]: [FAQ documentation](https://dev.risczero.com/faq)
[^66]: [Precompile timing warning](https://dev.risczero.com/api/zkvm/precompiles#timing-attacks)
[^67]: [Unstable feature requirements](https://dev.risczero.com/api/zkvm/precompiles#stability)
[^68]: [Performance benchmarks](https://dev.risczero.com/api/zkvm/benchmarks)
[^69]: [Bug bounty program](https://github.com/risc0/risc0/blob/2384a958948366f9181032d4f9791e295df4e98d/SECURITY.md#L15)