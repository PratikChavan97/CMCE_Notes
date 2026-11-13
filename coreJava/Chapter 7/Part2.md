# Module 07: Concurrency, Multithreading, and Modern Java

## Unit 7.2: Synchronization and Thread Communication

### 1. Introduction to Synchronization

When multiple threads execute concurrently within the same JVM process, they share the same **Heap** and **Metaspace** memory regions. While concurrent reads are inherently safe, concurrent read-modify-write operations on shared mutable state introduce non-deterministic state corruption.

**Thread Synchronization** is the coordination mechanism that enforces mutually exclusive access to shared mutable resources, ensuring that only one thread can execute a sensitive sequence of instructions at any given instant.

```
Without Synchronization (Data Corruption):
Thread A ───► Read balance ($1000) ───► Deduct $200 ──────────────► Write balance ($800)
                  ▲                                                      │
                  │ [RACE WINDOW: Concurrent Read]                       │ [Overwrites!]
                  │                                                      ▼
Thread B ─────────┴─► Read balance ($1000) ───► Deduct $500 ──────► Write balance ($500)
(Result: Balance becomes $500 instead of $300! $200 is permanently lost.)

With Synchronization (Serialized Access via Lock):
Thread A ───► [ACQUIRE LOCK] ──► Read ($1000) ──► Deduct $200 ──► Write ($800) ──► [RELEASE LOCK]
                                                                                        │
Thread B ───► [WAITING/BLOCKED................................................] ───────► [ACQUIRE LOCK] ──► Read ($800)...

```

### 2. Race Conditions & Critical Sections

#### 2.1 The Critical Section

A **Critical Section** is any segment of code that accesses and mutates shared variables, collections, or I/O resources that could be accessed by more than one concurrent thread.

#### 2.2 The Race Condition

A **Race Condition** is an undesirable situation where the execution outcome of concurrent operations depends unpredictably on the relative order, timing, or scheduling slices allocated to concurrent threads by the operating system kernel.

#### 2.3 The Non-Atomic Read-Modify-Write Trap

Even basic operations like incrementation (`count++`) are **not atomic** at the bytecode machine level. A single `count++` expands into three distinct low-level operations:

```
Source Code:
count++;

Bytecode Representation:
1: getfield  #2 // Field count:I (READ from Heap into Local Operand Stack)
4: iconst_1     // Load constant 1
5: iadd         // MODIFY: Add 1 to value
6: putfield  #2 // Field count:I (WRITE back to Heap memory)

```

If the CPU context-switches a thread between `getfield` and `putfield`, updates written by other threads in that interval are silently wiped out (Lost Update Problem).

### 3. The `synchronized` Keyword: Methods vs. Blocks

Java implements synchronization using the **`synchronized`** keyword, which leverages the JVM's built-in **Intrinsic Lock** (also called a **Monitor Lock**).

```
                             Synchronization Styles
                                       │
         ┌─────────────────────────────┴─────────────────────────────┐
         ▼                                                           ▼
Synchronized Methods                                        Synchronized Blocks
- Locks the entire method body                              - Locks only the specific lines of code
- Coarse-grained concurrency                                - Fine-grained concurrency
- Monitors 'this' or Class object                           - Can use any custom object monitor
- Higher contention / slower throughput                     - Minimizes contention / higher performance

```

#### 3.1 `synchronized` Methods

```
public class BankAccount {
    private double balance = 1000.0;

    // Locks the intrinsic monitor of the calling instance ('this')
    public synchronized void withdraw(double amount) {
        if (balance >= amount) {
            balance -= amount;
        }
    }
}

```

#### 3.2 `synchronized` Blocks

A synchronized block isolates only the critical lines requiring protection:

```
public class InventoryService {
    private int stockCount = 100;
    private final Object stockLock = new Object(); // Dedicated monitor lock

    public void orderItem(int quantity) {
        // Unsynchronized section: validation, parsing, preparation
        validatePayload(quantity);

        // Synchronized section: critical section only
        synchronized (stockLock) {
            if (stockCount >= quantity) {
                stockCount -= quantity;
            }
        }

        // Unsynchronized section: network I/O, email dispatch
        sendConfirmationEmail();
    }
}

```

#### 3.3 Performance Rule: Favor Synchronized Blocks Over Methods

