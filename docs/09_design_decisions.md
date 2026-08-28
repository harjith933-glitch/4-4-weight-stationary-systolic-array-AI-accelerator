# Design Decisions

## 1. Overview

This document records the major architectural and RTL design decisions made during development of the 4x4 weight-stationary systolic array accelerator.

The purpose of documenting design decisions is to explain not only what was implemented, but also why particular architectural choices were made.

This document should be updated whenever a significant architectural or RTL decision changes.

The current project is being developed using Verilog RTL.


## 2. Project Objective

The objective is to design and verify a 4x4 weight-stationary systolic array for accelerating matrix multiplication.

The primary computation is:

    C = A x B

where:

    A = Activation Matrix
    B = Weight Matrix
    C = Output Matrix

The architecture contains:

    4 x 4 = 16 Processing Elements


## 3. Decision: Use a Systolic Array

### Decision

A systolic array architecture was selected as the main accelerator architecture.

### Reason

Matrix multiplication contains a large number of repeated multiply-and-accumulate operations.

A systolic array maps these operations spatially across multiple processing elements.

Instead of repeatedly sending all data back to a central processing unit, data can move between neighboring PEs.

Conceptually:

    PE ---> PE ---> PE ---> PE
     |      |      |      |
     v      v      v      v
    PE ---> PE ---> PE ---> PE
     |      |      |      |
     v      v      v      v

This provides:

- Parallel computation
- Regular hardware structure
- Local communication
- Data reuse
- Suitable mapping for matrix multiplication


## 4. Decision: Use a 4x4 Array

### Decision

The initial accelerator size was selected as 4x4.

### Reason

A 4x4 array provides a meaningful demonstration of systolic computation while keeping the RTL, simulation, debugging, and verification manageable.

The architecture contains:

    4 x 4 = 16 PEs

A 4x4 array is also large enough to demonstrate:

- Multiple simultaneous MAC operations
- Activation propagation
- Partial-sum accumulation
- Weight-stationary dataflow
- Pipeline behavior
- Matrix-level computation


## 5. Decision: Use Processing Elements

### Decision

The array is constructed using individual Processing Elements.

### Reason

A PE provides a modular hardware building block.

Instead of implementing the complete array as one large block, the design can be constructed from repeated PE instances.

Conceptually:

    +------+------+------+------+
    | PE00 | PE01 | PE02 | PE03 |
    +------+------+------+------+
    | PE10 | PE11 | PE12 | PE13 |
    +------+------+------+------+
    | PE20 | PE21 | PE22 | PE23 |
    +------+------+------+------+
    | PE30 | PE31 | PE32 | PE33 |
    +------+------+------+------+

Advantages include:

- Modular design
- Easier verification
- Easier debugging
- Reusability
- Scalable architecture
- Clear hardware hierarchy


## 6. Decision: Weight-Stationary Dataflow

### Decision

The accelerator uses a weight-stationary dataflow.

### Reason

Weights are loaded into the PEs and retained while activations are streamed through the array.

Conceptually:

    Load Weight
         |
         v
    Store Weight
         |
         v
    Keep Weight Stationary
         |
         v
    Stream Activations
         |
         v
    Perform MAC

This allows the same stored weight to participate in multiple MAC operations without repeatedly loading it from an external source.

The approach also matches the intended PE architecture.


## 7. Decision: Store Weight Inside the PE

### Decision

Each PE contains storage for its weight.

### Reason

The weight must remain available while activation data moves through the array.

Conceptually:

    Weight
      |
      v
    +-------+
    | Weight|
    | Reg   |
    +---+---+
        |
        v
    Multiplier

This makes the weight stationary during the computation phase.


## 8. Decision: Activation Forwarding

### Decision

The PE forwards activation data to the neighboring PE.

### Reason

A systolic architecture depends on controlled movement of data between neighboring processing elements.

Conceptually:

    Activation
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

This reduces the need for separate external activation inputs for every PE.

The PE-level forwarding behavior was included and verified during simulation.


## 9. Decision: Local Data Movement

### Decision

