# Synthesis and Implementation

## 1. Overview

This document describes the synthesis and ASIC implementation flow planned for the 4x4 weight-stationary systolic array accelerator.

The accelerator is designed using Verilog RTL.

The implementation flow follows the standard digital ASIC design methodology:

    Verilog RTL
        |
        v
    RTL Simulation
        |
        v
    Logic Synthesis
        |
        v
    Gate-Level Netlist
        |
        v
    Static Timing Analysis
        |
        v
    Floorplanning
        |
        v
    Power Planning
        |
        v
    Placement
        |
        v
    Clock Tree Synthesis
        |
        v
    Routing
        |
        v
    Physical Verification
        |
        v
    GDSII


## 2. Current Implementation Status

The project is currently at the RTL development and functional verification stage.

Completed:

    Verilog RTL development

    Processing Element implementation

    Processing Element testbench

    PE functional simulation

    PE MAC verification

    Weight loading verification

    Activation forwarding verification


Not yet completed:

    Complete 4x4 array implementation

    RTL synthesis

    Gate-level netlist generation

    Static timing analysis

    Floorplanning

    Placement

    Clock tree synthesis

    Routing

    DRC

    LVS

    GDSII generation


Therefore, no physical implementation results are claimed at this stage.


## 3. RTL to GDSII Flow

The complete intended ASIC flow is:

    RTL Design
        |
        v
    Functional Verification
        |
        v
    Synthesis
        |
        v
    Gate-Level Netlist
        |
        v
    Floorplan
        |
        v
    Power Planning
        |
        v
    Placement
        |
        v
    Clock Tree Synthesis
        |
        v
    Routing
        |
        v
    Physical Verification
        |
        v
    GDSII


Each stage has a specific purpose.


## 4. RTL Design

The design begins with Verilog RTL.

The RTL describes the intended hardware behavior of the systolic array.

The major hardware block is the Processing Element.

The PE performs:

    PSUM_out =
        PSUM_in + (Activation x Weight)


The complete accelerator is constructed by connecting multiple PEs into a 4x4 array.


## 5. RTL Functional Verification

Before synthesis, the RTL must be functionally verified.

The current verification approach starts from the PE.

The PE testbench verifies:

    Reset

    Weight Loading

    Weight Storage

    Multiplication

    Partial-Sum Accumulation

    MAC Operation

    Activation Forwarding


After PE verification, the complete array will be verified using matrix-level tests.


## 6. Synthesis

Synthesis converts the RTL description into a gate-level representation.

Conceptually:

    Verilog RTL
         |
         v
    Logic Synthesis
         |
         v
    Gate-Level Netlist


The synthesis tool maps RTL operations to cells from a target standard-cell library.


For example, RTL operations may be mapped to:

    Flip-Flops

    Logic Gates

    Multipliers or synthesized multiplier logic

    Adders

    Buffers

    Other Standard Cells


The actual mapped cells depend on the selected technology library and synthesis constraints.


## 7. Synthesis Inputs

A typical synthesis flow requires:

    Verilog RTL

    Standard Cell Library

    Timing Constraints

    Clock Definition

    Input Constraints

    Output Constraints

    Operating Conditions


The exact files depend on the selected technology and EDA tool.


## 8. Standard Cell Library

The synthesis tool uses a standard-cell library to map the RTL design into physical cells.

A library may contain:

    AND Gates

    OR Gates

    NAND Gates

    NOR Gates

    XOR Gates

    Multiplexers

    Flip-Flops

    Buffers

    Inverters


The library also provides timing and power information for the cells.


## 9. Technology Dependency

RTL is largely technology-independent.

Physical implementation is technology-dependent.

Therefore:

    Verilog RTL
        |
        v
    Technology Independent


while:

    Standard Cell Mapping
        |
        v
    Technology Dependent


The final area, timing, and power results depend on the target technology library.


## 10. Synthesis Optimization

During synthesis, the tool attempts to optimize the design according to constraints.

Possible objectives include:

    Timing

    Area

    Power


The synthesis tool may transform the RTL logic while preserving its functional behavior.


## 11. Timing Constraints

Timing constraints describe the expected operating conditions of the design.

Important constraints include:

    Clock Period

    Clock Frequency

    Input Delay

    Output Delay

    Clock Uncertainty

    Input Drive

    Output Load


A typical clock constraint can be represented conceptually as:

    create_clock


The exact constraint syntax depends on the synthesis tool.


## 12. Clock Definition

The accelerator is a synchronous digital design.

