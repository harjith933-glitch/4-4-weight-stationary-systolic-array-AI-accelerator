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

The Processing Element (PE) has already been simulated and verified.

The same methodology will later be extended to the complete 4×4 systolic array.

---

## 2. Purpose of Simulation

Simulation is used to verify that the RTL behaves as intended.

The primary objectives are:

- Verify reset behavior.
- Verify weight loading.
- Verify weight storage.
- Verify activation processing.
- Verify activation forwarding.
- Verify multiplication.
- Verify partial-sum accumulation.
- Verify MAC operation.
- Verify output timing.
- Identify RTL design errors before synthesis.

Simulation does not replace synthesis or physical timing analysis.

---

## 3. Simulation Directory

Simulation-related files are maintained separately from the RTL.

The project structure is:

    ~/systolic/
    |
    +-- rtl/
    |     +-- Verilog design files
    |
    +-- tb/
    |     +-- Verilog testbenches
    |
    +-- sim/
    |     +-- Simulation-related files
    |
    +-- wave/
    |     +-- Waveform files
    |
    +-- docs/
          +-- Documentation

This separation keeps the project organized.

---

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
