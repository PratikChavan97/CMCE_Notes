# Module 01: Core Java Platform & Language Fundamentals

## Unit 1.4: Control Statements and Arrays

---

### 1. Conditional Statements Architecture

Conditional statements alter the strictly linear sequence of execution by evaluating boolean expressions at runtime, causing control to branch based on the result.

```
                         Conditional Branching
                                  │
         ┌────────────────────────┴────────────────────────┐
         ▼                                                 ▼
if-else Decision Ladder                           switch Multi-Way Branch
- Boolean expressions only                        - Integral, Enum, String types
- Dynamic evaluated conditions                    - Jump tables (O(1) / O(log n))
- Linear evaluation flow (top-to-bottom)          - Direct label matching
```

---

### 2. The `if` Statement Family

Unlike C and C++, where numbers can evaluate to conditions (e.g., `if (1)`), Java mandates that **all conditional expressions evaluate strictly to the primitive `boolean` type**.

#### 2.1 Basic `if` and `if-else`

```java
// Formal Syntax:
if (booleanCondition) {
    // Statements executed if booleanCondition == true
} else {
    // Statements executed if booleanCondition == false
}
```

```
           [ booleanCondition ]
                 │
        ┌────────┴────────┐
   true │                 │ false
        ▼                 ▼
  [ if-block ]      [ else-block ]
        │                 │
        └────────┬────────┘
                 ▼
     [ Resume Execution ]
```

#### 2.2 Dangling-else Invariant

When `if` statements are nested without explicit curly braces `{ }`, an `else` clause binds to the **closest preceding unmatched `if` statement** within the same block:

```java
int x = 10, y = 5;

// Code without braces:
if (x > 5)
    if (y > 10)
        System.out.println("Path A");
else
    System.out.println("Path B"); // Binds to: if (y > 10), NOT if (x > 5)!

// Output: Nothing is printed!
// (Because x > 5 is true, but y > 10 is false, and the else belongs to y > 10).
```

#### 2.3 Nested `if` and `if-else-if` Ladder

```java
if (score >= 90) {
    grade = 'A';
} else if (score >= 80) { // Evaluated ONLY if previous conditions are false
    grade = 'B';
} else if (score >= 70) {
    grade = 'C';
} else {
    grade = 'F';          // Fallback if all conditions evaluate to false
}
```

---

### 3. The `switch` Statement

The `switch` statement provides a multi-way branch that transfers control to a matching `case` label based on an evaluated expression.

#### 3.1 Permitted Data Types for `switch` Expressions

| Standard      | Permitted Types                                                           | Disallowed Types                              |
| :------------ | :------------------------------------------------------------------------ | :-------------------------------------------- |
| **Java 1.0+** | `byte`, `short`, `char`, `int` (and their literal compile-time constants) | `long`, `float`, `double`, `boolean`          |
| **Java 5+**   | Wrapper classes (`Byte`, `Short`, `Character`, `Integer`), `enum` types   | Any non-wrapper object reference              |
| **Java 7+**   | `java.lang.String`                                                        | Floating-point primitives (`float`, `double`) |

> ⚠️ **The 64-Bit and Floating-Point Restriction:**
>
> - `long` is **prohibited** because the underlying bytecode instructions (`tableswitch`, `lookupswitch`) accept only 32-bit integer keys.
> - `float` and `double` are **prohibited** because IEEE 754 precision inaccuracies make exact equality checks unreliable.

#### 3.2 Switch Fall-Through Mechanics

Execution jumps directly to the matching `case` label. Unless terminated by an explicit `break` or `return`, control **falls through** and executes all subsequent statements across remaining labels unconditionally until a `break` or the closing brace `}` is encountered.

```java
int day = 2;
switch (day) {
    case 1:  System.out.print("Mon ");
    case 2:  System.out.print("Tue "); // Jumps here
    case 3:  System.out.print("Wed "); // Falls through!
    default: System.out.print("End");  // Falls through!
}
// Console Output: Tue Wed End
```

#### 3.3 Bytecode Internals: `tableswitch` vs. `lookupswitch`

The Java compiler converts `switch` blocks into one of two bytecode instructions based on the density of the case values:

```
                  Case Value Distribution
                             │
         ┌───────────────────┴───────────────────┐
         ▼                                       ▼
  Contiguous / Dense                      Sparse / Dispersed
  (e.g., 1, 2, 3, 4, 5)                  (e.g., 10, 500, 100000)
         │                                       │
         ▼                                       ▼
   tableswitch                             lookupswitch
  - Direct O(1) jump table                - Sorted key-offset pairs
  - Array index lookup                    - Binary search O(log n)
```

1. **`tableswitch`:** Used when case constants are continuous or close together. It generates a jump table indexed directly by key, executing in constant time **$\mathcal{O}(1)$**. Empty slots are filled with offsets to `default`.
2. **`lookupswitch`:** Used when case constants are widely dispersed. It stores a sorted table of keys and bytecode offsets, resolving targets via binary search in **$\mathcal{O}(\log n)$** time.

#### 3.4 Modern Switch Expressions (Java 14+)

Modern Java introduces arrow syntax (`->`) and the `yield` statement, which eliminate accidental fall-through and allow `switch` to return values directly:

```java
// Switch Expression with Arrow Syntax (No break needed; zero fall-through)
String quarter = switch (month) {
    case 1, 2, 3    -> "Q1";
    case 4, 5, 6    -> "Q2";
    case 7, 8, 9    -> "Q3";
    case 10, 11, 12 -> "Q4";
    default         -> {
        // Multi-line block yields value via 'yield'
        System.err.println("Invalid month index: " + month);
        yield "UNKNOWN";
    }
};
```

---

### 4. Looping Statements

Loops repeatedly execute a block of code while a controlling condition evaluates to `true`.

```
                        Looping Constructs
                                │
         ┌──────────────────────┴──────────────────────┐
         ▼                                             ▼
  Pre-Test Loops                                Post-Test Loop
  (Condition verified BEFORE entry)             (Body executed AT LEAST ONCE)
  ├── while loop                                └── do-while loop
  ├── traditional for loop
  └── enhanced for-each loop
```

#### 4.1 The `while` Loop

- **Mechanism:** Pre-test loop. The condition expression is evaluated before each iteration.
- **Invariant:** If the condition evaluates to `false` initially, the loop body **never executes**.

```java
int count = 1;
while (count <= 5) {
    System.out.print(count + " ");
    count++;
}
// Output: 1 2 3 4 5
```

#### 4.2 The `do-while` Loop

- **Mechanism:** Post-test loop. The condition is evaluated **after** the loop body executes.
- **Invariant:** The loop body is guaranteed to execute **at least once**, regardless of the initial truth value of the condition.
- **Syntax Rule:** The terminating condition must conclude with a semicolon (`;`).

```java
int status = 0;
do {
    System.out.println("Executed once unconditionally");
} while (status != 0); // Note terminating semicolon
```

#### 4.3 The Traditional `for` Loop

- **Structure:** Consolidates initialization, condition evaluation, and step modification into a single statement.

```java
for (initialization; condition; update) {
    // Loop body
}
```

```
             ┌────────────────────────┐
             │ 1. Initialization      │ (Executes ONCE upon loop entry)
             └───────────┬────────────┘
                         ▼
             ┌────────────────────────┐
   ┌────────►│ 2. Condition Check     │
   │         └───────────┬────────────┘
   │               true  │        │ false
   │                     ▼        └──────────────┐
   │         ┌────────────────────────┐          │
   │         │ 3. Body Execution      │          │
   │         └───────────┬────────────┘          │
   │                     ▼                       ▼
   │         ┌────────────────────────┐    ┌───────────┐
   └─────────┤ 4. Update / Increment  │    │ Loop Exit │
             └────────────────────────┘    └───────────┘
```

##### Evaluation Invariants:

1. All three sections are optional. `for (;;)` creates an **infinite loop**, equivalent to `while (true)`.
2. Multiple variables of the **same type** can be declared in the initialization section, separated by commas.
3. The update section can contain multiple comma-separated statements.

```java
for (int i = 0, j = 10; i < j; i++, j--) {
    System.out.println("i=" + i + ", j=" + j);
}
```

#### 4.4 The Enhanced `for` Loop (For-Each Loop)

Introduced in Java 5 to traverse elements of an array or any collection implementing `java.lang.Iterable` sequentially without managing an index variable.

```java
int[] numbers = {10, 20, 30, 40};
for (int val : numbers) {
    System.out.print(val + " ");
}
```

##### Under the Hood (Compiler Lowering):

The compiler desugars the enhanced `for` loop over arrays into an indexed loop at the bytecode level:

```java
// How the compiler rewrites the array loop:
int[] a = numbers;
int len = a.length;
for (int i = 0; i < len; i++) {
    int val = a[i];
    // Loop body
}
```

##### Limitations of Enhanced `for`:

- **Read-Only Traversal for Primitives:** Assigning a new value to the iteration variable (`val = 50`) modifies only the local iteration variable, **not** the underlying array element.
- **No Backward or Stepped Iteration:** Traversal is strictly forward by single steps.
- **No Index Tracking:** Does not expose the current index position.

---

### 5. Branching Statements: `break` and `continue`

```
  break Statement:                              continue Statement:
  ┌─────────────────────────┐                  ┌─────────────────────────┐
  │ while (condition) {     │                  │ for (int i = 0; ...) {  │
  │     if (trigger)        │                  │     if (skipCondition)  │
  │         break; ─────────┼──────┐           │         continue; ──────┼─────┐
  │     body();             │      │           │     body();             │     │
  │ }                       │      │           │ }                       │     │
  │ <───────────────────────┘      │           └─────────────────────────┘     │
  │ // Jumps directly here! ◄──────┘                 ▲                         │
  └─────────────────────────┘                        └─ Skips remainder of ────┘
                                                        current iteration!
```

#### 5.1 Unlabeled `break` and `continue`

- **`break`:** Terminates the innermost enclosing `switch`, `for`, `while`, or `do-while` statement immediately.
- **`continue`:** Halts the current iteration of the innermost loop and evaluates the loop's update/condition step next.

#### 5.2 Labeled `break` and `continue`

Java has no `goto` statement. To break or continue outer loops from deeply nested code, use **labeled blocks**:

```java
outerLoop: // Label identifier followed by colon
for (int i = 1; i <= 3; i++) {
    for (int j = 1; j <= 3; j++) {
        if (i == 2 && j == 2) {
            break outerLoop; // Terminates BOTH loops immediately!
        }
        System.out.println("i=" + i + ", j=" + j);
    }
}
```

---

### 6. One-Dimensional Arrays

An **array** is a dynamically allocated data structure that stores a fixed-size, contiguous sequence of elements of the **same data type**.

#### 6.1 Core Array Invariants in Java

1. **Arrays are Objects:** In Java, every array is a first-class object allocated dynamically on the **Heap**, even if its component elements are primitive types.
2. **Inheritance:** Array types inherit directly from `java.lang.Object` and implement `java.lang.Cloneable` and `java.io.Serializable`.
3. **Fixed Length:** Once instantiated, an array's size cannot be altered. The length is tracked by a public, final field: **`array.length`** (not a method).
4. **Boundary Checks:** Every array access is bounds-checked at runtime by the JVM. Accessing an index where $\text{index} < 0$ or $\text{index} \ge \text{length}$ throws a **`java.lang.ArrayIndexOutOfBoundsException`**.

#### 6.2 Declaration and Allocation Syntax

```java
// 1. Declaration (No memory allocated yet; reference sits on stack as null)
int[] arr1; // Preferred Java style
int arr2[]; // Legal C/C++ style

// 2. Instantiation (Heap memory allocated; elements zero-initialized)
arr1 = new int[5]; // Allocates contiguous heap block for 5 integers

// 3. Combined Declaration and Initialization
int[] primes = new int[]{2, 3, 5, 7, 11}; // Explicit syntax
int[] scores = {95, 88, 72, 90};           // Anonymous array literal shortcut
```

#### 6.3 Memory Layout: Primitives on the Heap

```java
int[] data = new int[4];
data[0] = 10;
data[1] = 20;
```

```
STACK MEMORY (Thread Frame)                        HEAP MEMORY (Runtime Space)
┌───────────────────────────────┐                  ┌───────────────────────────────────────────┐
│ Reference Variable:           │                  │ Array Object: int[4]                      │
│ data = 0x55AF2000             │ ───────────────► │ Address: 0x55AF2000                       │
└───────────────────────────────┘                  ├───────────────────────────────────────────┤
                                                   │ Mark Word (8 bytes)  - Lock & GC state    │
                                                   │ Klass Word (4 bytes) - Class metadata ref │
                                                   │ length = 4 (4 bytes) - Array bounds data  │
                                                   ├───────────────────────────────────────────┤
                                                   │ data[0] = 10 (4 bytes)                    │
                                                   │ data[1] = 20 (4 bytes)                    │
                                                   │ data[2] = 0  (4 bytes, default)           │
                                                   │ data[3] = 0  (4 bytes, default)           │
                                                   │ Padding (Zero-fill to 8-byte boundary)    │
                                                   └───────────────────────────────────────────┘
```

