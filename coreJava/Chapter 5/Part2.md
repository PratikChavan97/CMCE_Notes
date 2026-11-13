# Module 05: Packages, APIs, and Core Library Ecosystem

## Unit 5.2: Java I/O and File Handling

### 1. Introduction to Java I/O

The **Java Input/Output (I/O)** subsystem (`java.io`) provides an abstraction layer to transfer sequential streams of data between a Java application and external targets—such as secondary storage files, hardware sockets, RAM arrays, or network endpoints.

```
                          The Stream Abstraction Model
                                       │
     Source Target                                          Destination Target
┌───────────────────────┐         Sequential Data Stream    ┌───────────────────────┐
│ File, Socket, RAM     │ ════════════════════════════════► │ Java Memory / Program │
│ Keyboard, System Pipe │           (Input Stream)          │ Local Variable Arrays │
└───────────────────────┘                                   └───────────────────────┘
                                       │
┌───────────────────────┐         Sequential Data Stream    ┌───────────────────────┐
│ Java Memory / Program │ ════════════════════════════════► │ File, Socket, RAM     │
│ Local Variable Arrays │           (Output Stream)         │ Screen, Network Port  │
└───────────────────────┘                                   └───────────────────────┘
```

#### Core Invariants of the Stream Abstraction:

1. **Unidirectional:** A stream flows in one direction only. Input streams consume data; output streams emit data. (Bidirectional communication requires two separate streams or modern NIO channels).
2. **Sequential Access:** Bytes/characters are read or written sequentially one after another. Seeking backwards requires specialized random-access wrappers (`RandomAccessFile`).
3. **Blocking by Default:** Traditional `java.io` stream methods (`read()`, `write()`) halt thread execution until data is available, an OS buffer completes its flush, or an End-Of-File (EOF) marker is reached.

---

### 2. The Four Abstract Foundation Classes

The `java.io` library is partitioned cleanly into a $2 \times 2$ matrix based on **data unit size** (8-bit raw bytes vs. 16-bit Unicode characters) and **data flow direction** (Input vs. Output).

```
                                java.io Root Hierarchy
                                          │
         ┌────────────────────────────────┴────────────────────────────────┐
         ▼                                                                 ▼
    Byte Streams                                                     Character Streams
  (8-bit binary data)                                              (16-bit Unicode characters)
  ┌───────────────────────┐                                        ┌───────────────────────┐
  │ InputStream  (Input)  │                                        │ Reader       (Input)  │
  ├───────────────────────┤                                        ├───────────────────────┤
  │ OutputStream (Output) │                                        │ Writer       (Output) │
  └───────────────────────┘                                        └───────────────────────┘
```

| Foundation Class   | Data Unit      | Flow Direction | Primary Method      | End-of-Stream (EOF) Indicator |
| :----------------- | :------------- | :------------- | :------------------ | :---------------------------- |
| **`InputStream`**  | 8-bit Raw Byte | Input (Read)   | `int read()`        | Returns `-1`                  |
| **`OutputStream`** | 8-bit Raw Byte | Output (Write) | `void write(int b)` | N/A (Flushes or closes)       |
| **`Reader`**       | 16-bit Char    | Input (Read)   | `int read()`        | Returns `-1`                  |
| **`Writer`**       | 16-bit Char    | Output (Write) | `void write(int c)` | N/A (Flushes or closes)       |

> ⚠️ **Why does `read()` return an `int` instead of a `byte` or `char`? (Top Interview Question)**
> An 8-bit `byte` in Java has a signed range of $-128 \text{ to } +127$. If `read()` returned a `byte`, the binary bit sequence `11111111` would represent decimal value `-1` (a valid data byte).
> The stream would be **unable to distinguish a valid data byte `-1` from the End-Of-File marker `-1`**.
> Therefore, `read()` returns a **32-bit `int`**:
>
> - Valid byte values are zero-extended into the range $0 \text{ to } 255$ (`0x00` to `0xFF`).
> - The literal `-1` (`0xFFFFFFFF`) is reserved exclusively to signal **EOF (End of File)**.

---

### 3. Byte Streams: `InputStream` and `OutputStream`

Byte streams process raw binary octets ($8$ bits) directly. They are format-agnostic and should be used for compiled `.class` files, images, PDFs, audio/video streams, or encrypted network packets.

