# Simulation and Verification Procedure

## 1. Overview

This document describes the simulation and verification procedure for the 4x4 weight-stationary systolic array accelerator.

The project uses Verilog RTL and a separate Verilog testbench.

The verification process is performed progressively, beginning with the individual Processing Element (PE) and continuing toward the complete 4x4 systolic array.

The verification flow is:

    Verilog RTL
        |
        v
    Compile
        |
        v
    Elaborate
        |
        v
    Simulate
        |
        v
    Observe Outputs
        |
        v
    Analyze Waveforms
        |
        v
    Compare Expected vs Actual
        |
        v
    PASS / FAIL


## 2. Verification Philosophy

The project follows a bottom-up verification methodology.

The individual PE is verified before integrating multiple PEs.

The verification sequence is:

    Level 1:
        Processing Element


    Level 2:
        Two-PE Integration


    Level 3:
        Four-PE Chain


    Level 4:
        Complete 4x4 Array


    Level 5:
        Matrix Multiplication


    Level 6:
        Regression Testing


This approach makes debugging easier because problems can be isolated at the smallest possible level.


## 3. HDL

The design and testbench use:

    Verilog HDL


SystemVerilog is not used in this project.


## 4. Simulation Directory

Simulation-related files are maintained under:

    sim/


The exact files generated depend on the simulator being used.


Typical simulation outputs may include:

    Compiled simulation files

    Log files

    Console output

    VCD waveform files


Generated files should be kept separate from the RTL source.


## 5. Waveform Directory

Waveform files are maintained under:

    wave/


Waveforms are used for detailed cycle-by-cycle debugging and verification.


Typical waveform file:

    waveform.vcd


The exact waveform filename depends on the testbench implementation.


## 6. Testbench Directory

Testbench source files are maintained under:

    tb/


The testbench is separate from synthesizable RTL.

Conceptually:

    rtl/
        |
        +-- Hardware Design
        

    tb/
        |
        +-- Verification Environment


## 7. PE Simulation

The first simulation target is the Processing Element.

The PE simulation verifies:

    Reset

    Weight Loading

    Weight Storage

    Activation Input

    Multiplication

    Partial-Sum Accumulation

    Activation Forwarding

    MAC Operation


The PE simulation has already been completed successfully.


## 8. PE Verification Flow

The PE verification flow is:

    Start Simulation
          |
          v
        Reset
          |
          v
    Load Weight
          |
          v
    Apply Activation
          |
          v
    Apply Partial Sum
          |
          v
    Clock Edge
          |
          v
    Observe Output
          |
          v
    Compare Expected Result
          |
          v
       PASS / FAIL


## 9. Reset Verification

Reset is applied at the beginning of simulation.

Purpose:

    Establish a known initial state.


General sequence:

    Simulation Start
        |
        v
    Reset Asserted
        |
        v
    Clock
        |
        v
    Reset Released
        |
        v
    Normal Operation


The actual reset behavior must match the implemented Verilog RTL.


## 10. Weight Loading Verification

A known weight is applied to the PE.

Example:

    Weight = 5


The weight-load control is asserted.

After the appropriate clock edge:

    Stored Weight = 5


The weight-load control is then disabled.


The PE is subsequently tested using the stored weight.


## 11. Weight Retention Verification

The PE should retain the loaded weight during normal computation.

Example:

    Loaded Weight = 5


Then:

    Activation = 4

    PSUM = 0


Expected:

    PSUM_out = 4 x 5

    PSUM_out = 20


This verifies that the previously loaded weight is used.


## 12. MAC Verification

The primary PE arithmetic test is:

    PSUM_out =
        PSUM_in + (Activation x Weight)


Example:

    Activation = 4

    Weight = 5

    PSUM_in = 10


Expected:

    PSUM_out =
        10 + (4 x 5)

    PSUM_out = 30


The simulation must show the expected result at the correct cycle.


## 13. Activation Forwarding Verification

Activation forwarding is verified by observing:

    Activation Input

and:

    Activation Output


