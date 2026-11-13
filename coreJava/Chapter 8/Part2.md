# Module 08: Reflection, Annotations, and JVM Mechanics

## Unit 8.2: Advanced Java Concepts

### 1. Java Memory Model (JMM) Fundamentals

The **Java Memory Model (JMM)**, codified under **JSR 133** (Java 5+), formalizes the operational contract between the Java Virtual Machine, the execution thread stacks, local CPU cache hierarchies, and physical hardware RAM.

The JMM defines when writes to a shared variable performed by one thread become visible to reads executed by another thread (**Memory Visibility**), and restricts the conditions under which the compiler, JIT, or CPU execution pipeline can reorder machine instructions (**Ordering**).

```
                      Java Memory Model Architectural Layout
┌─────────────────────────────────────────────────────────────────────────────┐
│ Thread 1 (Core 1)                         Thread 2 (Core 2)                 │
│ ┌────────────────────────┐                ┌────────────────────────┐        │
│ │ Thread Stack           │                │ Thread Stack           │        │
│ │ - Local Primitive Data │                │ - Local Primitive Data │        │
│ │ - Object References    │                │ - Object References    │        │
│ └───────────┬────────────┘                └───────────┬────────────┘        │
│             │                                         │                     │
│             ▼                                         ▼                     │
│ ┌────────────────────────┐                ┌────────────────────────┐        │
│ │ CPU L1/L2 Cache Copy   │                │ CPU L1/L2 Cache Copy   │        │
│ └───────────┬────────────┘                └───────────┬────────────┘        │
└─────────────┼─────────────────────────────────────────┼─────────────────────┘
              │                                         │
              ▼                                         ▼
═══════════════════════════════════════════════════════════════════════════════
                      Main Shared Memory (JVM Heap / RAM)
   • Shared Objects, Instance Fields, Static Fields, Array Slots
═══════════════════════════════════════════════════════════════════════════════
```

#### Core JMM Guarantees:

1. **Atomicity:** Ensures operations complete uninterrupted or not at all (guaranteed for 32-bit primitive reads/writes; compound operations like `i++` are non-atomic).
2. **Visibility:** Guarantees that changes made by one thread to shared variables are propagated accurately to other threads (enforced via `volatile`, `synchronized`, and `final`).
3. **Happens-Before Relationship:** A formal partial ordering relation over memory actions:
   - **Program Order Rule:** Each action in a thread happens-before every action that comes later in program order.
   - **Monitor Lock Rule:** An unlock on a monitor lock happens-before every subsequent lock on the same monitor.
   - **Volatile Variable Rule:** A write to a `volatile` variable happens-before every subsequent read of that same variable.
   - **Thread Start Rule:** Calling `thread.start()` happens-before any actions in the started thread.
   - **Thread Termination Rule:** Any action in a thread happens-before any other thread successfully returns from `thread.join()`.

---

### 2. JVM Runtime Data Areas: Stack vs. Heap Memory

The JVM specifications segment execution memory into distinct functional zones:

```
                            JVM Memory Topography
                                      │
         ┌────────────────────────────┴────────────────────────────┐
         ▼                                                         ▼
Thread-Private Memory Areas                               Shared Memory Areas
(Allocated per thread; zero contention)                   (Shared across all threads; GC managed)
├── JVM Stack Frames (Local variables, Operands)          ├── Managed Heap (Young/Old Generations)
├── Program Counter (PC) Registers                        └── Metaspace (Loaded Classes, Bytecode)
└── Native Method Stacks (C/C++ runtime)
```

```
STACK MEMORY (Thread-Specific)                      HEAP MEMORY (Shared Globally)
┌──────────────────────────────────────┐          ┌──────────────────────────────────────────────┐
│ [Frame: processOrder()]              │          │ 0x8200: Order Object                         │
│ - int orderId = 101                  │          │ - id: 101                                    │
│ - Order ref ───────────(0x8200)──────┼─────────►│ - customer ──────────────┐                  │
│                                      │          │                              │               │
│ [Frame: main()]                      │          ├──────────────────────────────┼───────────────┤
│ - Customer custRef ────(0x9400)──────┼─────────►│ 0x9400: Customer Object      │               │
│                                      │          │ - name: "Aarav" ◄────────────┘               │
└──────────────────────────────────────┘          └──────────────────────────────────────────────┘
```

