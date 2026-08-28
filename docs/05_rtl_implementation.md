# RTL Implementation

## 1. Overview

The systolic array accelerator is implemented using synthesizable Verilog RTL.

The RTL development follows a modular design methodology in which the Processing Element (PE) is implemented as the fundamental reusable hardware block.

The project is being developed incrementally:

    Processing Element
          ↓
    PE Verification
          ↓
    Multiple PE Integration
          ↓
    4×4 Systolic Array
          ↓
    Array Verification
          ↓
    Top-Level Accelerator

The RTL is intended to be suitable for simulation and subsequent synthesis.

---

## 2. RTL Language

The hardware design is implemented in:

    Verilog HDL

RTL source files use the:

    .v

file extension.

The project does not use SystemVerilog-specific constructs.

The design therefore follows standard Verilog coding constructs such as:

    module
    endmodule
    input
    output
    wire
    reg
    assign
    always
    parameter

---

## 3. RTL Directory

The main RTL source files are maintained under:

    rtl/

The project structure is:

    ~/systolic/
    |
    +-- rtl/
    |
    +-- tb/
    |
    +-- sim/
    |
    +-- wave/
    |
    +-- docs/

The `rtl/` directory contains synthesizable hardware design files.

The `tb/` directory contains verification testbenches.

The `sim/` directory contains simulation-related files.

The `wave/` directory contains waveform-related artifacts.

The `docs/` directory contains project documentation.

---

## 4. RTL Design Philosophy

The RTL follows several design principles:

### Modularity

Each hardware block should have a clearly defined function.

### Reusability

The Processing Element should be reusable for multiple PE instances.

### Synthesizability

The RTL should describe hardware that can be mapped to gates and other physical resources by a synthesis tool.

### Deterministic Behavior

The hardware should behave predictably for defined input conditions.

### Clear Dataflow

Signal movement should correspond directly to the intended systolic architecture.

### Incremental Verification

Each block should be verified before being integrated into a larger block.

---

## 5. Processing Element as RTL Building Block

The Processing Element is the fundamental RTL block.

Its primary functions are:

    Weight Loading
          ↓
    Weight Storage
          ↓
    Activation Input
          ↓
    Multiplication
          ↓
    Accumulation
          ↓
    Partial Sum Output

At the same time:

    Activation Input
          ↓
    Activation Forwarding
          ↓
    Activation Output

The PE is implemented as a reusable Verilog module.

---

## 6. Verilog Module Structure

A Verilog module provides the interface and implementation of the hardware block.

The general structure is:

    module pe (
        input  ...,
        output ...
    );

        // Internal declarations

        // RTL logic

    endmodule

The exact module name, ports, signal widths, and implementation are defined in the project's actual RTL source file.

---

## 7. Inputs and Outputs

The PE interface conceptually contains:

### Inputs

- Clock
- Reset
- Activation input
- Weight input
- Weight-load control
- Partial-sum input

### Outputs

- Activation output
- Partial-sum output

The exact interface must always be taken from the current Verilog RTL source.

Documentation should not override the actual RTL interface.

---

## 8. Internal Signals

The PE requires internal signals for operations such as:

- Stored weight
- Multiplication result
- Accumulation
- Forwarded activation
- Intermediate values

Conceptually:

    Weight Input
         |
         v
    Weight Register
         |
         v
    Stored Weight
         |
         v
    Multiplier
         |
         v
      Product
         |
         v
       Adder <----- PSUM Input
         |
         v
      PSUM Output

The exact internal signal names are implementation-specific.

---

## 9. Sequential Logic

Sequential logic represents state that changes with the clock.

The standard Verilog form is:

    always @(posedge clk)

For example:

    always @(posedge clk) begin
        register <= next_value;
    end

Non-blocking assignment:

    <=

is used for clocked register updates.

The weight register is an example of state that must retain its value between clock cycles.

---

## 10. Weight Register

