# Dataflow and Operation

## 1. Overview

The 4×4 systolic array uses a weight-stationary dataflow architecture.

The purpose of the dataflow is to determine how:

- Weights
- Activations
- Partial sums

are stored, moved, reused, and processed inside the accelerator.

The fundamental computation performed by each Processing Element (PE) is:

    PSUM_out = PSUM_in + (Activation × Weight)

The weight is stored locally in the PE, while activation data is propagated through the array.

The Processing Elements operate synchronously with the system clock.

---

## 2. Weight-Stationary Dataflow

In a weight-stationary architecture, the weight remains stationary inside the Processing Element during computation.

The basic concept is:

    Weight
       |
       v
    PE Weight Register
       |
       v
    Stored locally
       |
       +----------------+
       |                |
       v                v
    MAC Operation 1   MAC Operation 2
       |                |
       +----------------+
                |
                v
             Result

The weight does not need to be fetched again for every activation.

This allows the same weight to be reused while different activation values pass through the PE.

---

## 3. Why Weight-Stationary?

Weight-stationary dataflow is useful because weights can be reused locally.

Without local weight storage, a weight could need to be transferred repeatedly:

    Memory
      |
      v
     PE
      |
      v
    Memory
      |
      v
     PE
      |
      v
    Memory

This increases data movement.

With weight-stationary operation:

    Memory
      |
      v
    PE Weight Register
      |
      v
    Weight reused locally

The architecture therefore emphasizes computation near stored weights.

---

## 4. Three Main Data Types

The systolic array operates on three major types of data.

### Weight

The weight is loaded into and retained by a PE.

### Activation

The activation enters the PE and participates in the multiplication.

It is then forwarded toward the neighboring PE.

### Partial Sum

The partial sum represents the accumulated result of previous MAC operations.

The PE adds the current multiplication result to the incoming partial sum.

---

## 5. Fundamental PE Operation

The PE performs:

    Product = Activation × Weight

followed by:

    PSUM_out = PSUM_in + Product

Therefore:

    PSUM_out = PSUM_in + (Activation × Weight)

This operation forms the basic computational unit of the complete accelerator.

---

## 6. Dataflow Through One PE

The basic dataflow is:

    Weight Input
        |
        v
    Weight Register
        |
        v
    Stored Weight
        |
        +----------------+
                         |
    Activation ----------+
        |
        v
    Multiplier
        |
        v
      Product
        |
        v
       Adder <----- PSUM_in
        |
        v
     PSUM_out

At the same time:

    Activation_in
         |
         v
    Activation Forwarding
         |
         v
    Activation_out

---

## 7. Activation Propagation

The activation is forwarded from one PE to the next.

For a row:

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

The output of one PE becomes the input of the next PE.

This creates the horizontal data-propagation path.

---

## 8. Four-by-Four Array Dataflow

The target array is:

    +------+    +------+    +------+    +------+
    | PE00 | -> | PE01 | -> | PE02 | -> | PE03 |
    +------+    +------+    +------+    +------+
       |           |           |           |
       v           v           v           v
    +------+    +------+    +------+    +------+
    | PE10 | -> | PE11 | -> | PE12 | -> | PE13 |
    +------+    +------+    +------+    +------+
       |           |           |           |
       v           v           v           v
    +------+    +------+    +------+    +------+
    | PE20 | -> | PE21 | -> | PE22 | -> | PE23 |
    +------+    +------+    +------+    +------+
       |           |           |           |
       v           v           v           v
    +------+    +------+    +------+    +------+
    | PE30 | -> | PE31 | -> | PE32 | -> | PE33 |
    +------+    +------+    +------+    +------+

The arrows represent the conceptual propagation paths.

The exact final array wiring is established in the array-level Verilog RTL.

---

## 9. Systolic Operation

The word "systolic" refers to the regular, synchronized movement of data through the processing elements.

The PEs operate according to the system clock.

Conceptually:

    Clock
      |
      +---- PE00
      |
      +---- PE01
      |
      +---- PE02
      |
      +---- ...
      |
      +---- PE33

Data advances through the array in a controlled sequence.

This allows computation to occur concurrently across multiple PEs.

---

## 10. Clock-by-Clock Concept

A simplified operation can be represented as:

    Cycle 0
        |
        v
    Initial activation enters array

    Cycle 1
        |
        v
    Activation propagates

    Cycle 2
        |
        v
    Multiple PEs perform MAC operations

    Cycle 3
        |
        v
    Additional data propagates

    ...

    Final cycles
        |
        v
    Results become available

