# CHAPTER 7 – TREES AND TREE ALGORITHMS

# PART 1 – TREE FUNDAMENTALS AND BINARY TREES WITH JAVA

## 1. INTRODUCTION TO TREES

> A Tree is a non-linear data structure used to represent hierarchical relationships between elements.

Unlike arrays and linked lists, where elements are generally organized sequentially, trees organize data in a parent-child relationship.

A tree consists of:

- Nodes
- Edges
- One root node
- Zero or more child nodes

Example

Consider the following organizational structure:

```text
                 CEO
                  |
          ----------------
          |              |
       Manager A      Manager B
          |              |
       --------       --------
       |      |       |      |
      E1     E2      E3     E4
```

This is a hierarchical structure.

Similarly, a computer file system can be represented using a tree.

Tree Data Structure Example

```text
                 10
               /    \
             20      30
            /  \    /  \
           40  50  60  70
```

Here:

- 10 is the root.
- 20 and 30 are children of 10.
- 40 and 50 are children of 20.
- 60 and 70 are children of 30.

**Important Point**

A tree is a non-linear data structure because one node can connect to multiple child nodes.

## 2. WHY DO WE NEED TREES?

Consider storing these values:

````text
10, 20, 30, 40, 50, 60, 70
In an array, elements are stored sequentially.

```text
10 → 20 → 30 → 40 → 50 → 60 → 70
In a linked list, elements are connected through references.

However, hierarchical relationships cannot be represented naturally using a simple sequential structure.

Trees solve this problem.

Applications of Trees
File systems – Directories and subdirectories.
Database indexing – B-Trees and B+ Trees.
Searching – Binary Search Trees.
Expression evaluation – Expression Trees.
Network routing – Tree-based structures.
Artificial Intelligence – Decision Trees.
Compilers – Abstract Syntax Trees.
HTML and XML – Document Object Models.
Priority queues – Binary Heaps.
Data compression – Huffman Trees.
Advantages
Represents hierarchical information naturally.
Supports efficient searching in suitable tree structures.
Dynamic insertion and deletion.
Useful for sorting and indexing.
Supports recursive problem-solving.
Disadvantages
More complicated than arrays and linked lists.
Requires additional memory for references.
Unbalanced trees may have poor performance.
Tree implementation and traversal require careful handling.
3. TREE TERMINOLOGY
Tree terminology is very important for CDAC CMCE objective questions.

Consider:

```text
                 A
               /   \
              B     C
             / \   / \
            D   E F   G
               /
              H
3.1 Node
A node is an individual element in a tree.

Examples: A, B, C, D, E, F, G, H.

Total nodes = 8.

3.2 Root Node
The topmost node of a tree is called the root.

Example:

A is the root.

Important: A tree has exactly one root.

3.3 Edge
An edge represents a connection between two nodes.

Examples:

A–B, A–C, B–D.

For a tree with n nodes:

Number of edges = n - 1.

Here:

8 nodes = 7 edges.

3.4 Parent Node
A node that has one or more children is called a parent.

Examples:

A is the parent of B and C.
B is the parent of D and E.
E is the parent of H.
3.5 Child Node
A node directly connected below another node is its child.

Examples:

B and C are children of A.

3.6 Siblings
Nodes having the same parent are called siblings.

Examples:

B and C are siblings.
D and E are siblings.
F and G are siblings.
3.7 Leaf Node
A node that has no children is called a leaf node.

Examples:

D, F, G and H.

Leaf nodes = 4.

3.8 Internal Node
A node having at least one child is called an internal node.

Examples:

A, B, C and E.

Internal nodes = 4.

3.9 Degree of a Node
The degree of a node is the number of its children.

Examples:

Degree of A = 2.
Degree of B = 2.
Degree of E = 1.
Degree of D = 0.
3.10 Degree of a Tree
The degree of a tree is the maximum degree of any node in that tree.

For the above tree:

Degree of tree = 2.

3.11 Path
A path is a sequence of nodes connected through edges.

Example:

A → B → E → H.

Length of this path = 3 edges.

3.12 Ancestor
An ancestor is any node that appears above a given node along the path from the root.

For H:

Ancestors = E, B, A.

3.13 Descendant
A descendant is any node that appears below a particular node.

For B:

Descendants = D, E, H.

3.14 Level
Level represents the position of a node in the tree.

We will use root level = 0.

```text
                 A           Level 0
               /   \
              B     C        Level 1
             / \   / \
            D   E F   G       Level 2
               /
              H               Level 3
3.15 Depth
Depth of a node is the number of edges from the root to that node.

Depth of A = 0.

Depth of B = 1.

Depth of H = 3.

3.16 Height of a Node
Height of a node is the number of edges in the longest downward path from that node to a leaf.

Height of H = 0.

Height of E = 1.

Height of B = 2.

Height of A = 3.

3.17 Height of a Tree
The height of a tree is the height of its root.

For the example:

Height = 3.

Important Difference
Term	Meaning
Depth	Distance from root to node
Height	Longest distance from node to a leaf
Level	Position of node, using our root-level-0 convention
Degree	Number of children
4. IMPORTANT TREE PROPERTIES
Property 1: Number of Edges
For a tree containing n nodes:

Edges = n - 1.

Example:

If nodes = 10:

Edges = 9.

Property 2: Unique Path
There is exactly one simple path between any two nodes in a tree.

Property 3: Connected Structure
Every node must be reachable from the root.

Property 4: No Cycles
A tree cannot contain a cycle.

Property 5: Removing an Edge
Removing any edge from a tree disconnects it into two components.

Property 6: Adding an Edge
Adding an edge between two existing nodes of a tree creates a cycle.

These properties are frequently used in graph and tree questions.

5. TYPES OF TREES
There are several types of trees.

Understanding these classifications is important before studying BST, AVL and Heap.

5.1 General Tree
A General Tree is a tree in which a node can have any number of children.

Example:

```text
                  A
              /   |   \
             B    C    D
           / |        / \
          E  F       G   H
A can have three children.

There is no fixed maximum number of children.

5.2 Binary Tree
A Binary Tree is a tree in which each node can have at most two children.

These are called:

Left child.
Right child.
Example:

```text
             10
            /  \
           20   30
5.3 Full Binary Tree
A Full Binary Tree is a binary tree in which every node has either:

Zero children, or
Exactly two children.
A node cannot have only one child.

Example:

```text
              10
             /  \
            20   30
           / \
          40  50
This is a Full Binary Tree.

5.4 Complete Binary Tree
A Complete Binary Tree is a binary tree in which:

Every level except possibly the last is completely filled.
The last level is filled from left to right without gaps.
Example:

```text
              10
             /  \
            20   30
           / \   /
          40 50 60
This is complete.

But the following is not complete:

```text
              10
             /  \
            20   30
             \   /
              50 60
There is a gap at the leftmost position of the last level.

5.5 Perfect Binary Tree
A Perfect Binary Tree is a binary tree in which:

Every internal node has exactly two children.
All leaf nodes are at the same level.
Example:

```text
              10
             /  \
            20   30
           / \   / \
          40 50 60 70
Every level is completely filled.

5.6 Skewed Binary Tree
A Skewed Binary Tree is a tree in which every node has only one child, except the leaf.

Left Skewed
```text
       10
      /
     20
    /
   30
  /
 40
Right Skewed
```text
10
  \
   20
     \
      30
        \
         40
A skewed tree behaves similarly to a linked list.

Its height can become n - 1.

5.7 Balanced Binary Tree
A height-balanced binary tree maintains a controlled difference between the heights of its left and right subtrees.

For AVL trees, the height difference is at most 1 at every node.

Example:

```text
              40
             /  \
            20   60
           / \   / \
          10 30 50 70
A balanced structure can provide efficient operations.

6. GENERAL TREE VS BINARY TREE
Feature	General Tree	Binary Tree
Maximum children	Unlimited	2
Child ordering	Depends on representation	Left and right
Applications	Hierarchies	Searching, heaps
Representation	Multiple possible methods	Array or linked
Degree	Any number	Maximum 2
Important MCQ
The maximum number of children of a Binary Tree node is:

Answer: 2.

7. BINARY TREE REPRESENTATION
A Binary Tree can be represented in two major ways:

Array representation.
Linked representation.
7.1 Array Representation
Consider:

```text
              10
             /  \
            20   30
           / \   / \
          40 50 60 70
Store the nodes level by level in an array.

```text
Index:  0   1   2   3   4   5   6

Value: 10  20  30  40  50  60  70
Using zero-based indexing:

For a node at index i:

Left child index = 2*i + 1.

Right child index = 2*i + 2.

Parent index = (i - 1) / 2, using integer division for non-root nodes.

Example
Node 20 is at index 1.

Left child:

2*1 + 1 = 3.

Right child:

2*1 + 2 = 4.

Therefore:

Index 3 contains 40.
Index 4 contains 50.
Java Program
java
public class Main {

    public static void main(String[] args) {

        int[] tree = {
            10, 20, 30, 40, 50, 60, 70
        };

        int i = 1;

        int left = 2 * i + 1;

        int right = 2 * i + 2;

        System.out.println("Parent: " + tree[i]);

        System.out.println("Left Child: " + tree[left]);

        System.out.println("Right Child: " + tree[right]);
    }
}
Output:

```text
Parent: 20
Left Child: 40
Right Child: 50
Advantages
Simple representation.
No additional reference fields.
Efficient for complete binary trees.
Useful for Heap implementation.
Disadvantages
Wastes space for sparse trees.
Difficult to represent highly skewed trees efficiently.
Fixed array capacity may be restrictive.
7.2 Linked Representation
In linked representation, each node stores:

Data.
Reference to left child.
Reference to right child.
Structure
```text
class Node {

    int data;

    Node left;

    Node right;
}
Example:

```text
              10
             /  \
            20   30
           / \
          40 50
Each node connects to its children through references.

Advantages
Dynamic memory allocation.
Suitable for sparse trees.
Easy insertion and deletion.
No need to reserve unused array positions.
Disadvantages
Extra memory required for references.
More complex implementation than an array.
8. CREATING A BINARY TREE IN JAVA
Java does not have a built-in primitive Binary Tree class.

We create our own Node class.

Node Class
java
class Node {

    int data;

    Node left;

    Node right;

    Node(int value) {

        data = value;

        left = null;

        right = null;
    }
}
Explanation:

data stores the node value.
left stores the reference to the left child.
right stores the reference to the right child.
Constructor initializes a new node.
Creating a Tree
java
public class Main {

    public static void main(String[] args) {

        Node root = new Node(10);

        root.left = new Node(20);

        root.right = new Node(30);

        root.left.left = new Node(40);

        root.left.right = new Node(50);

        root.right.left = new Node(60);

        root.right.right = new Node(70);
    }
}
Resulting Tree
```text
              10
             /  \
            20   30
           / \   / \
          40 50 60 70
Important Concept
The statement:

java
Node root = new Node(10);
creates the root node.

The statement:

java
root.left = new Node(20);
creates a new node and connects it to the left of the root.

9. TREE TRAVERSAL
Traversal means visiting every node of a tree in a particular order.

There are two major categories:

Depth First Search (DFS).
Breadth First Search (BFS).
DFS Traversals
Inorder.
Preorder.
Postorder.
BFS Traversal
Level Order Traversal.
Consider:

```text
              A
             / \
            B   C
           / \ / \
          D  E F  G
We will use this tree for all traversal examples.

10. INORDER TRAVERSAL
Inorder traversal follows:

Left → Root → Right.

Step-by-Step
Visit left subtree.
Visit root.
Visit right subtree.
For our tree:

```text
              A
             / \
            B   C
           / \ / \
          D  E F  G
First visit B's left subtree:

D.

Then B.

Then E.

Then A.

Then C's left subtree:

F.

Then C.

Then G.

Output
```text
D B E A F C G
Java Code
java
static void inorder(Node root) {

    if (root == null) {

        return;
    }

    inorder(root.left);

    System.out.print(root.data + " ");

    inorder(root.right);
}
Dry Run
For:

```text
       A
      / \
     B   C
Execution:

```text
inorder(A)
    |
    inorder(B)
        |
        inorder(null)
        |
        print(B)
        |
        inorder(null)
    |
    print(A)
    |
    inorder(C)
        |
        inorder(null)
        |
        print(C)
        |
        inorder(null)
Output:

```text
B A C
````

Important Point
Inorder traversal of a Binary Search Tree produces sorted order when using distinct keys.

For an ordinary Binary Tree, inorder does not necessarily produce sorted order.

11. PREORDER TRAVERSAL
    Preorder traversal follows:

Root → Left → Right.

Steps
Visit root.
Visit left subtree.
Visit right subtree.
For:

`````text
              A
             / \
            B   C
           / \ / \
          D  E F  G
Output:


```text
A B D E C F G
Java Code

java
static void preorder(Node root) {

    if (root == null) {

        return;
    }

    System.out.print(root.data + " ");

    preorder(root.left);

    preorder(root.right);
}
Applications
Copying a tree.
Prefix expression generation.
Serialization of tree structures.
12. POSTORDER TRAVERSAL
Postorder traversal follows:

Left → Right → Root.

Steps
Visit left subtree.
Visit right subtree.
Visit root.
For:


```text
              A
             / \
            B   C
           / \ / \
          D  E F  G
Output:


```text
D E B F G C A
Java Code

java
static void postorder(Node root) {

    if (root == null) {

        return;
    }

    postorder(root.left);

    postorder(root.right);

    System.out.print(root.data + " ");
}
Applications
Deleting a tree.
Evaluating expression trees.
Postfix expression generation.
Comparison of Three Traversals

Traversal	Order	Output
Inorder	Left, Root, Right	D B E A F C G
Preorder	Root, Left, Right	A B D E C F G
Postorder	Left, Right, Root	D E B F G C A
Memory trick:

Inorder: Root in the middle.
Preorder: Root first.
Postorder: Root last.
13. LEVEL ORDER TRAVERSAL
Level Order Traversal visits nodes level by level from left to right.

It uses a Queue data structure.

For:


```text
              A
             / \
            B   C
           / \ / \
          D  E F  G
Traversal:

Level 0: A

Level 1: B C

Level 2: D E F G

Output:


```text
A B C D E F G
Algorithm
Create an empty queue.
Insert root into queue.
While queue is not empty:
Remove the front node.
Print its value.
Insert its left child if present.
Insert its right child if present.
Java Code

