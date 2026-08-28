# Matrix Multiplication Mapping

## 1. Overview

The primary computational objective of the 4x4 weight-stationary systolic array is matrix multiplication.

The accelerator is designed to perform:

    C = A x B

where:

    A = Input Activation Matrix
    B = Weight Matrix
    C = Output Matrix

For the current architecture, the matrices are mapped onto a 4x4 Processing Element (PE) array.

The PE performs the fundamental multiply-and-accumulate operation required for matrix multiplication.


## 2. Matrix Dimensions

For the initial implementation, the matrices are considered to be 4x4.

Therefore:

    A = 4x4

    B = 4x4

    C = 4x4

The multiplication is:

    C(4x4) = A(4x4) x B(4x4)


## 3. Matrix Representation

Let matrix A be:

    A =

    | A00  A01  A02  A03 |
    | A10  A11  A12  A13 |
    | A20  A21  A22  A23 |
    | A30  A31  A32  A33 |


Let matrix B be:

    B =

    | B00  B01  B02  B03 |
    | B10  B11  B12  B13 |
    | B20  B21  B22  B23 |
    | B30  B31  B32  B33 |


The output matrix C is:

    C =

    | C00  C01  C02  C03 |
    | C10  C11  C12  C13 |
    | C20  C21  C22  C23 |
    | C30  C31  C32  C33 |


## 4. Matrix Multiplication Equation

Each output element is calculated as a dot product.

The general equation is:

    Cij = SUM(Aik x Bkj)

where:

    k = 0 to 3

For the 4x4 matrix:

    Cij = Ai0 x B0j
        + Ai1 x B1j
        + Ai2 x B2j
        + Ai3 x B3j


## 5. Example Output Element

Consider C00.

The calculation is:

    C00 = A00 x B00
        + A01 x B10
        + A02 x B20
        + A03 x B30

This means four multiplication operations contribute to one output element.

The systolic array performs these operations through multiple PEs and accumulation over time.


## 6. C01 Calculation

The second element of the first output row is:

    C01 = A00 x B01
        + A01 x B11
        + A02 x B21
        + A03 x B31


## 7. C02 Calculation

The third element of the first output row is:

    C02 = A00 x B02
        + A01 x B12
        + A02 x B22
        + A03 x B32


## 8. C03 Calculation

The fourth element of the first output row is:

    C03 = A00 x B03
        + A01 x B13
        + A02 x B23
        + A03 x B33


## 9. General Output Matrix

The complete output matrix is:

    C =

    | A00B00 + A01B10 + A02B20 + A03B30 |
    | A00B01 + A01B11 + A02B21 + A03B31 |
    | A00B02 + A01B12 + A02B22 + A03B32 |
    | A00B03 + A01B13 + A02B23 + A03B33 |

The above representation shows only the first row.

The complete 4x4 result is obtained by applying the same dot-product operation to all rows and columns.


## 10. Complete Matrix Equation

The complete multiplication is:

    | C00  C01  C02  C03 |
    | C10  C11  C12  C13 |
    | C20  C21  C22  C23 |
    | C30  C31  C32  C33 |

where:

    C00 = A00B00 + A01B10 + A02B20 + A03B30

    C01 = A00B01 + A01B11 + A02B21 + A03B31

    C02 = A00B02 + A01B12 + A02B22 + A03B32

    C03 = A00B03 + A01B13 + A02B23 + A03B33

    C10 = A10B00 + A11B10 + A12B20 + A13B30

    C11 = A10B01 + A11B11 + A12B21 + A13B31

    C12 = A10B02 + A11B12 + A12B22 + A13B32

    C13 = A10B03 + A11B13 + A12B23 + A13B33

    C20 = A20B00 + A21B10 + A22B20 + A23B30

    C21 = A20B01 + A21B11 + A22B21 + A23B31

    C22 = A20B02 + A21B12 + A22B22 + A23B32

    C23 = A20B03 + A21B13 + A22B23 + A23B33

    C30 = A30B00 + A31B10 + A32B20 + A33B30

    C31 = A30B01 + A31B11 + A32B21 + A33B31

    C32 = A30B02 + A31B12 + A32B22 + A33B32

    C33 = A30B03 + A31B13 + A32B23 + A33B33


## 11. Conventional Matrix Multiplication

A conventional software implementation can be represented by three nested loops:

    for i = 0 to 3
        for j = 0 to 3
            C[i][j] = 0

            for k = 0 to 3
                C[i][j] =
                    C[i][j] + A[i][k] x B[k][j]

