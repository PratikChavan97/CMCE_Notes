# Module 02: Procedural Extensions & Modular Programming

## Unit 2.1: Functions

### 1. Function Declaration, Definition, and Execution Flow

A function is an independent, self-contained block of statements designed to perform a specific task, promote code reusability, and modularize programs.

```
       [Caller: main()]                            [Callee: add()]
+-----------------------------+               +-----------------------------+
| int res = add(5, 10);       | ------------> | int add(int a, int b) {     |
|                             |               |     return a + b;           |
| // Continues execution      | <------------ | }                           |
+-----------------------------+  (Returns 15) +-----------------------------+
```

#### 1.1 Declaration (Prototype) vs Definition

- **Function Declaration (Signature / Prototype):**
  - Informs the compiler about the function's name, return type, and parameter list (types and count).
  - Allocates **no memory**.
  - Allows a function to be called before its full body is compiled.
  - Syntax:
    ```cpp
    ReturnType FunctionName(ParameterType1, ParameterType2, ...);
    ```

- **Function Definition:**
  - Contains the actual executable body (implementation) of the function.
  - Causes the compiler to generate executable machine instructions and allocate code-segment memory.
  - Syntax:
    ```cpp
    ReturnType FunctionName(ParamType1 param1, ParamType2 param2) {
        // Function Body
        return value; // (if ReturnType != void)
    }
    ```

#### 1.2 Function Call Stack & Activation Record

When a function is called at runtime:

1. The caller's execution state (registers, next Instruction Pointer `EIP`/`RIP`) is pushed onto the **Call Stack**.
2. An **Activation Record (Stack Frame)** is allocated for the callee containing:
   - Function parameters (actual arguments copied or referenced).
   - Return address (where to resume in caller).
   - Local automatic variables.
   - Saved frame pointer (`EBP`/`RBP`).
3. Upon hitting `return` or the closing brace `}`, the stack frame is popped (deallocated), invoking automatic destructors for local stack objects in reverse order of construction.

---

### 2. Function Prototypes

In C++, function prototypes are **strictly mandatory**. Unlike traditional C (C89), which allowed implicit function declarations defaulting to `int`, C++ rejects calls to undeclared functions.

```cpp
#include <iostream>

// Valid prototype declarations:
void display(int);            // Parameter name omitted (legal in prototype)
void display(int value);      // Parameter name included (recommended for clarity)

int main() {
    display(100);             // Legal: Compiler validates arguments via prototype
    return 0;
}

void display(int value) {     // Definition
    std::cout << "Value: " << value << "\n";
}
```

- **Scope of a Prototype:**
  - If declared outside all functions (global scope), it is visible to all subsequent functions in that translation unit.
  - If declared inside a function block (local scope), it is only callable within that specific block.

---

### 3. Call by Value vs Call by Reference

Parameter passing in C++ determines how arguments are transmitted from the caller to the callee's activation frame.

```
Call by Value:                      Call by Reference:
Caller Frame       Callee Frame     Caller Frame       Callee Frame
+-----------+      +-----------+    +-----------+      +-----------+
| x = 10    | ---> | a = 10    |    | x = 10    | <--- | ref_x     | (ref_x aliases
+-----------+ (Copy+-----------+    +-----------+      +-----------+  address of x)
```

#### 3.1 Call by Value

- The callee receives a **copy** of the caller's actual arguments.
- Changes made inside the function affect only the local copy; caller variables remain unmodified.
- **Overhead:** For primitive types (`int`, `char`), copying is fast. For user-defined objects (`class`, `struct`, `std::vector`), it invokes copy constructors and destructors, causing memory and runtime overhead.

```cpp
void modifyVal(int a) {
    a = 99; // Affects only local 'a'
}
```

#### 3.2 Call by Reference (C++ Reference & Pointer Styles)

C++ provides two ways to achieve reference semantics:

1. **C++ True Reference (`Type&`):**
   - An alias for the caller's variable (shares the exact same memory address).
   - Syntactically clean: No dereferencing (`*`) required.
   - Cannot be `nullptr`.
2. **Address / Pointer Pass (`Type*`):**
   - Passes the memory address by value.
   - Requires dereferencing (`*ptr`) inside the function.
   - Can accept `nullptr`.

#### 3.3 Comprehensive Comparison

