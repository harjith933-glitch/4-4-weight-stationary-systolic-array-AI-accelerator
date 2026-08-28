# Verification Plan

## 1. Overview

This document defines the verification strategy for the 4x4 weight-stationary systolic array accelerator.

The objective of verification is to prove that the implemented Verilog RTL performs the intended multiply-and-accumulate operations correctly and that data moves through the Processing Elements according to the intended architecture.

Verification is performed progressively, starting from the individual Processing Element and moving toward the complete 4x4 systolic array.

The verification strategy prioritizes:

- Functional correctness
- Cycle-level correctness
- Data propagation correctness
- Weight loading correctness
- Partial-sum accumulation correctness
- Matrix multiplication correctness
- Boundary-condition testing
- Regression testing


## 2. Verification Objectives

The main objectives are:

1. Verify the Processing Element independently.
2. Verify weight loading and storage.
3. Verify activation input handling.
4. Verify activation forwarding.
5. Verify multiplication.
6. Verify partial-sum accumulation.
7. Verify reset behavior.
8. Verify multi-PE communication.
9. Verify the complete 4x4 array.
10. Verify matrix multiplication.
11. Compare RTL outputs against independently calculated expected results.
12. Detect timing and data-alignment errors.
13. Verify operation across multiple test vectors.


## 3. Verification Philosophy

The project follows a bottom-up verification methodology.

The verification flow is:

    PE
     |
     v
    PE Verification
     |
     v
    Multi-PE Integration
     |
     v
    Row / Column Verification
     |
     v
    4x4 Array Verification
     |
     v
    Matrix Multiplication Verification
     |
     v
    Stress Testing


The lower-level block must be verified before moving to the next integration level.


## 4. Verification Environment

The project uses a dedicated testbench environment.

Project structure:

    systolic/
    |
    +-- rtl/
    |     |
    |     +-- PE RTL
    |     +-- Array RTL
    |     +-- Supporting RTL
    |
    +-- tb/
    |     |
    |     +-- PE Testbench
    |     +-- Array Testbench
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


## 5. Verification Levels

Verification is divided into multiple levels.

### Level 1

Individual PE verification.

### Level 2

Two-PE communication verification.

### Level 3

Small PE-chain verification.

### Level 4

4x4 systolic array verification.

### Level 5

Matrix multiplication verification.

### Level 6

Stress, boundary, and regression testing.


## 6. Level 1: PE Verification

The first verification target is the Processing Element.

The PE is the fundamental computational unit of the systolic array.

The PE must be verified for:

- Reset
- Weight loading
- Weight retention
- Activation input
- Activation forwarding
- Multiplication
- Partial-sum input
- Partial-sum accumulation
- Output generation


## 7. PE Functional Model

The fundamental PE computation is:

    PSUM_out =
        PSUM_in + (Activation x Weight)


For example:

    Activation = 4

    Weight = 5

    PSUM_in = 10


Then:

    Activation x Weight
        = 4 x 5
        = 20


Therefore:

    PSUM_out
        = 10 + 20
        = 30


Expected:

    PSUM_out = 30


## 8. PE Weight Loading Test

The first major PE test verifies that a weight can be loaded correctly.

Test procedure:

    1. Apply reset.
    2. Enable weight loading.
    3. Apply a known weight.
    4. Apply the required clock.
    5. Disable weight loading.
    6. Verify that the weight is retained.
    7. Perform a MAC operation.


Example:

    Weight = 5

After loading:

    Stored Weight = 5


The stored weight should remain unchanged during normal computation until another valid weight-loading operation occurs.


## 9. PE Activation Test

A known activation value is applied to the PE.

Example:

    Activation = 4

    Weight = 5

Expected multiplication:

    4 x 5 = 20


The resulting product must contribute correctly to the partial sum.


## 10. PE Partial-Sum Test

A known partial sum is supplied.

Example:

    Activation = 4
    Weight = 5
    PSUM_in = 10


Expected:

    PSUM_out = 10 + (4 x 5)

    PSUM_out = 30


The waveform should show the correct accumulation after the relevant clock edge.


## 11. PE Activation Forwarding Test

The PE must forward activation data according to the implemented architecture.

Test procedure:

    Apply activation
        |
        v
       PE
        |
        v
    Activation Output


The output activation should correspond to the expected forwarded input value at the expected cycle.


## 12. PE Reset Test

Reset behavior must be verified before normal computation.