java
import java.util.LinkedList;
import java.util.Queue;

static void levelOrder(Node root) {

    if (root == null) {

        return;
    }

    Queue<Node> queue = new LinkedList<>();

    queue.add(root);

    while (!queue.isEmpty()) {

        Node current = queue.remove();

        System.out.print(current.data + " ");

        if (current.left != null) {

            queue.add(current.left);
        }

        if (current.right != null) {

            queue.add(current.right);
        }
    }
}
Output

```text
A B C D E F G
Why Do We Use Queue?
Queue follows FIFO (First In First Out).

Nodes discovered earlier are processed earlier.

This naturally produces level-by-level traversal.

14. ITERATIVE TREE TRAVERSALS
Recursive traversal is easy to understand.

However, we can also perform traversals using an explicit Stack.

14.1 Iterative Inorder Traversal
Algorithm:

Initialize an empty stack.
Move as far left as possible, pushing nodes.
Pop and visit a node.
Move to its right subtree.
Repeat until both current and stack are empty.
Java Code

java
import java.util.Stack;

static void iterativeInorder(Node root) {

    Stack<Node> stack = new Stack<>();

    Node current = root;

    while (current != null || !stack.isEmpty()) {

        while (current != null) {

            stack.push(current);

            current = current.left;
        }

        current = stack.pop();

        System.out.print(current.data + " ");

        current = current.right;
    }
}
14.2 Iterative Preorder Traversal
Algorithm:

Push root onto stack.
Pop and print.
Push right child.
Push left child.
Repeat.
We push the right child first because the stack is LIFO.

Java Code

java
static void iterativePreorder(Node root) {

    if (root == null) return;

    Stack<Node> stack = new Stack<>();

    stack.push(root);

    while (!stack.isEmpty()) {

        Node current = stack.pop();

        System.out.print(current.data + " ");

        if (current.right != null) {

            stack.push(current.right);
        }

        if (current.left != null) {

            stack.push(current.left);
        }
    }
}
14.3 Iterative Postorder Traversal
One simple approach uses two stacks.

Java Code

java
static void iterativePostorder(Node root) {

    if (root == null) return;

    Stack<Node> s1 = new Stack<>();

    Stack<Node> s2 = new Stack<>();

    s1.push(root);

    while (!s1.isEmpty()) {

        Node current = s1.pop();

        s2.push(current);

        if (current.left != null) {

            s1.push(current.left);
        }

        if (current.right != null) {

            s1.push(current.right);
        }
    }

    while (!s2.isEmpty()) {

        System.out.print(s2.pop().data + " ");
    }
}
Traversal Complexity
For all these traversals:

Time Complexity = O(n).

Auxiliary space:

Recursive DFS: O(h) call stack.
Iterative DFS: O(h) for stack-based approaches, depending on tree shape and method.
Level order: O(w), where w is maximum width of the tree.
15. COUNTING NODES IN BINARY TREE
Suppose we have:


```text
             10
            /  \
           20   30
          / \
         40 50
Total nodes = 5.

Formula
Total nodes:

1 + Nodes in left subtree + Nodes in right subtree.

Java Code

java
static int countNodes(Node root) {

    if (root == null) {

        return 0;
    }

    return 1 +
        countNodes(root.left) +
        countNodes(root.right);
}
Output

```text
5
Explanation
The recursion counts:

Current node: 1.
All nodes in left subtree.
All nodes in right subtree.
16. COUNTING LEAF NODES
A leaf node has no children.

Condition:


java
root.left == null && root.right == null
Java Code

java
static int countLeafNodes(Node root) {

    if (root == null) {

        return 0;
    }

    if (root.left == null &&
        root.right == null) {

        return 1;
    }

    return countLeafNodes(root.left) +
           countLeafNodes(root.right);
}
Example

```text
              10
             /  \
            20   30
           / \     \
          40 50     60
Leaf nodes:

40, 50, 60.

Output:


```text
3
17. FINDING HEIGHT AND DEPTH
17.1 Height of Binary Tree
We will use the edge-based convention:

Empty tree = -1.
Leaf node = 0.
Java Code

java
static int height(Node root) {

    if (root == null) {

        return -1;
    }

    int leftHeight = height(root.left);

    int rightHeight = height(root.right);

    return 1 + Math.max(leftHeight, rightHeight);
}
Example

```text
              10
             /  \
            20   30
           /
          40
         /
        50
Longest path:

10 → 20 → 40 → 50.

Number of edges = 3.

Output:


```text
3
```Note: Some textbooks define height by counting nodes instead of edges. Always check the convention in the question.

17.2 Finding Depth of a Node
Depth is the number of edges from root to a particular node.

Java Code

java
static int findDepth(Node root, int key, int depth) {

    if (root == null) {

        return -1;
    }

    if (root.data == key) {

        return depth;
    }

    int left = findDepth(root.left, key, depth + 1);

    if (left != -1) {

        return left;
    }

    return findDepth(root.right, key, depth + 1);
}
Example:


java
System.out.println(findDepth(root, 50, 0));
Output:


```text
3
18. FINDING MAXIMUM AND MINIMUM VALUES
Unlike a BST, an ordinary Binary Tree does not maintain ordering.

Therefore, we may need to examine every node.

Maximum Value

java
static int findMaximum(Node root) {

    if (root == null) {

        return Integer.MIN_VALUE;
    }

    int max = root.data;

    int leftMax = findMaximum(root.left);

    int rightMax = findMaximum(root.right);

    return Math.max(max, Math.max(leftMax, rightMax));
}
Minimum Value

java
static int findMinimum(Node root) {

    if (root == null) {

        return Integer.MAX_VALUE;
    }

    int min = root.data;

    int leftMin = findMinimum(root.left);

    int rightMin = findMinimum(root.right);

    return Math.min(min, Math.min(leftMin, rightMin));
}
Example

```text
              40
             /  \
            20   70
           / \
          10  30
Maximum = 70.

Minimum = 10.

Time Complexity = O(n).

19. SEARCHING IN A BINARY TREE
In a general Binary Tree, there is no ordering rule.

Therefore, we may need to search both subtrees.

Java Code

java
static boolean search(Node root, int key) {

    if (root == null) {

        return false;
    }

    if (root.data == key) {

        return true;
    }

    return search(root.left, key) ||
           search(root.right, key);
}
Example
Search for 50:


```text
              10
             /  \
            20   30
           / \
          40 50
Output:


```text
true
Complexity
Best Case: O(1), if the root contains the key.

Worst Case: O(n).

Important distinction:

Binary Tree searching: O(n) worst case.
Balanced BST searching: O(log n).
20. COPYING AND COMPARING BINARY TREES
20.1 Copying a Binary Tree
We can create a new tree with the same structure and values.

Java Code

java
static Node copyTree(Node root) {

    if (root == null) {

        return null;
    }

    Node newNode = new Node(root.data);

    newNode.left = copyTree(root.left);

    newNode.right = copyTree(root.right);

    return newNode;
}
This creates a separate tree rather than simply copying the root reference.

20.2 Comparing Two Binary Trees
Two binary trees are identical if:

Both have the same structure.
Corresponding nodes contain equal values.
Java Code

java
static boolean identical(Node a, Node b) {

    if (a == null && b == null) {

        return true;
    }

    if (a == null || b == null) {

        return false;
    }

    return a.data == b.data &&
           identical(a.left, b.left) &&
           identical(a.right, b.right);
}
Complexity
Both operations take O(n) time.

21. FULL VS COMPLETE VS PERFECT BINARY TREE
This is one of the most frequently confused topics in objective examinations.

Full Binary Tree
Every node has either zero or two children.


```text
        10
       /  \
      20   30
     / \
    40 50
Complete Binary Tree
All levels are full except possibly the last, which is filled from left to right.


```text
        10
       /  \
      20   30
     / \   /
    40 50 60
Perfect Binary Tree
All internal nodes have two children and all leaves are at the same level.


```text
        10
       /  \
      20   30
     / \   / \
    40 50 60 70
Comparison Table

Feature	Full	Complete	Perfect
Every node has 0 or 2 children	Yes	Not necessary	Yes
Last level filled left to right	Not necessary	Yes	Yes
All leaves at same level	Not necessary	Not necessary	Yes
All levels completely filled	Not necessary	Except possibly last	Yes
Important Relationships
Every perfect binary tree is full.
Every perfect binary tree is complete.
Every complete binary tree is not necessarily full.
Every full binary tree is not necessarily complete.
22. IMPORTANT BINARY TREE FORMULAS
Assume root is at level 0.

For a Binary Tree:

Maximum Nodes at Level L
2^L

Example:

Maximum nodes at level 3:

2³ = 8.

Maximum Nodes in a Binary Tree of Height H
2^(H+1) - 1.

Example:

Height = 3.

Maximum nodes:

2⁴ - 1 = 15.

Maximum Leaf Nodes
For height H:

2^H.

This is the maximum number of leaf nodes in a binary tree of that height.

Minimum Height for N Nodes
The minimum possible edge-based height is:

ceil(log₂(N + 1)) - 1.

This occurs when the tree is as full as possible.

Nodes in a Perfect Binary Tree
For height H:

N = 2^(H+1) - 1.

Number of Edges
For N nodes:

E = N - 1.

Full Binary Tree Property
If:

I = Number of internal nodes.
L = Number of leaf nodes.
Then:

L = I + 1.

Total nodes:

N = 2I + 1.

These formulas are useful for numerical MCQs.

23. TIME AND SPACE COMPLEXITY
Let:

n = Number of nodes.
h = Height of tree.
w = Maximum width.

Operation	Time Complexity
Inorder Traversal	O(n)
Preorder Traversal	O(n)
Postorder Traversal	O(n)
Level Order Traversal	O(n)
Count Nodes	O(n)
Count Leaves	O(n)
Find Height	O(n)
Find Maximum	O(n)
Find Minimum	O(n)
Search in General Binary Tree	O(n)
Copy Tree	O(n)
Compare Trees	O(n)
Auxiliary Space

Operation	Space Complexity
Recursive DFS	O(h)
Iterative DFS	O(h) for standard stack-based traversal
Level Order	O(w)
Copy Tree	O(h) recursion stack, plus O(n) for the newly created tree
24. JAVA OUTPUT-BASED QUESTIONS
These are important for CDAC CMCE output-prediction practice.

Question 1: Inorder Traversal

java
class Node {

    int data;

    Node left, right;

    Node(int value) {

        data = value;
    }
}

public class Main {

    static void inorder(Node root) {

        if (root == null) return;

        inorder(root.left);

        System.out.print(root.data + " ");

        inorder(root.right);
    }

    public static void main(String[] args) {

        Node root = new Node(10);

        root.left = new Node(20);

        root.right = new Node(30);

        inorder(root);
    }
}
Output:


```text
20 10 30
Question 2: Preorder Traversal

java
static void preorder(Node root) {

    if (root == null) return;

    System.out.print(root.data + " ");

    preorder(root.left);

    preorder(root.right);
}
Using the same tree:

Output:


```text
10 20 30
Question 3: Postorder Traversal

java
static void postorder(Node root) {

    if (root == null) return;

    postorder(root.left);

    postorder(root.right);

    System.out.print(root.data + " ");
}
Output:


```text
20 30 10
Question 4: Counting Nodes

java
Node root = new Node(10);

root.left = new Node(20);

root.right = new Node(30);

root.left.left = new Node(40);

System.out.println(countNodes(root));
Output:


```text
4
Question 5: Counting Leaf Nodes

java
Node root = new Node(10);

root.left = new Node(20);

root.right = new Node(30);

root.left.left = new Node(40);

System.out.println(countLeafNodes(root));
Output:


```text
2
Question 6: Height

java
Node root = new Node(10);

root.left = new Node(20);

root.right = new Node(30);

root.left.left = new Node(40);

System.out.println(height(root));
Output:


```text
2
Question 7: Level Order

java
Node root = new Node(10);

root.left = new Node(20);

root.right = new Node(30);

root.left.left = new Node(40);

root.left.right = new Node(50);

levelOrder(root);
Output:


```text
10 20 30 40 50
Question 8: Searching

java
Node root = new Node(10);

root.left = new Node(20);

root.right = new Node(30);

System.out.println(search(root, 20));

System.out.println(search(root, 50));
Output:


```text
true
false
Question 9: Maximum Value

java
Node root = new Node(10);

root.left = new Node(40);

root.right = new Node(30);

System.out.println(findMaximum(root));
Output:


```text
40
Question 10: Array Representation

java
int[] tree = {10, 20, 30, 40, 50};

int i = 1;

System.out.println(tree[2 * i + 1]);

System.out.println(tree[2 * i + 2]);
Output:


```text
40
Invalid index
Important: The second expression accesses index 4, which is valid, so the actual output is:


```text
40
Exception? No.
Correction: Index 4 contains 50.

Actual output:


```text
40
50

26. INTERVIEW CODING PROBLEMS
Practise the following problems after completing the theory.

Beginner Level
Create a Binary Tree in Java.
Print inorder traversal.
Print preorder traversal.
Print postorder traversal.
Perform level order traversal.
Count total nodes.
Count leaf nodes.
Find tree height.
Find maximum value.
Search an element in a Binary Tree.
Intermediate Level
Count internal nodes.
Find depth of a given node.
Copy a Binary Tree.
Check whether two trees are identical.
Find the maximum width of a Binary Tree.
Find the sum of all nodes.
Find the number of nodes at a given level.
Print left view of a Binary Tree.
Print right view of a Binary Tree.
Print all root-to-leaf paths.
Advanced Level
Find diameter of a Binary Tree.
Find Lowest Common Ancestor.
Check whether a tree is height-balanced.
Construct a tree from inorder and preorder traversals.
Construct a tree from inorder and postorder traversals.
Find boundary traversal.
Perform vertical order traversal.
Find maximum path sum.
Serialize and deserialize a Binary Tree.
Convert a Binary Tree into its mirror.
27. IMPORTANT INTERVIEW CONCEPTS
Question 1: What is the difference between a Binary Tree and a BST?
A Binary Tree allows at most two children per node without requiring a particular ordering.

A BST additionally maintains:

Left subtree < Root < Right subtree.

Question 2: What is the difference between height and depth?
Depth measures distance from root to a node.

Height measures the longest downward path from a node to a leaf.

Question 3: Why is recursion useful in tree traversal?
Because each subtree is itself a tree.

We can solve the problem recursively by processing the left and right subtrees.

Question 4: Why does level order traversal use a queue?
A queue processes nodes in FIFO order, ensuring nodes are visited level by level.

Question 5: What is the difference between complete and perfect binary trees?
A complete tree may have an incompletely filled final level, but nodes must be filled from left to right.

A perfect tree has all levels completely filled.

Question 6: What is the maximum number of nodes in a binary tree of height h?
2^(h+1) - 1.

Question 7: What happens when a Binary Tree becomes skewed?
Its height can become n - 1, and many tree operations may require O(n) time.

Question 8: Can a Binary Tree contain duplicate values?
Yes. A general Binary Tree can contain duplicates.

A BST's duplicate-key policy depends on its implementation.

28. QUICK REVISION SHEET

Concept	Important Answer
Tree	Non-linear hierarchical structure
Root	Topmost node
Leaf	Node with zero children
Internal node	Node with at least one child
Degree	Number of children
Tree edges	N - 1
Binary Tree maximum children	2
Inorder	Left, Root, Right
Preorder	Root, Left, Right
Postorder	Left, Right, Root
Level Order	Queue
Iterative DFS	Stack
Array left child	2i + 1
Array right child	2i + 2
Maximum nodes at level L	2^L
Maximum nodes at height H	2^(H+1) - 1
Full Binary Tree	0 or 2 children
Complete Binary Tree	Last level filled left to right
Perfect Binary Tree	All levels completely filled
Skewed Tree	One-sided tree
Traversal complexity	O(n)
General Binary Tree search	O(n)

---

# PART 2 – BINARY SEARCH TREE (BST)

1. INTRODUCTION TO BINARY SEARCH TREE
   A Binary Search Tree (BST) is a special type of Binary Tree in which nodes are arranged according to a specific ordering rule.

Unlike an ordinary Binary Tree, a BST maintains an order among its elements.

This ordering makes searching, insertion and deletion efficient when the tree is reasonably balanced.

Definition
A Binary Search Tree is a binary tree in which:

All values in the left subtree of a node are smaller than the node's value.
All values in the right subtree of a node are greater than the node's value.
Both left and right subtrees must also satisfy the BST property.
For this chapter, we will use distinct integer values and the strict rule:

Left subtree < Root < Right subtree.

Example
Consider the following BST:

````text
              50
             /  \
           30    70
          / \    / \
        20  40  60  80
Let's examine the root node 50.

Left subtree:

20, 30, 40

All values are smaller than 50.

Right subtree:

60, 70, 80

All values are greater than 50.

Therefore, this is a valid Binary Search Tree.

2. PROPERTIES OF BST
The following properties are important for examinations and interviews.

Property 1: Left Subtree Contains Smaller Values
For any node:

All values in its left subtree must be smaller than the node's value.

Example:


```text
       50
      /
     30
    /
   20
20 and 30 are smaller than 50.

Property 2: Right Subtree Contains Greater Values
All values in its right subtree must be greater than the node's value.

Example:


```text
    50
      \
       70
         \
          90
70 and 90 are greater than 50.

Property 3: Both Subtrees Must Be BSTs
A tree does not become a BST merely because its immediate left child is smaller and its immediate right child is greater.

The entire left and right subtrees must follow the BST ordering rules.

Consider:


```text
          50
         /  \
        30   70
       / \
      20  60
This is NOT a valid BST.

Why?

60 is in the left subtree of 50, but 60 is greater than 50.

This is a common examination trap.

Property 4: Inorder Traversal Produces Sorted Order
For a valid BST, inorder traversal produces values in ascending order when distinct values are used.

Example:

BST:


```text
          50
         /  \
        30   70
       / \   / \
      20 40 60 80
Inorder traversal:


```text
20 30 40 50 60 70 80
This is one of the most important properties of BST.

3. BINARY TREE VS BINARY SEARCH TREE
A Binary Search Tree is a special type of Binary Tree.


Feature	Binary Tree	Binary Search Tree
Maximum children	2	2
Ordering rule	No mandatory ordering	Left < Root < Right
Searching	O(n) generally	O(log n) average
Insertion	Depends on implementation	Based on BST ordering
Inorder traversal	Not necessarily sorted	Sorted for distinct keys
Applications	Hierarchical structures	Searching and ordered data
Important MCQ Concept
Every BST is a Binary Tree.

But every Binary Tree is not a BST.

4. BST EXAMPLE AND STRUCTURE
Let's construct a BST using these elements:


```text
50, 30, 70, 20, 40, 60, 80
Insertion order matters because it determines the shape of the tree.

Step 1: Insert 50
The tree is empty.

50 becomes the root.


```text
50
Step 2: Insert 30
30 < 50.

Insert 30 to the left.


```text
    50
   /
  30
Step 3: Insert 70
70 > 50.

Insert 70 to the right.


```text
    50
   /  \
  30   70
Step 4: Insert 20
20 < 50.

Move left.

20 < 30.

Insert to the left of 30.


```text
       50
      /  \
     30   70
    /
   20
Step 5: Insert 40
40 < 50.

Move left.

40 > 30.

Insert to the right of 30.


```text
       50
      /  \
     30   70
    / \
   20 40
Step 6: Insert 60 and 80
Following the same comparison process:


```text
          50
         /  \
        30   70
       / \   / \
      20 40 60 80
This is our final BST.

5. SEARCHING IN BST
Searching means finding whether a particular value exists in the BST.

BST searching uses comparisons to decide whether to move left or right.

Searching Algorithm
Suppose we want to search for 60.

Starting tree:


```text
          50
         /  \
        30   70
       / \   / \
      20 40 60 80
Steps:

Compare 60 with root 50.
60 > 50, so move right.
Compare 60 with 70.
60 < 70, so move left.
Compare 60 with 60.
Element found.
Searching Pseudocode

```text
SEARCH(root, key)

1. If root == NULL:
      Return NOT FOUND.

2. If root.data == key:
      Return FOUND.

3. If key < root.data:
      Search(root.left, key).

4. Otherwise:
      Search(root.right, key).
Java Code – Recursive Searching

java
class Node {

    int data;

    Node left, right;

    Node(int value) {

        data = value;

        left = right = null;
    }
}

public class Main {

    static Node search(Node root, int key) {

        if (root == null || root.data == key) {

            return root;
        }

        if (key < root.data) {

            return search(root.left, key);
        }

        return search(root.right, key);
    }

    public static void main(String[] args) {

        Node root = new Node(50);

        root.left = new Node(30);

        root.right = new Node(70);

        root.left.left = new Node(20);

        root.left.right = new Node(40);

        root.right.left = new Node(60);

        root.right.right = new Node(80);

        int key = 60;

        Node result = search(root, key);

        if (result != null) {

            System.out.println("Element Found");
        } else {

            System.out.println("Element Not Found");
        }
    }
}
Output

```text
Element Found
Explanation
The search path is:


```text
50 → 70 → 60
Only three comparisons are required.
`````

6. ITERATIVE SEARCHING IN BST
   We can also search without recursion.

This implementation uses a loop.

Java Code

java
static Node searchIterative(Node root, int key) {

    while (root != null) {

        if (root.data == key) {

            return root;
        }

        else if (key < root.data) {

            root = root.left;
        }

        else {

            root = root.right;
        }
    }

    return null;

}
Advantages
No recursive function calls.
Uses constant auxiliary space.
Easy to understand.
Avoids recursion stack overhead.
Complexity

Case Time Complexity
Best O(1)
Average in a reasonably balanced BST O(log n)
Worst O(n)
Iterative search uses O(1) auxiliary space.

7. INSERTION IN BST
   Insertion means adding a new node while maintaining the BST property.

The insertion position is determined through comparisons.

Example
Insert 65 into:

````text
          50
         /  \
        30   70
       / \   / \
      20 40 60 80
Step 1:

65 > 50.

Move right.

Step 2:

65 < 70.

Move left.

Step 3:

65 > 60.

Move right.

The right child of 60 is empty.

Insert 65 there.

Final BST

```text
          50
         /  \
        30   70
       / \   / \
      20 40 60 80
               \
                65
Insertion Algorithm

```text
INSERT(root, value)

1. If root == NULL:
      Create a new node.
      Return it.

2. If value < root.data:
      root.left = INSERT(root.left, value).

3. Else if value > root.data:
      root.right = INSERT(root.right, value).

4. Return root.
Java Code

java
static Node insert(Node root, int value) {

    if (root == null) {

        return new Node(value);
    }

    if (value < root.data) {

        root.left = insert(root.left, value);
    }

    else if (value > root.data) {

        root.right = insert(root.right, value);
    }

    return root;
}
Notice that we do not insert duplicates in this implementation.

Different applications may adopt a different duplicate-key policy.

8. COMPLETE JAVA PROGRAM FOR BST INSERTION

java
class Node {

    int data;

    Node left, right;

    Node(int value) {

        data = value;

        left = right = null;
    }
}

public class Main {

    static Node insert(Node root, int value) {

        if (root == null) {

            return new Node(value);
        }

        if (value < root.data) {

            root.left = insert(root.left, value);
        }

        else if (value > root.data) {

            root.right = insert(root.right, value);
        }

        return root;
    }

    static void inorder(Node root) {

        if (root == null) {

            return;
        }

        inorder(root.left);

        System.out.print(root.data + " ");

        inorder(root.right);
    }

    public static void main(String[] args) {

        Node root = null;

        root = insert(root, 50);

        root = insert(root, 30);

        root = insert(root, 70);

        root = insert(root, 20);

        root = insert(root, 40);

        root = insert(root, 60);

        root = insert(root, 80);

        System.out.println("Inorder Traversal:");

        inorder(root);
    }
}
Output

```text
Inorder Traversal:
20 30 40 50 60 70 80
Important Concept
Why do we write:


java
root = insert(root, value);
Because if the tree is empty, insertion creates a new root.

The returned node must be assigned to root.

For subsequent insertions, this assignment also preserves the root reference.

9. BST TRAVERSALS
A BST supports the same traversal techniques as an ordinary Binary Tree.

Let's use:


```text
          50
         /  \
        30   70
       / \   / \
      20 40 60 80
9.1 Inorder Traversal
Order:

Left → Root → Right

Output:


```text
20 30 40 50 60 70 80
Important:

Inorder traversal of a BST produces sorted order for distinct keys.

9.2 Preorder Traversal
Order:

Root → Left → Right

Output:


```text
50 30 20 40 70 60 80
9.3 Postorder Traversal
Order:

Left → Right → Root

Output:


```text
20 40 30 60 80 70 50
Java Code

java
static void inorder(Node root) {

    if (root == null) return;

    inorder(root.left);

    System.out.print(root.data + " ");

    inorder(root.right);
}

static void preorder(Node root) {

    if (root == null) return;

    System.out.print(root.data + " ");

    preorder(root.left);

    preorder(root.right);
}

static void postorder(Node root) {

    if (root == null) return;

    postorder(root.left);

    postorder(root.right);

    System.out.print(root.data + " ");
}
10. FINDING MINIMUM ELEMENT IN BST
In a BST, the minimum value is always located at the leftmost node.

Why?

Because every left child is smaller than its parent.

Therefore, repeatedly moving left eventually reaches the smallest element.

Example

```text
          50
         /  \
        30   70
       / \
      20 40
     /
    10
Minimum = 10.

Algorithm

```text
MINIMUM(root)

1. If root == NULL:
      Return NULL.

2. While root.left != NULL:
      root = root.left.

3. Return root.
Java Code

java
static Node findMin(Node root) {

    if (root == null) {

        return null;
    }

    while (root.left != null) {

        root = root.left;
    }

    return root;
}
Output Example

java
Node min = findMin(root);

System.out.println(min.data);
Output:


```text
10
Time Complexity: O(h)

Where h is the height of the tree.

11. FINDING MAXIMUM ELEMENT IN BST
The maximum value is located at the rightmost node.

Example

```text
          50
         /  \
        30   70
            /  \
           60   90
                  \
                   100
Maximum = 100.

Java Code

java
static Node findMax(Node root) {

    if (root == null) {

        return null;
    }

    while (root.right != null) {

        root = root.right;
    }

    return root;
}
Important MCQ
In a BST, the maximum element is found at:

A. Root node

B. Leftmost node

C. Rightmost node

D. Leaf node only

Answer: C. Rightmost node.

12. DELETION IN BST
Deletion is one of the most important BST operations.

Unlike insertion, deletion requires handling different structural situations.

There are three cases:

Deleting a leaf node.
Deleting a node with one child.
Deleting a node with two children.
Understanding these three cases is essential.

13. THREE CASES OF BST DELETION
CASE 1: DELETING A LEAF NODE
A leaf node is a node that has no children.

Consider:


```text
          50
         /  \
        30   70
       / \   / \
      20 40 60 80
Delete 20.

20 has no children.

Simply remove it.

Final tree:


```text
          50
         /  \
        30   70
         \   / \
         40 60 80
Explanation
No restructuring is required.

The parent node's corresponding child reference becomes null.

CASE 2: DELETING A NODE WITH ONE CHILD
Consider:


```text
          50
         /  \
        30   70
       /
      20
        \
         25
Delete 20.

20 has one child: 25.

Instead of simply removing 20, connect its parent directly to 25.

Final tree:


```text
          50
         /  \
        30   70
       /
      25
Explanation
The deleted node is replaced by its only child.

This preserves the BST ordering property.

CASE 3: DELETING A NODE WITH TWO CHILDREN
This is the most important case.

Consider:


```text
          50
         /  \
        30   70
       / \   / \
      20 40 60 80
Delete 50.

50 has two children.

We cannot simply remove it because both subtrees must remain connected.

There are two standard approaches:

Replace it with its inorder successor.
Replace it with its inorder predecessor.
Method 1: Inorder Successor
The inorder successor is the smallest value in the right subtree.

For 50:

Right subtree:


```text
       70
      /  \
     60   80
Smallest value = 60.

Replace 50 with 60.

Then remove the original 60 node from the right subtree.

Final tree:


```text
          60
         /  \
        30   70
       / \     \
      20 40    80
Method 2: Inorder Predecessor
The inorder predecessor is the largest value in the left subtree.

For 50:

Left subtree:


```text
       30
      /  \
     20   40
Largest value = 40.

Replace 50 with 40.

Then remove the original 40 node from the left subtree.

Final tree:


```text
          40
         /  \
        30   70
       /     / \
      20    60 80
