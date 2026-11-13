# CHAPTER 6 – STACK AND QUEUE

# PART 1 – STACK FUNDAMENTALS

## 6.1 Introduction to Data Structures

Before understanding Stack, we should understand what a data structure is.

> A data structure is a particular way of organizing, storing, and managing data in a computer so that operations such as insertion, deletion, searching, and accessing can be performed efficiently.

For example, imagine storing the following numbers:

10, 20, 30, 40, 50

We can store these numbers in an array, linked list, stack, or queue.

Although the data is the same, the way we organize and access it differs.

**Common Types of Data Structures**

Data structures can broadly be classified into two categories.

### A. Linear Data Structures

In linear data structures, elements are arranged sequentially, one after another.

Examples:

- Array
- Linked List
- Stack
- Queue

### B. Non-Linear Data Structures

In non-linear data structures, elements are not arranged sequentially. They may form hierarchical or interconnected relationships.

Examples:

- Tree
- Graph
- Heap

**Where Does Stack Belong?**

Stack is a linear data structure because its elements are arranged sequentially.

However, unlike an array, a stack restricts how elements can be inserted and removed.

Insertion and deletion happen only at one end, called the TOP.

This restriction is the fundamental characteristic of a stack.

## 2. Introduction to Stack

A Stack is a linear data structure that follows the principle of LIFO (Last In, First Out).

This means:

The element inserted last into the stack will be the first element removed from it.

A stack allows insertion and deletion of elements from only one end, known as the TOP.

Example: Stack of Plates

Imagine a stack of five plates.

You place the first plate on a table.

Then you place another plate on top of it.

You continue placing plates one above another.

Now suppose you want to remove a plate.

You will naturally remove the plate at the top first.

You cannot directly remove the bottom plate without removing all the plates above it.

This is exactly how a stack works.

Visual Representation
Initially, the stack is empty.

Insert 10:

```text
TOP
|
+----+
| 10 |
+----+
```

Insert 20:

```text
TOP
|
+----+
| 20 |
+----+
| 10 |
+----+
```

Insert 30:

```text
TOP
|
+----+
| 30 |
+----+
| 20 |
+----+
| 10 |
+----+
```

> Notice that every new element is inserted above the previous element.

If we remove an element now, 30 will be removed first.

This demonstrates LIFO.

## 3. Understanding LIFO (Last In, First Out)

LIFO is the fundamental principle behind Stack.

It means that the element inserted most recently is removed first.

Consider the following insertion sequence:

10 → 20 → 30 → 40

The insertion order is:

- 10 inserted first.
- 20 inserted second.
- 30 inserted third.
- 40 inserted last.

The stack will look like this:

```text
TOP
|
+------+
| 40 |
+------+
| 30 |
+------+
| 20 |
+------+
| 10 |
+------+
```

Now perform deletion.

The deletion order will be:

40 → 30 → 20 → 10

Therefore:

- Last inserted: 40
- First removed: 40

**Important Exam Point**

Stack follows LIFO, not FIFO.

FIFO (First In, First Out) is the principle followed by Queue, which we will study later.

## 4. Real-Life Applications of Stack

Stacks are used in many real-world applications.

### 4.1 Stack of Plates

The last plate placed on top is removed first.

### 4.2 Browser History

When navigating backward through browser history, previously visited pages can be retrieved using stack-like behavior.

For example:

Google → YouTube → Wikipedia

Pressing the Back button takes you to:

Wikipedia → YouTube → Google

### 4.3 Undo and Redo Operations

Text editors maintain a history of operations.

Suppose you perform:

- Type A
- Type B
- Type C

When you press Undo, the most recent operation is reversed first.

This follows LIFO.

### 4.4 Function Calls and Recursion

Programming languages use a call stack to manage function calls.

Whenever a function is called, its execution information is maintained in a stack frame.

When the function finishes, its stack frame is removed.

### 4.5 Expression Evaluation

Stacks are used to evaluate mathematical expressions, particularly:

- Infix expressions
- Prefix expressions
- Postfix expressions

### 4.6 Parentheses Matching

Stacks help determine whether brackets are properly balanced.

Example:

```text
{[()]}
```

This expression is balanced.

But:

```text
{[(])}
```

This expression is not balanced.

We will implement this problem in Part 2.

## 5. Basic Terminology of Stack

Before implementing a stack, understand the following terminology.

### 5.1 TOP

TOP is a variable that indicates the position of the topmost element in the stack.

In an array-based stack, TOP stores an index.

Initially:

```java
int top = -1;
```

Why -1?

Because array indexing starts from 0.

When the stack is empty, no valid array index represents the top element.

Therefore, -1 is used to indicate an empty stack.

### 5.2 Capacity

Capacity represents the maximum number of elements a stack can store.

For example:

```java
int capacity = 5;
```

This stack can store a maximum of five elements.

### 5.3 Size

Size represents the current number of elements present in the stack.

For an array-based stack:

```java
int size = top + 1;
```

Example:

If top = 2, then:

Size = 2 + 1 = 3

There are three elements in the stack.

### 5.4 Empty Stack

A stack is empty when:

```java
top == -1
```

### 5.5 Full Stack

For an array-based stack, a stack is full when:

```java
top == capacity - 1
```

For capacity = 5:

```java
top == 4
```

The array indices are:

0, 1, 2, 3, 4

Therefore, index 4 is the last available position.

## 6. Fundamental Stack Operations

The four fundamental operations are:

| Operation | Description                                 |
| --------- | ------------------------------------------- |
| PUSH      | Inserts an element into the stack           |
| POP       | Removes the top element                     |
| PEEK      | Returns the top element without removing it |
| isEmpty() | Checks whether the stack is empty           |

Another useful operation is:

isFull(): Checks whether an array-based stack has reached its maximum capacity.

Let's understand each operation in detail.

## 7. PUSH Operation

PUSH is used to insert a new element into the stack.

Whenever an element is inserted, it becomes the new TOP element.

**Algorithm for PUSH**

Assume:

- Stack is implemented using an array.
- TOP initially equals -1.
- Capacity represents the maximum array size.

**Algorithm:**

```text
PUSH(value)
```

1. Check whether TOP == capacity - 1.
2. If true:
   Display "Stack Overflow".
3. Otherwise:
   TOP = TOP + 1.
   stack[TOP] = value.
4. Stop.

Example

Consider an empty stack with capacity 5.

Initially:

```text
top = -1
```

Insert 10.

```text
top = top + 1
```

top = 0

stack[0] = 10
Stack:

```text
TOP
|
+-----+
| 10 |
+-----+
```

Now insert 20.

```text
top = 1
stack[1] = 20
```

Stack:

```text
TOP
|
+-----+
| 20 |
+-----+
| 10 |
+-----+
```

Java Code for PUSH

```java
public void push(int value) {

    if (top == capacity - 1) {

        System.out.println("Stack Overflow");

    } else {

        top++;

        stack[top] = value;

        System.out.println(value + " inserted");
    }

}
```

**Explanation**

1. First, we check whether the stack is full.
2. If it is full, insertion is not possible.
3. Otherwise, increment TOP.
4. Insert the new value at the TOP position.

Time Complexity: O(1)

Only a constant number of operations are performed.

## 8. POP Operation

POP removes the topmost element from the stack.

After removal, the element immediately below it becomes the new TOP.

**Algorithm for POP**

```text
POP()

1. Check whether TOP == -1.
2. If true:
   Display "Stack Underflow".
3. Otherwise:
   value = stack[TOP].
   TOP = TOP - 1.
   Return value.
4. Stop.
```

Example

Consider:

```text
TOP
|
+------+
| 30 |
+------+
| 20 |
+------+
| 10 |
+------+
```

Perform POP.

The value 30 is removed.

New stack:

```text
TOP
|
+------+
| 20 |
+------+
| 10 |
+------+
```

Now TOP points to 20.

Java Code for POP

```java
public int pop() {

    if (top == -1) {

        System.out.println("Stack Underflow");

        return -1;

    } else {

        int value = stack[top];

        top--;

        return value;
    }

}
```

**Explanation**

1. Check whether the stack is empty.
2. If empty, deletion is impossible.
3. Otherwise, store the top element in a temporary variable.
4. Decrease TOP by one.
5. Return the removed value.

**Important**: In this implementation, -1 is used as an error indicator. A production implementation should use a separate error mechanism or an appropriate exception because -1 could also be a valid stack element.

Time Complexity: O(1)

## 9. PEEK Operation

PEEK returns the topmost element without removing it.

This is different from POP.

**Difference Between POP and PEEK**

| POP                     | PEEK                            |
| ----------------------- | ------------------------------- |
| Removes the top element | Does not remove the top element |
| Changes TOP             | Does not change TOP             |
| Returns removed element | Returns current top element     |

Example

Stack:

```text
TOP
|
+------+
| 50 |
+------+
| 40 |
+------+
| 30 |
+------+
```

Perform:

```java
peek();
```

Output:

```text
Top element: 50
```

The stack remains unchanged.

**Java Code**

```java
public int peek() {

    if (top == -1) {

        System.out.println("Stack is Empty");

        return -1;

    } else {

        return stack[top];
    }

}
```

Time Complexity: O(1)

## 10. isEmpty() and isFull()

    isEmpty()
    Checks whether the stack contains zero elements.

Condition:

```java
top == -1
```

**Java implementation:**

```java
public boolean isEmpty() {

    return top == -1;

}
```

isFull()
Checks whether the stack has reached its maximum capacity.

Condition:

```java
top == capacity - 1
```

**Java implementation:**

```java
public boolean isFull() {

    return top == capacity - 1;

}
```

Both operations take O(1) time.

## 11. Stack Implementation Using Arrays

A stack can be implemented using different data structures.

Two common implementations are:

1. Array
2. Linked List

In this section, we will implement a stack using an array.

**Why Use an Array?**
An array stores elements in contiguous indexed positions.

For example:

```java
int[] stack = new int[5];
```

This creates an integer array that can hold five elements.

Its indices are:

```text
Index: 0 1 2 3 4
Value: \_ \_ \_ \_ \_
```

We use another variable called TOP to track the last inserted element.

Initially:

java
int top = -1;
Suppose we insert 10, 20, and 30.

The array becomes:

text
Index: 0 1 2 3 4
Value: 10 20 30 \_ \_
|
TOP
TOP = 2.

The actual stack is logically viewed from the top:

text
TOP
|
+------+
| 30 |
+------+
| 20 |
+------+
| 10 |
+------+
Remember:

The array stores elements from index 0 upward, but the logical TOP is the most recently inserted element.

12. Complete Java Program for Stack
    The following program implements a stack using an array.

It supports:

PUSH
POP
PEEK
isEmpty
isFull
Display
Java Code

java
class Stack {

    int capacity;
    int top;
    int[] stack;

    // Constructor
    Stack(int size) {

        capacity = size;

        stack = new int[capacity];

        top = -1;
    }

    // PUSH operation
    void push(int value) {

        if (top == capacity - 1) {

            System.out.println("Stack Overflow");

        } else {

            top++;

            stack[top] = value;

            System.out.println(value + " pushed into stack");
        }
    }

    // POP operation
    int pop() {

        if (top == -1) {

            System.out.println("Stack Underflow");

            return -1;

        } else {

            int removed = stack[top];

            top--;

            return removed;
        }
    }

    // PEEK operation
    int peek() {

        if (top == -1) {

            System.out.println("Stack is Empty");

            return -1;

        } else {

            return stack[top];
        }
    }

    // Check whether stack is empty
    boolean isEmpty() {

        return top == -1;
    }

    // Check whether stack is full
    boolean isFull() {

        return top == capacity - 1;
    }

    // Display stack elements
    void display() {

        if (isEmpty()) {

            System.out.println("Stack is Empty");

        } else {

            System.out.println("Stack Elements:");

            for (int i = top; i >= 0; i--) {

                System.out.println(stack[i]);
            }
        }
    }

}

