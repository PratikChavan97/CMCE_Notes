# Module 04: Exception Handling, Strings, and Standard APIs

## Unit 4.4: Advanced Exception Handling & Core JVM Exceptions

### 1. User-Defined (Custom) Exceptions

While the Java standard library provides hundreds of built-in exception classes, enterprise applications require custom exceptions to capture domain-specific failure conditions (e.g., `InsufficientFundsException`, `TokenExpiredException`, `InvalidOrderPayloadException`).

```
                          Custom Exception Derivation
                                       │
         ┌─────────────────────────────┴─────────────────────────────┐
         ▼                                                           ▼
Custom Checked Exception                                    Custom Unchecked Exception
extends java.lang.Exception                                 extends java.lang.RuntimeException
- Enforces Catch-or-Declare at compile time                 - Skipped by compiler during compilation
- For recoverable domain errors                             - For unrecoverable programming errors
```

---

### 2. Custom Checked vs. Custom Unchecked Exceptions

#### 2.1 Implementing a Custom Checked Exception

Derive directly from **`java.lang.Exception`**. Callers must either handle it within a `try-catch` block or declare it via `throws`.

```java
package com.cdac.banking.exceptions;

public class InsufficientFundsException extends Exception {
    private final double attemptedAmount;
    private final double currentBalance;

    // Default No-Arg Constructor
    public InsufficientFundsException() {
        super("Transaction declined: Insufficient funds in account.");
        this.attemptedAmount = 0.0;
        this.currentBalance = 0.0;
    }

    // Message-Only Constructor
    public InsufficientFundsException(String message) {
        super(message);
        this.attemptedAmount = 0.0;
        this.currentBalance = 0.0;
    }

    // Cause-Chaining Constructor
    public InsufficientFundsException(String message, Throwable cause) {
        super(message, cause);
        this.attemptedAmount = 0.0;
        this.currentBalance = 0.0;
    }

    // Domain-Enriched Constructor
    public InsufficientFundsException(String message, double attempted, double balance) {
        super(String.format("%s [Attempted: %.2f, Available: %.2f]", message, attempted, balance));
        this.attemptedAmount = attempted;
        this.currentBalance = balance;
    }

    public double getAttemptedAmount() { return attemptedAmount; }
    public double getCurrentBalance() { return currentBalance; }
}
```

#### 2.2 Implementing a Custom Unchecked (Runtime) Exception

Derive directly from **`java.lang.RuntimeException`**. Does not require explicit `throws` signatures.

```java
package com.cdac.auth.exceptions;

public class SessionTimeoutException extends RuntimeException {
    private final String sessionId;

    public SessionTimeoutException(String sessionId) {
        super("User session has expired: ID=" + sessionId);
        this.sessionId = sessionId;
    }

    public SessionTimeoutException(String message, Throwable cause) {
        super(message, cause);
        this.sessionId = "N/A";
    }

    public String getSessionId() { return sessionId; }
}
```

#### Decision Matrix: When to Use Which?

- **Extend `Exception` (Checked):** When the client code can realistically take an alternative recovery path (e.g., prompting the user for an alternative credit card when payment fails).
- **Extend `RuntimeException` (Unchecked):** When the failure is unrecoverable, represents a client coding bug, or in modern microservices/REST APIs where errors bubble up to a global `@ExceptionHandler` filter.

---

### 3. Exception Chaining (The Root-Cause Pattern)

**Exception Chaining** (introduced in Java 1.4) allows an application to catch a low-level technical exception, wrap it inside a high-level business exception, and rethrow it **without losing the original diagnostic stack trace**.

```
Original Low-Level Exception:
[ SQLException: Connection timed out on port 5432 ]
                     │
                     │ Wrapped via: new DataAccessException("Failed query", sqlEx)
                     ▼
High-Level Business Exception:
[ DataAccessException: Order persistence failed ]
   └── Caused by: [ SQLException: Connection timed out on port 5432 ]
```

```java
public class OrderRepository {
    public void persistOrder(Order order) throws DataAccessException {
        try {
            // Low-level database driver call
            DriverManager.getConnection("invalid_url");
        } catch (SQLException rootCause) {
            // EXCEPTION CHAINING: Wrapping rootCause into high-level business exception
            throw new DataAccessException("Database operation failed for Order #" + order.getId(), rootCause);
        }
    }
}
```