```
                           InputStream Hierarchy (Partial)
                                  java.lang.Object
                                         │
                                         ▼
                                java.io.InputStream
                                         │
         ┌───────────────────────────────┼───────────────────────────────┐
         ▼                               ▼                               ▼
FileInputStream                  ByteArrayInputStream             FilterInputStream
(Reads raw bytes from disk)      (Reads bytes from RAM array)            │
                                                                 ┌───────┴───────┐
                                                                 ▼               ▼
                                                        BufferedInputStream  DataInputStream
```

#### 3.1 Fundamental Methods of `InputStream`:

- `public abstract int read() throws IOException`: Reads a single byte ($0\text{--}255$), returning `-1` on EOF.
- `public int read(byte[] b) throws IOException`: Reads up to `b.length` bytes into an array. Returns the actual number of bytes read, or `-1` on EOF.
- `public int read(byte[] b, int off, int len) throws IOException`: Reads up to `len` bytes starting at offset `off`.
- `public int available() throws IOException`: Estimates the number of bytes that can be read without blocking.
- `public void close() throws IOException`: Releases the underlying OS file descriptor.

#### 3.2 Fundamental Methods of `OutputStream`:

- `public abstract void write(int b) throws IOException`: Writes the lower 8 bits of the integer argument.
- `public void write(byte[] b) throws IOException`: Emits the entire byte array to the target sink.
- `public void flush() throws IOException`: Forces any buffered output bytes down to the underlying OS storage driver.
- `public void close() throws IOException`: Flushes the buffer and releases OS resources.

---

### 4. Character Streams: `Reader` and `Writer`

Character streams process 16-bit Unicode characters ($2$ bytes). They integrate internal character encoding conversions (e.g., UTF-8, UTF-16, ISO-8859-1, ASCII) using the host platform's default charset or an explicitly passed `CharsetDecoder`.

```
                            Reader Hierarchy (Partial)
                                 java.lang.Object
                                        │
                                        ▼
                                 java.io.Reader
                                        │
         ┌──────────────────────────────┼──────────────────────────────┐
         ▼                              ▼                              ▼
  BufferedReader                 InputStreamReader               CharArrayReader
  (Buffers chars & readLine())   (Bridge: Bytes -> Chars)        (Reads char[] from RAM)
                                        │
                                        ▼
                                   FileReader
                                 (Convenience wrapper)
```

#### 4.1 Fundamental Methods of `Reader`:

- `public int read() throws IOException`: Reads a single character ($0\text{--}65535$), returning `-1` on EOF.
- `public int read(char[] cbuf) throws IOException`: Reads characters into an array; returns total characters read.
- `public abstract void close() throws IOException`: Releases underlying resources.

#### 4.2 Fundamental Methods of `Writer`:

- `public void write(int c) throws IOException`: Writes the lower 16 bits of the integer.
- `public void write(char[] cbuf) throws IOException`: Writes an array of characters.
- `public void write(String str) throws IOException`: Writes a sequence of characters directly from a `String`.
- `public abstract void flush() throws IOException`: Pushes buffered characters to the sink.
- `public abstract void close() throws IOException`: Flushes and closes the stream.

---

### 5. Concrete Node Streams: File Processing

A **Node Stream** connects directly to a physical data source (disk file, memory block, socket).

#### 5.1 `FileInputStream` & `FileOutputStream`

```java
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;

public class RawByteCopy {
    public static void main(String[] args) {
        // Appending flag: true appends to existing file; false overwrites
        try (FileInputStream in = new FileInputStream("source.bin");
             FileOutputStream out = new FileOutputStream("dest.bin", false)) {

            byte[] buffer = new byte[4096]; // 4 KB read chunk
            int bytesRead;

            // Loop reads chunk-by-chunk until EOF (-1)
            while ((bytesRead = in.read(buffer)) != -1) {
                out.write(buffer, 0, bytesRead); // Write exact count read!
            }
        } catch (IOException e) {
            System.err.println("I/O Failure: " + e.getMessage());
        }
    }
}
```

> ⚠️ **The Write Array Off-by-One Trap (High-Yield MCQ):**
> Notice the call: `out.write(buffer, 0, bytesRead);`.
> Writing `out.write(buffer);` is a **critical bug**! If the final read chunk retrieves only 1200 bytes into a 4096-byte array, writing the whole buffer writes 2896 bytes of leftover stale data from the previous iteration, corrupting the destination file.

#### 5.2 `FileReader` & `FileWriter`

Convenience classes bridging files directly to character streams using default/specified charsets.