Test procedure:

    1. Assert reset.
    2. Apply one or more clock cycles.
    3. Deassert reset.
    4. Check PE outputs and internal state.
    5. Begin normal operation.


The exact reset values must match the implemented Verilog RTL.


## 13. PE Test Cases

The initial PE testbench should include:

    Test Case 1:
        Reset

    Test Case 2:
        Weight Loading

    Test Case 3:
        Weight Retention

    Test Case 4:
        Basic Multiplication

    Test Case 5:
        MAC with Zero Partial Sum

    Test Case 6:
        MAC with Non-Zero Partial Sum

    Test Case 7:
        Activation Forwarding

    Test Case 8:
        Multiple Consecutive MAC Operations


## 14. Zero Partial-Sum Test

The PE should be tested with:

    PSUM_in = 0


Example:

    Activation = 3
    Weight = 7
    PSUM_in = 0


Expected:

    PSUM_out = 0 + (3 x 7)

    PSUM_out = 21


This verifies the basic multiplication path without an existing accumulated value.


## 15. Non-Zero Partial-Sum Test

The PE should also be tested with a non-zero partial sum.

Example:

    Activation = 3
    Weight = 7
    PSUM_in = 10


Expected:

    PSUM_out = 10 + 21

    PSUM_out = 31


This confirms that the accumulator correctly includes the incoming partial sum.


## 16. Multiple MAC Test

The PE should be tested across multiple clock cycles.

Example:

    Cycle 1:
        Activation = 2
        Weight = 3
        PSUM_in = 0

    Product:
        2 x 3 = 6


    Cycle 2:
        Activation = 4
        Weight = 3

    Product:
        4 x 3 = 12


The accumulated result should progress according to the implemented PE timing.


## 17. Expected Result Generation

Expected results should be calculated independently.

For a simple MAC:

    Expected =
        PSUM_in + (Activation x Weight)


For matrix multiplication:

    Expected C =
        A x B


The expected result must not be generated by copying the same RTL calculation being tested.


## 18. Scoreboard Concept

A scoreboard can be used to compare expected and actual values.

Conceptually:

    Input Data
        |
        +----------------------+
        |                      |
        v                      v
    Reference Model          RTL
        |                      |
        v                      v
    Expected Result        Actual Result
        |                      |
        +----------+-----------+
                   |
                   v
                Compare
                   |
              +----+----+
              |         |
             PASS      FAIL


The scoreboard should report the exact location of any mismatch.


## 19. PASS Condition

A test should be considered PASS only when all required outputs match the expected values.

For example:

    Expected = 30

    Actual = 30

Therefore:

    PASS


## 20. FAIL Condition

A test should fail when:

    Expected != Actual


Example:

    Expected = 30

    Actual = 28

Therefore:

    FAIL


The testbench should provide enough information to identify the failing input and cycle.


## 21. Error Reporting

A useful verification environment should report:

- Test case name
- Cycle number
- Expected value
- Actual value
- PE or output location
- Relevant input values


Example format:

    TEST: PE_MAC_TEST

    STATUS: FAIL

    Cycle: 15

    Expected PSUM: 30

    Actual PSUM: 28


This makes debugging easier.


## 22. Waveform Verification

Waveforms are used to investigate signal-level behavior.

Important signals include:

    Clock
    Reset
    Weight Load Enable
    Weight Input
    Activation Input
    Activation Output
    Partial Sum Input
    Partial Sum Output


The exact signal names depend on the implemented Verilog module.


## 23. What to Check in the Waveform

During weight loading:

    Weight input
        |
        v
    Weight load enable
        |
        v
    Stored weight


During computation:

    Activation
        |
        v
    Multiply
        |
        v
    Product
        |
        v
    Partial Sum
        |
        v
    Updated Partial Sum


During forwarding:

    Activation Input
        |
        v
    PE
        |
        v
    Activation Output


## 24. Cycle-Level Verification

The verification must account for clock cycles.

A value may not appear immediately after applying an input.

Therefore, the testbench should determine:

    Input Cycle

    Processing Cycle

    Output Cycle


The expected output should be compared at the correct cycle.


## 25. Timing Alignment

Timing alignment is especially important in a systolic array.

For example:

    Activation A0
        |
        v
    PE00
        |
        v
    PE01
        |
        v
    PE02


The activation reaches each PE at a different cycle.

Therefore, the verification environment must account for propagation delays in terms of clock cycles.


## 26. Multi-PE Verification

After the PE is verified, multiple PEs should be connected.

Initial integration can use two PEs.

