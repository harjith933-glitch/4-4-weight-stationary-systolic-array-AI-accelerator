# Performance Analysis

## 1. Overview

This document defines the performance metrics and analysis methodology for the 4x4 weight-stationary systolic array accelerator.

The purpose of the analysis is to evaluate the accelerator from a hardware-performance perspective using measurable parameters such as:

    Latency

    Throughput

    Clock Frequency

    Number of Processing Elements

    MAC Operations

    Parallelism

    Data Reuse

    Computational Efficiency

    Area Efficiency

    Power Efficiency


Only measured results are reported as actual results.

Parameters that have not yet been measured are explicitly marked as pending.


## 2. Current Performance Status

The current project has completed Processing Element level functional verification.

Verified:

    PE MAC operation

    Weight loading

    Weight storage

    Activation forwarding

    Partial-sum accumulation


Not yet measured:

    Complete 4x4 array latency

    Complete 4x4 array throughput

    Maximum operating frequency

    Area

    Power

    Energy per operation

    Hardware utilization


Therefore, the current performance analysis primarily describes the architecture and the methodology that will be used for final measurement.


## 3. Accelerator Configuration

Current target architecture:

    Array Size:
        4x4


    Number of Processing Elements:
        16


    Dataflow:
        Weight Stationary


    Computational Operation:
        Multiply-Accumulate


    HDL:
        Verilog


    Architecture:
        Systolic Array


    Target Application:
        Matrix Multiplication / AI Acceleration


## 4. Processing Element Computation

Each Processing Element performs a multiply-and-accumulate operation.

The fundamental operation is:

    PSUM_out =
        PSUM_in + (Activation x Weight)


Therefore, each PE can perform one MAC operation per applicable computation cycle, subject to the actual control and pipeline implementation.


## 5. Parallelism

The 4x4 systolic array contains:

    4 x 4 = 16 PEs


Therefore, up to 16 PE computation units can operate in parallel.

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

This spatial parallelism is the primary source of computational acceleration.


## 6. Matrix Multiplication

The accelerator targets matrix multiplication:

    C = A x B


For two 4x4 matrices:

    A = 4x4

    B = 4x4

    C = 4x4


Each output element is calculated as:

    Cij =
        Ai0B0j
        + Ai1B1j
        + Ai2B2j
        + Ai3B3j


Each output therefore requires:

    4 multiplications

and:

    3 additions


or, when implemented through a MAC accumulation sequence:

    4 MAC contributions


## 7. Total MAC Operations

For a 4x4 matrix multiplication:

    Number of output elements:

        4 x 4 = 16


Each output requires:

    4 multiplications


Therefore:

    Total multiplications =
        16 x 4

    = 64


The accelerator therefore performs:

    64 MAC contributions


for one complete 4x4 matrix multiplication.


## 8. Theoretical Parallel Computation

The array contains:

    16 PEs


If all PEs are active during a computation cycle, the architecture can perform up to:

    16 MAC operations per cycle


subject to:

    Data Availability

    Pipeline State

    Control Signals

    Array Mapping

    Actual RTL Implementation


This is a theoretical architectural capability and should not be confused with measured sustained throughput.


## 9. Clock Frequency

The operating frequency is determined by the maximum clock frequency supported by the implementation.

The relationship is:

    Frequency =
        1 / Clock Period


For example:

    Clock Period = 10 ns


then:

    Frequency = 100 MHz


This is only an illustrative example.

The actual project frequency must be obtained from synthesis and timing analysis.


## 10. Latency

Latency is the number of clock cycles required to produce the required output after valid input data begins entering the accelerator.

Conceptually:

    Latency =
        Number of clock cycles from
        input acceptance to output availability


For a systolic array, latency depends on:

    Array Dimensions

    Data Scheduling

    Pipeline Structure

    Input Injection

    Weight Loading

    Output Collection


Therefore, the exact latency must be determined from the implemented RTL and verified using the waveform.


## 11. Systolic Pipeline Behavior

The systolic array does not necessarily produce all results immediately after the first input.

Data propagates through the array.

Conceptually:

    Cycle 0:
        Input Injection


    Cycle 1:
        Data Propagation


    Cycle 2:
        Further Propagation


    Cycle 3:
        Further Propagation


    ...


    Final Cycles:
        Output Collection


The exact timing depends on the implemented dataflow and control architecture.


## 12. Pipeline Fill Time

Before the complete array reaches steady-state operation, the systolic array requires a pipeline fill period.

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


Data requires multiple cycles to propagate through multiple computational stages.


