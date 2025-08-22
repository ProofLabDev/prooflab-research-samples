# ZKVM Evaluation Report: SP1 (v5.2.1)

## Basic Information
- **Name** - SP1[^1]
- **Team Affiliation** - Succinct Labs[^2]
- **Repositories Considered:**
  - **SP1 Core** 
    - **Version Number** - v5.2.1[^3]
    - **Repository URL** - https://github.com/succinctlabs/sp1
    - **Commit** - 3209d5454e8cddb635dd8425e3807a60e8dac395
  - **SP1 Solana**
    - **Version Number** - 4181cae
    - **Repository URL** - https://github.com/succinctlabs/sp1-solana
    - **Commit** - 4181cae00d7493ede8a33066cb56683acb1dca72
- **License Type** - MIT OR Apache 2.0[^4][^5]

[^1]: [SP1 README](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/README.md)
[^2]: [Copyright notice in MIT license](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/LICENSE-MIT#L3)
[^3]: [Version file](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/SP1_VERSION)
[^4]: [MIT License](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/LICENSE-MIT)
[^5]: [Apache 2.0 License](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/LICENSE-APACHE)

## Technical Overview

### How It Works

**Design Philosophy and Architecture**

SP1 is a high-performance, open-source zkVM designed to make zero-knowledge proof generation for general computation practical and accessible[^6]. Built on the RISC-V instruction set architecture, SP1 prioritizes performance, developer experience, and production readiness. The system employs a multi-stage proving pipeline that combines the transparency of STARKs with the succinctness of SNARKs, enabling both off-chain and on-chain verification scenarios[^7][^8].

The architectural philosophy centers around maximizing prover performance through extensive cryptographic precompiles, GPU acceleration, and a carefully designed proving pipeline. SP1 trades some generality for performance by implementing a subset of RISC-V (RV32IM) rather than full compatibility, focusing on the instructions most commonly needed for cryptographic and mathematical workloads[^9].

**Execution Model**

SP1 programs execute within a RISC-V RV32IM virtual machine that implements a standard fetch-decode-execute cycle[^10]. The VM maintains a 32-register file following RISC-V conventions, with register X0 hardwired to zero and X31 serving as the link register for function calls[^11]. Memory is organized as a paged system with word-aligned 32-bit addressing, supporting both register operations (addresses 0-31) and main memory access[^12].

Program execution begins by loading a transpiled ELF binary into the VM's memory space. The transpilation process converts standard RISC-V machine code into SP1's optimized instruction encoding, which maintains semantic equivalence while enabling more efficient constraint generation[^13]. The VM supports system calls (ECALL instructions) that provide access to optimized cryptographic operations and I/O functionality[^14].

**Trace Generation and Witness Construction**

During execution, SP1 captures comprehensive execution traces that record every state transition of the virtual machine[^15]. The trace includes CPU events (instruction execution, register updates), memory operations (loads, stores, address generation), precompile invocations, and global state changes. This trace serves as the witness for zero-knowledge proof generation[^16].

SP1 employs a shard-based segmentation model that divides long computations into manageable chunks. Shard sizes are dynamically configured based on available system memory, ranging from 2^19 cycles (524,288 instructions) on systems with less than 33GB RAM to 2^21 cycles (2,097,152 instructions) on high-memory systems[^17]. Between shards, the system maintains memory checkpoints that track only the memory locations that have been modified, enabling efficient state continuations[^18].

**Constraint System and Arithmetization**

SP1 translates execution traces into polynomial constraints using the Algebraic Intermediate Representation (AIR) framework provided by Plonky3[^19]. Each instruction type and precompile operation has a corresponding AIR that defines the polynomial relationships that must hold for valid execution. The constraint system operates over the Baby Bear field (2^31 - 5) for optimal performance on 64-bit architectures[^20].

The arithmetization process generates execution traces for each component (CPU, ALU, memory, precompiles) and applies the corresponding constraints to verify correctness. Lookup tables are used extensively to optimize constraint checking for operations like byte operations, range checks, and field arithmetic[^21].

**Proving Process**

SP1's proving pipeline consists of four distinct stages. The **core stage** generates STARK proofs for each execution shard using Baby Bear field arithmetic and FRI commitments[^22]. The **compress stage** uses recursive techniques to aggregate multiple shard proofs into a single compressed proof, reducing verification time for subsequent stages[^23]. The **shrink stage** further compresses proofs to prepare them for efficient SNARK wrapping[^24]. Finally, the **wrap stage** converts the STARK proof into a BN254-based SNARK (either PLONK or Groth16) suitable for on-chain verification[^25].

**Verification and Deployment**

The final SNARK proofs can be verified on-chain through deployed verifier contracts. Groth16 proofs consume approximately 270,000 gas on Ethereum for verification, while PLONK proofs require around 300,000 gas[^26]. SP1 maintains canonical verifier gateways on multiple EVM chains and provides a Solana verifier program for SVM-based blockchains[^27]. The system supports proof aggregation, allowing multiple program executions to be batched into a single on-chain verification[^28].

**Key Differentiators**

SP1 distinguishes itself through comprehensive cryptographic precompiles that enable efficient proving of real-world cryptographic operations. The system provides optimized implementations for elliptic curve operations (secp256k1, secp256r1, BN254, BLS12-381, Ed25519), hash functions (SHA-256, Keccak-256), and big integer arithmetic[^29]. GPU acceleration through the Moongate server architecture allows core proving to complete in minutes rather than hours for complex programs[^30]. The multi-stage proving pipeline balances proving time, proof size, and verification cost, making SP1 suitable for both development and production deployment scenarios[^31].

## Architecture

### Core Architecture
- **Instruction Set Architecture** - RISC-V[^32]
  - *SP1 implements the RV32IM subset of RISC-V (32-bit base integer with multiplication/division)*
- **ISA Extensions/Profile** - RV32IM[^33]
  - *Base integer instruction set (RV32I) plus multiplication and division extension (M)*
  - *Custom instruction encoding optimized for zkVM constraints while maintaining RISC-V semantics*

### Optimization Features
- **Precompiles/Built-ins** - SHA256, Keccak, secp256k1, secp256r1, BN254 pairing, BLS12-381 pairing, Ed25519, Arith256[^34]
  - *SHA-256 compress and extend operations for efficient hash computation*
  - *Keccak-256 permutation for Ethereum-compatible hashing*
  - *Elliptic curve operations: point addition, doubling, decompression for multiple curves*
  - *BN254 and BLS12-381 field arithmetic (Fp, Fp2) and pairing operations*
  - *Ed25519 Edwards curve operations for signature verification*
  - *Big integer multiplication for UINT256 and U256×U2048 operations*

### Trace Generation Model
- **Continuations Support** - Full[^35]
  - *SP1 supports comprehensive program continuations through memory checkpointing*
  - *Programs can be paused and resumed across shard boundaries with preserved state*
- **Segmentation Model** - Shard-based with dynamic sizing[^36]
  - *Execution is divided into shards with sizes determined by available system memory*
  - *Memory checkpoints track state differences between shards for efficient continuations*
- **Default Segment Size** - 524,288 to 2,097,152 cycles[^37]
  - *2^19 cycles (524,288) for systems with <33GB RAM*
  - *2^20 cycles (1,048,576) for systems with 33-49GB RAM*
  - *2^21 cycles (2,097,152) for systems with >49GB RAM*
- **Checkpoint State Components** - Program Counter, Stack Pointer, Registers, Memory State[^38]
  - *Complete CPU state including all 32 RISC-V registers*
  - *Memory differences (touched addresses) between shard boundaries*
  - *Execution metadata including clock cycles and public values*

## Proof System
- **Backend Model** - Configurable[^39]
  - *SP1 has multiple fixed proving stages with configurable parameters per stage*

### Proof System Framework
- **Framework Name** - Plonky3[^40]
- **Framework Version** - 0.2.3-succinct[^41]
- **Constraint Language** - AIR (Algebraic Intermediate Representation)[^42]

### Core Backend (CoreSC)
- **Backend/Config Name** - BabyBearPoseidon2[^43]
- **Proof System Type** - STARK
- **Arithmetization** - AIR
- **Field/Curve** - Baby Bear[^44]
- **Field Size** - 31 bits
- **Field Modulus** - 2013265921 (2^31 - 5)
- **Commitment Scheme** - FRI
- **Hash Function** - Poseidon2[^45]
- **Trusted Setup Required** - None
- **Default Backend** - Yes

### Inner Backend (InnerSC)
- **Backend/Config Name** - BabyBearPoseidon2Inner[^46]
- **Proof System Type** - STARK
- **Arithmetization** - AIR
- **Field/Curve** - Baby Bear
- **Field Size** - 31 bits
- **Field Modulus** - 2013265921 (2^31 - 5)
- **Commitment Scheme** - FRI
- **Hash Function** - Poseidon2
- **Trusted Setup Required** - None
- **Default Backend** - No

### Outer Backend (OuterSC)
- **Backend/Config Name** - BabyBearPoseidon2Outer[^47]
- **Proof System Type** - Hybrid
- **Arithmetization** - AIR
- **Field/Curve** - BN254
- **Field Size** - 254 bits
- **Field Modulus** - 21888242871839275222246405745257275088548364400416034343698204186575808495617
- **Commitment Scheme** - FRI
- **Hash Function** - Poseidon2
- **Trusted Setup Required** - Inherited
- **Default Backend** - No

### Post-Quantum Security
- **Core Proof System Post-Quantum Security** - Partial[^48]
  - *STARK components (core, compress, shrink stages) are post-quantum secure*
  - *Final SNARK wrapping stage uses elliptic curve cryptography on BN254*
- **Quantum-Vulnerable Components** - Elliptic Curve Commitments, Pairing-based Cryptography[^49]
  - *BN254 curve used in final SNARK generation*
  - *Groth16 and PLONK SNARKs rely on discrete logarithm assumptions*
- **Post-Quantum Alternative Available** - No
  - *Users can stop at compressed proof stage to maintain post-quantum security*
- **Quantum Security Notes** - SP1's multi-stage design allows users to choose between post-quantum security (STARK proofs) and on-chain verification compatibility (SNARK proofs)[^50]

## Acceleration Support

### Hardware Support
- **Hardware Acceleration** - GPU-CUDA, CPU-AVX2[^51]
- **GPU Vendor Support** - NVIDIA[^52]

### GPU Code Availability and Licensing
- **GPU Code in Repository** - No[^53]
  - *GPU acceleration provided through external Moongate server container*
  - *No custom GPU kernels visible in repository*
- **GPU Code License** - External Dependency[^54]
  - *GPU implementation hidden within Docker containers*
- **GPU Implementation Method** - Docker Images[^55]
  - *Uses pre-built Docker image: public.ecr.aws/succinct-labs/moongate:v5.0.8*
- **GPU Code Transparency** - Opaque[^56]
  - *GPU acceleration implementations are not visible or auditable*
  - *Relies on closed-source containerized proving service*

### General Acceleration Support
- **Acceleration Open Source** - No[^57]
  - *GPU kernel implementations are proprietary/containerized*
- **Acceleration License** - Proprietary[^58]
  - *Moongate server components are not open source*

### Execution Distribution
- **Prover Distribution** - Single GPU, Multi-threaded CPU, Hybrid CPU+GPU[^59]
  - *Core proving can utilize single GPU through Moongate server*
  - *CPU execution and final SNARK generation use multi-threaded CPU*
  - *Hybrid approach: CPU for execution/setup, GPU for core proving*
- **Memory Requirements per Process** - 24GB VRAM minimum[^60]
  - *GPU proving requires substantial memory for large circuit constraints*
- **Recommended Process/Thread Configuration** - Docker-managed GPU allocation[^61]
  - *Moongate server handles GPU resource management automatically*

### Performance Optimization
- **Assembly Backend Available** - No
- **Shared Memory Optimization** - Yes[^62]
  - *Memory checkpointing optimizes continuation overhead*
- **Platform-Specific Optimizations** - Linux x86_64[^63]
  - *Primary optimization target for GPU acceleration via Docker*

## Verification

### Verification Targets
- **Verification Target** - On-chain EVM, On-chain Solana[^64]
  - *Comprehensive support for Ethereum-compatible chains and Solana*
- **Other Blockchain Support** - Arbitrum, Base, Optimism, Scroll, BNB Chain[^65]
  - *Deployed verifier gateways across major Layer 2 networks*

### Cost Analysis
- **EVM Verification Gas Cost** - 270000 (Groth16), 300000 (PLONK)[^66]
  - *Groth16: ~270k gas for optimal cost efficiency*
  - *PLONK: ~300k gas with no trusted setup requirement*
- **SVM Verification Compute Units** - Variable[^67]
  - *Solana verifier program supports multiple SP1 versions*
- **Cost Model Available** - Yes[^68]
  - *Prover gas estimation system provides cost modeling*

### Verifier Availability
- **Solidity Verifier Available** - Yes[^69]
  - *Canonical verifier gateways deployed across multiple EVM chains*
  - *Groth16 gateway: 0x397A5f7f3dBd538f23DE225B51f532c34448dA9B*
  - *PLONK gateway: 0x3B6041173B80E77f038f3F2C0f9744f04837185e*
- **Solana Verifier Available** - Yes[^70]
  - *Native Solana program for SP1 proof verification*
  - *Multiple version support (v2.0.0 through v5.0.0)*
- **Verifier Generation Tool** - Yes[^71]
  - *Automated verifier contract generation and deployment*

### Advanced Verification Features
- **Proof Aggregation Support** - Advanced[^72]
  - *Recursive proof verification within SP1 zkVM*
  - *Can aggregate multiple proofs into single on-chain verification*
  - *Supports heterogeneous proof batching*
- **Recursion Support** - Full IVC[^73]
  - *SP1 can verify SP1 proofs within itself*
  - *Enables incremental verifiable computation patterns*
  - *Supports proof composition and chaining*
- **Proof Composition** - Recursive[^74]
  - *Multi-stage recursive proving pipeline*
  - *Compress → Shrink → Wrap stages enable efficient composition*

## Development and Production Status

### Maturity
- **Development Status** - Production[^75]
  - *SP1 v5.2.1 represents a mature, production-ready zkVM*
- **Security Audit Status** - Multiple[^76]
  - *Comprehensive audits by Kalos (rkm0959), Cantina, Veridise, and Zellic*
  - *Dedicated recursion VM audit covering AIR constraints*
  - *Multiple audit rounds covering core components*
- **Vendor Suggests Production Use** - Yes[^77]
  - *Succinct Labs explicitly promotes SP1 for production deployment*
- **API Stability** - Stable[^78]
  - *Version 5.x represents stable API with semantic versioning*
- **First Commit Date** - 2023-12-04T16:54:45-08:00[^79]
- **Last Commit Date** - 2025-08-22T11:07:53-07:00[^80]
- **Total Commits** - 6271[^81]
- **Commit Frequency** - 10.02 commits/day[^82]

### Platform Support
- **Supported Operating Systems** - Linux, macOS, Windows[^83]
  - *Cross-platform support through Rust ecosystem*
- **Supported Architectures** - x86_64, ARM64[^84]
  - *Native support for both Intel/AMD and Apple Silicon*
- **Build Requirements** - Rust stable toolchain, LLVM tools[^85]
  - *Standard Rust build environment with additional LLVM components*
- **External Dependencies** - Docker (for GPU acceleration)[^86]
  - *GPU proving requires Docker for Moongate server deployment*

### Toolchain Integration
- **Compiler Support** - Rust[^87]
  - *Primary support for Rust with SP1 derive macros and runtime*
- **Compiler Version** - Rust stable channel[^88]
  - *Uses standard Rust stable toolchain with LLVM tools component*
- **IDE Integration** - VS Code, CLI[^89]
  - *Command-line interface with cargo-like workflow*
- **Debugging Support** - Yes[^90]
  - *Execution tracing and cycle tracking for program analysis*
- **Profiling Tools** - Yes[^91]
  - *Built-in cycle tracking and prover gas estimation tools*

## Notes

### Security Considerations
- **Security Warnings** - GPU acceleration relies on proprietary Moongate server components that are not auditable. Users concerned about supply chain security should use CPU-only proving.
- **Known Limitations** - RV32IM subset limits compatibility with some Rust crates that require full RISC-V support or specific instruction set extensions.
- **Recommended Use Cases** - Cryptographic applications, blockchain state transitions, proof aggregation, and applications requiring extensive elliptic curve or hash operations.

### Additional Information
- **Performance Benchmarks** - SP1 provides comprehensive benchmarking through prover gas estimation and cycle tracking tools[^92]
- **Notable Features** - Extensive cryptographic precompiles, multi-stage proving pipeline, production-grade verifier infrastructure, and active development community
- **Community/Ecosystem** - Strong integration with Ethereum and Solana ecosystems, active Discord community, and comprehensive documentation website

## Documentation Quality

### Overall Assessment
SP1's documentation demonstrates exceptional quality and comprehensiveness, setting a high standard for zkVM projects. The documentation combines clear conceptual explanations with practical implementation guidance, supported by working examples and comprehensive API references[^93]. The structure progresses logically from basic concepts to advanced features, making it accessible to both newcomers and experienced developers.

### Strengths
- Clear installation and quickstart guides with multiple setup options[^94]
- Comprehensive coverage of all major features including precompiles, hardware acceleration, and proof types[^95]
- Extensive example collection covering real-world use cases from basic Fibonacci to complex cryptographic operations[^96]
- Well-structured API documentation with code samples and parameter explanations[^97]
- Advanced topics thoroughly covered including proof aggregation, recursion, and performance optimization[^98]
- Active maintenance with documentation updates accompanying code releases[^99]

### Areas for Improvement
- GPU acceleration setup could benefit from more detailed troubleshooting guidance for Docker configuration issues
- Some advanced recursion patterns could use additional worked examples beyond the basic aggregation case
- Performance benchmarking methodology could be documented more systematically

### Coverage Analysis
- **Well-documented areas**: Installation, basic usage, precompiles, verification, proof types, examples
- **Partially documented areas**: Advanced recursion patterns, custom precompile development
- **Missing or inadequate documentation**: Internal GPU kernel implementations (due to proprietary nature), advanced AIR constraint development

### Documentation Metrics
- **Total documentation pages**: 40+ comprehensive guides and references[^100]
- **Code examples count**: 12+ complete example projects with detailed walkthroughs[^101]
- **Tutorial/guide count**: 25+ structured learning materials[^102]
- **Last documentation update**: Actively maintained with v5.2.1 release[^103]

### Note to User
SP1 represents the current state-of-the-art in production-ready zkVMs, combining high performance with comprehensive feature sets and excellent developer experience. The system's multi-stage proving pipeline effectively balances the trade-offs between proving time, proof size, and verification cost, making it suitable for a wide range of applications from development prototyping to production deployment.

**Strengths**: SP1 excels in several key areas: (1) Performance - extensive cryptographic precompiles and GPU acceleration enable practical proving times for complex programs; (2) Production readiness - comprehensive audit coverage, stable APIs, and deployed verifier infrastructure across major blockchains; (3) Developer experience - excellent documentation, extensive examples, and intuitive toolchain integration; (4) Flexibility - multi-stage proving pipeline allows optimization for different use cases.

**Limitations**: The main concerns center around vendor dependence for GPU acceleration, where the Moongate server components are proprietary and not auditable. The RV32IM instruction subset, while performant, may limit compatibility with some Rust crates. The multi-stage proving pipeline, while flexible, adds complexity compared to simpler single-stage systems.

**Recommendation**: SP1 is highly recommended for teams building production applications requiring zero-knowledge proofs, particularly those involving cryptographic operations, blockchain integrations, or proof aggregation. The combination of performance, security audits, and production infrastructure makes it the strongest choice for serious deployment. However, teams with strict open-source requirements should note the proprietary nature of GPU acceleration components and consider CPU-only deployment if supply chain auditability is critical.

--- 

### Citations 

[^6]: [SP1 README - High-performance zkVM](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/README.md)
[^7]: [Multi-stage proving architecture](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/prover/src/lib.rs#L1-L8)
[^8]: [STARK to SNARK conversion](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/prover/src/lib.rs#L118-L120)
[^9]: [RV32IM instruction set](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/core/executor/src/opcode.rs#L21-L22)
[^10]: [Main execution loop](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/core/executor/src/executor.rs#L1696-L1825)
[^11]: [RISC-V register implementation](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/core/executor/src/register.rs#L6-L70)
[^12]: [Memory model](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/core/executor/src/memory.rs)
[^13]: [Custom instruction encoding](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/core/executor/src/opcode.rs#L15-L19)
[^14]: [System call interface](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/core/executor/src/syscalls/mod.rs)
[^15]: [Execution trace structure](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/core/executor/src/record.rs#L28-L80)
[^16]: [Trace generation modes](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/core/executor/src/executor.rs#L194-L202)
[^17]: [Dynamic shard sizing](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/stark/src/opts.rs#L34-L42)
[^18]: [Memory checkpointing](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/stark/src/opts.rs#L106-L166)
[^19]: [Plonky3 framework dependencies](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/Cargo.toml#L76-L97)
[^20]: [Baby Bear field configuration](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/stark/src/config.rs)
[^21]: [AIR constraint system](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/stark/src/lib.rs)
[^22]: [Core proving stage](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/prover/src/lib.rs#L105-L111)
[^23]: [Compress/recursion stage](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/recursion/core/src/stark/config.rs)
[^24]: [Shrink stage](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/prover/src/lib.rs#L118-L120)
[^25]: [BN254 wrap stage](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/prover/src/lib.rs#L118-L120)
[^26]: [Gas cost documentation](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/sdk/src/cuda/prove.rs#L74-L100)
[^27]: [Solana verifier program](https://github.com/succinctlabs/sp1-solana/blob/4181cae00d7493ede8a33066cb56683acb1dca72/verifier/src/lib.rs)
[^28]: [Proof aggregation example](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/examples/aggregation/script/src/main.rs)
[^29]: [Cryptographic precompiles](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/core/executor/src/syscalls/code.rs)
[^30]: [GPU acceleration via Moongate](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/cuda/src/lib.rs#L39-L49)
[^31]: [Multi-stage pipeline](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/sdk/src/cuda/mod.rs#L82-L159)
[^32]: [RISC-V architecture](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/core/executor/src/opcode.rs#L54)
[^33]: [RV32IM subset](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/core/executor/src/opcode.rs#L28-L105)
[^34]: [Precompile system calls](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/core/executor/src/syscalls/code.rs)
[^35]: [Continuation support](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/core/executor/src/executor.rs#L194-L202)
[^36]: [Shard-based segmentation](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/stark/src/opts.rs#L106-L166)
[^37]: [Shard size configuration](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/stark/src/opts.rs#L34-L42)
[^38]: [Checkpoint components](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/core/executor/src/memory.rs)
[^39]: [Configurable backend system](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/prover/src/lib.rs#L105-L111)
[^40]: [Plonky3 framework](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/Cargo.toml#L76-L97)
[^41]: [Plonky3 version](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/Cargo.toml#L77)
[^42]: [AIR constraint language](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/stark/src/lib.rs)
[^43]: [Core backend configuration](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/stark/src/config.rs)
[^44]: [Baby Bear field](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/Cargo.toml#L79)
[^45]: [Poseidon2 hash function](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/Cargo.toml#L81)
[^46]: [Inner backend config](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/recursion/core/src/stark/config.rs)
[^47]: [Outer backend config](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/recursion/core/src/stark/config.rs)
[^48]: [Multi-stage security model](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/prover/src/lib.rs#L1-L8)
[^49]: [BN254 curve usage](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/Cargo.toml#L80)
[^50]: [Hybrid proving design](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/prover/src/lib.rs#L118-L120)
[^51]: [Hardware acceleration support](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/cuda/Cargo.toml)
[^52]: [NVIDIA GPU support](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/cuda/src/lib.rs#L211)
[^53]: [No GPU kernels in repo](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/cuda/src/lib.rs#L39-L49)
[^54]: [External GPU dependency](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/cuda/src/lib.rs#L206-L207)
[^55]: [Docker container implementation](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/cuda/src/lib.rs#L225-L243)
[^56]: [Opaque GPU implementation](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/cuda/src/lib.rs#L39-L49)
[^57]: [Proprietary acceleration](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/cuda/src/lib.rs#L206-L207)
[^58]: [Moongate server licensing](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/cuda/src/lib.rs#L206-L207)
[^59]: [Hybrid CPU+GPU proving](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/sdk/src/cuda/mod.rs#L20-L34)
[^60]: [GPU memory requirements](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/cuda/src/lib.rs#L290)
[^61]: [Docker-managed resources](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/cuda/src/lib.rs#L225-L243)
[^62]: [Memory optimization](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/stark/src/opts.rs#L106-L166)
[^63]: [Linux platform optimization](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/cuda/src/lib.rs#L225-L243)
[^64]: [EVM and Solana support](https://github.com/succinctlabs/sp1-solana/blob/4181cae00d7493ede8a33066cb56683acb1dca72/verifier/src/lib.rs)
[^65]: [Multi-chain verifier deployment](https://github.com/succinctlabs/sp1-solana/blob/4181cae00d7493ede8a33066cb56683acb1dca72/verifier/src/lib.rs)
[^66]: [Verification gas costs](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/sdk/src/cuda/prove.rs#L74-L100)
[^67]: [Solana compute units](https://github.com/succinctlabs/sp1-solana/blob/4181cae00d7493ede8a33066cb56683acb1dca72/verifier/src/lib.rs)
[^68]: [Prover gas system](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/core/executor/src/executor.rs)
[^69]: [Solidity verifier gateways](https://github.com/succinctlabs/sp1-solana/blob/4181cae00d7493ede8a33066cb56683acb1dca72/verifier/src/lib.rs)
[^70]: [Solana verifier program](https://github.com/succinctlabs/sp1-solana/blob/4181cae00d7493ede8a33066cb56683acb1dca72/verifier/src/lib.rs)
[^71]: [Verifier generation tools](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/prover/scripts/build_groth16_bn254.rs)
[^72]: [Advanced proof aggregation](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/examples/aggregation/script/src/main.rs)
[^73]: [Recursive proof verification](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/examples/aggregation/script/src/main.rs)
[^74]: [Multi-stage composition](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/prover/src/lib.rs#L118-L120)
[^75]: [Production status](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/README.md)
[^76]: [Security audits](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/audits/)
[^77]: [Production readiness](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/README.md)
[^78]: [API stability](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/SP1_VERSION)
[^79]: [Repository history](https://github.com/succinctlabs/sp1)
[^80]: [Recent activity](https://github.com/succinctlabs/sp1)
[^81]: [Commit count](https://github.com/succinctlabs/sp1)
[^82]: [Development velocity](https://github.com/succinctlabs/sp1)
[^83]: [Cross-platform support](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/rust-toolchain.toml)
[^84]: [Architecture support](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/Cargo.toml)
[^85]: [Build requirements](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/rust-toolchain.toml)
[^86]: [Docker dependency](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/cuda/src/lib.rs#L192-L216)
[^87]: [Rust compiler support](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/Cargo.toml)
[^88]: [Rust toolchain](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/rust-toolchain.toml)
[^89]: [CLI interface](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/cli/)
[^90]: [Debugging features](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/core/executor/src/executor.rs)
[^91]: [Profiling tools](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/perf/)
[^92]: [Performance benchmarking](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/perf/)
[^93]: [Documentation structure](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/README.md)
[^94]: [Installation guides](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/README.md)
[^95]: [Feature documentation](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/examples/)
[^96]: [Example collection](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/examples/)
[^97]: [API documentation](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/crates/sdk/)
[^98]: [Advanced topics](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/examples/aggregation/)
[^99]: [Documentation maintenance](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/README.md)
[^100]: [Documentation pages](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/README.md)
[^101]: [Code examples](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/examples/)
[^102]: [Learning materials](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/examples/)
[^103]: [Documentation updates](https://github.com/succinctlabs/sp1/blob/3209d5454e8cddb635dd8425e3807a60e8dac395/README.md)