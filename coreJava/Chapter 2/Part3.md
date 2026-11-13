# Module 02: Object-Oriented Programming (OOP) in Java

## Unit 2.3: Static Members and Reference Variables

### 1. Static Variables (Class Variables)

A **static variable** (or class variable) is a field declared with the `static` keyword inside a class body, outside any method, constructor, or block.

```
Metaspace (Class Metadata)
┌──────────────────────────────────────────────┐
│ Class: Student                               │
│ - Bytecode definition                        │
│ - Method tables                              │
│ - static String universityName = "CDAC" <────┼──┐ (Single shared memory slot)
└──────────────────────────────────────────────┘  │
                                                  │ Shared by all instances
Heap Memory                                       │
┌───────────────────────────┐                     │
│ Instance 1: Student 0x100 │ ────────────────────┤
│ - int rollNo = 101        │                     │
└───────────────────────────┘                     │
┌───────────────────────────┐                     │
│ Instance 2: Student 0x200 │ ────────────────────┘
│ - int rollNo = 102        │
└───────────────────────────┘
```

#### 1.1 Key Invariants of Static Variables

1. **Per-Class Allocation:** Only **one single copy** of a static variable exists, regardless of whether 0, 1, or 10,000 instances of the class are instantiated.
2. **Storage Location:**
   - **Java 7 and earlier:** Stored inside the _Permanent Generation (PermGen)_ on the managed JVM Heap.
   - **Java 8 and later:** Class metadata resides in **Metaspace** (native process memory), while the mirror `java.lang.Class` object that physically holds static fields resides in the **Heap**.
3. **Lifecycle:** Created when the class is loaded and prepared by the ClassLoader; destroyed only when the ClassLoader unloads the class (typically on JVM shutdown).
4. **Default Initialization:** Automatically initialized to default values (`0`, `0.0`, `false`, `null`) during the ClassLoader _Preparation_ phase before any constructor or static block executes.

---

### 2. Static Methods (Class Methods)

A **static method** is a method bound to the class itself rather than to any individual object instance.

```java
public class MathEngine {
    // Static utility method
    public static int square(int x) {
        return x * x;
    }
}
```

#### 2.1 Critical Restrictions on Static Methods

1. **No `this` or `super`:** Static methods have no implicit instance context. Attempting to use `this` or `super` produces a compile-time error: _non-static variable this cannot be referenced from a static context_.
2. **Cannot Directly Access Instance Members:** A static method cannot read/write instance fields or invoke instance methods without creating/receiving an explicit object instance first:

   ```java
   public class ContextDemo {
       int instanceVal = 10;
       static int staticVal = 20;

       public static void display() {
           // System.out.println(instanceVal); // COMPILE ERROR!
           System.out.println(staticVal);       // Legal: static to static

           ContextDemo obj = new ContextDemo();
           System.out.println(obj.instanceVal); // Legal: accessed via explicit instance
       }
   }
   ```

3. **No Dynamic Method Overriding (Method Hiding):** Static methods cannot be overridden polymorphically. If a subclass declares a static method with the same signature, it **hides** the parent's method rather than overriding it (resolved via static binding at compile time).
4. **Bytecode Invocation:** Compiled to the **`invokestatic`** bytecode opcode, which performs static resolution without virtual table (vtable) dispatch.

---

### 3. Static Blocks (Static Initializer Blocks)

A **static block** is an initializer enclosed in `static { ... }` that executes when the class is initialized by the JVM.

```java
public class DriverLoader {
    static String dbUrl;

    // Static Initializer Block (<clinit>)
    static {
        dbUrl = "jdbc:postgresql://localhost:5432/production";
        System.out.println("DriverLoader: Static Initializer Executed");
    }
}
```

#### 3.1 Mechanics & JVM Invariants

- **Executed Once:** Runs exactly once when the class is loaded into Metaspace, prior to:
  - Any instance creation (`new DriverLoader()`).
  - Any static method invocation (`DriverLoader.run()`).
  - Any static field read/write (except compile-time constants inlined by `javac`).
- **Execution Order:** If multiple static blocks are declared, the JVM aggregates them into the **`<clinit>`** bytecode method and executes them in **top-to-bottom textual order**.
- **Exception Handling:** Cannot throw checked exceptions directly. If an unhandled exception escapes a static block, the JVM wraps it and throws **`java.lang.ExceptionInInitializerError`**.

---

### 4. Accessing Static Members

