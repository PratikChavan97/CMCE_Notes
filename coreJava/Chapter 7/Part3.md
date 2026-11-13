# Module 07: Concurrency, Multithreading, and Modern Java

## Unit 7.3: Advanced Concurrency Concepts

### 1. Introduction to `java.util.concurrent` (JUC)

Introduced in **Java 5 (JSR 166)**, led by Doug Lea, the `java.util.concurrent` package (along with its subpackages `atomic` and `locks`) transformed concurrency in Java.

#### Limitations of Traditional Concurrency (Java 1.0–1.4):

- **Thread Creation Overhead:** Spawning an unmanaged `new Thread()` for every request creates native OS threads indiscriminately, consuming roughly 1 MB of stack memory per thread and risking operating system exhaustion.
- **No Return Values or Checked Exceptions:** `Runnable.run()` has a `void` return signature and cannot throw checked exceptions.
- **Coarse-Grained Locking:** Intrinsic `synchronized` locks lack timeouts, fairness policies, interruptibility, and the ability to inspect lock availability.
- **Low-Level Coordination:** Using `wait()`, `notify()`, and `notifyAll()` requires manual, error-prone condition loops and is prone to lost wakeups and deadlocks.

```
                    java.util.concurrent Architectural Layers
┌─────────────────────────────────────────────────────────────────────────────┐
│ High-Level Utilities: CountDownLatch, CyclicBarrier, Semaphore, Phaser      │
├─────────────────────────────────────────────────────────────────────────────┤
│ Execution Framework: Executor, ExecutorService, ForkJoinPool, Future        │
├─────────────────────────────────────────────────────────────────────────────┤
│ Concurrent Collections: ConcurrentHashMap, CopyOnWriteArrayList, Queues     │
├─────────────────────────────────────────────────────────────────────────────┤
│ Low-Level Synchronizers: AbstractQueuedSynchronizer (AQS), ReentrantLock    │
├─────────────────────────────────────────────────────────────────────────────┤
│ Hardware Primitives: Lock-free CAS (Unsafe / VarHandle), Atomic Variables   │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

### 2. The Executor Framework Architecture

The Executor framework decouples **task submission** from the underlying **mechanics of task execution** (thread allocation, scheduling, and pooling).

```
                                java.util.concurrent.Executor
                                              │
                                              ▼
                             java.util.concurrent.ExecutorService
                                              │
         ┌────────────────────────────────────┼────────────────────────────────────┐
         ▼                                    ▼                                    ▼
ScheduledExecutorService            ThreadPoolExecutor                       ForkJoinPool
(Delayed/Periodic Execution)        (Standard Fixed/Cached Pools)            (Work-Stealing / Parallel)
```

#### 2.1 The Three Core Interfaces

1. **`Executor`:** The root interface containing a single method:
   ```java
   void execute(Runnable command);
   ```
2. **`ExecutorService`:** Extends `Executor`, adding lifecycle management (`shutdown()`, `shutdownNow()`), task submission returning tracking handles (`submit()`), and bulk execution (`invokeAll()`, `invokeAny()`).
3. **`ScheduledExecutorService`:** Extends `ExecutorService` with delayed execution and periodic task scheduling capabilities.

---

### 3. `Callable<V>` vs. `Runnable`

`Callable<V>` (introduced in Java 5) models a value-returning, exception-capable task.

```
+--------------------------------------------------------------------------------------------------+
| java.lang.Runnable (Java 1.0)                    java.util.concurrent.Callable<V> (Java 5)       |
+--------------------------------------------------------------------------------------------------+
| public interface Runnable {                      @FunctionalInterface                            |
|     void run();                                  public interface Callable<V> {                  |
| }                                                    V call() throws Exception;                  |
|                                                  }                                               |
| • Returns void                                   • Returns a parameterized value of type V       |
| • Cannot throw checked exceptions                • Can throw checked Exception                   |
| • Invoked via new Thread(r) or executor          • Submitted to ExecutorService via submit()     |
+--------------------------------------------------------------------------------------------------+
```

---

### 4. The `Future<V>` Interface

A `Future<V>` represents the deferred result of an asynchronous computation. When a `Callable` is submitted to an `ExecutorService`, the caller receives a `Future` handle immediately.

```
Caller Thread                                                Worker Thread (Pool)
     │                                                                │
     │─── submit(Callable) ──────────────────────────────────────────►│ (Executes task...)
     │◄── Returns Future<V> immediately                               │
     │                                                                │
     │ (Performs other non-blocking work)                             │
     │                                                                │
     │─── future.get() ───────────────────────────────────────────────┼── (Blocks until done)
     │◄── Yields computed result V ───────────────────────────────────│
