# Module 06: Java Collections Framework & Generics

## Unit 6.4: Generics and Java Stream API

---

### 1. Introduction to Generics

Introduced in **Java 5 (JSR 14)**, **Generics** introduce parameter-driven types into the language. They allow types (classes, interfaces) and methods to operate on parameterized abstractions while enforcing compile-time type safety.

Generics provide an abstraction over types, similar to how formal method parameters provide an abstraction over values.

```
Formal Method Parameters (Values):
void printValue(int x)  <── 'x' is a placeholder for runtime integer values

Generic Type Parameters (Types):
class Container<T>     <── 'T' is a placeholder for compile-time types (String, Employee, Integer)
```

---

### 2. Need for Generics: The Legacy Java Vulnerability

Prior to Java 5, collections operated exclusively on raw `java.lang.Object` references. This design introduced two major issues:

```
Legacy Collections (Java 1.4):
┌─────────────────────────┐     add("Hello")      ┌─────────────────────────────────┐
│ ArrayList list          │ ────────────────────► │ Object[] elementData            │
└─────────────────────────┘     add(Integer(10))  │ - [0] = 0x1100 (String "Hello") │
                                                  │ - [1] = 0x2200 (Integer 10)     │
                                                  └─────────────────────────────────┘
                                                                   │
                                                                   ▼ (String) list.get(1)
                                                  java.lang.ClassCastException!
                                                  (Discovered at RUNTIME in production)
```

1. **Mandatory Explicit Downcasting:** Every retrieval from a collection required an explicit cast:
   ```java
   List list = new ArrayList();
   list.add("CDAC");
   String s = (String) list.get(0); // Explicit cast mandatory
   ```
2. **Absence of Compile-Time Type Safety:** Without type parameters, collections accepted any reference type. Type mismatches passed compilation silently and surfaced at runtime as fatal **`java.lang.ClassCastException`** crashes:

   ```java
   List list = new ArrayList();
   list.add("CDAC");
   list.add(Integer.valueOf(100)); // Compiles without warnings!

   for (Object obj : list) {
       String str = (String) obj; // Crashes on second iteration with ClassCastException!
   }
   ```

#### Generics Solution:

```java
List<String> list = new ArrayList<>();
list.add("CDAC");
// list.add(100); // COMPILE ERROR: Incompatible types! Caught instantly by javac.
String s = list.get(0); // Zero casting required; compiler verifies the type.
```

---

### 3. Generic Classes, Interfaces, and Methods

#### 3.1 Generic Classes

A generic class declares one or more **type parameters** in angle brackets (`<T>`, `<K, V>`) following the class name.

```java
public class KeyValuePair<K, V> {
    private final K key;
    private final V value;

    public KeyValuePair(K key, V value) {
        this.key = key;
        this.value = value;
    }

    public K getKey()   { return key; }
    public V getValue() { return value; }
}

// Instantiation via Diamond Operator (<> - Java 7+):
KeyValuePair<String, Integer> entry = new KeyValuePair<>("RollNo", 101);
```

#### Standard Naming Conventions for Type Parameters:

- **`E`**: Element (used extensively in Collections).
- **`K`**: Key (used in Maps).
- **`V`**: Value (used in Maps).
- **`T`**: Type (general-purpose first type parameter).
- **`U, S`**: 2nd and 3rd general-purpose type parameters.
- **`N`**: Number.

#### 3.2 Generic Interfaces

```java
public interface Repository<T, ID> {
    void save(T entity);
    T findById(ID id);
    void delete(ID id);
}

// Concrete class implementing generic interface:
public class UserRepository implements Repository<User, Long> {
    @Override
    public void save(User entity) { /* DB save */ }

    @Override
    public User findById(Long id) { return new User(); }

    @Override
    public void delete(Long id)   { /* DB delete */ }
}
```

#### 3.3 Generic Methods

A method can define its own type parameters, independent of whether its enclosing class is generic. The type parameter section `<T>` must appear **before the method's return type**.

```java
public class UtilityArrayEngine {
    // Generic static method definition
    public static <T> void swap(T[] array, int i, int j) {
        T temp = array[i];
        array[i] = array[j];
        array[j] = temp;
    }
}

// Invocation with type inference:
String[] names = {"Aarav", "Bhavna"};
UtilityArrayEngine.swap(names, 0, 1); // T is inferred as String
```

---

### 4. Bounded Type Parameters

Bounded type parameters restrict the types that can be substituted for a type variable.