The PE contains storage for the weight.

Conceptually:

    Weight Input
         |
         v
    Weight Register
         |
         v
    Stored Weight

When the weight-load condition is active, the incoming weight is captured.

After the loading operation, the stored value remains available for MAC operations.

This provides the weight-stationary behavior.

---

## 11. Weight Loading RTL Behavior

The conceptual sequential operation is:

    At clock edge:

    if weight_load is active

        stored_weight <= weight_input

Otherwise:

        stored_weight remains unchanged

The exact reset and control conditions are defined by the actual RTL implementation.

---

## 12. Combinational Logic

Combinational logic produces outputs based on current inputs and internal state without requiring storage.

Standard Verilog continuous assignments can be used:

    assign output_signal = expression;

For procedural combinational logic, the standard form is:

    always @(*)

For example:

    always @(*) begin
        result = a + b;
    end

The actual PE implementation determines which operations are implemented as combinational logic and which are registered.

---

## 13. Multiply Operation

The core arithmetic operation is:

    Product = Activation × Stored_Weight

The multiplier therefore receives:

    Activation

and:

    Stored Weight

The result is supplied to the accumulation logic.

Conceptually:

    Activation
        |
        v
    +-----------+
    | Multiplier|
    +-----+-----+
          |
          v
       Product
          |
          v
       Accumulator

---

## 14. Accumulation Operation

The accumulation equation is:

    PSUM_out = PSUM_in + Product

Substituting the multiplication:

    PSUM_out =
        PSUM_in + (Activation × Weight)

This is the fundamental MAC operation.

The exact register placement and output timing are determined by the Verilog RTL.

---

## 15. Activation Forwarding

Activation forwarding is required for systolic operation.

Conceptually:

    activation_in
          |
          v
         PE
          |
          v
    activation_out

The activation output can then be connected to the activation input of another PE.

This enables:

    PE00 → PE01 → PE02 → PE03

and similar paths across the array.

---

## 16. Partial-Sum Path

The partial-sum path allows the accumulated computation to continue.

Conceptually:

    psum_in
       |
       v
    +------+
    |  PE  |
    +--+---+
       |
       v
    psum_out

Inside the PE:

    psum_out =
        psum_in + (activation × weight)

This allows the PE to function as a MAC stage.

---

## 17. Reset Logic

Reset places the internal PE state into a known condition.

The reset behavior may initialize registers such as:

- Stored weight
- Partial-sum state
- Forwarded data state

The exact reset polarity and synchronous/asynchronous behavior must match the actual Verilog RTL.

The documentation intentionally does not assume a reset implementation that is not present in the RTL.

---

## 18. Clocking

The design uses synchronous digital logic.

The system clock provides the timing reference for sequential state changes.

Conceptually:

    Clock
      |
      +------ PE00
      |
      +------ PE01
      |
      +------ PE02
      |
      +------ ...
      |
      +------ PE33

A common clock allows the Processing Elements to operate in a coordinated manner.

---

## 19. Blocking and Non-Blocking Assignments

The RTL follows standard Verilog assignment conventions.

### Non-Blocking Assignment

Use:

    <=

for sequential register updates.

Example:

    always @(posedge clk) begin
        q <= d;
    end

### Blocking Assignment

Use:

    =

for procedural combinational calculations where appropriate.

Example:

    always @(*) begin
        y = a + b;
    end

Correct assignment selection helps prevent simulation behavior that does not accurately represent the intended hardware.

---

## 20. Continuous Assignments

Simple combinational relationships can be described using:

    assign

For example:

    assign product = activation * weight;

Continuous assignments are useful for combinational datapath relationships.

The actual RTL should be preferred over this conceptual example when describing the implemented design.

---

## 21. Parameterization

Parameters can be used when a design requires configurable values such as:

- Data width
- Weight width
- Partial-sum width
- Array dimensions

A general Verilog parameter can be declared using:

    parameter DATA_WIDTH = ...