| Architectural Metric | Stack Memory                                                                                                 | Heap Memory                                                                                          |
| :------------------- | :----------------------------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------- |
| **Allocation Scope** | **Thread-private**. Created when a thread starts.                                                            | **Global / Shared**. Accessible by all threads.                                                      |
| **What It Stores**   | Stack frames, local primitive variables, method execution state, and **reference pointers** to heap objects. | **All Object instances**, arrays, and their instance fields.                                         |
| **Lifecycle**        | Scope-bound. Automatically allocated when a method is invoked and cleared when the frame returns.            | Dynamic. Survives method exits; managed and reclaimed exclusively by the **Garbage Collector (GC)**. |
| **Error Modes**      | **`java.lang.StackOverflowError`** (e.g., deep or infinite recursion).                                       | **`java.lang.OutOfMemoryError: Java heap space`** (object allocation failure).                       |
| **Tuning Flags**     | `-Xss<size>` (e.g., `-Xss1m`, default 1 MB per thread).                                                      | `-Xms<size>` (initial), `-Xmx<size>` (maximum).                                                      |

---

### 3. Method Area and Metaspace

The **Method Area** is a logical partition of shared memory that stores class structures, runtime constant pools, field/method metadata, method bytecode, and static variables.

```
Evolution of Class Metadata Storage:
Java 7 and Earlier:
  [ JVM Heap (-Xmx) ]             [ PermGen (Permanent Generation) (-XX:MaxPermSize) ]
                                   - Fixed size inside JVM memory layout
                                   - Stored Class metadata, interned Strings, static variables
                                   - Prone to java.lang.OutOfMemoryError: PermGen space

Java 8 and Later:
  [ Managed JVM Heap (-Xmx) ]     [ Metaspace (Native OS Memory) ]
  - Objects and Arrays            - Off-heap native memory (bounded only by physical OS RAM)
  - String Constant Pool (Moved)  - Stores class bytecode, runtime constant pool, vtables
  - Static References (Moved)     - Configured via -XX:MaxMetaspaceSize
```

#### Why PermGen was Replaced with Metaspace:

1. **Dynamic Resizing:** PermGen had a fixed contiguous memory ceiling (`-XX:MaxPermSize`). Heavy class-loading applications (e.g., application servers reloading WAR files) frequently crashed with `OutOfMemoryError: PermGen space`.
2. **Native Memory Scaling:** Metaspace allocates directly from host OS native memory via C-level `malloc`, growing dynamically as needed.
3. **Moved Components:** Static variables and the String Constant Pool (SCP) were migrated out of class metadata storage and into the **standard JVM Heap** starting in Java 7.

---

### 4. Garbage Collector (GC) Fundamentals

The **Garbage Collector** is the automatic memory management subsystem inside the JVM that reclaims memory occupied by unreferenced heap objects.

#### 4.1 Object Reachability & GC Roots

Java uses **Tracing Garbage Collection** via **Root Reachability Analysis** (not reference counting, which fails on cyclic dependencies $A \rightarrow B \rightarrow A$).

```
                      GC Root Reachability Tree
 [ Active Thread Stack Frame ]  [ Static Class Variables ]  [ JNI Native Handles ]
          (GC Root)                      (GC Root)                    (GC Root)
              │                              │                            │
              ▼                              ▼                            │
      ┌───────────────┐              ┌───────────────┐                    │
      │ Object A      │              │ Object C      │                    │
      └───────┬───────┘              └───────────────┘                    │
              │                                                           │
              ▼                                                           ▼
      ┌───────────────┐              ┌───────────────┐            ┌───────────────┐
      │ Object B      │              │ Object D      │◄───────────┤ Object E      │
      └───────────────┘              └───────┬───────┘            └───────────────┘
                                             │
      ═══════════════════════════════════════╪═════════════════════════════════════
         UNREACHABLE OBJECTS (ORPHANED)      │
                                             ▼
                                     ┌───────────────┐
                                     │ Object X      │ ◄─── Cyclic Reference Island!
                                     └───┬───────▲───┘      Eligible for GC sweep!
                                         │       │
                                         ▼───────┘
                                     ┌───────────────┐
                                     │ Object Y      │
                                     └───────────────┘
```

**Common GC Roots:**

1. Local variables and parameters within active method frames on thread stacks.
2. Active running Thread instances.
3. Static variables referenced by loaded `Class` objects in Metaspace.
4. JNI (Java Native Interface) global and local C-pointers.

