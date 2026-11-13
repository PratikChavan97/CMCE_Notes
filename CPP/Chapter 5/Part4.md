# Module 05: I/O Streams & Formatting

## Unit 5.4: File Handling

### 1. Introduction to File Handling

File handling in C++ provides persistent secondary storage for data, enabling programs to retain state across process executions. Secondary storage I/O is managed using stream abstractions provided in the `<fstream>` header.

```
+------------------+         Byte Stream         +------------------+
|   RAM / Memory   |  =======================>   | Secondary Disk   |
|   (Variables)    |        (Write / Put)        | (data.txt / .bin)|
|                  |  <=======================   |                  |
+------------------+         (Read / Get)        +------------------+
```

- **Text Files:** Store data as human-readable ASCII or UTF-8 characters. Line breaks are converted based on host OS conventions (`\r\n` on Windows, `\n` on Linux/macOS).
- **Binary Files:** Store data in the exact binary format as it appears in RAM (bit-for-bit memory images). No line break conversions or formatting translations occur.

---

### 2. File Stream Classes (`<fstream>`)

The file stream classes derive from the fundamental `ios` stream hierarchy:

```
                            ios_base
                               |
                           basic_ios
                               |
             +-----------------+-----------------+
             |                                   |
          istream                             ostream
             |       \                       /   |
             |        +----------+----------+    |
             |                   |               |
             v                   v               v
          ifstream            fstream         ofstream
       (File Input)       (Bidirectional)  (File Output)
```

#### Primary File Stream Classes:

1. **`std::ifstream` (Input File Stream):**
   - Derived from `std::istream`.
   - Designed strictly for **reading** data from secondary storage files.
   - Default open mode: `std::ios::in`.

2. **`std::ofstream` (Output File Stream):**
   - Derived from `std::ostream`.
   - Designed strictly for **creating and writing** data to files.
   - Default open mode: `std::ios::out | std::ios::trunc`.

3. **`std::fstream` (File Stream - Bidirectional):**
   - Derived from `std::iostream`.
   - Capable of **simultaneous reading and writing** on the same file.
   - Default open mode: `std::ios::in | std::ios::out`.

---

### 3. Opening and Closing Files

#### 3.1 Opening Files

Files can be opened either via the stream **constructor** or via the explicit **`.open()`** member function:

```
#include <fstream>

// Method A: Constructor Opening
std::ofstream outFile("records.txt", std::ios::out);

// Method B: Member Function Opening
std::ifstream inFile;
inFile.open("records.txt", std::ios::in);
```

#### 3.2 Validating File Open Status

Always verify that a file opened successfully before attempting operations:

```
// Technique 1: is_open() method (Recommended)
if (!inFile.is_open()) {
    std::cerr << "Failed to locate and open target file!\n";
}

// Technique 2: Overloaded operator! or boolean conversion
if (!inFile) {
    std::cerr << "Stream entered an error state upon opening!\n";
}
```

#### 3.3 Closing Files & RAII

- The member function `.close()` flushes pending internal stream buffers to disk, releases the OS file descriptor, and disconnects the stream object.
- **RAII Guarantee:** File stream destructors automatically call `.close()` when a stream object leaves scope. Manual `.close()` calls are only required if you need to reuse the stream object for another file in the same scope or force immediate flushing.

---

### 4. File Opening Modes (`std::ios::openmode`)

Multiple flags can be combined using the **bitwise OR operator (`|`)**:

| Open Mode Flag         | Meaning / Behavior                                                                                                                      |
| :--------------------- | :-------------------------------------------------------------------------------------------------------------------------------------- |
| **`std::ios::in`**     | Opens file for **reading** (input). File _must exist_ beforehand.                                                                       |
| **`std::ios::out`**    | Opens file for **writing** (output). Creates new file if missing; truncates by default if used alone.                                   |
| **`std::ios::app`**    | **Append mode.** Repositions the write pointer to EOF before _every single write_. Does not overwrite existing content.                 |
| **`std::ios::ate`**    | **"At End."** Opens the file and moves the pointer to EOF immediately upon opening; reading/writing can still occur anywhere via seeks. |
| **`std::ios::trunc`**  | **Truncate.** If the file exists, discards all existing contents (truncates length to 0).                                               |
| **`std::ios::binary`** | Opens in **binary mode**. Suppresses platform-specific newline transformations (`\n` $\leftrightarrow$ `\r\n`).                         |

