# Testbench Documentation

## 1. Overview

This document describes the verification testbench used for the 4x4 weight-stationary systolic array accelerator.

The testbench is developed separately from the synthesizable Verilog RTL.

The primary purpose of the testbench is to provide controlled input stimulus, observe the design outputs, and determine whether the hardware behaves according to the intended architecture.

The verification methodology follows a bottom-up approach:

    Processing Element
          |
          v
    PE Testbench
          |
          v
    PE Simulation
          |
          v
    Multi-PE Testbench
          |
          v
    4x4 Array Testbench
          |
          v
    Matrix Multiplication Verification


## 2. HDL Used

The testbench is written using:

    Verilog


The project does not use SystemVerilog.

All testbench examples and implementation should remain compatible with the Verilog-based design.


## 3. Testbench Purpose

The testbench is responsible for verifying:

    Reset behavior

    Weight loading

    Weight storage

    Activation input

    Activation forwarding

    Multiplication

    Partial-sum accumulation

    MAC operation

    PE timing

    PE output behavior


For the complete array, the testbench will additionally verify:

    PE connectivity

    Weight mapping

    Activation scheduling

    Partial-sum propagation

    Output mapping

    Matrix multiplication


## 4. Testbench Architecture

The basic PE-level verification environment can be represented as:

                    +----------------+
                    |   Testbench    |
                    +----------------+
                       |    |    |
                       |    |    |
                       v    v    v
                    Inputs  Clock Reset
                       |
                       v
                 +-------------+
                 |     PE      |
                 |    RTL      |
                 +-------------+
                    |       |
                    v       v
              Activation  PSUM
                 Output    Output


The testbench generates the required stimulus and monitors the outputs.


## 5. Device Under Test

The hardware module being verified is called the:

    Device Under Test


For PE-level verification:

    DUT = Processing Element


The DUT is instantiated inside the Verilog testbench.


Conceptually:

    testbench
        |
        +---- DUT
              |
              +---- PE


The exact module name must match the actual RTL.


## 6. Testbench Components

The basic testbench consists of:

    1. Clock Generation

    2. Reset Generation

    3. Input Stimulus

    4. DUT Instance

    5. Output Monitoring

    6. Result Checking

    7. Simulation Termination


Conceptually:

    Clock Generator
          |
          v
    Reset Generator
          |
          v
    Stimulus Generator
          |
          v
        DUT
          |
          v
    Output Monitor
          |
          v
    Result Checker


## 7. Clock Generation

The testbench generates the clock required by the synchronous PE.

Conceptually:

    Clock
      |
      +---- HIGH
      |
      +---- LOW
      |
      +---- HIGH
      |
      +---- LOW


The clock period must match the timing assumptions used during simulation.

The exact clock period is determined by the testbench implementation.


## 8. Clock Operation

The testbench continuously generates clock transitions during simulation.

Conceptually:

    initial
        begin
            clock = 0;
            forever
                clock = ~clock;
        end


The exact implementation may use a delay corresponding to the desired clock period.


## 9. Reset Generation

The testbench applies reset before normal operation.

General sequence:

    Simulation Start
          |
          v
        Reset
          |
          v
    Clock Cycles
          |
          v
    Release Reset
          |
          v
    Normal Operation


This ensures that the DUT begins from a known state.


## 10. Input Stimulus

The testbench applies input values to the DUT.

Important inputs include:

    Weight

    Weight Load Control

    Activation

    Partial Sum

    Reset

    Clock


The exact signal names depend on the actual PE RTL.


## 11. Weight Loading Sequence

The testbench first loads a known weight.

Conceptual sequence:

    Reset
      |
      v
    Apply Weight
      |
      v
    Assert Weight Load
      |
      v
    Clock Edge
      |
      v
    Weight Stored
      |
      v
    Disable Weight Load


Example:

    Weight = 5


After the appropriate clock edge:

    Stored Weight = 5


The testbench then uses the stored weight for computation.


## 12. Weight Storage Test

After loading the weight, the testbench performs a computation without loading a new weight.

This verifies that the PE retains the previously loaded weight.

