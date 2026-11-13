# Module 04: Exception Handling, Strings, and Standard APIs

## Unit 4.1: String Fundamentals

### 1. Introduction to the `String` Class

In Java, a string is a sequence of characters. Unlike C/C++—where strings are null-terminated character arrays (`char[]` terminating with `'\0'`) manipulated via raw memory pointers—Java implements strings as **first-class objects** encapsulated by the **`java.lang.String`** class.

```
C / C++ Model (Raw Mutable Buffer):
char str[] = {'C', 'D', 'A', 'C', '\0'}; // Flat array, unchecked pointer arithmetic

Java Model (Encapsulated Heap Object):
String str = "CDAC"; // Instance of java.lang.String on Heap backed by SCP
```

#### 1.1 Structural Declaration of `java.lang.String`

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence
```

- **`final` Class:** `String` cannot be extended. This prevents subclasses from overriding string behaviors or breaking immutability guarantees.
- **Interfaces Implemented:**
  - `Serializable`: Allows string instances to be serialized across network streams or written to disk.
  - `Comparable<String>`: Provides natural lexicographical sorting via `compareTo()`.
  - `CharSequence`: Standard read-only interface implemented by `String`, `StringBuilder`, and `StringBuffer`.

#### 1.2 Internal Storage Representation (Compact Strings - Java 9+)

Prior to Java 9, `String` stored characters internally in a UTF-16 character array:

```java
// Java 8 and earlier:
private final char[] value; // Every character consumes strictly 2 bytes (16 bits)
```

Because most applications use predominantly Latin-1 characters (which require only 1 byte per character), half of the heap memory allocated to strings was filled with zero bytes.

Starting with **Java 9**, Java adopted **Compact Strings**:

```java
// Java 9 and later:
private final byte[] value; // Encoded dynamically as 1 byte or 2 bytes
private final byte coder;   // 0 = LATIN1 (1 byte/char), 1 = UTF16 (2 bytes/char)
```

The JVM inspects character contents at runtime:

- If all characters fit into ISO-8859-1 / Latin-1 ($0 \text{ to } 255$), `coder` is set to `0`, consuming **1 byte per character**.
- If a single character requires UTF-16 ($> 255$, such as Devanagari, emojis, or CJK scripts), `coder` is set to `1`, consuming **2 bytes per character**.

---

### 2. String Creation: String Literals vs. `new String()`

Java provides two fundamentally distinct ways to instantiate a `String`:

```
                           String Instantiation Paths
                                       │
         ┌─────────────────────────────┴─────────────────────────────┐
         ▼                                                           ▼
String Literal Syntax                                         new String() Operator
String s = "CDAC";                                            String s = new String("CDAC");
- Stored in String Constant Pool (SCP)                        - Stored in general Heap memory
- Reuses existing instances                                   - Bypasses SCP reuse (forces new object)
- 0 or 1 object created                                       - 1 or 2 objects created
```

```java
// Method 1: String Literal
String s1 = "Java";
String s2 = "Java"; // Does NOT create a new object; reuses s1 from SCP

// Method 2: new Operator
String s3 = new String("Java"); // Forces a new object on general Heap
```

---

### 3. String Immutability

Once a `java.lang.String` instance is constructed on the Heap, its character sequence, length, and hash code **can never be modified**.

```java
String s = "Java";
s.concat(" SE"); // Attempts to append " SE"

System.out.println(s); // Prints "Java" (Original string remains unmodified!)
```

```
s.concat(" SE") Memory Lifecycle:
Step 1: "Java" remains intact at 0x1000 in SCP
Step 2: A BRAND NEW String object "Java SE" is allocated at 0x2000 on the Heap
Step 3: Because the return value was not reassigned, 0x2000 is orphaned and eligible for GC!