Data movement is performed between neighboring PEs rather than relying entirely on a centralized datapath.

### Reason

Local communication provides a regular structure suitable for systolic architectures.

Advantages include:

- Reduced long-distance communication
- Regular interconnect
- Predictable structure
- Better scalability
- Easier physical implementation planning


## 10. Decision: MAC as the Fundamental PE Operation

### Decision

The Processing Element performs a multiply-and-accumulate operation.

The fundamental operation is:

    PSUM_out =
        PSUM_in + (Activation x Weight)

### Reason

Matrix multiplication consists of dot products.

A dot product requires repeated multiplication followed by accumulation.

Therefore, a MAC operation is the natural computational primitive for the PE.


## 11. Decision: Partial Sum as a PE Input

### Decision

The PE receives a partial sum input.

### Reason

A complete matrix multiplication result requires multiple products to be accumulated.

The PE therefore accepts an existing partial sum and adds the current product.

Conceptually:

    Incoming Partial Sum
             |
             v
    +-------------------+
    |                   |
    |  Activation x     |
    |      Weight       |
    |                   |
    +---------+---------+
              |
              v
           Addition
              |
              v
       Updated Partial Sum


## 12. Decision: Partial Sum Accumulation

### Decision

The PE updates the partial sum using:

    New_PSUM =
        Old_PSUM + (Activation x Weight)

### Reason

For a dot product:

    Cij =
        Ai0B0j
        + Ai1B1j
        + Ai2B2j
        + Ai3B3j

Each PE operation contributes one multiplication term.

The accumulated value eventually becomes the final output.


## 13. Decision: Sequential Storage for Weight

### Decision

The weight is stored in a sequential storage element according to the implemented Verilog RTL.

### Reason

Weight loading occurs as a controlled operation.

The weight must remain stable after loading until the computation requires a new weight.

This supports the weight-stationary behavior.


## 14. Decision: Synchronous Computation

### Decision

The PE operates with clocked sequential behavior.

### Reason

A systolic array is inherently cycle-oriented.

Data moves through the architecture according to clock cycles.

A clocked implementation provides:

- Deterministic timing
- Controlled data movement
- Pipeline behavior
- Easier cycle-level verification


## 15. Decision: Separate RTL and Testbench

### Decision

Design RTL and verification testbench files are maintained separately.

The project uses:

    rtl/

for hardware RTL and:

    tb/

for testbench files.

### Reason

The separation follows standard hardware-development practice.

The RTL represents the design under test.

The testbench represents the verification environment.

This prevents verification code from becoming mixed with synthesizable design logic.


## 16. Decision: PE-First Development

### Decision

The project was developed starting from a single Processing Element before constructing the complete array.

### Reason

The PE is the fundamental computational unit.

Verifying the PE first reduces the complexity of debugging the complete array.

Development flow:

    PE Design
       |
       v
    PE Simulation
       |
       v
    PE Verification
       |
       v
    Multi-PE Integration
       |
       v
    4x4 Array


## 17. Decision: Verify Before Integration

### Decision

The PE was functionally verified before proceeding to complete-array integration.

### Reason

If the basic PE is incorrect, debugging the complete 4x4 array becomes significantly more difficult.

PE-level verification establishes confidence in:

- Weight loading
- Weight storage
- Activation processing
- Activation forwarding
- Multiplication
- Partial-sum accumulation
- MAC operation


## 18. Decision: Use Waveform-Based Debugging

### Decision

Waveforms are used as an important debugging and verification method.

### Reason

Numerical pass/fail information alone may not reveal why an error occurred.

A waveform provides cycle-level visibility into:

- Clock
- Reset
- Weight loading
- Activation
- Partial sum
- Outputs
- Data propagation

Therefore, waveform inspection is useful for identifying timing and data-alignment errors.


## 19. Decision: Use Deterministic Test Vectors

### Decision

Initial verification uses known deterministic input values.

### Reason

Known values make it easier to calculate expected results manually and identify errors.

For example:

    Weight = 5
    Activation = 4
    PSUM_in = 10

Expected:

    4 x 5 = 20

    10 + 20 = 30

