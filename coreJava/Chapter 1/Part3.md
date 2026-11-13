# Module 01: Core Java Platform & Language Fundamentals

## Unit 1.3: Operators and Expressions

### 1. Introduction to Java Operators

An **operator** is a special symbol that instructs the compiler to perform specific mathematical, relational, logical, or bitwise manipulations on one, two, or three operands.

```
                             Java Operator Taxonomy
                                       │
     ┌─────────────────────────────────┼─────────────────────────────────┐
     ▼                                 ▼                                 ▼
Unary Operators                Binary Operators                  Ternary Operator
(Takes 1 operand)              (Takes 2 operands)                (Takes 3 operands)
- Postfix: expr++, expr--      - Arithmetic: +, -, *, /, %       - Conditional: ? :
- Prefix:  ++expr, --expr      - Relational: ==, !=, <, >, <=, >=
- Unary:   +, -, ~, !          - Logical:    &&, ||
                               - Bitwise:    &, |, ^
                               - Shift:      <<, >>, >>>
                               - Assignment: =, +=, -=, etc.
                               - Type-test:  instanceof
```

---

### 2. Arithmetic Operators

Arithmetic operators perform fundamental numeric computation on primitive numeric types (`byte`, `short`, `char`, `int`, `long`, `float`, `double`).

| Operator | Operation           | Description                                                             | Example ($a=10, b=3$) |
| :------- | :------------------ | :---------------------------------------------------------------------- | :-------------------- |
| `+`      | Addition            | Sum of operands (or string concatenation if either operand is `String`) | $a + b = 13$          |
| `-`      | Subtraction         | Difference between left and right operands                              | $a - b = 7$           |
| `*`      | Multiplication      | Product of operands                                                     | $a * b = 30$          |
| `/`      | Division            | Quotient of operands; truncates fractional part if both are integers    | $a / b = 3$           |
| `%`      | Modulus (Remainder) | Returns remainder after division; sign follows the **left operand**     | $a \% b = 1$          |

#### 2.1 The Division by Zero Dilemma: Integers vs. Floating-Point

```
Integer Division by 0:
int x = 10 / 0;   // Throws java.lang.ArithmeticException: / by zero at runtime!

Floating-Point Division by 0:
double y = 10.0 / 0.0;   // Evaluates to Double.POSITIVE_INFINITY (No exception thrown!)
double z = -10.0 / 0.0;  // Evaluates to Double.NEGATIVE_INFINITY
double nan = 0.0 / 0.0;  // Evaluates to Double.NaN (Not-a-Number)
```

#### 2.2 Sign Behavior in Modulus (`%`)

In Java, the sign of the result of $a \% b$ is **always equal to the sign of the dividend (left operand $a$)**, regardless of the sign of the divisor ($b$):

$$a \pmod b = a - (a / b) * b$$

```java
System.out.println( 10 %  3); //  1
System.out.println(-10 %  3); // -1 (Sign follows left operand)
System.out.println( 10 % -3); //  1 (Right operand sign is completely ignored!)
System.out.println(-10 % -3); // -1
```

---

### 3. Relational Operators

Relational operators determine the relative ordering or equality between operands. They always evaluate to a boolean primitive: **`true`** or **`false`**.

| Operator | Name                     | Applies To                                                                     | Example                       |
| :------- | :----------------------- | :----------------------------------------------------------------------------- | :---------------------------- |
| `==`     | Equal to                 | Primitives (value), References (address identity)                              | `5 == 5` $\rightarrow$ `true` |
| `!=`     | Not equal to             | Primitives (value), References (address identity)                              | `5 != 3` $\rightarrow$ `true` |
| `<`      | Strictly less than       | Numeric primitives (`byte`, `short`, `char`, `int`, `long`, `float`, `double`) | `3 < 5` $\rightarrow$ `true`  |
| `>`      | Strictly greater than    | Numeric primitives                                                             | `5 > 3` $\rightarrow$ `true`  |
| `<=`     | Less than or equal to    | Numeric primitives                                                             | `5 <= 5` $\rightarrow$ `true` |
| `>=`     | Greater than or equal to | Numeric primitives                                                             | `4 >= 2` $\rightarrow$ `true` |