Example:

    Loaded Weight = 5

    Activation = 4

    PSUM = 0


Expected:

    0 + (4 x 5)
        = 20


The result confirms that the stored weight was used.


## 13. Activation Stimulus

The testbench applies a known activation value.

Example:

    Activation = 4


The activation is supplied relative to the clock according to the intended timing of the PE.


## 14. Partial-Sum Stimulus

The testbench also provides a partial-sum input.

Example:

    PSUM_in = 10


The PE should calculate:

    PSUM_out =
        10 + (4 x 5)

    PSUM_out = 30


The testbench checks the result against this expected value.


## 15. MAC Test

The main functional test is the MAC operation.

Example:

    Weight = 5

    Activation = 4

    PSUM_in = 10


Expected:

    PSUM_out =
        10 + (4 x 5)

    PSUM_out =
        10 + 20

    PSUM_out = 30


The testbench verifies that the actual output matches the expected result.


## 16. Expected Result Calculation

Expected results should be calculated independently inside the verification environment.

For a MAC:

    Expected =
        PSUM_in + (Activation x Weight)


The expected value is then compared with the DUT output.


## 17. Output Checking

The testbench observes the DUT outputs.

Important outputs include:

    Activation Output

    Partial Sum Output


The output is checked after the appropriate clock event.

The comparison must account for the registered behavior of the DUT.


## 18. Basic Comparison

Conceptually:

    if (actual == expected)
        PASS
    else
        FAIL


A useful failure report should include:

    Expected Value

    Actual Value

    Test Name

    Simulation Time


## 19. PASS Message

A successful test may report:

    --------------------------------
    TEST PASSED
    --------------------------------

    Expected : 30
    Actual   : 30

    --------------------------------


The exact formatting can be changed according to the testbench style.


## 20. FAIL Message

A failed test should provide enough information for debugging.

Example:

    --------------------------------
    TEST FAILED
    --------------------------------

    Expected : 30
    Actual   : 28

    --------------------------------


Additional information such as input values and simulation time should be included where useful.


## 21. Test Case 1: Reset

Purpose:

    Verify that the PE enters the expected initial state.


Procedure:

    1. Start simulation.

    2. Assert reset.

    3. Apply clock cycles.

    4. Check reset-state outputs.

    5. Release reset.


Expected:

    DUT enters the reset state defined by the RTL.


Status:

    VERIFIED


## 22. Test Case 2: Weight Loading

Purpose:

    Verify that a weight can be loaded into the PE.


Procedure:

    1. Reset the PE.

    2. Apply weight value.

    3. Assert weight-loading control.

    4. Apply clock edge.

    5. Disable weight-loading control.


Expected:

    The supplied weight is stored inside the PE.


Status:

    VERIFIED


## 23. Test Case 3: Weight Retention

Purpose:

    Verify that the loaded weight remains available during computation.


Example:

    Weight = 5

    Activation = 4

    PSUM = 0


Expected:

    PSUM_out = 20


Status:

    VERIFIED


## 24. Test Case 4: Basic Multiplication

Purpose:

    Verify multiplication between activation and stored weight.


Example:

    Activation = 3

    Weight = 7


Expected:

    Product = 21


The product contributes to the partial-sum calculation.


Status:

    VERIFIED


## 25. Test Case 5: Zero Partial Sum

Purpose:

    Verify the multiplication path with no previous accumulation.


Example:

    Activation = 3

    Weight = 7

    PSUM_in = 0


Expected:

    PSUM_out = 21


Status:

    VERIFIED


## 26. Test Case 6: Non-Zero Partial Sum

Purpose:

    Verify accumulation with an existing partial sum.


Example:

    Activation = 3

    Weight = 7

    PSUM_in = 10


Expected:

    PSUM_out =
        10 + 21

    PSUM_out = 31


Status:

    VERIFIED


## 27. Test Case 7: Activation Forwarding

Purpose:

    Verify that activation data is forwarded according to the PE architecture.


Conceptually:

    Activation In
         |
         v
        PE
         |
         v
    Activation Out


