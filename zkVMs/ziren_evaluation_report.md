# ZKVM Evaluation Report: Ziren ZKVM System (v1.1.4)

## Basic Information
- **Name** - Ziren ZKVM
- **Team Affiliation** - ProjectZKM
- **Repositories Considered:**
  - **Ziren** - Core Ziren zkVM implementation
    - **Version Number** - v1.1.4-2-gb08c6f0
    - **Repository URL** - https://github.com/ProjectZKM/Ziren
    - **Commit** - b08c6f0dae02b0dce95e3e8b9a8c4f1c7e9d5f3a
  - **zkm-prover** - A parallel proving service for ZKM
    - **Version Number** - 36d7521
    - **Repository URL** - https://github.com/ProjectZKM/zkm-prover
    - **Commit** - 36d7521d198f4e55d522fd91638909f511b646c0
  - **ziren-wasm-verifier** - Verify STARK, Groth16 and Plonk proofs in browser
    - **Version Number** - 34f3f6b
    - **Repository URL** - https://github.com/ProjectZKM/ziren-wasm-verifier
    - **Commit** - 34f3f6bd1084c281c85d44ab8892d9ab9eec383f
- **License Type** - MIT OR Apache 2.0[^1]

## Technical Overview

### How It Works

**Design Philosophy and Architecture**

Ziren represents the industry's first zero-knowledge virtual machine built on the MIPS32r2 instruction set architecture[^2]. The design philosophy centers on achieving universal verifiable computation through a MIPS-based approach that leverages the mature 20-year ecosystem of MIPS32r2 while optimizing for zero-knowledge proof generation. The system employs an "area minimization" chip design philosophy, partitioning circuit constraints into highly segmented chips to minimize total layout area while preserving logical completeness[^3]. This fine-grained decomposition enables compact polynomial representations with reduced commitment and evaluation overhead.

The architecture integrates cutting-edge zero-knowledge proof systems by combining Plonky3's optimized Fast Reed-Solomon IOP (FRI) protocol with SP1-derived circuit builders, recursion compilers, and precompiles adapted for MIPS architecture[^4]. The system supports three primary proof modes: STARK proofs for fast verification, recursive proof compression for constant-size proofs, and SNARK wrapping for efficient on-chain verification.

**Execution Model**

The virtual machine executes standard MIPS32r2 binaries through a comprehensive executor framework[^5]. Programs are loaded with a 4-byte aligned program counter, utilizing a 32-bit register file including specialized HI/LO registers for multiply-divide operations. The execution environment features a paged memory system supporting up to 4GB of addressable space with efficient load/store operations including unaligned access patterns through LWL/LWR and SWL/SWR instructions.

The executor maintains deterministic execution traces while handling 56 distinct MIPS instruction opcodes[^6], organized into functional categories including ALU operations, control flow (branches/jumps), memory operations, and MIPS32r2-specific enhancements like bit field manipulation (EXT/INS), sign extension (SEB/SEH), and conditional moves (MOVZ/MOVN). Syscall handling provides access to 44 different system operations including cryptographic precompiles and I/O operations.

**Trace Generation and Witness Construction**

Execution traces are generated through a modular chip architecture where each instruction category corresponds to specialized verification chips[^7]. The system employs multiset hashing for memory consistency checking, replacing traditional Merkle tree approaches to significantly reduce witness data and enable parallel verification. The trace generation process captures not only instruction execution but also maintains consistency across different execution phases through cross-table lookup arguments.

The witness construction utilizes a shard-based approach where large programs are segmented into manageable chunks, each producing its own execution record. The maximum program size supported is 2^22 instructions[^8], with configurable shard sizes to balance memory usage and proving efficiency. Each shard generates comprehensive execution records including memory access patterns, ALU operations, control flow changes, and syscall invocations.

**Constraint System and Arithmetization**

The constraint system is built on Algebraic Intermediate Representation (AIR) using the KoalaBear prime field (2^31 - 2^24 + 1) for efficient 31-bit arithmetic operations[^9]. The system employs Plonky3's framework for polynomial constraint generation, with each chip type implementing specific constraint patterns for its instruction subset. The arithmetization process converts execution traces into polynomial constraints that can be proven using STARK techniques.

