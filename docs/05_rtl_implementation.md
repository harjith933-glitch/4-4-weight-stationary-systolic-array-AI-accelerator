# RTL Implementation

## 1. Overview

The systolic array accelerator is being developed using synthesizable SystemVerilog RTL.

The RTL development follows a modular and bottom-up methodology.

The Processing Element (PE) is implemented first and verified independently before being integrated into the complete 4×4 systolic array.

The overall development hierarchy is:

    Processing Element
           ↓
    PE Testbench
           ↓
    PE Verification
           ↓
    4×4 PE Array
           ↓
    Array-Level Testbench
           ↓
    Matrix Multiplication Verification
           ↓
    Top-Level Accelerator
           ↓
    Synthesis and Hardware Analysis

The current verified RTL implementation is the Processing Element.

---

## 2. RTL Design Objectives

The RTL implementation is designed with the following objectives:

- Synthesizable hardware description
- Modular architecture
- Reusable Processing Element
- Synchronous operation
- Predictable data movement
- Weight-stationary dataflow
- Parameterizable datapath widths where appropriate
- Easy simulation and debugging
- Scalability toward larger systolic arrays
- Compatibility with standard ASIC design flow

---

## 3. RTL Design Methodology

The project follows a bottom-up RTL design methodology.

The design is first divided into small functional blocks.

The PE is the smallest major computational block.

Once the PE functionality is verified, multiple instances can be connected to form the complete systolic array.

This approach reduces debugging complexity because errors can be isolated at the block level before system-level integration.

The methodology is:

    Architecture Definition
            ↓
    RTL Specification
            ↓
    Module Coding
            ↓
    Compilation
            ↓
    Simulation
            ↓
    Waveform Inspection
            ↓
    Functional Verification
            ↓
    Integration

---

## 4. Processing Element RTL

The Processing Element is the primary RTL module currently implemented.

Its functional purpose is to:

1. Store a weight.
2. Receive an activation.
3. Multiply activation and stored weight.
4. Add the product to the incoming partial sum.
5. Produce an updated partial sum.
6. Forward the activation.

The fundamental computation is:

    PSUM_out = PSUM_in + (Activation × Weight)

---

## 5. PE RTL Concept

The RTL implementation can be represented conceptually as:

    +--------------------------------------+
    |          Processing Element          |
    |                                      |
    |  Weight Input -----> Weight Register |
    |                           |          |
    |                           v          |
    |  Activation Input ---> Multiplier   |
    |                           |          |
    |                           v          |
    |  PSUM Input ----------> Adder        |
    |                           |          |
    |                           v          |
    |                       PSUM Output    |
    |                                      |
    |  Activation Input ---> Forward Path  |
    |                           |          |
    |                           v          |
    |                    Activation Output |
    +--------------------------------------+

The actual RTL implementation determines which paths are registered and how outputs are timed.

---

## 6. Sequential and Combinational Logic

The PE contains both state-holding and arithmetic behavior.

### Sequential logic

Sequential logic is used for values that must retain state between clock cycles.

Examples include:

- Stored weight
- Registered partial sum, if implemented
- Registered activation forwarding, if implemented

Sequential logic is controlled by the clock.

### Combinational logic

Combinational logic is used for arithmetic operations that do not independently store state.

Examples include:

- Multiplication
- Addition
- Intermediate arithmetic signals

The exact implementation should be taken from the current RTL source.

---

## 7. Clock

The PE operates synchronously using a clock signal.

The clock provides the timing reference for registered operations.

Conceptually:

    Clock
      |
      +----> Weight Register
      |
      +----> Activation Register
      |
      +----> Partial-Sum Register

The clocked design allows the PE to participate in a deterministic systolic pipeline.

---

## 8. Reset

Reset initializes the internal state of the Processing Element.

Reset ensures that the PE begins operation from a known state.

Depending on the RTL implementation, reset may initialize:

- Weight register
- Partial-sum state
- Activation forwarding state
- Other internal registers

The reset behavior is verified in the PE testbench.

The exact reset polarity and synchronous/asynchronous implementation must always be taken from the actual RTL source.

---

## 9. Weight Register

The weight register is responsible for implementing the weight-stationary behavior.

When the weight-load control is active, the incoming weight is stored.

Conceptually:

    if weight_load:
        stored_weight <= weight_input;

When weight loading is inactive:

    stored_weight remains unchanged.

This allows the weight to remain stationary inside the PE during subsequent computation.

---

## 10. Weight Loading

The weight-loading sequence is:

    Weight input
         |
         v
    weight_load asserted
         |
         v
    Clock edge
         |
         v
    Weight stored
         |
         v
    weight_load deasserted
         |
         v
    Weight retained