The expected behavior is:

    Activation In
         |
         v
        PE
         |
         v
    Activation Out


The output value and timing must match the intended PE implementation.


## 14. Waveform Verification

Waveforms are used to verify internal and external signal behavior.

Important signals include:

    Clock

    Reset

    Weight Load

    Weight Input

    Activation Input

    Activation Output

    Partial Sum Input

    Partial Sum Output


The waveform should be inspected around the clock edges where data is loaded and processed.


## 15. Clock-Cycle Analysis

Hardware verification must consider both:

    Value

and:

    Time


A correct value appearing one cycle too early or too late is still incorrect behavior.

Therefore, every important output should be checked against:

    Expected Value

    Expected Cycle


## 16. Example Cycle Analysis

Assume:

    Weight = 5

    Activation = 4

    PSUM_in = 10


The testbench should identify:

    Cycle N:
        Inputs Applied


    Active Clock Edge:
        DUT Samples Inputs


    Following Cycle / Registered Point:
        Output Observed


The exact cycle relationship must be determined from the implemented RTL.


## 17. Expected vs Actual

Verification should compare:

    Expected Result

against:

    Actual DUT Result


Example:

    Expected = 30

    Actual   = 30


Result:

    PASS


If:

    Expected = 30

    Actual   = 28


Result:

    FAIL


## 18. PE Test Result

The PE-level simulation has successfully verified:

    Weight Loading
        PASS

    Weight Storage
        PASS

    Multiplication
        PASS

    Partial-Sum Accumulation
        PASS

    MAC Operation
        PASS

    Activation Forwarding
        PASS


Therefore:

    PE Verification = PASS


## 19. Simulation Logs

Simulation logs should be reviewed after each test.

A useful log should identify:

    Test Name

    Input Values

    Expected Value

    Actual Value

    PASS / FAIL

    Simulation Time


Example:

    Test: MAC

    Weight: 5

    Activation: 4

    PSUM: 10

    Expected: 30

    Actual: 30

    Result: PASS


## 20. Failure Analysis

When a simulation fails, the following procedure should be followed:

    1. Identify the failing test.

    2. Record expected value.

    3. Record actual value.

    4. Check input stimulus.

    5. Check reset.

    6. Check clock timing.

    7. Check weight loading.

    8. Check activation propagation.

    9. Check partial-sum behavior.

    10. Inspect waveform.

    11. Identify first incorrect cycle.

    12. Correct RTL or testbench.

    13. Re-run simulation.


The first incorrect cycle is often more useful than the final incorrect output when debugging.


## 21. Waveform Debugging Method

The recommended waveform debugging method is:

    Start at Input
        |
        v
    Check Clock
        |
        v
    Check Control
        |
        v
    Check Data
        |
        v
    Check Internal Operation
        |
        v
    Check Output


This should be performed cycle by cycle.


## 22. Multi-PE Verification

After PE-level verification, the next stage is multi-PE verification.

The first target is:

    Two PEs


Conceptually:

    PE0
      |
      v
    PE1


The main purpose is to verify inter-PE communication.


## 23. Two-PE Verification

The two-PE test must verify:

    PE0 Input

    PE0 Computation

    PE0 Activation Output

    PE1 Activation Input

    PE1 Computation

    PE1 Output


It must also verify correct cycle alignment.


## 24. Four-PE Verification

The next stage is:

    PE0 -> PE1 -> PE2 -> PE3


This verifies a longer systolic path.


Important checks:

    No activation loss

    No unexpected duplication

    Correct propagation order

    Correct cycle alignment

    Correct MAC operation


## 25. 4x4 Array Verification

The complete architecture contains:

    16 PEs


Structure:

    +------+------+------+------+
    | PE00 | PE01 | PE02 | PE03 |
    +------+------+------+------+
    | PE10 | PE11 | PE12 | PE13 |
    +------+------+------+------+
    | PE20 | PE21 | PE22 | PE23 |
    +------+------+------+------+
    | PE30 | PE31 | PE32 | PE33 |
    +------+------+------+------+

The array-level testbench must verify the entire structure.


