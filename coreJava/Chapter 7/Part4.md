# Module 07: Concurrency, Multithreading, and Modern Java

## Unit 7.4: Lambda Expressions and Functional Programming

### 1. Introduction to Functional Programming in Java

Prior to **Java 8 (JSR 335)**, Java was an almost purely object-oriented language. Functions could not exist independently; to pass logic into a method, developers had to wrap code inside an object—typically using verbose **Anonymous Inner Classes**.

Java 8 introduced a **hybrid programming model**, blending Object-Oriented Programming (OOP) with core tenets of **Functional Programming (FP)**:

```
                            Programming Paradigm Shift
                                        │
         ┌──────────────────────────────┴──────────────────────────────┐
         ▼                                                             ▼
Object-Oriented Paradigm (Imperative)                         Functional Paradigm (Declarative)
- Focuses on "HOW" to execute steps                           - Focuses on "WHAT" outcome is desired
- State mutation and side-effects                             - Pure functions and immutability
- Code passed wrapped in classes / objects                    - Code passed directly as behavior / data
- Heavy memory and classloader footprints                     - Lightweight invokedynamic dispatch

```

#### Core Tenets Adopted by Modern Java:

1. **First-Class Functions (Approximated):** Functions can be treated as first-class citizens: assigned to variables, passed as arguments to methods, and returned from methods.

2. **Behavior Parameterization:** The ability to pass executable logic into a method as easily as passing primitive or reference data.

3. **Immutability & Pure Functions:** Discouraging shared mutable state to eliminate concurrency race conditions.

---

### 2. Functional Interfaces (SAM - Single Abstract Method)

A **Functional Interface** is an interface that contains **exactly one abstract method**. It serves as the structural target type for Lambda Expressions and Method References.

```
                           Functional Interface Invariants
                                         │
        ┌────────────────────────────────┴────────────────────────────────┐
        ▼                                                                 ▼
Exactly 1 Abstract Method                                    Unrestricted Other Members
- The SAM (Single Abstract Method)                           - Unlimited default methods
- Defines the functional signature                           - Unlimited static methods
                                                             - Public methods inherited from java.lang.Object

```

#### 2.1 The `@FunctionalInterface` Annotation

An informative annotation that instructs `javac` to enforce the Single Abstract Method rule at compile time:

```
@FunctionalInterface
public interface DataFilter<T> {
    // 1. The Single Abstract Method (SAM)
    boolean evaluate(T data);

    // 2. Default methods DO NOT invalidate SAM status
    default void printDiagnostics() {
        System.out.println("Executing diagnostic check...");
    }

    // 3. Static methods DO NOT invalidate SAM status
    static void logVersion() {
        System.out.println("DataFilter v1.0");
    }

    // 4. Overrides of public java.lang.Object methods DO NOT count as abstract!
    @Override
    boolean equals(Object obj);
}

```

> ⚠️ **The Non-SAM Compilation Trap:**
> If an interface annotated with `@FunctionalInterface` declares zero abstract methods, or more than one abstract method, the compiler immediately raises an error:
> `Unexpected @FunctionalInterface annotation: interface is not a functional interface`.

---

### 3. Lambda Expressions & Syntax Rules

A **Lambda Expression** is an anonymous function—a block of code with formal parameters and a body, but without an identifier name.

```
                          Anatomy of a Lambda Expression
                 (parameters)       ->          { body }
                ┌────────────┐     ┌──┐     ┌──────────────┐
                │ (int a, b) │ ──► │->│ ──► │ return a + b;│
                └────────────┘     └──┘     └──────────────┘
                    Inputs        Lambda       Operation /
                                  Arrow        Expression

```

#### 3.1 The 5 Syntactic Rules:

1. **Parameter Type Inference:** Specifying parameter types is optional; the compiler infers them from the target functional interface context (_Type Inference_):

   ```
   (int x, int y) -> x + y;  // Explicit
   (x, y) -> x + y;          // Inferred (Identical)

   ```

2. **Single Parameter Parentheses Omission:** If there is exactly **one inferred parameter**, the parentheses may be omitted:

   ```
   x -> x * x;               // Valid
   (x) -> x * x;             // Valid
   () -> 42;                 // Mandatory parentheses for ZERO parameters
   (x, y) -> x + y;          // Mandatory parentheses for MULTIPLE parameters

   ```

3. **Single-Expression Return Omission:** If the body consists of a single expression, curly braces `{}` and the `return` keyword **must both be omitted**:

   ```
   (a, b) -> a + b;                 // Valid: implicit return
   (a, b) -> { return a + b; };     // Valid: explicit block return
   // (a, b) -> return a + b;       // COMPILE ERROR: Cannot use return without braces!

   ```