Both methods are correct.

Important Exam Point
Inorder successor = Minimum element in right subtree.
Inorder predecessor = Maximum element in left subtree.
14. COMPLETE BST DELETION JAVA PROGRAM

java
class Node {

    int data;

    Node left, right;

    Node(int value) {

        data = value;

        left = right = null;
    }
}

public class Main {

    static Node insert(Node root, int value) {

        if (root == null) {

            return new Node(value);
        }

        if (value < root.data) {

            root.left = insert(root.left, value);
        }

        else if (value > root.data) {

            root.right = insert(root.right, value);
        }

        return root;
    }

    static Node findMin(Node root) {

        while (root.left != null) {

            root = root.left;
        }

        return root;
    }

    static Node delete(Node root, int key) {

        if (root == null) {

            return null;
        }

        if (key < root.data) {

            root.left = delete(root.left, key);
        }

        else if (key > root.data) {

            root.right = delete(root.right, key);
        }

        else {

            // CASE 1: No child

            if (root.left == null &&
                root.right == null) {

                return null;
            }

            // CASE 2: One child

            if (root.left == null) {

                return root.right;
            }

            if (root.right == null) {

                return root.left;
            }

            // CASE 3: Two children

            Node successor = findMin(root.right);

            root.data = successor.data;

            root.right = delete(root.right, successor.data);
        }

        return root;
    }

    static void inorder(Node root) {

        if (root == null) return;

        inorder(root.left);

        System.out.print(root.data + " ");

        inorder(root.right);
    }

    public static void main(String[] args) {

        Node root = null;

        int[] values = {50, 30, 70, 20, 40, 60, 80};

        for (int value : values) {

            root = insert(root, value);
        }

        System.out.println("Before Deletion:");

        inorder(root);

        root = delete(root, 50);

        System.out.println("\nAfter Deletion:");

        inorder(root);
    }
}
Output

```text
Before Deletion:
20 30 40 50 60 70 80

After Deletion:
20 30 40 60 70 80
Important Explanation
In the deletion method:


java
Node successor = findMin(root.right);
We find the smallest node in the right subtree.

Then:


java
root.data = successor.data;
We replace the deleted node's value.

Finally:


java
root.right = delete(root.right, successor.data);
We remove the original successor node to avoid duplication.

15. INORDER SUCCESSOR
The inorder successor of a node is the next node in inorder traversal.

In simple words:

It is the next greater value in the BST.

Example:


```text
          50
         /  \
        30   70
       / \   / \
      20 40 60 80
Inorder:


```text
20 30 40 50 60 70 80
Successor examples:


Node	Successor
20	30
30	40
40	50
50	60
60	70
70	80
80	NULL
Two Cases
Case 1: Node has a right subtree.

The successor is the minimum node in its right subtree.

Case 2: Node does not have a right subtree.

Move upward until reaching an ancestor for which the current node lies in its left subtree.

That ancestor is the successor.

Java Implementation

java
static Node inorderSuccessor(Node root, Node target) {

    if (target == null) {

        return null;
    }

    // Case 1: Right subtree exists

    if (target.right != null) {

        return findMin(target.right);
    }

    // Case 2: Move upward using BST search

    Node successor = null;

    while (root != null) {

        if (target.data < root.data) {

            successor = root;

            root = root.left;
        }

        else if (target.data > root.data) {

            root = root.right;
        }

        else {

            break;
        }
    }

    return successor;
}
This implementation assumes distinct keys and that target belongs to the BST.

16. INORDER PREDECESSOR
The inorder predecessor is the previous node in inorder traversal.

In simple words:

It is the next smaller value in the BST.

For:


```text
20 30 40 50 60 70 80
Predecessor examples:


Node	Predecessor
20	NULL
30	20
40	30
50	40
60	50
70	60
80	70
Two Cases
Case 1: Node has a left subtree.

Predecessor is the maximum node in its left subtree.

Case 2: Node does not have a left subtree.

Move upward until reaching an ancestor for which the current node lies in its right subtree.

Java Code

java
static Node inorderPredecessor(Node root, Node target) {

    if (target == null) {

        return null;
    }

    if (target.left != null) {

        return findMax(target.left);
    }

    Node predecessor = null;

    while (root != null) {

        if (target.data > root.data) {

            predecessor = root;

            root = root.right;
        }

        else if (target.data < root.data) {

            root = root.left;
        }

        else {

            break;
        }
    }

    return predecessor;
}
17. FINDING HEIGHT OF BST
Height measures the longest downward path from a node to a leaf.

We will use the convention:

Empty tree height = -1.
Leaf node height = 0.
Example

```text
          50
         /  \
        30   70
       /
      20
     /
    10
Longest path:


```text
50 → 30 → 20 → 10
There are three edges.

Therefore:

Height = 3.

Java Code

java
static int height(Node root) {

    if (root == null) {

        return -1;
    }

    int leftHeight = height(root.left);

    int rightHeight = height(root.right);

    return 1 + Math.max(leftHeight, rightHeight);
}
Explanation
The height of a node is:

1 + Maximum of left and right subtree heights.

Time Complexity: O(n).

18. COUNTING NODES AND LEAF NODES
18.1 Count Total Nodes
A node count includes every node in the tree.

Formula:


```text
Count = 1 + Left Subtree Count + Right Subtree Count
Java Code

java
static int countNodes(Node root) {

    if (root == null) {

        return 0;
    }

    return 1 +
        countNodes(root.left) +
        countNodes(root.right);
}
18.2 Count Leaf Nodes
A leaf node has no children.

Condition:


java
root.left == null && root.right == null
Java Code

java
static int countLeaves(Node root) {

    if (root == null) {

        return 0;
    }

    if (root.left == null &&
        root.right == null) {

        return 1;
    }

    return countLeaves(root.left) +
           countLeaves(root.right);
}
18.3 Count Internal Nodes
Internal nodes have at least one child.

Java Code

java
static int countInternal(Node root) {

    if (root == null ||
        (root.left == null && root.right == null)) {

        return 0;
    }

    return 1 +
        countInternal(root.left) +
        countInternal(root.right);
}
19. CHECK WHETHER A BINARY TREE IS A BST
This is an important interview question.

A common incorrect approach is checking only:


java
root.left.data < root.data

root.right.data > root.data
This is insufficient because a node must satisfy the ordering constraints of every ancestor.

Correct Approach: Range Validation
Every node must lie within an allowed minimum and maximum range.

For example:

Root 50: range (-∞, +∞).
Left child 30: range (-∞, 50).
Right child 70: range (50, +∞).
Right child of 30: range (30, 50).
Java Code

java
static boolean isBST(Node root) {

    return validate(root, Long.MIN_VALUE, Long.MAX_VALUE);
}

static boolean validate(Node root, long min, long max) {

    if (root == null) {

        return true;
    }

    if (root.data <= min || root.data >= max) {

        return false;
    }

    return validate(root.left, min, root.data) &&
           validate(root.right, root.data, max);
}
Example 1: Valid BST

```text
       50
      /  \
     30   70
Output:


```text
true
Example 2: Invalid BST

```text
       50
      /  \
     30   70
    / \
   20 60
Output:


```text
false
Why?

60 is greater than 50 but appears inside the left subtree of 50.

The range-validation algorithm catches this.

20. LOWEST COMMON ANCESTOR IN BST
Lowest Common Ancestor (LCA) is the lowest node that is an ancestor of two given nodes.

Consider:


```text
          50
         /  \
        30   70
       / \   / \
      20 40 60 80
Find LCA of 20 and 40.

Both nodes belong to the left subtree of 50.

At node 30, one value is smaller and the other is greater.

Therefore:

LCA = 30.

Rules
At current node:

If both keys are smaller, move left.
If both keys are greater, move right.
Otherwise, current node is the LCA.
Java Code

java
static Node lowestCommonAncestor(
    Node root, int n1, int n2) {

    while (root != null) {

        if (n1 < root.data && n2 < root.data) {

            root = root.left;
        }

        else if (n1 > root.data && n2 > root.data) {

            root = root.right;
        }

        else {

            return root;
        }
    }

    return null;
}
This implementation assumes both keys are present in the BST.

Example
LCA of 20 and 40:


```text
30
LCA of 20 and 80:


```text
50
LCA of 60 and 80:


```text
70
21. BALANCED VS SKEWED BST
The shape of a BST strongly affects its performance.

Balanced BST
A balanced tree has a height proportional to log₂(n).

Example:


```text
              50
             /  \
           30    70
          / \    / \
        20  40  60  80
Searching is efficient.

Time Complexity: O(log n) for a balanced tree.

Skewed BST
Consider inserting:


```text
10, 20, 30, 40, 50
The resulting tree:


```text
10
  \
   20
     \
      30
        \
         40
           \
            50
This behaves like a Linked List.

Searching may require checking every node.

Time Complexity: O(n).

Comparison

Feature	Balanced BST	Skewed BST
Height	O(log n)	O(n)
Searching	O(log n)	O(n)
Insertion	O(log n)	O(n)
Deletion	O(log n)	O(n)
Structure	Approximately balanced	One-sided
Important:

An ordinary BST does not automatically balance itself.

AVL Trees and Red-Black Trees are examples of self-balancing BST structures.

We will study AVL Trees in Part 3.

22. BST TIME AND SPACE COMPLEXITY
Let:

n = Number of nodes.
h = Height of the tree.

Operation	Average in balanced BST	Worst case
Search	O(log n)	O(n)
Insert	O(log n)	O(n)
Delete	O(log n)	O(n)
Find Minimum	O(log n)	O(n)
Find Maximum	O(log n)	O(n)
Find Height	O(n)	O(n)
Count Nodes	O(n)	O(n)
Inorder Traversal	O(n)	O(n)
For search, insertion and deletion, the more general complexity is O(h).

Space Complexity
Recursive operations use O(h) auxiliary stack space.

For a balanced BST:

O(log n).

For a skewed BST:

O(n).

Iterative searching uses O(1) auxiliary space.

23. IMPORTANT BST APPLICATIONS
Binary Search Trees are used in:

Maintaining sorted data.
Efficient searching.
Dynamic ordered collections.
Finding minimum and maximum values.
Finding predecessor and successor.
Range searching.
Implementing symbol tables.
Maintaining ordered records.
Java's TreeSet and TreeMap use Red-Black Tree-based implementations rather than a simple unbalanced BST.
````

26. IMPORTANT INTERVIEW CODING PROBLEMS
    Practise these problems independently.

Beginner Level
Search an element in BST.
Insert a node in BST.
Find minimum and maximum values.
Perform inorder traversal.
Count total nodes.
Count leaf nodes.
Find the height of a BST.
Intermediate Level
Delete a node from BST.
Find inorder successor.
Find inorder predecessor.
Validate whether a tree is a BST.
Find Lowest Common Ancestor.
Find Kth smallest element in BST.
Find Kth largest element in BST.
Advanced Level
Convert a sorted array into a balanced BST.
Recover a BST where two nodes are swapped.
Find the diameter of a Binary Tree.
Find the boundary traversal of a Binary Tree.
Find the vertical order traversal.
Construct a BST from preorder traversal.
Find the distance between two nodes.
Find the largest BST subtree inside a Binary Tree. 27. CHAPTER 7 – PART 2 SUMMARY
We have covered:

Topic Important Concept
BST Fundamentals Left < Root < Right
Searching Compare and move left/right
Insertion Insert at correct position
Inorder Produces sorted order
Minimum Leftmost node
Maximum Rightmost node
Deletion Three structural cases
Successor Minimum of right subtree, when it exists
Predecessor Maximum of left subtree, when it exists
Height Longest root-to-leaf path
Node Count Recursive counting
BST Validation Minimum/maximum range
LCA Find first splitting point
Balanced BST O(log n) operations
Skewed BST O(n) operations

---

# PART 3 – ADVANCED TREES, AVL TREES, HEAPS AND TREE ALGORITHMS WITH JAVA

---

1. INTRODUCTION TO ADVANCED TREES
   In previous parts, we studied:

Binary Trees.
Binary Search Trees.
Tree traversals.
Searching and insertion.
Deletion.
Height and depth.
However, ordinary Binary Search Trees have a limitation.

They can become unbalanced.

Consider inserting these elements into a BST:

10, 20, 30, 40, 50.

The resulting tree becomes:

````text
10
  \
   20
     \
      30
        \
         40
           \
            50
This is a skewed BST.

Searching for 50 may require visiting every node.

Time Complexity = O(n).

A balanced tree maintains a controlled height and can support searching, insertion and deletion in O(log n) time.

Advanced tree structures include:

AVL Trees.
Red-Black Trees.
Heaps.
B-Trees.
B+ Trees.
Tries.
Expression Trees.
Huffman Trees.
2. WHY DO WE NEED BALANCED TREES?
Consider two BSTs containing the same elements.

Unbalanced BST
```text
10
  \
   20
     \
      30
        \
         40
           \
            50
Height = 4 edges.

Balanced BST
```text
          30
         /  \
       20    40
      /        \
     10         50
Height = 2 edges.

The balanced tree requires fewer comparisons for many searches.

Comparison
Operation	Balanced BST	Skewed BST
Searching	O(log n)	O(n)
Insertion	O(log n)	O(n)
Deletion	O(log n)	O(n)
Traversal	O(n)	O(n)
Important: These balanced BST complexities assume the tree maintains its balance after updates.

3. BALANCED VS UNBALANCED TREES
A balanced tree maintains its height within an acceptable limit.

Different balanced tree structures use different balancing rules.

Examples:

AVL Tree: Strict height balancing.
Red-Black Tree: Colour-based balancing rules.
B-Tree: Multiple keys per node and multiple children.
B+ Tree: All records are stored at leaf level.
Why Are Balanced Trees Useful?
Faster searching.
Predictable performance.
Efficient insertion and deletion.
Useful for databases and memory management.
4. AVL TREE
An AVL Tree is a self-balancing Binary Search Tree.

It was introduced by Adelson-Velsky and Landis.

The name AVL comes from the surnames of its inventors.

An AVL Tree maintains the following condition:

For every node, the height difference between its left and right subtrees must be at most 1.

This is achieved using rotations.

