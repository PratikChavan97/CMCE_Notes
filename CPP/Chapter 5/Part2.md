# Module 05: Exception Handling & Advanced Robustness

## Unit 5.2: Advanced Exception Handling

### 1. Exception Propagation

**Exception Propagation** is the process by which an unhandled exception automatically travels upward through nested function calls along the call stack until a matching `catch` block is encountered.

```
[ Call Stack Traversal ]
Level 3: funcC()  ---> throw runtime_error("Fail"); (No local catch found!)
                           |
                           v  [Stack Frame 3 Unwound & Popped]
Level 2: funcB()  ---> Calls funcC()                (No local catch found!)
                           |
                           v  [Stack Frame 2 Unwound & Popped]
Level 1: funcA()  ---> Calls funcB()                (Inside try block)
                           |
                           v  [Matching catch found in funcA!]
                      catch (const std::runtime_error& e) { ... }
```

#### 1.1 Unhandled Propagation to `main()`

If an exception reaches `main()` without finding a matching handler:

1. The global runtime calls `std::terminate()`.
2. By default, `std::terminate()` invokes `std::abort()`.
3. The process terminates abruptly, and the operating system produces a core dump or crash log.

---

### 2. Stack Unwinding

**Stack Unwinding** is the runtime mechanism that systematically destroys all local, automatic stack-allocated objects between the point where a `throw` occurs and the point where the matching `catch` handler takes control.

```
+-------------------------------------------------------------------------+
|                        Stack Unwinding Sequence                         |
+-------------------------------------------------------------------------+
| 1. Exception thrown at instruction pointer IP                           |
| 2. Runtime runtime lookup queries exception handling table (.eh_frame)  |
| 3. Stack pointer moves backward frame by frame                          |
| 4. Destructors called for every local object in reverse order of        |
|    construction for each vacated scope                                  |
| 5. Program counter resumes execution inside the matched catch block     |
+-------------------------------------------------------------------------+
```

#### 2.1 Stack Unwinding Demonstration

```cpp
#include <iostream>
#include <stdexcept>

class ResourceTracker {
    std::string name;
public:
    ResourceTracker(std::string n) : name(n) {
        std::cout << "Acquired: " << name << "\n";
    }
    ~ResourceTracker() {
        std::cout << "Cleaned up: " << name << "\n";
    }
};

void levelThree() {
    ResourceTracker r3("Resource_C");
    throw std::runtime_error("Crash at Level 3");
    ResourceTracker rNever("Unreachable"); // Never constructed
}

void levelTwo() {
    ResourceTracker r2("Resource_B");
    levelThree();
}

void levelOne() {
    ResourceTracker r1("Resource_A");
    try {
        levelTwo();
    } catch (const std::exception& e) {
        std::cout << "Caught in levelOne: " << e.what() << "\n";
    }
}

int main() {
    levelOne();
    return 0;
}
```

- **Output:**
  ```text
  Acquired: Resource_A
  Acquired: Resource_B
  Acquired: Resource_C
  Cleaned up: Resource_C
  Cleaned up: Resource_B
  Caught in levelOne: Crash at Level 3
  Cleaned up: Resource_A
  ```
- **Key Takeaway:** Notice `Resource_C` and `Resource_B` are destructed **before** the `catch` block body executes. `Resource_A` lives on because its enclosing block has not yet exited.

---

### 3. Standard Exception Hierarchy (`<stdexcept>`)

All standard C++ exception classes derive directly or indirectly from the base class `std::exception` declared in `<exception>`.

```
                              std::exception
                             (virtual what())
                                    |
            +-----------------------+-----------------------+
            |                                               |
     std::logic_error                               std::runtime_error
(Compile/Design logic faults)                  (Unavoidable runtime conditions)
            |                                               |
  +---------+---------+                           +---------+---------+
  |         |         |                           |         |         |
invalid_  out_of_   length_                     range_    overflow_ underflow_
argument  range     error                       error     error     error
            |                                               |
      domain_error                                   system_error (C++11)

Other Direct Children of std::exception:
- std::bad_alloc      (Memory exhaustion via operator new)
- std::bad_cast       (Failed dynamic_cast on reference)
- std::bad_typeid     (typeid on nullptr)
- std::bad_optional_access (C++17)
```

