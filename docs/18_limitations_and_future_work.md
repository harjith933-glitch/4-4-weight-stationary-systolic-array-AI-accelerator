# Limitations and Future Work

## 1. Overview

This document describes the current limitations of the 4x4 weight-stationary systolic array accelerator and the planned improvements for future development.

The project is being developed incrementally, starting from a verified Processing Element and progressing toward a complete 4x4 systolic array accelerator.

The documentation intentionally distinguishes between:

    Completed Work

    Current Limitations

    Planned Work

    Future Enhancements


## 2. Current Project State

The current project has successfully implemented and verified the Processing Element using Verilog HDL.

Completed:

    Processing Element RTL

    PE Testbench

    Weight Loading

    Weight Storage

    Multiplication

    Partial-Sum Accumulation

    MAC Operation

    Activation Forwarding

    PE-Level Functional Simulation


The complete 4x4 accelerator is still under development.


## 3. Current Architecture

The target architecture consists of:

    4 x 4 Processing Elements

    Total PEs = 16


The architecture uses:

    Weight-Stationary Dataflow

    Systolic Communication

    Local Data Movement

    Parallel MAC Computation


The final objective is to perform matrix multiplication efficiently using the systolic array.


## 4. Current Limitation: PE-Level Verification

The current verified boundary is primarily the Processing Element.

The PE has been functionally simulated and verified.

However, complete system-level verification requires:

    Multi-PE Integration

    Complete 4x4 Array Integration

    Matrix-Level Testing

    Cycle-Level Verification

    Randomized Testing

    Regression Testing


These stages are planned as part of the next development phase.


## 5. Current Limitation: Complete 4x4 Integration

The complete 4x4 array has not yet been fully integrated and verified.

The intended structure is:

    +------+------+------+------+
    | PE00 | PE01 | PE02 | PE03 |
    +------+------+------+------+
    | PE10 | PE11 | PE12 | PE13 |
    +------+------+------+------+
    | PE20 | PE21 | PE22 | PE23 |
    +------+------+------+------+
    | PE30 | PE31 | PE32 | PE33 |
    +------+------+------+------+

The next implementation stage is to connect all 16 PEs and verify their communication.


## 6. Current Limitation: Matrix-Level Verification

Although the PE MAC operation has been verified, complete matrix multiplication must still be demonstrated at the array level.

The target operation is:

    C = A x B


For 4x4 matrices:

    Cij =
        Ai0B0j
        + Ai1B1j
        + Ai2B2j
        + Ai3B3j


A complete reference-model comparison will be performed after array integration.


## 7. Current Limitation: Data Scheduling

A systolic array requires carefully scheduled data movement.

The final implementation must ensure correct alignment of:

    Weights

    Activations

    Partial Sums

    Valid Signals

    Control Signals


Incorrect scheduling can produce mathematically incorrect results even when individual PEs are functionally correct.

Therefore, array-level timing must be verified carefully.


## 8. Current Limitation: Pipeline Timing

The systolic architecture operates through a sequence of clock cycles.

The complete implementation must determine:

    Pipeline Fill Time

    Computation Time

    Pipeline Drain Time

    Total Latency


These values should be measured from the final RTL simulation rather than estimated.


## 9. Current Limitation: Throughput

The theoretical architecture contains:

    16 PEs


Therefore, the array has the potential for:

    16 MAC operations per cycle


However, this is only the theoretical computational capacity.

Actual sustained throughput depends on:

    Data Availability

    Control

    Pipeline Scheduling

    PE Utilization

    Memory Interface

    Clock Frequency


Measured throughput will be reported after complete array verification.


## 10. Current Limitation: No Synthesis Results

The current project has not yet completed the synthesis stage.

Therefore, the following values are currently unavailable:

    Cell Area

    Cell Count

    Gate Count

    Maximum Frequency

    Timing Slack

    Power


These values must be obtained using an actual synthesis flow and technology library.


## 11. Current Limitation: No Static Timing Analysis

Static Timing Analysis has not yet been performed on the complete accelerator.

Therefore, no actual values are currently claimed for:

    Setup Slack

    Hold Slack

    Critical Path

    Maximum Frequency


These will be measured after synthesis and timing constraints are established.


## 12. Current Limitation: No Physical Design

