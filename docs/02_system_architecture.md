# System Architecture

## 1. Architecture Overview

The project is a 4×4 weight-stationary systolic array accelerator designed for matrix multiplication workloads.

The architecture is built around a two-dimensional array of Processing Elements (PEs). Each PE performs a multiply-accumulate (MAC) operation and communicates with neighboring PEs through structured data paths.

The target array contains:

    4 rows
    4 columns
    16 Processing Elements

The architecture is designed to exploit parallel computation and data reuse while reducing unnecessary movement of weights.

The fundamental computational operation of the PE is:

    PSUM_out = PSUM_in + (Activation × Weight)

The complete accelerator is being developed incrementally, beginning with the Processing Element and progressing toward the integrated 4×4 array.

---

## 2. Top-Level Architectural Concept

The target 4×4 systolic array can be represented as:

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

Each PE represents one computational unit.

Therefore:

    Total PEs = 4 × 4 = 16

The PEs operate synchronously and exchange data with neighboring elements.

---

## 3. Architectural Building Blocks

The complete target accelerator is organized around the following functional blocks:

    +----------------------+
    | Input / Activation   |
    | Interface            |
    +----------+-----------+
               |
               v
    +----------------------+
    | Activation Data      |
    | Handling             |
    +----------+-----------+
               |
               v
    +--------------------------------+
    |                                |
    |      4 × 4 Systolic Array      |
    |                                |
    |   +----+ +----+ +----+ +----+ |
    |   | PE | | PE | | PE | | PE | |
    |   +----+ +----+ +----+ +----+ |
    |   +----+ +----+ +----+ +----+ |
    |   | PE | | PE | | PE | | PE | |
    |   +----+ +----+ +----+ +----+ |
    |   +----+ +----+ +----+ +----+ |
    |   | PE | | PE | | PE | | PE | |
    |   +----+ +----+ +----+ +----+ |
    |   +----+ +----+ +----+ +----+ |
    |   | PE | | PE | | PE | | PE | |
    |   +----+ +----+ +----+ +----+ |
    |                                |
    +---------------+----------------+
                    |
                    v
          +--------------------+
          | Output / Partial   |
          | Sum Handling       |
          +--------------------+
                    |
                    v
                 Output

Control and buffering structures will be integrated as the project progresses.

At the current stage, the Processing Element is the implemented and verified hardware block.

---

## 4. Processing Element Architecture

The Processing Element is the fundamental computational unit of the accelerator.

Each PE performs the following functions:

1. Receives an activation.
2. Stores a weight.
3. Multiplies the activation by the stored weight.
4. Adds the multiplication result to the incoming partial sum.
5. Produces an updated partial sum.
6. Forwards the activation to the next PE.

The conceptual PE structure is:

                    Weight
                       |
                       v
               +---------------+
               |               |
    Activation |      PE       | Activation
    ---------->|               |---------->
               |               |
    PSUM_in -->|    MAC Unit   |
               |               |
               +-------+-------+
                       |
                       v
                   PSUM_out

The fundamental operation is:

    PSUM_out = PSUM_in + (Activation × Weight)

---

## 5. Weight-Stationary Dataflow

The selected dataflow for the architecture is weight-stationary.

In a weight-stationary architecture, weights are loaded into the Processing Elements and remain locally stored during computation.

Activation data is then propagated through the array.

Conceptually:

                 Weight
                   |
                   v
             +-----------+
             |           |
    Input -->|    PE     |--> Output activation
             |           |
             +-----+-----+
                   |
                   v
                PSUM

The key idea is:

    Weight → remains inside PE
    Activation → moves through PE
    Partial Sum → updated during MAC operation

This allows a weight to be reused while different activation values arrive at the PE.

---

## 6. Reason for Weight-Stationary Dataflow

Weight-stationary dataflow was selected because it provides an efficient way to reuse weights during repeated MAC operations.

The major advantages are:

### 6.1 Weight Reuse

A loaded weight can participate in multiple MAC operations without requiring the weight to be reloaded for every activation.

### 6.2 Reduced Data Movement

Keeping weights local reduces unnecessary movement of weight data between memory and computation units.

### 6.3 Localized Computation

The multiplication takes place directly inside the PE using the locally stored weight.

### 6.4 Regular Architecture

The dataflow produces a regular communication pattern between neighboring PEs.

### 6.5 Scalability

The same PE structure can be replicated to create larger arrays.

For example:

    4 × 4   = 16 PEs
    8 × 8   = 64 PEs
    16 × 16 = 256 PEs

The current design target is a 4×4 array.

---

## 7. Activation Data Path

Activation data propagates between neighboring PEs.

A simplified row is:

    Activation
        |
        v
    +------+      +------+      +------+      +------+
    | PE00 | ---> | PE01 | ---> | PE02 | ---> | PE03 |
    +------+      +------+      +------+      +------+

The PE receives an activation at its input and produces an activation output that is connected to the next PE.

This creates a pipeline-like movement of activation data through the array.

The activation forwarding mechanism has already been implemented and verified at the PE level.

---

## 8. Partial-Sum Data Path

