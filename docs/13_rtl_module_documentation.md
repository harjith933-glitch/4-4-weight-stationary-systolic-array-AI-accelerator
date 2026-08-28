# RTL Module Documentation

## 1. Overview

This document describes the Verilog RTL modules used in the 4x4 weight-stationary systolic array accelerator.

The RTL is organized hierarchically so that individual hardware blocks can be developed and verified independently before being integrated into the complete accelerator.

The fundamental computational block is the Processing Element (PE).

The intended hierarchy is:

    Top-Level Systolic Array
            |
            +-----------------------+
            |                       |
            v                       v
       PE Instances            Control / Routing
            |
            v
       MAC Operation
            |
            v
       Partial Sum


## 2. HDL

The project is implemented using:

    Verilog HDL

The project does not use SystemVerilog.

All RTL modules should therefore use Verilog-compatible syntax and constructs.


## 3. RTL Directory

The main RTL source files are maintained under:

    rtl/


The RTL directory contains the synthesizable hardware description files.

The testbench files are maintained separately under:

    tb/


Simulation-generated files are maintained under:

    sim/


Waveform files are maintained under:

    wave/


Documentation is maintained under:

    docs/


## 4. Design Hierarchy

The intended hardware hierarchy is:

    systolic_array
          |
          +-- PE00
          +-- PE01
          +-- PE02
          +-- PE03
          |
          +-- PE10
          +-- PE11
          +-- PE12
          +-- PE13
          |
          +-- PE20
          +-- PE21
          +-- PE22
          +-- PE23
          |
          +-- PE30
          +-- PE31
          +-- PE32
          +-- PE33


Total number of Processing Elements:

    4 x 4 = 16


## 5. Processing Element

The Processing Element is the fundamental computational unit of the systolic array.

Its primary function is to perform:

    Multiply + Accumulate


The basic equation is:

    PSUM_out =
        PSUM_in + (Activation x Weight)


The PE also forwards activation data to the next PE according to the systolic dataflow.


## 6. PE Responsibilities

The Processing Element is responsible for:

    1. Receiving activation data.

    2. Receiving or loading a weight.

    3. Storing the weight.

    4. Multiplying activation by weight.

    5. Receiving a partial sum.

    6. Adding the multiplication result to the partial sum.

    7. Producing the updated partial sum.

    8. Forwarding activation data.


## 7. PE Conceptual Block Diagram

The Processing Element can be represented as:

                 +------------------+
    Activation ->|                  |-> Activation Out
                 |       PE         |
    Weight ----->|                  |
                 |      MAC         |
    PSUM In ---->|                  |-> PSUM Out
                 |                  |
                 +------------------+


Internally:

    Activation
        |
        v
    Multiplier
        |
        v
      Product
        |
        v
      Adder <-------- PSUM_in
        |
        v
    PSUM_out


The weight is stored locally in the PE.


## 8. Weight-Stationary Behavior

The PE follows the weight-stationary dataflow concept.

The weight is loaded into the PE and remains locally stored while activations move through the array.

Conceptually:

    Weight
      |
      v
    PE Weight Storage
      |
      v
    +---------+
    | Weight  |
    | remains |
    | local   |
    +---------+


Meanwhile:

    Activation
        |
        v
       PE
        |
        v
    Next PE


This reduces the need to repeatedly move the same weight.


## 9. PE Inputs

The exact signal names must match the actual Verilog RTL.

Conceptually, the PE requires:

    Clock

    Reset

    Activation Input

    Weight Input

    Weight Load Control

    Partial Sum Input


These signals provide the control and data required for the MAC operation.


## 10. PE Outputs

Conceptually, the PE provides:

    Activation Output

    Partial Sum Output


Activation Output:

    Used to forward the activation to the next PE.


Partial Sum Output:

    Contains the updated accumulated value.


## 11. Clock

The PE operates synchronously with a clock.

Conceptually:

    always @(posedge clk)


State changes occur at the active clock edge according to the implemented Verilog RTL.


## 12. Reset

Reset initializes the PE state.

The reset behavior must match the actual RTL implementation.

Reset may initialize:

    Stored Weight

    Partial Sum

    Registered Outputs

    Other sequential state


Only signals actually reset in the RTL should be documented as reset values.


## 13. Weight Register

The PE contains local storage for the weight.

Conceptually:

    Weight Input
        |
        v
    Weight Register
        |
        v
    Multiplier


