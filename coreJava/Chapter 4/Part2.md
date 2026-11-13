# Module 04: Exception Handling, Strings, and Standard APIs

## Unit 4.2: Wrapper Classes and Java Utility Classes

### 1. Introduction to Wrapper Classes

In Java, the eight primitive data types (`byte`, `short`, `int`, `long`, `float`, `double`, `char`, `boolean`) are not objects. They hold raw binary values directly on the stack frame or within object fields, lacking object overhead (headers, virtual method tables).

To integrate primitives into object-oriented frameworks—such as the **Java Collections Framework** (`List<T>`, `Map<K, V>`), reflection APIs, and concurrency utilities—Java provides **Wrapper Classes** in `java.lang`. A wrapper class encapsulates a single primitive value inside an immutable heap object.

```
                            java.lang.Object
                                   │
         ┌─────────────────────────┴─────────────────────────┐
         ▼                                                   ▼
  java.lang.Number                                   java.lang.Boolean
  (Abstract Superclass)                              java.lang.Character
         ├── Byte                                    (Direct Object Subclasses)
         ├── Short
         ├── Integer
         ├── Long
         ├── Float
         └── Double
```

---

### 2. Primitive vs. Wrapper Types: Architectural Breakdown

| Metric               | Primitive Data Type                                                | Wrapper Class                                                        |
| :------------------- | :----------------------------------------------------------------- | :------------------------------------------------------------------- |
| **Storage Location** | Local Variable Array (LVA) on Stack or inlined inside Heap objects | Always a reference on Stack/Heap pointing to an Object on the Heap   |
| **Memory Footprint** | `int`: 4 bytes, `byte`: 1 byte, `double`: 8 bytes                  | `Integer`: 16 to 24 bytes (12B header + 4B field + padding)          |
| **Default Value**    | `0`, `0.0`, `false`, `'\u0000'`                                    | `null` (unassigned references)                                       |
| **Nullability**      | Cannot hold `null`                                                 | Can hold `null` (risks `NullPointerException`)                       |
| **Generics Support** | Prohibited (`List<int>` is illegal)                                | Supported (`List<Integer>`)                                          |
| **Equality Check**   | `==` compares bit values                                           | `==` compares memory references; `.equals()` compares wrapped values |
| **Performance**      | High (direct CPU register arithmetic)                              | Slower (incurs heap traversal and GC overhead)                       |

---

### 3. Core Wrapper Classes

#### 3.1 `Integer`, `Long`, `Short`, `Byte`

Subclasses of `java.lang.Number`. Each provides constants for numeric boundaries:

- `Integer.MIN_VALUE` ($-2^{31}$) and `Integer.MAX_VALUE` ($2^{31} - 1$)
- `Integer.BYTES` ($4$) and `Integer.SIZE` ($32$ bits)

#### 3.2 `Double` and `Float`

Support IEEE 754 floating-point standards, providing special mathematical constants:

- `Double.NaN`: Represents undefined results (e.g., $0.0 / 0.0$).
- `Double.POSITIVE_INFINITY`: Represents division of positive values by zero ($1.0 / 0.0$).
- `Double.NEGATIVE_INFINITY`: ($-1.0 / 0.0$).

```java
// Testing IEEE 754 edge cases:
double val = 0.0 / 0.0;
System.out.println(Double.isNaN(val));                // true
System.out.println(Double.isInfinite(10.0 / 0.0));    // true
```

#### 3.3 `Character`

Wraps a single 16-bit UTF-16 code point. Provides static inspection utilities:

- `Character.isDigit(char ch)`
- `Character.isLetter(char ch)`
- `Character.isWhitespace(char ch)`
- `Character.toUpperCase(char ch)` / `toLowerCase(char ch)`

#### 3.4 `Boolean`

Wraps a `boolean` primitive. Provides pre-allocated constants:

- `Boolean.TRUE` and `Boolean.FALSE`

```java
// String-to-boolean parsing behavior:
Boolean b1 = Boolean.valueOf("true");   // true
Boolean b2 = Boolean.valueOf("True");   // true (case-insensitive)
Boolean b3 = Boolean.valueOf("yes");    // false (any non-"true" string evaluates to false!)
Boolean b4 = Boolean.valueOf(null);     // false
```

---

### 4. Autoboxing and Unboxing (Java 5+)

```
                          Autoboxing (javac injection)
                      Integer.valueOf(primitiveVal)
        Primitive Type ──────────────────────────────► Wrapper Object
         (e.g., int)   ◄──────────────────────────────  (e.g., Integer)
                          boxedObject.intValue()
                           Unboxing (javac injection)
```