```

#### Core Methods of `Future<V>`:

- **`V get()`:** Blocks until the computation completes and returns the result. Throws `InterruptedException` and `ExecutionException` (which wraps any exception thrown by `call()`).
- **`V get(long timeout, TimeUnit unit)`:** Blocks for up to the specified timeout. Throws `TimeoutException` if the result is not ready within the window.
- **`boolean isDone()`:** Returns `true` if the task completed, was cancelled, or failed.
- **`boolean cancel(boolean mayInterruptIfRunning)`:** Attempts to cancel task execution.
- **`boolean isCancelled()`:** Returns `true` if the task was cancelled before normal completion.

```java
ExecutorService executor = Executors.newSingleThreadExecutor();
Callable<Integer> calculationTask = () -> {
    Thread.sleep(1000);
    return 42 * 2;
};

Future<Integer> future = executor.submit(calculationTask);

// Non-blocking status check:
while (!future.isDone()) {
    System.out.println("Processing background calculation...");
    Thread.sleep(200);
}

// Blocks if not yet finished, then extracts value:
Integer result = future.get(2, TimeUnit.SECONDS);
System.out.println("Result: " + result);

executor.shutdown();
```

---

### 5. Thread Pools & `ThreadPoolExecutor`

A **Thread Pool** contains a collection of pre-allocated, reusable worker threads that process tasks from an internal **Work Queue** (`BlockingQueue`). This avoids the high overhead of repeatedly creating and tearing down threads.

```
Task Submissions
 ──► [ Task 1 ] ──► ┌────────────────────────────────────┐
 ──► [ Task 2 ] ──► │ BlockingQueue (Work Queue)         │
 ──► [ Task 3 ] ──► └─────────────────┬──────────────────┘
                                      │
            ┌─────────────────────────┼─────────────────────────┐
            ▼                         ▼                         ▼
   ┌─────────────────┐       ┌─────────────────┐       ┌─────────────────┐
   │ Worker Thread 1 │       │ Worker Thread 2 │       │ Worker Thread 3 │
   └─────────────────┘       └─────────────────┘       └─────────────────┘
