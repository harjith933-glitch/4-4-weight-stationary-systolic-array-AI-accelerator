# System Architecture

## 1. Overview

The proposed accelerator is a 4×4 weight-stationary systolic array designed for parallel matrix multiplication and AI-oriented computational workloads.

The architecture consists of 16 Processing Elements (PEs) arranged in four rows and four columns.

The basic computational operation performed by each PE is a Multiply-Accumulate (MAC):

    PSUM_out = PSUM_in + (Activation × Weight)

The architecture is designed around three fundamental principles:

1. Weights are stored locally inside Processing Elements.
2. Activation data propagates through the array.
3. Partial sums are accumulated as computation progresses.

The architecture is implemented using Verilog RTL.

---

## 2. Top-Level Architecture

The target architecture can be represented as:

                    Activation Flow →

          +--------+--------+--------+--------+
          |  PE00  |  PE01  |  PE02  |  PE03  |
          +--------+--------+--------+--------+
             |        |        |        |
             v        v        v        v
          +--------+--------+--------+--------+
          |  PE10  |  PE11  |  PE12  |  PE13  |
          +--------+--------+--------+--------+
             |        |        |        |
             v        v        v        v
          +--------+--------+--------+--------+
          |  PE20  |  PE21  |  PE22  |  PE23  |
          +--------+--------+--------+--------+
             |        |        |        |
             v        v        v        v
          +--------+--------+--------+--------+
          |  PE30  |  PE31  |  PE32  |  PE33  |
          +--------+--------+--------+--------+

The above represents the target 4×4 array.

Each PE operates as an independent computational unit while exchanging data with neighboring PEs.

---

## 3. Number of Processing Elements

The array dimensions are:

    Rows    = 4
    Columns = 4

Therefore:

    Total PEs = Rows × Columns

    Total PEs = 4 × 4

    Total PEs = 16

The 16 Processing Elements provide spatial parallelism for matrix multiplication.

---

## 4. Processing Element Organization

Each Processing Element contains the basic functionality required for a MAC-based systolic architecture.

Conceptually:

    +--------------------------------+
    |       Processing Element       |
    |                                |
    |   Weight Register              |
    |        |                       |
    |        v                       |
    |   +-----------+                |
    |   | Multiplier|                |
    |   +-----+-----+                |
    |         |                      |
    |         v                      |
    |   +-----------+                |
    |   |   Adder   | <--- PSUM_in   |
    |   +-----+-----+                |
    |         |                      |
    |         v                      |
    |      PSUM_out                  |
    |                                |
    | Activation_in                  |
    |       |                        |
    |       v                        |
    | Activation_out                 |
    +--------------------------------+

The PE therefore combines computation, local weight storage, and data forwarding.

---

## 5. PE Inputs

The Processing Element requires the following conceptual inputs:

### Clock

The clock synchronizes sequential operations.

### Reset

Reset initializes the internal state of the PE.

### Activation Input

The activation value enters the PE and participates in the MAC operation.

### Weight Input

The weight value is supplied during the weight-loading operation.

### Weight Load Control

This control determines when the incoming weight should be stored.

### Partial Sum Input

The incoming partial sum is added to the multiplication result.

The exact signal names and widths are defined by the Verilog RTL implementation.

---

## 6. PE Outputs

The Processing Element provides the following conceptual outputs:

### Activation Output

The activation is forwarded to the next PE in the data path.

### Partial Sum Output

The updated accumulated result is produced.

Conceptually:

    activation_in
          |
          v
         PE
          |
          +-----> activation_out
          |
          +-----> psum_out

The exact output timing depends on the RTL implementation.

---

## 7. Weight-Stationary Architecture

The selected dataflow is weight-stationary.

The main idea is to load a weight into a PE and keep that weight locally available during computation.

The conceptual sequence is:

    Weight Input
         |
         v
    Weight Load
         |
         v
    PE Weight Register
         |
         v
    Weight remains stored
         |
         v
    Multiple MAC operations

While the weight remains stationary:

    Activation 0
         |
         v
        PE
         |
         v
    Activation 1
         |
         v
        PE
         |
         v
    Activation 2
         |
         v
        PE

This provides weight reuse.

---

## 8. Activation Data Path

Activation data moves between Processing Elements.

A simplified row-level path is:

    Activation Input
          |
          v
        PE00
          |
          v
        PE01
          |
          v
        PE02
          |
          v
        PE03
          |
          v
      Row Output

The activation output of one PE is connected to the activation input of the next PE.

This creates the systolic propagation mechanism.

---

## 9. Partial Sum Data Path

The partial sum represents the accumulated result of previous MAC operations.

A simplified PE datapath is:

    PSUM_in
       |
       v
    +------+
    |  PE  |
    +--+---+
       |
       v
    PSUM_out

