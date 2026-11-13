# Module 07: Concurrency, Multithreading, and Modern Java

## Unit 7.1: Multithreading Fundamentals

### 1. Introduction to Processes and Threads

Modern operating systems provide multitasking capabilities to maximize Central Processing Unit (CPU) utilization and system responsiveness. Concurrency is modeled around two distinct execution constructs: **Processes** and **Threads**.

```
                  Operating System Execution Contexts
                                   │
         ┌─────────────────────────┴─────────────────────────┐
         ▼                                                   ▼
Process-Based Multitasking                          Thread-Based Multitasking
(Heavyweight Isolation)                             (Lightweight Concurrency)
- Executes multiple applications concurrently       - Executes multiple execution paths
  (e.g., JVM instance, browser, database engine)      within the SAME application process
- Isolated virtual memory address space             - Shared memory space (Heap + Metaspace)
- Inter-Process Communication (IPC) is slow         - Inter-Thread Communication is fast (direct RAM)
- High Context-Switching overhead                   - Low Context-Switching overhead
```

#### 1.1 What is a Process?

A **Process** is a self-contained, isolated operating system execution environment containing its own dedicated address space, memory pages, open file descriptors, security tokens, and environment variables. When you launch a Java application via `java Application`, the OS provisions a distinct OS process hosting the Java Virtual Machine.

#### 1.2 What is a Thread?

A **Thread** is the smallest unit of dispatchable execution that can be scheduled by an operating system CPU scheduler. A single process can spawn multiple concurrent threads. In Java, threads are often referred to as **lightweight processes**.

---

### 2. Architectural Comparison: Process vs. Thread

```
PROCESS MEMORY BOUNDARY (JVM Instance)
┌─────────────────────────────────────────────────────────────────────────────┐
│ Shared Process Resources:                                                   │
│ • Heap Memory (Object instances, static fields)                             │
│ • Metaspace / Method Area (Bytecode definitions, runtime constant pool)     │
│ • Operating System Handles (File descriptors, socket channels)              │
│                                                                             │
│   Thread 1                    Thread 2                    Thread 3          │
│  ┌──────────────────────┐    ┌──────────────────────┐    ┌────────────────┐ │
│  │ PC Register          │    │ PC Register          │    │ PC Register    │ │
│  ├──────────────────────┤    ├──────────────────────┤    ├────────────────┤ │
│  │ Local Variable Array │    │ Local Variable Array │    │ LVA            │ │
│  ├──────────────────────┤    ├──────────────────────┤    ├────────────────┤ │
│  │ Operand Stack        │    │ Operand Stack        │    │ Operand Stack  │ │
│  └──────────────────────┘    └──────────────────────┘    └────────────────┘ │
│     [ Private Stack ]           [ Private Stack ]          [ Private Stack ]│
└─────────────────────────────────────────────────────────────────────────────┘
```

| Dimension             | Process                                                                                                                  | Thread                                                                                                 |
| :-------------------- | :----------------------------------------------------------------------------------------------------------------------- | :----------------------------------------------------------------------------------------------------- |
| **Memory Isolation**  | **Completely Isolated**. Cannot read/write memory of other processes without OS IPC.                                     | **Shared Address Space**. All threads within a process share the same Heap and Metaspace.              |
| **Private Data**      | Entire address space is private.                                                                                         | Each thread maintains its own private **Program Counter (PC) Register** and **JVM Stack (Frames)**.    |
| **Creation Cost**     | **Heavyweight**. Requires OS memory page allocations, process control blocks (PCB).                                      | **Lightweight**. Requires allocating a thread stack frame (default `-Xss1m`).                          |
| **Context Switching** | **High Overhead**. Involves invalidating CPU TLB (Translation Lookaside Buffer) caches and switching memory page tables. | **Low Overhead**. Only registers, program counter, and stack pointer must be swapped.                  |
| **Communication**     | Slow via IPC (Pipes, Unix Sockets, Shared Memory, Network calls).                                                        | Fast via shared Heap references (requires synchronization).                                            |
| **Fault Isolation**   | High. If one process crashes, other processes remain unaffected.                                                         | Low. An unhandled exception or memory corruption in one thread can destabilize the entire JVM process. |

