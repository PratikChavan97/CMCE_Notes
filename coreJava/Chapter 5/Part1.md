# Module 05: Packages, APIs, and Core Library Ecosystem

## Unit 5.1: Packages and Java API

### 1. Introduction to Packages

A **package** in Java is a namespace mechanism that encapsulates a group of related classes, interfaces, enumerations, records, and sub-packages. Packages serve as the fundamental unit of modularization and access boundary enforcement in the Java runtime.

```
                           Core Functions of Packages
                                       │
         ┌─────────────────────────────┼─────────────────────────────┐
         ▼                             ▼                             ▼
Namespace Conflict Resolution    Access Protection Boundary     Code Organization & Search
Eliminates naming collisions     Enforces package-private       Maps classes logically into
between disparate modules        and protected access levels    hierarchical domain components
(e.g., date models)              across application layers      on filesystem storage
```

#### The Namespace Collision Problem

Without packages, class names must be globally unique across every library, vendor dependency, and enterprise code module:

```java
// Two distinct domain concepts requiring identical class identifiers:
java.util.Date        // Represents a specific moment in time (legacy epoch time)
java.sql.Date         // Represents an SQL DATE (year, month, day without time)
```

By prefixing types with unique package qualifiers, the compiler and runtime eliminate naming ambiguity entirely.

---

### 2. Built-in Packages vs. User-Defined Packages

```
                              Java Package Taxonomy
                                        │
         ┌──────────────────────────────┴──────────────────────────────┐
         ▼                                                             ▼
Built-in (Standard Platform) Packages                       User-Defined Packages
Bundled directly within Java runtime modules                Defined by application developers
(e.g., java.lang, java.util, java.io)                       (e.g., com.cdac.acts.service)
```

#### 2.1 Built-in Platform Packages

Shipped with the JDK. Starting with **Java 9 (Project Jigsaw)**, built-in packages are grouped inside runtime **Modules** (e.g., `java.base`, `java.sql`, `java.desktop`, `java.net.http`).

#### 2.2 User-Defined Packages

Created by software engineers to group domain-specific classes into distinct functional components (e.g., controllers, services, repositories, entities).

---

### 3. Package Declaration & Creation Mechanics

A package is declared using the **`package`** keyword.

```java
package com.cdac.banking.service; // MUST be the first non-comment statement in file

public class TransferService {
    // Implementation
}
```

#### 3.1 Structural Rules for the `package` Keyword

1. **Position Rule:** If present, the `package` statement **must be the very first statement** in the Java source file (preceded only by comments or whitespace).
2. **Cardinality:** There can be **at most one** `package` declaration per source file.
3. **Absence:** If no package statement is declared, the types inside the file belong to the **default package** (unnamed package).

> ⚠️ **The Default Package Hazard (CDAC Exam Critical):**
> Classes residing in a named package **cannot import or access classes residing in the default (unnamed) package**, even if they share the same directory! Placing production classes in the default package is a severe anti-pattern.

#### 3.2 Compilation Mechanics: The `-d` (Directory) Flag

The package declaration establishes a contract with the host operating system: **the directory path must match the package name verbatim**.

```bash
# Compilation without -d (leaves .class file in current folder):
javac TransferService.java

# Compilation WITH -d (Instructs javac to synthesize the directory tree automatically):
javac -d . TransferService.java
# Result: Creates path: ./com/cdac/banking/service/TransferService.class

# Compilation targeting a dedicated build/classes output folder:
javac -d ./bin src/com/cdac/banking/service/TransferService.java
```

#### 3.3 Execution Mechanics: Fully Qualified Class Names (FQCN)

Once compiled into a package, a class cannot be executed by its simple identifier alone. You must provide its **Fully Qualified Class Name (FQCN)** from the root of the classpath:

```bash
# INCORRECT (Triggers: Error: Could not find or load main class TransferService):
cd com/cdac/banking/service
java TransferService

# CORRECT (Run from the classpath root using the FQCN):
cd /project/bin
java com.cdac.banking.service.TransferService
```

---

### 4. Package Naming Conventions

The Java Language Specification (JLS §6.1) specifies standard conventions to guarantee global uniqueness:

```
                            Reverse Domain Name Formula
┌─────────────────────────────────┬─────────────────┬─────────────────────────┐
│ Top-Level Domain (TLD)          │ Company / Org   │ Project / Layer Name    │
│ com, org, in, edu, io, net      │ cdac, oracle    │ acts.banking.controller │
└─────────────────────────────────┴─────────────────┴─────────────────────────┘
```

#### Lexical Rules & Conventions:

1. **Strictly Lowercase:** Package names are written entirely in lowercase characters to prevent filename collision issues on case-insensitive filesystems (e.g., Windows vs. Linux).
2. **Invalid Characters / Keyword Handling:** If a domain segment contains a reserved Java keyword or hyphen, prefix it with an underscore:
   - Domain: `switch.com` $\longrightarrow$ Package: `com._switch.core`
   - Domain: `my-app.org` $\longrightarrow$ Package: `org.my_app.service`

---

### 5. Import Statements

The `import` statement allows classes declared in other packages to be referenced by their simple names rather than their verbose fully qualified class names.

```
+-------------------------------------------------------------------------+
| Import Statement Placement In Compilation Unit                          |
|                                                                         |
| 1. package declaration;      (e.g., package com.app;)                   |
| 2. import statements;        (e.g., import java.util.List;)             |
| 3. class / interface body;   (e.g., public class Main { ... })          |
+-------------------------------------------------------------------------+
```

#### 5.1 Explicit vs. Wildcard Imports

```java
// 1. Explicit Class Import (Best Practice):
import java.util.ArrayList; // Imports ONLY ArrayList

// 2. Wildcard Package Import:
import java.util.*;         // Imports ALL public types directly inside java.util
```

#### 5.2 Wildcard Import Mechanics & Performance Myths

- **Myth:** _"Wildcard imports (`import java.util._`) inflate `.class` bytecode size and slow down runtime execution."\*
- **Reality:** **False.** Imports are strictly compile-time syntactic sugar. `javac` resolves names to their fully qualified identifiers during symbol resolution. A compiled `.class` file contains zero references to wildcard imports.
- **The Real Penalty:** Wildcard imports introduce **potential compile-time namespace collisions** and slightly increase compilation time during the parse phase.

```java
import java.util.*;
import java.sql.*;

public class ConflictDemo {
    // COMPILE ERROR: reference to Date is ambiguous!
    // Both java.util.Date and java.sql.Date match!
    Date today = new Date();

    // MANDATORY RESOLUTION: Use Fully Qualified Class Name (FQCN)
    java.util.Date utilDate = new java.util.Date();
    java.sql.Date sqlDate   = new java.sql.Date(System.currentTimeMillis());
}
```

> ⚠️ **The Sub-Package Import Fallacy:**
> Package hierarchies are strictly flat in the eyes of the JVM. **Sub-packages are completely distinct packages.**
> Importing `java.awt.*` **DOES NOT** import `java.awt.event.*`!

---

### 6. Static Imports (Java 5+)

Standard `import` statements bring **types (classes, interfaces)** into scope. **Static imports** bring **static members (fields and methods)** directly into lexical scope, eliminating the need to qualify them with class names.

```java
// Standard Import
import java.lang.Math;

// Static Import of individual members
import static java.lang.Math.PI;
import static java.lang.Math.sqrt;

// Static Wildcard Import
import static java.lang.Math.*;

public class StaticImportDemo {
    public static void main(String[] args) {
        // Without static import:
        double r1 = Math.PI * Math.sqrt(25.0);

        // With static import:
        double r2 = PI * sqrt(25.0); // Direct access!
    }
}
```

#### Static Import Collision Trap

If two classes provide identically named static members, a static wildcard import creates a compile-time ambiguity error:

```java
import static java.lang.Integer.MAX_VALUE;
// import static java.lang.Long.MAX_VALUE; // CONFLICT: MAX_VALUE is ambiguous!
```

---

### 7. Accessing Classes Across Packages: Classpath (`-cp`)

When code references an external class, the JVM locates the corresponding `.class` file using the **Classpath**.

```
                           Classpath Lookup Order
                                     │
         ┌───────────────────────────┴───────────────────────────┐
         ▼                                                       ▼
Standard Platform Modules                               User Application Classpath
(Loaded via Bootstrap / Platform ClassLoaders)          (Configured via -cp, -classpath, or CLASSPATH)
Stored in runtime jimage ($JAVA_HOME/lib/modules)       Directories, JAR files, ZIP archives
```

#### Setting the Classpath:

```bash
# 1. Specifying multiple search directories / JARs:
java -cp ".:./libs/mysql-connector.jar:./libs/log4j.jar" com.cdac.app.Application

# Note: On Windows, use a semicolon (;) as path separator:
# java -cp ".;.\libs\mysql-connector.jar" com.cdac.app.Application
```

---

### 8. Java API Documentation & Javadoc Specification

The **Javadoc** utility parses Java source code and doc-comments (`/** ... */`) to generate standard HTML documentation.