The exact parameterization depends on the final project RTL.

If a value is fixed in the current implementation, the documentation should not incorrectly describe it as parameterized.

---

## 22. Arithmetic Width

Arithmetic width is an important consideration in the PE.

If activation has width:

    N bits

and weight has width:

    M bits

the multiplication result may require up to:

    N + M bits

The accumulator may require additional bits because multiple products are added together.

For four accumulated products, the required accumulator range must be considered carefully.

The actual widths used by the project are defined in the Verilog RTL.

---

## 23. Signedness

The interpretation of arithmetic operands depends on whether the signals are signed or unsigned.

Possible representations include:

    Unsigned

or:

    Signed two's-complement

The RTL must consistently define the intended representation.

Signedness affects:

- Multiplication
- Addition
- Comparison
- Overflow
- Expected simulation results

Therefore, testbench expected values must use the same arithmetic interpretation as the RTL.

---

## 24. Synthesizable RTL

The PE is intended to use synthesizable Verilog constructs.

Synthesis tools translate synthesizable RTL into a gate-level implementation.

Conceptually:

    Verilog RTL
         |
         v
    RTL Synthesis
         |
         v
    Gate-Level Netlist

The final synthesized implementation can then be analyzed for:

- Timing
- Area
- Power

---

## 25. Avoiding Simulation-Only Constructs

For synthesis-oriented RTL, care must be taken with constructs intended only for simulation.

Examples of testbench-oriented constructs include:

    $display
    $monitor
    $finish
    #delay

These may be appropriate in verification environments but should not be used indiscriminately inside synthesizable design logic.

The design RTL should describe actual hardware behavior.

---

## 26. Testbench Separation

The project separates synthesizable RTL from verification code.

Design:

    rtl/

Verification:

    tb/

This separation is important because testbench code may contain simulation-only constructs that are not intended for synthesis.

The Processing Element RTL remains independent from its testbench.

---

## 27. RTL and Testbench Relationship

The relationship is:

    +----------------+
    |   Testbench    |
    |                |
    | Stimulus       |
    | Clock          |
    | Reset          |
    | Checking       |
    +-------+--------+
            |
            v
    +----------------+
    | Processing     |
    | Element RTL    |
    +-------+--------+
            |
            v
         Outputs
            |
            v
       Verification

The testbench does not become part of the synthesized hardware.

---

## 28. RTL Simulation Flow

The basic simulation flow is:

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
        +---- Console Results
        |
        +---- Waveform
        |
        v
    Functional Verification

The project uses simulation to verify the PE before array integration.

---

## 29. Compilation

Before simulation, the required Verilog files must be compiled.

Conceptually:

    PE RTL
       +
    PE Testbench
       |
       v
    Verilog Compiler
       |
       v
    Simulation Model

Compilation errors must be resolved before functional simulation can proceed.

---

## 30. Simulation Execution

After successful compilation:

    Start Simulation
          ↓
    Apply Reset
          ↓
    Load Weight
          ↓
    Apply Activation
          ↓
    Apply Partial Sum
          ↓
    Perform MAC
          ↓
    Observe Output
          ↓
    Compare Expected Result

This verifies the implemented behavior.

---

## 31. Waveform Generation

Simulation waveforms provide visibility into internal and external signals.

Important signals include:

    clk
    reset
    activation
    weight
    weight_load
    psum_in
    psum_out
    activation_out

The exact signal names depend on the current RTL.

Waveforms can be used to verify cycle-by-cycle behavior.

---

## 32. RTL Debugging Method

When an unexpected result occurs, debugging should proceed systematically.

### Step 1

Check reset.

### Step 2

Check weight loading.

### Step 3

Check stored weight.

### Step 4

Check activation input.

### Step 5

Check multiplication result.

### Step 6

Check partial-sum input.

### Step 7

Check partial-sum output.

### Step 8

Check activation forwarding.

### Step 9