Therefore:

    Expected PSUM_out = 30

Such tests are useful during initial RTL development.


## 20. Decision: Independent Reference Calculation

### Decision

The expected matrix multiplication result should be calculated independently from the RTL.

### Reason

The RTL should not be used to generate its own expected result.

The verification flow should be:

    Matrix A
       +
    Matrix B
       |
       v
    Independent Reference
       |
       v
    Expected C

and:

    Matrix A
       +
    Matrix B
       |
       v
    Systolic Array RTL
       |
       v
    Actual C

Then:

    Expected C
       |
       v
    Compare
       ^
       |
    Actual C


## 21. Decision: Verify Numerical Correctness

### Decision

Verification must compare actual RTL outputs against mathematically expected values.

### Reason

A waveform that appears visually reasonable is not sufficient proof of correct computation.

The final matrix result must be numerically correct.

For each output:

    Actual Cij == Expected Cij

must be satisfied.


## 22. Decision: Verify Timing as Well as Value

### Decision

Verification must check both numerical value and cycle timing.

### Reason

A result can be numerically correct but appear at the wrong time.

For example:

    Expected Result:
        Cycle N+1

    Actual Result:
        Cycle N+2

This represents a timing error even though the value itself is correct.


## 23. Decision: Keep Exact RTL as Source of Truth

### Decision

The actual Verilog RTL is treated as the authoritative source for:

- Signal names
- Signal widths
- Reset behavior
- Signedness
- Timing
- Module interfaces
- Implemented functionality

### Reason

Documentation can describe the intended architecture, but the RTL defines what is actually implemented.

Documentation must not claim functionality that is not present in the source code.


## 24. Decision: Use Verilog

### Decision

The project is implemented using Verilog.

### Reason

The project objective is to demonstrate RTL design using a widely used hardware description language.

All RTL and testbench documentation should therefore refer to:

    Verilog

and not SystemVerilog unless SystemVerilog is intentionally introduced in a future revision.


## 25. Decision: Modular Project Directory

### Decision

The project uses separate directories for RTL, testbench, simulation, waveforms, and documentation.

Current structure:

    ~/systolic/
    |
    +-- rtl/
    |
    +-- tb/
    |
    +-- sim/
    |
    +-- wave/
    |
    +-- docs/

### Reason

This provides clear separation of responsibilities and makes the project easier for other engineers to navigate.


## 26. Decision: Documentation as Part of the Project

### Decision

Documentation is maintained as part of the hardware project rather than being treated as an afterthought.

### Reason

An industry-standard hardware project should explain:

- Architecture
- Design decisions
- RTL
- Verification
- Simulation
- Mathematical mapping
- Results
- Limitations
- Future work

This allows another engineer to understand and reproduce the work.


## 27. Decision: Do Not Claim Unverified Features

### Decision

Only completed and verified functionality should be marked as completed.

### Reason

Professional engineering documentation must distinguish between:

    Implemented

    Simulated

    Verified

    Planned

For example:

    PE MAC operation:
        Implemented and verified

while:

    Complete 4x4 matrix multiplication:
        Planned / under integration

until complete-array simulation has actually passed.


## 28. Decision: Progressive Verification

### Decision

Verification is performed progressively.

The intended verification hierarchy is:

    Level 1:
        PE

    Level 2:
        Two-PE Integration

    Level 3:
        Four-PE Row

    Level 4:
        4x4 PE Array

    Level 5:
        Matrix Multiplication

    Level 6:
        Stress and Boundary Testing


## 29. Decision: Identity Matrix as a Sanity Test

### Decision

An identity matrix is included as a planned matrix-level verification test.

### Reason

For:

    C = A x I

the expected result is:

    C = A

This makes it easy to detect errors in:

- Weight placement
- Data ordering
- Activation propagation
- Output mapping


## 30. Decision: Zero Matrix as a Sanity Test

### Decision

A zero matrix is included as a planned verification test.

### Reason

If one matrix is zero:

    A x 0 = 0

This provides a simple check for unwanted non-zero outputs and accumulation errors.


## 31. Decision: Boundary Testing

