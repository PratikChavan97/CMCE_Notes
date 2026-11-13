# Module 05: Exception Handling & Advanced Robustness

## Unit 5.1: Exception Handling Fundamentals

### 1. Introduction to Exception Handling

**Exception handling** is a structured, language-level mechanism designed to handle runtime anomalies (such as divide-by-zero, out-of-memory, file not found, or network disconnects) by transferring program control from the point where an error occurs to a dedicated handling block.

#### 1.1 Limitations of Traditional C-Style Error Handling:

Before formal exception handling, developers used return codes, global flags, and pointer parameters:

- **Global Error Flags (`errno`):** Easily overwritten by consecutive system calls; frequently ignored by developers.
- **Special Return Codes (`-1`, `NULL`):** Conflates error signaling with actual returned calculation data; forces caller functions to check status after every call.
- **Code Pollution:** Business logic becomes buried under repetitive `if (err != 0)` checks.
- **Inability to Signal Constructor Failures:** Constructors have no return type and cannot return error codes if object initialization fails.

#### 1.2 The C++ Exception Handling Architecture

C++ separates **error detection** (where the issue arises) from **error handling** (where the issue can be meaningfully resolved) using three keywords:

1. `try`: Encloses code that might trigger an exceptional condition.
2. `throw`: Halts normal execution and transmits an exception object up the call stack.
3. `catch`: Catches and resolves the exception matching its parameter type.

```
Caller Scope                                Callee Scope
+------------------------------------+      +------------------------------------+
| try {                              |      | double divide(double a, double b)  |
|    double res = divide(10.0, 0.0); | ---> | {                                  |
|    cout << res;                    |      |    if (b == 0.0)                   |
| }                                  |      |        throw "Divide by Zero!"; --+
| catch (const char* msg) { <--------+------+-----------------------------------+
|    cerr << "Error: " << msg;       |  (Stack unwinds, skips remaining try statements,
| }                                  |   and jumps directly to matching catch handler)
+------------------------------------+
```

---

### 2. Errors vs Exceptions

Understanding the architectural distinction between structural system errors and software exceptions is a recurring exam topic:

| Dimension           | Error                                                                            | Exception                                                                  |
| :------------------ | :------------------------------------------------------------------------------- | :------------------------------------------------------------------------- |
| **Origin**          | Typically caused by hardware failures, OS resource depletion, or fatal bugs.     | Caused by unusual but foreseeable runtime events inside application code.  |
| **Examples**        | `Stack Overflow`, `Segmentation Fault` (Bus Error), CPU Power Failure.           | `std::bad_alloc`, array out-of-bounds, invalid user inputs, missing files. |
| **Catchability**    | **Cannot be caught** or recovered from using standard `try-catch` blocks.        | **Can be caught**, managed, and gracefully recovered from via `catch`.     |
| **Handling Method** | Operating system issues a signal (`SIGSEGV`, `SIGKILL`) terminating the process. | Structured language keywords: `try`, `throw`, `catch`.                     |
| **Recovery**        | Termination is generally required; programmatic recovery is rarely possible.     | The system can log, clean up resources, prompt the user, or safely resume. |

---

### 3. The `try` Block

A `try` block serves as an execution context monitor. Any statement placed inside this block is tracked for exceptions thrown directly within it or deeply inside downstream function calls made from it.

#### 3.1 Syntax and Structural Rules:

```cpp
try {
    // Monitored operations
    performDatabaseWrite();
    allocateMemory();
}
// A try block CANNOT stand alone!
// It must be followed IMMEDIATELY by at least one catch block.
catch (/* type */) {
    // Handler
}
```

- **No Code Between `try` and `catch`:** Placing any statements between the closing brace `}` of a `try` block and the opening `catch` keyword causes a compile-time syntax error.
- **Nesting:** `try` blocks can be nested hierarchically inside other `try` blocks or inside functions invoked from a `try` block.

---

### 4. The `throw` Statement

The `throw` keyword signals that an exceptional event has occurred. It halts execution of the current function immediately and initiates **Stack Unwinding**.

#### 4.1 What Can Be Thrown?

In C++, **any data type** can be thrown:

1. **Primitive types:** `throw 404;`, `throw -1;`, `throw 'E';`
2. **Strings & Literals:** `throw "Disk read failure";`, `throw std::string("Network timeout");`
3. **Pointers:** `throw new std::runtime_error("Heap error");` _(Generally discouraged due to ownership leaks)_
4. **Standard Exception Objects:** `throw std::out_of_range("Index too large");`
5. **Custom Classes:** `throw InsufficientFundsException(requested, balance);`

```cpp
double calculateSquareRoot(double val) {
    if (val < 0.0) {
        // Constructs and throws an exception object by value
        throw std::invalid_argument("Negative inputs not permitted!");
    }
    return std::sqrt(val);
}
```

#### 4.2 Lifecycle of a Thrown Object

- When an exception is thrown, the compiler creates a **temporary copy** of the thrown object in a dedicated, implementation-defined runtime exception area (outside standard stack frames).
- This temporary object persists until the corresponding `catch` block finishes executing.

---

### 5. The `catch` Block (Exception Handlers)

The `catch` block handles an exception thrown from the associated `try` block.

#### 5.1 Syntax and Pass-By Mechanics

```cpp
catch (ParameterType param) {
    // Exception handling code
}
```

Exceptions can be caught in three ways:

1. **Catch by Value:** `catch (std::exception e)`
   - _Problem:_ Causes **Object Slicing** if polymorphic exceptions are thrown, and incurs unnecessary copy overhead.
2. **Catch by Pointer:** `catch (std::exception* e)`
   - _Problem:_ Forces the caller to manage memory ownership (who calls `delete`?).