The pipeline fill time should be measured from the actual RTL simulation.


## 13. Pipeline Drain Time

After the final input data is injected, intermediate values may still be propagating through the array.

The time required for the remaining results to emerge is the pipeline drain period.

Therefore, total execution time can be considered as:

    Total Cycles =
        Fill Cycles
        + Computation Cycles
        + Drain Cycles


The exact values depend on the implemented schedule.


## 14. Throughput

Throughput describes how much computation the accelerator can perform per unit time.

For a MAC-oriented accelerator:

    MAC Throughput =
        MACs per Cycle x Clock Frequency


For the theoretical maximum of the 4x4 array:

    MACs per Cycle = 16


Therefore:

    Theoretical MAC Throughput =
        16 x Clock Frequency


For example, at an illustrative 100 MHz:

    16 x 100,000,000

    = 1.6 billion MAC/s


This is only an example.

No actual TOPS or MAC/s result should be claimed until the actual clock frequency and sustained utilization are measured.


## 15. Operations Per Second

For matrix multiplication, each MAC contribution can be considered as:

    1 multiplication
    +
    1 addition


Therefore, if a convention counts each multiplication and addition as separate arithmetic operations:

    1 MAC = 2 operations


Under that convention:

    Operations per Second =
        MACs per Second x 2


The convention used for final performance reporting must always be stated.


## 16. TOPS Calculation

TOPS means:

    Tera Operations Per Second


If the operation-counting convention is:

    1 MAC = 2 operations


then:

    TOPS =
        MAC/s x 2 / 10^12


For example, if a design sustained:

    1 x 10^9 MAC/s


then:

    Operations/s =
        2 x 10^9


and:

    TOPS =
        0.002


This is only an example and is not a measured result from this project.


## 17. Weight-Stationary Advantage

The architecture uses weight-stationary dataflow.

The main idea is:

    Weights
       |
       v
    Stored in PE
       |
       v
    Reused across computations


Activation data moves through the array while the stored weights are reused.

This can reduce unnecessary movement of weight data.


## 18. Data Reuse

Data reuse is an important accelerator characteristic.

The same weight can participate in multiple MAC operations without being reloaded for every operation.

Conceptually:

    Load Weight
         |
         v
    Store Weight
         |
         +----> MAC 1
         |
         +----> MAC 2
         |
         +----> MAC 3
         |
         +----> MAC 4


This improves the utilization of locally stored weight data.


## 19. Local Communication

The systolic architecture primarily uses local communication between neighboring PEs.

Conceptually:

    PE00 -> PE01 -> PE02 -> PE03


and:

    PE10 -> PE11 -> PE12 -> PE13


This structured communication can reduce the need for centralized data movement.


## 20. Memory Movement Consideration

In AI accelerators, data movement can consume significant energy.

The weight-stationary approach attempts to reduce weight movement by storing weights locally inside the PEs.

Therefore:

    Weight Reuse
        |
        v
    Reduced Weight Movement
        |
        v
    Potentially Lower Data-Movement Cost


Actual power savings must be demonstrated through power analysis and should not be assumed solely from the architecture.


## 21. PE Utilization

PE utilization indicates how effectively the available PEs are performing useful computation.

Conceptually:

    PE Utilization =
        Active PE Cycles
        /
        Total Available PE Cycles


For example, if all 16 PEs are active for the relevant computation period:

    Utilization approaches 100%


However, pipeline fill and drain periods can reduce average utilization.


## 22. Array Utilization

Array utilization can be calculated as:

    Array Utilization =
        Useful Compute Cycles
        /
        Total Compute Cycles


A more detailed implementation may calculate:

    Total Active PE Cycles
        /
    Total PE Capacity


The final metric should be calculated from simulation or performance traces.


## 23. Latency Measurement Method

Latency should be measured using the following procedure:

    1. Identify the cycle where the first valid input is accepted.

    2. Identify the cycle where the corresponding output becomes valid.

    3. Calculate the cycle difference.

    4. Repeat for representative test cases.

    5. Record the measured latency.


Example format:

    Input Valid Cycle:
        <Measured Cycle>


    Output Valid Cycle:
        <Measured Cycle>


    Latency:
        <Measured Cycles>


Actual values must come from simulation.


## 24. Throughput Measurement Method

Throughput should be measured using:

    1. Number of completed results.

    2. Number of cycles.

    3. Clock frequency.


For example:

    Completed MAC Operations:
        N


    Elapsed Cycles:
        C


    Clock Frequency:
        F


