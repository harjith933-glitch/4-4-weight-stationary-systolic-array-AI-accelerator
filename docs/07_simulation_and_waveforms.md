# Simulation and Waveforms

## 1. Overview

Simulation is used to verify the functional behavior of the Verilog RTL before moving to synthesis and physical implementation.

The current project follows a simulation-driven development methodology:

    Verilog RTL
         +
    Verilog Testbench
         |
         v
    Compilation
         |
         v
    Simulation
         |
         +----------------+
         |                |
         v                v
    Console Output     Waveform
         |                |
         +-------+--------+
                 |
                 v
          Functional Verification

The Processing Element (PE) has already been implemented and successfully simulated.

The same methodology will later be extended to the complete 4x4 systolic array.


## 2. Purpose of Simulation

Simulation is used to verify that the RTL behaves according to the intended hardware architecture.

The primary objectives are:

- Verify reset behavior.
- Verify weight loading.
- Verify weight storage.
- Verify activation processing.
- Verify activation forwarding.
- Verify multiplication.
- Verify partial-sum accumulation.
- Verify MAC operation.
- Verify output behavior.
- Verify cycle-level timing.
- Identify RTL design errors before synthesis.


## 3. Simulation Environment

The project separates RTL design files, testbench files, simulation files, waveform files, and documentation.

Project structure:

    ~/systolic/
    |
    +-- rtl/
    |     +-- Verilog RTL files
    |
    +-- tb/
    |     +-- Verilog testbench files
    |
    +-- sim/
    |     +-- Simulation-related files
    |
    +-- wave/
    |     +-- Waveform files
    |
    +-- docs/
          +-- Documentation files

The RTL contains the synthesizable hardware.

The testbench contains the verification environment.

The simulation directory contains simulation-related files.

The waveform directory contains waveform-related artifacts.

The documentation directory contains the project documentation.


## 4. Design Under Test

The current Design Under Test (DUT) is the Processing Element.

Conceptually:

    +-------------------------+
    |       Testbench         |
    |                         |
    | Clock                   |
    | Reset                   |
    | Weight                  |
    | Weight Load             |
    | Activation              |
    | Partial Sum             |
    +------------+------------+
                 |
                 v
          +-------------+
          |     PE      |
          | Verilog RTL |
          +------+------+
                 |
                 v
              Outputs
                 |
                 v
             Waveform

The testbench controls the PE inputs and observes its outputs.


## 5. Simulation Inputs

The testbench provides controlled stimulus to the PE.

The major input signals are conceptually:

    Clock
    Reset
    Weight
    Weight Load Control
    Activation
    Partial Sum

The exact signal names, widths, and directions are defined by the current Verilog RTL and testbench.


## 6. Simulation Outputs

The important PE outputs include:

    Activation Output
    Partial-Sum Output

The testbench can also produce console messages indicating whether individual tests pass or fail.

The exact output signal names must always be taken from the actual Verilog source.


## 7. Basic Simulation Flow

The PE simulation follows this general sequence:

    1. Compile Verilog RTL
    2. Compile Verilog testbench
    3. Start simulation
    4. Generate clock
    5. Apply reset
    6. Load weight
    7. Apply activation
    8. Apply partial sum
    9. Observe outputs
    10. Compare expected result
    11. Inspect waveform
    12. Report PASS or FAIL


## 8. Compilation

The Verilog RTL and testbench must be compiled before simulation.

Conceptually:

    PE RTL
       +
    PE Testbench
       |
       v
    Verilog Compilation
       |
       v
    Simulation Model

Compilation errors must be resolved before functional simulation can be performed.

Typical compilation problems can include:

- Syntax errors
- Incorrect module names
- Incorrect port connections
- Missing source files
- Width mismatches
- Undeclared signals


## 9. Elaboration

During simulation setup, the simulator resolves the Verilog module hierarchy.

Conceptually:

    Testbench
       |
       v
       PE
       |
       +---- Internal Registers
       |
       +---- Arithmetic Logic
       |
       +---- Forwarding Logic

The simulator must be able to locate all required Verilog modules and correctly connect their ports.


## 10. Clock Generation