Physical implementation has not yet been performed.

The following stages remain future work:

    Floorplanning

    Power Planning

    Placement

    Clock Tree Synthesis

    Routing

    Parasitic Extraction


Consequently, no physical implementation metrics are currently reported.


## 13. Current Limitation: No DRC/LVS Results

Design Rule Check and Layout Versus Schematic have not yet been performed.

Therefore:

    DRC = Not Performed

    LVS = Not Performed


These checks will become relevant after physical implementation.


## 14. Current Limitation: No GDSII

GDSII generation has not yet been performed.

The project currently remains at the RTL and functional verification stage.

The intended future flow is:

    Verilog RTL
        |
        v
    Synthesis
        |
        v
    Physical Design
        |
        v
    DRC / LVS
        |
        v
    GDSII


## 15. Current Limitation: No Silicon Validation

The project has not been fabricated as an ASIC.

Therefore, the project does not currently claim:

    Silicon Measurements

    Fabricated Chip Results

    Tapeout Results

    Measured Silicon Frequency

    Measured Silicon Power


All current results are based on RTL-level development and simulation.


## 16. Current Limitation: Fixed Array Size

The current target architecture is:

    4x4


This provides:

    16 Processing Elements


A fixed array simplifies the initial implementation and verification process.

However, it limits scalability compared with configurable or parameterized architectures.


## 17. Future Work: Complete 4x4 Integration

The immediate development objective is to complete the 4x4 array.

The planned structure is:

    PE00 -> PE01 -> PE02 -> PE03

    PE10 -> PE11 -> PE12 -> PE13

    PE20 -> PE21 -> PE22 -> PE23

    PE30 -> PE31 -> PE32 -> PE33


The integration must correctly implement:

    Activation Propagation

    Weight Mapping

    Partial-Sum Flow

    Control

    Reset

    Output Collection


## 18. Future Work: Two-PE Verification

Before verifying the complete array, a smaller integration test can be used.

Structure:

    PE0 -> PE1


The purpose is to verify:

    Inter-PE Communication

    Activation Timing

    Partial-Sum Timing

    Control Alignment


This provides an intermediate verification stage between PE-level and array-level verification.


## 19. Future Work: Four-PE Verification

After successful two-PE verification, a four-PE chain can be tested.

Structure:

    PE0 -> PE1 -> PE2 -> PE3


This test increases confidence in:

    Systolic Data Propagation

    Cycle Alignment

    Repeated PE Connectivity


## 20. Future Work: Complete Array Verification

The complete 4x4 array will be tested after multi-PE verification.

Verification will include:

    Directed Tests

    Matrix Multiplication

    Boundary Tests

    Random Tests

    Regression


The outputs will be compared against an independent reference model.


## 21. Future Work: Identity Matrix Test

The identity matrix provides a useful verification case.

The mathematical property is:

    A x I = A


Therefore, if one matrix is the identity matrix, the output should match the other input matrix.

This test helps verify:

    Weight Mapping

    Data Scheduling

    Output Mapping


## 22. Future Work: Zero Matrix Test

The zero matrix provides another simple verification case.

The mathematical property is:

    A x 0 = 0


Therefore, all output values should be zero.

This test helps detect:

    Initialization Problems

    Accumulation Errors

    Dataflow Errors


## 23. Future Work: Ones Matrix Test

A matrix containing only ones provides a predictable accumulation pattern.

For 4x4 matrices containing ones:

    Each output =
        1x1 + 1x1 + 1x1 + 1x1


Therefore:

    Output = 4


This provides a simple test for accumulation and data scheduling.


## 24. Future Work: Boundary Testing

Boundary testing will use the minimum and maximum values supported by the implemented data widths.

The purpose is to identify:

    Overflow

    Truncation

    Sign Errors

    Width Mismatch

    Arithmetic Errors


Boundary testing is important before random testing.


## 25. Future Work: Randomized Verification

Randomized testing will be used to increase verification coverage.

The general flow is:

    Random Input Matrices
            |
            +----------------+
            |                |
            v                v
    Reference Model          RTL
            |                |
            v                v
    Expected Results       Actual Results
            |                |
            +-------+--------+
                    |
                    v
                 Compare


The test should report mismatches automatically.