Then:

    MAC/cycle =
        N / C


and:

    MAC/s =
        (N / C) x F


This provides a measurable throughput value.


## 25. Benchmark Methodology

Performance benchmarking should use repeatable input sets.

Recommended tests include:

    Zero Matrix

    Identity Matrix

    Ones Matrix

    Small Integer Values

    Maximum Supported Values

    Random Matrices


The same test vectors should be used when comparing different implementations.


## 26. Baseline Comparison

A useful benchmark can compare the systolic architecture against a sequential MAC-based implementation.

Conceptually:

    Sequential Architecture
        vs
    4x4 Systolic Architecture


Comparison parameters:

    Latency

    Throughput

    Parallelism

    Hardware Resources

    Power

    Area


The comparison should use the same data width and technology assumptions.


## 27. Sequential Baseline

A simple sequential matrix multiplication implementation may reuse a smaller number of MAC units.

This can reduce:

    Hardware Area


but may increase:

    Execution Time


The systolic architecture trades additional hardware resources for increased parallelism.


## 28. Parallelism vs Area Trade-Off

The 4x4 architecture uses:

    16 PEs


This provides high spatial parallelism compared with a single-MAC architecture.

However:

    More PEs
        ->
    More Area


and potentially:

    More Switching
        ->
    More Dynamic Power


Therefore, accelerator design requires balancing:

    Performance

    Area

    Power


## 29. Scalability

The architecture can conceptually be extended to larger arrays.

Examples:

    2x2
        = 4 PEs


    4x4
        = 16 PEs


    8x8
        = 64 PEs


    16x16
        = 256 PEs


Increasing array size increases available parallelism but also increases:

    Area

    Power

    Routing Complexity

    Data Scheduling Complexity


## 30. Scaling Relationship

For an N x N systolic array:

    Number of PEs =
        N^2


For matrix multiplication of two N x N matrices:

    Total MAC Contributions =
        N^3


Therefore:

    4x4 array:

        PEs = 4^2
            = 16


        MAC Contributions = 4^3
            = 64


This relationship demonstrates the scalability of the architecture.


## 31. Theoretical Compute Capacity

For an N x N array:

    Maximum MACs per Cycle
        ≈ N^2


Therefore:

    4x4:
        16 MAC/cycle


    8x8:
        64 MAC/cycle


    16x16:
        256 MAC/cycle


This is theoretical parallel capacity and assumes appropriate data availability and full PE utilization.


## 32. Performance Bottlenecks

Potential performance bottlenecks include:

    Clock Frequency

    MAC Critical Path

    Data Injection Rate

    Pipeline Fill

    Pipeline Drain

    Control Overhead

    Memory Bandwidth

    Data Width

    Routing Delay


These factors must be evaluated using implementation and simulation results.


## 33. Critical Path Consideration

The MAC operation contains arithmetic logic.

Conceptually:

    Activation
        |
        v
    Multiplier
        |
        v
    Adder
        |
        v
    Register


The multiplier and accumulator path may contribute significantly to the critical path.

However, the actual critical path must be determined using timing analysis.


## 34. Data Width Impact

Data width directly affects hardware complexity.

Increasing data width can increase:

    Multiplier Area

    Adder Area

    Register Area

    Routing

    Power


Therefore, the selected data width should provide an appropriate balance between:

    Numerical Range

    Accuracy

    Hardware Cost


The final project data width should be taken directly from the implemented Verilog design.


## 35. Performance and Power Relationship

Higher performance may require:

    Higher Frequency

    Greater Parallelism

    More Hardware


These can increase:

    Dynamic Power


Therefore, performance should not be evaluated independently from power.


A useful future metric is:

    Performance per Watt


## 36. Performance per Area

Another useful accelerator metric is:

    Performance per Area =
        Throughput / Area


For example:

    MAC/s/mm^2


This metric allows different accelerator architectures to be compared more meaningfully.


Actual area and throughput values must come from implementation results.


## 37. Energy per Operation

Energy efficiency can be expressed as:

    Energy per Operation =
        Power / Operations per Second


For example:

    Energy/Operation =
        Watts / Operations per Second


This metric is particularly useful for AI accelerator evaluation.


Actual power and throughput measurements are required before reporting this metric.


## 38. Current Measured Results

The current verified performance-related information is:

    Array Size:
        4x4


    Number of PEs:
        16


    PE MAC:
        Functionally Verified


    Weight Stationary:
        Implemented at PE level


    Complete Array Throughput:
        Pending


    Complete Array Latency:
        Pending


    Maximum Frequency:
        Pending


    Area:
        Pending


    Power:
        Pending


    Energy per Operation:
        Pending


