# Module 01: Basics of C++

## Unit 1.4: Control Flow Statements

### 1. Conditional (Selection) Statements

Conditional statements alter program execution flow based on whether an expression evaluates to `true` (non-zero) or `false` (zero).

---

#### 1.1 `if`, `if-else`, and `else-if` Ladder

- **Syntax:**

  ```cpp
  if (condition1) {
      // executes if condition1 is true
  } else if (condition2) {
      // executes if condition1 is false AND condition2 is true
  } else {
      // executes if all conditions are false
  }
  ```

- **Dangling Else Problem:**
  An `else` clause always binds to the **nearest preceding unbounded `if`**, regardless of code indentation.

  ```cpp
  int x = 10, y = 5;
  if (x > 5)
      if (y > 10)
          std::cout << "A";
  else
      std::cout << "B"; // Belongs to 'if (y > 10)', NOT 'if (x > 5)'!
  // Output: Nothing is printed because x > 5 is true, but y > 10 is false.
  ```

  _Solution:_ Always use braces `{}` to make scope and intent explicit.

- **C++17 Init-Statement `if`:**
  Enables initializing variables within the `if` scope, limiting variable lifetime and preventing scope pollution.
  ```cpp
  if (auto it = myMap.find(key); it != myMap.end()) {
      std::cout << it->second;
  } // 'it' goes out of scope here
  ```

---

#### 1.2 `switch-case` Statement

The `switch` statement performs multi-way branching by comparing an integral or enumeration expression against constant values.

- **Syntax & Rules:**

  ```cpp
  switch (integral_expression) {
      case CONSTANT_1:
          // code
          break;
      case CONSTANT_2:
          // code
          [[fallthrough]]; // C++17 attribute to suppress fallthrough warnings
      case CONSTANT_3:
          // code
          break;
      default:
          // executed if no case matches
          break;
  }
  ```

- **Mandatory Rules for `switch` in C++:**
  1. The switch expression **must evaluate to an integral or enum type** (`int`, `char`, `short`, `long`, `enum`, `enum class`). Floating-point types (`float`, `double`) and strings (`std::string`) are **illegal**.
  2. Case labels must be **compile-time constant expressions** (`case 5:`, `case 'A':`, `case CONST_VAL:`). Variables are not allowed.
  3. No duplicate case values are permitted in the same switch block.
  4. The `default` label is optional and can appear anywhere inside the switch body, though placing it at the end is standard practice.

- **Fallthrough Behavior:**
  Without an explicit `break`, `return`, or `throw`, control cascades into subsequent cases regardless of whether their condition matches.

- **Variable Initialization Inside `switch` (CMCE Classic Pitfall):**
  Jumping over an initialization into a case causes a compile error:

  ```cpp
  switch (x) {
      case 1:
          int val = 10; // COMPILE ERROR: crosses initialization of 'val'
          std::cout << val;
          break;
      case 2:
          std::cout << "Two";
          break;
  }
  ```

  _Fix:_ Enclose the case body in braces `{}` to give the variable block scope:

  ```cpp
  case 1: {
      int val = 10; // OK: Scoped to this block
      std::cout << val;
      break;
  }
  ```

- **Under the Hood (V-Table / Jump Table):**
  If cases are dense, the compiler constructs a **jump table** (an array of code addresses) offering $\mathcal{O}(1)$ dispatch complexity, unlike an `if-else` ladder which degrades to $\mathcal{O}(n)$.

---

### 2. Looping (Iteration) Statements

#### 2.1 The `while` Loop (Entry-Controlled)

Tests the condition _before_ executing the loop body. If the condition is false initially, the body executes zero times.

```cpp
while (condition) {
    // Body executes 0 or more times
}
```

#### 2.2 The `do-while` Loop (Exit-Controlled)

Tests the condition _after_ executing the loop body. The body is **guaranteed to execute at least once**.

```cpp
do {
    // Body executes 1 or more times
} while (condition); // NOTE: Semicolon is mandatory!
```

#### 2.3 The `for` Loop (Entry-Controlled)

Combines initialization, condition testing, and iteration step into a single compact header.

```cpp
for (initialization; condition; increment/decrement) {
    // Body
}
```

- **Execution Order:**
  $$\text{Initialization} \longrightarrow \text{Condition Check} \longrightarrow \text{Loop Body} \longrightarrow \text{Increment/Decrement} \longrightarrow \text{Condition Check} \dots$$

- **Omission of Expressions:**
  ```cpp
  for (;;) {
      // Infinite loop (condition defaults to true)
  }
  ```

#### 2.4 Range-Based `for` Loop (C++11)

Iterates over elements in a container or array using `begin()` and `end()` iterators under the hood.

```cpp
int arr[] = {1, 2, 3, 4, 5};

// Read-only (by const reference to avoid copies)
for (const auto& elem : arr) {
    std::cout << elem << " ";
}

// In-place modification (by non-const reference)
for (auto& elem : arr) {
    elem *= 2;
}
```

---

### 3. Jump Statements

Jump statements unconditionally transfer program control to another line of code.

| Keyword    | Target / Behavior                                         | Scope of Effect                         |
| :--------- | :-------------------------------------------------------- | :-------------------------------------- |
| `break`    | Terminates the loop or `switch` entirely                  | Nearest enclosing loop or `switch` only |
| `continue` | Skips remaining body statements; begins next iteration    | Nearest enclosing loop only             |
| `return`   | Exits the current function and optionally returns a value | Current function frame                  |
| `goto`     | Jumps unconditionally to a labeled statement (`label:`)   | Within local function scope only        |