Check clock-cycle alignment.

This approach makes it easier to isolate RTL errors.

---

## 33. PE RTL Verification

The PE RTL was verified using a dedicated Verilog testbench.

The verification confirmed the intended behavior of:

- Weight loading
- Weight retention
- Activation input
- Activation forwarding
- Multiplication
- Partial-sum accumulation
- Output behavior

The successful PE verification provides the foundation for array-level integration.

---

## 34. Modular Array Construction

The complete array can be constructed by instantiating the PE module multiple times.

Conceptually:

    PE Module
       |
       +---- PE00
       +---- PE01
       +---- PE02
       +---- PE03
       |
       +---- PE10
       +---- PE11
       +---- PE12
       +---- PE13
       |
       +---- PE20
       +---- PE21
       +---- PE22
       +---- PE23
       |
       +---- PE30
       +---- PE31
       +---- PE32
       +---- PE33

This avoids duplicating the PE implementation.

---

## 35. PE Interconnection

During array integration, PE outputs are connected to neighboring PE inputs.

Conceptually:

    PE00.activation_out
              |
              v
    PE01.activation_in

Similarly:

    PE01.activation_out
              |
              v
    PE02.activation_in

and so on.

The exact connections depend on the final systolic dataflow.

---

## 36. Top-Level RTL

After PE verification, a higher-level module can instantiate the array.

Conceptually:

    Top-Level
        |
        v
    Systolic Array
        |
        +---- PE00
        +---- PE01
        ...
        +---- PE33

The top-level module will eventually provide:

- External inputs
- External outputs
- Clock
- Reset
- Control
- Data interfaces

The final interface will be defined during top-level integration.

---

## 37. RTL Hierarchy

The intended RTL hierarchy is:

    Top-Level Accelerator
             |
             v
       Systolic Array
             |
             v
        Processing Element
             |
             +---- Weight Register
             |
             +---- Multiplier
             |
             +---- Accumulator
             |
             +---- Forwarding Logic

This hierarchical structure makes the design easier to understand and maintain.

---

## 38. Design Reuse

The PE is designed to be reusable.

A single verified PE implementation can be instantiated multiple times.

Advantages include:

- Less duplicated RTL
- Easier maintenance
- Consistent behavior
- Simplified verification
- Easier scalability

A change to the PE implementation can therefore be propagated to all PE instances through the common module.

---

## 39. RTL Quality Considerations

The RTL should be checked for:

- Correct reset behavior
- Correct clocking
- Correct assignment types
- Width mismatches
- Signedness mismatches
- Unintended latches
- Multiple drivers
- Undriven signals
- Combinational loops
- Synthesis compatibility

These checks become increasingly important as the design grows.

---

## 40. Latch Avoidance

Combinational procedural blocks must assign outputs for all relevant conditions.

For example:

    always @(*) begin

        if (condition)
            y = a;
        else
            y = b;

    end

Providing complete assignments prevents unintended latch inference.

The actual RTL should be reviewed accordingly.

---

## 41. Multiple Driver Avoidance

A signal should not unintentionally be driven by multiple sources.

Incorrect:

    assign result = a;

and simultaneously:

    always @(*) begin
        result = b;
    end

Such conflicting drivers can produce incorrect hardware or synthesis errors.

The RTL should maintain clear ownership of each signal.

---

## 42. Width Matching

Signals connected between modules must have compatible widths.

For example:

    Activation Output Width
             =
    Activation Input Width

Similarly:

    Partial Sum Output Width
             =
    Partial Sum Input Width

where required by the architecture.

Width mismatches should be identified during compilation and RTL review.

---

## 43. Hierarchical Verification

The RTL should be verified hierarchically.

The recommended sequence is:

    PE
     ↓
    Multiple PEs
     ↓
    PE Row
     ↓
    4×4 Array
     ↓
    Top-Level Accelerator

This approach reduces debugging complexity.

---