## 39. Performance Results Table

The final performance table should be updated using actual measurements.

    ==================================================
                 PERFORMANCE RESULTS
    ==================================================

    Array Size:
        4x4


    Number of PEs:
        16


    Data Width:
        <Actual RTL Width>


    Clock Frequency:
        <Measured Value>


    Clock Period:
        <Measured Value>


    Latency:
        <Measured Cycles>


    Throughput:
        <Measured MAC/s>


    Peak Compute:
        <Measured Value>


    PE Utilization:
        <Measured Percentage>


    Area:
        <Measured Value>


    Power:
        <Measured Value>


    Energy per Operation:
        <Measured Value>

    ==================================================


No estimated value should be presented as a measured project result.


## 40. Performance Verification

Performance measurements should be correlated with functional correctness.

The procedure is:

    Functional Verification
            |
            v
    Correct Output
            |
            v
    Measure Cycle Count
            |
            v
    Measure Frequency
            |
            v
    Calculate Throughput
            |
            v
    Analyze Utilization


Performance without functional correctness is not considered a valid accelerator result.


## 41. Recommended Performance Tests

The following tests should eventually be included:

    Test 1:
        Zero Matrix


    Test 2:
        Identity Matrix


    Test 3:
        Ones Matrix


    Test 4:
        Small Integer Matrix


    Test 5:
        Maximum Value Matrix


    Test 6:
        Random Matrix


    Test 7:
        Multiple Consecutive Matrix Operations


These tests can reveal both functional and performance issues.


## 42. Consecutive Operation Testing

The accelerator should eventually be tested with multiple matrix operations executed consecutively.

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

            |
            v

           ...


This test helps determine sustained throughput rather than single-operation latency only.


## 43. Steady-State Throughput

Single-operation latency includes startup and drain overhead.

Steady-state throughput measures the rate at which completed operations can be produced during continuous operation.

Therefore:

    Latency
        !=
    Throughput


A systolic accelerator may have a relatively large initial latency while still achieving high sustained throughput.


## 44. Performance Reporting Guidelines

Final project reports should distinguish between:

    Theoretical Peak Performance

and:

    Measured Performance


For example:

    Theoretical:
        16 MAC/cycle


    Measured:
        <Actual MAC/cycle>


Theoretical peak should never be presented as measured silicon or implementation performance.


## 45. Current Project Performance Conclusion

The architecture provides:

    16 Processing Elements

    Parallel MAC Computation

    Weight Reuse

    Local Data Movement

    Structured Systolic Communication

    Potential High Throughput


The PE functionality has already been verified.

Complete accelerator performance remains to be measured after the 4x4 array is integrated and simulated.


## 46. Next Performance Milestones

The next milestones are:

    1. Complete 4x4 Verilog integration.

    2. Verify matrix multiplication.

    3. Measure cycle-level latency.

    4. Determine sustained throughput.

    5. Synthesize the design.

    6. Determine maximum operating frequency.

    7. Measure area.

    8. Measure power.

    9. Calculate performance per area.

    10. Calculate energy per operation.


## 47. Final Performance Checklist

    [ ] Complete 4x4 array implemented

    [ ] Matrix multiplication verified

    [ ] Input/output timing verified

    [ ] Latency measured

    [ ] Throughput measured

    [ ] Maximum frequency measured

    [ ] PE utilization measured

    [ ] Synthesis completed

    [ ] Area measured

    [ ] Power measured

    [ ] Energy per operation calculated

    [ ] Performance per area calculated

    [ ] Final results documented


## 48. Summary

The 4x4 weight-stationary systolic array provides a structured architecture for parallel matrix multiplication.

The key architectural performance characteristics are:

    16 Processing Elements

    Parallel MAC Computation

    Weight Stationary Dataflow

    Local Communication

    Data Reuse

    Systolic Pipeline Operation


For a 4x4 matrix multiplication:

    Number of Output Elements:
        16


    MAC Contributions:
        64


Theoretical peak computational parallelism is:

    16 MAC/cycle


The actual latency, throughput, frequency, area, power, and energy efficiency must be obtained from the completed RTL implementation, simulation, synthesis, and physical implementation flows.

The project therefore maintains a strict distinction between:

    Architectural Capability

    Simulated Performance

    Synthesized Performance

    Physical Implementation Results

    Silicon Results


Only experimentally or tool-generated verified values will be reported as final project performance results.
