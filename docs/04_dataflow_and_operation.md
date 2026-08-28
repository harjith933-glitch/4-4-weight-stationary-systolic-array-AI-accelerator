# Dataflow and Operation

## 1. Overview

The 4×4 systolic array uses a weight-stationary dataflow architecture.

The main objective of the dataflow is to keep weights locally stored inside the Processing Elements while activation data propagates through the array.

Each Processing Element performs a multiply-accumulate operation:

    PSUM_out = PSUM_in + (Activation × Weight)

The combination of local weight storage, activation propagation, and partial-sum accumulation enables the array to perform matrix multiplication using parallel hardware.

The dataflow is synchronized with the system clock, allowing data to move through the Processing Elements in a predictable sequence.

---

## 2. Matrix Multiplication

The target computation is matrix multiplication:

    C = A × B

For the 4×4 architecture:

    A(4×4) × B(4×4) = C(4×4)

Each output element is calculated using four multiplication operations followed by accumulation.

For example:

    C[0][0] =
        A[0][0] × B[0][0]
      + A[0][1] × B[1][0]
      + A[0][2] × B[2][0]
      + A[0][3] × B[3][0]

Similarly:

    C[i][j] =
        A[i][0] × B[0][j]
      + A[i][1] × B[1][j]
      + A[i][2] × B[2][j]
      + A[i][3] × B[3][j]

The systolic array distributes these operations across multiple Processing Elements.

---

## 3. Weight-Stationary Dataflow

The selected dataflow is weight-stationary.

The basic principle is:

    Weight → Stored inside PE
    Activation → Propagates through PE
    Partial Sum → Accumulated by PE

Once a weight is loaded into a PE, it remains available for computation while activation values arrive.

Conceptually:

                   Weight
                      |
                      v
                +-----------+
                |           |
    Activation->|    PE     |-> Activation
                |           |
    PSUM_in --->|           |
                +-----+-----+
                      |
                      v
                  PSUM_out

The stored weight is therefore reused rather than being continuously transferred from an external source.

---

## 4. Why Weight-Stationary?

Weight-stationary dataflow provides several useful characteristics for an AI accelerator.

### Weight reuse

The same stored weight can be used with multiple activation values.

### Reduced data movement

Weights do not need to travel between multiple processing stages during every computation.

### Local computation

The multiplication occurs directly inside the PE.

### Regular communication

Activation data follows a predictable path through neighboring PEs.

### Hardware scalability

The same PE structure can be replicated to construct larger arrays.

---

## 5. Activation Movement

Activation data moves horizontally across each row of the systolic array.

For example:

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

Conceptually, the actual horizontal data path is:

    PE00 → PE01 → PE02 → PE03

The activation output of one PE becomes the activation input of the next PE.

This allows the same activation value to interact with multiple stored weights as it travels through the array.

---

## 6. Activation Propagation

The activation propagation mechanism can be represented as:

    Cycle N:

    A0 → PE00

    Cycle N+1:

    A0 → PE01

    Cycle N+2:

    A0 → PE02

    Cycle N+3:

    A0 → PE03

The exact cycle relationship depends on the register implementation of the final array.

The important architectural principle is that activation data propagates from one PE to the next in a synchronized manner.

---

## 7. Partial-Sum Movement

Partial sums represent intermediate results of the matrix multiplication.

The fundamental operation is:

    PSUM_out = PSUM_in + (Activation × Weight)

A simplified flow is:

    PSUM_in
       |
       v
    +------+
    |  PE  |
    +------+
       |
       v
    PSUM_out

The updated partial sum can then be used by the next computational stage according to the final array dataflow.

The partial-sum path is therefore essential for accumulating multiple multiplication results into a final matrix element.

---

## 8. MAC Operation

The MAC operation consists of two arithmetic operations:

    1. Multiplication
    2. Accumulation

First:

    Product = Activation × Weight

Then:

    PSUM_out = PSUM_in + Product

Combining both:

    PSUM_out = PSUM_in + (Activation × Weight)

This operation is performed by every active Processing Element.

---

## 9. Example of MAC Accumulation

Assume:

    Weight = 5

and activation values arrive sequentially:

    A0 = 2
    A1 = 3
    A2 = 4
    A3 = 6

Starting with:

    PSUM = 0

First operation:

    PSUM = 0 + (2 × 5)
         = 10

Second operation:

    PSUM = 10 + (3 × 5)
         = 25

Third operation:

    PSUM = 25 + (4 × 5)
         = 45

Fourth operation:

    PSUM = 45 + (6 × 5)
         = 75

Final result:

    PSUM = 75