The testbench generates the clock required by the synchronous PE.

Conceptually:

    clk

    ____|‾‾‾|____|‾‾‾|____|‾‾‾|____

The clock provides the timing reference for sequential operations.

The exact clock period must be taken from the current Verilog testbench.


## 11. Reset Sequence

The simulation begins by placing the PE into a known state.

General sequence:

    Start
      |
      v
    Reset Asserted
      |
      v
    Required Clock Event
      |
      v
    PE Initialized
      |
      v
    Reset Released
      |
      v
    Normal Operation

The reset behavior must match the implementation in the PE RTL.


## 12. Weight Loading Sequence

After reset, the testbench loads a known weight.

Conceptually:

    Weight = W
       |
       v
    Weight Load = Active
       |
       v
    Clock Edge
       |
       v
    Weight Stored
       |
       v
    Weight Load = Inactive

The stored weight is then used for the MAC operation.


## 13. Weight Storage Verification

After loading, the weight must remain stored during normal operation.

Conceptually:

    Load Weight = W
          |
          v
    Weight Register
          |
          v
    Stored Weight = W
          |
          v
    Weight Load Disabled
          |
          v
    Stored Weight remains W

This verifies the weight-stationary behavior of the PE.


## 14. Activation Input

After the weight is loaded, the testbench applies activation data.

For example:

    Activation = A

The activation participates in the multiplication:

    Product = Activation x Weight

The activation is also forwarded to the next PE as required by the systolic architecture.


## 15. Partial-Sum Input

The testbench provides an initial partial sum.

For example:

    PSUM_in = P

The PE performs:

    PSUM_out = PSUM_in + (Activation x Weight)

This verifies that the incoming partial sum is correctly incorporated into the MAC operation.


## 16. MAC Simulation Example

Consider:

    Weight = 5
    Activation = 4
    PSUM_in = 10

First:

    Product = Activation x Weight

    Product = 4 x 5

    Product = 20

Then:

    PSUM_out = PSUM_in + Product

    PSUM_out = 10 + 20

    PSUM_out = 30

Therefore:

    Expected PSUM_out = 30

The actual RTL output is compared with this expected value.


## 17. Expected and Actual Results

The verification process compares two values:

    Expected Result

and:

    Actual RTL Result

Conceptually:

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


## 18. PASS Condition

A test passes when:

    Actual Result == Expected Result

Example:

    Expected = 30
    Actual   = 30

Therefore:

    PASS

A passing result indicates that the RTL produced the expected numerical output for the tested condition.


## 19. FAIL Condition

A test fails when:

    Actual Result != Expected Result

Example:

    Expected = 30
    Actual   = 29

Therefore:

    FAIL

A failure must be investigated using the test inputs, RTL, simulation output, and waveform.


## 20. Waveform Generation

A waveform provides a graphical representation of signal changes during simulation.

Waveforms are useful for checking:

- Clock transitions
- Reset behavior
- Weight loading
- Activation movement
- Partial-sum behavior
- Output timing
- Signal relationships

Waveform-related files are maintained under:

    wave/

The exact waveform format depends on the simulator being used.


## 21. Important Waveform Signals

Important PE signals include:

    clk
    reset
    weight
    weight_load
    activation_in
    activation_out
    psum_in
    psum_out

The exact names may differ depending on the current RTL.

The actual Verilog source is always the authoritative source for signal names.


## 22. Reading the Clock

The clock should be the first signal examined when analyzing a synchronous waveform.

The designer should identify the active clock edge and then determine:

    What were the inputs before the edge?

and:

    What changed after the edge?

This allows sequential behavior to be verified cycle by cycle.


## 23. Reading Reset

Reset should normally be examined before analyzing normal operation.

Expected sequence:

    Reset Asserted
         |
         v
    Internal State Initialized
         |
         v
    Reset Released
         |
         v
    Normal Operation

If unexpected values appear immediately after reset, reset implementation or initialization should be investigated.


## 24. Reading Weight Loading

To verify weight loading in the waveform:

    1. Locate the appropriate clock edge.
    2. Check the weight input.
    3. Check the weight-load control.
    4. Confirm that the weight is captured.
    5. Confirm that the stored weight remains available afterward.