Static members can be accessed in two ways:

```java
// Method 1: Via Class Identifier (BEST PRACTICE)
double result = Math.sqrt(25.0);

// Method 2: Via Object Reference Handle (DISCOURAGED / CODE SMELL)
Student s = new Student();
System.out.println(s.universityName); // Compiles, but triggers compiler warning!
```

#### 4.1 The Null Reference Trap with Static Members (High-Yield MCQ)

Because static method calls are resolved at **compile-time** based on the declared reference type, calling a static method on a reference that holds `null` **does NOT throw a `NullPointerException`**!

```java
public class StaticNullTrap {
    public static void printMessage() {
        System.out.println("Static method executed cleanly!");
    }

    public static void main(String[] args) {
        StaticNullTrap trap = null;
        trap.printMessage(); // Prints cleanly! NO NullPointerException!
        // The compiler replaces 'trap.printMessage()' directly with:
        // StaticNullTrap.printMessage(); (invokestatic)
    }
}
```

---

### 5. Architectural Comparison: Instance Variables vs. Static Variables

| Dimension             | Instance Variable (Field)                                       | Static Variable (Class Field)                               |
| :-------------------- | :-------------------------------------------------------------- | :---------------------------------------------------------- |
| **Physical Location** | **Heap** (embedded inside each object layout)                   | **Heap** (attached to the `Class` object mirror in Java 8+) |
| **Copies in Memory**  | 1 copy **per instantiated object** ($N$ instances = $N$ copies) | **Exactly 1 copy** shared across the entire JVM ClassLoader |
| **Created When**      | Object is instantiated via `new`                                | Class is loaded and prepared by ClassLoader                 |
| **Destroyed When**    | Object is collected by Garbage Collector                        | Class is unloaded from Metaspace                            |
| **Access Mechanics**  | Via object reference: `obj.field`                               | Via class name: `ClassName.field` (or `obj.field`)          |
| **Default Values**    | Assigned during object allocation on Heap                       | Assigned during class preparation                           |
| **Binding Time**      | Resolved dynamically at runtime (Dynamic Dispatch)              | Resolved statically at compile-time (Static Binding)        |

---

### 6. Primitive vs. Reference Data Types: Memory Anatomy

```
Primitive Type in Memory:
┌─────────────────────────┐
│ int age = 25            │ ───► Stores raw binary value directly in the slot: [ 00011001 ]
└─────────────────────────┘

Reference Type in Memory:
Stack Frame                                   Heap Memory
┌─────────────────────────┐                   ┌──────────────────────────────────────┐
│ Point p = 0x7FFF40      │ ────────────────► │ Point Object (0x7FFF40)              │
└─────────────────────────┘                   ├──────────────────────────────────────┤
                                              │ Header: Mark Word + Klass Word       │
                                              │ int x = 10;                          │
                                              │ int y = 20;                          │
                                              └──────────────────────────────────────┘
```

| Dimension                       | Primitive Data Types (`int`, `double`, etc.)                        | Reference Data Types (`Object`, `int[]`, etc.)                                    |
| :------------------------------ | :------------------------------------------------------------------ | :-------------------------------------------------------------------------------- |
| **Storage Value**               | The **literal data bits** are stored directly in the variable slot. | Stores a **memory address handle** (reference pointer) targeting a heap location. |
| **Physical Location**           | Stack Frame (if local) or inside Heap object (if field).            | Pointer on Stack/Heap; actual Object payload **always on the Heap**.              |
| **Default Value**               | `0`, `0.0`, `false`, `'\u0000'`                                     | Always **`null`** (points to no memory address).                                  |
| **Memory Footprint**            | Fixed per type: 1 byte (`byte`), 4 bytes (`int`), 8 bytes (`long`). | 4 bytes (with 32-bit compressed OOPs) or 8 bytes (uncompressed 64-bit pointers).  |
| **Dereferencing (`.` or `[]`)** | Not supported (compile-time error).                                 | Supported; throws **`NullPointerException`** if variable holds `null`.            |
| **Equality Check (`==`)**       | Compares **bit value equality**.                                    | Compares **memory address identity** (do they target the same heap block?).       |

---

### 7. Reference Variables & Object References

- **Reference Variable:** A variable declared with a class, interface, or array type that acts as a container for an object's memory address.
- **Object Reference:** The underlying 32-bit or 64-bit address handle (pointer value) that designates the location of the object on the Heap.
- **Object:** The physical block of contiguous memory on the Heap that holds the object header and instance fields.