// Main class
public class Main {

    public static void main(String[] args) {

        Stack s = new Stack(5);

        s.push(10);

        s.push(20);

        s.push(30);

        s.push(40);

        s.display();

        System.out.println("Top Element: " + s.peek());

        System.out.println("Removed: " + s.pop());

        s.display();

        System.out.println("Is Empty: " + s.isEmpty());

        System.out.println("Is Full: " + s.isFull());
    }

} 13. Detailed Code Explanation
Let's understand the important sections.

Step 1: Declare Stack Variables

java
int capacity;
int top;
int[] stack;
capacity stores the maximum size.
top tracks the index of the top element.
stack stores the actual elements.
Step 2: Constructor

java
Stack(int size) {

    capacity = size;

    stack = new int[capacity];

    top = -1;

}
When the object is created, the constructor initializes the array and sets TOP to -1.

For:

java
Stack s = new Stack(5);
The array size becomes 5.

Step 3: PUSH

java
top++;

stack[top] = value;
TOP is incremented first, and then the value is inserted.

Step 4: POP

java
int removed = stack[top];

top--;
The current top element is saved, and TOP is decreased.

Step 5: PEEK

java
return stack[top];
Returns the topmost element without modifying TOP.

Step 6: DISPLAY

java
for (int i = top; i >= 0; i--) {

    System.out.println(stack[i]);

}
The loop starts at TOP and moves towards index 0.

This displays the elements from top to bottom.

14. Step-by-Step Dry Run
    Consider:

java
Stack s = new Stack(5);

s.push(10);
s.push(20);
s.push(30);
s.push(40);

s.pop();

s.push(50);

s.peek();
Let's execute each operation.

Operation TOP Stack (Bottom → Top)
Initial -1 Empty
PUSH(10) 0 10
PUSH(20) 1 10, 20
PUSH(30) 2 10, 20, 30
PUSH(40) 3 10, 20, 30, 40
POP() 2 10, 20, 30
PUSH(50) 3 10, 20, 30, 50
PEEK() 3 10, 20, 30, 50
Final TOP element:

text
50
Notice that POP removes 40, and the next PUSH inserts 50 into the newly available logical top position.

15. Stack Overflow
    Stack Overflow occurs when we try to insert an element into a stack that is already full.

Suppose:

java
Stack s = new Stack(3);
The stack can store only three elements.

Perform:

java
s.push(10);
s.push(20);
s.push(30);
s.push(40);
After inserting the first three elements:

text
TOP
|
+------+
| 30 |
+------+
| 20 |
+------+
| 10 |
+------+
Now we attempt to insert 40.

But the stack is full.

Output:

text
Stack Overflow
Condition:

java
top == capacity - 1
Exam Point: Stack Overflow means attempting insertion beyond the available capacity of a bounded stack.

16. Stack Underflow
    Stack Underflow occurs when we try to remove an element from an empty stack.

Example:

java
Stack s = new Stack(5);

s.pop();
Initially:

java
top = -1;
There are no elements to remove.

Output:

text
Stack Underflow
Condition:

java
top == -1
Exam Point: Stack Underflow occurs when a POP operation is performed on an empty stack.

17. Time and Space Complexity
    For an array-based stack:

Operation Time Complexity
PUSH O(1)
POP O(1)
PEEK O(1)
isEmpty O(1)
isFull O(1)
DISPLAY O(n)
Why Is PUSH O(1)?
Because insertion requires only:

Incrementing TOP.
Assigning a value at the TOP index.
These operations take constant time.

Why Is POP O(1)?
Because deletion requires only:

Accessing the TOP element.
Decreasing TOP.
No shifting of elements is required.

Space Complexity
For an array of capacity n:

java
int[] stack = new int[n];
Space complexity is:

O(n)

Because memory is allocated for n elements.

18. Advantages and Disadvantages of Stack
    Advantages
    Simple to implement.
    PUSH and POP operations take O(1) time.
    Useful in recursion and function calls.
    Useful for expression evaluation.
    Helps implement undo and redo functionality.
    Useful for reversing data.
    Disadvantages
    Array-based stacks have fixed capacity.
    Overflow can occur when the array becomes full.
    Only the topmost element can be directly accessed.
    Searching for an arbitrary element requires O(n) time.
    Accessing middle elements is not directly supported.
19. Applications of Stack in Computer Science
    Important applications include:

Application How Stack Is Used
Recursion Maintains function call information
Function calls Uses the call stack
Undo operation Reverses the most recent operation
Browser history Supports backward navigation
Expression evaluation Evaluates postfix and prefix expressions
Parentheses matching Checks balanced brackets
String reversal Stores characters and retrieves them in reverse order
Backtracking Stores previous states or decisions
Depth First Search Uses a stack to explore graph nodes

21. Quick Revision Notes

Concept Important Point
Stack Linear data structure
Principle LIFO
Insertion PUSH
Deletion POP
Top access PEEK
Empty condition TOP = -1
Full condition TOP = capacity - 1
PUSH complexity O(1)
POP complexity O(1)
PEEK complexity O(1)
Overflow Insertion into a full bounded stack
Underflow Deletion from an empty stack
Array implementation Uses an array and TOP variable
Recursion Uses call stack 22. Final Concept Check
Before moving to Part 2, make sure you can answer these questions:

What is a Stack, and why is it called a linear data structure?
Explain LIFO with an example.
What is the purpose of the TOP variable?
Why is TOP initialized to -1?
Explain PUSH and POP with algorithms.
Differentiate between POP and PEEK.
What is Stack Overflow?
What is Stack Underflow?
Explain array-based Stack implementation in Java.
What is the time complexity of PUSH, POP, and PEEK?
Why are recursive function calls managed using a stack?
Write a complete Java program to implement a Stack.

---

# PART 2 – ADVANCED STACK CONCEPTS

2. STACK IMPLEMENTATION USING LINKED LIST
   A Stack can also be implemented using a Linked List.

A Linked List is a linear data structure consisting of nodes.

Each node contains:

Data
Reference to the next node
Unlike arrays, linked-list nodes do not need to occupy contiguous memory locations.

Basic Node Structure

```text
+---------+---------+
|  DATA   |  NEXT   |
+---------+---------+

```

For example, consider these elements:

10, 20, 30

They can be represented as:

```text
HEAD
 |
 v
+------+-----+    +------+-----+    +------+------+
|  10  |  o-----> |  20  |  o-----> |  30  | NULL |
+------+-----+    +------+-----+    +------+------+
```

For a stack, we will maintain a reference called TOP.

We can use the beginning of the linked list as TOP.

Our representation will be:

```text
TOP
TOP
 |
 v
+------+-----+    +------+-----+    +------+------+
|  30  |  o-----> |  20  |  o-----> |  10  | NULL |
+------+-----+    +------+-----+    +------+------+
```

The newest element is always inserted at the beginning.

Therefore, the first node is always the TOP element.

3. UNDERSTANDING NODE STRUCTURE
   Let's create a Node class in Java.

java
class Node {

    int data;

    Node next;

    Node(int value) {

        data = value;

        next = null;
    }

}
Explanation

1. int data

Stores the actual value of the node.

2. Node next

Stores the reference of the next node.

3. Constructor

java
Node(int value)
Initializes the node with a value.

Initially, next is null because the new node is not connected to another node.

Example:

java
Node n = new Node(10);
Representation:

```text
+------+------+
|  10  | NULL |
+------+------+
```

4. PUSH OPERATION USING LINKED LIST
   In an array-based stack, PUSH inserts an element at the next available array position.

In a linked-list stack, PUSH inserts a new node at the beginning of the linked list.

Algorithm

text
PUSH(value)

1. Create a new node.
2. Store value in the new node.
3. Set newNode.next = TOP.
4. Set TOP = newNode.
5. Stop.
   Example
   Initially:

text
TOP = NULL
Insert 10.

```text
TOP
 |
 v
+------+------+
|  10  | NULL |
+------+------+
```

Insert 20.

First:

java
newNode.next = top;
Then:

java
top = newNode;
Result:

```text
TOP
TOP
 |
 v
+------+-----+    +------+------+
|  20  |  o-----> |  10  | NULL |
+------+-----+    +------+------+
```

Insert 30.

Result:

```text
TOP
 |
 v
+------+-----+    +------+-----+    +------+------+
|  30  |  o-----> |  20  |  o-----> |  10  | NULL |
+------+-----+    +------+-----+    +------+------+
```

The newest element is always at the beginning.

Java Code

java
void push(int value) {

    Node newNode = new Node(value);

    newNode.next = top;

    top = newNode;

    System.out.println(value + " pushed into stack");

}
Time Complexity
O(1)

Only a constant number of pointer operations are performed.

5. POP OPERATION USING LINKED LIST
   POP removes the first node of the linked list because the first node represents TOP.

Algorithm

text
POP()

1. Check whether TOP == NULL.
2. If true:
   Display Stack Underflow.
3. Otherwise:
   Store TOP.data.
   Move TOP to TOP.next.
   Return the removed value.
4. Stop.

Example

Current stack:

```text
TOP
 |
 v
+------+-----+    +------+-----+    +------+------+
|  30  |  o-----> |  20  |  o-----> |  10  | NULL |
+------+-----+    +------+-----+    +------+------+
```

Perform POP.

The element 30 is removed.

Now TOP moves to the next node.

text
TOP
|
v
+------+-----+ +------+------+
| 20 | o-----> | 10 | NULL |
+------+-----+ +------+------+
Java Code

java
int pop() {

    if (top == null) {

        System.out.println("Stack Underflow");

        return -1;
    }

    int removed = top.data;

    top = top.next;

    return removed;

}
Explanation

java
int removed = top.data;
Stores the current top value.

java
top = top.next;
Moves TOP to the next node.

The old node is no longer reachable through the stack and can be reclaimed by Java's garbage collector when eligible.

Time Complexity
O(1)

6. PEEK AND DISPLAY OPERATIONS
   6.1 PEEK
   PEEK returns the data stored in the TOP node without deleting it.

java
int peek() {

    if (top == null) {

        System.out.println("Stack is Empty");

        return -1;
    }

    return top.data;

}
Time Complexity: O(1)

6.2 isEmpty()

java
boolean isEmpty() {

    return top == null;

}
If TOP is null, the stack is empty.

Time Complexity: O(1)

6.3 DISPLAY
Display prints all elements starting from TOP.

java
void display() {

    if (top == null) {

        System.out.println("Stack is Empty");

        return;
    }

    Node temp = top;

    while (temp != null) {

        System.out.println(temp.data);

        temp = temp.next;
    }

}
Time Complexity: O(n)

7. COMPLETE JAVA PROGRAM – LINKED LIST STACK

java
class Node {

    int data;

    Node next;

    Node(int value) {

        data = value;

        next = null;
    }

}

class Stack {

    Node top;

    // PUSH
    void push(int value) {

        Node newNode = new Node(value);

        newNode.next = top;

        top = newNode;

        System.out.println(value + " pushed");
    }

    // POP
    int pop() {

        if (top == null) {

            System.out.println("Stack Underflow");

            return -1;
        }

        int removed = top.data;

        top = top.next;

        return removed;
    }

    // PEEK
    int peek() {

        if (top == null) {

            System.out.println("Stack Empty");

            return -1;
        }

        return top.data;
    }

    // isEmpty
    boolean isEmpty() {

        return top == null;
    }

    // DISPLAY
    void display() {

        Node temp = top;

        if (temp == null) {

            System.out.println("Stack Empty");

            return;
        }

        System.out.println("Stack Elements:");

        while (temp != null) {

            System.out.println(temp.data);

            temp = temp.next;
        }
    }

}

public class Main {

    public static void main(String[] args) {

        Stack s = new Stack();

        s.push(10);

        s.push(20);

        s.push(30);

        s.push(40);

        s.display();

        System.out.println("Top: " + s.peek());

        System.out.println("Removed: " + s.pop());

        s.display();
    }

}
Expected Output