Inside the PE:

    PSUM_out = PSUM_in + (Activation × Weight)

The partial sum is therefore updated every time a valid MAC operation takes place.

---

## 10. MAC Datapath

The complete computational path is:

    Activation
        |
        v
    +-----------+
    | Multiplier|
    +-----+-----+
          |
          v
       Product
          |
          v
    +-----------+
    |   Adder   | <----- PSUM_in
    +-----+-----+
          |
          v
       PSUM_out

The multiplication is performed using the activation and locally stored weight.

The multiplication result is then added to the incoming partial sum.

---

## 11. Mathematical Operation

For one PE:

    Product = Activation × Weight

Then:

    PSUM_out = PSUM_in + Product

Therefore:

    PSUM_out = PSUM_in + (Activation × Weight)

For multiple operations:

    PSUM_final =
        PSUM_initial
        + A0×W0
        + A1×W1
        + A2×W2
        + A3×W3

For a new calculation:

    PSUM_initial = 0

Therefore:

    PSUM_final =
        A0×W0
        + A1×W1
        + A2×W2
        + A3×W3

This represents a dot product.

---

## 12. Matrix Multiplication Mapping

The target operation is:

    C = A × B

For two 4×4 matrices:

    A = 4×4

    B = 4×4

The result is:

    C = 4×4

Each output element is:

    C[i][j] = Σ(A[i][k] × B[k][j])

where:

    i = 0, 1, 2, 3
    j = 0, 1, 2, 3
    k = 0, 1, 2, 3

Each output therefore requires four multiplication operations and three additions.

The systolic architecture distributes these operations across the PE array.

---

## 13. Example Output Calculation

For example:

    C[0][0] =
        A[0][0]×B[0][0]
      + A[0][1]×B[1][0]
      + A[0][2]×B[2][0]
      + A[0][3]×B[3][0]

Assuming:

    A[0] = [1, 2, 3, 4]

and:

    B column 0 = [5, 6, 7, 8]

then:

    C[0][0] =
        (1×5)
      + (2×6)
      + (3×7)
      + (4×8)

    C[0][0] =
        5 + 12 + 21 + 32

    C[0][0] = 70

The hardware performs the same mathematical operation using the PE array.

---

## 14. Systolic Array Data Movement

The defining characteristic of the architecture is synchronized data movement.

A simplified view is:

    Activation →
    
    PE00 → PE01 → PE02 → PE03
      ↓      ↓      ↓      ↓
    PE10 → PE11 → PE12 → PE13
      ↓      ↓      ↓      ↓
    PE20 → PE21 → PE22 → PE23
      ↓      ↓      ↓      ↓
    PE30 → PE31 → PE32 → PE33

The exact final interconnection of activation and partial-sum paths will be established in the array RTL.

The architecture is designed so that neighboring PEs communicate rather than requiring every PE to communicate directly with a global source.

---

## 15. Boundary Processing Elements

The PEs at the edges of the array have different connectivity compared with internal PEs.

For example:

    PE00

is located at the top-left corner.

It does not have a PE on its left or above it.

Similarly:

    PE33

is located at the bottom-right corner.

It does not have a PE on its right or below it.

Therefore, boundary connections must be handled explicitly during array integration.

---

## 16. Internal Processing Elements

Internal PEs have neighboring Processing Elements around them.

For example:

    PE11

is surrounded by:

    PE01
    PE10
    PE12
    PE21

depending on the direction of the selected data paths.

Internal PEs therefore participate in the main systolic communication network.

---

## 17. Weight Loading Architecture

Before computation, weights must be loaded into the Processing Elements.

Conceptually:

    Weight Source
         |
         v
    Weight Distribution
         |
         v
    PE Weight Registers

For the complete array, each PE must receive the correct weight corresponding to the intended matrix multiplication mapping.

The weight-loading schedule is therefore an important part of array-level control.

---

## 18. Weight Reuse

After a weight has been loaded:

    Weight
       ↓
    PE Register
       ↓
    MAC Operation 1
       ↓
    MAC Operation 2
       ↓
    MAC Operation 3
       ↓
    MAC Operation 4

The weight can participate in multiple calculations without requiring a new external weight transfer for every MAC operation.

This is one of the main motivations for selecting weight-stationary dataflow.

---

## 19. Activation Reuse

Activation data also participates in multiple computations as it moves through the array.

For example:

    A0
     |
     v
    PE00
     |
     v
    PE01
     |
     v
    PE02
     |
     v
    PE03

The same activation can therefore interact with multiple stored weights as it propagates.

The exact scheduling determines which matrix elements are being computed at each cycle.

---

## 20. Parallel Computation

The array provides parallel hardware resources.

Instead of calculating:

    C[0][0]
    C[0][1]
    C[0][2]
    ...

one after another using a single MAC unit, multiple PEs can operate concurrently.

