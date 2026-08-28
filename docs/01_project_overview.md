# 4×4 Weight-Stationary Systolic Array AI Accelerator

## 1. Project Overview

This project focuses on the design and RTL implementation of a 4×4 weight-stationary systolic array accelerator for matrix multiplication and AI-oriented computational workloads.

The accelerator is designed using Verilog RTL and follows a modular hardware architecture based on Processing Elements (PEs).

The target architecture consists of:

    4 rows
    4 columns
    16 Processing Elements

Each Processing Element performs a multiply-accumulate (MAC) operation and participates in structured data movement through the systolic array.

The fundamental PE operation is:

    PSUM_out = PSUM_in + (Activation × Weight)

The architecture is based on the weight-stationary dataflow, where weights are stored locally within the Processing Elements while activation data is propagated through the array.

The project is being developed incrementally, beginning with the Processing Element, followed by array integration, functional verification, and eventually ASIC-oriented implementation and analysis.

---

## 2. Project Objective

The primary objective of this project is to design and verify a hardware accelerator capable of performing matrix multiplication using a 4×4 systolic array.

The project focuses on the following hardware concepts:

- Digital hardware design
- Verilog RTL design
- Processing Element architecture
- Multiply-accumulate computation
- Weight-stationary dataflow
- Systolic data movement
- Parallel processing
- Pipeline-based computation
- RTL simulation
- Functional verification
- Hardware-oriented optimization
- ASIC design methodology

The project is intended to provide practical experience in designing an AI accelerator at the RTL level.

---

## 3. Target Architecture

The target architecture contains 16 Processing Elements arranged in a 4×4 array.

The conceptual structure is:

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

Each Processing Element performs local computation while communicating with neighboring Processing Elements.

The regular structure makes the architecture suitable for parallel matrix multiplication.

---

## 4. Processing Element

The Processing Element is the fundamental computational unit of the accelerator.

A PE performs:

    Product = Activation × Weight

followed by:

    PSUM_out = PSUM_in + Product

Therefore:

    PSUM_out = PSUM_in + (Activation × Weight)

The PE also provides activation forwarding so that activation data can propagate through the systolic array.

The major functions of the PE are:

- Weight loading
- Weight storage
- Activation input
- Activation forwarding
- Multiplication
- Partial-sum accumulation
- MAC computation
- Clocked operation
- Reset initialization

The PE has been implemented and functionally verified through RTL simulation.

---

## 5. Weight-Stationary Dataflow

The selected dataflow for the accelerator is weight-stationary.

The main principle is:

    Weight
       ↓
    Stored locally in PE
       ↓
    Reused during computation

while:

    Activation
       ↓
    PE
       ↓
    Next PE
       ↓
    Next PE

The weight remains stationary inside the PE while activation values move through the processing elements.

This approach provides weight reuse and reduces unnecessary movement of weight data.

---

## 6. Matrix Multiplication

The target computation is:

    C = A × B

For the target architecture:

    A(4×4) × B(4×4) = C(4×4)

Each output element is a dot product.

For example:

    C[0][0] =
        A[0][0] × B[0][0]
      + A[0][1] × B[1][0]
      + A[0][2] × B[2][0]
      + A[0][3] × B[3][0]

In general:

    C[i][j] = Σ(A[i][k] × B[k][j])

where:

    i = 0 to 3
    j = 0 to 3
    k = 0 to 3

The systolic array is intended to perform these operations using multiple Processing Elements operating in parallel.

---

## 7. Why a Systolic Array?

A systolic array is useful for matrix multiplication because computation and data movement are distributed across multiple Processing Elements.

Instead of using one MAC unit repeatedly:

    Input
      ↓
    Single MAC
      ↓
    Result
      ↓
    Repeat

the systolic architecture provides multiple MAC units:

    +----+ +----+ +----+ +----+
    | PE | | PE | | PE | | PE |
    +----+ +----+ +----+ +----+
    +----+ +----+ +----+ +----+
    | PE | | PE | | PE | | PE |
    +----+ +----+ +----+ +----+
    +----+ +----+ +----+ +----+
    | PE | | PE | | PE | | PE |
    +----+ +----+ +----+ +----+
    +----+ +----+ +----+ +----+
    | PE | | PE | | PE | | PE |
    +----+ +----+ +----+ +----+