Conceptually:

    Activation
        |
        v
      PE0
        |
        v
      PE1
        |
        v
      Output


The verification should confirm:

- PE0 computation
- PE0 activation forwarding
- PE1 activation reception
- PE1 computation
- Correct timing between the PEs


## 27. Two-PE Test

Example:

    PE0:
        Weight = W0

    PE1:
        Weight = W1


Activation enters PE0.

Then:

    Activation -> PE0 -> PE1


Each PE performs its own MAC operation.

The expected outputs should be calculated independently.


## 28. Four-PE Chain Verification

The next stage is a four-PE chain.

Conceptually:

    Input
      |
      v
    PE0
      |
      v
    PE1
      |
      v
    PE2
      |
      v
    PE3


This test helps verify longer activation propagation paths before moving to the complete 4x4 array.


## 29. 4x4 Array Verification

The final architecture contains:

    4 x 4 = 16 PEs


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

The complete array must be verified for:

- PE connectivity
- Weight mapping
- Activation propagation
- Partial-sum propagation
- Output mapping
- Timing


## 30. Matrix Multiplication Verification

The complete matrix multiplication is:

    C = A x B


For a 4x4 matrix:

    Cij =
        Ai0B0j
        + Ai1B1j
        + Ai2B2j
        + Ai3B3j


The testbench should calculate the expected matrix independently and compare it with the RTL result.


## 31. Identity Matrix Test

The identity matrix is one of the first recommended matrix-level tests.

If:

    B = I


then:

    A x I = A


Therefore, if:

    A =

    | 1  2  3  4 |
    | 5  6  7  8 |
    | 9 10 11 12 |
    |13 14 15 16 |


the expected result is:

    C =

    | 1  2  3  4 |
    | 5  6  7  8 |
    | 9 10 11 12 |
    |13 14 15 16 |


This is an excellent sanity test.


## 32. Zero Matrix Test

If:

    B = 0


then:

    A x B = 0


Every output should therefore be zero, assuming the initial partial sums and reset state are correctly controlled.


## 33. Small-Value Test

Use small numerical values first.

Example:

    A =

    | 1  2 |
    | 3  4 |


    B =

    | 5  6 |
    | 7  8 |


The expected result is:

    C00 = 1x5 + 2x7
        = 5 + 14
        = 19


    C01 = 1x6 + 2x8
        = 6 + 16
        = 22


    C10 = 3x5 + 4x7
        = 15 + 28
        = 43


    C11 = 3x6 + 4x8
        = 18 + 32
        = 50


Expected:

    C =

    | 19 22 |
    | 43 50 |


This type of test is useful during early integration.


## 34. Increasing-Value Test

A deterministic increasing-value matrix can be used.

Example:

    A =

    |  1  2  3  4 |
    |  5  6  7  8 |
    |  9 10 11 12 |
    | 13 14 15 16 |


and:

    B =

    | 16 15 14 13 |
    | 12 11 10  9 |
    |  8  7  6  5 |
    |  4  3  2  1 |


An independent reference calculation should be used to determine the expected output.


## 35. Repeated-Value Test

Repeated values can expose data duplication and routing problems.

Example:

    A =

    | 1 1 1 1 |
    | 2 2 2 2 |
    | 3 3 3 3 |
    | 4 4 4 4 |


This makes it easier to identify whether data is arriving at the correct PE.


## 36. Boundary Testing

Boundary testing should include:

    Minimum supported value

    Maximum supported value

    Zero

    One


These tests help identify:

- Overflow
- Underflow
- Width errors
- Signedness errors
- Arithmetic errors


The exact boundary values must be derived from the actual Verilog signal widths.


## 37. Signedness Verification

The testbench must verify whether the design operates with:

    Signed arithmetic

or:

    Unsigned arithmetic


The testbench should use values appropriate to the RTL declaration.

Signed arithmetic should only be tested if signed operands are intentionally supported by the design.


## 38. Overflow Testing

The accumulator should be tested with values that approach the supported numerical limit.

The test should determine whether:

- The result remains correct
- Overflow occurs as expected
- Overflow behavior is documented


If saturation or overflow protection is not implemented, the behavior must be clearly documented rather than assumed.


## 39. Reset Between Operations

The design should be tested after reset.

Test sequence:

    Reset
      |
      v
    Load Weights
      |
      v
    Compute
      |
      v
    Check Result


Then repeat:

    Reset
      |
      v
    Load New Weights
      |
      v
    Compute
      |
      v
    Check New Result