Partial sums represent intermediate results generated during matrix multiplication.

The PE receives a partial sum and combines it with the current multiplication result.

The operation is:

    PSUM_out = PSUM_in + (Activation × Weight)

The data path is:

    PSUM_in
       |
       v
    +--------+
    |   PE   |
    +--------+
       ^
       |
    Activation × Weight
       |
       v
    PSUM_out

Repeated accumulation allows multiple products to contribute to the final output value.

---

## 9. Matrix Multiplication Mapping

The target computation is:

    C = A × B

For a 4×4 matrix:

    A(4×4) × B(4×4) = C(4×4)

Each output element is:

    C[i][j] =
        A[i][0] × B[0][j]
      + A[i][1] × B[1][j]
      + A[i][2] × B[2][j]
      + A[i][3] × B[3][j]

The systolic architecture distributes these multiplication and accumulation operations across the PE array.

The weights are mapped to the PEs according to the selected weight-stationary dataflow.

Activation values are then propagated through the array so that the required multiplication operations can occur.

The exact cycle-by-cycle matrix mapping will be finalized and documented during array-level integration.

---

## 10. PE-to-PE Communication

The architecture uses local communication between neighboring Processing Elements.

Instead of requiring every PE to communicate with a centralized processing unit, the systolic array uses structured connections.

Conceptually:

    PE00 → PE01 → PE02 → PE03
      ↓      ↓      ↓      ↓
    PE10 → PE11 → PE12 → PE13
      ↓      ↓      ↓      ↓
    PE20 → PE21 → PE22 → PE23
      ↓      ↓      ↓      ↓
    PE30 → PE31 → PE32 → PE33

The exact direction and connection of each data path will depend on the final array implementation.

The architecture is designed around regular neighboring communication.

---

## 11. Clocking

The Processing Elements operate synchronously with a common clock.

The clock provides the timing reference for:

- Weight loading
- Activation propagation
- Partial-sum updates
- Register updates
- Synchronous data movement

The PE-level implementation uses clocked RTL to control state changes.

The complete array will use the same synchronous design principle.

---

## 12. Reset Architecture

Reset is provided to initialize the internal state of the Processing Element.

Reset is required because the PE contains state associated with:

- Stored weight
- Partial sum
- Other registered signals

The reset operation places the PE into a known initial state before normal computation begins.

Reset behavior has been included in the PE verification.

The complete array will use the PE reset mechanism during initialization.

---

## 13. Parallel Computation

One of the main advantages of the architecture is spatial parallelism.

The target 4×4 array contains:

    16 Processing Elements

Therefore, when all PEs are active, the architecture provides:

    16 parallel MAC units

This allows multiple multiplication and accumulation operations to occur concurrently.

Compared with a single-MAC sequential implementation, the systolic architecture can exploit substantially greater hardware parallelism.

---

## 14. Pipeline Operation

The systolic array processes data over multiple clock cycles.

Data does not appear at every output simultaneously when computation starts. Instead, values propagate through the PE network over successive clock cycles.

The general behavior is:

    Cycle 0
       |
       v
    Initial data enters the array

    Cycle 1
       |
       v
    Data moves to neighboring PEs

    Cycle 2
       |
       v
    Additional MAC operations occur

    Cycle 3
       |
       v
    Data continues propagating

      ...

    Final cycles
       |
       v
    Accumulated results become available

Once the pipeline reaches steady state, multiple PEs can perform MAC operations simultaneously.

The exact latency and cycle schedule will be measured during array-level verification.

---

## 15. Architectural Scalability

The architecture is based on a reusable PE.

The same PE can be instantiated multiple times to construct larger systolic arrays.

For example:

    2×2 → 4 PEs
    4×4 → 16 PEs
    8×8 → 64 PEs
    16×16 → 256 PEs

The 4×4 configuration is used as the current project target because it provides a practical balance between architectural complexity and verification effort.

The modular PE design also makes it possible to study the effect of increasing array size in future work.

---

## 16. Data Reuse

Data reuse is a major motivation for the systolic architecture.

The architecture attempts to minimize unnecessary movement of frequently used operands.

In the selected weight-stationary approach:

    Weight
       ↓
    Stored locally in PE
       ↓
    Reused for incoming activations

At the same time:

    Activation
       ↓
    PE
       ↓
    Next PE
       ↓
    Next PE
       ↓
    ...

This structured data movement allows computation to take place close to where data is being used.

---

## 17. Processing Element as a Reusable IP Block

The PE is designed as a modular unit rather than as logic specifically tied to one location in the array.

This provides several benefits:

- Reusability
- Easier verification
- Easier debugging
- Consistent behavior
- Simplified array construction
- Scalability

The same PE design can be instantiated multiple times to form the 4×4 array.

---

## 18. Architectural Development Methodology

The accelerator is being developed using a bottom-up methodology.

The development sequence is:

    1. Define system architecture
                ↓
    2. Define PE architecture
                ↓
    3. Implement PE RTL
                ↓
    4. Develop PE testbench
                ↓
    5. Verify PE functionality
                ↓
    6. Integrate 16 PEs
                ↓
    7. Verify array dataflow
                ↓
    8. Verify matrix multiplication
                ↓
    9. Integrate control and buffering
                ↓
    10. Verify complete accelerator
                ↓
    11. Synthesize RTL
                ↓
    12. Analyze timing, area and power

