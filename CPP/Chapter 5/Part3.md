# Module 05: I/O Streams & Formatting

## Unit 5.3: Console Input and Output

### 1. Introduction to Streams

In C++, I/O operations are abstracted using the concept of **Streams**. A stream is an abstract sequence of bytes flowing between a program and an input/output device (keyboard, terminal screen, disk file, or network socket).

- **Input Stream:** A sequence of bytes flowing **into** the program (e.g., from keyboard/file).
- **Output Stream:** A sequence of bytes flowing **out of** the program (e.g., to screen/printer/file).

```
[ Input Device: Keyboard ]  ===> (Stream of Bytes) ===> [ istream ] ===> [ C++ Variables ]
                                                                                |
                                                                                v
[ Output Device: Screen  ]  <=== (Stream of Bytes) <=== [ ostream ] <=== [ Data / State  ]
```

#### 1.1 The Stream Class Hierarchy (`<iostream>`)

The C++ I/O system is built upon an object-oriented class hierarchy:

```
                                  ios_base
                           (Format flags, state flags)
                                     |
                                   basic_ios
                              (Tied streams, buffer)
                                     |
             +-----------------------+-----------------------+
             |                                               |
        basic_istream                                   basic_ostream
        (cin, extraction >>)                            (cout, insertion <<)
             |       \                               /       |
             |        +--------------+--------------+        |
             |                       |                       |
             v                       v                       v
      basic_ifstream           basic_iostream          basic_ofstream
       (File input)             (Bidirectional)         (File output)
                                     |
                                     v
                               basic_fstream
```

- Typedefs for standard character types:
  - `std::istream` $\equiv$ `std::basic_istream<char>`
  - `std::ostream` $\equiv$ `std::basic_ostream<char>`
  - `std::iostream` $\equiv$ `std::basic_iostream<char>`

---

### 2. Standard Stream Objects: Overview

When `<iostream>` is included, four global stream objects are automatically initialized before `main()` executes:

| Stream Object   | Class Type     | Device Associated                    | Standard Redirection Target | Buffering Behavior                                  |
| :-------------- | :------------- | :----------------------------------- | :-------------------------- | :-------------------------------------------------- |
| **`std::cin`**  | `std::istream` | Standard Input (`stdin`, Keyboard)   | Keyboard input buffer       | **Buffered** (Flushes on `\n` or `cin.tie()`)       |
| **`std::cout`** | `std::ostream` | Standard Output (`stdout`, Terminal) | Standard output pipeline    | **Buffered** (Flushes on buffer full, `endl`, exit) |
| **`std::cerr`** | `std::ostream` | Standard Error (`stderr`, Terminal)  | Standard error device       | **Unbuffered** (Immediate output, no data loss)     |
| **`std::clog`** | `std::ostream` | Standard Log (`stderr`, Terminal)    | Standard error / log pipe   | **Buffered** (Efficient batch logging)              |

---

### 3. Input Stream (`std::cin`)

`std::cin` is used to read data from standard input using the **stream extraction operator (`>>`)** or member functions.

#### 3.1 Extraction Operator (`>>`) Mechanics

- Skips leading whitespace characters (spaces, tabs, newlines `\n`).
- Reads contiguous non-whitespace characters until the next whitespace or EOF is hit.
- Performs **automatic type conversion** based on the receiving variable's data type.

```cpp
int age;
std::cin >> age; // Reads characters, parses digits to integer
```

#### 3.2 Common Member Functions of `std::istream`

1. **`cin.get()`:**
   - Reads a single character (including spaces and newlines).

   ```cpp
   char ch;
   ch = std::cin.get();
   // or:
   std::cin.get(ch);
   ```

2. **`cin.getline(char* buffer, streamsize count, char delim = '\n')`:**
   - Reads an entire line into a C-style string buffer, up to `count - 1` characters or until delimiter `delim` is met.
   - Extracts and **discards** the delimiter character from the stream.

3. **`std::getline(std::istream& is, std::string& str, char delim = '\n')`:**
   - Defined in `<string>`; preferred for Modern C++.
   - Reads directly into a dynamically sized `std::string` without risk of buffer overflow.

