# Module 05: Packages, APIs, and Core Library Ecosystem

## Unit 5.3: Serialization and NIO

### 1. Introduction to Serialization

**Serialization** is the mechanism of converting the in-memory state of an active Java object into an architecture-neutral, contiguous stream of bytes.

This byte stream encodes:

- The object's class metadata (type descriptor, class signature).
- The runtime values of all non-static, non-transient instance fields.
- The complete graph of all dependent objects reachable from this instance (**Object Graph Serialization**).

**Deserialization** is the exact reverse process: reconstructing an identical, fully operational clone of the object on the JVM Heap from the incoming byte stream.

```
                            Object Serialization Pipeline
┌───────────────────────────────┐                             ┌───────────────────────────────┐
│ JVM Heap Memory               │                             │ Storage / Network Sink        │
│ ┌───────────────────────────┐ │      ObjectOutputStream     │ ┌───────────────────────────┐ │
│ │ Customer Object           │ │ ──────────────────────────► │ │ Flat File (cust.ser),     │ │
│ │ - id: 101                 │ │     writeObject(cust)       │ │ Socket Pipe, DB BLOB,     │ │
│ │ - address: (0x7F00)       │ │                             │ │ Redis / Kafka Payload     │ │
│ └─────────────┬─────────────┘ │                             │ └───────────────────────────┘ │
│               ▼               │                                             │
│ ┌───────────────────────────┐ │      ObjectInputStream                      │
│ │ Address Object            │ │ ◄───────────────────────────────────────────┘
│ │ - city: "Pune"            │ │      readObject() (Bypasses Constructors!)
│ └───────────────────────────┘ │
└───────────────────────────────┘
```

#### Primary Use Cases:

1. **Network RPC & Distributed Caching:** Passing object graphs across network sockets (e.g., Spring Remoting, legacy RMI, Redis serialization).
2. **Session Persistence:** Persisting stateful HTTP user sessions to disk across application server reboots (e.g., Apache Tomcat session recovery).
3. **Deep Cloning:** Generating complete, deep copies of complex object graphs without manual cloning routines.

---

### 2. The `Serializable` Marker Interface

To make an object eligible for serialization, its class must implement the **`java.io.Serializable`** interface.

```java
package java.io;

public interface Serializable {
    // EMPTY BODY: Zero abstract methods, zero constants!
}
```

- **Marker (Tagging) Interface:** `Serializable` contains no methods or fields. It serves as an informative metadata flag to the JVM execution engine and `ObjectOutputStream`.
- **Runtime Verification:** When `ObjectOutputStream.writeObject(obj)` is called, the JVM performs an `instanceof Serializable` check. If the object does not implement `Serializable`, the JVM throws an immediate **`java.io.NotSerializableException`**.

---

### 3. Object Streams: `ObjectOutputStream` & `ObjectInputStream`

#### 3.1 ObjectOutputStream (`writeObject()`)

A processing stream filter that serializes primitive types and full Java object graphs to an underlying `OutputStream`.

```java
import java.io.FileOutputStream;
import java.io.ObjectOutputStream;
import java.io.Serializable;

class Employee implements Serializable {
    private static final long serialVersionUID = 1L;

    private int empId;
    private String name;
    private transient double temporarySessionToken; // Omitted from serialization

    public Employee(int empId, String name, double token) {
        this.empId = empId;
        this.name = name;
        this.temporarySessionToken = token;
    }
}

public class SerializationWriter {
    public static void main(String[] args) {
        Employee emp = new Employee(101, "Aarav", 98765.43);

        try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("employee.ser"))) {
            oos.writeObject(emp); // Serializes entire object graph to disk
            System.out.println("Serialization complete: employee.ser generated");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

#### 3.2 ObjectInputStream (`readObject()`)

Reconstitutes objects from an incoming byte stream.

```java
import java.io.FileInputStream;
import java.io.ObjectInputStream;