```
                      Bounded Type Parameter Syntax
                                    │
         ┌──────────────────────────┴──────────────────────────┐
         ▼                                                     ▼
Single Upper Bound                                    Multiple Bounds
<T extends Number>                                    <T extends Number & Comparable<T>>
(Restricts T to Number or its subclasses)             (Must extend class AND implement interface)
```

```java
// T must be Number or a subclass of Number (Integer, Double, Float, etc.)
public class NumericBox<T extends Number> {
    private T value;

    public NumericBox(T value) {
        this.value = value;
    }

    public double square() {
        // Safe to call Number methods directly on 'value':
        return value.doubleValue() * value.doubleValue();
    }
}

NumericBox<Integer> b1 = new NumericBox<>(10);   // Valid: Integer extends Number
NumericBox<Double>  b2 = new NumericBox<>(5.5);  // Valid: Double extends Number
// NumericBox<String>  b3 = new NumericBox<>("Hi"); // COMPILE ERROR: String is not a Number!
```

#### Multiple Bounds Rule:

Java supports multiple bounds using the `&` operator:

```java
public class Processor<T extends Comparable<T> & Serializable> { ... }
```

> ⚠️ **The Class Bound Invariant:**
> In a multiple bound declaration, if one of the bounds is a **class**, it **must be listed first**. Interfaces follow after `&`. Specifying `<T extends Serializable & Number>` results in a compile-time error.

---

### 5. Wildcards in Generics (`?`)

A wildcard represented by the question mark (`?`) designates an **unknown type**.

```
                                  Wildcard Taxonomy
                                          │
         ┌────────────────────────────────┼────────────────────────────────┐
         ▼                                ▼                                ▼
Upper-Bounded Wildcard            Lower-Bounded Wildcard            Unbounded Wildcard
      <? extends T>                     <? super T>                         <?>
(Covariance: Read-Only)           (Contravariance: Write-Only)       (Unknown: Read Objects)
```

#### 5.1 Invariance in Java Generics (The Subtyping Trap)

Although `Integer` is a subclass of `Number`, **`List<Integer>` is NOT a subclass of `List<Number>`!** Generics are **invariant**:

```java
List<Integer> intList = new ArrayList<>();
// List<Number> numList = intList; // COMPILE ERROR: Incompatible types!
```

_Why?_ If `numList = intList` were allowed, someone could call `numList.add(Double.valueOf(3.14))`, corrupting an underlying integer list with a floating-point value.

#### 5.2 Upper-Bounded Wildcards: `<? extends T>` (Covariance)

Allows a collection to accept any type that is `T` or a **subclass of `T`**.

```java
public static double sumOfList(List<? extends Number> list) {
    double s = 0.0;
    for (Number n : list) {
        s += n.doubleValue(); // Safe read: guaranteed to be at least a Number
    }
    return s;
}

// Valid for List<Integer>, List<Double>, List<Number>:
sumOfList(List.of(1, 2, 3));
sumOfList(List.of(1.1, 2.2, 3.3));
```

#### 5.3 Lower-Bounded Wildcards: `<? super T>` (Contravariance)

Restricts the unknown type to be `T` or a **superclass of `T`**.

```java
public static void addIntegers(List<? super Integer> list) {
    list.add(1);  // Safe write: List accepts Integer or any of its parents
    list.add(2);
}

// Valid for List<Integer>, List<Number>, List<Object>:
List<Object> objList = new ArrayList<>();
addIntegers(objList);
```

#### 5.4 The PECS Guideline (Producer `extends`, Consumer `super`)

Authored by Joshua Bloch (_Effective Java_), the **PECS** rule clarifies when to use which wildcard:

```
┌────────────────────────────────────────────────────────────────────────┐
│ PECS Rule: Producer Extends, Consumer Super                            │
├────────────────────────────────────────────────────────────────────────┤
│ • If your method READS data from a collection parameter (it acts as a   │
│   PRODUCER of data), use <? extends T>.                                │
│                                                                        │
│ • If your method WRITES data into a collection parameter (it acts as   │
│   a CONSUMER of data), use <? super T>.                                │
│                                                                        │
│ • If you need to both READ from and WRITE to a collection, do not use   │
│   wildcards; use an exact type match <T>.                              │
└────────────────────────────────────────────────────────────────────────┘
```

#### 5.5 Unbounded Wildcards: `<?>`

Represents any type. Used when code only uses methods provided by `java.lang.Object`, or when the logic is type-agnostic (e.g., `list.size()`, `list.clear()`).

---

### 6. Type Erasure

**Type Erasure** is the process by which the Java compiler converts generic code into non-generic bytecode, preserving binary backward compatibility with pre-Java 5 JVM class files.