3. **Catch by Const Reference (BEST PRACTICE):** `catch (const std::exception& e)`
   - _Advantages:_ **Zero-copy overhead**, preserves polymorphic behavior without slicing, prevents accidental mutation of the exception object.

---

### 6. Multiple Catch Blocks & Matching Rules

A single `try` block can be followed by multiple `catch` blocks to handle different types of exceptions separately:

```cpp
#include <iostream>
#include <stdexcept>
#include <string>

void executeTask(int code) {
    if (code == 1) throw 404;                             // int
    if (code == 2) throw 3.14159;                         // double
    if (code == 3) throw "Hard drive disconnected!";      // const char*
    if (code == 4) throw std::out_of_range("Range error");// Derived from std::exception
}

int main() {
    try {
        executeTask(4);
    }
    catch (int errCode) {
        std::cerr << "Integer Error Code: " << errCode << "\n";
    }
    catch (double errVal) {
        std::cerr << "Floating-point Exception: " << errVal << "\n";
    }
    catch (const char* str) {
        std::cerr << "String Literal Exception: " << str << "\n";
    }
    catch (const std::exception& ex) {
        std::cerr << "Standard Exception: " << ex.what() << "\n";
    }
    catch (...) { // Catch-all handler
        std::cerr << "Unknown, unexpected exception caught!\n";
    }
    return 0;
}
```

#### 6.1 Order of Catch Blocks (Top-to-Bottom Linear Search)

The runtime searches `catch` handlers sequentially from **top to bottom**. The **first handler whose type matches the thrown object** executes. All subsequent catch handlers are skipped.

#### 6.2 The Inheritance Catch-Order Trap (CMCE High-Yield Question)

When catching exceptions within an inheritance hierarchy (e.g., `std::runtime_error` derived from `std::exception`):

- **Rule:** Always place **Derived class catch handlers BEFORE Base class handlers**.

```cpp
// INCORRECT (Base catches derived; derived is shadowed):
try {
    throw std::runtime_error("Disk full");
}
catch (const std::exception& e) { // BASE: Intercepts everything!
    std::cout << "Base handler: " << e.what() << "\n";
}
catch (const std::runtime_error& e) { // DERIVED: DEAD CODE / UNREACHABLE!
    std::cout << "Derived handler\n"; // Compiler warning / Logic flaw
}

// CORRECT:
try {
    throw std::runtime_error("Disk full");
}
catch (const std::runtime_error& e) { // Derived handler evaluated first
    std::cout << "Specific handler: " << e.what() << "\n";
}
catch (const std::exception& e) {     // Fallback for other standard exceptions
    std::cout << "Generic base handler: " << e.what() << "\n";
}
```

#### 6.3 The Catch-All Handler: `catch (...)`

- The ellipsis syntax `catch (...)` catches **any exception of any type**, including primitive types and third-party classes.
- **Placement Rule:** Must always appear as the **very last catch block** in the handler list. Placing `catch (...)` before other catch handlers causes a compile-time error.

---

### 7. Re-throwing Exceptions

A catch block can intercept an exception, perform partial cleanup or logging, and re-throw the **exact same exception** up to the next enclosing `try-catch` scope using `throw;` without operands:

```cpp
void auditAndPropagate() {
    try {
        parseConfigFile();
    }
    catch (const std::exception& e) {
        std::cerr << "[LOG]: Error detected: " << e.what() << ". Forwarding to manager...\n";
        throw; // Re-throws current active exception (preserves dynamic type!)
    }
}
```

> ⚠️ **The Slicing Re-throw Trap:**
>
> - `throw;` (without operands) preserves the original exception object and its dynamic polymorphic type.
> - `throw e;` creates a brand-new copy of `e`, causing **Object Slicing** if `e` was caught by base reference!

---

### 8. High-Yield CMCE Traps & Interview Questions

- **Q1: What happens if an exception is thrown but no matching `catch` block is found?**
  - _Answer:_ The runtime calls `std::terminate()` (which by default invokes `abort()`), abnormally ending the entire process without guaranteeing that destructors for local stack objects in other frames will execute.

- **Q2: Does C++ support automatic type conversion during exception matching?**
  - _Answer:_ **No.** Exception matching is much stricter than function overload resolution:
    - `char` does **not** promote to `int`.
    - `int` does **not** convert to `double`.
    - `float` does **not** promote to `double`.
    - The only conversions allowed are:
      1. Derived-class to Base-class conversions (upcasting).
      2. Non-const to const qualification conversions.

- **Q3: What is the output of the following code snippet?**

  ```cpp
  #include <iostream>
  int main() {
      try {
          throw 'A';
      }
      catch (int x) {
          std::cout << "Int: " << x;
      }
      catch (char c) {
          std::cout << "Char: " << c;
      }
      return 0;
  }
  ```

  - _Output:_ `Char: A`
  - _Explanation:_ Unlike function calls where `'A'` implicitly promotes to `int` (`65`), exceptions do not undergo integral promotion. It bypasses `catch (int)` and lands in `catch (char)`.

- **Q4: Why should exceptions never be thrown from a destructor?**
  - _Answer:_ Destructors default to `noexcept(true)` in modern C++. If a destructor throws while another exception is already actively unwinding the stack (two active exceptions simultaneously), the C++ runtime triggers `std::terminate()` immediately.

- **Q5: Can you throw an exception from inside a constructor?**
  - _Answer:_ **Yes, and it is the recommended way to signal initialization failures.** Because constructors cannot return error codes, throwing an exception stops construction and signals failure. _Note:_ The destructor of the object being constructed will **not** execute, but destructors for fully initialized member subobjects and base classes will run cleanly during stack unwinding.