Conceptually:

    weight_load = 1
         |
         v
    Clock Edge
         |
         v
    Weight Register Updated


## 25. Reading Weight Retention

After weight loading is disabled, the stored weight should remain unchanged.

Conceptually:

    Weight Load
        |
        v
    W stored
        |
        v
    Weight Load = 0
        |
        v
    Stored Weight remains W

This behavior is fundamental to the weight-stationary architecture.


## 26. Reading Activation

The activation waveform should be examined at the PE input.

For example:

    activation_in

        A0
        |
        A1
        |
        A2

The designer should verify when each activation becomes available relative to the clock.


## 27. Reading Activation Forwarding

Activation forwarding is observed using the input and output activation signals.

Conceptually:

    activation_in
          |
          v
         PE
          |
          v
    activation_out

The waveform should demonstrate the expected propagation behavior.

The exact cycle relationship depends on the RTL implementation.


## 28. Reading Partial Sum

The partial-sum input should be checked in the waveform.

For example:

    psum_in = 10

The PE should use this value as the starting accumulation value.

The expected result is:

    psum_out =
        psum_in + (activation x weight)


## 29. Reading Partial-Sum Output

The partial-sum output is one of the most important verification signals.

The designer should compare:

    PSUM_out

against:

    Expected =
        PSUM_in + (Activation x Weight)

The value must also appear at the correct time according to the RTL.


## 30. Waveform-Based MAC Verification

The waveform verification process is:

    Identify Weight
          |
          v
    Identify Activation
          |
          v
    Identify PSUM_in
          |
          v
    Calculate Product
          |
          v
    Calculate Expected PSUM
          |
          v
    Observe PSUM_out
          |
          v
    Compare
          |
          v
       PASS/FAIL


## 31. Example Waveform Interpretation

Assume:

    Weight = 5
    Activation = 4
    PSUM_in = 10

Expected calculation:

    Product = 4 x 5

    Product = 20

Then:

    PSUM_out = 10 + 20

    PSUM_out = 30

During waveform inspection:

    weight       = 5
    activation   = 4
    psum_in      = 10

Expected:

    psum_out     = 30

The output must appear according to the actual RTL timing.


## 32. Console Verification

Console output can provide automated PASS/FAIL information.

A professional testbench can produce output similar to:

    ----------------------------
    PE Verification
    ----------------------------

    Weight Load Test : PASS
    MAC Test         : PASS
    Forwarding Test  : PASS

    ----------------------------
    Overall Result   : PASS
    ----------------------------

The exact output depends on the implemented testbench.


## 33. Simulation Debugging

When a test fails, waveform inspection should be used to locate the first incorrect signal.

Recommended debugging sequence:

    Check Reset
        |
        v
    Check Weight Load
        |
        v
    Check Stored Weight
        |
        v
    Check Activation
        |
        v
    Check Product
        |
        v
    Check PSUM Input
        |
        v
    Check PSUM Output
        |
        v
    Check Activation Forwarding
        |
        v
    Check Timing

The earliest incorrect signal is generally the best place to begin debugging.


## 34. Unknown Values

Simulation may show unknown values as:

    X

High-impedance values may be shown as:

    Z

An unexpected X can indicate:

- Uninitialized register
- Missing reset
- Undriven signal
- Multiple conflicting drivers
- Incorrect testbench stimulus

An unexpected Z can indicate that a signal is not being driven as intended.


## 35. Reset-Related Unknown Values

If a register has not been initialized correctly, it may begin simulation with:

    X

For example:

    stored_weight = X

Then:

    activation x stored_weight

may also produce:

    X

Therefore, reset behavior must be checked whenever unknown values appear.


## 36. Timing Errors

A result may be numerically correct but appear at the wrong clock cycle.

Example:

    Expected:
        Result at Cycle N+1

    Actual:
        Result at Cycle N+2

This represents a timing mismatch.

Therefore, verification must check both:

    Correct Value

and:

    Correct Cycle


## 37. Input Alignment Errors

Incorrect input scheduling can cause incorrect MAC results.