When the weight-loading condition is active:

    Weight Register <= Weight Input


After loading, the stored weight is used during computation.


## 14. Weight Loading

Weight loading occurs during the weight-loading phase.

Conceptually:

    Weight Load Enable = 1

    Weight Input = W


At the active clock edge:

    Stored Weight <= W


After loading:

    Weight Load Enable = 0


The stored weight remains available for computation.


## 15. Weight Retention

During normal computation, the stored weight should remain unchanged unless the RTL explicitly performs another weight update.

Conceptually:

    Load W
      |
      v
    Stored W
      |
      +-------------------+
      |                   |
      v                   v
    Cycle 1             Cycle 2
      |                   |
      +-------- W --------+


This behavior is central to the weight-stationary architecture.


## 16. Multiplier

The multiplier computes:

    Product =
        Activation x Weight


Example:

    Activation = 4

    Weight = 5


Then:

    Product = 4 x 5

    Product = 20


The multiplier output feeds the accumulation logic.


## 17. Accumulator

The accumulator combines the incoming partial sum with the current product.

Equation:

    PSUM_out =
        PSUM_in + Product


Since:

    Product =
        Activation x Weight


the complete equation is:

    PSUM_out =
        PSUM_in + (Activation x Weight)


Example:

    PSUM_in = 10

    Activation = 4

    Weight = 5


Therefore:

    PSUM_out =
        10 + (4 x 5)

    PSUM_out = 30


## 18. Activation Forwarding

The PE forwards the activation according to the implemented systolic dataflow.

Conceptually:

    Activation In
         |
         v
        PE
         |
         v
    Activation Out


When PEs are connected:

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


The activation propagates through the row.


## 19. Partial-Sum Flow

The partial-sum path depends on the implemented array architecture.

Conceptually, the accumulated result moves through the computational path:

    PSUM Input
        |
        v
       PE
        |
        v
    Updated PSUM
        |
        v
    Next PE / Output


The exact direction and connection must follow the implemented RTL.


## 20. PE Timing

Because the PE is synchronous, data processing is associated with clock cycles.

Conceptually:

    Cycle N:
        Input sampled


    Cycle N+1:
        Registered result available


The exact timing must be determined from the actual Verilog RTL and waveform.


## 21. PE Data Width

The data widths must be taken directly from the Verilog source.

The important widths include:

    Activation Width

    Weight Width

    Product Width

    Partial-Sum Width


The product width must be sufficient for the supported multiplication result.

The partial-sum width must be sufficient for the intended accumulation range.


## 22. Product Width

For unsigned operands:

    Activation Width = Wa

    Weight Width = Ww


The mathematical product can require approximately:

    Wa + Ww


bits.


For example:

    8-bit Activation
    8-bit Weight


can produce a product requiring:

    16 bits


The actual RTL width must be verified against the implemented design.


## 23. Partial-Sum Width

The partial sum must accommodate the accumulation of multiple products.

For a 4-element dot product:

    PSUM =
        A0B0
        + A1B1
        + A2B2
        + A3B3


Therefore, the required width may be larger than the individual product width.

The final supported width must be based on the actual RTL implementation.


## 24. Arithmetic Considerations

The RTL must explicitly define the intended arithmetic behavior.

Important considerations include:

    Operand Width

    Product Width

    Partial-Sum Width

    Signedness

    Truncation

    Overflow


These properties must be verified during simulation and synthesis.


## 25. Signedness

The current implementation must be treated according to the actual Verilog declarations.

If signals are declared as unsigned:

    Arithmetic is unsigned.


If signals are declared as signed:

    Arithmetic is signed.


The documentation should never assume signed arithmetic unless the RTL explicitly implements it.


## 26. Top-Level Systolic Array

The eventual top-level module represents the complete 4x4 systolic array.

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


The top-level module is responsible for:

    PE Instantiation

    PE Connectivity

    Input Distribution

    Weight Distribution

    Partial-Sum Connections

    Output Collection


## 27. PE Instantiation

The complete array contains:

    16 PE instances


Conceptually:

    PE00
    PE01
    PE02
    PE03

    PE10
    PE11
    PE12
    PE13

    PE20
    PE21
    PE22
    PE23

    PE30
    PE31
    PE32
    PE33


Each PE performs the same basic computation.


## 28. PE Reuse

The PE should be designed as a reusable module.

Instead of writing separate arithmetic logic for every location, the same PE module can be instantiated multiple times.