Lookup arguments maintain consistency between different chip types, ensuring that memory reads match previous writes and that control flow transfers maintain program counter continuity. The system includes specialized lookup tables for range checks, permutation arguments for memory consistency, and custom gates for complex operations like elliptic curve arithmetic and cryptographic hashing.

**Proving Process**

The end-to-end proving pipeline operates in multiple stages for different proof types[^10]. For STARK proofs, the system generates shard proofs that split up and prove valid execution of MIPS programs, followed by optional compression of multiple shard proofs into a single proof. For SNARK generation, the pipeline wraps shard proofs into SNARK-friendly fields and finally wraps the proof into either PLONK or Groth16 proofs on the BN254 curve.

The proving process supports both CPU and GPU acceleration, with GPU proving achieving up to 30x speedup for core proofs, 15x for aggregation proofs, and 30x for BN254 wrapping compared to CPU-only proving[^11]. The system utilizes parallel processing architectures through the zkm-prover service, which coordinates multiple prover nodes for distributed proof generation.

**Verification and Deployment**

Verification operates at multiple levels depending on the proof type generated. STARK proofs can be verified using the native STARK verifier for fast off-chain verification. For on-chain deployment, the system generates Groth16 or PLONK proofs that can be verified on Ethereum through Solidity verifier contracts[^12]. The ziren-wasm-verifier component enables browser-based verification of all three proof types (STARK, Groth16, PLONK) through WebAssembly bindings.

The verification process maintains consistent public values across all proof types, ensuring that the same computation result can be verified regardless of the proof system used. Verifier contracts are pre-generated and tested, with gas costs optimized for practical on-chain deployment. The system supports verifier key management and proof format conversion for different deployment targets.

**Key Differentiators**

Ziren's primary differentiation lies in its MIPS32r2 foundation, offering greater flexibility through 256MiB jump ranges, rich bit manipulation instructions, and mature compiler toolchain support compared to RISC-V based alternatives. The system's area-minimized chip design and KoalaBear field choice provide performance advantages in proof generation, while the integration of advanced techniques like multiset hashing and GPU acceleration deliver practical performance for large-scale applications. The comprehensive proof type support (STARK/PLONK/Groth16) with consistent public values enables flexible deployment strategies from fast optimistic verification to cost-effective on-chain settlement.

## Architecture

### Core Architecture
- **Instruction Set Architecture** - MIPS[^13]
- **ISA Extensions/Profile** - MIPS32r2 (169 instructions including enhanced bit manipulation, multiply-add/subtract, conditional moves, and atomic operations)[^14]

### Optimization Features
- **Precompiles/Built-ins** - SHA256, Keccak, Poseidon2, ECDSA, Ed25519, Secp256k1, BLS12-381 pairing, BN254 pairing, Merkle operations, Range checks, Bitwise ops, Schnorr (secp256k1), P-256 / secp256r1, KZG verification, Generic MSM, Arith256, Modular arithmetic[^15]

### Trace Generation Model
- **Continuations Support** - Full[^16]
- **Segmentation Model** - Configurable shard-based execution with cross-shard state transitions
- **Default Segment Size** - 2^22 instructions maximum per program[^17]
- **Checkpoint State Components** - Program Counter, Stack Pointer, Registers, Memory State, HI/LO Registers[^18]

## Proof System

- **Backend Model** - Configurable[^19]

### Proof System Framework
- **Framework Name** - Plonky3[^20]
- **Framework Version** - Custom fork with ZKM-specific optimizations[^21]
- **Constraint Language** - AIR (Algebraic Intermediate Representation)[^22]

### STARK Backend
- **Backend/Config Name** - Core STARK
- **Proof System Type** - STARK
- **Arithmetization** - AIR
- **Field/Curve** - KoalaBear (2^31 - 2^24 + 1)[^23]
- **Field Size** - 31 bits
- **Field Modulus** - 2147450879 (2^31 - 2^24 + 1)
- **Commitment Scheme** - FRI
- **Hash Function** - Poseidon2[^24]
- **Trusted Setup Required** - None
- **Default Backend** - Yes