| Metric                | Call by Value                   | Call by Reference (`T&`)                     | Call by Address (`T*`)                           |
| :-------------------- | :------------------------------ | :------------------------------------------- | :----------------------------------------------- |
| **Syntax**            | `void fn(int x)`                | `void fn(int& x)`                            | `void fn(int* x)`                                |
| **Invocation**        | `fn(val);`                      | `fn(val);`                                   | `fn(&val);`                                      |
| **Memory Allocated?** | Yes (New duplicate copy)        | No (Internal pointer, aliases caller's slot) | Yes (Allocates pointer variable to hold address) |
| **Modifies Caller?**  | No                              | **Yes**                                      | **Yes**                                          |
| **Null Safety**       | N/A                             | **Guaranteed non-null**                      | Must check `if (ptr != nullptr)`                 |
| **Object Overhead**   | High (invokes copy constructor) | Zero (passes alias)                          | Zero (passes machine address)                    |

#### 3.4 Passing by `const` Reference (`const T&`) — The Industry Standard

For non-primitive objects, pass by `const` reference to get zero-copy performance without risking accidental mutations:

```cpp
void printVector(const std::vector<int>& vec) {
    // vec[0] = 10; // COMPILE ERROR: Read-only reference
    std::cout << vec.size() << "\n"; // Zero copying overhead!
}
```

---

### 4. Function Overloading

Function overloading is a form of **Compile-Time (Static) Polymorphism** where two or more functions in the same scope share the same name but have different parameter lists.

#### 4.1 Conditions for Legal Overloading

Functions **MUST** differ in:

1. **Number of parameters**, OR
2. **Data types of parameters**, OR
3. **Sequence/Order of parameter types**.

```cpp
void compute(int a, double b);  // Base
void compute(int a);            // Valid: Different parameter count
void compute(double a, int b);  // Valid: Different parameter order
void compute(double a, double b);// Valid: Different types
```

#### 4.2 What CANNOT Overload a Function:

- **Return Type Alone:**

  ```cpp
  int calculate(int x);
  double calculate(int x); // COMPILE ERROR: Cannot overload based solely on return type
  ```

  _Reason:_ The compiler cannot infer the intended overload if the function is called without capturing the return value (e.g., `calculate(5);`).

- **Parameter Passed by Value vs `const` by Value:**
  ```cpp
  void process(int x);
  void process(const int x); // COMPILE ERROR: Redefinition (top-level const ignored)
  ```
  _(Note: Overloading on reference to const `void process(int&)` vs `void process(const int&)` IS valid)._

#### 4.3 Name Mangling (Decoration)

Compilers assign unique linkage symbols to overloaded functions by encoding the parameter types into the binary symbol:

```
Source Code:            GCC/Clang Mangled Symbol:
void print(int)   ----> _Z5printi
void print(double)----> _Z5printd
void print(int, char)-> _Z5printic
```

To disable name mangling (e.g., when linking C libraries into C++):

```cpp
extern "C" {
    void c_function(int a); // Preserves exact symbol name '_c_function'
}
```

---

### 5. Default Arguments

Default arguments allow functions to be called without specifying all arguments. Missing arguments are filled automatically by the compiler at the call site.

#### 5.1 Syntax and Right-to-Left Rule

Default arguments **must be assigned from right to left**. Once a parameter has a default argument, all parameters to its right must also have defaults:

```cpp
// LEGAL:
void display(int a, int b = 10, int c = 20);
void logMsg(int code, const char* msg = "Success");

// ILLEGAL (Compile Error):
void badFunc(int a = 1, int b);          // ERROR: Leftmost has default, rightmost does not
void invalid(int a, int b = 2, int c);   // ERROR: Gap in default arguments
```

#### 5.2 Declaration vs Definition Constraint

Default arguments **must be specified only once**, typically in the **function prototype (declaration)**, NOT in the function definition:

```cpp
// In header (.h) or prototype:
void calculate(int x, int y = 100);

// In source (.cpp) or definition:
void calculate(int x, int y) { // DO NOT write 'int y = 100' here!
    std::cout << x + y << std::endl;
}
```

_If a default argument is repeated in both declaration and definition, the compiler throws a "redefinition of default argument" error._

---

### 6. Overloading vs Default Arguments: Ambiguity Pitfalls

Combining function overloading with default arguments frequently causes **ambiguity errors** during compilation:

```cpp
void show(int a);
void show(int a, int b = 0);

int main() {
    show(10, 20); // OK: Unambiguously calls show(int, int)
    show(10);     // COMPILE ERROR: Ambiguous call!
                  // Matches show(int) directly, AND
                  // matches show(int, int=0) via default argument.
    return 0;
}
```

#### Ambiguity with Type Promotions:

```cpp
void evaluate(float f);
void evaluate(double d);

int main() {
    evaluate(5.5); // Calls evaluate(double); 5.5 literal is double by default
    evaluate(5);   // COMPILE ERROR: Ambiguous call!
                   // int '5' can be promoted to float OR double with equal ranking.
    return 0;
}
```

---

### 7. High-Yield CMCE Traps & Interview Questions

- **Q1: Can a function return a reference to a local variable?**
  - _Answer:_ **Never.** Local variables are allocated on the function's stack frame and destroyed when the function returns. Returning a reference (`int&`) to a local variable results in a dangling reference and causes undefined behavior when accessed:
    ```cpp
    int& getNumber() {
        int x = 10;
        return x; // CRITICAL BUG: Warning/Undefined behavior
    }
    ```

- **Q2: Does function overloading happen at runtime or compile-time?**
  - _Answer:_ At **compile time** (Static Binding / Early Binding). Overload resolution inspects the number and types of arguments during compilation and directly hard-codes the call to the mangled symbol.

- **Q3: What is the output of this code?**

  ```cpp
  #include <iostream>
  void test(int x) { std::cout << "int "; }
  void test(char x) { std::cout << "char "; }
  int main() {
      test('A');
      test(65);
      test(true);
      return 0;
  }
  ```

  - _Output:_ `char int int `
  - _Explanation:_ `'A'` matches `char` directly. `65` matches `int` directly. `true` (`bool`) undergoes integral promotion to `int`, matching `test(int)`.

- **Q4: How does `inline` relate to functions?**
  - _Answer:_ The `inline` keyword suggests that the compiler replace the function call directly with the function's machine code body, eliminating call-stack overhead (pushing registers, jumping, popping frames). The compiler may ignore this suggestion if the function contains loops, static variables, or recursion.

- **Q5: What is the type of a function pointer?**
  - _Answer:_ A function pointer's type matches its exact signature (return type and parameter list).
    ```cpp
    int add(int a, int b) { return a + b; }
    int (*fptr)(int, int) = &add; // Pointer to function
    int result = fptr(3, 4);      // Calls add(3, 4)
    ```
