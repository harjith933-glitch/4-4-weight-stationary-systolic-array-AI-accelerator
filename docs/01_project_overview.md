# Project Overview

## 1. Project Title

# 4×4 Weight-Stationary Systolic Array AI Accelerator

A SystemVerilog RTL implementation of a 4×4 weight-stationary systolic array architecture for accelerating matrix multiplication workloads used in Artificial Intelligence (AI) and Machine Learning (ML) applications.

---

## 2. Introduction

Artificial Intelligence and Machine Learning workloads rely heavily on matrix and vector operations. Among these operations, matrix multiplication is one of the most computationally intensive tasks in many neural-network and signal-processing applications.

A conventional processor performs multiplication and accumulation operations sequentially or with a limited number of parallel execution units. Dedicated hardware accelerators can improve performance by exploiting parallelism and data reuse.

A systolic array is a hardware architecture specifically suited for this type of computation. It consists of multiple Processing Elements (PEs) arranged in a regular structure. Data moves between neighboring Processing Elements in a synchronized manner while computation takes place simultaneously.

This project focuses on designing a 4×4 weight-stationary systolic array accelerator using SystemVerilog RTL.

The fundamental computational unit of the architecture is the Processing Element, which performs a multiply-accumulate (MAC) operation while forwarding activation data to the next Processing Element.

---

## 3. Project Objective

The primary objective of this project is to design and verify a hardware accelerator capable of performing matrix multiplication using a 4×4 systolic array architecture.

The target architecture contains:

    4 × 4 = 16 Processing Elements

The design uses a weight-stationary dataflow in which weights are loaded into the Processing Elements and remain locally available while activation data propagates through the array.

The project follows a bottom-up hardware design methodology:

    Architecture Definition
            ↓
    Processing Element Design
            ↓
    Processing Element Verification
            ↓
    4×4 Array Integration
            ↓
    Array-Level Verification
            ↓
    Top-Level Integration
            ↓
    Synthesis and Analysis

The Processing Element stage has been implemented and verified. The remaining stages will be developed incrementally.

---

## 4. Target Computation

The primary computation targeted by the accelerator is matrix multiplication.

For two matrices A and B:

    C = A × B

For the target 4×4 configuration:

    A(4×4) × B(4×4) = C(4×4)

Each output element is calculated by multiplying corresponding elements and accumulating the products.

For a 4×4 matrix:

    C[i][j] =
        A[i][0] × B[0][j]
      + A[i][1] × B[1][j]
      + A[i][2] × B[2][j]
      + A[i][3] × B[3][j]

The general matrix multiplication operation is:

    C[i][j] = Σ(A[i][k] × B[k][j])

where:

- A is the input or activation matrix.
- B is the weight matrix.
- C is the output matrix.
- i represents the output row.
- j represents the output column.
- k represents the accumulation dimension.

Matrix multiplication is selected because it is a fundamental operation in many AI and ML workloads.

---

## 5. Systolic Array Concept

A systolic array consists of multiple Processing Elements connected in a structured and regular arrangement.

The target architecture contains 16 Processing Elements organized as a 4×4 array.

Conceptually:

                     Activation Flow →

              +------+ +------+ +------+ +------+
              | PE00 | | PE01 | | PE02 | | PE03 |
              +------+ +------+ +------+ +------+

              +------+ +------+ +------+ +------+
              | PE10 | | PE11 | | PE12 | | PE13 |
              +------+ +------+ +------+ +------+

              +------+ +------+ +------+ +------+
              | PE20 | | PE21 | | PE22 | | PE23 |
              +------+ +------+ +------+ +------+

              +------+ +------+ +------+ +------+
              | PE30 | | PE31 | | PE32 | | PE33 |
              +------+ +------+ +------+ +------+

Each Processing Element performs part of the overall computation and communicates with neighboring Processing Elements.

This distributed computation enables multiple MAC operations to take place in parallel.

---

## 6. Weight-Stationary Dataflow

The architecture uses a weight-stationary dataflow.

In this dataflow, the weight is loaded into a Processing Element and remains stationary while activation data moves through the array.