#### 4.1 Autoboxing

Automatic conversion of a primitive value into its corresponding wrapper object:

```java
// Developer writes:
Integer box = 42;

// Compiler (javac) emits:
Integer box = Integer.valueOf(42);
```

#### 4.2 Unboxing

Automatic extraction of the primitive value from a wrapper object:

```java
// Developer writes:
int unbox = box;

// Compiler (javac) emits:
int unbox = box.intValue();
```

#### 4.3 The Hidden `NullPointerException` Hazard

If a wrapper reference is `null` and an unboxing operation occurs (such as in arithmetic, comparisons, or method parameter passing), the JVM attempts to invoke `.intValue()` on a `null` reference, throwing an immediate **`java.lang.NullPointerException`**.

```java
Integer counter = null;

// Implicit unboxing triggers: counter.intValue() + 1
// RUNTIME EXCEPTION: java.lang.NullPointerException
int updated = counter + 1;
```

---

### 5. Wrapper Class Utility Methods

Wrapper classes serve as repositories of static conversion routines:

```
                            Three-Way Conversion Pipeline
                                        String
                                       ▲      │
            String.valueOf(prim)      │      │ parseXxx(str)
            ┌─────────────────────────┘      └─────────────────────────┐
            │                                                          ▼
      Primitive Type ──────────────────────────────────────────► Wrapper Object
                      ◄────────────────────────────────────────
                        Autoboxing: Integer.valueOf(prim)
                        Unboxing:   wrapper.intValue()
```

#### 5.1 Parsing Methods: `parseXxx(String)` vs. `valueOf(String)`

- **`parseXxx(String s)`**: Returns a **primitive** type. Throws `NumberFormatException` if parsing fails.
- **`valueOf(String s)`**: Returns a **boxed wrapper instance**. Leverages internal caching.

```java
// 1. String to Primitive
int pVal = Integer.parseInt("123");         // returns primitive int

// 2. String to Primitive with Radix (Base conversion)
int hexVal = Integer.parseInt("FF", 16);    // parses base-16 -> 255
int binVal = Integer.parseInt("1010", 2);   // parses base-2  -> 10

// 3. String to Wrapper Object
Integer oVal = Integer.valueOf("123");      // returns Integer object

// 4. Primitive / Object to String
String s1 = Integer.toString(500);          // "500"
String s2 = Integer.toHexString(255);       // "ff"
String s3 = Integer.toBinaryString(10);      // "1010"
```

---

### 6. Integer Caching & Wrapper Constant Pools

To optimize memory usage and performance, Java caches frequently used wrapper instances.

#### 6.1 The Caching Invariant

`valueOf()` delegates to an internal static cache array (`IntegerCache`). If the requested value falls within the cache range, it returns an existing object reference instead of creating a new one on the Heap.

| Wrapper Class         | Cached Value Range                               | Configurable?                                            |
| :-------------------- | :----------------------------------------------- | :------------------------------------------------------- |
| **`Byte`**            | $-128$ to $+127$                                 | Fixed                                                    |
| **`Short`**           | $-128$ to $+127$                                 | Fixed                                                    |
| **`Integer`**         | **$-128$ to $+127$**                             | High limit configurable via `-XX:AutoBoxCacheMax=<size>` |
| **`Long`**            | $-128$ to $+127$                                 | Fixed                                                    |
| **`Character`**       | $0$ to $127$ (ASCII range: `\u0000` to `\u007F`) | Fixed                                                    |
| **`Boolean`**         | `TRUE` and `FALSE` (both instances)              | Fixed                                                    |
| **`Float`, `Double`** | **No Caching**                                   | Not applicable (infinite real intervals)                 |

```java
// Demonstrating IntegerCache:
Integer a = 127;
Integer b = 127;
System.out.println(a == b);      // true (Both refer to same cached instance in IntegerCache)

Integer x = 128;
Integer y = 128;
System.out.println(x == y);      // false (Exceeds cache boundary; distinct heap objects!)
System.out.println(x.equals(y)); // true  (Values are identical)
```