This enables spatial parallelism.

---

## 8. Main Design Characteristics

The target accelerator has the following characteristics:

| Characteristic | Description |
|----------------|-------------|
| Architecture | Systolic Array |
| Array Size | 4×4 |
| Number of PEs | 16 |
| Dataflow | Weight-Stationary |
| Core Operation | Multiply-Accumulate |
| RTL Language | Verilog |
| Design Style | Synchronous Digital RTL |
| Primary Workload | Matrix Multiplication |
| Verification | RTL Simulation |
| Target Flow | ASIC-oriented digital design |

---

## 9. Project Development Methodology

The project follows a bottom-up hardware development methodology.

The development sequence is:

    Architecture Definition
            ↓
    Processing Element Design
            ↓
    Verilog RTL Implementation
            ↓
    PE Testbench Development
            ↓
    RTL Simulation
            ↓
    PE Functional Verification
            ↓
    4×4 Array Integration
            ↓
    Array-Level Verification
            ↓
    Matrix Multiplication Verification
            ↓
    Top-Level Integration
            ↓
    RTL Quality Checks
            ↓
    Logic Synthesis
            ↓
    Timing Analysis
            ↓
    Area Analysis
            ↓
    Power Analysis
            ↓
    Physical Design
            ↓
    GDSII

The project is being developed incrementally so that each stage can be verified before proceeding to the next stage.

---

## 10. Current Implementation Status

The Processing Element is the first major hardware block implemented in the project.

The following PE functionality has been implemented and verified:

- PE reset behavior
- Weight loading
- Weight storage
- Activation input
- Activation forwarding
- Multiplication
- Partial-sum accumulation
- MAC operation
- RTL simulation

The PE testbench was used to provide input stimulus and verify the expected output behavior.

The successful PE verification establishes the foundation for integrating multiple PEs into the target 4×4 systolic array.

---

## 11. Current Project Boundary

At the current stage, the verified hardware boundary is the Processing Element.

The development status can be represented as:

    +----------------------+
    | Processing Element   |
    |                      |
    | Weight Storage       |
    | Activation Path      |
    | Multiplier           |
    | Accumulator           |
    +----------+-----------+
               |
               v
          PE Testbench
               |
               v
          RTL Simulation
               |
               v
       Functional Verification
               |
               v
              PASS

The complete 4×4 accelerator is the target architecture and requires further integration and verification.

---

## 12. Repository Structure

The project directory is organized as:

    ~/systolic

with the following main directories:

    rtl/
    tb/
    sim/
    wave/
    docs/

### rtl/

Contains synthesizable Verilog RTL design files.

### tb/

Contains Verilog testbench files used for functional verification.

### sim/

Contains simulation-related files and scripts.

### wave/

Contains waveform-related simulation artifacts.

### docs/

Contains project documentation.

This structure separates design RTL, verification code, simulation artifacts, waveform files, and documentation.

---

## 13. Design Philosophy

The project follows the following design principles:

### Modularity

The Processing Element is implemented as an independent reusable hardware block.

### Reusability

The same PE can be instantiated multiple times to construct the systolic array.

### Incremental Verification

Each major block is verified before integration.

### Structured Dataflow

The architecture uses predictable data movement between Processing Elements.

### Hardware Parallelism

Multiple Processing Elements operate concurrently.

### Data Reuse

The weight-stationary architecture keeps weights local to the Processing Elements.

### Scalability

The PE-based architecture can potentially be extended to larger systolic arrays.

---

## 14. Industry-Oriented Development

The project is being documented and developed using practices commonly associated with professional RTL development.

These include:

- Modular RTL design
- Clear directory organization
- Version control using Git
- Dedicated testbenches
- Functional verification
- Waveform-based debugging
- Reproducible simulation
- Design documentation
- Incremental development
- Synthesis-oriented RTL
- Timing analysis
- Area analysis
- Power analysis

The final objective is to move beyond simple RTL simulation and demonstrate an end-to-end digital hardware design methodology.

---

## 15. Expected Final Deliverables

The completed project is intended to contain:

### RTL

