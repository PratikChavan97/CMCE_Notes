# Module 01: Core Java Platform & Language Fundamentals

## Unit 1.2: Java Language Fundamentals

### 1. Structure of a Java Program

A Java source file (`.java`) follows a rigid structural order enforced by the compiler. Every compilation unit can contain the following declarative components in exact sequence:

```
+-------------------------------------------------------------------------+
| 1. Documentation Section    : Multi-line comments / Javadoc (/** ... */)|
| 2. Package Declaration      : package com.cdac.acts; (Max 1 per file)   |
| 3. Import Statements        : import java.util.*;    (0 or more)        |
| 4. Interface Declarations   : public interface Servicable { ... }       |
| 5. Class Definitions        : Primary class & non-public helper classes |
+-------------------------------------------------------------------------+
```

```java
/**
 * @file Application.java
 * @brief Entry point for inventory processing.
 */
package com.cdac.core;                 // 1. Package statement (Must be FIRST non-comment line)

import java.io.Serializable;          // 2. Explicit Class Import
import java.util.*;                   // 3. Wildcard Package Import
import static java.lang.Math.PI;      // 4. Static Import (Brings static members directly into scope)

public class Application implements Serializable { // 5. Public class matching filename
    public static void main(String[] args) {
        System.out.println("Inventory Engine Initialized. PI: " + PI);
    }
}

class HelperUtility {                 // 6. Package-private secondary class (Zero or more)
    // Non-public auxiliary definitions
}
```

#### Compilation & File-Naming Rules (CDAC Exam Critical):

1. **The Public Class Rule:** A single `.java` file can contain at most **one** `public` class. If a class is declared `public`, the filename **must match** that class name verbatim (including case sensitivity).
2. **Multiple Classes:** A `.java` file can define multiple classes without compilation errors, provided that **only one** (or none) is marked `public`.
3. **Compiler Output:** The compiler emits **one distinct `.class` file for every single class or interface** declared inside the source file:
   ```bash
   javac Application.java
   # Produces TWO separate binary files:
   # 1. Application.class
   # 2. HelperUtility.class
   ```

---

### 2. Java Class Structure

A Java class acts as a blueprint containing both static (class-level) and non-static (instance-level) members.

```
┌────────────────────────────────────────────────────────────────────────┐
│ Class: Account                                                         │
│                                                                        │
│ ┌────────────────────────────────────────────────────────────────────┐ │
│ │ 1. Static Initializer & Fields (<clinit>)                          │ │
│ │    - Shared across all instances, loaded once into Metaspace       │ │
│ └────────────────────────────────────────────────────────────────────┘ │
│ ┌────────────────────────────────────────────────────────────────────┐ │
│ │ 2. Instance Fields (State)                                         │ │
│ │    - Allocated on Heap per object instantiation                    │ │
│ └────────────────────────────────────────────────────────────────────┘ │
│ ┌────────────────────────────────────────────────────────────────────┐ │
│ │ 3. Instance Initializer Blocks (IIB) & Constructors (<init>)       │ │
│ │    - Executed sequentially per object allocation                   │ │
│ └────────────────────────────────────────────────────────────────────┘ │
│ ┌────────────────────────────────────────────────────────────────────┐ │
│ │ 4. Methods (Behavior)                                              │ │
│ │    - Instance methods (invoke via dynamic dispatch / invokevirtual)│ │
│ │    - Static methods   (invoke directly via invokestatic)           │ │
│ └────────────────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────────────┘
```

---

### 3. Deep-Dive: The `main()` Method

```java
public static void main(String[] args)
```

The standard entry method signature is heavily scrutinized in placement interviews:

```
  public           static          void            main          (String[] args)
  ──────           ──────          ────            ────          ───────────────
    │                │               │               │                  │
Accessible to   Called before    Returns zero   Fixed method   Array of CLI arguments
JVM runtime     instantiating    data to caller identifier     passed from OS shell
from any scope  class objects    environment    searched by JVM
```

#### 3.1 Parameter Variations & Semantic Rules:

1. **Array Notation Flexibility:** All of the following are 100% syntactically legal:
   ```java
   public static void main(String[] args)    // Canonical form
   public static void main(String args[])    // C/C++ bracket style
   public static void main(String []args)    // Alternative bracket spacing
   public static void main(String... args)   // Varargs syntax (C++11/Java 5+)
   ```
