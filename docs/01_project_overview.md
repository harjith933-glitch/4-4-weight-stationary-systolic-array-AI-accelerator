2.1 Introduction

Explain what a systolic array is.

A systolic array is a network of interconnected processing elements in which data moves rhythmically between neighboring processing elements while computation occurs concurrently. The architecture is particularly suitable for highly parallel matrix multiplication and AI/ML workloads.

2.2 Project objective

Explain:

The objective of this project is to design and verify a 4×4 weight-stationary systolic array accelerator using synthesizable SystemVerilog RTL. The architecture is intended to accelerate matrix multiplication by exploiting spatial parallelism and data reuse.

2.3 Target computation

Explain:

$$ C=A\times B $$

For:

$$ A_{4\times4}\times B_{4\times4}=C_{4\times4} $$

Each output element is:

$$ C_{ij}=\sum_{k=0}^{3}A_{ik}B_{kj} $$
2.4 Why systolic architecture?

Document:

Parallel computation
Data reuse
Local communication
Reduced movement of operands
Regular structure
Scalability
Suitability for AI accelerators
2.5 Why 4×4?

Explain that the 4×4 architecture is selected as a manageable proof-of-concept that demonstrates:

Spatial parallelism
PE-to-PE communication
Weight reuse
Partial-sum accumulation
Systolic data movement
2.6 Target application

Mention:

Matrix multiplication
Neural-network workloads
AI/ML acceleration
Convolution-derived matrix operations
Dense linear algebra

Don't claim that the current implementation supports convolution unless we actually implement it.

2.7 Project scope

Separate:

Current scope

PE design and verification.

Overall target

Complete 4×4 accelerator.

This distinction is important.
