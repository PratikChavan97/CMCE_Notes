# Module 02: Procedural Extensions & Modular Programming

## Unit 2.2: Advanced Function Concepts

### 1. Inline Functions

In standard function calls, the CPU performs context switching: saving register states, pushing parameters and the return address onto the call stack, jumping to the function address, executing the body, and returning back. This generates **runtime overhead**, especially for small, frequently called helper functions.

An **inline function** requests the compiler to expand the function body directly at each call site, eliminating call stack overhead.

```
Standard Call:
main() ----------> [Push args, jump to 0x004010] ----------> add() ----------> [Pop frame, jump back]

Inlined Call:
main() ----------> [int res = a + b; executed directly in place]
```

#### 1.1 Syntax & Macro Comparison

```cpp
inline int add(int a, int b) {
    return a + b;
}
```

| Dimension         | `inline` Functions                        | Preprocessor Macros (`#define`)                                                       |
| :---------------- | :---------------------------------------- | :------------------------------------------------------------------------------------ |
| **Parsing Stage** | Handled by **Compiler**                   | Handled by **Preprocessor**                                                           |
| **Type Checking** | Strict compile-time type validation       | Blind text substitution (No type safety)                                              |
| **Debugging**     | Traceable in debuggers; generates symbols | Difficult to debug; no symbols                                                        |
| **Side Effects**  | Safe (evaluates expressions once)         | Prone to double-evaluation hazards: `#define SQ(x) ((x)*(x))` $\rightarrow$ `SQ(++i)` |
| **Scope Rules**   | Honors class and namespace scoping        | Global text substitution; ignores scope                                               |

#### 1.2 Compiler Discretion & Rejection Rules

The `inline` keyword is merely a **request**, not an absolute command. The compiler silently ignores `inline` and generates a regular function call if:

1. The function body is large or complex.
2. The function contains **loops** (`for`, `while`, `do-while`), `switch`, or `goto`.
3. The function is **recursive**.
4. The function contains **static variables**.
5. The function address is taken (e.g., passed to a function pointer).

> **Class Member Inlining Rule:** Any member function **defined inside the class definition** is implicitly treated as `inline` by the compiler.

---

### 2. Recursive Functions

A function that calls itself, directly or indirectly, to solve a smaller instance of the same problem.

#### 2.1 Essential Components

Every recursive function requires:

1. **Base Case (Termination Condition):** The condition where recursion stops and returns a value without further calls.
2. **Recursive Step (Progressive Call):** Moving arguments toward the base case.

```cpp
unsigned long long factorial(int n) {
    if (n <= 1) return 1;          // Base case
    return n * factorial(n - 1);   // Recursive step
}
```

#### 2.2 Call Stack Growth & Stack Overflow

Each recursive call allocates a new stack frame (parameters, local variables, return address).

```
Stack Growth (n = 3):
+-------------------------------+
| factorial(1) -> returns 1     |  <- Base reached, unwinding begins
+-------------------------------+
| factorial(2) -> waits for (1) |
+-------------------------------+
| factorial(3) -> waits for (2) |
+-------------------------------+
| main()                        |
+-------------------------------+
```

- **Stack Overflow:** Occurs when recursion exceeds the available stack memory boundary due to:
  - Missing or unreachable base case.
  - Infinite recursion.
  - Excessively deep call trees.

#### 2.3 Tail Recursion & Optimization (TCO)

A function is **tail-recursive** if the recursive call is the **very last operation** performed before returning (i.e., no pending computations like `n * ...` remain).

```cpp
// Non-tail recursive: Needs multiplication AFTER return
int fact(int n) {
    if (n <= 1) return 1;
    return n * fact(n - 1);
}

// Tail recursive: Accumulator carries state; nothing pending
int factTail(int n, int accumulator = 1) {
    if (n <= 1) return accumulator;
    return factTail(n - 1, n * accumulator);
}
```

- **Tail Call Optimization (TCO):** Modern compilers can optimize tail recursion into a flat iterative loop, reusing the existing stack frame and achieving $O(1)$ stack space complexity.

---

### 3. Function Pointers

A function pointer holds the **memory address of executable code** in the code segment (`.text`), enabling dynamic function dispatch and callbacks.

#### 3.1 Syntax and Anatomy

```
ReturnType (*PointerName)(ParameterTypes);
```

```cpp
#include <iostream>

int add(int a, int b) { return a + b; }
int subtract(int a, int b) { return a - b; }

int main() {
    // 1. Declaration and Initialization
    int (*opPtr)(int, int) = &add; // '&' is optional: '= add' is also valid

    // 2. Invocation
    std::cout << opPtr(10, 5) << "\n";    // Implicit dereference (Standard style): 15
    std::cout << (*opPtr)(10, 5) << "\n";  // Explicit dereference: 15

    // 3. Reassignment
    opPtr = &subtract;
    std::cout << opPtr(10, 5) << "\n";    // 5

    return 0;
}
```

#### 3.2 Simplification using `typedef` and `using`

Function pointer declarations quickly become unwieldy. Simplify them using aliases:

```cpp
// C-Style typedef:
typedef int (*ArithmeticFunc)(int, int);

// Modern C++ style (Recommended):
using ArithmeticFunc = int (*)(int, int);

ArithmeticFunc f = add; // Clean and readable
```

#### 3.3 Callbacks and Higher-Order Functions

Function pointers allow passing functions as arguments to other functions (e.g., custom sorting or event handling):

