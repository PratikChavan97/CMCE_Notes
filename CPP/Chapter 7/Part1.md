# Module 07: Standard Template Library (STL)

## Unit 7.1: STL Fundamentals & Architecture

### 1. Introduction to the Standard Template Library (STL)

The **Standard Template Library (STL)** is a collection of general-purpose, type-safe, generic C++ class templates and algorithms. Developed originally by **Alexander Stepanov** and **Meng Lee** at Hewlett-Packard in 1994, it was formally integrated into the ANSI/ISO C++ standard in 1998.

#### 1.1 Philosophy & Design Principles

- **Separation of Algorithms from Data Structures:** Rather than making search or sort algorithms member functions of every container (like `list.sort()`), STL treats algorithms as independent template functions that operate on _any_ container via **Iterators**.

- **Zero Runtime Overhead:** Relies entirely on compile-time templates and inlining; avoids dynamic dispatch (`virtual` functions) within core containers.

- **Algorithmic Efficiency Guarantees:** The C++ standard mandates strict Big-O time and space complexity requirements for every container operation (e.g., $O(1)$ random access for `vector`, $O(\log n)$ lookup for `std::map`).

### 2. STL Architecture: The 4 Core Pillars

The STL is organized into four interconnected component groups:

```
+--------------------------------------------------------------------------+
|                                ALGORITHMS                                |
|           (std::sort, std::find, std::for_each, std::transform)          |
+--------------------------------------------------------------------------+
                                     ^
                                     |  Operate via
                                     v
+--------------------------------------------------------------------------+
|                                ITERATORS                                 |
|            (Glue layer: act as generalized smart pointers)               |
+--------------------------------------------------------------------------+
                                     ^
                                     |  Traverse / Expose
                                     v
+--------------------------------------------------------------------------+
|                                CONTAINERS                                |
|          (Sequence, Associative, Unordered, Adapters: memory)            |
+--------------------------------------------------------------------------+
                                     ^
                                     |  Parametrized by
                                     v
+--------------------------------------------------------------------------+
|                             FUNCTION OBJECTS                             |
|              (Functors, Predicates, Lambdas, std::greater)               |
+--------------------------------------------------------------------------+

```

1. **Containers:** Manage data collections in memory (e.g., `vector`, `list`, `map`).

2. **Iterators:** Act as the abstract bridge ("glue") connecting algorithms to containers without exposing underlying memory arrangements.

3. **Algorithms:** Implement common computational procedures (e.g., searching, sorting, permutations, counting).

4. **Functors (Function Objects):** Wrap functions/operations in class objects to customize algorithmic behavior (e.g., sorting orders, conditional predicates).
   _(Note: **Allocators** also form a foundational 5th subsystem handling memory allocation/deallocation strategies, but are rarely interacted with directly)._

### 3. Containers Overview & Classification

Containers manage the physical layout and lifecycle of elements in memory.

```
                                STL Containers
                                       |
    +--------------------+-------------+-------------+--------------------+
    |                    |                           |                    |
Sequence            Associative                 Unordered             Container
Containers          Containers                  Associative           Adapters
(Linear Storage)    (Sorted / Balanced Trees)   (Hash Tables: O(1))   (Restricted Wrappers)
    |                    |                           |                    |
 - std::vector       - std::set                  - std::unordered_set  - std::stack
 - std::deque        - std::multiset             - std::unordered_multiset - std::queue
 - std::list         - std::map                  - std::unordered_map  - std::priority_queue
 - std::forward_list - std::multimap             - std::unordered_multimap
 - std::array (C++11)

```

#### 3.1 Classification Summary

| Container Family | Internal Implementation                       | Key Characteristic                           | Typical Complexity                                 |
| ---------------- | --------------------------------------------- | -------------------------------------------- | -------------------------------------------------- |
| **Sequence**     | Contiguous arrays, doubly/singly linked lists | Preserves element insertion order            | Random: $O(1)$ (`vector`), Linear: $O(n)$ (`list`) |
| **Associative**  | Self-balancing BST (Red-Black Trees)          | Elements ordered automatically by `Key`      | Search/Insert/Delete: $O(\log n)$                  |
| **Unordered**    | Bucketed Hash Tables with buckets & chaining  | Keys mapped via hash functions (`std::hash`) | Avg: $O(1)$, Worst case: $O(n)$                    |
| **Adapters**     | Internal wrappers over sequence containers    | Restrict API to provide LIFO/FIFO interfaces | Push/Pop: $O(1)$ amortized                         |