## 44. Synthesis-Oriented Development

The RTL is being developed with eventual synthesis in mind.

The intended flow is:

    Verilog RTL
         ↓
    Simulation
         ↓
    Functional Verification
         ↓
    Synthesis
         ↓
    Gate-Level Netlist
         ↓
    Timing Analysis
         ↓
    Area Analysis
         ↓
    Power Analysis

This allows functional correctness to be established before physical implementation.

---

## 45. Timing Considerations

The critical path may involve:

    Activation
        ↓
    Multiplier
        ↓
    Adder
        ↓
    Register

The exact critical path can only be determined after synthesis and timing analysis.

The RTL documentation therefore does not claim a specific maximum frequency until timing analysis is performed.

---

## 46. Area Considerations

The main arithmetic resources include:

- Multipliers
- Adders
- Registers
- Routing/interconnect

For 16 PEs, these resources are replicated across the array.

The actual silicon area depends on the selected technology library and synthesis results.

---

## 47. Power Considerations

Dynamic power can result from switching activity in:

- Multipliers
- Adders
- Registers
- Interconnects

The systolic array can have significant concurrent switching because multiple PEs may operate simultaneously.

Actual power must be measured or estimated using a suitable power-analysis flow.

No power number should be claimed without analysis data.

---

## 48. RTL Portability

Using standard Verilog constructs improves portability between HDL simulators and synthesis tools.

The project therefore avoids unnecessary dependence on tool-specific RTL constructs.

The objective is to maintain clean, synthesizable Verilog that can be simulated and later synthesized using compatible EDA tools.

---

## 49. Current RTL Status

| RTL Component | Status |
|---------------|--------|
| PE module | Completed |
| Weight storage | Completed |
| Weight loading | Completed |
| MAC datapath | Completed |
| Activation forwarding | Completed |
| Partial-sum operation | Completed |
| PE testbench | Completed |
| PE simulation | Completed |
| PE functional verification | Completed |
| Multiple PE integration | Next stage |
| 4×4 array RTL | Under development |
| Array testbench | Planned |
| Top-level RTL | Planned |
| Synthesis | Planned |

---

## 50. RTL Development Rules

The following rules should be maintained throughout the project:

1. Use Verilog RTL only.
2. Keep synthesizable design code inside `rtl/`.
3. Keep testbench code inside `tb/`.
4. Use non-blocking assignments for sequential logic.
5. Use blocking assignments for procedural combinational logic where appropriate.
6. Use `always @(posedge clk)` for clocked logic.
7. Use `always @(*)` for combinational procedural logic.
8. Avoid unintended latches.
9. Avoid multiple drivers.
10. Keep signal widths consistent.
11. Clearly define signedness.
12. Verify each module before integration.
13. Do not claim performance without measurement.
14. Keep the documentation synchronized with the actual RTL.

---

## 51. Source-of-Truth Principle

The actual Verilog RTL is the authoritative source for:

- Module names
- Port names
- Port directions
- Signal widths
- Reset polarity
- Reset behavior
- Clock behavior
- Register implementation
- Combinational logic
- Data timing

Documentation should describe the RTL rather than inventing implementation details.

If the RTL changes, the corresponding documentation should also be updated.

---

## 52. Summary

The accelerator is implemented using standard Verilog RTL.

The Processing Element is the fundamental reusable hardware block.

The PE implements:

    PSUM_out = PSUM_in + (Activation × Weight)

and provides activation forwarding required for systolic data movement.

The RTL development follows a modular and verification-driven approach:

    Verilog PE
        ↓
    PE Testbench
        ↓
    PE Simulation
        ↓
    Functional Verification
        ↓
    Array Integration
        ↓
    Array Verification
        ↓
    Synthesis

The current verified RTL foundation is the Processing Element.

The next development stage is to integrate the verified PE modules into the complete 4×4 systolic array and verify the complete dataflow and matrix multiplication operation.