Synchronizing an entire method holds the monitor lock across non-critical operations (such as blocking disk reads, JSON serialization, and remote HTTP requests). A synchronized block limits lock-holding time to the shortest possible window, maximizing concurrency.

### 4. Object-Level Locking vs. Class-Level Locking

The JVM provides two distinct locking scopes based on the underlying object monitor acquired:

```
                       Object-Level vs. Class-Level Locking
                                       │
         ┌─────────────────────────────┴─────────────────────────────┐
         ▼                                                           ▼
Object-Level Locking                                        Class-Level Locking
- Locks an individual Heap instance                         - Locks the java.lang.Class mirror in Metaspace
- Multiple instances have independent locks                 - Exactly ONE lock exists per ClassLoader
- Protects INSTANCE-level state                             - Protects STATIC / Class-level state
- Syntax:                                                   - Syntax:
  • synchronized void method()                                • static synchronized void method()
  • synchronized (this) { ... }                               • synchronized (ClassName.class) { ... }

```

```
HEAP MEMORY
┌──────────────────────────────────────┐     ┌──────────────────────────────────────┐
│ Instance A (0x1000)                  │     │ Instance B (0x2000)                  │
│ Monitor Lock: [Held by Thread 1]     │     │ Monitor Lock: [FREE - Accessible]    │
└──────────────────────────────────────┘     └──────────────────────────────────────┘
(Thread 2 CAN access Instance B simultaneously while Thread 1 works on Instance A!)

METASPACE / HEAP CLASS MIRROR
┌───────────────────────────────────────────────────────────────────────────────────┐
│ java.lang.Class: Account.class (0x9999)                                           │
│ Class Monitor Lock: [Held by Thread 3]                                            │
└───────────────────────────────────────────────────────────────────────────────────┘
(ALL threads across the entire JVM are BLOCKED from static synchronized methods in Account!)

```

#### Code Comparison:

```
public class LockDistinction {
    // 1. Object-Level Lock: Locks instance calling the method
    public synchronized void instanceMethod() {
        // Critical section for instance fields
    }

    // 2. Object-Level Lock via Block:
    public void blockInstanceMethod() {
        synchronized (this) {
            // Identical lock target to instanceMethod()
        }
    }

    // 3. Class-Level Lock: Locks LockDistinction.class
    public static synchronized void staticMethod() {
        // Critical section for static fields across ALL instances
    }

    // 4. Class-Level Lock via Block:
    public void blockClassMethod() {
        synchronized (LockDistinction.class) {
            // Identical lock target to staticMethod()
        }
    }
}

```

> ⚠️ **The Independence Invariant:**
> A thread executing an **instance-level synchronized method** does **NOT** block another thread from concurrently executing a **static synchronized method** on the same class! They acquire two completely different locks (`this` vs. `ClassName.class`).

### 5. JVM Intrinsic Lock Mechanics: `monitorenter` and `monitorexit`

Every object header in the JVM Heap contains an 8-byte **Mark Word** that stores locking metadata (Lock Bits, Biased Locking tags, and pointers to Heavyweight OS Mutexes).

```
Inside the Object Header (Mark Word):
┌────────────────────────────────────────────────┬──────────────┐
│ Lock State Metadata / Pointer to ObjectMonitor │ Lock Bits: 2 │
└────────────────────────────────────────────────┴──────────────┘

```

#### Bytecode Level Implementation of `synchronized`:

When a synchronized block compiles, `javac` injects two specific bytecode opcodes:

```
public void execute() {
    synchronized(this) {
        doTask();
    }
}

```

Disassembly (`javap -c`):

```
 2: aload_0
 3: dup
 4: astore_1
 5: monitorenter      // <── ACQUIRE INTRINSIC MONITOR LOCK
 6: aload_0
 7: invokevirtual #2  // Method doTask:()V
10: aload_1
11: monitorexit       // <── RELEASE INTRINSIC MONITOR LOCK (Normal Path)
12: goto          20
15: astore_2
16: aload_1
17: monitorexit       // <── RELEASE INTRINSIC MONITOR LOCK (Exception Path!)
18: aload_2
19: athrow

```

#### 5.1 Reentrant Synchronization (Reentrancy)