## 26. Array Connectivity Verification

Before testing complete matrix multiplication, the connections between PEs should be verified.

Important connections include:

    Activation paths

    Partial-sum paths

    Weight inputs

    Outputs


A connection error can produce incorrect matrix results even when every individual PE works correctly.


## 27. Weight Mapping Verification

Each PE must receive the correct weight.

The testbench should verify:

    PE00 -> Correct Weight

    PE01 -> Correct Weight

    ...

    PE33 -> Correct Weight


The mapping should correspond to the intended matrix multiplication architecture.


## 28. Activation Scheduling Verification

The testbench must apply activations according to the systolic schedule.

The correct values must arrive at the correct PEs at the correct cycles.

Therefore verification must consider:

    Value

    PE Position

    Clock Cycle


## 29. Partial-Sum Verification

Partial sums must be traced through the computational path.

For each PE:

    PSUM_out =
        PSUM_in + (Activation x Weight)


The testbench should verify that the partial sum entering each PE is correct.


## 30. Output Mapping Verification

Each output must correspond to the correct matrix element.

For a 4x4 result:

    C00

    C01

    C02

    C03

    C10

    C11

    C12

    C13

    C20

    C21

    C22

    C23

    C30

    C31

    C32

    C33


The output ordering must be documented and verified.


## 31. Matrix-Level Verification

The final computational goal is:

    C = A x B


For a 4x4 matrix:

    Cij =
        Ai0B0j
        + Ai1B1j
        + Ai2B2j
        + Ai3B3j


The expected result should be calculated independently.


## 32. Independent Reference Model

The reference model should not simply duplicate the internal PE connection logic.

Instead, it should mathematically calculate:

    C = A x B


The RTL result is then compared against the mathematical reference result.


## 33. Example Matrix Verification

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


The RTL result should match this expected matrix.


## 34. Identity Matrix Test

An identity matrix provides a useful verification case.

For:

    A x I = A


Therefore:

    Expected Output = A


This can verify:

    Weight Mapping

    Activation Scheduling

    Output Mapping


## 35. Zero Matrix Test

For:

    A x 0 = 0


all expected outputs should be zero.


This test is useful for identifying:

    Unwanted Accumulation

    Incorrect Initialization

    Incorrect Data Routing


## 36. One Matrix Test

Using ones provides a simple predictable accumulation pattern.

For example:

    A = all ones

    B = all ones


Each output becomes the sum of four products:

    1x1 + 1x1 + 1x1 + 1x1

    = 4


This is a useful sanity test for the complete array.


## 37. Increasing-Value Test

Matrices containing increasing values can be used to make data-routing errors easier to detect.

Example:

    A:

    1  2  3  4
    5  6  7  8
    9 10 11 12
    13 14 15 16


A corresponding B matrix can be generated.

Because values are unique, incorrect routing is easier to identify.


## 38. Repeated-Value Test

Repeated values can verify that the design does not depend on unique input values.

Example:

    A = all 2

    B = all 3


Each output should be:

    2x3 + 2x3 + 2x3 + 2x3

    = 24


This verifies repeated computation.


## 39. Boundary Testing

Boundary tests should use the minimum and maximum values supported by the actual Verilog data width.

Purpose:

    Detect Overflow

    Detect Truncation

    Detect Width Errors

    Detect Arithmetic Errors


The exact values depend on the actual RTL widths.


## 40. Signedness Verification

If the design uses unsigned arithmetic, verification should use unsigned input values.

If signed arithmetic is implemented later, dedicated negative-value tests should be added.

The testbench must match the actual RTL arithmetic definition.


## 41. Random Testing

After directed tests are passing, randomized matrix tests should be introduced.

Flow:

    Random A
        |
        +----------+
        |          |
        v          v
    Reference     RTL
       Model       |
        |          |
        v          v
    Expected     Actual
        |          |
        +----+-----+
             |
             v
          Compare


Random testing increases coverage.


## 42. Regression Testing

A regression suite should eventually run all important tests automatically.

