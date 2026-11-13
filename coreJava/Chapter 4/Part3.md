# Module 04: Exception Handling, Strings, and Standard APIs

## Unit 4.3: Exception Handling Fundamentals

### 1. Introduction to Exception Handling

An **exception** is an abnormal condition or event that disrupts the normal sequential flow of instructions during runtime execution.

In legacy environments (such as C), errors are signaled through integer return error codes (e.g., `-1`, `errno`). This pattern forces repetitive, cluttered `if-else` verification blocks and allows errors to be silently ignored. Java resolves this by separating operational logic from error handling through structured **Exception Handling**.

```
Traditional Procedural Model (Error Flags):
int status = openFile("data.bin");
if (status == ERROR_NOT_FOUND) { /* handle */ }
int bytes = readFile(buf);
if (bytes == ERROR_IO) { /* handle */ }

Java Structured Exception Model:
try {
    openFile("data.bin");
    readFile(buf);
    processData();
} catch (IOException e) {
    // Consolidated error recovery logic
}
```

#### 1.1 Architectural Goals of Java Exception Handling

1. **Clean Separation of Concerns:** Separates normal business logic from failure-recovery paths.
2. **Mandatory Handling Policies:** Uses compile-time checks (**Checked Exceptions**) to enforce handling of recoverable environmental failures.
3. **Propagated Call Context:** Automatically builds a descriptive diagnostic snapshot (**Stack Trace**) tracking class names, method frames, and source line numbers.
4. **Deterministic Resource Cleanliness:** Guarantees execution of cleanup code (via `finally` and `try-with-resources`) even if a runtime failure occurs.

---

### 2. Errors vs. Exceptions

```
                                java.lang.Throwable
                                         │
         ┌───────────────────────────────┴───────────────────────────────┐
         ▼                                                               ▼
  java.lang.Error                                              java.lang.Exception
  (Fatal Infrastructure Failures)                              (Application-Level Failures)
  • Hardware/JVM level malfunctions                            • Predictable programmatic or I/O faults
  • Unchecked (Not checked by compiler)                        • Subdivided into Checked & Unchecked
  • Application recovery is IMPOSSIBLE                         • Application MUST handle or recover
  • Examples: OutOfMemoryError, StackOverflowError             • Examples: IOException, SQLException
```

| Metric                   | `java.lang.Error`                                                     | `java.lang.Exception`                                                |
| :----------------------- | :-------------------------------------------------------------------- | :------------------------------------------------------------------- |
| **Origin**               | Fatal system-level or JVM runtime resource exhaustion                 | Application logic errors, invalid input, or external failures        |
| **Handling Expectation** | **Do NOT attempt to catch or recover.** Let the thread/JVM terminate. | **Expected to be caught and handled** gracefully                     |
| **Compiler Check**       | **Unchecked** (Ignored by compiler during build)                      | Can be **Checked** (e.g., `IOException`) or **Unchecked**            |
| **Typical Triggers**     | Exhausted native RAM, infinite native recursion, corrupt bytecode     | Missing files, bad network connections, null dereferences, bad casts |
| **Representative Types** | `OutOfMemoryError`, `StackOverflowError`, `VirtualMachineError`       | `IOException`, `SQLException`, `NullPointerException`                |

---

### 3. The Java Exception Hierarchy

Every class participating in exception handling is a direct or indirect descendant of **`java.lang.Throwable`**.

```
                              java.lang.Object
                                     │
                                     ▼
                            java.lang.Throwable
                                     │
         ┌───────────────────────────┴───────────────────────────┐
         ▼                                                       ▼
  java.lang.Error                                       java.lang.Exception
  ├── VirtualMachineError                               ├── IOException (Checked)
  │   ├── OutOfMemoryError                              │   ├── FileNotFoundException
  │   └── StackOverflowError                            │   └── SocketException
  ├── LinkageError                                      ├── SQLException (Checked)
  │   ├── NoClassDefFoundError                          ├── ClassNotFoundException (Checked)
  │   └── UnsupportedClassVersionError                  └── RuntimeException (Unchecked)
  └── AssertionError                                        ├── NullPointerException
                                                            ├── ArithmeticException
                                                            ├── ArrayIndexOutOfBoundsException
                                                            ├── ClassCastException
                                                            └── IllegalArgumentException
                                                                └── NumberFormatException
```

#### 3.1 `java.lang.Throwable` Root Methods