text
10 pushed
20 pushed
30 pushed
40 pushed
Stack Elements:
40
30
20
10
Top: 40
Removed: 40
Stack Elements:
30
20
10 8. ARRAY STACK VS LINKED LIST STACK
This is an important conceptual comparison.

Feature Array Stack Linked List Stack
Memory allocation Fixed array allocation Dynamic node allocation
Size Fixed capacity Grows and shrinks dynamically
Memory layout Contiguous array positions Nodes can be scattered
PUSH O(1) O(1)
POP O(1) O(1)
PEEK O(1) O(1)
Overflow When array is full When memory allocation fails
Extra memory No pointer per element Stores a next reference per node
Implementation Relatively simple Requires pointer/reference management
Which Is Better?
Use an array when:

Maximum capacity is known.
Memory overhead should be low.
Simple implementation is preferred.
Use a Linked List when:

The number of elements is unpredictable.
Dynamic growth is required.
Avoiding a fixed capacity is important.
Remember: A linked-list stack is not unlimited. It is still restricted by available memory and runtime limits.

9. REVERSING A STRING USING STACK
   This is one of the simplest and most useful applications of Stack.

Problem
Given a string:

text
HELLO
Reverse it using a stack.

Expected result:

text
OLLEH
Why Does Stack Work?
Because Stack follows LIFO.

If we insert each character into the stack, the last character inserted will be removed first.

Step-by-Step Example
Original string:

text
H E L L O
PUSH characters:

text
PUSH(H)
PUSH(E)
PUSH(L)
PUSH(L)
PUSH(O)
Stack:

text
TOP
|
v
O
L
L
E
H
Now POP all characters.

text
O L L E H
Reversed string:

text
OLLEH
Java Program

java
import java.util.Stack;

public class Main {

    public static void main(String[] args) {

        String str = "HELLO";

        Stack<Character> stack = new Stack<>();

        for (int i = 0; i < str.length(); i++) {

            stack.push(str.charAt(i));
        }

        String reversed = "";

        while (!stack.isEmpty()) {

            reversed += stack.pop();
        }

        System.out.println("Original: " + str);

        System.out.println("Reversed: " + reversed);
    }

}
Output

text
Original: HELLO
Reversed: OLLEH
Code Explanation

java
stack.push(str.charAt(i));
Extracts each character and pushes it onto the stack.

java
while (!stack.isEmpty())
Continues until all characters have been removed.

java
reversed += stack.pop();
Appends the removed characters in reverse order.

Complexity Note: The stack operations take O(n) time overall. However, repeated string concatenation can make this particular Java implementation O(n²). A StringBuilder can make the overall solution O(n).

Improved Java Implementation

java
import java.util.Stack;

public class Main {

    public static void main(String[] args) {

        String str = "HELLO";

        Stack<Character> stack = new Stack<>();

        for (char ch : str.toCharArray()) {

            stack.push(ch);
        }

        StringBuilder reversed = new StringBuilder();

        while (!stack.isEmpty()) {

            reversed.append(stack.pop());
        }

        System.out.println(reversed);
    }

}
Time Complexity: O(n)

Space Complexity: O(n)

10. BALANCED PARENTHESES PROBLEM
    This is a very important Stack application.

Problem Statement
Given an expression containing brackets, determine whether all brackets are properly balanced.

Common brackets:

()
{}
[]
What Does Balanced Mean?
An expression is balanced when:

Every opening bracket has a corresponding closing bracket.
Opening and closing brackets are of the same type.
Brackets close in the correct order.
Examples

Expression Result
() Balanced
{} Balanced
[] Balanced
{[]} Balanced
{[()]} Balanced
(] Not Balanced
([)] Not Balanced
(( Not Balanced
)) Not Balanced
Why Use Stack?
Whenever we encounter an opening bracket, we push it onto the stack.

Whenever we encounter a closing bracket, we check whether it matches the top opening bracket.

If it matches, we POP.

At the end, the stack must be empty.

Algorithm

text
BALANCED(expression)

1. Create an empty stack.

2. Traverse each character.

3. If character is an opening bracket:
   PUSH it.

4. If character is a closing bracket:
   If stack is empty:
   Return false.

   POP the top opening bracket.

   If it does not match:
   Return false.

5. After traversal:
   If stack is empty:
   Return true.
   Otherwise:
   Return false.
   Dry Run
   Expression:

text
{[()]}