The systolic array implements the same mathematical operation using spatially distributed hardware.


## 12. Mapping to the Systolic Array

The 4x4 matrix multiplication is mapped onto a 4x4 PE array.

Conceptually:

             Weight Matrix B
                  |
                  v

        +------+------+------+------+
        | PE00 | PE01 | PE02 | PE03 |
        +------+------+------+------+
        | PE10 | PE11 | PE12 | PE13 |
        +------+------+------+------+
        | PE20 | PE21 | PE22 | PE23 |
        +------+------+------+------+
        | PE30 | PE31 | PE32 | PE33 |
        +------+------+------+------+

                  ^
                  |
             Activation Matrix A


Each PE performs a multiply-and-accumulate operation.


## 13. PE Mapping

Each PE corresponds to one spatial position in the array.

The PE positions are:

    PE00  PE01  PE02  PE03

    PE10  PE11  PE12  PE13

    PE20  PE21  PE22  PE23

    PE30  PE31  PE32  PE33

The exact relationship between PE position and output element depends on the selected dataflow and timing schedule.


## 14. Weight-Stationary Dataflow

The architecture follows a weight-stationary approach.

The basic idea is:

    Load Weight
         |
         v
    Store Weight in PE
         |
         v
    Keep Weight Stationary
         |
         v
    Stream Activations
         |
         v
    Perform MAC
         |
         v
    Forward Activation
         |
         v
    Accumulate Partial Sum


The weights remain inside the PEs while activation data moves through the array.


## 15. Weight Loading

Before matrix multiplication begins, the required weights are loaded into the PE array.

Conceptually:

    B00 -> PE00
    B01 -> PE01
    B02 -> PE02
    B03 -> PE03

    B10 -> PE10
    B11 -> PE11
    B12 -> PE12
    B13 -> PE13

    B20 -> PE20
    B21 -> PE21
    B22 -> PE22
    B23 -> PE23

    B30 -> PE30
    B31 -> PE31
    B32 -> PE32
    B33 -> PE33

The exact physical loading sequence depends on the RTL architecture and control logic.


## 16. Activation Movement

Activation data is streamed through the PE array.

Conceptually:

    Activation
        |
        v
      PE00 -> PE01 -> PE02 -> PE03
        |
        v
      PE10 -> PE11 -> PE12 -> PE13
        |
        v
      PE20 -> PE21 -> PE22 -> PE23
        |
        v
      PE30 -> PE31 -> PE32 -> PE33

The exact direction and timing are determined by the implemented array architecture.

The PE itself has already been designed to support activation forwarding.


## 17. Partial-Sum Movement

Partial sums represent accumulated multiplication results.

At the PE level:

    PSUM_out =
        PSUM_in + (Activation x Weight)

Therefore, the PE combines:

    Incoming Partial Sum

with:

    Current Multiplication Result

to produce:

    Updated Partial Sum


## 18. PE Computation

The fundamental PE operation is:

    Product = Activation x Weight

followed by:

    Updated_PSUM =
        Incoming_PSUM + Product

Therefore:

    Updated_PSUM =
        Incoming_PSUM + (Activation x Weight)


## 19. Dot-Product Formation

A matrix multiplication output element requires multiple products.

For example:

    C00 = A00B00
        + A01B10
        + A02B20
        + A03B30

The systolic architecture accumulates these products through successive PE operations.

Conceptually:

    A00B00
       |
       v
    Partial Sum 1
       |
       v
    + A01B10
       |
       v
    Partial Sum 2
       |
       v
    + A02B20
       |
       v
    Partial Sum 3
       |
       v
    + A03B30
       |
       v
    C00


## 20. Example Matrix

For functional verification, a small deterministic matrix is useful.

Example:

    A =

    | 1  2  3  4 |
    | 5  6  7  8 |
    | 9 10 11 12 |
    |13 14 15 16 |


and:

    B =

    | 1  0  0  0 |
    | 0  1  0  0 |
    | 0  0  1  0 |
    | 0  0  0  1 |


Here B is the identity matrix.


## 21. Expected Result for Identity Matrix

For:

    C = A x I

the result should be:

    C = A

Therefore:

    C =

    | 1  2  3  4 |
    | 5  6  7  8 |
    | 9 10 11 12 |
    |13 14 15 16 |

This is a useful sanity test for the matrix multiplication datapath.


## 22. Numerical Test Matrix