4. **Void Returns:** For methods returning `void`, a single method call can stand without braces:

   ```
   msg -> System.out.println(msg);  // Valid void expression

   ```

---

### 4. Architectural Deep Dive: Anonymous Inner Classes vs. Lambdas

While lambdas syntactically resemble anonymous inner classes, their low-level JVM implementation and runtime memory footprint are completely different.

```
Anonymous Inner Class (Pre-Java 8):               Lambda Expression (Java 8+):
┌───────────────────────────────┐                 ┌───────────────────────────────┐
│ Generates Main$1.class disk   │                 │ ZERO extra .class generated   │
│ file via javac                │                 │ Compiled via invokedynamic    │
├───────────────────────────────┤                 ├───────────────────────────────┤
│ Allocates fresh Heap object   │                 │ Lazily creates call-site via  │
│ every single invocation       │                 │ LambdaMetafactory (No alloc   │
│                               │                 │ if non-capturing)             │
├───────────────────────────────┤                 ├───────────────────────────────┤
│ Creates hidden 'this$0'       │                 │ No hidden enclosing pointer   │
│ enclosing pointer             │                 │ (Zero memory leak retention)  │
└───────────────────────────────┘                 └───────────────────────────────┘

```

| Dimension               | Anonymous Inner Class                                                       | Lambda Expression                                                                 |
| ----------------------- | --------------------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| **Bytecode Generation** | Generates an auxiliary `.class` file on disk (`Outer$1.class`)              | **No extra `.class` file generated**. Uses `invokedynamic` opcode.                |
| **Memory Allocation**   | Forces a new Heap object allocation (`new`) every time it is encountered    | **Reuses singleton call-site instances** if non-capturing (zero GC churn).        |
| **Scope of `this`**     | `this` points to the **anonymous inner class instance itself**              | **Lexical Scoping**: `this` refers to the **enclosing class instance**.           |
| **Applicable Targets**  | Can extend an abstract class, concrete class, or interface with $N$ methods | Can **only** implement a Functional Interface with **exactly 1** abstract method. |

#### 4.1 The Scope of `this` Invariant

```
public class ScopeDemo {
    private final String context = "ENCLOSING_CLASS";

    public void testScopes() {
        // 1. Anonymous Inner Class
        Runnable r1 = new Runnable() {
            private final String context = "INNER_CLASS";
            @Override
            public void run() {
                System.out.println(this.context); // PRINTS: "INNER_CLASS"
            }
        };

        // 2. Lambda Expression (Lexically Scoped)
        Runnable r2 = () -> {
            // Cannot declare variable 'context' (already in lexical scope)
            System.out.println(this.context); // PRINTS: "ENCLOSING_CLASS"
        };

        r1.run();
        r2.run();
    }
}

```

#### 4.2 Variable Capture & "Effectively Final" Invariant

A lambda expression can capture variables declared in its enclosing method, but **only if they are `final` or effectively final** (their value is never changed after initialization).

```
public void captureDemo() {
    int port = 8080; // Effectively final
    Runnable r = () -> System.out.println("Connecting to port: " + port); // Legal capture

    // port = 9090; // UNCOMMENTING CAUSES COMPILE ERROR:
    // "local variables referenced from a lambda expression must be final or effectively final"
}

```

---

### 5. Method References (`::`)

A **Method Reference** is a shorthand syntactic alias for a lambda expression that simply forwards arguments directly to an existing named method.

```
Lambda Equivalent:               Method Reference:
x -> System.out.println(x)  ──►  System.out::println

```

#### The Four Types of Method References:

```
                            Method Reference Taxonomy
                                        │
     ┌──────────────────────┬───────────┴───────────┬──────────────────────┐
     ▼                      ▼                       ▼                      ▼
Reference to Static    Reference to Bound      Reference to Unbound    Reference to Constructor
Method                 Instance Method         Instance Method         ClassName::new
Class::staticMethod    ref::instanceMethod     Class::instanceMethod
(String::valueOf)      (System.out::println)   (String::toLowerCase)   (ArrayList::new)

```