---

### 3. Advantages of Multithreading

1. **Enhanced CPU Utilization:** Prevents CPU cores from sitting idle while executing blocking operations (e.g., waiting on secondary storage I/O or network responses).
2. **Improved Application Responsiveness:** In graphical interfaces and web servers, background worker threads handle time-consuming computations while the UI or request accept thread remains responsive to user events.
3. **Simultaneous Task Processing:** Real-world domain problems (e.g., serving HTTP client connections concurrently in Netty/Tomcat, parallel data sorting, video rendering) map naturally to multiple threads.
4. **Economical Resource Usage:** Spawning a new thread inside an existing JVM consumes significantly less physical RAM and OS resources than spawning an independent JVM process.

---

### 4. The Thread Lifecycle & The 6 Thread States

The `java.lang.Thread.State` enum defines exactly **six distinct thread states** recognized by the JVM:

```
                                Thread State Machine
                                         │
                                  [ NEW STATE ]
                                         │
                             thread.start() invoked
                                         │
                                         ▼
                                 [ RUNNABLE STATE ]
                     ┌───────────────────┴───────────────────┐
                     │ Ready to run on CPU   Executing on CPU│
                     └───────────────────┬───────────────────┘
                                         │
         ┌───────────────────────────────┼───────────────────────────────┐
         │                               │                               │
         ▼ Waiting on lock               ▼ sleep() / wait(t)             ▼ wait() / join()
  [ BLOCKED STATE ]           [ TIMED_WAITING STATE ]            [ WAITING STATE ]
         │                               │                               │
         │ Lock acquired                 │ Timeout or notify             │ notify() / notifyAll()
         └───────────────────────────────┼───────────────────────────────┘
                                         │
                                         ▼ Run method exits
                                [ TERMINATED STATE ]
```

#### Detailed Breakdown of the 6 States:

1. **`NEW`:** The thread object has been instantiated (`new Thread()`), but `start()` has not yet been invoked. It is simply a Java object on the Heap; no native OS thread has been allocated.
2. **`RUNNABLE`:** The thread is executing inside the JVM. This state actually combines two operating-system concepts:
   - **Ready:** Waiting in the OS ready queue for the CPU scheduler to allocate a timeslice.
   - **Running:** Actively executing instructions on a CPU core.
3. **`BLOCKED`:** The thread is suspended, waiting to acquire an intrinsic monitor lock to enter a `synchronized` block or method.
4. **`WAITING`:** The thread is waiting indefinitely for another thread to perform a specific action. Triggered by:
   - `Object.wait()` (without timeout)
   - `Thread.join()` (without timeout)
   - `LockSupport.park()`
5. **`TIMED_WAITING`:** The thread is paused for a specified waiting period. Triggered by:
   - `Thread.sleep(millis)`
   - `Object.wait(timeout)`
   - `Thread.join(timeout)`
   - `LockSupport.parkNanos()` / `parkUntil()`
6. **`TERMINATED`:** The thread has completed execution either by returning from its `run()` method normally or by terminating due to an uncaught exception. A terminated thread cannot be restarted.

---

### 5. Thread Implementation Mechanisms: `Thread` vs. `Runnable`

Java provides two primary foundational mechanisms for defining executable thread units: extending `java.lang.Thread` or implementing `java.lang.Runnable`.

```
                         Thread Creation Strategies
                                     │
         ┌───────────────────────────┴───────────────────────────┐
         ▼                                                       ▼
Extending java.lang.Thread                                Implementing java.lang.Runnable
class Worker extends Thread {                             class Task implements Runnable {
    public void run() { ... }                                 public void run() { ... }
}                                                         }
Worker t = new Worker();                                  Thread t = new Thread(new Task());
- Consumes single inheritance slot                        - Preserves single inheritance (extends another class)
- Couples task to thread mechanics                        - Separates task definition from thread execution
- Inflexible for thread pooling                           - Ideal for ExecutorService thread pools
```

#### 5.1 Strategy 1: Extending `java.lang.Thread`