#### 3.1 `logic_error` vs `runtime_error`

| Metric         | `std::logic_error`                                                                                          | `std::runtime_error`                                                                                 |
| :------------- | :---------------------------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------- |
| **Definition** | Violations of program logic and preconditions that could theoretically be prevented by writing better code. | Errors triggered by external environmental factors that cannot be detected until runtime.            |
| **Examples**   | Passing an invalid argument (`invalid_argument`), index out of bounds (`out_of_range`).                     | Running out of disk space, network sockets dropping, numeric arithmetic overflow (`overflow_error`). |
| **Detection**  | Preventable via assertions and input validation.                                                            | Must be anticipated and handled defensively.                                                         |

#### 3.2 The `what()` Member Function

- Signature: `virtual const char* what() const noexcept;`
- Returns an explanatory, null-terminated C-string describing the error.
- Overridden in custom exceptions to pass human-readable diagnostic messages.

---

### 4. User-Defined Custom Exceptions

Creating custom domain-specific exceptions ensures clear, typed error handling across enterprise applications.

#### 4.1 Inheriting from `std::exception` or `std::runtime_error`

```cpp
#include <iostream>
#include <string>
#include <stdexcept>

// Good Practice: Inherit from std::runtime_error or std::exception
class InsufficientFundsException : public std::runtime_error {
private:
    double currentBalance;
    double requestedAmount;

public:
    InsufficientFundsException(double balance, double requested)
        : std::runtime_error("Transaction declined: Insufficient funds."),
          currentBalance(balance),
          requestedAmount(requested) {}

    // Custom inspectors to extract diagnostic state
    double getBalance() const noexcept { return currentBalance; }
    double getRequested() const noexcept { return requestedAmount; }
};

class BankAccount {
    double balance;
public:
    BankAccount(double b) : balance(b) {}

    void withdraw(double amount) {
        if (amount > balance) {
            throw InsufficientFundsException(balance, amount);
        }
        balance -= amount;
    }
};

int main() {
    BankAccount acc(500.0);
    try {
        acc.withdraw(1200.0);
    } catch (const InsufficientFundsException& e) {
        std::cerr << e.what() << "\n";
        std::cerr << "Balance: $" << e.getBalance()
                  << " | Attempted: $" << e.getRequested() << "\n";
    }
    return 0;
}
```

---

### 5. Exception Handling with Classes

#### 5.1 Exceptions in Constructors

A constructor's sole responsibility is to establish object invariants. If an invariant cannot be satisfied, throwing an exception is the **only standard-compliant way** to signal failure.

```cpp
class DynamicArray {
    int* data;
    size_t size;
public:
    DynamicArray(size_t s) : size(s), data(nullptr) {
        if (s == 0) {
            throw std::invalid_argument("Size must be positive.");
        }
        data = new int[s]; // If this throws bad_alloc, construction aborts!
    }

    ~DynamicArray() {
        delete[] data;
    }
};
```

> ⚠️ **The Constructor Exception Trap:**
> If an exception is thrown inside a constructor body, **the object is considered non-existent**. Its own destructor `~DynamicArray()` will **NOT** run. Any raw heap resource allocated before the throw statement leaks unless wrapped in RAII guards or cleaned up in a function-level `try` block!

#### 5.2 Function-Try Blocks for Constructors

Used to intercept exceptions thrown by base class constructors or Member Initializer Lists (MIL):