2. **Access Modifier Reordering:**
   ```java
   static public void main(String[] args)    // Valid: specifier order does not matter
   ```
3. **Optional Modifiers:**
   ```java
   public static final void main(String[] args)        // Valid: prevents child overriding
   public static synchronized void main(String[] args) // Valid: thread acquires class lock
   public static strictfp void main(String[] args)     // Valid: forces strict IEEE 754 math
   ```
4. **Invalid Signatures (Compiles, but JVM will reject execution with `NoSuchMethodError`):**
   - `public void main(String[] args)` (Missing `static`)
   - `static void main(String[] args)` (Not `public`)
   - `public static int main(String[] args)` (Non-void return type)
   - `public static void Main(String[] args)` (Capitalized 'M' — Java is case-sensitive)

---

### 4. Java Tokens

A **token** is the smallest individual building block of a program that the compiler recognizes during the lexical analysis phase.

```
                                Java Tokens
                                     │
    ┌──────────────┬─────────────────┼─────────────────┬──────────────┐
    ▼              ▼                 ▼                 ▼              ▼
Keywords      Identifiers         Literals         Operators      Separators
(e.g., class,  (e.g., totalSum,   (e.g., 42,       (e.g., +, &&,  (e.g., ;, {},
 while, int)    EmployeeRecord)    "CDAC", true)    instanceof)    [], (), .)
```

- **Separators / Punctuators:**
  - `{}`: Defines code blocks, scopes, classes, and array literals.
  - `()`: Contains parameter lists, control flow expressions, and type casts.
  - `[]`: Declares arrays and dereferences index positions.
  - `;`: Statement terminator.
  - `,`: Separates parameter declarations and variable definitions.
  - `.`: Package qualifier and member-access selector.
  - `::`: Method reference delimiter (Java 8+).

---

### 5. Keywords and Identifiers

#### 5.1 Keywords

Java reserves **67 keywords** (including modern restricted keywords and contextual keywords). All keywords are composed strictly of **lowercase** characters.

```
                        Categorized Standard Keywords
+------------------+-----------------------------------------------------------+
| Category         | Keywords                                                  |
+------------------+-----------------------------------------------------------+
| Primitive Types  | byte, short, int, long, char, float, double, boolean, void|
| Flow Control     | if, else, switch, case, default, while, do, for, break,   |
|                  | continue, return, yield (Java 14+)                        |
| Exception Handling| try, catch, finally, throw, throws, assert                |
| Modifiers        | public, protected, private, static, final, abstract,      |
|                  | synchronized, volatile, transient, native, strictfp       |
| Class/OOP        | class, interface, enum, extends, implements, package,     |
|                  | import, this, super, new, instanceof, record (Java 16+)   |
| Unused/Reserved  | goto, const (Compile error if used in code)               |
| Literals (Values)| true, false, null (Technically reserved literals, not kws)|
+------------------+-----------------------------------------------------------+
```

#### 5.2 Identifiers: Rules & Invariants

An identifier is a programmer-defined name assigned to variables, methods, classes, and packages.

##### Strict Lexical Rules (Enforced by Compiler):

1. **Permitted Characters:** Can contain Unicode letters, digits (`0-9`), the dollar sign (`$`), and the underscore (`_`).
2. **First Character Rule:** **Must NOT begin with a digit**. (`1total` is illegal; `total1` is legal).
3. **No Reserved Words:** Cannot be a keyword, reserved literal (`null`, `true`, `false`).
4. **No Whitespace:** Cannot contain spaces, tabs, or hyphens.
5. **Length:** Conceptually unlimited length.

```java
// Legal Identifiers:
int $salary = 50000;
int _counter = 0;
int total_count$ = 10;
int valid123Variable = 99;

// Illegal Identifiers (Compilation Errors):
int 123number;       // Error: Starts with numeric digit
int class;           // Error: 'class' is a reserved keyword
int employee-id;     // Error: Hyphen parsed as subtraction operator
int user balance;    // Error: Contains whitespace
```

> ⚠️ **Modern Java Note (`_` as an identifier):**
> Starting in **Java 9**, using a single underscore `_` as a standalone identifier is a **compile-time error** (reserved for future unnamed parameters/variables in Java 22+).