Therefore, the clock must be explicitly defined during synthesis and timing analysis.

Conceptually:

    Clock
      |
      +----------------+
      |                |
      v                v
    PE00             PE01
      |                |
      v                v
     ...


All sequential elements must operate according to the defined clock.


## 13. Timing Analysis

After synthesis, timing analysis determines whether the design can operate at the required clock frequency.

Important timing concepts include:

    Setup Time

    Hold Time

    Clock-to-Q Delay

    Combinational Delay

    Clock Skew

    Slack


The main objective is to ensure that timing constraints are satisfied.


## 14. Setup Timing

For a synchronous path, data must arrive at the receiving flip-flop before the active clock edge.

Conceptually:

    Launch FF
        |
        v
    Combinational Logic
        |
        v
    Capture FF


The data must arrive within the available clock period.


## 15. Setup Slack

Setup slack can be represented conceptually as:

    Setup Slack =
        Required Time - Arrival Time


Positive slack indicates that the path meets the setup requirement.

Negative slack indicates a setup timing violation.


Example:

    Required Time = 10 ns

    Arrival Time = 9 ns


Therefore:

    Slack =
        10 - 9

    Slack = +1 ns


This path satisfies the setup requirement.


## 16. Hold Timing

Hold analysis verifies that data does not change too early after the receiving clock edge.

The timing path must satisfy the hold requirement.

A hold violation may require changes such as:

    Additional Delay

    Buffer Insertion

    Cell Sizing

    Routing Adjustment


The exact correction depends on the physical implementation.


## 17. Static Timing Analysis

Static Timing Analysis, or STA, checks timing behavior without applying simulation vectors.

It analyzes timing paths mathematically using:

    Cell Delays

    Interconnect Delays

    Clock Information

    Timing Constraints


STA is used throughout the implementation flow.


## 18. Timing Paths

Important paths in the accelerator may include:

    Input Register
        |
        v
    PE Computation
        |
        v
    Output Register


and:

    PE Register
        |
        v
    Next PE
        |
        v
    Next Register


The critical path must be identified after synthesis and implementation.


## 19. Critical Path

The critical path is the timing path with the smallest timing margin.

For the systolic array, the critical path may be influenced by:

    MAC Logic

    Adder Delay

    Multiplier Logic

    Routing

    Register-to-register paths


The actual critical path must be determined using the synthesis or STA report.


## 20. Frequency Estimation

The maximum operating frequency is related to the critical timing path.

Conceptually:

    Maximum Frequency
        ≈
    1 / Minimum Clock Period


For example, if the minimum achievable clock period were:

    10 ns


then:

    Frequency =
        1 / 10 ns


    Frequency =
        100 MHz


This is only an example.

No actual project frequency should be claimed until timing analysis is performed.


## 21. Area Analysis

After synthesis, the design area can be estimated from the mapped standard cells.

Area may include:

    Sequential Cells

    Combinational Cells

    Buffers

    Other Supporting Cells


The total area depends on:

    Technology

    Cell Library

    Synthesis Constraints

    RTL Architecture


## 22. Power Analysis

Power analysis estimates the power consumed by the design.

Power can generally be divided into:

    Dynamic Power

    Leakage Power


Dynamic power is associated with signal switching.

Leakage power is associated with transistor leakage when cells are powered.


The actual power must be obtained from an appropriate power-analysis flow.


## 23. Floorplanning

After synthesis, physical implementation begins with floorplanning.

Floorplanning determines:

    Core Size

    Aspect Ratio

    Macro Locations

    I/O Locations

    Placement Regions


The systolic array contains a regular grid of PEs, which makes its structure naturally suitable for structured physical planning.


## 24. Systolic Array Physical Structure

The logical structure is:

    +------+------+------+------+
    | PE00 | PE01 | PE02 | PE03 |
    +------+------+------+------+
    | PE10 | PE11 | PE12 | PE13 |
    +------+------+------+------+
    | PE20 | PE21 | PE22 | PE23 |
    +------+------+------+------+
    | PE30 | PE31 | PE32 | PE33 |
    +------+------+------+------+
    

The physical implementation can preserve this regular organization where practical.


## 25. Placement

Placement determines the physical location of standard cells inside the core.

The goal is to achieve:

    Timing

    Area Efficiency

    Routability

    Power Efficiency


The regular PE structure may help simplify placement and routing.


## 26. Placement Considerations

Important physical considerations include:

    PE-to-PE Distance

    Clock Distribution

    Data Routing

    Partial-Sum Routing

    Activation Routing

    Power Distribution


