# Chapter 5: Linked Lists with Java

# PART 1: Introduction to Linked Lists and Singly Linked Lists

## Table of Contents

1. Introduction to Linked Lists
2. Why Linked Lists?
3. Array vs Linked List
4. Types of Linked Lists
5. Node Structure in Java
6. Creating a Singly Linked List
7. Traversing a Linked List
8. Counting Nodes
9. Searching an Element
10. Insertion at the Beginning
11. Insertion at the End
12. Insertion at a Given Position
13. Deletion from the Beginning
14. from the End
15. Deletion at a Given Position
16. Complete Singly Linked List Program
17. Time and Space Complexity
18. Practice Questions

---

## 5.1 Introduction to Linked Lists

A Linked List is a linear data structure in which elements are stored in separate objects called Nodes.

Each node contains:

- Data: Stores the actual value.
- Next: Stores the reference to the next node.

Unlike arrays, linked list elements are not required to occupy contiguous memory locations.

Example

```text
Head
 |
 v
[10 | Next] -> [20 | Next] -> [30 | null]
```

Here:

- 10, 20, 30 are data elements.
- Each node points to the next node.
- The last node points to null.
- Head stores the reference to the first node.
- Important Terminology

| Term      | Meaning                |
| --------- | ---------------------- |
| Node      | Individual element     |
| Head      | First Node             |
| Tail      | Last Node              |
| Next      | reference to next node |
| Null      | Indicates end of list  |
| Traversal | Visitig each node      |

## 5.2 Why Linked Lists?

Arrays have a fixed size after creation in Java.

> Linked Lists provide dynamic memory allocation and make insertion and deletion easier when the required node position is already known.

**Advantages**

1. Dynamic size.
2. Efficient insertion at the beginning.
3. Efficient deletion at the beginning.
4. No requirement for contiguous memory.
5. Useful for implementing stacks and queues.

**Disadvantages**

1. No direct/random access.
2. Extra memory required for references.
3. Searching takes O(n) time.
4. Pointer/reference manipulation can be complex.

## 5.3 Array vs Linked List

| Feature             | Array                    | Linked List         |
| ------------------- | ------------------------ | ------------------- |
| Memory              | Contiguous               | non-contigous       |
| Size                | Fixed after creation     | dynamic             |
| Access by index     | $$\mathcal{O}(1)$$       | $$\mathcal{O}(n)$$  |
| Searching           | $$\mathcal{O}(n)$$       | $$\mathcal{O}(n)$$  |
| Insert at beginning | $$\mathcal{O}(n)$$       | $$\mathcal{O}(1)$$  |
| Delete at beginning | $$\mathcal{O}(n)$$       | $$\mathcal{O}(1)$$  |
| Extra memory        | Usually no link overhead | Requires references |
| Cache performance   | Generally better         | Generally poorer    |

**Important for CDAC**: Understand the difference between array indexing and linked list traversal.

## 5.4 Types of Linked List

There are four commonly discussed types

### 1. Singly Linked List

Each node points to the next node.

```text
10 -> 20 -> 30 -> 40 -> null
```

### 2. Doubly Linked List

Each node contains references to both previous and next node.

```text
null <- 10 <-> 20 <-> 30 -> null
```

### 3. Circular Linked List

The last node points back to the first node.

```text
10 -> 20 -> 30
^           |
|___________|
```

### 4. Circular Doubly Linked List

Each node contains previous and next references, and the first and last nodes are connected.

These types are covered in Parts 2 and 3.

# 5.5 Node Structure in Java

A node is the basic building block of a linked list.

**Java Implementation**

```java
class Node {

    int data;

    Node next;

    Node(int data) {

        this.data = data;

        this.next = null;
    }
}
```

Explanation:

- int data stores the value.
- Node next stores the reference to the next node.
- Constructor initializes the node.

**Creating Individual Nodes**

```java
class Main {

    public static void main(String[] args) {

        Node first = new Node(10);

        Node second = new Node(20);

        Node third = new Node(30);

        first.next = second;

        second.next = third;

        Node head = first;
    }
}
```

Linked List:

```text
10 -> 20 -> 30 -> null
```

## 5.6 Creating a Singly Linked List

A singly linked list can be created by connecting nodes using their next references.

**Java Implementation**

```java
class Node {

    int data;

    Node next;

    Node(int data) {

        this.data = data;

        this.next = null;
    }
}

class Main {

    public static void main(String[] args) {

        Node head = new Node(10);

        Node second = new Node(20);

        Node third = new Node(30);

        head.next = second;

        second.next = third;

        System.out.println(head.data);

        System.out.println(head.next.data);

        System.out.println(head.next.next.data);
    }
}
```

Output:

```text
10
20
30
```

## 5.7 Traversing a Linked List

Traversal means visiting every node from the head to the last node.

Approach

- Start from head.
- Print the current node's data.
- Move to the next node.
- Repeat until current becomes null.

**Java Implementation**

```java
class Node {

    int data;

    Node next;

    Node(int data) {
        this.data = data;
    }
}

class Main {

    static void display(Node head) {

        Node current = head;

        while (current != null) {

            System.out.print(current.data + " ");

            current = current.next;
        }
    }

    public static void main(String[] args) {

        Node head = new Node(10);

        head.next = new Node(20);

        head.next.next = new Node(30);

        display(head);
    }
}
```

Output:

```text
10 20 30
```

Time Complexity: O(n)

Space Complexity: O(1)

**Important**: Never move the original head reference if you still need it. Use a temporary pointer such as current.

## 5.8 Counting Nodes in a Linked List

Find the total number of nodes.

**Java Implementation**

```java
class Main {

    static int countNodes(Node head) {

        int count = 0;

        Node current = head;

        while (current != null) {

            count++;

            current = current.next;
        }

        return count;
    }

    public static void main(String[] args) {

        Node head = new Node(10);

        head.next = new Node(20);

        head.next.next = new Node(30);

        System.out.println(countNodes(head));
    }
}
```

Output:

```text
3
```

Time Complexity: O(n)

Space Complexity: O(1)

## 5.9 Searching an Element in a Linked List

Find whether a given value exists in the linked list.

Example

```text
10 -> 20 -> 30 -> 40 -> null
```

Search = 30

Output:

```text
element found
```

**Java Implementation**

```java
class Main {

    static boolean search(Node head, int target) {

        Node current = head;

        while (current != null) {

            if (current.data == target) {
                return true;
            }

            current = current.next;
        }

        return false;
    }

    public static void main(String[] args) {

        Node head = new Node(10);

        head.next = new Node(20);

        head.next.next = new Node(30);

        System.out.println(search(head, 30));
    }
}
```

Output:

```text
true
```

Time Complexity: O(n)

Space Complexity: O(1)

## 5.10 Insertion at the Beginning

Insertion means adding a new node to the linked list.

Example

Original:

```text
20 -> 30 -> 40 -> null

```

Insert 10 at beginning

```text
10 -> 20 -> 30 -> 40 -> null
```

Steps

1. Create a new node.
2. Set its next reference to the current head.
3. Update head to the new node.

**Java Implementation**

```java
class Main {

    static Node insertAtBeginning(Node head, int data) {

        Node newNode = new Node(data);

        newNode.next = head;

        head = newNode;

        return head;
    }

    static void display(Node head) {

        while (head != null) {

            System.out.print(head.data + " ");

            head = head.next;
        }
    }

    public static void main(String[] args) {

        Node head = new Node(20);

        head.next = new Node(30);

        head.next.next = new Node(40);

        head = insertAtBeginning(head, 10);

        display(head);
    }
}
```

Output:

```text
10 20 30 40
```

Time Complexity: O(1)

Space Complexity: O(1) auxiliary space, excluding the new node.

## 5.11 Insertion at the End

Add a new node at the end of the linked list.

Example
Original:

```text
10 -> 20 -> 30 -> null
```

Insert 40:

```text
10 -> 20 -> 30 -> 40 -> null
```

Steps

1. Create a new node.
2. If the list is empty, make it the head.
3. Otherwise, traverse to the last node.
4. Connect the last node to the new node.

**Java Implementation**

```java
class Main {

    static Node insertAtEnd(Node head, int data) {

        Node newNode = new Node(data);

        if (head == null) {
            return newNode;
        }

        Node current = head;

        while (current.next != null) {

            current = current.next;
        }

        current.next = newNode;

        return head;
    }

    public static void main(String[] args) {

        Node head = new Node(10);

        head.next = new Node(20);

        head.next = insertAtEnd(head.next, 30);

        head = insertAtEnd(head, 40);
    }
}
```

Time Complexity: O(n)

Space Complexity: O(1) auxiliary space.

Note: Maintaining a tail reference allows insertion at the end in O(1).

## 5.12 Insertion at a Given Position

Insert an element at a specified position.

Assume positions start from 1.

Example

Original:

```text
10 -> 20 -> 40 -> null
```

Insert 30 at position 3:

```text
10 -> 20 -> 30 -> 40 -> null
```

**Java Implementation**

```java
class Main {

    static Node insertAtPosition(
        Node head, int data, int position
    ) {

        if (position < 1) {
            throw new IllegalArgumentException("Invalid position");
        }

        Node newNode = new Node(data);

        if (position == 1) {

            newNode.next = head;

            return newNode;
        }

        Node current = head;

        for (int i = 1; i < position - 1; i++) {

            if (current == null) {
                throw new IndexOutOfBoundsException();
            }

            current = current.next;
        }

        if (current == null) {
            throw new IndexOutOfBoundsException();
        }

        newNode.next = current.next;

        current.next = newNode;

        return head;
    }
}
```

Time Complexity: O(n)

Space Complexity: O(1) auxiliary space.

## 5.13 Deletion from the Beginning

Remove the first node from the linked list.

Example

Original:

```text
10 -> 20 -> 30 -> null
```

After deletion:

```text
20 -> 30 -> null
```

**Java Implementation**

```java
class Main {

    static Node deleteBeginning(Node head) {

        if (head == null) {
            return null;
        }

        head = head.next;

        return head;
    }
}
```

Time Complexity: O(1)

Space Complexity: O(1)

## 5.14 Deletion from the End

Remove the last node.

Example

Original:

```text
10 -> 20 -> 30 -> null
```

After deletion:

```text
10 -> 20 -> null
```

**Java Implementation**

```java
class Main {

    static Node deleteEnd(Node head) {

        if (head == null || head.next == null) {
            return null;
        }

        Node current = head;

        while (current.next.next != null) {

            current = current.next;
        }

        current.next = null;

        return head;
    }
}
```