```
Compilation Phase (javac):
Source Code (.java)                                Bytecode (.class - Runtime)
┌─────────────────────────────────────┐            ┌─────────────────────────────────────┐
│ List<String> list = new ArrayList<>();           │ List list = new ArrayList();        │
│ list.add("CDAC");                   │ ─────────► │ list.add("CDAC");                   │
│ String s = list.get(0);             │   Type     │ String s = (String) list.get(0);    │
└─────────────────────────────────────┘  Erasure   └─────────────────────────────────────┘
 (Type arguments verified by compiler)              (Types erased; synthetic casts added)
```

#### 6.1 The Three Mechanisms of Type Erasure:

1. **Replaces Type Parameters with Bounds:**
   - Unbounded types `<T>` are replaced with `java.lang.Object`.
   - Bounded types `<T extends Number>` are replaced with their first bound: `Number`.
2. **Injects Synthetic Type Casts:** Adds explicit downcasts in bytecode where necessary to maintain type safety.
3. **Generates Synthetic Bridge Methods:** Synthesizes bridge methods in subclasses to preserve polymorphism during method overriding.

#### 6.2 Consequences and Restrictions of Type Erasure:

- **Cannot Instantiate Type Parameters with Primitives:**
  `List<int>` is illegal; primitives do not inherit from `Object`. Use `List<Integer>`.
- **Cannot Create Instances of `T`:**
  `T obj = new T();` is illegal because `T` is not known at runtime.
- **Cannot Declare `static` Fields of Type `T`:**
  `private static T value;` is illegal because the static variable would be shared across all parameterized variations (`Box<String>` vs. `Box<Integer>`).
- **Cannot Use `instanceof` with Parameterized Types:**
  `if (list instanceof ArrayList<String>)` is illegal. You can only test unbounded types: `if (list instanceof ArrayList<?>)`.
- **Cannot Instantiate Arrays of Generic Types:**
  `T[] arr = new T[10];` and `List<String>[] arr = new List<String>[10];` are compile-time errors.

---

### 7. Introduction to the Java Stream API

Introduced in **Java 8 (JSR 335)** under `java.util.stream`, the **Stream API** provides a declarative, functional pipeline for processing sequences of data elements.

```
                      Imperative vs. Declarative Processing
                                        │
         ┌──────────────────────────────┴──────────────────────────────┐
         ▼                                                             ▼
Imperative Style (Pre-Java 8)                                 Declarative / Functional (Streams)
- Describes HOW to compute step-by-step                       - Describes WHAT result is desired
- Manual external loops (for, while)                          - Internal iteration pipeline
- Explicit mutable tracking variables                         - Stateless transformations, zero mutability
- Hard to parallelize safely                                  - Trivially parallelized via parallelStream()
```

#### Core Invariants of Streams:

1. **Not a Data Structure:** A Stream does not store data elements in memory; it conveys data from a source (collection, array, I/O channel) through an algorithmic computational pipeline.
2. **Non-Mutating:** Operations on a stream do not modify the underlying data source; they produce fresh output containers.
3. **Lazily Evaluated:** **Intermediate operations are never executed until a Terminal Operation is invoked.**
4. **Single-Use (Consumable Once):** A Stream instance can be traversed **only once**. Attempting to invoke operations on an already terminated stream throws **`java.lang.IllegalStateException: stream has already been operated upon or closed`**.

---

### 8. The Stream Processing Pipeline

```
┌─────────────────┐       ┌──────────────────────────────┐       ┌──────────────────────┐
│  Data Source    │ ────► │ Intermediate Operations      │ ────► │ Terminal Operation   │
│ (Collection,    │       │ (Lazy, transformational)     │       │ (Eager, yields result│
│  Array, I/O)    │       │ filter(), map(), sorted()    │       │  or side-effect)     │
└─────────────────┘       └──────────────────────────────┘       │ collect(), reduce()  │
                                                                 └──────────────────────┘
```

#### 8.1 Creating Streams

```java
// 1. From a Collection:
List<String> list = List.of("A", "B", "C");
Stream<String> s1 = list.stream();

// 2. From an Array:
String[] arr = {"A", "B", "C"};
Stream<String> s2 = Arrays.stream(arr);

// 3. From Static Factory Methods:
Stream<String> s3 = Stream.of("A", "B", "C");
Stream<Integer> infinite = Stream.iterate(0, n -> n + 2); // Unbounded infinite stream
Stream<Double> randoms  = Stream.generate(Math::random);

// 4. Primitive Specialized Streams (Zero Autoboxing Overhead):
IntStream intStream = IntStream.rangeClosed(1, 100); // 1 to 100 inclusive
```