### 4. Iterators: The Bridge of STL

An **iterator** is a generalized abstraction of a pointer. It points to an element within a container and provides operators to traverse the range $[begin, end)$.

#### 4.1 The Half-Open Range Invariant: `[begin, end)`

```
Container elements:
       +--------+--------+--------+--------+
       |   10   |   20   |   30   |   40   |
       +--------+--------+--------+--------+
            ^                                  ^
            |                                  |
         c.begin()                          c.end() (Past-the-end sentinel!)

```

- `c.begin()`: Points to the **first valid element**.

- `c.end()`: Points to the **one-past-the-end** imaginary element. Dereferencing `*c.end()` is **Undefined Behavior**.

- **Empty Container Check:** `c.begin() == c.end()` evaluates to `true` if and only if the container is empty.

### 5. Iterator Categories & Hierarchy

Iterators are divided into five primary hierarchical categories based on the operations and memory traversal patterns they support:

```
                            Input Iterator       Output Iterator
                            (Read-only, 1-pass)  (Write-only, 1-pass)
                                   \                 /
                                    \               /
                                  Forward Iterator
                                  (Multi-pass, Read/Write)
                                         |
                                         v
                              Bidirectional Iterator
                              (Bi-directional: ++ and --)
                                         |
                                         v
                              Random-Access Iterator
                              (Jump: it += n, it1 - it2, it[n])
                                         |
                                         v  (C++20)
                              Contiguous Iterator
                              (Physically contiguous in RAM)

```

#### 5.1 Category Capabilities Matrix

| Iterator Category      | Operations Supported                                                                | Multi-pass?         | Example Containers / Streams                            |
| ---------------------- | ----------------------------------------------------------------------------------- | ------------------- | ------------------------------------------------------- |
| **Input Iterator**     | `*it` (read-only), `++it`, `it++`, `==`, `!=`                                       | ❌ No (Single pass) | `std::istream_iterator`                                 |
| **Output Iterator**    | `*it = val` (write-only), `++it`, `it++`                                            | ❌ No (Single pass) | `std::ostream_iterator`, `back_inserter`                |
| **Forward Iterator**   | Reads & Writes, `++it`, `it++`, multi-pass                                          | ✅ Yes              | `std::forward_list`, `std::unordered_set`               |
| **Bidirectional**      | All Forward operations PLUS `--it`, `it--`                                          | ✅ Yes              | `std::list`, `std::set`, `std::map`                     |
| **Random-Access**      | All Bidirectional PLUS `it + n`, `it - n`, `it += n`, `it[n]`, `<`, `>`, `<=`, `>=` | ✅ Yes              | `std::vector`, `std::deque`, `std::array`, Raw C-arrays |
| **Contiguous (C++20)** | All Random Access PLUS guaranteed contiguous memory (`&(*(it + n)) == &(*it) + n`)  | ✅ Yes              | `std::vector`, `std::array`, `std::string`              |

> ⚠️ **Key Exam Rule on Algorithms & Iterators:**
> Algorithms like `std::sort()` demand **Random-Access Iterators**. Therefore:
>
> - `std::sort(vec.begin(), vec.end())` $\rightarrow$ **Valid** (`vector` provides random-access iterators).
> - `std::sort(myList.begin(), myList.end())` $\rightarrow$ ❌ **COMPILE ERROR!** (`list` provides only Bidirectional iterators; you must use `myList.sort()` instead).

### 6. Algorithms Overview (`<algorithm>`)

STL algorithms are non-member, template-based functions operating on iterator ranges. They do not allocate container memory directly.

#### 6.1 Major Algorithmic Categories

1. **Non-Modifying Sequence Operations:**
   - `std::find(first, last, val)`: Linear search.

   - `std::count(first, last, val)`: Counts occurrences.

   - `std::binary_search(first, last, val)`: $O(\log n)$ existence check on **sorted ranges**.