Time Complexity: O(n)

Space Complexity: O(1)

## 5.15 Deletion at a Given Position

Delete the node at a specified position.

Assume positions start from 1.

Example

Original:

```text
10 -> 20 -> 30 -> 40 -> null
```

Delete position 3:

```text
10 -> 20 -> 40 -> null
```

**Java Implementation**

```java
class Main {

    static Node deleteAtPosition(Node head, int position) {

        if (position < 1) {
            throw new IllegalArgumentException("Invalid position");
        }

        if (head == null) {
            return null;
        }

        if (position == 1) {
            return head.next;
        }

        Node current = head;

        for (int i = 1; i < position - 1; i++) {

            if (current.next == null) {
                throw new IndexOutOfBoundsException();
            }

            current = current.next;
        }

        if (current.next == null) {
            throw new IndexOutOfBoundsException();
        }

        current.next = current.next.next;

        return head;
    }
}
```

Time Complexity: O(n)

Space Complexity: O(1)

## 5.16 Complete Singly Linked List Program

The following program combines common singly linked list operations.

```java
class Node {

    int data;

    Node next;

    Node(int data) {

        this.data = data;

        this.next = null;
    }
}

class SinglyLinkedList {

    Node head;

    void insertAtBeginning(int data) {

        Node newNode = new Node(data);

        newNode.next = head;

        head = newNode;
    }

    void insertAtEnd(int data) {

        Node newNode = new Node(data);

        if (head == null) {

            head = newNode;

            return;
        }

        Node current = head;

        while (current.next != null) {

            current = current.next;
        }

        current.next = newNode;
    }

    void deleteBeginning() {

        if (head != null) {

            head = head.next;
        }
    }

    void display() {

        Node current = head;

        while (current != null) {

            System.out.print(current.data + " -> ");

            current = current.next;
        }

        System.out.println("null");
    }

    int countNodes() {

        int count = 0;

        Node current = head;

        while (current != null) {

            count++;

            current = current.next;
        }

        return count;
    }
}

class Main {

    public static void main(String[] args) {

        SinglyLinkedList list = new SinglyLinkedList();

        list.insertAtEnd(10);

        list.insertAtEnd(20);

        list.insertAtEnd(30);

        list.insertAtBeginning(5);

        System.out.println("Linked List:");

        list.display();

        System.out.println("Total Nodes: " + list.countNodes());

        list.deleteBeginning();

        System.out.println("After Deletion:");

        list.display();
    }
}
```

Output:

```text
Linked List:
5 -> 10 -> 20 -> 30 -> null

Total Nodes: 4

After Deletion:
10 -> 20 -> 30 -> null
```

## 5.17 Time and Space Complexity Summary

| Opearion            | Time Complxity     |
| ------------------- | ------------------ |
| Traversal           | $$\mathcal{O}(n)$$ |
| Searching           | $$\mathcal{O}(n)$$ |
| Counting Nodes      | $$\mathcal{O}(n)$$ |
| Insert at beginning | $$\mathcal{O}(1)$$ |
| Insert at end       | $$\mathcal{O}(n)$$ |
| Insert at position  | $$\mathcal{O}(n)$$ |
| Delete beginning    | $$\mathcal{O}(1)$$ |
| delete end          | $$\mathcal{O}(n)$$ |
| delete at position  | $$\mathcal{O}(n)$$ |

Space Complexity:

- Traversal: O(1) auxiliary space.
- Searching: O(1) auxiliary space.
- Insertion: O(1) auxiliary space, excluding new node.
- Deletion: O(1) auxiliary space.

## 5.18 Practice Questions

**Beginner Level**

1. Create a singly linked list.
2. Display all nodes.
3. Count the total number of nodes.
4. Search for an element.
5. Insert a node at the beginning.
6. Insert a node at the end.
7. Insert a node at a given position.
8. Delete the first node.
9. Delete the last node.
10. Delete a node at a given position.

**Intermediate Level**

1. Find the middle element of a linked list.
2. Find the nth node from the end.
3. Reverse a linked list.
4. Find the maximum element.
5. Find the minimum element.
6. Count occurrences of a given element.
7. Remove duplicates from a sorted linked list.
8. Find the length of a linked list recursively.

**Advanced Level**

1. Detect a loop in a linked list.
2. Find the starting point of a loop.
3. Merge two sorted linked lists.
4. Check whether a linked list is a palindrome.
5. Find the intersection point of two linked lists.
6. Remove the nth node from the end.

## 5.19 CDAC CMCE Preparation Points

Focus on these concepts:

1. Definition and structure of Linked Lists.
2. Difference between Arrays and Linked Lists.
3. Node and Head references.
4. Singly Linked List creation.
5. Traversal and searching.
6. Insertion at beginning and end.
7. Insertion at a specified position.
8. Deletion at beginning and end.
9. Deletion at a specified position.
10. Pointer/reference manipulation.
11. Time and Space Complexity.
12. Dry-run linked list operations.

**Important Java Concepts**

- Classes and objects
- Constructors
- Object references
- Null references
- Passing object references to methods
- Returning updated head references

---

# PART 2: Doubly Linked Lists and Circular Linked Lists

## 5.20 Introduction to Doubly Linked Lists

> A Doubly Linked List is a linear data structure in which each node contains three components:

1. Previous: Reference to the previous node.
2. Data: Stores the actual value.
3. Next: Reference to the next node.

Unlike a singly linked list, a doubly linked list supports traversal in both directions.

**Structure**

```text
null <- 10 <-> 20 <-> 30 -> null
```

Explanation:

- First node's previous reference is null.
- Last node's next reference is null.
- Each node maintains references to both previous and next nodes.

**Advantages**

1. Forward and backward traversal.
2. Easier deletion when a reference to the node is available.
3. Efficient insertion before or after a known node.
4. Useful for browser history and navigation systems.

**Disadvantages**

1. Requires extra memory for previous references.
2. More pointer manipulation.
3. Implementation is comparatively complex.

## 5.21 Node Structure of Doubly Linked List

Each node contains:

Previous reference.
Data.
Next reference.

**Java Implementation**

```java
class Node {

    int data;

    Node prev;

    Node next;

    Node(int data) {

        this.data = data;

        this.prev = null;

        this.next = null;
    }
}
```

**Creating Nodes**

```java
class Main {

    public static void main(String[] args) {

        Node first = new Node(10);

        Node second = new Node(20);

        Node third = new Node(30);

        first.next = second;

        second.prev = first;

        second.next = third;

        third.prev = second;
    }
}
```

Linked List;

```text
null <- 10 <-> 20 <-> 30 -> null
```

## 5.22 Creating a Doubly Linked List

We can create a doubly linked list by connecting the previous and next references.

**Java Implementation**

```java
class Node {

    int data;

    Node prev;

    Node next;

    Node(int data) {

        this.data = data;
    }
}

class Main {

    public static void main(String[] args) {

        Node head = new Node(10);

        Node second = new Node(20);

        Node third = new Node(30);

        head.next = second;

        second.prev = head;

        second.next = third;

        third.prev = second;

        System.out.println(head.data);

        System.out.println(head.next.data);

        System.out.println(head.next.next.data);
    }
}
```

Output:

```text
10
20
30
```

## 5.23 Traversal in Forward Direction

Forward traversal means visiting nodes from the first node to the last node.

**Approach**

1. Start from head.
2. Print current node data.
3. Move to current.next.
4. Continue until current becomes null.

**Java Implementation**

```java
class Main {

    static void displayForward(Node head) {

        Node current = head;

        while (current != null) {

            System.out.print(current.data + " ");

            current = current.next;
        }
    }
}
```

Example:

```text
10 <-> 20 <-> 30
```

```text
10 20 30
```

Time Complexity: $$\mathcal{O}(n)$$
Space Complexity: $$\mathcal{O}(1)$$

## 5.24 Traversal in Backward Direction

Backward traversal means visiting nodes from the last node to the first node.

**Approach**

1. Start from head.
2. Move to the last node.
3. Traverse using prev references.
4. Stop when current becomes null.

**Java Implementation**

```java
class Main {

    static void displayBackward(Node head) {

        if (head == null) {
            return;
        }

        Node current = head;

        // Move to last node

        while (current.next != null) {

            current = current.next;
        }

        // Traverse backward

        while (current != null) {

            System.out.print(current.data + " ");

            current = current.prev;
        }
    }
}
```

Output:

```text
30 20 10
```

Time Complexity: O(n)

Space Complexity: O(1)

## 5.25 Insertion at Beginning

Insert a new node before the current head.

Example

Original:

```text
10 <-> 20 <-> 30
```

Insert 5:

```text
5 <-> 10 <-> 20 <-> 3 <->
```

**Approach**

1. Create a new node.
2. Set newNode.next to head.
3. Set head.prev to newNode, if head exists.
4. Update head to newNode.

**Java Implementation**

```java
class DoublyLinkedList {

    Node head;

    void insertBeginning(int data) {

        Node newNode = new Node(data);

        if (head != null) {

            newNode.next = head;

            head.prev = newNode;
        }

        head = newNode;
    }
}
```

Time Complexity: $$\mathcal{O}(1)$$
Space Complexity: $$\mathcal{O}(1)$$ auxiliary space

## 5.26 Insertion at End

Insert a new node after the last node.

Example

Original:

```text
10 <-> 20 <-> 30 <->
```

Insert 40:

```text
10 <-> 20 <-> 30 <-> 40
```

**Java Implementation**

```java
class DoublyLinkedList {

    Node head;

    void insertEnd(int data) {

        Node newNode = new Node(data);

        if (head == null) {

            head = newNode;

            return;
        }

        Node current = head;

        while (current.next != null) {

            current = current.next;
        }

        current.next = newNode;

        newNode.prev = current;
    }
}
```

Time Complexity: O(n)

Space Complexity: O(1) auxiliary space.

Note: If a tail reference is maintained, insertion at the end becomes O(1).

## 5.27 Insertion at a Given Position

Insert a new node at a specified position.

Assume positions start from 1.

Example

Original:

```text
10 <-> 20 <-> 40
```

Insert 30 at position 3:

```text
10 <-> 20 <-> 30 <-> 40
```

**Java Implementation**