public class SerializationReader {
    public static void main(String[] args) {
        try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream("employee.ser"))) {
            // readObject() returns java.lang.Object; explicit downcast required!
            Employee emp = (Employee) ois.readObject();
            System.out.println("Employee loaded successfully.");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

> ⚠️ **THE DESERIALIZATION CONSTRUCTOR TRAP (Top Interview Question):**
> _Does deserialization execute the class constructor?_
> **NO.** Deserialization **bypasses all constructors of the serializable class completely!**
> The JVM uses native reflection mechanisms (specifically `sun.reflect.ReflectionFactory`) to allocate raw memory on the Heap and directly writes the serialized field states into that memory.
>
> **Exception (Non-Serializable Superclasses):**
> If a class extends a **non-serializable superclass**, the JVM **MUST run the no-argument constructor of that non-serializable superclass** to initialize inherited state! If that superclass does not have an accessible no-arg constructor, deserialization throws **`java.io.InvalidClassException`**.

---

### 4. The `transient` Keyword & `static` Field Invariants

#### 4.1 The `transient` Modifier

Applied to instance fields to **exclude them from the serialization process**.

```java
public class UserAccount implements Serializable {
    private String username;
    private transient String passwordHash; // Will NOT be written to stream!
    private transient int pinCode;          // Will NOT be written to stream!
}
```

- **Deserialization State:** When an object is deserialized, transient fields are reset to their **default primitive/reference zero-values** (`0`, `0.0`, `false`, or `null`), regardless of what values they held when serialized.
- **Primary Use Cases:**
  1. **Security:** Excluding sensitive credentials (passwords, encryption keys, PINs).
  2. **Non-Serializable System Resources:** File descriptors, database connections, socket handles, thread instances.
  3. **Derived/Redundant Data:** Fields that can be easily recomputed at runtime (e.g., caching a calculated `taxTotal`).

#### 4.2 Behavior of `static` Fields

- **Static variables are NEVER serialized.**
- _Why?_ Serialization captures **object state** (instance fields on the Heap). `static` variables belong to **class metadata** (Metaspace / Class mirror).
- During deserialization, reading a `static` field simply yields whatever value that static field currently holds in the active JVM's Metaspace.

---

### 5. Deep-Dive: `serialVersionUID`

`serialVersionUID` is a unique 64-bit signed integer (`long`) that serves as a **version stamp** for a `Serializable` class.

```java
private static final long serialVersionUID = 1L; // Explicit declaration
```

```
                          Version Verification Workflow
                                       │
     During Serialization:                          During Deserialization:
Object is tagged with current UID.             Incoming byte stream UID read.
     serialVersionUID = 1L                          Stream UID: 1L
               │                                           │
               ▼                                           ▼
       Writen to Stream                        JVM compares Stream UID with
                                               Local Class compiled UID
                                                           │
                                        ┌──────────────────┴──────────────────┐
                                        │ Matches                             │ Mismatch
                                        ▼                                     ▼
                               Object Deserialized                  InvalidClassException!
```

#### 5.1 What Happens if You Omit `serialVersionUID`?

If a class does not explicitly declare `serialVersionUID`, the compiler calculates a default hash at compile time based on class details (field names, types, method signatures, access modifiers).

**The Risk:** Any minor change to the class (adding an unused private helper method, renaming an internal field) causes the compiler to generate a **different default hash**.
Attempting to deserialize an older serialized file with this modified class triggers an immediate runtime crash:

```
java.io.InvalidClassException: Employee;
local class incompatible: stream classdesc serialVersionUID = -4589218491823,
local class serialVersionUID = 892371948192384
```

**Best Practice:** Always declare `private static final long serialVersionUID` explicitly in any production `Serializable` class.

---

### 6. Object Cloning: Shallow Copy vs. Deep Copy

Cloning creates an exact copy of an existing object. The fundamental difference lies in how referenced objects are duplicated.

```
Shallow Copy:                                  Deep Copy:
┌───────────────┐                              ┌───────────────┐
│ Original Obj  │                              │ Original Obj  │
│ - data = 10   │                              │ - data = 10   │
│ - ref ────────┼──────┐                       │ - ref ────────┼──────► [ Heap Target A ]
└───────────────┘      │                       └───────────────┘
                       ▼
┌───────────────┐   [ Heap Target A ]          ┌───────────────┐
│ Cloned Obj    │   (SHARED!)                  │ Cloned Obj    │
│ - data = 10   │                              │ - data = 10   │
│ - ref ────────┼──────┘                       │ - ref ────────┼──────► [ Heap Target B ]
└───────────────┘                              └───────────────┘        (INDEPENDENT CLONE!)
```

| Dimension             | Shallow Copy                                                                            | Deep Copy                                                                                 |
| :-------------------- | :-------------------------------------------------------------------------------------- | :---------------------------------------------------------------------------------------- |
| **Primitive Fields**  | Copied by value (completely independent)                                                | Copied by value (completely independent)                                                  |
| **Object References** | **Copies reference addresses only**. Both point to the **same underlying heap object**. | **Recursively copies referenced objects**, creating new independent heap instances.       |
| **Mutation Hazard**   | Modifying an inner referenced object impacts **both** copies!                           | Completely isolated; modifying the clone never affects the original.                      |
| **Implementation**    | Default behavior of `Object.clone()`                                                    | Custom copy constructors, manual recursive cloning, or **Serialization/Deserialization**. |

#### Deep Copy via Serialization:

A popular way to produce a 100% deep copy of a complex object graph without writing manual cloning routines:

```java
import java.io.*;

public class DeepCloneUtility {
    @SuppressWarnings("unchecked")
    public static <T extends Serializable> T deepCopy(T object) throws Exception {
        // Serialize to RAM buffer:
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        try (ObjectOutputStream oos = new ObjectOutputStream(baos)) {
            oos.writeObject(object);
        }

        // Deserialize from RAM buffer:
        ByteArrayInputStream bais = new ByteArrayInputStream(baos.toByteArray());
        try (ObjectInputStream ois = new ObjectInputStream(bais)) {
            return (T) ois.readObject(); // Returns a completely detached deep clone!
        }
    }
}
```

---

### 7. Introduction to Java NIO (New I/O - `java.nio`)

Introduced in Java 1.4 and enhanced in Java 7 (**NIO.2** / JSR 203), `java.nio` addresses the performance bottlenecks of the classic `java.io` stream model.

```
                      Classic I/O vs. Java NIO Architecture
                                        │
         ┌──────────────────────────────┴──────────────────────────────┐
         ▼                                                             ▼
   Classic I/O (java.io)                                     New I/O (java.nio)
   • Stream-Oriented (Byte-by-byte / char)                   • Buffer-Oriented (Blocks of data)
   • Blocking Execution (Synchronous wait)                   • Non-Blocking Execution (Selectable)
   • Unidirectional Streams (In or Out)                      • Bidirectional Channels (Read & Write)
   • High thread memory overhead                             • Scalable Selector-driven multiplexing
```

---

### 8. Modern NIO.2: The `Path` and `Files` APIs (Java 7+)

Java 7 introduced `java.nio.file.Path` and `java.nio.file.Files`, which replace the legacy, limited `java.io.File` class.

#### 8.1 The `Path` Interface

An immutable, hierarchical representation of an absolute or relative filesystem path.

```java
import java.nio.file.Path;
import java.nio.file.Paths;

// Factory instantiation:
Path path1 = Paths.get("/var/log/application.log");
Path path2 = Path.of("configs", "database.properties"); // Java 11+ convenient factory

// Path manipulation:
Path normalized = path1.normalize(); // Resolves redundancies like '.' and '..'
Path resolved   = path1.resolve("subfolder"); // Combines paths
```

#### 8.2 The `Files` Utility Class

A collection of static methods for file and directory manipulation.

```java
import java.nio.file.*;
import java.util.List;

public class FilesApiDemo {
    public static void main(String[] args) throws Exception {
        Path source = Path.of("data.txt");
        Path target = Path.of("backup.txt");

        // 1. Reading all content directly:
        List<String> lines = Files.readAllLines(source);
        String fullContent = Files.readString(source); // Java 11+

        // 2. High-performance atomic writing:
        Files.writeString(target, "New Content", StandardOpenOption.CREATE, StandardOpenOption.APPEND);

        // 3. File metadata inspection:
        boolean exists   = Files.exists(source);
        boolean readable = Files.isReadable(source);
        long byteSize    = Files.size(source);

        // 4. Atomic File Operations:
        Files.copy(source, target, StandardCopyOption.REPLACE_EXISTING);
        Files.deleteIfExists(target);
    }
}
```

---

### 9. NIO Core Architecture: Buffers & Channels

Data in NIO is always transferred between **Buffers** and **Channels**.
You read data from a Channel into a Buffer, and you write data from a Buffer into a Channel.

```
                        NIO Data Exchange Loop
                      ┌────────────────────────┐
                      │ Physical Storage / NIC │
                      └───────────┬────────────┘
                                  │
                                  ▼
                      ┌────────────────────────┐
                      │      FileChannel       │
                      └───────────▲────────────┘
                Channel.read(buf) │  │ Channel.write(buf)
                                  │  │
                                  ▼  │
                      ┌────────────────────────┐
                      │      ByteBuffer        │
                      │  [ In-Memory Block ]   │
                      └────────────────────────┘
```

#### 9.1 Buffers (`java.nio.Buffer`)

A **Buffer** is a fixed-capacity, linear memory container for primitive data types (`ByteBuffer`, `CharBuffer`, `IntBuffer`, `DoubleBuffer`).

#### The 4 Core Pointers of a Buffer:

Every buffer maintains four internal markers that dictate its state:

$$0 \le \text{mark} \le \text{position} \le \text{limit} \le \text{capacity}$$

```
Initial State (Writing mode - Buffer Capacity = 8):
  position = 0, limit = 8, capacity = 8
 ┌───┬───┬───┬───┬───┬───┬───┬───┐
 │   │   │   │   │   │   │   │   │
 └───┴───┴───┴───┴───┴───┴───┴───┘
   ▲                               ▲
   │ position                      │ limit & capacity

After Writing 4 Bytes ("JAVA"):
  position = 4, limit = 8, capacity = 8
 ┌───┬───┬───┬───┬───┬───┬───┬───┐
 │ J │ A │ V │ A │   │   │   │   │
 └───┴───┴───┴───┴───┴───┴───┴───┘
                   ▲               ▲
                   │ position      │ limit & capacity

After flip() (Switched to Reading mode!):
  position = 0, limit = 4, capacity = 8
 ┌───┬───┬───┬───┬───┬───┬───┬───┐
 │ J │ A │ V │ A │   │   │   │   │
 └───┴───┴───┴───┴───┴───┴───┴───┘
   ▲               ▲               ▲
   │ position      │ limit         │ capacity
```

1. **`capacity`:** Total number of elements the buffer can hold. Set at allocation; never changes.
2. **`position`:** Index of the next element to be read or written.
3. **`limit`:** Index boundary beyond which reading or writing is prohibited.
4. **`mark`:** A saved index bookmark. Calling `mark()` sets `mark = position`. Calling `reset()` restores `position = mark`.

#### Essential Buffer Methods:

- `flip()`: **Switches buffer from Writing Mode to Reading Mode.** Sets $\text{limit} = \text{position}$, and then $\text{position} = 0$.
- `clear()`: **Switches back to Writing Mode.** Sets $\text{position} = 0$ and $\text{limit} = \text{capacity}$ (does not erase underlying bytes; simply resets markers).
- `rewind()`: Resets $\text{position} = 0$, leaving the limit unchanged so data can be re-read.
- `compact()`: Copies unread bytes (between `position` and `limit`) down to the start of the buffer, allowing further writes without losing pending data.

---

### 10. `FileChannel` & Fast File Copy Operations

A `FileChannel` provides a high-throughput, direct connection to an open disk file for reading, writing, and memory mapping.

```java
import java.io.RandomAccessFile;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;

public class ChannelReadDemo {
    public static void main(String[] args) throws Exception {
        try (RandomAccessFile file = new RandomAccessFile("input.txt", "rw");
             FileChannel channel = file.getChannel()) {

            // Allocate a 48-byte buffer in RAM:
            ByteBuffer buffer = ByteBuffer.allocate(48);

            int bytesRead = channel.read(buffer); // Read from Channel INTO Buffer
            while (bytesRead != -1) {
                buffer.flip(); // Prepare buffer for reading

                while (buffer.hasRemaining()) {
                    System.out.print((char) buffer.get()); // Read byte-by-byte
                }

                buffer.clear(); // Reset pointers to accept new writes from channel
                bytesRead = channel.read(buffer);
            }
        }
    }
}
```

#### Zero-Copy Channel Transfers:

`FileChannel` supports direct, OS-level zero-copy data streaming via DMA (Direct Memory Access), bypassing JVM application memory entirely:

```java
try (FileChannel src = FileChannel.open(Path.of("large_video.mp4"), StandardOpenOption.READ);
     FileChannel dest = FileChannel.open(Path.of("copy.mp4"), StandardOpenOption.CREATE, StandardOpenOption.WRITE)) {

    // Direct kernel-space transfer: fast, zero-copy, minimal CPU load!
    src.transferTo(0, src.size(), dest);
}
```

---

### 11. High-Yield CDAC Exam Traps & Interview Questions

- **Q1: What happens if a class implements `Serializable`, but one of its instance fields is an object that does NOT implement `Serializable`?**
  - _Answer:_ **`java.io.NotSerializableException`** is thrown at runtime during the `writeObject()` call.
  - _Fix:_ Either mark the offending field **`transient`**, or ensure that the referenced object's class implements `Serializable`.

- **Q2: Does deserialization execute a class's Instance Initializer Blocks (IIB) or field initializers?**
  - _Answer:_ **No.** Deserialization does not run constructors, IIBs, or field initializers of the serialized class. All instance values are restored directly from the byte stream.

- **Q3: What does `buffer.flip()` do, and what happens if you forget to call it before reading?**
  - _Answer:_ `flip()` transitions the buffer from **writing mode to reading mode** by setting `limit = position` and `position = 0`. If omitted, `position` remains at the end of the written data. Attempting to read from the buffer will either read blank/stale data or throw a `BufferUnderflowException`.

- **Q4: Can we serialize a singleton class safely?**
  - _Answer:_ **Not without protection.** Standard deserialization creates a **brand-new object instance**, breaking the singleton invariant!
  - _Protection Pattern:_ Implement the special hook method:
    ```java
    protected Object readResolve() {
        return INSTANCE; // Return the canonical singleton instance!
    }
    ```

- **Q5: What is the difference between `ByteBuffer.allocate(1024)` and `ByteBuffer.allocateDirect(1024)`?**
  - _Answer:_
    - `allocate(1024)`: Allocates an on-heap buffer inside normal JVM heap memory. Managed by GC; involves an extra memory copy when interacting with OS native I/O.
    - `allocateDirect(1024)`: Allocates an **off-heap buffer in native OS memory** using C-level `malloc`. It bypasses JVM heap memory and GC management, providing high-throughput zero-copy I/O for network and disk channels.