This demonstrates how repeated MAC operations accumulate a result over multiple clock cycles.

---

## 10. Example Matrix Computation

Consider:

    A =

        1  2  3  4
        5  6  7  8
        9 10 11 12
       13 14 15 16

and:

    B =

       16 15 14 13
       12 11 10  9
        8  7  6  5
        4  3  2  1

The first output element is:

    C[0][0] =
        (1 × 16)
      + (2 × 12)
      + (3 × 8)
      + (4 × 4)

Therefore:

    C[0][0] =
        16 + 24 + 24 + 16

    C[0][0] = 80

The same multiply-and-accumulate principle is applied to every output element.

---

## 11. Data Reuse

Data reuse is an important feature of the systolic architecture.

For weights:

    Weight
       |
       v
    Stored in PE
       |
       v
    Reused during computation

For activations:

    Activation
        |
        v
       PE
        |
        v
      Next PE
        |
        v
      Next PE

This structured movement avoids treating every multiplication as an independent operation requiring completely new data transfers.

---

## 12. Systolic Operation

The term "systolic" refers to the regular, synchronized movement of data through a network of processing elements.

Each PE performs computation and communicates with neighboring PEs.

Conceptually:

    +------+     +------+     +------+     +------+
    | PE00 | --> | PE01 | --> | PE02 | --> | PE03 |
    +------+     +------+     +------+     +------+
       |            |            |            |
       v            v            v            v
    +------+     +------+     +------+     +------+
    | PE10 | --> | PE11 | --> | PE12 | --> | PE13 |
    +------+     +------+     +------+     +------+
       |            |            |            |
       v            v            v            v
    +------+     +------+     +------+     +------+
    | PE20 | --> | PE21 | --> | PE22 | --> | PE23 |
    +------+     +------+     +------+     +------+
       |            |            |            |
       v            v            v            v
    +------+     +------+     +------+     +------+
    | PE30 | --> | PE31 | --> | PE32 | --> | PE33 |
    +------+     +------+     +------+     +------+

The exact connection structure will be finalized during RTL array integration.

---

## 13. Clock-by-Clock Concept

The systolic array operates over multiple clock cycles.

A simplified conceptual sequence is:

    Cycle 0
    -------
    Initial data begins entering the array.

    Cycle 1
    -------
    Data propagates to neighboring Processing Elements.

    Cycle 2
    -------
    Additional MAC operations occur.

    Cycle 3
    -------
    Data continues propagating.

    Cycle 4
    -------
    Additional accumulated results are produced.

    ...

    Final cycles
    ------------
    Required matrix results become available.

The exact latency depends on:

- PE register structure
- Array dimensions
- Input scheduling
- Weight-loading schedule
- Output collection method

Therefore, the exact cycle schedule will be established during array-level verification.

---

## 14. Pipeline Filling

At the beginning of operation, the complete array is not immediately processing useful data at every PE.

The pipeline gradually fills as data propagates through the array.

Conceptually:

    Initial state:

    PE  PE  PE  PE
    PE  PE  PE  PE
    PE  PE  PE  PE
    PE  PE  PE  PE

    Data enters:

    A → PE → PE → PE → PE

As more clock cycles occur, additional data occupies the array.

Eventually, the array reaches a steady operating state in which multiple PEs are performing MAC operations simultaneously.

---

## 15. Pipeline Drain

After all required input data has entered the array, the remaining intermediate results continue propagating until the final outputs are produced.

This is known as pipeline draining.

The complete computation therefore consists of:

    Pipeline Fill
         ↓
    Steady-State Computation
         ↓
    Pipeline Drain

The latency associated with these phases will be measured during array-level verification.

---

## 16. PE-Level Dataflow

At the Processing Element level:

    Weight
       |
       v
    Weight Register
       |
       v
    +----------------------+
    |                      |
    |         PE           |
    |                      |
    | Activation × Weight  |
    |         +            |
    |       PSUM_in        |
    |                      |
    +----------+-----------+
               |
               v
           PSUM_out

And:

    Activation_in
          |
          v
         PE
          |
          v
    Activation_out

The PE therefore performs computation and forwarding simultaneously.

---

## 17. Row-Level Dataflow

A row of the systolic array can be viewed as:

    +------+     +------+     +------+     +------+
    | PE00 | --> | PE01 | --> | PE02 | --> | PE03 |
    +------+     +------+     +------+     +------+

Activation enters from the left and propagates toward the right.

Each PE uses its locally stored weight to perform a MAC operation.

The row therefore provides a chain of computational stages.

---

## 18. Array-Level Dataflow