Example
```text
          30
         /  \
       20    40
      /        \
     10         50
This is balanced.

Important Properties
AVL is a Binary Search Tree.
It follows BST ordering rules.
Every node maintains a balance factor between -1 and +1.
Rotations restore balance.
Searching takes O(log n).
Insertion takes O(log n).
Deletion takes O(log n).
5. BALANCE FACTOR
Balance Factor determines whether an AVL Tree node is balanced.

Formula:

Balance Factor = Height(Left Subtree) - Height(Right Subtree).

Possible Values
Balance Factor	Meaning
0	Left and right subtree heights are equal
+1	Left subtree is one level taller
-1	Right subtree is one level taller
+2	Left-heavy imbalance
-2	Right-heavy imbalance
For an AVL Tree, every node must have:

```text
-1 <= Balance Factor <= +1
Example
```text
          30
         /  \
       20    40
      /
     10
At node 30:

Left subtree height = 1.

Right subtree height = 0.

Balance Factor = 1 - 0 = +1.

Therefore, it is balanced.

6. AVL ROTATIONS
Rotations are operations that rearrange nodes while preserving BST ordering.

There are four major cases:

LL Rotation.
RR Rotation.
LR Rotation.
RL Rotation.
Understanding these four cases is extremely important for CDAC CMCE and technical interviews.

7. LL ROTATION – LEFT LEFT CASE
This occurs when insertion happens in the left subtree of the left child of an unbalanced node.

Insert:

30, 20, 10.

Before Rotation
```text
          30
         /
        20
       /
      10
Node 30 has balance factor +2.

This is an LL imbalance.

Solution: Right Rotation
After rotation:

```text
          20
         /  \
        10   30
Node 20 becomes the new root.

Java Code
java
static Node rightRotate(Node y) {

    Node x = y.left;

    Node T2 = x.right;

    x.right = y;

    y.left = T2;

    updateHeight(y);

    updateHeight(x);

    return x;
}
Explanation:

Store the left child of y in x.
Store x's right subtree in T2.
Make y the right child of x.
Attach T2 as the left child of y.
Update heights.
Return x as the new root.
8. RR ROTATION – RIGHT RIGHT CASE
This occurs when insertion happens in the right subtree of the right child.

Insert:

10, 20, 30.

Before Rotation
```text
10
  \
   20
     \
      30
Node 10 becomes unbalanced.

This is an RR imbalance.

Solution: Left Rotation
After rotation:

```text
          20
         /  \
        10   30
Java Code
java
static Node leftRotate(Node x) {

    Node y = x.right;

    Node T2 = y.left;

    y.left = x;

    x.right = T2;

    updateHeight(x);

    updateHeight(y);

    return y;
}
9. LR ROTATION – LEFT RIGHT CASE
This occurs when insertion happens in the right subtree of the left child.

Insert:

30, 10, 20.

Before Rotation
```text
          30
         /
        10
          \
           20
Node 30 becomes unbalanced.

This is an LR imbalance.

Solution
Two rotations are required.

Step 1: Left rotation on node 10.

```text
          30
         /
        20
       /
      10
Step 2: Right rotation on node 30.

```text
          20
         /  \
        10   30
Java Code
java
root.left = leftRotate(root.left);

return rightRotate(root);
10. RL ROTATION – RIGHT LEFT CASE
This occurs when insertion happens in the left subtree of the right child.

Insert:

10, 30, 20.

Before Rotation
```text
10
  \
   30
  /
 20
This is an RL imbalance.

Solution
Two rotations are required.

Step 1: Right rotation on node 30.

```text
10
  \
   20
     \
      30
Step 2: Left rotation on node 10.

```text
          20
         /  \
        10   30
Java Code
java
root.right = rightRotate(root.right);

return leftRotate(root);
Rotation Summary
Case	Insertion Position	Rotation
LL	Left of left child	Right Rotation
RR	Right of right child	Left Rotation
LR	Right of left child	Left then Right
RL	Left of right child	Right then Left
Memory trick:

LL → Right.
RR → Left.
LR → Left + Right.
RL → Right + Left.

11. COMPLETE AVL INSERTION IN JAVA
This is one of the most important implementations in advanced tree preparation.


java
class Node {

    int key;

    int height;

    Node left, right;

    Node(int value) {

        key = value;

        height = 1;
    }
}

public class AVLTree {

    Node root;

    int height(Node node) {

        return node == null ? 0 : node.height;
    }

    int getBalance(Node node) {

        if (node == null) return 0;

        return height(node.left) - height(node.right);
    }

    void updateHeight(Node node) {

        node.height = 1 + Math.max(
            height(node.left),
            height(node.right)
        );
    }

    Node rightRotate(Node y) {

        Node x = y.left;

        Node T2 = x.right;

        x.right = y;

        y.left = T2;

        updateHeight(y);

        updateHeight(x);

        return x;
    }

    Node leftRotate(Node x) {

        Node y = x.right;

        Node T2 = y.left;

        y.left = x;

        x.right = T2;

        updateHeight(x);

        updateHeight(y);

        return y;
    }

    Node insert(Node node, int key) {

        if (node == null) {

            return new Node(key);
        }

        if (key < node.key) {

            node.left = insert(node.left, key);

        } else if (key > node.key) {

            node.right = insert(node.right, key);

        } else {

            return node;
        }

        updateHeight(node);

        int balance = getBalance(node);

        // LL Case
        if (balance > 1 && key < node.left.key) {

            return rightRotate(node);
        }

        // RR Case
        if (balance < -1 && key > node.right.key) {

            return leftRotate(node);
        }

        // LR Case
        if (balance > 1 && key > node.left.key) {

            node.left = leftRotate(node.left);

            return rightRotate(node);
        }

        // RL Case
        if (balance < -1 && key < node.right.key) {

            node.right = rightRotate(node.right);

            return leftRotate(node);
        }

        return node;
    }

    void inorder(Node node) {

        if (node == null) return;

        inorder(node.left);

        System.out.print(node.key + " ");

        inorder(node.right);
    }

    public static void main(String[] args) {

        AVLTree tree = new AVLTree();

        tree.root = tree.insert(tree.root, 30);

        tree.root = tree.insert(tree.root, 20);

        tree.root = tree.insert(tree.root, 10);

        tree.inorder(tree.root);
    }
}
Output

```text
10 20 30
Final Tree:


```text
          20
         /  \
        10   30
Complexity

Operation	Complexity
Searching	O(log n)
Insertion	O(log n)
Deletion	O(log n)
Traversal	O(n)
Rotation	O(1)
12. AVL DELETION
Deletion in AVL Trees is similar to BST deletion.

After deleting a node, we update heights and check balance factors.

If any node becomes unbalanced, we perform the appropriate rotation.

Three BST Deletion Cases
Node is a leaf.
Node has one child.
Node has two children.
After Deletion
The tree must be rebalanced.

Unlike insertion, deletion may require rebalancing at multiple ancestors on the path toward the root.

Important Exam Point
AVL deletion is more complex than ordinary BST deletion because height and balance factor must be updated.

Time Complexity = O(log n).

13. INTRODUCTION TO HEAPS
A Heap is a specialized Complete Binary Tree that satisfies the Heap Property.

There are two major types:

Min Heap.
Max Heap.
Heaps are commonly used to implement Priority Queues.

Applications
Priority Queue.
CPU scheduling.
Dijkstra's shortest path algorithm.
Prim's minimum spanning tree algorithm.
Heap Sort.
Finding K largest or smallest elements.
14. MIN HEAP
A Min Heap is a Complete Binary Tree in which every parent node is smaller than or equal to its children.

Property:

Parent <= Children.

Example:


```text
          10
         /  \
        20   30
       / \   /
      40 50 60
The smallest element is always at the root.

Minimum = 10.

Important:

The left child does not necessarily have to be smaller than the right child.

Only the parent-child relationship matters.

15. MAX HEAP
A Max Heap is a Complete Binary Tree in which every parent node is greater than or equal to its children.

Property:

Parent >= Children.

Example:


```text
          90
         /  \
        70   80
       / \   /
      40 50 60
The maximum element is always at the root.

Maximum = 90.

16. HEAP PROPERTIES

Property	Min Heap	Max Heap
Root	Minimum	Maximum
Parent	Smaller than children	Greater than children
Structure	Complete Binary Tree	Complete Binary Tree
Insertion	O(log n)	O(log n)
Delete root	O(log n)	O(log n)
Peek root	O(1)	O(1)
Important:

A Heap is not a Binary Search Tree.

A Heap does not guarantee that all elements in the left subtree are smaller than all elements in the right subtree.

17. ARRAY REPRESENTATION OF HEAP
Heaps are generally implemented using arrays.

Consider a Max Heap:


```text
          90
         /  \
        70   80
       / \   /
      40 50 60
Array:


```text
Index:  0   1   2   3   4   5

Value: 90  70  80  40  50  60
For zero-based indexing:

Left child = 2i + 1.

Right child = 2i + 2.

Parent = (i - 1) / 2.

These are the same index formulas used for array-based complete binary trees.

18. INSERTION IN HEAP
Insertion is performed in two steps:

Insert the new element at the end.
Restore the heap property using Heapify Up.
Example
Insert 100 into this Max Heap:


```text
          90
         /  \
        70   80
       / \
      40 50
Step 1: Insert at the next available position.


```text
          90
         /  \
        70   80
       / \   /
      40 50 100
Step 2: Compare 100 with its parent 80.

Since 100 > 80, swap.


```text
          90
         /  \
        70   100
       / \   /
      40 50 80
Step 3: Compare 100 with parent 90.

Swap again.

Final Heap:


```text
          100
         /   \
        70    90
       / \   /
      40 50 80
Time Complexity = O(log n).

19. DELETION IN HEAP
Usually, deletion means removing the root element.

For a Max Heap:

Remove root.
Move last element to root.
Remove the last array position.
Apply Heapify Down.
Example
Initial Heap:


```text
          90
         /  \
        70   80
       / \
      40 50
Remove 90.

Move last element 50 to root.


```text
          50
         /  \
        70   80
       /
      40
Compare 50 with children 70 and 80.

Swap with the larger child 80.

Final Heap:


```text
          80
         /  \
        70   50
       /
      40
Time Complexity = O(log n).

20. HEAPIFY ALGORITHM
Heapify is used to restore the Heap Property.

There are two common operations:

Heapify Up.
Heapify Down.
Heapify Up
Used after insertion.

The new node is compared with its parent.

If the heap property is violated, swap them.

Continue until the property is restored.

Heapify Down
Used after deleting the root.

Compare the parent with its children.

Swap with the appropriate child.

Continue until the heap property is restored.

Java: Max Heapify Down

java
static void heapify(int[] arr, int n, int i) {

    int largest = i;

    int left = 2 * i + 1;

    int right = 2 * i + 2;

    if (left < n && arr[left] > arr[largest]) {

        largest = left;
    }

    if (right < n && arr[right] > arr[largest]) {

        largest = right;
    }

    if (largest != i) {

        int temp = arr[i];

        arr[i] = arr[largest];

        arr[largest] = temp;

        heapify(arr, n, largest);
    }
}
21. BUILDING A HEAP
We can build a Max Heap from an ordinary array.

Example:


```text
[4, 10, 3, 5, 1]
Using bottom-up heap construction, begin from the last non-leaf node.

Formula:

Last non-leaf index = n/2 - 1.

For n = 5:

Last non-leaf index = 1.

Apply heapify from index 1 down to index 0.

Result:


```text
[10, 5, 3, 4, 1]
Java Code

java
static void buildMaxHeap(int[] arr) {

    int n = arr.length;

    for (int i = n / 2 - 1; i >= 0; i--) {

        heapify(arr, n, i);
    }
}
Important Complexity
Building a Heap using bottom-up heap construction:

O(n).

This is a very important MCQ.

Although an individual heapify operation can take O(log n), building the entire heap takes O(n), not O(n log n).

22. HEAP SORT
Heap Sort is a comparison-based sorting algorithm that uses a Binary Heap.

For ascending order, we generally use a Max Heap.

Algorithm
Build a Max Heap.
Swap root with the last element.
Reduce the heap size.
Apply heapify to the root.
Repeat until the heap is exhausted.
Example
Input:


```text
[4, 10, 3, 5, 1]
After building Max Heap:


```text
[10, 5, 3, 4, 1]
Repeatedly move the maximum element to the end.

Final output:


```text
[1, 3, 4, 5, 10]
Complete Java Code

java
public class HeapSort {

    static void heapify(int[] arr, int n, int i) {

        int largest = i;

        int left = 2 * i + 1;

        int right = 2 * i + 2;

        if (left < n && arr[left] > arr[largest]) {

            largest = left;
        }

        if (right < n && arr[right] > arr[largest]) {

            largest = right;
        }

        if (largest != i) {

            int temp = arr[i];

            arr[i] = arr[largest];

            arr[largest] = temp;

            heapify(arr, n, largest);
        }
    }

    static void heapSort(int[] arr) {

        int n = arr.length;

        // Build Max Heap

        for (int i = n / 2 - 1; i >= 0; i--) {

            heapify(arr, n, i);
        }

        // Extract maximum elements

        for (int i = n - 1; i > 0; i--) {

            int temp = arr[0];

            arr[0] = arr[i];

            arr[i] = temp;

            heapify(arr, i, 0);
        }
    }

    public static void main(String[] args) {

        int[] arr = {4, 10, 3, 5, 1};

        heapSort(arr);

        for (int x : arr) {

            System.out.print(x + " ");
        }
    }
}
Output

```text
1 3 4 5 10
Heap Sort Complexity

Case	Complexity
Best	O(n log n)
Average	O(n log n)
Worst	O(n log n)
Auxiliary Space	O(log n) for this recursive heapify implementation
Important characteristics:

In-place sorting.
Not stable.
Guaranteed O(n log n) time.
Does not require an additional array proportional to n.
23. PRIORITY QUEUE IN JAVA
A Priority Queue is an abstract data type in which elements are processed according to priority rather than ordinary insertion order.

Java provides PriorityQueue in java.util.

By default, Java PriorityQueue behaves as a Min Priority Queue.

Example

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

        System.out.println(pq.peek());

        while (!pq.isEmpty()) {

            System.out.print(pq.poll() + " ");
        }
    }
}
Output