---

### 9. Intermediate vs. Terminal Operations

| Trait                | Intermediate Operations                                            | Terminal Operations                                               |
| :------------------- | :----------------------------------------------------------------- | :---------------------------------------------------------------- |
| **Return Type**      | Always returns a new `Stream<T>`                                   | Returns a non-stream result (`List`, `long`, `Optional`, `void`)  |
| **Execution Timing** | **Lazy** (Queued in a pipeline; zero processing happens)           | **Eager** (Triggers data pull and processing across the pipeline) |
| **Multiplicity**     | Can be chained together in sequence ($0$ to $N$ operations)        | Ends the pipeline; exactly **one** allowed per stream             |
| **Examples**         | `filter()`, `map()`, `sorted()`, `distinct()`, `limit()`, `skip()` | `collect()`, `reduce()`, `forEach()`, `count()`, `findFirst()`    |

#### 9.1 Verification of Lazy Evaluation

```java
List<String> items = List.of("one", "two", "three");

// Pipeline declared with intermediate operations only:
Stream<String> stream = items.stream().filter(s -> {
    System.out.println("Filter evaluated: " + s);
    return s.length() > 3;
});

System.out.println("Pipeline configured.");
// CONSOLE OUTPUT:
// Pipeline configured.
// (Notice that "Filter evaluated" was NEVER printed! Nothing ran because no terminal op was called).

stream.count(); // Terminal operation invoked! NOW the filter prints.
```

---

### 10. Core Stream Operations Deep-Dive

#### 10.1 `filter(Predicate<T>)`

Filters elements by evaluating a boolean condition. Retains elements where `predicate.test(e) == true`.

```java
List<Integer> evens = List.of(1, 2, 3, 4, 5, 6)
                          .stream()
                          .filter(n -> n % 2 == 0)
                          .toList(); // [2, 4, 6]
```

#### 10.2 `map(Function<T, R>)`

Transforms each incoming element of type `T` into a corresponding element of type `R` ($1$-to-$1$ projection).

```java
List<String> names = List.of("alice", "bob", "charlie");
List<String> uppercase = names.stream()
                              .map(String::toUpperCase) // Maps String -> String
                              .toList(); // ["ALICE", "BOB", "CHARLIE"]

List<Integer> nameLengths = names.stream()
                                 .map(String::length)   // Maps String -> Integer
                                 .toList(); // [5, 3, 7]
```

#### 10.3 `sorted()` and `sorted(Comparator<T>)`

Stateful intermediate operation that sorts elements.

```java
List<String> sortedNames = names.stream()
                                .sorted(Comparator.reverseOrder())
                                .toList();
```

#### 10.4 `distinct()`

Filters out duplicates. Uses `equals()` and `hashCode()` to determine uniqueness ($\mathcal{O}(n)$ stateful operation).

```java
List<Integer> unique = List.of(1, 2, 2, 3, 3, 3).stream().distinct().toList(); // [1, 2, 3]
```

#### 10.5 `reduce()` (Aggregation / Folding)

Combines all elements of a stream into a single aggregated result using an associative accumulator function.

$$
\text{reduce}(T_{\text{identity}}, \text{BinaryOperator}<T>)
$$

```java
List<Integer> numbers = List.of(1, 2, 3, 4, 5);

// 1. Reduce with Identity element:
int sum = numbers.stream().reduce(0, (a, b) -> a + b); // 15

// 2. Reduce without Identity (returns Optional in case stream is empty):
Optional<Integer> max = numbers.stream().reduce(Integer::max);
```

#### 10.6 `collect(Collector<T, A, R>)`

Converts stream elements into a concrete result container (`List`, `Set`, `Map`, `String`).

```java
import java.util.stream.Collectors;

List<String> cities = List.of("Pune", "Mumbai", "Delhi", "Pune");

// Collect to Set:
Set<String> uniqueCities = cities.stream().collect(Collectors.toSet());

// Join strings with delimiter:
String joined = cities.stream().collect(Collectors.joining(", ")); // "Pune, Mumbai, Delhi, Pune"

// Group By operation:
Map<Integer, List<String>> byLength = cities.stream()
    .collect(Collectors.groupingBy(String::length));
```

#### 10.7 `count()` and `forEach()`

```java
long matchCount = cities.stream().filter(c -> c.startsWith("P")).count(); // 2

// forEach terminal operation (Terminal side-effect):
cities.stream().distinct().forEach(System.out::println);
```