### Decision

Boundary values should be included in later verification.

Examples:

    Minimum Supported Value

    Maximum Supported Value

    Zero

    One

### Reason

Boundary testing can expose:

- Overflow
- Incorrect signedness
- Width problems
- Incorrect arithmetic
- Reset/state issues


## 32. Decision: Consider Arithmetic Width Carefully

### Decision

Operand and accumulator widths must be selected based on the numerical range required by the design.

### Reason

A multiplication can require more bits than either operand.

Accumulating multiple products can require additional bits.

For example:

    Product = Activation x Weight

and:

    PSUM =
        Product0
        + Product1
        + Product2
        + Product3

Therefore, the partial-sum width must be sufficient for the expected range.

The exact implemented widths must be taken from the Verilog RTL.


## 33. Decision: Consider Signedness Explicitly

### Decision

The project must explicitly define whether arithmetic operands are signed or unsigned.

### Reason

Verilog arithmetic behavior depends on operand declarations and expression sizing.

The same binary bit pattern can represent different numerical values depending on signedness.

Therefore, signedness must be documented and verified against the actual RTL.


## 34. Decision: Locality of Communication

### Decision

The array uses neighboring PE communication as the fundamental data movement concept.

### Reason

A regular local interconnect is more suitable for a systolic architecture than a centralized communication structure.

The conceptual structure is:

    PE00 ---> PE01 ---> PE02 ---> PE03
      |        |        |        |
      v        v        v        v
    PE10 ---> PE11 ---> PE12 ---> PE13
      |        |        |        |
      v        v        v        v
    PE20 ---> PE21 ---> PE22 ---> PE23
      |        |        |        |
      v        v        v        v
    PE30 ---> PE31 ---> PE32 ---> PE33


## 35. Decision: Pipeline-Oriented Operation

### Decision

The array is designed around cycle-by-cycle data movement.

### Reason

The systolic architecture derives its performance from overlapping operations across multiple PEs.

Conceptually:

    Cycle 1:
        PE00 works

    Cycle 2:
        PE00 + PE01 work

    Cycle 3:
        PE00 + PE01 + PE02 work

    Cycle 4:
        PE00 + PE01 + PE02 + PE03 work

The exact timing will be established from the final RTL implementation and simulation.


## 36. Decision: Separate Fill, Steady-State, and Drain Phases

### Decision

The complete array will be analyzed in three conceptual phases:

    Fill Phase

    Steady-State Phase

    Drain Phase

### Reason

Systolic arrays require time for data to enter and propagate through the array.

The first valid result is therefore not necessarily available immediately after the first input.

This classification will help analyze array latency and throughput.


## 37. Decision: Reusable PE Architecture

### Decision

The PE is intended to be reusable across all array positions.

### Reason

The 4x4 array should be built by instantiating the same PE architecture rather than creating 16 unrelated implementations.

Conceptually:

    PE Module
       |
       +---- PE00
       +---- PE01
       +---- PE02
       +---- PE03
       |
       +---- PE10
       +---- PE11
       ...
       |
       +---- PE33

This improves consistency and reduces design duplication.


## 38. Decision: Hierarchical Design

### Decision

The final design should use a hierarchical structure.

Conceptually:

    Top-Level Systolic Array
              |
              +----------------+
              |                |
              v                v
        Control/Interface    PE Array
                                  |
                                  v
                           Multiple PE Instances

### Reason

Hierarchy makes the design easier to understand, verify, synthesize, and maintain.


## 39. Decision: Verify Integration Separately

### Decision

Each integration stage should have its own verification focus.

### Reason

Integration introduces new problems that cannot be detected at the individual PE level.

Examples include:

- Incorrect PE connections
- Incorrect activation routing
- Incorrect partial-sum routing
- Incorrect weight mapping
- Cycle misalignment
- Output ordering errors


## 40. Decision: Maintain a Clear Project Status

### Decision

The project documentation will clearly distinguish completed, ongoing, and future work.