#### 3.1 Mechanics of `continue` in `for` vs `while`

- In a `while` / `do-while` loop, `continue` jumps **directly to the condition test**.
- In a `for` loop, `continue` jumps **to the increment/decrement expression first**, then tests the condition.

```cpp
// Caution in while loops:
int i = 0;
while (i < 5) {
    if (i == 3) {
        continue; // INFINITE LOOP: i is never incremented!
    }
    i++;
}
```

#### 3.2 The `goto` Statement and Cleanup Hazards

```cpp
    goto error_handler;
    // ...
error_handler:
    // handle error
```

- **Restriction:** `goto` cannot jump past a variable declaration that has an initializer in the same scope.
- **Modern Best Practice:** Avoid `goto`. Rely on RAII (Resource Acquisition Is Initialization), exception handling, and structured loops instead.

---

### 4. Nested Loops

A loop placed inside the body of another loop. The inner loop executes completely for **every single iteration** of the outer loop.

- **Total Iterations:** If the outer loop runs $N$ times and the inner loop runs $M$ times, the innermost statements execute $N \times M$ times.

```cpp
// Generating a 2D coordinate grid
for (int i = 0; i < 3; ++i) {
    for (int j = 0; j < 2; ++j) {
        std::cout << "(" << i << "," << j << ") ";
    }
    std::cout << "\n";
}
/* Output:
   (0,0) (0,1)
   (1,0) (1,1)
   (2,0) (2,1)
*/
```

- **Breaking Out of Nested Loops:**
  `break` only exits the **innermost** loop. To exit all nested levels simultaneously:
  1. Use a boolean control flag:
     ```cpp
     bool stop = false;
     for (...) {
         for (...) {
             if (condition) { stop = true; break; }
         }
         if (stop) break;
     }
     ```
  2. Encapsulate the loop in a function and use `return`.

---

### 5. High-Yield Output-Based Questions (CDAC CMCE Focus)

#### Question 1: Tricky Switch Fallthrough

```cpp
#include <iostream>
int main() {
    int i = 2;
    switch (i) {
        case 1: std::cout << "1";
        case 2: std::cout << "2";
        case 3: std::cout << "3";
        default: std::cout << "D";
    }
    return 0;
}
```

- **Output:** `23D`
- **Explanation:** `i == 2` matches `case 2`. Because there are no `break` statements, execution falls through all remaining branches (`case 3` and `default`).

---

#### Question 2: Pre-increment vs Post-increment in Loop Condition

```cpp
#include <iostream>
int main() {
    int x = 3;
    while (x-- > 0) {
        std::cout << x << " ";
    }
    std::cout << "\n";
    int y = 3;
    while (--y > 0) {
        std::cout << y << " ";
    }
    return 0;
}
```

- **Output:**
  ```
  2 1 0
  2 1
  ```
- **Explanation:**
  - First loop (`x-- > 0`): Checks `3 > 0` (true), decrements `x` to `2`, prints `2`. Next checks `2 > 0` $\rightarrow$ prints `1`. Checks `1 > 0` $\rightarrow$ prints `0`. Checks `0 > 0` (false), decrements `x` to `-1`, terminates.
  - Second loop (`--y > 0`): Decrements `y` to `2`, checks `2 > 0` (true) $\rightarrow$ prints `2`. Decrements to `1`, checks `1 > 0` $\rightarrow$ prints `1`. Decrements to `0`, checks `0 > 0` (false), terminates.

---

#### Question 3: Comma Operator in `for` Loop Header

```cpp
#include <iostream>
int main() {
    int i, j;
    for (i = 0, j = 0; i < 5, j < 3; i++, j++) {
        // loop body empty
    }
    std::cout << i << " " << j << std::endl;
    return 0;
}
```

- **Output:** `3 3`
- **Explanation:** In the test condition `i < 5, j < 3`, the comma operator evaluates `i < 5`, discards the result, and returns the result of `j < 3`. The loop terminates purely based on `j < 3`. When `j` reaches `3`, the loop stops with `i = 3` and `j = 3`.

---

#### Question 4: Semicolon after Loop Condition

```cpp
#include <iostream>
int main() {
    int i = 0;
    for (i = 0; i < 5; i++);
    {
        std::cout << i << " ";
    }
    return 0;
}
```

- **Output:** `5`
- **Explanation:** The semicolon immediately following `for (...);` terminates the loop with an empty body. The loop increments `i` until `i == 5`. The subsequent block `{ ... }` is an independent compound statement executed once after loop termination, printing `5`.

---

#### Question 5: Float / Type Precision in `for` Loops

```cpp
#include <iostream>
int main() {
    float a = 0.7f;
    if (a < 0.7)
        std::cout << "CDAC\n";
    else
        std::cout << "ACTS\n";
    return 0;
}
```

- **Output:** `CDAC`
- **Explanation:** Double literals like `0.7` have higher precision than `float` literals (`0.7f`). In binary floating-point representation, `0.7f` approximations yield a value slightly less than double-precision `0.7` (`0.699999988... < 0.70000000000000007`). Hence the condition evaluates to `true`.