#### Low-Level Invariant: `initCause()` vs. Constructor Chaining

`Throwable` provides two ways to attach a root cause:

1. Passing the cause directly into `super(message, cause)`.
2. Calling `initCause(Throwable cause)` after instantiation (can be called **only once**; throwing `IllegalStateException` if the cause was already configured).

---

### 4. Nested `try` Blocks

A `try` block nested inside an outer `try` or `catch` block allows fine-grained error isolation.

```java
public class NestedTryDemo {
    public static void main(String[] args) {
        try { // Outer try-block
            int[] numbers = {10, 20, 0};

            try { // Inner try-block 1
                int result = numbers[0] / numbers[2]; // Triggers ArithmeticException
            } catch (ArithmeticException ae) {
                System.err.println("Handled locally in Inner Catch 1: " + ae.getMessage());
            }

            try { // Inner try-block 2
                System.out.println(numbers[5]); // Triggers ArrayIndexOutOfBoundsException
            } catch (NullPointerException npe) {
                System.err.println("Inner Catch 2: NPE");
            }
            // Inner Catch 2 does NOT match ArrayIndexOutOfBoundsException!
            // Control escapes immediately to Outer Catch!

        } catch (ArrayIndexOutOfBoundsException aie) {
            System.err.println("Handled in Outer Catch: Array index out of range!");
        } finally {
            System.out.println("Outer finally executed.");
        }
    }
}
```

#### Propagation Dynamics in Nested Blocks

1. If an inner `try` throws an exception with no matching inner `catch`, the inner `finally` block runs immediately.
2. The exception then propagates outward to the enclosing outer `try-catch` structures.
3. If an outer `catch` matches, execution stabilizes. If not, the stack frame unwinds to the caller.

---

### 5. `try-with-resources` (ARM - Automatic Resource Management)

Introduced in Java 7 to eliminate error-prone manual `finally` cleanup code. Any resource implementing **`java.lang.AutoCloseable`** or **`java.io.Closeable`** declared within the `try(...)` header is guaranteed to be closed automatically upon block exit.

```java
// Traditional Legacy Pattern (Prone to leaks & boilerplate):
FileReader fr = null;
BufferedReader br = null;
try {
    fr = new FileReader("data.txt");
    br = new BufferedReader(fr);
    System.out.println(br.readLine());
} catch (IOException e) {
    // Handle exception
} finally {
    try {
        if (br != null) br.close();
        if (fr != null) fr.close();
    } catch (IOException ex) { /* ignored or logged */ }
}

// Modern try-with-resources Pattern:
try (FileReader fr = new FileReader("data.txt");
     BufferedReader br = new BufferedReader(fr)) {
    System.out.println(br.readLine());
} catch (IOException e) {
    // Both br and fr are GUARANTEED to be closed before catch executes!
}
```

#### 5.1 The `AutoCloseable` Contract

```java
public interface AutoCloseable {
    void close() throws Exception;
}
```

```
                 try-with-resources Execution Timeline
                                   │
      ┌────────────────────────────┴────────────────────────────┐
      │ 1. Resources instantiated in try(...) header            │
      │ 2. Statements inside try { ... } body execute           │
      │ 3. Resources closed in REVERSE order of declaration     │
      │    (br.close() runs, then fr.close() runs)              │
      │ 4. catch blocks execute (if an exception occurred)      │
      │ 5. finally block executes                               │
      └─────────────────────────────────────────────────────────┘
```

#### 5.2 Reverse Closing Order

Resources are closed in the **reverse order of their declaration** inside the parenthesis header. This guarantees dependencies (e.g., wrapping streams like `BufferedReader`) close safely before their underlying raw descriptors (`FileReader`).

#### 5.3 Suppressed Exceptions

In legacy Java, if both the `try` block and the `finally` block threw an exception, the `finally` exception would silently overwrite the original `try` exception.

`try-with-resources` solves this:

- The exception thrown inside the `try` block is the **primary exception**.
- Any exceptions thrown during `.close()` calls are attached to the primary exception as **Suppressed Exceptions**.