```java
Point p1;                 // Reference variable declared on Stack; currently unassigned/null.
p1 = new Point(10, 20);   // 'new' allocates on Heap; address is assigned into p1.
Point p2 = p1;            // The address in p1 is COPIED into p2. Both reference the same heap object.
```

---

### 8. Passing Objects to Methods: The "Pass-by-Value" Proof

> ⚠️ **THE #1 MOST TESTED JAVA INTERVIEW TOPIC:**
>
> _Is Java Pass-by-Value or Pass-by-Reference?_
>
> **Java is 100% strictly PASS-BY-VALUE for EVERYTHING.** There is zero pass-by-reference in Java.

When an object is passed as an argument to a method, Java **does NOT pass the object itself**, nor does it pass an alias/reference to the caller's variable. It passes a **copy of the reference address** (a copy of the bits representing the pointer).

#### 8.1 Scenario A: Mutating State via the Copied Reference

```java
class Container {
    int value = 50;
}

public class ParameterTrace {
    public static void modifyState(Container c) {
        c.value = 999; // Mutates field inside the heap object
    }

    public static void main(String[] args) {
        Container original = new Container();
        modifyState(original);
        System.out.println(original.value); // Prints: 999
    }
}
```

- **Why it changes:** `original` holds address `0x4000`. The address `0x4000` is **copied** by value into the parameter `c` of `modifyState`. Both references now point to the exact same heap memory block. Mutating `c.value` modifies the object at `0x4000`.

---

### 9. Reassigning References Inside Methods

The definitive proof that Java is pass-by-value is observed when a method **reassigns** its parameter reference:

```java
public class ReferenceReassignmentProof {
    public static void reassign(Container c) {
        // Reassignment creates a BRAND NEW object on the Heap
        c = new Container();
        c.value = 888;
    }

    public static void main(String[] args) {
        Container original = new Container();
        original.value = 50;

        reassign(original);

        System.out.println(original.value); // Prints: 50! (NOT 888!)
    }
}
```

#### Step-by-Step Memory Trace of Reassignment:

```
Step 1: Inside main() before call
STACK (main)                                     HEAP MEMORY
┌───────────────────────────────┐               ┌───────────────────────────────┐
│ original: 0x1000 ─────────────┼─────────────► │ Container Object @ 0x1000     │
└───────────────────────────────┘               │ - int value = 50              │
                                                └───────────────────────────────┘

Step 2: Method reassign(original) invoked -> Value 0x1000 is COPIED into slot 'c'
STACK (reassign)
┌───────────────────────────────┐
│ c: 0x1000 ────────────────────┼─────────────┐
└───────────────────────────────┘             │
STACK (main)                                  ▼
┌───────────────────────────────┐               ┌───────────────────────────────┐
│ original: 0x1000 ─────────────┼─────────────► │ Container Object @ 0x1000     │
└───────────────────────────────┘               │ - int value = 50              │
                                                └───────────────────────────────┘

Step 3: Inside reassign() -> 'c = new Container()' executes
STACK (reassign)
┌───────────────────────────────┐               ┌───────────────────────────────┐
│ c: 0x9000 (OVERWRITTEN!) ─────┼─────────────► │ Container Object @ 0x9000     │
└───────────────────────────────┘               │ - int value = 888             │
STACK (main)                                    └───────────────────────────────┘
┌───────────────────────────────┐               ┌───────────────────────────────┐
│ original: 0x1000 (UNTOUCHED!) ┼─────────────► │ Container Object @ 0x1000     │
└───────────────────────────────┘               │ - int value = 50              │
                                                └───────────────────────────────┘

Step 4: reassign() finishes and its stack frame is popped.
0x9000 is orphaned and collected by GC.
'original' in main() still points to 0x1000 with value = 50.
```

---

### 10. The Canonical Swap Failure Proof

```java
public class SwapTest {
    public static void swap(Integer a, Integer b) {
        Integer temp = a;
        a = b;
        b = temp;
    }

    public static void main(String[] args) {
        Integer num1 = 10;
        Integer num2 = 20;

        swap(num1, num2);

        System.out.println("num1: " + num1 + ", num2: " + num2);
        // OUTPUT: num1: 10, num2: 20 (SWAP FAILED!)
    }
}
```