```java
public class CustomThread extends Thread {
    @Override
    public void run() {
        System.out.println("Executing thread: " + Thread.currentThread().getName());
    }

    public static void main(String[] args) {
        CustomThread t1 = new CustomThread();
        t1.start(); // Allocates OS thread and invokes run() asynchronously
    }
}
```

#### 5.2 Strategy 2: Implementing `java.lang.Runnable` (Industry Standard)

The `Runnable` interface represents an abstract task contract containing a single abstract method:

```java
@FunctionalInterface
public interface Runnable {
    void run();
}
```

```java
public class DatabaseSyncTask implements Runnable {
    @Override
    public void run() {
        System.out.println("Running background synchronization task.");
    }

    public static void main(String[] args) {
        DatabaseSyncTask task = new DatabaseSyncTask();
        Thread worker = new Thread(task, "SyncWorker-Thread");
        worker.start();
    }
}
```

#### 5.3 Why `Runnable` is Superior to Extending `Thread`:

1. **Preserves Inheritance:** Java allows a class to extend only **one** superclass. Extending `Thread` prevents the class from extending any domain class (e.g., `extends BaseService`).
2. **Separation of Concerns:** `Runnable` models a unit of computation (**the task**), whereas `Thread` models the execution infrastructure (**the worker**). Decoupling them allows tasks to be queued, submitted to `ExecutorService` pools, or scheduled independently.
3. **Reusability:** A single `Runnable` instance can be passed to and executed across multiple distinct threads or thread pools.

---

### 6. Starting Threads: `start()` vs. `run()`

> ⚠️ **THE #1 MOST COMMON CDAC MULTITHREADING TRAP:**
> _What happens if you invoke `t.run()` instead of `t.start()`?_

```
Invocation: thread.start()                        Invocation: thread.run()
┌─────────────────────────────────┐               ┌─────────────────────────────────┐
│ Thread-1 Stack Frame            │               │ main() Thread Stack Frame       │
│ - run() executes independently  │               │ - run() executes SYNCHRONOUSLY  │
├─────────────────────────────────┤               │   on the CALLER stack           │
│ (Allocates new OS native thread)│               │ (NO new thread is created!)     │
└─────────────────────────────────┘               └─────────────────────────────────┘
```

- **`thread.start()`:**
  1. Performs JVM state verification.
  2. Spawns a **new native operating system thread**.
  3. Allocates a new JVM Stack frame.
  4. Places the thread in the `RUNNABLE` state.
  5. The OS scheduler calls the internal native method `start0()`, which invokes `run()` asynchronously on the newly created thread.
- **`thread.run()`:**
  - Treats `run()` as a **standard regular instance method call**.
  - Executes synchronously inside the **caller's current thread stack** (e.g., the `main` thread).
  - **Zero multithreading occurs.**

```java
Thread t = new Thread(() -> {
    System.out.println("Active: " + Thread.currentThread().getName());
});

t.run();   // PRINTS: "Active: main" (Runs synchronously on caller thread!)
t.start(); // PRINTS: "Active: Thread-0" (Runs asynchronously on newly spawned thread!)
```

#### Can You Restart a Thread?

**NO.** A thread can only be started **once**. Once a thread enters `RUNNABLE` or transitions to `TERMINATED`, calling `start()` a second time throws **`java.lang.IllegalThreadStateException`**.

```java
Thread t = new Thread(() -> {});
t.start();
t.start(); // RUNTIME ERROR: java.lang.IllegalThreadStateException!
```

---

### 7. Thread Priorities & OS Scheduling

Every thread in Java is assigned a priority integer ranging from **1 to 10**:

```java
public static final int MIN_PRIORITY = 1;
public static final int NORM_PRIORITY = 5; // Default priority assigned to main thread
public static final int MAX_PRIORITY = 10;
```

```java
Thread t = new Thread(task);
t.setPriority(Thread.MAX_PRIORITY); // 10
```

#### 7.1 How Thread Scheduling Operates

The JVM relies on the host Operating System's native scheduler (preemptive, priority-based time-slicing).