```cpp
void executeOperation(int x, int y, ArithmeticFunc callback) {
    std::cout << "Computed Result: " << callback(x, y) << std::endl;
}
```

#### 3.4 Array of Function Pointers (Dispatch Table / State Machine)

```cpp
ArithmeticFunc operations[] = {add, subtract};
int result = operations[0](20, 10); // Dispatches to add -> 30
```

---

### 4. Scope Resolution Operator (`::`)

The scope resolution operator (`::`) qualifies identifiers and resolves ambiguities across different namespaces, classes, and scopes.

#### 4.1 Primary Use Cases

1. **Accessing Global Variables Hidden by Local Variables:**

   ```cpp
   int val = 100; // Global

   int main() {
       int val = 20; // Local
       std::cout << "Local: "  << val   << "\n"; // Prints 20
       std::cout << "Global: " << ::val << "\n"; // Prints 100 via global resolution
   }
   ```

2. **Defining Class Member Functions Outside the Class:**

   ```cpp
   class Engine {
   public:
       void start(); // Prototype
   };

   void Engine::start() { // Resolves start() to Engine scope
       // implementation
   }
   ```

3. **Disambiguating Namespaces:**

   ```cpp
   namespace DataLayer { int count = 5; }
   int c = DataLayer::count;
   ```

4. **Accessing Static Members of a Class:**

   ```cpp
   class Account {
   public:
       static int totalAccounts;
   };
   int Account::totalAccounts = 0; // Definition and access via ::
   ```

5. **Resolving Multiple Inheritance Ambiguity (The Diamond Problem):**
   ```cpp
   DerivedObj.BaseClassA::display();
   ```

---

### 5. Static and Global Variables

#### 5.1 Storage & Lifetime Breakdown

| Variable Type                | Storage Segment | Lifetime                | Default Value               | Scope                     | Linkage                                               |
| :--------------------------- | :-------------- | :---------------------- | :-------------------------- | :------------------------ | :---------------------------------------------------- |
| **Global Variable**          | Data / BSS      | Entire program duration | `0` / Zero-initialized      | Entire translation unit   | External (can share via `extern`)                     |
| **Static Global Variable**   | Data / BSS      | Entire program duration | `0`                         | Restricted to file        | **Internal Linkage** (Hidden from other `.cpp` files) |
| **Static Local Variable**    | Data / BSS      | Entire program duration | `0`                         | Enclosing block `{}` only | None                                                  |
| **Local Automatic Variable** | Stack           | Enclosing block `{}`    | **Garbage** (indeterminate) | Enclosing block `{}` only | None                                                  |

#### 5.2 Static Local Variables: Initialization Mechanics

- Initialized **only once**, the very first time control passes through their declaration.
- Retain their state between repeated function calls.

```cpp
#include <iostream>

void counter() {
    static int count = 0; // Initialized once at startup/first call
    count++;
    std::cout << count << " ";
}

int main() {
    counter(); // 1
    counter(); // 2
    counter(); // 3
    return 0;
}
```

- **Thread Safety (Magic Statics / C++11):** Since C++11, if multiple threads attempt to initialize a static local variable concurrently, the initialization is guaranteed to be **thread-safe** (used in Thread-Safe Meyer's Singleton pattern).

#### 5.3 `extern` Keyword for Global Variables

To access a global variable defined in another translation unit without creating duplicate symbols:

```cpp
// File1.cpp
int globalSharedCounter = 42; // Definition and allocation

// File2.cpp
extern int globalSharedCounter; // Declaration only (tells linker to find it elsewhere)

void printCount() {
    std::cout << globalSharedCounter;
}
```

---

### 6. High-Yield CMCE Traps & Interview Questions

- **Q1: Can an inline function be recursive?**
  - _Answer:_ An inline function _can_ be written recursively, but the compiler **will not inline it**. Because the compiler cannot determine the depth of recursion at compile time, it ignores the `inline` request and falls back to a standard function call with stack frames.

- **Q2: What is the output of the following code?**

  ```cpp
  #include <iostream>
  int x = 50;
  int main() {
      int x = 10;
      {
          int x = 5;
          std::cout << ::x << " ";
      }
      std::cout << x;
      return 0;
  }
  ```

  - _Output:_ `50 10`
  - _Explanation:_ `::x` always resolves to the global scope (`50`), completely bypassing all enclosing local blocks. The second print statement targets the local variable of `main()` (`10`).

- **Q3: What happens if two different `.cpp` files define a global variable with the same name without `static`?**
  - _Answer:_ The linker throws an **ODR (One Definition Rule) violation** or "duplicate symbol / multiple definition" error during phase 4 of compilation. Mark them `static` or put them inside anonymous namespaces to give them internal linkage.

- **Q4: How does a function pointer differ from a standard data pointer?**
  - _Answer:_ A data pointer holds the address of a variable residing in the Stack, Heap, or Data/BSS segment. A function pointer points to the entry address of executable machine instructions residing in the **Code Segment (`.text`)**. Function pointers cannot be modified via pointer arithmetic (`ptr++` is invalid).

- **Q5: Can `sizeof` be used on a function?**
  - _Answer:_ No. Applying `sizeof` directly to an expression of function type is illegal in standard ISO C++. However, applying `sizeof` to a **function pointer** is valid and returns the machine pointer width (4 bytes on 32-bit systems, 8 bytes on 64-bit systems).