- Processing Element
- 4×4 systolic array
- Required control logic
- Input handling
- Weight handling
- Output handling
- Top-level accelerator

### Verification

- PE testbench
- Array-level testbench
- Matrix multiplication test cases
- Expected-result checking
- Waveform analysis
- Regression tests

### Documentation

- Project overview
- Architecture
- PE design
- Dataflow
- RTL implementation
- Verification methodology
- Simulation methodology
- Matrix multiplication mapping
- Design decisions
- Project status
- Future work

### Hardware Analysis

After functional verification:

- RTL synthesis
- Timing analysis
- Area analysis
- Power estimation
- Physical design, if supported by the available tool flow

---

## 16. Performance Perspective

The target array contains:

    4 × 4 = 16 Processing Elements

Each PE contains the capability to perform a MAC operation.

Therefore, when all Processing Elements are active:

    Theoretical MAC operations per cycle = 16

For a clock frequency of F Hz:

    Theoretical MAC throughput = 16 × F MAC/s

For example, at 100 MHz:

    16 × 100,000,000
    = 1,600,000,000 MAC/s
    = 1.6 GMAC/s

The 100 MHz value is only an example calculation.

It is not a measured frequency or performance result of the current design.

Actual performance will be reported only after implementation and timing analysis.

---

## 17. Verification Philosophy

Functional correctness is established progressively.

The verification hierarchy is:

    PE Verification
          ↓
    Multi-PE Verification
          ↓
    4×4 Array Verification
          ↓
    Matrix Multiplication Verification
          ↓
    Top-Level Verification

At each stage, the RTL output should be compared against an independently calculated expected result.

A successful PE simulation does not automatically mean that the complete 4×4 accelerator is verified.

Array-level verification is required to verify:

- Correct PE interconnection
- Correct weight mapping
- Correct activation scheduling
- Correct partial-sum handling
- Correct output generation
- Correct matrix multiplication

---

## 18. ASIC Design Perspective

The project is intended to provide a foundation for understanding an ASIC-oriented digital design flow.

The planned flow is:

    Specification
        ↓
    Architecture
        ↓
    Verilog RTL
        ↓
    Functional Verification
        ↓
    RTL Quality Checks
        ↓
    Logic Synthesis
        ↓
    Gate-Level Netlist
        ↓
    Static Timing Analysis
        ↓
    Floorplanning
        ↓
    Placement
        ↓
    Clock Tree Synthesis
        ↓
    Routing
        ↓
    Physical Verification
        ↓
    GDSII

The current project stage is RTL implementation and functional verification.

---

## 19. Important Scope Definition

The project should clearly distinguish between:

### Implemented

- PE architecture
- PE Verilog RTL
- PE testbench
- PE simulation
- PE functional verification
- Weight loading
- Activation forwarding
- MAC computation
- Partial-sum behavior

### Target / Under Development

- Complete 4×4 PE array
- Array-level dataflow
- Matrix multiplication verification
- Top-level accelerator
- Control logic
- Buffering
- Synthesis
- Timing analysis
- Area analysis
- Power analysis
- Physical implementation

This distinction is important for maintaining technically accurate project documentation.

---

## 20. Project Significance

This project demonstrates practical implementation of a hardware accelerator architecture rather than only theoretical study.

It combines:

    Digital Design
         +
    Verilog RTL
         +
    MAC Architecture
         +
    Systolic Dataflow
         +
    Parallel Processing
         +
    Functional Verification
         +
    ASIC Design Concepts

The project provides a foundation for understanding how AI-oriented computation can be mapped onto dedicated digital hardware.

---

## 21. Summary

This project implements a 4×4 weight-stationary systolic array architecture for accelerating matrix multiplication.

The target architecture consists of 16 Processing Elements.

Each Processing Element performs:

    PSUM_out = PSUM_in + (Activation × Weight)

The architecture keeps weights locally stored in the Processing Elements while activation data propagates through the array.

The Processing Element has already been implemented in Verilog RTL and functionally verified through simulation.

The next major development stage is to integrate the verified Processing Element into the complete 4×4 systolic array and verify matrix multiplication at the array level.

The project is structured to eventually progress from RTL design and simulation toward synthesis, timing, area, power, and physical-design analysis.