#### Common Mode Combinations:

- `std::ios::in | std::ios::out`: Open an existing file for reading and writing without automatic truncation.
- `std::ios::in | std::ios::out | std::ios::trunc`: Open for read/write, wiping existing contents.
- `std::ios::out | std::ios::app`: Write-only mode where output is appended to EOF.
- `std::ios::in | std::ios::binary`: Read raw binary bytes without character transformation.

---

### 5. Reading and Writing Text Files

#### 5.1 Writing Text Data

```
#include <iostream>
#include <fstream>

void writeText() {
    std::ofstream out("student.txt");
    if (out.is_open()) {
        out << 101 << " " << "Aryan_Sharma" << " " << 92.5 << "\n";
        out << 102 << " " << "Pooja_Verma"  << " " << 88.0 << "\n";
        out.close();
    }
}
```

#### 5.2 Reading Text Data

```
void readText() {
    std::ifstream in("student.txt");
    if (!in.is_open()) return;

    // Approach 1: Formatted Extraction (Word/Primitive-by-word)
    int roll;
    std::string name;
    double marks;
    while (in >> roll >> name >> marks) {
        std::cout << roll << " | " << name << " | " << marks << "\n";
    }

    in.clear(); // Reset EOF flag if reusing stream
    in.seekg(0, std::ios::beg); // Rewind to start

    // Approach 2: Line-by-Line Reading (Preserves whitespace)
    std::string line;
    while (std::getline(in, line)) {
        std::cout << "Line: " << line << "\n";
    }
    in.close();
}
```

---

### 6. File Pointers & Random Access

File streams maintain internal position indicators tracking where the next read or write operation will occur:

- **Get Pointer (`g`):** Associated with **input** streams (`ifstream`).
- **Put Pointer (`p`):** Associated with **output** streams (`ofstream`).

```
                +----+----+----+----+----+----+----+----+
File Bytes:     | A  | B  | C  | D  | E  | F  | G  | H  |
                +----+----+----+----+----+----+----+----+
                  ^                        ^
                  |                        |
         tellg() == 0 (Beginning)        tellp() == 5 (Current Offset)
```

#### 6.1 Inspector Functions (`tellg` & `tellp`)

- `std::streampos tellg();`: Returns the current byte offset of the **get** pointer from the beginning.
- `std::streampos tellp();`: Returns the current byte offset of the **put** pointer from the beginning.

#### 6.2 Mutator Functions (`seekg` & `seekp`)

Used for random access navigation within a file:

```
// Syntax:
stream.seekg(offset, seek_direction);
stream.seekp(offset, seek_direction);
```

#### Seek Directions (`std::ios::seekdir`):

1. **`std::ios::beg`**: Offset relative to the **beginning** of the file.
2. **`std::ios::cur`**: Offset relative to the **current position** of the file pointer.
3. **`std::ios::end`**: Offset relative to the **end** of the file (usually negative offsets).

```
std::ifstream in("data.txt");

in.seekg(10, std::ios::beg);  // Move to 10th byte from beginning
in.seekg(5, std::ios::cur);   // Advance 5 bytes forward from current spot
in.seekg(-10, std::ios::end); // Move to 10 bytes before EOF

// Practical Trick: Determine File Size in Bytes
in.seekg(0, std::ios::end);
std::streampos fileSize = in.tellg();
std::cout << "Total file size: " << fileSize << " bytes\n";
```

---

### 7. Binary File Handling & Object Serialization

Binary files read and write raw byte representations directly to/from memory buffers without text encoding overhead.

#### 7.1 Binary Member Functions

```
// Write binary bytes from memory buffer to file:
ostream& write(const char* buffer, std::streamsize numBytes);

// Read binary bytes from file into memory buffer:
istream& read(char* buffer, std::streamsize numBytes);
```

#### 7.2 Writing and Reading Entire Class Objects