## 26. Future Work: Regression Testing

A regression suite will eventually execute all important tests automatically.

The regression suite should include:

    Reset

    Weight Loading

    MAC

    Activation Forwarding

    Two-PE Integration

    Four-PE Integration

    Identity Matrix

    Zero Matrix

    Ones Matrix

    Boundary Testing

    Random Testing


A test should be considered passing only when the expected and actual results match.


## 27. Future Work: Parameterization

A future version may make the architecture parameterized.

Potential parameters include:

    Array Size

    Data Width

    Weight Width

    Partial-Sum Width


For example:

    ARRAY_SIZE = 4

could potentially be changed to:

    ARRAY_SIZE = 8


This would allow the same RTL architecture to support multiple configurations.


## 28. Future Work: Configurable Data Width

The current implementation can be extended to support configurable numerical precision.

Potential configurations include:

    8-bit

    16-bit

    Other application-specific widths


Changing data width affects:

    Area

    Power

    Timing

    Numerical Range


Therefore, each configuration should be independently verified.


## 29. Future Work: Improved Control Architecture

The control logic can be enhanced to support:

    Multiple Matrix Operations

    Continuous Data Streaming

    Start/Done Signaling

    Valid/Ready Interfaces

    Better Pipeline Control


The goal is to improve usability and sustained throughput.


## 30. Future Work: Streaming Interface

A future implementation could use streaming interfaces for:

    Activations

    Weights

    Outputs


Conceptually:

    Input Stream
        |
        v
    Systolic Array
        |
        v
    Output Stream


This would make the accelerator easier to integrate into a larger SoC or AI-processing system.


## 31. Future Work: Memory Interface

A practical accelerator requires interaction with memory.

Future development may include:

    Input Buffer

    Weight Buffer

    Output Buffer

    SRAM Interface

    DMA Interface


The objective is to reduce unnecessary external memory traffic.


## 32. Future Work: On-Chip Buffering

Local buffering can improve data reuse.

A future architecture may contain:

    Input Buffer
          |
          v
    Weight Buffer
          |
          v
    Systolic Array
          |
          v
    Output Buffer


This would provide a more complete accelerator subsystem.


## 33. Future Work: Quantization

The accelerator can potentially be extended for reduced-precision computation.

Possible future approaches include:

    INT8

    INT16

    Other Fixed-Point Formats


Lower precision can potentially reduce:

    Area

    Power

    Memory Traffic


while increasing computational density.


## 34. Future Work: Quantization Accuracy Analysis

If reduced precision is implemented, the project should compare:

    Floating-Point Reference

against:

    Fixed-Point / Integer Hardware


The analysis should evaluate:

    Numerical Error

    Accuracy

    Dynamic Range


This would make the accelerator more relevant to practical AI workloads.


## 35. Future Work: Synthesis

After RTL verification, synthesis should be performed.

The flow will be:

    Verilog RTL
        |
        v
    Synthesis
        |
        v
    Gate-Level Netlist


The resulting reports should include:

    Area

    Cell Count

    Timing

    Power Estimates


## 36. Future Work: Static Timing Analysis

After synthesis, STA should be performed.

Important metrics include:

    Worst Setup Slack

    Worst Hold Slack

    Critical Path

    Maximum Frequency


These values should be extracted directly from the STA reports.


## 37. Future Work: Physical Design

A future ASIC implementation can proceed through:

    Floorplanning

    Power Planning

    Placement

    Clock Tree Synthesis

    Routing

    Parasitic Extraction


The physical design should preserve the regularity of the systolic array where practical.


## 38. Future Work: Physical Verification

After routing, the design should undergo:

    DRC

    LVS

    Post-Route STA

    Power Analysis


Only after successful verification should final physical data be considered complete.


## 39. Future Work: GDSII Generation

The final physical implementation can be exported as GDSII.

The intended flow is:

    RTL
        |
        v
    Synthesis
        |
        v
    Physical Implementation
        |
        v
    DRC / LVS
        |
        v
    GDSII


This would represent a major future milestone for the project.


## 40. Future Work: Performance Benchmarking

Once the complete array is operational, detailed performance measurements should be performed.

Metrics include:

    Latency

    Throughput

    MAC/s

    Operations/s

    PE Utilization

    Clock Frequency

    Area

    Power

    Energy per Operation