```java
/**
 * Processes financial transactions securely across accounts.
 *
 * @author CDAC Team
 * @version 2.1
 * @since 1.8
 */
public class PaymentGateway {

    /**
     * Deducts funds from the source account.
     *
     * @param accountId Unique identifier of the target account
     * @param amount    Monetary value to withdraw (must be positive)
     * @return {@code true} if settlement succeeds; {@code false} otherwise
     * @throws IllegalArgumentException if amount is non-positive
     * @see com.cdac.banking.Account
     */
    public boolean debit(String accountId, double amount) throws IllegalArgumentException {
        if (amount <= 0) {
            throw new IllegalArgumentException("Amount must be positive");
        }
        return true;
    }
}
```

#### Standard Javadoc Tags:

- `@author`: Specifies author metadata.
- `@version`: Specifies current build/release version.
- `@param`: Documents method parameter types and expectations.
- `@return`: Documents the return value and meaning.
- `@throws` / `@exception`: Documents checked or runtime exceptions thrown.
- `@see`: Provides cross-references to other classes or methods.
- `@since`: Documents the JDK or library release version in which the feature debuted.
- `{@link ClassName#method}`: Creates an inline clickable hyperlink to another member.

```bash
# Generating HTML documentation:
javadoc -d ./doc src/com/cdac/banking/*.java
```

---

### 9. Core Standard Library Packages (The Java API)

```
                            Standard Java API
                                    │
    ┌───────────────────┬───────────┴───────────┬───────────────────┐
    ▼                   ▼                       ▼                   ▼
java.lang           java.util               java.io             java.nio
Fundamental runtime Collections, Date,      Stream-based        Buffer & Channel
types & lifecycle   Concurrency, Utilities  blocking I/O        non-blocking I/O
```

---

### 10. `java.lang`: The Language Foundation Package

`java.lang` contains classes central to the design of the Java programming language.

> ⚠️ **The Automatic Import Invariant:**
> The package **`java.lang` is imported automatically** into every single Java compilation unit. You never need to write `import java.lang.*;`.

#### Core Components of `java.lang`:

1. **Root Type Hierarchy:** `Object`, `Class`, `Throwable`, `Cloneable`.
2. **Strings & Buffers:** `String`, `StringBuilder`, `StringBuffer`, `CharSequence`.
3. **Primitive Wrappers:** `Byte`, `Short`, `Integer`, `Long`, `Float`, `Double`, `Character`, `Boolean`, `Void`.
4. **Runtime & System Execution:**
   - `System`: Provides access to standard I/O streams (`in`, `out`, `err`), system properties (`System.getProperty()`), garbage collection hints (`System.gc()`), and high-resolution timers (`System.currentTimeMillis()`, `System.nanoTime()`).
   - `Runtime`: Interacts with the JVM environment; manages memory metrics (`totalMemory()`, `freeMemory()`, `maxMemory()`) and host process spawning (`exec()`).
5. **Math:** `Math` (standard math algorithms), `StrictMath` (bit-for-bit identical floating-point specifications across CPUs).
6. **Threading & Concurrency:** `Thread`, `Runnable`, `ThreadLocal`, `ThreadGroup`.
7. **Reflective Metadata:** `Package`, `Module`, `Record`, `Enum`.

---

### 11. `java.util`: The Utility & Collections Framework Package

`java.util` contains the **Java Collections Framework (JCF)**, legacy date-time utilities, random number generators, string tokenizers, and concurrent structures.

#### Core Components of `java.util`:

1. **Collections Framework Interfaces:** `Collection`, `List`, `Set`, `Queue`, `Deque`, `Map`, `SortedSet`, `SortedMap`.
2. **Standard Collection Implementations:**
   - Lists: `ArrayList`, `LinkedList`, `Vector` (legacy synchronized), `Stack`.
   - Sets: `HashSet`, `LinkedHashSet`, `TreeSet`.
   - Maps: `HashMap`, `LinkedHashMap`, `TreeMap`, `Hashtable` (legacy synchronized).
   - Queues/Deques: `PriorityQueue`, `ArrayDeque`.
3. **Algorithms & Helpers:** `Collections` (static sorting, searching, synchronization wrappers), `Arrays` (array sorting, search, deep comparisons).
4. **Functional & Modern Constructs:** `Optional<T>`, `Objects`, `UUID`, `Base64`.
5. **I/O Parsing & Scanning:** `Scanner`, `StringTokenizer`, `Properties`.
6. **Concurrency Sub-packages:** `java.util.concurrent`, `java.util.concurrent.atomic`, `java.util.concurrent.locks`.

---

### 12. `java.io` vs. `java.nio`: The I/O Subsystem

Java provides two fundamentally distinct I/O models:

```
Stream-based I/O (java.io - Java 1.0)          Buffer/Channel I/O (java.nio - Java 1.4 & 7)
┌──────────────┐       ┌──────────────┐       ┌──────────────┐   Channel   ┌──────────────┐
│ Memory Space │ ────► │ Byte Stream  │       │ Memory Space │ ◄─────────► │ Buffer Area  │
└──────────────┘       └──────────────┘       └──────────────┘ (Bidirection)└──────────────┘
• One-way (Byte/Char at a time)               • Block-oriented (Fast bulk byte buffers)
• Synchronous & Blocking                      • Non-blocking & Scalable (Selectors)
```

| Dimension                 | `java.io` (Classic I/O)                                          | `java.nio` (New I/O)                                                      |
| :------------------------ | :--------------------------------------------------------------- | :------------------------------------------------------------------------ |
| **Architectural Model**   | Stream-Oriented (`InputStream`, `OutputStream`)                  | Buffer- and Channel-Oriented (`Buffer`, `Channel`)                        |
| **Execution Paradigm**    | **Blocking I/O** (Thread halts until data arrives)               | **Non-Blocking I/O (NIO)** (Threads query status via Selectors)           |
| **Directionality**        | Unidirectional (Separate input and output streams)               | **Bidirectional** (Channels read and write to Buffers)                    |
| **Throughput / Scale**    | High thread-per-connection overhead; poor high-scale performance | Low thread footprint; handles tens of thousands of concurrent connections |
| **Filesystem Operations** | `java.io.File` (Limited metadata, poor error handling)           | `java.nio.file.Path`, `Files`, `Paths` (Modern, robust file operations)   |

#### 12.1 `java.io` Core Architecture

- **Byte Streams (8-bit raw bytes):** `InputStream`, `OutputStream` (subclasses: `FileInputStream`, `FileOutputStream`, `BufferedInputStream`, `ObjectInputStream`).
- **Character Streams (16-bit Unicode characters):** `Reader`, `Writer` (subclasses: `FileReader`, `FileWriter`, `BufferedReader`, `PrintWriter`).
- **Serialization:** `Serializable`, `Externalizable`.

#### 12.2 `java.nio` Core Architecture

- **Buffers:** Containers for contiguous raw primitive data (`ByteBuffer`, `CharBuffer`, `IntBuffer`).
- **Channels:** Open connections to hardware devices, files, or network sockets capable of non-blocking I/O operations (`FileChannel`, `SocketChannel`, `ServerSocketChannel`).
- **Selectors:** Multiplexers that allow a single thread to monitor multiple channels for incoming I/O events (key to scalable servers like Netty).
- **NIO.2 Filesystem API (Java 7):**
  - `java.nio.file.Path`: Object-oriented representation of a filesystem path (replaces `java.io.File`).
  - `java.nio.file.Files`: Static utility class providing atomic moves, file walks, streams, and permission inspections.

```java
import java.nio.file.Path;
import java.nio.file.Paths;
import java.nio.file.Files;
import java.io.IOException;
import java.util.List;

public class NioDemo {
    public static void main(String[] args) throws IOException {
        Path path = Paths.get("data.txt");

        // Modern concise file reading:
        List<String> lines = Files.readAllLines(path);
        lines.forEach(System.out::println);
    }
}
```

---

### 13. High-Yield CDAC Exam Traps & Interview Questions

- **Q1: Does `package com.cdac.acts;` automatically make classes inside `com.cdac` visible?**
  - _Answer:_ **No.** Java treats each package as an independent namespace. There is no parent-child inheritance relationship between packages. Classes in `com.cdac.acts` must explicitly import classes from `com.cdac`, and package-private members in `com.cdac` are inaccessible to classes in `com.cdac.acts`.

- **Q2: Why does `java.lang` not require an import statement?**
  - _Answer:_ The compiler automatically prepends `import java.lang.*;` to every compilation unit during the lexical phase. This makes core types (`Object`, `String`, `System`, `Integer`, `Exception`) universally available without boilerplate imports.

- **Q3: What happens if two imported packages declare a class with the same name, but that class is never used in the source file?**
  - _Answer:_ **The code compiles cleanly without errors.** The compiler performs conflict resolution on demand. An ambiguity error is raised **only if the colliding class name is actually referenced** in code without qualification.

- **Q4: Can a class belong to multiple packages simultaneously?**
  - _Answer:_ **No.** A class file can declare at most one `package` statement at the top of its compilation unit, meaning it belongs exclusively to a single namespace and directory structure.

- **Q5: What is the difference between `java.util.Date` and `java.sql.Date`?**
  - _Answer:_
    - `java.util.Date`: Represents an exact timestamp (both date and time down to milliseconds) based on the Unix epoch.
    - `java.sql.Date`: A subclass of `java.util.Date` tailored specifically for SQL `DATE` fields; it strips out hour, minute, and second components to store only year, month, and day ($00:00:00.000$ UTC).