```
#include <iostream>
#include <fstream>
#include <cstring>

class Employee {
private:
    int empId;
    char name[50]; // Use fixed-size arrays, NOT std::string for raw binary writes!
    double salary;

public:
    Employee(int id = 0, const char* n = "", double s = 0.0) : empId(id), salary(s) {
        std::strncpy(name, n, sizeof(name) - 1);
        name[sizeof(name) - 1] = '\0';
    }

    void display() const {
        std::cout << "ID: " << empId << " | Name: " << name
                  << " | Salary: $" << salary << "\n";
    }
};

void writeBinary() {
    Employee e1(101, "Alice Smith", 75000.0);
    Employee e2(102, "Bob Jones", 62000.0);

    std::ofstream out("employees.dat", std::ios::out | std::ios::binary);
    if (!out) return;

    // reinterpret_cast to (char*) required for binary write
    out.write(reinterpret_cast<const char*>(&e1), sizeof(Employee));
    out.write(reinterpret_cast<const char*>(&e2), sizeof(Employee));
    out.close();
}

void readBinary() {
    Employee temp;
    std::ifstream in("employees.dat", std::ios::in | std::ios::binary);
    if (!in) return;

    // Read loop: read() returns a reference to the stream; loop checks status
    while (in.read(reinterpret_cast<char*>(&temp), sizeof(Employee))) {
        temp.display();
    }
    in.close();
}
```

> ⚠️ **The Pointer Serialization Hazard (High-Yield Interview Trap):**
>
> Never write objects containing **raw pointers** or dynamic structures (such as `std::string`, `std::vector`) directly to disk using `write(reinterpret_cast<char*>(&obj), sizeof(obj))`!
>
> _Reason:_ `sizeof(obj)` will write the object's _internal pointer address_ to disk, rather than the heap contents it points to. When read back in during a later program run, that pointer address is **dangling and invalid**, causing immediate memory corruption or crashes.

---

### 8. Stream State Flags in File I/O

Files encounter hardware boundaries and unexpected states regularly:

```
+------------+-------------------------------------------------------------+
| State Bit  | Meaning in File Context                                     |
+------------+-------------------------------------------------------------+
| goodbit    | Everything succeeded; stream is healthy.                    |
| eofbit     | The read pointer crossed the end of the file.              |
| failbit    | Formatting mismatch or file failed to open in requested mode.|
| badbit     | Hardware read/write failure; corrupted filesystem.          |
+------------+-------------------------------------------------------------+
```

```
// Pattern: Robust EOF Loop
while (in.peek() != EOF) {
    // Process next item safely without processing garbage on last iteration
}
```

---

### 9. High-Yield CMCE Traps & Interview Questions

- **Q1: What is the difference between `ios::app` and `ios::ate`?**
  - _Answer:_
    - `ios::app` (Append) forces **every single write operation** to the end of the file. You cannot reposition the write pointer using `seekp()` to overwrite earlier data.
    - `ios::ate` (At End) opens the file and moves the pointer to the end **only once upon opening**. After opening, you can freely use `seekp()` or `seekg()` to move the pointer anywhere in the file.

- **Q2: Why is testing `while (!file.eof())` considered an antipattern?**
  - _Answer:_ The `eofbit` flag is set **only after an attempt to read past the end of the file has already failed**. Using `while (!file.eof())` causes the loop body to execute one extra time using stale or garbage data from the failed final read. The correct pattern is to check the result of the read operation itself: `while (file >> val)` or `while (file.read(...))`.

- **Q3: What happens if you open a file with `ofstream` using only `ios::out`?**
  - _Answer:_ If a file is opened with `ios::out` and neither `ios::in`, `ios::app`, nor `ios::ate` is specified, the compiler implicitly sets **`ios::trunc`**. Any existing file with that name is immediately cleared to zero bytes upon opening.

- **Q4: Can we write an object of a class having virtual functions directly to a binary file?**
  - _Answer:_ **No, it is dangerous.** Classes with virtual functions contain an internal Virtual Table Pointer (`_vptr`). Writing the object to disk writes the process-specific virtual table memory address. When loaded in a different run or on another machine, the restored `_vptr` will point to invalid or random memory, causing an immediate crash upon calling any virtual function.

- **Q5: What is the return type of `tellg()` and `tellp()`?**
  - _Answer:_ They return an instance of **`std::streampos`** (defined in `<ios>`), which represents the absolute byte position in the stream and can be converted to an integer offset.