4. **`cin.ignore(streamsize n = 1, int delim = EOF)`:**
   - Discards up to `n` characters or until delimiter `delim` is encountered.

#### 3.3 The Classic "cin >> followed by getline" Trap (CMCE MCQ Classic)

```cpp
int id;
std::string name;

std::cin >> id;              // User enters: 101<Enter>
// '101' is extracted into 'id', but '\n' remains inside the stream buffer!
std::getline(std::cin, name); // Immediately consumes leftover '\n'!
// 'name' becomes empty ("")!

// Fix: Clear leftover characters before getline
std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');
std::getline(std::cin, name); // Now correctly waits for user input
```

#### 3.4 Stream State Flags

Streams maintain an internal state bitmask (`iostate`) describing their health:

| Flag      | Meaning                                                      | Inspection Function |
| :-------- | :----------------------------------------------------------- | :------------------ |
| `goodbit` | No error; stream is fully operational                        | `cin.good()`        |
| `eofbit`  | Reached End-Of-File (EOF or Ctrl+D / Ctrl+Z)                 | `cin.eof()`         |
| `failbit` | Recoverable formatting error (e.g., entered "abc" for `int`) | `cin.fail()`        |
| `badbit`  | Irrecoverable fatal hardware / I/O error                     | `cin.bad()`         |

- **Resetting Stream After Errors:**
  ```cpp
  if (std::cin.fail()) {
      std::cin.clear(); // Clears error flags back to goodbit
      std::cin.ignore(1000, '\n'); // Purges invalid characters from buffer
  }
  ```

---

### 4. Output Stream (`std::cout`)

`std::cout` represents the standard output stream, utilized with the **stream insertion operator (`<<`)**.

- **Chaining:** The `<<` operator returns a reference to `std::ostream&`, enabling method chaining:
  ```cpp
  std::cout << "Val: " << 42 << " Hex: " << std::hex << 42 << '\n';
  ```

---

### 5. Error Stream (`std::cerr`) vs Logging Stream (`std::clog`)

Understanding the buffering differences between `cerr` and `clog` is essential for systems programming and interview questions:

```
[ Application Crash ]
          |
          +---> cerr << "Fatal pointer fault!"; ===> Printed IMMEDIATELY to terminal
          |
          +---> clog << "Worker thread exited."; ===> Stored in memory buffer...
                                                      (Lost if process aborts!)
```

#### 5.1 `std::cerr` (Unbuffered)

- Tied to `stderr`.
- **Zero Buffering (`unitbuf` set):** Flushes characters to the console immediately.
- **Use Case:** Critical diagnostic errors, crash notices, and fault tracing where buffering delays could cause log loss during an unexpected termination (`abort()` / segmentation fault).

#### 5.2 `std::clog` (Buffered)

- Also tied to `stderr`.
- **Fully Buffered:** Buffers output characters in memory and writes them in batches when the buffer is full or flushed.
- **Use Case:** High-volume logging, telemetry, and background progress tracking where unbuffered I/O would cause excessive system-call overhead.

---

### 6. Stream Manipulators (`<iostream>` and `<iomanip>`)

**Manipulators** are special functions or objects passed into streams using `<<` or `>>` to modify formatting, base representations, or buffering behavior.

```
                         Stream Manipulators
                                  |
         +------------------------+------------------------+
         |                                                 |
Parameterless (In <iostream>)                   Parameterized (In <iomanip>)
- endl, flush, ws                               - setw(n)
- dec, hex, oct                                 - setprecision(n)
- boolalpha, noboolalpha                        - setfill(char)
- fixed, scientific                             - setbase(n)
- left, right, internal
```

#### 6.1 Buffer Control Manipulators

- **`std::endl`:** Inserts a newline `'\n'` and calls `.flush()` on the stream.
- **`std::flush`:** Flushes output buffer immediately without adding a newline.
- **`std::ws`:** Input manipulator that extracts and discards leading whitespaces.

> ⚠️ **Performance Note:** Repeatedly using `std::endl` inside tight loops causes severe I/O bottlenecks due to repetitive OS system flushes. Prefer `'\n'` for speed.

---

### 7. Output Formatting Reference

#### 7.1 Numeric Base Formatting

