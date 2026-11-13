# Module 01: Basics of C++

## Unit 1.1: Introduction to C++

---

### 1. Introduction to C++

- **Origin & History:**
  - Developed by **Bjarne Stroustrup** at **Bell Laboratories** (AT&T) in **1979**.
  - Initially named **"C with Classes"**, renamed to **C++** in **1983** (coined by Rick Mascitti; `++` is the C increment operator, signifying an enhanced version of C).
  - Standardized by ISO/IEC: C++98, C++03, C++11 (Modern C++), C++14, C++17, C++20, and C++23.

- **Core Design Philosophy:**
  - **"Zero-Overhead Principle":** What you don't use, you don't pay for; what you do use, you couldn't hand-code any better.
  - Direct hardware mapping with low-level memory manipulation (carried over from C).
  - Expressive high-level abstractions (Object-Oriented, Generic, and Functional).
  - Multi-paradigm: Does not force pure OOP like Java or Smalltalk.

---

### 2. C vs C++: Comprehensive Comparison

| Feature / Dimension            | C Language                                                     | C++ Language                                                                |
| :----------------------------- | :------------------------------------------------------------- | :-------------------------------------------------------------------------- |
| **Programming Paradigm**       | Procedural / Imperative                                        | Multi-paradigm (Procedural, Object-Oriented, Generic, Functional)           |
| **Data Focus**                 | Focuses on **functions & algorithms**; data moves freely       | Focuses on **data security & encapsulation**; binds data to functions       |
| **Design Approach**            | **Top-Down** design approach                                   | **Bottom-Up** design approach                                               |
| **Encapsulation & Access**     | No access specifiers (`struct` members are public by default)  | `public`, `private`, `protected` access specifiers                          |
| **Namespaces**                 | Absent (global name collision risk)                            | Supported (`namespace`) to avoid symbol collisions                          |
| **Polymorphism & Overloading** | Not supported (cannot have functions with the same name)       | Supported (Function Overloading, Operator Overloading, Virtual Functions)   |
| **Dynamic Memory**             | `malloc()`, `calloc()`, `realloc()`, `free()` via `<stdlib.h>` | `new` and `delete` operators (allocate + construct / destruct + deallocate) |
| **Reference Variables**        | Pointers only; no native reference type                        | First-class reference variables supported (`int& ref = a;`)                 |
| **Exception Handling**         | Error codes, `errno`, `setjmp()` / `longjmp()`                 | Formal `try`, `catch`, `throw` blocks with RAII stack unwinding             |
| **Standard Library**           | C Standard Library (POSIX / C Lib)                             | Standard Template Library (STL) + C Standard Library                        |
| **Default Input / Output**     | `printf()` / `scanf()` via `<stdio.h>`                         | `std::cout` / `std::cin` streams via `<iostream>`                           |
| **File Extension**             | `.c`                                                           | `.cpp`, `.cc`, `.cxx`                                                       |

---

### 3. Key Features of C++

1. **Object-Oriented Programming (OOP):**
   - **Encapsulation:** Bundling data and functions into a single unit (`class`) and restricting direct access via visibility specifiers.
   - **Data Abstraction:** Exposing essential interfaces while hiding internal implementation details.
   - **Inheritance:** Reusability mechanism allowing a class to acquire properties and behaviors of another class.
   - **Polymorphism:** Ability of a message or function call to behave differently based on the context (Compile-time via overloading/templates, Run-time via virtual functions).

2. **Generic Programming (Templates):**
   - Write algorithms and data structures independently of any specific data type (e.g., `template <typename T>`, STL `std::vector<T>`).

3. **Direct Memory Control & Low-Level Manipulation:**
   - Raw pointers, explicit pointer arithmetic, bitwise manipulations, custom memory allocators.

4. **Stronger Static Type Checking:**
   - C++ enforces stricter type rules than C (e.g., `void*` cannot be implicitly converted to typed pointers like `int*` without a cast).

5. **Namespaces:**
   - Group related classes, functions, and variables under distinct logical identifiers to prevent identifier collisions in large codebases.

6. **Extensible Operator Overloading:**
   - Allows user-defined types to use built-in operator syntax (`+`, `-`, `<<`, `[]`).

---

### 4. The C++ Compilation & Build Process

A C++ source file progresses through four distinct phases to become an executable binary.

```
+--------------------+
|  Source: main.cpp  |
+--------------------+
          |
          v   [1. Preprocessor]  (g++ -E)
+--------------------+  --> Handles #include, #define, #ifdef
| Expanded Source    |  --> Strips comments
|     (main.i)       |
+--------------------+
          |
          v   [2. Compiler]      (g++ -S)
+--------------------+  --> Syntax/Semantic analysis, optimization
|  Assembly Code     |  --> Produces architecture-specific instructions
|     (main.s)       |
+--------------------+
          |
          v   [3. Assembler]     (g++ -c)
+--------------------+  --> Converts assembly mnemonics to machine code
| Relocatable Object |  --> Symbol table created (offsets unresolved)
|     (main.o)       |
+--------------------+
          |
          +<--- Other .o files + Static/Dynamic Libraries (libc++, libstdc++)
          |
          v   [4. Linker]        (g++ / ld)
+--------------------+  --> Resolves external symbols, function addresses
| Executable Binary  |  --> Formats file (ELF on Linux, PE on Windows)
|    (a.out / .exe)  |
+--------------------+
```

