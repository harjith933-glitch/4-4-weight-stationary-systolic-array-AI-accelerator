# Verification

## 1. Overview

Verification is a critical part of the systolic array development process.

The objective of verification is to ensure that the implemented Verilog RTL behaves according to the intended hardware architecture.

The verification strategy follows a bottom-up approach:

    Processing Element
          ↓
    PE-Level Verification
          ↓
    Multiple PE Integration
          ↓
    4×4 Systolic Array Verification
          ↓
    Matrix Multiplication Verification
          ↓
    Complete Accelerator Verification

The Processing Element has already been implemented and functionally verified through simulation.

The complete 4×4 array is the next verification stage.

---

## 2. Verification Objectives

The primary objectives are:

1. Verify correct reset behavior.
2. Verify correct weight loading.
3. Verify weight retention.
4. Verify activation input behavior.
5. Verify activation forwarding.
6. Verify multiplication.
7. Verify partial-sum accumulation.
8. Verify MAC operation.
9. Verify output behavior.
10. Verify cycle-level timing.
11. Detect incorrect or unexpected RTL behavior.
12. Establish a reliable foundation for array-level integration.

---

## 3. Verification Methodology

The project uses simulation-based functional verification.

The general process is:

    Verilog RTL
         +
    Verilog Testbench
         |
         v
    HDL Simulator
         |
         v
    Simulation
         |
         +------------+
         |            |
         v            v
    Console Output   Waveform
         |            |
         +------v-----+
                |
                v
           Verification

The testbench provides controlled inputs and checks the resulting outputs.

---

## 4. Verification Environment

The project is organized into separate RTL and testbench directories.

    ~/systolic/
    |
    +-- rtl/
    |     |
    |     +-- PE RTL
    |
    +-- tb/
    |     |
    |     +-- PE Testbench
    |
    +-- sim/
    |     |
    |     +-- Simulation Files
    |
    +-- wave/
    |     |
    |     +-- Waveform Files
    |
    +-- docs/
          |
          +-- Documentation

This separation keeps the synthesizable design independent from verification code.

---

## 5. Design Under Test

The Processing Element is the current Design Under Test (DUT).

Conceptually:

    +----------------------+
    |      Testbench       |
    |                      |
    | Clock                |
    | Reset                |
    | Weight               |
    | Weight Load          |
    | Activation           |
    | Partial Sum          |
    +----------+-----------+
               |
               v
        +--------------+
        |      PE      |
        |    Verilog   |
        |     RTL      |
        +------+-------+
               |
               v
        PE Output Signals
               |
               v
           Verification

The PE is tested independently before integration into the complete array.

---

## 6. Why PE-Level Verification Comes First

The Processing Element is the fundamental computational block of the accelerator.

If the PE is incorrect, integrating many PEs will make debugging significantly more difficult.

Therefore:

    Verify One PE
         ↓
    Verify PE Functionality
         ↓
    Reuse Verified PE
         ↓
    Build Array

This bottom-up strategy reduces debugging complexity.

---

## 7. Clock Generation

The testbench provides the clock required by the synchronous PE.

Conceptually:

    clk:

    __|‾‾|__|‾‾|__|‾‾|__|‾‾|__

Each relevant clock edge allows sequential RTL operations to occur.

The testbench clock must be compatible with the clocking behavior implemented by the PE.

---

## 8. Reset Verification

Reset is applied before normal computation.

The general sequence is:

    Apply Reset
         ↓
    Wait for Required Clock Event
         ↓
    Release Reset
         ↓
    Begin Normal Operation

The purpose is to ensure that the PE begins from a known state.

Reset verification is important because unknown initial register values can produce incorrect simulation results.

---

## 9. Weight Loading Test

The first major functional test verifies weight loading.

The sequence is:

    Apply Weight
         ↓
    Assert Weight Load
         ↓
    Clock Edge
         ↓
    Weight Stored
         ↓
    Deassert Weight Load
         ↓
    Perform Computation