This checks that previous state does not incorrectly affect the next operation.


## 40. Back-to-Back Operation Test

After basic functionality is verified, multiple matrix operations should be tested sequentially.

Conceptually:

    Matrix A1 x Matrix B1
             |
             v
           C1
             |
             v
    Matrix A2 x Matrix B2
             |
             v
           C2


The outputs must correspond to the correct input matrices.


## 41. Invalid Input Testing

The testbench should eventually investigate behavior when inputs are not valid.

Depending on the implemented interface, this may include:

- Invalid control signals
- Missing valid signals
- Inputs during reset
- Weight load disabled
- Unexpected input changes


The expected behavior must be based on the actual RTL specification.


## 42. Verification of Weight Stationarity

The testbench should confirm that weights remain unchanged during the intended computation period.

Conceptually:

    Weight Load
        |
        v
    Weight Stored
        |
        v
    Computation
        |
        v
    Weight remains stable


The waveform can be used to verify this behavior.


## 43. Verification of Activation Propagation

Activation propagation should be checked across multiple PEs.

For example:

    Cycle N:
        PE00 receives A0

    Cycle N+1:
        PE01 receives A0

    Cycle N+2:
        PE02 receives A0

The exact cycle offsets must be determined from the implemented architecture.


## 44. Verification of Partial-Sum Propagation

Where the architecture uses partial-sum movement, the testbench must verify that the accumulated value reaches the correct PE at the correct cycle.

The sequence is conceptually:

    Initial PSUM
         |
         v
    PE computation
         |
         v
    Updated PSUM
         |
         v
    Next PE
         |
         v
    Further accumulation


## 45. Verification of Output Ordering

The testbench must confirm that each output corresponds to the correct matrix element.

For example:

    C00 -> Output corresponding to row 0, column 0

    C01 -> Output corresponding to row 0, column 1

    ...

    C33 -> Output corresponding to row 3, column 3


Incorrect output ordering is an integration error even if every individual value is numerically correct.


## 46. Latency Measurement

The verification environment should measure the number of clock cycles required to obtain valid output.

Latency should be measured from a clearly defined starting event.

For example:

    Start:
        First valid input accepted

    End:
        Required output becomes valid


The exact latency should be obtained from simulation rather than assumed.


## 47. Throughput Measurement

After the array reaches steady-state operation, throughput can be evaluated.

Throughput can be described as the amount of completed computation per unit time.

The testbench should determine how frequently valid results can be produced once the pipeline is full.


## 48. Functional Coverage

The verification process should eventually track whether important scenarios have been exercised.

Coverage targets include:

    Weight Loading
        YES / NO

    Weight Retention
        YES / NO

    Activation Forwarding
        YES / NO

    MAC Operation
        YES / NO

    Zero Input
        YES / NO

    Non-Zero Input
        YES / NO

    Boundary Values
        YES / NO

    Multi-PE Operation
        YES / NO

    4x4 Matrix Multiplication
        YES / NO

    Back-to-Back Operation
        YES / NO


## 49. Regression Testing

Whenever RTL changes are made, previously passing tests should be rerun.

Regression flow:

    RTL Change
        |
        v
    Compile
        |
        v
    Run Existing Tests
        |
        v
    Run New Tests
        |
        v
    Compare Results
        |
        v
    PASS / FAIL


This helps prevent new changes from breaking previously verified functionality.


## 50. Test Naming Convention

A consistent test naming scheme should be used.

Examples:

    tb_pe_reset

    tb_pe_weight_load

    tb_pe_mac

    tb_pe_forward

    tb_two_pe

    tb_array_4x4

    tb_matrix_multiplication

    tb_boundary

    tb_regression


The actual filenames may be adjusted according to the project structure.


## 51. Simulation Artifacts

Important verification artifacts should be stored separately from RTL.

Examples include:

    Simulation logs

    Waveform files

    Compiled simulation outputs

    Test reports


The exact file formats depend on the selected simulator.


## 52. Waveform Storage

Waveforms should be stored in:

    wave/


This keeps waveform artifacts separate from RTL and testbench source files.


## 53. Simulation Output

Simulation logs and generated simulation files should be stored in:

    sim/


This helps keep the project repository organized.


## 54. Verification Completion Criteria

The PE verification milestone can be considered complete when:

    Weight Loading
        PASS

    MAC Operation
        PASS

    Activation Forwarding
        PASS

    Partial-Sum Behavior
        PASS

    Reset Behavior
        PASS