The target architecture provides:

    16 Processing Elements

and therefore potentially:

    16 MAC operations per cycle

when all PEs are active and the dataflow is fully utilized.

This is a theoretical architectural capability.

Actual sustained throughput depends on scheduling, pipeline fill/drain, control overhead, memory access, and PE utilization.

---

## 21. Clocked Operation

The systolic array is synchronized using a common clock.

A conceptual cycle sequence is:

    Clock Cycle 0
        ↓
    Initial data enters

    Clock Cycle 1
        ↓
    Data propagates

    Clock Cycle 2
        ↓
    Additional MAC operations

    Clock Cycle 3
        ↓
    Data continues propagating

    ...

    Final cycles
        ↓
    Results become available

The exact latency will be determined by the final RTL implementation and verified through simulation.

---

## 22. Pipeline Fill

At the beginning of a computation, not all PEs contain valid computation data simultaneously.

The array therefore experiences a pipeline fill phase.

Conceptually:

    Cycle 0:

    PE00 active

    Cycle 1:

    PE00 PE01 active

    Cycle 2:

    PE00 PE01 PE02 active

    Cycle 3:

    PE00 PE01 PE02 PE03 active

The actual pattern depends on the selected data scheduling.

Once the pipeline is filled, multiple PEs can operate concurrently.

---

## 23. Steady-State Computation

After the initial pipeline fill, the array reaches a state where multiple Processing Elements perform MAC operations during the same clock cycle.

Conceptually:

    +----+ +----+ +----+ +----+
    |MAC | |MAC | |MAC | |MAC |
    +----+ +----+ +----+ +----+

    +----+ +----+ +----+ +----+
    |MAC | |MAC | |MAC | |MAC |
    +----+ +----+ +----+ +----+

    +----+ +----+ +----+ +----+
    |MAC | |MAC | |MAC | |MAC |
    +----+ +----+ +----+ +----+

    +----+ +----+ +----+ +----+
    |MAC | |MAC | |MAC | |MAC |
    +----+ +----+ +----+ +----+

This is the main source of parallelism in the architecture.

---

## 24. Pipeline Drain

After the final input data has entered the array, the remaining intermediate results continue through the pipeline.

This is called pipeline drain.

The complete operation therefore consists of:

    Pipeline Fill
         ↓
    Steady-State Computation
         ↓
    Pipeline Drain

The final output latency will be determined during array-level simulation.

---

## 25. Reset State

Reset places the Processing Elements into a known initial state.

Conceptually:

    Reset
      |
      v
    PE State Initialization
      |
      +----> Weight State
      |
      +----> Partial-Sum State
      |
      +----> Activation State

The exact reset values and reset behavior are defined by the Verilog RTL.

Reset behavior is verified at the PE level before array integration.

---

## 26. New Computation

A new matrix multiplication operation requires the internal state to be prepared for a new calculation.

The conceptual sequence is:

    Reset / Initialize
          ↓
    Load weights
          ↓
    Initialize partial sums
          ↓
    Start activation stream
          ↓
    Perform MAC operations
          ↓
    Collect output results

The exact control mechanism will be implemented at the array/top level.

---

## 27. Data Validity

The complete accelerator must distinguish valid computation cycles from idle cycles.

Validity considerations include:

- Weight loading
- Activation input
- Partial-sum input
- Output generation

A valid-control mechanism may be incorporated during array-level integration depending on the final architecture.

The PE-level verification establishes the arithmetic and forwarding behavior required by the higher-level design.

---

## 28. Architecture Hierarchy

The intended hardware hierarchy is:

    Top-Level Accelerator
            |
            v
    4×4 Systolic Array
            |
            v
    16 Processing Elements
            |
            v
    MAC Datapath
            |
            +---- Multiplier
            |
            +---- Accumulator
            |
            +---- Weight Register
            |
            +---- Activation Forwarding

This hierarchical structure supports modular development and verification.

---

## 29. Reusability of Processing Element

A major architectural advantage is that the same PE module can be instantiated multiple times.

Instead of creating 16 different PE implementations:

    PE00
    PE01
    PE02
    ...
    PE33

the same Verilog PE module can be reused.

Conceptually:

    PE module
       |
       +---- PE00
       +---- PE01
       +---- PE02
       ...
       +---- PE33

This improves maintainability and reduces duplicated RTL.

---

## 30. Scalability

Although the current target is a 4×4 architecture, the PE-based structure can potentially be scaled.

For example:

    4×4  → 16 PEs

    8×8  → 64 PEs

    16×16 → 256 PEs

Scaling the array increases computational parallelism but also increases:

- Area
- Power
- Routing complexity
- Data movement requirements
- Control complexity
- Timing challenges

Therefore, larger arrays require careful architectural and physical-design analysis.

---

## 31. Hardware Resource Perspective

