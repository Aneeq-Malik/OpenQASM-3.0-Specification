# OpenQASM 3.0 Complete Language Reference Guide
## A Comprehensive Technical Documentation

**Version**: 1.0  
**Date**: August 2025  
**Based on**: OpenQASM 3.0 Specification

---

## Table of Contents

1. [Introduction](#introduction)
2. [Language Fundamentals](#language-fundamentals)
3. [Type System](#type-system)
4. [Quantum Operations](#quantum-operations)
5. [Classical Control](#classical-control)
6. [Subroutines and Functions](#subroutines-and-functions)
7. [Circuit Timing](#circuit-timing)
8. [Pulse-Level Control](#pulse-level-control)
9. [Advanced Features](#advanced-features)
10. [References](#references)

---

## 1. Introduction

### 1.1 Overview

OpenQASM (Open Quantum Assembly Language) is a programming language designed for describing quantum circuits and algorithms for execution on quantum computers. It is designed to be an intermediate representation that can be used by higher-level compilers to communicate with quantum hardware, and allows for the description of a wide range of quantum operations, as well as classical feed-forward flow control based on measurement outcomes.

OpenQASM 3.0 represents a significant evolution from version 2.0, introducing comprehensive classical computing capabilities, advanced control flow, and pulse-level control mechanisms. This document provides an exhaustive reference for all language constructs available in OpenQASM 3.0.

### 1.2 Design Philosophy

The human-readable form of OpenQASM is a simple, case-sensitive textual language. Statements are separated by semicolons and whitespace is ignored. The language possesses a dual nature, functioning both as an assembly language for quantum operations and as a hardware description language for quantum circuits.

**Key Design Principles:**
- Hardware-agnostic representation while allowing hardware-specific optimizations
- Strong type system with explicit casting rules
- Real-time classical computation capabilities
- Hierarchical composition through gates and subroutines
- Explicit timing control for synchronized operations
- Extensibility through pragmas and annotations

---

## 2. Language Fundamentals

### 2.1 Version Declaration

The version declaration should be the first non-comment statement in any OpenQASM program. It identifies the language version being used and ensures compatibility with parsers and compilers.

```qasm
OPENQASM 3.0;
```

**Syntax Rules:**
- Must appear before any other statements (except comments)
- Can only appear once per file
- Optional but strongly recommended for version control
- Format: `OPENQASM <major>.<minor>;`

**Reference**: [OpenQASM Language Specification](https://openqasm.com/language/)

### 2.2 Comments

OpenQASM supports two comment styles inherited from C/C++:

```qasm
// Single-line comment extending to end of line
/* Multi-line comment
   spanning multiple
   lines */
   
/* Comments can /* be nested */ in OpenQASM 3.0 */
```

**Technical Details:**
- Comments are completely ignored by the parser
- Can appear anywhere whitespace is allowed
- Unicode characters are permitted within comments
- Nested block comments are supported (unlike C/C++)
- Documentation comments are not formally specified but may be supported by tools

**Reference**: [Comments - OpenQASM Documentation](https://openqasm.com/versions/3.0/language/comments.html)

### 2.3 Identifiers

Identifiers must begin with a letter [A-Za-z], an underscore or an element from the Unicode character categories Lu/Ll/Lt/Lm/Lo/Nl. The set of permissible continuation characters consists of all members of the aforementioned character sets with the addition of decimal numerals [0-9].

**Identifier Rules:**
- Case-sensitive
- Cannot be reserved keywords
- Must be unique within their scope
- Physical qubits use special `$n` syntax
- Maximum length is implementation-defined (typically 1024 characters)

**Reserved Keywords** (cannot be used as identifiers):
```
angle, array, barrier, bit, bool, box, cal, case, complex, const, 
creg, ctrl, def, default, defcal, defcalgrammar, delay, duration, 
durationof, else, end, extern, false, float, for, gate, gphase, 
if, in, include, input, int, inv, let, measure, negctrl, OPENQASM, 
output, pow, pragma, qreg, qubit, readonly, reset, return, sizeof, 
stretch, switch, true, U, uint, void, while
```

**Reference**: [Types and Casting - OpenQASM Documentation](https://openqasm.com/language/types.html)

### 2.4 File Inclusion

The `include` directive allows importing definitions from external files. OpenQASM 3 specifications are available at https://openqasm.com/, where standard library files are defined.

```qasm
include "stdgates.inc";  // Standard gate library
include "qelib1.inc";    // Legacy compatibility library
include "custom_gates.qasm"; // User-defined libraries
```

**Include Mechanism Details:**
- Textual inclusion (preprocessor-style)
- Relative paths resolved from current file location
- Circular dependencies result in compilation error
- Include guards not built-in (must be implemented via pragmas if needed)
- Must appear at global scope only

**Standard Library Contents** (`stdgates.inc`):
- Single-qubit gates: `X, Y, Z, H, S, T, SX, SXdg, Sdg, Tdg`
- Rotation gates: `RX(θ), RY(θ), RZ(θ), U1(λ), U2(φ,λ), U3(θ,φ,λ)`
- Two-qubit gates: `CX, CY, CZ, CH, SWAP, iSWAP, DCX, ECR`
- Three-qubit gates: `CCX (Toffoli), CSWAP (Fredkin)`
- Phase gates: `P(λ), CP(λ), CPhase(λ)`
- Rotation variants: `CRX(θ), CRY(θ), CRZ(θ)`

**Reference**: [Language - OpenQASM Documentation](https://openqasm.com/language/)

---

## 3. Type System

### 3.1 Quantum Types

#### 3.1.1 Virtual Qubits

Virtual qubits represent logical quantum bits that may be mapped to physical hardware qubits by the compiler.

```qasm
qubit q;           // Single qubit declaration
qubit[5] qreg;     // Quantum register of 5 qubits
qubit q1, q2, q3;  // ERROR: Multiple declarations not allowed
```

**Qubit Properties:**
- Global scope only (cannot be declared in gates or subroutines)
- Start in undefined state (must be reset before use)
- Indexed access: `qreg[0]`, `qreg[1]`, etc.
- Compile-time constant indices required for safety
- Runtime indexing may be allowed (implementation-defined)
- Cannot be copied or cloned (no-cloning theorem)

#### 3.1.2 Physical Qubits

Physical qubits directly reference hardware qubits using the `$` notation:

```qasm
reset $0;          // Reset physical qubit 0
CX $0, $1;        // CNOT between physical qubits 0 and 1
defcal X $2 { ... } // Calibration for X gate on physical qubit 2
```

**Physical Qubit Characteristics:**
- No explicit declaration needed
- Global and immutable
- Primarily used in `defcal` blocks and low-level control
- Bypass logical-to-physical mapping
- Hardware-specific numbering

**Reference**: [Types - OpenQASM Documentation](https://openqasm.com/language/types.html)

### 3.2 Classical Types

#### 3.2.1 Bit Type

The fundamental classical type representing binary values:

```qasm
bit b;                    // Single bit
bit[8] byte;             // 8-bit register
bit[16] word = "0000111100001111"; // Initialized with bit string
bit flag = 1;            // Single bit initialized to 1
```

**Bit Operations:**
- Bitwise: `~` (NOT), `&` (AND), `|` (OR), `^` (XOR)
- Shifts: `<<` (left), `>>` (right)
- Comparisons: `==`, `!=`
- Measurement results stored as bits

#### 3.2.2 Integer Types

Signed and unsigned integers with explicit bit widths:

```qasm
int[32] counter = -42;    // 32-bit signed integer
uint[16] index = 1000;    // 16-bit unsigned integer
int x;                    // Default width (implementation-defined)
const int[8] MAX = 127;   // Compile-time constant
```

**Integer Specifications:**
- Width: 1 to implementation-defined maximum (typically 64)
- Two's complement representation for signed
- Overflow behavior: Wraparound (modulo 2^n)
- Arithmetic: `+`, `-`, `*`, `/`, `%` (mod)
- Bitwise operations apply to fixed-width types

#### 3.2.3 Floating-Point Types

IEEE 754 compliant floating-point numbers:

```qasm
float[32] x = 3.14159;    // Single precision
float[64] y = 2.718281828; // Double precision
float[16] z = 1.5;        // Half precision
float w;                  // Default precision (implementation-defined)
```

**Floating-Point Details:**
- Supported widths: 16 (half), 32 (single), 64 (double), 128 (quad)
- Special values: `inf`, `-inf`, `nan`
- Scientific notation: `1.23e-4`, `6.022e23`
- Mathematical functions available for compile-time evaluation

#### 3.2.4 Complex Numbers

Complex numbers built from floating-point components:

```qasm
complex[float[32]] z1 = 3.0 + 4.0im;     // 3+4i
complex[float[64]] z2 = 2.5 - 1.5im;     // 2.5-1.5i
complex[float[32]] z3 = 5.0im;           // 0+5i
```

**Complex Operations:**
- Arithmetic: `+`, `-`, `*`, `/`
- Functions: `real()`, `imag()`, `abs()`, `arg()`
- Conjugate via `conj()` function
- Imaginary unit: `im` or `j`

#### 3.2.5 Angle Type

Specialized type for rotation angles in radians:

```qasm
angle[20] theta = pi/2;    // 20-bit precision angle
angle phi = 0.7854;        // Default precision
angle[32] alpha = 2*pi/3;  // High-precision angle
```

**Angle Properties:**
- Internally stored as fixed-point in [0, 2π)
- Automatic modulo 2π on assignment
- Used for gate parameters
- Convertible to/from float

#### 3.2.6 Boolean Type

Logical boolean values:

```qasm
bool flag = true;
bool result = (x > 5) && (y < 10);
bool isReady = false;
```

**Boolean Details:**
- Values: `true`, `false` only
- No size parameter (unlike bit)
- Result type of comparison operators
- Used in control flow conditions

#### 3.2.7 Duration and Stretch Types

Time-related types for circuit timing:

```qasm
duration t1 = 100ns;       // 100 nanoseconds
duration t2 = 2.5us;       // 2.5 microseconds
stretch s;                 // Variable timing placeholder
duration total = t1 + t2;  // Duration arithmetic
```

**Time Units:**
- `dt` - Device time unit (hardware-specific)
- `ns` - Nanoseconds
- `us`/`μs` - Microseconds
- `ms` - Milliseconds
- `s` - Seconds

**Reference**: [Types and Casting - OpenQASM Documentation](https://openqasm.com/language/types.html)

### 3.3 Composite Types

#### 3.3.1 Arrays

Fixed-size, multi-dimensional classical arrays:

```qasm
array[int[32], 5] vec = {1, 2, 3, 4, 5};
array[float[64], 3, 3] matrix = {{1,0,0}, {0,1,0}, {0,0,1}};
array[bit, 8] flags;
array[complex[float[32]], 10] amplitudes;
```

**Array Features:**
- Fixed size at compile time
- Zero-indexed
- Row-major ordering for multi-dimensional
- Element access: `arr[i]`, `matrix[i][j]`
- Negative indices count from end: `arr[-1]` is last element
- Cannot contain quantum types

Two or more classical arrays of the same fundamental type can be concatenated to form an array of the same type whose size is the sum of the sizes of the individual arrays.

#### 3.3.2 Register Aliasing

The `let` statement creates aliases to register subsets:

```qasm
qubit[8] qreg;
let first_half = qreg[0:3];      // Alias to qubits 0-3
let second_half = qreg[4:7];     // Alias to qubits 4-7
let even = qreg[{0,2,4,6}];      // Non-contiguous selection
let reversed = qreg[7:0:-1];     // Reverse order

bit[16] breg;
let nibble = breg[0:3];          // Works for classical registers too
```

**Aliasing Properties:**
- No data copying (reference semantics)
- Original and alias share same storage
- Modifications visible through all aliases
- Scope rules apply to alias names

### 3.4 Type Casting

Explicit and implicit type conversions with defined semantics:

```qasm
// Explicit casting
int[8] i = 42;
float[32] f = float[32](i);      // int to float
bit[8] b = bit[8](i);           // int to bits
angle[20] a = angle[20](f);     // float to angle (mod 2π)

// Implicit casting (promotions)
float[64] result = i + 3.14;    // i promoted to float[64]
complex[float[32]] z = 5;       // 5 promoted to 5.0+0.0im
```

**Casting Rules Matrix:**
| From → To | bit | int | uint | float | angle | bool | complex |
|-----------|-----|-----|------|-------|-------|------|---------|
| bit       | ✓   | ✓   | ✓    | ✓     | ✓     | ✓    | ✓       |
| int       | ✓*  | ✓   | ✓    | ✓     | ✓     | ✓    | ✓       |
| uint      | ✓*  | ✓   | ✓    | ✓     | ✓     | ✓    | ✓       |
| float     | ✗   | ✓†  | ✓†   | ✓     | ✓‡    | ✓    | ✓       |
| angle     | ✗   | ✗   | ✗    | ✓     | ✓     | ✗    | ✗       |
| bool      | ✓   | ✓   | ✓    | ✓     | ✗     | ✓    | ✗       |
| complex   | ✗   | ✗   | ✗    | ✗     | ✗     | ✗    | ✓       |

*Width must match  
†Truncation occurs  
‡Modulo 2π applied

**Reference**: [Types and Casting - OpenQASM Documentation](https://openqasm.com/language/types.html)

---

## 4. Quantum Operations

### 4.1 Gate Definition

Gates are unitary quantum operations defined using the `gate` keyword:

```qasm
// Simple gate definition
gate hadamard q {
    U(pi/2, 0, pi) q;
}

// Parameterized gate
gate rx(angle[20] theta) q {
    U(theta, -pi/2, pi/2) q;
}

// Multi-qubit gate
gate cphase(angle[20] lambda) c, t {
    ctrl @ U(0, 0, lambda) c, t;
}

// Composite gate
gate bell a, b {
    hadamard a;
    CX a, b;
}
```

**Gate Definition Rules:**
- Parameters must be `angle` type or convertible to angle
- Body contains only quantum operations (no classical computation)
- No variable declarations within gate body
- Cannot be recursive
- Must be defined before use
- Empty body equals identity operation
- Global scope only

### 4.2 Built-in Gates

OpenQASM 3.0 provides two fundamental built-in gates:

#### 4.2.1 Universal Gate U(θ, φ, λ)

The universal single-qubit unitary operation:

```qasm
U(theta, phi, lambda) q;
```

**Matrix Representation:**
```
U(θ,φ,λ) = [cos(θ/2)           -e^(iλ)sin(θ/2)]
           [e^(iφ)sin(θ/2)   e^(i(φ+λ))cos(θ/2)]
```

**Properties:**
- Can represent any single-qubit unitary (up to global phase)
- Parameters are angles (radians)
- Common gates expressible via U:
  - X gate: `U(pi, 0, pi)`
  - Y gate: `U(pi, pi/2, pi/2)`
  - Z gate: `U(0, 0, pi)`
  - H gate: `U(pi/2, 0, pi)` (up to phase)

#### 4.2.2 Global Phase Gate

The `gphase` gate applies a global phase to the quantum state:

```qasm
gphase(pi/4);           // Global phase e^(iπ/4)
ctrl @ gphase(pi/2) q;  // Controlled phase
```

**Global Phase Properties:**
- Acts as e^(iγ)I on the entire state
- Unobservable for isolated systems
- Matters when controlled or in interference
- No qubit arguments for standalone use

### 4.3 Gate Modifiers

Gate modifiers transform existing gates into new operations:

#### 4.3.1 Control Modifier

```qasm
ctrl @ X q0, q1;              // Controlled-X (CNOT)
ctrl(2) @ X q0, q1, q2;       // Toffoli gate (CCX)
ctrl @ RY(pi/4) c, t;         // Controlled-RY
negctrl @ X q0, q1;           // Anti-controlled X
ctrl @ ctrl @ X q0, q1, q2;  // Same as ctrl(2) @ X
```

**Control Modifier Rules:**
- Prepends control qubit(s) to argument list
- `ctrl(n)` specifies n control qubits
- `negctrl` triggers on |0⟩ instead of |1⟩
- Can be chained for multiple controls
- Affects gate semantics and decomposition

#### 4.3.2 Inverse Modifier

```qasm
inv @ S q;                    // S-dagger gate
inv @ RX(theta) q;            // RX(-theta)
inv @ inv @ H q;              // H (self-inverse)
inv @ (ctrl @ X) q0, q1;     // Inverse of CNOT (itself)
```

**Inverse Properties:**
- Computes gate inverse (conjugate transpose)
- For parameterized gates, negates angles
- Can combine with other modifiers
- Compile-time transformation

#### 4.3.3 Power Modifier

```qasm
pow(2) @ S q;                 // Z gate (S²)
pow(3) @ T q;                 // Equivalent to T·T·T
pow(-1) @ X q;                // Inverse of X (X itself)
pow(0.5) @ Z q;               // Square root of Z
```

**Power Modifier Details:**
- Raises gate to specified power
- Supports fractional powers (if decomposable)
- Negative powers compute inverse
- Power 0 gives identity

### 4.4 Gate Application

Gates are applied to qubits using call syntax:

```qasm
// Single-qubit application
H q;
RX(pi/2) q;

// Multi-qubit application
CX control, target;
CCX q0, q1, q2;

// Broadcasting over registers
qubit[4] qreg;
H qreg;                       // Apply H to each qubit
RZ(pi/4) qreg;               // Apply RZ(π/4) to each

// Broadcasting with multiple registers
qubit[3] qreg1, qreg2;
CX qreg1, qreg2;             // CX qreg1[i], qreg2[i] for each i
```

**Broadcasting Rules:**
- Single qubit gate on register applies to each qubit
- Multi-qubit gate on equal-size registers applies element-wise
- Size mismatch causes compilation error
- Order matters for non-commuting operations

### 4.5 Quantum Instructions

#### 4.5.1 Reset

Initialize qubits to |0⟩ state:

```qasm
reset q;                      // Single qubit reset
reset qreg;                   // Reset entire register
reset $0;                     // Reset physical qubit
```

**Reset Semantics:**
- Non-unitary operation
- Discards existing quantum state
- Required before first use of qubits
- Can be mid-circuit (active reset)

#### 4.5.2 Measurement

Project qubits onto computational basis:

```qasm
bit b;
b = measure q;                // Measure single qubit

bit[4] results;
results = measure qreg;       // Measure register

// Legacy syntax (backward compatible)
measure q -> b;
measure qreg -> results;
```

**Measurement Properties:**
- Z-basis measurement (default)
- Destroys superposition
- Result is classical bit(s)
- Mid-circuit measurement allowed
- Can be conditioned upon

#### 4.5.3 Barrier

Prevent gate reordering optimizations:

```qasm
barrier q;                    // Single qubit barrier
barrier q1, q2, q3;          // Multi-qubit barrier
barrier;                      // Global barrier (all qubits)
```

**Barrier Uses:**
- Enforce temporal ordering
- Prevent commutation through point
- Debugging and visualization
- No effect on quantum state
- Compiler optimization boundary

**Reference**: [Gates - OpenQASM Documentation](https://openqasm.com/language/gates.html)

---

## 5. Classical Control

### 5.1 Conditional Statements

#### 5.1.1 If-Else Statements

Standard conditional branching based on boolean expressions:

```qasm
bit result;
result = measure q;

if (result == 1) {
    X q;
    RY(pi/4) q;
} else {
    Z q;
}

// Nested conditions
if (count > 5) {
    if (flag == true) {
        H q;
    }
} else if (count == 5) {
    Y q;
} else {
    // Default case
    reset q;
}
```

**Conditional Features:**
- Condition must evaluate to `bool`
- Braces required for multi-statement blocks
- `else if` chains supported
- Can contain quantum and classical operations
- Scope rules apply to variables declared within

### 5.2 Loops

#### 5.2.1 For Loops

Iterate over sets, ranges, or registers:

```qasm
// Iterate over explicit set
for int i in {0, 1, 2, 3} {
    RZ(i * pi/4) q;
}

// Range iteration [start:step:stop]
for uint[4] j in [0:2:8] {    // 0, 2, 4, 6, 8
    H qreg[j];
}

// Iterate over register elements
bit[8] results;
for bit b in results {
    if (b == 1) {
        count += 1;
    }
}

// Nested loops
for int i in [0:1:3] {
    for int j in [0:1:3] {
        CX qreg[i], qreg[j];
    }
}
```

**For Loop Specifications:**
- Loop variable scope limited to loop body
- Range syntax: `[start:step:stop]` (inclusive)
- Step can be negative for reverse iteration
- Set syntax: `{val1, val2, ...}`
- Break and continue supported

#### 5.2.2 While Loops

Conditional iteration:

```qasm
int attempts = 0;
bit success = 0;

while (success == 0 && attempts < 10) {
    reset q;
    H q;
    success = measure q;
    attempts += 1;
}

// Infinite loop with break
while (true) {
    RX(pi/8) q;
    bit m = measure q;
    if (m == 1) {
        break;
    }
}
```

**While Loop Properties:**
- Condition evaluated before each iteration
- Empty body allowed (but discouraged)
- Can modify condition variables within loop
- Break and continue supported

### 5.3 Loop Control

#### 5.3.1 Break Statement

Exit innermost loop immediately:

```qasm
for int i in [0:1:100] {
    H q;
    bit result = measure q;
    if (result == 1) {
        break;  // Exit for loop
    }
    reset q;
}
```

#### 5.3.2 Continue Statement

Skip to next iteration:

```qasm
for int i in [0:1:10] {
    if (i % 2 == 0) {
        continue;  // Skip even numbers
    }
    RZ(i * pi/10) q;
}
```

#### 5.3.3 End Statement

Terminate program execution:

```qasm
if (error_condition) {
    end;  // Stop entire program
}
```

### 5.4 Switch Statements

Multi-way branching on discrete values:

```qasm
int[3] state = 2;

switch (state) {
    case 0: {
        H q;
        break;
    }
    case 1: {
        X q;
        break;
    }
    case 2: {
        Y q;
        break;
    }
    default: {
        Z q;
    }
}
```

**Switch Properties:**
- Works with integer and boolean types
- Break required to prevent fall-through
- Default case optional but recommended
- Cases must be compile-time constants
- Braces recommended for case blocks

**Reference**: [Classical Instructions - OpenQASM Documentation](https://openqasm.com/language/classical.html)

---

## 6. Subroutines and Functions

### 6.1 Subroutine Definition

The subroutine will define zero or more parameters as input, consisting of both quantum and classical arguments. Quantum bits and classical values are passed to the subroutine by reference or name, while classical types are passed in by value.

```qasm
// Subroutine with return value
def parity(bit[4] x) -> bit {
    bit result = 0;
    for int i in [0:1:3] {
        result ^= x[i];
    }
    return result;
}

// Subroutine modifying qubits
def qft(qubit[4] qreg) {
    for int i in [0:1:3] {
        H qreg[i];
        for int j in [i+1:1:3] {
            ctrl @ RZ(pi/2^(j-i)) qreg[j], qreg[i];
        }
    }
    // Swap qubits for proper ordering
    SWAP qreg[0], qreg[3];
    SWAP qreg[1], qreg[2];
}

// Subroutine with multiple parameters
def controlled_rotation(qubit control, qubit target, 
                        angle[20] theta) -> bit {
    ctrl @ RY(theta) control, target;
    return measure control;
}
```

**Subroutine Characteristics:**
- Can contain both quantum and classical operations
- Quantum arguments passed by reference
- Classical arguments passed by value
- Can declare local variables
- Return statement required if return type specified
- Cannot be recursive (implementation-defined)
- Must be defined before use

### 6.2 External Functions

Link to external classical functions:

```qasm
// Declare external function
extern gaussian(float[64], float[64]) -> float[64];
extern random_bit() -> bit;
extern get_calibration_data(int[32]) -> float[64];

// Use external function
float[64] mean = 0.0;
float[64] stddev = 1.0;
float[64] sample = gaussian(mean, stddev);

bit random = random_bit();
```

**External Function Properties:**
- Implementation provided by runtime/compiler
- Classical computation only (no quantum operations)
- Useful for complex mathematical functions
- Platform-specific implementations
- May have side effects

### 6.3 Const Functions

Compile-time evaluated functions on constants:

```qasm
const float[64] PI_HALF = pi / 2;
const float[64] ANGLE = arcsin(0.5);
const int[8] BITS = popcount("11010110");  // Returns 5
const float[32] VAL = sin(pi/6);  // 0.5

// Built-in const functions
const float x = sqrt(2.0);
const float y = exp(1.0);
const float z = log(10.0);
const int n = floor(3.7);
const int m = ceil(3.2);
```

**Available Const Functions:**
- Trigonometric: `sin`, `cos`, `tan`, `arcsin`, `arccos`, `arctan`
- Exponential: `exp`, `log`, `log10`, `sqrt`, `pow`
- Rounding: `floor`, `ceil`, `round`
- Bit manipulation: `popcount`, `rotl`, `rotr`
- Complex: `real`, `imag`, `abs`, `arg`, `conj`

**Reference**: [Subroutines - OpenQASM Documentation](https://openqasm.com/language/subroutines.html)

---

## 7. Circuit Timing

### 7.1 Duration Types

Explicit representation of time intervals:

```qasm
duration t1 = 50ns;          // 50 nanoseconds
duration t2 = 2.5us;         // 2.5 microseconds
duration t3 = 100dt;         // 100 device time units
duration total = t1 + t2;    // Duration arithmetic

// Duration in box statements
duration box_time = 1ms;
```

**Duration Specifications:**
- Represented as 64-bit unsigned integer
- Base unit is device-dependent
- Arithmetic operations supported
- Can be compared with relational operators

### 7.2 Stretch Types

Variable timing placeholders resolved during scheduling:

```qasm
stretch s1;
stretch s2;

// Used in delay statements
delay[s1] q;
delay[s2] q;

// Compiler solves for stretch values
// to meet timing constraints
```

**Stretch Properties:**
- Symbolic duration variable
- Resolved by compiler/scheduler
- Enables flexible timing specifications
- Cannot be used in arithmetic

### 7.3 Delay Instruction

Insert explicit time delays:

```qasm
delay[100ns] q;              // Delay single qubit
delay[50dt] q1, q2;          // Delay multiple qubits
delay[stretch_var] $0;       // Delay with stretch

// Delay in context
H q;
delay[200ns] q;              // Wait 200ns
X q;
```

**Delay Semantics:**
- Blocks operations on specified qubits
- Maintains coherence (ideally)
- Multi-qubit delays synchronized
- Can use duration or stretch types

### 7.4 Box Statements

Group operations with unified timing:

```qasm
// Basic box
box {
    H q[0];
    CX q[0], q[1];
    H q[0];
}

// Box with duration
box [300ns] {
    RX(pi/4) q;
    RY(pi/4) q;
}

// Box with stretch
stretch s;
box [s] {
    // Operations to fit in stretch s
    X q;
    delay[10ns] q;
    Y q;
}
```

**Box Properties:**
- Contents treated as atomic unit
- Can specify total duration
- Enables local scheduling optimization
- Prevents external optimization across boundary

### 7.5 Barrier Instruction

Enforce ordering without explicit delays:

```qasm
H q[0];
H q[1];
barrier q[0], q[1];          // Synchronization point
CX q[0], q[1];

// Global barrier
barrier;                      // All qubits synchronized
```

**Reference**: [Circuit Timing - OpenQASM Documentation](https://openqasm.com/versions/3.0/language/delays.html)

---

## 8. Pulse-Level Control

### 8.1 Calibration Definitions

defcal rz(angle[20] theta) $0 { ... } defcal measure $0 -> bit { ... } defcal measure_iq q -> complex[float[32]] { ... }

Define pulse-level implementations of gates:

```qasm
// Define calibration for RZ gate on physical qubit 0
defcal rz(angle[20] theta) $0 {
    // Pulse sequence implementation
    shift_phase(drive_frame, theta);
}

// Define measurement calibration
defcal measure $0 -> bit {
    play(measure_frame, measurement_pulse);
    return capture_v2(acquire_frame, kernel);
}

// IQ measurement returning complex value
defcal measure_iq $0 -> complex[float[32]] {
    play(measure_frame, measurement_pulse);
    return capture_v3(acquire_frame);
}
```

**Defcal Properties:**
- Override default gate implementations
- Physical qubit arguments only
- Can return classical values
- Hardware-specific implementations
- Take precedence over gate definitions

### 8.2 OpenPulse Grammar

a frame can be initializated in either a cal or defcal block which means that the time with which it is initialized is the start time of the containing block

Specify pulse grammar for low-level control:

```qasm
defcalgrammar "openpulse";

// Declare external ports
extern port drive_port;
extern port measure_port;
extern port acquire_port;

// Define frames
frame drive_frame = newframe(drive_port, 5.0e9, 0.0);
frame measure_frame = newframe(measure_port, 6.5e9, 0.0);

// Define waveforms
waveform gaussian_pulse = gaussian(1.0, 160dt, 40dt);
waveform square_pulse = square(0.5, 100dt);

// Calibration block
cal {
    // Calibration-wide definitions
    duration pulse_length = 160dt;
    float[64] amplitude = 0.5;
}
```

### 8.3 Pulse Instructions

OpenPulse-specific instructions:

```qasm
defcal x $0 {
    // Play waveform on frame
    play(drive_frame, gaussian_pulse);
    
    // Shift frame phase
    shift_phase(drive_frame, pi/2);
    
    // Set frame phase
    set_phase(drive_frame, 0.0);
    
    // Shift frequency
    shift_frequency(drive_frame, 1e6);
    
    // Capture and kernel
    capture_v2(acquire_frame, kernel);
}

defcal cx $0, $1 {
    // Cross-resonance pulse
    play(cr_frame, cr_pulse);
    delay[200dt];
    
    // Echoed cross-resonance
    play(drive_frame_1, pi_pulse);
    play(cr_frame, cr_pulse);
    play(drive_frame_1, pi_pulse);
}
```

**Pulse Operations:**
- `play(frame, waveform)` - Output waveform
- `capture_v2(frame, kernel)` - Acquire with kernel
- `capture_v3(frame)` - Raw IQ acquisition
- `set_phase(frame, phase)` - Set absolute phase
- `shift_phase(frame, delta)` - Relative phase shift
- `set_frequency(frame, freq)` - Set frequency
- `shift_frequency(frame, delta)` - Relative frequency shift

**Reference**: [Pulse-level Descriptions - OpenQASM Documentation](https://openqasm.com/language/pulses.html)

---

## 9. Advanced Features

### 9.1 Input/Output Parameters

Declare circuit inputs and outputs:

```qasm
// Input parameters
input float[32] theta;
input int[8] num_iterations;
input qubit[4] input_state;

// Output parameters  
output bit[4] measurement_results;
output float[64] expectation_value;

// Use in circuit
for int i in [0:1:num_iterations-1] {
    RZ(theta * i) input_state[0];
}
measurement_results = measure input_state;
```

**I/O Parameter Rules:**
- Must be declared at global scope
- Input parameters are read-only
- Output parameters can be modified
- Enable parameterized circuit execution
- Default: all globals are outputs if not specified

### 9.2 Pragma Directives

pragma ibm.user alice account 12345678 // Assert that the QPU is healthy to run this circuit pragma ibm.max_temp qpu 0.4

Compiler and hardware hints:

```qasm
// Optimization hints
#pragma optimize_level 3
#pragma unroll_loops 4

// Hardware constraints
#pragma max_circuit_depth 1000
#pragma require_connectivity linear

// Vendor-specific
#pragma ibm.backend "ibmq_mumbai"
#pragma ionq.noise_model "native"

// Calibration selection
#pragma cal.use_robust_pulses true
```

### 9.3 Annotations

Metadata attachments to declarations:

```qasm
// Gate annotations
@deprecated("Use RZ instead")
@cost(2.5)
gate old_phase(angle[20] lambda) q {
    U(0, 0, lambda) q;
}

// Variable annotations
@critical_path
qubit[5] main_register;

@calibration_date("2024-01-15")
defcal x $0 { ... }

// Subroutine annotations
@pure
@inline
def helper_function(int[32] x) -> int[32] {
    return x * 2;
}
```

**Annotation Properties:**
- Begin with `@` symbol
- Can have parameters
- Tool-specific interpretation
- Do not affect program semantics
- Enable extended metadata

### 9.4 Scoping Rules

Variable visibility and lifetime:

```qasm
// Global scope
const int GLOBAL_CONST = 42;
qubit[4] qreg;               // Qubits always global

def function(int param) -> int {
    // Function scope
    int local = param + 1;
    
    // Block scope
    if (local > 5) {
        int block_var = 10;  // Only visible in if block
        local += block_var;
    }
    // block_var not accessible here
    
    return local;
}

// Loop variable scope
for int i in [0:1:5] {
    // i only exists here
    RZ(i * pi/6) qreg[0];
}
// i not accessible here
```

**Scope Rules:**
- Global: Outside all blocks
- Function: Within def/gate body
- Block: Within {} braces
- Loop: Loop variable in loop body only
- Inner scopes can shadow outer names
- Qubits cannot be declared in local scopes

### 9.5 Register Operations

Advanced register manipulations:

```qasm
// Concatenation
bit[4] low = "0011";
bit[4] high = "1100";
bit[8] combined = high ++ low;  // "11000011"

// Slicing with ranges
qubit[8] qreg;
let first_half = qreg[0:3];     // qreg[0] to qreg[3]
let every_other = qreg[0:2:7];  // qreg[0,2,4,6]
let reversed = qreg[7:0:-1];    // Reverse order

// List indexing
let selected = qreg[{1,3,5,7}]; // Non-contiguous

// Bit extraction from integers
int[16] value = 0x5A3C;
bit[4] nibble = value[4:7];     // Extract bits 4-7
bit msb = value[15];            // Most significant bit
```

### 9.6 Mathematical Constants and Functions

Built-in mathematical support:

```qasm
// Constants
const float[64] PI = pi;        // 3.14159...
const float[64] TAU = τ;        // 2π
const float[64] E = e;          // Euler's number
const float[64] EULER = ℇ;      // Same as e

// Compile-time math
const float[64] x = sin(pi/6);  // 0.5
const float[64] y = cos(0);     // 1.0
const float[64] z = exp(1);     // e
const float[64] w = log(e);     // 1.0
const float[64] v = sqrt(2);    // 1.414...

// Complex math
const complex[float[32]] j = 1im;
const complex[float[32]] z1 = exp(pi * j);  // -1
const float[32] magnitude = abs(3 + 4im);    // 5
const float[32] phase = arg(1 + 1im);        // π/4
```

**Reference**: [Language Features - OpenQASM Documentation](https://openqasm.com/language/)

---

## 10. References

### Official Documentation

1. **OpenQASM 3.0 Specification**: https://openqasm.com/versions/3.0/
2. **OpenQASM Live Specification**: https://openqasm.com/
3. **Language Reference**: https://openqasm.com/language/
4. **Grammar Specification**: https://openqasm.com/grammar/index.html
5. **OpenPulse Grammar**: https://openqasm.com/language/openpulse.html

### Technical Papers

6. **OpenQASM 3 Paper**: Cross, A. W., et al. "OpenQASM 3: A Broader and Deeper Quantum Assembly Language." ACM Transactions on Quantum Computing (2022). https://dl.acm.org/doi/10.1145/3505636

7. **Original OpenQASM Paper**: Cross, A. W., et al. "Open Quantum Assembly Language." arXiv:1707.03429 (2017).

### Implementation Resources

8. **GitHub Repository**: https://github.com/openqasm/openqasm
9. **Qiskit OpenQASM 3 Support**: https://qiskit.org/documentation/
10. **OpenQASM 3 Parser**: https://github.com/openqasm/openqasm/tree/main/source

### Additional Resources

11. **Types and Casting Reference**: https://openqasm.com/language/types.html
12. **Classical Instructions**: https://openqasm.com/language/classical.html
13. **Gates Documentation**: https://openqasm.com/language/gates.html
14. **Built-in Instructions**: https://openqasm.com/language/insts.html
15. **Circuit Timing**: https://openqasm.com/versions/3.0/language/delays.html
16. **Pulse Descriptions**: https://openqasm.com/language/pulses.html
17. **Subroutines**: https://openqasm.com/language/subroutines.html
18. **Directives**: https://openqasm.com/language/directives.html
19. **Scoping Rules**: https://openqasm.com/versions/3.0/language/scope.html
20. **Comments**: https://openqasm.com/versions/3.0/language/comments.html

---

## Appendices

### Appendix A: Complete Grammar

The complete ANTLR grammar specification for OpenQASM 3.0 is available at the official repository. The grammar defines all syntactic constructs and their relationships.

### Appendix B: Standard Library Gates

The standard library (`stdgates.inc`) provides implementations for commonly used quantum gates. All gates are built from the fundamental `U` and `gphase` operations.

### Appendix C: Error Codes

OpenQASM 3.0 implementations should report standardized error codes for common issues:
- **E001**: Syntax error
- **E002**: Undefined identifier
- **E003**: Type mismatch
- **E004**: Index out of bounds
- **E005**: Incompatible register sizes
- **E006**: Recursive definition
- **E007**: Missing return statement
- **E008**: Invalid cast
- **E009**: Circular dependency
- **E010**: Hardware constraint violation

### Appendix D: Version Compatibility

OpenQASM 3.0 maintains backward compatibility with OpenQASM 2.0 for core features while adding significant new capabilities. Migration guides are available in the official documentation.

---

**Document Version**: 1.0  
**Last Updated**: August 2025  
**Copyright**: OpenQASM Contributors  

This document provides a comprehensive reference for all OpenQASM 3.0 language constructs. For the most current information, always refer to the official OpenQASM documentation at https://openqasm.com/.
