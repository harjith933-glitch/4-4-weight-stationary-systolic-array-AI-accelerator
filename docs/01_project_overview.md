## 2.3 Target Computation

The primary mathematical operation targeted by the systolic array is matrix multiplication.

For two matrices:

A = [a_ij]
B = [b_ij]

the output matrix is:

C = A × B

For a 4×4 matrix multiplication:

A(4×4) × B(4×4) = C(4×4)

Each element of the output matrix is calculated as:

C[i][j] = A[i][0] × B[0][j]
        + A[i][1] × B[1][j]
        + A[i][2] × B[2][j]
        + A[i][3] × B[3][j]

In general:

C[i][j] = Σ(A[i][k] × B[k][j])

where:

- i = output row index
- j = output column index
- k = accumulation index
- A = input/activation matrix
- B = weight matrix
- C = output matrix

For a 4×4 systolic array, 16 Processing Elements (PEs) operate in parallel to perform the multiply-accumulate (MAC) operations required for matrix multiplication.