The basic PE operation can be represented as:

                 Weight
                    |
                    v
             +-------------+
    Activation -->|    PE    |--> Activation
             |   +-------------+
             |          |
             |          v
             |       PSUM
             |
          PSUM input

The fundamental computation performed by the PE is:

    PSUM_out = PSUM_in + (Activation × Weight)

The weight-stationary approach allows the stored weight to be reused for incoming activation values.

This reduces unnecessary movement of weight data and keeps the multiplication operation close to the stored weight.

---

## 7. Processing Element

The Processing Element is the fundamental building block of the systolic array.

The PE is responsible for:

1. Loading and storing a weight.
2. Receiving activation data.
3. Performing multiplication between activation and weight.
4. Adding the multiplication result to the incoming partial sum.
5. Forwarding activation data to the next PE.
6. Updating its internal state synchronously with the clock.

The fundamental MAC operation is:

    PSUM_out = PSUM_in + (Activation × Weight)

For example:

    Activation = 3
    Weight     = 5
    PSUM_in    = 10

Then:

    Activation × Weight = 3 × 5
                        = 15

Therefore:

    PSUM_out = 10 + 15
             = 25

This MAC operation forms the computational basis of the complete systolic array.

---

## 8. Activation Data Movement

Activation data is designed to propagate between neighboring Processing Elements.

At the PE level:

    Activation_in
          |
          v
        +----+
        | PE |
        +----+
          |
          v
    Activation_out

When multiple PEs are connected, this forwarding mechanism forms a data path through the array.

Conceptually:

    PE00 → PE01 → PE02 → PE03

The same concept is extended across the rows of the 4×4 array.

The PE implementation developed in this project has been simulated to verify activation forwarding behavior.

---

## 9. Partial-Sum Accumulation

Matrix multiplication requires multiple products to be added together to produce each output element.

The Processing Element therefore performs partial-sum accumulation.

The operation is:

    PSUM_out = PSUM_in + (Activation × Weight)

The incoming partial sum represents the accumulated result from previous computation stages.

The PE adds the current multiplication result to this partial sum and produces the updated partial sum.

This repeated accumulation enables multiple MAC operations to contribute to a final matrix result.

---

## 10. Motivation for the Project

The project is motivated by the increasing demand for specialized hardware accelerators for AI and ML workloads.

General-purpose processors provide flexibility but may require significant data movement and sequential execution for highly repetitive matrix operations.

A systolic array provides:

- Parallel MAC computation
- Local data communication
- Data reuse
- Regular hardware structure
- Predictable data movement
- Scalable architecture
- Efficient utilization of dedicated hardware resources

The project therefore provides practical exposure to the design principles used in hardware accelerators and AI-oriented digital architectures.

---

## 11. Why a 4×4 Architecture?

A 4×4 array was selected as the initial architecture because it provides a meaningful demonstration of systolic computation while keeping the RTL design and verification manageable.

The architecture contains:

    4 rows
    4 columns
    16 Processing Elements

A 4×4 configuration is large enough to demonstrate:

- Parallel computation
- PE-to-PE communication
- Weight reuse
- Activation propagation
- Partial-sum accumulation
- Pipeline behavior

At the same time, the relatively small array size makes functional verification and debugging practical.

The PE-based architecture can later be scaled to larger configurations.

For example:

    4×4   → 16 PEs
    8×8   → 64 PEs
    16×16 → 256 PEs

The current project focuses on the 4×4 configuration.

---

## 12. Technology and Design Methodology

The hardware is being developed using SystemVerilog RTL.

The design methodology follows a modular and bottom-up approach.

The development starts with the Processing Element because it is the fundamental computational unit.

After the PE is verified, multiple PE instances can be connected to form the complete systolic array.

The overall methodology is:

    Specification
        ↓
    Architecture
        ↓
    RTL Design
        ↓
    Unit-Level Verification
        ↓
    Integration
        ↓
    System-Level Verification
        ↓
    Synthesis
        ↓
    Timing / Area / Power Analysis

This methodology reduces design complexity and allows individual blocks to be verified before system integration.