#### 4.2 Generational Garbage Collection Hypothesis

Most modern garbage collectors leverage the **Weak Generational Hypothesis**:

1. _Most allocated objects become unreachable shortly after creation (die young)._
2. _Very few references exist from older objects to newer objects._

```
                             JVM Generational Heap Structure
┌───────────────────────────────────────────────────────────┬────────────────────────────────┐
│ Young Generation                                          │ Old (Tenured) Generation       │
│ ┌─────────────────────────┬───────────────┬─────────────┐ │ ┌────────────────────────────┐ │
│ │ Eden Space              │ Survivor S0   │ Survivor S1 │ │ │ Long-lived objects promoted  │ │
│ │ (Fresh new allocations) │ (FromSpace)   │ (ToSpace)   │ │ │ after surviving N minor GC   │ │
│ └─────────────────────────┴───────────────┴─────────────┘ │ │ tenuring cycles (threshold) │ │
│                                                           │ └────────────────────────────┘ │
└───────────────────────────────────────────────────────────┴────────────────────────────────┘
 ◄────────────── Minor GC (Fast & Frequent) ───────────────► ◄── Major/Full GC (Infrequent) ─►
```

#### 4.3 Generational Lifecycle Flow:

1. All new instances are allocated in the **Eden Space**.
2. When Eden fills, a **Minor GC** triggers:
   - Live surviving objects are moved to Survivor space **S0**. Eden is cleared.
3. On the next Minor GC cycle, survivors from Eden and **S0** are aged and copied to **S1**. S0 and Eden are cleared.
4. An object increments an internal age counter (**Tenuring Threshold**) every time it survives a sweep.
5. When an object's age exceeds the threshold (default: 15, configured via `-XX:MaxTenuringThreshold`), it is promoted to the **Old (Tenured) Generation**.
6. When the Old Generation fills, a **Major GC / Full GC** runs, sweeping and compacting the tenured space.

#### 4.4 The 3 Core GC Phases

- **Mark:** Traverses GC roots and marks all live, reachable objects in memory.
- **Sweep:** Sweeps through the heap, reclaiming memory slots of un-marked objects.
- **Compact:** Shifts surviving live objects to the start of the memory space, resolving heap fragmentation so large arrays can be allocated in contiguous space.

---

### 5. Java Reference Strengths

The `java.lang.ref` package provides reference wrappers that give developers control over object lifetimes and interactions with the Garbage Collector.

```
                           Reference Strength Hierarchy
 (Strongest)                                                               (Weakest)
    Strong Reference  ───►  Soft Reference  ───►  Weak Reference  ───►  Phantom Reference
```

```
┌────────────────────────────────────────────────────────────────────────┐
│ Reference Types Comparison                                             │
├───────────────────┬───────────────────┬────────────────────────────────┤
│ Reference Type    │ GC Reclamation    │ Primary Real-World Application │
├───────────────────┼───────────────────┼────────────────────────────────┤
│ **Strong**        │ Never reclaimed   │ Normal business objects        │
│ `Object obj = new`│ while reachable   │ (`User u = new User()`)        │
├───────────────────┼───────────────────┼────────────────────────────────┤
│ **Soft**          │ Reclaimed **only**│ In-memory caches               │
│ `SoftReference<T>`│ if JVM hits low   │ (e.g., Image/Document buffers  │
│                   │ memory conditions │ that can be rebuilt if dropped)│
├───────────────────┼───────────────────┼────────────────────────────────┤
│ **Weak**          │ Reclaimed on the  │ Canonical mappings & metadata  │
│ `WeakReference<T>`│ **very next** GC  │ (`WeakHashMap`, ThreadLocalMap │
│                   │ cycle, regardless │ keys preventing memory leaks)  │
├───────────────────┼───────────────────┼────────────────────────────────┤
│ **Phantom**       │ Reclaimed during  │ Clean-up post-mortem actions   │
│`PhantomReference` │ GC; `.get()`      │ replacing finalized hooks      │
│                   │ always returns    │ (`Cleaner` API, direct buffer  │
│                   │ `null`            │ off-heap memory deallocation)  │
└───────────────────┴───────────────────┴────────────────────────────────┘
```

#### Code Demonstration:

