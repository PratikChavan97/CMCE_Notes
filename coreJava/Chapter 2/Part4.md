# Module 02: Object-Oriented Programming (OOP) in Java

## Unit 2.4: Object Relationships and Memory Management

### 1. Object Structural Relationships Hierarchy

Object-oriented systems model real-world interactions using structural relationships that govern how classes share data, delegate tasks, and manage mutual lifecycles.

```
                         Object Relationships
                                  │
         ┌────────────────────────┴────────────────────────┐
         ▼                                                 ▼
   Inheritance (IS-A)                              Association (HAS-A)
   - Tight compile-time coupling                   - Peer or container linkage
   - Derived extends Base                          - Objects communicate via references
                                                           │
                                   ┌───────────────────────┴───────────────────────┐
                                   ▼                                               ▼
                              Aggregation                                     Composition
                              (Weak HAS-A)                                    (Strong HAS-A)
                              - Independent lifecycles                        - Dependent lifecycles
                              - "Part-of" without ownership                   - Ownership & co-destruction
                              - Aggregated object survives                    - Contained object dies with parent
```

---

### 2. Association

**Association** represents a broad binary relationship where two independent classes know about each other and communicate via references.

- **Cardinality:** Can be $1$-to-$1$, $1$-to-$N$, or $N$-to-$N$.
- **Directionality:** Can be unidirectional (Class A references Class B, but B is unaware of A) or bidirectional.
- **Lifecycle:** Neither object controls the memory allocation or destruction of the other.

```java
// Unidirectional Association: Driver knows about Car
public class Car {
    private String model;
    public Car(String model) { this.model = model; }
    public void ignite() { System.out.println(model + " engine running"); }
}

public class Driver {
    private String name;
    public Driver(String name) { this.name = name; }

    // Car reference passed as an operation parameter (loose association)
    public void operate(Car vehicle) {
        System.out.print(name + " drives: ");
        vehicle.ignite();
    }
}
```

---

### 3. Aggregation (Weak "HAS-A" Relationship)

**Aggregation** is a specialized, loose form of association where:

1. One object acts as a container or collection of other objects.
2. The contained object **can exist independently** of the container.
3. The container holds a reference to an object that was instantiated **outside** of it.
4. If the container is destroyed, the aggregated child objects **survive in memory**.

```
Aggregation (Weak HAS-A):
┌──────────────────────────────┐
│ Department                   │
│ - name: "Computer Science"   │
│ - Professor[] faculty ───────┼──────┐ References external instances
└──────────────────────────────┘      │
                                      ▼
                       ┌──────────────────────────────┐
                       │ Professor Object             │ (Survives if Department
                       │ - name: "Dr. Roy"            │  is Garbage Collected)
                       └──────────────────────────────┘
```

```java
import java.util.List;

public class Professor {
    private String name;
    public Professor(String name) { this.name = name; }
    public String getName() { return name; }
}

public class Department {
    private String deptName;
    private List<Professor> faculty; // Aggregation

    // Dependency injection via constructor: instances are external
    public Department(String deptName, List<Professor> faculty) {
        this.deptName = deptName;
        this.faculty = faculty;
    }
}
```

_If `Department` is collected by the GC, the `Professor` objects remain reachable from whatever list was passed into it._

---

### 4. Composition (Strong "HAS-A" Relationship)

**Composition** is a strict, tightly coupled form of association where:

1. The container (**Composite**) owns the lifecycle of the contained parts (**Components**).
2. The component **cannot exist meaningfully without the composite**.
3. The component objects are typically instantiated **inside** the composite's constructor or initializers.
4. If the composite object is destroyed or garbage-collected, the contained objects are **marked for destruction as well**.

```
Composition (Strong HAS-A / Death Relationship):
┌────────────────────────────────────────────────────────┐
│ Order Object (Composite)                               │
│ ┌────────────────────────────────────────────────────┐ │
│ │ OrderItem Object (Component - Alloc inside Order)  │ │ (Dies immediately when
│ │ - sku: "MNT-4K", quantity: 2                       │ │  Order is garbage
│ └────────────────────────────────────────────────────┘ │  collected)
└────────────────────────────────────────────────────────┘
```

```java
import java.util.ArrayList;
import java.util.List;

public class Order {
    // Component defined as an inner or dependent entity
    public static class OrderItem {
        private String itemCode;
        public OrderItem(String itemCode) { this.itemCode = itemCode; }
    }

    private final List<OrderItem> items;

    public Order() {
        this.items = new ArrayList<>();
    }

    public void addItem(String code) {
        // Order creates and tightly owns the lifecycle of OrderItem
        this.items.add(new OrderItem(code));
    }
}
```

