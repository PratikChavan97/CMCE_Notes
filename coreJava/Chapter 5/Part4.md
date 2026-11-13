# Module 05: Packages, APIs, and Core Library Ecosystem

## Unit 5.4: Object Class and Object Methods

### 1. Introduction to the `Object` Class

In Java, **`java.lang.Object`** is the cosmic root superclass of the entire type hierarchy. Every single class in Java—whether user-defined, part of the standard library, an array, or an enum—inherits directly or indirectly from `Object`.

```
                        The Ultimate Root of Java
                            java.lang.Object
                                   │
         ┌─────────────────────────┼─────────────────────────┐
         ▼                         ▼                         ▼
   Custom Classes         Standard API Classes            Arrays
  class Account { }       class String / Integer     int[], Object[][]
```

#### 1.1 Structural Invariants

1. **Implicit Inheritance:** If a class declaration lacks an explicit `extends` clause, `javac` automatically synthesizes `extends java.lang.Object`.
2. **Universal Reference Type:** A reference variable of type `Object` can hold a pointer to any reference type on the Heap:
   ```java
   Object ref1 = new String("CDAC");
   Object ref2 = new int[10];
   Object ref3 = (Runnable) () -> System.out.println("Lambda");
   ```
3. **No Superclass:** `Object` is the only class in the Java runtime environment that has no superclass (`Object.class.getSuperclass()` returns `null`).

---

### 2. The 11 Core Methods of `java.lang.Object`

The `Object` class defines 11 methods that establish the universal contract for all Java reference types:

| Method Signature                                  | Default Behavior                                        | Overridable?    |
| :------------------------------------------------ | :------------------------------------------------------ | :-------------- |
| `public String toString()`                        | Returns `ClassName@HexHashCode`                         | ✅ Yes          |
| `public boolean equals(Object obj)`               | Reference identity comparison (`this == obj`)           | ✅ Yes          |
| `public int hashCode()`                           | Internal memory address-derived integer                 | ✅ Yes          |
| `protected Object clone()`                        | Bitwise shallow copy of fields                          | ✅ Yes          |
| `public final Class<?> getClass()`                | Returns runtime `Class` metadata descriptor             | ❌ No (`final`) |
| `protected void finalize()`                       | Garbage collection hook (Deprecated in Java 9, removed) | ⚠️ Deprecated   |
| `public final void wait()`                        | Thread releases monitor lock and sleeps                 | ❌ No (`final`) |
| `public final void wait(long timeout)`            | Timed monitor wait                                      | ❌ No (`final`) |
| `public final void wait(long timeout, int nanos)` | High-precision timed monitor wait                       | ❌ No (`final`) |
| `public final void notify()`                      | Wakes up a single thread waiting on this monitor        | ❌ No (`final`) |
| `public final void notifyAll()`                   | Wakes up all threads waiting on this monitor            | ❌ No (`final`) |

---

### 3. The `toString()` Method

The `toString()` method yields a human-readable textual representation of an object.

#### 3.1 Default Implementation

```java
// Implementation inside java.lang.Object:
public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```

When an object is passed to `System.out.println(obj)`, string concatenation (`"User: " + obj`), or string builders, `String.valueOf(obj)` is called implicitly, which invokes `obj.toString()`. If not overridden, it outputs the class name followed by the unsigned hexadecimal representation of its hash code (e.g., `com.cdac.Account@7b23ec81`).

#### 3.2 Canonical Override Pattern

```java
public class Employee {
    private final int id;
    private final String name;
    private final double salary;

    public Employee(int id, String name, double salary) {
        this.id = id;
        this.name = name;
        this.salary = salary;
    }

    @Override
    public String toString() {
        return "Employee{id=" + id + ", name='" + name + "', salary=" + salary + "}";
    }
}
```

---

### 4. The `equals()` Method vs. the `==` Operator

