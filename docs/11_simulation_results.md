# Simulation Results

## 1. Overview

This document records the simulation and verification results of the 4x4 weight-stationary systolic array accelerator.

The project follows a bottom-up verification methodology in which the Processing Element (PE) is first implemented and verified before integrating multiple PEs into the complete 4x4 systolic array.

The current documented results therefore distinguish between:

- Completed and verified PE-level functionality
- Integration work that is still in progress
- Future 4x4 array-level verification

This distinction is important to ensure that the project documentation reflects the actual implementation status.


## 2. Simulation Objective

The primary objective of simulation is to verify that the Verilog RTL performs the intended hardware operations correctly.

The simulation verifies:

- Reset behavior
- Weight loading
- Weight storage
- Activation input
- Activation forwarding
- Multiplication
- Partial-sum accumulation
- MAC operation
- Cycle-by-cycle behavior


## 3. HDL Used

The design and testbench are implemented using:

    Verilog

SystemVerilog is not used for the current implementation.

All signal names, widths, timing behavior, and reset behavior documented here must correspond to the actual Verilog RTL.


## 4. Verification Approach

The project uses a bottom-up simulation approach.

The current flow is:

    PE RTL
       |
       v
    PE Testbench
       |
       v
    Simulation
       |
       v
    Waveform Analysis
       |
       v
    Functional Verification
       |
       v
    Multi-PE Integration
       |
       v
    4x4 Array Verification


The PE-level verification is the first completed milestone.


## 5. Processing Element Verification

The Processing Element is the fundamental computational unit of the accelerator.

The PE performs the basic MAC operation:

    PSUM_out =
        PSUM_in + (Activation x Weight)


The PE also participates in activation data movement through the systolic array.


## 6. PE Testbench

A dedicated Verilog testbench was created for the Processing Element.

The testbench provides the required input stimulus and checks the resulting outputs.

The testbench is responsible for applying:

- Clock
- Reset
- Weight data
- Weight load control
- Activation data
- Partial-sum input

and observing:

- Activation output
- Partial-sum output


## 7. Weight Loading Verification

Weight loading was verified at the PE level.

The test applies a known weight to the PE while the appropriate weight-loading control is asserted.

Conceptually:

    Weight Input
         |
         v
    Weight Load
         |
         v
    Weight Register
         |
         v
    Stored Weight


The loaded weight is then used during the computation phase.


## 8. Weight Retention Verification

After loading, the weight must remain available for subsequent MAC operations.

The expected behavior is:

    Weight Load
        |
        v
    Weight Stored
        |
        v
    Computation
        |
        v
    Weight Remains Available


This behavior is fundamental to the weight-stationary architecture.


## 9. MAC Operation Verification

The PE MAC operation was successfully verified.

The operation is:

    PSUM_out =
        PSUM_in + (Activation x Weight)


Example verification:

    Activation = 4

    Weight = 5

    PSUM_in = 10


Multiplication:

    4 x 5 = 20


Accumulation:

    10 + 20 = 30


Expected:

    PSUM_out = 30


The simulation confirmed the expected MAC behavior.


## 10. Activation Forwarding Verification

Activation forwarding was also verified at the PE level.

The conceptual data path is:

    Activation Input
         |
         v
        PE
         |
         v
    Activation Output


This behavior is required for cascading multiple PEs into the systolic array.


## 11. Partial-Sum Verification

The PE accepts an incoming partial sum and combines it with the current multiplication result.

The operation is:

    New Partial Sum =
        Incoming Partial Sum
        +
        Activation x Weight


Example:

    Incoming Partial Sum = 10

    Activation = 4

    Weight = 5


Therefore:

    New Partial Sum =
        10 + (4 x 5)

    New Partial Sum = 30


This confirms the accumulation behavior of the PE.


## 12. Zero Partial-Sum Verification

A zero partial sum provides a simple way to verify the basic multiplication path.