```java
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;
import java.nio.charset.StandardCharsets;

public class TextFileHandler {
    public static void main(String[] args) {
        // Always pass explicit StandardCharsets to avoid OS-dependent encoding corruption!
        try (FileReader reader = new FileReader("input.txt", StandardCharsets.UTF_8);
             FileWriter writer = new FileWriter("output.txt", StandardCharsets.UTF_8)) {

            char[] cbuf = new char[1024];
            int charsRead;
            while ((charsRead = reader.read(cbuf)) != -1) {
                writer.write(cbuf, 0, charsRead);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

---

### 6. The Decorator Pattern & Buffered Filter Streams

Direct, unbuffered node streams interact with the OS kernel via native system calls (`read`/`write` syscalls) for **every single byte/character**. Reading a 1 MB file byte-by-byte triggers roughly 1,000,000 OS context switches, degrading performance.

To resolve this, Java uses the **Decorator (Wrapper) Pattern**. Buffered streams wrap around unbuffered node streams, filling an in-memory byte/char buffer in large chunks.

```
                  Decorator Pattern Assembly
┌─────────────────────────────────────────────────────────────┐
│ BufferedReader                                              │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ FileReader (Node Stream)                                │ │
│ │ ┌─────────────────────────────────────────────────────┐ │ │
│ │ │ OS Kernel File Descriptor: "data.txt"               │ │ │
│ │ └─────────────────────────────────────────────────────┘ │ │
│ └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

#### 6.1 `BufferedInputStream` & `BufferedOutputStream`

- Intercepts byte streams using an internal array (default: 8192 bytes / 8 KB).
- Reads large chunks from the disk into the buffer; application `read()` calls consume directly from fast RAM.

```java
// Wrapping node stream with buffer decorator:
try (BufferedInputStream bis = new BufferedInputStream(new FileInputStream("video.mp4"));
     BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("copy.mp4"))) {

    int data;
    while ((data = bis.read()) != -1) {
        bos.write(data);
    }
    // bos.flush(); // Automatically invoked upon close()
}
```

#### 6.2 `BufferedReader` & `BufferedWriter`

- Enhances character reading with line-oriented capabilities.
- Provides the high-frequency method: **`String readLine()`** (returns `null` upon hitting EOF).
- `BufferedWriter` provides **`newLine()`** for platform-independent line breaks (`\n` on Linux/macOS, `\r\n` on Windows).

```java
import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;

public class LineByLineProcessor {
    public static void main(String[] args) {
        try (BufferedReader br = new BufferedReader(new FileReader("server.log"));
             BufferedWriter bw = new BufferedWriter(new FileWriter("errors.log"))) {

            String line;
            // readLine() strips trailing \r and \n characters!
            while ((line = br.readLine()) != null) {
                if (line.contains("[ERROR]")) {
                    bw.write(line);
                    bw.newLine(); // Emits OS-specific line terminator
                }
            }
        } catch (IOException e) {
            System.err.println("Log processing failed: " + e.getMessage());
        }
    }
}
```

---

### 7. The `java.io.File` Class

The `File` class is an abstract, immutable representation of **file and directory pathnames**, not the data within the file itself.

```
                                java.io.File
                                     │
         ┌───────────────────────────┴───────────────────────────┐
         ▼                                                       ▼
File Metadata Inspection                               Filesystem Mutation
- exists()                                             - createNewFile()
- isFile() / isDirectory()                             - mkdir() / mkdirs()
- length() [Bytes]                                     - delete() / deleteOnExit()
- getAbsolutePath()                                    - renameTo(File dest)
- canRead() / canWrite() / canExecute()                - listFiles() [Directory traversal]
```

#### 7.1 Common `File` Class Methods:

```java
import java.io.File;
import java.io.IOException;

public class FileSystemDemo {
    public static void main(String[] args) throws IOException {
        File dir = new File("reports/2026");
        // mkdirs() creates parent directories if absent; mkdir() fails if parents missing
        if (!dir.exists()) {
            dir.mkdirs();
        }

        File file = new File(dir, "financials.csv");
        if (!file.exists()) {
            boolean created = file.createNewFile(); // Creates file on physical disk
            System.out.println("File created: " + created);
        }

        System.out.println("Is Directory: " + file.isDirectory()); // false
        System.out.println("Is File:      " + file.isFile());      // true
        System.out.println("Size (Bytes):  " + file.length());      // File size in bytes
        System.out.println("Absolute Path: " + file.getAbsolutePath());

        // Directory listing:
        File root = new File(".");
        File[] entries = root.listFiles(); // Returns File objects array
        if (entries != null) {
            for (File f : entries) {
                System.out.printf("%s [%s]%n", f.getName(), f.isDirectory() ? "DIR" : "FILE");
            }
        }
    }
}
```