```java
class DoublyLinkedList {

    Node head;

    void insertAtPosition(int data, int position) {

        if (position < 1) {

            throw new IllegalArgumentException(
                "Invalid position"
            );
        }

        if (position == 1) {

            insertBeginning(data);

            return;
        }

        Node current = head;

        for (int i = 1; i < position - 1; i++) {

            if (current == null) {

                throw new IndexOutOfBoundsException();
            }

            current = current.next;
        }

        if (current == null) {

            throw new IndexOutOfBoundsException();
        }

        Node newNode = new Node(data);

        newNode.next = current.next;

        newNode.prev = current;

        if (current.next != null) {

            current.next.prev = newNode;
        }

        current.next = newNode;
    }
}
```

Time Complexity: O(n)

Space Complexity: O(1) auxiliary space.

## 5.28 Deletion from Beginning

Remove the first node.

Example

Original:

```text
10 <-> 20 <-> 30
```

After deletion:

```text
20 <-> 30
```

**Java Implementation**

```java
class DoublyLinkedList {

    Node head;

    void deleteBeginning() {

        if (head == null) {
            return;
        }

        head = head.next;

        if (head != null) {

            head.prev = null;
        }
    }
}
```

Time Complexity: O(1)

Space Complexity: O(1)

## 5.29 Deletion from End

Remove the last node.

Example

Original:

```text
10 <-> 20 <-> 30
```

After deletion:

```text
10 <-> 20
```

**Java Implementation**

```java
class DoublyLinkedList {

    Node head;

    void deleteEnd() {

        if (head == null) {
            return;
        }

        if (head.next == null) {

            head = null;

            return;
        }

        Node current = head;

        while (current.next != null) {

            current = current.next;
        }

        current.prev.next = null;
    }
}
```

Time Complexity: O(n)

Space Complexity: O(1)

## 5.30 Deletion at a Given Position

Delete a node at a specified position.

Assume positions start from 1.

Example

Original:

```text
10 <-> 20 <-> 30 <-> 40
```

Delete position 3:

```text
10 <-> 20 <-> 40
```

**Java Implementation**

```java
class DoublyLinkedList {

    Node head;

    void deleteAtPosition(int position) {

        if (position < 1) {

            throw new IllegalArgumentException(
                "Invalid position"
            );
        }

        if (head == null) {
            return;
        }

        if (position == 1) {

            deleteBeginning();

            return;
        }

        Node current = head;

        for (int i = 1; i < position; i++) {

            if (current == null) {

                throw new IndexOutOfBoundsException();
            }

            current = current.next;
        }

        if (current == null) {

            throw new IndexOutOfBoundsException();
        }

        current.prev.next = current.next;

        if (current.next != null) {

            current.next.prev = current.prev;
        }
    }
}
```

Time Complexity: O(n)

Space Complexity: O(1)

## 5.31 Complete Doubly Linked List Program

This program demonstrates:

- Insertion at beginning.
- Insertion at end.
- Deletion at beginning.
- Forward traversal.
- Backward traversal.

**Java Implementation**

```java
class Node {

    int data;

    Node prev;

    Node next;

    Node(int data) {

        this.data = data;
    }
}

class DoublyLinkedList {

    Node head;

    Node tail;

    void insertBeginning(int data) {

        Node newNode = new Node(data);

        if (head == null) {

            head = tail = newNode;

            return;
        }

        newNode.next = head;

        head.prev = newNode;

        head = newNode;
    }

    void insertEnd(int data) {

        Node newNode = new Node(data);

        if (tail == null) {

            head = tail = newNode;

            return;
        }

        tail.next = newNode;

        newNode.prev = tail;

        tail = newNode;
    }

    void deleteBeginning() {

        if (head == null) {
            return;
        }

        head = head.next;

        if (head == null) {

            tail = null;

        } else {

            head.prev = null;
        }
    }

    void displayForward() {

        Node current = head;

        while (current != null) {

            System.out.print(current.data + " ");

            current = current.next;
        }

        System.out.println();
    }

    void displayBackward() {

        Node current = tail;

        while (current != null) {

            System.out.print(current.data + " ");

            current = current.prev;
        }

        System.out.println();
    }
}

class Main {

    public static void main(String[] args) {

        DoublyLinkedList list = new DoublyLinkedList();

        list.insertEnd(10);

        list.insertEnd(20);

        list.insertEnd(30);

        list.insertBeginning(5);

        System.out.println("Forward:");

        list.displayForward();

        System.out.println("Backward:");

        list.displayBackward();

        list.deleteBeginning();

        System.out.println("After deletion:");

        list.displayForward();
    }
}
```

Output:

```text
Forward:
5 10 20 30

Backward:
30 20 10 5

After deletion:
10 20 30
```

---

## 5.32 Introduction to Circular Linked Lists

A Circular Linked List is a linked list in which the last node points back to the first node.

Unlike a normal linked list, the last node does not contain null in its next reference.

Example

```text
       10 -> 20 -> 30
       ^           |
       |___________|
```

**Characteristics**

1. The last node points to the first node.
2. No node necessarily points to null.
3. Traversal can start from any node.
4. Traversal must stop when we reach the starting node again.

**Applications**

- Round-robin scheduling.
- Multiplayer turn-based games.
- Circular queues.
- Repeated task scheduling.

## 5.33 Circular Singly Linked List

In a Circular Singly Linked List, each node contains:

- Data.
- Next reference.

The last node points to head.

**Structure**

```text
10 -> 20 -> 30
^           |
|___________|
```

**Java Node Implementation**

```java
class Node {

    int data;

    Node next;

    Node(int data) {

        this.data = data;
    }
}
```

**Creating a Circular Linked List**

```java
class Main {

    public static void main(String[] args) {

        Node first = new Node(10);

        Node second = new Node(20);

        Node third = new Node(30);

        first.next = second;

        second.next = third;

        third.next = first;
    }
}
```

Important:

```java
third.next = first;
```

This creates the circular connection.

## 5.34 Traversal in Circular Linked List

In a normal linked list, traversal stops when current becomes null.

In a circular linked list, traversal stops when current reaches head again.

**Java Implementation**

```java
class Main {

    static void display(Node head) {

        if (head == null) {
            return;
        }

        Node current = head;

        do {

            System.out.print(current.data + " ");

            current = current.next;

        } while (current != head);
    }
}
```

Output:

```text
10 20 30
```

**Important**: A do-while loop is commonly used because the first node must be visited before checking the stopping condition.

## 5.35 Insertion in Circular Singly Linked List

### A. Insertion at Beginning

Example

Original:

```text
10 -> 20 -> 30 -> back to 10
```

Insert 5:

```text
5 -> 10 -> 20 -> 30 -> back to 5
```

**Java Implementation**

```java
class CircularLinkedList {

    Node head;

    void insertBeginning(int data) {

        Node newNode = new Node(data);

        if (head == null) {

            head = newNode;

            newNode.next = head;

            return;
        }

        Node current = head;

        while (current.next != head) {

            current = current.next;
        }

        newNode.next = head;

        current.next = newNode;

        head = newNode;
    }
}
```

Time Complexity: O(n)

Space Complexity: O(1) auxiliary space.

### B. Insertion at End

**Java Implementation**

```java
class CircularLinkedList {

    Node head;

    void insertEnd(int data) {

        Node newNode = new Node(data);

        if (head == null) {

            head = newNode;

            newNode.next = head;

            return;
        }

        Node current = head;

        while (current.next != head) {

            current = current.next;
        }

        current.next = newNode;

        newNode.next = head;
    }
}

```

Time Complexity: O(n)

Space Complexity: O(1) auxiliary space.

Note: Maintaining a tail reference can make insertion at the end O(1).

## 5.36 Deletion in Circular Singly Linked List

### A. Deletion from Beginning

**Java Implementation**

```java
class CircularLinkedList {

    Node head;

    void deleteBeginning() {

        if (head == null) {
            return;
        }

        if (head.next == head) {

            head = null;

            return;
        }

        Node current = head;

        while (current.next != head) {

            current = current.next;
        }

        head = head.next;

        current.next = head;
    }
}
```

Time Complexity: O(n)

Space Complexity: O(1)

### B. Deletion from End

**Java Implementation**

```java
class CircularLinkedList {

    Node head;

    void deleteEnd() {

        if (head == null) {
            return;
        }

        if (head.next == head) {

            head = null;

            return;
        }

        Node current = head;

        while (current.next.next != head) {

            current = current.next;
        }

        current.next = head;
    }
}
```

Time Complexity: O(n)

Space Complexity: O(1)

## 5.37 Circular Doubly Linked List

A Circular Doubly Linked List combines:

- Doubly Linked List.
- Circular Linked List.

Each node contains:

- Previous reference.
- Data.
- Next reference.

The last node connects to the first node, and the first node connects backward to the last node.

Structure

```text
       <-----------------
       |                 |
      10 <-> 20 <-> 30
       |                 |
       ------------------>
```

For a list with head and tail:

```java
tail.next = head;

head.prev = tail;
```

**Java Node Structure**

```java
class Node {

    int data;

    Node prev;

    Node next;

    Node(int data) {

        this.data = data;
    }
}
```

**Creating a Circular Doubly Linked List**

```java
class Main {

    public static void main(String[] args) {

        Node first = new Node(10);

        Node second = new Node(20);

        Node third = new Node(30);

        first.next = second;

        second.prev = first;

        second.next = third;

        third.prev = second;

        third.next = first;

        first.prev = third;
    }
}
```

**Forward Traversal**

```java
static void displayForward(Node head) {

    if (head == null) {
        return;
    }

    Node current = head;

    do {

        System.out.print(current.data + " ");

        current = current.next;

    } while (current != head);
}
```

**Backward Traversal**

```java
static void displayBackward(Node head) {

    if (head == null) {
        return;
    }

    Node current = head.prev;

    Node tail = current;

    do {

        System.out.print(current.data + " ");

        current = current.prev;

    } while (current != tail);

}
```

## 5.38 Array vs Singly vs Doubly Linked List

| Feature             | Array              | Singly Linked List | Doubly Linked List |
| ------------------- | ------------------ | ------------------ | ------------------ |
| Memory              | Contigous          | Non-contigous      | Non-contigous      |
| Access              | $$\mathcal{O}(1)$$ | $$\mathcal{O}(n)$$ | $$\mathcal{O}(n)$$ |
| Foward Traversal    | Yes                | Yes                | Yes                |
| Backward Traversal  | Yes, using index   | No                 | Yes                |
| Insert at beginning | $$\mathcal{O}(n)$$ | $$\mathcal{O}(1)$$ | $$\mathcal{O}(1)$$ |
| Delete at beginning | $$\mathcal{O}(n)$$ | $$\mathcal{O}(1)$$ | $$\mathcal{O}(1)$$ |
| Extra references    | No link overhead   | One per node       | Two per node       |