```text
10
10 20 30 40
Important Methods

Method	Description
add()	Inserts an element
offer()	Inserts an element
peek()	Returns highest-priority element without removing
poll()	Removes and returns highest-priority element
remove()	Removes an element
size()	Returns number of elements
isEmpty()	Checks whether queue is empty
Creating a Max Priority Queue

java
PriorityQueue<Integer> pq =
    new PriorityQueue<>(
        Collections.reverseOrder()
    );
Complete example:


java
import java.util.*;

public class Main {

    public static void main(String[] args) {

        PriorityQueue<Integer> pq =
            new PriorityQueue<>(
                Collections.reverseOrder()
            );

        pq.add(10);

        pq.add(40);

        pq.add(20);

        while (!pq.isEmpty()) {

            System.out.print(pq.poll() + " ");
        }
    }
}
Output:


```text
40 20 10
Complexity

Operation	Complexity
Insert	O(log n)
Peek	O(1)
Poll	O(log n)
Search for arbitrary element	O(n)
24. BINARY HEAP VS BST

Feature	Binary Heap	Binary Search Tree
Structure	Complete Binary Tree	No completeness requirement
Ordering	Parent-child heap property	Left < Root < Right
Minimum/Maximum	Root, depending on heap type	Depends on tree shape
Search arbitrary value	O(n)	O(h)
Insert	O(log n)	O(h)
Delete root	O(log n)	Depends on operation
Main application	Priority Queue	Searching and ordered data
Important:

A Heap is not suitable for efficient arbitrary searching in the same way as a balanced BST.

25. B-TREE
A B-Tree is a self-balancing multiway search tree.

Unlike a Binary Search Tree, a B-Tree node can contain multiple keys and have multiple children.

It is widely used in database systems and storage systems.

Example

```text
                 [20 | 40]
               /     |     \
          [10,15] [25,30] [50,60]
The root contains multiple keys.

These keys divide the ranges for child subtrees.

Important Properties
For a B-Tree of order m, using the convention that order means maximum children:

A node can have at most m children.
A node can contain at most m - 1 keys.
Except for the root, an internal node has at least ceil(m/2) children.
Keys inside a node are sorted.
All leaves are at the same depth.
The tree remains balanced.
The root has special minimum-occupancy rules.

Why B-Trees Are Used
Database indexing.
File systems.
Disk-based searching.
Large datasets.
Advantages
Small tree height.
Efficient disk access.
Supports searching, insertion and deletion.
Multiple keys per node reduce tree height.
Important Exam Point
B-Trees are designed to reduce the number of disk accesses when working with large datasets.

26. B+ TREE
A B+ Tree is a variation of a B-Tree.

Its important characteristics are:

Internal nodes store keys used for navigation.
Actual records or record pointers are stored at leaf nodes.
Leaf nodes are generally linked.
All leaves appear at the same depth.
Efficient range searching is supported.
Example

```text
                [20 | 40]
               /    |    \
          [10,15] [20,30] [40,50,60]

Leaf links:

[10,15] → [20,30] → [40,50,60]
B-Tree vs B+ Tree

Feature	B-Tree	B+ Tree
Data storage	Can be in internal and leaf nodes	Records stored at leaves
Internal nodes	Keys and possibly records	Navigation keys
Leaf linking	Not required	Generally linked
Range queries	Supported	Particularly efficient
Database use	Used	Widely used
Why Are B+ Trees Popular in Databases?
Linked leaf nodes make sequential and range-based retrieval efficient.

27. TRIE DATA STRUCTURE
A Trie is a tree-based data structure used to store strings.

It is also known as a Prefix Tree.

It is particularly useful for prefix searching.

Example Words
CAT, CAR, CARE, DOG.


```text
             Root
            /    \
           C      D
           |      |
           A      O
          / \     |
         T   R    G
             |
             E
A Trie shares common prefixes.

CAT, CAR and CARE share the prefix CA.

Applications
Autocomplete.
Dictionary implementation.
Spell checking.
Prefix matching.
Word search.
Trie Complexity
For a word of length L:

Insertion: O(L).
Search: O(L).
Prefix search: O(L).
The actual memory requirement depends on the alphabet and the number of stored prefixes.

Java Trie Implementation

java
class TrieNode {

    TrieNode[] children = new TrieNode[26];

    boolean isEnd;
}

class Trie {

    TrieNode root = new TrieNode();

    void insert(String word) {

        TrieNode current = root;

        for (char ch : word.toCharArray()) {

            int index = ch - 'a';

            if (current.children[index] == null) {

                current.children[index] = new TrieNode();
            }

            current = current.children[index];
        }

        current.isEnd = true;
    }

    boolean search(String word) {

        TrieNode current = root;

        for (char ch : word.toCharArray()) {

            int index = ch - 'a';

            if (current.children[index] == null) {

                return false;
            }

            current = current.children[index];
        }

        return current.isEnd;
    }
}

public class Main {

    public static void main(String[] args) {

        Trie trie = new Trie();

        trie.insert("cat");

        trie.insert("car");

        System.out.println(trie.search("cat"));

        System.out.println(trie.search("can"));

    }
}
Output:


```text
true
false
Note: This simple implementation supports lowercase English letters only.

28. EXPRESSION TREES
An Expression Tree is a Binary Tree used to represent arithmetic expressions.

Operands are stored in leaf nodes.

Operators are stored in internal nodes.

Example
Expression:


```text
A + B * C
Because multiplication has higher precedence:


```text
A + (B * C)
Expression Tree:


```text
          +
         / \
        A   *
           / \
          B   C
Traversal Results
Inorder:


```text
A + B * C
Preorder:


```text
+ A * B C
Postorder:


```text
A B C * +
Important Exam Point
Inorder gives infix representation.
Preorder gives prefix representation.
Postorder gives postfix representation.
Parentheses may be necessary when converting an arbitrary expression tree into an unambiguous infix expression.

29. HUFFMAN TREE
Huffman Coding is a lossless data compression technique.

It uses a binary tree to assign variable-length binary codes to characters.

Frequently occurring characters receive shorter codes.

Less frequently occurring characters receive longer codes.

Example Frequencies

Character	Frequency
A	5
B	9
C	12
D	13
E	16
F	45
Huffman Algorithm
Insert all characters into a Min Priority Queue.
Remove the two nodes having the smallest frequencies.
Create a new internal node with their combined frequency.
Insert the new node into the queue.
Repeat until one node remains.
The final node becomes the root.
Important Properties
Lossless compression.
Uses variable-length codes.
Prefix-free encoding.
Frequently occurring characters receive shorter codes.
Applications
File compression.
Data transmission.
Compression algorithms.
30. IMPORTANT TREE ALGORITHMS
The following algorithms are important for interviews and coding examinations.

30.1 Find Diameter of Binary Tree
Diameter is the longest path between any two nodes.

The path does not necessarily pass through the root.

Java Code

java
static int diameter = 0;

static int heightForDiameter(Node root) {

    if (root == null) {

        return 0;
    }

    int left = heightForDiameter(root.left);

    int right = heightForDiameter(root.right);

    diameter = Math.max(diameter, left + right);

    return 1 + Math.max(left, right);
}
Usage:


java
diameter = 0;

heightForDiameter(root);

System.out.println(diameter);
This implementation measures diameter in edges.

Time Complexity = O(n).

30.2 Mirror a Binary Tree
A mirror tree is created by swapping the left and right children of every node.

Java Code

java
static void mirror(Node root) {

    if (root == null) return;

    Node temp = root.left;

    root.left = root.right;

    root.right = temp;

    mirror(root.left);

    mirror(root.right);
}
Time Complexity = O(n).

30.3 Check Whether Two Trees Are Mirrors

java
static boolean isMirror(Node a, Node b) {

    if (a == null && b == null) {

        return true;
    }

    if (a == null || b == null) {

        return false;
    }

    return a.data == b.data &&
           isMirror(a.left, b.right) &&
           isMirror(a.right, b.left);
}
30.4 Find Sum of All Nodes

java
static int sumNodes(Node root) {

    if (root == null) {

        return 0;
    }

    return root.data +
           sumNodes(root.left) +
           sumNodes(root.right);
}
30.5 Count Nodes at a Given Level
Assume root is at level 0.


java
static int countAtLevel(Node root, int level) {

    if (root == null) {

        return 0;
    }

    if (level == 0) {

        return 1;
    }

    return countAtLevel(root.left, level - 1) +
           countAtLevel(root.right, level - 1);
}
30.6 Check Whether a Tree Is Balanced
A simple approach computes heights repeatedly, but that can take O(n²) time.

A better approach returns -1 when an unbalanced subtree is detected.


java
static int checkHeight(Node root) {

    if (root == null) {

        return 0;
    }

    int left = checkHeight(root.left);

    if (left == -1) return -1;

    int right = checkHeight(root.right);

    if (right == -1) return -1;

    if (Math.abs(left - right) > 1) {

        return -1;
    }

    return 1 + Math.max(left, right);
}

static boolean isBalanced(Node root) {

    return checkHeight(root) != -1;
}
Time Complexity = O(n).

31. JAVA CODING PRACTICE
Practise these problems in increasing order of difficulty.

Beginner
Implement Min Heap.
Implement Max Heap.
Find the maximum element in a Max Heap.
Find the minimum element in a Min Heap.
Implement PriorityQueue.
Implement Heap Sort.
Build a Heap from an array.
Implement Trie insertion.
Implement Trie searching.
Find the diameter of a Binary Tree.
Intermediate
Implement AVL insertion.
Implement all four AVL rotations.
Implement AVL deletion.
Check whether a Binary Tree is balanced.
Find the mirror of a Binary Tree.
Check whether two trees are mirrors.
Find the maximum width of a Binary Tree.
Find the kth smallest element in a BST.
Find the kth largest element in a BST.
Implement a Prefix Tree.
Advanced
Find Lowest Common Ancestor.
Construct a Huffman Tree.
Implement B-Tree insertion.
Implement B+ Tree searching.
Find maximum path sum in a Binary Tree.
Serialize and deserialize a Binary Tree.
Implement a Trie with prefix search.
Find top K frequent elements using PriorityQueue.
Merge K sorted arrays using a Min Heap.
Find median from a running data stream using two Heaps.
````

35. COMPLEXITY COMPARISON

Data Structure Search Insert Delete
Ordinary BST O(h) O(h) O(h)
AVL Tree O(log n) O(log n) O(log n)
Min/Max Heap O(n) O(log n) O(log n) for root
Trie O(L) O(L) O(L) for word removal with pruning
B-Tree O(log n) O(log n) O(log n)
B+ Tree O(log n) O(log n) O(log n)
Here:

n = Number of elements.
h = Height of tree.
L = Length of searched string.
For B-Trees and B+ Trees, complexity assumes a fixed node capacity; practical disk-based performance also depends on page size and I/O operations.

36. QUICK REVISION SHEET

Topic Important Fact
AVL Tree Self-balancing BST
AVL Inventors Adelson-Velsky and Landis
Balance Factor Height(left) - Height(right)
AVL Valid Balance -1, 0, +1
LL Rotation Right Rotation
RR Rotation Left Rotation
LR Rotation Left then Right
RL Rotation Right then Left
AVL Search O(log n)
Min Heap Minimum at root
Max Heap Maximum at root
Heap Structure Complete Binary Tree
Heap Insertion O(log n)
Heap Root Deletion O(log n)
Build Heap O(n)
Heap Sort O(n log n)
Heap Sort Stability Not stable
Java PriorityQueue Min Heap by default
B-Tree Multiway search tree
B+ Tree Records at leaves
Trie Prefix Tree
Huffman Tree Lossless compression
Expression Tree Arithmetic expressions

---

# Part 4 – Advanced Tree Algorithms, Problem Solving and Interview Preparation

## SECTION 4A – ADVANCED BINARY TREE PROBLEMS

1. INTRODUCTION
   In previous parts, we learned about:

Binary Tree fundamentals.
Binary Search Trees.
Tree traversals.
AVL Trees.
Heaps.
B-Trees and B+ Trees.
Tries.
Expression Trees.
Huffman Trees.
Now we will solve more advanced Binary Tree problems.

These problems are important because they test whether we understand tree traversal, recursion, node relationships and tree structure.

We will use the following Node class for several examples.

Java Node Class
java
class Node {

    int data;

    Node left;
    Node right;

    Node(int data) {

        this.data = data;

        left = null;

        right = null;
    }

}
Consider this example tree:

````text
             1
           /   \
          2     3
         / \   / \
        4   5 6   7
Its properties:

Root = 1
Leaf nodes = 4, 5, 6, 7
Height = 2 edges
Total nodes = 7
Total leaves = 4
For this section, we use root level = 0 and measure height in edges unless stated otherwise.

2. LEFT VIEW OF A BINARY TREE
What is Left View?
The Left View of a Binary Tree consists of the nodes visible when the tree is viewed from its left side.

Only the first node at each level is visible.

Consider:

```text
             1
           /   \
          2     3
         / \   / \
        4   5 6   7
From the left side, we see:

```text
1
2
4
Therefore:

Output = 1 2 4

Approach 1: Recursive DFS
We maintain a variable called maxLevel.

Whenever we visit a new level for the first time, we print that node.

Java Code
java
class LeftView {

    static int maxLevel = -1;

    static void leftView(Node root, int level) {

        if (root == null) {
            return;
        }

        if (level > maxLevel) {

            System.out.print(root.data + " ");

            maxLevel = level;
        }

        leftView(root.left, level + 1);

        leftView(root.right, level + 1);
    }

    public static void main(String[] args) {

        Node root = new Node(1);

        root.left = new Node(2);
        root.right = new Node(3);

        root.left.left = new Node(4);
        root.left.right = new Node(5);

        root.right.left = new Node(6);
        root.right.right = new Node(7);

        leftView(root, 0);
    }
}
Output
```text
1 2 4
Explanation
The recursive function visits the left subtree before the right subtree.

The first node encountered at every level is printed.

Time Complexity = O(n).

Space Complexity = O(h), where h is the recursion stack height.

Important: maxLevel must be reset before processing another tree.

3. RIGHT VIEW OF A BINARY TREE
What is Right View?
The Right View contains the nodes visible when looking at a Binary Tree from its right side.

For the same tree:

```text
             1
           /   \
          2     3
         / \   / \
        4   5 6   7
Right View:

```text
1
3
7
Output = 1 3 7

Java Code
java
static int maxLevel = -1;

static void rightView(Node root, int level) {

    if (root == null) {
        return;
    }

    if (level > maxLevel) {

        System.out.print(root.data + " ");

        maxLevel = level;
    }

    rightView(root.right, level + 1);

    rightView(root.left, level + 1);
}
The only important difference from Left View is traversal order.

Left View: Left before Right.
Right View: Right before Left.
Time Complexity = O(n).

4. TOP VIEW OF A BINARY TREE
What is Top View?
Top View contains the nodes visible when looking at the tree from above.

We use Horizontal Distance (HD).

Rules:

Root has HD = 0.
Left child has HD = HD - 1.
Right child has HD = HD + 1.
Consider:

```text
             1
           /   \
          2     3
         / \   / \
        4   5 6   7