The stored weight is then used by the multiplier.

---

## 11. Weight Retention

After loading, the PE retains the weight until another valid weight-loading operation occurs or reset initializes the register.

This behavior is important for weight reuse.

For example:

    Cycle 1:
        Load W

    Cycle 2:
        A0 × W

    Cycle 3:
        A1 × W

    Cycle 4:
        A2 × W

    Cycle 5:
        A3 × W

The same locally stored weight can therefore participate in multiple MAC operations.

---

## 12. Multiplier

The multiplier calculates:

    Product = Activation × Stored_Weight

The multiplication is the primary arithmetic operation in the PE.

Conceptually:

    activation_in
          |
          v
       +------+
       |  ×   | <---- stored_weight
       +--+---+
          |
          v
       product

The multiplier width depends on the operand widths defined in the RTL.

---

## 13. Partial-Sum Adder

The product generated by the multiplier is added to the incoming partial sum.

The operation is:

    PSUM_out = PSUM_in + Product

Therefore:

    PSUM_out =
        PSUM_in + (Activation × Stored_Weight)

This is the accumulate portion of the MAC operation.

---

## 14. MAC Datapath

The complete arithmetic datapath is:

    Activation
        |
        v
    +-----------+
    | Multiplier|
    +-----+-----+
          |
          | Product
          v
    +-----------+
    |   Adder   | <----- PSUM_in
    +-----+-----+
          |
          v
      PSUM_out

The PE therefore performs a complete MAC operation.

---

## 15. Activation Forwarding

The activation is also forwarded to the next PE.

Conceptually:

    activation_in
          |
          v
        +-----+
        | PE  |
        +--+--+
           |
           v
    activation_out

This forwarding path is necessary to construct the systolic communication network.

The activation output of one PE can become the activation input of the neighboring PE.

---

## 16. PE-to-PE Connection

When multiple PEs are instantiated, their activation paths can be connected.

For example:

    PE0
     |
     | activation_out
     v
    PE1
     |
     | activation_out
     v
    PE2
     |
     | activation_out
     v
    PE3

This creates a regular data propagation path.

The complete 4×4 connection structure will be implemented at the array level.

---

## 17. Partial-Sum Path

The partial-sum path provides the accumulation mechanism.

Conceptually:

    PSUM_in
       |
       v
    +------+
    |  PE  |
    +--+---+
       |
       v
    PSUM_out

The exact connection of PSUM signals between PEs depends on the final systolic dataflow implementation.

The PE provides the basic arithmetic operation required for this connection.

---

## 18. RTL Coding Style

The RTL should follow synthesizable and maintainable SystemVerilog coding practices.

Important principles include:

- Use `always_ff` for clocked sequential logic where appropriate.
- Use `always_comb` for combinational logic where appropriate.
- Avoid unintended latches.
- Use non-blocking assignments for sequential logic.
- Use blocking assignments for combinational procedural logic when appropriate.
- Keep datapath and control behavior understandable.
- Avoid unnecessary logic duplication.
- Use meaningful signal names.
- Keep modules modular.
- Avoid simulation-only constructs in synthesizable RTL.

---

## 19. Blocking and Non-Blocking Assignments

Sequential state updates should use non-blocking assignments.

Conceptually:

    always_ff @(posedge clk) begin
        register <= next_value;
    end

This models hardware registers correctly.

Combinational calculations may use blocking assignments where procedural combinational logic is required.

The chosen coding style should maintain a clear distinction between:

    Current state

and:

    Next state

---

## 20. Synthesizability

The RTL is intended for synthesis into hardware.

Therefore, synthesizable constructs should be used.

The following should be avoided in synthesizable datapath RTL unless specifically supported for synthesis:

- Arbitrary delays
- `#` timing controls
- Testbench-only constructs
- File I/O inside synthesizable modules
- Infinite simulation loops
- Unsupported dynamic constructs

The testbench may use simulation-specific constructs, but the design RTL should remain synthesizable.

---

## 21. Parameterization

Where appropriate, datapath widths should be parameterized rather than hard-coded.

Potential parameters include:

    DATA_WIDTH
    WEIGHT_WIDTH
    PSUM_WIDTH

For example:

    parameter DATA_WIDTH = ...
    parameter WEIGHT_WIDTH = ...
    parameter PSUM_WIDTH = ...

Parameterization allows the architecture to be evaluated with different numerical precisions.

The exact parameters must match the current RTL implementation.

---

## 22. Datapath Width Considerations

The accumulator generally requires greater width than an individual input operand.

For signed operands:

    Activation × Weight

produces a result whose width depends on the operand widths.

Accumulating multiple products may require additional bits.