For example:

    Expected:

    Activation A
          x
    Weight W

but the actual operation uses:

    Activation B
          x
    Weight W

The resulting numerical error may appear as a MAC failure.

The waveform can reveal the input alignment problem.


## 38. Weight Loading Timing Error

A possible error occurs if activation data is processed before the intended weight has been stored.

Conceptually:

    Cycle N:
        Weight Load

    Cycle N+1:
        Activation

The exact timing depends on the RTL.

The waveform should be used to verify that the intended weight is available when the MAC operation occurs.


## 39. Partial-Sum Timing Error

The partial sum must also arrive at the correct time.

For example:

    Expected:
        PSUM_in = P

but the PE receives:

    PSUM_in = Previous_P

The multiplication may be correct while the final accumulated result is incorrect.

Waveform inspection can identify this problem.


## 40. Activation Forwarding Timing Error

Activation forwarding must preserve the intended value and timing.

For example:

    Cycle N:
        activation_in = A0

    Cycle N+1:
        activation_out = A0

If another value appears at the output, the forwarding path should be investigated.

The exact expected cycle depends on the implemented RTL.


## 41. Waveform Debugging Strategy

When debugging a waveform, it is better to inspect important signals systematically rather than attempting to analyze every signal simultaneously.

Recommended order:

    1. Clock
    2. Reset
    3. Weight Load
    4. Weight
    5. Activation Input
    6. Partial Sum Input
    7. Partial Sum Output
    8. Activation Output

This makes cycle-level debugging easier.


## 42. Simulation Reproducibility

A professional GitHub project should allow another engineer to reproduce the simulation.

The repository should contain:

- RTL source
- Testbench source
- Simulation instructions
- Required source files
- Expected behavior
- Waveform instructions

The intended workflow is:

    Clone Repository
          |
          v
    Compile Verilog RTL
          |
          v
    Compile Testbench
          |
          v
    Run Simulation
          |
          v
    Observe PASS
          |
          v
    Inspect Waveform

The exact simulator commands should be documented once the simulation environment is finalized.


## 43. Simulator Selection

The RTL is intended to remain compatible with standard Verilog simulation tools.

Possible simulators include:

- Icarus Verilog
- Verilator
- Other compatible Verilog simulators

The simulator actually used for the project should be recorded in the repository.


## 44. Waveform File Management

Waveform files can become large.

Therefore, generated waveform files should be managed separately from the RTL source.

The project uses:

    wave/

for waveform-related artifacts.

Large generated files should not automatically be committed to GitHub unless there is a specific reason to preserve them.


## 45. Simulation Artifacts

Simulation may generate:

- Compiled simulation files
- Log files
- Waveform files
- Temporary simulator files

These should remain separate from the source RTL.

A .gitignore file should eventually be used to prevent unnecessary generated files from being committed to GitHub.


## 46. Simulation and GitHub

A professional repository should make the simulation process reproducible.

Recommended repository organization:

    rtl/
        Verilog Design Source

    tb/
        Verilog Testbench

    sim/
        Simulation Scripts
        Supporting Files

    wave/
        Selected Waveform Evidence

    docs/
        Documentation

    README.md
        Project Overview and Usage

This allows another engineer to understand both the design and its verification flow.


## 47. Current PE Simulation Status

The Processing Element has successfully completed simulation.

The verified functions include:

    Weight Loading
          |
          v
    Weight Storage
          |
          v
    Activation Input
          |
          v
    Multiplication
          |
          v
    Partial-Sum Accumulation
          |
          v
    MAC Output

and:

    Activation Input
          |
          v
    Activation Forwarding
          |
          v
    Activation Output

The PE simulation represents the first major functional verification milestone of the project.


## 48. Current Simulation Status

    Simulation Item                         Status

    PE RTL compilation                     Completed
    PE testbench                           Completed
    Clock generation                       Completed
    Reset testing                          Completed
    Weight loading                         Verified
    Weight storage                         Verified
    Activation input                       Verified
    Activation forwarding                  Verified
    Multiplication                         Verified
    Partial-sum operation                  Verified
    MAC operation                          Verified
    Waveform inspection                    Completed
    PE functional simulation               Passed

    Two-PE simulation                      Pending
    PE row simulation                      Pending
    4x4 array simulation                   Pending
    Matrix multiplication simulation       Pending
    Full accelerator simulation            Pending