```

#### 5.1 Factory Presets via `java.util.concurrent.Executors`

- **`Executors.newFixedThreadPool(int nThreads)`:** Creates a pool with a fixed number of threads backed by an unbounded `LinkedBlockingQueue`.
- **`Executors.newCachedThreadPool()`:** Creates a dynamic pool that spawns new threads as needed and reclaims threads idle for more than 60 seconds. Uses a zero-capacity `SynchronousQueue`.
- **`Executors.newSingleThreadExecutor()`:** Uses a single worker thread backed by an unbounded queue. Guarantees sequential task execution.
- **`Executors.newScheduledThreadPool(int corePoolSize)`:** Creates a pool capable of executing commands after a given delay or periodically.

> ⚠️ **The Production Warning (OutOfMemoryError Hazard):**
> Pre-configured factory methods from `Executors` should be avoided in production environments:
>
> - `newFixedThreadPool` uses an **unbounded queue** (`LinkedBlockingQueue` without capacity limits). Under high load, pending tasks consume heap memory unchecked until the JVM crashes with `OutOfMemoryError: Java heap space`.
> - `newCachedThreadPool` has an **unbounded maximum thread limit** (`Integer.MAX_VALUE`). High load can spawn thousands of threads, causing CPU thrashing and `OutOfMemoryError: unable to create new native thread`.
>
> **Best Practice:** Always instantiate **`ThreadPoolExecutor`** directly with explicit, bounded constraints.

#### 5.2 Direct `ThreadPoolExecutor` Constructor Anatomy

```java
public ThreadPoolExecutor(
    int corePoolSize,                      // 1. Base number of threads kept alive
    int maximumPoolSize,                   // 2. Maximum ceiling of threads permitted
    long keepAliveTime, TimeUnit unit,     // 3. Max idle duration for threads > corePoolSize
    BlockingQueue<Runnable> workQueue,     // 4. Bounded queue holding pending tasks
    ThreadFactory threadFactory,           // 5. Factory for naming/daemon status
    RejectedExecutionHandler handler       // 6. Policy invoked when queue & pool are full
)
```

```
                        Task Ingestion Decision Tree
                                     │
                             New Task Arrives
                                     │
                 Active Threads < corePoolSize?
                               ┌─────┴─────┐
                          Yes  │           │ No
                               ▼           ▼
                      Spawn Core Thread   Is workQueue full?
                                           ┌─────┴─────┐
                                      No   │           │ Yes
                                           ▼           ▼
                                   Enqueue Task   Active Threads < maximumPoolSize?
                                                       ┌─────┴─────┐
                                                  Yes  │           │ No
                                                       ▼           ▼
                                               Spawn Extra Thread  Trigger RejectedExecutionHandler!
```

#### 5.3 Saturation / Rejection Policies (`RejectedExecutionHandler`)

1. **`AbortPolicy` (Default):** Throws `RejectedExecutionException`.
2. **`CallerRunsPolicy`:** Runs the task directly on the **caller thread** that invoked `submit()`. Slows down incoming tasks naturally by throttling the producer.
3. **`DiscardPolicy`:** Silently drops the rejected task without notification.
4. **`DiscardOldestPolicy`:** Drops the oldest unhandled task from the head of the queue and retries task submission.

---

### 6. `ScheduledExecutorService`

Used for tasks that need to run after a delay or repeat at fixed intervals.

```java
ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(2);

// 1. One-shot delayed execution:
scheduler.schedule(() -> System.out.println("Delayed task"), 5, TimeUnit.SECONDS);

// 2. Fixed Rate: Fires at constant intervals regardless of task run duration
// (Period = time between START of execution N and START of execution N+1)
scheduler.scheduleAtFixedRate(() -> doWork(), 1, 3, TimeUnit.SECONDS);

// 3. Fixed Delay: Waits for a set pause AFTER the task finishes
// (Delay = time between COMPLETION of execution N and START of execution N+1)
scheduler.scheduleWithFixedDelay(() -> doLongCleanup(), 1, 3, TimeUnit.SECONDS);
```

---

### 7. Advanced Synchronization Utilities

```
                        JUC High-Level Synchronizers
                                     │
         ┌───────────────────────────┼───────────────────────────┐
         ▼                           ▼                           ▼
  CountDownLatch                CyclicBarrier                 Semaphore
- One-time countdown gate    - Reusable rendezvous point   - Resource permit throttle
- Workers decrement count    - Workers wait for each other - Restricts concurrent access
- Non-reusable               - Reusable via reset()        - Supports acquisition permits
```

#### 7.1 `CountDownLatch`

A synchronization utility that blocks one or more threads until a set of operations performed in other threads completes.

- **Constructor:** `new CountDownLatch(int count)` initializes the internal counter.
- **`countDown()`:** Decrements the counter. Once it hits zero, all waiting threads are released.
- **`await()`:** Causes the calling thread to block until the counter reaches zero.
- **Non-Reusable:** Once the latch reaches zero, it cannot be reset.

```java
CountDownLatch latch = new CountDownLatch(3);

// Three worker threads:
for (int i = 0; i < 3; i++) {
    new Thread(() -> {
        initService();
        latch.countDown(); // Decrements internal counter
    }).start();
}