To capture the result:
s = s.concat(" SE"); // s now points to 0x2000 ("Java SE")
```

#### 3.1 Why Is `String` Designed as Immutable? (High-Yield Interview Topic)

1. **String Constant Pool (SCP) Optimization:** If strings were mutable, modifying a string through one reference would silently corrupt all other variables pointing to that same pooled literal.
2. **Thread Safety:** Immutable objects are inherently thread-safe. Multiple threads can read shared strings concurrently without synchronization locks or race conditions.
3. **Security (Class Loading & Networking):** Sensitive parameters—such as database URLs, usernames, passwords, file paths, and network socket addresses—are passed as strings. If strings were mutable, a malicious thread could alter a verified file path after security checks ran (**Time-of-Check to Time-of-Use / TOCTOU vulnerability**).
4. **HashCode Caching:** The hash code of a `String` is computed once and cached inside a private field (`private int hash`). Because the content never changes, collection lookups in `HashMap<String, V>` and `HashSet<String>` run in $\mathcal{O}(1)$ without recalculating hashes.

---

### 4. The String Constant Pool (SCP)

The **String Constant Pool (SCP)** is a dedicated storage region inside the JVM Heap managed by the runtime to optimize memory consumption via string deduplication.

```
JVM Memory Evolution:
Java 6 and Earlier:
  [ Managed Heap ]                 [ PermGen (Fixed Size, Native/Meta) ]
                                    └── String Constant Pool (Prone to OutOfMemoryError: PermGen)

Java 7 and Later:
  [ Managed JVM Heap (-Xmx) ]
   ├── Young Generation (Eden, S0, S1)
   ├── Old (Tenured) Generation
   └── String Constant Pool (Garbage collected just like normal heap objects!)
```

#### 4.1 How the SCP Functions

When the JVM encounters a string literal:

1. It queries the internal SCP hash table.
2. If an identical character sequence already exists in the pool, it **returns the address of the existing instance**.
3. If the sequence is absent, it creates a new `String` instance inside the SCP and returns its reference.

---

### 5. Memory Map: Literals vs. `new String()`

Consider the following allocations:

```java
String s1 = "Hello";
String s2 = "Hello";
String s3 = new String("Hello");
String s4 = new String("Hello");
```

```
STACK MEMORY (Frame)                               HEAP MEMORY
┌───────────────────────────┐                      ┌───────────────────────────────────────────┐
│ s1: 0x5000 ───────────────┼─────────┐            │ String Constant Pool (SCP)                │
│ s2: 0x5000 ───────────────┼─────────┤            │ ┌───────────────────────────────────────┐ │
│                           │         └───────────►│ │ 0x5000: String Object ("Hello")       │ │
│                           │                      │ └───────────────────▲───────────────────┘ │
│                           │                      └─────────────────────┼─────────────────────┘
│                           │                      General Heap Space    │
│                           │                      ┌─────────────────────┼───────────────────┐
│ s3: 0x8100 ───────────────┼─────────────────────►│ 0x8100: String Object (points to 0x5000)│
│                           │                      ├─────────────────────────────────────────┤
│ s4: 0x9200 ───────────────┼─────────────────────►│ 0x9200: String Object (points to 0x5000)│
└───────────────────────────┘                      └─────────────────────────────────────────┘
```

#### Object Count Breakdown:

- `String s1 = "Hello";` $\rightarrow$ **1 object** created in the SCP (if not already present).
- `String s2 = "Hello";` $\rightarrow$ **0 objects** created (reuses `0x5000` from the SCP).
- `String s3 = new String("Hello");` $\rightarrow$ **1 object** created in general Heap memory (`0x8100`). The `"Hello"` literal was already in the SCP from line 1.
- `String s4 = new String("World");` (Assuming `"World"` did not exist before) $\rightarrow$ **2 objects** created: one in the SCP and one on the general Heap.

---

### 6. String Comparison: `==` vs. `.equals()`

```
                                String Comparison
                                        │
         ┌──────────────────────────────┴──────────────────────────────┐
         ▼                                                             ▼