```
// 1. Strong Reference
String data = new String("Crucial Data");

// 2. Soft Reference (Memory-Sensitive Cache)
SoftReference<String> softCache = new SoftReference<>(data);

// 3. Weak Reference (Transient Association)
WeakReference<String> weakRef = new WeakReference<>(data);

data = null; // Clear strong reference handle

// Force explicit GC (for demonstration purposes only)
System.gc();

// Weak reference is reclaimed immediately:
System.out.println("Weak: " + weakRef.get());   // Prints: null

// Soft reference remains alive if JVM heap space is sufficient:
System.out.println("Soft: " + softCache.get()); // Prints: "Crucial Data"
```

---

### 6. Class Loading Mechanism (The 3-Phase Lifecycle)

The Java ClassLoader subsystem dynamically loads, links, and initializes binary `.class` bytecode files into memory on demand.

```
                         Class Loading Lifecycle
                                    │
    ┌───────────────────────────────┼───────────────────────────────┐
    ▼                               ▼                               ▼
1. Loading                      2. Linking                      3. Initialization
Reads .class binary byte        Validates and prepares          Executes <clinit>
stream; constructs Class<?>     internal structures             • static variables initialized
instance on the JVM Heap        ├── Verification                • static blocks executed
                                ├── Preparation
                                └── Resolution
```

#### Detailed Phase Breakdown:

1. **Phase 1: Loading**
   - Locates the `.class` byte array corresponding to the fully qualified class name (from disk, JAR, network stream).
   - Parses the byte array into JVM C++ internal runtime metadata structures in Metaspace.
   - Instantiates a `java.lang.Class<T>` instance on the JVM Heap to represent the type.

2. **Phase 2: Linking**
   - **Verification:** Validates bytecode safety (checks magic number `0xCAFEBABE`, verifies stack bounds, ensures no illegal pointer manipulation, validates type safety).
   - **Preparation:** Allocates memory for **static fields** and initializes them to their **default zero-values** (`0`, `null`, `false`), _not_ their assigned values.
   - **Resolution:** Replaces symbolic references in the runtime constant pool with direct memory pointers.

3. **Phase 3: Initialization (`<clinit>`)**
   - Executes the class's static initializers: assigns declared static field values and runs `static { ... }` blocks in top-to-bottom source order.

---

### 7. ClassLoader Hierarchy & Delegation Model

ClassLoaders are organized into a strict hierarchical parent-child structure.

```
                           ClassLoader Hierarchy
                                     │
                    Bootstrap ClassLoader (C++ Native)
                    Loads java.base, core JDK ($JAVA_HOME/lib)
                                     ▲
                                     │ Parent
                        Platform / Extension ClassLoader
                        Loads extensions, standard platform modules
                                     ▲
                                     │ Parent
                        Application / System ClassLoader
                        Loads classes on application -classpath / -cp
                                     ▲
                                     │ Parent
                         Custom User-Defined ClassLoader
                         (Plugin Loaders, Network Decrypting Loaders)
```

#### The Delegation Principle:

When a ClassLoader receives a request to load a class:

1. It first checks its local cache to see if the class has already been loaded.
2. If not, it **delegates the request up to its parent ClassLoader**.
3. This delegation cascades upward to the **Bootstrap ClassLoader**.
4. Only if the parent ClassLoader reports that it cannot find the class does the child ClassLoader invoke its own `findClass()` method to search its configured paths.

```
// Visualizing the parent-child delegation link:
ClassLoader appLoader = Application.class.getClassLoader();
System.out.println(appLoader);                         // jdk.internal.loader.ClassLoaders$AppClassLoader
System.out.println(appLoader.getParent());             // jdk.internal.loader.ClassLoaders$PlatformClassLoader
System.out.println(appLoader.getParent().getParent()); // null (Bootstrap Loader is native C++)
```

> ⚠️ **Why Does the Delegation Model Exist? (Security & Integrity):**
> It prevents malicious code from replacing core JVM types. If an attacker introduces a rogue `java.lang.String` on the application classpath, the application ClassLoader delegates loading up to the Bootstrap ClassLoader, which loads the genuine platform `String` from `java.base` and ignores the rogue implementation.

---

### 8. Java Platform Module System (JPMS - Java 9+)

Introduced under **Project Jigsaw (Java 9)**, JPMS organizes packages into higher-level units called **Modules**. A module declares its boundaries and dependencies using a **`module-info.java`** file located at the source root.