Character Operation Stack
{ PUSH {
[ PUSH { [
( PUSH { [ (
) POP { [
] POP {
} POP Empty
Since the stack is empty at the end:

Result = Balanced.

Java Program

java
import java.util.Stack;

public class Main {

    static boolean isBalanced(String expression) {

        Stack<Character> stack = new Stack<>();

        for (char ch : expression.toCharArray()) {

            if (ch == '(' || ch == '{' || ch == '[') {

                stack.push(ch);
            }

            else if (ch == ')' || ch == '}' || ch == ']') {

                if (stack.isEmpty()) {

                    return false;
                }

                char top = stack.pop();

                if (ch == ')' && top != '(') {

                    return false;
                }

                if (ch == '}' && top != '{') {

                    return false;
                }

                if (ch == ']' && top != '[') {

                    return false;
                }
            }
        }

        return stack.isEmpty();
    }

    public static void main(String[] args) {

        String expression = "{[()]}";

        if (isBalanced(expression)) {

            System.out.println("Balanced Expression");

        } else {

            System.out.println("Not Balanced");
        }
    }

}
Output

text
Balanced Expression
Complexity
Time Complexity: O(n)

Space Complexity: O(n)

Each character is processed once, and in the worst case all opening brackets are stored.

Important: This implementation assumes the input contains only brackets. In a full mathematical-expression parser, other characters are handled separately.

11. INFIX, PREFIX AND POSTFIX EXPRESSIONS
    Before understanding expression conversion, we must understand three different expression notations.

They are:

Infix
Prefix
Postfix
These notations describe how operators and operands are arranged.

What Is an Operand?
An operand is a value or variable on which an operation is performed.

Examples:

text
A, B, C, 10, 20, 50
What Is an Operator?
An operator performs an operation on operands.

Examples:

text
+, -, \*, /, %
Consider:

text
A + B
A and B are operands.

is an operator. 12. INFIX EXPRESSION
In an Infix expression, the operator is placed between the operands.

This is the notation commonly used in mathematics.

Examples:

text
A + B
A - B
A _ B
A + B _ C
Example:

text
10 + 20
The + operator appears between 10 and 20.

Therefore, it is an Infix expression.

Advantages
Easy for humans to understand.
Commonly used in mathematical expressions.
Familiar programming syntax.
Disadvantage
Computers must consider operator precedence and parentheses to evaluate infix expressions correctly.

13. PREFIX EXPRESSION
    In a Prefix expression, the operator is placed before its operands.

It is also called Polish Notation.

Example:

Infix:

text
A + B
Prefix:

text

- A B
  Another example:

Infix:

text
A \* B
Prefix:

text

- A B
  Complex example:

Infix:

text
A + B \* C
Prefix:

text

- A \* B C
  Why?

Because multiplication has higher precedence than addition.

First calculate B \* C.

Then add A.

14. POSTFIX EXPRESSION
    In a Postfix expression, the operator appears after its operands.

It is also called Reverse Polish Notation.

Example:

Infix:

text
A + B
Postfix:

text
A B +
Another example:

Infix:

text
A \* B
Postfix:

text
A B \*
Complex example:

Infix:

text
A + B \* C
Postfix:

text
A B C \* +
Multiplication is performed first, followed by addition.

Comparison

Infix Prefix Postfix
A + B + A B A B +
A - B - A B A B -
A _ B _ A B A B _
A + B _ C + A _ B C A B C _ +
(A+B)_C _ + A B C A B + C \*
Important Exam Point
Infix: Operator between operands.
Prefix: Operator before operands.
Postfix: Operator after operands. 15. OPERATOR PRECEDENCE AND ASSOCIATIVITY
To convert expressions correctly, we must understand precedence and associativity.

Operator Precedence
Precedence determines which operator is evaluated first.

Common precedence order:

Operator Meaning Precedence
^ Exponentiation Highest
\*, /, % Multiplication, division, modulus Next
+, - Addition, subtraction Lowest
For conventional mathematical notation:

text
A + B \* C
Multiplication is performed first.

Therefore:

text
A + (B \* C)
Associativity
Associativity determines the order in which operators of the same precedence are evaluated.

Examples:

Left Associativity

Most arithmetic operators such as +, -, \*, / are left-associative.

Example:

text
A - B - C
Means:

text
(A - B) - C
Right Associativity

Exponentiation is conventionally right-associative.

Example:

text
A ^ B ^ C
Means:

text
A ^ (B ^ C)
Note: Java's ^ operator means bitwise XOR, not exponentiation. The mathematical exponentiation notation is being used here to explain expression conversion.

16. INFIX TO POSTFIX CONVERSION
    This is one of the most important Stack applications.

Problem
Convert the following Infix expression into Postfix:

text
A + B \* C
Rules
If the character is an operand, add it directly to the output.
If the character is an opening parenthesis, PUSH it.
If the character is a closing parenthesis, POP until the matching opening parenthesis is found.
If the character is an operator, compare its precedence with the operator at the top of the stack.
POP operators of higher precedence, and operators of equal precedence when the incoming operator is left-associative.
PUSH the incoming operator.
After traversal, POP all remaining operators.
Example
Expression:

text
A + B \* C
Step-by-step:

Character Action Output Stack
A Add operand A Empty

- PUSH A +
  B Add operand AB +

* PUSH (higher precedence) AB + _
  C Add operand ABC + _
  End POP _ ABC_ +
  End POP + ABC\*+ Empty
  Final Postfix:

text
ABC\*+
Another Example
Infix:

text
(A + B) \* C
Conversion:

Character Action Output Stack
( PUSH Empty (
A Output A (

- PUSH A ( +
  B Output AB ( +
  ) POP until ( AB+ Empty

* PUSH AB+ _
  C Output AB+C _
  End POP AB+C\* Empty
  Final Postfix:

text
AB+C* 17. JAVA PROGRAM – INFIX TO POSTFIX
The following program supports single-character operands and the operators +, -, *, /, %, and ^, with parentheses.

It assumes valid expressions without spaces and uses conventional mathematical exponentiation precedence.

java
import java.util.Stack;

public class Main {

    static int precedence(char op) {

        switch (op) {

            case '^':
                return 3;

            case '*':
            case '/':
            case '%':
                return 2;

            case '+':
            case '-':
                return 1;
        }

        return -1;
    }

    static boolean isOperator(char ch) {

        return ch == '+' ||
               ch == '-' ||
               ch == '*' ||
               ch == '/' ||
               ch == '%' ||
               ch == '^';
    }

    static String infixToPostfix(String expression) {

        Stack<Character> stack = new Stack<>();

        StringBuilder result = new StringBuilder();

        for (char ch : expression.toCharArray()) {

            if (Character.isLetterOrDigit(ch)) {

                result.append(ch);
            }

            else if (ch == '(') {

                stack.push(ch);
            }

            else if (ch == ')') {

                while (!stack.isEmpty() &&
                       stack.peek() != '(') {

                    result.append(stack.pop());
                }

                if (stack.isEmpty()) {
                    throw new IllegalArgumentException(
                        "Mismatched parentheses"
                    );
                }

                stack.pop();
            }

            else if (isOperator(ch)) {

                while (!stack.isEmpty() &&
                       stack.peek() != '(' &&
                       (precedence(stack.peek()) >
                        precedence(ch) ||
                        (precedence(stack.peek()) ==
                         precedence(ch) && ch != '^'))) {

                    result.append(stack.pop());
                }

                stack.push(ch);
            }

            else {

                throw new IllegalArgumentException(
                    "Invalid character: " + ch
                );
            }
        }

        while (!stack.isEmpty()) {

            if (stack.peek() == '(') {
                throw new IllegalArgumentException(
                    "Mismatched parentheses"
                );
            }

            result.append(stack.pop());
        }

        return result.toString();
    }

    public static void main(String[] args) {

        String infix = "A+B*C";

        String postfix = infixToPostfix(infix);

        System.out.println("Infix: " + infix);

        System.out.println("Postfix: " + postfix);
    }

}
Output

text
Infix: A+B*C
Postfix: ABC*+
Important Limitation
This introductory implementation supports single-character operands. Multi-digit numbers, unary operators, and decimal numbers require tokenization and additional parsing logic.

18. POSTFIX EXPRESSION EVALUATION
    Postfix expressions can be evaluated efficiently using a Stack.

Example
Evaluate:

text
5 3 +
Result:

text
8
Why Use Stack?
When an operand appears, PUSH it onto the stack.

When an operator appears:

POP the second operand.
POP the first operand.
Perform the operation.
PUSH the result.
Important Operand Order
For subtraction and division, order matters.

Suppose the stack contains:

text
TOP
|
3
5
For the expression:

text
5 3 -
We must calculate:

text
5 - 3 = 2
Therefore:

java
int b = stack.pop();

int a = stack.pop();

int result = a - b;
Do not reverse the operands.

Dry Run
Expression:

text
5 3 2 \* +
This means:

text
5 + (3 \* 2)

Token Operation Stack
5 PUSH 5 5
3 PUSH 3 5, 3
2 PUSH 2 5, 3, 2

- POP 2 and 3; PUSH 6 5, 6

* POP 6 and 5; PUSH 11 11
  Final Answer:

text
11 19. JAVA PROGRAM – POSTFIX EVALUATION
This program evaluates postfix expressions containing non-negative single-digit integers and the operators +, -, \*, /, and %.

java
import java.util.Stack;

public class Main {

    static int evaluatePostfix(String expression) {

        Stack<Integer> stack = new Stack<>();

        for (char ch : expression.toCharArray()) {

            if (ch == ' ') {
                continue;
            }

            if (Character.isDigit(ch)) {

                stack.push(ch - '0');
            }

            else {

                if (stack.size() < 2) {

                    throw new IllegalArgumentException(
                        "Invalid postfix expression"
                    );
                }

                int b = stack.pop();

                int a = stack.pop();

                int result = 0;

                switch (ch) {

                    case '+':
                        result = a + b;
                        break;

                    case '-':
                        result = a - b;
                        break;

                    case '*':
                        result = a * b;
                        break;

                    case '/':
                        if (b == 0) {
                            throw new ArithmeticException(
                                "Division by zero"
                            );
                        }

                        result = a / b;
                        break;

                    case '%':
                        if (b == 0) {
                            throw new ArithmeticException(
                                "Division by zero"
                            );
                        }

                        result = a % b;
                        break;

                    default:
                        throw new IllegalArgumentException(
                            "Invalid operator: " + ch
                        );
                }

                stack.push(result);
            }
        }

        if (stack.size() != 1) {

            throw new IllegalArgumentException(
                "Invalid postfix expression"
            );
        }

        return stack.pop();
    }

    public static void main(String[] args) {

        String expression = "532*+";

        int result = evaluatePostfix(expression);

        System.out.println("Postfix: " + expression);

        System.out.println("Result: " + result);
    }

}
Output

text
Postfix: 532\*+
Result: 11
Explanation
The expression:

text
532\*+
Represents:

text
5 + (3 \* 2)
First:

text
3 \* 2 = 6
Then:

text
5 + 6 = 11
Complexity
Time Complexity: O(n)

Space Complexity: O(n)

Each token is processed once, and the stack can hold up to O(n) values.

20. APPLICATIONS OF STACK IN EXPRESSION PROCESSING
    Stack plays a major role in expression processing.

Application Purpose
Infix to Postfix Converts mathematical notation
Infix to Prefix Converts mathematical notation
Postfix Evaluation Calculates expression result
Prefix Evaluation Calculates prefix expression
Parentheses Matching Validates brackets
Syntax Parsing Helps analyze expressions
Compiler Design Used in parsing and evaluation
Important Differences

Expression Operator Position
Infix Between operands
Prefix Before operands
Postfix After operands 21. TIME AND SPACE COMPLEXITY

Operation / Problem Time Complexity Space Complexity
Linked List PUSH O(1) O(1) per insertion
Linked List POP O(1) O(1)
Linked List PEEK O(1) O(1)
Linked List DISPLAY O(n) O(1) auxiliary
String Reversal O(n) O(n)
Balanced Parentheses O(n) O(n)
Infix to Postfix O(n) O(n)
Postfix Evaluation O(n) O(n)

24. QUICK REVISION NOTES

Topic Important Concept
Linked List Stack Uses nodes and references
TOP Points to first node
PUSH Insert at beginning
POP Delete from beginning
PEEK Returns TOP data
Linked List PUSH O(1)
Linked List POP O(1)
String Reversal Uses LIFO
Balanced Parentheses Uses PUSH and POP
Infix Operator between operands
Prefix Operator before operands
Postfix Operator after operands
Polish Notation Prefix
Reverse Polish Notation Postfix
Infix to Postfix Uses operator stack
Postfix Evaluation Uses operand stack 25. FINAL CONCEPT CHECK

Before moving to Part 3, ensure you can answer the following:

1. How is Stack implemented using a Linked List?
2. Why is the first node used as TOP?
3. Explain PUSH and POP in Linked List implementation.
4. Compare Array Stack and Linked List Stack.
5. Write a Java program to reverse a string using Stack.
6. Explain the Balanced Parentheses problem.
7. Differentiate Infix, Prefix, and Postfix.
8. Explain operator precedence and associativity.
9. Convert A+B\*C into Postfix.
10. Convert (A+B)\*C into Postfix.
11. Explain Postfix expression evaluation.
12. a Java program to evaluate a Postfix expression.

# PART 3 – QUEUE FUNDAMENTALS

1. INTRODUCTION TO QUEUE
   A Queue is a linear data structure that follows the principle of FIFO (First In, First Out).

This means:

The element inserted first into the queue will be removed first.

Unlike Stack, where insertion and deletion happen at the same end, Queue generally performs:

Insertion at the REAR.
Deletion at the FRONT.
Real-Life Example: Ticket Counter
Imagine people standing in a line to purchase movie tickets.

Suppose three people arrive:

Person A
Person B
Person C
Their arrangement is:

```text
FRONT                         REAR
  |                             |
  v                             v
+-----+     +-----+     +-----+
|  A  | --> |  B  | --> |  C  |
+-----+     +-----+     +-----+

```

Person A arrived first, so Person A gets the ticket first.

Then Person B.

Then Person C.

This is exactly how a Queue works.

Important Definition
A Queue is a linear data structure in which insertion takes place at one end called REAR, while deletion takes place at the other end called FRONT.

2. UNDERSTANDING FIFO PRINCIPLE
   FIFO stands for:

First In, First Out.

The element inserted first is the first element to be removed.

Consider the following insertion sequence:

text
10, 20, 30, 40
Insertion order:

10 inserted first.
20 inserted second.
30 inserted third.
40 inserted fourth.
Queue representation:

```text
FRONT                         REAR
  |                             |
  v                             v
+------+    +------+    +------+    +------+
|  10  | -> |  20  | -> |  30  | -> |  40  |
+------+    +------+    +------+    +------+
```

Now perform DEQUEUE.

The first element, 10, will be removed.

Remaining Queue:

```text
FRONT                         REAR
  |                             |
  v                             v
+------+    +------+    +------+
|  20  | -> |  30  | -> |  40  |
+------+    +------+    +------+
```

Next DEQUEUE removes 20.

Therefore, the removal sequence is:

text
10 → 20 → 30 → 40
This demonstrates FIFO.

Important Exam Point
Stack follows LIFO.
Queue follows FIFO. 3. REAL-LIFE APPLICATIONS OF QUEUE
Queues are widely used in computer systems and everyday applications.

3.1 Ticket Booking Systems
Customers requesting tickets are generally served in arrival order.

3.2 Printer Queue
When multiple documents are sent to a printer, they are typically placed in a queue.

Example:

text
Document A
Document B
Document C
Document A is processed first.

3.3 CPU Scheduling
Operating systems use different scheduling algorithms involving queues.

For example, Round Robin scheduling maintains a ready queue of processes.

3.4 Call Center Systems
Incoming calls can be placed in a waiting queue until an agent becomes available.

3.5 Network Data Packets
Routers and network devices commonly use queues to buffer packets before transmission.

3.6 Breadth First Search (BFS)
Queue is used in graph and tree traversal algorithms.

BFS explores nodes level by level.

3.7 Keyboard and Input Buffers
Operating systems and applications may buffer input events for later processing.

4. DIFFERENCE BETWEEN STACK AND QUEUE
   This is an important conceptual topic.

Feature Stack Queue
Principle LIFO FIFO
Full Form Last In, First Out First In, First Out
Insertion PUSH ENQUEUE
Deletion POP DEQUEUE
Insertion End TOP REAR
Deletion End TOP FRONT
Access Topmost element Front element
Applications Recursion, Undo Scheduling, BFS
Visual Difference
Stack:

```text
       TOP
        |
     +------+
     |  30  |
     +------+
     |  20  |
     +------+
     |  10  |
     +------+
```

Queue:

```text
FRONT                         REAR
  |                             |
  v                             v
+------+    +------+    +------+
|  10  | -> |  20  | -> |  30  |
+------+    +------+    +------+
```

Important Exam Question
Which data structure follows FIFO?

Answer: Queue.

Which data structure follows LIFO?

Answer: Stack.

5. BASIC TERMINOLOGY OF QUEUE
   Before implementing a Queue, we must understand its important terminology.

5.1 FRONT
FRONT represents the position of the first element in the Queue.

The element at FRONT is the next element to be removed.

Example:

```text
FRONT
  |
  v
+----+----+----+
| 10 | 20 | 30 |
+----+----+----+
```

FRONT points to 10.

5.2 REAR
REAR represents the position of the last element in the Queue.

New elements are inserted at REAR.

Example:

```text
          REAR
            |
            v
+----+----+----+
| 10 | 20 | 30 |
+----+----+----+
```

REAR points to 30.

5.3 Capacity
Capacity represents the maximum number of elements a bounded Queue can store.

Example:

java
int capacity = 5;
5.4 Size
Size represents the current number of elements present in the Queue.

For a conventional linear array Queue with valid FRONT and REAR indices:

java
int size = rear - front + 1;
This formula applies when the Queue is non-empty and no circular wrapping is involved.

5.5 Empty Queue
In the implementation we will develop, an empty Queue is represented by:

java
front == -1
Initially:

java
front = -1;
rear = -1;
5.6 Full Queue
For a linear array Queue:

java
rear == capacity - 1
This means REAR has reached the final array position.

6. FRONT AND REAR VARIABLES
   A Queue uses two variables:

FRONT
REAR
Initially:

java
front = -1;
rear = -1;
Suppose the array capacity is 5.

Initially:

text
Index: 0 1 2 3 4
Value: \_ \_ \_ \_ \_

front = -1
rear = -1
Insert 10.

text
Index: 0 1 2 3 4
Value: 10 \_ \_ \_ \_

front = 0
rear = 0
Insert 20.

text
Index: 0 1 2 3 4
Value: 10 20 \_ \_ \_

front = 0
rear = 1
Insert 30.

text
Index: 0 1 2 3 4
Value: 10 20 30 \_ \_

front = 0
rear = 2
Now the Queue looks like:

```text
FRONT                   REAR
  |                       |
  v                       v
+------+    +------+    +------+
|  10  | -> |  20  | -> |  30  |
+------+    +------+    +------+
```

Remember:

FRONT is used for deletion.

REAR is used for insertion.

7. FUNDAMENTAL QUEUE OPERATIONS
   The main Queue operations are:

Operation Purpose
ENQUEUE Inserts an element
DEQUEUE Removes the front element
PEEK Returns the front element
isEmpty() Checks whether Queue is empty
isFull() Checks whether Queue is full
DISPLAY Displays all elements
Let's understand each operation.

8. ENQUEUE OPERATION
   ENQUEUE is used to insert a new element into a Queue.

Insertion always takes place at REAR.

Algorithm

text
ENQUEUE(value)

1. Check whether REAR == capacity - 1.

2. If true:
   Display Queue Overflow.

3. Otherwise:

   If FRONT == -1:
   FRONT = 0.

   REAR = REAR + 1.

   QUEUE[REAR] = value.

4. Stop.
   Example
   Initially:

text
front = -1
rear = -1
Insert 10.

Since the Queue is empty:

text
front = 0
rear = 0
Insert 20.

text
rear = rear + 1
rear = 1
Insert 20 at index 1.

Queue:

```text
FRONT              REAR
  |                   |
  v                   v
+------+           +------+
|  10  |           |  20  |
+------+           +------+
```

Insert 30.

```text
FRONT                         REAR
  |                             |
  v                             v
+------+    +------+    +------+
|  10  | -> |  20  | -> |  30  |
+------+    +------+    +------+
```

Java Code

java
void enqueue(int value) {

    if (rear == capacity - 1) {

        System.out.println("Queue Overflow");

    } else {

        if (front == -1) {

            front = 0;
        }

        rear++;

        queue[rear] = value;

        System.out.println(value + " inserted");
    }

}
Explanation
First, we check whether REAR has reached the last array index.

If the Queue is not full:

Initialize FRONT if this is the first insertion.
Increment REAR.
Insert the value at queue[rear].
Time Complexity
O(1)

9. DEQUEUE OPERATION
   DEQUEUE removes the element at FRONT.

The first inserted element is removed first.

Algorithm

text
DEQUEUE()

1. Check whether FRONT == -1.

2. If true:
   Display Queue Underflow.

3. Otherwise:

   VALUE = QUEUE[FRONT].

   If FRONT == REAR:
   FRONT = -1.
   REAR = -1.

   Otherwise:
   FRONT = FRONT + 1.

4. Return VALUE.

5. Stop.
   Example
   Current Queue:

```text
FRONT                         REAR
  |                             |
  v                             v
+------+    +------+    +------+
|  10  | -> |  20  | -> |  30  |
+------+    +------+    +------+
```

Perform DEQUEUE.

10 is removed.

Now FRONT moves to the next position.

```text
FRONT              REAR
  |                   |
  v                   v
+------+           +------+
|  20  | ->        |  30  |
+------+           +------+
```

Java Code

java
int dequeue() {

    if (front == -1) {

        System.out.println("Queue Underflow");

        return -1;
    }

    int removed = queue[front];

    if (front == rear) {

        front = -1;

        rear = -1;

    } else {

        front++;
    }

    return removed;

}
Explanation
Check whether the Queue is empty.
Store the FRONT element.
If it was the only element, reset both FRONT and REAR.
Otherwise, increment FRONT.
Return the removed element.
Time Complexity
O(1)

10. PEEK OPERATION
    PEEK returns the element at FRONT without removing it.

Example:

```text
FRONT                         REAR
  |                             |
  v                             v
+------+    +------+    +------+
|  10  | -> |  20  | -> |  30  |
+------+    +------+    +------+
```

Perform:

java
peek();
Output:

text
Front Element: 10
The Queue remains unchanged.

Java Code

java
int peek() {

    if (front == -1) {

        System.out.println("Queue is Empty");

        return -1;
    }

    return queue[front];

}
Time Complexity: O(1)

11. isEmpty() AND isFull()
    isEmpty()
    Checks whether the Queue is empty.

java
boolean isEmpty() {

    return front == -1;

}
isFull()
Checks whether the linear array Queue is full.

java
boolean isFull() {

    return rear == capacity - 1;

}
Important:

This is the full condition for the linear Queue implementation, not the circular Queue implementation.

Both operations take O(1) time.

12. QUEUE IMPLEMENTATION USING ARRAYS
    Now let's implement the complete Queue using an array.

Step 1: Create Array

java
int[] queue = new int[5];
This creates an array with five positions.

Step 2: Declare FRONT and REAR

java
int front = -1;

int rear = -1;
Step 3: ENQUEUE
Insert elements at REAR.

Step 4: DEQUEUE
Remove elements from FRONT.

Step 5: DISPLAY
Print elements from FRONT to REAR.

13. COMPLETE JAVA PROGRAM – ARRAY QUEUE

java
class Queue {

    int capacity;

    int front;

    int rear;

    int[] queue;

    // Constructor
    Queue(int size) {

        capacity = size;

        queue = new int[capacity];

        front = -1;

        rear = -1;
    }

    // ENQUEUE
    void enqueue(int value) {

        if (isFull()) {

            System.out.println("Queue Overflow");

        } else {

            if (front == -1) {

                front = 0;
            }

            rear++;

            queue[rear] = value;

            System.out.println(value + " inserted");
        }
    }

    // DEQUEUE
    int dequeue() {

        if (isEmpty()) {

            System.out.println("Queue Underflow");

            return -1;
        }

        int removed = queue[front];

        if (front == rear) {

            front = -1;

            rear = -1;

        } else {

            front++;
        }

        return removed;
    }

    // PEEK
    int peek() {

        if (isEmpty()) {

            System.out.println("Queue is Empty");

            return -1;
        }

        return queue[front];
    }

    // Check Empty
    boolean isEmpty() {

        return front == -1;
    }

    // Check Full
    boolean isFull() {

        return rear == capacity - 1;
    }

    // DISPLAY
    void display() {

        if (isEmpty()) {

            System.out.println("Queue is Empty");

            return;
        }

        System.out.println("Queue Elements:");

        for (int i = front; i <= rear; i++) {

            System.out.println(queue[i]);
        }
    }

}

public class Main {

    public static void main(String[] args) {

        Queue q = new Queue(5);

        q.enqueue(10);

        q.enqueue(20);

        q.enqueue(30);

        q.enqueue(40);

        q.display();

        System.out.println("Front: " + q.peek());

        System.out.println("Removed: " + q.dequeue());

        q.display();

        System.out.println("Is Empty: " + q.isEmpty());

        System.out.println("Is Full: " + q.isFull());
    }

}
Expected Output

text
10 inserted
20 inserted
30 inserted
40 inserted
Queue Elements:
10
20
30
40
Front: 10
Removed: 10
Queue Elements:
20
30
40
Is Empty: false
Is Full: false 14. DETAILED CODE EXPLANATION
Step 1: Variables

java
int capacity;
int front;
int rear;
int[] queue;
capacity stores maximum capacity.
front tracks the first valid element.
rear tracks the last inserted element.
queue stores the values.
Step 2: Constructor

java
Queue(int size) {

    capacity = size;

    queue = new int[capacity];

    front = -1;

    rear = -1;

}
Initializes the array and both pointers.

Step 3: ENQUEUE

java
rear++;

queue[rear] = value;
Moves REAR forward and inserts the element.

Step 4: DEQUEUE

java
int removed = queue[front];

front++;
Retrieves the first element and advances FRONT.

Step 5: Resetting the Queue

java
if (front == rear) {

    front = -1;

    rear = -1;

}
This condition means only one element was present.

After removing it, the Queue becomes empty.

Therefore, both variables are reset.

15. STEP-BY-STEP DRY RUN
    Consider:

java
Queue q = new Queue(5);

q.enqueue(10);

q.enqueue(20);

q.enqueue(30);

q.dequeue();

q.enqueue(40);

q.enqueue(50);
Let's understand the execution.

Operation FRONT REAR Active Elements
Initial -1 -1 Empty
ENQUEUE(10) 0 0 10
ENQUEUE(20) 0 1 10, 20
ENQUEUE(30) 0 2 10, 20, 30
DEQUEUE() 1 2 20, 30
ENQUEUE(40) 1 3 20, 30, 40
ENQUEUE(50) 1 4 20, 30, 40, 50
Final Queue:

```text
FRONT                              REAR
  |                                  |
  v                                  v
+------+    +------+    +------+    +------+
|  20  | -> |  30  | -> |  40  | -> |  50  |
+------+    +------+    +------+    +------+
```

Notice that the position previously occupied by 10 is not reused.

This leads to an important limitation of a linear array Queue.

16. QUEUE OVERFLOW
    Queue Overflow occurs when an attempt is made to insert an element into a full bounded Queue.

Suppose:

java
Queue q = new Queue(3);
Insert:

java
q.enqueue(10);

q.enqueue(20);

q.enqueue(30);

q.enqueue(40);
After three insertions:

text
Index: 0 1 2
Value: 10 20 30

front = 0
rear = 2
The Queue is full.

Now inserting 40 produces:

text
Queue Overflow
Condition:

java
rear == capacity - 1
Remember: This is the overflow condition for the simple linear array Queue.

17. QUEUE UNDERFLOW
    Queue Underflow occurs when DEQUEUE is performed on an empty Queue.

Example:

java
Queue q = new Queue(5);

q.dequeue();
Initially:

java
front = -1;

rear = -1;
There is no element to remove.

Output:

text
Queue Underflow
Condition:

java
front == -1 18. LIMITATIONS OF LINEAR QUEUE
This is an extremely important topic because it explains why Circular Queue was introduced.

Consider a Queue of capacity 5.

Insert:

text
10, 20, 30, 40, 50
Array:

text
Index: 0 1 2 3 4
Value: 10 20 30 40 50
Now perform two DEQUEUE operations.

10 and 20 are removed.

The active Queue becomes:

```text
FRONT                   REAR
  |                       |
  v                       v
+------+    +------+    +------+
|  30  | -> |  40  | -> |  50  |
+------+    +------+    +------+
```

Array representation:

text
Index: 0 1 2 3 4
Value: \_ \_ 30 40 50
Now try to insert 60.

There are two unused positions at indices 0 and 1.

However:

java
rear == capacity - 1
is true.

Therefore, our simple linear Queue rejects the insertion.

This is called False Overflow or wasted-space limitation in a linear array Queue.

Why Does This Happen?
Because REAR only moves forward.

It does not return to the beginning of the array.

Solution
Circular Queue.

Circular Queue connects the last array position logically back to the first position.

We will cover this in Part 4.

19. QUEUE IMPLEMENTATION USING LINKED LIST
    A Queue can also be implemented using a Linked List.

In this implementation, we maintain two references:

FRONT
REAR
FRONT points to the first node.

REAR points to the last node.

Representation

```text
FRONT                                REAR
  |                                    |
  v                                    v
+------+-----+    +------+-----+    +------+------+
|  10  |  o-----> |  20  |  o-----> |  30  | NULL |
+------+-----+    +------+-----+    +------+------+
```

Why Use Linked List?
Unlike an array, a Linked List Queue can grow dynamically as new nodes are created.

There is no fixed array capacity.

However, available memory still limits its size.

ENQUEUE Using Linked List
New nodes are inserted at the end.

Algorithm:

text
ENQUEUE(value)

1. Create a new node.

2. If FRONT == NULL:
   FRONT = newNode.
   REAR = newNode.

3. Otherwise:
   REAR.next = newNode.
   REAR = newNode.

4. Stop.
   Example
   Initially:

text
FRONT = NULL
REAR = NULL
Insert 10.

```text
FRONT
  |
  v
+------+------+
|  10  | NULL |
+------+------+
  ^
  |
 REAR
```

Insert 20.

```text
FRONT                  REAR
  |                      |
  v                      v
+------+-----+    +------+------+
|  10  |  o-----> |  20  | NULL |
+------+-----+    +------+------+
```

Insert 30.

```text
FRONT                                REAR
  |                                    |
  v                                    v
+------+-----+    +------+-----+    +------+------+
|  10  |  o-----> |  20  |  o-----> |  30  | NULL |
+------+-----+    +------+-----+    +------+------+
```

Time Complexity: O(1)

DEQUEUE Using Linked List
Deletion takes place at FRONT.

Algorithm:

text
DEQUEUE()

1. If FRONT == NULL:
   Display Queue Underflow.

2. Otherwise:
   Store FRONT.data.

   FRONT = FRONT.next.

3. If FRONT == NULL:
   REAR = NULL.

4. Return removed value.
   Time Complexity: O(1)

5. COMPLETE JAVA PROGRAM – LINKED LIST QUEUE

java
class Node {

    int data;

    Node next;

    Node(int value) {

        data = value;

        next = null;
    }

}

class Queue {

    Node front;

    Node rear;

    // ENQUEUE
    void enqueue(int value) {

        Node newNode = new Node(value);

        if (rear == null) {

            front = newNode;

            rear = newNode;

        } else {

            rear.next = newNode;

            rear = newNode;
        }

        System.out.println(value + " inserted");
    }

    // DEQUEUE
    int dequeue() {

        if (front == null) {

            System.out.println("Queue Underflow");

            return -1;
        }

        int removed = front.data;

        front = front.next;

        if (front == null) {

            rear = null;
        }

        return removed;
    }

    // PEEK
    int peek() {

        if (front == null) {

            System.out.println("Queue is Empty");

            return -1;
        }

        return front.data;
    }

    // isEmpty
    boolean isEmpty() {

        return front == null;
    }

    // DISPLAY
    void display() {

        if (front == null) {

            System.out.println("Queue is Empty");

            return;
        }

        Node temp = front;

        System.out.println("Queue Elements:");

        while (temp != null) {

            System.out.println(temp.data);

            temp = temp.next;
        }
    }

}

public class Main {

    public static void main(String[] args) {

        Queue q = new Queue();

        q.enqueue(10);

        q.enqueue(20);

        q.enqueue(30);

        q.enqueue(40);

        q.display();

        System.out.println("Front: " + q.peek());

        System.out.println("Removed: " + q.dequeue());

        q.display();
    }

}
Expected Output

text
10 inserted
20 inserted
30 inserted
40 inserted
Queue Elements:
10
20
30
40
Front: 10
Removed: 10
Queue Elements:
20
30
40
Important Code Explanation

1. FRONT and REAR

java
Node front;

Node rear;
Both references initially contain null.

2. First Insertion

java
if (rear == null) {

    front = newNode;

    rear = newNode;

}
When inserting the first node, both FRONT and REAR point to it.

3. Subsequent Insertion

java
rear.next = newNode;

rear = newNode;
Connects the new node after the existing last node and updates REAR.

4. Deletion

java
front = front.next;
Moves FRONT to the next node.

5. Last Element Removal

java
if (front == null) {

    rear = null;

}
When the final node is removed, both references must be reset.

This is important to avoid an incorrect REAR reference when the Queue becomes empty.

21. ARRAY QUEUE VS LINKED LIST QUEUE

Feature Array Queue Linked List Queue
Memory allocation Array Individual nodes
Capacity Fixed Dynamic
ENQUEUE O(1) O(1)
DEQUEUE O(1) O(1)
PEEK O(1) O(1)
Memory overhead Lower per element Extra next reference
Overflow When array becomes full When memory allocation fails
Memory usage Entire array allocated Nodes allocated as needed
Implementation Uses FRONT and REAR indices Uses FRONT and REAR references
Note: A simple linear array Queue has a wasted-space limitation. A Circular Queue solves that limitation.

22. JAVA COLLECTIONS FRAMEWORK – QUEUE
    Java provides a built-in Queue interface in the Collections Framework.

It is available in:

java
import java.util.Queue;
A common implementation is LinkedList.

Example:

java
import java.util.Queue;

import java.util.LinkedList;

public class Main {

    public static void main(String[] args) {

        Queue<Integer> q = new LinkedList<>();

        q.add(10);

        q.add(20);

        q.add(30);

        System.out.println(q);

        System.out.println(q.peek());

        System.out.println(q.poll());

        System.out.println(q);
    }

}
Output

text
[10, 20, 30]
10
10
[20, 30]
Understanding Important Methods

Method Description
add() Inserts element; may throw exception if insertion fails
offer() Inserts element; returns false if insertion fails
remove() Removes FRONT; throws exception if empty
poll() Removes FRONT; returns null if empty
element() Returns FRONT; throws exception if empty
peek() Returns FRONT; returns null if empty
Important Difference: remove() vs poll()
Suppose Queue is empty.

java
q.remove();
Throws:

text
NoSuchElementException
Whereas:

java
q.poll();
Returns:

text
null
Similarly:

element() throws an exception on an empty Queue.
peek() returns null on an empty Queue.
This distinction is frequently tested in Java programming questions.

23. PRIORITYQUEUE INTRODUCTION
    Java provides a PriorityQueue implementation.

Unlike a normal FIFO Queue, a PriorityQueue orders elements according to their priority.

By default, Java's PriorityQueue uses natural ordering, so the smallest element is retrieved first for integers.

Example:

java
import java.util.PriorityQueue;

public class Main {

    public static void main(String[] args) {

        PriorityQueue<Integer> pq =
            new PriorityQueue<>();

        pq.add(40);

        pq.add(10);

        pq.add(30);

        pq.add(20);

        while (!pq.isEmpty()) {

            System.out.println(pq.poll());
        }
    }

}
Output

text
10
20
30
40
Important Point
A PriorityQueue is generally implemented using a heap.

In Java, it is a min-heap by default.

Therefore, the smallest element has the highest retrieval priority.

It is not a simple FIFO Queue.

We will study Priority Queue in more detail in Part 4.

24. TIME AND SPACE COMPLEXITY
    For a basic linear array Queue:

Operation Time Complexity
ENQUEUE O(1)
DEQUEUE O(1)
PEEK O(1)
isEmpty O(1)
isFull O(1)
DISPLAY O(n)
For a Linked List Queue:

Operation Time Complexity
ENQUEUE O(1)
DEQUEUE O(1)
PEEK O(1)
isEmpty O(1)
DISPLAY O(n)
Space Complexity
For a Queue containing n elements:

Space Complexity = O(n)

This applies to both implementations. 27. QUICK REVISION NOTES

Concept Important Point
Queue Linear data structure
Principle FIFO
Insertion ENQUEUE
Deletion DEQUEUE
First element FRONT
Last element REAR
Initial FRONT -1
Initial REAR -1
Array Queue ENQUEUE O(1)
Array Queue DEQUEUE O(1)
Linked List ENQUEUE O(1)
Linked List DEQUEUE O(1)
Overflow Insertion into full bounded Queue
Underflow Deletion from empty Queue
BFS Uses Queue
PriorityQueue Uses heap
poll() Removes and returns FRONT
peek() Returns FRONT without removal 28. FINAL CONCEPT CHECK
Before moving to Part 4, ensure you can answer these questions:

What is a Queue?
Explain FIFO with a real-life example.
Differentiate between Stack and Queue.
Explain FRONT and REAR.
Explain ENQUEUE and DEQUEUE with algorithms.
Write a Java program to implement Queue using an array.
Explain Queue Overflow and Underflow.
What is False Overflow in a linear Queue?
Explain Queue implementation using Linked List.
Differentiate Array Queue and Linked List Queue.
Differentiate poll(), peek(), remove(), and element().
What is PriorityQueue?
Which data structure is used in BFS?
What is the time complexity of ENQUEUE and DEQUEUE?

---

# PART 4 – ADVANCED QUEUE CONCEPTS

1. INTRODUCTION TO ADVANCED QUEUE
   In Part 3, we studied:

Queue fundamentals.
FIFO principle.
FRONT and REAR.
ENQUEUE and DEQUEUE.
Array implementation.
Linked List implementation.
Queue Overflow and Underflow.
Java Queue interface.
PriorityQueue introduction.
Now we will explore advanced Queue variations.

The important types of Queue are:

Linear Queue
Circular Queue
Double Ended Queue (Deque)
Priority Queue
Each type has different insertion and deletion rules.

2. CIRCULAR QUEUE
   A Circular Queue is a linear data structure in which the last position is logically connected to the first position.

Unlike a simple linear Queue, a Circular Queue reuses the empty positions created after deletion.

It follows the FIFO principle.

Definition
A Circular Queue is a Queue in which the last position is connected to the first position to form a circular structure.

Visual Representation

```text
        +-----+
        |  0  |
        +-----+
       /       \
   +-----+   +-----+
   |  4  |   |  1  |
   +-----+   +-----+
      |         |
   +-----+   +-----+
   |  3  |---|  2  |
   +-----+   +-----+
```

The actual implementation uses an array. The circular connection is logical, not a physical pointer connecting the array's last and first elements.

Why Is It Called Circular?
Because after REAR reaches the last array position, it can return to index 0 if space is available.

Similarly, FRONT can move forward using circular indexing.

3. WHY CIRCULAR QUEUE IS REQUIRED
   Let's understand the limitation of Linear Queue.

Suppose we have an array of capacity 5.

java
int[] queue = new int[5];
Insert five elements:

java
enqueue(10);
enqueue(20);
enqueue(30);
enqueue(40);
enqueue(50);
Array:

text
Index: 0 1 2 3 4
Value: 10 20 30 40 50
Now perform two deletions.

java
dequeue();
dequeue();
10 and 20 are removed.

Remaining elements:

text
Index: 0 1 2 3 4
Value: \_ \_ 30 40 50
There are two empty positions.

Now try inserting 60.

In a simple linear Queue, insertion fails because REAR has already reached the last index.

This creates wasted space.

Circular Queue Solution
Circular Queue allows REAR to move back to index 0.

Now 60 can be inserted at index 0.

Then 70 can be inserted at index 1.

New array:

```text
Index:  0   1   2   3   4
Value: 60  70  30  40  50
```

Logical Queue order:

```text
FRONT                         REAR
  |                             |
  v                             v
+------+    +------+    +------+    +------+    +------+
|  30  | -> |  40  | -> |  50  | -> |  60  | -> |  70  |
+------+    +------+    +------+    +------+    +------+
```

This demonstrates how Circular Queue reuses free positions.

4. WORKING OF CIRCULAR QUEUE
   Circular Queue uses the modulo (%) operator.

This is one of the most important concepts for CDAC CMCE.

Formula

java
rear = (rear + 1) % capacity;
Similarly:

java
front = (front + 1) % capacity;
Understanding Modulo
Suppose:

java
capacity = 5;
If:

java
rear = 3;
Then:

java
rear = (3 + 1) % 5;

rear = 4;
If:

java
rear = 4;
Then:

java
rear = (4 + 1) % 5;

rear = 0;
Therefore, REAR returns to index 0.

Modulo Examples

Expression Result
1 % 5 1
2 % 5 2
3 % 5 3
4 % 5 4
5 % 5 0
6 % 5 1
7 % 5 2
Remember:

The modulo operator helps implement circular movement.

5. CIRCULAR QUEUE CONDITIONS
   There are three important conditions.

5.1 Empty Condition
In our implementation:

java
front == -1
Initially:

java
front = -1;
rear = -1;
5.2 Full Condition
The Circular Queue is full when:

java
(rear + 1) % capacity == front
Explanation
The next position after REAR is calculated using modulo.

If that position is FRONT, there is no available position for insertion.

Example:

text
capacity = 5

front = 2
rear = 1
The next position is:

java
(rear + 1) % capacity

(1 + 1) % 5

2
Since the next position equals FRONT, the Queue is full.

5.3 Normal Insertion Condition
If the Queue is not full, insertion is possible.

6. ENQUEUE IN CIRCULAR QUEUE
   ENQUEUE inserts an element at REAR.

Algorithm

text
ENQUEUE(value)

1. Check whether:
   (rear + 1) % capacity == front

2. If true:
   Display Queue Overflow.

3. Otherwise:

   If front == -1:
   front = 0
   rear = 0

   Else:
   rear = (rear + 1) % capacity

4. queue[rear] = value

5. Stop.
   Java Code

java
void enqueue(int value) {

    if ((rear + 1) % capacity == front) {

        System.out.println("Queue Overflow");

    } else {

        if (front == -1) {

            front = 0;

            rear = 0;

        } else {

            rear = (rear + 1) % capacity;
        }

        queue[rear] = value;
    }

}
Time Complexity
O(1)

7. DEQUEUE IN CIRCULAR QUEUE
   DEQUEUE removes the element at FRONT.

Algorithm

text
DEQUEUE()

1. Check whether front == -1.

2. If true:
   Display Queue Underflow.

3. Otherwise:

   Store queue[front].

4. If front == rear:
   front = -1
   rear = -1

5. Otherwise:
   front = (front + 1) % capacity

6. Return removed element.

7. Stop.
   Java Code

java
int dequeue() {

    if (front == -1) {

        System.out.println("Queue Underflow");

        return -1;
    }

    int removed = queue[front];

    if (front == rear) {

        front = -1;

        rear = -1;

    } else {

        front = (front + 1) % capacity;
    }

    return removed;

}
Time Complexity
O(1)

8. COMPLETE CIRCULAR QUEUE JAVA PROGRAM

java
class CircularQueue {

    int front;

    int rear;

    int capacity;

    int[] queue;

    CircularQueue(int size) {

        capacity = size;

        queue = new int[capacity];

        front = -1;

        rear = -1;
    }

    // ENQUEUE
    void enqueue(int value) {

        if ((rear + 1) % capacity == front) {

            System.out.println("Queue Overflow");

            return;
        }

        if (front == -1) {

            front = 0;

            rear = 0;

        } else {

            rear = (rear + 1) % capacity;
        }

        queue[rear] = value;

        System.out.println(value + " inserted");
    }

    // DEQUEUE
    int dequeue() {

        if (front == -1) {

            System.out.println("Queue Underflow");

            return -1;
        }

        int removed = queue[front];

        if (front == rear) {

            front = -1;

            rear = -1;

        } else {

            front = (front + 1) % capacity;
        }

        return removed;
    }

    // PEEK
    int peek() {

        if (front == -1) {

            System.out.println("Queue is Empty");

            return -1;
        }

        return queue[front];
    }

    // isEmpty
    boolean isEmpty() {

        return front == -1;
    }

    // isFull
    boolean isFull() {

        return (rear + 1) % capacity == front;
    }

    // DISPLAY
    void display() {

        if (isEmpty()) {

            System.out.println("Queue is Empty");

            return;
        }

        System.out.println("Queue Elements:");

        int i = front;

        while (true) {

            System.out.print(queue[i] + " ");

            if (i == rear) {

                break;
            }

            i = (i + 1) % capacity;
        }

        System.out.println();
    }

}

public class Main {

    public static void main(String[] args) {

        CircularQueue q = new CircularQueue(5);

        q.enqueue(10);

        q.enqueue(20);

        q.enqueue(30);

        q.enqueue(40);

        q.enqueue(50);

        q.display();

        System.out.println("Removed: " + q.dequeue());

        System.out.println("Removed: " + q.dequeue());

        q.enqueue(60);

        q.enqueue(70);

        q.display();

        System.out.println("Front Element: " + q.peek());
    }

}
Expected Output

text
10 inserted
20 inserted
30 inserted
40 inserted
50 inserted
Queue Elements:
10 20 30 40 50
Removed: 10
Removed: 20
60 inserted
70 inserted
Queue Elements:
30 40 50 60 70
Front Element: 30 9. CIRCULAR QUEUE DRY RUN
Consider:

java
CircularQueue q = new CircularQueue(5);

q.enqueue(10);
q.enqueue(20);
q.enqueue(30);
q.enqueue(40);
q.enqueue(50);

q.dequeue();
q.dequeue();

q.enqueue(60);
q.enqueue(70);
Let's examine every operation.

Operation FRONT REAR Logical Queue
Initial -1 -1 Empty
ENQUEUE(10) 0 0 10
ENQUEUE(20) 0 1 10, 20
ENQUEUE(30) 0 2 10, 20, 30
ENQUEUE(40) 0 3 10, 20, 30, 40
ENQUEUE(50) 0 4 10, 20, 30, 40, 50
DEQUEUE() 1 4 20, 30, 40, 50
DEQUEUE() 2 4 30, 40, 50
ENQUEUE(60) 2 0 30, 40, 50, 60
ENQUEUE(70) 2 1 30, 40, 50, 60, 70
Final array:

text
Index: 0 1 2 3 4
Value: 60 70 30 40 50
Logical Queue:

```text
FRONT
  |
  v
  30 → 40 → 50 → 60 → 70
                       ^
                       |
                      REAR
```

Notice:

FRONT = 2

REAR = 1

Even though REAR is numerically smaller than FRONT, the Queue is completely full.

This is a very important exam concept.

10. CIRCULAR QUEUE VS LINEAR QUEUE

Feature Linear Queue Circular Queue
Structure Linear Circular logically
Principle FIFO FIFO
Rear movement Forward only Wraps around
Space reuse Limited Yes
Modulo operator Not required Used
Overflow condition rear == capacity - 1 (rear + 1) % capacity == front
Deletion FRONT increases FRONT moves circularly
Efficiency Can waste array positions Better array-space utilization
Important MCQ
Which Queue overcomes the space wastage problem of Linear Queue?

Answer: Circular Queue.

11. DEQUE – DOUBLE ENDED QUEUE
    Deque stands for Double Ended Queue.

It is a Queue in which insertion and deletion are allowed at both ends.

Both FRONT and REAR can be used for insertion and deletion.

Definition
A Deque is a linear data structure in which elements can be inserted and removed from both FRONT and REAR.

Diagram

```text
        INSERT / DELETE
               |
               v
FRONT                         REAR
  |                             |
  v                             v
+------+    +------+    +------+
|  10  | -> |  20  | -> |  30  |
+------+    +------+    +------+
  ^                             ^
  |                             |
INSERT / DELETE          INSERT / DELETE
```

Four Main Operations
insertFront()
insertRear()
deleteFront()
deleteRear()
Applications
Palindrome checking.
Sliding Window Maximum.
Task scheduling.
Undo and redo systems.
Browser navigation. 12. INPUT RESTRICTED DEQUE
In an Input Restricted Deque:

Insertion is allowed at only one end.
Deletion is allowed at both ends.
Example:

text
Insertion: REAR only

Deletion: FRONT or REAR
Diagram

```text
          INSERT
             |
             v
FRONT                       REAR
  |                           |
  v                           v
+------+    +------+    +------+
|  10  | -> |  20  | -> |  30  |
+------+    +------+    +------+
  |                           |
DELETE                     DELETE
```

Important Point
Input Restricted means insertion is restricted to one end.

13. OUTPUT RESTRICTED DEQUE
    In an Output Restricted Deque:

Insertion is allowed at both ends.
Deletion is allowed at only one end.

Example:

text
Insertion: FRONT or REAR

Deletion: FRONT only
Diagram

```text
INSERT                       INSERT
   |                            |
   v                            v
FRONT                         REAR
  |                             |
  v                             v
+------+    +------+    +------+
|  10  | -> |  20  | -> |  30  |
+------+    +------+    +------+
  |
DELETE
```

Important Difference

Feature Input Restricted Deque Output Restricted Deque
Insertion One end Both ends
Deletion Both ends One end
This is a frequently tested theoretical question.

14. DEQUE IMPLEMENTATION USING JAVA
    Java provides a Deque interface.

Import:

java
import java.util.Deque;

import java.util.ArrayDeque;
Example:

java
import java.util.Deque;

import java.util.ArrayDeque;

public class Main {

    public static void main(String[] args) {

        Deque<Integer> dq = new ArrayDeque<>();

        dq.addFirst(10);

        dq.addLast(20);

        dq.addFirst(5);

        dq.addLast(30);

        System.out.println(dq);

        dq.removeFirst();

        dq.removeLast();

        System.out.println(dq);
    }

}
Output

text
[5, 10, 20, 30]

[10, 20]
Important Deque Methods

Method Function
addFirst() Insert at FRONT
addLast() Insert at REAR
offerFirst() Insert at FRONT
offerLast() Insert at REAR
removeFirst() Delete from FRONT
removeLast() Delete from REAR
pollFirst() Delete from FRONT; returns null if empty
pollLast() Delete from REAR; returns null if empty
peekFirst() Read FRONT
peekLast() Read REAR
Important Point
ArrayDeque does not permit null elements.

It is generally an efficient implementation for Stack and Queue operations.

15. PRIORITY QUEUE IN DETAIL
    A Priority Queue is a Queue in which elements are removed according to priority rather than their insertion order.

In a normal Queue:

text
First inserted → First removed
In a Priority Queue:

text
Highest priority → Removed first
Example
Suppose we have:

text
Patient A: Priority 3
Patient B: Priority 1
Patient C: Priority 2
Assuming smaller numbers indicate higher priority:

Removal order:

text
Patient B
Patient C
Patient A
Important Note
Priority Queue does not necessarily preserve FIFO order among elements with equal priority.

A separate sequence number is needed if stable FIFO ordering is required.

Applications
CPU scheduling.
Dijkstra's shortest path algorithm.
Prim's Minimum Spanning Tree algorithm.
Event-driven simulation.
Network scheduling. 16. MIN HEAP
A Min Heap is a complete binary tree in which every parent node is smaller than or equal to its children.

Therefore, the smallest element is at the root.

Example

```text
             10
           /    \
         20      30
        /  \
       40   50
```

The root is 10.

Because:

10 <= 20

10 <= 30

20 <= 40

20 <= 50

Array Representation

text
Index: 0 1 2 3 4
Value: 10 20 30 40 50
Important Formulas
For a zero-indexed array:

Left Child:

java
2 \* i + 1
Right Child:

java
2 \* i + 2
Parent:

java
(i - 1) / 2
These formulas are frequently tested.

Time Complexity

Operation Complexity
Find Minimum O(1)
Insert O(log n)
Remove Minimum O(log n)
Build Heap O(n) 17. MAX HEAP
A Max Heap is a complete binary tree in which every parent node is greater than or equal to its children.

Therefore, the maximum element is at the root.

Example

```text
             50
           /    \
         40      30
        /  \
       20   10
```

Difference Between Min Heap and Max Heap

Feature Min Heap Max Heap
Root Smallest Largest
Parent <= Children >= Children
First removed Smallest Largest
Java default PriorityQueue Min Heap Not default
Important Exam Point
Java's PriorityQueue uses natural ordering by default.

For Integer values, it behaves as a Min Heap.

18. PRIORITYQUEUE JAVA IMPLEMENTATION
    Example 1: Min Heap

java
import java.util.PriorityQueue;

public class Main {

    public static void main(String[] args) {

        PriorityQueue<Integer> pq =
            new PriorityQueue<>();

        pq.add(50);

        pq.add(10);

        pq.add(40);

        pq.add(20);

        pq.add(30);

        while (!pq.isEmpty()) {

            System.out.println(pq.poll());
        }
    }

}
Output

text
10
20
30
40
50
Example 2: Max Heap
We can use Collections.reverseOrder().

java
import java.util.PriorityQueue;

import java.util.Collections;

public class Main {

    public static void main(String[] args) {

        PriorityQueue<Integer> pq =
            new PriorityQueue<>(Collections.reverseOrder());

        pq.add(50);

        pq.add(10);

        pq.add(40);

        pq.add(20);

        pq.add(30);

        while (!pq.isEmpty()) {

            System.out.println(pq.poll());
        }
    }

}
Output

text
50
40
30
20
10
Important Exam Point

java
new PriorityQueue<>()
Creates a Min Heap for naturally ordered Integer values.

java
new PriorityQueue<>(Collections.reverseOrder())
Creates a Max Heap for naturally ordered Integer values.

19. CUSTOM COMPARATOR IN PRIORITYQUEUE
    A Comparator allows us to define our own ordering.

Example: Sort integers in descending order.

java
import java.util.PriorityQueue;

import java.util.Comparator;

public class Main {

    public static void main(String[] args) {

        PriorityQueue<Integer> pq =
            new PriorityQueue<>(Comparator.reverseOrder());

        pq.add(15);

        pq.add(5);

        pq.add(25);

        pq.add(10);

        while (!pq.isEmpty()) {

            System.out.println(pq.poll());
        }
    }

}
Output

text
25
15
10
5
PriorityQueue of Objects
Consider students with marks.

java
import java.util.PriorityQueue;

class Student {

    String name;

    int marks;

    Student(String name, int marks) {

        this.name = name;

        this.marks = marks;
    }

}

public class Main {

    public static void main(String[] args) {

        PriorityQueue<Student> pq =
            new PriorityQueue<>(
                (a, b) -> Integer.compare(b.marks, a.marks)
            );

        pq.add(new Student("Amit", 80));

        pq.add(new Student("Rahul", 95));

        pq.add(new Student("Neha", 90));

        while (!pq.isEmpty()) {

            Student s = pq.poll();

            System.out.println(s.name + " " + s.marks);
        }
    }

}
Output

text
Rahul 95
Neha 90
Amit 80
Explanation:

The Comparator compares marks in descending order.

The student with the highest marks is removed first.

20. IMPLEMENT QUEUE USING TWO STACKS
    This is an important data structure interview and programming problem.

Normally:

Stack follows LIFO.
Queue follows FIFO.
We can implement Queue behaviour using two Stacks.

Let:

java
Stack<Integer> s1;

Stack<Integer> s2;
Basic Idea
Use:

s1 for incoming elements.
s2 for outgoing elements.
ENQUEUE
Push the element into s1.

java
s1.push(value);
DEQUEUE
If s2 is not empty, pop from s2.
If s2 is empty, transfer all elements from s1 to s2.
Pop from s2.
Why Does This Work?
Suppose we insert:

text
10, 20, 30
Stack s1:

text
TOP
|
v
+----+
| 30 |
+----+
| 20 |
+----+
| 10 |
+----+
Transfer all elements into s2.

Stack s2:

text
TOP
|
v
+----+
| 10 |
+----+
| 20 |
+----+
| 30 |
+----+
Now popping s2 returns 10 first.

Thus FIFO behaviour is achieved.

Complete Java Program

java
import java.util.Stack;

class MyQueue {

    Stack<Integer> s1 = new Stack<>();

    Stack<Integer> s2 = new Stack<>();

    void enqueue(int value) {

        s1.push(value);
    }

    int dequeue() {

        if (s1.isEmpty() && s2.isEmpty()) {

            System.out.println("Queue Underflow");

            return -1;
        }

        if (s2.isEmpty()) {

            while (!s1.isEmpty()) {

                s2.push(s1.pop());
            }
        }

        return s2.pop();
    }

}

public class Main {

    public static void main(String[] args) {

        MyQueue q = new MyQueue();

        q.enqueue(10);

        q.enqueue(20);

        q.enqueue(30);

        System.out.println(q.dequeue());

        System.out.println(q.dequeue());

        q.enqueue(40);

        System.out.println(q.dequeue());

        System.out.println(q.dequeue());
    }

}
Output

text
10
20
30
40
Complexity

Operation Complexity
ENQUEUE O(1)
DEQUEUE Amortized O(1)
Worst-case DEQUEUE O(n)
Why amortized O(1)?

Each element is pushed into s1 once, transferred to s2 at most once, and popped from s2 once.

Therefore, across many operations, the total work is linear.

21. IMPLEMENT STACK USING TWO QUEUES
    We can also implement Stack behaviour using two Queues.

Remember:

Stack follows LIFO.

Queue follows FIFO.

We use two Queues to reproduce Stack operations.

Let:

java
Queue<Integer> q1;

Queue<Integer> q2;
Approach: Costly PUSH
Insert the new element into q2.
Move all elements from q1 to q2.
Swap q1 and q2.
Now the newest element is always at the front of q1.

Complete Java Program

java
import java.util.LinkedList;

import java.util.Queue;

class MyStack {

    Queue<Integer> q1 = new LinkedList<>();

    Queue<Integer> q2 = new LinkedList<>();

    void push(int value) {

        q2.add(value);

        while (!q1.isEmpty()) {

            q2.add(q1.remove());
        }

        Queue<Integer> temp = q1;

        q1 = q2;

        q2 = temp;
    }

    int pop() {

        if (q1.isEmpty()) {

            System.out.println("Stack Underflow");

            return -1;
        }

        return q1.remove();
    }

    int peek() {

        if (q1.isEmpty()) {

            return -1;
        }

        return q1.peek();
    }

}

public class Main {

    public static void main(String[] args) {

        MyStack s = new MyStack();

        s.push(10);

        s.push(20);

        s.push(30);

        System.out.println(s.pop());

        System.out.println(s.pop());

        System.out.println(s.peek());
    }

}
Output

text
30
20
10
Complexity

Operation Complexity
PUSH O(n)
POP O(1)
PEEK O(1) 22. ARRAYDEQUE IN JAVA
ArrayDeque is a resizable-array implementation of the Deque interface.

It can be used as:

A Queue.
A Deque.
A Stack.
It generally provides efficient operations at both ends.

Queue Example

java
import java.util.ArrayDeque;

import java.util.Queue;

public class Main {

    public static void main(String[] args) {

        Queue<Integer> q = new ArrayDeque<>();

        q.offer(10);

        q.offer(20);

        q.offer(30);

        System.out.println(q);

        System.out.println(q.poll());

        System.out.println(q.peek());
    }

}
Output

text
[10, 20, 30]
10
20
Stack Example Using ArrayDeque

java
import java.util.ArrayDeque;

import java.util.Deque;

public class Main {

    public static void main(String[] args) {

        Deque<Integer> stack = new ArrayDeque<>();

        stack.push(10);

        stack.push(20);

        stack.push(30);

        System.out.println(stack.pop());

        System.out.println(stack.peek());
    }

}
Output

text
30
20
Important Exam Point
ArrayDeque is often preferred over the legacy Stack class for ordinary single-threaded stack operations.

ArrayDeque does not allow null elements.

23. BREADTH FIRST SEARCH USING QUEUE
    Breadth First Search (BFS) is a graph traversal algorithm.

It explores vertices level by level.

Queue is used to maintain the order of vertices waiting to be visited.

Example Graph

```text
        A
       / \
      B   C
     / \   \
    D   E   F
```

BFS starting from A:

text
A → B → C → D → E → F
BFS Algorithm

text
BFS(start)

1.  Create an empty Queue.

2.  Mark start as visited.

3.  Insert start into Queue.

4.  While Queue is not empty:

    Remove FRONT element.

    Visit the removed element.

    For every unvisited neighbour:

          Mark it visited.

          Insert it into Queue.

5.  Stop.
    Complete Java Program

java
import java.util.\*;

public class Main {

    public static void main(String[] args) {

        int vertices = 6;

        ArrayList<ArrayList<Integer>> graph =
            new ArrayList<>();

        for (int i = 0; i < vertices; i++) {

            graph.add(new ArrayList<>());
        }

        // Add edges

        graph.get(0).add(1);

        graph.get(0).add(2);

        graph.get(1).add(3);

        graph.get(1).add(4);

        graph.get(2).add(5);

        boolean[] visited = new boolean[vertices];

        Queue<Integer> q = new LinkedList<>();

        int start = 0;

        visited[start] = true;

        q.add(start);

        System.out.println("BFS Traversal:");

        while (!q.isEmpty()) {

            int current = q.poll();

            System.out.print(current + " ");

            for (int neighbour : graph.get(current)) {

                if (!visited[neighbour]) {

                    visited[neighbour] = true;

                    q.add(neighbour);
                }
            }
        }
    }

}
Output

text
BFS Traversal:
0 1 2 3 4 5
Explanation
Start from vertex 0.
Insert its neighbours 1 and 2.
Remove 1 and insert its unvisited neighbours 3 and 4.
Remove 2 and insert its unvisited neighbour 5.
Continue until the Queue becomes empty.
Complexity
For an adjacency-list representation:

Time Complexity: O(V + E)

Space Complexity: O(V)

Where:

V = Number of vertices.
E = Number of edges. 24. QUEUE APPLICATIONS
Important real-world and programming applications include:

Application Queue Type
Printer scheduling Normal Queue
CPU Round Robin Circular Queue
BFS traversal Normal Queue
Hospital priority handling Priority Queue
Sliding Window Maximum Deque
Task scheduling Queue / Priority Queue
Network packet buffering Queue
Dijkstra's algorithm Priority Queue
Undo operations Stack
Palindrome checking Deque
Important Distinction
BFS uses a normal Queue.

Dijkstra's shortest path algorithm commonly uses a PriorityQueue.

Round Robin scheduling commonly uses a Circular Queue or equivalent cyclic scheduling logic.

25. TIME AND SPACE COMPLEXITY
    Queue Complexity Comparison

Operation Linear Queue Circular Queue Linked List Queue
ENQUEUE O(1) O(1) O(1)
DEQUEUE O(1) O(1) O(1)
PEEK O(1) O(1) O(1)
DISPLAY O(n) O(n) O(n)
isEmpty O(1) O(1) O(1)
PriorityQueue Complexity

Operation Complexity
offer() O(log n)
add() O(log n)
poll() O(log n)
remove() O(log n)
peek() O(1)
contains() O(n)
Deque Complexity
For Java ArrayDeque, insertion and removal at either end are generally O(1) amortized.

28. IMPORTANT CODING QUESTIONS
    Practice these programs before attempting the final Chapter 6 test.

Beginner Level
Implement Queue using an array.
Implement Queue using Linked List.
Implement Circular Queue.
Reverse a Queue.
Find the first element of a Queue.
Count the elements in a Queue.
Intermediate Level
Implement Deque using Java.
Implement Stack using two Queues.
Implement Queue using two Stacks.
Generate binary numbers from 1 to N using Queue.
Reverse the first K elements of a Queue.
Find the first non-repeating character in a stream.
Advanced Level
Implement BFS using Queue.
Implement PriorityQueue with custom Comparator.
Find the K largest elements using PriorityQueue.
Find the Sliding Window Maximum using Deque.
Implement Dijkstra's algorithm using PriorityQueue.
Implement a circular Queue with dynamic capacity.

Important Data Structures at a Glance

Data Structure Principle Main Operations
Stack LIFO PUSH, POP
Queue FIFO ENQUEUE, DEQUEUE
Circular Queue FIFO ENQUEUE, DEQUEUE
Deque Both ends Insert/Delete at both ends
Priority Queue Priority-based Insert, Remove highest priority
Min Heap Parent <= Children Insert, Remove Minimum
Max Heap Parent >= Children Insert, Remove Maximum
Important Java Classes

Java Class Purpose
Stack Stack implementation
LinkedList Implements Queue and Deque
PriorityQueue Priority-based Queue
ArrayDeque Efficient Deque implementation
ArrayList Dynamic array
Comparator Custom ordering
Must-Remember Formulas
Circular Queue next position:

```java
(rear + 1) % capacity
```

Circular Queue previous position:

```java
(front - 1 + capacity) % capacity
```

Circular Queue full condition:

```java
(rear + 1) % capacity == front
```

Left child in a zero-indexed heap:

```java
2 \* i + 1
```

Right child in a zero-indexed heap:

```java
2 \* i + 2
```

Parent in a zero-indexed heap:

```java
(i - 1) / 2
```