The subsequent MAC result is used to confirm that the intended weight was successfully loaded.

---

## 10. Weight Retention Test

After loading the weight, the weight should remain stored during normal computation.

For example:

    Load Weight = W

    Weight Load = active
        ↓
    W stored

    Weight Load = inactive
        ↓
    Stored Weight = W

The stored weight is then reused for subsequent activation values.

This verifies the weight-stationary behavior of the PE.

---

## 11. Activation Input Test

The testbench applies activation data to the PE.

The activation is used as one operand of the multiplication.

Conceptually:

    Activation
         |
         v
        PE
         |
         v
    Multiplication

The output behavior is then checked against the expected MAC result.

---

## 12. Activation Forwarding Test

The PE also provides activation forwarding.

The test verifies:

    Activation_in
         |
         v
        PE
         |
         v
    Activation_out

The output activation must follow the timing defined by the RTL.

This functionality is especially important because it will later connect neighboring PEs.

---

## 13. Partial-Sum Input Test

The testbench supplies an initial partial sum.

For example:

    PSUM_in = 10

The PE must use this value as the starting point for the MAC operation.

The expected calculation is:

    PSUM_out =
        10 + (Activation × Weight)

This verifies that the incoming partial sum is correctly incorporated.

---

## 14. MAC Verification

The central PE operation is:

    PSUM_out =
        PSUM_in + (Activation × Weight)

A known test vector can be used.

Example:

    Activation = 4
    Weight = 5
    PSUM_in = 10

Calculation:

    Product = 4 × 5
            = 20

Expected:

    PSUM_out = 10 + 20
             = 30

The testbench compares the actual RTL output with:

    Expected PSUM_out = 30

If they match, the MAC operation passes for that test vector.

---

## 15. Multiple MAC Verification

A single MAC test is not sufficient to establish confidence in repeated computation.

Multiple input combinations should be used.

For example:

    Test 1:
        Activation = 2
        Weight = 3
        PSUM = 0

        Expected = 6

    Test 2:
        Activation = 4
        Weight = 3
        PSUM = 6

        Expected = 18

    Test 3:
        Activation = 5
        Weight = 3
        PSUM = 18

        Expected = 33

These tests verify repeated accumulation behavior.

---

## 16. Expected Result Calculation

For each test vector, the expected result is calculated independently.

The fundamental formula is:

    Expected PSUM =
        PSUM_in + (Activation × Weight)

The testbench compares this expected value against the RTL output.

This creates an independent reference for functional verification.

---

## 17. Expected vs Actual Verification

The verification process can be represented as:

    Test Inputs
        |
        +--------------------+
        |                    |
        v                    v
    Reference           Verilog RTL
    Calculation             |
        |                    |
        v                    v
    Expected Result     Actual Result
        |                    |
        +---------+----------+
                  |
                  v
              Comparison
                  |
          +-------+-------+
          |               |
        Match          Mismatch
          |               |
         PASS             FAIL

This approach makes functional verification objective and repeatable.

---

## 18. PASS Condition

A test passes when:

    Actual Result == Expected Result

For example:

    Expected = 30
    Actual   = 30

Therefore:

    PASS

A mismatch indicates that the RTL behavior does not match the expected behavior for that test vector.

---

## 19. FAIL Condition

A test fails when:

    Actual Result != Expected Result

For example:

    Expected = 30
    Actual   = 29

Therefore:

    FAIL

A failure should then be investigated using:

- Input values
- Clock cycle
- Weight loading
- Activation timing
- Partial-sum value
- Waveform
- RTL logic

---

## 20. Timing Verification

Functional correctness alone is not sufficient for synchronous hardware.

The correct result must also appear at the expected time.

Therefore, verification checks:

    What value?

and:

    At which clock cycle?

For example:

    Cycle N:
        Weight loaded

    Cycle N+1:
        Activation processed

    Cycle N+1 / N+2:
        Output according to RTL timing