> ⚠️ **The Reference Equality Trap (`==` vs. `.equals()`):**
>
> - For primitive types, `==` compares the raw binary value.
> - For reference types, `==` compares whether both variables hold the **exact same memory address on the Heap**. To compare deep structural equality (content), invoke `.equals()`.

---

### 4. Logical Operators & Short-Circuit Evaluation

Logical operators operate exclusively on boolean expressions.

```
       Logical AND (&&)                              Logical OR (||)
   (Both must be true)                           (At least one must be true)
┌────────┬────────┬─────────┐                 ┌────────┬────────┬─────────┐
│ Oper A │ Oper B │ Result  │                 │ Oper A │ Oper B │ Result  │
├────────┼────────┼─────────┤                 ├────────┼────────┼─────────┤
│ false  │  ...   │ false   │                 │ true   │  ...   │ true    │
│ true   │ false  │ false   │                 │ false  │ true   │ true    │
│ true   │ true   │ true    │                 │ false  │ false  │ false   │
└────────┴────────┴─────────┘                 └────────┴────────┴─────────┘
  ▲                                             ▲
  │ If Oper A is FALSE, Oper B                  │ If Oper A is TRUE, Oper B
  │ is NEVER evaluated!                         │ is NEVER evaluated!
  └─────── Short-Circuit ───────┘               └─────── Short-Circuit ───────┘
```

#### 4.1 Short-Circuit (`&&`, `||`) vs. Non-Short-Circuit (`&`, `|`)

- **`&&` (Conditional AND / Short-Circuit):** If the left operand evaluates to `false`, the overall expression cannot be true. The runtime **skips evaluation of the right operand entirely**.
- **`||` (Conditional OR / Short-Circuit):** If the left operand evaluates to `true`, the overall expression cannot be false. The runtime **skips evaluation of the right operand entirely**.
- **`&` (Logical Non-Short-Circuit AND):** Always evaluates **both** left and right operands, even if the left is `false`.
- **`|` (Logical Non-Short-Circuit OR):** Always evaluates **both** left and right operands, even if the left is `true`.

```java
String text = null;

// Safe with Short-Circuit: text != null is false; length() check is NEVER evaluated!
if (text != null && text.length() > 0) {
    System.out.println("Valid string");
}

// Catastrophic Crash with Non-Short-Circuit:
if (text != null & text.length() > 0) {
    // RUNTIME EXCEPTION: java.lang.NullPointerException!
    // Both sides are evaluated unconditionally!
}
```

---

### 5. Assignment Operators

The simple assignment operator (`=`) assigns the value of the right-hand operand to the variable on the left.

#### 5.1 Compound Assignment Operators

Java provides syntactic sugar combining binary operations with assignment: `+=`, `-=`, `*=`, `/=`, `%=`, `&=`, `|=`, `^=`, `<<=`, `>>=`, `>>>=`.

#### 5.2 The Implicit Cast Invariant (High-Yield CDAC Trap)

According to the Java Language Specification (JLS §15.26.2), a compound assignment of the form:

$$E_1 \text{ op}= E_2$$

is mathematically and structurally equivalent to:

$$E_1 = (T)(E_1 \text{ op } E_2)$$

where $T$ is the declared static type of $E_1$.

```java
short s = 10;

// Case A: Explicit arithmetic assignment
// s = s + 5; // COMPILE ERROR: s + 5 promotes to int; cannot assign int to short!

// Case B: Compound assignment
s += 5; // COMPILES PERFECTLY!
// Compiler automatically desugars this to: s = (short)(s + 5);

// The Overflow Hazard:
byte b = 127;
b += 1; // Desugars to: b = (byte)(b + 1);
System.out.println(b); // Prints: -128 (Implicit cast hides numeric overflow!)
```