For a general dot product:

    PSUM =
        A0×W0 +
        A1×W1 +
        A2×W2 +
        A3×W3

the accumulator width must be selected so that expected results do not overflow.

The final width selection should therefore be documented based on the actual RTL design.

---

## 23. Signed Arithmetic Considerations

If the accelerator supports signed values, the RTL must explicitly handle signed arithmetic.

The design must ensure that:

- Activation interpretation is correct.
- Weight interpretation is correct.
- Multiplication produces the intended signed result.
- Partial-sum accumulation preserves sign.
- Output interpretation is correct.

If the current implementation uses unsigned operands, this must remain consistent throughout the PE and testbench.

The actual signed/unsigned definition should be taken directly from the RTL source.

---

## 24. Overflow Considerations

Arithmetic overflow is an important consideration in MAC-based hardware.

For example, if the accumulator is too narrow:

    PSUM + Product

may exceed the representable range.

This can result in incorrect output values.

Therefore, the design should determine:

- Operand range
- Product range
- Number of accumulated products
- Required accumulator width

Overflow behavior should eventually be included in verification.

---

## 25. RTL Module Organization

The project follows a modular directory structure.

The project root is:

    ~/systolic

The main directories are:

    rtl/
    tb/
    sim/
    wave/
    docs/

The intended purpose of each directory is:

### rtl/

Contains synthesizable SystemVerilog design files.

### tb/

Contains verification/testbench files.

### sim/

Contains simulation scripts, generated simulation artifacts, or simulator-specific files as applicable.

### wave/

Contains waveform files and waveform-related artifacts.

### docs/

Contains project documentation.

---

## 26. RTL File Organization

The RTL directory is intended to contain the hardware modules.

The PE RTL is the first major implemented design block.

Future RTL modules are expected to include components such as:

    PE
    Systolic Array
    Controller
    Input Buffer
    Weight Buffer
    Output Handler
    Top-Level Accelerator

Only modules that have actually been implemented should be listed as completed.

---

## 27. Testbench Relationship

The RTL module is verified using a dedicated testbench.

The relationship is:

    +------------------+
    |     Testbench    |
    |                  |
    |  Stimulus        |
    |  Expected Result |
    |  Checks          |
    +--------+---------+
             |
             v
    +------------------+
    |       PE         |
    |                  |
    |   RTL Design     |
    +--------+---------+
             |
             v
          Outputs
             |
             v
       Testbench Check

The testbench provides inputs and checks whether the RTL produces the expected behavior.

---

## 28. PE Verification Through RTL Simulation

The PE was compiled and simulated using the project simulation environment.

The verification included:

    Reset
       ↓
    Weight loading
       ↓
    Activation input
       ↓
    Partial-sum input
       ↓
    MAC computation
       ↓
    Activation forwarding
       ↓
    Output checking

The PE simulation successfully demonstrated the intended functionality.

---

## 29. Waveform Analysis

Waveforms are used to inspect the internal and external behavior of the RTL during simulation.

Important signals to inspect include:

    clk
    reset
    weight
    weight_load
    activation_in
    activation_out
    psum_in
    psum_out

The waveform should be used to verify:

- Correct reset behavior
- Correct weight loading
- Weight retention
- Correct activation propagation
- Correct multiplication
- Correct accumulation
- Correct output timing

Waveform inspection is an important debugging step before array integration.

---

## 30. RTL Verification Philosophy

Verification is performed incrementally.

The principle is:

    Verify small block
          ↓
    Integrate block
          ↓
    Verify integration
          ↓
    Build larger system
          ↓
    Verify complete system

This prevents errors from being hidden inside a large design.

For the current project:

    PE
      ↓
    Verified

The next stage is:

    4×4 PE Array
      ↓
    Verify

---

## 31. Current RTL Implementation Status

| RTL Component | Status |
|---------------|--------|
| Processing Element | Completed |
| PE weight register | Completed |
| PE weight loading | Completed |
| PE MAC datapath | Completed |
| PE activation forwarding | Completed |
| PE partial-sum handling | Completed |
| PE reset logic | Completed |
| PE testbench | Completed |
| PE simulation | Completed |
| PE functional verification | Completed |
| 4×4 array RTL | In Progress |
| Array interconnection | Planned |
| Array controller | Planned |
| Input buffer | Planned |
| Weight buffer | Planned |
| Output handling | Planned |
| Top-level RTL | Planned |

---

## 32. RTL Quality Requirements

Before the RTL is considered ready for synthesis, it should satisfy:

- Clean compilation
- No unintended latches
- No unintended combinational loops
- No unresolved signals
- No width-mismatch warnings where avoidable
- Correct reset behavior
- Correct clocked behavior
- Deterministic outputs
- Verified functionality
- Consistent coding style
- Synthesizable constructs