The output activation should match the expected forwarded value at the expected cycle.


Status:

    VERIFIED


## 28. Test Case 8: Multiple MAC Operations

Purpose:

    Verify repeated MAC operations.

Example:

    Cycle 1:

        Activation = 2

        Weight = 3

        PSUM = 0


    Product:

        2 x 3 = 6


    Cycle 2:

        Activation = 4

        Weight = 3

        PSUM = 6


    Product:

        4 x 3 = 12


    Accumulated Result:

        6 + 12 = 18


The exact cycle behavior must match the implemented RTL.


## 29. Testbench Timing

The testbench must apply stimulus at appropriate times relative to the clock.

A typical synchronous verification sequence is:

    Apply Inputs
        |
        v
    Wait for Clock Edge
        |
        v
    DUT Processes Data
        |
        v
    Wait for Output
        |
        v
    Compare Result


The exact number of cycles depends on the RTL implementation.


## 30. Avoiding Race Conditions

The testbench should avoid changing inputs at the exact moment the DUT samples them unless that behavior is intentional.

Inputs should be applied with sufficient timing margin relative to the active clock edge.

This helps ensure deterministic simulation results.


## 31. Simulation Time

The testbench may display simulation time when reporting results.

Example:

    $display("Time = %0t", $time);


Such simulation-only statements belong in the testbench and not in synthesizable RTL.


## 32. Testbench Organization

A clean testbench can be organized into:

    Clock Generation

    Reset Task / Procedure

    Weight Loading Task / Procedure

    MAC Test

    Forwarding Test

    Result Checking

    Main Test Sequence


This organization improves readability and reuse.


## 33. Test Tasks

Reusable Verilog tasks can simplify repeated tests.

Conceptually:

    task load_weight;

        input weight_value;

        begin

            ...

        end

    endtask


A MAC test task can similarly apply:

    Activation

    Partial Sum

    Expected Result


The exact implementation should match the actual interface.


## 34. Main Test Sequence

The main simulation sequence should follow a controlled order.

Conceptually:

    Start
      |
      v
    Initialize
      |
      v
    Reset
      |
      v
    Load Weight
      |
      v
    Test MAC
      |
      v
    Test Forwarding
      |
      v
    Test Multiple Operations
      |
      v
    Report Results
      |
      v
    Finish Simulation


## 35. Testbench for Multi-PE Integration

After PE verification, a higher-level testbench will be required.

Conceptually:

    +----------------------+
    |      Testbench       |
    +----------+-----------+
               |
               v
    +----------------------+
    |      PE0 -> PE1      |
    +----------------------+
               |
               v
             Output


The testbench must verify both computation and inter-PE communication.


## 36. Two-PE Testbench

The two-PE testbench should verify:

    PE0 activation input

    PE0 activation output

    PE1 activation input

    PE0 computation

    PE1 computation

    Correct timing


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


The output of PE0 should be connected to the input of PE1 according to the intended architecture.


## 37. Four-PE Testbench

The next integration test can use:

    PE0 -> PE1 -> PE2 -> PE3


Purpose:

    Verify activation propagation across a longer chain.


This helps identify:

    Timing Errors

    Connection Errors

    Data Loss

    Incorrect Forwarding


before testing the full 4x4 array.


## 38. 4x4 Array Testbench

The final array-level testbench will instantiate the complete accelerator.

Conceptually:

    Testbench
        |
        v
    +----------------------+
    |      4x4 Array       |
    |                      |
    | 16 Processing        |
    | Elements             |
    +----------------------+
        |
        v
      Outputs


The testbench will provide:

    Matrix A

    Matrix B

    Weight-loading sequence

    Activation schedule

    Control signals


and will check:

    Matrix C


## 39. Matrix Reference Calculation

The array-level testbench should calculate the expected matrix independently.

For:

    C = A x B


each element is:

    Cij =
        Ai0B0j
        + Ai1B1j
        + Ai2B2j
        + Ai3B3j


The expected matrix is compared against the RTL output.


## 40. Example Matrix Test

Example:

    A =

    | 1 2 |
    | 3 4 |


    B =

    | 5 6 |
    | 7 8 |