The exact latency must be determined from the actual RTL implementation.

---

## 21. Cycle-Level Verification

Cycle-level verification is especially important for a systolic architecture.

The testbench must eventually verify:

- When data enters the array
- When data reaches each PE
- When each PE performs its MAC
- When partial sums move
- When results become valid

A correct numerical result at the wrong cycle is still a functional timing problem.

---

## 22. Waveform-Based Verification

Simulation waveforms provide detailed visibility into signal behavior.

Important signals include:

    clk
    reset
    weight
    weight_load
    activation_in
    activation_out
    psum_in
    psum_out

The waveform can be used to observe:

    Reset
      ↓
    Weight Loading
      ↓
    Activation Arrival
      ↓
    MAC Operation
      ↓
    Partial-Sum Update
      ↓
    Activation Forwarding
      ↓
    Output

---

## 23. Weight Loading Waveform

The expected conceptual waveform relationship is:

    clk
       __|‾|__|‾|__|‾|__

    reset
       ‾‾‾‾\___________

    weight_load
       ____/‾‾‾\_______

    weight
       ----[ W ]-------

At the appropriate clock event while weight loading is active, the PE captures the weight.

After loading, the stored weight should remain available.

The actual waveform must be interpreted according to the RTL implementation.

---

## 24. MAC Waveform

The MAC operation can be observed conceptually as:

    activation
       ----[ A ]-------

    weight
       ----[ W ]-------

    psum_in
       ----[ P ]-------

             |
             v

    PSUM_out
       ----[P + A×W]--

The output timing depends on whether the relevant datapath is combinational or registered.

The actual RTL waveform is the final authority.

---

## 25. Activation Forwarding Waveform

The activation propagation can be checked by observing:

    activation_in
          |
          v
        PE
          |
          v
    activation_out

If forwarding is registered, the output will appear after the corresponding clock event.

If forwarding is combinational, the output relationship will follow the RTL combinational path.

The verification must therefore be based on the implemented RTL rather than an assumed latency.

---

## 26. Reset Verification Cases

Reset verification should include:

### Case 1

Reset asserted before any operation.

### Case 2

Reset followed by weight loading.

### Case 3

Reset followed by MAC operation.

### Case 4

Reset after previous activity, if supported by the design.

The purpose is to ensure that stale state does not incorrectly affect subsequent computations.

---

## 27. Weight Loading Verification Cases

Weight loading should be tested with multiple values.

For example:

    Weight = 0

    Weight = 1

    Weight = positive value

    Different weight values

The objective is to verify that the correct weight is captured and used.

---

## 28. Activation Verification Cases

Activation inputs should include different values.

Examples:

    Activation = 0

    Activation = 1

    Small positive value

    Larger value

Additional signed test values should be used if the RTL supports signed arithmetic.

The purpose is to verify multiplication behavior over the intended input range.

---

## 29. Partial-Sum Verification Cases

The PE should be tested with different incoming partial sums.

Examples:

    PSUM_in = 0

    PSUM_in = small value

    PSUM_in = larger value

This verifies that the PE correctly adds the multiplication result to the incoming partial sum.

---

## 30. Zero-Value Test

A useful boundary test is:

    Activation = 0

Then:

    Activation × Weight = 0

Therefore:

    PSUM_out = PSUM_in

This checks whether zero activation produces the expected behavior.

---

## 31. Zero-Weight Test

Another boundary test is:

    Weight = 0

Then:

    Activation × Weight = 0

Therefore:

    PSUM_out = PSUM_in

This verifies the multiplier and accumulation behavior when the stored weight is zero.

---

## 32. Zero Partial-Sum Test

For:

    PSUM_in = 0

the result becomes:

    PSUM_out =
        Activation × Weight

This is useful for verifying the basic multiplication path independently of a non-zero accumulated value.

---

## 33. Repeated Computation Test

The same stored weight should be usable across multiple activation values.

For example:

    Stored Weight = 3

    Activation 1 = 2
    Activation 2 = 4
    Activation 3 = 5

