# ZKVM Evaluation Report: Jolt (v1.1.1)

## Basic Information
- **Name** - Jolt[^1]
- **Team Affiliation** - a16z crypto[^2]
- **Repositories Considered:**
  - **Name** - jolt
    - **Version Number** - v1.1.1 (spike-v1.1.1-51-g1a62a0d7)[^3]
    - **Repository URL** - https://github.com/a16z/jolt
    - **Commit** - 1a62a0d73fb84ededb27e72422ebc112e96262da
- **License Type** - MIT OR Apache 2.0[^4][^5]

## Technical Overview

### How It Works

**Design Philosophy and Architecture**

Jolt is a zkVM designed around the principle of "Just One Lookup Table"[^6], implementing a novel approach to zero-knowledge virtual machines. The system is built specifically for RISC-V and emphasizes simplicity and extensibility over other design goals. Rather than using traditional STARK-based approaches, Jolt employs a custom protocol centered on Lasso lookup arguments[^7] combined with sum-check protocols for efficient constraint satisfaction.

The architecture prioritizes proving simplicity through lookup tables rather than complex constraint systems. Jolt decomposes RISC-V instruction execution into lookups against precomputed tables, avoiding the need for complex arithmetic circuits that characterize other zkVM designs. This "lookup singularity" approach trades some generality for significant improvements in proving time and system complexity.

**Execution Model**

Programs are executed through a RISC-V emulator that generates execution traces[^8]. The VM implements the RV32IM instruction set architecture (32-bit base integer instruction set plus multiplication/division extension)[^9]. The execution environment provides a standard RISC-V memory model with 32-bit addressing, register file management, and stack organization.

The system uses a tracer-based approach where RISC-V programs are first executed to completion while recording every instruction execution[^10]. Each instruction execution creates a cycle record containing program counter, register states, memory accesses, and instruction details. The tracer handles both real RISC-V instructions and virtual instruction sequences that decompose complex operations into simpler lookup-friendly components.

**Trace Generation and Witness Construction**

Execution traces capture the complete execution history including program counter values, register file states, memory access patterns, and instruction operands[^11]. The witness construction process converts these traces into polynomial representations suitable for the proving system. 

Key components include bytecode preprocessing[^12] that converts the program binary into prover-friendly representations, RAM preprocessing[^13] for memory initialization state, and instruction lookup preprocessing that builds the necessary lookup tables for instruction validation.

**Constraint System and Arithmetization**

Rather than using traditional R1CS or AIR-based constraint systems, Jolt employs a hybrid approach combining Lasso lookup arguments with sum-check protocols[^14]. The constraint generation focuses on three main areas: instruction lookups (validating that each instruction execution is correct), memory consistency (ensuring memory reads/writes are coherent), and register file management.

The system uses specialized sum-check instances for different components: bytecode checking, memory checking, register checking, and instruction lookup validation[^15]. This modular approach allows for efficient proving by avoiding monolithic constraint systems.

**Proving Process**

The end-to-end proving pipeline operates through a multi-stage process managed by the JoltDAG system[^16]. The prover first preprocesses the execution trace, then generates polynomial commitments for all trace components. Multiple sum-check protocols run in parallel to validate different aspects of the execution: instruction correctness, memory consistency, and register file operations.

The proving process uses the Dory polynomial commitment scheme over the BN254 curve[^17], providing the cryptographic foundations for the sum-check protocols. Memory management during proving is optimized through segment-based processing and efficient polynomial operations.

**Verification and Deployment**

The verification process validates the sum-check proofs and polynomial openings without requiring knowledge of the execution trace[^18]. The verifier performs approximately 10 sum-check verifications covering instruction lookups, memory checking, and register operations, along with validating polynomial commitment openings.

Verification costs are estimated at approximately 2 million gas for on-chain EVM deployment[^19], with proof sizes in the dozens of kilobytes. The system includes plans for Groth16 composition to reduce on-chain verification costs through proof compression techniques.

**Key Differentiators**

Jolt's primary innovation lies in its lookup-centric approach to zkVM design, avoiding complex constraint generation in favor of precomputed lookup tables. This results in significantly simpler proving circuits and potentially faster proving times compared to traditional approaches. The system also provides inline instruction support[^20] for optimized cryptographic operations like SHA256, allowing efficient integration of precompiled functionality.

The non-recursive approach to space control represents another key differentiator, with plans for "streaming prover" techniques that avoid the complexity of proof recursion while maintaining constant memory usage[^21].

## Architecture

