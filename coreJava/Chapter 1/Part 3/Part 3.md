# Module 01: Basics of C++

## Unit 1.3: Operators and Expressions

---

### 1. Arithmetic Operators

Arithmetic operators perform fundamental mathematical calculations on numerical operands.

| Operator | Meaning            | Example (`a = 10, b = 3`) | Result | Notes                                       |
| :------- | :----------------- | :------------------------ | :----- | :------------------------------------------ |
| `+`      | Addition           | `a + b`                   | `13`   | Also acts as unary plus (`+a`)              |
| `-`      | Subtraction        | `a - b`                   | `7`    | Also acts as unary negation (`-a`)          |
| `*`      | Multiplication     | `a * b`                   | `30`   | Precision depends on operand types          |
| `/`      | Division           | `a / b`                   | `3`    | **Integer division truncates towards zero** |
| `%`      | Modulo (Remainder) | `a % b`                   | `1`    | **Operands MUST be integral types only**    |

#### Critical Rules & Pitfalls:

1. **Integer Division:** If both operands are integers, the fraction is truncated (not rounded):
   ```cpp
   double res = 5 / 2;     // Evaluates to 2, then converts to 2.0!
   double correct = 5.0 / 2; // Evaluates to 2.5
   ```
2. **Modulo `%` Restrictions:**
   - Operands **cannot** be `float` or `double`. Using `5.5 % 2` causes a compile error. For floating-point remainders, use `<cmath>` function `std::fmod()`.
   - **Sign of Modulo Result in C++11 and later:** The sign of `a % b` is determined by the sign of the **dividend (`a`)**, regardless of the sign of the divisor (`b`):
     ```cpp
     -7 % 3  == -1
      7 % -3 ==  1
     -7 % -3 == -1
     ```
3. **Division by Zero:** Division or modulo by `0` produces **undefined behavior (UB)** (typically causing a `SIGFPE` crash).

---

### 2. Relational Operators

Relational operators compare two values and produce a boolean result (`true` or `false`, convertible to `1` or `0`).

| Operator | Description                                           | Example (`x = 5, y = 8`) | Result                       |
| :------- | :---------------------------------------------------- | :----------------------- | :--------------------------- |
| `==`     | Equal to                                              | `x == y`                 | `false` (0)                  |
| `!=`     | Not equal to                                          | `x != y`                 | `true` (1)                   |
| `<`      | Less than                                             | `x < y`                  | `true` (1)                   |
| `>`      | Greater than                                          | `x > y`                  | `false` (0)                  |
| `<=`     | Less than or equal to                                 | `x <= y`                 | `true` (1)                   |
| `>=`     | Greater than or equal to                              | `x >= y`                 | `false` (0)                  |
| `<=>`    | **Three-way comparison** _(C++20 spaceship operator)_ | `x <=> y`                | `std::strong_ordering::less` |

#### Critical Rules & Pitfalls:

- **The Assignment Trap:** `if (x = 5)` assigns `5` to `x`, evaluates as non-zero (true), and executes the block. Always use `if (x == 5)`. (Writing `if (5 == x)` is the "Yoda condition" trick to catch this error at compile time).
- **Floating-Point Equality:** Never compare floats with `==` due to precision limits:
  ```cpp
  double a = 0.1 + 0.2;
  if (a == 0.3) { /* RISKY: Likely false! */ }
  // Correct approach (Epsilon comparison):
  if (std::abs(a - 0.3) < 1e-9) { /* Safe */ }
  ```
- **Chained Comparison Pitfall:**
  ```cpp
  int a = 5, b = 3, c = 1;
  if (a > b > c) { /* Evaluates as: (5 > 3) -> 1; then (1 > 1) -> false! */ }
  // Correct:
  if (a > b && b > c) { /* true */ }
  ```

---

### 3. Logical Operators

Used to combine or invert boolean conditions.

| Operator | Meaning     | Syntax             | Short-Circuit Evaluation?                           |
| :------- | :---------- | :----------------- | :-------------------------------------------------- |
| `&&`     | Logical AND | `cond1 && cond2`   | **Yes** (if `cond1` is `false`, `cond2` is skipped) |
| `\|\|`   | Logical OR  | `cond1 \|\| cond2` | **Yes** (if `cond1` is `true`, `cond2` is skipped)  |
| `!`      | Logical NOT | `!cond`            | N/A (Unary)                                         |