The corresponding products are:

    2 × 3 = 6

    4 × 3 = 12

    5 × 3 = 15

This verifies weight reuse.

---

## 34. Weight Reload Test

A further test should verify that a new weight can replace an existing stored weight when the weight-load control is asserted again.

Conceptually:

    Initial Weight = W1
          ↓
    Compute
          ↓
    Load Weight = W2
          ↓
    Compute
          ↓
    Verify W2 is used

This verifies correct behavior when changing the stored weight.

---

## 35. Activation Forwarding Test Cases

Activation forwarding should be tested using:

- Zero activation
- Small activation
- Different consecutive activations
- Repeated activation
- Activation changes between clock cycles

The objective is to ensure that the forwarding path behaves consistently with the intended timing.

---

## 36. Back-to-Back Operations

The PE should eventually be tested with consecutive operations without unnecessary idle cycles.

Conceptually:

    Cycle N:
        Operation 1

    Cycle N+1:
        Operation 2

    Cycle N+2:
        Operation 3

This verifies that the datapath can operate continuously under the intended control conditions.

---

## 37. Boundary and Stress Testing

After basic functionality is verified, boundary values should be tested.

Important cases include:

- Minimum representable value
- Maximum representable value
- Zero
- Maximum multiplication result
- Maximum accumulation result
- Potential overflow conditions

These tests are particularly important once the final arithmetic widths are fixed.

---

## 38. Overflow Verification

If the arithmetic is limited to a fixed number of bits, overflow behavior must be understood.

For example:

    Product0
      +
    Product1
      +
    Product2
      +
    Product3

may exceed the accumulator range.

The testbench should eventually include values that approach the maximum supported range.

The expected behavior must match the intended RTL arithmetic.

---

## 39. Signed Arithmetic Verification

If the final RTL uses signed operands, signed test vectors must be included.

For example:

    Activation = -2
    Weight = 3

Then:

    Product = -6

If:

    PSUM_in = 10

then:

    PSUM_out = 10 + (-6)

    PSUM_out = 4

The exact supported signed range depends on the RTL widths.

---

## 40. Testbench Responsibilities

The Verilog testbench is responsible for:

1. Generating the clock.
2. Applying reset.
3. Applying input stimulus.
4. Loading weights.
5. Applying activations.
6. Applying partial sums.
7. Waiting for the appropriate timing.
8. Observing outputs.
9. Comparing expected and actual results.
10. Reporting verification results.

The testbench itself is not part of the synthesized accelerator.

---

## 41. Testbench Independence

The expected result should be determined independently from the DUT's internal implementation.

For example:

    Expected =
        PSUM_in + (Activation × Weight)

The testbench should not simply reproduce the same internal RTL logic in a way that could hide identical implementation mistakes.

An independent expected-value calculation provides stronger verification.

---

## 42. Console-Based Verification

The simulation can report results in the console.

A conceptual output format is:

    Test 1:
    Expected = ...
    Actual   = ...
    PASS

    Test 2:
    Expected = ...
    Actual   = ...
    PASS

    Test 3:
    Expected = ...
    Actual   = ...
    PASS

This makes automated verification easier to inspect.

---

## 43. Waveform Verification

Console checking tells whether the result is correct.

Waveform inspection helps explain why the result is correct.

Therefore, both should be used:

    Numerical Checking
          +
    Waveform Inspection
          |
          v
    Stronger Verification

This is particularly important for clocked hardware.

---

## 44. Current PE Verification Result

The Processing Element has been successfully simulated.

The verification demonstrated the intended PE functionality, including:

- Weight loading
- Activation input
- Activation forwarding
- Multiplication
- Partial-sum accumulation
- MAC operation
- Output behavior

The successful PE verification provides confidence in the PE before array-level integration.

---

## 45. Verification Status

