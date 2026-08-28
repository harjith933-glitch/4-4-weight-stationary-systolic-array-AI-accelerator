# Processing Element (PE)

## 1. Overview

The Processing Element (PE) is the fundamental computational unit of the 4×4 weight-stationary systolic array.

The PE performs a Multiply-Accumulate (MAC) operation while also providing the data-forwarding functionality required to construct the systolic array.

The fundamental computation is:

    PSUM_out = PSUM_in + (Activation × Weight)

The PE also forwards the activation to the next Processing Element.

The PE has been implemented using Verilog RTL and functionally verified through simulation.

---

## 2. Role of the Processing Element

The PE is responsible for four primary functions:

1. Accepting and storing a weight.
2. Receiving activation data.
3. Performing multiplication and accumulation.
4. Forwarding activation data to the next PE.

Conceptually:

             Weight
                |
                v
        +---------------+
        | Weight        |
        | Register      |
        +-------+-------+
                |
                v
    Activation --> Multiplier
                       |
                       v
                   Product
                       |
                       v
    PSUM_in --------> Adder
                       |
                       v
                    PSUM_out

    Activation_in ---> Forwarding ---> Activation_out

The PE combines local computation with inter-PE communication.

---

## 3. PE Architecture

The conceptual PE architecture is:

    +------------------------------------------------+
    |             Processing Element                 |
    |                                                |
    |  Weight Input                                  |
    |       |                                        |
    |       v                                        |
    |  +-----------+                                 |
    |  |  Weight   |                                 |
    |  |  Register |                                 |
    |  +-----+-----+                                 |
    |        |                                       |
    |        v                                       |
    |  +-----------+                                 |
    |  | Multiplier| <----- Activation Input         |
    |  +-----+-----+                                 |
    |        |                                       |
    |        v                                       |
    |  +-----------+                                 |
    |  |   Adder   | <----- Partial Sum Input        |
    |  +-----+-----+                                 |
    |        |                                       |
    |        v                                       |
    |   Partial Sum Output                           |
    |                                                |
    |  Activation Input                              |
    |        |                                       |
    |        v                                       |
    |  Activation Forwarding                         |
    |        |                                       |
    |        v                                       |
    |  Activation Output                             |
    |                                                |
    +------------------------------------------------+

---

## 4. PE Inputs

The PE contains the signals required for weight loading, computation, synchronization, and data movement.

The conceptual inputs are:

### Clock

Synchronizes sequential operations.

### Reset

Initializes the PE state.

### Activation Input

Carries the activation value used by the multiplier.

### Weight Input

Carries the weight that can be loaded into the PE.

### Weight Load Control

Controls when the weight input is stored inside the PE.

### Partial Sum Input

Carries the accumulated value from a previous computational stage.

The exact signal names and widths are defined by the Verilog RTL source.

---

## 5. PE Outputs

The conceptual PE outputs are:

### Activation Output

Forwards the activation to the next PE.

### Partial Sum Output

Provides the updated accumulated result.

The fundamental output relationship is:

    PSUM_out = PSUM_in + (Activation × Stored_Weight)

The exact timing of these outputs is determined by the Verilog implementation.

---

## 6. Weight Storage

The PE contains local storage for the weight.

This is the key feature that enables weight-stationary operation.

The conceptual operation is:

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
         |
         v
    MAC Computation

Once the weight has been loaded, the PE retains it for subsequent computations until another weight-loading operation or reset changes the stored value.

---

## 7. Weight Loading Operation

The weight-loading operation occurs when the weight-load control is asserted.

Conceptually:

    if weight_load = 1

        stored_weight <= weight_input

The weight is captured on the appropriate clock edge according to the Verilog sequential logic.

After loading:

    weight_load = 0

the stored weight remains unchanged.

This behavior enables the PE to reuse the same weight for multiple activation values.

---

## 8. Weight-Stationary Behavior

The PE follows the weight-stationary principle:

    Weight → Stored locally
    Activation → Moves through PE
    Partial Sum → Accumulated

The weight therefore does not need to be continuously reloaded for every activation.

For example:

    Load Weight = W

    Activation A0 → MAC with W
    Activation A1 → MAC with W
    Activation A2 → MAC with W
    Activation A3 → MAC with W