```
import java.util.function.*;

public class MethodReferenceCatalog {
    public static void main(String[] args) {
        // Type 1: Reference to a Static Method (ContainingClass::staticMethodName)
        Function<Double, Double> sqrtLambda = x -> Math.sqrt(x);
        Function<Double, Double> sqrtRef    = Math::sqrt;

        // Type 2: Reference to a Bound Instance Method (existingInstance::methodName)
        String prefix = "AUTH_";
        Predicate<String> startsWithLambda = s -> prefix.startsWith(s);
        Predicate<String> startsWithRef    = prefix::startsWith;

        // Type 3: Reference to an Unbound Instance Method (ClassName::methodName)
        // (First argument becomes the target on which the method is called!)
        BiPredicate<String, String> equalsLambda = (s1, s2) -> s1.equalsIgnoreCase(s2);
        BiPredicate<String, String> equalsRef    = String::equalsIgnoreCase;

        // Type 4: Reference to a Constructor (ClassName::new)
        Supplier<StringBuilder> sbLambda = () -> new StringBuilder();
        Supplier<StringBuilder> sbRef    = StringBuilder::new;
    }
}

```

---

### 6. Core Functional Interfaces Catalog (`java.util.function`)

Java 8 bundles 43 specialized functional interfaces inside `java.util.function`. The entire package is built upon four foundational archetypes:

```
                      The 4 Core Functional Archetypes
                                     │
         ┌───────────────────────────┼───────────────────────────┐
         ▼                           ▼                           ▼
   Predicate<T>                 Function<T, R>              Consumer<T>                Supplier<T>
  T ──► boolean                     T ──► R                    T ──► void                void ──► T
 (Condition Check)              (Transformation)            (Side-Effect Sink)        (Factory / Source)

```

#### 6.1 `Predicate<T>` (The Condition Evaluator)

- **Functional Method:** `boolean test(T t)`

- **Intent:** Filters and validates conditions on incoming arguments.

- **Default Methods for Boolean Logic Composition:**
  - `and(Predicate<? super T> other)`: Logical AND ($\land$)

  - `or(Predicate<? super T> other)`: Logical OR ($\lor$)

  - `negate()`: Logical NOT ($\neg$)

```
Predicate<String> hasMinLength = s -> s.length() >= 8;
Predicate<String> containsSpecial = s -> s.contains("@");

// Composing predicates:
Predicate<String> validPassword = hasMinLength.and(containsSpecial);
System.out.println(validPassword.test("secret@123")); // true

```

#### 6.2 `Function<T, R>` (The Transformer)

- **Functional Method:** `R apply(T t)`

- **Intent:** Maps an input element of type `T` to a distinct result of type `R`.

- **Default Methods for Pipeline Chaining:**
  - `andThen(Function after)`: Executes current function first, then pipes result into `after`.

  - `compose(Function before)`: Executes `before` function first, then pipes result into current function.

```
Function<String, Integer> parse = Integer::parseInt;
Function<Integer, Integer> square = x -> x * x;

// andThen vs compose pipeline:
Function<String, Integer> parseAndSquare = parse.andThen(square);
System.out.println(parseAndSquare.apply("5")); // 25 (parses "5" -> 5, then 5^2 -> 25)

```

#### 6.3 `Consumer<T>` (The Side-Effect Consumer)

- **Functional Method:** `void accept(T t)`

- **Intent:** Consumes an input and performs a side-effect (e.g., logging, printing, writing to a database, emitting a metric). Returns nothing.

- **Chaining:** `andThen(Consumer after)` executes multiple consumers in sequence.

```
Consumer<String> logConsole = msg -> System.out.println("[INFO]: " + msg);
Consumer<String> logAudit   = msg -> System.out.println("[AUDIT]: " + msg);

Consumer<String> compositeLog = logConsole.andThen(logAudit);
compositeLog.accept("User logged in successfully");

```

#### 6.4 `Supplier<T>` (The Value Producer / Factory)

- **Functional Method:** `T get()`

- **Intent:** Takes no input parameters, but produces/supplies an instance of type `T`. Used for lazy evaluation, default generation, and factory patterns.

```
Supplier<Double> randomProvider = Math::random;
Supplier<List<String>> listFactory = ArrayList::new;

System.out.println(randomProvider.get());

```

---

### 7. Binary Extensions: `BiFunction`, `BiPredicate`, `BiConsumer`

When operations require **two inputs**, Java provides two-arity variations:

| Interface                 | Method Signature         | Functionality / Purpose                                         |
| ------------------------- | ------------------------ | --------------------------------------------------------------- |
| **`BiPredicate<T, U>`**   | `boolean test(T t, U u)` | Evaluates a boolean condition over two distinct arguments.      |
| **`BiFunction<T, U, R>`** | `R apply(T t, U u)`      | Maps two distinct inputs to a single output result of type `R`. |
| **`BiConsumer<T, U>`**    | `void accept(T t, U u)`  | Consumes two input arguments for side-effects; returns `void`.  |

```
// BiConsumer used natively in Map.forEach:
Map<String, Integer> map = Map.of("A", 1, "B", 2);
BiConsumer<String, Integer> entryPrinter = (k, v) -> System.out.println(k + " = " + v);
map.forEach(entryPrinter);

```