Simulation warnings should be reviewed rather than automatically ignored.

---

## 33. Future RTL Integration

The next RTL development stage is the 4×4 systolic array.

The expected hierarchy is:

    top
     |
     +-- systolic_array
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

This hierarchy allows the verified PE to be reused rather than rewriting the MAC logic for every array element.

---

## 34. Array Integration Principles

When integrating the 16 PEs, the following principles will be maintained:

### Identical PE instances

All PEs should use the same verified PE module.

### Structured interconnection

Neighboring PEs should be connected according to the defined systolic dataflow.

### Controlled weight loading

Weights should be loaded into the correct PE locations.

### Correct activation scheduling

Activations should reach the intended PEs at the correct cycles.

### Correct partial-sum handling

Partial sums must be initialized and accumulated correctly.

### Boundary handling

PEs at the edges of the array must have correct external connections.

---

## 35. Simulation Flow

The general simulation workflow is:

    Write RTL
       ↓
    Write Testbench
       ↓
    Compile
       ↓
    Elaborate
       ↓
    Run Simulation
       ↓
    Generate Waveform
       ↓
    Inspect Signals
       ↓
    Compare Expected vs Actual
       ↓
    Fix RTL if required
       ↓
    Re-run Verification

The same methodology will be applied to array-level verification.

---

## 36. Regression Testing

As the project grows, previously verified functionality should continue to be tested.

For example, after array integration:

    PE tests
        +
    Array tests
        +
    Matrix multiplication tests

should all be retained.

This prevents new changes from breaking previously verified functionality.

---

## 37. Version Control Considerations

RTL source files should be maintained under Git version control.

The repository should preserve:

- RTL source
- Testbench source
- Simulation scripts
- Documentation
- Relevant configuration files

Generated files should generally not be committed unless they are intentionally required for reproducibility.

Examples of files that may be excluded:

    simulator-generated binaries
    temporary files
    large waveform databases
    build artifacts

A `.gitignore` file should be used to manage generated artifacts.

---

## 38. Reproducible Simulation

The project should aim for reproducible simulation.

A new user should eventually be able to:

    Clone repository
          ↓
    Install required simulator
          ↓
    Run simulation command/script
          ↓
    Observe PASS/FAIL result

The exact simulator and commands should be documented in the project's simulation documentation.

---

## 39. ASIC Design Flow Compatibility

The RTL is being developed with the eventual goal of following an ASIC-oriented flow.

The planned flow is:

    System Specification
          ↓
    Architecture
          ↓
    RTL Design
          ↓
    Functional Verification
          ↓
    RTL Lint
          ↓
    Logic Synthesis
          ↓
    Gate-Level Netlist
          ↓
    Static Timing Analysis
          ↓
    Floorplanning
          ↓
    Placement
          ↓
    Clock Tree Synthesis
          ↓
    Routing
          ↓
    Physical Verification
          ↓
    GDSII

The current project is at the RTL implementation and functional verification stage.

---

## 40. RTL to GDS Perspective

The long-term objective is to take the verified RTL through the standard digital ASIC implementation flow.

The important distinction is:

    RTL simulation
          ≠
    Synthesized hardware
          ≠
    Physical layout

Therefore, successful RTL simulation establishes functional correctness at the RTL level but does not by itself establish:

- Timing closure
- Area efficiency
- Power efficiency
- Physical design correctness
- Manufacturability

These will require later stages of the project.

---

## 41. Current Development Boundary

The current verified implementation boundary is:

    +----------------------+
    | Processing Element   |
    |                      |
    | Weight Storage       |
    | Activation Path     |
    | Multiplier           |
    | Accumulator           |
    +----------+-----------+
               |
               v
          PE Testbench
               |
               v
           Simulation
               |
               v
          Functional PASS

The complete array and accelerator are still under development.

---

## 42. Summary

The accelerator is being implemented using modular SystemVerilog RTL.

The Processing Element is the fundamental RTL block and contains the required functionality for:

- Weight storage
- Weight loading
- Activation reception
- Activation forwarding
- Multiplication
- Partial-sum accumulation
- MAC computation
- Synchronous operation
- Reset

The PE has been successfully simulated and functionally verified.

The next RTL stage is to instantiate the verified PE 16 times and construct the complete 4×4 systolic array.

After array-level verification, the design can be extended toward control, buffering, top-level integration, synthesis, timing analysis, physical design, and eventually GDSII generation.

The RTL methodology emphasizes modularity, synthesizability, reproducibility, incremental verification, and eventual ASIC-flow compatibility.