```java
try (CustomResource res = new CustomResource()) {
    throw new RuntimeException("Primary failure in business logic");
} catch (Exception e) {
    System.out.println("Caught: " + e.getMessage()); // Primary failure

    // Inspect suppressed exceptions thrown during auto-closing:
    Throwable[] suppressed = e.getSuppressed();
    for (Throwable t : suppressed) {
        System.out.println("Suppressed during close(): " + t.getMessage());
    }
}
```

---

### 6. The Anatomy of a Stack Trace

A **Stack Trace** is an execution history generated by the JVM showing the exact execution chain from thread initialization up to the failure site.

```
Exception in thread "main" java.lang.NullPointerException: Cannot invoke "String.length()"
    at com.cdac.services.Parser.parsePayload(Parser.java:42)
    at com.cdac.services.OrderService.process(OrderService.java:18)
    at com.cdac.app.Application.main(Application.java:10)
```

```
How to read a Stack Trace:
Line 1: Failing Thread ("main") + Exception Type + Detail Message
Line 2: Exact Point of Failure (Class, Method, Source File, Line Number: 42)
Line 3: Immediate Caller that triggered the call chain
Line 4: Original Entry Point / Bottom of the Call Stack
```

---

### 7. Exception Handling Best Practices for 12+ LPA Technical Interviews

1. **Never Swallow Exceptions (Anti-Pattern):**

   ```java
   // CATASTROPHIC: Silently eating exceptions hides failures
   try { doOperation(); } catch (Exception e) {}

   // CORRECT: Log context or rethrow wrapped in runtime exception
   try { doOperation(); } catch (Exception e) {
       logger.error("Operation failed for context id: {}", ctxId, e);
       throw new ServiceExecutionException("Failed operation", e);
   }
   ```

2. **Never Catch `Throwable` or `Error`:** Catching `Throwable` intercepts fatal JVM errors like `OutOfMemoryError` or `StackOverflowError`, preventing the JVM from shutting down cleanly.
3. **Catch Specific Subclasses First:** Always catch concrete exceptions (`FileNotFoundException`) before general ones (`IOException`).
4. **Preserve the Cause When Wrapping:** Always pass `cause` into constructor calls (`new CustomException("msg", rootCause)`) to maintain the causal trace.
5. **Use Standard Exceptions Where Applicable:** Don't create new custom exception classes if built-ins (`IllegalArgumentException`, `IllegalStateException`, `UnsupportedOperationException`) accurately represent the condition.
6. **Clean Up Resources via `try-with-resources`:** Avoid manual resource closing inside `finally` blocks.

---

### 8. Common Java Exceptions (Profiles, Causes & Prevention)

#### 1. `NullPointerException` (NPE)

- **Type:** `java.lang.NullPointerException` | **Category:** Unchecked (`RuntimeException`)
- **Trigger:** Dereferencing an object reference variable that currently holds `null` (e.g., `obj.method()`, `obj.field`, `arr.length`, throwing `null`).
- **Prevention:**
  - Guard with null checks: `if (obj != null)`.
  - Use Java 8 `Optional<T>`.
  - Leverage `Objects.requireNonNull(val, "Parameter cannot be null")`.
  - Place string literals first in comparisons: `"CONSTANT".equals(dynamicVariable)`.

#### 2. `ArithmeticException`

- **Type:** `java.lang.ArithmeticException` | **Category:** Unchecked (`RuntimeException`)
- **Trigger:** An exceptional arithmetic condition occurs, primarily **integer division or modulo by zero** (`10 / 0` or `10 % 0`).
- _(Note: Floating-point division by zero `10.0 / 0.0` does NOT throw this; it evaluates to `Infinity` or `NaN`)._
- **Prevention:** Verify divisors before operations: `if (divisor != 0)`.

#### 3. `ArrayIndexOutOfBoundsException`

- **Type:** `java.lang.ArrayIndexOutOfBoundsException` | **Category:** Unchecked (`RuntimeException`)
- **Trigger:** Attempting to access an array with an illegal index that is either negative ($< 0$) or greater than or equal to the array's allocated size ($\ge \text{array.length}$).
- **Prevention:** Use enhanced for-loops (`for (int x : arr)`) or bound loops strictly by `0 <= i < arr.length`.