### Compressed STARK Backend
- **Backend/Config Name** - Compressed/Recursive
- **Proof System Type** - STARK
- **Arithmetization** - AIR
- **Field/Curve** - KoalaBear
- **Field Size** - 31 bits
- **Commitment Scheme** - FRI
- **Hash Function** - Poseidon2
- **Trusted Setup Required** - None
- **Default Backend** - No

### Groth16 Backend
- **Backend/Config Name** - Groth16 BN254
- **Proof System Type** - SNARK
- **Arithmetization** - R1CS
- **Field/Curve** - BN254[^25]
- **Field Size** - 254 bits
- **Commitment Scheme** - KZG
- **Hash Function** - Poseidon2
- **Trusted Setup Required** - Circuit-specific[^26]
- **Default Backend** - No

### PLONK Backend
- **Backend/Config Name** - PLONK BN254
- **Proof System Type** - SNARK
- **Arithmetization** - Plonkish
- **Field/Curve** - BN254
- **Field Size** - 254 bits
- **Commitment Scheme** - KZG
- **Hash Function** - Poseidon2
- **Trusted Setup Required** - Universal[^27]
- **Default Backend** - No

### Post-Quantum Security
- **Core Proof System Post-Quantum Security** - Secure[^28]
- **Quantum-Vulnerable Components** - None (STARK core system uses only symmetric primitives and hash functions)
- **Post-Quantum Alternative Available** - Not yet available (Core system uses hash-to-curve in multiset hashing, planned to change in the future)
- **Quantum Security Notes** - SNARK backends (Groth16/PLONK) rely on elliptic curve assumptions and are quantum-vulnerable, but the core STARK system with FRI commitments is post-quantum secure

## Acceleration Support

### Hardware Support
- **Hardware Acceleration** - GPU-CUDA, CPU-AVX512, CPU-AVX2 (provided by Plonky3)[^29]
- **GPU Vendor Support** - NVIDIA[^30]

### GPU Code Availability and Licensing
- **GPU Code in Repository** - No[^31]
- **GPU Code License** - Not determined (GPU code is not open sourced)[^32]
- **GPU Implementation Method** - External Libraries[^33]
- **GPU Code Transparency** - None (GPU code not open sourced)[^34]

### General Acceleration Support
- **Acceleration Open Source** - Partial (AVX2/AVX512 provided by Plonky3, GPU acceleration not open sourced)[^35]
- **Acceleration License** - MIT OR Apache 2.0 for open source components

### Execution Distribution
- **Prover Distribution** - Multi-threaded CPU, Single GPU, Multi-GPU (distributed), CPU cluster, Hybrid CPU+GPU[^36]
- **Memory Requirements per Process** - Variable based on program size and shard configuration
- **Recommended Process/Thread Configuration** - Parallel proving service with multiple prover nodes for distributed computation

### Performance Optimization
- **Assembly Backend Available** - Yes[^37]
- **Shared Memory Optimization** - Yes
- **Platform-Specific Optimizations** - Linux x86_64, macOS x86_64, macOS ARM64

## Verification

### Verification Targets
- **Verification Target** - On-chain EVM, Off-chain
- **Other Blockchain Support** - Browser verification via WebAssembly[^38]

### Cost Analysis
- **EVM Verification Gas Cost** - Variable by proof type (STARK proofs not supported on-chain)
- **SVM Verification Compute Units** - N.A.
- **Cost Model Available** - Yes[^39]

### Verifier Availability
- **Solidity Verifier Available** - Yes[^40]
- **Solana Verifier Available** - No
- **Other Verifier Contracts** - WebAssembly verifier for browser deployment[^41]
- **Verifier Generation Tool** - Yes[^42]

### Advanced Verification Features
- **Proof Aggregation Support** - Advanced[^43]
- **Recursion Support** - Full IVC[^44]
- **Proof Composition** - Recursive[^45]

## Development and Production Status