---

### 5. Architectural Contrast: IS-A vs. HAS-A

| Dimension              | `IS-A` Relationship (Inheritance)                                  | `HAS-A` Relationship (Composition / Aggregation)                |
| :--------------------- | :----------------------------------------------------------------- | :-------------------------------------------------------------- |
| **Language Keyword**   | `extends` (Classes), `implements` (Interfaces)                     | No dedicated keyword; modeled using instance fields             |
| **Coupling Level**     | **Very High** (Subclass exposes superclass implementation details) | **Low to Moderate** (Interactions through clean API interfaces) |
| **Binding Mechanism**  | Compile-time static hierarchy; runtime virtual dispatch            | Runtime dynamic object composition                              |
| **Flexibility**        | Rigid; cannot swap superclass behavior at runtime                  | High; referenced components can be swapped dynamically          |
| **Design Principle**   | _"Specialize when necessary"_                                      | _"Favor Composition over Inheritance"_ (GoF Invariant)          |
| **Real-World Example** | `SavingsAccount IS-A BankAccount`                                  | `Car HAS-A Engine`, `Customer HAS-A Address`                    |

---

### 6. The Complete Object Lifecycle

An object proceeds through four distinct operational phases within the JVM:

```
     1. Creation                 2. In Use               3. Eligible for GC            4. Deallocation
┌──────────────────┐       ┌──────────────────┐       ┌──────────────────────┐       ┌─────────────────┐
│ • Class Loading  │       │ • Stack frame    │       │ • Unreachable from   │       │ • GC sweeps     │
│ • Heap Allocation│ ────► │   holds reference│ ────► │   all active         │ ────► │   memory block  │
│ • Zeroing & Init │       │ • State mutated  │       │   GC Roots           │       │ • Page reused   │
└──────────────────┘       └──────────────────┘       └──────────────────────┘       └─────────────────┘
```

1. **Creation:** Class metadata verified in Metaspace; contiguous bytes reserved on Heap; fields zeroed; `<init>` constructor executes; reference handle returned.
2. **In Use (Live):** Object is reachable via at least one active chain of references originating from a **GC Root**.
3. **Eligible for Garbage Collection (Dead / Orphaned):** The object becomes completely detached from the GC Root graph. It cannot be reached or used again by executing application threads.
4. **Finalization & Deallocation:** The memory occupied by the dead object is swept and reclaimed by the Garbage Collector, making that heap offset available for future `new` allocations.

---

### 7. The 4 Types of Object References (`java.lang.ref`)

Java provides four levels of reference strengths that allow developers to control how eagerly the Garbage Collector reclaims objects (e.g., in memory caches):

```
                        Reference Strengths
                                 │
     ┌───────────────────────────┼───────────────────────────┐
     ▼                           ▼                           ▼
Strong Reference           SoftReference               WeakReference           PhantomReference
(Default in Java)          (Cleared on low RAM)        (Cleared next GC)       (Post-mortem cleanup)
```

1. **Strong Reference (Default):**

   ```java
   StringBuilder sb = new StringBuilder("Data");
   ```

   An object with an active strong reference will **never be collected by the GC**, even if the JVM runs out of heap space and crashes with `OutOfMemoryError`.

2. **Soft Reference (`SoftReference<T>`):**

   ```java
   SoftReference<Bitmap> cache = new SoftReference<>(new Bitmap());
   ```

   Collected **only when the JVM is running critically low on memory** before an `OutOfMemoryError` is thrown. Ideal for memory-sensitive caches.

3. **Weak Reference (`WeakReference<T>`):**

   ```java
   WeakReference<MetaData> weakRef = new WeakReference<>(new MetaData());
   ```

   Collected **immediately during the very next GC cycle**, regardless of whether memory is plenty or low. Used heavily in `java.util.WeakHashMap` to prevent key retention leaks.

4. **Phantom Reference (`PhantomReference<T>`):**
   Enqueued to a `ReferenceQueue` **after** the object has been completely finalized and physically reclaimed. Used as a safe, zero-leak replacement for the deprecated `finalize()` method to track off-heap resource deallocations.

---

### 8. Garbage Collection: The Mechanics

The **Garbage Collector (GC)** is an automatic memory management daemon running inside the JVM Execution Engine. Its purpose is to reclaim heap memory allocated to dead objects without manual developer intervention (eliminating C++ hazards like dangling pointers, double frees, and memory leaks).

#### 8.1 Why Java Does NOT Use Reference Counting

In simplistic systems, objects track how many pointers reference them. When the count hits zero, the object dies. **The JVM rejects reference counting** because it fails to detect **Cyclic References**:

```
Cyclic Reference Trap (Memory Leak in Reference Counting):
Node A (refCount = 1) ──points to──► Node B (refCount = 1)
   ▲                                    │
   └────────────points back to──────────┘
Even if the application loses all stack references to BOTH Node A and Node B,
their reference counts never hit 0! They leak permanently.
```

#### 8.2 The Solution: Tracing Garbage Collection (Root Reachability)

The JVM uses **Reachability Analysis**. The engine starts at designated trusted entry points called **GC Roots** and traverses the entire graph of references:

```
GC Roots (Active Stack Frames, Static Vars, JNI)
   │
   ▼
[ Live Node A ] ────► [ Live Node B ]
                         │
                         ▼
                      [ Live Node C ]

                      [ Unreachable Node X ] <═══ Cyclic ═══> [ Unreachable Node Y ]
                      (ELIGIBLE FOR GC! Not connected to any GC Root)
```

#### What Qualifies as a GC Root?

1. **Thread Stack Local Variables:** References stored in the Local Variable Array of any active stack frame.
2. **Static Variables:** References anchored to class objects loaded into Metaspace.
3. **JNI References:** Pointers held by native C/C++ methods via the Java Native Interface.
4. **Synchronization Monitors:** Objects actively locked as thread synchronization monitors (`synchronized(obj)`).

---

### 9. Four Ways an Object Becomes Eligible for GC

An object becomes eligible for Garbage Collection the instant it has **zero paths of reachability from any GC Root**.

#### 1. Nulling a Reference

Explicitly severing the link to the heap object by overwriting its reference handle with `null`:

```java
Employee emp = new Employee("Alice"); // Reachable via Stack Root
emp = null; // Reference severed! Employee object on Heap is now eligible for GC.
```

#### 2. Reassigning a Reference

Directing an existing reference handle to a newly instantiated or different object:

```java
Employee emp = new Employee("Bob"); // Heap Object 1 (Bob) allocated
emp = new Employee("Charlie");       // emp now holds address to Heap Object 2 (Charlie).
                                     // Object 1 (Bob) is orphaned -> Eligible for GC!
```

#### 3. Out-of-Scope Execution (Stack Frame Destruction)

Objects referenced purely by local variables inside a method become eligible for GC once the method exits and its stack frame is popped:

```java
public void processBatch() {
    DataBuffer buf = new DataBuffer(); // Heap allocated
    buf.load();
} // Method returns -> Stack frame popped -> 'buf' variable destroyed.
  // DataBuffer on Heap is now disconnected from GC Roots -> Eligible for GC.
```

#### 4. The Island of Isolation

A scenario where two or more objects reference each other cyclically, but the entire group has **no incoming references from any valid GC Root**:

```java
public class Node {
    Node neighbor;

    public static void main(String[] args) {
        Node a = new Node(); // Heap Object A (0x1000)
        Node b = new Node(); // Heap Object B (0x2000)

        a.neighbor = b; // 0x1000 points to 0x2000
        b.neighbor = a; // 0x2000 points to 0x1000

        // Disconnect external GC Roots:
        a = null;
        b = null;

        // BOTH Object A and Object B are now an ISLAND OF ISOLATION!
        // Even though they hold references to each other, neither is reachable
        // from any thread stack or static root. BOTH ARE COLLECTED BY GC!
    }
}
```

```
Stack (main frame)              Heap Memory
┌──────────────┐
│ a = null     │                ┌───────────────────────┐         ┌───────────────────────┐
│ b = null     │                │ Node A (0x1000)       │ ◄─────► │ Node B (0x2000)       │
└──────────────┘                │ neighbor = 0x2000     │         │ neighbor = 0x1000     │
                                └───────────────────────┘         └───────────────────────┘
                                ▲                                                         ▲
                                └──────────────── Isolated Subgraph ──────────────────────┘
                                                 (COLLECTED BY GC!)
```

---

### 10. Manual GC Invocation: `System.gc()`

The methods `System.gc()` and `Runtime.getRuntime().gc()` suggest that the JVM run a Garbage Collection cycle.

```java
System.gc(); // Merely a HINT / REQUEST to the JVM!
```

#### Critical Exam & Production Invariants:

1. **Zero Guarantees:** Calling `System.gc()` **does not guarantee** that the GC will run immediately, completely, or at all. The JVM may ignore the request entirely based on internal memory heuristics.
2. **Stop-The-World Latency:** If the JVM honors `System.gc()`, it typically triggers a **Full GC**, pausing all application threads (Stop-The-World pause).
3. **Production Anti-Pattern:** Manual invocation is heavily discouraged in enterprise systems. Production flags often disable it entirely: `-XX:+DisableExplicitGC`.