# 5.39 Time and Space Complexity Summary

| Operations          | Singly Linked List     | Double Linked List |
| ------------------- | ---------------------- | ------------------ |
| Traversal           | $$\mathcal{O}(n)$$     | $$\mathcal{O}(n)$$ |
| Search              | $$\mathcal{O}(n)$$     | $$\mathcal{O}(n)$$ |
| Insert at beginning | $$\mathcal{O}(1)$$     | $$\mathcal{O}(1)$$ |
| Insert at End       | $$\mathcal{O}(n)$$     | $$\mathcal{O}(n)$$ |
| Delete Beginning    | $$\mathcal{O}(1)$$     | $$\mathcal{O}(1)$$ |
| Delete End          | $$\mathcal{O}(n)$$     | $$\mathcal{O}(n)$$ |
| Forward traversal   | $$\mathcal{O}(n)$$     | $$\mathcal{O}(n)$$ |
| Backward Traversal  | Not directly supported | $$\mathcal{O}(n)$$ |

With a maintained tail reference:

- Insert at end: O(1).
- Delete at end in a doubly linked list: O(1).

For circular linked lists, complexity depends on whether head and tail references are maintained.

## 5.40 Practice Questions

### A. Beginner Level

1. Create a doubly linked list.
2. Display elements in forward order.
3. elements in backward order.
4. Insert a node at the beginning.
5. Insert a node at the end.
6. Delete a node from the beginning.
7. Delete a node from the end.
8. Count nodes in a doubly linked list.

### B. Intermediate Level

1. Insert a node at a given position.
2. Delete a node at a given position.
3. Search for an element in a doubly linked list.
4. Reverse a doubly linked list.
5. Create a circular singly linked list.
6. Traverse a circular linked list.
7. Insert a node in a circular linked list.
8. Delete a node from a circular linked list.

### C. Advanced Level

1. Implement a circular doubly linked list.
2. Implement a deque using a doubly linked list.
3. Implement a browser history system.
4. Implement a music playlist using a circular doubly linked list.
5. Implement a round-robin scheduling simulation.
6. Reverse a circular linked list.
7. Split a circular linked list into two halves.
8. Check whether a linked list is circular.

## 5.41 CDAC CMCE Preparation Points

Focus on these concepts:

1. Doubly Linked List node structure.
2. Previous and next references.
3. Forward and backward traversal.
4. Insertion at beginning and end.
5. Deletion at beginning and end.
6. Insertion and deletion at a given position.
7. Circular Singly Linked List.
8. Circular Doubly Linked List.
9. Difference between singly and doubly linked lists.
10. Difference between normal and circular linked lists.
11. Applications of circular linked lists.
    12 Time and Space Complexity.

**Important Java Concepts**

- Object references.
- Constructors.
- Null references.
- .Reference manipulation.
- Class-based implementation.
- Returning updated head references.

---

## 5.42 Reverse a Singly Linked List

Reversing a linked list means changing the direction of all next references.

Example

Original:

```text
10 -> 20 -> 30 -> 40 -> null
```

After Reversal:

```text
40 -> 30 -> 20 -> 10 -> null
```

**Approach**: Three Pointer Technique
Use three pointers:

- prev: Stores the previous node.
- current: Stores the current node.
- next: Temporarily stores the next node.

**Algorithm**

1. Initialize prev = null.
2. Initialize current = head.
3. Store current.next in next.
4. Change current.next to prev.
5. Move prev to current.
6. Move current to next.
7. Repeat until current becomes null.
8. Return prev as the new head.

**Java Implementation**

```java
class Main {

    static Node reverse(Node head) {

        Node prev = null;

        Node current = head;

        Node next = null;

        while (current != null) {

            next = current.next;

            current.next = prev;

            prev = current;

            current = next;
        }

        return prev;
    }

    static void display(Node head) {

        while (head != null) {

            System.out.print(head.data + " ");

            head = head.next;
        }
    }

    public static void main(String[] args) {

        Node head = new Node(10);

        head.next = new Node(20);

        head.next.next = new Node(30);

        head.next.next.next = new Node(40);

        head = reverse(head);

        display(head);
    }

}
```

Output:

```text
40 30 20 10
```

Time Complexity: O(n)

Space Complexity: O(1)

**CDAC Important**: Understand the three-pointer technique and dry-run the algorithm.

## 5.43 Reverse a Linked List Recursively

A linked list can also be reversed using recursion.

**Approach**

1. Base condition: If head is null or only one node exists, return head.
2. Recursively reverse the remaining linked list.
3. the direction of the current node's next reference.
4. Return the new head.

**Java Implementation**

```java
class Main {

    static Node reverseRecursive(Node head) {

        if (head == null || head.next == null) {
            return head;
        }

        Node newHead = reverseRecursive(head.next);

        head.next.next = head;

        head.next = null;

        return newHead;
    }

}
```

Example:

```text
Original:
10 -> 20 -> 30 -> null

Reversed:
30 -> 20 -> 10 -> null
```

Time Complexity: O(n)

Space Complexity: O(n) due to recursion stack.

## 5.44 Find the Middle Element of a Linked List

Find the middle node without calculating the length of the linked list.

Example

Input:

```text
10 -> 20 -> 30 -> 40 -> 50 -> null
```

Output:

```text
30
```

**Approach**: Slow and Fast Pointer

Use two pointers:

- Slow moves one step.
- Fast moves two steps.

When fast reaches the end, slow points to the middle.

**Java Implementation**

```java
class Main {

    static int findMiddle(Node head) {

        if (head == null) {
            throw new IllegalArgumentException("Empty list");
        }

        Node slow = head;

        Node fast = head;

        while (fast != null && fast.next != null) {

            slow = slow.next;

            fast = fast.next.next;
        }

        return slow.data;
    }

}
```

Output:

```text
30
```

Time Complexity: O(n)

Space Complexity: O(1)

**Important**: For an even number of nodes, this implementation returns the second middle node.

## 5.45 Find the Nth Node from the End

Find the nth node from the end of a linked list.

Example

Input:

```text
10 -> 20 -> 30 -> 40 -> 50 -> null
```

N = 2

Output:

```text
40
```

**Approach**: Two Pointer Technique

1. Move the first pointer N steps ahead.
2. Start the second pointer from head.
3. Move both pointers one step at a time.
4. When the first pointer reaches null, the second pointer points to the required node.

**Java Implementation**

```java
class Main {

    static int nthFromEnd(Node head, int n) {

        if (n <= 0) {
            throw new IllegalArgumentException("Invalid N");
        }

        Node first = head;

        Node second = head;

        for (int i = 0; i < n; i++) {

            if (first == null) {
                throw new IllegalArgumentException(
                    "N is greater than list length"
                );
            }

            first = first.next;
        }

        while (first != null) {

            first = first.next;

            second = second.next;
        }

        return second.data;
    }

}
```

Time Complexity: O(n)

Space Complexity: O(1)

## 5.46 Detect a Loop in a Linked List

A loop exists when a node points to a previously visited node instead of null.

Example

```text
10 -> 20 -> 30 -> 40
          ^         |
          |_________|
```

Here, node 40 points back to node 20.

**Approach**: Floyd's Cycle Detection Algorithm

Also called the Tortoise and Hare Algorithm.

Use:

- Slow pointer: Moves one step.
- Fast pointer: Moves two steps.

If both pointers meet, a cycle exists.

**Java Implementation**

```java
class Main {

    static boolean detectLoop(Node head) {

        Node slow = head;

        Node fast = head;

        while (fast != null && fast.next != null) {

            slow = slow.next;

            fast = fast.next.next;

            if (slow == fast) {
                return true;
            }
        }

        return false;
    }

}
```

Output:

```text
true
```

Time Complexity: O(n)

Space Complexity: O(1)

**CDAC Important**: Floyd's Cycle Detection is a frequently tested linked list concept.

## 5.47 Find the Starting Point of a Loop

After detecting a cycle, find the node where the loop begins.

Example

```text
10 -> 20 -> 30 -> 40 -> 50
          ^              |
          |______________|

```

Loop starts at node 30.

**Approach**

1. Use slow and fast pointers.
2. Move slow by one step and fast by two steps.
3. When they meet, move slow back to head.
4. Move both pointers one step at a time.
5. Their next meeting point is the loop starting node.

**Java Implementation**

```java
class Main {

    static Node findLoopStart(Node head) {

        Node slow = head;

        Node fast = head;

        while (fast != null && fast.next != null) {

            slow = slow.next;

            fast = fast.next.next;

            if (slow == fast) {

                slow = head;

                while (slow != fast) {

                    slow = slow.next;

                    fast = fast.next;
                }

                return slow;
            }
        }

        return null;
    }

}
```

Time Complexity: O(n)

Space Complexity: O(1)

## 5.48 Remove a Loop from a Linked List

Remove a cycle so that the linked list becomes a normal singly linked list.

**Approach**

1. Detect the loop using Floyd's algorithm.
2. Find the starting node of the loop.
3. Traverse the loop until reaching the node whose next reference points to the loop starting node.
4. Set that next reference to null.

**Java Implementation**

```java
class Main {

    static void removeLoop(Node head) {

        Node loopStart = findLoopStart(head);

        if (loopStart == null) {
            return;
        }

        Node current = loopStart;

        while (current.next != loopStart) {

            current = current.next;
        }

        current.next = null;
    }

    static Node findLoopStart(Node head) {

        Node slow = head;

        Node fast = head;

        while (fast != null && fast.next != null) {

            slow = slow.next;

            fast = fast.next.next;

            if (slow == fast) {

                slow = head;

                while (slow != fast) {

                    slow = slow.next;

                    fast = fast.next;
                }

                return slow;
            }
        }

        return null;
    }

}
```

Time Complexity: O(n)

Space Complexity: O(1)

## 5.49 Find the Length of a Loop

Find how many nodes are present inside a cycle.

**Approach**

1. Detect a meeting point using slow and fast pointers.
2. If no meeting occurs, return zero.
3. Keep one pointer at the meeting node.
4. Move another pointer around the loop until it returns to the meeting node.
5. Count the number of steps.