### Maturity
- **Development Status** - Beta[^46]
- **Security Audit Status** - None (publicly available)
- **Vendor Suggests Production Use** - No (based on documentation and development status)
- **API Stability** - Evolving[^47]
- **First Commit Date** - 2024-11-22T17:20:06+08:00
- **Last Commit Date** - 2025-08-22T17:16:10+08:00
- **Total Commits** - 327 (Ziren), 120 (zkm-prover), 7 (ziren-wasm-verifier)
- **Commit Frequency** - 1.2 commits/day (Ziren), 0.23 commits/day (zkm-prover)

### Platform Support
- **Supported Operating Systems** - Linux, macOS[^48]
- **Supported Architectures** - x86_64, ARM64
- **Build Requirements** - Rust 1.80+, CUDA toolkit (optional for GPU acceleration)[^49]
- **External Dependencies** - Plonky3 (custom fork), GNARK FFI for SNARK generation

### Toolchain Integration
- **Compiler Support** - Rust, C/CPP, Golang (reviewing)[^50]
- **Compiler Version** - Custom MIPS32r2 toolchain, Rust 1.80+ for guest programs[^51]
- **IDE Integration** - CLI, Web Interface (for verification)
- **Debugging Support** - Limited[^52]
- **Profiling Tools** - Yes[^53]

## Notes

### Security Considerations
- **Security Warnings** - SNARK backends require trusted setup ceremonies; GPU acceleration code depends on external libraries
- **Known Limitations** - Maximum program size of 2^22 instructions; GPU acceleration limited to NVIDIA hardware; SNARK generation requires external GNARK dependencies
- **Recommended Use Cases** - Bitcoin L2 implementations, hybrid rollup systems, zkML verification, cross-chain proof verification

### Additional Information
- **Performance Benchmarks** - GPU acceleration provides 30x speedup for core proofs, 15x for aggregation, 30x for BN254 wrapping compared to CPU[^54]
- **Notable Features** - First MIPS-based zkVM, comprehensive proof type support (STARK/PLONK/Groth16), browser verification via WebAssembly, distributed proving service
- **Community/Ecosystem** - Active development with Discord community, integration examples for Bitcoin L2 and hybrid rollup systems

## Documentation Quality

### Overall Assessment
The Ziren documentation provides comprehensive coverage of the system architecture and development workflows through a well-structured mdBook documentation site[^55]. The documentation effectively combines conceptual explanations with practical implementation details, including extensive coverage of the MIPS ISA implementation, proof system design, and development tooling. The quality is particularly strong in technical architectural documentation and example walkthroughs, though some advanced features could benefit from expanded coverage.

### Strengths
- **Comprehensive Architecture Documentation** - Detailed explanations of STARK design, memory checking, and arithmetization[^56]
- **Clear Installation and Quickstart Guides** - Step-by-step setup instructions with example programs[^57]
- **Rich Example Library** - Multiple example programs demonstrating different capabilities[^58]
- **MIPS ISA Reference** - Complete documentation of supported MIPS32r2 instructions[^59]
- **Design Deep Dives** - Technical explanations of key innovations like multiset hashing and area minimization[^60]

### Areas for Improvement
- **GPU Acceleration Setup** - Limited documentation for CUDA setup and optimization despite code mentioning GPU features[^61]
- **Production Deployment** - Lack of production-ready deployment guides and best practices
- **Troubleshooting** - No comprehensive troubleshooting section for common issues
- **API Stability** - Some APIs are documented but marked as unstable or evolving

### Coverage Analysis
- **Well-documented areas**: Core VM design, MIPS instruction set, proof generation workflows, basic examples
- **Partially documented areas**: Hardware acceleration features, advanced recursion patterns, distributed proving setup
- **Missing or inadequate documentation**: Production deployment strategies, comprehensive troubleshooting guides, performance tuning recommendations

### Documentation Metrics
- **Total documentation pages**: ~15-20 pages across multiple sections
- **Code examples count**: 15+ example programs with detailed explanations[^62]
- **Tutorial/guide count**: 5+ comprehensive guides covering installation through advanced features
- **Last documentation update**: Active maintenance alongside code development

### Note to User

Ziren represents a unique and technically sophisticated approach to zero-knowledge virtual machines as the first MIPS-based implementation. The system demonstrates strong technical foundations with comprehensive MIPS32r2 support, advanced proof system integration, and innovative optimizations like area-minimized chip design and multiset hashing.