== Operator (Reference Comparison)                           .equals() (Content Comparison)
- Checks whether both variables point to                      - Compares actual character sequences
  the exact same heap memory address                          - Overridden from Object to check deep equality
```

```java
String a = "Test";
String b = "Test";
String c = new String("Test");

System.out.println(a == b);      // true  (Both reference identical SCP address)
System.out.println(a == c);      // false (a is in SCP; c is a separate Heap object)
System.out.println(a.equals(c)); // true  (Character contents are identical)
```

#### 6.1 `compareTo()` Method (Lexicographical Comparison)

Defined by `Comparable<String>`:

$$\text{result} = s_1.\text{compareTo}(s_2)$$

- Returns `0` if $s_1.\text{equals}(s_2)$.
- Returns a **negative integer** if $s_1$ precedes $s_2$ alphabetically.
- Returns a **positive integer** if $s_1$ succeeds $s_2$ alphabetically.
- Evaluates characters by their Unicode code point values ($s_1[i] - s_2[i]$ at the first differing index, or $\text{length}(s_1) - \text{length}(s_2)$ if one is a prefix of the other).

```java
System.out.println("Apple".compareTo("Banana")); // -1 ('A' is 65, 'B' is 66 -> 65 - 66)
System.out.println("Java".compareTo("Java"));     //  0
```

---

### 7. String Interning: `intern()` Method

The `public String intern()` method manually searches the String Constant Pool for an equivalent string:

- If the SCP already contains a string equal to this `String` object (per `equals()`), it returns the reference from the pool.
- If not, this string object is added to the SCP, and a reference to it is returned.

```java
String heapString = new String("CDAC"); // Allocated on general Heap
String pooledString = heapString.intern(); // Fetches canonical reference from SCP
String literalString = "CDAC";            // Direct SCP reference

System.out.println(heapString == literalString);   // false
System.out.println(pooledString == literalString); // true! Both share the SCP address
```

---

### 8. String Concatenation Mechanics

#### 8.1 Compile-Time vs. Runtime Concatenation

The Java compiler optimizes string concatenation differently depending on whether operands are **compile-time constants**:

```java
// Case 1: Compile-Time Constant Folding
String s1 = "CDAC" + " ACTS"; // Operands are string literals
// javac replaces this directly in bytecode with:
// String s1 = "CDAC ACTS"; (Evaluated at compile-time, placed directly into SCP)

// Case 2: Runtime Dynamic Concatenation
String part1 = "CDAC";
String part2 = " ACTS";
String s2 = part1 + part2; // Operands are variables!
// Evaluated at RUNTIME on the general Heap (NOT folded into SCP)