**Java Implementation**

```java
class Main {

    static int loopLength(Node head) {

        Node slow = head;

        Node fast = head;

        while (fast != null && fast.next != null) {

            slow = slow.next;

            fast = fast.next.next;

            if (slow == fast) {

                int count = 1;

                Node current = slow.next;

                while (current != slow) {

                    count++;

                    current = current.next;
                }

                return count;
            }
        }

        return 0;
    }

}
```

Time Complexity: O(n)

Space Complexity: O(1)

## 5.50 Merge Two Sorted Linked Lists

Merge two sorted linked lists into a single sorted linked list.

Example

List 1:

```text
10 -> 30 -> 50
```

List 2:

```text
20 -> 40 -> 60
```

Output:

```text
10 -> 20 -> 30 -> 40 -> 50 -> 60
```

**Approach**

1. Compare the first nodes of both lists.
2. Select the smaller node.
3. Attach it to the result list.
4. the pointer of the selected list.
5. Repeat until one list becomes empty.
6. Attach the remaining nodes.

**Java Implementation**

```java
class Main {

    static Node merge(Node a, Node b) {

        Node dummy = new Node(0);

        Node tail = dummy;

        while (a != null && b != null) {

            if (a.data <= b.data) {

                tail.next = a;

                a = a.next;

            } else {

                tail.next = b;

                b = b.next;
            }

            tail = tail.next;
        }

        if (a != null) {
            tail.next = a;
        }

        if (b != null) {
            tail.next = b;
        }

        return dummy.next;
    }

}
```

Time Complexity: O(n + m)

Space Complexity: O(1) auxiliary space.

## 5.51 Remove Duplicates from a Sorted Linked List

Remove repeated elements from a sorted linked list.

Example

Input:

```text
10 -> 10 -> 20 -> 30 -> 30 -> 40
```

Output:

```text
10 -> 20 -> 30 -> 40
```

**Approach**

1. Since the list is sorted, duplicates are adjacent.
2. Compare the current node with the next node.
3. If both values are equal, skip the next node.

**Java Implementation**

```java
class Main {

    static Node removeDuplicates(Node head) {

        Node current = head;

        while (current != null && current.next != null) {

            if (current.data == current.next.data) {

                current.next = current.next.next;

            } else {

                current = current.next;
            }
        }

        return head;
    }

}
```

Time Complexity: O(n)

Space Complexity: O(1)

## 5.52 Remove Duplicates from an Unsorted Linked List

Remove duplicate values from an unsorted linked list.

Example

Input:

```text
10 -> 20 -> 10 -> 30 -> 20
```

Output:

```text
10 -> 20 -> 30
```

**Approach**: HashSet

1. Create a HashSet.
2. Traverse the linked list.
3. Insert values into the set.
4. If a value already exists, remove that node by changing the previous node's next reference.

**Java Implementation**

```java
import java.util.HashSet;

class Main {

    static Node removeDuplicates(Node head) {

        HashSet<Integer> set = new HashSet<>();

        Node current = head;

        Node prev = null;

        while (current != null) {

            if (set.contains(current.data)) {

                prev.next = current.next;

            } else {

                set.add(current.data);

                prev = current;
            }

            current = current.next;
        }

        return head;
    }

}
```

Time Complexity: O(n) average

Space Complexity: O(n)

## 5.53 Check Whether a Linked List is a Palindrome

A palindrome reads the same forward and backward.

Example

Input:

```text
10 -> 20 -> 30 -> 20 -> 10
```

Output:

```text
true
```

**Approach**

1. Find the middle node.
2. Reverse the second half.
3. Compare the first and second halves.
4. Return the comparison result.
5. Restore the original list if the operation must preserve the input.

**Java Implementation**

```java
class Main {

    static boolean isPalindrome(Node head) {

        if (head == null || head.next == null) {
            return true;
        }

        Node slow = head;

        Node fast = head;

        while (fast.next != null && fast.next.next != null) {

            slow = slow.next;

            fast = fast.next.next;
        }

        Node secondHalf = reverse(slow.next);

        Node firstHalf = head;

        Node current = secondHalf;

        boolean result = true;

        while (current != null) {

            if (firstHalf.data != current.data) {

                result = false;

                break;
            }

            firstHalf = firstHalf.next;

            current = current.next;
        }

        slow.next = reverse(secondHalf);

        return result;
    }

    static Node reverse(Node head) {

        Node prev = null;

        Node current = head;

        while (current != null) {

            Node next = current.next;

            current.next = prev;

            prev = current;

            current = next;
        }

        return prev;
    }

}
```

Time Complexity: O(n)

Space Complexity: O(1)

## 5.54 Find Intersection Point of Two Linked Lists

Two linked lists intersect when they share the same node object, not merely the same data value.

Example

```text
List A: 10 -> 20 \
                 40 -> 50
List B: 30 ------/

```

Intersection node = 40

**Approach**: Two Pointer Technique

1. Initialize pointer A at head A.
2. Initialize pointer B at head B.
3. Move both pointers one step at a time.
4. When a pointer reaches null, redirect it to the other list's head.
5. If an intersection exists, both pointers meet at the intersection node.

**Java Implementation**

```java
class Main {

    static Node intersection(Node headA, Node headB) {

        Node a = headA;

        Node b = headB;

        while (a != b) {

            a = (a == null) ? headB : a.next;

            b = (b == null) ? headA : b.next;
        }

        return a;
    }

}
```

Time Complexity: O(n + m)

Space Complexity: O(1)

**Important**: Compare node references using ==, not just their data values.

## 5.55 Find the Kth Node from the Beginning

Find the kth node using 1-based indexing.

Example

Input:

```text
10 -> 20 -> 30 -> 40 -> 50
```

K = 3

Output:

```text
30
```

**Java Implementation**

```java
class Main {

    static int kthNode(Node head, int k) {

        if (k <= 0) {
            throw new IllegalArgumentException("Invalid K");
        }

        Node current = head;

        int count = 1;

        while (current != null) {

            if (count == k) {
                return current.data;
            }

            count++;

            current = current.next;
        }

        throw new IllegalArgumentException(
            "K exceeds list length"
        );
    }

}
```

Time Complexity: O(n)

Space Complexity: O(1)

## 5.56 Delete a Node Without Head Reference

In certain interview problems, we are given only the node to be deleted, not the head reference.

The given node is guaranteed not to be the last node.

**Approach**

1. Copy the next node's data into the current node.
2. Change the current node's next reference to skip the next node.

**Java Implementation**

```java
class Main {

    static void deleteNode(Node node) {

        if (node == null || node.next == null) {
            throw new IllegalArgumentException(
                "Node cannot be deleted using this method"
            );
        }

        node.data = node.next.data;

        node.next = node.next.next;
    }

}
```

Time Complexity: O(1)

Space Complexity: O(1)

**Important**: This method does not physically delete the given node object. It replaces its value and removes the next node from the chain.

## 5.57 Find the Length of a Linked List Recursively

Recursion can also be used to count nodes.

**Java Implementation**

```java
class Main {

    static int length(Node head) {

        if (head == null) {
            return 0;
        }

        return 1 + length(head.next);
    }

}
```

Example:

```text
10 -> 20 -> 30 -> null
```

Output:

```text
3
```

Time Complexity: O(n)

Space Complexity: O(n) due to recursion stack.

## 5.58 Swap Nodes in Pairs

Swap every two adjacent nodes in a linked list.

Example

Input:

```text
10 -> 20 -> 30 -> 40 -> null
```

Output:

```text
20 -> 10 -> 40 -> 30 -> null
```

**Java Implementation**

```java
class Main {

    static Node swapPairs(Node head) {

        Node dummy = new Node(0);

        dummy.next = head;

        Node prev = dummy;

        while (prev.next != null &&
               prev.next.next != null) {

            Node first = prev.next;

            Node second = first.next;

            first.next = second.next;

            second.next = first;

            prev.next = second;

            prev = first;
        }

        return dummy.next;
    }

}
```

Time Complexity: O(n)

Space Complexity: O(1)

## 5.59 Rotate a Linked List

Rotate a linked list to the right by K positions.

Example

Input:

```text
10 -> 20 -> 30 -> 40 -> 50
```

K = 2

Output:

```text
40 -> 50 -> 10 -> 20 -> 30
```

Approach

1. Find the length of the list.
2. Reduce K using K % length.
3. Connect the last node to the head temporarily.
4. Find the new tail.
5. Break the circular connection.
6. Return the new head.

**Java Implementation**

```java
class Main {

    static Node rotateRight(Node head, int k) {

        if (head == null || head.next == null || k == 0) {
            return head;
        }

        int length = 1;

        Node tail = head;

        while (tail.next != null) {

            tail = tail.next;

            length++;
        }

        k = k % length;

        if (k == 0) {
            return head;
        }

        tail.next = head;

        int steps = length - k;

        Node newTail = head;

        for (int i = 1; i < steps; i++) {

            newTail = newTail.next;
        }

        Node newHead = newTail.next;

        newTail.next = null;

        return newHead;
    }

}
```

Time Complexity: O(n)

Space Complexity: O(1)

## 5.60 Important Linked List Problem Patterns

| Pattern                | Important Problems     |
| ---------------------- | ---------------------- |
| Three Pointers         | Reverse Linked List    |
| Slow and Fast Pointers | Middle Element         |
| Two Pointers           | Nth Node from End      |
| Floyd's Algorithm      | Cycle Detection        |
| Two Pointer Switching  | Intersection Point     |
| Dummy Node             | Merge Two Sorted Lists |
| HashSet                | Remove Duplicates      |
| Reverse Second Half    | Palindrome Linked List |
| Recursion              | Recursive Reversal     |
| Pointer Manipulation   | Swap Nodes in Pairs    |

## 5.61 Time and Space Complexity Summary

| Problem                    | Time Complexity | Space Complexity |
| -------------------------- | --------------- | ---------------- |
| Reverse Iteratively        | O(n)            | O(1)             |
| Reverse Recursively        | O(n)            | O(n)             |
| Find Middle                | O(n)            | O(1)             |
| Nth Node from End          | O(n)            | O(1)             |
| Detect Loop                | O(n)            | O(1)             |
| Find Loop Start            | O(n)            | O(1)             |
| Remove Loop                | O(n)            | O(1)             |
| Find Loop Length           | O(n)            | O(1)             |
| Merge Sorted Lists         | O(n + m)        | O(1) auxiliary   |
| Remove Sorted Duplicates   | O(n)            | O(1)             |
| Remove Unsorted Duplicates | O(n) average    | O(n)             |
| Palindrome Check           | O(n)            | O(1)             |
| Find Intersection          | O(n + m)        | O(1)             |
| Kth Node                   | O(n)            | O(1)             |
| Delete Given Node          | O(1)            | O(1)             |
| Recursive Length           | O(n)            | O(n)             |
| Swap Pairs                 | O(n)            | O(1)             |
| Rotate List                | O(n)            | O(1)             |