- `public String getMessage()`: Returns a detailed description of why the failure occurred.
- `public void printStackTrace()`: Prints the execution backtrace to standard error (`System.err`).
- `public StackTraceElement[] getStackTrace()`: Provides programmatic access to the stack frames.
- `public synchronized Throwable getCause()`: Returns the underlying nested exception (Exception Chaining).
- `public synchronized Throwable initCause(Throwable cause)`: Binds a causal underlying exception to the current instance.

---

### 4. Checked vs. Unchecked vs. Runtime Exceptions

```
                            Exception Categorization
                                       │
         ┌─────────────────────────────┴─────────────────────────────┐
         ▼                                                           ▼
Checked Exceptions                                          Unchecked Exceptions
(Compile-Time Enforced)                                     (Runtime Enforced)
• Direct subclasses of Exception (excluding RuntimeException)• Subclasses of RuntimeException AND Error
• Enforces Catch-or-Declare rule                             • Ignored by compiler during compilation
• Caused by external environment factors                    • Caused by flawed program logic / bugs
• Examples: IOException, SQLException                       • Examples: NullPointerException, ArithmeticException
```

#### 4.1 Checked Exceptions

- **Definition:** Exceptions checked at **compile-time**. If a method contains code that could raise a checked exception, it **must** satisfy the **Catch-or-Declare Rule**:
  1. Handle it using a `try-catch` block, OR
  2. Declare it in its method signature using the `throws` clause.
- **Philosophical Intent:** Represents conditions outside the application's direct control where recovery is plausible (e.g., a file is temporarily locked or a database connection drops).

```java
// Won't compile without try-catch or throws clause!
public void readFile(String path) throws FileNotFoundException {
    FileReader fr = new FileReader(path); // Declares "throws FileNotFoundException"
}
```

#### 4.2 Unchecked Exceptions (Runtime Exceptions)