```cpp
int num = 255;
std::cout << std::dec << num << '\n'; // 255 (Decimal default)
std::cout << std::hex << num << '\n'; // ff  (Hexadecimal)
std::cout << std::oct << num << '\n'; // 377 (Octal)

// Formatting additions:
std::cout << std::showbase << std::uppercase;
std::cout << std::hex << num << '\n'; // 0XFF
```

#### 7.2 Boolean Formatting

```cpp
bool status = true;
std::cout << status << '\n';               // 1 (Default numeric)
std::cout << std::boolalpha << status;     // true
std::cout << std::noboolalpha << status;   // 1
```

#### 7.3 Width, Padding, and Alignment (`<iomanip>`)

```cpp
#include <iostream>
#include <iomanip>

int main() {
    int val = 42;

    // setw applies ONLY to the very next item!
    std::cout << std::setw(8) << val << '\n';                  // "      42" (Right-aligned by default)
    std::cout << std::setw(8) << std::left << val << '\n';     // "42      " (Left-aligned)
    std::cout << std::setfill('*') << std::setw(8) << val;     // "42******" (Custom padding)
    return 0;
}
```

> ⚠️ **Crucial Rule:** Unlike alignment or base manipulators, **`std::setw()` is NOT persistent**! It resets back to default (0 width) immediately after formatting the very next item.

#### 7.4 Floating-Point Precision Formatting

```cpp
#include <iostream>
#include <iomanip>

int main() {
    double pi = 3.1415926535;

    // Default mode: setprecision sets total SIGNIFICANT DIGITS
    std::cout << std::setprecision(4) << pi << '\n'; // 3.142

    // Fixed mode: setprecision sets DIGITS AFTER DECIMAL POINT
    std::cout << std::fixed << std::setprecision(2) << pi << '\n'; // 3.14

    // Scientific notation:
    std::cout << std::scientific << pi << '\n'; // 3.1416e+00
    return 0;
}
```

---

### 8. Stream Synchronization & Fast I/O (Competitive / CDAC Tip)

By default, C++ streams synchronize their buffers with standard C I/O buffers (`printf`/`scanf`) to allow mixing them safely in the same program. This synchronization introduces measurable runtime overhead.

```cpp
// Disable C/C++ I/O synchronization
std::ios_base::sync_with_stdio(false);

// Untie cin from cout (Prevents auto-flushing cout prior to cin input)
std::cin.tie(nullptr);
```

---

### 9. High-Yield CMCE Traps & Interview Questions

- **Q1: What is the exact difference between `\n` and `std::endl`?**
  - _Answer:_ `\n` emits only the newline escape character into the buffer. `std::endl` emits the newline character and forces a buffer flush (`std::cout.flush()`). Using `endl` excessively inside loops causes redundant disk/terminal system calls and degrades performance.

- **Q2: Why does `cin.tie()` exist?**
  - _Answer:_ By default, `std::cin` is "tied" to `std::cout` (`cin.tie(&cout)`). This ensures that any pending output buffer in `cout` is automatically flushed **before** input is requested through `cin`, guaranteeing prompts (e.g., `cout << "Enter Age: "; cin >> age;`) appear on the screen before the console blocks for input.

- **Q3: What will `setw(10)` do if the output value is longer than 10 characters?**
  - _Answer:_ It will **not truncate** the output. If the formatted data exceeds the width specified by `setw`, C++ expands the field automatically to fit the entire value without clipping.

- **Q4: What happens if you input a character when `cin >> intVar` is executed?**
  - _Answer:_ The stream enters a **failure state**:
    1. `intVar` retains its old value (or 0 in C++11).
    2. The `failbit` flag is set on `std::cin`.
    3. The offending character remains in the input stream buffer.
    4. All subsequent extractions fail immediately until `cin.clear()` and `cin.ignore()` are executed.

- **Q5: Which stream manipulators are non-persistent?**
  - _Answer:_ **`std::setw()`** is the primary non-persistent manipulator; it resets to default width after the immediate next data insertion. Almost all other manipulators (such as `std::hex`, `std::fixed`, `std::setprecision`, `std::setfill`, `std::boolalpha`) remain persistent until explicitly altered.