The target 4×4 array contains:

    16 Processing Elements

Each PE requires hardware for:

    Weight storage
    Multiplier
    Accumulator
    Activation forwarding
    Partial-sum forwarding/storage
    Control logic as required

Therefore, the complete array contains multiple parallel arithmetic datapaths.

The actual synthesized area will depend on:

- Operand widths
- Multiplier implementation
- Accumulator width
- Technology library
- Synthesis constraints
- Optimization settings

---

## 32. Memory and Data Movement

The systolic array requires a mechanism to supply:

- Activations
- Weights
- Initial partial sums

At the current architectural stage, the Processing Element provides the fundamental computation and data movement interface.

Future top-level integration may introduce:

    Input Buffer
    Weight Buffer
    Output Buffer
    Control Logic

The final memory architecture should be designed according to the required workload and target hardware constraints.

---

## 33. Control Architecture

The complete accelerator will require control logic for operations such as:

    Reset
      ↓
    Weight Loading
      ↓
    Computation Start
      ↓
    Activation Scheduling
      ↓
    MAC Processing
      ↓
    Output Collection
      ↓
    Computation Complete

The exact control implementation will be defined after the PE array interconnection is finalized.

---

## 34. Input and Output Concept

The top-level accelerator is expected to have interfaces corresponding to:

### Inputs

- Clock
- Reset
- Activation data
- Weight data
- Control signals

### Outputs

- Computed matrix results
- Status/control indication as required

The exact top-level interface will be finalized during RTL integration.

---

## 35. Architectural Advantages

The proposed architecture provides several advantages.

### Parallelism

Multiple PEs perform computation simultaneously.

### Weight Reuse

Weights remain locally available in PEs.

### Regular Dataflow

The systolic structure provides predictable data movement.

### Modular Design

The PE can be reused throughout the array.

### Scalability

The same basic architecture can be extended to larger arrays.

### Hardware Acceleration

Dedicated MAC hardware can perform matrix operations more efficiently than repeatedly executing the same operations on a general-purpose processor.

---

## 36. Architectural Challenges

The design also introduces several challenges.

### Data Scheduling

Activations and weights must arrive at the correct PEs at the correct cycles.

### Pipeline Latency

The array requires pipeline fill and drain cycles.

### Data Alignment

Operands must be synchronized correctly.

### Partial-Sum Management

Partial sums must be accumulated without corruption or overflow.

### Boundary Handling

Edge PEs have different connectivity.

### Routing

A larger PE array increases interconnect complexity.

### Power

Multiple MAC units switching simultaneously can increase dynamic power.

### Timing

Long datapaths and interconnects can affect maximum operating frequency.

---

## 37. Verification Requirements

Architecture-level verification must confirm:

- Correct PE interconnection
- Correct weight mapping
- Correct activation propagation
- Correct partial-sum propagation
- Correct cycle alignment
- Correct matrix multiplication
- Correct output timing
- Correct reset behavior
- Correct handling of multiple computations

A PE passing its standalone testbench does not guarantee that the complete array will operate correctly.

Therefore, array-level verification is required.

---

## 38. Current Architecture Status

The current development status is:

| Architecture Block | Status |
|--------------------|--------|
| PE architecture | Completed |
| Weight-stationary concept | Completed |
| PE MAC operation | Completed |
| Activation forwarding concept | Completed |
| PE-level verification | Completed |
| 4×4 array architecture | Defined |
| 16-PE integration | In Progress |
| Array data scheduling | In Progress |
| Array-level verification | Planned |
| Matrix multiplication verification | Planned |
| Top-level accelerator | Planned |

---

## 39. Design Boundary

It is important to distinguish between the architecture that has been defined and the hardware that has already been verified.

### Verified

The Processing Element has been implemented in Verilog and verified through simulation.

### Defined

The target 4×4 systolic array architecture has been established.

### Not Yet Fully Verified

The following require array-level implementation and simulation:

- Complete 4×4 PE interconnection
- Full matrix multiplication
- Complete cycle-by-cycle data schedule
- Top-level control
- Full accelerator throughput
- Synthesis performance
- Physical implementation

This distinction is maintained throughout the project documentation.

---

## 40. Summary

The proposed accelerator is a 4×4 weight-stationary systolic array containing 16 Processing Elements.

Each PE performs:

    PSUM_out = PSUM_in + (Activation × Weight)

Weights are stored locally within the PEs, while activation data is propagated through the array.

The architecture provides parallel MAC computation and structured data movement suitable for matrix multiplication.

The Processing Element has already been implemented and functionally verified in Verilog.

The next major implementation step is to integrate the verified PE into the complete 4×4 array, establish the exact cycle-level data schedule, and verify complete matrix multiplication.

The architecture is designed with future synthesis, timing analysis, area analysis, power analysis, and physical implementation in mind.