## 5.62 Practice Questions

**A. Beginner Level**

1. Reverse a singly linked list.
2. Find the middle element.
3. Find the kth node from the beginning.
4. Count nodes recursively.
5. Find the maximum element.
6. Find the minimum element.
7. Find the nth node from the end.
8. Remove duplicates from a sorted list.

**B. Intermediate Level**

1. Detect a cycle in a linked list.
2. Find the starting point of a cycle.
3. Find the length of a cycle.
4. Remove a cycle.
5. Merge two sorted linked lists.
6. Check whether a linked list is a palindrome.
7. Find the intersection point of two linked lists.
8. Remove duplicates from an unsorted linked list.

**C. Advanced Level**

1. Reverse a linked list recursively.
2. Reverse a linked list in groups of K.
3. Rotate a linked list by K positions.
4. Swap nodes in pairs.
5. Sort a linked list using merge sort.
6. Add two numbers represented by linked lists.
7. Flatten a multilevel linked list.
8. Reorder a linked list.
9. Find the intersection of two linked lists without extra space.

## 5.63 CDAC CMCE Preparation Points

Prioritize these problems:

1. Reverse Linked List.
2. Find Middle Element.
3. Nth Node from End.
4. Floyd's Cycle Detection.
5. Find Starting Point of Cycle.
6. Remove Loop.
7. Merge Two Sorted Linked Lists.
8. Remove Duplicates.
9. Check Palindrome.
10. Find Intersection Point.
11. Recursive Linked List Traversal.
12. Swap Nodes in Pairs.
13. Rotate Linked List.

**Important Concepts to Understand**

- Slow and fast pointers.
- Three-pointer reversal.
- Dummy node technique.
- Reference comparison.
- Recursion.
- Loop detection.
- In-place modification.
- Time and space complexity.

### PART 3: QUICK REVISION

| Algorithm          | Main Technique                   |
| ------------------ | -------------------------------- |
| Reverse List       | Three Pointers                   |
| Find Middle        | Slow and Fast                    |
| Nth from End       | Two Pointers                     |
| Detect Loop        | Floyd's Algorithm                |
| Find Loop Start    | Floyd's Algorithm                |
| Remove Loop        | Cycle Detection + Pointer Update |
| Merge Sorted Lists | Dummy Node                       |
| Remove Duplicates  | HashSet / Adjacent Comparison    |
| Palindrome         | Reverse Second Half              |
| Intersection       | Two Pointer Switching            |
| Swap Pairs         | Pointer Manipulation             |
| Rotate List        | Circular Connection              |

---

## PART 4: Advanced Linked List Problems, Applications, and Interview Preparation

## 5.64 Introduction to Advanced Linked List Problems

Advanced linked list problems require understanding how references connect nodes.

Important techniques include:

1. Multiple pointer manipulation.
2. Slow and fast pointer techniques.
3. Recursion.
4. Merge sort.
5. In-place reversal.
6. Dummy node technique.
7. HashMap and HashSet.
8. Doubly linked list operations.

These concepts are useful for CDAC CMCE preparation, coding assessments, and technical interviews.

## 5.65 Reverse Linked List in Groups of K

Reverse every K nodes in a linked list.

Example

Input:

```text
1 -> 2 -> 3 -> 4 -> 5 -> 6 -> 7 -> 8
```

K = 3

Output:

```text
3 -> 2 -> 1 -> 6 -> 5 -> 4 -> 8 -> 7
```

**Approach**

1. Reverse the first K nodes.
2. Recursively process the remaining list.
3. Connect the reversed group to the remaining list.
4. Return the new head.

**Java Implementation**

```java
class Main {

    static Node reverseKGroup(Node head, int k) {

        if (head == null || k <= 1) {
            return head;
        }

        Node current = head;

        Node prev = null;

        Node next = null;

        int count = 0;

        while (current != null && count < k) {

            next = current.next;

            current.next = prev;

            prev = current;

            current = next;

            count++;
        }

        if (next != null) {

            head.next = reverseKGroup(next, k);
        }

        return prev;
    }

}
```

Time Complexity: O(n)

Space Complexity: O(n/k) recursion stack.

**Note**: This implementation also reverses the final group if it contains fewer than K nodes.

## 5.66 Sort a Linked List Using Merge Sort

Merge Sort is an efficient sorting algorithm for linked lists.

It follows the Divide and Conquer technique.

Example

Input:

```text
40 -> 10 -> 30 -> 20
```

Output:

```text
10 -> 20 -> 30 -> 40
```

**Approach**

1. Divide the linked list into two halves.
2. Recursively sort both halves.
3. Merge the two sorted lists.
4. Return the sorted head.

**Java Implementation**

```java
class Main {

    static Node mergeSort(Node head) {

        if (head == null || head.next == null) {
            return head;
        }

        Node middle = getMiddle(head);

        Node secondHalf = middle.next;

        middle.next = null;

        Node left = mergeSort(head);

        Node right = mergeSort(secondHalf);

        return merge(left, right);
    }

    static Node getMiddle(Node head) {

        Node slow = head;

        Node fast = head.next;

        while (fast != null && fast.next != null) {

            slow = slow.next;

            fast = fast.next.next;
        }

        return slow;
    }

    static Node merge(Node a, Node b) {

        Node dummy = new Node(0);

        Node tail = dummy;

        while (a != null && b != null) {

            if (a.data <= b.data) {

                tail.next = a;

                a = a.next;

            } else {

                tail.next = b;

                b = b.next;
            }

            tail = tail.next;
        }

        tail.next = (a != null) ? a : b;

        return dummy.next;
    }

}
```

Time Complexity: O(n log n)

Space Complexity: O(log n) due to recursion stack.

**Important**: Merge Sort is generally preferred over Quick Sort for linked list sorting.

## 5.67 Add Two Numbers Represented by Linked Lists

Two numbers are represented using linked lists.

Each node contains one digit.

The digits are stored in reverse order.

Example

Number 1:

```text
2 -> 4 -> 3
```

Represents 342.

Number 2:

```text
5 -> 6 -> 4
```

Represents 465.

Output:

```text
7 -> 0 -> 8
```

Represents 807.

**Approach**

1. Initialize carry = 0.
2. Add corresponding digits.
3. Calculate digit using sum % 10.
4. Calculate carry using sum / 10.
5. Continue until both lists and carry are exhausted.

**Java Implementation**

```java
class Main {

    static Node addNumbers(Node l1, Node l2) {

        Node dummy = new Node(0);

        Node current = dummy;

        int carry = 0;

        while (l1 != null || l2 != null || carry != 0) {

            int sum = carry;

            if (l1 != null) {

                sum += l1.data;

                l1 = l1.next;
            }

            if (l2 != null) {

                sum += l2.data;

                l2 = l2.next;
            }

            carry = sum / 10;

            current.next = new Node(sum % 10);

            current = current.next;
        }

        return dummy.next;
    }

}
```

Time Complexity: O(max(n, m))

Space Complexity: O(max(n, m)) for the result list.

## 5.68 Subtract Two Numbers Represented by Linked Lists

Two numbers can also be represented using linked lists in forward digit order.

Example:

```text
Number 1: 5 -> 6 -> 3
Number 2: 1 -> 2 -> 3
```

Result:

```text
4 -> 4 -> 0
```

Represents 563 - 123 = 440.

**Approach**

1. Compare the two numbers.
2. Identify the larger number.
3. Subtract digit by digit from right to left.
4. Handle borrowing.
5. Remove leading zeros from the result.

A common approach is to reverse both lists, subtract corresponding digits, and reverse the result.

**Java Implementation**

```java
class Main {

    static Node subtract(Node a, Node b) {

        a = reverse(a);

        b = reverse(b);

        Node dummy = new Node(0);

        Node tail = dummy;

        int borrow = 0;

        while (a != null) {

            int x = a.data;

            int y = (b != null) ? b.data : 0;

            int difference = x - y - borrow;

            if (difference < 0) {

                difference += 10;

                borrow = 1;

            } else {

                borrow = 0;
            }

            tail.next = new Node(difference);

            tail = tail.next;

            a = a.next;

            if (b != null) {
                b = b.next;
            }
        }

        Node result = reverse(dummy.next);

        while (result != null &&
               result.data == 0 &&
               result.next != null) {

            result = result.next;
        }

        return result;
    }

    static Node reverse(Node head) {

        Node prev = null;

        while (head != null) {

            Node next = head.next;

            head.next = prev;

            prev = head;

            head = next;
        }

        return prev;
    }

}
```

**Assumption**: The first input number is greater than or equal to the second, and both lists contain valid decimal digits without leading zeros.

Time Complexity: O(n + m)

Space Complexity: O(1) auxiliary space, excluding the returned result.

## 5.69 Reorder a Linked List

Reorder a linked list in the following manner:

L0 → Ln → L1 → Ln-1 → L2 → Ln-2

Example

Input:

```text
1 -> 2 -> 3 -> 4 -> 5
```

Output:

```text
1 -> 5 -> 2 -> 4 -> 3
```

**Approach**

1. Find the middle node.
2. Reverse the second half.
3. Merge both halves alternately.

**Java Implementation**

```java
class Main {

    static void reorder(Node head) {

        if (head == null || head.next == null) {
            return;
        }

        Node slow = head;

        Node fast = head;

        while (fast.next != null &&
               fast.next.next != null) {

            slow = slow.next;

            fast = fast.next.next;
        }

        Node second = reverse(slow.next);

        slow.next = null;

        Node first = head;

        while (second != null) {

            Node temp1 = first.next;

            Node temp2 = second.next;

            first.next = second;

            second.next = temp1;

            first = temp1;

            second = temp2;
        }
    }

    static Node reverse(Node head) {

        Node prev = null;

        while (head != null) {

            Node next = head.next;

            head.next = prev;

            prev = head;

            head = next;
        }

        return prev;
    }

}
```

Time Complexity: O(n)