---

### 8. Text vs. Binary Processing Architecture

```
                               Choosing the Correct API
                                          │
                 Is data human-readable character text?
                                ┌─────────┴─────────┐
                           Yes  │                   │ No
                                ▼                   ▼
                        Character Streams      Byte Streams
                        - Reader / Writer      - InputStream / OutputStream
                        - Handles Encodings    - Raw 8-bit octets
                        - Charset translations - Zero translation/corruption
                        - .txt, .json, .csv    - .png, .class, .zip, .mp3
```

> ⚠️ **The Encoding Corruption Danger:**
> Reading a compiled Java `.class` file or a `.png` image using a `FileReader` will corrupt the data. Character streams attempt to map raw byte sequences to valid characters in a character set. Any non-valid or control byte sequence will be mapped to the replacement character (`\uFFFD` / `?`), altering the binary structure and destroying the file.

---

### 9. Resource Deallocation Mechanics

Operating systems enforce strict limits on the number of concurrently open **File Descriptors (FD)**. Failing to close streams results in **Resource Leaks**, leading to system-level errors like `IOException: Too many open files`.

```
                    Closing Resource Pipelines
                                │
    ┌───────────────────────────┴───────────────────────────┐
    │ 1. Closing outer decorator (e.g., BufferedReader)     │
    │ 2. Decorator flushes internal buffer                  │
    │ 3. Decorator cascades call to inner node stream       │
    │ 4. Inner node stream closes OS file descriptor handle │
    └───────────────────────────────────────────────────────┘
```

#### 9.1 The Cascade Closing Invariant:

When streams are wrapped via decorators, **you only need to close the outermost decorator**.
Calling `bufferedReader.close()` automatically delegates to and closes the underlying `FileReader` and its OS file descriptor.

#### 9.2 Comparison of Cleanup Paradigms:

| Paradigm                 | Era       | Cleanup Reliability                                              | Exception Handling                                       | Boilerplate Level |
| :----------------------- | :-------- | :--------------------------------------------------------------- | :------------------------------------------------------- | :---------------- |
| **Manual `finally`**     | Java 1.0+ | Low (Prone to developer omissions and nested `try-catch` errors) | Closing exceptions can suppress original body exceptions | Extremely High    |
| **`try-with-resources`** | Java 7+   | **Absolute** (Enforced by compiler bytecode lowering)            | Uses `addSuppressed()` to preserve all failures          | Minimal           |

---

### 10. High-Yield CDAC Exam Traps & Interview Questions

- **Q1: What does `FileOutputStream("data.txt", true)` do versus `FileOutputStream("data.txt")`?**
  - _Answer:_ The 2-argument constructor takes a boolean `append` flag. When set to `true`, writes append to the end of the existing file. The single-argument constructor sets `append = false` by default, **truncating and overwriting** the file's previous contents immediately upon opening.

- **Q2: Why must you explicitly invoke `flush()` on a buffered output stream before application shutdown if not using try-with-resources?**
  - _Answer:_ A `BufferedOutputStream` or `BufferedWriter` accumulates data in an internal RAM array (e.g., 8192 bytes) to minimize disk writes. If the program terminates while the buffer is only partially full (e.g., containing 500 bytes), those unwritten bytes never reach physical disk storage, causing **silent data loss**. Calling `flush()` forces the buffer to write its contents immediately.

- **Q3: What does `BufferedReader.readLine()` return when it reaches the end of the file?**
  - _Answer:_ It returns **`null`**. (Unlike `read()`, which returns integer `-1`). Attempting to check `while (br.readLine() != -1)` causes a compilation error because `readLine()` returns a `String`, which cannot be compared to an `int`.

- **Q4: What is the difference between `File.mkdir()` and `File.mkdirs()`?**
  - _Answer:_
    - `mkdir()` creates the designated directory only if its parent directories already exist. It returns `false` if any parent directory is missing.
    - `mkdirs()` creates the target directory **along with all missing intermediate parent directories** in the path.

- **Q5: Can `File` objects be used to directly read or write file data?**
  - _Answer:_ **No.** The `java.io.File` class represents only pathname metadata and filesystem navigation handles. To read or write actual data bytes or characters, the `File` object must be passed to a stream or reader/writer (such as `new FileInputStream(file)` or `new FileReader(file)`), or used via modern NIO methods (`Files.readAllBytes(file.toPath())`).