Horizontal distances:

```text
                 1(0)
                /    \
             2(-1)   3(+1)
             /  \    /  \
          4(-2) 5(0) 6(0) 7(+2)
The first node encountered at each horizontal distance is visible from the top.

Top View:

```text
4 2 1 3 7
Java Implementation Using BFS
We use:

Queue for level order traversal.
HashMap to store the first node at each horizontal distance.
java
import java.util.*;

class TopView {

    static class Pair {

        Node node;

        int hd;

        Pair(Node node, int hd) {

            this.node = node;

            this.hd = hd;
        }
    }

    static void topView(Node root) {

        if (root == null) {
            return;
        }

        Queue<Pair> queue = new LinkedList<>();

        TreeMap<Integer, Integer> map = new TreeMap<>();

        queue.add(new Pair(root, 0));

        while (!queue.isEmpty()) {

            Pair current = queue.poll();

            Node node = current.node;

            int hd = current.hd;

            if (!map.containsKey(hd)) {

                map.put(hd, node.data);
            }

            if (node.left != null) {

                queue.add(new Pair(node.left, hd - 1));
            }

            if (node.right != null) {

                queue.add(new Pair(node.right, hd + 1));
            }
        }

        for (int value : map.values()) {

            System.out.print(value + " ");
        }
    }
}
Output
```text
4 2 1 3 7
Complexity
Time Complexity = O(n log n) using TreeMap.

Space Complexity = O(n).

5. BOTTOM VIEW OF A BINARY TREE
What is Bottom View?
Bottom View contains the nodes visible when looking at the tree from below.

For each horizontal distance, we retain the lowest node.

Example:

```text
             1
           /   \
          2     3
         / \   / \
        4   5 6   7
Bottom View:

```text
4 2 6 3 7
At horizontal distance 0, nodes 1, 5 and 6 may compete.

The bottom-most node is 6.

Java Code
java
import java.util.*;

class BottomView {

    static class Pair {

        Node node;

        int hd;

        Pair(Node node, int hd) {

            this.node = node;

            this.hd = hd;
        }
    }

    static void bottomView(Node root) {

        if (root == null) {
            return;
        }

        Queue<Pair> queue = new LinkedList<>();

        TreeMap<Integer, Integer> map = new TreeMap<>();

        queue.add(new Pair(root, 0));

        while (!queue.isEmpty()) {

            Pair current = queue.poll();

            Node node = current.node;

            int hd = current.hd;

            map.put(hd, node.data);

            if (node.left != null) {

                queue.add(new Pair(node.left, hd - 1));
            }

            if (node.right != null) {

                queue.add(new Pair(node.right, hd + 1));
            }
        }

        for (int value : map.values()) {

            System.out.print(value + " ");
        }
    }
}
Output
```text
4 2 6 3 7
Important: If multiple nodes have the same horizontal distance and depth, the result can depend on the specified tie-breaking convention. This implementation uses BFS and lets the later-visited node replace the previous value.

6. ZIGZAG LEVEL ORDER TRAVERSAL
What is Zigzag Traversal?
Normal Level Order Traversal visits nodes from left to right at every level.

Zigzag Traversal alternates direction.

Example:

```text
             1
           /   \
          2     3
         / \   / \
        4   5 6   7
Normal Level Order:

```text
1 2 3 4 5 6 7
Zigzag Traversal:

```text
1
3 2
4 5 6 7
Output:

```text
1 3 2 4 5 6 7
Java Code Using Queue and Deque
java
import java.util.*;

class Zigzag {

    static void zigzag(Node root) {

        if (root == null) {
            return;
        }

        Queue<Node> queue = new LinkedList<>();

        boolean leftToRight = true;

        queue.add(root);

        while (!queue.isEmpty()) {

            int size = queue.size();

            Deque<Integer> level = new ArrayDeque<>();

            for (int i = 0; i < size; i++) {

                Node node = queue.poll();

                if (leftToRight) {

                    level.addLast(node.data);

                } else {

                    level.addFirst(node.data);
                }

                if (node.left != null) {

                    queue.add(node.left);
                }

                if (node.right != null) {

                    queue.add(node.right);
                }
            }

            while (!level.isEmpty()) {

                System.out.print(level.pollFirst() + " ");
            }

            leftToRight = !leftToRight;
        }
    }
}
Output
```text
1 3 2 4 5 6 7
Time Complexity = O(n).

Space Complexity = O(n).

7. BOUNDARY TRAVERSAL
Boundary Traversal visits the outer boundary of a Binary Tree in an anti-clockwise direction.

It generally includes:

Root.
Left boundary excluding leaves.
All leaf nodes from left to right.
Right boundary excluding leaves, in reverse order.
Example:

```text
             1
           /   \
          2     3
         / \   / \
        4   5 6   7
           / \
          8   9
Boundary Traversal:

```text
1 2 4 8 9 6 7 3
Java Code
java
import java.util.*;

class BoundaryTraversal {

    static boolean isLeaf(Node node) {

        return node != null &&
               node.left == null &&
               node.right == null;
    }

    static void addLeftBoundary(Node root, List<Integer> result) {

        Node current = root.left;

        while (current != null) {

            if (!isLeaf(current)) {

                result.add(current.data);
            }

            if (current.left != null) {

                current = current.left;

            } else {

                current = current.right;
            }
        }
    }

    static void addLeaves(Node root, List<Integer> result) {

        if (root == null) {
            return;
        }

        if (isLeaf(root)) {

            result.add(root.data);

            return;
        }

        addLeaves(root.left, result);

        addLeaves(root.right, result);
    }

    static void addRightBoundary(Node root, List<Integer> result) {

        Node current = root.right;

        Stack<Integer> stack = new Stack<>();

        while (current != null) {

            if (!isLeaf(current)) {

                stack.push(current.data);
            }

            if (current.right != null) {

                current = current.right;

            } else {

                current = current.left;
            }
        }

        while (!stack.isEmpty()) {

            result.add(stack.pop());
        }
    }

    static List<Integer> boundary(Node root) {

        List<Integer> result = new ArrayList<>();

        if (root == null) {
            return result;
        }

        if (!isLeaf(root)) {

            result.add(root.data);
        }

        addLeftBoundary(root, result);

        addLeaves(root, result);

        addRightBoundary(root, result);

        return result;
    }
}
Time Complexity = O(n).

Space Complexity = O(n), including the result list.

8. CHECK WHETHER A BINARY TREE IS SYMMETRIC
A Binary Tree is symmetric if its left and right subtrees are mirror images.

Example:

```text
             1
           /   \
          2     2
         / \   / \
        3   4 4   3
This tree is symmetric.

Java Code
java
class SymmetricTree {

    static boolean isMirror(Node a, Node b) {

        if (a == null && b == null) {

            return true;
        }

        if (a == null || b == null) {

            return false;
        }

        return a.data == b.data &&
               isMirror(a.left, b.right) &&
               isMirror(a.right, b.left);
    }

    static boolean isSymmetric(Node root) {

        if (root == null) {
            return true;
        }

        return isMirror(root.left, root.right);
    }
}
Output:

```text
true
Time Complexity = O(n).

9. ROOT-TO-LEAF PATHS
A root-to-leaf path is a sequence of nodes beginning at the root and ending at a leaf.

Example:

```text
             1
           /   \
          2     3
         / \
        4   5
Root-to-leaf paths:

```text
1 -> 2 -> 4
1 -> 2 -> 5
1 -> 3
Java Code
java
import java.util.*;

class RootToLeaf {

    static void printPaths(
        Node root,
        List<Integer> path
    ) {

        if (root == null) {
            return;
        }

        path.add(root.data);

        if (root.left == null && root.right == null) {

            System.out.println(path);

        } else {

            printPaths(root.left, path);

            printPaths(root.right, path);
        }

        path.remove(path.size() - 1);
    }
}
Output
```text
[1, 2, 4]
[1, 2, 5]
[1, 3]
The important concept here is Backtracking.

We add the current node, explore its children, and remove the node before returning to the parent.

10. MAXIMUM PATH SUM IN A BINARY TREE
This is an important interview problem.

The maximum path sum is the largest sum of node values along any valid path.

The path can:

Start at any node.
End at any node.
Pass through the root, but does not have to.
Example:

```text
             -10
            /   \
           9     20
                /  \
               15   7
Possible maximum path:

```text
15 -> 20 -> 7
Sum:

15 + 20 + 7 = 42.

Answer = 42.

Java Code
java
class MaximumPathSum {

    static int maxSum;

    static int helper(Node root) {

        if (root == null) {
            return 0;
        }

        int left = Math.max(0, helper(root.left));

        int right = Math.max(0, helper(root.right));

        int currentSum = root.data + left + right;

        maxSum = Math.max(maxSum, currentSum);

        return root.data + Math.max(left, right);
    }

    static int maximumPathSum(Node root) {

        maxSum = Integer.MIN_VALUE;

        helper(root);

        return maxSum;
    }
}
Complexity
Time = O(n).

Auxiliary Space = O(h).

Important: The use of Math.max(0, subtreeSum) ensures that a negative contribution is not included in the best path.
````

## SECTION 4B – ADVANCED BINARY SEARCH TREE ALGORITHMS

11. LOWEST COMMON ANCESTOR (LCA)
    What is LCA?
    The Lowest Common Ancestor of two nodes is the lowest node in a tree that has both nodes as descendants.

A node can be considered a descendant of itself.

Example:

````text
             1
           /   \
          2     3
         / \
        4   5
LCA of 4 and 5 = 2.

LCA of 4 and 3 = 1.

Java Code for a General Binary Tree

java
class LCA {

    static Node findLCA(Node root, int n1, int n2) {

        if (root == null) {
            return null;
        }

        if (root.data == n1 || root.data == n2) {

            return root;
        }

        Node left = findLCA(root.left, n1, n2);

        Node right = findLCA(root.right, n1, n2);

        if (left != null && right != null) {

            return root;
        }

        return left != null ? left : right;
    }
}
Time Complexity = O(n).

Important limitation: This basic implementation assumes both requested keys are present. If existence is not guaranteed, first verify both nodes exist.

LCA in a BST
Because a BST is ordered, we can solve the problem more efficiently.

Rules:

If both values are smaller than the current node, go left.
If both are greater, go right.
Otherwise, the current node is the LCA.

java
static Node lcaBST(Node root, int n1, int n2) {

    if (root == null) {
        return null;
    }

    if (n1 < root.data && n2 < root.data) {

        return lcaBST(root.left, n1, n2);
    }

    if (n1 > root.data && n2 > root.data) {

        return lcaBST(root.right, n1, n2);
    }

    return root;
}
Time Complexity:

Balanced BST: O(log n).
Skewed BST: O(n).
12. VALIDATE A BINARY SEARCH TREE
A Binary Tree is a BST only if it satisfies the ordering rule throughout the entire tree.

For every node:

All values in the left subtree must be smaller.
All values in the right subtree must be greater.
Checking only immediate children is insufficient.

Example:


```text
          10
         /  \
        5    15
            /
           6
This is NOT a valid BST.

Although 6 is smaller than 15, it belongs to the right subtree of 10 and must therefore be greater than 10.

Java Code Using Range Limits

java
class ValidateBST {

    static boolean isValid(
        Node root,
        long min,
        long max
    ) {

        if (root == null) {
            return true;
        }

        if (root.data <= min || root.data >= max) {

            return false;
        }

        return isValid(root.left, min, root.data) &&
               isValid(root.right, root.data, max);
    }

    static boolean isBST(Node root) {

        return isValid(
            root,
            Long.MIN_VALUE,
            Long.MAX_VALUE
        );
    }
}
Time Complexity = O(n).

This implementation assumes duplicate keys are not allowed.

13. KTH SMALLEST ELEMENT IN BST
Inorder traversal of a BST visits values in sorted ascending order.

Therefore, the kth visited node is the kth smallest element.

Example:


```text
             5
           /   \
          3     7
         / \   / \
        2   4 6   8
Inorder:


```text
2 3 4 5 6 7 8
3rd smallest = 4.

Java Code

java
class KthSmallest {

    static int count = 0;

    static int answer;

    static void inorder(Node root, int k) {

        if (root == null) {
            return;
        }

        inorder(root.left, k);

        count++;

        if (count == k) {

            answer = root.data;

            return;
        }

        inorder(root.right, k);
    }

    static int findKthSmallest(Node root, int k) {

        count = 0;

        answer = -1;

        inorder(root, k);

        return count >= k ? answer : -1;
    }
}
For a production-quality implementation, an object or wrapper can be used instead of static state.

Time Complexity = O(h + k) when traversal stops immediately at the kth node.

Worst Case = O(n).

14. KTH LARGEST ELEMENT IN BST
The reverse of inorder traversal gives descending order.

That is:

Right → Root → Left.

Example:


```text
             5
           /   \
          3     7
         / \   / \
        2   4 6   8
Reverse Inorder:


```text
8 7 6 5 4 3 2
3rd largest = 6.

Java Code

java
class KthLargest {

    static int count;

    static int answer;

    static void reverseInorder(Node root, int k) {

        if (root == null || count >= k) {
            return;
        }

        reverseInorder(root.right, k);

        if (count < k) {

            count++;

            if (count == k) {

                answer = root.data;

                return;
            }
        }

        reverseInorder(root.left, k);
    }

    static int findKthLargest(Node root, int k) {

        count = 0;

        answer = -1;

        reverseInorder(root, k);

        return count >= k ? answer : -1;
    }
}
15. CONSTRUCT TREE FROM INORDER AND PREORDER
This is a very important tree construction problem.

Traversal Basics
Preorder:

Root → Left → Right.

Inorder:

Left → Root → Right.

Example:

Preorder:


```text
1 2 4 5 3 6 7
Inorder:


```text
4 2 5 1 6 3 7
The first element in preorder is always the root.

Here, root = 1.

Find 1 in inorder:


```text
4 2 5 | 1 | 6 3 7
Everything to the left belongs to the left subtree.

Everything to the right belongs to the right subtree.