latch.await(); // Main thread blocks until counter reaches 0
System.out.println("All services initialized. System startup complete.");
```

#### 7.2 `CyclicBarrier`

A reusable synchronization point where a fixed number of threads must all converge before any are allowed to proceed.

- **Constructor:** `new CyclicBarrier(int parties, Runnable barrierAction)`.
- **`await()`:** Signals that the calling thread has reached the barrier and then blocks. Once all $N$ threads call `await()`, the optional `barrierAction` executes, the barrier trips open, and all threads resume.
- **Reusable:** Automatically resets its count to $N$ after tripping, or can be reset manually via `reset()`.

```java
CyclicBarrier barrier = new CyclicBarrier(3, () -> {
    System.out.println("Phase complete. Aggregating sub-results...");
});

Runnable task = () -> {
    System.out.println(Thread.currentThread().getName() + " working on Step 1");
    barrier.await(); // Waits for all 3 threads to reach this point
    System.out.println(Thread.currentThread().getName() + " working on Step 2");
};
```

#### 7.3 `Semaphore`

Maintains a set of virtual permits to throttle access to a bounded physical or logical resource (e.g., limiting concurrent outbound HTTP calls).

- **Constructor:** `new Semaphore(int permits, boolean fair)`.
- **`acquire()`:** Acquires a permit, blocking if none are available.
- **`release()`:** Returns a permit back to the semaphore pool.

```java
Semaphore limiter = new Semaphore(3); // Allows at most 3 concurrent accesses

public void callExternalApi() throws InterruptedException {
    limiter.acquire(); // Blocks if 3 threads are already executing
    try {
        executeHttpCall();
    } finally {
        limiter.release(); // Guarantees permit is returned
    }
}
```

---

### 8. Explicit Locks: `ReentrantLock` & Condition Variables

`java.util.concurrent.locks.ReentrantLock` implements `Lock`, providing an explicit, reentrant mutual exclusion lock that serves as an alternative to intrinsic `synchronized` blocks.

```
+--------------------------------------------------------------------------------------------------+
| Intrinsic Lock (synchronized)                   Explicit Lock (ReentrantLock)                    |
+--------------------------------------------------------------------------------------------------+
| Implicit acquisition and release                 Explicit lock() and unlock() in try-finally     |
| Cannot poll for locks (blocks indefinitely)      Supports tryLock(timeout) with back-off         |
| Cannot be interrupted while waiting              Supports lockInterruptibly()                    |
| Unfair lock acquisition only                     Supports configurable fairness (FIFO ordering)  |
| Single wait-set per object (wait/notify)         Multiple wait-sets via Condition objects        |
+--------------------------------------------------------------------------------------------------+
```

#### 8.1 The Canonical `ReentrantLock` Template

```java
Lock lock = new ReentrantLock(true); // 'true' enables FIFO fairness policy

lock.lock(); // Explicit acquisition
try {
    // Critical Section: mutate shared state
} finally {
    lock.unlock(); // ALWAYS release in a finally block to avoid permanent deadlocks!
}
```

#### 8.2 Non-Blocking Lock Polling via `tryLock()`

```java
if (lock.tryLock(500, TimeUnit.MILLISECONDS)) {
    try {
        processTask();
    } finally {
        lock.unlock();
    }
} else {
    // Lock could not be acquired within 500ms; execute fallback to prevent deadlock
    handleContentionFallback();
}
```

#### 8.3 Multiple Wait-Sets via `Condition`

`Condition` decouples object monitor methods (`wait`, `notify`) into distinct, independent notification wait-sets bound to a single `Lock` instance:

```java
Lock lock = new ReentrantLock();
Condition notFull  = lock.newCondition(); // Dedicated queue for producers
Condition notEmpty = lock.newCondition(); // Dedicated queue for consumers