---

### 6. Variables and Constants

A variable is an abstraction for a named memory storage cell whose value can fluctuate during runtime execution.

#### 6.1 Variable Classifications

```
                                Variable Taxonomy
                                        │
         ┌──────────────────────────────┼──────────────────────────────┐
         ▼                              ▼                              ▼
  Local Variables                Instance Variables             Class Variables
  (Stack Frame / LVA)           (Heap / Inside Object)         (Metaspace / Static)
  - Scope: Block/Method          - Scope: Lifetime of object    - Scope: Entire Application
  - Initialization: MANDATORY    - Default values applied       - Default values applied
```

| Trait                 | Local Variable                      | Instance Variable                                                  | Static (Class) Variable        |
| :-------------------- | :---------------------------------- | :----------------------------------------------------------------- | :----------------------------- |
| **Physical Location** | Stack Frame (LVA)                   | Heap (Inside Object)                                               | Metaspace (Static Class Area)  |
| **Created When**      | Method/block is entered             | Object is instantiated (`new`)                                     | Class is loaded by ClassLoader |
| **Destroyed When**    | Method/block exits                  | Object is Garbage Collected                                        | Class is unloaded              |
| **Default Value**     | **None** (Must initialize manually) | **Yes** (Zero, false, or null)                                     | **Yes** (Zero, false, or null) |
| **Access Modifiers**  | Only `final` permitted              | `public`, `protected`, `private`, `final`, `transient`, `volatile` | Same as instance + `static`    |

```java
public class VariableScopeTracker {
    static int globalTracker = 100; // Static variable (Metaspace)
    int instanceData = 200;          // Instance variable (Heap)

    public void execute(int parameterVal) { // parameterVal is local
        int localCalculation;                // Uninitialized local variable

        // System.out.println(localCalculation);
        // COMPILE ERROR: variable localCalculation might not have been initialized

        localCalculation = parameterVal + instanceData;
        System.out.println(localCalculation); // Valid after assignment
    }
}
```

#### 6.2 Constants (`final` Variables)

In Java, constants are declared using the **`final`** keyword. Once assigned, a `final` variable's value cannot be modified.

```java
public class ConstantsRegistry {
    // Compile-time Constant (Inlined directly into bytecode during compilation)
    public static final double SALES_TAX_RATE = 0.18;

    // Blank Final Variable (Must be explicitly initialized in EVERY constructor)
    private final long timestamp;

    public ConstantsRegistry(long time) {
        this.timestamp = time; // Successfully initialized
        // this.timestamp = 0; // COMPILE ERROR: cannot assign a value to final variable timestamp
    }
}
```

---

### 7. Primitive Data Types

Java defines exactly **8 primitive data types**. Unlike C/C++, their bit-widths, numeric ranges, and byte layouts are strictly standardized and invariant across all CPU platforms.

```
                           8 Primitive Data Types
                                     │
         ┌───────────────────────────┴───────────────────────────┐
         ▼                                                       ▼
   Numeric Types                                         Non-Numeric Types
         │                                                       │
   ┌─────┴───────────────┐                             ┌─────────┴─────────┐
   ▼                     ▼                             ▼                   ▼
Integers           Floating-Point                   Character           Boolean
(byte, short,      (float, double)                   (char)             (boolean)
 int, long)
```

#### 7.1 Detailed Primitive Specifications