#### 7.1 Primitive Specializations (Avoiding Autoboxing Performance Penalties)

Using `Function<Integer, Integer>` incurs significant overhead from boxing and unboxing primitives into `java.lang.Integer` wrapper objects on the Heap.

To achieve maximum mechanical performance, the JDK provides dedicated primitive specializations:

- **Predicates:** `IntPredicate`, `LongPredicate`, `DoublePredicate`

- **Consumers:** `IntConsumer`, `LongConsumer`, `DoubleConsumer`

- **Suppliers:** `IntSupplier`, `LongSupplier`, `DoubleSupplier`

- **Unary/Binary Operators:** `IntUnaryOperator`, `LongBinaryOperator`

---

### 8. Functional Programming with Collections

Java 8 modernized collection interfaces by adding default functional methods directly onto `Iterable`, `Collection`, `List`, and `Map`:

```
                             Modern Collection Methods
                                         │
         ┌───────────────────────────────┼───────────────────────────────┐
         ▼                               ▼                               ▼
    forEach()                      removeIf()                      replaceAll()
(Iterable / Map)                  (Collection)                        (List)
Consumes each element via        Removes matching elements        Replaces every element with
Consumer / BiConsumer            using a Predicate                result of UnaryOperator

```

#### 8.1 `Iterable.forEach(Consumer<? super T> action)`

Executes a side-effect for every element directly using an internal traversal iterator:

```
List<String> cities = List.of("Mumbai", "Pune", "Bengaluru");
cities.forEach(System.out::println);

```

#### 8.2 `Collection.removeIf(Predicate<? super E> filter)`

Removes all elements that satisfy the given predicate in-place without throwing `ConcurrentModificationException`:

```
List<Integer> scores = new ArrayList<>(List.of(45, 88, 32, 95, 20));
scores.removeIf(score -> score < 40); // Safely purges failing scores
System.out.println(scores); // [45, 88, 95]

```

#### 8.3 `List.replaceAll(UnaryOperator<E> operator)`

Updates all list elements in-place with the output of the specified transformation:

```
List<String> codes = new ArrayList<>(List.of("in", "us", "uk"));
codes.replaceAll(String::toUpperCase);
System.out.println(codes); // ["IN", "US", "UK"]

```

#### 8.4 `Map.computeIfAbsent` & `Map.getOrDefault`

```
Map<String, List<String>> index = new HashMap<>();

// If key is absent, supplies a new ArrayList, adds item, and stores it in the map:
index.computeIfAbsent("Java", k -> new ArrayList<>()).add("Module 7");

```

---

### 9. High-Yield CDAC Exam Traps & Interview Questions

- **Q1: Can a lambda expression access non-final local variables from its enclosing scope?**
  - _Answer:_ **No.** Local variables captured by a lambda **must be `final` or effectively final**. If an enclosing local variable is modified after being referenced in a lambda, or if the lambda attempts to modify it, compilation fails immediately.
  - _Reason:_ Local variables live on the Stack and disappear once the method exits. Lambdas run on the Heap and may outlive the method. Java copies local variables into the lambda instance; if they were mutable, the stack and heap values would diverge.

- **Q2: Does writing a lambda expression cause the creation of a `.class` file on disk during compilation?**
  - _Answer:_ **No.** Unlike anonymous inner classes (which produce files like `Enclosing$1.class`), lambdas are translated using the **`invokedynamic` (Indy)** bytecode instruction paired with `LambdaMetafactory`. The call-site is linked dynamically at runtime, avoiding disk footprint and classloader overhead.

- **Q3: What is the difference between `Supplier<T>` and `Callable<V>`?**
  - _Answer:_ Both take zero arguments and return an object. However:
    1. `Supplier.get()` **cannot throw checked exceptions**; it is designed for general-purpose lazy data generation.
    2. `Callable.call()` declares **`throws Exception`**; it is designed for asynchronous concurrent execution within the Executor framework.

- **Q4: Can an interface have multiple abstract methods and still be used in a lambda expression?**
  - _Answer:_ **No.** A lambda expression maps directly to a **Single Abstract Method (SAM)**. If an interface declares more than one abstract method, the compiler cannot determine which method the lambda is providing the body for, causing an `incompatible types` compilation error.

- **Q5: What happens when evaluating `Function.identity()`?**
  - _Answer:_ It returns a functional transformer that **always returns its input argument untouched** (`t -> t`). It is used heavily in `Collectors.toMap()` when an object itself should serve as the value of the map entry:
    ```
    Map<Integer, Employee> map = employees.stream()
        .collect(Collectors.toMap(Employee::getId, Function.identity()));
    ```