Space Complexity: O(1)

## 5.70 Segregate Even and Odd Nodes

Arrange even-valued nodes before odd-valued nodes.

Example

Input:

```text
17 -> 15 -> 8 -> 12 -> 10 -> 5
```

Output:

```text
8 -> 12 -> 10 -> 17 -> 15 -> 5
```

**Java Implementation**

```java
class Main {

    static Node segregate(Node head) {

        Node evenStart = null;

        Node evenEnd = null;

        Node oddStart = null;

        Node oddEnd = null;

        Node current = head;

        while (current != null) {

            Node next = current.next;

            current.next = null;

            if (current.data % 2 == 0) {

                if (evenStart == null) {

                    evenStart = evenEnd = current;

                } else {

                    evenEnd.next = current;

                    evenEnd = current;
                }

            } else {

                if (oddStart == null) {

                    oddStart = oddEnd = current;

                } else {

                    oddEnd.next = current;

                    oddEnd = current;
                }
            }

            current = next;
        }

        if (evenStart == null) {
            return oddStart;
        }

        evenEnd.next = oddStart;

        return evenStart;
    }

}
```

Time Complexity: O(n)

Space Complexity: O(1)

**Note**: This preserves the original relative order of even and odd nodes.

## 5.71 Move Last Node to First

Move the last node of a linked list to the beginning.

Example

Input:

```text
10 -> 20 -> 30 -> 40
```

Output:

```text
40 -> 10 -> 20 -> 30
```

**Java Implementation**

```java
class Main {

    static Node moveLastToFirst(Node head) {

        if (head == null || head.next == null) {
            return head;
        }

        Node current = head;

        while (current.next.next != null) {

            current = current.next;
        }

        Node last = current.next;

        current.next = null;

        last.next = head;

        return last;
    }

}
```

Time Complexity: O(n)

Space Complexity: O(1)

## 5.72 Pairwise Swap Elements

Swap adjacent elements in pairs.

Example

Input:

```text
10 -> 20 -> 30 -> 40
```

Output:

```text
20 -> 10 -> 40 -> 30
```

This problem is also covered in Part 3.

**Important Interview Concept**

There are two approaches:

1. Swap node data.
2. Swap actual node references.

Swapping actual nodes is generally more appropriate when the problem requires changing the linked list structure.

Time Complexity: O(n)

Space Complexity: O(1)

## 5.73 Find the Intersection of Two Sorted Linked Lists

Given two sorted linked lists, create a new list containing only their common elements.

Example

List 1:

```text
10 -> 20 -> 30 -> 40 -> 50
```

List 2:

```text
20 -> 40 -> 60
```

Output:

```text
20 -> 40
```

**Java Implementation**

```java
class Main {

    static Node intersection(Node a, Node b) {

        Node dummy = new Node(0);

        Node tail = dummy;

        while (a != null && b != null) {

            if (a.data == b.data) {

                tail.next = new Node(a.data);

                tail = tail.next;

                a = a.next;

                b = b.next;

            } else if (a.data < b.data) {

                a = a.next;

            } else {

                b = b.next;
            }
        }

        return dummy.next;
    }

}
```

Time Complexity: O(n + m)

Space Complexity: O(k), where k is the number of common elements in the result.

## 5.74 Remove Nodes Having Greater Value on Right

Remove every node that has a node with a greater value somewhere to its right.

Example

Input:

```text
12 -> 15 -> 10 -> 11 -> 5 -> 6 -> 2 -> 3
```

Output:

```text
15 -> 11 -> 6 -> 3
```

**Approach**

1. Reverse the linked list.
2. Keep track of the maximum value seen.
3. Remove nodes smaller than the maximum.
4. Reverse the list again.

**Java Implementation**

```java
class Main {

    static Node removeNodes(Node head) {

        head = reverse(head);

        Node current = head;

        int max = Integer.MIN_VALUE;

        Node prev = null;

        while (current != null) {

            if (current.data >= max) {

                max = current.data;

                prev = current;

                current = current.next;

            } else {

                prev.next = current.next;

                current = current.next;
            }
        }

        return reverse(head);
    }

    static Node reverse(Node head) {

        Node prev = null;

        while (head != null) {

            Node next = head.next;

            head.next = prev;

            prev = head;

            head = next;
        }

        return prev;
    }

}
```

Time Complexity: O(n)

Space Complexity: O(1)

## 5.75 Delete N Nodes After M Nodes

Given M and N:

- Keep M nodes.
- Delete the next N nodes.
- Repeat until the list ends.

Example

Input:

```text
1 -> 2 -> 3 -> 4 -> 5 -> 6 -> 7 -> 8
```

M = 2, N = 2

Output:

```text
1 -> 2 -> 5 -> 6
```

**Java Implementation**

```java
class Main {

    static Node deleteNAfterM(Node head, int m, int n) {

        if (m <= 0) {
            return null;
        }

        if (n <= 0) {
            return head;
        }

        Node current = head;

        while (current != null) {

            for (int i = 1; i < m && current != null; i++) {

                current = current.next;
            }

            if (current == null) {
                break;
            }

            Node temp = current.next;

            for (int i = 0; i < n && temp != null; i++) {

                temp = temp.next;
            }

            current.next = temp;

            current = temp;
        }

        return head;
    }

}
```

Time Complexity: O(n), where n is the number of nodes in the list.

Space Complexity: O(1)

## 5.76 Split a Linked List into Two Halves

Divide a singly linked list into two approximately equal halves.

Example

Input:

```text
10 -> 20 -> 30 -> 40 -> 50
```

Output:

```text
First: 10 -> 20 -> 30

Second: 40 -> 50
```

**Java Implementation**

```java
class Main {

    static Node[] split(Node head) {

        if (head == null) {
            return new Node[]{null, null};
        }

        Node slow = head;

        Node fast = head.next;

        while (fast != null && fast.next != null) {

            slow = slow.next;

            fast = fast.next.next;
        }

        Node second = slow.next;

        slow.next = null;

        return new Node[]{head, second};
    }

}
```

Time Complexity: O(n)

Space Complexity: O(1) auxiliary space, excluding the returned array.

## 5.77 Split Circular Linked List into Two Halves

Divide a circular linked list into two circular linked lists.

Example

Input:

```text
10 -> 20 -> 30 -> 40 -> 50 -> back to 10
```

Output:

```text
First:
10 -> 20 -> 30 -> back to 10

Second:
40 -> 50 -> back to 40
```

**Java Implementation**

```java
class Main {

    static Node[] splitCircular(Node head) {

        if (head == null) {
            return new Node[]{null, null};
        }

        if (head.next == head) {
            return new Node[]{head, null};
        }

        Node slow = head;

        Node fast = head;

        while (fast.next != head &&
               fast.next.next != head) {

            slow = slow.next;

            fast = fast.next.next;
        }

        if (fast.next.next == head) {
            fast = fast.next;
        }

        Node head1 = head;

        Node head2 = slow.next;

        slow.next = head1;

        fast.next = head2;

        return new Node[]{head1, head2};
    }

}
```

Time Complexity: O(n)

Space Complexity: O(1) auxiliary space, excluding the returned array.

## 5.78 Flatten a Multilevel Linked List

A multilevel linked list contains nodes with additional child references.

The objective is to convert it into a single-level linked list.

Example

```text
1 -> 2 -> 3 -> 4
     |
     7 -> 8 -> 9
```

Flattened:

```text
1 -> 2 -> 7 -> 8 -> 9 -> 3 -> 4
```

**Important Concepts**

- Multilevel node structure.
- Child references.
- Recursive traversal.
- .Depth-first traversal.
- manipulation.

The exact implementation depends on the node structure and whether the problem requires depth-first or level-order flattening.

## 5.79 Clone a Linked List with Random Pointer

Each node contains three references:

- Next.
- Random.
- Data.

The random reference can point to any node or null.

Example

```text
Node 1:
next -> Node 2
random -> Node 3
```

The objective is to create a deep copy of the list.

**Approach**: HashMap

1. Create a HashMap mapping original nodes to cloned nodes.
2. Traverse the original list and create all cloned nodes.
3. Traverse again and assign next and random references.
4. Return the cloned head.

**Java Implementation**

```java
import java.util.HashMap;
import java.util.Map;

class RandomNode {

    int data;

    RandomNode next;

    RandomNode random;

    RandomNode(int data) {

        this.data = data;
    }

}

class Main {

    static RandomNode cloneList(RandomNode head) {

        if (head == null) {
            return null;
        }

        Map<RandomNode, RandomNode> map = new HashMap<>();

        RandomNode current = head;

        while (current != null) {

            map.put(current, new RandomNode(current.data));

            current = current.next;
        }

        current = head;

        while (current != null) {

            RandomNode copy = map.get(current);

            copy.next = map.get(current.next);

            copy.random = map.get(current.random);

            current = current.next;
        }

        return map.get(head);
    }

}
```

Time Complexity: O(n) average

Space Complexity: O(n)

## 5.80 LRU Cache Using Doubly Linked List

LRU means Least Recently Used.

An LRU Cache removes the least recently used item when its capacity is full.

Example

Capacity = 2

Operations:

```text
put(1, 10)
put(2, 20)
get(1)
put(3, 30)
```

Since key 1 was recently accessed, key 2 is removed.

Data Structures Used

- HashMap.
- Doubly Linked List.

**Why Doubly Linked List?**

It supports:

- Removing a node in O(1).
- Moving a node to the front in O(1).
- Inserting a node at the front in O(1).
- Removing the least recently used node from the end in O(1).

**Java Implementation**

```java
import java.util.HashMap;

class LRUCache {

    class Node {

        int key;

        int value;

        Node prev;

        Node next;

        Node(int key, int value) {

            this.key = key;

            this.value = value;
        }
    }

    private int capacity;

    private HashMap<Integer, Node> map;

    private Node head;

    private Node tail;

    LRUCache(int capacity) {

        this.capacity = capacity;

        map = new HashMap<>();

        head = new Node(0, 0);

        tail = new Node(0, 0);

        head.next = tail;

        tail.prev = head;
    }

    void remove(Node node) {

        node.prev.next = node.next;

        node.next.prev = node.prev;
    }

    void addFirst(Node node) {

        node.next = head.next;

        node.prev = head;

        head.next.prev = node;

        head.next = node;
    }

    int get(int key) {

        if (!map.containsKey(key)) {
            return -1;
        }

        Node node = map.get(key);

        remove(node);

        addFirst(node);

        return node.value;
    }

    void put(int key, int value) {

        if (map.containsKey(key)) {

            Node node = map.get(key);

            node.value = value;

            remove(node);

            addFirst(node);

            return;
        }

        if (map.size() == capacity) {

            Node lru = tail.prev;

            remove(lru);

            map.remove(lru.key);
        }

        Node node = new Node(key, value);

        map.put(key, node);

        addFirst(node);
    }

}
```