---

### 6. Unary Operators

Unary operators accept exactly one operand.

| Operator | Type             | Name              | Description                                                         |
| :------- | :--------------- | :---------------- | :------------------------------------------------------------------ |
| `+`      | Prefix           | Unary Plus        | Indicates positive value; promotes `byte`, `short`, `char` to `int` |
| `-`      | Prefix           | Unary Negation    | Negates the numeric operand                                         |
| `++`     | Prefix / Postfix | Increment         | Increases value by 1                                                |
| `--`     | Prefix / Postfix | Decrement         | Decreases value by 1                                                |
| `~`      | Prefix           | Bitwise Inversion | Inverts every bit ($0 \leftrightarrow 1$); works on integer types   |
| `!`      | Prefix           | Logical NOT       | Inverts a boolean condition (`!true` $\rightarrow$ `false`)         |

#### 6.1 Prefix vs. Postfix Evaluation Lifecycle

- **Prefix (`++i`, `--i`):** Increment / decrement occurs **first**, before the variable's value is yielded to the enclosing expression (**"Update-then-Use"**).
- **Postfix (`i++`, `i--`):** The original value is loaded onto the Operand Stack for use in the enclosing expression **first**, and the variable's memory slot is modified **afterwards** (**"Use-then-Update"**).

```
Prefix: ++x                                Postfix: x++
┌──────────────────────────────┐           ┌──────────────────────────────┐
│ 1. Increment value in memory │           │ 1. Push current value to OS  │
│ 2. Push updated value to OS  │           │ 2. Increment value in memory │
└──────────────────────────────┘           └──────────────────────────────┘
```

```java
int a = 5;
int b = ++a; // a is updated to 6, then 6 is assigned to b. -> a=6, b=6

int x = 5;
int y = x++; // Old value 5 is assigned to y, then x is updated to 6. -> x=6, y=5

// Classic Interview Trap:
int count = 10;
count = count++;
System.out.println(count);
// Output: 10!
// Bytecode trace:
// 1. iload: loads 10 onto the operand stack
// 2. iinc: increments local variable 'count' in memory to 11
// 3. istore: pops the old 10 from the stack and overwrites 'count' back to 10!
```

---

### 7. Bitwise Operators

Bitwise operators manipulate individual binary bits of integer primitives (`byte`, `short`, `char`, `int`, `long`). Operands are automatically promoted to at least `int` (32 bits) before computation.

```
Inputs: A = 0101 (5), B = 0011 (3)

Bitwise AND (&):   1 only if BOTH bits are 1
  0101 & 0011 = 0001 (1)

Bitwise OR (|):    1 if AT LEAST ONE bit is 1
  0101 | 0011 = 0111 (7)

Bitwise XOR (^):   1 if bits are DIFFERENT
  0101 ^ 0011 = 0110 (6)

Bitwise NOT (~):   Inverts ALL bits (Two's complement: ~N = -(N + 1))
  ~00000000...0101 = 11111111...1010 (-6)
```

| Bit A | Bit B | A & B | A \| B | A ^ B | ~A  |
| :---: | :---: | :---: | :----: | :---: | :-: |
|   0   |   0   |   0   |   0    |   0   |  1  |
|   0   |   1   |   0   |   1    |   1   |  1  |
|   1   |   0   |   0   |   1    |   1   |  0  |
|   1   |   1   |   1   |   1    |   0   |  0  |

---

### 8. Shift Operators

Shift operators transpose the binary bit patterns of integer types left or right.