---

### 7. Two-Dimensional & Multidimensional Arrays

Java does not allocate multidimensional arrays as flat, contiguous matrices (like C/C++ row-major layouts). Instead, Java implements multidimensional arrays as **arrays of array references** ("ragged" or "jagged" pointer trees).

```
Two-Dimensional Array Memory Structure:

Matrix Reference (Stack)
┌──────────────┐
│ matrix ──────┼───► Heap: Master Array of References
└──────────────┘     ┌───────────────────────┐
                     │ matrix.length = 3     │
                     ├───────────────────────┤
                     │ [0] = 0xAA01 ─────────┼──► [ Heap Row 0: int[4] ]
                     │ [1] = 0xBB02 ─────────┼──► [ Heap Row 1: int[4] ]
                     │ [2] = 0xCC03 ─────────┼──► [ Heap Row 2: int[4] ]
                     └───────────────────────┘
```

#### 7.1 Rectangular Array Instantiation

```java
// Allocates a 3x4 rectangular grid
int[][] matrix = new int[3][4];

// Master array length:
System.out.println(matrix.length);    // 3 (Rows)
// Individual sub-array length:
System.out.println(matrix[0].length); // 4 (Columns in Row 0)
```

#### 7.2 Jagged (Ragged) Arrays

Because each row is an independent array on the heap, sub-arrays can have **different lengths**:

```java
// Declare master array with 3 rows, leaving column sizes undefined
int[][] jagged = new int[3][];

// Allocate individual rows with different capacities
jagged[0] = new int[2]; // Row 0 has 2 columns
jagged[1] = new int[4]; // Row 1 has 4 columns
jagged[2] = new int[1]; // Row 2 has 1 column

// Initialization via nested array literals:
int[][] irregular = {
    {1, 2},
    {3, 4, 5, 6},
    {7}
};
```

```
Jagged Array Memory Layout:
jagged
  │
  ▼
┌──────────────┐
│ length = 3   │
├──────────────┤
│ [0] ─────────┼──► [ 0, 0 ]             (Length: 2)
│ [1] ─────────┼──► [ 0, 0, 0, 0 ]       (Length: 4)
│ [2] ─────────┼──► [ 0 ]                (Length: 1)
└──────────────┘
```

---

### 8. Array Traversal Techniques

```java
int[][] matrix = {
    {10, 20, 30},
    {40, 50, 60}
};

// Technique 1: Standard Nested for-loop (Provides direct index tracking)
for (int i = 0; i < matrix.length; i++) {
    for (int j = 0; j < matrix[i].length; j++) {
        System.out.print(matrix[i][j] + " ");
    }
    System.out.println();
}

// Technique 2: Nested Enhanced for-each loop (Cleaner read-only access)
for (int[] row : matrix) {
    for (int cell : row) {
        System.out.print(cell + " ");
    }
    System.out.println();
}

// Technique 3: Deep Printing using Utility Classes
System.out.println(java.util.Arrays.deepToString(matrix));
// Output: [[10, 20, 30], [40, 50, 60]]
```

---

### 9. Array of Objects

Instantiating an array of objects creates an array of **reference slots** pointing to `null`. It does **not** instantiate the underlying objects themselves.

```java
class Student {
    int id;
    String name;
    Student(int id, String name) {
        this.id = id;
        this.name = name;
    }
}

public class ObjectArrayDemo {
    public static void main(String[] args) {
        // Step 1: Allocate the array of references on the Heap
        Student[] batch = new Student[3]; // All slots initially hold null!

        // Attempting to dereference batch[0].id here throws NullPointerException!

        // Step 2: Instantiate actual objects and assign references
        batch[0] = new Student(101, "Aarav");
        batch[1] = new Student(102, "Bhavna");
        batch[2] = new Student(103, "Chetan");

        for (Student s : batch) {
            System.out.println(s.id + " : " + s.name);
        }
    }
}
```

#### Memory Layout: Array of Objects