These metrics should be measured using actual simulation and implementation data.


## 41. Future Work: Baseline Comparison

The systolic architecture can eventually be compared with a simpler sequential matrix multiplication implementation.

Comparison parameters:

    Latency

    Throughput

    Area

    Power

    Parallelism

    Resource Utilization


The comparison should use equivalent:

    Data Width

    Matrix Size

    Technology

    Clock Conditions


where applicable.


## 42. Future Work: Larger Arrays

The architecture can eventually be extended beyond 4x4.

Examples:

    4x4
        = 16 PEs


    8x8
        = 64 PEs


    16x16
        = 256 PEs


Larger arrays can provide greater parallelism but also increase:

    Area

    Power

    Routing Complexity

    Verification Complexity


## 43. Future Work: Hardware-Software Integration

A future version could integrate the accelerator with a processor or SoC.

A possible architecture is:

    Processor
        |
        v
    Control Interface
        |
        v
    Systolic Accelerator
        |
        v
    Output Buffer


The processor could configure:

    Matrix Dimensions

    Data Addresses

    Start Command

    Control Registers


The accelerator could provide:

    Busy

    Done

    Interrupt

    Status


## 44. Future Work: Real AI Workloads

After basic matrix multiplication verification, the accelerator could be evaluated using workloads inspired by:

    Neural Network Layers

    Convolution

    Fully Connected Layers

    Matrix Multiplication


Convolution operations can potentially be transformed into matrix multiplication through appropriate data transformations.


## 45. Future Work: Convolution Acceleration

A future extension could map convolution operations onto the systolic array.

Conceptually:

    Input Feature Maps
            |
            v
    Data Transformation
            |
            v
    Matrix Representation
            |
            v
    Systolic Array
            |
            v
    Output Feature Maps


This would increase the practical relevance of the accelerator for AI applications.


## 46. Future Work: Power Optimization

After functional correctness is established, power optimization can be explored.

Potential techniques include:

    Clock Gating

    Data Gating

    Reduced Precision

    Lower Switching Activity

    Local Data Reuse


Optimization should be guided by actual power reports.


## 47. Future Work: Area Optimization

Area optimization may include:

    Logic Simplification

    Resource Sharing

    Cell Optimization

    Data Width Optimization

    Architecture Optimization


The goal is to maintain required performance while reducing hardware cost.


## 48. Future Work: Timing Optimization

Timing optimization may include:

    Critical Path Optimization

    Pipeline Optimization

    Cell Sizing

    Buffer Insertion

    Placement Optimization

    Routing Optimization


The appropriate technique depends on the actual timing report.


## 49. Future Work: Verification Coverage

Future verification should track coverage across:

    Functional Cases

    Data Values

    Control States

    PE Interactions

    Matrix Configurations

    Boundary Conditions


Coverage metrics should be added when the verification environment becomes sufficiently mature.


## 50. Future Work: Automated Verification

A future verification environment should automatically:

    Generate Inputs

    Calculate Reference Results

    Run Simulation

    Compare Outputs

    Report Failures

    Produce Regression Summaries


This would make the project easier to maintain and reproduce.


## 51. Future Work: Continuous Integration

Once the RTL and testbench are stable, a CI workflow can automatically execute basic tests whenever code changes are pushed.

Conceptually:

    Git Push
        |
        v
    Automated Build
        |
        v
    Verilog Compilation
        |
        v
    Simulation
        |
        v
    Test Result
        |
        v
    PASS / FAIL


This can improve project reliability and make the GitHub repository more industry-oriented.


## 52. Future Work: Documentation Improvements

As the project develops, documentation should be updated with actual evidence.

Future documentation may include:

    RTL Screenshots

    Waveform Screenshots

    Simulation Logs

    Synthesis Reports

    Timing Reports

    Area Reports

    Power Reports

    Physical Design Screenshots

    DRC Results

    LVS Results


The documentation should always distinguish measured results from theoretical expectations.


## 53. Current Limitations Summary

Current major limitations are:

    Complete 4x4 array verification is pending.

    Matrix-level verification is pending.

    Complete throughput measurement is pending.

    Maximum operating frequency is not yet measured.

    Synthesis has not yet been completed.

    STA has not yet been completed.

    Physical design has not yet been completed.

    DRC/LVS have not yet been performed.

    GDSII has not yet been generated.

    Silicon validation has not been performed.