#### Short-Circuit Evaluation Mechanics (High-Yield MCQ Topic):

```cpp
int x = 0, y = 5;

// Short-circuit AND:
if (x != 0 && ++y > 5) {
    // ++y is NEVER executed because x != 0 is false!
}
// y remains 5!

// Short-circuit OR:
if (y == 5 || ++x > 0) {
    // ++x is NEVER executed because y == 5 is true!
}
// x remains 0!
```

- **Use case for Short-Circuiting (Null Safety):**
  ```cpp
  if (ptr != nullptr && ptr->data == 42) {
      // Safe! If ptr is null, ptr->data is never dereferenced.
  }
  ```

---

### 4. Bitwise Operators

Bitwise operators manipulate data directly at the binary bit level. They **only operate on integral types** (`char`, `int`, `long`, etc.).

| Operator | Name                     | Bit Operation Rule                  | Example (`a = 5` [0101], `b = 3` [0011])                      |
| :------- | :----------------------- | :---------------------------------- | :------------------------------------------------------------ |
| `&`      | Bitwise AND              | `1 & 1 = 1`, else `0`               | `a & b` $\rightarrow$ `0001` (1)                              |
| `\|`     | Bitwise OR               | `0 \| 0 = 0`, else `1`              | `a \| b` $\rightarrow$ `0111` (7)                             |
| `^`      | Bitwise XOR              | Same bits = `0`, Different = `1`    | `a ^ b` $\rightarrow$ `0110` (6)                              |
| `~`      | Bitwise NOT (1's Compl.) | Inverts all bits (`0 -> 1, 1 -> 0`) | `~a` $\rightarrow$ `...11111010` (-6)                         |
| `<<`     | Left Shift               | Shifts bits left, fills with `0`    | `a << 1` $\rightarrow$ `1010` (10) ($a \times 2^1$)           |
| `>>`     | Right Shift              | Shifts bits right                   | `a >> 1` $\rightarrow$ `0010` (2) ($\lfloor a / 2^1 \rfloor$) |

#### Must-Know Bitwise Tricks for Technical Interviews:

1. **Check if a number is Odd or Even:**
   ```cpp
   if (n & 1) /* Odd */ else /* Even */
   ```
2. **Check if $N$ is a Power of 2:**
   ```cpp
   bool isPowerOfTwo = (n > 0) && ((n & (n - 1)) == 0);
   ```
3. **Swap Two Variables without a Temporary Variable:**
   ```cpp
   a = a ^ b;
   b = a ^ b;
   a = a ^ b;
   ```
4. **Clear the lowest set bit:**
   ```cpp
   n = n & (n - 1);
   ```
5. **Set, Clear, and Toggle $k$-th bit ($0$-indexed):**
   - **Set $k$-th bit:** `n |= (1 << k);`
   - **Clear $k$-th bit:** `n &= ~(1 << k);`
   - **Toggle $k$-th bit:** `n ^= (1 << k);`
   - **Check $k$-th bit:** `bool isSet = (n & (1 << k)) != 0;`

---

### 5. Assignment and Unary Operators

#### 5.1 Assignment Operators

- **Simple Assignment:** `=` (stores right-hand value in left-hand lvalue).
- **Compound (Augmented) Assignment:** `+=`, `-=`, `*=`, `/=`, `%=`, `&=`, `|=`, `^=`, `<<=`, `>>=`.
  - `a += 5;` is functionally equivalent to `a = a + 5;` (except `a` is evaluated only once).

#### 5.2 Unary Operators

Operate on a single operand. Includes `+`, `-`, `!`, `~`, `&` (address-of), `*` (dereference), `sizeof`, `typeid`, and increment/decrement.

#### 5.3 Prefix vs. Postfix Increment/Decrement

```cpp
int a = 5;
int pre = ++a;   // Pre-increment: Increment first, then return reference to updated variable (pre = 6, a = 6)

int b = 5;
int post = b++;  // Post-increment: Save copy of current value, increment variable, return saved copy (post = 5, b = 6)
```

| Dimension                    | Prefix (`++x`)                           | Postfix (`x++`)                                                     |
| :--------------------------- | :--------------------------------------- | :------------------------------------------------------------------ |
| **Mechanics**                | Increment $\rightarrow$ Return new value | Save old copy $\rightarrow$ Increment $\rightarrow$ Return old copy |
| **Return Value**             | Lvalue reference (`T&`)                  | Rvalue copy (`T`)                                                   |
| **Performance (User Types)** | **Faster** (No temporary object created) | **Slower** (Creates a temporary copy of previous state)             |
| **Chainability**             | `++++x;` (Valid C++ syntax!)             | `x++++;` (Compile error: Cannot increment an rvalue)                |

---

### 6. Operator Precedence and Associativity

Precedence determines **which operator binds first** when an expression contains multiple operators without parentheses. Associativity determines the direction of evaluation when operators share the **same precedence level**.

#### Comprehensive Precedence Table (Highest to Lowest):

| Level  | Operator Category       | Operators                                                                  |   Associativity   |
| :----: | :---------------------- | :------------------------------------------------------------------------- | :---------------: |
| **1**  | Scope Resolution        | `::`                                                                       |   Left-to-Right   |
| **2**  | Postfix / Member Access | `()` `[]` `->` `.` `x++` `x--` `typeid` `dynamic_cast...`                  |   Left-to-Right   |
| **3**  | Unary Operators         | `++x` `--x` `+` `-` `!` `~` `*` (deref) `&` (addr) `sizeof` `new` `delete` | **Right-to-Left** |
| **4**  | C-style Cast            | `(type)`                                                                   | **Right-to-Left** |
| **5**  | Pointer-to-Member       | `.*` `->*`                                                                 |   Left-to-Right   |
| **6**  | Multiplicative          | `*` `/` `%`                                                                |   Left-to-Right   |
| **7**  | Additive                | `+` `-`                                                                    |   Left-to-Right   |
| **8**  | Bitwise Shift           | `<<` `>>`                                                                  |   Left-to-Right   |
| **9**  | Three-way (Spaceship)   | `<=>`                                                                      |   Left-to-Right   |
| **10** | Relational              | `<` `<=` `>` `>=`                                                          |   Left-to-Right   |
| **11** | Equality                | `==` `!=`                                                                  |   Left-to-Right   |
| **12** | Bitwise AND             | `&`                                                                        |   Left-to-Right   |
| **13** | Bitwise XOR             | `^`                                                                        |   Left-to-Right   |
| **14** | Bitwise OR              | `\|`                                                                       |   Left-to-Right   |
| **15** | Logical AND             | `&&`                                                                       |   Left-to-Right   |
| **16** | Logical OR              | `\|\|`                                                                     |   Left-to-Right   |
| **17** | Ternary Conditional     | `? :`                                                                      | **Right-to-Left** |
| **18** | Assignment Operators    | `=` `+=` `-=` `*=` `/=` `%=` `<<=` `>>=` `&=` `^=` `\|=`                   | **Right-to-Left** |
| **19** | Throw Operator          | `throw`                                                                    | **Right-to-Left** |
| **20** | Comma Operator          | `,`                                                                        |   Left-to-Right   |

> ⚠️ **Key Memory Hack for Exams:**
> Unary, Ternary (`? :`), and Assignment operators group **Right-to-Left**. Almost everything else groups **Left-to-Right**.

---

### 7. Expressions and Evaluation Order

#### 7.1 Lvalues vs Rvalues (Foundational Concept)

- **Lvalue (Locator Value):** An expression that designates a persistent memory location. You can take its address using `&`. (e.g., named variables, references).
- **Rvalue (Read/Right Value):** A temporary value that does not persist beyond the expression that uses it. You cannot take its address. (e.g., literals like `42`, temporary calculations `a + b`).

```cpp
int x = 10;      // x is lvalue; 10 is rvalue
int* p = &x;     // Legal: x has an address
// int* q = &(x + 5); // ERROR: Cannot take address of rvalue expression (x + 5)
```

#### 7.2 The Comma Operator (`,`)

- Has the **lowest precedence** of all C++ operators.
- Evaluates expressions from left to right, discards intermediate results, and returns the **value and type of the rightmost expression**:

  ```cpp
  int a;
  a = (1, 2, 3); // a gets assigned 3

  int b = 1, 2, 3; // ERROR or unexpected: '=' has higher precedence than ','
                   // Parses as: (int b = 1), 2, 3;
  ```

#### 7.3 Undefined Evaluation Order Pitfalls (CMCE MCQ Classic)

Prior to C++17, the evaluation order of function arguments or operands in an expression was mostly unspecified:

```cpp
// UNDEFINED BEHAVIOR prior to C++17:
int i = 1;
cout << i << " " << ++i << endl; // Unspecified which executes first!

// NEVER modify a variable twice without a sequence point:
i = ++i + i++; // Undefined behavior!
```

---

### 8. Introduction to Operator Overloading

Operator overloading allows user-defined classes and structs to define custom behaviors for built-in operators, giving user types first-class language semantics.

```
Syntax:
ReturnType operator<Symbol>(ParameterList);
```

#### 8.1 Operators That CANNOT Be Overloaded (MEMORIZE FOR CMCE):

1. Scope Resolution: `::`
2. Direct Member Access: `.`
3. Pointer-to-Member: `.*`
4. Ternary Conditional: `?:`
5. Size Query: `sizeof`
6. Type Identification: `typeid`

#### 8.2 Operators That MUST Be Overloaded as Class Members:

These four operators **cannot** be implemented as standalone/friend functions:

1. Assignment: `=`
2. Subscript: `[]`
3. Function Call: `()`
4. Class Member Access through Pointer: `->`

#### 8.3 Conceptual Example: Overloading `+` and `<<`

```cpp
#include <iostream>

class Complex {
private:
    double real, imag;
public:
    Complex(double r = 0, double i = 0) : real(r), imag(i) {}

    // Overloading binary + as a member function: c1 + c2
    Complex operator+(const Complex& other) const {
        return Complex(this->real + other.real, this->imag + other.imag);
    }

    // Overloading << as a friend (non-member) function: cout << c1
    friend std::ostream& operator<<(std::ostream& os, const Complex& c) {
        os << c.real << " + " << c.imag << "i";
        return os; // Enables chaining: cout << c1 << c2;
    }
};

int main() {
    Complex c1(2.5, 3.5), c2(1.5, 2.0);
    Complex c3 = c1 + c2; // Calls c1.operator+(c2)
    std::cout << "Sum: " << c3 << std::endl; // Calls operator<<(cout, c3)
    return 0;
}
```

---

### 9. High-Yield CMCE Traps & Interview Questions

- **Q1: What is the difference between prefix and postfix increment in terms of performance?**
  - _Answer:_ For primitive types, compilers optimize both equally. For custom classes with overloaded operators, `++obj` (prefix) returns by reference with no overhead, whereas `obj++` (postfix) must create a temporary object to hold the old state, copy the state, increment the original, and return the temporary copy by value, incurring constructor/destructor overhead.

- **Q2: Why must stream insertion (`<<`) and extraction (`>>`) operators be overloaded as friend/non-member functions?**
  - _Answer:_ If overloaded as a member function of a class `MyClass`, the left operand must be an instance of `MyClass` (`myObj << cout`), which breaks conventional syntax. Because the left operand is `std::ostream` (`cout << myObj`), and we cannot modify the `std::ostream` standard class to add member functions, it must be implemented as a non-member/friend function taking `ostream&` as its first parameter.

- **Q3: What does the bitwise expression `x & (x - 1)` do?**
  - _Answer:_ It clears the lowest set bit (the rightmost bit equal to `1`) of `x`. If `x` is a power of 2, it has only one set bit, so `(x & (x - 1)) == 0`.

- **Q4: What will `cout << (1 << 2 + 1);` print?**
  - _Answer:_ It prints `8`. Additive operator `+` has higher precedence (Level 7) than bitwise shift `<<` (Level 8). Therefore, `2 + 1` evaluates to `3` first, followed by `1 << 3`, which yields `8`.

- **Q5: Can you change operator precedence, associativity, or arity via operator overloading?**
  - _Answer:_ **No.** Overloading an operator preserves its original precedence, associativity, and arity (number of operands). You cannot invent new operator tokens (e.g., `**`) either.