String s3 = "CDAC ACTS";
System.out.println(s1 == s3); // true  (Both resolve to the same SCP literal)
System.out.println(s2 == s3); // false (s2 was built at runtime on the Heap!)
```

#### 8.2 Compiler Lowering of the `+` Operator

- **Java 8 and earlier:** The `+` operator on string variables was desugared by `javac` into:
  ```java
  new StringBuilder().append(part1).append(part2).toString();
  ```
- **Java 9 and later (JEP 280):** Converted to the `invokedynamic` bytecode instruction calling `StringConcatFactory.makeConcatWithConstants()`, which optimizes memory allocation strategies at runtime.

---

### 9. Essential `java.lang.String` Methods

| Method Signature                     | Return Type | Description & Complexity                                                                             |
| :----------------------------------- | :---------- | :--------------------------------------------------------------------------------------------------- |
| `charAt(int index)`                  | `char`      | Returns character at index; throws `StringIndexOutOfBoundsException` ($\mathcal{O}(1)$).             |
| `length()`                           | `int`       | Returns number of UTF-16 code units / characters ($\mathcal{O}(1)$).                                 |
| `substring(int begin, int end)`      | `String`    | Returns substring from $[begin, end)$; end is exclusive.                                             |
| `indexOf(String str)`                | `int`       | Returns index of first occurrence, or `-1` if absent.                                                |
| `contains(CharSequence s)`           | `boolean`   | Checks if substring exists in sequence.                                                              |
| `equals(Object anObject)`            | `boolean`   | Case-sensitive character sequence equality.                                                          |
| `equalsIgnoreCase(String s)`         | `boolean`   | Case-insensitive character sequence equality.                                                        |
| `toUpperCase()` / `toLowerCase()`    | `String`    | Returns a new string with casing converted.                                                          |
| `trim()` / `strip()` (Java 11+)      | `String`    | `trim()` removes ASCII $\le U+0020$; `strip()` removes all Unicode whitespace.                       |
| `replace(char old, char new)`        | `String`    | Returns a new string with all occurrences replaced.                                                  |
| `split(String regex)`                | `String[]`  | Splits string around matches of the regular expression.                                              |
| `isEmpty()` / `isBlank()` (Java 11+) | `boolean`   | `isEmpty()` checks `length == 0`; `isBlank()` checks if string is empty or contains only whitespace. |

---

### 10. Mutable Alternatives: `StringBuffer` vs. `StringBuilder`

Because `String` is immutable, performing repeated concatenations inside loops creates many temporary objects, triggering garbage collection pressure:

```java
// Anti-Pattern: Allocates O(n) temporary String instances on Heap
String log = "";
for (int i = 0; i < 1000; i++) {
    log += i; // Incurs constant reallocations and array copies
}
```

To solve this, Java provides **`StringBuffer`** and **`StringBuilder`**, which wrap **mutable character arrays**.

```
                           CharSequence Implementations
                                        │
         ┌──────────────────────────────┼──────────────────────────────┐
         ▼                              ▼                              ▼
  java.lang.String             java.lang.StringBuffer         java.lang.StringBuilder
  (Immutable)                  (Mutable & Thread-Safe)        (Mutable & Non-Thread-Safe)
  - Read-only                  - Synchronized methods         - Unsynchronized (Fast)
  - String Constant Pool       - Thread-safe                  - Single-threaded workloads
  - Heavy memory in loops      - Slower due to locks          - High performance
```

#### 10.1 `StringBuffer` vs. `StringBuilder` Architectural Comparison

| Dimension             | `java.lang.String`                 | `java.lang.StringBuffer`                     | `java.lang.StringBuilder`                   |
| :-------------------- | :--------------------------------- | :------------------------------------------- | :------------------------------------------ |
| **Introduced In**     | Java 1.0                           | Java 1.0                                     | Java 5 (1.5)                                |
| **Mutability**        | **Immutable**                      | **Mutable**                                  | **Mutable**                                 |
| **Storage**           | SCP & Heap                         | Heap only                                    | Heap only                                   |
| **Thread Safety**     | **Yes** (Inherently thread-safe)   | **Yes** (All methods are `synchronized`)     | **No** (Unsynchronized)                     |
| **Performance Speed** | Fast for reads; poor for mutations | Slower (incurs monitor lock/unlock overhead) | **Fastest** (Zero locking overhead)         |
| **Capacity Growth**   | None (fixed length)                | Dynamically doubles: $(\text{old} * 2) + 2$  | Dynamically doubles: $(\text{old} * 2) + 2$ |

---

### 11. Core Methods of `StringBuffer` and `StringBuilder`

Both classes share an identical core API through their shared package-private superclass `java.lang.AbstractStringBuilder`:

```java
StringBuilder sb = new StringBuilder("Java"); // Default capacity = 16 + length = 20

// 1. append(): Adds data to end of buffer
sb.append(" SE").append(21); // "Java SE21"

// 2. insert(): Inserts data at specific offset
sb.insert(4, " Platform");  // "Java Platform SE21"

// 3. delete() and deleteCharAt(): Removes character range
sb.delete(4, 13);           // "Java SE21"