Java Code

java
import java.util.*;

class BuildTree {

    static int preIndex = 0;

    static Map<Integer, Integer> map;

    static Node build(
        int[] preorder,
        int left,
        int right
    ) {

        if (left > right) {
            return null;
        }

        int value = preorder[preIndex++];

        Node root = new Node(value);

        int position = map.get(value);

        root.left = build(
            preorder,
            left,
            position - 1
        );

        root.right = build(
            preorder,
            position + 1,
            right
        );

        return root;
    }

    static Node construct(int[] preorder, int[] inorder) {

        preIndex = 0;

        map = new HashMap<>();

        for (int i = 0; i < inorder.length; i++) {

            map.put(inorder[i], i);
        }

        return build(
            preorder,
            0,
            inorder.length - 1
        );
    }
}
Complexity
Time Complexity = O(n) average with HashMap.

Space Complexity = O(n).

Assumption: All node values are distinct.

16. CONSTRUCT TREE FROM INORDER AND POSTORDER
Postorder:

Left → Right → Root.

The last element in postorder is the root.

Example:

Postorder:


```text
4 5 2 6 7 3 1
Inorder:


```text
4 2 5 1 6 3 7
Root = 1.

The important difference is that while processing postorder backwards, we must build the right subtree before the left subtree.

Java Code

java
import java.util.*;

class BuildFromPostorder {

    static int postIndex;

    static Map<Integer, Integer> map;

    static Node build(
        int[] postorder,
        int left,
        int right
    ) {

        if (left > right) {
            return null;
        }

        int value = postorder[postIndex--];

        Node root = new Node(value);

        int position = map.get(value);

        root.right = build(
            postorder,
            position + 1,
            right
        );

        root.left = build(
            postorder,
            left,
            position - 1
        );

        return root;
    }

    static Node construct(int[] inorder, int[] postorder) {

        map = new HashMap<>();

        postIndex = postorder.length - 1;

        for (int i = 0; i < inorder.length; i++) {

            map.put(inorder[i], i);
        }

        return build(
            postorder,
            0,
            inorder.length - 1
        );
    }
}
Time Complexity = O(n) average.

Assumption: Distinct values and valid traversal sequences.

17. SERIALIZATION AND DESERIALIZATION OF BINARY TREE
Serialization means converting a Binary Tree into a string or another linear representation.

Deserialization means reconstructing the original tree from that representation.

This is useful in:

Data storage.
Network communication.
Tree copying.
Saving and restoring tree structures.
Example:


```text
          1
         / \
        2   3
           / \
          4   5
One possible serialized representation:


```text
1,2,#,#,3,4,#,#,5,#,#
Here, # represents a null child.

Java Code

java
import java.util.*;

class Codec {

    void serializeHelper(Node root, List<String> list) {

        if (root == null) {

            list.add("#");

            return;
        }

        list.add(String.valueOf(root.data));

        serializeHelper(root.left, list);

        serializeHelper(root.right, list);
    }

    String serialize(Node root) {

        List<String> list = new ArrayList<>();

        serializeHelper(root, list);

        return String.join(",", list);
    }

    Node deserializeHelper(Queue<String> queue) {

        String value = queue.poll();

        if (value.equals("#")) {

            return null;
        }

        Node root = new Node(Integer.parseInt(value));

        root.left = deserializeHelper(queue);

        root.right = deserializeHelper(queue);

        return root;
    }

    Node deserialize(String data) {

        Queue<String> queue = new LinkedList<>(
            Arrays.asList(data.split(","))
        );

        return deserializeHelper(queue);
    }
}
Time Complexity = O(n).

Space Complexity = O(n).

The serialized representation preserves the tree structure because null children are also recorded.

SECTION 4C – ADVANCED TREE STRUCTURES AND ALGORITHMS
18. RED-BLACK TREE
A Red-Black Tree is a self-balancing Binary Search Tree that uses node colours to maintain balance.

Each node is either:

Red.
Black.
Properties of Red-Black Tree
Every node is either red or black.
The root is black.
All NIL leaves are black.
A red node cannot have a red child.
Every path from a node to its descendant NIL leaves contains the same number of black nodes.
Example

```text
              20(B)
             /     \
          10(R)    30(R)
B = Black.

R = Red.

Why Use Red-Black Trees?
They provide logarithmic worst-case search, insertion and deletion.

They are useful when frequent insertions and deletions are expected.

AVL vs Red-Black Tree

Feature	AVL	Red-Black
Balancing	Height-based	Colour-based
Balance	More strictly balanced	Less strictly balanced
Searching	Often faster in practice for lookup-heavy workloads	O(log n)
Updates	May require more rebalancing	Often fewer rotations
Main use	Lookup-heavy applications	Frequently updated ordered data
Both support O(log n) worst-case search, insertion and deletion.

Java's TreeMap and TreeSet use Red-Black Tree-based implementations.

19. SEGMENT TREE
A Segment Tree is a tree-based data structure used for efficient range queries and updates on arrays.

It is useful when we need to repeatedly answer questions such as:

What is the sum from index 2 to index 6?
What is the minimum value in a range?
What is the maximum value in a range?
Update an element and answer another range query.
Example
Array:


```text
[2, 4, 1, 7, 3, 5, 6, 8]
Suppose we want the sum of elements from index 1 to index 4.


```text
4 + 1 + 7 + 3 = 15
A normal loop takes O(n) in the worst case.

A Segment Tree can answer a range-sum query in O(log n).

Main Operations

Operation	Complexity
Build	O(n)
Range Sum Query	O(log n)
Point Update	O(log n)
Space	O(n)
Java Implementation: Range Sum Segment Tree

java
class SegmentTree {

    int[] tree;

    int n;

    SegmentTree(int[] arr) {

        n = arr.length;

        tree = new int[4 * n];

        build(arr, 0, 0, n - 1);
    }

    void build(
        int[] arr,
        int node,
        int start,
        int end
    ) {

        if (start == end) {

            tree[node] = arr[start];

        } else {

            int mid = (start + end) / 2;

            build(arr, 2 * node + 1, start, mid);

            build(arr, 2 * node + 2, mid + 1, end);

            tree[node] =
                tree[2 * node + 1] +
                tree[2 * node + 2];
        }
    }

    int query(
        int node,
        int start,
        int end,
        int left,
        int right
    ) {

        if (right < start || end < left) {

            return 0;
        }

        if (left <= start && end <= right) {

            return tree[node];
        }

        int mid = (start + end) / 2;

        int a = query(
            2 * node + 1,
            start,
            mid,
            left,
            right
        );

        int b = query(
            2 * node + 2,
            mid + 1,
            end,
            left,
            right
        );

        return a + b;
    }

    void update(
        int node,
        int start,
        int end,
        int index,
        int value
    ) {

        if (start == end) {

            tree[node] = value;

        } else {

            int mid = (start + end) / 2;

            if (index <= mid) {

                update(
                    2 * node + 1,
                    start,
                    mid,
                    index,
                    value
                );

            } else {

                update(
                    2 * node + 2,
                    mid + 1,
                    end,
                    index,
                    value
                );
            }

            tree[node] =
                tree[2 * node + 1] +
                tree[2 * node + 2];
        }
    }
}
Main Method

java
public class Main {

    public static void main(String[] args) {

        int[] arr = {2, 4, 1, 7, 3, 5, 6, 8};

        SegmentTree st = new SegmentTree(arr);

        System.out.println(
            st.query(0, 0, 7, 1, 4)
        );

        st.update(0, 0, 7, 2, 10);

        System.out.println(
            st.query(0, 0, 7, 1, 4)
        );
    }
}
Output:


```text
15
24
Explanation:

Original sum:

4 + 1 + 7 + 3 = 15.

After changing index 2 from 1 to 10:

4 + 10 + 7 + 3 = 24.

20. FENWICK TREE – BINARY INDEXED TREE
A Fenwick Tree is also called a Binary Indexed Tree (BIT).

It supports efficient prefix-sum queries and point updates.

It is generally simpler to implement than a Segment Tree for prefix-sum operations.

Main Operations
Update an element.
Calculate prefix sum.
Calculate range sum.
Complexity

Operation	Complexity
Build using repeated updates	O(n log n)
Point Update	O(log n)
Prefix Sum	O(log n)
Range Sum	O(log n)
Space	O(n)
Important Concept: Lowest Set Bit
Fenwick Tree uses:


java
i & -i
This extracts the lowest set bit of an integer.

Example:


```text
i = 12

Binary = 1100

-i = 0100 in the relevant two's-complement bit pattern

i & -i = 0100

Result = 4
Java Implementation

java
class FenwickTree {

    int[] bit;

    int n;

    FenwickTree(int n) {

        this.n = n;

        bit = new int[n + 1];
    }

    void update(int index, int value) {

        while (index <= n) {

            bit[index] += value;

            index += index & -index;
        }
    }

    int prefixSum(int index) {

        int sum = 0;

        while (index > 0) {

            sum += bit[index];

            index -= index & -index;
        }

        return sum;
    }

    int rangeSum(int left, int right) {

        return prefixSum(right) -
               prefixSum(left - 1);
    }
}
Example

java
public class Main {

    public static void main(String[] args) {

        FenwickTree ft = new FenwickTree(5);

        int[] arr = {2, 4, 1, 7, 3};

        for (int i = 0; i < arr.length; i++) {

            ft.update(i + 1, arr[i]);
        }

        System.out.println(ft.prefixSum(3));

        System.out.println(ft.rangeSum(2, 4));
    }
}
Output:


```text
7
12
Explanation:

Prefix sum of first 3 elements:

2 + 4 + 1 = 7.

Range sum from position 2 to 4:

4 + 1 + 7 = 12.

Important: Fenwick Tree uses 1-based indexing in this implementation.

21. MORRIS TRAVERSAL
Morris Traversal is a technique to perform tree traversal without using recursion or an explicit stack.

It uses temporary links called threads.

Its main advantage is:

Time Complexity = O(n).
Auxiliary Space = O(1).
It is commonly used to perform Inorder Traversal.

Normal Inorder
Left → Root → Right.

Normally, recursion or a stack is used.

Morris Traversal temporarily modifies certain right pointers and restores them during traversal.

Java Code

java
class MorrisTraversal {

    static void inorder(Node root) {

        Node current = root;

        while (current != null) {

            if (current.left == null) {

                System.out.print(current.data + " ");

                current = current.right;

            } else {

                Node predecessor = current.left;

                while (
                    predecessor.right != null &&
                    predecessor.right != current
                ) {

                    predecessor = predecessor.right;
                }

                if (predecessor.right == null) {

                    predecessor.right = current;

                    current = current.left;

                } else {

                    predecessor.right = null;

                    System.out.print(current.data + " ");

                    current = current.right;
                }
            }
        }
    }
}
Example

```text
             1
           /   \
          2     3
         / \
        4   5
Output:


```text
4 2 5 1 3
Why Is Morris Traversal Important?
It achieves O(1) auxiliary space, unlike recursive traversal, which requires O(h) stack space.

Important: The tree must be restored before the operation finishes. This implementation restores all temporary threads during normal completion.

SECTION 4D – CDAC CMCE EXAM AND INTERVIEW PREPARATION
22. IMPORTANT TREE COMPLEXITY TABLE

Algorithm	Time Complexity	Auxiliary Space
Inorder Traversal	O(n)	O(h)
Preorder Traversal	O(n)	O(h)
Postorder Traversal	O(n)	O(h)
Level Order Traversal	O(n)	O(n)
Left View	O(n)	O(h)
Right View	O(n)	O(h)
Top View	O(n log n)	O(n)
Bottom View	O(n log n)	O(n)
Zigzag Traversal	O(n)	O(n)
Boundary Traversal	O(n)	O(n)
Diameter	O(n)	O(h)
Maximum Path Sum	O(n)	O(h)
LCA in Binary Tree	O(n)	O(h)
LCA in Balanced BST	O(log n)	O(log n) recursive
Validate BST	O(n)	O(h)
Construct Tree	O(n) average	O(n)
Serialize Tree	O(n)	O(n)
Segment Tree Query	O(log n)	O(log n) recursion
Fenwick Prefix Sum	O(log n)	O(1)
Morris Inorder	O(n)	O(1)
Here:

n = Number of nodes.
h = Height of the tree.
````

25. IMPORTANT JAVA CODING INTERVIEW QUESTIONS
    These are useful practice problems for CDAC CMCE preparation and general technical interviews.

Beginner Level
Find height of Binary Tree.
Count total nodes.
Count leaf nodes.
Find sum of all nodes.
Print left view.
Print right view.
Find maximum element.
Find minimum element.
Check whether two trees are identical.
Check whether a tree is symmetric.
Intermediate Level
Zigzag Level Order Traversal.
Top View of Binary Tree.
Bottom View of Binary Tree.
Boundary Traversal.
Lowest Common Ancestor.
Validate BST.
Kth Smallest in BST.
Kth Largest in BST.
Root-to-Leaf Paths.
Construct Tree from Inorder and Preorder.
Advanced Level
Construct Tree from Inorder and Postorder.
Maximum Path Sum.
Diameter of Binary Tree.
Serialize and Deserialize Tree.
Morris Inorder Traversal.
Recover a BST where two nodes were swapped.
Maximum Width of Binary Tree.
Vertical Order Traversal.
Segment Tree Range Sum Query.
Fenwick Tree Prefix Sum.

27. FINAL CHAPTER 7 REVISION SHEET

Topic Key Point
Binary Tree Maximum two children
BST Left < Root < Right
Inorder Left, Root, Right
Preorder Root, Left, Right
Postorder Left, Right, Root
Level Order BFS using Queue
Left View First node at each level from left
Right View First node at each level from right
Top View First node at each horizontal distance
Bottom View Bottom-most node at each horizontal distance
Zigzag Alternating level direction
Boundary Anti-clockwise boundary
Symmetric Tree Left and right are mirrors
LCA Lowest Common Ancestor
Kth Smallest BST Inorder
Kth Largest BST Reverse Inorder
Tree Construction Inorder + Preorder/Postorder
Serialization Tree to linear representation
Morris Traversal O(1) auxiliary space
AVL Tree Height-balanced BST
Red-Black Tree Colour-balanced BST
Segment Tree Range queries
Fenwick Tree Prefix sums
B-Tree Multiway search tree
B+ Tree Records at leaf level
Trie Prefix-based string storage
Huffman Tree Lossless compression