#### Detailed Breakdown of Each Stage:

1. **Preprocessing (`cpp` / `g++ -E`):**
   - Evaluates preprocessor directives (lines starting with `#`).
   - `#include <header>`: Inserts header contents directly into the file.
   - `#define MACRO val`: Performs text macro substitution.
   - `#ifdef` / `#ifndef` / `#endif`: Filters conditional compilation blocks.
   - Strips source code comments (`//` and `/* ... */`).
   - Output file: `.i` (preprocessed source).

2. **Compilation proper (`cc1plus` / `g++ -S`):**
   - Parses the preprocessed code, checks grammar, type correctness, and syntax.
   - Performs code optimizations.
   - Performs **Name Mangling** (decorating function names with parameter types to enable function overloading).
   - Generates architecture-dependent assembly language.
   - Output file: `.s` (assembly code).

3. **Assembly (`as` / `g++ -c`):**
   - Translates assembly instructions into raw binary machine code.
   - Produces relocatable machine code with unresolved symbol addresses.
   - Output file: `.o` (Linux) or `.obj` (Windows).

4. **Linking (`ld` / `g++`):**
   - Merges multiple `.o` / `.obj` files into a unified executable.
   - Resolves external symbols (e.g., binds calls to `std::cout` to the compiled implementation in `libstdc++`).
   - Configures the program entry point (default: `main`).
   - Output: Executable file (`a.out` on Linux or `.exe` on Windows).

---

### 5. Structure of a C++ Program

#### Annotated Code Example

```cpp
// 1. Preprocessor Directive
// Tells the preprocessor to include standard input/output stream declarations
#include <iostream>

// 2. Namespace Declaration
// Brings all symbols of the std namespace into current file scope
using namespace std;

// 3. Global Declarations / Classes / Functions (Optional)
const double PI = 3.14159; // Global constant

void printGreeting();       // Function prototype/declaration

class Calculator {          // Class definition
private:
    double factor;          // Member variable (encapsulated)
public:
    Calculator(double f) : factor(f) {} // Constructor
    double multiply(double val) {       // Member function
        return val * factor;
    }
};

// 4. Main Entry Point Function
// Returns an integer status code to the operating system
int main(int argc, char* argv[]) {
    // Local variable declaration and initialization
    double radius = 5.0;

    // Function call
    printGreeting();

    // Object instantiation on stack
    Calculator calc(2.0);

    // Stream Output (<< is the insertion operator)
    cout << "Area factor output: " << calc.multiply(radius) << endl;

    // 5. Exit Status
    // 0 indicates successful execution to OS; non-zero indicates error
    return 0;
}

// 6. Function Definition
void printGreeting() {
    cout << "=== Execution Started ===" << endl;
}
```

#### Anatomical Components:

1. **Header Files (`#include <iostream>`):**
   - Supplies declarations for stream objects (`cin`, `cout`, `cerr`). No `.h` suffix is used for standard modern C++ headers.
2. **Namespaces (`using namespace std;`):**
   - All standard library entities are scoped within the `std` namespace.
   - Alternatives: `std::cout` (Explicit qualification, preferred in header files to avoid namespace pollution).
3. **The `main()` Function:**
   - Mandatory entry point for every standalone C++ program.
   - Return type **must** be `int` (returning `void main()` is non-standard and rejected by compliant compilers like modern GCC/Clang).
   - Implicit return: If control reaches the end of `main()` without a `return` statement, the compiler implicitly inserts `return 0;`.
4. **I/O Operations:**
   - `cout`: Instance of `std::ostream` representing standard output (buffered).
   - `cin`: Instance of `std::istream` representing standard input.
   - `<<`: Stream insertion operator.
   - `>>`: Stream extraction operator.
   - `endl`: Inserts a newline character `'\n'` and **flushes** the output buffer.

---

### 6. High-Yield CMCE Traps & Interview Questions

- **Q1: Why is `void main()` illegal in standard C++?**
  - _Answer:_ According to ISO C++ standard (ISO/IEC 14882), `main()` must return `int` to report termination status back to the host operating system/parent process.
- **Q2: What is "Name Mangling" and why is it necessary?**
  - _Answer:_ C++ supports function overloading. Because the linker only works with unique symbol names, the compiler mangles (encodes) function names with information about their parameter types (e.g., `_Z3addii` for `add(int, int)`).
- **Q3: What happens if you define a function in C and call it in C++?**
  - _Answer:_ The C++ compiler mangles the name by default, leading to an "Undefined Reference" linker error. To prevent this, declare the function using `extern "C"` to force standard C linkage without name mangling.
- **Q4: What is the difference between `\n` and `std::endl`?**
  - _Answer:_ `\n` outputs an escape sequence for a newline. `std::endl` outputs a newline **and** explicitly flushes the stream buffer (`flush()`), which incurs an I/O performance penalty if executed repeatedly inside loops.
- **Q5: Can dynamic memory allocated with `malloc()` be freed with `delete`?**
  - _Answer:_ No. Mixing `malloc/delete` or `new/free` leads to undefined behavior. `new` invokes constructors and `delete` invokes destructors; `malloc` and `free` only manage raw bytes.