| Type          | Bits    | Bytes              | Default    | Range / Formula                                                             | Internal Representation              |
| :------------ | :------ | :----------------- | :--------- | :-------------------------------------------------------------------------- | :----------------------------------- |
| **`byte`**    | 8       | 1                  | `0`        | $-2^7$ to $2^7 - 1$ ($-128$ to $127$)                                       | Signed 2's complement                |
| **`short`**   | 16      | 2                  | `0`        | $-2^{15}$ to $2^{15} - 1$ ($-32,768$ to $32,767$)                           | Signed 2's complement                |
| **`int`**     | 32      | 4                  | `0`        | $-2^{31}$ to $2^{31} - 1$ ($-2.14 \times 10^9$ to $2.14 \times 10^9$)       | Signed 2's complement                |
| **`long`**    | 64      | 8                  | `0L`       | $-2^{63}$ to $2^{63} - 1$ ($-9.22 \times 10^{18}$ to $9.22 \times 10^{18}$) | Signed 2's complement                |
| **`float`**   | 32      | 4                  | `0.0f`     | $\pm 1.4 \times 10^{-45}$ to $\pm 3.4028235 \times 10^{38}$                 | IEEE 754 Single Precision            |
| **`double`**  | 64      | 8                  | `0.0d`     | $\pm 4.9 \times 10^{-324}$ to $\pm 1.7976931348623157 \times 10^{308}$      | IEEE 754 Double Precision            |
| **`char`**    | 16      | 2                  | `'\u0000'` | $0$ to $2^{16} - 1$ ($0$ to $65,535$)                                       | **Unsigned** UTF-16 Code Point       |
| **`boolean`** | JVM-dep | 1 (or 4 in arrays) | `false`    | `{true, false}`                                                             | Evaluated as `int` (1/0) by bytecode |

```java
// Literal Formats:
int decimal       = 100;
int binary        = 0b1100100;   // Prefix '0b' (Java 7+)
int octal         = 0144;        // Prefix '0'
int hexadecimal   = 0x64;        // Prefix '0x'
long largeVal     = 9876543210L; // Suffix 'L' mandatory for out-of-int-range values
float precision   = 3.14159f;    // Suffix 'f' or 'F' mandatory; default floating literal is double!
double highPrec   = 3.14159;     // Default double literal

// Underscores in Numeric Literals (Java 7+ readability):
int creditCardNumber = 4111_2222_3333_4444;
```

---

### 8. Reference Data Types

A **Reference Type** stores the memory handle (memory address) referencing an object or array allocated on the Heap.

```
STACK MEMORY (Thread Frame)                        HEAP MEMORY (Runtime Space)
┌───────────────────────────────┐                  ┌───────────────────────────────┐
│ Reference Variable:           │                  │ Actual Object Instance:       │
│ String str = 0x7FFF12A0       │ ───────────────► │ "CDAC ACTS"                   │
├───────────────────────────────┤                  ├───────────────────────────────┤
│ int[] scores = 0x7FFF99BC     │ ───────────────► │ [ 95, 88, 72, 100 ]           │
└───────────────────────────────┘                  └───────────────────────────────┘
```

#### 8.1 Distinct Categories of Reference Types:

1. **Class Types:** `String`, `Scanner`, custom domain classes.
2. **Array Types:** `int[]`, `String[][]`, `Object[]`. (All arrays in Java are true objects residing on the Heap).
3. **Interface Types:** `List`, `Comparable`, `Runnable` (Hold references to instances implementing the interface contract).
4. **Enum Types:** Specialized class types containing a fixed set of instances.
5. **Record Types (Java 16+):** Transparent, immutable data carriers.

#### 8.2 The `null` Literal:

Any reference variable can point to **`null`**, indicating it currently references no physical address on the Heap. Attempting to dereference a `null` handle (`.` or `[]`) triggers an immediate runtime **`NullPointerException` (NPE)**.

---

### 9. Default Values Matrix

Only **fields** (class static variables and instance variables) receive automatic default values when their containing structure is prepared or instantiated. Local variables do not.

| Data Type                                           | Field Default Value                          |
| :-------------------------------------------------- | :------------------------------------------- |
| `byte`, `short`, `int`                              | `0`                                          |
| `long`                                              | `0L`                                         |
| `float`                                             | `0.0f`                                       |
| `double`                                            | `0.0d`                                       |
| `char`                                              | `'\u0000'` (Null Character, integer value 0) |
| `boolean`                                           | `false`                                      |
| **Any Reference Type** (`String`, `Object`, arrays) | **`null`**                                   |

---

### 10. Type Conversion, Casting, and Compatibility

Java is a **strongly typed** language. The compiler verifies every variable assignment for type compatibility. Conversions fall into two categories:

```
                            Type Conversions
                                   │
         ┌─────────────────────────┴─────────────────────────┐
         ▼                                                   ▼
Widening Conversion (Implicit)                      Narrowing Conversion (Explicit)
- Automatically performed by compiler               - Requires explicit cast: (TargetType)
- Zero data truncation risk                         - High risk: bit truncation / precision loss
- Direct path to larger capacity type               - Squeezes larger bit-width into smaller
```