---

## 13. Current Project Implementation

The Processing Element has been implemented as the first hardware block of the project.

The implemented PE provides the fundamental operations required by the target systolic architecture.

The PE implementation includes:

- Weight loading
- Weight storage
- Activation input
- Activation forwarding
- Multiplication
- Partial-sum accumulation
- Clocked operation
- Reset functionality

The PE has been integrated with a dedicated testbench for functional verification.

---

## 14. Current Verification Status

The PE-level verification has been successfully completed for the fundamental operations implemented so far.

The following behaviors have been verified through simulation:

- Reset behavior
- Weight loading
- Activation input handling
- Activation forwarding
- Multiply operation
- Partial-sum accumulation
- MAC operation

The PE simulation confirmed that the implemented MAC behavior produces the expected accumulated result.

The current verification stage is therefore:

    Processing Element
            ↓
       RTL implemented
            ↓
        Testbench
            ↓
        Simulation
            ↓
        Verification
            ↓
          PASS

The complete 4×4 array has not yet been verified at the system level.

---

## 15. Project Development Structure

The project is organized into separate directories for RTL, testbenches, simulation files, waveform results, and documentation.

Current project structure:

    systolic/
    │
    ├── rtl/
    │
    ├── tb/
    │
    ├── sim/
    │
    ├── wave/
    │
    └── docs/

### rtl/

Contains the synthesizable RTL implementation of the hardware design.

### tb/

Contains testbench files used to verify the RTL modules.

### sim/

Contains simulation-related files, scripts, and generated simulation information.

### wave/

Contains waveform outputs used for debugging and functional verification.

### docs/

Contains architecture, implementation, verification, and project documentation.

---

## 16. Verification Philosophy

Verification is being performed incrementally rather than waiting until the entire accelerator is completed.

The verification strategy follows:

    PE-level verification
            ↓
    Array-level verification
            ↓
    Top-level verification

The PE is verified independently before integrating multiple PEs.

This approach makes it easier to identify whether an error originates from:

- The PE computation
- Data forwarding
- PE interconnection
- Control logic
- Input/output handling
- Array-level scheduling

The next verification stage will focus on the integrated 4×4 systolic array.

---

## 17. Expected Accelerator Architecture

The complete target accelerator is expected to contain the following major functional blocks:

    Input / Activation Interface
              |
              v
       Activation Handling
              |
              v
       Weight-Stationary
         4×4 PE Array
              |
              v
       Partial Sum / Output
           Handling
              |
              v
            Output

Additional control and buffering structures will be introduced during later development stages.

These components are part of the planned complete accelerator and are not considered completed until implemented and verified.

---

## 18. Performance Considerations

The 4×4 architecture provides 16 Processing Elements.

When all Processing Elements are active, the array contains:

    16 parallel MAC units

The theoretical MAC throughput depends on the operating frequency and PE utilization.

For an operating frequency of F Hz:

    MAC throughput = 16 × F MAC/s

For example, at 100 MHz:

    MAC throughput = 16 × 100,000,000
                   = 1.6 × 10^9 MAC/s
                   = 1.6 GMAC/s

The 100 MHz example is only an illustration and is not a measured result of the current design.

Actual operating frequency, throughput, area, timing, and power will be reported after synthesis and implementation analysis.

---

## 19. Industry-Oriented Design Goals

The project is being developed with an ASIC/RTL engineering workflow in mind.

The intended final project will include:

- Modular RTL
- Functional verification
- Automated testbenches
- Waveform-based debugging
- Matrix-level verification
- Synthesis
- Timing analysis
- Area analysis
- Power estimation
- Reproducible simulation
- Engineering documentation

The goal is not only to demonstrate that the hardware works, but also to demonstrate the complete digital-design workflow from architecture to implementation and analysis.

---

## 20. Project Scope

### Included in the project

- Systolic array architecture study
- Weight-stationary dataflow
- Processing Element design
- MAC operation
- Activation forwarding
- Partial-sum accumulation
- SystemVerilog RTL
- Functional simulation
- PE-level verification
- 4×4 array architecture

### Planned for later stages