Java intrinsic monitors are **reentrant**. If a thread currently holding an object's monitor lock encounters another synchronized block or method requiring the _same_ lock, it is granted access immediately without deadlock.

- Under the hood, the JVM increments an internal **Recursion Counter** on the `ObjectMonitor`.

- Each `monitorenter` increments the counter; each `monitorexit` decrements it.

- The lock is released to other threads only when the recursion counter drops back to **zero**.

### 6. Inter-Thread Communication: `wait()`, `notify()`, and `notifyAll()`

Polite, coordinated thread collaboration cannot rely on busy-waiting (spinning inside `while(!ready)` loops), which consumes 100% CPU time. Java provides **Condition Synchronization** via three fundamental methods on `java.lang.Object`:

```
                       Inter-Thread Signaling Pipeline
                                      │
         ┌────────────────────────────┼────────────────────────────┐
         ▼                            ▼                            ▼
   lock.wait()                 lock.notify()                lock.notifyAll()
- Releases monitor lock      - Wakes ONE arbitrary        - Wakes ALL threads
- Thread enters Wait Set       thread from Wait Set         from the Wait Set
- Suspends execution         - Woken thread re-contends   - Threads re-contend
  until signaled               for the monitor lock         for the monitor lock

```

#### 6.1 The Monitor Ownership Invariant

> ⚠️ **The `IllegalMonitorStateException` Rule:**
> `wait()`, `notify()`, and `notifyAll()` can **ONLY be invoked from within a synchronized context** (method or block) owning that exact object's monitor! Invoking them on an unacquired object throws **`java.lang.IllegalMonitorStateException`** at runtime.

#### 6.2 `notify()` vs. `notifyAll()`

- **`notify()`:** Picks a single arbitrary thread from the Wait Set and moves it to the Entry Set (Blocked state) to compete for the monitor lock. If multiple threads are waiting on different conditions, `notify()` might wake the wrong thread, causing **Deadlock / Signal Loss**.

- **`notifyAll()`:** Wakes every thread currently waiting in the Wait Set. While all threads re-contend for the lock, each can inspect its respective state predicate. This is the **standard recommended practice** in production code.

### 7. The Classical Producer-Consumer Problem

The Producer-Consumer pattern represents a standard coordination problem where one or more producer threads generate data into a bounded buffer, while consumer threads extract data from it.

```
Producer Threads                                                Consumer Threads
┌──────────────┐         Bounded Shared Buffer (Capacity = 2)   ┌──────────────┐
│  Producer 1  │ ──► [ PUT ] ──► [ [ Item 1 ] [ Item 2 ] ] ──► [ GET ] ──► │  Consumer 1  │
└──────────────┘                    ▲                ▲          └──────────────┘
                                    │ Buffer Full:   │ Buffer Empty:
                                    │ Producer waits │ Consumer waits

```

#### 7.1 Production-Grade Implementation:

```
import java.util.LinkedList;
import java.util.Queue;

public class BoundedBuffer<T> {
    private final Queue<T> queue = new LinkedList<>();
    private final int capacity;

    public BoundedBuffer(int capacity) {
        this.capacity = capacity;
    }

    // Producer Method
    public synchronized void produce(T item) throws InterruptedException {
        // ALWAYS check guard conditions in a WHILE loop, NEVER an IF block!
        while (queue.size() == capacity) {
            System.out.println(Thread.currentThread().getName() + ": Buffer full. Waiting...");
            wait(); // Releases lock; suspends producer
        }

        queue.add(item);
        System.out.println(Thread.currentThread().getName() + " produced: " + item);

        // Notify waiting consumers that data is available
        notifyAll();
    }

    // Consumer Method
    public synchronized T consume() throws InterruptedException {
        // Guard check against spurious wakeups
        while (queue.isEmpty()) {
            System.out.println(Thread.currentThread().getName() + ": Buffer empty. Waiting...");
            wait(); // Releases lock; suspends consumer
        }

        T item = queue.poll();
        System.out.println(Thread.currentThread().getName() + " consumed: " + item);

        // Notify waiting producers that a slot has freed up
        notifyAll();
        return item;
    }
}

```

#### 7.2 Why Must `wait()` ALWAYS Be Inside a `while` Loop?