These limitations represent the current development boundary rather than failures of the architecture.


## 54. Development Roadmap

The overall development roadmap is:

    PE RTL
        |
        | COMPLETED
        v
    PE Verification
        |
        | COMPLETED
        v
    Multi-PE Verification
        |
        | NEXT
        v
    Complete 4x4 Array
        |
        v
    Matrix-Level Verification
        |
        v
    Randomized Verification
        |
        v
    Regression
        |
        v
    Synthesis
        |
        v
    STA
        |
        v
    Physical Design
        |
        v
    DRC / LVS
        |
        v
    GDSII
        |
        v
    Future Silicon / FPGA Validation


## 55. Priority of Future Work

The development priorities are:

    Priority 1:
        Complete 4x4 Verilog integration.


    Priority 2:
        Verify matrix multiplication.


    Priority 3:
        Complete directed and randomized verification.


    Priority 4:
        Establish regression testing.


    Priority 5:
        Perform synthesis.


    Priority 6:
        Analyze timing, area, and power.


    Priority 7:
        Perform physical implementation.


    Priority 8:
        Complete physical verification.


    Priority 9:
        Generate GDSII if a suitable technology flow is available.


## 56. Project Maturity Levels

The project can be viewed as progressing through the following maturity levels:

    Level 1:
        PE RTL


    Level 2:
        PE Functional Verification


    Level 3:
        Multi-PE Integration


    Level 4:
        Complete 4x4 Functional Verification


    Level 5:
        Synthesis


    Level 6:
        Timing / Power / Area Analysis


    Level 7:
        Physical Implementation


    Level 8:
        Physical Verification


    Level 9:
        GDSII


    Level 10:
        Silicon / FPGA Validation


Current project maturity:

    Level 2


The project will progress to higher levels as the corresponding stages are actually completed.


## 57. Engineering Philosophy

The project follows an incremental hardware-development methodology:

    Build Small
        |
        v
    Verify
        |
        v
    Integrate
        |
        v
    Verify Again
        |
        v
    Optimize
        |
        v
    Implement
        |
        v
    Sign Off


This approach reduces debugging complexity and improves confidence in the final hardware.


## 58. Avoiding Unsupported Claims

All future project documentation should follow one important rule:

    Never report an unmeasured value as an actual result.


For example:

    Correct:
        "The architecture provides a theoretical
        capacity of 16 MAC operations per cycle."


    Incorrect:
        "The accelerator achieves 16 MAC/cycle."


unless sustained performance has actually been measured.


Similarly:

    Correct:
        "ASIC implementation is planned."


    Incorrect:
        "The design is ASIC-ready."


unless the required implementation and signoff stages have actually been completed.


## 59. Final Future Vision

The long-term objective is to evolve the current PE-level design into a complete and verifiable hardware accelerator.

The intended progression is:

    Verified PE
        |
        v
    Verified 4x4 Systolic Array
        |
        v
    Synthesized Accelerator
        |
        v
    Timing / Power / Area Analysis
        |
        v
    Physically Implemented Design
        |
        v
    DRC / LVS Clean Layout
        |
        v
    GDSII
        |
        v
    Potential FPGA / Silicon Validation


## 60. Conclusion

The project has established a functional Processing Element as the fundamental building block of the 4x4 weight-stationary systolic array.

The PE has been verified for:

    Weight Loading

    Weight Storage

    Multiplication

    Partial-Sum Accumulation

    MAC Operation

    Activation Forwarding


The major remaining work is to integrate the PEs into the complete 4x4 architecture and verify matrix multiplication at the array level.

Following functional verification, the project can progress toward:

    Synthesis

    Static Timing Analysis

    Area Analysis

    Power Analysis

    Physical Design

    DRC

    LVS

    GDSII


The project documentation will be updated incrementally as each stage is actually completed.

The current goal is therefore:

    Build a correctly functioning 4x4 Verilog
    weight-stationary systolic array first,

followed by:

    Measure and optimize its implementation,

and finally:

    Demonstrate a complete ASIC-oriented
    hardware implementation flow.