The same stored weight participates in multiple operations.

---

## 9. Multiplication

The multiplication stage calculates:

    Product = Activation × Stored_Weight

The activation comes from the PE input while the weight comes from the internal weight register.

Conceptually:

    Activation Input
          |
          v
       +------+
       |  ×   | <----- Stored Weight
       +--+---+
          |
          v
       Product

The product is then supplied to the accumulation stage.

---

## 10. Accumulation

The accumulation stage adds the multiplication result to the incoming partial sum.

The operation is:

    PSUM_out = PSUM_in + Product

Since:

    Product = Activation × Stored_Weight

the complete operation is:

    PSUM_out =
        PSUM_in + (Activation × Stored_Weight)

This forms the MAC operation.

---

## 11. MAC Operation

The PE performs:

    Multiply
        ↓
    Accumulate

or mathematically:

    PSUM_out = PSUM_in + (Activation × Weight)

For example, assume:

    Activation = 4
    Weight = 5
    PSUM_in = 10

Then:

    Product = 4 × 5
            = 20

Therefore:

    PSUM_out = 10 + 20

    PSUM_out = 30

This is the fundamental operation performed by the PE.

---

## 12. Multiple MAC Operations

A series of MAC operations produces a dot product.

Assume:

    Activation values:

    A0 = 2
    A1 = 3
    A2 = 4
    A3 = 5

and:

    Weight values:

    W0 = 1
    W1 = 2
    W2 = 3
    W3 = 4

Starting with:

    PSUM = 0

The accumulation is:

    PSUM1 = 0 + (2 × 1)
          = 2

    PSUM2 = 2 + (3 × 2)
          = 8

    PSUM3 = 8 + (4 × 3)
          = 20

    PSUM4 = 20 + (5 × 4)
          = 40

Final result:

    PSUM = 40

This represents:

    2×1 + 3×2 + 4×3 + 5×4 = 40

---

## 13. Activation Forwarding

In addition to performing the MAC operation, the PE forwards activation data.

Conceptually:

    Activation_in
          |
          v
        +-----+
        | PE  |
        +--+--+
           |
           v
    Activation_out

The forwarded activation can then be supplied to the neighboring PE.

This is required to construct the horizontal systolic data path.

---

## 14. Activation Propagation

When multiple PEs are connected:

    PE00 → PE01 → PE02 → PE03

the activation can propagate from one PE to the next.

For example:

    Activation
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

The activation forwarding mechanism implemented in the PE forms the basis for this interconnection.

---

## 15. Partial-Sum Interface

The PE receives a partial sum and produces an updated partial sum.

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

The operation is:

    PSUM_out = PSUM_in + (Activation × Weight)

The exact direction and interconnection of partial sums will be defined during complete array integration.

---

## 16. Reset Operation

Reset places the PE into a known initial state.

Reset is important because internal registers must not begin operation with unknown values.

The reset operation is intended to initialize the PE state required for:

- Weight storage
- Partial-sum operation
- Activation forwarding
- Normal computation

The exact reset polarity and reset implementation must match the actual Verilog RTL.

---

## 17. Clocked Operation

The PE operates synchronously with the system clock.

Sequential state changes occur on the appropriate clock edge.

The conceptual timing is:

    Clock Edge
        |
        +---- Weight Register Update
        |
        +---- Data Register Update
        |
        +---- Output State Update

The exact register placement determines the cycle latency of the PE.

---

## 18. Verilog RTL Structure

The PE is implemented as a Verilog module.

The basic structure of a Verilog module is:

    module pe (
        input  ...,
        input  ...,
        output ...
    );

        // Internal signals

        // Sequential logic

        // Combinational logic

    endmodule

The actual source code in the `rtl/` directory is the authoritative definition of the module interface and implementation.

---

## 19. Sequential Logic

Sequential behavior is implemented using clocked Verilog procedural blocks.

The standard form is:

    always @(posedge clk)

For example, a clocked register operation follows the form:

    always @(posedge clk) begin
        register <= next_value;
    end

Non-blocking assignment (`<=`) is used for sequential register updates.

---

## 20. Combinational Logic

Combinational operations can be described using continuous assignments or combinational procedural blocks.