```
1. Signed Left Shift (<<): Shifts bits left; fills vacated right positions with 0.
   Bit Pattern: [ b31 b30 ... b2 b1 b0 ] << 2 ===> [ b29 ... b0 0 0 ]
   Formula: val << n === val * (2^n)

2. Signed Right Shift (>>): Shifts bits right; fills left with SIGN BIT (Arithmetic Shift).
   Positive Number: Fills vacated left bits with 0
   Negative Number: Fills vacated left bits with 1 (Preserves negative sign!)
   Formula: val >> n === floor(val / (2^n))

3. Unsigned Right Shift (>>>): Shifts bits right; ALWAYS fills left with 0 (Logical Shift).
   Turns negative numbers into large positive integers!
```

#### 8.1 Shift Operations Comparison

```java
int pos = 8;  // Binary: 00000000 00000000 00000000 00001000
int neg = -8; // Binary: 11111111 11111111 11111111 11111000

// 1. Signed Left Shift (<<)
System.out.println(pos << 2);  //  32 (8 * 2^2)
System.out.println(neg << 2);  // -32 (-8 * 2^2)

// 2. Signed Right Shift (>>)
System.out.println(pos >> 2);  //  2 (8 / 2^2)
System.out.println(neg >> 2);  // -2 (Sign preserved; vacated bits filled with 1s)

// 3. Unsigned Right Shift (>>>)
System.out.println(pos >>> 2); //  2 (Vacated bits filled with 0s)
System.out.println(neg >>> 2); //  1073741822 !
// Explanatory bit trace for neg >>> 2:
// Before: 11111111 11111111 11111111 11111000
// After:  00111111 11111111 11111111 11111110 (Sign bit becomes 0; massive positive int)
```

#### 8.2 Shift Distance Masking Rule

The CPU cannot shift an `int` more than 31 bits, or a `long` more than 63 bits. Java shields against illegal shifts by **masking the right-hand shift operand**:

- For `int`: Shift distance is masked with `0x1F` (only the lower 5 bits are used: `distance % 32`).
- For `long`: Shift distance is masked with `0x3F` (only the lower 6 bits are used: `distance % 64`).

```java
int a = 10;
System.out.println(a << 32); // Prints: 10! (32 % 32 = 0 shifts)
System.out.println(a << 33); // Prints: 20! (33 % 32 = 1 shift: 10 * 2^1)
```

---

### 9. Conditional Operator (Ternary Operator `? :`)

The only ternary operator in Java; it evaluates a condition and acts as an expression returning a value.

```
Syntax:
variable = (booleanCondition) ? expressionIfTrue : expressionIfFalse;
```

#### 9.1 Type Compatibility and Numeric Promotion in Ternary Operations

The compiler must deduce a single unified common return type for the ternary expression. It applies automatic numeric promotion to balance the second and third operands:

```java
// Trap: Operands have different numeric types
Object obj = (true) ? Integer.valueOf(1) : Double.valueOf(2.0);
System.out.println(obj); // Prints: 1.0 (NOT 1!)
// Explanation: The presence of Double causes Integer to be unboxed and promoted to double!
```

---

### 10. The `instanceof` Operator

The `instanceof` binary operator tests whether an object reference is an instance of a specified class, subclass, or interface.

```java
class Animal {}
class Dog extends Animal {}

Animal myPet = new Dog();
System.out.println(myPet instanceof Dog);    // true  (Actual runtime type)
System.out.println(myPet instanceof Animal); // true  (Superclass match)
System.out.println(myPet instanceof Object); // true  (Root object match)

// The Null Invariant:
System.out.println(null instanceof Dog);    // ALWAYS false! Never throws NPE.
```

#### 10.1 Pattern Matching for `instanceof` (Java 16+)

Eliminates repetitive, boiler-plate explicit casts:

```java
// Legacy Java approach:
if (myPet instanceof Dog) {
    Dog d = (Dog) myPet; // Manual cast required
    d.bark();
}

// Modern Java (Pattern Matching):
if (myPet instanceof Dog d) {
    d.bark(); // 'd' is automatically scoped and cast in one step!
}
```

---

### 11. Operator Precedence and Associativity

