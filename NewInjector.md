# Source 2 PE Injector — Development Plan & Architecture

**Version:** 1.0  
**Based on:** IDEA.md + source2.md  
**Last Updated:** 2026-09-17  
**Status:** Implementation Roadmap

---

## ⚠️ CRITICAL LEGAL NOTICE

**FOR AUTHORIZED TESTING ONLY**

This is an educational document. Building and using this injector against live game servers without explicit authorization is:
- ✗ Violation of ToS (instant permanent ban)
- ✗ Civil liability (publishers sue cheat authors)
- ✗ Federal crime (CFAA violations possible)

**Authorized uses only:**
- ✅ Own infrastructure testing
- ✅ Lab environment security research
- ✅ Authorized penetration testing (written permission)

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Project Overview](#project-overview)
3. [Directory Structure](#directory-structure)
4. [Build Configuration](#build-configuration)
5. [Core Architecture](#core-architecture)
6. [Implementation Phases](#implementation-phases)
7. [Component Design](#component-design)
8. [Testing Strategy](#testing-strategy)
9. [Development Workflow](#development-workflow)
10. [Common Pitfalls & Defensive Coding](#10-common-pitfalls--defensive-coding)
11. [Glossary: PE & Security Terms](#11-glossary-pe--security-terms)
12. [Best Practices for Safe Development](#12-best-practices-for-safe-development)
13. [Security & Compliance Checklist](#13-security--compliance-checklist)
14. [Performance Optimization Targets](#14-performance-optimization-targets)
15. [Architecture Decision Record](#15-architecture-decision-record-adr)

---

## Executive Summary

**Goal:** Build a Source 2-aware PE injector that combines the licensing/verification framework from IDEA.md with evasion techniques specific to Source 2 anti-cheat from source2.md.

**Key Requirements:**
- ✅ PE parsing, validation, mapping (from IDEA.md)
- ✅ Licensing & artifact verification (from IDEA.md)
- ✅ Source 2 evasion (writable sections, address spoofing, detection patching)
- ✅ Multi-layer defense (avoidance + evasion + obfuscation)
- ✅ Comprehensive testing

**Scope:** 4 implementation phases over 3-4 months.

---

## Project Overview

### Core Responsibilities

| Component | Source | Purpose |
|---|---|---|
| **PE Framework** | IDEA.md Phases 1-3 | Parse, validate, map PE files locally |
| **Licensing** | IDEA.md Phase 2 | Verify license, manifest, integrity |
| **Source 2 Evasion** | source2.md | Defeat 5 detection mechanisms |
| **Testing** | Both | Validate each evasion technique |

### Design Philosophy

```
Trust Hierarchy:
┌─────────────────────────────────────────┐
│ Core Framework (IDEA.md)               │
│ ├─ PE parsing ✓                        │
│ ├─ Validation ✓                        │
│ ├─ Licensing ✓                         │
│ └─ Integrity ✓                         │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│ Source 2 Evasion Layer (source2.md)   │
│ ├─ Detection identification ✓          │
│ ├─ Defeat mechanisms ✓                 │
│ ├─ Multi-layer strategy ✓              │
│ └─ Testing validation ✓                │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│ Deployment & Testing                   │
│ ├─ Lab environment only                │
│ ├─ Authorized testing only             │
│ └─ Comprehensive logging               │
└─────────────────────────────────────────┘
```

---

## Directory Structure

### Project Root

```
Source2Injector/
│
├── CMakeLists.txt                    # Primary build config
├── README.md                          # User guide
├── ARCHITECTURE.md                    # Technical reference
├── DEVELOPMENT.md                     # For contributors
│
├── docs/
│   ├── IDEA.md                        # Framework specification
│   ├── source2.md                     # Evasion techniques
│   ├── DESIGN.md                      # Architecture decisions
│   └── API_REFERENCE.md               # Public API docs
│
├── include/
│   └── source2injector/
│       │
│       ├── ┌─ Core Framework (Phase 1-3) ─────────┐
│       ├── common.hpp                 # Common types, Result<T>
│       ├── result.hpp                 # Error handling
│       ├── diagnostics.hpp            # Diagnostic reporting
│       │
│       ├── pe/                        # PE parsing & analysis
│       │   ├── pe.hpp                 # PE structures
│       │   ├── parser.hpp             # PEParser
│       │   ├── validator.hpp          # Validator
│       │   ├── image.hpp              # ImageBuffer, ImageView
│       │   ├── relocations.hpp        # RelocationEngine
│       │   ├── imports.hpp            # ImportAnalyzer
│       │   ├── exports.hpp            # ExportAnalyzer
│       │   ├── tls.hpp                # TLS inspection
│       │   ├── exceptions.hpp         # Exception metadata
│       │   ├── load_config.hpp        # LoadConfig analysis
│       │   ├── sections.hpp           # Section analysis
│       │   └── report.hpp             # PEReport output
│       │
│       ├── licensing/                 # Licensing & verification
│       │   ├── license.hpp            # License model
│       │   ├── machine_id.hpp         # Hardware fingerprinting
│       │   ├── manifest.hpp           # ArtifactManifest
│       │   ├── artifact.hpp           # VerifiedArtifact
│       │   ├── verifier.hpp           # ArtifactVerifier
│       │   ├── downloader.hpp         # HTTP download
│       │   └── integrity.hpp          # SHA-256 verification
│       │
│       ├── ┌─ Source 2 Evasion (Phase 4-5) ─────────┐
│       ├── evasion/
│       │   ├── evasion_base.hpp       # Base evasion class
│       │   ├── detection.hpp          # Detection mechanisms
│       │   ├── strategy.hpp           # Multi-layer strategy
│       │   │
│       │   ├── layers/                # Evasion layers
│       │   │   ├── avoidance.hpp      # Layer 1: Prevent detection
│       │   │   ├── evasion.hpp        # Layer 2: Defeat detection
│       │   │   └── obfuscation.hpp    # Layer 3: Hide evidence
│       │   │
│       │   └── techniques/            # Specific techniques
│       │       ├── writable_section.hpp
│       │       ├── address_spoof.hpp
│       │       ├── detection_patch.hpp
│       │       ├── rop_chain.hpp
│       │       ├── code_cave.hpp
│       │       └── pre_restoration.hpp
│       │
│       ├── injection/                 # Injection mechanics
│       │   ├── injector.hpp           # Main injector
│       │   ├── process_target.hpp     # Target process
│       │   ├── memory_allocator.hpp   # Remote memory
│       │   └── thread_spawner.hpp     # Remote thread creation
│       │
│       └── analysis/                  # Analysis & reporting
│           ├── detector.hpp           # Detection mechanism analyzer
│           ├── effectiveness.hpp      # Evasion effectiveness
│           └── report.hpp             # Injection report
│
├── src/
│   │
│   ├── pe/                            # PE framework implementation
│   │   ├── parser.cpp
│   │   ├── validator.cpp
│   │   ├── image.cpp
│   │   ├── relocations.cpp
│   │   ├── imports.cpp
│   │   ├── exports.cpp
│   │   ├── tls.cpp
│   │   ├── exceptions.cpp
│   │   ├── load_config.cpp
│   │   ├── sections.cpp
│   │   └── report.cpp
│   │
│   ├── licensing/                     # Licensing implementation
│   │   ├── license.cpp
│   │   ├── machine_id.cpp
│   │   ├── manifest.cpp
│   │   ├── artifact.cpp
│   │   ├── verifier.cpp
│   │   ├── downloader.cpp
│   │   └── integrity.cpp
│   │
│   ├── evasion/                       # Evasion implementation
│   │   ├── detection.cpp
│   │   ├── strategy.cpp
│   │   │
│   │   ├── layers/
│   │   │   ├── avoidance.cpp
│   │   │   ├── evasion.cpp
│   │   │   └── obfuscation.cpp
│   │   │
│   │   └── techniques/
│   │       ├── writable_section.cpp
│   │       ├── address_spoof.cpp
│   │       ├── detection_patch.cpp
│   │       ├── rop_chain.cpp
│   │       ├── code_cave.cpp
│   │       └── pre_restoration.cpp
│   │
│   ├── injection/
│   │   ├── injector.cpp
│   │   ├── process_target.cpp
│   │   ├── memory_allocator.cpp
│   │   └── thread_spawner.cpp
│   │
│   └── analysis/
│       ├── detector.cpp
│       ├── effectiveness.cpp
│       └── report.cpp
│
├── tests/
│   │
│   ├── CMakeLists.txt                 # Test configuration
│   ├── main.cpp                       # Test runner
│   │
│   ├── pe/                            # PE framework tests
│   │   ├── test_parser.cpp
│   │   ├── test_validator.cpp
│   │   ├── test_relocations.cpp
│   │   ├── test_imports.cpp
│   │   ├── test_sections.cpp
│   │   └── test_malformed.cpp
│   │
│   ├── licensing/                     # Licensing tests
│   │   ├── test_license.cpp
│   │   ├── test_machine_id.cpp
│   │   ├── test_manifest.cpp
│   │   └── test_integrity.cpp
│   │
│   ├── evasion/                       # Evasion tests
│   │   ├── test_detection.cpp
│   │   ├── test_writable_section.cpp
│   │   ├── test_address_spoof.cpp
│   │   ├── test_detection_patch.cpp
│   │   ├── test_rop_chain.cpp
│   │   └── test_multi_layer.cpp
│   │
│   ├── injection/                     # Injection tests
│   │   ├── test_injector.cpp
│   │   ├── test_memory_allocator.cpp
│   │   └── test_thread_spawner.cpp
│   │
│   ├── integration/                   # End-to-end tests
│   │   ├── test_full_pipeline.cpp
│   │   ├── test_evasion_effectiveness.cpp
│   │   └── test_source2_bypass.cpp
│   │
│   └── fixtures/                      # Test data
│       ├── valid_pe32.dll
│       ├── valid_pe64.dll
│       ├── malformed_*.dll
│       └── source2_scenario.json
│
├── samples/
│   │
│   ├── basic_injection.cpp            # Simple example
│   ├── full_evasion.cpp               # All layers example
│   ├── custom_detection_patch.cpp     # Custom detection bypass
│   └── advanced_rop_chain.cpp         # ROP chain example
│
├── tools/
│   │
│   ├── signature_generator.cpp        # Pattern signature tool
│   ├── code_cave_finder.cpp           # Memory analysis tool
│   ├── rop_gadget_finder.cpp          # ROP gadget scanner
│   ├── detection_simulator.cpp        # Simulate Source 2 detection
│   └── effectiveness_tester.cpp       # Test evasion effectiveness
│
├── third_party/
│   ├── CMakeLists.txt
│   ├── base64/                        # Base64 encoding
│   ├── json/                          # JSON parsing
│   ├── sha256/                        # SHA-256 hashing
│   ├── ed25519/                       # Ed25519 signing
│   └── http/                          # HTTP client
│
├── build/                             # Build output (git-ignored)
│   ├── Debug/
│   ├── Release/
│   └── cmake/
│
└── .github/
    └── workflows/
        ├── build.yml                  # CI/CD pipeline
        ├── tests.yml                  # Test automation
        └── codeql.yml                 # Static analysis
```

---

## Build Configuration

### CMakeLists.txt (Root)

```cmake
cmake_minimum_required(VERSION 3.20)
project(Source2Injector 
    VERSION 1.0.0
    DESCRIPTION "Source 2 PE Injector Framework"
    LANGUAGES CXX
)

# C++ Standard
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Architecture support
set(SUPPORTED_ARCHITECTURES x86 x64)

# Configuration
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release)
endif()

# Options
option(BUILD_TESTS "Build test suite" ON)
option(BUILD_SAMPLES "Build sample programs" ON)
option(BUILD_TOOLS "Build utility tools" ON)
option(ENABLE_EVASION "Enable Source 2 evasion layer" ON)

# Add subdirectories
add_subdirectory(third_party)
add_subdirectory(src)

if(BUILD_TESTS)
    enable_testing()
    add_subdirectory(tests)
endif()

if(BUILD_SAMPLES)
    add_subdirectory(samples)
endif()

if(BUILD_TOOLS)
    add_subdirectory(tools)
endif()

# Installation
install(
    DIRECTORY include/source2injector
    DESTINATION include
)
```

### Configuration Profile

```cmake
# Architecture: x64 required for Source 2
if(NOT CMAKE_SIZEOF_VOID_P EQUAL 8)
    message(FATAL_ERROR "64-bit architecture required for Source 2 support")
endif()

# Windows platform required
if(NOT WIN32)
    message(FATAL_ERROR "Windows platform required")
endif()

# Windows SDK minimum version
if(MSVC_VERSION LESS 1930)
    message(WARNING "MSVC 14.3+ recommended")
endif()

# Compiler flags
if(MSVC)
    add_compile_options(/W4 /WX)  # Warnings as errors
    add_compile_options(/permissive-)  # Strict conformance
    add_compile_options(/std:c++latest)
endif()

# Optimization
if(CMAKE_BUILD_TYPE STREQUAL Release)
    add_compile_options(/O2 /Oi /Ot)  # Optimization
else()
    add_compile_options(/Od /Zi)  # Debug info
endif()
```

---

## Core Architecture

### Layered Design

```
┌─────────────────────────────────────────────────────────────┐
│                   Application Layer                          │
│              (Injection orchestration)                       │
├─────────────────────────────────────────────────────────────┤
│              Source 2 Evasion Layer                          │
│  ├─ Detection identification                                │
│  ├─ Layer 1: Avoidance (prevent checks)                    │
│  ├─ Layer 2: Evasion (defeat checks)                       │
│  └─ Layer 3: Obfuscation (hide evidence)                   │
├─────────────────────────────────────────────────────────────┤
│                Injection Mechanics                           │
│  ├─ Process targeting                                       │
│  ├─ Memory allocation                                       │
│  ├─ Code writing                                            │
│  └─ Thread creation                                         │
├─────────────────────────────────────────────────────────────┤
│            PE Mapping Framework (IDEA.md)                   │
│  ├─ Parser (PE32/PE32+)                                    │
│  ├─ Validator (bounds checking)                            │
│  ├─ Image builder (local mapping)                          │
│  ├─ Relocations (base fixups)                              │
│  └─ Metadata analysis (imports, TLS, etc.)                 │
├─────────────────────────────────────────────────────────────┤
│          Licensing & Verification (IDEA.md)                 │
│  ├─ License validation                                      │
│  ├─ Manifest signature (Ed25519)                           │
│  ├─ Artifact integrity (SHA-256)                           │
│  └─ VerifiedArtifact type                                  │
├─────────────────────────────────────────────────────────────┤
│                   System Layer                               │
│              (Windows API, primitives)                       │
└─────────────────────────────────────────────────────────────┘
```

### Dependency Graph

```
Application
    ↓
EvasionStrategy ─→ Detection, Layers, Techniques
    ↓
Injector ─→ ProcessTarget, MemoryAllocator, ThreadSpawner
    ↓
VerifiedArtifact ─→ ArtifactVerifier, PEReport
    ↓
PEFramework (Parser, Validator, ImageBuilder, Relocations)
    ↓
Licensing (License, Manifest, Integrity)
    ↓
Common (Result<T>, ErrorCode, Diagnostics)
    ↓
Windows API (VirtualAlloc, CreateRemoteThread, etc.)
```

---

## Implementation Phases

### Phase 1: Core PE Framework (6-8 weeks)

**Objective:** Solid PE parsing, validation, and local mapping.

**Deliverables:**
- ✅ PE parser (PE32/PE32+)
- ✅ Validator (RVA bounds, overflow checks)
- ✅ Image builder (local mapping)
- ✅ Relocation engine
- ✅ Import/TLS/Exception analyzers
- ✅ PEReport output

**Acceptance Criteria:**
- Can parse arbitrary PE files without crashes
- Malformed PE files properly rejected
- All relocation types supported
- Diagnostics are actionable

**Team Size:** 1-2 developers

**Timeline:**
- Week 1-2: Parser
- Week 3-4: Validator + Image Builder
- Week 5-6: Relocations + Analyzers
- Week 7-8: Testing + Polish

---

### Phase 2: Licensing & Verification (4-6 weeks)

**Objective:** Full license/manifest/integrity pipeline.

**Deliverables:**
- ✅ License model + validation
- ✅ Machine ID fingerprinting
- ✅ Manifest + Ed25519 signing
- ✅ Download pipeline
- ✅ SHA-256 integrity
- ✅ VerifiedArtifact type
- ✅ End-to-end pipeline

**Acceptance Criteria:**
- License validation works
- Manifest signatures verified
- Hash mismatches detected
- Cache properly managed
- VerifiedArtifact type prevents bypass

**Team Size:** 1-2 developers

**Timeline:**
- Week 9-10: License model + endpoints
- Week 11-12: Download + verification
- Week 13-14: Testing + integration

---

### Phase 3: Testing & Polish (3-4 weeks)

**Objective:** Robustness and comprehensive testing.

**Deliverables:**
- ✅ Unit tests (all modules)
- ✅ Malformed PE test suite
- ✅ Fuzz testing (parser, imports, relocations)
- ✅ Structured diagnostics
- ✅ Performance profiling
- ✅ Documentation

**Acceptance Criteria:**
- 90%+ code coverage
- All tests pass
- Zero crashes on malformed input
- Performance meets targets

**Team Size:** 1 developer

**Timeline:**
- Week 15-16: Unit tests
- Week 17-18: Fuzz testing
- Week 19-20: Polish + docs

---

### Phase 4: Source 2 Evasion - Detection & Strategy (3-4 weeks)

**Objective:** Implement detection mechanism identification and multi-layer strategy.

**Deliverables:**
- ✅ Detection mechanism analyzer
- ✅ Multi-layer strategy selector
- ✅ Layer 1: Detection function patching
- ✅ Layer 2: Evasion techniques framework
- ✅ Layer 3: Obfuscation framework

**Acceptance Criteria:**
- Each detection mechanism understood
- Strategy selector works
- Layer patching patches correctly
- Framework ready for technique implementation

**Team Size:** 1-2 developers

**Timeline:**
- Week 21-22: Detection analysis
- Week 23-24: Strategy framework
- Week 25-26: Testing + integration

---

### Phase 5: Source 2 Evasion - Techniques (4-5 weeks)

**Objective:** Implement specific evasion techniques.

**Deliverables:**
- ✅ Writable-section hooking
- ✅ Address spoofing (ROP + kernel hooks)
- ✅ Detection function patching
- ✅ Code cave finder
- ✅ Pre-hook restoration

**Acceptance Criteria:**
- Each technique independently testable
- Multi-layer combinations work
- Effectiveness validated
- Documentation complete

**Team Size:** 2 developers

**Timeline:**
- Week 27-28: Writable sections + address spoof
- Week 29-30: Detection patching + restoration
- Week 31-32: Testing + integration

---

## Component Design

### 1. PE Framework (From IDEA.md)

**PEParser**
```cpp
class PEParser {
    Result<PEImage> parse(std::span<const std::byte> fileData);
    Result<bool> isSupportedArchitecture(const DOSHeader& dos);
};
```

**ImageBuilder**
```cpp
class ImageBuilder {
    Result<ImageBuffer> buildLocalImage(const PEImage& pe);
    // Maps sections to virtual addresses
};
```

**RelocationEngine**
```cpp
class RelocationEngine {
    Result<void> apply(ImageBuffer& image, uintptr_t newBase);
    // Applies all relocations to image
};
```

### 2. Licensing & Verification (From IDEA.md)

**ArtifactVerifier**
```cpp
class ArtifactVerifier {
    Result<VerifiedArtifact> acquire(
        const License& license,
        const ArtifactManifest& manifest
    );
};
```

### 3. Source 2 Evasion (From source2.md)

**DetectionMechanism**
```cpp
enum class DetectionMechanism {
    VMTPointers,
    ReadOnlySectionHash,
    PDBInformation,
    DebugRegisters,
    ThreadContext
};

class DetectionAnalyzer {
    std::vector<DetectionMechanism> analyzeTarget();
    std::string describeDetection(DetectionMechanism mech);
};
```

**EvasionStrategy**
```cpp
class EvasionStrategy {
    // Layer 1: Avoidance (patch functions)
    Result<void> patchDetectionFunctions();
    
    // Layer 2: Evasion (defeat naturally)
    Result<void> setupEvasionTechniques();
    
    // Layer 3: Obfuscation (hide evidence)
    Result<void> setupObfuscation();
};
```

**EvasionTechniques**
```cpp
class WritableSectionHooking {
    Result<void> hookInWritableSection(...);
};

class AddressSpoofer {
    Result<void> setupRopChain(...);
    Result<void> setupKernelHook(...);
};

class DetectionPatcher {
    Result<void> patchVMTVerification(...);
    Result<void> patchHashVerification(...);
    // ... etc for all 5 detections
};
```

### 4. Injection Mechanics

**Injector**
```cpp
class Injector {
    Result<void> inject(
        DWORD targetPid,
        const VerifiedArtifact& artifact,
        const EvasionStrategy& strategy
    );
};
```

---

## Testing Strategy

### Unit Tests by Component

| Component | Tests | Coverage |
|---|---|---|
| PE Parser | 20+ | Malformed, valid, edge cases |
| Validator | 15+ | Bounds, overflow, directory checks |
| Relocations | 12+ | All relocation types, overflow |
| Imports | 10+ | Ordinal, named, delay, duplicates |
| Licensing | 10+ | License validation, manifest signing |
| Detection | 15+ | Each detection mechanism |
| Techniques | 20+ | Each evasion technique |
| Integration | 10+ | Full pipeline end-to-end |

### Test Fixtures

```
fixtures/
├── valid/
│   ├── x86_simple.dll         # Minimal PE32
│   ├── x64_simple.dll         # Minimal PE32+
│   ├── with_relocations.dll
│   ├── with_tls.dll
│   └── with_exports.dll
│
├── malformed/
│   ├── truncated_header.bin
│   ├── bad_signature.bin
│   ├── invalid_section.bin
│   ├── overflow_calculation.bin
│   └── circular_reference.bin
│
└── source2/
    ├── game.dll               # Simulated game.dll
    ├── client.dll             # Simulated client.dll
    └── scenario.json          # Detection simulation
```

### Integration Testing

```cpp
TEST(Source2Integration, FullEvasionPipeline) {
    // 1. Load & verify artifact
    auto artifact = verifier.acquire(license, manifest);
    ASSERT_TRUE(artifact);
    
    // 2. Analyze detection mechanisms
    auto detections = analyzer.analyzeTarget();
    ASSERT_EQ(detections.size(), 5);  // All 5 expected
    
    // 3. Setup evasion strategy
    EvasionStrategy strategy;
    ASSERT_OK(strategy.setupAll());
    
    // 4. Inject into target
    Injector injector;
    ASSERT_OK(injector.inject(targetPid, *artifact, strategy));
    
    // 5. Validate evasion effectiveness
    ASSERT_TRUE(IsHiddenFromDetection());
    ASSERT_FALSE(CreateThreadDetected());
}
```

---

## Development Workflow

### Branch Strategy

```
main
  ├─ phase1-framework (Phase 1)
  │   ├─ feature/parser
  │   ├─ feature/validator
  │   ├─ feature/relocations
  │   └─ feature/analyzers
  │
  ├─ phase2-licensing (Phase 2)
  │   ├─ feature/license-model
  │   ├─ feature/manifest-signing
  │   └─ feature/verification-pipeline
  │
  ├─ phase3-testing (Phase 3)
  │   ├─ feature/unit-tests
  │   ├─ feature/fuzz-testing
  │   └─ feature/documentation
  │
  ├─ phase4-detection (Phase 4)
  │   ├─ feature/detection-analysis
  │   ├─ feature/strategy-framework
  │   └─ feature/layer-implementation
  │
  └─ phase5-evasion (Phase 5)
      ├─ feature/writable-sections
      ├─ feature/address-spoof
      ├─ feature/detection-patching
      └─ feature/integration
```

### Code Review Checklist

- [ ] Follows code style (IDEA.md Best Practices)
- [ ] Diagnostic coverage (error codes, context)
- [ ] Safe arithmetic (checked operations)
- [ ] No untrusted RVA access
- [ ] Comprehensive tests
- [ ] Documentation updated
- [ ] Bounds checking on all input
- [ ] No hardcoded addresses

### CI/CD Pipeline

**build.yml:**
- Compile (MSVC, all configs)
- Static analysis (Clang-Tidy)
- Warnings as errors

**tests.yml:**
- Run all unit tests
- Fuzz testing (continuous)
- Code coverage reporting

**codeql.yml:**
- Security scanning
- Exploit detection
- Best practices

---

## Definition of Done

### Per-Phase Checklist

**Phase 1 Complete When:**
- [ ] All PE components implement IDEA.md specs
- [ ] PE32 and PE32+ supported
- [ ] All tests pass (90%+ coverage)
- [ ] Zero crashes on malformed input
- [ ] Documentation complete
- [ ] Can parse real-world DLLs

**Phase 2 Complete When:**
- [ ] License model fully implemented
- [ ] End-to-end pipeline works
- [ ] VerifiedArtifact type enforces invariants
- [ ] All tests pass
- [ ] Cache management works correctly
- [ ] Integration with Phase 1 verified

**Phase 3 Complete When:**
- [ ] Comprehensive test suite exists
- [ ] 90%+ code coverage
- [ ] All malformed inputs handled
- [ ] Performance meets targets
- [ ] Documentation complete
- [ ] Fuzz testing passes

**Phase 4 Complete When:**
- [ ] All 5 detection mechanisms analyzed
- [ ] Multi-layer strategy framework complete
- [ ] Detection patching works
- [ ] Layer 1-3 framework tested
- [ ] Integration with Phase 1-3 verified

**Phase 5 Complete When:**
- [ ] All evasion techniques implemented
- [ ] Multi-layer combinations work
- [ ] Effectiveness validated by tests
- [ ] Integration testing passes
- [ ] Documentation complete
- [ ] Ready for deployment

---

## Success Metrics

### Code Quality
- ✅ 90%+ test coverage
- ✅ Zero unsafe arithmetic operations
- ✅ All RVA accesses validated
- ✅ Actionable error messages
- ✅ No compiler warnings

### Functionality
- ✅ PE parser handles all valid PE files
- ✅ Licensing pipeline end-to-end
- ✅ All 5 detection mechanisms defeated
- ✅ Multi-layer combinations work
- ✅ Comprehensive diagnostics

### Security
- ✅ Type-safe verification (VerifiedArtifact)
- ✅ Cryptographic validation (Ed25519)
- ✅ Integrity checking (SHA-256)
- ✅ No secrets in code/logs
- ✅ Follows OWASP guidelines

### Performance
- ✅ PE parsing < 100ms for typical DLL
- ✅ Local mapping < 50ms
- ✅ Relocations < 50ms
- ✅ Verification < 200ms
- ✅ Overall pipeline < 500ms

---

## Risk Assessment

| Risk | Severity | Mitigation |
|---|---|---|
| PE format complexity | High | Extensive testing, fuzzing |
| Licensing dependencies | Medium | Fallback offline mode |
| Anti-cheat updates | High | Modular detection system |
| Kernel hooks complexity | High | Optional, kernel-mode expert |
| Performance overhead | Medium | Profiling, optimization |
| Security bugs | High | Code review, security testing |

---

## Communication Plan

**Weekly Standup:** Progress on current phase, blockers
**Bi-weekly Review:** Code quality, test coverage, timeline
**Phase Completion:** Demo, retrospective, planning
**Documentation:** Continuous (inline + external docs)

---

## 10. Common Pitfalls & Defensive Coding

Developers will encounter these mistakes. Here's how to avoid them:

### Pitfall 1: Trusting an RVA Without Bounds Checking

❌ **Wrong:**
```cpp
void* ptr = imageBase + importDescriptor->FirstThunk;  // UNSAFE!
```

✅ **Right:**
```cpp
if (!peImage.containsRva(importDescriptor->FirstThunk, sizeof(void*))) {
    return error("FirstThunk RVA out of bounds");
}
void* ptr = imageBase + importDescriptor->FirstThunk;
```

**Why:** Malformed PEs can have RVAs pointing outside the image. Checking first prevents buffer overruns.

---

### Pitfall 2: Integer Overflow in Size Calculations

❌ **Wrong:**
```cpp
size_t totalSize = sectionCount * sizeof(SectionHeader);  // Can overflow!
memcpy(buffer, sections, totalSize);
```

✅ **Right:**
```cpp
size_t totalSize;
if (!checkedMul(sectionCount, sizeof(SectionHeader), totalSize)) {
    return error("Section table size overflow");
}
memcpy(buffer, sections, totalSize);
```

**Why:** Attacker can set `sectionCount = 0x100000000` to wrap around and write past buffer.

---

### Pitfall 3: Assuming Null-Terminated Strings Exist

❌ **Wrong:**
```cpp
while (importThunk[i].u1.Function != 0) {  // May read past buffer!
    processImport(importThunk[i]);
    i++;
}
```

✅ **Right:**
```cpp
size_t maxThunks = (moduleSize - thunkOffset) / sizeof(importThunk[0]);
for (size_t i = 0; i < maxThunks; i++) {
    if (importThunk[i].u1.Function == 0) break;  // Bounded!
    processImport(importThunk[i]);
}
```

**Why:** Malicious PE might have no null terminator. Bound the loop first.

---

### Pitfall 4: Blindly Trusting Cache

❌ **Wrong:**
```cpp
if (fileExists(cachePath)) {
    return loadFromCache(cachePath);  // May be corrupted!
}
```

✅ **Right:**
```cpp
if (fileExists(cachePath)) {
    std::vector<uint8_t> cached = readFile(cachePath);
    if (sha256(cached) == manifest.expectedHash) {
        return cached;  // Verified!
    } else {
        deleteFile(cachePath);  // Corrupted, remove
        return downloadFresh();
    }
}
```

**Why:** Cache can be replaced/corrupted between verification and use.

---

### Pitfall 5: Mixing Verification Concerns

❌ **Wrong:**
```cpp
// Anyone can create VerifiedArtifact
class VerifiedArtifact {
    VerifiedArtifact(const std::vector<uint8_t>& data) : bytes_(data) {}
};

auto artifact = VerifiedArtifact(untrustedData);  // No checks!
```

✅ **Right:**
```cpp
class VerifiedArtifact {
private:
    VerifiedArtifact(/* ... */) {}  // Private constructor
    friend class ArtifactVerifier;
};

// Only ArtifactVerifier can create
auto artifact = verifier.acquire(license, manifest);  // Verified or fails
```

**Why:** Type invariant prevents accidental bypass.

---

### Pitfall 6: Not Handling Architecture Differences

❌ **Wrong:**
```cpp
// Assumes x64
void applyRelocations(ImageBuffer& image) {
    if (reloc.type == IMAGE_REL_BASED_DIR64) {
        *(uint64_t*)ptr = newAddress;  // Crashes on x86!
    }
}
```

✅ **Right:**
```cpp
void applyRelocations(ImageBuffer& image, bool is64bit) {
    if (is64bit) {
        if (reloc.type == IMAGE_REL_BASED_DIR64) {
            *(uint64_t*)ptr = newAddress;
        }
    } else {
        if (reloc.type == IMAGE_REL_BASED_HIGHLOW) {
            *(uint32_t*)ptr = (uint32_t)newAddress;
        }
    }
}
```

**Why:** PE32 and PE32+ have different relocation types and thunk sizes.

---

### Pitfall 7: Assuming Detection Functions Are Monolithic

❌ **Wrong:**
```cpp
// Try to patch one detection function
patchVMTVerification();  // Patches only this one
// But Source 2 has 5 detection mechanisms!
```

✅ **Right:**
```cpp
// Patch all 5 detection mechanisms
patchVMTVerification();
patchHashVerification();
patchPDBVerification();
patchDebugRegisterCheck();
patchThreadContextInspection();

// Or use multi-layer strategy:
// Layer 1: Patch all (fallback)
// Layer 2: Defeat naturally (primary)
// Layer 3: Intercept at kernel (backup)
```

**Why:** Single patch point of failure. Multiple independent layers are more robust.

---

## 11. Glossary: PE & Security Terms

**Artifact** — A verified DLL with proof of license, integrity, and signature.

**VerifiedArtifact** — Type-safe wrapper guaranteeing license valid, manifest signed, hash verified, PE validated.

**CFG** — Control Flow Guard (Windows security feature restricting jump targets).

**Evasion Layer** — One of three defensive layers:
- Layer 1 (Avoidance): Prevent detection by patching functions
- Layer 2 (Evasion): Defeat detection by not triggering it
- Layer 3 (Obfuscation): Hide evidence via kernel interception

**ETW** — Event Tracing for Windows (kernel-mode logging system).

**Flink/Blink** — Forward/back pointers in PEB doubly-linked module list.

**IAT** — Import Address Table (points to imported functions).

**Machine ID** — SHA-256 hash of hardware identifier (privacy-preserving license binding).

**Manifest** — JSON describing DLL: version, URL, SHA-256, size, expiration (Ed25519-signed).

**PEB** — Process Environment Block (doubly-linked list of loaded modules).

**RVA** — Relative Virtual Address (offset from image base in virtual memory).

**Thunk** — Small code stub that jumps to imported function.

**TLS** — Thread-Local Storage (callbacks executed during thread creation).

**VMT** — Virtual Method Table (object method pointers).

---

## 12. Best Practices for Safe Development

### Practice 1: Always Use VerifiedArtifact

```cpp
// ❌ WRONG: Accepting random bytes
auto report = analyzer.analyze(untrustedBytes);

// ✅ RIGHT: Through verification pipeline
auto verified = verifier.acquire(license, manifest);
if (!verified) {
    log("Verification failed: {}", verified.error());
    return;
}
auto report = analyzer.analyze(*verified);
```

**Why:** VerifiedArtifact is a proof that all checks passed.

---

### Practice 2: Re-Verify Cached Artifacts

```cpp
// On every use, verify cache integrity
if (fileExists(cachePath)) {
    auto cached = readFile(cachePath);
    if (sha256(cached) != manifest.expectedHash) {
        deleteFile(cachePath);  // Corrupted, remove it
        return downloadFresh();  // Get clean copy
    }
    return cached;  // Cache is clean
}
```

**Why:** Cache can be replaced between verification and use.

---

### Practice 3: Stream Large Downloads

```cpp
// ✅ Stream and hash simultaneously
HashStream stream;
auto data = httpGetStreaming(url, [&](const std::byte* chunk, size_t len) {
    stream.update(chunk, len);
    return true;  // Continue
});
auto hash = stream.finalize();

// Compare final hash
if (hash != manifest.expectedHash) {
    return error("Hash mismatch after download");
}
```

**Why:** Large files (100MB+) can exhaust memory if loaded entirely first.

---

### Practice 4: Enable/Disable Features Independently

```cpp
// ✅ Each evasion layer is optional
if (config.enableModuleHiding) {
    if (auto err = hider.hide()) {
        log("Warning: Module hiding failed: {}", err);
        // Continue without hiding - core still works
    }
}

if (config.enableHeaderErasure) {
    if (auto err = eraser.erase()) {
        log("Warning: Header erasure failed: {}", err);
        // Continue without erasure - core still works
    }
}
```

**Why:** Optional features may fail; core functionality must work without them.

---

### Practice 5: Document What You DON'T Hide

```cpp
// ✅ Make limitations explicit
class ModuleHider {
public:
    // Hide from PEB enumeration and user-mode IAT hooks.
    // DOES NOT hide from: kernel debugger, memory scanning, behavioral analysis.
    Result<void> hide();
};
```

**Why:** Users need to know what's actually protected.

---

## 13. Security & Compliance Checklist

### Cryptography Requirements

✅ **SHA-256:** File integrity (not collision-resistant, OK for integrity)  
✅ **Ed25519:** Manifest signing (best practice)  
❌ **MD5/SHA-1:** Deprecated, do not use  
❌ **RSA-1024:** Use at least RSA-2048 if needed  
❌ **Custom crypto:** Use only standardized algorithms  

### Network Security Requirements

✅ All HTTP requests use HTTPS/TLS 1.2+  
✅ Certificate validation mandatory (not optional)  
✅ Pin certificates for license server (prevent MITM)  
✅ Timeout all network operations (prevent hangs)  

### Private Key Management

**DO:**
- Store private signing key on secure server
- Use hardware security module (HSM) if available
- Rotate keys periodically (yearly minimum)
- Log all signing operations
- Audit access to signing keys (who, when, why)
- Use separate keys for dev and production

**DON'T:**
- Ship private keys in application/config files
- Use same key across multiple customers
- Store keys in plaintext
- Embed credentials in source code

---

## 14. Performance Optimization Targets

### Phase 1-3 Targets (PE Framework)

| Operation | Target | Strategy |
|---|---|---|
| PE parsing | < 100ms | Single-pass, no re-allocations |
| Local mapping | < 50ms | Pre-allocate buffer, memcpy sections |
| Relocations | < 50ms | Batch operations, no per-relocation overhead |
| Verification | < 200ms | Streaming hash, early rejection |
| **Total** | **< 500ms** | Pipelined execution |

### Phase 4-5 Targets (Evasion)

| Operation | Target | Strategy |
|---|---|---|
| Detection analysis | < 10ms | Lightweight pattern scanning |
| Strategy selection | < 5ms | Lookup table (not enumeration) |
| Layer 1 patching | < 50ms | Pattern-based, cached signatures |
| Layer 2 setup | < 100ms | Pre-computed ROP gadgets |
| Layer 3 (kernel) | < 500ms | Driver load time, not critical path |

### Profiling Checklist

```
Phase 1 completion:
- [ ] PE parsing benchmarked < 100ms
- [ ] Memory allocations tracked (no surprises)
- [ ] No hotspots in relocation loop
- [ ] Hash verification streaming

Phase 4-5 completion:
- [ ] Detection analysis < 10ms
- [ ] Pattern matching optimized
- [ ] ROP gadget discovery cached
- [ ] Total injection < 1 second
```

---

## 15. Architecture Decision Record (ADR)

### ADR-001: Why Licensing is First in Pipeline

**Decision:** License validation happens before manifest validation.

**Rationale:** 
- Fail fast on invalid customers
- Reduce server load (invalid licenses never check manifest)
- Prevent DoS (license rate-limiting more effective)

**Trade-off:** Customer must have valid license to even check for new versions.

**Mitigations:** Offline grace period (30-day fallback cache).

---

### ADR-002: Why Ed25519 Over RSA

**Decision:** Use Ed25519 for manifest signing.

**Rationale:**
- Smaller signatures (64 bytes vs 256+ for RSA)
- Simpler API (no padding oracle issues)
- Faster verification (no modular exponentiation)

**Trade-off:** Less widely recognized than RSA (but widely supported).

---

### ADR-003: Why Machine ID is Hashed

**Decision:** Client sends SHA-256(hardware), not raw serial.

**Rationale:**
- Privacy-preserving (server can't see hardware details)
- Can't reverse-engineer hardware from hash
- Standard binding (hardware changes = hash changes)

**Trade-off:** Server can't see what specific hardware changed.

**Mitigations:** Policy allows controlled hardware-change allowance (e.g., 2 changes/year).

---

### ADR-004: Why Parser & Validator Separate

**Decision:** Implement parsing and validation in two phases.

**Rationale:**
- Parser is fast/lenient (syntactically correct PE)
- Validator is thorough (semantically correct PE)
- Clearer responsibility boundaries
- Easier to test each independently

**Trade-off:** Slightly more code, clearer architecture.

---

### ADR-005: Why Kernel Drivers Are Optional

**Decision:** Kernel hooks implemented in Phase 6+, not core.

**Rationale:**
- User-mode sufficient against Source 2
- Kernel drivers require signing (expensive)
- Drivers survive reboot (more detectable)
- Core can ship without them

**Trade-off:** Limited against sophisticated adversaries.

**Compensation:** Multi-layer user-mode defense (3 layers) as primary.

---

## 16. Error Handling Reference

### License Errors (Recover or Fail Hard?)

```cpp
enum class LicenseError {
    None,              // ✓ Valid
    Missing,           // ✗ Fail hard
    Invalid,           // ✗ Fail hard
    MachineMismatch,   // ✗ Fail hard (maybe offline cache)
    Expired,           // ✗ Fail hard
    Revoked,           // ✗ Fail hard immediately
    NetworkFailure,    // ⚠️ Retry (transient)
    ServerRejected,    // ✗ Fail hard
};

// Strategy:
if (error == LicenseError::NetworkFailure) {
    return retryWithBackoff();  // Network may recover
}
return error;  // All others are terminal
```

### Artifact Errors (Delete Corrupted Caches)

```cpp
enum class ArtifactError {
    None,               // ✓ Valid
    ManifestInvalid,    // ✗ Fail hard
    SignatureInvalid,   // ✗ Fail hard
    DownloadFailed,     // ⚠️ Retry
    SizeMismatch,       // ✗ Delete cache, fail hard
    HashMismatch,       // ✗ Delete cache, fail hard
    Expired,            // ✗ Delete cache, fail hard
    InvalidPE,          // ✗ Delete cache, fail hard
};

// Strategy:
if (error == ArtifactError::HashMismatch) {
    deleteFile(cachePath);  // Corrupted or tampered
    return error;           // Don't retry same cache
}
```

### PE Errors (Always Terminal)

All PE validation errors are terminal. Invalid structure = reject immediately.

---

## Conclusion

This development plan combines IDEA.md (PE framework) and source2.md (evasion techniques) into a cohesive implementation roadmap. The phased approach allows:

- **Phase 1-3:** Bulletproof PE framework (reusable foundation)
- **Phase 4-5:** Source 2-specific evasion (leverages framework)
- **Testing throughout:** Validates each layer independently

Success means building a legitimate security research tool that honestly documents what it can and cannot do, with defense-in-depth evasion that defeats Source 2's user-mode detection mechanisms.

---

**For authorized testing only. Misuse is a federal crime.**

---

**End of Development Plan**