Long interconnects can increase delay and power.


## 27. Clock Tree Synthesis

Clock Tree Synthesis, or CTS, distributes the clock signal to sequential elements.

Conceptually:

    Clock Source
         |
         v
    Clock Tree
      / | \
     /  |  \
    v   v   v
    FF  FF  FF


The objective is to control:

    Clock Skew

    Clock Latency

    Transition


A well-balanced clock network is important for reliable timing.


## 28. Clock Skew

Clock skew is the difference in clock arrival time between sequential elements.

Ideally, clock arrival should be controlled so that timing requirements are satisfied.

Excessive skew can cause:

    Setup Violations

    Hold Violations


CTS attempts to manage these effects.


## 29. Routing

Routing connects all placed cells.

Routing includes:

    Signal Routing

    Clock Routing

    Power Routing


For the systolic array, important signal routes include:

    Activation Paths

    Partial-Sum Paths

    Control Signals

    Clock


## 30. Routing Congestion

Routing congestion occurs when too many connections compete for limited routing resources.

Potential causes include:

    High Cell Density

    Long Interconnects

    Poor Placement

    High Connectivity


The regular systolic structure can help organize communication, but actual congestion must be evaluated after placement and routing.


## 31. Design Rule Check

Design Rule Check, or DRC, verifies that the physical layout follows the manufacturing rules of the target technology.

DRC can check:

    Minimum Width

    Minimum Spacing

    Via Rules

    Metal Rules

    Other Technology-Specific Constraints


A design must pass required DRC checks before final tapeout.


## 32. Layout Versus Schematic

Layout Versus Schematic, or LVS, verifies that the physical implementation corresponds to the intended logical design.

Conceptually:

    RTL / Netlist
          |
          | Compare
          v
    Physical Layout


The objective is to confirm connectivity and device-level correspondence according to the technology flow.


## 33. Parasitic Extraction

After routing, physical interconnect introduces parasitic effects.

These may include:

    Resistance

    Capacitance


Parasitic extraction generates information used for more accurate timing and power analysis.


## 34. Post-Route Timing Analysis

After routing, timing should be checked again.

The flow is:

    Placement
        |
        v
    CTS
        |
        v
    Routing
        |
        v
    Parasitic Extraction
        |
        v
    Post-Route STA


Post-route timing is more realistic because interconnect effects are included.


## 35. Timing Closure

Timing closure means that the design satisfies the required timing constraints.

Typical goals include:

    Setup Slack >= 0

    Hold Slack >= 0


along with acceptable:

    Clock Skew

    Transition

    Capacitance


If timing fails, the design may require optimization.


## 36. Timing Optimization

Possible optimization methods include:

    Cell Upsizing

    Buffer Insertion

    Logic Optimization

    Placement Optimization

    Routing Optimization

    Register Pipelining


The appropriate method depends on the type and location of the violation.


## 37. Area Optimization

If the design occupies excessive area, possible approaches include:

    Logic Simplification

    Cell Optimization

    Sharing Hardware Where Appropriate

    Removing Unnecessary Logic

    Improving RTL Architecture


Area optimization must not compromise required performance.


## 38. Power Optimization

Possible power optimization techniques include:

    Reducing Switching Activity

    Clock Gating

    Logic Optimization

    Voltage Scaling Where Supported

    Reducing Unnecessary Transitions


Any optimization must preserve the intended functionality.


## 39. Accelerator-Specific Considerations

The systolic architecture has several implementation advantages.

The architecture contains:

    Regular PE Structure

    Local Data Movement

    Repeated MAC Operations

    Structured Interconnect

    Weight Stationary Dataflow


Local communication can reduce unnecessary long-distance data movement.


## 40. Data Movement and Physical Design

The architecture is designed so that data moves between neighboring PEs.

Conceptually:

    Activation
        |
        v
    PE00 -> PE01 -> PE02 -> PE03
        |
        v
       ...


Partial sums also move through the computational structure according to the selected mapping.

This regular communication pattern can be beneficial during physical implementation.


## 41. PE Replication

The complete accelerator contains:

    4 x 4

    = 16 Processing Elements


The repeated structure makes the architecture scalable.

For example:

    2x2

    4x4

    8x8

    16x16


could be constructed using the same general PE concept, subject to architectural and physical constraints.


## 42. Scalability Considerations

Increasing the array size increases:

    Number of PEs

    Number of Connections

    Routing Resources

    Power Consumption

    Area

    Data Scheduling Complexity


Therefore, larger arrays require careful physical planning.