Example:

    Reset Test

    Weight Load Test

    MAC Test

    Forwarding Test

    Two-PE Test

    Four-PE Test

    Identity Test

    Zero Test

    Ones Test

    Boundary Test

    Random Matrix Test


The regression should produce a final summary.


## 43. Regression Result Format

Example:

    ========================================
              REGRESSION RESULT
    ========================================

    Reset Test              PASS

    Weight Load Test        PASS

    MAC Test                PASS

    Forwarding Test         PASS

    Two-PE Test             PASS

    Four-PE Test            PASS

    Identity Test           PASS

    Zero Test               PASS

    Boundary Test           PASS

    Random Test             PASS

    ========================================

    Total Tests:            10

    Passed:                 10

    Failed:                 0

    Overall Result:         PASS

    ========================================


This is only an example format.

Actual results must come from simulation.


## 44. Verification Coverage

Verification coverage should eventually include:

    Functional Coverage

    Input Coverage

    Output Coverage

    PE Coverage

    Dataflow Coverage

    Boundary Coverage

    Error Coverage


The project should progressively increase coverage as the architecture becomes larger.


## 45. Functional Coverage

Important functional behaviors include:

    Weight Loading

    Weight Retention

    Activation Propagation

    Multiplication

    Accumulation

    Output Generation


Each should have at least one dedicated verification case.


## 46. Dataflow Coverage

Dataflow verification should ensure:

    Activation reaches required PEs.

    Weight remains stationary.

    Partial sums propagate correctly.

    Outputs appear at correct locations.

    Data arrives at expected cycles.


This is particularly important for systolic architectures.


## 47. Cycle-Level Verification

Systolic arrays depend heavily on timing.

Therefore, verification must not only check:

    "Is the result correct?"


It must also check:

    "Did the result appear at the correct time?"


This distinction is important for hardware accelerators.


## 48. Latency Measurement

Latency should be measured from a clearly defined starting event to a clearly defined ending event.

Example:

    Start:
        First valid input accepted


    End:
        Corresponding output becomes valid


Then:

    Latency =
        End Cycle - Start Cycle


The final latency value should be obtained from actual simulation.


## 49. Throughput Measurement

Throughput should be measured after the array reaches steady-state operation.

It can be expressed in terms such as:

    Results per cycle

or:

    Operations per second


The final value depends on:

    Clock Frequency

    Array Utilization

    Pipeline Schedule


## 50. Waveform Evidence

Waveform screenshots may be included in the GitHub documentation as evidence.

Recommended evidence includes:

    PE Reset

    Weight Loading

    MAC Operation

    Activation Forwarding

    Multi-PE Communication

    Complete Array Operation


Only actual simulation waveforms should be included.


## 51. Simulation Reproducibility

The simulation environment should be reproducible.

Another engineer should be able to:

    1. Clone the repository.

    2. Enter the project directory.

    3. Compile the Verilog RTL.

    4. Compile the Verilog testbench.

    5. Run simulation.

    6. Observe PASS/FAIL.

    7. Open waveform output.


The exact commands depend on the selected simulator and should be documented in the project README.


## 52. Simulator Independence

The RTL should ideally remain compatible with commonly used Verilog simulators.

Potential simulators include:

    Icarus Verilog

    Verilator

    Commercial Verilog simulators


The final repository should document the simulator actually used for official verification results.


## 53. Official Verification Environment

For reproducibility, the project should identify:

    Simulator Name

    Simulator Version

    Operating System

    Compilation Command

    Simulation Command


Example format:

    Simulator:
        Icarus Verilog


    Version:
        <actual installed version>


    OS:
        <actual operating system>


The actual values should be filled using the environment in which the official results were generated.


## 54. Simulation Artifacts

The following artifacts may be stored or referenced:

    Simulation Logs

    Waveform Files

    Screenshots

    PASS/FAIL Reports


Large generated files should not unnecessarily clutter the GitHub repository.

Source files and important verification evidence should remain organized.


## 55. Current Verification Status

Current verified milestone:

    PE RTL
        |
        v
    PE Testbench
        |
        v
    PE Simulation
        |
        v
    PE Verification
        |
        v
    PASS