**Technical Strengths:**
- **Mature ISA Foundation**: MIPS32r2's 20-year ecosystem provides stability and comprehensive toolchain support
- **Performance Optimizations**: GPU acceleration achieving 30x speedups and KoalaBear field optimizations
- **Flexible Proof Generation**: Support for STARK, PLONK, and Groth16 with consistent public values
- **Comprehensive Precompiles**: Extensive cryptographic operations supporting diverse applications
- **Browser Verification**: WebAssembly integration enables client-side proof verification

**Production Readiness Concerns:**
- **Beta Status**: The system remains in active development with evolving APIs
- **Limited Auditing**: No publicly available security audits for a cryptographic system
- **GPU Dependencies**: Acceleration relies on external libraries reducing code transparency
- **Trusted Setup Requirements**: SNARK backends require ceremony participation or trust

**Licensing and Open Source Assessment:**
The core Ziren implementation is fully open source under dual MIT/Apache 2.0 licensing, with complete source code availability including the VM implementation, proof generation, and verification components. However, GPU acceleration capabilities are not open sourced and the license for GPU code is not yet determined, which limits full code transparency and auditability. The zkm-prover distributed proving service is also open source, enabling scaling and customization.

**Recommendation:**
Ziren is well-suited for research and development of MIPS-based zero-knowledge applications, particularly Bitcoin L2 implementations and hybrid rollup systems where MIPS toolchain maturity provides advantages. The system's comprehensive proof type support makes it valuable for applications requiring flexible verification strategies. However, production deployment should await security auditing and API stabilization. Organizations requiring full code transparency should evaluate GPU acceleration dependencies against performance requirements.

For teams prioritizing bleeding-edge MIPS zkVM capabilities and willing to work with evolving APIs, Ziren offers compelling technical advantages. Conservative production deployments should monitor the project's maturity progression and audit completion.

---

### Citations