## 43. Synthesis Verification

After synthesis, the synthesized netlist should be verified to ensure that synthesis preserved functionality.

Possible verification approaches include:

    RTL vs Gate-Level Simulation

    Formal Equivalence Checking

    Netlist Simulation


The exact approach depends on the available EDA tools.


## 44. Gate-Level Simulation

Gate-level simulation can use the synthesized netlist instead of the RTL.

Conceptually:

    RTL
      |
      v
    Synthesis
      |
      v
    Gate-Level Netlist
      |
      v
    Gate-Level Simulation


This provides additional confidence that synthesis produced the intended logic.


## 45. Equivalence Checking

Formal equivalence checking can compare:

    RTL Design

against:

    Synthesized Netlist


The objective is to confirm that the synthesized implementation preserves the intended functionality.


## 46. Physical Verification Flow

The intended physical verification sequence is:

    Synthesis
        |
        v
    Floorplan
        |
        v
    Placement
        |
        v
    CTS
        |
        v
    Routing
        |
        v
    DRC
        |
        v
    LVS
        |
        v
    Signoff


Each stage must meet its respective requirements before progressing.


## 47. GDSII

GDSII is a standard layout database format used to represent integrated-circuit physical geometry.

The final physical implementation can be exported as:

    GDSII


The intended final flow is:

    RTL
      |
      v
    Synthesis
      |
      v
    Physical Design
      |
      v
    Verification
      |
      v
    GDSII


The current project has not yet reached GDSII generation.


## 48. Signoff

Before a design is considered ready for fabrication, appropriate signoff checks are required.

Depending on the technology and flow, these may include:

    Functional Verification

    STA

    DRC

    LVS

    Power Analysis

    Signal Integrity

    Other Technology-Specific Checks


The exact signoff requirements depend on the target process.


## 49. Technology and PDK

Physical implementation requires a Process Design Kit, commonly called a PDK.

The PDK provides technology-specific information required by the EDA tools.

Depending on the process, this may include:

    Design Rules

    Standard Cell Libraries

    Timing Models

    Physical Abstracts

    Extraction Rules

    Verification Rules


No specific foundry or process node is assumed for the current RTL project.


## 50. Open-Source Implementation Possibility

The RTL can eventually be taken through an open-source ASIC implementation flow if a compatible technology and PDK are available.

A possible flow can include:

    Verilog RTL
        |
        v
    Synthesis
        |
        v
    Floorplan
        |
        v
    Placement
        |
        v
    CTS
        |
        v
    Routing
        |
        v
    DRC / LVS
        |
        v
    GDSII


The actual tools and technology must be documented when implementation is performed.


## 51. Current Project Boundary

The current project should be clearly separated into three categories.

COMPLETED:

    PE Verilog RTL

    PE Testbench

    PE Functional Simulation

    PE MAC Verification

    Weight Loading Verification

    Activation Forwarding Verification


IN PROGRESS / NEXT:

    Multi-PE Integration

    4x4 Array RTL

    Array-Level Verification


FUTURE:

    Synthesis

    STA

    Physical Design

    DRC

    LVS

    GDSII


## 52. No Fabrication Claims

The project currently does not claim:

    Silicon Fabrication

    Tapeout

    ASIC Prototype

    Fabricated Chip Results


unless these activities are actually completed.

The current project is an RTL-level hardware accelerator development project progressing toward ASIC implementation.


## 53. Implementation Metrics

Once synthesis and physical implementation are performed, the following metrics should be recorded:

    Cell Area

    Total Area

    Cell Count

    Flip-Flop Count

    Combinational Cell Count

    Maximum Frequency

    Clock Period

    Setup Slack

    Hold Slack

    Dynamic Power

    Leakage Power

    Total Power

    Utilization

    Routing Congestion


Actual values should be taken directly from tool reports.


## 54. Implementation Results Table

The following table should be updated after implementation.

    ==============================================
             IMPLEMENTATION RESULTS
    ==============================================

    Technology:
        <Technology / PDK>


    Standard Cell Library:
        <Library>


    Total Cell Area:
        <Actual Value>


    Utilization:
        <Actual Value>


    Clock Period:
        <Actual Value>


    Maximum Frequency:
        <Actual Value>


    Worst Setup Slack:
        <Actual Value>


    Worst Hold Slack:
        <Actual Value>


    Dynamic Power:
        <Actual Value>


    Leakage Power:
        <Actual Value>


    Total Power:
        <Actual Value>


    DRC:
        <PASS / FAIL>


    LVS:
        <PASS / FAIL>


    GDSII:
        <Generated / Not Generated>

    ==============================================