```
STACK MEMORY                                        HEAP MEMORY
┌───────────────────────────┐                      ┌───────────────────────────────────────────┐
│ a: 0x1000 ────────────────┼─────────┐            │ IntegerCache Array                        │
│ b: 0x1000 ────────────────┼─────────┤            │ ┌───────────────────────────────────────┐ │
└───────────────────────────┘         └───────────►│ │ 0x1000: Integer(127)                  │ │
                                                   │ └───────────────────────────────────────┘ │
┌───────────────────────────┐                      ├───────────────────────────────────────────┤
│ x: 0x5500 ────────────────┼─────────────────────►│ 0x5500: Integer(128) [New Heap Object]   │
│ y: 0x6600 ────────────────┼─────────────────────►│ 0x6600: Integer(128) [New Heap Object]   │
└───────────────────────────┘                      └───────────────────────────────────────────┘
```

---

### 7. Legacy Date-Time API (`java.util.Date`, `java.util.Calendar`)

Prior to Java 8, date and time manipulation relied on `java.util.Date` and `java.util.Calendar`.

```java
// Legacy Date usage:
java.util.Date legacyDate = new java.util.Date();
System.out.println(legacyDate); // Prints current timestamp

// Legacy Calendar usage:
java.util.Calendar cal = java.util.Calendar.getInstance();
cal.set(2026, java.util.Calendar.NOVEMBER, 10);
```

#### 7.1 Why the Legacy API Was Replaced (Architectural Flaws)

1. **Mutability (Thread-Safety Hazard):** `java.util.Date` is mutable. If a date instance is passed across methods or threads, its state can be modified via `setTime()`, forcing developers to write defensive copies.
2. **Poor Design / Zero-Based Offsets:**
   - Months are zero-indexed ($0 = \text{January}, 11 = \text{December}$).
   - Years are offset from $1900$ (e.g., year $2026$ was represented as $126$).
3. **No Separation of Concerns:** A single class (`Date`) attempted to represent a date, a time, and a timestamp simultaneously, with default behavior tied to the JVM's local timezone.
4. **Formatting Bottleneck:** `java.text.SimpleDateFormat` is **not thread-safe**. Sharing a formatter instance across threads corrupts internal state.

---

### 8. Modern Date-Time API (`java.time` - JSR 310, Java 8+)

Java 8 introduced an immutable, thread-safe, domain-driven date-time framework based on the ISO-8601 calendar system.

```
                           java.time Core Taxonomy
                                      │
         ┌────────────────────────────┼────────────────────────────┐
         ▼                            ▼                            ▼
LocalDate                    LocalTime                    LocalDateTime
(Date without time/zone)     (Time without date/zone)     (Combines Date + Time)
e.g., 2026-11-10             e.g., 16:45:00               e.g., 2026-11-10T16:45:00
         │                            │                            │
         └────────────────────────────┼────────────────────────────┘
                                      ▼
                               ZonedDateTime
                     (Date + Time + ZoneId e.g., Asia/Kolkata)
```

#### 8.1 Key Design Invariants of `java.time`

- **Immutability:** Every class is `final` and immutable. Methods like `plusDays()` do not modify the calling instance; they return a new object.
- **Fluent API:** Supports clear method chaining (e.g., `.plusDays(5).minusHours(2)`).
- **Sensible Indexing:** Months are indexed naturally from $1$ ($\text{January}$) to $12$ ($\text{December}$) or accessed via the `Month` enum.

---

### 9. Core Classes in `java.time`

#### 9.1 `LocalDate`

Represents an ISO-8601 date (year, month, day) without a time or timezone component.

```java
import java.time.LocalDate;
import java.time.Month;

// Factory instantiation:
LocalDate today = LocalDate.now();                     // System current date
LocalDate releaseDate = LocalDate.of(2026, Month.NOVEMBER, 10);
LocalDate parsedDate = LocalDate.parse("2026-11-10");  // Default ISO-8601 parsing

// Date arithmetic (Immutable: returns a fresh instance):
LocalDate nextWeek = today.plusDays(7);
LocalDate pastMonth = today.minusMonths(1);
```

#### 9.2 `LocalTime`

Represents a time (hour, minute, second, nanosecond) without a date or timezone context.

```java
import java.time.LocalTime;

LocalTime now = LocalTime.now();
LocalTime specificTime = LocalTime.of(16, 45, 0); // 16:45:00
LocalTime adjusted = now.plusMinutes(30);
```

#### 9.3 `LocalDateTime`

Combines `LocalDate` and `LocalTime` to represent date and time without a timezone.

```java
import java.time.LocalDateTime;

LocalDateTime currentLDT = LocalDateTime.now();
LocalDateTime customLDT = LocalDateTime.of(2026, 11, 10, 16, 45);
```

#### 9.4 `Instant` and `Duration` / `Period`

