# Master Claude Code Prompts: Advanced Systems & Architectural Blueprints

A curated collection of production-grade, ultra-complex prompts designed specifically for **Claude Code CLI**. These prompts leverage Claude Code's long-context window, file-system interaction, terminal execution, and deep reasoning capabilities to architect complex software systems, game engines, and infrastructure.

---

## 📋 Table of Contents
1. [Game Engineering: Fortnite AAA Prototype & Engine Architecture](#1-game-engineering-fortnite-aaa-prototype--engine-architecture)
2. [AI & Systems: Autonomous Multi-Agent Software Engineering Company](#2-ai--systems-autonomous-multi-agent-software-engineering-company)
3. [FinTech: Ultra-Low Latency HFT Matching Engine & FIX Protocol](#3-fintech-ultra-low-latency-hft-matching-engine--fix-protocol)
4. [Cybersecurity: Zero-Trust Security Sentinel & Auto-Patch Agent](#4-cybersecurity-zero-trust-security-sentinel--auto-patch-agent)
5. [Cloud Infrastructure: Kubernetes-Native Scale-to-Zero Serverless Controller](#5-cloud-infrastructure-kubernetes-native-scale-to-zero-serverless-controller)
6. [Low-Level Systems: Microkernel Operating System in Rust](#6-low-level-systems-microkernel-operating-system-in-rust)
7. [Database Engineering: Distributed Time-Series Database Engine](#7-database-engineering-distributed-time-series-database-engine)
8. [Graphics & Engine: Real-Time Vulkan Ray Tracing Engine](#8-graphics--engine-real-time-vulkan-ray-tracing-engine)
9. [Web3 & Distributed Systems: Custom Layer-1 Blockchain Node](#9-web3--distributed-systems-custom-layer-1-blockchain-node)
10. [Compiler Construction: Production-Grade Programming Language Compiler](#10-compiler-construction-production-grade-programming-language-compiler)

---

## 1. Game Engineering: Fortnite AAA Prototype & Engine Architecture

```text
Act as a Principal Game Architect and Senior C++/Engine Developer. I want to construct a production-grade, modular prototype of a Fortnite-style Battle Royale game using modern C++20 and Unreal Engine C++ / Native Game Engine Architecture.

You will generate the full directory structure, build scripts (CMake/Unreal Build Tool), header files, and fully commented, production-grade implementation files step-by-step.

### ARCHITECTURAL MODULES & SUBSYSTEMS:

1. Building & Grid Destruction Framework:
   - Dynamic 3D Grid System: Snap building modules (Walls, Ramp/Stairs, Floors, Pyramids/Cones) to a world space spatial grid.
   - Edit Mode Subsystem: Real-time dynamic mesh slicing and sub-grid toggling (editing 3x3 wall tiles to create windows, single doors, double doors, archways).
   - Graph-Based Structural Integrity Algorithm: Directed Acyclic Graph (DAG) for structural dependencies. If ground connection nodes are destroyed, execute topological sorting to detect orphan structures, triggering a physics destruction impulse cascade with fragment spawning.
   - Resource & Material Physics: Wood, Stone, and Metal material classes with distinct hitpoints, build-up opacity/scaling animations, and structural weakness critical hit spots.

2. Ballistics & Weapon Handling System:
   - Ballistics Architecture: Hybrid engine handling instant-hit (Hitscan with distance falloff & dynamic spread) and simulated Physical Projectiles (Bullet drop via gravity vector, wind drift, air resistance drag equation, dynamic velocity simulation).
   - Seed-Based Bloom & Dynamic Recoil: Pseudo-random deterministic seed bloom system, dynamic recoil recovery curves, crouch/sprint/jump/ADS spread multipliers.
   - Dynamic Loot & Rarity Engine: Weapon tiers (Common, Uncommon, Rare, Epic, Legendary, Mythic) with dynamic stat modifiers, auto-pickup radius checks, and inventory stack management.

3. Server-Authoritative Netcode & Prediction:
   - Client-Side Prediction & Reconciliation: Deterministic movement physics execution on client with server re-simulation on client error drift.
   - Rewind Buffer Lag Compensation: Server maintains a circular history buffer (200ms) of player bounding boxes. Rewind player hitboxes back in time to match high-latency client timestamp on firing events.
   - Spatial Prioritization & Relevance Culling: Dynamic grid-based network relevenancy manager to optimize packet bandwidth for up to 100 concurrent players.

4. Game State & Safe Zone (Storm) System:
   - Dynamic Safe Zone Shrink Mechanics: Multi-phase shrinking circle algorithm with non-concentric target selection, random displacement vectors, and storm wall particle mesh generation.
   - Dynamic Storm Damage Engine: Scalable damage-over-time tick system with dynamic post-processing color grading and audio filtering shifts based on player distance inside the storm.

Output the full project structure, CMakeLists.txt build file, interface headers, and complete C++ source files for the Building Structural Integrity Graph and the Ballistics Engine.
```

---

## 2. AI & Systems: Autonomous Multi-Agent Software Engineering Company

```text
Act as an AI Systems Architect and Distributed Systems Developer. Build a complete, production-ready Python framework that instantiates an autonomous, self-correcting multi-agent AI software development enterprise capable of generating full-stack software applications from a single natural language spec.

### ARCHITECTURE & AGENT SPECIFICATIONS:

1. Agent Roles & State Machine:
   - `ProductManagerAgent`: Translates user requests into Software Requirements Specifications (SRS), User Stories, and API contracts.
   - `SoftwareArchitectAgent`: Generates complete project dependency trees, database schemas (Prisma/SQLAlchemy), folder structures, and design pattern choices.
   - `SeniorDeveloperAgent`: Generates modular, clean, and typed code file by file across backend, frontend, and database layers.
   - `QAEngineerAgent`: Generates pytest/jest testing suites, executes them in an isolated subprocess container, captures stdout/stderr/exit codes, and feeds stack traces back to `SeniorDeveloperAgent` for auto-fixing.

2. Core Infrastructure & Orchestration:
   - Event-Driven Message Bus: Asyncio-based Event Loop using Pub/Sub model for agent communication.
   - Context Memory Engine: Vector database abstraction layer (ChromaDB/FAISS) with sliding context window optimization to maintain state awareness across hundreds of files.
   - Self-Healing Loop: Iterative execution loop (max retries: 5) that intercepts linter errors, compile errors, and runtime test failures, generating diff patches automatically until zero errors remain.

Generate the full repository layout, `pyproject.toml`, main CLI entry point, Agent Orchestrator, and the automated QA/Self-Healing execution pipeline.
```

---

## 3. FinTech: Ultra-Low Latency HFT Matching Engine & FIX Protocol

```text
Act as a Quantitative Systems Engineer and Low-Latency C++ Specialist. Construct an ultra-low latency, multi-threaded Limit Order Book (LOB) and Matching Engine in C++20 optimized for high-frequency trading (HFT).

### PERFORMANCE CONSTRAINTS & REQUIREMENTS:

1. Zero-Dynamic Memory Allocation:
   - Custom lock-free memory pools, intrusive doubly-linked lists, and lock-free ring buffers (`std::atomic` with explicit memory orderings).
   - Zero heap allocation (`malloc`/`new`) during operational execution loops.

2. Matching Engine Core Architecture:
   - Price-Time Priority (FIFO) matching algorithm achieving $O(1)$ time complexity for order insertion, cancellation, and execution at the top of the book.
   - Support for Limit, Market, Cancel-Replace, Fill-or-Kill (FOK), and Immediate-or-Cancel (IOC) order types.

3. FIX Protocol Engine & Network Stack:
   - Custom non-blocking parser for FIX 4.2 protocol and Simple Binary Encoding (SBE).
   - High-performance event loop built on Linux `epoll` or `io_uring` with kernel bypass abstractions.

4. Microsecond Benchmarking Suite:
   - Microsecond and nanosecond benchmarking hooks using hardware CPU cycle counters (`rdtsc`).

Provide the full project layout, CMakeLists.txt with high-level optimization flags (`-O3 -march=native -flto`), `OrderBook.hpp`, `MemoryPool.hpp`, and a latency benchmark suite targeting > 10M orders/sec.
```

---

## 4. Cybersecurity: Zero-Trust Security Sentinel & Auto-Patch Agent

```text
Act as a Staff Cybersecurity Engineer and Systems Programmer. Build a complete Python/Rust CLI utility named `ZeroTrust-Sentinel` that executes static application security testing (SAST), software composition analysis (SCA), and automated remediation patching on target software repositories.

### CORE ENGINE REQUIREMENTS:

1. Multi-Language AST SAST Engine:
   - Custom Abstract Syntax Tree (AST) analyzer for Python, JavaScript/TypeScript, and Rust.
   - Detect OWASP Top 10 vulnerabilities (SQL Injection, Remote Code Execution, SSRF, XSS, Unsafe Deserialization) and hardcoded secret entropy signatures.

2. AI-Driven Remediation & Containerized Verification Sandbox:
   - Automatically generate precise git patch diffs fixing identified security flaws.
   - Docker SDK / Podman integration to spin up ephemeral containers, run project test suites against applied patches, and verify non-regression before commit.

3. Rich Terminal Interface & SARIF Export:
   - Interactive terminal UI displaying interactive call graphs, risk scores (CVSS 3.1 calculation), and standardized SARIF report outputs for GitHub Security tab integration.

Output the entire directory tree, dependency configuration, AST rule engine module, Docker sandbox test manager, and interactive CLI implementation.
```

---

## 5. Cloud Infrastructure: Kubernetes-Native Scale-to-Zero Serverless Controller

```text
Act as a Principal Cloud Infrastructure Engineer and Go Developer. Design and build a production-grade Kubernetes Custom Controller and Custom Resource Definition (CRD) in Go (`controller-runtime`, `client-go`) named `MicroFunction`.

### SYSTEM CAPABILITIES:

1. Scale-To-Zero & Instant Cold-Start Activation:
   - Custom HTTP reverse proxy interceptor that buffers incoming requests when function replica count is 0.
   - Direct interaction with Kubernetes API to scale pods from 0 to 1 within <200ms upon initial HTTP trigger, then proxy the buffered payload.
   - Idle duration timer daemon that scales down deployments to 0 after configurable inactivity.

2. Custom Resource Definition (CRD) Architecture:
   - Go structs defining `MicroFunctionSpec` (image, environment, concurrency limit, idleTimeout, minReplicas, maxReplicas) and `MicroFunctionStatus`.

3. Telemetry & Metrics Exporter:
   - Native Prometheus metrics exporter tracking invocation latency histograms, cold start duration, and concurrency saturation.

Provide the complete `api/v1alpha1/microfunction_types.go`, the controller logic (`controllers/microfunction_controller.go`), the HTTP buffering proxy, and the deployment YAML manifests.
```

---

## 6. Low-Level Systems: Microkernel Operating System in Rust

```text
Act as an Operating Systems Architect and Rust Embedded Systems Engineer. Create a bare-metal `no_std` Microkernel in Rust targeting the x86_64 architecture.

### KERNEL SUBSYSTEMS:

1. Bootstrapping & Memory Management:
   - Multiboot2/UEFI boot sequence integration with entry point definition.
   - Physical Frame Allocator (Bitmap / Buddy Allocator) and Page Table Manager for dynamic virtual memory mapping.

2. Preemptive Multitasking & Scheduler:
   - Thread Control Block (TCB) structure, context switching assembly routines (`pusha`/`popa` state retention), and Round-Robin scheduler with hardware timer interrupts (PIT / APIC).

3. Inter-Process Communication (IPC):
   - Zero-copy synchronous message passing channel between isolated user-space driver tasks.

Output the target spec JSON, `Cargo.toml`, linker script (`linker.ld`), boot entry assembly, kernel main function, and page allocator module.
```

---

## 7. Database Engineering: Distributed Time-Series Database Engine

```text
Act as a Database Storage Engineer and Distributed Systems Developer. Construct a high-throughput, distributed Time-Series Database (TSDB) in Rust or C++ built for streaming IoT sensor metrics.

### CORE ARCHITECTURE:

1. Storage Engine (LSM-Tree Variant):
   - In-memory MemTable using Skiplist with Write-Ahead Logging (WAL) for durability.
   - Immutable SSTables (Sorted String Tables) with Bloom Filters, Gorilla compression algorithm for timestamp delta-of-delta and float XOR compression.

2. Query Engine & Aggregations:
   - SQL-like parser for time-window queries (e.g., `SELECT AVG(cpu) EVERY 5m GROUP BY host`).
   - Parallel query executor utilizing vectorized SIMD operations for rapid aggregations over compressed data blocks.

3. Distributed Consensus & Clustering:
   - Raft Consensus Protocol implementation for cluster leader election and log replication across nodes.

Provide the full directory structure, core storage layout, Gorilla compression codec, MemTable implementation, and node raft state synchronization logic.
```

---

## 8. Graphics & Engine: Real-Time Vulkan Ray Tracing Engine

```text
Act as a Graphics Hardware Specialist and Vulkan Engine Developer. Modern C++20 project creating a hardware-accelerated real-time Ray Tracing Engine using Vulkan (VK_KHR_ray_tracing_pipeline).

### ENGINE FEATURES:

1. Acceleration Structure Pipeline:
   - Bottom-Level Acceleration Structures (BLAS) for geometric mesh instances.
   - Top-Level Acceleration Structures (TLAS) managing spatial transformations and instance descriptors.

2. Shader Binding Table (SBT) & Ray Tracing Shaders:
   - GLSL / HLSL Ray Generation, Miss, Closest Hit, and Any Hit shader code.
   - Dynamic Monte Carlo path tracing with importance sampling for global illumination, glossy reflections, and soft shadows.

3. Vulkan Resource & Memory Management:
   - Integration with VMA (Vulkan Memory Allocator), swapchain recreation loops, and double/triple frame buffering sync primitives (`VkSemaphore`, `VkFence`).

Provide the CMakeLists.txt script, main C++ application structure, Vulkan initializers, acceleration structure build wrappers, and Ray Tracing GLSL shaders.
```

---

## 9. Web3 & Distributed Systems: Custom Layer-1 Blockchain Node

```text
Act as a Distributed Systems Cryptographer and Core Blockchain Developer. Construct a complete Layer-1 Blockchain Node implementation in Rust.

### CORE SYSTEM SPECIFICATIONS:

1. Cryptography & Transaction Engine:
   - Ed25519 keypair signing/verification, UTXO / Account-based state transition machine.
   - Merkle Patricia Trie for world state storage and fast cryptographic state proofs.

2. Consensus Engine (Proof of Stake - PoS):
   - Validator selection mechanism based on staked tokens, slot/epoch timers, and Casper-FFG style finality gadget for block confirmation.

3. P2P Networking & RPC API:
   - Libp2p network implementation for gossipsub block propagation and peer discovery.
   - JSON-RPC API server handling transaction submission and balance queries.

Provide project configuration (`Cargo.toml`), Merkle Trie module, PoS consensus engine logic, P2P network state, and JSON-RPC interface.
```

---

## 10. Compiler Construction: Production-Grade Programming Language Compiler

```text
Act as a Compiler Architect and Programming Language Theorist. Design and build a compiled programming language targeting LLVM IR or Native Machine Code.

### COMPILER PIPELINE STAGES:

1. Lexer & Parser Engine:
   - Hand-written Lexer producing rich token streams with source code location metadata.
   - Recursive Descent / Pratt Parser constructing a strongly typed Abstract Syntax Tree (AST) supporting algebraic data types, type inference, control flow, and function definitions.

2. Semantic Analysis & Type Checker:
   - Hindley-Milner type inference engine, scope symbol tables, and static borrow/lifetime validation analysis.

3. Code Generation (LLVM Intermediate Representation):
   - C++ / Rust LLVM API bindings converting AST nodes into optimized LLVM IR with SSA form, function calls, and struct memory layout allocation.

Output the formal grammar specification, AST node definitions, Pratt parser code, type checker code, and LLVM IR generation module.
```

---

## 🚀 Recommended Usage Workflow in Claude Code CLI

To execute these prompts effectively within your local terminal using **Claude Code**:

1. **Clear Terminal Context:**
   ```bash
   /clear
   ```
2. **Set Target Directory & Environment:**
   Initialize your target project repository and run `claude` inside the working directory.
3. **Execute Scaffold Execution:**
   Paste the chosen prompt from above. Allow Claude Code to create folder structures, build scripts (`CMakeLists.txt`, `Cargo.toml`, or `package.json`), and primary source files.
4. **Iterative Terminal Validation:**
   Prompt Claude Code to execute the build command automatically (e.g., `cargo test`, `cmake --build build`, `pytest`) to catch compile-time and runtime errors interactively.