---

### 11. The `java.util.Optional<T>` Container Class

Introduced in Java 8, **`Optional<T>`** is a single-element container object that may or may not contain a non-null value. It was created to provide a clear, type-level representation of return types that could be absent, reducing accidental **`NullPointerException`** crashes.

```
                      Optional Container States
                                  │
         ┌────────────────────────┴────────────────────────┐
         ▼                                                 ▼
Optional.of(value)                                Optional.empty()
(Present: encapsulates target instance)           (Absent: represents cleanly missing value)
```

#### 11.1 Instantiation Patterns:

```java
Optional<String> opt1 = Optional.of("Hello");      // Throws NullPointerException if arg is null!
Optional<String> opt2 = Optional.ofNullable(null); // Safe: returns Optional.empty()
Optional<String> opt3 = Optional.empty();          // Explicit empty representation
```

#### 11.2 Idiomatic Consumption Patterns (Avoiding `.get()`):

Calling `opt.get()` on an empty `Optional` throws `NoSuchElementException`. Instead, use functional accessor methods:

```java
Optional<String> opt = Optional.ofNullable(fetchData());

// 1. orElse (Provides fallback default value eagerly):
String val1 = opt.orElse("Default Value");

// 2. orElseGet (Provides fallback lazily via Supplier - Preferred for heavy calculations):
String val2 = opt.orElseGet(() -> computeExpensiveFallback());

// 3. orElseThrow (Throws custom business exception if absent):
String val3 = opt.orElseThrow(() -> new ResourceNotFoundException("Entity missing"));

// 4. ifPresent (Executes Consumer side-effect only if present):
opt.ifPresent(val -> System.out.println("Retrieved: " + val));

// 5. Functional Transformation via map():
Optional<Integer> lengthOpt = opt.map(String::length);
```

---

### 12. High-Yield CDAC Exam Traps & Interview Questions

- **Q1: What is the exact difference between `List<Object>` and `List<?>`?**
  - _Answer:_
    - `List<Object>` is a concrete parameterized type that **can only hold a reference to `List<Object>`** (due to generic invariance). You can add any `Object` to it.
    - `List<?>` is an **unbounded wildcard type representing a list of an unknown type**. It can hold a reference to `List<String>`, `List<Integer>`, or any other list. However, because the type is unknown, **you cannot add any elements to it** (except `null`).

- **Q2: Why does the following generic method fail to compile?**

  ```java
  public static <T> T[] createArray(int size) {
      return new T[size]; // COMPILE ERROR!
  }
  ```

  - _Answer:_ **Type erasure removes `T` at compile time.** At runtime, the JVM needs to know the exact component type of an array to manage array store checks (`ArrayStoreException`). Because `T` is erased to `Object`, the JVM cannot allocate a specialized array of `T`.
  - _Fix:_ Pass an explicit class token: `(T[]) java.lang.reflect.Array.newInstance(clazz, size);`.

- **Q3: What happens when the following stream pipeline executes?**

  ```java
  Stream<String> s = Stream.of("A", "B", "C");
  s.filter(x -> x.equals("A"));
  s.forEach(System.out::println);
  ```

  - _Answer:_ **Throws `java.lang.IllegalStateException: stream has already been operated upon or closed`**.
  - _Explanation:_ `s.filter(...)` returns a **brand-new stream** and marks stream `s` as consumed. Attempting to call `.forEach()` on the original stream reference `s` fails because streams are strictly single-use.

- **Q4: Why should `Optional` NOT be used as a class field or method parameter?**
  - _Answer:_
    1. **Not Serializable:** `Optional` does not implement `java.io.Serializable`. Using it as a field breaks serialization of the enclosing class.
    2. **Memory Overhead:** Wrapping every field in an `Optional` adds an extra 16-to-24-byte object reference per field, increasing garbage collection overhead.
    3. **Design Intent:** The designers of Java intended `Optional` strictly as a **library method return type** to signal potential absence without returning raw `null`.

- **Q5: What is the difference between `Stream.map()` and `Stream.flatMap()`?**
  - _Answer:_
    - `map(Function<T, R>)`: Performs a $1$-to-$1$ transformation. If the mapper function returns a stream (e.g., `s -> Arrays.stream(s.split(""))`), `map()` yields a nested stream: `Stream<Stream<String>>`.
    - `flatMap(Function<T, Stream<R>>)`: Performs a $1$-to-$N$ transformation and **flattens** the nested streams into a single unified stream: `Stream<String>`.