Expected:

    C =

    | 19 22 |
    | 43 50 |


because:

    C00 =
        1x5 + 2x7
        = 19


    C01 =
        1x6 + 2x8
        = 22


    C10 =
        3x5 + 4x7
        = 43


    C11 =
        3x6 + 4x8
        = 50


This type of test is useful for initial matrix-level verification.


## 41. 4x4 Identity Test

For the complete 4x4 array:

    A x I = A


A deterministic 4x4 matrix can be used as the input.

Expected output:

    Output = Input


This test is particularly useful for identifying:

    Incorrect Weight Placement

    Incorrect Activation Ordering

    Incorrect Output Mapping


## 42. 4x4 Zero Test

For:

    A x 0 = 0


all output values should be zero, assuming correct initialization and operation.


This test verifies the handling of zero-valued operands.


## 43. Boundary Test

Boundary tests should use the minimum and maximum supported values based on the actual Verilog signal widths.

Purpose:

    Detect overflow

    Detect truncation

    Detect width mismatch

    Detect signedness problems


The exact values should be generated from the implemented widths.


## 44. Randomized Testing

After deterministic tests are working, randomized input testing can be added.

Conceptually:

    Random A
    Random B
       |
       v
    Reference Model
       |
       v
    Expected C


and:

    Random A
    Random B
       |
       v
    RTL Array
       |
       v
    Actual C


The two results are compared.


Random testing can expose data-routing and timing problems that may not appear in simple deterministic tests.


## 45. Directed Testing

Directed tests should be retained even after randomized testing is added.

Important directed tests include:

    Zero

    One

    Identity

    Repeated Values

    Increasing Values

    Maximum Supported Values


Directed tests make debugging easier because the expected behavior is predictable.


## 46. Scoreboard

A scoreboard can be introduced at the array level.

Conceptually:

    Inputs
      |
      +-------------------+
      |                   |
      v                   v
 Reference Model        RTL
      |                   |
      v                   v
 Expected Result      Actual Result
      |                   |
      +---------+---------+
                |
                v
             Compare
                |
           +----+----+
           |         |
          PASS      FAIL


The scoreboard should compare every expected output against its corresponding RTL output.


## 47. Output Mapping Verification

The testbench must verify that each output corresponds to the correct matrix location.

For example:

    Output C00
        -> Matrix[0][0]

    Output C01
        -> Matrix[0][1]

    ...

    Output C33
        -> Matrix[3][3]


A numerically correct result at the wrong output position is still a failure.


## 48. Latency Measurement

The testbench should measure latency.

The starting point should be clearly defined.

Example:

    Start:
        First valid input accepted


    End:
        Required output becomes valid


Then:

    Latency =
        Output Cycle - Input Cycle


The exact value should be obtained from simulation.


## 49. Throughput Measurement

The testbench can also measure throughput.

Once the systolic array reaches steady-state operation, the number of valid outputs generated per unit time can be measured.

The result should be based on actual simulation behavior.


## 50. Waveform Dumping

The testbench can generate waveform files for debugging.

For simulators supporting VCD:

    $dumpfile("waveform.vcd");

    $dumpvars(0, testbench);


These commands belong in the testbench and are simulation-specific.


## 51. Waveform Analysis

Important signals to inspect include:

    Clock

    Reset

    Weight Load

    Weight Input

    Activation Input

    Activation Output

    Partial Sum Input

    Partial Sum Output


For the complete array, additional internal PE signals may be inspected.


## 52. Simulation Completion

The testbench should explicitly terminate the simulation after all tests complete.

Conceptually:

    All Tests
        |
        v
    Final PASS/FAIL
        |
        v
    Simulation Finish


The exact Verilog simulation command depends on the simulator.


## 53. Current Testbench Status

Current status:

    PE Testbench:
        COMPLETED


    PE Simulation:
        PASSED


    PE MAC Verification:
        PASSED


    Weight Loading:
        PASSED


    Activation Forwarding:
        PASSED