### Core Architecture
- **Instruction Set Architecture** - RISC-V[^22]
- **ISA Extensions/Profile** - RV32IM (32-bit Base Integer + M Standard Extension for Integer Multiplication and Division)[^23]

### Optimization Features
- **Precompiles/Built-ins** - SHA256[^24]

### Trace Generation Model
- **Continuations Support** - None (planned)[^25]
- **Segmentation Model** - Monolithic chunking planned for memory control[^26]
- **Default Segment Size** - Fixed trace length, padded to next power of 2[^27]
- **Checkpoint State Components** - Program Counter, Register File, Memory State[^28]

## Proof System
- **Backend Model** - Fixed[^29]

### Proof System Framework
- **Framework Name** - Custom Jolt Protocol (Lasso + Sum-check)[^30]
- **Framework Version** - v1.1.1
- **Constraint Language** - Custom lookup + sum-check DSL[^31]

### Default Backend Configuration:
- **Backend/Config Name** - JoltRV32IM[^32]
- **Proof System Type** - Hybrid (Custom lookup arguments + sum-check)[^33]
- **Arithmetization** - Custom (Lasso lookup arguments)[^34]
- **Field/Curve** - BN254[^35]
- **Field Size** - 254 bits[^36]
- **Field Modulus** - BN254 scalar field modulus[^37]
- **Commitment Scheme** - Dory[^38]
- **Hash Function** - Blake2b[^39]
- **Trusted Setup Required** - Inherited (uses Dory's universal setup)[^40]
- **Default Backend** - Yes

### Post-Quantum Security
- **Core Proof System Post-Quantum Security** - Vulnerable[^41]
- **Quantum-Vulnerable Components** - Elliptic Curve Commitments[^42]
- **Post-Quantum Alternative Available** - No
- **Quantum Security Notes** - Dory commitment scheme relies on elliptic curve assumptions vulnerable to quantum attacks

## Acceleration Support

### Hardware Support
- **Hardware Acceleration** - None (planned)[^43]
- **GPU Vendor Support** - N.A.

### GPU Code Availability and Licensing
- **GPU Code in Repository** - No
- **GPU Code License** - N.A.
- **GPU Implementation Method** - N.A.
- **GPU Code Transparency** - N.A.

### General Acceleration Support
- **Acceleration Open Source** - N.A.
- **Acceleration License** - N.A.

### Execution Distribution
- **Prover Distribution** - Multi-threaded CPU[^44]
- **Memory Requirements per Process** - Variable (depends on trace length)
- **Recommended Process/Thread Configuration** - Uses Rayon for parallelization[^45]

### Performance Optimization
- **Assembly Backend Available** - No
- **Shared Memory Optimization** - Yes (Rayon parallel iterators)[^46]
- **Platform-Specific Optimizations** - N.A.

## Verification

### Verification Targets
- **Verification Target** - Off-chain (on-chain planned)[^47]
- **Other Blockchain Support** - None currently

### Cost Analysis
- **EVM Verification Gas Cost** - ~2,000,000 (estimated)[^48]
- **SVM Verification Compute Units** - N.A.
- **Cost Model Available** - Yes (rough estimates)[^49]

### Verifier Availability
- **Solidity Verifier Available** - No (planned via Groth16 composition)[^50]
- **Solana Verifier Available** - No
- **Verifier Generation Tool** - No

### Advanced Verification Features
- **Proof Aggregation Support** - None (planned via Groth16)[^51]
- **Recursion Support** - None (example implementation exists)[^52]
- **Proof Composition** - None (planned via Spartan + Groth16)[^53]

## Development and Production Status

### Maturity
- **Development Status** - Alpha[^54]
- **Security Audit Status** - None
- **Vendor Suggests Production Use** - No[^55]
- **API Stability** - Evolving
- **First Commit Date** - 2019-12-16[^56]
- **Last Commit Date** - 2025-08-21[^57]
- **Total Commits** - 3024[^58]
- **Commit Frequency** - 1.46 commits/day[^59]

### Platform Support
- **Supported Operating Systems** - Linux, macOS, Windows[^60]
- **Supported Architectures** - x86_64, ARM64[^61]
- **Build Requirements** - Rust nightly toolchain[^62]
- **External Dependencies** - Dory commitment scheme library[^63]

### Toolchain Integration
- **Compiler Support** - Rust[^64]
- **Compiler Version** - Custom RISC-V Rust toolchain[^65]
- **IDE Integration** - CLI[^66]
- **Debugging Support** - Limited (cycle tracking)[^67]
- **Profiling Tools** - Yes (execution and memory profiling)[^68]

## Notes

### Security Considerations
- **Security Warnings** - Alpha software, not production ready[^69]
- **Known Limitations** - Limited ISA support (RV32IM only), no continuations, no on-chain verifier
- **Recommended Use Cases** - Research, prototyping, educational purposes

### Additional Information
- **Performance Benchmarks** - Benchmarking infrastructure available[^70]
- **Notable Features** - Lookup-centric design, inline SHA256 support, non-recursive proving approach
- **Community/Ecosystem** - Academic and research focused, open development

### Note to User

Jolt represents an innovative approach to zkVM design with its lookup-centric architecture that significantly differs from traditional STARK or SNARK-based systems. The system shows promise for improved proving performance through its "Just One Lookup Table" philosophy, but remains in alpha stage with limited production readiness.

**Strengths:**
- Novel and potentially more efficient proving approach through lookup tables
- Clean, well-documented codebase with strong theoretical foundations
- Active development with frequent commits and good engineering practices
- Dual MIT/Apache licensing provides flexibility

**Limitations:**
- Alpha maturity level - not suitable for production use
- Limited ISA support (only RV32IM, no floating point or other extensions)
- No hardware acceleration support
- No on-chain verification capabilities (planned)
- Post-quantum vulnerable due to elliptic curve dependencies
- No security audits completed

**Recommendation:**
Jolt is excellent for research, prototyping, and educational purposes, particularly for teams interested in exploring lookup-based zkVM architectures. However, it is not recommended for production applications due to its alpha status, limited feature set, and lack of security audits. Teams requiring production-ready zkVMs with broader ISA support, hardware acceleration, or on-chain verification should consider more mature alternatives. For those willing to work with cutting-edge technology and potentially contribute to development, Jolt offers an interesting alternative approach to zkVM design.

---

### Citations

[^1]: [README.md](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/README.md)
[^2]: [Cargo.toml - Repository field](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/Cargo.toml#L19)
[^3]: [Cargo.toml - Version](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/Cargo.toml#L3)
[^4]: [LICENSE-MIT](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/LICENSE-MIT)
[^5]: [LICENSE-APACHE](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/LICENSE-APACHE)
[^6]: [README.md - "Just One Lookup Table"](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/README.md#L5)
[^7]: [README.md - Lasso paper link](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/README.md#L22)
[^8]: [jolt-core/src/guest/program.rs](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/jolt-core/src/guest/program.rs)
[^9]: [README.md - RV32IM description](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/README.md#L7)
[^10]: [tracer/src/emulator/cpu.rs](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/tracer/src/emulator/cpu.rs)
[^11]: [jolt-core/src/zkvm/witness.rs](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/jolt-core/src/zkvm/witness.rs)
[^12]: [jolt-core/src/zkvm/bytecode/mod.rs](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/jolt-core/src/zkvm/bytecode/mod.rs)
[^13]: [jolt-core/src/zkvm/ram/mod.rs](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/jolt-core/src/zkvm/ram/mod.rs)
[^14]: [jolt-core/src/subprotocols/sumcheck.rs](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/jolt-core/src/subprotocols/sumcheck.rs)
[^15]: [jolt-core/src/zkvm/instruction_lookups/mod.rs](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/jolt-core/src/zkvm/instruction_lookups/mod.rs)
[^16]: [jolt-core/src/zkvm/dag/jolt_dag.rs](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/jolt-core/src/zkvm/dag/jolt_dag.rs)
[^17]: [jolt-core/src/zkvm/mod.rs#L310](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/jolt-core/src/zkvm/mod.rs#L310)
[^18]: [jolt-core/src/zkvm/mod.rs#L255-L306](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/jolt-core/src/zkvm/mod.rs#L255-L306)
[^19]: [book/src/future/on-chain-verifier.md](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/book/src/future/on-chain-verifier.md)
[^20]: [jolt-inlines/sha2/src/lib.rs](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/jolt-inlines/sha2/src/lib.rs)
[^21]: [book/src/future/continuations.md](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/book/src/future/continuations.md)
[^22]: [tracer/src/instruction/mod.rs](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/tracer/src/instruction/mod.rs)
[^23]: [book/src/background/risc-v.md](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/book/src/background/risc-v.md)
[^24]: [jolt-inlines/sha2/src/sdk.rs](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/jolt-inlines/sha2/src/sdk.rs)
[^25]: [book/src/future/continuations.md](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/book/src/future/continuations.md)
[^26]: [book/src/future/continuations.md](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/book/src/future/continuations.md)
[^27]: [jolt-core/src/zkvm/mod.rs#L236-L237](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/jolt-core/src/zkvm/mod.rs#L236-L237)
[^28]: [common/src/jolt_device.rs](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/common/src/jolt_device.rs)
[^29]: [jolt-core/src/zkvm/mod.rs#L309-L311](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/jolt-core/src/zkvm/mod.rs#L309-L311)
[^30]: [jolt-core/src/zkvm/mod.rs](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/jolt-core/src/zkvm/mod.rs)
[^31]: [jolt-core/src/zkvm/lookup_table/mod.rs](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/jolt-core/src/zkvm/lookup_table/mod.rs)
[^32]: [jolt-core/src/zkvm/mod.rs#L309](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/jolt-core/src/zkvm/mod.rs#L309)
[^33]: [jolt-core/src/zkvm/mod.rs](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/jolt-core/src/zkvm/mod.rs)
[^34]: [jolt-core/src/zkvm/instruction_lookups/mod.rs](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/jolt-core/src/zkvm/instruction_lookups/mod.rs)
[^35]: [jolt-core/src/zkvm/mod.rs#L23](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/jolt-core/src/zkvm/mod.rs#L23)
[^36]: [jolt-core/src/field/ark.rs](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/jolt-core/src/field/ark.rs)
[^37]: [jolt-core/src/field/ark.rs](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/jolt-core/src/field/ark.rs)
[^38]: [jolt-core/src/poly/commitment/dory.rs](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/jolt-core/src/poly/commitment/dory.rs)
[^39]: [jolt-core/src/transcripts/blake2b.rs](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/jolt-core/src/transcripts/blake2b.rs)
[^40]: [jolt-core/src/poly/commitment/dory.rs](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/jolt-core/src/poly/commitment/dory.rs)
[^41]: [jolt-core/src/poly/commitment/dory.rs](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/jolt-core/src/poly/commitment/dory.rs)
[^42]: [jolt-core/src/poly/commitment/dory.rs](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/jolt-core/src/poly/commitment/dory.rs)
[^43]: [book/src/future/improvements-since-release.md](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/book/src/future/improvements-since-release.md)
[^44]: [jolt-core/src/zkvm/mod.rs#L200](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/jolt-core/src/zkvm/mod.rs#L200)
[^45]: [jolt-core/src/zkvm/mod.rs#L200](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/jolt-core/src/zkvm/mod.rs#L200)
[^46]: [jolt-core/src/zkvm/mod.rs#L200](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/jolt-core/src/zkvm/mod.rs#L200)
[^47]: [book/src/future/on-chain-verifier.md](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/book/src/future/on-chain-verifier.md)
[^48]: [book/src/future/on-chain-verifier.md#L3](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/book/src/future/on-chain-verifier.md#L3)
[^49]: [book/src/future/on-chain-verifier.md](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/book/src/future/on-chain-verifier.md)
[^50]: [book/src/future/groth-16.md](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/book/src/future/groth-16.md)
[^51]: [book/src/future/groth-16.md](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/book/src/future/groth-16.md)
[^52]: [examples/recursion/src/main.rs](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/examples/recursion/src/main.rs)
[^53]: [book/src/future/groth-16.md](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/book/src/future/groth-16.md)
[^54]: [README.md#L28](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/README.md#L28)
[^55]: [README.md#L28](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/README.md#L28)
[^56]: Git log analysis from repository metadata
[^57]: Git log analysis from repository metadata  
[^58]: Git log analysis from repository metadata
[^59]: Git log analysis from repository metadata
[^60]: [rust-toolchain.toml](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/rust-toolchain.toml)
[^61]: [rust-toolchain.toml](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/rust-toolchain.toml)
[^62]: [README.md#L36](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/README.md#L36)
[^63]: [Cargo.toml - Dory dependency](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/jolt-core/Cargo.toml)
[^64]: [rust-toolchain.toml](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/rust-toolchain.toml)
[^65]: [guest-toolchain-tag](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/guest-toolchain-tag)
[^66]: [src/main.rs](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/src/main.rs)
[^67]: [jolt-platform/src/cycle_tracking.rs](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/jolt-platform/src/cycle_tracking.rs)
[^68]: [README.md#L72-L103](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/README.md#L72-L103)
[^69]: [README.md#L28](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/README.md#L28)
[^70]: [README.md#L100-L102](https://github.com/a16z/jolt/blob/1a62a0d73fb84ededb27e72422ebc112e96262da/README.md#L100-L102)