```
STACK MEMORY                                        HEAP MEMORY
┌───────────────────────────────┐                  ┌───────────────────────────────────────────┐
│ batch = 0x7700                │ ───────────────► │ Student[] Array Object: Address 0x7700    │
└───────────────────────────────┘                  ├───────────────────────────────────────────┤
                                                   │ length = 3                                │
                                                   │ [0] = 0x8801 ──┐                          │
                                                   │ [1] = 0x8802 ──┼──────────┐               │
                                                   │ [2] = null     │          │               │
                                                   └────────────────┼──────────┼───────────────┘
                                                                    ▼          ▼
                                                   ┌─────────────────────┐    ┌─────────────────────┐
                                                   │ Student Instance    │    │ Student Instance    │
                                                   │ Address: 0x8801     │    │ Address: 0x8802     │
                                                   ├─────────────────────┤    ├─────────────────────┤
                                                   │ id: 101             │    │ id: 102             │
                                                   │ name: -> "Aarav"    │    │ name: -> "Bhavna"   │
                                                   └─────────────────────┘    └─────────────────────┘
```

---

### 10. Core Array Operations (`java.util.Arrays`)

The `java.util.Arrays` utility class provides static helper methods for common array operations:

```java
import java.util.Arrays;

int[] arr = {5, 2, 8, 1, 9};

// 1. In-place Sorting (Dual-Pivot Quicksort for primitives: O(n log n))
Arrays.sort(arr); // {1, 2, 5, 8, 9}

// 2. Binary Search (Requires sorted array; returns index or negative insertion point)
int index = Arrays.binarySearch(arr, 5); // 2

// 3. String Representation
System.out.println(Arrays.toString(arr)); // "[1, 2, 5, 8, 9]"

// 4. Fill Memory
Arrays.fill(arr, -1); // {-1, -1, -1, -1, -1}

// 5. Array Copying & Resizing (Allocates fresh heap block)
int[] expanded = Arrays.copyOf(arr, 10); // Length expanded to 10; padded with zeros

// 6. High-Performance System Copy (Direct C-level memory block transfer)
int[] target = new int[5];
System.arraycopy(arr, 0, target, 0, arr.length);
```

---

### 11. High-Yield CDAC Exam Traps & Interview Questions

- **Q1: What is the output of the following array initialization?**

  ```java
  int[] arr = new int[0];
  System.out.println(arr.length);
  System.out.println(arr[0]);
  ```

  - _Answer:_ Prints `0`, followed by a runtime **`java.lang.ArrayIndexOutOfBoundsException: Index 0 out of bounds for length 0`**.
  - _Explanation:_ Creating an array of size zero (`new int[0]`) is legal in Java. Its `.length` property evaluates to `0`. However, because it contains no elements, accessing index `0` throws an out-of-bounds exception.

- **Q2: What is the output of this code snippet?**

  ```java
  int x = 0;
  if (x == 1) ;
  {
      System.out.println("Output A");
  }
  ```

  - _Answer:_ `Output A`
  - _Explanation:_ Notice the standalone semicolon `;` directly after `if (x == 1)`. This semicolon acts as an **empty statement**, which terminates the `if` construct immediately. The subsequent curly braces `{ ... }` are treated as an independent initialization block that executes unconditionally.

- **Q3: What happens when this array declaration executes?**

  ```java
  int[][] grid = new int[][4];
  ```

  - _Answer:_ **Compilation Error:** _']' expected_.
  - _Explanation:_ When instantiating a multidimensional array, the **dimension for the first (master) array must always be specified**. The subordinate column dimensions can be omitted (e.g., `new int[3][]`), but leaving the row dimension empty while providing a column dimension (`new int[][4]`) is illegal syntax.

- **Q4: What is the output of the following enhanced `for` loop?**

  ```java
  int[] values = {1, 2, 3};
  for (int v : values) {
      v = v * 10;
  }
  System.out.println(values[0]);
  ```

  - _Answer:_ `1`
  - _Explanation:_ The enhanced `for` loop copies primitive elements **by value** into the iteration variable `v`. Multiplying `v * 10` modifies only that local variable, leaving the original array stored on the heap untouched.

- **Q5: What are the differences between `arr.length` and `str.length()`?**
  - _Answer:_
    - `arr.length`: A **public final field** of an array object reflecting its fixed allocation capacity.
    - `str.length()`: A **public member method** of the `java.lang.String` class that computes the number of UTF-16 code units in the character sequence.