// Producer waits exclusively on notFull:
lock.lock();
try {
    while (isFull()) notFull.await();
    insertItem();
    notEmpty.signal(); // Wakes ONLY waiting consumers!
} finally {
    lock.unlock();
}
```

---

### 9. Atomic Variables & Hardware CAS (Compare-And-Swap)

The `java.util.concurrent.atomic` package provides classes (`AtomicInteger`, `AtomicLong`, `AtomicBoolean`, `AtomicReference`) that support lock-free, thread-safe operations on single variables.

```
                             Hardware-Level CAS Operation
                                          │
            CAS(Memory Address V, Expected Old Value A, New Update Value B)
                                          │
                        Does value at V equal Expected A?
                                   ┌──────┴──────┐
                              Yes  │             │ No
                                   ▼             ▼
             Atomically swap V to B (Success)   Do nothing & return false (Failed)
```

#### 9.1 How CAS Operates

Traditional locks rely on **Pessimistic Concurrency Control**: they block other threads assuming collisions are likely.
Atomic classes rely on **Optimistic Concurrency Control** via hardware-supported atomic CPU instructions (such as `CMPXCHG` on x86 architectures):

1. A thread reads the current value $A$.
2. It computes the new value $B = A + 1$.
3. It calls the CPU's atomic instruction: _"If memory address $V$ still contains $A$, swap it to $B$. If not, fail and return false."_
4. If CAS fails because another thread updated $V$ first, the thread simply retries in a loop (**lock-free spin**).

```java
AtomicInteger counter = new AtomicInteger(0);

// Thread-safe atomic increment without locks:
int nextVal = counter.incrementAndGet(); // Under the hood: CAS retry loop

// Atomic update using functional lambda:
counter.updateAndGet(x -> x * 2);
```

#### 9.2 High-Contention Optimization: `LongAdder` (Java 8+)

Under high thread contention, thousands of threads spinning on a single `AtomicLong` CAS variable cause CPU bus thrashing and repeated retries.
**`LongAdder`** solves this by maintaining an internal array of cell counters. Threads hash to different cells and increment them independently without contention. When the final total is needed, calling `.sum()` aggregates all cells.

---

### 10. The `volatile` Keyword & The Java Memory Model (JMM)

Modern CPU architectures use multi-level hardware caches (L1, L2, L3) to avoid slow main memory access. Each CPU core reads and writes data to its local cache.

```
Core 1 Cache                                                 Core 2 Cache
┌─────────────────────────────────┐                         ┌─────────────────────────────────┐
│ Local Cache Copy: flag = false  │                         │ Local Cache Copy: flag = true   │
└────────────────┬────────────────┘                         └────────────────┬────────────────┘
                 │                                                           │
                 ▼                                                           ▼
═══════════════════════════════════════════════════════════════════════════════════════════════════
                             Main RAM Memory (Shared State)
                                     flag = true