Example:

    PSUM_in = 0

    Activation = 3

    Weight = 7


Expected:

    PSUM_out =
        0 + (3 x 7)

    PSUM_out = 21


This test verifies that the multiplication result is correctly transferred into the accumulation path.


## 13. Non-Zero Partial-Sum Verification

A non-zero partial sum verifies that an existing accumulated value is correctly preserved and added to the new product.

Example:

    PSUM_in = 10

    Activation = 3

    Weight = 7


Expected:

    PSUM_out =
        10 + (3 x 7)

    PSUM_out = 31


This verifies the fundamental accumulation behavior.


## 14. Clock-Based Verification

The PE operates synchronously with the clock.

The testbench therefore applies input stimulus relative to clock cycles.

Conceptually:

    Clock
      |
      v
    Input Sampling
      |
      v
    PE Computation
      |
      v
    Registered Output


The exact cycle at which each output becomes valid must be determined from the actual RTL and simulation waveform.


## 15. Reset Verification

Reset behavior is checked before normal PE operation.

The general sequence is:

    Reset Asserted
         |
         v
    Clock Cycle(s)
         |
         v
    Reset Released
         |
         v
    Normal Operation


The resulting signal values are checked against the reset behavior implemented in the Verilog RTL.


## 16. Waveform Analysis

Waveform analysis is used to inspect the cycle-by-cycle behavior of the PE.

Important signals include:

    Clock

    Reset

    Weight Load Control

    Weight Input

    Activation Input

    Activation Output

    Partial Sum Input

    Partial Sum Output


The waveform provides visibility into the order in which data is loaded, processed, and forwarded.


## 17. Waveform Verification Method

The waveform is analyzed in the following order:

    1. Check reset.

    2. Check weight-loading control.

    3. Check weight input.

    4. Check that the weight is stored.

    5. Check activation input.

    6. Check partial-sum input.

    7. Check MAC result.

    8. Check activation forwarding.

    9. Check output timing.


This approach makes it easier to identify the source of any mismatch.


## 18. Functional Verification Result

The PE-level functional verification was successful.

Verified functionality includes:

    Weight Loading
        PASS

    Weight Storage
        PASS

    Activation Processing
        PASS

    Activation Forwarding
        PASS

    Multiplication
        PASS

    Partial-Sum Accumulation
        PASS

    MAC Operation
        PASS


These results establish the PE as a verified computational building block for the larger systolic array.


## 19. PE Verification Status

Current status:

    +--------------------------------+
    | PE Verification                |
    +--------------------------------+
    | RTL Implementation      PASS   |
    | Testbench               PASS   |
    | Weight Loading          PASS   |
    | MAC Operation           PASS   |
    | Activation Forwarding   PASS   |
    | Partial Sum             PASS   |
    +--------------------------------+


The PE milestone is therefore considered completed.


## 20. Example PE Calculation

The fundamental calculation verified during PE testing is:

    Activation = 4

    Weight = 5

    Partial Sum = 10


Therefore:

    Product =
        4 x 5

    Product = 20


Then:

    Output =
        10 + 20

    Output = 30


Expected result:

    30


The simulation result was checked against the expected arithmetic result.


## 21. Multiple MAC Operations

The PE architecture is intended to support repeated MAC operations.

For example:

    Operation 1:

        Activation = A0
        Weight = W
        Partial Sum = 0

        Result:
            A0 x W


    Operation 2:

        Activation = A1
        Weight = W
        Partial Sum = Previous Result

        Result:
            A0W + A1W


    Operation 3:

        Activation = A2
        Weight = W
        Partial Sum = Previous Result

        Result:
            A0W + A1W + A2W


This repeated accumulation forms the basis of dot-product computation.


## 22. Relationship to Matrix Multiplication

The PE operation is a building block for matrix multiplication.

For:

    C = A x B


each output element is calculated as:

    Cij =
        Ai0B0j
        + Ai1B1j
        + Ai2B2j
        + Ai3B3j