| Verification Item | Status |
|-------------------|--------|
| PE reset | Verified |
| Weight loading | Verified |
| Weight storage | Verified |
| Activation input | Verified |
| Activation forwarding | Verified |
| Multiplication | Verified |
| Partial-sum operation | Verified |
| MAC operation | Verified |
| PE simulation | Passed |
| Waveform inspection | Performed |
| Multiple PE verification | Pending |
| Row-level verification | Pending |
| 4×4 array verification | Pending |
| Matrix multiplication verification | Pending |
| Full accelerator verification | Pending |

---

## 46. Verification Hierarchy

The planned verification hierarchy is:

    Level 1
    PE Verification
        ↓
    Level 2
    Two-PE Verification
        ↓
    Level 3
    PE Row Verification
        ↓
    Level 4
    4×4 Array Verification
        ↓
    Level 5
    Matrix Multiplication Verification
        ↓
    Level 6
    Top-Level Accelerator Verification

Each level should be verified before moving to the next.

---

## 47. Two-PE Verification

The next practical verification stage is to connect two PEs.

Conceptually:

    Activation
        |
        v
      PE0
        |
        v
      PE1

The test should verify:

- Activation forwarding
- Independent weight storage
- Correct multiplication in each PE
- Correct partial-sum propagation
- Correct cycle alignment

This provides an intermediate step before building the complete 4×4 array.

---

## 48. Row-Level Verification

After two-PE verification, a complete row can be constructed.

For example:

    PE00 → PE01 → PE02 → PE03

The row-level test should verify:

- Activation propagation
- Weight placement
- MAC operations
- Timing
- Output behavior

This establishes confidence in horizontal data movement.

---

## 49. 4×4 Array Verification

After row-level verification, the complete array will be tested.

Target structure:

    +------+ +------+ +------+ +------+
    | PE00 | | PE01 | | PE02 | | PE03 |
    +------+ +------+ +------+ +------+
       ↓        ↓        ↓        ↓
    +------+ +------+ +------+ +------+
    | PE10 | | PE11 | | PE12 | | PE13 |
    +------+ +------+ +------+ +------+
       ↓        ↓        ↓        ↓
    +------+ +------+ +------+ +------+
    | PE20 | | PE21 | | PE22 | | PE23 |
    +------+ +------+ +------+ +------+
       ↓        ↓        ↓        ↓
    +------+ +------+ +------+ +------+
    | PE30 | | PE31 | | PE32 | | PE33 |
    +------+ +------+ +------+ +------+

The final interconnection must follow the selected systolic dataflow.

---

## 50. Matrix Multiplication Verification

The final functional objective is to verify:

    C = A × B

For each output element:

    C[i][j] =
        A[i][0]×B[0][j]
      + A[i][1]×B[1][j]
      + A[i][2]×B[2][j]
      + A[i][3]×B[3][j]

The testbench should calculate the expected matrix using a reference model.

The RTL output matrix is then compared against the expected matrix.

---

## 51. Reference Model

A software reference calculation can be used to determine expected matrix results.

For a 4×4 matrix:

    for i = 0 to 3
        for j = 0 to 3

            C[i][j] = 0

            for k = 0 to 3

                C[i][j] =
                    C[i][j]
                    + A[i][k] × B[k][j]

The hardware result should match the reference calculation.

The reference model should remain independent of the RTL implementation.

---

## 52. Test Matrix Selection

Verification should use multiple matrix patterns.

Recommended patterns include:

### Matrix of Ones

Useful for simple predictable results.

### Identity Matrix

Useful for verifying correct data routing.

### Zero Matrix

Useful for checking zero propagation.

### Small Known Values

Useful for manually calculating expected results.

### Random Matrices

Useful for broader functional coverage.

### Boundary Values

Useful for arithmetic range and overflow testing.

---

## 53. Identity Matrix Test

For:

    B = Identity Matrix

the expected result is:

    C = A

Therefore, the test is useful for checking whether the array correctly routes and multiplies operands.

For example:

    A × I = A

If the result differs from A, the dataflow or weight mapping may be incorrect.