No placeholder value should be presented as an actual result.


## 55. Implementation Optimization Goals

The implementation should aim for a balanced design.

Primary objectives:

    Correct Functionality

    Timing Closure

    Reasonable Area

    Controlled Power

    Reliable Routing


The final optimization priority depends on the intended accelerator application.


## 56. Hardware Accelerator Perspective

The systolic array is intended to accelerate matrix multiplication by exploiting:

    Parallel MAC Operations

    Data Reuse

    Local Communication

    Pipelined Data Movement

    Weight Stationary Dataflow


The physical implementation should preserve these architectural advantages where practical.


## 57. Relationship Between RTL and Physical Design

The RTL architecture directly affects physical implementation.

For example:

    More PEs
        ->
    More Area


    More Interconnect
        ->
    More Routing Complexity


    More Parallel Computation
        ->
    Higher Potential Throughput


    More Switching
        ->
    Potentially Higher Dynamic Power


Therefore, architectural decisions made at RTL influence:

    Area

    Timing

    Power

    Routability


## 58. Design-for-Implementation Considerations

During RTL development, the following should be considered:

    Avoid unnecessary logic.

    Keep data widths controlled.

    Maintain regular PE structure.

    Keep interfaces clearly defined.

    Avoid unnecessary long combinational paths.

    Make clocked behavior explicit.

    Maintain synthesizable Verilog coding style.


These practices make the RTL easier to synthesize and implement.


## 59. Current Implementation Roadmap

The planned implementation roadmap is:

    1. Complete PE Verification
            |
            v

    2. Implement Multi-PE Structure
            |
            v

    3. Implement Complete 4x4 Array
            |
            v

    4. Verify Matrix Multiplication
            |
            v

    5. Synthesize RTL
            |
            v

    6. Analyze Area and Timing
            |
            v

    7. Perform Floorplanning
            |
            v

    8. Perform Placement
            |
            v

    9. Perform CTS
            |
            v

    10. Perform Routing
            |
            v

    11. Run DRC and LVS
            |
            v

    12. Perform Signoff
            |
            v

    13. Generate GDSII


## 60. Implementation Completion Criteria

The implementation stage should eventually satisfy:

    [ ] RTL fully verified

    [ ] Complete 4x4 array implemented

    [ ] Synthesis completed

    [ ] No unintended synthesis errors

    [ ] Timing constraints defined

    [ ] STA completed

    [ ] Floorplan completed

    [ ] Placement completed

    [ ] CTS completed

    [ ] Routing completed

    [ ] Post-route timing analyzed

    [ ] DRC passed

    [ ] LVS passed

    [ ] Power analyzed

    [ ] Final implementation reports documented

    [ ] GDSII generated


## 61. Documentation of Results

Every implementation stage should produce documented evidence.

Recommended evidence includes:

    Synthesis Report

    Area Report

    Timing Report

    Power Report

    Floorplan Screenshot

    Placement Screenshot

    CTS Report

    Routing Screenshot

    DRC Report

    LVS Report

    Final GDSII Information


Only actual generated reports should be uploaded as project evidence.


## 62. Reproducibility

A professional implementation repository should document:

    RTL Source

    Constraints

    Library Information

    Tool Versions

    Commands

    Configuration Files

    Reports

    Verification Results


Another engineer should be able to understand how the implementation was produced.


## 63. Industry Relevance

The project demonstrates understanding of the complete digital hardware development flow:

    Architecture

    RTL Design

    Functional Verification

    Synthesis

    Timing Analysis

    Physical Design

    Physical Verification

    GDSII


The current implementation stage is primarily RTL and functional verification, with physical implementation planned as a future stage.


## 64. Summary

The 4x4 weight-stationary systolic array is being developed using a standard digital hardware design methodology.

The current completed work includes:

    Verilog RTL

    Processing Element

    PE Testbench

    PE Simulation

    Functional Verification


The intended future implementation flow is:

    RTL
        |
        v
    Synthesis
        |
        v
    STA
        |
        v
    Floorplan
        |
        v
    Placement
        |
        v
    CTS
        |
        v
    Routing
        |
        v
    DRC / LVS
        |
        v
    GDSII


No synthesis, physical-design, timing, power, DRC, LVS, or GDSII results are claimed until those stages are actually executed.

The immediate engineering objective remains:

    Complete the verified 4x4 Verilog systolic array

followed by:

    Array-level verification

and then:

    ASIC synthesis and implementation.