This provides:

    Modularity

    Maintainability

    Easier Verification

    Easier Debugging

    Scalable Architecture


## 29. Array Connectivity

The PE connections establish the systolic dataflow.

Conceptually:

    Activation:

    PE00 -> PE01 -> PE02 -> PE03

    PE10 -> PE11 -> PE12 -> PE13

    PE20 -> PE21 -> PE22 -> PE23

    PE30 -> PE31 -> PE32 -> PE33


The exact interconnection must follow the final RTL implementation.


## 30. Weight Distribution

In a weight-stationary architecture, weights are loaded into the appropriate PEs.

Conceptually:

    Weight Matrix
          |
          v
    +------+------+------+------+
    | W00  | W01  | W02  | W03  |
    +------+------+------+------+
    | W10  | W11  | W12  | W13  |
    +------+------+------+------+
    | W20  | W21  | W22  | W23  |
    +------+------+------+------+
    | W30  | W31  | W32  | W33  |
    +------+------+------+------+


Each PE stores the weight associated with its computational position.


## 31. Activation Distribution

Activations are streamed through the array according to the systolic schedule.

Conceptually:

    Activation Stream
          |
          v
    +------+------+------+------+
    | PE00 | PE01 | PE02 | PE03 |
    +------+------+------+------+
          |
          v
    Activation progresses
    through the PE structure.


The exact schedule is defined by the top-level RTL and testbench.


## 32. Matrix Multiplication

The complete array is intended to perform:

    C = A x B


For each output:

    Cij =
        Sum(Aik x Bkj)


For a 4x4 matrix:

    Cij =
        Ai0B0j
        + Ai1B1j
        + Ai2B2j
        + Ai3B3j


The PEs collectively perform these multiply-and-accumulate operations.


## 33. Dot Product Interpretation

Each output matrix element is a dot product.

Example:

    C00 =

        A00B00
        + A01B10
        + A02B20
        + A03B30


The corresponding PE operations generate and accumulate these products.


## 34. Systolic Operation

The systolic array performs computation through regular movement of data between neighboring PEs.

Conceptually:

    Data
      |
      v
    PE
      |
      v
    PE
      |
      v
    PE
      |
      v
    PE


Each PE performs a small amount of computation every cycle.

This creates a spatially distributed pipeline.


## 35. Pipeline Behavior

Once data begins flowing through the array, multiple PEs can operate simultaneously.

Conceptually:

    Cycle 1:
        PE00 active


    Cycle 2:
        PE00 + PE01 active


    Cycle 3:
        PE00 + PE01 + PE02 active


    Cycle 4:
        Multiple PEs active


The exact schedule depends on the implemented dataflow and input-loading mechanism.


## 36. Array Latency

The total latency depends on:

    Array Size

    Dataflow

    Input Schedule

    Register Placement

    Output Collection


The final latency must be measured from simulation.


## 37. Array Throughput

Once the array reaches steady state, multiple operations may be performed concurrently.

The throughput depends on:

    Clock Frequency

    Pipeline Structure

    Input Scheduling

    Output Scheduling


The final throughput will be measured after complete array implementation.


## 38. Reset Strategy

All sequential elements in the design should follow a consistent reset strategy.

Reset should establish a known initial state before computation.

The testbench should verify:

    Reset Asserted
        |
        v
    Known State
        |
        v
    Reset Released
        |
        v
    Normal Operation


## 39. Control Signals

The design may use control signals for:

    Reset

    Weight Loading

    Computation

    Valid Data

    Output Validity


Only signals that actually exist in the RTL should be listed as implemented signals.

Future control signals should be clearly identified as planned rather than implemented.


## 40. Interface Documentation

For each RTL module, the final documentation should include a table containing:

    Signal Name
    Direction
    Width
    Description


Example format:

    Signal Name    Direction    Width    Description

    clk            input        1        Clock

    reset          input        1        Reset

    activation     input        N        Activation data

    weight         input        N        Weight data

    psum_in        input        M        Partial sum input

    activation_out output       N        Forwarded activation

    psum_out       output       M        Updated partial sum


The exact widths and names must be copied from the actual Verilog source.


## 41. Synthesizability

The RTL is intended to describe synthesizable hardware.

The design should avoid simulation-only constructs inside synthesizable modules.

Examples of constructs normally reserved for testbenches include:

    $display

    $monitor

    #delay


These should remain in the verification environment where required.


## 42. RTL Coding Principles

The RTL should follow standard hardware-design practices.