Another useful test case is:

    A =

    | 1  2  3  4 |
    | 5  6  7  8 |
    | 9 10 11 12 |
    |13 14 15 16 |


    B =

    | 16 15 14 13 |
    | 12 11 10  9 |
    |  8  7  6  5 |
    |  4  3  2  1 |


The expected result can be calculated independently before running the RTL simulation.


## 23. Independent Reference Calculation

The RTL result must not be used as the reference for its own verification.

Instead:

    Input Matrices
          |
          v
    Independent Calculation
          |
          v
    Expected Matrix C

and separately:

    Input Matrices
          |
          v
    Verilog Systolic Array
          |
          v
    Actual Matrix C

Then:

    Expected Matrix C
          |
          v
       Compare
          ^
          |
    Actual Matrix C


## 24. Why Independent Verification Is Required

Using the same RTL logic to calculate the expected result would not provide meaningful verification.

An independent reference calculation helps detect:

- Incorrect multiplication
- Incorrect accumulation
- Incorrect data mapping
- Incorrect weight placement
- Incorrect activation ordering
- Incorrect output ordering
- Timing-related errors


## 25. Matrix Multiplication Verification Flow

The complete verification flow will be:

    Select Matrix A
          |
          v
    Select Matrix B
          |
          v
    Calculate Expected C
          |
          v
    Load B as Weights
          |
          v
    Stream A as Activations
          |
          v
    Perform PE MAC Operations
          |
          v
    Collect Output C
          |
          v
    Compare Expected and Actual
          |
          v
       PASS / FAIL


## 26. Weight-Stable Operation

During the main computation phase, weights should remain stable inside their respective PEs.

Conceptually:

    Weight Load Phase
          |
          v
    B values stored
          |
          v
    Computation Phase
          |
          v
    Weights remain stationary
          |
          v
    Activations move
          |
          v
    MAC operations occur


## 27. Activation Reuse

A major advantage of the systolic architecture is that data can be reused as it moves through the array.

An activation does not necessarily need to be independently supplied to every PE.

Instead:

    Activation
        |
        v
       PE
        |
        v
    Forwarded Activation
        |
        v
       Next PE

This reduces unnecessary external data movement.


## 28. Partial-Sum Accumulation

The partial sum is progressively updated.

For a sequence of four products:

    P0 = A0 x W0

    P1 = A1 x W1

    P2 = A2 x W2

    P3 = A3 x W3

The accumulation becomes:

    PSUM1 = PSUM_initial + P0

    PSUM2 = PSUM1 + P1

    PSUM3 = PSUM2 + P2

    PSUM4 = PSUM3 + P3

The final value is the dot product.


## 29. Dot Product Example

Consider:

    A = [1, 2, 3, 4]

and:

    W = [5, 6, 7, 8]

The dot product is:

    1x5 + 2x6 + 3x7 + 4x8

    = 5 + 12 + 21 + 32

    = 70

Therefore:

    Dot Product = 70


## 30. Mapping the Dot Product to PEs

The PE array can implement the dot product through sequential accumulation.

Conceptually:

    PE0:
        1 x 5 = 5

    PE1:
        2 x 6 = 12

    PE2:
        3 x 7 = 21

    PE3:
        4 x 8 = 32

Accumulation:

    5
    |
    + 12
    |
    = 17
    |
    + 21
    |
    = 38
    |
    + 32
    |
    = 70


The exact spatial and temporal mapping depends on the final array implementation.


## 31. Data Movement and Computation

The systolic array combines computation and communication.

Each PE:

    Receives data
         |
         v
    Performs MAC
         |
         v
    Forwards required data
         |
         v
    Produces updated partial result

This creates a regular dataflow across the array.


## 32. Systolic Timing

The array operates over multiple clock cycles.

Different matrix elements enter the array at different times.

Therefore, the complete matrix multiplication cannot be treated as a single-cycle operation.

The verification must account for:

- Input injection cycle
- Weight loading cycle
- Activation propagation
- MAC cycle
- Partial-sum propagation
- Output availability cycle


## 33. Pipeline Behavior

The array can be viewed as a spatial pipeline.

Conceptually:

    Cycle 1
        PE00

    Cycle 2
        PE00 -> PE01

    Cycle 3
        PE00 -> PE01 -> PE02

    Cycle 4
        PE00 -> PE01 -> PE02 -> PE03

This is a simplified conceptual representation.

The actual timing must be derived from the implemented RTL.