```
                Classic Classpath vs. Java 9 Modular System
┌─────────────────────────────────┐       ┌─────────────────────────────────┐
│ Flat Classpath (Pre-Java 9)     │       │ Explicit Modules (Java 9+)      │
│ • "JAR Hell" collisions         │       │ • Strong encapsulation          │
│ • No boundary enforcement       │ ────► │ • Explicit module dependency    │
│ • Runtime ClassNotFound failures│       │   graphs (module-info.java)     │
│ • All public classes are open   │       │ • Unexported packages hidden!   │
└─────────────────────────────────┘       └─────────────────────────────────┘
```

#### Structure of `module-info.java`:

```
module com.cdac.orderservice {
    // 1. Explicit Dependencies (what this module requires)
    requires java.sql;
    requires transitive com.cdac.common; // Transitive dependency: callers inherit this dependency

    // 2. Explicit Exports (what public packages are exposed to external consumers)
    exports com.cdac.orderservice.api;
    exports com.cdac.orderservice.dto;

    // 3. Selective Reflection Access (Strong Encapsulation)
    // Allows Spring/Hibernate to inspect private fields reflectively without exporting API
    opens com.cdac.orderservice.entity to org.hibernate.orm.core, spring.core;
}
```

#### Core Benefits of JPMS:

- **Strong Encapsulation:** A `public` class inside a package is **no longer accessible** outside its module unless its package is explicitly exported via `exports`.
- **Reliable Configuration:** The JVM checks for missing module dependencies at startup rather than failing at runtime with `NoClassDefFoundError`.
- **Minimal Runtimes (`jlink`):** Allows developers to assemble stripped-down JVM runtimes containing only the specific platform modules required by their application (e.g., creating 35 MB microservice container images).

---

### 9. Immutable Objects & Defensive Copying

An **Immutable Object** is an instance whose observable state cannot change after construction.

#### The 5 Canonical Rules for Immutability:

1. Declare the class as **`final`** (prohibits subclasses from overriding methods to introduce mutable state).
2. Make all fields **`private`** and **`final`**.
3. Provide **no setter/mutator methods**.
4. If the class contains fields that reference mutable objects (e.g., `java.util.Date`, arrays, collections), implement **Defensive Copying** in constructors and getter methods.
5. Ensure exclusive access to any mutable components (do not share internal references directly).

```
                      The Defensive Copying Principle
 Incoming Request                              Constructed Immutable Instance
┌─────────────────────────┐                    ┌─────────────────────────┐
│ Date callerDate         │ ─── Clone / Copy ─►│ private final Date date │
│ (Caller retains handle) │                    │ (Isolated copy in Heap) │
└─────────────────────────┘                    └─────────────────────────┘
                                                            │
 Client Retrieval                                           │
┌─────────────────────────┐                    Clone / Copy │
│ Date leakedDate         │ ◄── Clone / Copy ───────────────┘
│ (Mutating this cannot   │
│  affect internal state) │
└─────────────────────────┘
```

#### Production-Grade Immutable Class Implementation:

```
import java.util.Date;
import java.util.List;
import java.util.ArrayList;
import java.util.Collections;

public final class ImmutableReport {
    private final String reportName;
    private final Date generatedDate;
    private final List<String> metrics;

    public ImmutableReport(String reportName, Date generatedDate, List<String> metrics) {
        this.reportName = reportName;

        // 1. Defensive copy of mutable Date argument
        this.generatedDate = (generatedDate != null) ? new Date(generatedDate.getTime()) : null;

        // 2. Defensive copy of mutable List
        this.metrics = (metrics != null) ? new ArrayList<>(metrics) : new ArrayList<>();
    }

    public String getReportName() {
        return reportName; // String is inherently immutable; safe to return directly
    }

    public Date getGeneratedDate() {
        // 3. Defensive copy returned: prevents caller from mutating internal state via setTime()
        return (generatedDate != null) ? new Date(generatedDate.getTime()) : null;
    }

    public List<String> getMetrics() {
        // 4. Return unmodifiable wrapper: prevents caller from calling .add() or .clear()
        return Collections.unmodifiableList(metrics);
    }
}
```

---

### 10. Shallow Copy vs. Deep Copy

Object duplication differs in how referenced heap dependencies are handled:

```
Shallow Copy:                                  Deep Copy:
┌───────────────┐                              ┌───────────────┐
│ Original Obj  │                              │ Original Obj  │
│ - id = 100    │                              │ - id = 100    │
│ - addrRef ────┼──────┐                       │ - addrRef ────┼──────► [ Address Object A ]
└───────────────┘      │                       └───────────────┘
                       ▼
┌───────────────┐   [ Address Object A ]       ┌───────────────┐
│ Cloned Obj    │   (SHARED!)                  │ Cloned Obj    │
│ - id = 100    │                              │ - id = 100    │
│ - addrRef ────┼──────┘                       │ - addrRef ────┼──────► [ Address Object B ]
└───────────────┘                              └───────────────┘        (INDEPENDENT COPY)
```

| Trait                 | Shallow Copy                                                                                          | Deep Copy                                                                             |
| :-------------------- | :---------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------ |
| **Primitives**        | Copied by value into the clone                                                                        | Copied by value into the clone                                                        |
| **Object References** | **Copies reference pointers only**. Both original and clone point to the **same underlying objects**. | **Recursively duplicates all referenced objects**, creating isolated heap instances.  |
| **Mutation Risk**     | Mutating an internal referenced object in the clone **corrupts the original**.                        | Fully isolated. Mutations to the clone have no effect on the original.                |
| **Implementation**    | Default `Object.clone()` behavior.                                                                    | Custom copy constructors, serialization/deserialization, or manual recursive copying. |

---

### 11. Design Principles & The SOLID Framework

The **SOLID** principles (formalized by Robert C. Martin / "Uncle Bob") provide core architectural guidelines for clean, maintainable, decoupled object-oriented systems.

```
                              The SOLID Principles
                                        │
    ┌───────────┬───────────┬───────────┴───────────┬───────────┐
    ▼           ▼           ▼                       ▼           ▼
Single-       Open-       Liskov                  Interface   Dependency
Responsibility Closed     Substitution            Segregation Inversion
(SRP)         (OCP)       (LSP)                   (ISP)       (DIP)
```

#### 1. Single Responsibility Principle (SRP)

- _Rule:_ A class should have **one, and only one, reason to change**.
- _Violation:_ An `OrderService` that validates line items, writes to the SQL database, formats invoice PDFs, and sends SMTP emails.
- _Fix:_ Split responsibilities across `OrderValidator`, `OrderRepository`, `InvoicePdfGenerator`, and `NotificationService`.

#### 2. Open-Closed Principle (OCP)

- _Rule:_ Software artifacts should be **open for extension, but closed for modification**.
- _Application:_ Add new behaviors by introducing new subclasses or interface implementations rather than modifying existing, tested classes with sprawling `switch` or `if-else` blocks.

#### 3. Liskov Substitution Principle (LSP)

- _Rule:_ Derived types must be completely **substitutable for their base types** without altering program correctness.
- _Classic Violation:_ A `Square` class extending a `Rectangle` class. Modifying `setHeight()` on `Square` implicitly alters `setWidth()`, breaking assumptions made by callers that work with `Rectangle`.

#### 4. Interface Segregation Principle (ISP)

- _Rule:_ Clients should **never be forced to depend on interfaces they do not use**.
- _Application:_ Prefer small, focused interfaces (`Printable`, `Closeable`, `Serializable`) over large, monolithic interfaces (`AllInOneWorker` with 30 methods).

#### 5. Dependency Inversion Principle (DIP)

- _Rule:_ High-level modules should **not depend on low-level modules**; both should depend on **abstractions**.
- _Application:_ Code against interfaces rather than concrete implementations:

  ```
  // VIOLATION: Directly coupled to concrete implementation
  private MySQLDatabase db = new MySQLDatabase();

  // CORRECT: Decoupled via abstraction; supports dependency injection
  private DatabaseChannel db;
  public OrderService(DatabaseChannel db) { this.db = db; }
  ```

---

### 12. Design Patterns Deep Dive

Design patterns are reusable, standard solutions to common software design challenges.

```
                        Core Enterprise Design Patterns
                                       │
         ┌─────────────────────────────┼─────────────────────────────┐
         ▼                             ▼                             ▼
Creational Patterns            Creational Patterns           Behavioral Patterns
- Singleton Pattern            - Builder Pattern             - Strategy Pattern
- Factory Pattern              - (Step-by-step assembly)     - (Interchangeable algorithms)
```

---

### 13. The Singleton Design Pattern