A thread can wake up without any `notify()` call being sent—a hardware/OS-level phenomenon known as a **Spurious Wakeup**. Additionally, if multiple threads are awakened by `notifyAll()`, another thread may claim the condition before this thread acquires the monitor. Checking the condition in a `while` loop guarantees the guard condition is verified before code proceeds:

```
// INCORRECT (Prone to Spurious Wakeup bugs and state corruption!):
if (queue.isEmpty()) {
    wait();
}

// CORRECT (Mandatory industry standard pattern):
while (queue.isEmpty()) {
    wait();
}

```

### 8. Deadlock, Livelock, and Starvation

```
                          Concurrency Hazard Taxonomy
                                       │
         ┌─────────────────────────────┼─────────────────────────────┐
         ▼                             ▼                             ▼
     Deadlock                      Livelock                      Starvation
Permanent mutual blockage;     Active state changes but      A thread is perpetually
threads wait forever for       zero forward progress;        denied CPU or lock access
locks held by each other       active spinning               due to unfair schedulers

```

#### 8.1 Deadlock

A **Deadlock** occurs when two or more threads are permanently blocked, each holding a lock that the other needs, creating an unbreakable circular dependency.

```
                    The Classical Deadlock Circle
┌──────────────┐         Holds Lock A          ┌──────────────┐
│   Thread 1   │ ────────────────────────────► │  Resource A  │
└──────────────┘                               └──────────────┘
       ▲                                               │
       │ Wants Lock B                                  │ Wants Lock A
       │                                               ▼
┌──────────────┐         Holds Lock B          ┌──────────────┐
│  Resource B  │ ◄──────────────────────────── │   Thread 2   │
└──────────────┘                               └──────────────┘

```

#### Deadlock Code Demonstration:

```
public class DeadlockSimulation {
    private static final Object LockA = new Object();
    private static final Object LockB = new Object();

    public static void main(String[] args) {
        // Thread 1 locks A, then attempts to lock B
        Thread t1 = new Thread(() -> {
            synchronized (LockA) {
                System.out.println("Thread 1: Acquired LockA");
                try { Thread.sleep(50); } catch (InterruptedException e) {}

                synchronized (LockB) {
                    System.out.println("Thread 1: Acquired LockB");
                }
            }
        });

        // Thread 2 locks B, then attempts to lock A (CIRCULAR WAIT!)
        Thread t2 = new Thread(() -> {
            synchronized (LockB) {
                System.out.println("Thread 2: Acquired LockB");
                try { Thread.sleep(50); } catch (InterruptedException e) {}

                synchronized (LockA) {
                    System.out.println("Thread 2: Acquired LockA");
                }
            }
        });

        t1.start();
        t2.start();
    }
}

```

#### 8.2 The Four Coffman Conditions for Deadlock

Deadlock can occur if and only if **all four** of the following conditions hold simultaneously:

1. **Mutual Exclusion:** At least one resource must be held in a non-shareable mode (only one thread can use it at a time).

2. **Hold and Wait:** A thread holds at least one resource while waiting to acquire additional resources held by other threads.

3. **No Preemption:** Resources cannot be forcibly confiscated from a thread; they can only be released voluntarily.

4. **Circular Wait:** A closed chain of threads exists such that each thread holds a resource needed by the next thread in the chain.

#### 8.3 Deadlock Prevention Strategies

To prevent deadlocks, software systems break at least one of Coffman's conditions:

1. **Lock Ordering (Breaks Circular Wait - #1 Enterprise Solution):** Always acquire locks in a globally defined, consistent order. If both Thread 1 and Thread 2 acquire `LockA` _before_ acquiring `LockB`, circular wait cannot form.

2. **Timed Lock Acquisition via Explicit Locks (Breaks No Preemption):** Use modern `java.util.concurrent.locks.ReentrantLock` with `.tryLock(timeout, unit)`. If a thread cannot acquire the second lock within the timeout window, it backs off, releases all held locks, and retries later.

3. **Deadlock Detection via Thread Dumps:** Using the JDK utility `jstack <pid>` or `jcmd <pid> Thread.dump_to_file`. The JVM actively analyzes its internal lock-dependency graphs and outputs:
   `Found 1 deadlock.` listing the exact threads, stack frames, and monitor addresses involved.

#### 8.4 Starvation

**Starvation** is a condition where a thread is ready to run, but is perpetually denied CPU execution time or monitor lock access because higher-priority threads consistently cut in front of it.

- **Resolution:** Use **Fair Locking** (`new ReentrantLock(true)`), which serves threads in strict FIFO order rather than allowing threads to jump the queue.

### 9. Thread-Local Storage: `java.lang.ThreadLocal<T>`

`ThreadLocal<T>` provides **thread-confined variables**. Instead of sharing a single variable across all threads, each thread that accesses a `ThreadLocal` receives its own completely independent, isolated copy of the variable.

```
                           ThreadLocal Architecture
       Thread-1 Object (Heap)                         Thread-2 Object (Heap)
┌───────────────────────────────────┐          ┌───────────────────────────────────┐
│ ThreadLocalMap                    │          │ ThreadLocalMap                    │
│ Key: [ThreadLocal ref]            │          │ Key: [ThreadLocal ref]            │
│ Value: Connection-Obj-A (0x1111)  │          │ Value: Connection-Obj-B (0x2222)  │
└───────────────────────────────────┘          └───────────────────────────────────┘

```

#### 9.1 Usage Patterns:

```
public class SecurityContextHolder {
    // ThreadLocal container initializing defaults via withInitial
    private static final ThreadLocal<String> userSessionToken =
        ThreadLocal.withInitial(() -> "ANONYMOUS");

    public static void setToken(String token) {
        userSessionToken.set(token);
    }

    public static String getToken() {
        return userSessionToken.get();
    }

    public static void clear() {
        userSessionToken.remove(); // CRITICAL: Prevent Memory Leaks!
    }
}

```

#### 9.2 The ThreadLocal Memory Leak Hazard in Thread Pools

In web applications running on thread pools (e.g., Spring Boot with embedded Tomcat), worker threads **never terminate**; they return to the pool to handle subsequent incoming HTTP requests.

- A `ThreadLocalMap` uses **WeakReferences** for its keys (the `ThreadLocal` object) but **StrongReferences** for its values.

- If a thread finishes processing a request and you fail to invoke **`ThreadLocal.remove()`**, the value object remains pinned in heap memory by the worker thread's internal map, leading to severe **PermGen/Metaspace/Heap memory leaks**.

- **Invariant:** Always clear thread-locals inside a `finally` block:

  ```
  try {
      SecurityContextHolder.setToken(incomingToken);
      processRequest();
  } finally {
      SecurityContextHolder.clear(); // Always clean up!
  }

  ```

### 10. High-Yield CDAC Exam Traps & Interview Questions

- **Q1: Can a thread call a non-synchronized method while another thread is executing a `synchronized` method on the same object?**
  - _Answer:_ **Yes.** The intrinsic monitor lock protects only the `synchronized` methods/blocks of that object. Non-synchronized methods do not attempt to acquire the monitor lock and can execute concurrently without restriction.

- **Q2: What happens if an unhandled runtime exception occurs inside a `synchronized` block? Does the lock remain stuck forever?**
  - _Answer:_ **No, the lock is automatically released.** The bytecode generated by `javac` wraps the synchronized block in an implicit `any` exception handler table that invokes `monitorexit` before propagating the exception up the stack.

- **Q3: What is the difference between `Thread.sleep()` and `Object.wait()` regarding lock handling?**
  - _Answer:_
    - `Thread.sleep()` **holds all acquired monitor locks** while paused.

    - `Object.wait()` **releases the monitor lock** of the target object, allowing waiting threads to contend for it while it sleeps in the Wait Set.

- **Q4: Why can't we use a primitive variable or a `String` literal as a synchronization lock?**
  - _Answer:_
    - Primitives (`int`, `boolean`) have no object headers and therefore lack an intrinsic monitor.

    - `String` literals reside in the shared **String Constant Pool**. Synchronizing on `"MY_LOCK"` means any unrelated library or component in the same JVM that synchronizes on an identical string literal will share the same lock, causing system-wide thread blocking and deadlocks. Always synchronize on a dedicated `new Object()`.

- **Q5: Can two different threads concurrently execute two different `static synchronized` methods of the same class?**
  - _Answer:_ **No.** Both methods synchronize on the same class-level monitor: `ClassName.class`. Because there is only one `Class` object per class per ClassLoader, the second thread is blocked until the first thread releases the class lock.