The project is currently between steps 5 and 6.

---

## 19. Current Implementation Boundary

The current verified hardware block is the Processing Element.

The development status is:

    Processing Element
           |
           v
    RTL Implementation
           |
           v
    Testbench
           |
           v
    Simulation
           |
           v
    Functional Verification
           |
           v
         PASS

The complete 4×4 systolic array is the next major implementation stage.

Therefore, the current documentation distinguishes between:

### Implemented and verified

- Processing Element
- Weight loading
- Activation forwarding
- MAC operation
- Partial-sum accumulation
- PE-level simulation

### Defined but not yet completed

- 4×4 PE integration
- Array-level communication
- Matrix multiplication verification
- Controller
- Buffering
- Top-level accelerator

---

## 20. Target Complete Accelerator

The final target architecture is expected to include:

    +----------------------+
    | Input Interface      |
    +----------+-----------+
               |
               v
    +----------------------+
    | Input / Activation   |
    | Handling             |
    +----------+-----------+
               |
               v
    +----------------------+
    | Weight Management    |
    +----------+-----------+
               |
               v
    +--------------------------------+
    |                                |
    |        4×4 PE Array            |
    |                                |
    |  16 Weight-Stationary PEs      |
    |                                |
    +---------------+----------------+
                    |
                    v
    +----------------------+
    | Output / PSUM        |
    | Handling             |
    +----------+-----------+
               |
               v
    +----------------------+
    | Output Interface     |
    +----------------------+

Control logic will coordinate the operation of the accelerator.

Input, weight, and output buffering will be developed as part of later integration stages.

---

## 21. Performance Model

The target architecture contains 16 PE-based MAC units.

When all PEs are active:

    MAC operations per cycle = 16

If the complete array operates at a clock frequency of F Hz:

    Theoretical MAC throughput = 16 × F MAC/s

For example, at 100 MHz:

    16 × 100,000,000
    = 1,600,000,000 MAC/s
    = 1.6 GMAC/s

The 100 MHz value is only an example and is not a measured characteristic of the current design.

Actual performance will be reported only after synthesis and timing analysis.

---

## 22. Design Considerations

The architecture must consider several hardware design factors during later development:

### Data precision

Operand widths directly affect:

- Multiplier size
- Accumulator size
- Area
- Power
- Throughput

### PE utilization

Maximum theoretical parallelism is achieved only when the PEs are sufficiently utilized.

### Data movement

The architecture must ensure that activations and partial sums arrive at the correct PEs at the correct clock cycles.

### Pipeline latency

The number of cycles required for data to propagate through the array affects overall latency.

### Memory bandwidth

The ability to supply activations and weights efficiently affects accelerator utilization.

### Timing

The MAC datapath and interconnect must satisfy the target clock period after synthesis.

---

## 23. Verification Considerations

The architecture will be verified progressively.

### PE level

Already completed:

- Reset
- Weight loading
- Activation forwarding
- Multiplication
- Partial-sum accumulation
- MAC operation

### Array level

Planned:

- PE-to-PE communication
- Activation propagation
- Weight mapping
- Partial-sum propagation
- Matrix multiplication
- Output correctness

### Top level

Planned:

- Control
- Input handling
- Weight handling
- Output handling
- End-to-end operation

---

## 24. Current Architecture Status

| Architectural Component | Status |
|--------------------------|--------|
| Systolic architecture definition | Completed |
| 4×4 array specification | Completed |
| Weight-stationary dataflow | Completed |
| PE architecture | Completed |
| PE RTL | Completed |
| PE verification | Completed |
| PE activation forwarding | Completed |
| PE MAC operation | Completed |
| 4×4 PE integration | In Progress |
| Array communication | Planned |
| Array-level verification | Planned |
| Matrix multiplication | Planned |
| Controller | Planned |
| Input buffering | Planned |
| Weight buffering | Planned |
| Output handling | Planned |
| Top-level integration | Planned |
| Synthesis | Planned |
| Timing analysis | Planned |
| Area analysis | Planned |
| Power estimation | Planned |

---

## 25. Summary

The target system is a 4×4 weight-stationary systolic array consisting of 16 Processing Elements.

The Processing Element is the fundamental computational unit and performs:

    PSUM_out = PSUM_in + (Activation × Weight)

Weights are intended to remain locally available within the Processing Elements while activation data propagates through the array.

The architecture exploits parallel MAC computation, local communication, and data reuse to accelerate matrix multiplication workloads.

The Processing Element has already been implemented and functionally verified through simulation. The PE verification confirmed the fundamental behaviors required for array integration, including weight loading, activation forwarding, MAC computation, and partial-sum accumulation.

The next major development step is to integrate the verified PE into the complete 4×4 systolic array and verify the array using actual matrix multiplication test cases.

The architecture will subsequently be extended with control, buffering, top-level integration, and ASIC-oriented synthesis and analysis.