The individual multiplication terms can be performed by the PEs and accumulated through the systolic architecture.


## 23. Systolic Data Movement

The PE verification establishes the basic behavior required for systolic data movement.

Conceptually:

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


The complete array will be verified after PE integration.


## 24. Current Integration Status

The PE has been successfully verified.

The next stage is integration of multiple PEs.

The intended progression is:

    Single PE
        |
        v
    Two PEs
        |
        v
    PE Chain
        |
        v
    4x4 Array
        |
        v
    Matrix Multiplication


The complete 4x4 matrix multiplication result should not be marked as verified until the corresponding simulation has passed.


## 25. Multi-PE Verification Plan

The first integration test should connect two PEs.

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


The test must verify:

    PE0 Activation Output
        =
    PE1 Activation Input


and confirm correct cycle alignment.


## 26. Four-PE Verification Plan

After two-PE verification, a four-PE chain can be tested.

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


This verifies longer activation propagation before full 4x4 integration.


## 27. 4x4 Array Verification Plan

The final architecture contains:

    4 x 4 = 16 PEs


The array will be verified for:

- Correct PE instantiation
- Correct PE connectivity
- Correct weight mapping
- Correct activation propagation
- Correct partial-sum behavior
- Correct output mapping
- Correct timing


The final objective is:

    C = A x B


with all output elements matching an independently calculated reference result.


## 28. Matrix-Level Test Plan

The following tests are planned for complete-array verification:

    Test 1:
        Identity Matrix

    Test 2:
        Zero Matrix

    Test 3:
        Small Values

    Test 4:
        Increasing Values

    Test 5:
        Repeated Values

    Test 6:
        Boundary Values

    Test 7:
        Multiple Matrix Operations

    Test 8:
        Regression Test


Each test should have an independently generated expected result.


## 29. Identity Matrix Test

For an identity matrix:

    A x I = A


This test is useful for checking:

- Weight placement
- Activation ordering
- PE connectivity
- Output mapping


The output should exactly reproduce the input matrix.


## 30. Zero Matrix Test

For a zero matrix:

    A x 0 = 0


All outputs should become zero assuming the design starts from the correct reset and partial-sum state.


## 31. Reference Model

The final matrix-level verification should use an independent reference model.

Conceptually:

    Matrix A
        |
        +------------------+
        |                  |
        v                  v
    Reference Model      RTL Array
        |                  |
        v                  v
    Expected C          Actual C
        |                  |
        +--------+---------+
                 |
                 v
              Compare


This prevents the expected result from depending on the same RTL logic being tested.


## 32. PASS and FAIL Criteria

A test passes when:

    Every required output
        =
    Corresponding expected output


A test fails when:

    Any required output
        !=
    Corresponding expected output


The testbench should identify the exact failing output whenever possible.


## 33. Timing Verification

Numerical correctness alone is not sufficient.

The verification must also confirm that the result appears at the expected clock cycle.

The following should be recorded:

    Input Cycle

    Processing Cycles

    Output Cycle

    Total Latency


The exact values will be determined from the completed array simulation.


## 34. Latency

Latency represents the number of clock cycles from the defined input-acceptance point to the corresponding valid output.

The final latency should be measured from simulation.

It should not be estimated or claimed without waveform evidence.


## 35. Throughput

Throughput represents how frequently valid computations can be completed after the array reaches steady-state operation.

The final throughput should be measured from the actual implementation and simulation.


## 36. Future Boundary Testing

Boundary testing will be performed after the complete array is functional.

Tests will include values close to the limits allowed by the actual signal widths.

The purpose is to identify:

- Overflow
- Truncation
- Incorrect arithmetic
- Signedness issues
- Width mismatches


## 37. Future Regression Testing

Whenever RTL is modified, previously passing tests should be rerun.