- **Definition:** Direct subclasses of **`java.lang.RuntimeException`** (and `java.lang.Error`). The compiler **does not verify or require** handling or declaration for these exceptions.
- **Philosophical Intent:** Typically indicate **preventable programming bugs**, contract violations, or logic errors (e.g., dividing by zero, indexing beyond an array's bounds, passing illegal arguments). These should be fixed through correct code rather than defensive `try-catch` blocks.

```java
// Compiles cleanly without try-catch or throws
public void divide(int a, int b) {
    int res = a / b; // Can throw ArithmeticException at runtime if b == 0
}
```

---

### 5. Core Keywords: The Exception Handling Blocks

Java exception handling is structured around five keywords: **`try`**, **`catch`**, **`finally`**, **`throw`**, and **`throws`**.

```
try {
    // Protected Code Block (Monitored for exceptions)
} catch (SpecificExceptionType ex) {
    // Exception Recovery & Remediation Block
} finally {
    // Guaranteed Cleanup Block (Runs unconditionally)
}
```

#### 5.1 The `try` Block

- Contains code that could trigger an exception.
- **Syntactic Rule:** A `try` block **cannot stand alone**. It must be immediately followed by **at least one `catch` block** OR **one `finally` block** (or use modern `try-with-resources`).

#### 5.2 The `catch` Block

- Executes only if an exception of a matching (or subclass) type is thrown inside the associated `try` block.
- Scopes the exception reference variable (`ex`).
- Multiple `catch` blocks can follow a single `try` block to handle different failure modes.

#### 5.3 The `finally` Block

- Guaranteed to execute **regardless of whether an exception occurs, is caught, or remains unhandled**.
- Runs even if a `return`, `break`, or `continue` statement executes inside the `try` or `catch` block.
- Primarily used to close open operating system resources (file handles, database connections, sockets) before modern `try-with-resources` was introduced.

```
                  try-catch-finally Execution Flow
                                  │
                                  ▼
                         ┌─────────────────┐
                         │ Enter try-block │
                         └────────┬────────┘
                                  │
                  Exception occurs?
                 ┌────────────────┴────────────────┐
            Yes  │                                 │ No
                 ▼                                 ▼
      ┌─────────────────────┐            ┌───────────────────┐
      │ Matches catch block?│            │ Finish try-block  │
      └──────────┬──────────┘            └─────────┬─────────┘
            Yes  │        │ No                     │
                 ▼        └──────────────┐         │
      ┌─────────────────────┐            │         │
      │ Execute catch-block │            │         │
      └──────────┬──────────┘            │         │
                 │                       │         │
                 └──────────────┬────────┘         │
                                │                  │
                                ▼                  ▼
                      ┌───────────────────────────────┐
                      │ Execute finally-block (ALWAYS)│
                      └───────────────┬───────────────┘
                                      │
                         Is exception unhandled?
                        ┌─────────────┴─────────────┐
                   Yes  │                           │ No
                        ▼                           ▼
            Propagate up Call Stack         Resume Next Line
```

---

### 6. The `throw` vs. `throws` Keywords

| Dimension          | `throw` Keyword                                            | `throws` Keyword                                                    |
| :----------------- | :--------------------------------------------------------- | :------------------------------------------------------------------ |
| **Purpose**        | **Explicitly raises/throws** an exception instance         | **Declares** that a method may produce specific exceptions          |
| **Position**       | Used **inside** method/constructor/block bodies            | Appended to the **method signature header**                         |
| **Operand**        | Followed by an **instantiated object** of type `Throwable` | Followed by one or more **Exception Class Names** (comma-separated) |
| **Multiplicity**   | Throws **exactly one** exception instance at a time        | Can declare **multiple** exception classes (`throws A, B`)          |
| **Syntax Example** | `throw new IllegalArgumentException("Invalid");`           | `public void run() throws IOException, SQLException`                |

```java
public class ValidatorService {
    // 'throws' declares the possibility of a Checked Exception to callers
    public void processAge(int age) throws InvalidAgeException {
        if (age < 18) {
            // 'throw' explicitly instantiates and raises the exception
            throw new InvalidAgeException("Age must be >= 18");
        }
        System.out.println("Access granted");
    }
}
```

---

### 7. Exception Propagation & Stack Unwinding

**Exception Propagation** is the runtime mechanism by which an unhandled exception travels backward through the call stack until an appropriate handler is found.

```
Thread Call Stack Progression:
┌────────────────────────────────────────────────────────┐
│ [Stack Frame 3: methodC()]                             │
│ Exception instantiated -> throw new IOException();     │
│ (No catch block found here)                            │
└───────────────────────────┬────────────────────────────┘
                            │ Drops frame / Unwinds stack
                            ▼
┌────────────────────────────────────────────────────────┐
│ [Stack Frame 2: methodB()]                             │
│ Invoked methodC()                                      │
│ (No catch block found here)                            │
└───────────────────────────┬────────────────────────────┘
                            │ Drops frame / Unwinds stack
                            ▼
┌────────────────────────────────────────────────────────┐
│ [Stack Frame 1: methodA()]                             │
│ Invoked methodB()                                      │
│ Contains: catch (IOException e) { ... }                │  <── HANDLER MATCH!
└────────────────────────────────────────────────────────┘      Stack stops unwinding;
                                                                normal execution resumes.
```

#### 7.1 Propagation Invariants:

1. **Unchecked Exceptions:** Propagate through all stack frames automatically without requiring `throws` declarations on intermediate methods.
2. **Checked Exceptions:** Intermediate methods along the propagation chain **must declare the exception using `throws`**, or the code will not compile.
3. **Thread Death:** If an exception propagates past `main()` without being caught, the runtime hands it to the thread's `UncaughtExceptionHandler`, prints the stack trace to `System.err`, and terminates that thread.

---

### 8. Multiple Catch Blocks: The Top-to-Bottom Rule

A single `try` block can be followed by multiple sequential `catch` blocks to handle distinct exception types.

```
                          Subclass Exception Rule
                                  try { ... }
                                       │
                                       ▼
                  ┌──────────────────────────────────────────┐
                  │ catch (FileNotFoundException e) { ... }  │  <── More Specific (Subclass)
                  └────────────────────┬─────────────────────┘
                                       │ (if not matched)
                                       ▼
                  ┌──────────────────────────────────────────┐
                  │ catch (IOException e) { ... }            │  <── Intermediate Class
                  └────────────────────┬─────────────────────┘
                                       │ (if not matched)
                                       ▼
                  ┌──────────────────────────────────────────┐
                  │ catch (Exception e) { ... }              │  <── Most General (Superclass)
                  └──────────────────────────────────────────┘
```

> ⚠️ **The Unreachable Catch Block Trap:**
> Java enforces **top-to-bottom catch order based on inheritance**. You must place subclasses before their parent classes.
>
> If a broader superclass catch block appears _before_ a more specific subclass catch block, the compiler issues an error: **`exception Xxx has already been caught`**.

```java
try {
    FileInputStream fis = new FileInputStream("data.txt");
}
// COMPILATION ERROR: Exception already caught!
catch (Exception e) {
    System.out.println("Generic catch");
}
catch (FileNotFoundException e) { // Child of IOException, which is child of Exception
    System.out.println("File not found");
}
```

---

### 9. Multi-Catch Block (Java 7+)

Java 7 introduced the **Multi-Catch** syntax (`|`) to eliminate duplicate error-handling code when multiple unrelated exceptions are handled with identical logic.

```java
// Java 7+ Multi-Catch Block
try {
    Class<?> clazz = Class.forName("com.cdac.Driver");
    Connection conn = DriverManager.getConnection("url");
} catch (ClassNotFoundException | SQLException e) {
    // 'e' is implicitly final!
    System.err.println("Database initialization failed: " + e.getMessage());
    // e = new SQLException(); // COMPILE ERROR: Multi-catch parameter 'e' cannot be reassigned!
}
```

#### 9.1 Multi-Catch Invariants:

1. **Disjoint Types Rule:** Types within a single multi-catch block **cannot have an inheritance relationship** (one cannot be a subclass of another).
   - `catch (IOException | FileNotFoundException e)` $\rightarrow$ **COMPILE ERROR!** (`FileNotFoundException` is already an `IOException`).
2. **Implicit `final` Parameter:** The exception variable `e` in a multi-catch block is **implicitly `final`**. Any attempt to reassign `e` causes a compile-time error.

---

### 10. How the JVM Implements Exception Handling (The Exception Table)

Modern HotSpot JVMs do not incur performance overhead for `try` blocks when no exception is thrown (the **"Zero-Cost Exception"** model). Instead of injecting runtime check instructions, `javac` compiles `try-catch` structures into an **Exception Table** embedded inside the class file's method metadata.

#### Bytecode Trace via `javap -c -v`:

```java
public void execute() {
    try {
        doWork();
    } catch (IOException e) {
        handleRecovery();
    }
}
```

Disassembled JVM metadata:

```text
Exception table:
   from    to  target type
       0     4     7   Class java/io/IOException
```

- `from`: Starting bytecode offset of the monitored `try` block ($0$).
- `to`: Ending bytecode offset of the `try` block ($4$).
- `target`: Bytecode offset of the `catch` handler code ($7$).
- `type`: Matching class reference in the constant pool (`IOException`).

**Runtime Workflow:** When a method throws an exception, the JVM checks the Program Counter (PC) against the ranges in the Exception Table. If a matching range and type are found, the JVM sets the PC directly to the handler's `target` offset. If no match is found, the JVM pops the current stack frame and repeats the search in the caller's frame.

---

### 11. High-Yield CDAC Exam Traps & Interview Questions

- **Q1: Can a `finally` block be skipped or prevented from running?**
  - _Answer:_ **Yes, in three specific scenarios:**
    1. **JVM Termination:** If `System.exit(0)` is invoked inside the `try` or `catch` block.
    2. **Host Machine Crash:** Hardware power loss, OS process kill (`kill -9`), or a fatal JVM core dump (`VirtualMachineError`).
    3. **Infinite Loop or Deadlock:** If the thread enters a non-terminating loop or permanent deadlock inside the `try` block.

- **Q2: What is the output of the following method?**

  ```java
  public static int testReturn() {
      try {
          return 10;
      } finally {
          return 20;
      }
  }
  ```

  - _Answer:_ `20`
  - _Explanation:_ A `return` statement in a `finally` block **overrides and discards** any pending `return` value or unhandled exception originating from the `try` or `catch` blocks. (Returning values from `finally` is considered an anti-pattern because it silently swallows exceptions).

- **Q3: What happens if an exception is thrown inside a `finally` block?**
  - _Answer:_ If an exception is thrown inside `finally`, it **suppresses and discards** any previous exception that was in the process of being thrown from the `try` block. The original failure context is lost unless preserved using Java 7's suppressed exceptions mechanism.

- **Q4: Can we write a `try` block without any `catch` block?**
  - _Answer:_ **Yes.** A `try` block can be followed immediately by a `finally` block without any `catch` blocks (`try { ... } finally { ... }`). This pattern ensures resources are cleaned up while allowing exceptions to propagate up the call stack. (It is also valid in a `try-with-resources` statement with neither `catch` nor `finally`).

- **Q5: Can an overridden subclass method throw a broader checked exception than the superclass method?**
  - _Answer:_ **No.** A subclass method **cannot declare new or broader checked exceptions** than the overridden superclass method. It can throw the same exceptions, narrower (subclass) exceptions, or omit them entirely. It may, however, throw any unchecked exceptions (`RuntimeException`) without restriction.