## 49. Future Array-Level Waveform

After the PE has been integrated into the 4x4 array, waveform analysis will include multiple PE interfaces.

Conceptually:

    PE00
      |
      +---- Activation
      +---- Partial Sum

    PE01
      |
      +---- Activation
      +---- Partial Sum

    PE02
      |
      +---- Activation
      +---- Partial Sum

    PE03
      |
      +---- Activation
      +---- Partial Sum

    ...

    PE33
      |
      +---- Activation
      +---- Partial Sum

The waveform will be used to verify data propagation across the complete array.


## 50. Array-Level Timing Verification

The complete array must eventually be verified cycle by cycle.

The verification should determine:

    When does an activation enter the array?

    When does it reach PE00?

    When does it reach PE01?

    When does it reach PE02?

    When does it reach PE03?

    When does each PE perform its MAC?

    When does each partial sum become available?

    When does each output become valid?

This establishes the actual latency and pipeline behavior of the array.


## 51. Matrix Multiplication Waveform

At the final verification stage, the waveform should demonstrate the following sequence:

    Matrix A
        +
    Matrix B
        |
        v
    Weight Loading
        |
        v
    Activation Propagation
        |
        v
    PE MAC Operations
        |
        v
    Partial-Sum Accumulation
        |
        v
    Output Matrix C

The final numerical output must match an independent matrix multiplication reference calculation.


## 52. Simulation Limitations

Simulation verifies functional behavior for the test cases applied.

Simulation alone does not establish:

- Maximum operating frequency
- Physical timing
- Silicon area
- Power consumption
- Routing congestion
- Physical design quality
- Manufacturing behavior

These require later synthesis, timing, power, and physical-design analysis.


## 53. Simulation Best Practices

The following practices should be maintained:

1. Reset the DUT before normal operation.
2. Use deterministic test vectors.
3. Compare expected and actual results.
4. Check cycle-level timing.
5. Inspect waveforms when failures occur.
6. Test boundary values.
7. Test repeated operations.
8. Keep RTL and testbench separate.
9. Keep generated files organized.
10. Document simulator requirements.
11. Maintain reproducible simulation steps.
12. Record actual verification results.
13. Do not claim unverified functionality.
14. Keep documentation synchronized with the RTL.


## 54. Verification Evidence

A professional hardware project should preserve evidence of successful verification.

Useful evidence includes:

- Verilog testbench source
- Simulation logs
- PASS/FAIL output
- Waveform screenshots
- Representative test vectors
- Expected results
- Actual results

Only verified results should be presented as completed project results.


## 55. Current Verification Milestone

The current milestone is:

    PE RTL
       |
       v
    PE Testbench
       |
       v
    Simulation
       |
       v
    Waveform Inspection
       |
       v
    Functional Verification
       |
       v
       PASS

This establishes the Processing Element as a verified computational building block.


## 56. Next Simulation Milestone

The next simulation milestone is:

    Two-PE Integration
          |
          v
    Simulation
          |
          v
    Activation Propagation Verification
          |
          v
    Partial-Sum Verification
          |
          v
    Timing Verification
          |
          v
        PASS

After this:

    Two PE
       |
       v
    Four PE Row
       |
       v
    4x4 Array
       |
       v
    Matrix Multiplication
       |
       v
    Full Array Verification


## 57. Summary

Simulation provides the primary functional verification environment for the current development stage.

The Processing Element has been successfully simulated and verified for its fundamental operations:

    Weight Loading
    Weight Storage
    Activation Processing
    Activation Forwarding
    Multiplication
    Partial-Sum Accumulation
    MAC Operation

Waveform inspection provides cycle-level visibility into the design and is an important debugging and verification method.

The next stage is to extend simulation from the verified PE to multi-PE integration and eventually to the complete 4x4 systolic array.

The project will only claim complete accelerator verification after the complete array has been simulated against known matrix multiplication results.