Time Complexity:

| Operation  | Complexity   |
| ---------- | ------------ |
| get()      | O(1) average |
| put()      | O(1) average |
| remove()   | O(1)         |
| addFirst() | O(1)         |

**Important**: LRU Cache is a popular application of HashMap and Doubly Linked List.

For production use, validate that capacity is positive and handle capacity-zero cases explicitly.

## 5.81 Implement Stack Using Linked List

A Stack follows LIFO:

Last In, First Out.

Operations

- push()
- pop()
- peek()
- isEmpty()

**Java Implementation**

```java
class Stack {

    Node top;

    void push(int data) {

        Node newNode = new Node(data);

        newNode.next = top;

        top = newNode;
    }

    int pop() {

        if (top == null) {

            throw new RuntimeException("Stack is empty");
        }

        int value = top.data;

        top = top.next;

        return value;
    }

    int peek() {

        if (top == null) {

            throw new RuntimeException("Stack is empty");
        }

        return top.data;
    }

    boolean isEmpty() {

        return top == null;
    }

}
```

Time Complexity:

| Operation | Complexity |
| --------- | ---------- |
| push      | O(1)       |
| pop       | O(1)       |
| peek      | O(1)       |
| isEmpty   | O(1)       |

## 5.82 Implement Queue Using Linked List

A Queue follows FIFO:

First In, First Out.

Operations

- enqueue()
- dequeue()
- peek()
- isEmpty()

**Java Implementation**

```java
class Queue {

    Node front;

    Node rear;

    void enqueue(int data) {

        Node newNode = new Node(data);

        if (rear == null) {

            front = rear = newNode;

            return;
        }

        rear.next = newNode;

        rear = newNode;
    }

    int dequeue() {

        if (front == null) {

            throw new RuntimeException("Queue is empty");
        }

        int value = front.data;

        front = front.next;

        if (front == null) {

            rear = null;
        }

        return value;
    }

    int peek() {

        if (front == null) {

            throw new RuntimeException("Queue is empty");
        }

        return front.data;
    }

    boolean isEmpty() {

        return front == null;
    }

}
```

Time Complexity:

| Operation | Complexity |
| --------- | ---------- |
| enqueue   | O(1)       |
| dequeue   | O(1)       |
| peek      | O(1)       |
| isEmpty   | O(1)       |

## 5.83 Linked List Applications

Linked Lists are used in many real-world applications.

| Application             | Linked List Type              |
| ----------------------- | ----------------------------- |
| Browser History         | Doubly Linked List            |
| Music Playlist          | Doubly / Circular Linked List |
| Undo and Redo           | Doubly Linked List            |
| LRU Cache               | Doubly Linked List            |
| Round Robin Scheduling  | Circular Linked List          |
| Stack Implementation    | Singly Linked List            |
| Queue Implementation    | Singly Linked List            |
| Graph Adjacency List    | Linked List / Collection      |
| Memory Management       | Linked Structures             |
| Image Viewer Navigation | Doubly Linked List            |

## 5.84 Java Collections Framework: LinkedList

Java provides a built-in LinkedList class in the Collections Framework.

Package:

```java
import java.util.LinkedList;
```

Java LinkedList implements:

- List interface.
- Deque interface.
- Queue interface.

It is implemented as a doubly linked list.

**Creating a LinkedList**

```java
import java.util.LinkedList;

class Main {

    public static void main(String[] args) {

        LinkedList<Integer> list = new LinkedList<>();

        list.add(10);

        list.add(20);

        list.add(30);

        System.out.println(list);
    }

}
```

Output:

```text
[10, 20, 30]
```

**Important Methods**

| Method          | Description                         |
| --------------- | ----------------------------------- |
| add(value)      | Adds element at end                 |
| addFirst(value) | Adds element at beginning           |
| addLast(value)  | Adds element at end                 |
| remove()        | Removes first element               |
| removeFirst()   | Removes first element               |
| removeLast()    | Removes last element                |
| get(index)      | Returns element at index            |
| getFirst()      | Returns first element               |
| getLast()       | Returns last element                |
| peek()          | Retrieves first element             |
| poll()          | Retrieves and removes first element |
| size()          | Returns number of elements          |
| contains(value) | Checks whether value exists         |
| clear()         | Removes all elements                |

Example Program

```java
import java.util.LinkedList;

class Main {

    public static void main(String[] args) {

        LinkedList<Integer> list = new LinkedList<>();

        list.add(10);

        list.add(20);

        list.add(30);

        list.addFirst(5);

        list.addLast(40);

        System.out.println(list);

        list.removeFirst();

        list.removeLast();

        System.out.println(list);

        System.out.println(list.getFirst());

        System.out.println(list.getLast());

        System.out.println(list.size());
    }

}
```

Output:

```text
[5, 10, 20, 30, 40]

[10, 20, 30]

10

30

3
```

**ArrayList vs LinkedList**

| Feature              | ArrayList     | LinkedList         |
| -------------------- | ------------- | ------------------ |
| Internal Structure   | Dynamic Array | Doubly Linked List |
| Random Access        | O(1)          | O(n)               |
| Add at Beginning     | O(n)          | O(1)               |
| Remove First         | O(n)          | O(1)               |
| Add at End Amortized | O(1)          | O(1)               |
| Memory Overhead      | Lower         | Higher             |
| Implements List      | Yes           | Yes                |
| Implements Deque     | No            | Yes                |

**Important**: Java LinkedList does not provide O(1) indexed access. Methods such as get(index) require traversal.

## 5.85 Important Interview Questions

**Basic Questions**

1. What is a Linked List?
2. What are the different types of Linked Lists?
3. What is the difference between Array and Linked List?
4. What is the difference between Singly and Doubly Linked List?
5. What are the advantages of Linked Lists?
6. What are the disadvantages of Linked Lists?
7. Why is random access not possible in a Linked List?
8. What is the time complexity of searching?

**Intermediate Questions**

1. How do you reverse a Linked List?
2. How do you find the middle node?
3. How do you detect a loop?
4. What is Floyd's Cycle Detection Algorithm?
5. How do you find the nth node from the end?
6. How do you merge two sorted linked lists?
7. How do you remove duplicates?
8. How do you check whether a Linked List is a palindrome?
9. How do you find the intersection point?
10. How do you reverse a Doubly Linked List?

**Advanced Questions**

1. How do you reverse nodes in groups of K?
2. How do you sort a Linked List?
3. How do you clone a Linked List with random pointers?
4. How do you implement LRU Cache?
5. How do you add two numbers represented by Linked Lists?
6. How do you reorder a Linked List?
7. How do you flatten a multilevel Linked List?
8. How do you split a Circular Linked List?
9. How do you implement Stack using Linked List?
10. How do you implement Queue using Linked List?

## 5.86 Time and Space Complexity Summary

| Problem                  | Time Complexity | Auxiliary Space    |
| ------------------------ | --------------- | ------------------ |
| Reverse in K Groups      | O(n)            | O(n/k)             |
| Merge Sort               | O(n log n)      | O(log n)           |
| Add Two Numbers          | O(max(n,m))     | O(1) auxiliary     |
| Subtract Two Numbers     | O(n+m)          | O(1) auxiliary     |
| Reorder List             | O(n)            | O(1)               |
| Segregate Even and Odd   | O(n)            | O(1)               |
| Move Last to First       | O(n)            | O(1)               |
| Sorted List Intersection | O(n+m)          | O(k)               |
| Remove Greater Nodes     | O(n)            | O(1)               |
| Delete N After M         | O(n)            | O(1)               |
| Split List               | O(n)            | O(1) auxiliary     |
| Clone Random List        | O(n)            | O(n)               |
| LRU Cache get            | O(1) average    | O(1) per operation |
| LRU Cache put            | O(1) average    | O(1) per operation |
| Stack push/pop           | O(1)            | O(1) per operation |
| Queue enqueue/dequeue    | O(1)            | O(1) per operation |

## 5.87 CDAC CMCE Preparation Checklist

**Must-Know Concepts**

- Singly Linked List.
- Doubly Linked List.
- Circular Linked List.
- Circular Doubly Linked List.
- Insertion and deletion.
- Forward and backward traversal.
- Reverse Linked List.
- Recursive Linked List operations.
- Slow and fast pointer technique.
- Floyd's Cycle Detection.
- Merge two sorted linked lists.
- Remove duplicates.
- Palindrome Linked List.
- Intersection of Linked Lists.
- Merge Sort on Linked List.
- Java Collections LinkedList.

**Important Coding Problems**

| Priority | Problem                   |
| -------- | ------------------------- |
| High     | Reverse Linked List       |
| High     | Find Middle Element       |
| High     | Detect Loop               |
| High     | Nth Node from End         |
| High     | Merge Sorted Lists        |
| High     | Remove Duplicates         |
| High     | Check Palindrome          |
| High     | Find Intersection         |
| Medium   | Reverse in K Groups       |
| Medium   | Sort Linked List          |
| Medium   | Add Two Numbers           |
| Medium   | Reorder Linked List       |
| Medium   | Clone Random Pointer List |
| Advanced | LRU Cache                 |

## 5.88 Final Chapter 5 Revision

Linked List Concepts at a Glance

| Concept                     | Key Point                               |
| --------------------------- | --------------------------------------- |
| Singly Linked List          | One next reference                      |
| Doubly Linked List          | Previous and next references            |
| Circular Linked List        | Last node points to head                |
| Circular Doubly Linked List | Circular connections in both directions |
| Reverse Linked List         | Three-pointer technique                 |
| Middle Element              | Slow and fast pointers                  |
| Cycle Detection             | Floyd's Algorithm                       |
| Merge Sorted Lists          | Two pointers and dummy node             |
| Palindrome                  | Reverse second half                     |
| Intersection                | Two pointer switching                   |
| Merge Sort                  | Divide and conquer                      |
| LRU Cache                   | HashMap + Doubly Linked List            |
| Stack                       | LIFO                                    |
| Queue                       | FIFO                                    |