**Precedence** dictates the order in which operators are bound to operands in expressions. **Associativity** determines the evaluation direction (Left-to-Right vs. Right-to-Left) when operators share identical precedence.

```
                        Evaluation Precedence Ladder
                                 (Highest)
   ┌──────────────────────────────────────────────────────────────────┐
   │ 1. Postfix Operators        : expr++  expr--                     │
   │ 2. Unary Operators          : ++expr  --expr  +  -  ~  !  (cast) │
   │ 3. Multiplicative           : *  /  %                            │
   │ 4. Additive                 : +  -                               │
   │ 5. Shift Operators          : <<  >>  >>>                        │
   │ 6. Relational Operators     : <  >  <=  >=  instanceof           │
   │ 7. Equality Operators       : ==  !=                             │
   │ 8. Bitwise AND              : &                                  │
   │ 9. Bitwise XOR              : ^                                  │
   │ 10. Bitwise OR              : |                                  │
   │ 11. Logical AND             : &&                                 │
   │ 12. Logical OR              : ||                                 │
   │ 13. Ternary Operator        : ? :                                │
   │ 14. Assignment Operators    : =  +=  -=  *=  /=  %=  &=  etc.    │
   └──────────────────────────────────────────────────────────────────┘
                                 (Lowest)
```

#### Complete Master Precedence & Associativity Table

| Precedence Rank | Operator Group        | Specific Operators                                                         |   Associativity   |
| :-------------: | :-------------------- | :------------------------------------------------------------------------- | :---------------: |
| **1 (Highest)** | Primary / Postfix     | `()`, `[]`, `.`, `expr++`, `expr--`                                        |   Left-to-Right   |
|      **2**      | Unary Operators       | `++expr`, `--expr`, `+`, `-`, `~`, `!`, `(type)`                           | **Right-to-Left** |
|      **3**      | Multiplicative        | `*`, `/`, `%`                                                              |   Left-to-Right   |
|      **4**      | Additive              | `+`, `-`                                                                   |   Left-to-Right   |
|      **5**      | Shift                 | `<<`, `>>`, `>>>`                                                          |   Left-to-Right   |
|      **6**      | Relational            | `<`, `>`, `<=`, `>=`, `instanceof`                                         |   Left-to-Right   |
|      **7**      | Equality              | `==`, `!=`                                                                 |   Left-to-Right   |
|      **8**      | Bitwise AND           | `&`                                                                        |   Left-to-Right   |
|      **9**      | Bitwise XOR           | `^`                                                                        |   Left-to-Right   |
|     **10**      | Bitwise OR            | `\|`                                                                       |   Left-to-Right   |
|     **11**      | Logical AND           | `&&`                                                                       |   Left-to-Right   |
|     **12**      | Logical OR            | `\|\|`                                                                     |   Left-to-Right   |
|     **13**      | Ternary / Conditional | `? :`                                                                      | **Right-to-Left** |
| **14 (Lowest)** | Assignment            | `=`, `+=`, `-=`, `*=`, `/=`, `%=`, `&=`, `^=`, `\|=`, `<<=`, `>>=`, `>>>=` | **Right-to-Left** |

---

### 12. Type Promotion in Expressions

When evaluating arithmetic expressions containing mixed types, Java applies strict binary numeric promotion rules (JLS §5.6.2):

```
                        Numeric Promotion Ladder
                                   double
                                     ▲
                                     │ (Any operand is double -> double)
                                   float
                                     ▲
                                     │ (Any operand is float -> float)
                                    long
                                     ▲
                                     │ (Any operand is long -> long)
                                    int
                                     ▲
                  ┌──────────────────┴──────────────────┐
                  │ (Promoted unconditionally to int!)  │
                byte               short              char
```

1. **The Sub-Int Rule:** Operands of type `byte`, `short`, or `char` are **always promoted to `int` first**, even in standalone expressions with no other types present:
   ```java
   byte b1 = 10;
   byte b2 = 20;
   // byte b3 = b1 + b2; // ERROR: b1 + b2 produces an int!
   byte b3 = (byte)(b1 + b2); // OK: Explicit narrowing
   ```
