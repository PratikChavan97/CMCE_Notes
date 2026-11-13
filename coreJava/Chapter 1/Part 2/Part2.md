# Module 01: Basics of C++

## Unit 1.2: Data Types, Variables, Type System & Storage Classes

---

### 1. Data Types in C++

C++ features a strictly typed, static type system. Types are categorized into three major groups:

```
                          Data Types in C++
                                 |
         +-----------------------+-----------------------+
         |                                               |
  Primitive (Built-in)        Derived Types           User-Defined Types
         |                           |                       |
   +-----+-----+               +-----+-----+           +-----+-----+
   |           |               |     |     |           |     |     |
 Integral   Floating-point   Array Pointer Reference  Class Struct Union
 (int, char, (float, double,                           |
  bool, etc)  long double)                            Enum (enum / enum class)
```

#### 1.1 Primitive (Fundamental) Data Types & Memory Footprints (x86-64 Architecture)

| Type                       | Keyword         | Typical Size (bytes) | Range / Precision                                            | Format Specifier (C-style) |
| :------------------------- | :-------------- | :------------------- | :----------------------------------------------------------- | :------------------------- |
| **Boolean**                | `bool`          | 1                    | `true` (1) or `false` (0)                                    | `%d`                       |
| **Character**              | `char`          | 1                    | Signed: $-128$ to $+127$<br>Unsigned: $0$ to $255$           | `%c`                       |
| **Wide Character**         | `wchar_t`       | 2 or 4               | 16-bit / 32-bit Unicode unit                                 | `%lc`                      |
| **Signed Integer**         | `int`           | 4                    | $-2^{31}$ to $2^{31}-1$ (approx $\pm 2.14 \times 10^9$)      | `%d` or `%i`               |
| **Short Integer**          | `short int`     | 2                    | $-32,768$ to $+32,767$                                       | `%hd`                      |
| **Long Integer**           | `long int`      | 4 (Win) / 8 (Linux)  | $\ge 4$ bytes (standard guarantee)                           | `%ld`                      |
| **Long Long Integer**      | `long long int` | 8                    | $-2^{63}$ to $2^{63}-1$                                      | `%lld`                     |
| **Single-Precision Float** | `float`         | 4                    | $\approx \pm 3.4 \times 10^{\pm 38}$ (7 decimal digits)      | `%f`                       |
| **Double-Precision Float** | `double`        | 8                    | $\approx \pm 1.7 \times 10^{\pm 308}$ (15-17 decimal digits) | `%lf`                      |
| **Extended Float**         | `long double`   | 8 / 12 / 16          | Platform-dependent (80-bit or 128-bit IEEE format)           | `%Lf`                      |
| **Void (Empty)**           | `void`          | 0 (incomplete)       | Represents absence of value or untyped memory (`void*`)      | N/A                        |

> **CMCE Trap:** C++ standards guarantee minimum sizes via relationships, not hard values:  
> $1 \le \text{sizeof(char)} \le \text{sizeof(short)} \le \text{sizeof(int)} \le \text{sizeof(long)} \le \text{sizeof(long long)}$.

---

### 2. Variables and Constants

#### 2.1 Variables: Definition, Declaration, and Scope

- **Declaration:** Informs the compiler of the variable's name and type without reserving memory:
  ```cpp
  extern int counter; // Declaration only (defined elsewhere)
  ```
- **Definition:** Allocates memory and optionally initializes the variable:
  ```cpp
  int counter = 0;    // Definition and copy initialization
  ```
- **Initialization Forms in Modern C++:**

  ```cpp
  int a = 10;     // Copy initialization
  int b(20);      // Direct initialization
  int c{30};      // Direct list/uniform initialization (C++11) - PREVENTS narrowing!
  int d{};        // Value-initialization (defaults to 0)

  // Narrowing conversion check:
  // int bad{3.14}; // COMPILE ERROR with list initialization!
  int ok = 3.14;   // Compiles with warning: truncated to 3
  ```

#### 2.2 Constants: `const` vs `#define` vs `constexpr`