The target 4×4 array consists of four rows and four columns.

Conceptually:

                     Activation Flow →

              PE00 → PE01 → PE02 → PE03
                ↓      ↓      ↓      ↓
              PE10 → PE11 → PE12 → PE13
                ↓      ↓      ↓      ↓
              PE20 → PE21 → PE22 → PE23
                ↓      ↓      ↓      ↓
              PE30 → PE31 → PE32 → PE33

The exact partial-sum and weight connection strategy is determined by the final RTL integration.

The important architectural principle is that each PE communicates with neighboring processing stages rather than requiring global communication.

---

## 19. Weight Loading Sequence

Before normal computation, the required weights must be loaded into the Processing Elements.

The conceptual sequence is:

    Reset
      ↓
    Weight input applied
      ↓
    Weight-load control asserted
      ↓
    Weight stored in PE
      ↓
    Weight-load control deasserted
      ↓
    Stored weight retained
      ↓
    Normal MAC operation

Once the weight is stored, it remains available for the subsequent computation.

---

## 20. Activation and Weight Interaction

During normal operation:

    Activation
        |
        v
    +-------------+
    |             |
    |     PE      |
    |             |
    |  Weight     |
    |     ↓       |
    | Activation  |
    |     ×       |
    |     ↓       |
    |   Product   |
    |     +       |
    |   PSUM_in   |
    +------+------+
           |
           v
       PSUM_out

The activation is therefore combined with the locally stored weight.

---

## 21. Partial-Sum Example

Suppose a PE receives:

    PSUM_in = 20
    Activation = 4
    Weight = 6

Then:

    Product = 4 × 6
            = 24

Therefore:

    PSUM_out = 20 + 24
             = 44

The updated result can then contribute to later computation.

---

## 22. Mapping to Matrix Multiplication

For a matrix multiplication:

    C = A × B

each output element requires a dot product.

For example:

    C[1][2] =
        A[1][0] × B[0][2]
      + A[1][1] × B[1][2]
      + A[1][2] × B[2][2]
      + A[1][3] × B[3][2]

The systolic array distributes the required operations among the PEs.

Each PE contributes to the overall computation by performing MAC operations according to the scheduled activation and weight data.

---

## 23. Parallelism

The target array contains:

    4 × 4 = 16 PEs

Each PE contains the computational capability required for a MAC operation.

Therefore, the architecture provides:

    16 parallel MAC datapaths

when all PEs are active.

This spatial parallelism is one of the primary advantages of the systolic architecture.

---

## 24. Throughput Concept

If all 16 PEs perform one MAC operation per clock cycle, the theoretical computational rate is:

    MACs per cycle = 16

For a clock frequency of F Hz:

    Theoretical MAC throughput = 16 × F MAC/s

For example, at 100 MHz:

    16 × 100,000,000
    = 1,600,000,000 MAC/s
    = 1.6 GMAC/s

This is a theoretical architectural calculation and is not a measured performance result.

Actual throughput depends on:

- PE utilization
- Input scheduling
- Pipeline efficiency
- Memory bandwidth
- Control overhead
- Clock frequency

---

## 25. Data Movement and Hardware Efficiency

A major goal of the architecture is to reduce unnecessary data movement.

The selected strategy is:

    Keep weights local
           +
    Move activations
           +
    Perform computation locally

This can reduce the amount of repeated weight transfer and allows the computational hardware to remain active while data propagates through the array.

---

## 26. Handling of Initial Partial Sums

For a new matrix multiplication operation, the initial partial sum is typically initialized to zero.

Therefore:

    PSUM_initial = 0

The first MAC operation becomes:

    PSUM = 0 + (Activation × Weight)

Subsequent operations accumulate additional products.

For example:

    PSUM0 = 0

    PSUM1 = PSUM0 + (A0 × W)

    PSUM2 = PSUM1 + (A1 × W)

    PSUM3 = PSUM2 + (A2 × W)

    PSUM4 = PSUM3 + (A3 × W)

The final value represents the accumulated dot-product result.

---

## 27. New Computation Initialization

Before starting a new matrix multiplication operation, the required state must be initialized.

The general sequence is:

    Reset / Initialization
            ↓
    Load weights
            ↓
    Initialize partial sums
            ↓
    Start activation stream
            ↓
    Perform MAC operations
            ↓
    Collect results
            ↓
    Complete computation

The exact control sequence will be implemented during top-level integration.

---

## 28. Data Validity

The complete accelerator will require a mechanism to distinguish valid computation data from idle cycles.

During array integration, valid-data handling will be considered for:

- Activation inputs
- Weight loading
- Partial sums
- Output results

The PE-level implementation is the foundation for this behavior, while array-level validity control will be defined during integration.

---

## 29. Boundary Processing Elements

PEs located at the boundaries of the array have fewer neighboring PEs than internal elements.

For example:

    PE00

is located at the top-left corner, while:

    PE33

is located at the bottom-right corner.

Boundary PEs therefore require appropriate handling for inputs and outputs that do not have neighboring PEs.

The final array RTL will explicitly define these boundary connections.

---

## 30. Internal vs Boundary PEs

An internal PE generally communicates with neighboring PEs on multiple sides.

A boundary PE may have:

- External input
- One neighboring PE
- External output

depending on its location and the selected dataflow.

This must be considered when constructing the complete 4×4 array.

---

## 31. Dataflow Verification Strategy

The dataflow will be verified progressively.

### Step 1 — PE verification

Verify:

    Weight loading
    Activation forwarding
    MAC operation
    Partial-sum behavior

Completed.

### Step 2 — Two-PE or small-chain verification

Verify:

    PE-to-PE activation propagation
    Timing relationship
    Weight retention
    Partial-sum behavior

### Step 3 — 4×4 array verification

Verify:

    All 16 PEs
    Activation propagation
    Weight mapping
    Partial-sum accumulation
    Output correctness

### Step 4 — Matrix-level verification

Provide complete matrices and compare the RTL output against a software reference model.

---

## 32. Software Reference Model

For array-level verification, the expected matrix multiplication result can be calculated independently using a software reference model.

The reference operation is:

    C[i][j] = Σ(A[i][k] × B[k][j])

The RTL result can then be compared against the reference result.

For every output:

    RTL_C[i][j] == Reference_C[i][j]

A complete matrix multiplication test passes only when all required output elements match the reference result.

---

## 33. Functional Correctness

Functional correctness requires verification of both computation and data movement.

The verification must confirm:

- Correct weights are stored
- Correct activations reach the intended PE
- Correct multiplication occurs
- Correct partial sums are accumulated
- Correct outputs are produced
- No data is lost during propagation
- No unintended data overwrites occur

This is especially important because a systolic array can produce incorrect results even when the individual PE MAC operation is correct if the data is scheduled incorrectly.

---

## 34. Current Dataflow Implementation Status

The current implementation status is:

| Dataflow Feature | Status |
|------------------|--------|
| Weight-stationary architecture | Completed |
| PE weight loading | Completed |
| PE weight storage | Completed |
| Activation input | Completed |
| Activation forwarding | Completed |
| MAC computation | Completed |
| Partial-sum accumulation | Completed |
| PE-level dataflow verification | Completed |
| Multi-PE dataflow | In Progress |
| 4×4 array dataflow | Planned/In Progress |
| Array-level scheduling | Planned |
| Matrix multiplication verification | Planned |
| Complete accelerator dataflow | Planned |

---

## 35. Important Architectural Distinction

The project currently has two different levels of implementation:

### Verified level

The Processing Element has been implemented and verified.

At this level:

    Weight
       ↓
    PE
       ↓
    MAC
       ↓
    PSUM

and:

    Activation_in → PE → Activation_out

have been verified.

### Target system level

The complete 4×4 systolic array is the target architecture.

This requires:

    16 PEs
       ↓
    Correct PE interconnection
       ↓
    Correct weight mapping
       ↓
    Correct activation scheduling
       ↓
    Correct partial-sum handling
       ↓
    Matrix multiplication verification

The target system should not be considered verified until these stages are completed.

---

## 36. Future Dataflow Improvements

After functional correctness is established, the architecture can be evaluated for:

- Higher PE utilization
- Reduced pipeline bubbles
- Improved data reuse
- Reduced memory traffic
- Improved clock frequency
- Lower switching activity
- Better area efficiency
- Improved throughput

These optimizations will be considered only after establishing a correct baseline implementation.

---

## 37. Summary

The accelerator uses a weight-stationary systolic dataflow.

The fundamental operation is:

    PSUM_out = PSUM_in + (Activation × Weight)

Weights are stored locally within the Processing Elements while activation data propagates through the array.

The 4×4 target architecture contains 16 Processing Elements, providing 16 parallel MAC datapaths when fully utilized.

The systolic organization enables computation and data movement to occur concurrently under synchronous clock control.

The Processing Element implementation and its fundamental dataflow have already been verified through simulation.

The next major stage is to connect multiple verified Processing Elements, establish the complete 4×4 dataflow, verify cycle-by-cycle operation, and demonstrate correct matrix multiplication at the array level.