The exact number of cycles depends on the final array scheduling and RTL implementation.

---

## 11. Pipeline Fill

At the beginning of a computation, the entire array is not immediately active with valid data.

The first input must propagate through the array.

This creates a pipeline fill period.

Conceptually:

    Initial state:

    +----+ +----+ +----+ +----+
    |    | |    | |    | |    |
    +----+ +----+ +----+ +----+

    After data begins entering:

    +----+ +----+ +----+ +----+
    |MAC | |    | |    | |    |
    +----+ +----+ +----+ +----+

    Later:

    +----+ +----+ +----+ +----+
    |MAC | |MAC | |    | |    |
    +----+ +----+ +----+ +----+

    Later:

    +----+ +----+ +----+ +----+
    |MAC | |MAC | |MAC | |MAC |
    +----+ +----+ +----+ +----+

The exact activation pattern depends on the selected scheduling scheme.

---

## 12. Steady-State Operation

After the pipeline is filled, multiple Processing Elements perform MAC operations during the same clock cycle.

For example:

    Cycle N:

    PE00 -> MAC
    PE01 -> MAC
    PE02 -> MAC
    PE03 -> MAC

    PE10 -> MAC
    PE11 -> MAC
    PE12 -> MAC
    PE13 -> MAC

    PE20 -> MAC
    PE21 -> MAC
    PE22 -> MAC
    PE23 -> MAC

    PE30 -> MAC
    PE31 -> MAC
    PE32 -> MAC
    PE33 -> MAC

If all 16 PEs are active:

    16 MAC operations

can theoretically occur in one cycle.

This is the main computational parallelism of the architecture.

---

## 13. Pipeline Drain

After the final input values are supplied, the remaining data continues propagating through the array.

This is called pipeline drain.

Therefore, the complete operation contains:

    Pipeline Fill
          ↓
    Steady-State Computation
          ↓
    Pipeline Drain

The output results must be collected only after the corresponding computations have completed.

---

## 14. Weight Loading Phase

Before normal computation, the required weights must be loaded.

Conceptually:

    Weight Source
         |
         v
    Weight Input
         |
         v
    PE Weight Register
         |
         v
    Weight Stored

Once the weight is stored:

    Weight Load = inactive

and the PE can use the stored weight during normal computation.

---

## 15. Weight Retention

After loading:

    Stored Weight = Loaded Weight

The stored weight remains available for subsequent MAC operations.

For example:

    Weight = 5

Then:

    Activation = 2
    Product = 2 × 5

    Activation = 4
    Product = 4 × 5

    Activation = 6
    Product = 6 × 5

The same weight is reused.

This is the defining characteristic of weight-stationary operation.

---

## 16. Activation Movement

Activations are not required to remain stationary.

Instead, they move through the PE network.

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

The activation can therefore interact with multiple locally stored weights.

---

## 17. Partial-Sum Operation

The partial sum is updated during each MAC operation.

The sequence is:

    PSUM_0 = Initial Partial Sum

    PSUM_1 =
        PSUM_0 + (Activation_0 × Weight)

    PSUM_2 =
        PSUM_1 + (Activation_1 × Weight)

    PSUM_3 =
        PSUM_2 + (Activation_2 × Weight)

and so on.

For a four-term dot product:

    PSUM_final =
        PSUM_0
        + A0×W0
        + A1×W1
        + A2×W2
        + A3×W3

If:

    PSUM_0 = 0

then:

    PSUM_final =
        A0×W0
        + A1×W1
        + A2×W2
        + A3×W3

---

## 18. Dot Product Interpretation

A matrix multiplication can be viewed as multiple dot products.

For example:

    C[0][0] =
        A[0][0]×B[0][0]
      + A[0][1]×B[1][0]
      + A[0][2]×B[2][0]
      + A[0][3]×B[3][0]

The four multiplication results are accumulated.

The PE provides the MAC primitive required to perform these calculations.

---

## 19. Example Dataflow

Consider a simplified PE with:

    Weight = 3

and activations:

    A0 = 2
    A1 = 4
    A2 = 5

Starting with:

    PSUM = 0

First operation:

    PSUM = 0 + (2 × 3)
         = 6

Second operation:

    PSUM = 6 + (4 × 3)
         = 18

Third operation:

    PSUM = 18 + (5 × 3)
         = 33

Final result:

    PSUM = 33

The weight remained stationary while the activation values changed.

---

## 20. Example of Activation Forwarding

Assume:

    Activation = 7