The standard Verilog procedural form is:

    always @(*)

Arithmetic relationships can also be represented using continuous assignments.

For example:

    assign product = activation * weight;

The exact implementation should follow the current RTL source.

---

## 21. Internal Weight Register

The weight register is an important state element inside the PE.

Its function is:

    Weight Input
         ↓
    Weight Register
         ↓
    Stored Weight
         ↓
    Multiplier

The register allows the weight to remain stationary during normal computation.

---

## 22. Arithmetic Datapath

The arithmetic datapath consists primarily of:

    Activation
         |
         v
    Multiplier
         |
         v
       Product
         |
         v
       Adder
         ^
         |
      PSUM_in
         |
         v
      PSUM_out

The datapath implements:

    PSUM_out = PSUM_in + (Activation × Weight)

This is the core computational path of the PE.

---

## 23. Data Path and Control Path

The PE can be conceptually divided into two categories.

### Data Path

The data path handles:

- Activation
- Weight
- Multiplication
- Partial sum
- Activation forwarding

### Control Path

The control path handles:

- Clock
- Reset
- Weight loading
- Any required validity/control signals

The separation of datapath and control simplifies understanding and debugging of the RTL.

---

## 24. PE Testbench

A dedicated Verilog testbench was created to verify the Processing Element.

The testbench provides stimulus to the PE and checks the resulting behavior.

Conceptually:

    +----------------------+
    |      Testbench       |
    |                      |
    |  Clock Generation    |
    |  Reset               |
    |  Weight Stimulus     |
    |  Activation Stimulus |
    |  PSUM Stimulus       |
    |  Output Checking     |
    +----------+-----------+
               |
               v
        +--------------+
        |      PE      |
        | Verilog RTL  |
        +--------------+
               |
               v
          PE Outputs
               |
               v
          Verification

---

## 25. PE Verification Sequence

The PE verification follows a controlled sequence.

### Step 1

Apply reset.

### Step 2

Load the required weight.

### Step 3

Provide activation data.

### Step 4

Provide the initial partial sum.

### Step 5

Allow the PE to perform the MAC operation.

### Step 6

Check the partial-sum output.

### Step 7

Check activation forwarding.

### Step 8

Repeat with additional values where required.

This verifies the primary functions of the PE.

---

## 26. Weight Loading Verification

Weight loading is verified by applying a known weight and asserting the weight-load control.

The testbench then verifies that the loaded weight is used during the MAC operation.

Conceptually:

    Testbench
        |
        | Weight = W
        | Load = 1
        v
       PE
        |
        v
    Weight stored
        |
        v
    Load = 0
        |
        v
    MAC operation

The correct result demonstrates that the weight was successfully captured and retained.

---

## 27. Activation Forwarding Verification

Activation forwarding is verified by applying an activation at the PE input and observing the activation output.

The expected behavior is:

    Activation_in
          |
          v
         PE
          |
          v
    Activation_out

The testbench confirms that the activation is forwarded according to the expected timing of the RTL implementation.

This functionality is essential for later PE-to-PE array integration.

---

## 28. MAC Verification

The MAC operation is verified using known input values.

For example:

    Weight = 5
    Activation = 4
    PSUM_in = 10

Expected:

    PSUM_out = 10 + (4 × 5)

    PSUM_out = 30

The testbench compares the RTL result with the expected result.

If the values match, the MAC operation passes the test.

---

## 29. Example Verification Calculation

Consider:

    Weight = 3
    Activation = 7
    PSUM_in = 2

Then:

    Product = 7 × 3
            = 21

Expected:

    PSUM_out = 2 + 21

    PSUM_out = 23

The testbench should verify:

    Actual PSUM_out = 23

A mismatch indicates an RTL or testbench problem that must be investigated.

---

## 30. Expected vs Actual Checking

Functional verification is based on comparing:

    Expected Result

against:

    Actual RTL Result

Conceptually:

    Expected = PSUM_in + (Activation × Weight)

    Actual   = PE PSUM_out

Then:

    Expected == Actual

If equal:

    PASS

Otherwise:

    FAIL

This method provides an objective functional check.

---

## 31. Waveform Verification

Waveform inspection is used in addition to numerical checking.