2. **Modifying Sequence Operations:**
   - `std::copy(first, last, out_it)`: Copies elements into an output iterator.

   - `std::reverse(first, last)`: Inverts element sequence.

   - `std::fill(first, last, val)`: Sets range to specific value.

3. **Sorting & Partitioning:**
   - `std::sort(first, last)`: Introsort ($O(n \log n)$ worst-case).

   - `std::stable_sort(first, last)`: Preserves relative order of equivalent items.

4. **Numeric Operations (`<numeric>`):**
   - `std::accumulate(first, last, init)`: Computes sum or fold reduction.

### 7. Function Objects (Functors)

A **Function Object (Functor)** is any class or struct that overloads the **function call operator: `operator()`**.

```
#include <iostream>
#include <vector>
#include <algorithm>

// Custom Functor
struct GreaterThan {
    int threshold;
    GreaterThan(int t) : threshold(t) {}

    // Overloaded call operator
    bool operator()(int val) const {
        return val > threshold;
    }
};

int main() {
    std::vector<int> nums = {5, 12, 8, 25, 3, 19};

    // Passing Functor instance with state (threshold = 10)
    int count = std::count_if(nums.begin(), nums.end(), GreaterThan(10));
    std::cout << "Count > 10: " << count << "\n"; // Output: 3
    return 0;
}

```

#### 7.1 Advantages of Functors over Raw Function Pointers

1. **State Retention:** Functors can store state across invocations using private data members (function pointers need ugly global/static variables).

2. **Performance / Inlining:** Compilers can easily **inline** functor `operator()` calls, whereas function pointers defeat inlining due to runtime address indirection.

3. **Type Integration:** A functor's signature forms part of its distinct type, enabling compile-time optimizations in templates like `std::set<int, CustomComparator>`.

#### 7.2 Built-In STL Functors (`<functional>`)

- **Arithmetic:** `std::plus<T>`, `std::minus<T>`, `std::multiplies<T>`

- **Relational / Comparison:** `std::less<T>` (Default for `std::sort` & `std::map`), `std::greater<T>`, `std::equal_to<T>`

- **Logical:** `std::logical_and<T>`, `std::logical_not<T>`

### 8. High-Yield CMCE Traps & Interview Questions

- **Q1: Why does `std::list` have its own `.sort()` member function instead of using `std::sort`?**
  - _Answer:_ `std::sort` requires **Random-Access Iterators** to perform element partitioning (like median-of-three jumps `it + n`). Doubly-linked lists (`std::list`) only support **Bidirectional Iterators** ($O(1)$ step-by-step navigation). Therefore, `std::list` provides an internal `myList.sort()` that implements an $O(n \log n)$ merge sort by manipulating node pointers rather than swapping elements.

- **Q2: What is Iterator Invalidation?**
  - _Answer:_ When a container's internal storage changes (e.g., `vector` reallocating memory during `push_back`, or erasing an element in `deque`), existing pointers or iterators referencing those slots become **dangling/invalid**. Accessing an invalidated iterator produces undefined behavior.

- **Q3: What is the difference between `c.end()` and `c.rbegin()`?**
  - _Answer:_ `c.end()` is a regular iterator pointing to the theoretical element _after_ the last element. `c.rbegin()` is a **reverse iterator** pointing directly to the **last valid element** of the container, so that advancing it (`++it`) traverses backward toward the beginning.

- **Q4: Why does `std::binary_search` return `bool` instead of an iterator?**
  - _Answer:_ `std::binary_search()` tests purely for the **existence** of an element in $O(\log n)$ time. If you need an iterator to the element, use `std::lower_bound()` (points to first element $\ge val$) or `std::equal_range()`.

- **Q5: What is the complexity guarantee of `std::sort`?**
  - _Answer:_ Since C++11, `std::sort` is guaranteed to have a worst-case time complexity of $\mathcal{O}(n \log n)$. It is typically implemented using **Introsort** (hybrid of Quicksort, switching to Heapsort if recursion depth exceeds $2 \log n$, and Insertion Sort for tiny partitions).