```
                             Object Comparison
                                     │
         ┌───────────────────────────┴───────────────────────────┐
         ▼                                                       ▼
   == Operator (Identity)                        .equals() Method (Equivalence)
   - Primitives: compares raw bit patterns       - Compares logical state/content
   - References: compares Heap memory addresses  - Can be overridden to inspect fields
   - Cannot be customized                        - Default in Object delegates to ==
```

#### 4.1 The Default Implementation

```java
// Inside java.lang.Object:
public boolean equals(Object obj) {
    return (this == obj); // Strict reference identity!
}
```

#### 4.2 The 5 Mathematical Laws of `equals()` (JLS §17.4.5)

An implementation of `equals()` must establish an **equivalence relation** for non-null references:

1. **Reflexive:** For any non-null reference $x$, $x\text{.equals}(x)$ must return `true`.
2. **Symmetric:** For any non-null references $x$ and $y$, $x\text{.equals}(y)$ must return `true` if and only if $y\text{.equals}(x)$ returns `true`.
3. **Transitive:** If $x\text{.equals}(y)$ returns `true` and $y\text{.equals}(z)$ returns `true`, then $x\text{.equals}(z)$ must return `true`.
4. **Consistent:** Repeated invocations of $x\text{.equals}(y)$ must yield identical boolean results, provided no fields are modified.
5. **Non-nullity:** For any non-null reference $x$, $x\text{.equals}(\text{null})$ must return `false` (must **never** throw `NullPointerException`).

#### 4.3 Robust Enterprise Template for `equals()`

```java
@Override
public boolean equals(Object obj) {
    // 1. Reflexivity Check: Fast-path identity optimization
    if (this == obj) return true;

    // 2. Non-nullity & Exact Type Compatibility Check
    if (obj == null || this.getClass() != obj.getClass()) return false;

    // 3. Cast to target type
    Employee other = (Employee) obj;

    // 4. State comparison across all significant fields
    return this.id == other.id &&
           Double.compare(this.salary, other.salary) == 0 &&
           Objects.equals(this.name, other.name);
}
```

> ⚠️ **`getClass()` vs. `instanceof` in `equals()`:**
>
> - `getClass() != obj.getClass()`: Strictly enforces that objects must belong to the **exact same concrete class**. This guarantees symmetry even across inheritance hierarchies.
> - `instanceof`: Permits subclasses to be considered equal to superclasses. However, if a subclass adds new state fields, maintaining symmetry and transitivity becomes mathematically impossible (The _Liskov Substitution / Point-ColorPoint_ dilemma).

---

### 5. The `hashCode()` Method

The `hashCode()` method returns an integer hash value used in hash-based collection structures: **`HashMap`**, **`HashSet`**, and **`Hashtable`**.

```
              Bucket Hashing Pipeline inside HashMap
                      key.hashCode()
Key: "Emp-101" ──────────────────────────► Integer: 2049182
                                                │
                                                ▼  h & (table.length - 1)
                                      Bucket Array Index: 6
                               ┌───┬───┬───┬───┬───┬───┬───┐
                               │ 0 │ 1 │ 2 │ 3 │ 4 │ 5 │ 6 │ ◄── Enters Linked Bucket Node
                               └───┴───┴───┴───┴───┴───┴───┘
```

#### 5.1 The `equals()` and `hashCode()` Contract (CRITICAL)

The contract between `equals()` and `hashCode()` consists of three rules:

$$
x\text{.equals}(y) == \text{true} \implies x\text{.hashCode}() == y\text{.hashCode}()
$$

$$
x\text{.hashCode}() == y\text{.hashCode}() \centernot\implies x\text{.equals}(y) == \text{true} \quad (\text{Hash Collision})
$$

1. **Internal Consistency:** If an object’s state used in `equals()` has not changed, `hashCode()` must consistently yield the same integer throughout application execution.
2. **Equality Implication:** If two objects are equal according to `equals()`, their `hashCode()` values **MUST be identical**.
3. **Collision Allowance:** If two objects have the same `hashCode()`, they are **not necessarily equal** (a hash collision occurs, and `equals()` resolves bucket candidates).