Guarantees that a class has **only one instance** throughout the application lifetime and provides a global access point to it.

```
Double-Checked Locking Flow:
Thread arrives at getInstance()
  │
  ▼
Is instance == null?
  ├─ No  ──► Return existing instance (Fast path, NO lock acquired!)
  └─ Yes ──► Synchronize on Singleton.class
               │
               ▼
             Is instance STILL null? (Double Check)
               ├─ Yes ──► Instantiate: instance = new Singleton()
               └─ No  ──► Return existing instance
```

#### 13.1 Production Implementation: Thread-Safe Double-Checked Locking

```
public class DatabaseConnectionPool {
    // volatile is MANDATORY to prevent instruction reordering!
    private static volatile DatabaseConnectionPool instance;

    // Private constructor blocks instantiation via 'new'
    private DatabaseConnectionPool() {
        if (instance != null) {
            // Defense against reflection attacks
            throw new RuntimeException("Instantiate via getInstance() only.");
        }
    }

    public static DatabaseConnectionPool getInstance() {
        if (instance == null) { // First Check (Non-synchronized for speed)
            synchronized (DatabaseConnectionPool.class) {
                if (instance == null) { // Second Check (Synchronized for safety)
                    instance = new DatabaseConnectionPool();
                }
            }
        }
        return instance;
    }
}
```

#### 13.2 The Modern Standard: Enum Singleton (Joshua Bloch)

The most resilient singleton implementation in Java:

```
public enum EnvironmentRegistry {
    INSTANCE;

    private String activeProfile = "PRODUCTION";

    public String getActiveProfile() { return activeProfile; }
    public void setActiveProfile(String p) { this.activeProfile = p; }
}
```

- **Guarantees:** Inherently thread-safe, handles serialization/deserialization safely without extra hooks, and prevents reflection-based instantiation attacks (the JVM prohibits reflective construction of enums).

---

### 14. The Factory Design Pattern

Defines an interface for creating objects, delegating the instantiation logic to subclasses or a specialized factory class.

```
                   Factory Pattern Architectural Layout
                              ┌──────────────────┐
                              │ NotificationFactory│
                              └────────┬─────────┘
                                       │ create(type)
             ┌─────────────────────────┼─────────────────────────┐
             ▼                         ▼                         ▼
   ┌───────────────────┐     ┌───────────────────┐     ┌───────────────────┐
   │ EmailNotification │     │  SmsNotification  │     │ PushNotification  │
   └───────────────────┘     └───────────────────┘     └───────────────────┘
```

#### Implementation:

```
// 1. Shared Product Interface
public interface Notification {
    void notifyUser(String message);
}

// 2. Concrete Product Implementations
class EmailNotification implements Notification {
    public void notifyUser(String message) { System.out.println("Email: " + message); }
}

class SmsNotification implements Notification {
    public void notifyUser(String message) { System.out.println("SMS: " + message); }
}

// 3. Centralized Factory Engine
public class NotificationFactory {
    public static Notification createNotification(String channel) {
        if (channel == null || channel.isEmpty()) return null;
        return switch (channel.toUpperCase()) {
            case "SMS"   -> new SmsNotification();
            case "EMAIL" -> new EmailNotification();
            default -> throw new IllegalArgumentException("Unknown channel: " + channel);
        };
    }
}
```

---

### 15. The Builder Design Pattern

Constructs complex objects step-by-step. Solves the **Telescoping Constructor Anti-Pattern** (constructors with 6–10 parameters where many are optional).

```
                      Builder Pattern Assembly Pipeline
 User.builder()
      .id(101)
      .name("Aarav")
      .email("aarav@cdac.in")  ──► Internal Builder State ──► .build() ──► Immutable User
      .roles(List.of("ADMIN"))
```

#### Implementation:

```
public class UserAccount {
    // Required parameters
    private final int userId;
    private final String username;

    // Optional parameters
    private final String email;
    private final boolean isTwoFactorEnabled;

    private UserAccount(Builder builder) {
        this.userId = builder.userId;
        this.username = builder.username;
        this.email = builder.email;
        this.isTwoFactorEnabled = builder.isTwoFactorEnabled;
    }

    public static class Builder {
        private final int userId;
        private final String username;
        private String email = "unspecified@domain.com";
        private boolean isTwoFactorEnabled = false;

        public Builder(int userId, String username) {
            this.userId = userId;
            this.username = username;
        }

        public Builder email(String email) {
            this.email = email;
            return this; // Method chaining
        }

        public Builder enableTwoFactor(boolean enable) {
            this.isTwoFactorEnabled = enable;
            return this;
        }

        public UserAccount build() {
            // Validation invariants can be verified here before instantiation
            return new UserAccount(this);
        }
    }
}

// Usage:
UserAccount account = new UserAccount.Builder(101, "aarav_acts")
                                     .email("aarav@cdac.in")
                                     .enableTwoFactor(true)
                                     .build();
```