#### 10.1 Widening Conversion (Implicit / Automatic)

Occurs when converting from a smaller bit-width compatible type to a larger target type:

$$\text{byte} \longrightarrow \text{short} \longrightarrow \text{int} \longrightarrow \text{long} \longrightarrow \text{float} \longrightarrow \text{double}$$
$$\text{char} \longrightarrow \text{int}$$

```java
int intVal = 1000;
long longVal = intVal;       // Implicit widening: 32-bit to 64-bit
float floatVal = longVal;    // Implicit widening: 64-bit int to 32-bit float (Note: possible precision loss)
double doubleVal = floatVal; // Implicit widening: 32-bit float to 64-bit double
```

> ⚠️ **The Precision Loss Edge Case:**
> Converting `long` (64 bits) to `float` (32 bits) is an **implicit widening conversion** because float's magnitude range ($\approx 10^{38}$) exceeds long's ($\approx 10^{18}$). However, because a 32-bit float reserves only 23 bits for the mantissa (significand), converting large `long` values to `float` may result in a **loss of lower-order precision bits**, even though it compiles without explicit casting!

#### 10.2 Narrowing Conversion (Explicit Casting)

Occurs when moving from a larger storage capacity type to a smaller target type. Requires the explicit cast operator: `(TargetType)`.

```
Narrowing 32-bit int to 8-bit byte:
Original int value: 130
Binary Representation (32 bits):
00000000 00000000 00000000 10000010

Casting to byte: (Takes ONLY lower 8 bits)
                        [ 10000010 ]
Result in Signed Two's Complement:
Sign Bit = 1 (Negative)
Magnitude = Invert bits + 1 -> 01111101 + 1 = 01111110 = 126
Evaluates to: -126 !
```

```java
int sourceVal = 130;
byte truncatedByte = (byte) sourceVal;
System.out.println(truncatedByte); // Prints: -126 (Bit overflow!)

double preciseVal = 99.987;
int flooredInt = (int) preciseVal;
System.out.println(flooredInt);    // Prints: 99 (Fractional digits truncated!)
```

#### 10.3 Automatic Type Promotion in Expressions

When evaluating arithmetic expressions, Java automatically promotes operand types according to strict hierarchy rules:

1. All `byte`, `short`, and `char` operands are **promoted to `int` first**, even before mathematical evaluation begins.
2. If any operand is `double`, the entire expression resolves to `double`.
3. Otherwise, if any operand is `float`, the entire expression resolves to `float`.
4. Otherwise, if any operand is `long`, the entire expression resolves to `long`.

```java
byte b1 = 10;
byte b2 = 20;
// byte b3 = b1 + b2; // COMPILE ERROR: b1 and b2 promote to int! Output is int.
byte b3 = (byte)(b1 + b2); // OK: Explicit cast

char ch = 'A'; // Unicode 65
int nextVal = ch + 1; // Evaluates to 66 (int)
```

---

### 11. Introduction to Wrapper Classes

The 8 primitive types do not inherit from `java.lang.Object`. To integrate them into object-oriented frameworks (such as Java Collections like `ArrayList<T>`, which operate strictly on objects), Java provides a **Wrapper Class** in the `java.lang` package for every primitive.

```
                           Primitive Wrapper Mappings
                                       │
         ┌─────────────────────────────┼─────────────────────────────┐
         ▼                             ▼                             ▼
  Primitive Type                 Wrapper Class               Direct Superclass
  byte                           Byte                        java.lang.Number
  short                          Short                       java.lang.Number
  int                            Integer                     java.lang.Number
  long                           Long                        java.lang.Number
  float                          Float                       java.lang.Number
  double                         Double                      java.lang.Number
  char                           Character                   java.lang.Object
  boolean                        Boolean                     java.lang.Object
```

#### 11.1 Autoboxing and Unboxing (Java 5+)

- **Autoboxing:** Automatic conversion of a primitive type into its corresponding wrapper object:
  ```java
  Integer boxed = 100; // Compiler rewrites to: Integer.valueOf(100);
  ```
- **Unboxing:** Automatic extraction of the primitive value from a wrapper object:
  ```java
  int unboxed = boxed; // Compiler rewrites to: boxed.intValue();
  ```