#### 5.2 The Hash Violation Catastrophe

If you override `equals()` but **fail to override `hashCode()`**, the class violates the contract. When used as keys in a `HashMap`, equal objects generate different hash codes and land in separate internal buckets:

```java
Employee e1 = new Employee(101, "Aarav", 50000);
Employee e2 = new Employee(101, "Aarav", 50000);

System.out.println(e1.equals(e2)); // true!

Map<Employee, String> map = new HashMap<>();
map.put(e1, "Software Engineer");

// Lookup with logically identical instance:
System.out.println(map.get(e2)); // Returns NULL! Key lookup completely fails!
```

#### 5.3 Overriding `hashCode()`

```java
@Override
public int hashCode() {
    // Prime multiplier 31 reduces hash collisions across field combinations
    return Objects.hash(id, name, salary);
}
```

---

### 6. The `getClass()` Method

```java
public final Class<?> getClass()
```

- **Purpose:** Returns the runtime `Class` instance representing the actual concrete type of the object stored on the Heap.
- **Non-Overridable:** Declared `final` to prevent subclasses from altering their runtime class representation.
- **Reflection Engine:** Serves as the primary entry point to Java Reflection (`obj.getClass().getDeclaredMethods()`).

```java
Number n = Integer.valueOf(42);
System.out.println(n.getClass().getName()); // java.lang.Integer (Runtime Heap type, NOT Number!)
```

---

### 7. The `clone()` Method & Object Duplication

```java
protected Object clone() throws CloneNotSupportedException
```

The `clone()` method creates a field-by-field copy of the target object.

```
                           Object.clone() Mechanics
                                      │
               Is the class an instanceof Cloneable?
                                ┌─────┴─────┐
                           Yes  │           │ No
                                ▼           ▼
           Allocates new Heap object     Throws CloneNotSupportedException!
           Field-by-field bitwise copy
```

#### 7.1 Clone Contract Requirements:

1. The class **must implement the `java.lang.Cloneable` marker interface**. If not implemented, calling `super.clone()` throws **`java.lang.CloneNotSupportedException`**.
2. Override `clone()` as `public` to make it accessible to outside packages (its default visibility in `Object` is `protected`).

```java
public class Department implements Cloneable {
    private String name;
    private int[] metrics;

    public Department(String name, int[] metrics) {
        this.name = name;
        this.metrics = metrics;
    }

    // Shallow Copy Implementation:
    @Override
    public Object clone() throws CloneNotSupportedException {
        return super.clone(); // Native bitwise copy of fields
    }

    // Deep Copy Implementation:
    public Department deepClone() throws CloneNotSupportedException {
        Department cloned = (Department) super.clone();
        cloned.metrics = this.metrics.clone(); // Duplicate reference arrays independently!
        return cloned;
    }
}
```

---

### 8. Concurrency Monitor Methods: `wait()`, `notify()`, `notifyAll()`

The JVM binds an internal **Intrinsic Lock (Monitor)** and a **Wait Set** to every single object on the Heap.

```
                         JVM Monitor State Machine
                                  │
    ┌─────────────────────────────┴─────────────────────────────┐
    │ Thread acquires object monitor via synchronized(lock)     │
    └─────────────────────────────┬─────────────────────────────┘
                                  │
                ┌─────────────────┴─────────────────┐
                │ Calling lock.wait()               │
                ▼                                   ▼
┌───────────────────────────────┐   ┌───────────────────────────────┐
│ Monitor Lock is RELEASED!     │   │ Calling lock.notify()         │
│ Thread suspended & enqueued   │   │ Wakes 1 arbitrary thread from │
│ into Object's Wait Set        │   │ the Wait Set -> Re-contends   │
└───────────────────────────────┘   │ for monitor lock              │
                                    └───────────────────────────────┘
```