```
                      Preemptive Time-Slicing Scheduler
┌─────────────────────────────────────────────────────────────────────────────┐
│ Ready Queue: [ Thread A (Pri 10) ] ──► Allocated CPU Timeslice (e.g., 20ms) │
│              [ Thread B (Pri 5)  ]                                          │
│              [ Thread C (Pri 1)  ]                                          │
└─────────────────────────────────────────────────────────────────────────────┘
```

#### 7.2 Why Priorities are NOT Guaranteed:

- **Platform Dependence:** Java thread priorities are merely hints to the host OS scheduler. Windows maps 10 priorities into 7 native scheduling levels; Linux completely ignores Java thread priorities by default (scheduling all normal threads via the Completely Fair Scheduler / CFS).
- **Starvation Hazard:** High-priority threads can starve lower-priority threads if the scheduler strictly favors higher priorities.
- **Production Rule:** Never write business logic that depends on Java thread priorities for correctness.

---

### 8. Core Thread Control Methods: `sleep()`, `join()`, and `yield()`

```
                                Thread Control Methods
                                          │
         ┌────────────────────────────────┼────────────────────────────────┐
         ▼                                ▼                                ▼
  Thread.sleep(ms)                   thread.join()                   Thread.yield()
- Pauses current thread            - Caller waits for target        - Voluntarily relinquishes
  for fixed duration                 thread to complete               current CPU timeslice
- Holds monitor locks              - Can wait indefinitely          - Hint to OS scheduler
- Enters TIMED_WAITING             - Enters WAITING/TIMED_WAITING   - Remains RUNNABLE
```

#### 8.1 `Thread.sleep(long millis)`

Suspends the **currently executing thread** for a specified duration.

- **Static Method:** Always affects the thread executing the call (`Thread.sleep()`), never a target object reference (`t1.sleep()` still puts the calling thread to sleep!).
- **Lock Retention Invariant:** `sleep()` **DOES NOT RELEASE MONITOR LOCKS**. If a thread sleeps inside a `synchronized` block, all other threads seeking that lock are blocked until it wakes and exits the block.
- **Interrupted Handling:** Throws **`java.lang.InterruptedException`** if another thread calls `interrupt()` while it is asleep.

#### 8.2 `thread.join()`

Forces the currently executing thread to **pause and wait** until the target thread on which `join()` was invoked terminates.

```java
public class JoinDemo {
    public static void main(String[] args) throws InterruptedException {
        Thread worker = new Thread(() -> {
            try { Thread.sleep(2000); } catch (InterruptedException e) {}
            System.out.println("Worker completed complex data loading.");
        });

        worker.start();

        System.out.println("Main waiting on worker...");
        worker.join(); // Main thread transitions to WAITING until 'worker' finishes!

        System.out.println("Main resumes processing loaded data.");
    }
}
```

- Under the hood, `join()` is implemented using a `while (isAlive()) { wait(0); }` loop synchronized on the thread's own object instance. When a thread terminates, the JVM internally invokes `notifyAll()` on that thread object.

#### 8.3 `Thread.yield()`

A static advisory hint from the current thread to the OS scheduler indicating that it is willing to yield its current CPU timeslice to allow other threads of **equal priority** to run.

- **No Guarantee:** The scheduler is free to ignore the hint entirely.
- **State Transition:** The yielding thread transitions from running back to the `RUNNABLE` ready queue; it **does not block** and does not enter `WAITING`.

---

### 9. Daemon Threads

Java threads fall into two distinct categories: **User Threads** and **Daemon Threads**.

```
                           JVM Termination Rule
                                     │
                 Are there ANY non-daemon User Threads alive?
                               ┌─────┴─────┐
                          Yes  │           │ No
                               ▼           ▼
                      JVM Continues Execution   JVM Halts & Shuts Down
                      (Daemon threads kept running) (All Daemons killed immediately!)
```

- **User Thread (Default):** High-priority threads designed to run application tasks. The JVM will **not terminate** as long as at least one user thread is alive, even if `main()` has finished.
- **Daemon Thread:** Low-priority background service provider threads (e.g., the JVM Garbage Collector, finalizer threads, monitoring daemons).
- **JVM Shutdown Invariant:** When the last user thread terminates, the JVM halts immediately. All remaining daemon threads are **abruptly killed**—their `finally` blocks may not execute, and resource streams may not flush cleanly!