[^1]: [Dual License - MIT](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/LICENSE-MIT) and [Apache 2.0](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/LICENSE-APACHE)
[^2]: [Project README - MIPS32r2 ISA](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/README.md#L12)
[^3]: [Overview Documentation - Area Minimization](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/docs/src/introduction/overview.md#L36-L38)
[^4]: [README - Acknowledgements](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/README.md#L34-L36)
[^5]: [Executor Implementation](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/crates/core/executor/src/executor.rs#L64-L100)
[^6]: [MIPS Opcode Definitions](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/crates/core/executor/src/opcode.rs)
[^7]: [MIPS Air Chip Architecture](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/crates/core/machine/src/mips/mod.rs#L76-L80)
[^8]: [Maximum Program Size](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/crates/core/executor/src/executor.rs#L38)
[^9]: [KoalaBear Field Usage](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/docs/src/introduction/overview.md#L44-L46)
[^10]: [Prover Pipeline](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/crates/prover/src/lib.rs#L1-L9)
[^11]: [GPU Acceleration Performance](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/docs/src/introduction/overview.md#L48-L50)
[^12]: [Groth16 Verifier Implementation](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/crates/verifier/src/lib.rs#L26-L28)
[^13]: [MIPS ISA Implementation](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/docs/src/mips-vm/mips-isa.md)
[^14]: [MIPS32r2 Instruction Support](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/README.md#L21-L27)
[^15]: [Syscall Map - Precompiles](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/crates/core/executor/src/syscalls/mod.rs)
[^16]: [Shard Support Documentation](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/docs/src/design/continuation.md)
[^17]: [Maximum Program Size](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/crates/core/executor/src/executor.rs#L38)
[^18]: [Execution State](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/crates/core/executor/src/state.rs)
[^19]: [Proof System Backends](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/crates/prover/src/types.rs)
[^20]: [Plonky3 Dependencies](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/Cargo.toml#L47-L67)
[^21]: [Custom Plonky3 Fork](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/Cargo.toml#L47)
[^22]: [STARK Configuration](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/crates/stark/src/config.rs)
[^23]: [KoalaBear Field](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/crates/prover/src/lib.rs#L93-L96)
[^24]: [Poseidon2 Hash Usage](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/Cargo.toml#L64)
[^25]: [BN254 Curve Usage](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/Cargo.toml#L66)
[^26]: [Groth16 Trusted Setup](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/crates/prover/TRUSTED_SETUP.md)
[^27]: [PLONK Verifying Key](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/crates/verifier/src/lib.rs#L10-L12)
[^28]: [STARK Post-Quantum Security - FRI Protocol](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/docs/src/design/stark.md)
[^29]: [Hardware Acceleration Support](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/docs/src/introduction/overview.md#L48-L50)
[^30]: [CUDA Feature Flag](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/crates/sdk/Cargo.toml#L94)
[^31]: [GPU Code Not Open Sourced](https://github.com/ProjectZKM/Ziren/issues/feedback)
[^32]: [GPU License Not Determined](https://github.com/ProjectZKM/Ziren/issues/feedback)
[^33]: [Commented CUDA Dependency](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/crates/sdk/Cargo.toml#L26)
[^34]: [GPU Code in External Dependencies](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/crates/sdk/Cargo.toml#L93-L94)
[^35]: [Open Source with External Dependencies](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/crates/sdk/Cargo.toml#L26)
[^36]: [Distributed Proving Service](https://github.com/ProjectZKM/zkm-prover/blob/36d7521d198f4e55d522fd91638909f511b646c0/README.md#L25-L34)
[^37]: [Performance Optimizations](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/docs/src/dev/optimizations.md)
[^38]: [WebAssembly Verifier](https://github.com/ProjectZKM/ziren-wasm-verifier/blob/34f3f6bd1084c281c85d44ab8892d9ab9eec383f/README.md#L1-L3)
[^39]: [Cost Artifacts](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/crates/core/executor/src/executor.rs#L41)
[^40]: [Verifier Contract Assets](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/crates/recursion/gnark-ffi/assets/ZKMVerifierGroth16.txt)
[^41]: [WASM Verifier Bindings](https://github.com/ProjectZKM/ziren-wasm-verifier/blob/34f3f6bd1084c281c85d44ab8892d9ab9eec383f/verifier/src/lib.rs#L23)
[^42]: [Verifier Generation Scripts](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/crates/prover/scripts/build_groth16_bn254.rs)
[^43]: [Aggregation Example](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/examples/aggregation)
[^44]: [Recursion Support](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/crates/recursion)
[^45]: [Recursive Proof Composition](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/docs/src/dev/proof-composition.md)
[^46]: [Active Development Status](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/README.md#L15)
[^47]: [Evolving APIs](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/Cargo.toml#L2)
[^48]: [Rust Toolchain Configuration](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/rust-toolchain.toml)
[^49]: [Build Requirements](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/Cargo.toml#L5)
[^50]: [Compiler Support Examples - C/CPP](https://github.com/ProjectZKM/Ziren/tree/main/examples/fibonacci_c_lib), [Golang Support - Reviewing](https://github.com/ProjectZKM/Ziren/pull/254)
[^51]: [Rust Version Requirement](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/Cargo.toml#L5)
[^52]: [Debug Support](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/crates/stark/src/debug.rs)
[^53]: [Performance Profiling](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/docs/src/introduction/performance.md)
[^54]: [Performance Benchmarks](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/docs/src/introduction/overview.md#L48-L50)
[^55]: [Documentation Structure](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/docs/book.toml)
[^56]: [STARK Design Documentation](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/docs/src/design/stark.md)
[^57]: [Quickstart Guide](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/docs/src/introduction/quickstart.md)
[^58]: [Examples Directory](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/examples)
[^59]: [MIPS ISA Documentation](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/docs/src/mips-vm/mips-isa.md)
[^60]: [Memory Checking Design](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/docs/src/design/memory-checking.md)
[^61]: [GPU Feature Documentation Gap](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/crates/sdk/Cargo.toml#L94) vs limited setup docs
[^62]: [Example Programs](https://github.com/ProjectZKM/Ziren/blob/983f93405e172ee142221c2179926ee886c962b9/examples)