2. If any operand is `double`, the other is promoted to `double`.
3. Otherwise, if any operand is `float`, the other is promoted to `float`.
4. Otherwise, if any operand is `long`, the other is promoted to `long`.
5. Otherwise, both operands are evaluated as `int`.

---

### 13. Expressions & Operand Evaluation Order

A critical distinction between C/C++ and Java:

- **In C/C++:** Order of operand evaluation is largely **unspecified** (undefined sequence points).
- **In Java:** The Java Language Specification strictly guarantees that **operands are evaluated from LEFT to RIGHT**, regardless of operator precedence!

```java
public class EvaluationOrderProof {
    public static void main(String[] args) {
        int a = 5;
        // Evaluation:
        // 1. Left operand 'a' is fetched -> 5
        // 2. Pre-increment '++a' evaluates -> increments a to 6, yields 6
        // 3. Multiplication '5 * 6' occurs -> 30
        int result = a * ++a;
        System.out.println("Result: " + result); // Prints: 30 (NOT 36!)
    }
}
```

```java
// Complex Multi-variable Tracing:
int x = 2;
int y = 4;
int z = x++ + ++y * x;
// Step-by-step resolution:
// 1. x++ is evaluated: loads 2 onto stack, memory x becomes 3
// 2. Expression is now: 2 + (++y * x)
// 3. ++y evaluates: increments y to 5, loads 5 onto stack
// 4. Current x is fetched: x is currently 3
// 5. Multiplicative precedence: 5 * 3 = 15
// 6. Addition: 2 + 15 = 17
// Final state: z = 17, x = 3, y = 5
```

---

### 14. High-Yield CDAC Exam Traps & Interview Questions

- **Q1: What will `System.out.println(10 >> 1);` and `System.out.println(-1 >> 1);` print?**
  - _Answer:_ `5` and `-1`.
  - _Explanation:_
    - `10 >> 1` divides 10 by $2^1 = 5$.
    - `-1` in 32-bit two's complement binary is all ones: `11111111 11111111 11111111 11111111`. Performing a signed right shift (`>>`) shifts bits right and fills vacated left slots with the sign bit (`1`). The resulting bit sequence remains entirely ones, which still equals `-1`.

- **Q2: What is the output of the following boolean expression?**

  ```java
  boolean flag = false;
  if (flag = false) {
      System.out.println("True branch");
  } else {
      System.out.println("False branch");
  }
  ```

  - _Answer:_ `False branch`
  - _Explanation:_ `flag = false` is an assignment expression, not a comparison. It assigns `false` to `flag` and evaluates to `false`. The `if` branch condition evaluates to `false`, executing the `else` block.

- **Q3: What does `System.out.println(~0);` output and why?**
  - _Answer:_ `-1`
  - _Explanation:_ The literal `0` in binary is all zeros (`00000000...00000000`). The bitwise complement operator (`~`) inverts every single bit to ones (`11111111...11111111`). In signed two's complement integer arithmetic, an all-ones bit pattern represents `-1`. Formula: `~N = -(N + 1)`.

- **Q4: Why does `byte b = 10; b = b * 2;` fail to compile, but `b *= 2;` succeeds?**
  - _Answer:_ In `b * 2`, the operand `b` is promoted to `int`, producing an `int` result. Assigning an `int` to a `byte` requires an explicit cast. Conversely, the compound assignment operator `b *= 2` includes an implicit cast specified by the language: `b = (byte)(b * 2)`.

- **Q5: What is the result of `Double.NaN == Double.NaN`?**
  - _Answer:_ `false`
  - _Explanation:_ According to IEEE 754 standards, `NaN` (Not-a-Number) is **not equal to anything, including itself**. To test whether a value is `NaN`, use `Double.isNaN(value)`.