---

### 11. Legacy Finalization: The `finalize()` Method

Historically, `java.lang.Object` provided the `protected void finalize() throws Throwable` method. The JVM guaranteed that this method would execute on a dedicated thread before the object was collected.

```java
public class LegacyResource {
    @Override
    protected void finalize() throws Throwable {
        try {
            System.out.println("Releasing low-level native handles");
        } finally {
            super.finalize();
        }
    }
}
```

#### Why `finalize()` Was Deprecated (Java 9) & Marked for Removal:

1. **Unpredictable Execution:** The JVM provides zero guarantees regarding _when_ `finalize()` runs. It may be delayed indefinitely or never run before application shutdown.
2. **Performance Degradation:** Objects with finalizers cannot be collected in a single GC pass. They are promoted to a finalizer queue, drastically increasing GC pressure and pauses.
3. **Finalizer Attacks (Security Hole):** If an exception is thrown in a constructor during validation, a malicious subclass can implement `finalize()` and store a static reference to the half-initialized, unvalidated object (`this`), reviving it from death.
4. **Modern Alternative:** Use **`AutoCloseable`** paired with **`try-with-resources`** for deterministic resource cleanup, or use `java.lang.ref.Cleaner` for safe asynchronous management.

---

### 12. JVM Memory Management Architecture Summary

```
┌──────────────────────────────────────────────────────────────────────────────┐
│ Physical RAM Allocated to JVM Process                                        │
├──────────────────────────────────────────────┬───────────────────────────────┤
│ 1. On-Heap Managed Memory (-Xmx)             │ 2. Off-Heap Native Memory     │
├───────────────────────┬──────────────────────┼───────────────────────────────┤
│ Young Generation      │ Old Generation       │ Metaspace (Class Metadata)    │
│ - Eden Space          │ - Tenured Objects    │ Thread Stacks (-Xss)          │
│ - S0 (FromSpace)      │ - Survives 15 minor  │ Code Cache (JIT Machine Code) │
│ - S1 (ToSpace)        │   GC cycles (default)│ Direct ByteBuffers (NIO)      │
└───────────────────────┴──────────────────────┴───────────────────────────────┘
```

- **Minor GC:** Fast, high-frequency collection executed exclusively on the **Young Generation** (Eden + Survivors).
- **Major GC / Full GC:** High-latency collection executed on the **Old (Tenured) Generation** and the entire Heap. Pauses all application mutator threads to compact memory and eliminate fragmentation.

---

### 13. High-Yield CDAC Exam Traps & Interview Questions

- **Q1: Can an object be resurrected from being eligible for Garbage Collection?**
  - _Answer:_ **Yes.** During the execution of its legacy `finalize()` method, an object can assign `this` to an active static reference variable:
    ```java
    public static Node resurrectionAnchor;
    protected void finalize() {
        resurrectionAnchor = this; // Object is saved from collection!
    }
    ```
    _Invariant:_ The JVM will invoke `finalize()` on an object **at most once**. If the object is abandoned a second time, it will be swept immediately without invoking `finalize()` again.

- **Q2: Does an object become eligible for GC as soon as a method containing it finishes?**
  - _Answer:_ **Only if the object reference was purely local and was not returned or escaped.** If the method returns the object reference, passes it to an external collection, or assigns it to a static field, it remains attached to external GC Roots and will **not** be collected.

- **Q3: What is the primary functional difference between Composition and Aggregation?**
  - _Answer:_ **Lifecycle dependency.** In Aggregation, the contained object exists independently of the container and survives the container's destruction. In Composition, the contained object's lifecycle is bound to the owner; when the owner is destroyed, the child component is marked for destruction as well.

- **Q4: How does the JVM handle cyclic references during Garbage Collection?**
  - _Answer:_ The JVM does not use reference counters. It uses **Reachability Analysis** from GC Roots. If a cycle of objects has no path connecting it back to any active thread stack, static field, or JNI pointer (an **Island of Isolation**), the entire cycle is identified as dead memory and swept during the next collection.

- **Q5: Can `OutOfMemoryError` occur if the JVM has active Garbage Collection running?**
  - _Answer:_ **Yes.** If live, reachable objects continuously consume heap space without becoming eligible for collection (e.g., unbounded growth in a static cache `List<Object>`), the GC will run repeatedly (thrashing CPU cycles). When free memory remains insufficient to satisfy a `new` allocation request, the JVM halts execution with `java.lang.OutOfMemoryError: Java heap space`.