Important signals include:

    clk
    reset
    weight
    weight_load
    activation_in
    activation_out
    psum_in
    psum_out

The waveform is inspected to verify:

- Reset timing
- Weight loading
- Weight retention
- Activation movement
- MAC timing
- Partial-sum update
- Output timing

Waveform analysis is particularly useful for identifying cycle-level errors.

---

## 32. PE Verification Result

The PE testbench successfully demonstrated the intended PE functionality.

The verified behavior includes:

    Weight Loading
          ↓
    Weight Storage
          ↓
    Activation Input
          ↓
    Multiplication
          ↓
    Partial-Sum Accumulation
          ↓
    Partial-Sum Output

and:

    Activation Input
          ↓
    Activation Forwarding
          ↓
    Activation Output

The PE therefore provides the required basic functionality for the systolic array.

---

## 33. What Has Been Verified

The following PE-level functionality has been verified:

- PE reset
- Weight loading
- Weight storage
- Activation input
- Activation forwarding
- MAC operation
- Partial-sum calculation
- Output behavior
- Clocked operation
- RTL simulation

The exact test vectors and waveform evidence are maintained in the project's verification and simulation artifacts.

---

## 34. What Has Not Yet Been Verified

PE-level verification does not establish complete accelerator functionality.

The following require higher-level verification:

- 4×4 PE interconnection
- Complete activation scheduling
- Complete weight mapping
- Full partial-sum routing
- 4×4 matrix multiplication
- Array-level latency
- Array throughput
- Full accelerator control
- Synthesis timing
- Area
- Power
- Physical implementation

These will be addressed in subsequent development stages.

---

## 35. PE Reusability

The PE is designed to be reused as the basic building block of the complete systolic array.

The same Verilog module can be instantiated multiple times:

    PE module
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

This provides modularity and reduces duplicated RTL.

---

## 36. PE as a Reusable IP Block

The verified PE can be considered the foundation of the accelerator's computational datapath.

A reusable PE should have:

- Clearly defined inputs
- Clearly defined outputs
- Deterministic timing
- Well-defined reset behavior
- Verified arithmetic functionality
- Verified data-forwarding functionality
- Synthesizable Verilog RTL

These properties make the PE suitable for integration into the larger systolic array.

---

## 37. Design Considerations

Several considerations are important when integrating the PE into the array.

### Timing

The neighboring PE must receive data at the correct cycle.

### Width

Activation, weight, and partial-sum widths must be compatible.

### Reset

All PEs must enter a known state.

### Weight Mapping

Each PE must receive the correct weight.

### Activation Scheduling

Activations must be injected at the correct time.

### Partial-Sum Initialization

The correct initial partial sum must be supplied.

### Boundary Connections

Edge PEs require appropriate external connections.

---

## 38. Integration Plan

The PE will be integrated incrementally.

The planned sequence is:

    Verified PE
         ↓
    Two-PE connection
         ↓
    Row-level connection
         ↓
    4×4 PE array
         ↓
    Array-level testbench
         ↓
    Matrix multiplication
         ↓
    Top-level accelerator

Testing at each stage reduces the risk of introducing difficult-to-debug system-level errors.

---

## 39. Current PE Status

| PE Feature | Status |
|------------|--------|
| Verilog module | Completed |
| Weight register | Completed |
| Weight loading | Completed |
| Activation input | Completed |
| Activation forwarding | Completed |
| Multiplier | Completed |
| Accumulator | Completed |
| Partial-sum interface | Completed |
| Reset | Completed |
| PE testbench | Completed |
| RTL simulation | Completed |
| Functional verification | Completed |
| Array integration | Next stage |

---

## 40. Summary

The Processing Element is the fundamental computational block of the 4×4 weight-stationary systolic array.

It performs:

    PSUM_out = PSUM_in + (Activation × Weight)

The PE stores the weight locally, receives activation data, performs multiplication and accumulation, and forwards the activation to the next Processing Element.

The PE has been implemented in Verilog RTL and successfully verified through simulation.

The verified PE now provides the foundation for constructing the complete 4×4 systolic array.

The next stage is to connect multiple verified PEs, establish the array-level dataflow, verify cycle-by-cycle propagation, and demonstrate complete matrix multiplication.