The complete accelerator verification milestone requires:

    4x4 Integration
        PASS

    Matrix Multiplication
        PASS

    Output Ordering
        PASS

    Timing Verification
        PASS

    Boundary Tests
        PASS

    Regression Tests
        PASS


## 55. Current Verification Status

Current project status:

    PE RTL
        COMPLETED

    PE Testbench
        COMPLETED

    PE Simulation
        COMPLETED

    MAC Operation
        VERIFIED

    Weight Loading
        VERIFIED

    Activation Forwarding
        VERIFIED


The following stages remain for complete accelerator verification:

    Multi-PE Integration
        IN PROGRESS / NEXT STAGE

    4x4 Array
        PLANNED

    Matrix Multiplication
        PLANNED

    Full Regression
        PLANNED


Only features that have actually been implemented and verified should be marked as PASS.


## 56. Debugging Methodology

When a test fails, debugging should proceed from the lowest level.

Recommended flow:

    Test Failure
        |
        v
    Check Testbench Inputs
        |
        v
    Check Expected Result
        |
        v
    Check Clock / Reset
        |
        v
    Check PE Inputs
        |
        v
    Check PE Outputs
        |
        v
    Check Data Forwarding
        |
        v
    Check PE Connections
        |
        v
    Check Array Mapping
        |
        v
    Check Output Collection


This prevents complex integration problems from being incorrectly attributed to arithmetic logic.


## 57. Common Failure Modes

Potential failure modes include:

### Incorrect Weight Loading

The wrong weight is stored in a PE.

### Incorrect Activation Timing

Activation reaches a PE at the wrong cycle.

### Incorrect Partial Sum

The PE does not accumulate the incoming partial sum correctly.

### Incorrect PE Connection

The output of one PE is connected to the wrong neighboring PE.

### Incorrect Output Mapping

The computed result is correct but assigned to the wrong matrix position.

### Width Mismatch

The arithmetic result is truncated.

### Signedness Error

Signed and unsigned operands are interpreted incorrectly.

### Reset Error

Old state remains after reset.


## 58. Verification Checklist

Before declaring the complete project verified:

    [ ] PE reset verified

    [ ] PE weight loading verified

    [ ] PE weight retention verified

    [ ] PE multiplication verified

    [ ] PE partial-sum accumulation verified

    [ ] PE activation forwarding verified

    [ ] Two-PE integration verified

    [ ] Multi-PE propagation verified

    [ ] 4x4 array integration verified

    [ ] Weight mapping verified

    [ ] Activation scheduling verified

    [ ] Partial-sum routing verified

    [ ] Output mapping verified

    [ ] Identity matrix test passed

    [ ] Zero matrix test passed

    [ ] Small-value test passed

    [ ] Increasing-value test passed

    [ ] Boundary tests passed

    [ ] Overflow behavior evaluated

    [ ] Reset between operations verified

    [ ] Back-to-back operations verified

    [ ] Latency measured

    [ ] Throughput measured

    [ ] Regression tests passed


## 59. Verification Deliverables

The final verification stage should produce:

    1. Verilog testbenches

    2. Simulation logs

    3. Waveform files

    4. Expected results

    5. Actual results

    6. PASS/FAIL reports

    7. Verification summary

    8. Matrix multiplication test results


These artifacts should be linked or referenced from the GitHub documentation where appropriate.


## 60. Final Verification Flow

The complete verification methodology is:

    RTL Design
        |
        v
    Compile
        |
        v
    Reset Test
        |
        v
    PE Functional Tests
        |
        v
    PE Waveform Analysis
        |
        v
    PE PASS
        |
        v
    Multi-PE Integration
        |
        v
    Communication Verification
        |
        v
    4x4 Array Integration
        |
        v
    Matrix Multiplication
        |
        v
    Independent Reference Comparison
        |
        v
    Boundary Tests
        |
        v
    Regression
        |
        v
    Final Verification Report


## 61. Summary

The verification strategy follows a bottom-up methodology.

The Processing Element is verified first because it is the fundamental computational unit.

The verified PE is then integrated into progressively larger structures until the complete 4x4 systolic array is verified.

The primary PE computation is:

    PSUM_out =
        PSUM_in + (Activation x Weight)


The primary matrix operation is:

    C = A x B


Verification must confirm both numerical correctness and cycle-level behavior.

The final verification objective is to demonstrate that the 4x4 weight-stationary systolic array correctly performs matrix multiplication using 16 Processing Elements while maintaining the intended weight-stationary dataflow and activation propagation behavior.