---

### 16. The Strategy Design Pattern

Defines a family of interchangeable algorithms, encapsulates each one in a separate class, and makes them swappable at runtime based on context.

```
                               Strategy Pattern Layout
                                ┌─────────────────────┐
                                │ ShoppingCartContext │
                                └──────────┬──────────┘
                                           │ uses PaymentStrategy
             ┌─────────────────────────────┼─────────────────────────────┐
             ▼                             ▼                             ▼
   ┌───────────────────┐         ┌───────────────────┐         ┌───────────────────┐
   │ CreditCardStrategy│         │    UpiStrategy    │         │  CryptoStrategy   │
   └───────────────────┘         └───────────────────┘         └───────────────────┘
```

#### Implementation:

```
// 1. Strategy Interface
public interface PaymentStrategy {
    void processPayment(double amount);
}

// 2. Concrete Strategy Algorithms
class UpiPayment implements PaymentStrategy {
    public void processPayment(double amount) { System.out.println("Processing UPI payment: ₹" + amount); }
}

class CreditCardPayment implements PaymentStrategy {
    public void processPayment(double amount) { System.out.println("Charging Credit Card: ₹" + amount); }
}

// 3. Execution Context
public class ShoppingCart {
    private PaymentStrategy strategy;

    public void setPaymentStrategy(PaymentStrategy strategy) {
        this.strategy = strategy; // Swap algorithm dynamically at runtime
    }

    public void checkout(double totalAmount) {
        if (strategy == null) throw new IllegalStateException("Payment strategy not set!");
        strategy.processPayment(totalAmount);
    }
}
```

---

### 17. High-Yield CDAC Exam Traps & Interview Questions

- **Q1: Why is instruction reordering dangerous in a Singleton's Double-Checked Locking without `volatile`?**
  - _Answer:_ Instantiating an object (`instance = new Singleton()`) breaks down into three lower-level steps:
    1. Allocate heap memory.
    2. Run constructor `<init>()` to initialize fields.
    3. Assign memory address to `instance`.
       The compiler or CPU can reorder this sequence to $1 \rightarrow 3 \rightarrow 2$. Without `volatile`, Thread 2 could see `instance != null` right after step 3, before step 2 finishes. Thread 2 then returns an **uninitialized, broken object**. The `volatile` modifier establishes a memory barrier that prevents step 3 from moving ahead of step 2.

- **Q2: Does calling `System.gc()` guarantee that the Garbage Collector will run immediately?**
  - _Answer:_ **No.** `System.gc()` and `Runtime.getRuntime().gc()` are merely **hints** to the JVM. The JVM chooses whether and when to act on this hint based on active load and memory availability.

- **Q3: What happens if two ClassLoaders load the exact same class from the same `.class` file on disk?**
  - _Answer:_ To the JVM, they are **two completely distinct, incompatible types**. Casting an instance of a class loaded by ClassLoader A to the reference type loaded by ClassLoader B throws a runtime **`java.lang.ClassCastException`**, even if their byte-for-byte source and names are identical. Type identity in Java is defined as: `(ClassName, DefiningClassLoader)`.

- **Q4: Why does defensive copying use `new Date(date.getTime())` rather than `date.clone()`?**
  - _Answer:_ `Date` is a non-final class. An external caller could pass a malicious subclass of `java.util.Date` that overrides `clone()` to return a shared reference rather than a new instance, bypassing the defensive copy. Instantiating via `new Date(...)` guarantees creation of the authentic standard library class.

- **Q5: Can an immutable object contain a reference to a mutable object?**
  - _Answer:_ **Yes, provided encapsulation and defensive copying are enforced.** The immutable class must never expose the mutable object directly. It must make defensive copies when receiving the mutable object in its constructor and when returning it from getter methods, or wrap collections in unmodifiable views (`Collections.unmodifiableList()`).