---

## 54. Zero Matrix Test

For:

    B = 0

the expected result is:

    C = 0

This test checks whether zero weights correctly produce zero contributions.

It also provides a simple way to detect unintended residual partial sums.

---

## 55. Randomized Verification

After deterministic tests pass, randomized input matrices can be used.

The general process is:

    Generate Matrix A
          +
    Generate Matrix B
          |
          v
    Reference Calculation
          |
          v
    Expected Matrix
          |
          +
          v
    RTL Simulation
          |
          v
    Actual Matrix
          |
          v
    Compare
          |
          v
        PASS/FAIL

Randomized testing increases the range of exercised input combinations.

---

## 56. Regression Testing

As the design grows, previously passing tests should continue to pass.

A regression suite can contain:

    Reset Tests
    Weight Tests
    Activation Tests
    MAC Tests
    Forwarding Tests
    Boundary Tests
    Array Tests
    Matrix Tests

Whenever RTL changes are made, the regression suite should be rerun.

This helps detect unintended functional regressions.

---

## 57. Functional Coverage Goals

The verification process should eventually cover:

- All PE control states
- Weight loading
- Weight retention
- Activation propagation
- Partial-sum propagation
- Different arithmetic values
- Boundary values
- Reset conditions
- Back-to-back operations
- Array-level operation
- Matrix multiplication
- Output timing

Coverage should increase as the design progresses.

---

## 58. Verification Limitations

Current PE-level verification does not prove:

- Complete 4×4 array correctness
- Complete matrix multiplication
- Maximum operating frequency
- Synthesis timing
- Area
- Power
- Physical implementation correctness

Those properties require additional verification and analysis stages.

---

## 59. Verification-to-Implementation Flow

The complete intended development flow is:

    RTL Design
        ↓
    PE Verification
        ↓
    Array Integration
        ↓
    Array Verification
        ↓
    Matrix Verification
        ↓
    RTL Signoff Checks
        ↓
    Synthesis
        ↓
    Timing Analysis
        ↓
    Power Analysis
        ↓
    Physical Design

Functional verification is therefore one stage of the complete hardware development lifecycle.

---

## 60. Verification Best Practices

The following practices should be maintained:

1. Verify small modules before integration.
2. Use deterministic test vectors.
3. Use independent expected-value calculations.
4. Check both values and timing.
5. Inspect waveforms for failures.
6. Test boundary conditions.
7. Test reset behavior.
8. Test repeated operations.
9. Test data forwarding separately.
10. Maintain regression tests.
11. Record actual results.
12. Do not claim unverified array-level functionality.
13. Keep verification results synchronized with RTL revisions.

---

## 61. Current Verification Milestone

The first major verification milestone has been achieved:

    PE RTL
       ↓
    PE Testbench
       ↓
    Simulation
       ↓
    Functional Verification
       ↓
       PASS

The verified PE can now be used as the building block for the next stage.

---

## 62. Next Verification Milestone

The next milestone is:

    Two-PE Integration
          ↓
    Verify Activation Forwarding
          ↓
    Verify Independent Weights
          ↓
    Verify Partial-Sum Flow
          ↓
    Verify Cycle Alignment

After this succeeds:

    Two PE
       ↓
    Four PE Row
       ↓
    4×4 Array
       ↓
    Matrix Multiplication

---

## 63. Summary

The verification process uses a bottom-up, simulation-driven methodology.

The Processing Element has been successfully verified for its core functions:

    Weight Loading
          ↓
    Weight Storage
          ↓
    Activation Processing
          ↓
    Multiplication
          ↓
    Partial-Sum Accumulation
          ↓
    MAC Result

and:

    Activation Input
          ↓
    Activation Forwarding
          ↓
    Activation Output

The next verification stage is multi-PE integration followed by complete 4×4 systolic array verification and matrix multiplication testing.

The project does not claim complete accelerator verification until these higher-level stages have been successfully simulated and verified.