Current high-level status:

    PE RTL
        |
        v
    Completed

    PE Testbench
        |
        v
    Completed

    PE Simulation
        |
        v
    Passed

    Multi-PE Integration
        |
        v
    Next Stage

    4x4 Matrix Multiplication
        |
        v
    Future Verification Stage


## 41. Design Trade-Off: Array Size

A larger array provides more parallelism.

However, increasing array size also increases:

- Area
- Routing
- Power
- Verification complexity
- Control complexity

Therefore, 4x4 was selected as the initial implementation size.


## 42. Design Trade-Off: Weight Stationary vs Other Dataflows

Matrix multiplication accelerators can use different dataflows.

Examples include:

    Weight Stationary
    Output Stationary
    Input Stationary

The current project selects weight stationary.

The primary reason is that keeping weights in the PE simplifies the intended reuse model and matches the project's architectural objective.


## 43. Design Trade-Off: Modular PE vs Monolithic Array

A monolithic implementation could place all arithmetic directly inside one large module.

However, a modular PE-based architecture was selected because it provides:

- Better hierarchy
- Easier debugging
- Better reuse
- Easier verification
- Easier scalability


## 44. Design Trade-Off: Early Optimization

The project prioritizes functional correctness before aggressive optimization.

The development sequence is:

    Correctness
        |
        v
    Integration
        |
        v
    Verification
        |
        v
    Synthesis
        |
        v
    Timing Analysis
        |
        v
    Optimization

This prevents premature optimization from making debugging more difficult.


## 45. Design Trade-Off: Simulation Before Synthesis

Functional simulation is performed before moving to synthesis.

### Reason

Simulation allows logical errors to be found before hardware implementation analysis.

The flow is:

    RTL
      |
      v
    Functional Simulation
      |
      v
    Verification
      |
      v
    Synthesis
      |
      v
    Timing / Area / Power Analysis


## 46. Future Design Decisions

The following decisions will be finalized during later implementation stages:

- Complete 4x4 array interface
- Exact matrix input protocol
- Exact weight loading sequence
- Exact activation scheduling
- Output collection mechanism
- Control FSM
- Valid/ready behavior if required
- Arithmetic widths
- Signed/unsigned support
- Synthesis target
- Clock constraints
- Timing optimization
- Physical design considerations


## 47. Design Decision Change Policy

If an architectural decision changes, the documentation should be updated.

For example:

    Previous:
        Weight-stationary

    New:
        Another dataflow

The reason for the change should be documented.

The corresponding RTL and testbench should also be updated.

This keeps the repository internally consistent.


## 48. Current Architectural Summary

The current architecture can be summarized as:

    Matrix Multiplication
            |
            v
       4x4 Systolic Array
            |
            v
       16 Processing Elements
            |
            v
       Weight Stationary
            |
            v
       Activation Streaming
            |
            v
       Local Data Movement
            |
            v
       MAC Computation
            |
            v
       Partial-Sum Accumulation
            |
            v
       Matrix Output


## 49. Current Verified Design Decisions

The following architectural concepts have already been demonstrated at PE level:

    PE-based architecture
        |
        v
    Weight loading
        |
        v
    Weight storage
        |
        v
    Activation processing
        |
        v
    Activation forwarding
        |
        v
    Multiplication
        |
        v
    Partial-sum accumulation
        |
        v
    MAC operation


## 50. Summary

The 4x4 systolic array is based on a modular Processing Element architecture.

The major design decisions are:

    1. Use a systolic array.
    2. Use a 4x4 PE organization.
    3. Use modular Processing Elements.
    4. Use weight-stationary dataflow.
    5. Store weights locally inside PEs.
    6. Stream and forward activation data.
    7. Perform MAC operations inside each PE.
    8. Accumulate partial sums.
    9. Use clocked sequential behavior.
    10. Develop and verify the PE before array integration.
    11. Separate RTL and testbench files.
    12. Use waveform-based debugging.
    13. Verify numerical results independently.
    14. Verify both data correctness and timing.
    15. Maintain Verilog as the current HDL.
    16. Document only implemented and verified functionality.

These decisions provide the architectural foundation for the complete 4x4 weight-stationary systolic array accelerator.