| Dimension             | Macro (`#define`)                | `const` Qualifier                       | `constexpr` (C++11)                                  |
| :-------------------- | :------------------------------- | :-------------------------------------- | :--------------------------------------------------- |
| **Evaluation Stage**  | Preprocessor (text substitution) | Runtime or compile-time                 | Strictly compile-time                                |
| **Type Checking**     | None (unsafe)                    | Statically type-checked                 | Statically type-checked                              |
| **Memory Allocation** | Never allocates memory directly  | May allocate memory if address is taken | No runtime allocation if used as constant expression |
| **Scope Rules**       | Global (from definition down)    | Follows standard lexical block scoping  | Follows lexical block scoping                        |
| **Symbol Table**      | Stripped before compilation      | Present in symbol table (easy to debug) | Present in symbol table                              |

- **Pointer and `const` Matrix (Crucial Interview Topic):**

  ```cpp
  int x = 10, y = 20;

  const int* p1 = &x;        // Pointer to CONST data (data cannot change, pointer can)
  // *p1 = 15;               // ERROR
  p1 = &y;                   // OK

  int* const p2 = &x;        // CONST pointer to mutable data (pointer cannot reseat)
  *p2 = 15;                  // OK
  // p2 = &y;                // ERROR

  const int* const p3 = &x;  // CONST pointer to CONST data (neither can change)
  ```

---

### 3. Type Conversion (Implicit Conversion / Coercion)

Handled automatically by the compiler to prevent data loss or adhere to function signatures.

#### 3.1 Implicit Promotion & Arithmetic Conversion Hierarchy

When an expression contains mixed data types, values are promoted according to the rank:

$$\text{bool} \rightarrow \text{char} \rightarrow \text{short} \rightarrow \text{int} \rightarrow \text{unsigned int} \rightarrow \text{long} \rightarrow \text{unsigned long} \rightarrow \text{float} \rightarrow \text{double} \rightarrow \text{long double}$$

- **Integral Promotion:** Any integral type smaller than `int` (`char`, `signed char`, `unsigned char`, `short`, `bool`) is promoted to `int` before arithmetic operations.
- **Potential Pitfall (Signed/Unsigned Mismatch):**
  ```cpp
  unsigned int u = 10;
  int s = -20;
  if (s < u) {
      // CMCE TRAP: Will evaluate to FALSE!
      // 's' is converted to unsigned int, becoming a massive number (2^32 - 20)
  }
  ```

---

### 4. Type Casting (Explicit Conversion)

#### 4.1 C-Style Cast vs C++ Named Casts

- **C-Style Cast:** `(type)expression` or `type(expression)`
  - Dangerous: Attempts `const_cast`, `static_cast`, and `reinterpret_cast` indiscriminately without compiler checks.
- **C++ Named Casts:** Explicit, search-friendly, and enforce strict compile/run-time rules.

#### 4.2 The Four Modern C++ Casts

##### 1. `static_cast<T>(expr)`

- Compile-time checked conversion between compatible types (e.g., `float` to `int`, non-virtual pointer conversions within class hierarchy).
- Reverses implicit conversions safely.

  ```cpp
  double pi = 3.14159;
  int truncated = static_cast<int>(pi); // OK

  void* pVoid = &truncated;
  int* pInt = static_cast<int*>(pVoid); // OK
  ```

##### 2. `dynamic_cast<T>(expr)`

- Run-time checked cast used **only** for polymorphic hierarchies (at least one `virtual` function in base class).
- Uses **RTTI** (Run-Time Type Information).
- Behaviors:
  - Pointer cast failure: Returns `nullptr`.
  - Reference cast failure: Throws `std::bad_cast` exception.

  ```cpp
  class Base { virtual void foo() {} };
  class Derived : public Base { void bar() {} };

  Base* b = new Derived();
  Derived* d = dynamic_cast<Derived*>(b); // Valid downcast: returns valid pointer

  Base* plainBase = new Base();
  Derived* d2 = dynamic_cast<Derived*>(plainBase); // Invalid: d2 becomes nullptr
  ```

##### 3. `const_cast<T>(expr)`