Recommended principles:

    Use clear module boundaries.

    Keep combinational and sequential behavior understandable.

    Use clocked logic for registered state.

    Avoid unnecessary logic duplication.

    Keep signal names descriptive.

    Maintain consistent indentation.

    Comment non-obvious logic.

    Keep testbench code separate from synthesizable RTL.


## 43. Sequential Logic

Sequential state should be implemented using clocked procedural blocks.

Conceptually:

    always @(posedge clk)


The exact reset style must follow the implemented design.


## 44. Combinational Logic

Combinational operations such as multiplication and addition should be described according to the actual RTL implementation.

The PE's core arithmetic is:

    Product =
        Activation x Weight


    Accumulation =
        PSUM_in + Product


The exact Verilog coding style is determined by the RTL.


## 45. Module Naming

The project should use consistent module names.

Recommended examples:

    pe

    systolic_array


The actual module names should match the source files.

If the implemented module uses a different name, this documentation should be updated accordingly.


## 46. File Naming

Recommended RTL organization:

    rtl/
    |
    +-- pe.v
    |
    +-- systolic_array.v


If additional modules are introduced, each module should have a clearly corresponding Verilog source file.


## 47. Dependency Structure

The intended dependency is:

    pe.v
      |
      v
    systolic_array.v
      |
      v
    Testbench


The top-level array instantiates the PE module.


## 48. Testbench Dependency

The testbench uses the RTL modules but is not part of the synthesizable design.

Conceptually:

    Testbench
       |
       +----> PE RTL
       |
       +----> Array RTL


The testbench generates stimulus and checks outputs.


## 49. RTL Verification Status

Current status:

    PE RTL:
        COMPLETED


    PE Testbench:
        COMPLETED


    PE Simulation:
        PASSED


    Complete Array RTL:
        UNDER DEVELOPMENT


The status must be updated when additional modules are completed.


## 50. PE Verification Evidence

The PE implementation has been verified through simulation.

Verified behaviors include:

    Weight Loading

    Weight Storage

    Multiplication

    Partial-Sum Accumulation

    MAC Operation

    Activation Forwarding


This provides confidence in the PE as a reusable hardware block.


## 51. Current RTL Limitation

The complete accelerator should not be considered fully implemented until the 4x4 PE array has been integrated and tested.

The current verified milestone is the PE-level design.


Therefore:

    PE:
        VERIFIED


    Full 4x4 Accelerator:
        NOT YET VERIFIED


This distinction should be maintained throughout the GitHub documentation.


## 52. Future RTL Enhancements

Potential future enhancements include:

    Parameterized Array Size

    Parameterized Data Width

    Configurable Matrix Size

    Valid / Ready Interface

    Input Buffers

    Output Buffers

    Control FSM

    Performance Counters

    Additional Pipeline Registers


These are future possibilities and should not be described as implemented features until they are actually added.


## 53. Design Scalability

The PE-based architecture provides a natural path toward larger arrays.

For example:

    4x4
      |
      v
    8x8
      |
      v
    16x16


A reusable PE allows the computational structure to be replicated.

The practical scalability depends on:

    Area

    Routing

    Memory Bandwidth

    Timing

    Power


## 54. RTL-to-GDS Relevance

The RTL represents the first major stage of a standard ASIC design flow.

The intended flow is:

    Verilog RTL
        |
        v
    Simulation
        |
        v
    Synthesis
        |
        v
    Gate-Level Netlist
        |
        v
    Floorplanning
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
    GDS


The current project is primarily at the RTL and functional-verification stages.


## 55. RTL Documentation Rule

All module documentation must remain synchronized with the actual source code.

If a signal is renamed:

    Update this document.


If a module is added:

    Add it to this document.


If a module is removed:

    Remove it from this document.


If a feature is planned but not implemented:

    Mark it as planned.


This ensures that the GitHub repository remains technically accurate.


## 56. Summary

The Processing Element is the fundamental reusable RTL block of the accelerator.

Its primary operation is:

    PSUM_out =
        PSUM_in + (Activation x Weight)


The PE stores the weight locally according to the weight-stationary architecture and forwards activation data through the systolic dataflow.

The complete accelerator will consist of:

    16 Processing Elements

arranged as:

    4 x 4


The PE RTL and PE-level verification have been completed.

The next major RTL milestone is integrating the verified PE into the complete systolic-array structure and verifying the resulting matrix-multiplication behavior.