```cpp
class Base {
public:
    Base(int) { throw std::runtime_error("Base MIL failed"); }
};

class Derived : public Base {
public:
    // Function-try block syntax
    Derived(int x) try : Base(x) {
        // Constructor body
    } catch (const std::exception& e) {
        std::cerr << "Caught in Derived MIL handler: " << e.what() << "\n";
        // NOTE: The exception is AUTOMATICALLY re-thrown here!
        // You cannot swallow exceptions in a constructor function-try block!
    }
};
```

#### 5.3 Destructors and Exceptions: The Golden Invariant

- Destructors in C++11 and later are implicitly marked **`noexcept(true)`**.
- **Rule:** A destructor must **never throw an exception** out of its body.
- If a destructor throws during stack unwinding (when another active exception is already propagating), C++ calls `std::terminate()`, killing the application instantly.

---

### 6. Exception Handling Best Practices & Safety Guarantees

#### 6.1 The Four Exception Safety Levels (Abrahams Guarantees)

1. **No-Throw Guarantee (`noexcept`):**
   - The operation is guaranteed to never fail and never throw an exception (e.g., move constructors, swap functions, destructors).
2. **Strong Exception Guarantee (Commit-or-Rollback):**
   - If an exception occurs, program state remains completely unchanged, as if the operation had never been called (atomic transactions).
3. **Basic Exception Guarantee:**
   - If an exception occurs, no memory or resources are leaked, and all objects remain in a valid, destructible, internally consistent state (invariants preserved).
4. **No Guarantee:**
   - If an exception occurs, memory leaks, resource corruption, or undefined behavior may result.

#### 6.2 Modern Best Practices Summary

1. **Rely on RAII (Resource Acquisition Is Initialization):**
   Never manage raw pointers in scopes where exceptions can fly. Use smart pointers (`std::unique_ptr`, `std::shared_ptr`) so stack unwinding automatically releases dynamic memory without manual `catch` blocks.
2. **Throw by Value, Catch by `const` Reference:**
   - Prevents object slicing.
   - Avoids heap allocation overhead.
   - Preserves polymorphic `what()` overrides.
3. **Mark Non-Throwing Operations with `noexcept`:**
   - Helps compilers optimize move semantics (e.g., `std::vector::resize` will only use a class's move constructor if it is marked `noexcept`).
4. **Don't Use Exceptions for Normal Flow Control:**
   Exceptions incur substantial CPU branch penalties during stack unwinding. Use them exclusively for exceptional runtime conditions.

---

### 7. High-Yield CMCE Traps & Interview Questions

- **Q1: Can a constructor catch an exception thrown in its own Member Initializer List (MIL) and suppress it?**
  - _Answer:_ **No.** Even if a constructor catches a MIL exception via a function-try block, the C++ standard enforces that the exception is **automatically re-thrown** when reaching the end of the `catch` block. This prevents code from interacting with an incompletely initialized object.

- **Q2: What is the purpose of the `noexcept` operator vs the `noexcept` specifier?**
  - _Answer:_
    - **`noexcept` specifier:** Declares that a function promises not to throw (`void swap() noexcept;`).
    - **`noexcept` operator:** Evaluates at compile-time whether an expression can throw, returning a boolean (`bool b = noexcept(swap());`).

- **Q3: What happens if an exception is thrown out of a `noexcept` function?**
  - _Answer:_ The runtime immediately invokes `std::terminate()`. The stack is **not guaranteed to unwind**, meaning object destructors along the path may not run.

- **Q4: Why does `throw;` differ from `throw e;` inside a catch block?**
  - _Answer:_ `throw;` re-throws the original polymorphic exception object as-is. `throw e;` constructs a new copy of `e` using its static type at the catch site, leading to **Object Slicing** if caught by a base class reference.

- **Q5: What is `std::current_exception()` and `std::rethrow_exception()`?**
  - _Answer:_ Introduced in C++11 (`<exception>`), these functions capture the current active exception into a reference-counted `std::exception_ptr` object, allowing exceptions to be safely transported across different threads.