- Used explicitly to add or strip away `const` or `volatile` qualifiers.
- **Warning:** Modifying an object that was originally defined as `const` causes **undefined behavior**.

  ```cpp
  void legacyFunction(char* str); // Old API that doesn't modify str, but lacks 'const'

  void caller(const char* readOnlyStr) {
      legacyFunction(const_cast<char*>(readOnlyStr)); // Safe ONLY if legacyFunction does not write to memory
  }
  ```

##### 4. `reinterpret_cast<T>(expr)`

- Lowest-level cast; purely reinterprets the bit pattern without altering binary representation.
- Used for pointer-to-pointer conversions of unrelated types, or pointer-to-integer conversions (`uintptr_t`).
  ```cpp
  long long address = 0x7ffee4b6c;
  int* ptr = reinterpret_cast<int*>(address); // Platform-dependent, dangerous
  ```

---

### 5. Storage Classes in C++

Storage classes define the **scope** (visibility), **linkage** (file accessibility), and **lifetime** (duration in memory) of variables and functions.

```
+------------------------------------------------------------------------------------+
| Storage Class | Storage Location | Lifetime       | Scope        | Default Value   |
+---------------+------------------+----------------+--------------+-----------------+
| auto          | Stack            | Function block | Local        | Garbage         |
| register      | CPU Register     | Function block | Local        | Garbage         |
| static        | Data Segment/BSS | Program life   | Local/File   | Zero (0 / NULL) |
| extern        | Data Segment/BSS | Program life   | Global (all) | Zero (0 / NULL) |
| mutable       | Heap / Stack     | Enclosing obj  | Class member | Member default  |
+------------------------------------------------------------------------------------+
```

#### Detailed Breakdown:

1. **`auto`:**
   - _C++98:_ Automatic local variable (default for block variables).
   - _C++11:_ Deduced type specifier (`auto x = 5;` deduces `int`). The storage class keyword meaning was removed.

2. **`register`:**
   - Requests compiler to store variable in a CPU register for fast access.
   - _Note:_ In C++, you can take the address of a `register` variable (unlike C), which forces the compiler to put it in RAM. Deprecated in C++11 and completely removed in C++17.

3. **`static`:**
   - **Inside a Function (Local static):** Initialized once the first time control reaches the declaration; survives across function invocations until the program terminates.
   - **At File Scope (Global static):** Limits linkage to the translation unit (internal linkage); prevents name collisions across files.
   - **Inside a Class:** Shared across all class instances (one copy per class).

4. **`extern`:**
   - Declares a variable or function with external linkage; informs the linker that the entity is defined in another source file.

5. **`mutable`:**
   - Applied to class data members to permit modification even inside `const` member functions or on `const` object instances.

---

### 6. High-Yield CMCE Traps & Interview Questions

- **Q1: What is the output of `sizeof('a')` in C vs C++?**
  - _Answer:_ In **C**, character literals have type `int`, so `sizeof('a') == sizeof(int)` (typically 4 bytes). In **C++**, character literals have type `char`, so `sizeof('a') == 1` byte.

- **Q2: What is the difference between `const` and `constexpr`?**
  - _Answer:_ `const` promises that a variable will not be modified after initialization, but the initializer can be a runtime value (e.g., `const int x = rand();`). `constexpr` guarantees that the value is known and evaluatable strictly at compile time (e.g., `constexpr int x = 5 * 10;`).

- **Q3: What happens when dynamic_cast fails on a reference?**
  - _Answer:_ Because references cannot be `nullptr` in C++, a failed `dynamic_cast` on a reference throws a `std::bad_cast` exception (located in `<typeinfo>`).

- **Q4: Why does uniform initialization (`{}`) prevent narrowing?**
  - _Answer:_ Uniform initialization checks value boundaries at compile time. Assigning a `float` to an `int` via `int x{3.14};` causes a compilation error, whereas `int x = 3.14;` silently truncates.

- **Q5: What is the storage location difference between uninitialized and initialized static variables?**
  - _Answer:_ Initialized static/global variables are placed in the **Data Segment (.data)**, while uninitialized static/global variables are stored in the **BSS (Block Started by Symbol) Segment (.bss)**, which the OS clears to all zeros at program load.