## 34. Fill Phase

At the beginning of computation, the array requires time for data to propagate into the correct PEs.

This period is commonly referred to as the array fill phase.

Conceptually:

    Input
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

The first valid complete results appear only after the required propagation and accumulation cycles.


## 35. Steady-State Phase

After the pipeline has been filled, multiple PEs can perform useful MAC operations during the same clock cycle.

Conceptually:

    Cycle N:

    PE00 -> MAC
    PE01 -> MAC
    PE02 -> MAC
    PE03 -> MAC
    ...

This spatial parallelism is one of the major benefits of the systolic architecture.


## 36. Drain Phase

After the final input data has entered the array, remaining partial sums may still need to propagate to their final destinations.

This is the drain phase.

Conceptually:

    Final Input
        |
        v
    Final MAC
        |
        v
    Partial Sum Propagation
        |
        v
    Output Matrix Complete


## 37. Latency

The matrix multiplication latency is determined by:

- Array dimensions
- Input scheduling
- PE pipeline behavior
- Weight loading
- Activation propagation
- Partial-sum propagation
- Output collection

The exact latency must be measured from the final RTL simulation.

It should not be assumed before timing verification.


## 38. Throughput

The systolic architecture is designed to provide high computational throughput by allowing multiple PEs to operate concurrently.

Throughput and latency are different concepts.

Latency:

    Time required to obtain a result after starting computation.

Throughput:

    Amount of computation that can be completed per unit time after the pipeline is operating.


## 39. Arithmetic Operation Count

For a 4x4 matrix multiplication:

    A(4x4) x B(4x4)

there are:

    4 x 4 x 4

multiplication operations.

Therefore:

    Number of Multiplications = 64

There are also accumulation operations associated with forming each output element.


## 40. Output Element Count

A 4x4 output matrix contains:

    4 x 4 = 16

output elements.

Each output element requires four multiplication terms.

Therefore:

    16 output elements
    x
    4 multiplication terms

    = 64 multiplication operations


## 41. PE Count

The architecture contains:

    4 x 4 = 16 PEs

Therefore, the accelerator uses:

    16 Processing Elements

Each PE contains the arithmetic and data movement logic required by the implemented architecture.


## 42. Parallelism

The architecture provides spatial parallelism because multiple PEs can perform operations concurrently.

Instead of using one multiplier and one accumulator for every operation sequentially, the systolic array distributes computation across:

    PE00
    PE01
    ...
    PE33

This allows multiple MAC operations to occur in parallel.


## 43. Data Reuse

The architecture is designed to improve data reuse.

Weights remain stationary within PEs.

Activations are forwarded between PEs.

Partial sums are accumulated locally and propagated as required.

This reduces unnecessary movement of data to and from external storage.


## 44. Memory Movement

One of the important motivations for a systolic architecture is reducing excessive memory movement.

Conceptually:

    Conventional Approach

    Memory
      |
      v
    Compute
      |
      v
    Memory
      |
      v
    Compute
      |
      v
    Memory

The systolic approach instead attempts to keep data moving locally between neighboring PEs.


## 45. Local Communication

The PE architecture uses local communication between neighboring processing elements.

Conceptually:

    PE00 ---> PE01 ---> PE02 ---> PE03
      |        |        |        |
      v        v        v        v
    Local    Local    Local    Local
    Data     Data     Data     Data

Local communication helps create a regular hardware structure.


## 46. Scalability

The 4x4 implementation is a manageable architecture for demonstrating systolic computation.

The same concept can be extended to larger arrays such as:

    8x8
    16x16
    32x32
    64x64

Larger arrays provide more parallel computation but also increase:

- Hardware area
- Routing complexity
- Power consumption
- Verification complexity
- Control complexity


## 47. Data Width Considerations

The multiplication result width must be sufficient to represent:

    Activation x Weight

The accumulated partial sum must be wide enough to represent the sum of multiple products.

For a 4-element dot product:

    PSUM =
        P0 + P1 + P2 + P3

The exact Verilog signal widths must be selected based on the operand widths and expected numerical range.

The current RTL signal declarations are the authoritative source for the implemented widths.


## 48. Signed and Unsigned Arithmetic

The arithmetic behavior depends on how the Verilog operands are declared.

The design documentation must clearly identify whether:

    Activation

and:

    Weight

are treated as signed or unsigned values.

This is important because signed and unsigned multiplication can produce different interpretations of the same bit pattern.

The current RTL implementation should be treated as the source of truth.