- **`Instant`**: Represents an instantaneous point on the timeline measured from the Unix epoch ($1970\text{-}01\text{-}01\text{T}00:00:00\text{Z}$) in UTC.
- **`Period`**: Models **date-based** time amounts (years, months, days).
- **`Duration`**: Models **time-based** amounts (seconds, nanoseconds).

```java
import java.time.Period;
import java.time.Duration;

Period daysBetween = Period.between(LocalDate.of(2026, 1, 1), LocalDate.of(2026, 1, 10));
System.out.println(daysBetween.getDays()); // 9

Duration duration = Duration.ofHours(5);
System.out.println(duration.toMinutes());   // 300
```

---

### 10. Date Formatting and Parsing (`DateTimeFormatter`)

The `java.time.format.DateTimeFormatter` class is **immutable and fully thread-safe**, replacing the error-prone `SimpleDateFormat`.

#### 10.1 Formatting (Date Object $\rightarrow$ Formatted String)

```java
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

LocalDateTime now = LocalDateTime.now();

// 1. Predefined standard formatters
String iso = now.format(DateTimeFormatter.ISO_DATE_TIME);

// 2. Custom Pattern Formatter
DateTimeFormatter customFormatter = DateTimeFormatter.ofPattern("dd-MM-yyyy HH:mm:ss");
String formattedDate = now.format(customFormatter);
System.out.println(formattedDate); // e.g., "10-11-2026 16:45:30"
```

#### 10.2 Parsing (String $\rightarrow$ Date Object)

```java
DateTimeFormatter dtf = DateTimeFormatter.ofPattern("dd/MM/yyyy");
String input = "10/11/2026";

LocalDate parsed = LocalDate.parse(input, dtf);
System.out.println(parsed); // "2026-11-10"
```

---

### 11. Bridging Legacy and Modern Date-Time APIs

Interoperability methods bridge older legacy codebases with modern `java.time` structures via the `Instant` class:

```java
// Legacy Date to Modern Instant/LocalDateTime:
java.util.Date oldDate = new java.util.Date();
java.time.Instant instant = oldDate.toInstant();
java.time.LocalDateTime modernLDT = instant.atZone(java.time.ZoneId.systemDefault())
                                           .toLocalDateTime();

// Modern Instant back to Legacy Date:
java.util.Date backToDate = java.util.Date.from(instant);
```

---

### 12. High-Yield CDAC Exam Traps & Interview Questions

- **Q1: What is the output of comparing two new wrapper instances vs. two autoboxed instances?**

  ```java
  Integer a = new Integer(10);
  Integer b = new Integer(10);
  Integer c = 10;
  Integer d = 10;
  System.out.println((a == b) + " " + (c == d));
  ```

  - _Answer:_ `false true`
  - _Explanation:_ `new Integer(10)` explicitly creates two separate objects on the Heap, bypassing the cache entirely (`a == b` is `false`). Conversely, `c` and `d` use autoboxing, resolving to `Integer.valueOf(10)`, which pulls the same cached instance from `IntegerCache` (`c == d` is `true`). _(Note: `new Integer()` is deprecated since Java 9)._

- **Q2: What happens when evaluating `Integer n1 = null; int n2 = n1;`?**
  - _Answer:_ Throws **`java.lang.NullPointerException`**.
  - _Explanation:_ To assign `n1` to primitive `n2`, the compiler inserts an implicit unboxing call: `n1.intValue()`. Invoking a method on `null` triggers an immediate NPE.

- **Q3: What does `Boolean.valueOf("True") == Boolean.valueOf("true")` evaluate to?**
  - _Answer:_ `true`
  - _Explanation:_ `Boolean.valueOf(String)` evaluates string contents using case-insensitive comparison (`"true".equalsIgnoreCase(s)`). Both evaluate to `true`, and both return the static singleton reference `Boolean.TRUE`.

- **Q4: Why does `Double a = 1.0; Double b = 1.0; a == b` evaluate to `false`?**
  - _Answer:_ Floating-point wrapper classes (`Double` and `Float`) **do not implement an object cache**. Because there are an infinite number of real floating-point values even between $0.0$ and $1.0$, caching is impractical. Each autoboxing operation creates a new heap object.

- **Q5: Is `DateTimeFormatter` thread-safe? How does it differ from `SimpleDateFormat`?**
  - _Answer:_ **Yes, `DateTimeFormatter` is completely immutable and thread-safe.** It can be stored as a `public static final` constant and safely shared across multiple concurrent worker threads. In contrast, `SimpleDateFormat` maintains internal mutable state structures (like calendar fields) during parsing, making concurrent use without synchronization a cause of race conditions and corrupted date values.