The conceptual movement is:

    Cycle N:

    PE00 receives 7

    Cycle N+1:

    PE01 receives 7

    Cycle N+2:

    PE02 receives 7

    Cycle N+3:

    PE03 receives 7

This illustrates the basic concept of systolic activation propagation.

The actual cycle timing must match the registered/unregistered implementation in the final Verilog array.

---

## 21. Data Reuse

The architecture attempts to maximize data reuse.

### Weight Reuse

Weights are stored locally and reused.

### Activation Reuse

An activation can propagate through multiple PEs.

### Partial-Sum Reuse

An accumulated partial sum can participate in subsequent MAC operations.

The combination reduces the need for unnecessary external data transfers.

---

## 22. Comparison with Non-Stationary Computation

A simple repeated computation approach could require:

    Load Weight
        ↓
    MAC
        ↓
    Load Weight
        ↓
    MAC
        ↓
    Load Weight
        ↓
    MAC

The weight-stationary approach instead provides:

    Load Weight
        ↓
    Store Weight
        ↓
    MAC
        ↓
    MAC
        ↓
    MAC
        ↓
    MAC

This improves local weight reuse.

---

## 23. Data Movement and Energy

Moving data between memory and computation units can consume significant hardware resources and energy.

The weight-stationary architecture attempts to reduce unnecessary weight movement by storing weights locally.

The architecture therefore follows the general principle:

    Move data less
        +
    Reuse data more
        =
    More efficient computation

Actual power savings must be established through synthesis and power analysis rather than assumed from the architecture alone.

---

## 24. Matrix Multiplication Scheduling

For:

    C = A × B

each output requires a dot product.

The general calculation is:

    C[i][j] = Σ(A[i][k] × B[k][j])

The dataflow must ensure that:

- A[i][k] reaches the correct PE
- B[k][j] is associated with the correct weight
- The partial sum is accumulated correctly
- The final result is produced at the correct time

This makes scheduling a critical part of array-level design.

---

## 25. Input Alignment

Inputs must be aligned with the clock cycles in which they are required.

For example:

    Cycle 0:
        Input data begins

    Cycle 1:
        Next input data

    Cycle 2:
        Next input data

    Cycle 3:
        Next input data

The correct sequence ensures that each PE receives the intended operand at the intended time.

Incorrect alignment can cause:

- Wrong multiplication pairs
- Incorrect partial sums
- Incorrect matrix results
- Timing-related functional errors

---

## 26. Weight Mapping

Each PE must contain the correct weight for the computation being performed.

Conceptually:

    Matrix B
       |
       v
    Weight Mapping
       |
       +----> PE00
       +----> PE01
       +----> PE02
       ...
       +----> PE33

The mapping between matrix elements and PE weight registers must be defined explicitly during array integration.

---

## 27. Partial-Sum Initialization

Before beginning a new dot product, the initial partial sum must be known.

For a standard matrix multiplication:

    Initial PSUM = 0

Then:

    PSUM_1 = 0 + A0×W0

    PSUM_2 = PSUM_1 + A1×W1

    PSUM_3 = PSUM_2 + A2×W2

    PSUM_4 = PSUM_3 + A3×W3

The final value becomes the matrix output element.

---

## 28. Signed and Unsigned Data

The arithmetic behavior of the accelerator depends on how activation and weight values are represented.

Possible representations include:

    Unsigned integers

or:

    Signed two's-complement integers

The exact representation must be defined by the Verilog RTL.

This affects:

- Multiplication
- Addition
- Result width
- Overflow behavior
- Expected simulation results

Therefore, arithmetic widths and signedness must remain consistent throughout the design.

---

## 29. Arithmetic Width Consideration

Multiplication increases the number of bits required to represent the result.

If:

    Activation width = N bits

and:

    Weight width = M bits

the multiplication result may require up to:

    N + M bits

The accumulator must provide sufficient width to hold the sum of multiple products.

For a four-term accumulation:

    PSUM =
        Product0
        + Product1
        + Product2
        + Product3

The accumulator width must therefore be selected carefully.

The exact widths used by the project are defined by the Verilog RTL.

---

## 30. Overflow Consideration

If the accumulator width is insufficient, arithmetic overflow can occur.

For example:

    Maximum Product
          +
    Maximum Product
          +
    Maximum Product
          +
    Maximum Product

may exceed the available accumulator range.

Therefore, accumulator width is an important design parameter.

Overflow behavior must be verified using suitable test cases.

---

## 31. Reset and Dataflow

Reset establishes a known starting state before computation.