Current pending verification:

    Two-PE Integration

    Four-PE Chain

    Complete 4x4 Array

    Matrix Multiplication

    Regression


## 56. Verification Checklist

Current checklist:

    [x] PE Reset

    [x] PE Weight Loading

    [x] PE Weight Storage

    [x] PE Multiplication

    [x] PE Partial-Sum Accumulation

    [x] PE MAC Operation

    [x] PE Activation Forwarding

    [ ] Two-PE Integration

    [ ] Four-PE Chain

    [ ] 4x4 Array

    [ ] Matrix Multiplication

    [ ] Identity Test

    [ ] Zero Test

    [ ] Boundary Test

    [ ] Random Test

    [ ] Regression

    [ ] Latency Measurement

    [ ] Throughput Measurement


## 57. Debugging Priority

When an integrated test fails, debugging should proceed in this order:

    1. Reset

    2. Clock

    3. Weight Loading

    4. Weight Mapping

    5. Activation Input

    6. Activation Propagation

    7. Partial Sum

    8. PE Connectivity

    9. Output Mapping

    10. Reference Model


This order helps isolate errors from the lowest level upward.


## 58. Verification Best Practices

The following practices should be followed:

    Use known deterministic tests first.

    Verify individual modules before integration.

    Check both value and timing.

    Keep expected-result calculation independent.

    Inspect waveforms when failures occur.

    Record simulation results.

    Maintain regression tests.

    Do not claim verification without simulation evidence.


## 59. Verification Completion Criteria

The complete 4x4 array should only be considered functionally verified after:

    [ ] All 16 PEs are correctly instantiated.

    [ ] All PE connections are verified.

    [ ] Weight mapping is verified.

    [ ] Activation scheduling is verified.

    [ ] Partial-sum propagation is verified.

    [ ] Output mapping is verified.

    [ ] Matrix multiplication matches reference results.

    [ ] Boundary tests pass.

    [ ] Regression tests pass.

    [ ] Timing behavior is verified.


## 60. Final Verification Flow

The final intended verification flow is:

    Architecture
        |
        v
    PE RTL
        |
        v
    PE Testbench
        |
        v
    PE Simulation
        |
        v
    PE PASS
        |
        v
    Two-PE Integration
        |
        v
    Four-PE Integration
        |
        v
    4x4 Array
        |
        v
    Matrix Tests
        |
        v
    Boundary Tests
        |
        v
    Random Tests
        |
        v
    Regression
        |
        v
    Functionally Verified Accelerator


## 61. Important Verification Rule

A test should only be marked PASS when the actual simulation result matches the expected result.

Documentation must not mark planned tests as completed.

For example:

    Correct:

        "PE MAC test passed."


    Incorrect:

        "4x4 matrix multiplication verified."


unless the corresponding 4x4 simulation has actually been executed and passed.


## 62. Current Achievement

The most important completed verification milestone is the successful PE-level simulation.

The verified PE demonstrates the fundamental computation required by the systolic array:

    PSUM_out =
        PSUM_in + (Activation x Weight)


The PE also demonstrates:

    Weight Loading

    Weight Storage

    Activation Forwarding


This provides a verified foundation for the complete accelerator.


## 63. Next Verification Milestone

The immediate next verification target is:

    Two-PE Integration


The objective is to confirm that the verified PE continues to behave correctly when connected to another PE.

After successful two-PE verification:

    Four-PE Chain


will be tested before moving to:

    Complete 4x4 Systolic Array


## 64. Summary

The project uses a bottom-up Verilog verification methodology.

The Processing Element has already been successfully simulated and verified for its core MAC functionality, weight loading, weight retention, partial-sum accumulation, and activation forwarding.

The next stage is integration verification.

The final verification objective is to demonstrate that the complete 4x4 weight-stationary systolic array correctly performs matrix multiplication while maintaining correct:

    Numerical Results

    Data Movement

    Weight Stationarity

    Partial-Sum Accumulation

    Output Mapping

    Cycle-Level Timing