═══════════════════════════════════════════════════════════════════════════════════════════════════
```

#### 10.1 The Visibility Problem

If Thread 1 updates a regular variable `flag = true`, that write may sit in Core 2's write buffer without being flushed to main RAM immediately. Thread 2 running on Core 1 reads its own local cache and continues to see `flag = false`, entering an infinite loop.

#### 10.2 Guarantees of `volatile`:

1. **Memory Visibility:** Writes to a `volatile` variable are immediately flushed to main memory. Reads from a `volatile` variable always bypass local CPU caches and pull directly from main memory (**Happens-Before Relationship**).
2. **Instruction Reordering Prevention:** The compiler and CPU out-of-order execution hardware are prohibited from reordering instructions across a `volatile` read or write, enforced via hardware **Memory Barriers** (`LoadLoad`, `LoadStore`, `StoreStore`, `StoreLoad`).

> ⚠️ **The Volatility Fallacy (volatile is NOT Atomic!):**
> Declaring `volatile int count = 0;` ensures visibility, but it **DOES NOT make `count++` thread-safe!**
> Incrementation is a compound 3-step operation (Read $\rightarrow$ Modify $\rightarrow$ Write). If two threads read the same volatile value simultaneously, both compute the same increment and write it back, losing an update.

---

### 11. Comparison: `synchronized` vs. `volatile`

| Dimension                | `volatile`                                              | `synchronized`                                              |
| ------------------------ | ------------------------------------------------------- | ----------------------------------------------------------- |
| **Scope**                | Can be applied to **fields only**                       | Can be applied to **methods and code blocks**               |
| **Visibility Guarantee** | ✅ **Yes** (Direct main memory sync)                    | ✅ **Yes** (Flushed upon lock release)                      |
| **Atomicity Guarantee**  | ❌ **No** (Non-atomic for compound actions like `++`)   | ✅ **Yes** (Ensures full mutual exclusion)                  |
| **Locking Overhead**     | **Non-blocking** (Zero context switches or locks)       | **Blocking** (Threads may enter `BLOCKED` state)            |
| **Best Used For**        | Status flags, shutdown triggers, double-checked locking | Compound state updates, critical sections, invariant checks |

---

### 12. Concurrency Best Practices

1. **Prefer Immutability:** Immutable objects (`final` fields, Java Records) are inherently thread-safe and can be shared freely without locks.
2. **Favor High-Level Frameworks Over Low-Level Constructs:** Use `ExecutorService` instead of manual `new Thread()`, and prefer concurrent collections over manual `synchronized` wrappers.
3. **Always Release Explicit Locks in `finally` Blocks:** Avoid holding locks across slow operations (network calls, disk I/O).
4. **Use Bounded Thread Pools:** Constrain thread counts and queue capacities to protect against memory exhaustion.
5. **Always Handle `InterruptedException` Properly:** Restore interrupted status or rethrow the exception; never swallow it silently:
   ```java
   try {
       Thread.sleep(1000);
   } catch (InterruptedException e) {
       Thread.currentThread().interrupt(); // Restore interrupted status!
       logger.warn("Task was cancelled or interrupted");
   }
   ```

---

### 13. High-Yield CDAC Exam Traps & Interview Questions

- **Q1: Why is Double-Checked Locking on singletons broken without the `volatile` keyword?**
  - _Answer:_ The statement `instance = new Singleton()` is not atomic. It involves three steps:
    1. Allocate memory.
    2. Run constructor `<init>()`.
    3. Assign memory address to `instance`.
       The compiler or CPU can reorder these steps to $1 \rightarrow 3 \rightarrow 2$. Without `volatile`, Thread B could observe `instance != null` after Step 3 before the constructor (Step 2) finishes, returning an uninitialized object. Marking the variable `volatile` prevents this instruction reordering.

- **Q2: What is the key functional difference between `CountDownLatch` and `CyclicBarrier`?**
  - _Answer:_
    - `CountDownLatch` cannot be reset; once its count reaches zero, it cannot be reused. It is designed for one-off coordination (e.g., waiting for initialization steps to finish).
    - `CyclicBarrier` can be reset and reused across multiple phases via `.reset()` or by letting all parties trip the barrier. It is designed for threads that must repeatedly synchronize at phase boundaries.

- **Q3: What happens if an exception is thrown inside a `Callable.call()` task submitted to an `ExecutorService`?**
  - _Answer:_ The exception is captured by the framework and stored. It is not surfaced immediately. When the calling thread attempts to retrieve the result via **`future.get()`**, the captured exception is rethrown wrapped inside an **`ExecutionException`**. Calling `e.getCause()` reveals the original exception.

- **Q4: Why does `newFixedThreadPool` risk an `OutOfMemoryError`?**
  - _Answer:_ `newFixedThreadPool` uses an unbounded `LinkedBlockingQueue` (capacity `Integer.MAX_VALUE`). If tasks arrive faster than worker threads can process them, the queue grows without bounds, eventually exhausting the JVM Heap and throwing `java.lang.OutOfMemoryError: Java heap space`.

- **Q5: Can `AtomicInteger` protect a compound transaction involving two different integers?**
  - _Answer:_ **No.** `AtomicInteger` operations are atomic only for individual variables. Coordinating state across two distinct variables requires an explicit lock (`ReentrantLock`, `synchronized`) or combining the state into a single immutable object managed by an `AtomicReference`.