Regression flow:

    RTL Modification
         |
         v
    Recompile
         |
         v
    Run PE Tests
         |
         v
    Run Integration Tests
         |
         v
    Run Matrix Tests
         |
         v
    Compare Results
         |
         v
    PASS / FAIL


This ensures that new changes do not break existing functionality.


## 38. Simulation Artifacts

The project should maintain the following simulation artifacts:

    Testbench Source
        |
        v
    Simulation Log
        |
        v
    Waveform
        |
        v
    Expected Result
        |
        v
    Actual Result
        |
        v
    Verification Report


The relevant generated files should be organized under the project simulation and waveform directories.


## 39. Reproducibility

The simulation should be reproducible by another engineer.

The repository should therefore contain:

- Verilog RTL
- Verilog testbench
- Simulation instructions
- Expected results
- Documentation
- Required scripts, where applicable


A clean checkout should allow the verification environment to be recreated without depending on undocumented local changes.


## 40. Current Result Summary

Current verified milestone:

    ========================================
             PE SIMULATION RESULT
    ========================================

    Verilog RTL                 PASS

    PE Testbench                PASS

    Weight Loading              PASS

    Weight Storage              PASS

    Activation Processing       PASS

    Activation Forwarding       PASS

    Multiplication              PASS

    Partial-Sum Accumulation    PASS

    MAC Operation               PASS

    ========================================

    PE VERIFICATION STATUS:
    PASSED
    ========================================


## 41. What Has Not Yet Been Claimed

The following should not be marked as completed until the corresponding RTL and simulation are finished:

    Complete 4x4 Array Verification

    Full 4x4 Matrix Multiplication

    Final Array Latency

    Final Array Throughput

    Synthesis Results

    Area Results

    Power Results

    Timing Results

    FPGA Results

    ASIC Results


This prevents the GitHub project from overstating the current implementation status.


## 42. Recommended Result Reporting Format

For every future simulation, record:

    Test Name:
        <test name>

    Date:
        <date>

    RTL Version:
        <commit/hash>

    Simulator:
        <simulator>

    Test Result:
        PASS / FAIL

    Expected:
        <expected result>

    Actual:
        <actual result>

    Latency:
        <measured cycles>

    Notes:
        <additional observations>


This creates a traceable verification history.


## 43. Verification Milestone History

### Milestone 1

    PE RTL implemented

    Status:
        COMPLETED


### Milestone 2

    PE testbench implemented

    Status:
        COMPLETED


### Milestone 3

    PE simulation performed

    Status:
        COMPLETED


### Milestone 4

    MAC operation verified

    Status:
        COMPLETED


### Milestone 5

    Weight loading verified

    Status:
        COMPLETED


### Milestone 6

    Activation forwarding verified

    Status:
        COMPLETED


### Milestone 7

    Multi-PE integration

    Status:
        NEXT STAGE


### Milestone 8

    Complete 4x4 array

    Status:
        PLANNED


### Milestone 9

    Matrix multiplication verification

    Status:
        PLANNED


## 44. Final Verification Objective

The final verification goal is to demonstrate that the complete 4x4 weight-stationary systolic array correctly performs:

    C = A x B


using:

    16 Processing Elements


while maintaining:

    Weight Stationarity

    Correct Activation Propagation

    Correct Partial-Sum Accumulation

    Correct Output Mapping

    Correct Cycle-Level Timing


The final result will be accepted only when the RTL output matches the independently calculated expected matrix result for the required verification tests.


## 45. Summary

The Processing Element has been successfully implemented and simulated using Verilog.

The PE-level simulation verified the fundamental operations required by the accelerator:

    Weight Loading
    Weight Storage
    Activation Processing
    Activation Forwarding
    Multiplication
    Partial-Sum Accumulation
    MAC Operation


The verified PE now serves as the foundation for the next development stage.

The next major milestone is multi-PE integration followed by complete 4x4 array verification and matrix multiplication testing.

All future performance metrics and implementation claims will be based on actual simulation, synthesis, and implementation results rather than assumptions.