#### 11.2 The Integer Cache Trap (High-Yield Interview Trap):

For memory optimization, Java pre-allocates an internal cache of `Integer` objects for the range **$-128$ to $+127$**.

```java
Integer a = 100;
Integer b = 100;
System.out.println(a == b); // TRUE: Both point to identical cached instance!

Integer x = 200;
Integer y = 200;
System.out.println(x == y); // FALSE: Values exceed cache threshold!
                            // Distinct heap objects allocated; reference comparison fails!
System.out.println(x.equals(y)); // TRUE: Checks actual value equality
```

---

### 12. Standard Java Naming Conventions

Adhering to standard camelCase conventions is mandatory in production Java environments and strictly checked on evaluation rubrics.

| Identifier Type          | Convention / Style                                                                  | Examples                                                   |
| :----------------------- | :---------------------------------------------------------------------------------- | :--------------------------------------------------------- |
| **Classes & Interfaces** | **PascalCase** (UpperCamelCase); nouns for classes, adjectives/nouns for interfaces | `StudentManager`, `HttpParser`, `Runnable`, `Serializable` |
| **Methods**              | **camelCase** (LowerCamelCase); verbs or verb-phrases                               | `calculateTax()`, `fetchDatabaseRecord()`, `getBalance()`  |
| **Variables**            | **camelCase**; descriptive nouns                                                    | `studentAge`, `accountBalance`, `payloadBytes`             |
| **Constants**            | **UPPER_SNAKE_CASE**; all uppercase with underscores                                | `MAX_CAPACITY`, `DEFAULT_TIMEOUT_MS`, `PI`                 |
| **Packages**             | **All lowercase**; reverse domain naming scheme                                     | `com.oracle.net`, `org.apache.kafka`, `com.cdac.acts`      |
| **Type Parameters**      | **Single uppercase letter**                                                         | `T` (Type), `E` (Element), `K` (Key), `V` (Value)          |

---

### 13. High-Yield CDAC Exam Traps & Interview Questions

- **Q1: What will be the output of executing `System.out.println(10 + 20 + "CDAC" + 30 + 40);`?**
  - _Answer:_ `30CDAC3040`
  - _Explanation:_ Operations evaluate **left-to-right**. `10 + 20` involves numeric integers, evaluating to `30`. Next, `30 + "CDAC"` encounters a `String`, promoting the operation to string concatenation (`"30CDAC"`). For all remaining operations, `"30CDAC" + 30` and then `+ 40` continue as string concatenation, yielding `"30CDAC3040"`.

- **Q2: Does `short s = 5; s = s + 5;` compile? What about `s += 5;`?**
  - _Answer:_
    - `s = s + 5;` **fails compilation**. Inside `s + 5`, the `short` variable `s` is automatically promoted to an `int`. The resulting expression evaluates to an `int`, which cannot be assigned back to a `short` without an explicit cast: `s = (short)(s + 5);`.
    - `s += 5;` **compiles successfully**. Compound assignment operators (`+=`, `-=`, `*=`, `/=`) contain an **implicit cast** defined by the Java Language Specification: $E_1 \text{ op}= E_2$ is equivalent to $E_1 = (T)(E_1 \text{ op } E_2)$.

- **Q3: What occurs when `char ch = -1;` is compiled?**
  - _Answer:_ **Compilation Error:** _incompatible types: possible lossy conversion from int to char_. The `char` type in Java is strictly **unsigned** (range: $0$ to $65,535$). Negative literal integers cannot be assigned to `char` without an explicit cast: `(char)-1`.

- **Q4: Can a local variable be declared with the `static` keyword?**
  - _Answer:_ **No.** The `static` modifier specifies that a member belongs to the class metadata. Local variables are bound exclusively to the lifecycle of their local stack frame; declaring a local variable `static` results in a compilation error.

- **Q5: What is the output of the following boolean evaluation?**
  ```java
  boolean status = false;
  if (status = true) {
      System.out.println("Condition Met");
  } else {
      System.out.println("Failed");
  }
  ```

  - _Answer:_ `Condition Met`
  - _Explanation:_ The statement uses the assignment operator (`=`), **not** the equality comparator (`==`). The expression `status = true` assigns `true` to `status` and resolves to the boolean literal `true`. The `if` branch condition evaluates to `true` and executes its block.