The following testbenches remain to be developed:

    Two-PE Testbench

    Four-PE Testbench

    4x4 Array Testbench

    Matrix-Level Verification


## 54. Current Verified Tests

The currently verified PE-level tests include:

    [x] Reset

    [x] Weight Loading

    [x] Weight Storage

    [x] Basic Multiplication

    [x] Zero Partial Sum

    [x] Non-Zero Partial Sum

    [x] Activation Forwarding

    [x] MAC Operation


The exact testbench source should remain the authoritative source for the exact stimulus values and timing.


## 55. Testbench Debugging Procedure

When a test fails:

    1. Check the test inputs.

    2. Check reset behavior.

    3. Check clock timing.

    4. Check weight loading.

    5. Check activation input.

    6. Check partial-sum input.

    7. Check expected result.

    8. Check actual result.

    9. Open waveform.

    10. Trace the failing signal cycle by cycle.


This approach helps isolate whether the problem is in:

    Testbench

    PE RTL

    Integration

    Timing


## 56. Testbench Quality Requirements

The final testbench should be:

    Reproducible

    Deterministic for directed tests

    Easy to understand

    Easy to extend

    Capable of reporting failures

    Independent of the RTL implementation for expected results


The testbench should not simply reproduce the internal RTL calculation and assume the DUT is correct.


## 57. Reproducibility

Another engineer should be able to run the testbench using the documented simulation procedure.

The repository should contain:

    RTL

    Testbench

    Simulation Instructions

    Expected Results

    Waveform Instructions


Any simulator-specific setup should be documented.


## 58. Regression Testbench

As the project grows, a regression flow should run multiple tests automatically.

Conceptually:

    Test 1
       |
       v
    Test 2
       |
       v
    Test 3
       |
       v
    Test 4
       |
       v
    Matrix Tests
       |
       v
    Final Summary


The regression should report:

    Total Tests

    Passed Tests

    Failed Tests


## 59. Example Regression Summary

Example format:

    =======================================
             REGRESSION SUMMARY
    =======================================

    PE Reset Test              PASS

    PE Weight Load Test        PASS

    PE MAC Test                PASS

    PE Forwarding Test         PASS

    Two-PE Test                PASS

    4x4 Matrix Test            PASS

    =======================================

    Total Tests:               6

    Passed:                    6

    Failed:                    0

    Overall Result:            PASS

    =======================================


This is an example format only.

Actual results must be generated from the real simulation.


## 60. Testbench and RTL Separation

The repository maintains a clear separation between:

    Synthesizable RTL

and:

    Simulation/Testbench Code


RTL:

    rtl/


Testbench:

    tb/


Simulation outputs:

    sim/


Waveforms:

    wave/


Documentation:

    docs/


This organization follows a professional hardware-development workflow.


## 61. Current Verification Milestone

The current completed milestone is:

    PE RTL
        +
    PE Testbench
        +
    PE Simulation
        =
    VERIFIED PE


This verified PE is the foundation for the next integration stage.


## 62. Next Testbench Milestone

The immediate next verification target is:

    Two-PE Integration


The intended sequence is:

    PE0
      |
      v
    PE1


After successful verification:

    Four-PE Chain
        |
        v
    Complete 4x4 Array
        |
        v
    Matrix Multiplication


## 63. Final Testbench Objective

The final testbench must demonstrate that:

    C = A x B


is correctly computed by the 4x4 systolic array.

It must verify:

    Numerical Correctness

    Data Movement

    Weight Stationarity

    Activation Propagation

    Partial-Sum Accumulation

    Output Ordering

    Cycle-Level Timing


## 64. Summary

The Verilog testbench is a critical part of the accelerator development flow.

The Processing Element testbench has already been implemented and used to verify:

    Weight Loading

    Weight Storage

    Multiplication

    Partial-Sum Accumulation

    MAC Operation

    Activation Forwarding


The next stage is to extend verification from the individual PE to multiple connected PEs and finally to the complete 4x4 weight-stationary systolic array.

The final verification environment will compare the RTL matrix multiplication result against an independently calculated reference result and will provide simulation and waveform evidence for the completed accelerator.