// 4. reverse(): Inverts buffer in-place
sb.reverse();               // "12ES avaJ"
sb.reverse();               // Restores "Java SE21"

// 5. capacity() vs length():
System.out.println(sb.length());   // 9 (number of characters stored)
System.out.println(sb.capacity()); // 20 (allocated buffer size)

// 6. ensureCapacity(): Pre-allocates buffer to prevent multiple reallocations
sb.ensureCapacity(100);
```

#### 11.1 Dynamic Buffer Growth Formula

When appending elements causes the character count to exceed the current `capacity()`, the buffer resizes using this formula:

$$\text{New Capacity} = (\text{Old Capacity} \times 2) + 2$$

---

### 12. Mutable vs. Immutable Objects: Architectural Summary

```
   Immutable Object (e.g., String, Integer, LocalDate)
   ┌──────────────────────────────────────────────┐
   │ - All fields marked final                    │
   │ - Class marked final (prohibits subclasses)  │
   │ - No setter / mutator methods                │
   │ - Defensive copies on constructor / getters  │
   │ - Inherently thread-safe                     │
   └──────────────────────────────────────────────┘

   Mutable Object (e.g., StringBuilder, ArrayList, Date)
   ┌──────────────────────────────────────────────┐
   │ - State fields can be updated in-place       │
   │ - Provides mutator methods (setters, append) │
   │ - Not thread-safe without synchronization   │
   │ - Lower memory allocation overhead in loops  │
   └──────────────────────────────────────────────┘
```

---

### 13. High-Yield CDAC Exam Traps & Interview Questions

- **Q1: How many objects are created by the following code?**

  ```java
  String s = new String("CDAC");
  ```

  - _Answer:_ **1 or 2 objects.**
  - _Explanation:_
    - If `"CDAC"` already exists in the String Constant Pool from earlier execution, only **1 object** is created (on the general Heap via `new`).
    - If `"CDAC"` is not yet present in the SCP, **2 objects** are created: one in the String Constant Pool and one on the general Heap.

- **Q2: What is the output of the following comparison?**

  ```java
  final String s1 = "CDAC";
  String s2 = "CDAC ACTS";
  String s3 = s1 + " ACTS";
  System.out.println(s2 == s3);
  ```

  - _Answer:_ `true`
  - _Explanation:_ Because `s1` is declared **`final`**, it is treated as a **compile-time constant**. The expression `s1 + " ACTS"` is folded by `javac` at compile-time into `"CDAC ACTS"`, resolving to the exact same reference in the SCP as `s2`. (If `s1` were not `final`, `s3` would be resolved dynamically at runtime on the Heap, making `s2 == s3` evaluate to `false`).

- **Q3: What does `StringBuffer.equals()` do?**
  - _Answer:_ It performs a **reference equality check (`==`)**!
  - _Explanation:_ Neither `StringBuffer` nor `StringBuilder` overrides the `equals()` method from `java.lang.Object`. Therefore, `sb1.equals(sb2)` evaluates to `true` **only if `sb1 == sb2`**. To compare their character contents, convert them to strings first: `sb1.toString().equals(sb2.toString())`.

- **Q4: Why does `char[]` provide better security than `String` for storing passwords?**
  - _Answer:_ Strings are immutable and stored in memory until reclaimed by the Garbage Collector. If an application stores a password in a `String`, it remains in plain text in the heap/SCP for an indeterminate period, where a memory dump can expose it. With a `char[]`, the application can **zero-fill the array immediately** (`Arrays.fill(pwd, '0')`) after authentication, clearing the credentials from RAM.

- **Q5: Can `String` be used as a key in `HashMap` safely?**
  - _Answer:_ **Yes, it is the ideal key.** Because `String` is immutable, its content never changes, ensuring its `hashCode()` remains constant. Additionally, `String` caches its hash code after the first calculation, speeding up map operations.