- 4×4 PE array integration
- Array-level matrix multiplication
- Controller design
- Input/activation buffering
- Weight buffering
- Output handling
- Top-level integration
- Automated array-level verification
- Synthesis
- Timing analysis
- Area analysis
- Power estimation
- Optimization

---

## 21. Current Limitations

The project is currently under active development.

The following items have not yet been completed:

- Complete 4×4 PE array integration
- Full matrix multiplication verification
- Accelerator-level controller
- Complete input/output buffering
- Top-level accelerator integration
- Synthesis
- Static timing analysis
- Power estimation
- Physical implementation

Therefore, the current project should be considered a verified PE-level implementation forming the foundation of the planned 4×4 systolic accelerator.

---

## 22. Project Status

Current development status:

| Component | Status |
|-----------|--------|
| Architecture definition | Completed |
| Weight-stationary dataflow | Completed |
| PE architecture | Completed |
| PE RTL | Completed |
| Weight loading | Completed |
| Activation forwarding | Completed |
| MAC operation | Completed |
| Partial-sum accumulation | Completed |
| PE testbench | Completed |
| PE simulation | Completed |
| PE functional verification | Completed |
| 4×4 PE array | In Progress |
| Array-level testbench | Planned |
| Matrix multiplication verification | Planned |
| Controller | Planned |
| Buffering | Planned |
| Top-level integration | Planned |
| Synthesis | Planned |
| Timing analysis | Planned |
| Area analysis | Planned |
| Power estimation | Planned |

---

## 23. Future Development

The project will be developed through the following stages.

### Phase 1 — Processing Element

Completed.

    PE RTL
    Weight loading
    Activation forwarding
    MAC
    Partial-sum accumulation
    PE testbench
    PE simulation

### Phase 2 — 4×4 Systolic Array

Next stage.

    Instantiate 16 PEs
    Connect activation paths
    Connect partial-sum paths
    Implement array-level dataflow
    Verify PE-to-PE communication

### Phase 3 — Matrix Multiplication Verification

    Develop matrix-level testbench
    Provide input matrices
    Calculate expected results
    Compare RTL results
    Verify complete matrix multiplication

### Phase 4 — Control and Data Handling

    Controller
    Input handling
    Weight loading control
    Output handling
    Buffering

### Phase 5 — Top-Level Integration

Integrate all functional blocks into a complete accelerator.

### Phase 6 — Synthesis and Analysis

Perform:

    RTL synthesis
    Area analysis
    Timing analysis
    Power estimation

### Phase 7 — Optimization

Investigate opportunities for:

- Higher operating frequency
- Improved PE utilization
- Reduced area
- Reduced switching activity
- Improved data movement efficiency
- Improved throughput

---

## 24. Expected Final Outcome

The final objective of the project is to produce a verified 4×4 weight-stationary systolic array accelerator implemented in SystemVerilog RTL.

The completed project will demonstrate the complete hardware-development flow:

    Architecture
        ↓
    Microarchitecture
        ↓
    RTL
        ↓
    Functional Verification
        ↓
    Integration
        ↓
    System-Level Verification
        ↓
    Synthesis
        ↓
    Timing / Area / Power Analysis

The project is intended to serve as a practical demonstration of RTL design, digital architecture, hardware verification, and AI accelerator design concepts.

---

## 25. Summary

This project implements the foundation of a 4×4 weight-stationary systolic array AI accelerator.

The architecture is based on 16 Processing Elements arranged in a two-dimensional array. Each Processing Element performs a multiply-accumulate operation while forwarding activation data through the systolic network.

The fundamental PE operation is:

    PSUM_out = PSUM_in + (Activation × Weight)

The Processing Element has been implemented and successfully verified through simulation for its fundamental operations, including weight loading, activation forwarding, multiplication, and partial-sum accumulation.

The next major development stage is the integration of the verified Processing Element into the complete 4×4 systolic array, followed by array-level matrix multiplication verification.

This project is being developed incrementally with the goal of eventually demonstrating an end-to-end RTL-to-synthesis workflow for a small AI hardware accelerator.