## 49. Overflow Considerations

During accumulation, the result must fit within the selected partial-sum width.

If the width is insufficient, overflow can occur.

Conceptually:

    Product
       |
       v
    Partial Sum
       |
       v
    Partial Sum
       |
       v
    Final Result

The accumulator width must therefore be considered during RTL design and verification.


## 50. Testbench Strategy

The matrix multiplication testbench should:

    1. Reset the design.
    2. Initialize input matrices.
    3. Load weights.
    4. Apply activation data.
    5. Wait for the required computation period.
    6. Capture outputs.
    7. Calculate expected results independently.
    8. Compare actual and expected outputs.
    9. Report PASS or FAIL.


## 51. Recommended Test Cases

The matrix multiplication verification should eventually include:

    Test 1:
        Identity Matrix

    Test 2:
        All-Zero Matrix

    Test 3:
        Small Positive Values

    Test 4:
        Repeated Values

    Test 5:
        Increasing Values

    Test 6:
        Different Weight Patterns

    Test 7:
        Maximum Supported Values

    Test 8:
        Multiple Consecutive Matrix Operations


## 52. Zero Matrix Test

A zero matrix provides a basic sanity check.

If:

    B = Zero Matrix

then:

    A x B = Zero Matrix

Therefore, every output should be zero, assuming no non-zero initial partial sum is intentionally supplied.


## 53. Identity Matrix Test

If:

    B = I

then:

    A x I = A

This test is useful for checking:

- Data ordering
- Activation propagation
- Weight placement
- Output ordering


## 54. Repeated Matrix Test

Using repeated values can help detect:

- Data duplication
- Incorrect forwarding
- Incorrect PE mapping
- Incorrect accumulation

For example:

    A =

    | 1  1  1  1 |
    | 2  2  2  2 |
    | 3  3  3  3 |
    | 4  4  4  4 |


## 55. Boundary Testing

Boundary values should eventually be tested.

Examples include:

    Minimum Supported Value

    Maximum Supported Value

    Zero

    One

The exact values depend on the operand widths and signedness used in the RTL.


## 56. Back-to-Back Matrix Operations

After basic matrix multiplication is verified, the design should eventually be tested with multiple matrix operations without unnecessary reset between every operation.

Conceptually:

    Matrix A1 x Matrix B1
            |
            v
        Result C1
            |
            v
    Matrix A2 x Matrix B2
            |
            v
        Result C2

This helps verify that old state does not incorrectly affect a new computation.


## 57. Verification Requirements

The final matrix multiplication verification should demonstrate:

    Correct Weight Loading
          |
          v
    Correct Activation Injection
          |
          v
    Correct Activation Propagation
          |
          v
    Correct MAC Computation
          |
          v
    Correct Partial-Sum Accumulation
          |
          v
    Correct Output Mapping
          |
          v
    Correct Matrix Result


## 58. Current Project Status

At the current development stage:

    Processing Element
        |
        v
    Implemented
        |
        v
    Simulated
        |
        v
    MAC Verified
        |
        v
    Activation Forwarding Verified

The complete 4x4 matrix multiplication datapath is the next integration stage.

Therefore, the complete matrix multiplication result should not yet be marked as fully verified until the 4x4 array and its testbench have been implemented and simulated.


## 59. Future Matrix-Level Verification

The planned verification flow is:

    PE Verification
          |
          v
    Two-PE Integration
          |
          v
    Four-PE Row
          |
          v
    4x4 PE Array
          |
          v
    Matrix Input Interface
          |
          v
    Weight Loading
          |
          v
    Activation Scheduling
          |
          v
    Matrix Multiplication
          |
          v
    Output Collection
          |
          v
    Reference Comparison
          |
          v
       PASS


## 60. Summary

The mathematical objective of the accelerator is:

    C = A x B

For the initial implementation:

    A = 4x4
    B = 4x4
    C = 4x4

The 4x4 systolic array contains:

    16 Processing Elements

A PE performs the fundamental operation:

    PSUM_out =
        PSUM_in + (Activation x Weight)

The weight-stationary dataflow keeps weights inside the PEs while activation data is streamed through the array.

A complete 4x4 matrix multiplication requires:

    16 output elements

and:

    64 multiplication operations

The final RTL implementation must be verified against an independent matrix multiplication reference.

The current project milestone is PE-level functional verification.

The next milestone is multi-PE integration followed by complete 4x4 matrix multiplication verification.