#### 4. `NumberFormatException`

- **Type:** `java.lang.NumberFormatException` | **Category:** Unchecked (`RuntimeException`, child of `IllegalArgumentException`)
- **Trigger:** Attempting to parse a non-numeric string into an integer/float (e.g., `Integer.parseInt("123a")` or `Double.parseDouble("")`).
- **Prevention:** Validate string contents via regular expressions (`str.matches("\\d+")`) before parsing, or handle via structured `try-catch`.

#### 5. `ClassCastException`

- **Type:** `java.lang.ClassCastException` | **Category:** Unchecked (`RuntimeException`)
- **Trigger:** Attempting to downcast an object instance to a class or interface of which it is not an instance (e.g., casting an `Animal` instance to `Dog`).
- **Prevention:** Perform type checks using the **`instanceof`** operator or Java 16+ Pattern Matching (`if (obj instanceof Dog d)`).

#### 6. `FileNotFoundException`

- **Type:** `java.io.FileNotFoundException` | **Category:** **Checked** (`IOException`)
- **Trigger:** Attempting to open a file stream (`FileInputStream`, `FileReader`) when no file exists with that pathname or the file is inaccessible.
- **Prevention:** Check existence prior to stream creation using `new File(path).exists()` or the modern NIO path: `Files.exists(Paths.get(path))`.

#### 7. `IOException`

- **Type:** `java.io.IOException` | **Category:** **Checked** (`java.lang.Exception`)
- **Trigger:** General signals that an I/O operation has failed or been interrupted (broken network pipes, dropped socket streams, hard disk write errors).
- **Prevention:** Implement retry algorithms, graceful degradations, and proper socket/stream lifecycle management.

---

### 9. High-Yield CDAC Exam Traps & Interview Questions

- **Q1: What happens if a resource's `close()` method throws an exception in `try-with-resources` while an exception is already active in the `try` block?**
  - _Answer:_ The exception from the `try` block takes priority and is thrown. The exception from the `close()` method is attached to it as a **suppressed exception** accessible via `e.getSuppressed()`.

- **Q2: Can we declare resources in `try-with-resources` outside the parentheses if they were instantiated earlier?**
  - _Answer:_
    - Prior to **Java 9**: No. Resources had to be declared and instantiated directly inside the `try(...)` parentheses.
    - Starting in **Java 9**: Yes. Any pre-existing variable that is **effectively final** can be passed directly into the header:
      ```java
      BufferedReader br = new BufferedReader(new FileReader("data.txt"));
      try (br) { // Legal in Java 9+
          System.out.println(br.readLine());
      }
      ```

- **Q3: What is the output of this code snippet?**

  ```java
  public class ExceptionOutput {
      public static void main(String[] args) {
          try {
              throw new NullPointerException();
          } catch (Exception e) {
              System.out.print("A");
              throw new ArithmeticException();
          } finally {
              System.out.print("B");
          }
      }
  }
  ```

  - _Answer:_ Prints `AB`, followed by an unhandled `ArithmeticException` stack trace.
  - _Explanation:_ The `catch` block catches `NullPointerException` and prints `"A"`. It then prepares to throw `ArithmeticException`. Before that exception escapes, the `finally` block runs unconditionally, printing `"B"`. Finally, `ArithmeticException` propagates up and terminates the thread.

- **Q4: Can a custom exception class be generic (e.g., `class MyException<T> extends Exception`)?**
  - _Answer:_ **No.** The compiler strictly prohibits generic classes from extending `Throwable` directly or indirectly: _a generic class may not extend java.lang.Throwable_. This is because type erasure prevents the JVM exception table from distinguishing types at runtime (`catch (MyException<String> e)` cannot be differentiated from `catch (MyException<Integer> e)`).

- **Q5: Does `catch (Exception e)` catch fatal JVM `Errors`?**
  - _Answer:_ **No.** `java.lang.Error` does not inherit from `java.lang.Exception`; they are sibling branches under `java.lang.Throwable`. Catching `Exception` will not intercept errors such as `OutOfMemoryError` or `StackOverflowError`.