- **Explanation:** `swap` swapped only the local pointer copies residing in its own stack frame slots (`a` and `b`). When the method exited and its stack frame was popped, those swapped local variables vanished. The caller's variables `num1` and `num2` remained completely untouched.

---

### 11. JVM Memory Layout: Runtime Data Areas Deep Dive

The JVM divides memory into five distinct runtime areas:

```
┌────────────────────────────────────────────────────────────────────────┐
│ JVM Process Memory                                                     │
├───────────────────────────────────┬────────────────────────────────────┤
│ Thread-Isolated (Per Thread)      │ Shared Across All Threads          │
├───────────────────────────────────┼────────────────────────────────────┤
│ 1. JVM Call Stack (Frames)        │ 4. Heap Memory                     │
│    - Local Variable Array (LVA)   │    - Young Generation (Eden, S0,S1)│
│    - Operand Stack (OS)           │    - Old (Tenured) Generation      │
│    - Frame Data (Constant Pool)   │    - Holds ALL Objects & Arrays    │
│ 2. Program Counter (PC) Registers ├────────────────────────────────────┤
│ 3. Native Method Stack            │ 5. Metaspace (Native OS Memory)    │
│                                   │    - Class Metadata, Method Tables │
│                                   │    - Runtime Constant Pool         │
└───────────────────────────────────┴────────────────────────────────────┘
```

#### 11.1 JVM Stack vs. Heap Memory

| Characteristic       | JVM Stack Memory                                                    | JVM Heap Memory                                             |
| :------------------- | :------------------------------------------------------------------ | :---------------------------------------------------------- |
| **Scope / Access**   | Thread-private (isolated per thread; zero synchronization overhead) | Global / Shared across all threads (requires thread safety) |
| **Stores**           | Stack frames, local primitive variables, reference handles          | Object instances, arrays, instance fields                   |
| **Allocation Model** | Strict LIFO order; push on method call, pop on return               | Dynamic allocation via `new`; non-contiguous                |
| **Size Constraints** | Governed by `-Xss` (e.g., `-Xss1m`)                                 | Governed by `-Xms` (min) and `-Xmx` (max)                   |
| **Failure State**    | **`java.lang.StackOverflowError`** (deep recursion)                 | **`java.lang.OutOfMemoryError: Java heap space`**           |
| **Management**       | Cleaned up automatically on frame pop (zero GC overhead)            | Cleaned up asynchronously by the Garbage Collector          |

---

### 12. Heap Memory Organization (Generational Architecture)

HotSpot JVM divides Heap space into distinct generations based on the **Weak Generational Hypothesis**: _most allocated objects die shortly after creation_.

```
┌──────────────────────────────────────────────────────────────────────────────┐
│ Entire JVM Heap Memory (-Xmx)                                                │
├───────────────────────────────────────────────┬──────────────────────────────┤
│ Young Generation (-Xmn)                       │ Old (Tenured) Generation     │
├───────────────────────┬───────────────┬───────┼──────────────────────────────┤
│ Eden Space            │ Survivor S0   │ S1    │ Long-lived surviving objects │
│ (Brand-new allocations│ (FromSpace)   │ (To)  │ (Tenured after surviving     │
│  occur here)          │               │       │  threshold cycles; default 15│
├───────────────────────┴───────────────┴───────┼──────────────────────────────┤
│ ◄─────── Minor GC (Fast / Frequent) ────────► │ ◄── Major / Full GC (Slow) ─►│
└───────────────────────────────────────────────┴──────────────────────────────┘
```

1. **Young Generation:**
   - **Eden Space:** All newly created objects are allocated here initially via `new`.
   - **Survivor Spaces (S0 / S1 or FromSpace / ToSpace):** Objects that survive a **Minor GC** in Eden are copied into a survivor space. After surviving a configurable number of collections (the **Tenuring Threshold**, default 15), they are promoted.
2. **Old (Tenured) Generation:** Holds long-lived objects (e.g., database connection pools, Spring application singletons, long-running caches). Reclaimed via **Major GC / Full GC**.

---

### 13. Object Creation & Memory Allocation Internals

When the JVM executes `new Account("ACC-101", 5000.0)`, it performs a low-level memory sequence:

```
                    Step-by-Step JVM Object Allocation
                                    │
   ┌────────────────────────────────┴────────────────────────────────┐
   │ 1. Class Loading Verification                                   │
   │    Verify Account.class is loaded in Metaspace; load if absent. │
   └────────────────────────────────┬────────────────────────────────┘
                                    │
   ┌────────────────────────────────┴────────────────────────────────┐
   │ 2. Memory Size Computation                                      │
   │    Object Header (12 or 16 bytes) + Fields + 8-byte Alignment.  │
   └────────────────────────────────┬────────────────────────────────┘
                                    │
   ┌────────────────────────────────┴────────────────────────────────┐
   │ 3. Memory Allocation Mechanism                                  │
   │    - Bump-the-pointer (if memory is contiguous via TLAB)        │
   │    - Free-list (if memory is fragmented)                        │
   └────────────────────────────────┬────────────────────────────────┘
                                    │
   ┌────────────────────────────────┴────────────────────────────────┐
   │ 4. Zero Initialization                                          │
   │    Zero-fills all instance fields (prevents garbage reads).     │
   └────────────────────────────────┬────────────────────────────────┘
                                    │
   ┌────────────────────────────────┴────────────────────────────────┐
   │ 5. Object Header Construction                                   │
   │    Writes Mark Word (GC, Lock bits) + Klass Word (Metaspace ptr)│
   └────────────────────────────────┬────────────────────────────────┘
                                    │
   ┌────────────────────────────────┴────────────────────────────────┐
   │ 6. Constructor Execution (<init>)                               │
   │    Runs super(), IIBs, and Account constructor body.             │
   └────────────────────────────────┬────────────────────────────────┘
                                    │
   ┌────────────────────────────────┴────────────────────────────────┐
   │ 7. Reference Return                                             │
   │    Pushes memory address (e.g., 0x8800) to caller operand stack.│
   └─────────────────────────────────────────────────────────────────┘
```

#### 13.1 TLAB (Thread-Local Allocation Buffer)

In multi-threaded environments, having multiple threads allocate memory on the shared Heap simultaneously causes lock contention on the heap pointer. The JVM solves this using **TLAB (Thread-Local Allocation Buffer)**:

- Each thread receives a dedicated, private chunk of memory inside Eden space.
- Objects allocated by that thread use **Bump-the-Pointer** inside its own TLAB without requiring thread synchronization locks ($\mathcal{O}(1)$ allocation speed).

---

### 14. High-Yield CDAC Exam Traps & Interview Questions

- **Q1: Can we override a static method in Java? What happens if a subclass writes an identical static method?**
  - _Answer:_ **No, static methods cannot be overridden.** They are bound at compile time (**Static Binding** via `invokestatic`). If a subclass declares an identical static method, it is called **Method Hiding**, not method overriding. The method invoked depends strictly on the **declared compile-time reference type**, not the runtime object instance on the heap.

- **Q2: What is the output of the following code?**

  ```java
  class Test {
      static int count = 0;
      Test() { count++; }
      public static void main(String[] args) {
          Test t1 = new Test();
          Test t2 = new Test();
          Test t3 = new Test();
          System.out.println(t1.count + " " + t2.count + " " + t3.count);
      }
  }
  ```

  - _Answer:_ `3 3 3`
  - _Explanation:_ `count` is a static variable shared across all instances. Every constructor call increments the single shared copy. Accessing `t1.count`, `t2.count`, or `t3.count` reads the same shared memory location, which now holds `3`.

- **Q3: What happens if an exception is thrown inside a static initialization block?**
  - _Answer:_ If a runtime (unchecked) exception is thrown inside a static block, the JVM catches it, terminates class loading, and throws a **`java.lang.ExceptionInInitializerError`**. Any subsequent attempt to use that class during the application's lifetime will fail immediately with a **`java.lang.NoClassDefFoundError`**.

- **Q4: Why does `void modify(int[] arr) { arr[0] = 99; }` change the caller's array, but `void modify(int[] arr) { arr = new int[5]; }` does not?**
  - _Answer:_ Java passes references by value. In both cases, the method receives a copied pointer pointing to the caller's array on the Heap.
    - `arr[0] = 99` dereferences the copied pointer and mutates the original heap array.
    - `arr = new int[5]` overwrites only the method's local pointer variable with the address of a new array, leaving the caller's original pointer and array completely untouched.

- **Q5: Can an abstract method be declared `static`?**
  - _Answer:_ **No.** `abstract` means the method has no implementation and relies on dynamic dispatch for a subclass to provide the implementation at runtime. `static` means the method belongs to the class and must be resolved at compile time without dynamic dispatch. The two modifiers are fundamentally contradictory and cause a **compile-time error**.