#### 8.1 Critical Monitor Rules

1. **Synchronized Scope Invariant:** A thread **must hold the monitor lock** of the target object before invoking `wait()`, `notify()`, or `notifyAll()`.
2. **The IllegalMonitorStateException:** If a thread invokes these methods outside a `synchronized(targetObject)` block, the JVM immediately throws **`java.lang.IllegalMonitorStateException`**.
3. **Lock Release:** `wait()` **releases the monitor lock**, allowing other threads to enter synchronized blocks on that object. (In contrast, `Thread.sleep()` pauses execution but **holds the lock**).

---

### 9. Object Comparison & Sorting Fundamentals

To compare and sort objects, the Java library relies on two distinct interfaces: **`Comparable`** and **`Comparator`**.

```
                        Object Ordering Strategies
                                     │
         ┌───────────────────────────┴───────────────────────────┐
         ▼                                                       ▼
java.lang.Comparable<T>                                java.util.Comparator<T>
- "Natural" ordering                                   - External/Custom ordering strategies
- Implemented inside target domain class               - Implemented as standalone utility classes/lambdas
- Method: int compareTo(T o)                           - Method: int compare(T o1, T o2)
```

#### 9.1 `Comparable<T>` Implementation

```java
public class Student implements Comparable<Student> {
    private final int rollNumber;
    private final String name;

    public Student(int rollNumber, String name) {
        this.rollNumber = rollNumber;
        this.name = name;
    }

    @Override
    public int compareTo(Student other) {
        // Natural ordering: ascending order by rollNumber
        return Integer.compare(this.rollNumber, other.rollNumber);
    }
}
```

#### Comparison Return Invariants:

$$
\text{compareTo}(o) =
\begin{cases}
< 0 & \text{if } \text{this} < o \quad (\text{precedes } o) \\
0 & \text{if } \text{this} == o \quad (\text{equal ranking}) \\
> 0 & \text{if } \text{this} > o \quad (\text{succeeds } o)
\end{cases}
$$

---

### 10. High-Yield CDAC Exam Traps & Interview Questions

- **Q1: What happens if an object used as a key in a `HashMap` is mutated after being inserted?**
  - _Answer:_ **The entry becomes unretrievable (Memory Leak).**
  - _Explanation:_ If a key’s state changes, its `hashCode()` changes as well. When querying `map.get(key)`, the map calculates the new hash code, which routes the lookup to the wrong bucket. The original value remains stuck in the old bucket and cannot be fetched or easily removed.

- **Q2: Why must `wait()` always be called inside a `while` loop rather than an `if` block?**
  - _Answer:_ To defend against **Spurious Wakeups**. A thread waiting on a monitor can wake up without receiving a notification, interrupt, or timeout. Using a `while` loop guarantees the condition is re-evaluated before execution continues:
    ```java
    synchronized (lock) {
        while (!conditionMet) {
            lock.wait(); // Re-evaluates condition when awakened
        }
        processAction();
    }
    ```

- **Q3: Why is `clone()` declared `protected` inside `java.lang.Object`?**
  - _Answer:_ To prevent outside code from cloning an object unless the author has explicitly opted in by overriding `clone()`, declaring it `public`, and implementing `Cloneable`.

- **Q4: Can we override `equals()` without checking `getClass()` or `instanceof`?**
  - _Answer:_ **No.** Omitting type verification causes a `ClassCastException` whenever an incompatible object type is passed to `equals(Object obj)`.

- **Q5: Does `Object.hashCode()` return the exact physical RAM address of an object?**
  - _Answer:_ **No.** While the default identity hash code in HotSpot is derived from internal native memory addresses or random number sequences generated during object allocation, it is not an exposed physical pointer. Pointers change as the Garbage Collector compacts memory across generations, but an object's identity hash code remains constant.