The conceptual sequence is:

    Reset
      ↓
    Clear / Initialize PE State
      ↓
    Load Weights
      ↓
    Start Computation
      ↓
    Propagate Activations
      ↓
    Perform MACs
      ↓
    Produce Results

Reset should be applied according to the reset behavior implemented in the Verilog RTL.

---

## 32. New Computation

When a new matrix multiplication begins, the accelerator must ensure that old computation state does not corrupt the new result.

The required preparation can include:

- Reset or initialization
- New weight loading
- Partial-sum initialization
- Correct input scheduling

The exact mechanism will be defined by the top-level control architecture.

---

## 33. Systolic Timing Concept

The defining timing characteristic is that different pieces of data can be in different PEs during the same clock cycle.

For example:

    Cycle N:

    PE00 processes A0
    PE01 processes A_previous
    PE02 processes another activation
    PE03 processes another activation

At the same time, different partial sums may be present in different parts of the array.

This spatial and temporal overlap enables high throughput.

---

## 34. Latency vs Throughput

Two important performance metrics are:

### Latency

The number of cycles required from computation start until the required output is available.

### Throughput

The amount of computation that can be performed per unit time once the array is operating efficiently.

The systolic array may have non-zero pipeline latency while still providing high throughput.

These metrics must be measured from the final RTL implementation.

---

## 35. Theoretical MAC Throughput

The target architecture contains:

    4 × 4 = 16 PEs

If every PE performs one MAC during a clock cycle:

    MACs per cycle = 16

At a hypothetical clock frequency of:

    100 MHz

the theoretical MAC rate is:

    16 × 100,000,000

    = 1,600,000,000 MAC/s

    = 1.6 GMAC/s

This is an architectural example, not a measured result.

Actual performance depends on:

- Clock frequency
- PE utilization
- Pipeline fill
- Pipeline drain
- Control overhead
- Data availability
- Memory bandwidth

---

## 36. Dataflow Verification

The dataflow must be verified at multiple levels.

### PE Level

Verify:

    Activation_in
         ↓
        PE
         ↓
    Activation_out

and:

    PSUM_out =
        PSUM_in + Activation × Weight

### Array Level

Verify:

    PE-to-PE propagation

### System Level

Verify:

    Matrix A
        +
    Matrix B
        ↓
    Matrix C

All three levels are necessary for complete verification.

---

## 37. Common Dataflow Errors

Potential errors include:

### Incorrect Weight

A PE receives the wrong weight.

### Incorrect Activation Timing

An activation arrives one or more cycles early or late.

### Incorrect Partial Sum

The wrong partial sum is supplied.

### Missing Reset

Old state remains in a PE.

### Incorrect Forwarding

Activation does not reach the neighboring PE correctly.

### Incorrect Boundary Handling

Edge PEs are connected incorrectly.

### Incorrect Output Timing

The output is read before the computation is complete.

These issues will be addressed during array-level verification.

---

## 38. Dataflow Debugging Method

When an array-level result is incorrect, debugging should proceed from the lowest level upward.

The recommended sequence is:

    Check PE Inputs
          ↓
    Check Weight Values
          ↓
    Check Activation Values
          ↓
    Check PE Product
          ↓
    Check PSUM
          ↓
    Check Activation Forwarding
          ↓
    Check Neighboring PE
          ↓
    Check Output Timing
          ↓
    Check Final Matrix Result

This prevents immediately assuming that the complete architecture is incorrect.

---

## 39. Current Dataflow Status

| Feature | Status |
|---------|--------|
| Weight-stationary concept | Defined |
| PE weight storage | Verified |
| Activation input | Verified |
| Activation forwarding | Verified |
| MAC operation | Verified |
| Partial-sum operation | Verified |
| PE-level dataflow | Verified |
| 4×4 dataflow | Defined |
| Full cycle schedule | To be finalized |
| Array-level dataflow verification | Pending |
| Matrix multiplication verification | Pending |

---

## 40. Summary

The accelerator uses weight-stationary dataflow.

The primary principle is:

    Weight stays in PE
    Activation moves through PE
    Partial sum is accumulated

The fundamental PE equation is:

    PSUM_out = PSUM_in + (Activation × Weight)

The weight-stationary approach provides local weight reuse, while the systolic organization enables structured activation movement and parallel MAC computation.

The PE-level dataflow has already been implemented and verified.

The next stage is to establish and verify the exact cycle-level dataflow of the complete 4×4 array.

This includes:

- Weight mapping
- Activation scheduling
- PE-to-PE connections
- Partial-sum handling
- Pipeline fill
- Steady-state operation
- Pipeline drain
- Output collection
- Complete matrix multiplication verification