```java
Thread daemonWorker = new Thread(() -> {
    while (true) {
        System.out.println("Background telemetry heartbeat...");
        try { Thread.sleep(1000); } catch (InterruptedException e) {}
    }
});

// MANDATORY INVARIANT: Must be marked before calling start()!
daemonWorker.setDaemon(true);
daemonWorker.start();
// If you call daemonWorker.setDaemon(true) AFTER start(),
// it throws java.lang.IllegalThreadStateException!
```

---

### 10. Thread Groups (`java.lang.ThreadGroup`)

A `ThreadGroup` represents a set of threads managed as a collective unit. It allows bulk operations (e.g., interrupting all threads, inspecting active thread counts).

```java
ThreadGroup networkGroup = new ThreadGroup("Network-Workers");
Thread t1 = new Thread(networkGroup, task, "SocketWorker-1");
Thread t2 = new Thread(networkGroup, task, "SocketWorker-2");

System.out.println("Active count: " + networkGroup.activeCount());
networkGroup.interrupt(); // Interrupts all threads in the group
```

- **Modern Assessment:** `ThreadGroup` is largely considered obsolete and flawed in modern Java (poor exception handling, non-thread-safe methods). It has been superseded by the **Java Concurrency Utilities** (`ExecutorService`, `ThreadPoolExecutor`).

---

### 11. Modern Thread Creation: Anonymous Inner Classes & Lambdas

#### 11.1 Anonymous Inner Class Approach (Pre-Java 8)

```java
Thread t = new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println("Anonymous inner class execution: " +
                           Thread.currentThread().getName());
    }
});
t.start();
```

#### 11.2 Lambda-Based Approach (Java 8+)

Because `Runnable` is a Single Abstract Method (`@FunctionalInterface`), it can be expressed directly as a concise lambda expression:

```java
// Inline lambda syntax:
Thread t = new Thread(() -> {
    System.out.println("Lambda thread running: " + Thread.currentThread().getName());
}, "LambdaWorker");

t.start();
```

---

### 12. High-Yield CDAC Exam Traps & Interview Questions

- **Q1: What happens if `main()` finishes executing while background threads are still running?**
  - _Answer:_ If the background threads are **User Threads**, the JVM **continues running** until all user threads finish their `run()` methods. If the background threads were marked **Daemon Threads** (`setDaemon(true)`), the JVM terminates immediately, killing the daemons mid-execution.

- **Q2: Does `Thread.sleep()` release object locks? How does it differ from `Object.wait()`?**
  - _Answer:_
    - `Thread.sleep()` **never releases monitor locks**. The sleeping thread holds all acquired synchronization monitors, preventing other threads from accessing those blocks.
    - `Object.wait()` **releases the monitor lock** of the target object, allowing waiting threads to contend for the monitor while it sleeps in the Wait Set.

- **Q3: What occurs if you invoke `t.start()` twice on the same Thread instance?**
  - _Answer:_ It throws a runtime **`java.lang.IllegalThreadStateException`**. Once a thread starts, its internal lifecycle state prevents it from ever being transitioned back to `NEW`.

- **Q4: What happens if you call `setDaemon(true)` after calling `start()`?**
  - _Answer:_ It throws **`java.lang.IllegalThreadStateException`**. The daemon status of a thread must be set while the thread is still in the `NEW` state, before it transitions to `RUNNABLE`.

- **Q5: What is the output of executing this snippet?**
  ```java
  public class ThreadTrap {
      public static void main(String[] args) {
          Thread t = new Thread() {
              public void run() {
                  System.out.print("A");
              }
          };
          t.start();
          t.run();
      }
  }
  ```

  - _Answer:_ `AA` (or non-deterministic ordering like `AA` with indeterminate thread execution interleaving).
  - _Explanation:_ `t.start()` requests a new thread to run `run()` asynchronously (printing `"A"`). Meanwhile, `t.run()` executes the `run()` method **synchronously on the `main` thread** directly (printing `"A"`). Both execute, resulting in two `"A"` prints.
