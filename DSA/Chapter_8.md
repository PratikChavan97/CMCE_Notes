# CHAPTER 8 – GRAPHS AND GRAPH ALGORITHMS WITH JAVA

# PART 1 – GRAPH FUNDAMENTALS AND REPRESENTATIONS

1. INTRODUCTION TO GRAPHS
   A Graph is a non-linear data structure consisting of a collection of vertices (nodes) and edges (connections between vertices).

Graphs are used to represent relationships and connections between different entities.

Unlike arrays, linked lists, stacks, queues and trees, graphs can represent complex relationships where one node can connect to multiple other nodes in different ways.

Basic Graph Structure
Consider the following graph:

````text
A -------- B
|          |
|          |
C -------- D
Here:

A, B, C and D are vertices.
The connecting lines are edges.
Vertices:

text
V = {A, B, C, D}
Edges:

text
E = {(A,B), (A,C), (B,D), (C,D)}
A graph is mathematically represented as:

text
G = (V, E)
Where:

G = Graph
V = Set of vertices
E = Set of edges
For the above example:

text
V = {A, B, C, D}

E = {
(A,B),
(A,C),
(B,D),
(C,D)
}
Important Terminology
Term Meaning
Vertex A node in a graph
Edge Connection between two vertices
Adjacent Vertices Vertices directly connected by an edge
Degree Number of edges incident on a vertex in an undirected graph
Path Sequence of vertices connected by edges
Cycle Path that starts and ends at the same vertex
Connected Graph Every pair of vertices has a connecting path
Component Maximal connected part of an undirected graph
Loop Edge connecting a vertex to itself
Parallel Edges Multiple edges connecting the same pair of vertices

2. REAL-WORLD APPLICATIONS OF GRAPHS
Graphs are widely used in computer science and real-world applications.

2.1 Social Networks
Consider users in a social networking application.

```text
       Rahul -------- Amit
         |             |
         |             |
       Priya -------- Neha
Each user represents a vertex.

A friendship represents an edge.

Applications:

Facebook friendships.
LinkedIn connections.
Instagram social relationships.
Friend recommendations.
Finding mutual friends.
2.2 Google Maps and Navigation
Locations represent vertices.

Roads represent edges.

```text
       Pune -------- Mumbai
         |              |
         |              |
      Nashik -------- Surat
Applications:

Finding shortest routes.
Finding alternative routes.
Calculating travel distances.
Navigation systems.
2.3 Computer Networks
Computers and routers represent vertices.

Network connections represent edges.

```text
        Router A
        /      \
    Router B  Router C
        \      /
        Router D
Applications:

Network routing.
Packet forwarding.
Network topology analysis.
Connectivity checking.
2.4 Web Pages and Search Engines
Each webpage can be represented as a vertex.

A hyperlink from one webpage to another represents a directed edge.

```text
Page A -----> Page B
  |              |
  v              v
Page C -----> Page D
Search engines use graph-based techniques to analyse webpage relationships.

2.5 Other Applications
Airline route networks.
Railway networks.
Recommendation systems.
Dependency management.
Project scheduling.
Compiler dependency graphs.
Knowledge graphs.
Electrical circuits.
````

3. TYPES OF GRAPHS
   Graphs can be classified according to their edge direction, weight, connectivity and structure.

3.1 Undirected Graph
An Undirected Graph is a graph in which edges do not have any direction.

If A is connected to B, then B is also connected to A.

Example:

text
A -------- B
| |
| |
C -------- D
Edges:

text
(A,B)
(A,C)
(B,D)
(C,D)
An edge between A and B represents the same connection as an edge between B and A.

Applications:

Friendship networks.
Two-way roads.
Computer connections.
Java representation:

java
graph.get(0).add(1);
graph.get(1).add(0);
Both directions are stored.

3.2 Directed Graph (Digraph)
A Directed Graph is a graph in which every edge has a specific direction.

An edge from A to B does not automatically mean that B connects to A.

Example:

text
A ------> B
|
|
v
C ------> D
Edges:

text
A -> B
A -> C
C -> D
Applications:

Twitter/X follower relationships.
Webpage hyperlinks.
One-way roads.
Task dependencies.
Course prerequisites.
Java representation:

java
graph.get(0).add(1);
This creates an edge from vertex 0 to vertex 1.

It does not automatically create the reverse edge.

3.3 Weighted Graph
A Weighted Graph is a graph in which each edge has an associated numerical value called weight.

Weights may represent:

Distance.
Cost.
Time.
Network latency.
Fuel consumption.
Example:

text
A ----10---- B
| |
5 15
| |
C ----20---- D
Edges:

text
A-B = 10
A-C = 5
B-D = 15
C-D = 20
Applications:

Finding minimum-cost routes.
Network optimization.
Shortest-path algorithms.
Transportation planning.
3.4 Unweighted Graph
An Unweighted Graph does not assign numerical weights to its edges.

All edges are treated equally for basic shortest-path calculations.

Example:

text
A -------- B
| |
| |
C -------- D
The shortest path is measured by the number of edges.

3.5 Connected Graph
An undirected graph is connected if every vertex is reachable from every other vertex.

Example:

text
A -------- B
| |
| |
C -------- D
All vertices are connected.

Therefore, the graph is connected.

3.6 Disconnected Graph
A graph is disconnected if at least one vertex cannot be reached from another vertex.

Example:

text
A -------- B

       C -------- D

       E

This graph contains three connected components:

text
Component 1 = {A, B}

Component 2 = {C, D}

Component 3 = {E}
3.7 Cyclic Graph
A graph is cyclic if it contains at least one cycle.

Example:

text
A -------- B
\ |
\ |
\ |
C ----
Cycle:

text
A -> B -> C -> A
3.8 Acyclic Graph
A graph is acyclic if it contains no cycles.

Example:

text
A
/ \
 B C
/ \
 D E
An undirected acyclic graph is called a forest if it is not necessarily connected.

A connected, undirected, acyclic graph is called a Tree.

A Directed Acyclic Graph is commonly known as a DAG.

DAGs are important in:

Task scheduling.
Course prerequisites.
Dependency resolution.
Build systems.
3.9 Complete Graph
A Complete Graph is a simple undirected graph in which every vertex is directly connected to every other vertex.

Example:

text
A
/|\
 / | \
 B--|--C
\ | /
\|/
D
For a complete undirected graph with n vertices:

text
Number of edges = n(n - 1) / 2
Example:

For 4 vertices:

text
E = 4(4 - 1) / 2

E = 4(3) / 2

E = 6
Therefore, a complete graph with four vertices has six edges.

3.10 Sparse Graph
A Sparse Graph contains relatively few edges compared to the maximum possible number of edges.

Example:

text
A -------- B

C -------- D
Sparse graphs are commonly represented using Adjacency Lists.

3.11 Dense Graph
A Dense Graph contains a large number of edges compared to the maximum possible number of edges.

A complete graph is an example of a dense graph.

Adjacency Matrices can be useful for dense graphs.

4. DEGREE OF A VERTEX
   Degree represents the number of edges connected to a vertex in an undirected graph.

Consider:

text
A
/|\
 / | \
 B C D
|
E
Degrees:

text
Degree(A) = 3

Degree(B) = 1

Degree(C) = 2

Degree(D) = 1

Degree(E) = 1
Handshaking Lemma
In an undirected graph, the sum of the degrees of all vertices equals twice the number of edges.

Formula:

text
Sum of Degrees = 2 × Number of Edges
Mathematically:

text
Σ degree(v) = 2|E|
Example:

text
Degrees = 3 + 1 + 2 + 1 + 1

Sum = 8
Number of edges = 4.

text
2 × 4 = 8
Therefore, the handshaking lemma is satisfied.

Important exam point:

The sum of degrees of all vertices in an undirected graph is always even.

4.1 In-degree
In a Directed Graph, In-degree is the number of incoming edges to a vertex.

Example:

text
A -----> C

B -----> C

D -----> C
In-degree of C = 3.

4.2 Out-degree
Out-degree is the number of outgoing edges from a vertex.

Example:

text
A
/|\
 v v v
B C D
Out-degree of A = 3.

Important Formula for Directed Graphs

text
Sum of In-degrees = Number of Edges

Sum of Out-degrees = Number of Edges
Unlike an undirected graph, the degree sum is not twice the number of edges when considering only in-degree or only out-degree.

5. PATH, WALK, TRAIL AND CYCLE
   These terms are frequently confused in examinations.

5.1 Walk
A Walk is a sequence of vertices in which consecutive vertices are connected by edges.

Vertices and edges may repeat.

Example:

text
A -> B -> C -> B -> D
This is a walk.

5.2 Trail
A Trail is a walk in which no edge is repeated.

Vertices may repeat.

Example:

text
A -> B -> C -> A -> D
Assuming all traversed edges are distinct, this is a trail.

5.3 Path
A Path is a sequence of vertices connected by edges in which vertices do not repeat.

Example:

text
A -> B -> C -> D
No vertex is repeated.

5.4 Cycle
A Cycle is a closed path that starts and ends at the same vertex, with no other repeated vertices in a simple cycle.

Example:

text
A -> B -> C -> A
Comparison

Property Walk Trail Simple Path Simple Cycle
Repeated vertices Allowed Allowed Not allowed Only start/end repeats
Repeated edges Allowed Not allowed Not allowed Not allowed
Starts and ends same Possible Possible No, by standard simple-path definition Yes 6. GRAPH REPRESENTATIONS
A graph can be stored in memory using several different representations.

The most important representations are:

Adjacency Matrix.
Adjacency List.
Edge List.
Incidence Matrix.
We will implement all four using Java.

7. ADJACENCY MATRIX
   An Adjacency Matrix is a two-dimensional array used to represent connections between vertices.

If there is an edge between two vertices, the corresponding matrix position contains 1.

Otherwise, it contains 0.

Consider:

text
0 -------- 1
| |
| |
2 -------- 3
Edges:

text
0-1
0-2
1-3
2-3
Adjacency Matrix:

text
0 1 2 3

0 0 1 1 0

1 1 0 0 1

2 1 0 0 1

3 0 1 1 0
Explanation
The value at matrix[i][j] represents whether vertex i is connected to vertex j.

For example:

java
matrix[0][1] = 1;
This means vertex 0 is connected to vertex 1.

Since this is an undirected graph:

java
matrix[1][0] = 1;
must also be stored.

Java Implementation

java
public class AdjacencyMatrix {

    public static void main(String[] args) {

        int vertices = 4;

        int[][] graph = new int[vertices][vertices];

        // Adding undirected edges

        graph[0][1] = 1;
        graph[1][0] = 1;

        graph[0][2] = 1;
        graph[2][0] = 1;

        graph[1][3] = 1;
        graph[3][1] = 1;

        graph[2][3] = 1;
        graph[3][2] = 1;

        // Printing matrix

        for (int i = 0; i < vertices; i++) {

            for (int j = 0; j < vertices; j++) {

                System.out.print(graph[i][j] + " ");
            }

            System.out.println();
        }
    }

}
Output

text
0 1 1 0
1 0 0 1
1 0 0 1
0 1 1 0
Adding an Edge
For an undirected graph:

java
graph[u][v] = 1;

graph[v][u] = 1;
For a directed graph:

java
graph[u][v] = 1;
Only one direction is stored.

Removing an Edge
For an undirected graph:

java
graph[u][v] = 0;

graph[v][u] = 0;
For a directed graph:

java
graph[u][v] = 0;
Advantages
Easy to implement.
Checking whether an edge exists takes O(1).
Suitable for dense graphs.
Simple mathematical representation.
Disadvantages
Requires O(V²) memory.
Wastes memory for sparse graphs.
Iterating over all neighbours requires O(V) time. 8. ADJACENCY LIST
An Adjacency List stores the neighbours of each vertex.

Instead of maintaining a complete V × V matrix, we maintain a list of connected vertices for every vertex.

Consider:

text
0 -------- 1
| |
| |
2 -------- 3
Adjacency List:

text
0 -> 1, 2

1 -> 0, 3

2 -> 0, 3

3 -> 1, 2
Java Implementation Using ArrayList

java
import java.util.\*;

public class AdjacencyList {

    public static void main(String[] args) {

        int vertices = 4;

        ArrayList<ArrayList<Integer>> graph =
            new ArrayList<>();

        // Create empty lists

        for (int i = 0; i < vertices; i++) {

            graph.add(new ArrayList<>());
        }

        // Add undirected edges

        graph.get(0).add(1);
        graph.get(1).add(0);

        graph.get(0).add(2);
        graph.get(2).add(0);

        graph.get(1).add(3);
        graph.get(3).add(1);

        graph.get(2).add(3);
        graph.get(3).add(2);

        // Display graph

        for (int i = 0; i < vertices; i++) {

            System.out.print(i + " -> ");

            for (int neighbour : graph.get(i)) {

                System.out.print(neighbour + " ");
            }

            System.out.println();
        }
    }

}
Output

text
0 -> 1 2
1 -> 0 3
2 -> 0 3
3 -> 1 2
Understanding the Code

java
ArrayList<ArrayList<Integer>> graph =
new ArrayList<>();
This creates an outer ArrayList.

Each element of the outer list stores another ArrayList.

For example:

java
graph.get(0)
returns the neighbour list of vertex 0.

Adding an edge:

java
graph.get(0).add(1);
means vertex 0 is connected to vertex 1.

For an undirected graph, add the reverse connection too.

java
graph.get(1).add(0);
Advantages
Memory efficient for sparse graphs.
Easy to iterate through neighbours.
Easy to add edges.
Commonly used in graph algorithms.
Disadvantages
Checking whether a particular edge exists can take O(degree of vertex).
Slightly more complex than an adjacency matrix. 9. ADJACENCY LIST USING HASHMAP
When graph vertices are strings or other objects rather than integer indices, a HashMap can be convenient.

Example:

text
Pune -> Mumbai, Nashik

Mumbai -> Pune, Surat

Nashik -> Pune
Java Code

java
import java.util.\*;

public class GraphHashMap {

    public static void main(String[] args) {

        Map<String, List<String>> graph =
            new HashMap<>();

        graph.put("Pune", new ArrayList<>());
        graph.put("Mumbai", new ArrayList<>());
        graph.put("Nashik", new ArrayList<>());
        graph.put("Surat", new ArrayList<>());

        // Undirected edges

        graph.get("Pune").add("Mumbai");
        graph.get("Mumbai").add("Pune");

        graph.get("Pune").add("Nashik");
        graph.get("Nashik").add("Pune");

        graph.get("Mumbai").add("Surat");
        graph.get("Surat").add("Mumbai");

        for (String city : graph.keySet()) {

            System.out.println(
                city + " -> " + graph.get(city)
            );
        }
    }

}
Possible Output

text
Pune -> [Mumbai, Nashik]
Mumbai -> [Pune, Surat]
Nashik -> [Pune]
Surat -> [Mumbai]
Note: HashMap does not guarantee iteration order.

10. EDGE LIST REPRESENTATION
    An Edge List stores all graph edges in a collection.

Consider:

text
0 -------- 1
| |
| |
2 -------- 3
Edge List:

text
(0,1)

(0,2)

(1,3)

(2,3)
Java Implementation

java
import java.util.\*;

public class EdgeList {

    public static void main(String[] args) {

        int[][] edges = {

            {0, 1},

            {0, 2},

            {1, 3},

            {2, 3}
        };

        System.out.println("Graph Edges:");

        for (int[] edge : edges) {

            System.out.println(
                edge[0] + " - " + edge[1]
            );
        }
    }

}
Output

text
Graph Edges:
0 - 1
0 - 2
1 - 3
2 - 3
Advantages
Simple representation.
Memory efficient: O(E).
Useful in algorithms that process edges directly.
Particularly useful for Kruskal's Minimum Spanning Tree algorithm.
Disadvantages
Finding all neighbours of a vertex requires scanning the edge list.
Checking whether an edge exists takes O(E) in a basic implementation. 11. INCIDENCE MATRIX
An Incidence Matrix represents the relationship between vertices and edges.

Rows represent vertices.

Columns represent edges.

Consider:

text
0 -------- 1
| |
| |
2 -------- 3
Edges:

text
e0 = (0,1)

e1 = (0,2)

e2 = (1,3)

e3 = (2,3)
Incidence Matrix:

text
e0 e1 e2 e3

0 1 1 0 0

1 1 0 1 0

2 0 1 0 1

3 0 0 1 1
A value of 1 indicates that the vertex is incident to that edge.

For a simple undirected graph, each edge column contains two 1s.

Java Code

java
public class IncidenceMatrix {

    public static void main(String[] args) {

        int vertices = 4;

        int edges = 4;

        int[][] matrix = new int[vertices][edges];

        // Edge 0: 0-1

        matrix[0][0] = 1;
        matrix[1][0] = 1;

        // Edge 1: 0-2

        matrix[0][1] = 1;
        matrix[2][1] = 1;

        // Edge 2: 1-3

        matrix[1][2] = 1;
        matrix[3][2] = 1;

        // Edge 3: 2-3

        matrix[2][3] = 1;
        matrix[3][3] = 1;

        for (int i = 0; i < vertices; i++) {

            for (int j = 0; j < edges; j++) {

                System.out.print(matrix[i][j] + " ");
            }

            System.out.println();
        }
    }

}
Output

text
1 1 0 0
1 0 1 0
0 1 0 1
0 0 1 1 12. WEIGHTED GRAPH IMPLEMENTATION IN JAVA
In a weighted graph, each edge contains a weight.

We can represent an edge using a class.

Edge Class

java
class Edge {

    int destination;

    int weight;

    Edge(int destination, int weight) {

        this.destination = destination;

        this.weight = weight;
    }

}
Complete Weighted Adjacency List

java
import java.util.\*;

public class WeightedGraph {

    public static void main(String[] args) {

        int vertices = 4;

        ArrayList<ArrayList<Edge>> graph =
            new ArrayList<>();

        for (int i = 0; i < vertices; i++) {

            graph.add(new ArrayList<>());
        }

        // Undirected weighted edges

        graph.get(0).add(new Edge(1, 10));
        graph.get(1).add(new Edge(0, 10));

        graph.get(0).add(new Edge(2, 5));
        graph.get(2).add(new Edge(0, 5));

        graph.get(1).add(new Edge(3, 15));
        graph.get(3).add(new Edge(1, 15));

        graph.get(2).add(new Edge(3, 20));
        graph.get(3).add(new Edge(2, 20));

        // Display graph

        for (int i = 0; i < vertices; i++) {

            System.out.print(i + " -> ");

            for (Edge edge : graph.get(i)) {

                System.out.print(
                    "(" + edge.destination +
                    ", weight=" + edge.weight + ") "
                );
            }

            System.out.println();
        }
    }

}
Output

text
0 -> (1, weight=10) (2, weight=5)

1 -> (0, weight=10) (3, weight=15)

2 -> (0, weight=5) (3, weight=20)

3 -> (1, weight=15) (2, weight=20)
Explanation

java
new Edge(1, 10)
means:

Destination vertex = 1.
Edge weight = 10.
Weighted adjacency lists are particularly important for Dijkstra's Algorithm and Minimum Spanning Tree algorithms.

13. GRAPH REPRESENTATION COMPARISON

Feature Adjacency Matrix Adjacency List Edge List Incidence Matrix
Storage V × V V + E E V × E
Edge lookup O(1) O(degree) O(E) O(E)
Neighbour iteration O(V) O(degree) O(E) O(E)
Suitable for Dense graphs Sparse graphs Edge-based algorithms Mathematical representation
Memory O(V²) O(V + E) O(E) O(VE)
Note: These complexities assume standard implementations and exclude special indexing or auxiliary data structures.

Which Representation Should We Use?
Adjacency Matrix: When the graph is dense or constant-time edge lookup is important.
Adjacency List: For most graph traversal and shortest-path problems.
Edge List: When algorithms primarily process edges.
Incidence Matrix: For mathematical graph representation and certain specialised applications. 14. IMPORTANT GRAPH FORMULAS
Undirected Simple Graph
Maximum number of edges:

text
E = V(V - 1) / 2
Directed Simple Graph Without Self-Loops
Maximum number of edges:

text
E = V(V - 1)
Complete Undirected Graph

text
Degree of every vertex = V - 1
Handshaking Lemma

text
Σ degree(v) = 2E
Directed Graph

text
Σ In-degree = E

Σ Out-degree = E
Tree
For a tree with V vertices:

text
E = V - 1
A tree is connected and acyclic.

Complete Graph Example
For 5 vertices:

text
E = 5(5 - 1) / 2

E = 5 × 4 / 2

E = 10
Therefore, a complete undirected graph with five vertices contains ten edges.

18. PART 1 – QUICK REVISION

Concept Important Point
Graph G = (V, E)
Vertex Node
Edge Connection
Undirected Graph No edge direction
Directed Graph Edges have direction
Weighted Graph Edges have weights
Unweighted Graph No explicit weights
Connected Graph All vertices mutually reachable
Disconnected Graph Multiple components
Cyclic Graph Contains a cycle
Acyclic Graph Contains no cycle
DAG Directed Acyclic Graph
Complete Graph Every vertex pair connected
Sparse Graph Relatively few edges
Dense Graph Relatively many edges
Adjacency Matrix O(V²) storage
Adjacency List O(V + E) storage
Edge List Stores edge pairs
Incidence Matrix Vertex-edge relationship
Handshaking Lemma Sum of degrees = 2E
Tree Connected undirected acyclic graph

---

# PART 2 – GRAPH TRAVERSAL ALGORITHMS: BFS AND DFS

1. INTRODUCTION TO GRAPH TRAVERSAL
   Graph Traversal is the process of visiting the vertices of a graph systematically.

Unlike arrays and linked lists, graphs can contain multiple connections and cycles.

Therefore, while traversing a graph, we must ensure that we do not repeatedly visit the same vertices.

For this purpose, we use a Boolean array called visited.

java
boolean[] visited = new boolean[V];
Initially, every vertex is unvisited.

Whenever a vertex is visited, we mark it:

java
visited[vertex] = true;
There are two fundamental graph traversal algorithms:

Breadth First Search (BFS).
Depth First Search (DFS).
Example Graph
Consider the following undirected graph:

````text
          0
         / \
        1   2
       / \   \
      3   4   5
Edges:


```text
0 - 1
0 - 2
1 - 3
1 - 4
2 - 5
Starting from vertex 0, both BFS and DFS can visit all six vertices, but their traversal orders differ.

BFS explores neighbouring vertices level by level.

DFS explores one branch as deeply as possible before backtracking.

2. BREADTH FIRST SEARCH (BFS)
2.1 What is BFS?
Breadth First Search is a graph traversal algorithm that explores vertices level by level.

It first visits the starting vertex, then all its immediate neighbours, followed by the neighbours of those vertices.

BFS uses a Queue data structure.

Remember:

BFS = Breadth First Search + Queue

A Queue follows FIFO:

First In, First Out.

The vertex inserted first is processed first.

2.2 BFS Working Example
Consider:


```text
          0
         / \
        1   2
       / \   \
      3   4   5
Start BFS from vertex 0.

Step 1: Visit vertex 0
Queue:


```text
[0]
Visited:


```text
0
Step 2: Remove 0 from Queue
Its neighbours are 1 and 2.

Insert them.

Queue:


```text
[1, 2]
Visited:


```text
0, 1, 2
Step 3: Remove 1
Its unvisited neighbours are 3 and 4.

Insert them.

Queue:


```text
[2, 3, 4]
Visited:


```text
0, 1, 2, 3, 4
Step 4: Remove 2
Its unvisited neighbour is 5.

Insert it.

Queue:


```text
[3, 4, 5]
Step 5: Remove 3
No unvisited neighbours.

Queue:


```text
[4, 5]
Step 6: Remove 4
No unvisited neighbours.

Queue:


```text
[5]
Step 7: Remove 5
No unvisited neighbours.

Queue:


```text
[]
BFS traversal:


```text
0 1 2 3 4 5
2.3 BFS Algorithm

```text
BFS(graph, start):

    Create visited array

    Create Queue

    Mark start as visited

    Insert start into Queue

    While Queue is not empty:

        Remove front vertex

        Print vertex

        For every neighbour:

            If neighbour is not visited:

                Mark neighbour as visited

                Insert neighbour into Queue
2.4 Java Implementation of BFS Using Adjacency List

java
import java.util.*;

public class BFSGraph {

    static void bfs(
        ArrayList<ArrayList<Integer>> graph,
        int start,
        int vertices
    ) {

        boolean[] visited = new boolean[vertices];

        Queue<Integer> queue = new LinkedList<>();

        // Mark starting vertex visited
        visited[start] = true;

        // Insert starting vertex
        queue.add(start);

        while (!queue.isEmpty()) {

            int current = queue.poll();

            System.out.print(current + " ");

            // Visit neighbours
            for (int neighbour : graph.get(current)) {

                if (!visited[neighbour]) {

                    visited[neighbour] = true;

                    queue.add(neighbour);
                }
            }
        }
    }

    public static void main(String[] args) {

        int vertices = 6;

        ArrayList<ArrayList<Integer>> graph =
            new ArrayList<>();

        for (int i = 0; i < vertices; i++) {

            graph.add(new ArrayList<>());
        }

        // Undirected edges

        graph.get(0).add(1);
        graph.get(1).add(0);

        graph.get(0).add(2);
        graph.get(2).add(0);

        graph.get(1).add(3);
        graph.get(3).add(1);

        graph.get(1).add(4);
        graph.get(4).add(1);

        graph.get(2).add(5);
        graph.get(5).add(2);

        System.out.println("BFS Traversal:");

        bfs(graph, 0, vertices);
    }
}
Output

```text
BFS Traversal:
0 1 2 3 4 5
2.5 Understanding Important Java Statements
Creating Visited Array

java
boolean[] visited = new boolean[vertices];
Initially:


```text
visited = [false, false, false, false, false, false]
Creating Queue

java
Queue<Integer> queue = new LinkedList<>();
Java's Queue interface can be implemented using LinkedList or ArrayDeque.

Adding Starting Vertex

java
visited[start] = true;

queue.add(start);
We mark the vertex visited before inserting it.

This prevents duplicate insertions when multiple vertices connect to the same neighbour.

Removing an Element

java
int current = queue.poll();
poll() removes and returns the front element.

It returns null if the queue is empty.

Processing Neighbours

java
for (int neighbour : graph.get(current))
This loops through every adjacent vertex.

Checking Visited Status

java
if (!visited[neighbour])
If the neighbour is not visited, we mark and enqueue it.

3. BFS DRY RUN
Consider the same graph:


```text
          0
         / \
        1   2
       / \   \
      3   4   5
Starting vertex = 0.


Step	Current Vertex	Queue After Processing	Visited Order
1	0	1, 2	0
2	1	2, 3, 4	0, 1
3	2	3, 4, 5	0, 1, 2
4	3	4, 5	0, 1, 2, 3
5	4	5	0, 1, 2, 3, 4
6	5	Empty	0, 1, 2, 3, 4, 5
Final traversal:


```text
0 1 2 3 4 5
Note: BFS traversal order can change depending on the order in which neighbours are stored in the adjacency list.

4. BFS USING ADJACENCY MATRIX
BFS can also be implemented using an Adjacency Matrix.

In this implementation, instead of iterating through an adjacency list, we check every possible vertex to determine whether an edge exists.

Java Code

java
import java.util.*;

public class BFSMatrix {

    static void bfs(int[][] graph, int start) {

        int vertices = graph.length;

        boolean[] visited = new boolean[vertices];

        Queue<Integer> queue = new LinkedList<>();

        visited[start] = true;

        queue.add(start);

        while (!queue.isEmpty()) {

            int current = queue.poll();

            System.out.print(current + " ");

            for (int i = 0; i < vertices; i++) {

                if (graph[current][i] == 1 &&
                    !visited[i]) {

                    visited[i] = true;

                    queue.add(i);
                }
            }
        }
    }

    public static void main(String[] args) {

        int[][] graph = {

            {0, 1, 1, 0, 0, 0},

            {1, 0, 0, 1, 1, 0},

            {1, 0, 0, 0, 0, 1},

            {0, 1, 0, 0, 0, 0},

            {0, 1, 0, 0, 0, 0},

            {0, 0, 1, 0, 0, 0}
        };

        bfs(graph, 0);
    }
}
Output

```text
0 1 2 3 4 5
Complexity
For an adjacency matrix:

Time Complexity: O(V²).
Space Complexity: O(V) auxiliary space, excluding the matrix.
5. BFS TIME AND SPACE COMPLEXITY
For an adjacency list:


Operation	Complexity
Visiting vertices	O(V)
Processing edges	O(E)
Total Time	O(V + E)
Visited Array	O(V)
Queue	O(V)
Total Auxiliary Space	O(V)
Therefore:


```text
BFS Time Complexity = O(V + E)

BFS Auxiliary Space = O(V)
For an adjacency matrix:


```text
Time Complexity = O(V²)

Auxiliary Space = O(V)
V = Number of vertices.

E = Number of edges.

6. APPLICATIONS OF BFS
BFS is used in many real-world applications.

6.1 Shortest Path in an Unweighted Graph
BFS finds the shortest path measured by the number of edges in an unweighted graph.

Example:


```text
A ---- B ---- D
 \          /
  \        /
     C
From A to D:

Possible paths:


```text
A -> B -> D

A -> C -> D
Both have two edges.

BFS discovers a shortest path.

Important: BFS does not guarantee minimum total weight in a weighted graph.

6.2 Level Order Traversal of Trees
BFS is used for Level Order Traversal.

Example:


```text
        1
       / \
      2   3
     / \
    4   5
Output:


```text
1 2 3 4 5
6.3 Social Networking
Finding people within two or three connection levels.

6.4 Web Crawling
Exploring linked webpages level by level.

6.5 Network Broadcasting
Sending information to neighbouring network devices systematically.

6.6 Minimum Number of Moves
BFS can solve problems such as finding the minimum moves required to reach a destination in an unweighted state space.

Examples include chessboard movement and maze traversal.

7. DEPTH FIRST SEARCH (DFS)
7.1 What is DFS?
Depth First Search is a graph traversal algorithm that explores one branch as deeply as possible before backtracking.

DFS uses:

Recursion (implicit call stack).
Explicit Stack (iterative implementation).
Remember:

DFS = Depth First Search + Stack

7.2 DFS Working Example
Consider:


```text
          0
         / \
        1   2
       / \   \
      3   4   5
Start from vertex 0.

Assuming neighbours are processed in ascending order:

Step 1
Visit 0.

Move to 1.

Step 2
Visit 1.

Move to 3.

Step 3
Visit 3.

No unvisited neighbours.

Backtrack to 1.

Step 4
Visit 4.

No unvisited neighbours.

Backtrack to 0.

Step 5
Visit 2.

Move to 5.

Step 6
Visit 5.

Traversal completed.

DFS traversal:


```text
0 1 3 4 2 5
8. DFS USING RECURSION
Recursive DFS is one of the most common graph traversal implementations in coding interviews.

It uses the Java method call stack automatically.

Algorithm

```text
DFS(graph, vertex):

    Mark vertex as visited

    Print vertex

    For each neighbour:

        If neighbour is not visited:

            DFS(graph, neighbour)
Java Implementation

java
import java.util.*;

public class DFSRecursive {

    static void dfs(
        ArrayList<ArrayList<Integer>> graph,
        int vertex,
        boolean[] visited
    ) {

        // Mark current vertex visited

        visited[vertex] = true;

        System.out.print(vertex + " ");

        // Explore neighbours

        for (int neighbour : graph.get(vertex)) {

            if (!visited[neighbour]) {

                dfs(graph, neighbour, visited);
            }
        }
    }

    public static void main(String[] args) {

        int vertices = 6;

        ArrayList<ArrayList<Integer>> graph =
            new ArrayList<>();

        for (int i = 0; i < vertices; i++) {

            graph.add(new ArrayList<>());
        }

        // Undirected edges

        graph.get(0).add(1);
        graph.get(1).add(0);

        graph.get(0).add(2);
        graph.get(2).add(0);

        graph.get(1).add(3);
        graph.get(3).add(1);

        graph.get(1).add(4);
        graph.get(4).add(1);

        graph.get(2).add(5);
        graph.get(5).add(2);

        boolean[] visited = new boolean[vertices];

        System.out.println("DFS Traversal:");

        dfs(graph, 0, visited);
    }
}
Output

```text
DFS Traversal:
0 1 3 4 2 5
Understanding the Recursive Code

java
visited[vertex] = true;
Marks the current vertex visited.


java
System.out.print(vertex + " ");
Prints the current vertex.


java
for (int neighbour : graph.get(vertex))
Iterates through the neighbours.


java
dfs(graph, neighbour, visited);
Calls DFS recursively for every unvisited neighbour.

The recursion continues until no unvisited neighbours remain.

Then Java automatically returns to the previous function call.

This process is called backtracking.

9. DFS RECURSION STACK DRY RUN
Consider:


```text
          0
         / \
        1   2
       / \
      3   4
Traversal order:


```text
0 -> 1 -> 3
At vertex 3, no unvisited neighbour exists.

The recursion backtracks to 1.

Then it visits 4.

After completing vertex 1, it backtracks to 0.

Finally, it visits 2.

Recursive Call Stack

Operation	Call Stack
Start	0
Visit 1	0, 1
Visit 3	0, 1, 3
Backtrack	0, 1
Visit 4	0, 1, 4
Backtrack	0
Visit 2	0, 2
Complete	Empty
Final DFS:


```text
0 1 3 4 2
10. DFS USING ITERATIVE STACK
DFS can also be implemented without recursion.

We explicitly create a Stack.

Java provides the Deque interface, which is generally preferred over the legacy Stack class for stack operations.

Java Implementation

java
import java.util.*;

public class DFSIterative {

    static void dfs(
        ArrayList<ArrayList<Integer>> graph,
        int start,
        int vertices
    ) {

        boolean[] visited = new boolean[vertices];

        Deque<Integer> stack = new ArrayDeque<>();

        stack.push(start);

        while (!stack.isEmpty()) {

            int current = stack.pop();

            if (!visited[current]) {

                visited[current] = true;

                System.out.print(current + " ");

                // Push neighbours

                for (int neighbour : graph.get(current)) {

                    if (!visited[neighbour]) {

                        stack.push(neighbour);
                    }
                }
            }
        }
    }

    public static void main(String[] args) {

        int vertices = 6;

        ArrayList<ArrayList<Integer>> graph =
            new ArrayList<>();

        for (int i = 0; i < vertices; i++) {

            graph.add(new ArrayList<>());
        }

        graph.get(0).add(1);
        graph.get(1).add(0);

        graph.get(0).add(2);
        graph.get(2).add(0);

        graph.get(1).add(3);
        graph.get(3).add(1);

        graph.get(1).add(4);
        graph.get(4).add(1);

        graph.get(2).add(5);
        graph.get(5).add(2);

        dfs(graph, 0, vertices);
    }
}
Output

```text
0 2 5 1 4 3
Why is the output different from recursive DFS?

Because vertex 1 is pushed first and vertex 2 is pushed second.

Since Stack follows LIFO, vertex 2 is processed first.

If we want an iterative DFS order matching recursive DFS for ascending neighbours, we can push the neighbours in reverse order.

Important Difference

Recursive DFS	Iterative DFS
Uses function call stack	Uses explicit stack
Uses recursion	Uses loops
Easier to understand	Avoids deep recursion calls
May cause StackOverflowError for deep recursion	Avoids Java recursion-depth limitations
11. BFS VS DFS
This is one of the most important comparison questions in examinations and interviews.


Feature	BFS	DFS
Full Form	Breadth First Search	Depth First Search
Working	Level by level	Depth first
Data Structure	Queue	Stack
Implementation	Usually iterative	Recursive or iterative
Shortest path	Yes, in unweighted graphs	Not guaranteed
Memory	Can be high for wide graphs	Depends on depth
Backtracking	Not fundamental	Fundamental
Complexity	O(V + E)	O(V + E)
Applications	Shortest paths, levels	Cycle detection, components, topological sorting
Important Exam Points
BFS uses Queue.
DFS uses Stack.
Recursive DFS uses the call stack.
Both have O(V + E) time complexity with adjacency lists.
BFS finds shortest paths in unweighted graphs.
DFS does not necessarily find shortest paths.
Both require visited tracking to avoid infinite traversal in cyclic graphs.
12. TRAVERSING A DISCONNECTED GRAPH
So far, we started traversal from vertex 0 and assumed all vertices were reachable.

But consider:


```text
    0 ---- 1

    2 ---- 3

    4 ---- 5
There are three disconnected components.

If BFS starts from vertex 0, it visits only:


```text
0 1
Vertices 2, 3, 4 and 5 remain unvisited.

To traverse the entire graph, we need an outer loop that starts a traversal from every unvisited vertex.

BFS for Disconnected Graph

java
import java.util.*;

public class DisconnectedBFS {

    static void bfs(
        ArrayList<ArrayList<Integer>> graph,
        int start,
        boolean[] visited
    ) {

        Queue<Integer> queue = new LinkedList<>();

        visited[start] = true;

        queue.add(start);

        while (!queue.isEmpty()) {

            int current = queue.poll();

            System.out.print(current + " ");

            for (int neighbour : graph.get(current)) {

                if (!visited[neighbour]) {

                    visited[neighbour] = true;

                    queue.add(neighbour);
                }
            }
        }
    }

    public static void main(String[] args) {

        int vertices = 6;

        ArrayList<ArrayList<Integer>> graph =
            new ArrayList<>();

        for (int i = 0; i < vertices; i++) {

            graph.add(new ArrayList<>());
        }

        // Component 1

        graph.get(0).add(1);
        graph.get(1).add(0);

        // Component 2

        graph.get(2).add(3);
        graph.get(3).add(2);

        // Component 3

        graph.get(4).add(5);
        graph.get(5).add(4);

        boolean[] visited = new boolean[vertices];

        for (int i = 0; i < vertices; i++) {

            if (!visited[i]) {

                bfs(graph, i, visited);

                System.out.println();
            }
        }
    }
}
Output

```text
0 1
2 3
4 5
Each line represents a different connected component.

Time Complexity:


```text
O(V + E)
13. COUNT CONNECTED COMPONENTS
A connected component is a maximal group of vertices in which every vertex can reach every other vertex.

We can count connected components using DFS or BFS.

Java Implementation Using DFS

java
import java.util.*;

public class ConnectedComponents {

    static void dfs(
        ArrayList<ArrayList<Integer>> graph,
        int vertex,
        boolean[] visited
    ) {

        visited[vertex] = true;

        for (int neighbour : graph.get(vertex)) {

            if (!visited[neighbour]) {

                dfs(graph, neighbour, visited);
            }
        }
    }

    static int countComponents(
        ArrayList<ArrayList<Integer>> graph,
        int vertices
    ) {

        boolean[] visited = new boolean[vertices];

        int count = 0;

        for (int i = 0; i < vertices; i++) {

            if (!visited[i]) {

                count++;

                dfs(graph, i, visited);
            }
        }

        return count;
    }

    public static void main(String[] args) {

        int vertices = 7;

        ArrayList<ArrayList<Integer>> graph =
            new ArrayList<>();

        for (int i = 0; i < vertices; i++) {

            graph.add(new ArrayList<>());
        }

        graph.get(0).add(1);
        graph.get(1).add(0);

        graph.get(1).add(2);
        graph.get(2).add(1);

        graph.get(3).add(4);
        graph.get(4).add(3);

        // Vertex 5 and 6 are isolated.

        System.out.println(
            "Connected Components: " +
            countComponents(graph, vertices)
        );
    }
}
Output

```text
Connected Components: 4
Components:


```text
{0, 1, 2}

{3, 4}

{5}

{6}
Important: An isolated vertex is also a connected component.

14. CYCLE DETECTION IN UNDIRECTED GRAPHS
Cycle Detection is a very important graph problem.

A graph contains a cycle if we can start from a vertex, follow edges and return to the starting vertex without repeating other vertices in a simple cycle.

We can detect cycles using DFS.

Concept: Parent Tracking
In an undirected graph, when we visit a vertex, we encounter its parent again because edges are stored in both directions.

This should not be treated as a cycle.

Therefore, we track the parent vertex.

If we encounter an already visited neighbour that is not the parent, a cycle exists.

Example

```text
       0
      / \
     1---2
There is a cycle:


```text
0 -> 1 -> 2 -> 0
Java Implementation

java
import java.util.*;

public class CycleUndirected {

    static boolean dfs(
        ArrayList<ArrayList<Integer>> graph,
        int vertex,
        int parent,
        boolean[] visited
    ) {

        visited[vertex] = true;

        for (int neighbour : graph.get(vertex)) {

            if (!visited[neighbour]) {

                if (dfs(
                    graph,
                    neighbour,
                    vertex,
                    visited
                )) {

                    return true;
                }
            }

            else if (neighbour != parent) {

                return true;
            }
        }

        return false;
    }

    static boolean hasCycle(
        ArrayList<ArrayList<Integer>> graph,
        int vertices
    ) {

        boolean[] visited = new boolean[vertices];

        for (int i = 0; i < vertices; i++) {

            if (!visited[i]) {

                if (dfs(graph, i, -1, visited)) {

                    return true;
                }
            }
        }

        return false;
    }

    public static void main(String[] args) {

        int vertices = 3;

        ArrayList<ArrayList<Integer>> graph =
            new ArrayList<>();

        for (int i = 0; i < vertices; i++) {

            graph.add(new ArrayList<>());
        }

        graph.get(0).add(1);
        graph.get(1).add(0);

        graph.get(1).add(2);
        graph.get(2).add(1);

        graph.get(2).add(0);
        graph.get(0).add(2);

        System.out.println(
            hasCycle(graph, vertices)
        );
    }
}
Output

```text
true
Time Complexity:


```text
O(V + E)
15. CYCLE DETECTION IN DIRECTED GRAPHS
Cycle detection in directed graphs requires a different approach.

We use two arrays:

visited[] – Tracks whether a vertex has been visited.
recursionStack[] – Tracks vertices currently present in the active DFS recursion path.
If we encounter a vertex that is already in the current recursion stack, a directed cycle exists.

Example

```text
0 ----> 1
^       |
|       v
2 <-----
Cycle:


```text
0 -> 1 -> 2 -> 0
Java Implementation

java
import java.util.*;

public class DirectedCycle {

    static boolean dfs(
        ArrayList<ArrayList<Integer>> graph,
        int vertex,
        boolean[] visited,
        boolean[] recursionStack
    ) {

        visited[vertex] = true;

        recursionStack[vertex] = true;

        for (int neighbour : graph.get(vertex)) {

            if (!visited[neighbour]) {

                if (dfs(
                    graph,
                    neighbour,
                    visited,
                    recursionStack
                )) {

                    return true;
                }
            }

            else if (recursionStack[neighbour]) {

                return true;
            }
        }

        recursionStack[vertex] = false;

        return false;
    }

    static boolean hasCycle(
        ArrayList<ArrayList<Integer>> graph,
        int vertices
    ) {

        boolean[] visited = new boolean[vertices];

        boolean[] recursionStack =
            new boolean[vertices];

        for (int i = 0; i < vertices; i++) {

            if (!visited[i]) {

                if (dfs(
                    graph,
                    i,
                    visited,
                    recursionStack
                )) {

                    return true;
                }
            }
        }

        return false;
    }

    public static void main(String[] args) {

        int vertices = 3;

        ArrayList<ArrayList<Integer>> graph =
            new ArrayList<>();

        for (int i = 0; i < vertices; i++) {

            graph.add(new ArrayList<>());
        }

        graph.get(0).add(1);

        graph.get(1).add(2);

        graph.get(2).add(0);

        System.out.println(
            hasCycle(graph, vertices)
        );
    }
}
Output

```text
true
Important Difference

Undirected Cycle Detection	Directed Cycle Detection
Uses parent tracking	Uses recursion stack
Checks visited neighbour other than parent	Checks neighbour in active recursion stack
Edges work in both directions	Edges have directions
DFS complexity O(V + E)	DFS complexity O(V + E)
16. BIPARTITE GRAPH
A Bipartite Graph is a graph whose vertices can be divided into two disjoint sets such that no edge connects vertices belonging to the same set.

Example:


```text
Set A          Set B

  0 ----------- 1

  0 ----------- 3

  2 ----------- 1

  2 ----------- 3
The two sets are:


```text
Set A = {0, 2}

Set B = {1, 3}
There are no edges within either set.

Important Property
An undirected graph is bipartite if and only if it contains no odd-length cycle.

Examples:

A graph with a triangle is not bipartite.
A graph with a square cycle is bipartite.
A tree is always bipartite.
Checking Bipartite Graph Using BFS
We assign colours to vertices.


```text
Colour 0 = First Set

Colour 1 = Second Set
Whenever we visit a neighbour, it must receive the opposite colour.

If two adjacent vertices have the same colour, the graph is not bipartite.

Java Implementation

java
import java.util.*;

public class BipartiteGraph {

    static boolean isBipartite(
        ArrayList<ArrayList<Integer>> graph,
        int vertices
    ) {

        int[] color = new int[vertices];

        Arrays.fill(color, -1);

        Queue<Integer> queue = new LinkedList<>();

        for (int start = 0; start < vertices; start++) {

            if (color[start] != -1) {
                continue;
            }

            color[start] = 0;

            queue.add(start);

            while (!queue.isEmpty()) {

                int current = queue.poll();

                for (int neighbour : graph.get(current)) {

                    if (color[neighbour] == -1) {

                        color[neighbour] =
                            1 - color[current];

                        queue.add(neighbour);
                    }

                    else if (
                        color[neighbour] == color[current]
                    ) {

                        return false;
                    }
                }
            }
        }

        return true;
    }

    public static void main(String[] args) {

        int vertices = 4;

        ArrayList<ArrayList<Integer>> graph =
            new ArrayList<>();

        for (int i = 0; i < vertices; i++) {

            graph.add(new ArrayList<>());
        }

        graph.get(0).add(1);
        graph.get(1).add(0);

        graph.get(0).add(3);
        graph.get(3).add(0);

        graph.get(2).add(1);
        graph.get(1).add(2);

        graph.get(2).add(3);
        graph.get(3).add(2);

        System.out.println(
            isBipartite(graph, vertices)
        );
    }
}
Output

```text
true
Time Complexity:


```text
O(V + E)
17. FLOOD FILL ALGORITHM
Flood Fill is an important application of graph traversal.

It is commonly used in image editing applications.

When a user clicks on a region of an image, all connected pixels having the same original colour are replaced with a new colour.

Example:


```text
1 1 1
1 1 0
1 0 1
Starting at pixel (1, 1), change connected pixels with original colour 1 into colour 2.

Result:


```text
2 2 2
2 2 0
2 0 1
The bottom-right pixel remains unchanged because it is not connected to the selected region.

Java Implementation

java
import java.util.*;

public class FloodFill {

    static void dfs(
        int[][] image,
        int row,
        int col,
        int originalColor,
        int newColor
    ) {

        int rows = image.length;

        int cols = image[0].length;

        // Boundary check

        if (row < 0 || col < 0 ||
            row >= rows || col >= cols) {

            return;
        }

        // Check original color

        if (image[row][col] != originalColor) {

            return;
        }

        // Change color

        image[row][col] = newColor;

        // Four directions

        dfs(image, row - 1, col,
            originalColor, newColor);

        dfs(image, row + 1, col,
            originalColor, newColor);

        dfs(image, row, col - 1,
            originalColor, newColor);

        dfs(image, row, col + 1,
            originalColor, newColor);
    }

    public static void main(String[] args) {

        int[][] image = {

            {1, 1, 1},

            {1, 1, 0},

            {1, 0, 1}
        };

        int startRow = 1;

        int startCol = 1;

        int newColor = 2;

        int originalColor =
            image[startRow][startCol];

        if (originalColor != newColor) {

            dfs(
                image,
                startRow,
                startCol,
                originalColor,
                newColor
            );
        }

        for (int[] row : image) {

            System.out.println(Arrays.toString(row));
        }
    }
}
Output

```text
[2, 2, 2]
[2, 2, 0]
[2, 0, 1]
Time Complexity:


```text
O(R × C)
Where R = Number of rows and C = Number of columns.

18. NUMBER OF ISLANDS
This is one of the most frequently asked graph traversal problems in coding interviews.

An island is a group of connected land cells.

Consider:


```text
1 1 0 0
0 1 0 1
1 0 0 1
0 0 1 1
Here:

1 represents land.
0 represents water.
Using four-directional connectivity, the number of islands is 3.

We can solve this problem using DFS.

Java Implementation

java
public class NumberOfIslands {

    static void dfs(
        char[][] grid,
        int row,
        int col
    ) {

        int rows = grid.length;

        int cols = grid[0].length;

        if (row < 0 || col < 0 ||
            row >= rows || col >= cols ||
            grid[row][col] == '0') {

            return;
        }

        // Mark land as visited

        grid[row][col] = '0';

        dfs(grid, row - 1, col);

        dfs(grid, row + 1, col);

        dfs(grid, row, col - 1);

        dfs(grid, row, col + 1);
    }

    static int numIslands(char[][] grid) {

        int rows = grid.length;

        int cols = grid[0].length;

        int count = 0;

        for (int i = 0; i < rows; i++) {

            for (int j = 0; j < cols; j++) {

                if (grid[i][j] == '1') {

                    count++;

                    dfs(grid, i, j);
                }
            }
        }

        return count;
    }

    public static void main(String[] args) {

        char[][] grid = {

            {'1','1','0','0'},

            {'0','1','0','1'},

            {'1','0','0','1'},

            {'0','0','1','1'}
        };

        System.out.println(
            "Number of Islands: " +
            numIslands(grid)
        );
    }
}
Output

```text
Number of Islands: 3
Explanation
The algorithm scans every cell.

Whenever it encounters an unvisited land cell:

Increase the island count.
Start DFS.
Mark every connected land cell visited.
Continue scanning the grid.
Each island is counted exactly once.

Time Complexity:


```text
O(R × C)
Space Complexity:


```text
O(R × C)
In the worst case, recursive DFS may reach every cell.

Note: This implementation modifies the original grid.

19. FIND PATH BETWEEN TWO VERTICES
Another common interview problem is checking whether a path exists between two vertices.

We can use DFS.

Java Implementation

java
import java.util.*;

public class PathExists {

    static boolean hasPath(
        ArrayList<ArrayList<Integer>> graph,
        int source,
        int destination,
        boolean[] visited
    ) {

        if (source == destination) {

            return true;
        }

        visited[source] = true;

        for (int neighbour : graph.get(source)) {

            if (!visited[neighbour]) {

                if (hasPath(
                    graph,
                    neighbour,
                    destination,
                    visited
                )) {

                    return true;
                }
            }
        }

        return false;
    }

    public static void main(String[] args) {

        int vertices = 5;

        ArrayList<ArrayList<Integer>> graph =
            new ArrayList<>();

        for (int i = 0; i < vertices; i++) {

            graph.add(new ArrayList<>());
        }

        graph.get(0).add(1);
        graph.get(1).add(0);

        graph.get(1).add(2);
        graph.get(2).add(1);

        graph.get(3).add(4);
        graph.get(4).add(3);

        boolean[] visited = new boolean[vertices];

        System.out.println(
            hasPath(graph, 0, 2, visited)
        );

        System.out.println(
            hasPath(
                graph,
                0,
                4,
                new boolean[vertices]
            )
        );
    }
}
Output

```text
true
false
Explanation:

A path exists from 0 to 2.
No path exists from 0 to 4.
Time Complexity:


```text
O(V + E)
20. IMPORTANT GRAPH TRAVERSAL COMPLEXITIES

Algorithm	Time Complexity	Auxiliary Space
BFS – Adjacency List	O(V + E)	O(V)
DFS – Recursive	O(V + E)	O(V)
DFS – Iterative	O(V + E)	O(V)
BFS – Adjacency Matrix	O(V²)	O(V)
Connected Components	O(V + E)	O(V)
Undirected Cycle Detection	O(V + E)	O(V)
Directed Cycle Detection	O(V + E)	O(V)
Bipartite Checking	O(V + E)	O(V)
Flood Fill	O(R × C)	O(R × C)
Number of Islands	O(R × C)	O(R × C)
Path Existence using DFS	O(V + E)	O(V)
````

23. INTERVIEW CODING PRACTICE
    These are important problems to solve independently.

No. Problem Expected Technique
1 Implement BFS Queue
2 Implement DFS recursively Recursion
3 Implement DFS iteratively Stack
4 Count Connected Components DFS/BFS
5 Detect Cycle in Undirected Graph DFS + Parent
6 Detect Cycle in Directed Graph DFS + Recursion Stack
7 Check Bipartite Graph BFS + Colouring
8 Flood Fill DFS/BFS
9 Number of Islands DFS/BFS
10 Find Path Between Vertices DFS/BFS
11 Shortest Path in Unweighted Graph BFS
12 Clone Graph DFS/BFS + HashMap
Suggested Coding Exercises
Exercise 1
Write a Java program to perform BFS on a graph containing eight vertices.

Exercise 2
Write a Java program to perform DFS using an explicit Stack.

Exercise 3
Write a program to count the number of connected components.

Exercise 4
Determine whether an undirected graph contains a cycle.

Exercise 5
Determine whether a graph is bipartite.

Exercise 6
Solve the Number of Islands problem.

Exercise 7
Find the shortest path between two vertices in an unweighted graph and print the actual path, not just its length.

25. PART 2 – QUICK REVISION

Topic Important Point
BFS Breadth First Search
BFS Data Structure Queue
DFS Depth First Search
DFS Data Structure Stack
Recursive DFS Uses Call Stack
BFS Traversal Level by Level
DFS Traversal Depth First
BFS Complexity O(V + E)
DFS Complexity O(V + E)
BFS Shortest Path Unweighted Graphs
Visited Array Prevents repeated visits
Disconnected Graph Requires outer traversal loop
Connected Components BFS or DFS
Undirected Cycle Parent Tracking
Directed Cycle Recursion Stack
Bipartite Graph Two-colourable graph
Flood Fill DFS or BFS
Number of Islands DFS or BFS
Path Finding DFS or BFS
END OF CHAPTER 8 – PART 2
What We Have Completed
We have covered:

BFS fundamentals and working.
BFS using Adjacency List.
BFS using Adjacency Matrix.
BFS dry run and complexity.
Applications of BFS.
DFS fundamentals.
Recursive DFS implementation.
DFS recursion stack dry run.
Iterative DFS using Stack.
BFS vs DFS comparison.
Traversal of disconnected graphs.
Connected Components.
Cycle Detection in Undirected Graphs.
Cycle Detection in Directed Graphs.
Bipartite Graph Checking.
Flood Fill.
Number of Islands.
Path Existence.
Java output-based questions.
20 CDAC CMCE MCQs.
Interview questions and coding exercises.

---

# Part 3 - SHORTEST PATH ALGORITHMS AND MINIMUM SPANNING TREES

1. INTRODUCTION TO SHORTEST PATH ALGORITHMS
   A Shortest Path Algorithm is used to find the minimum-cost path between vertices in a graph.

The cost of a path can represent:

Distance between cities.
Travelling time.
Network communication cost.
Transportation expenses.
Delivery costs.
Computer network routing costs.
Consider the following weighted graph:

````text
          4
      A ------- B
      |         |
    2 |         | 5
      |         |
      C ------- D
          3
Suppose we want to find the shortest path from A to D.

Possible paths:

Path 1:


```text
A -> B -> D

Cost = 4 + 5 = 9
Path 2:


```text
A -> C -> D

Cost = 2 + 3 = 5
Therefore:


```text
Shortest Path = A -> C -> D

Minimum Cost = 5
The shortest path is not necessarily the path containing the fewest edges.

It is the path having the minimum total weight.

1.1 Important Terminology

Term	Meaning
Source	Starting vertex
Destination	Target vertex
Edge Weight	Cost associated with an edge
Path	Sequence of connected vertices
Shortest Path	Path with minimum total weight
Distance Array	Stores minimum known distances
Relaxation	Process of improving a distance estimate
Negative Weight	Edge having a negative cost
Negative Cycle	Cycle whose total weight is negative
1.2 What is Relaxation?
Relaxation is the most important concept in shortest path algorithms.

Suppose:


```text
Distance[A] = 0

Distance[B] = 10

Weight(A, B) = 4
We can calculate a potentially better distance to B:


```text
New Distance = Distance[A] + Weight(A, B)

New Distance = 0 + 4

New Distance = 4
Since 4 is smaller than 10, we update:


```text
Distance[B] = 4
The general relaxation formula is:


java
if (distance[u] != INF &&
    distance[u] + weight < distance[v]) {

    distance[v] = distance[u] + weight;
}
Here:

u = Current vertex.
v = Neighbouring vertex.
weight = Edge weight.
distance[u] = Known shortest distance to u.
distance[v] = Current shortest known distance to v.
Relaxation is used in Dijkstra, Bellman-Ford and Floyd-Warshall.

2. TYPES OF SHORTEST PATH PROBLEMS
Shortest path algorithms can solve different types of problems.

2.1 Single Source Shortest Path
Find the shortest distance from one source vertex to every other vertex.

Example:


```text
Source = A
Find:


```text
A -> B
A -> C
A -> D
A -> E
Algorithms:

Dijkstra's Algorithm.
Bellman-Ford Algorithm.
BFS for unweighted graphs.
2.2 Single Pair Shortest Path
Find the shortest path between two specific vertices.

Example:


```text
Source = A

Destination = E
Dijkstra or BFS can be used depending on the graph.

2.3 All Pairs Shortest Path
Find the shortest distance between every possible pair of vertices.

Example:


```text
A -> B
A -> C
B -> A
B -> C
C -> A
C -> B
Algorithm:

Floyd-Warshall Algorithm.
2.4 Shortest Path in an Unweighted Graph
When all edges have equal cost, BFS can find the shortest path.

Time Complexity:


```text
O(V + E)
3. DIJKSTRA'S ALGORITHM
3.1 What is Dijkstra's Algorithm?
Dijkstra's Algorithm is a greedy shortest path algorithm used to find the minimum distance from a single source vertex to all other vertices in a weighted graph.

It works with non-negative edge weights.

Important:

Dijkstra's Algorithm does not work correctly with negative edge weights in general.

It repeatedly selects the unprocessed vertex having the smallest known distance and relaxes its outgoing edges.

3.2 Working Principle
Consider:


```text
          4
      0 ------- 1
      |         |
    2 |         | 5
      |         |
      2 ------- 3
          3
Edges:


```text
0 -> 1 = 4

0 -> 2 = 2

2 -> 3 = 3

1 -> 3 = 5
Find the shortest distance from vertex 0.

Initial distances:


```text
Distance[0] = 0

Distance[1] = INF

Distance[2] = INF

Distance[3] = INF
Here INF represents infinity.

Step 1: Select vertex 0
Current distance:


```text
Distance[0] = 0
Relax its outgoing edges.

For vertex 1:


```text
0 + 4 = 4
For vertex 2:


```text
0 + 2 = 2
Updated distances:


```text
[0, 4, 2, INF]
Step 2: Select vertex 2
Vertex 2 has the smallest unprocessed distance.


```text
Distance[2] = 2
Relax edge 2 -> 3.


```text
2 + 3 = 5
Updated distances:


```text
[0, 4, 2, 5]
Step 3: Select vertex 1

```text
Distance[1] = 4
Relax edge 1 -> 3.


```text
4 + 5 = 9
Current distance to vertex 3 is already 5.

Since 9 is greater than 5, no update is required.

Step 4: Select vertex 3

```text
Distance[3] = 5
No better distance is found.

Final Result

Vertex	Shortest Distance from 0
0	0
1	4
2	2
3	5
Shortest path from 0 to 3:


```text
0 -> 2 -> 3

Total Cost = 2 + 3 = 5
4. DIJKSTRA'S ALGORITHM USING JAVA
We will use:

Adjacency List.
PriorityQueue.
Distance Array.
Java's PriorityQueue implements a min-heap by default for comparable numeric values.

4.1 Java Implementation

java
import java.util.*;

public class Dijkstra {

    static class Edge {

        int destination;
        int weight;

        Edge(int destination, int weight) {

            this.destination = destination;

            this.weight = weight;
        }
    }

    static class Node {

        int vertex;
        int distance;

        Node(int vertex, int distance) {

            this.vertex = vertex;

            this.distance = distance;
        }
    }

    static void dijkstra(
        ArrayList<ArrayList<Edge>> graph,
        int source,
        int vertices
    ) {

        int[] distance = new int[vertices];

        Arrays.fill(distance, Integer.MAX_VALUE);

        PriorityQueue<Node> pq =
            new PriorityQueue<>(
                Comparator.comparingInt(n -> n.distance)
            );

        distance[source] = 0;

        pq.add(new Node(source, 0));

        while (!pq.isEmpty()) {

            Node current = pq.poll();

            int u = current.vertex;

            int currentDistance = current.distance;

            // Ignore outdated entries

            if (currentDistance != distance[u]) {

                continue;
            }

            // Process neighbours

            for (Edge edge : graph.get(u)) {

                int v = edge.destination;

                int weight = edge.weight;

                if (distance[u] != Integer.MAX_VALUE &&
                    distance[u] + weight < distance[v]) {

                    distance[v] = distance[u] + weight;

                    pq.add(new Node(v, distance[v]));
                }
            }
        }

        System.out.println("Shortest Distances:");

        for (int i = 0; i < vertices; i++) {

            if (distance[i] == Integer.MAX_VALUE) {

                System.out.println(i + " : INF");

            } else {

                System.out.println(i + " : " + distance[i]);
            }
        }
    }

    public static void main(String[] args) {

        int vertices = 4;

        ArrayList<ArrayList<Edge>> graph =
            new ArrayList<>();

        for (int i = 0; i < vertices; i++) {

            graph.add(new ArrayList<>());
        }

        graph.get(0).add(new Edge(1, 4));

        graph.get(0).add(new Edge(2, 2));

        graph.get(1).add(new Edge(3, 5));

        graph.get(2).add(new Edge(3, 3));

        dijkstra(graph, 0, vertices);
    }
}
Output

```text
Shortest Distances:

0 : 0
1 : 4
2 : 2
3 : 5
Note: This code represents a directed graph. To make it undirected, insert the reverse edge as well.

4.2 Understanding PriorityQueue

java
PriorityQueue<Node> pq =
    new PriorityQueue<>(
        Comparator.comparingInt(n -> n.distance)
    );
This creates a min-priority queue.

The node having the smallest distance is removed first.

Example:


```text
Insert:

(1, 10)
(2, 5)
(3, 2)
Removal order:


```text
(3, 2)

(2, 5)

(1, 10)
4.3 Why Do We Check Outdated Entries?

java
if (currentDistance != distance[u]) {

    continue;
}
Sometimes a vertex can be inserted into the priority queue more than once.

Suppose:


```text
First discovered distance = 10

Later improved distance = 5
The queue may contain both entries.

The old entry is outdated and should be ignored.

This avoids unnecessary processing.

4.4 Dijkstra Complexity
Using an adjacency list and binary heap:


```text
Time Complexity = O((V + E) log V)

Auxiliary Space = O(V + E)
The graph itself requires O(V + E) space.

The distance array and priority queue require additional working space.

5. PRINTING THE ACTUAL SHORTEST PATH
Sometimes an interviewer asks:

"Find the shortest distance and print the actual path."

A distance array alone cannot reconstruct the complete path.

We use a parent array.

Whenever a shorter path is found:


java
parent[v] = u;
This records the previous vertex on the shortest path.

Java Implementation

java
import java.util.*;

public class DijkstraPath {

    static class Edge {

        int to;
        int weight;

        Edge(int to, int weight) {

            this.to = to;

            this.weight = weight;
        }
    }

    static class Node {

        int vertex;
        int distance;

        Node(int vertex, int distance) {

            this.vertex = vertex;

            this.distance = distance;
        }
    }

    static void shortestPath(
        ArrayList<ArrayList<Edge>> graph,
        int source,
        int destination,
        int vertices
    ) {

        int[] distance = new int[vertices];

        int[] parent = new int[vertices];

        Arrays.fill(distance, Integer.MAX_VALUE);

        Arrays.fill(parent, -1);

        PriorityQueue<Node> pq =
            new PriorityQueue<>(
                Comparator.comparingInt(n -> n.distance)
            );

        distance[source] = 0;

        pq.add(new Node(source, 0));

        while (!pq.isEmpty()) {

            Node current = pq.poll();

            int u = current.vertex;

            if (current.distance != distance[u]) {

                continue;
            }

            for (Edge edge : graph.get(u)) {

                int v = edge.to;

                int weight = edge.weight;

                if (distance[u] != Integer.MAX_VALUE &&
                    distance[u] + weight < distance[v]) {

                    distance[v] = distance[u] + weight;

                    parent[v] = u;

                    pq.add(new Node(v, distance[v]));
                }
            }
        }

        if (distance[destination] == Integer.MAX_VALUE) {

            System.out.println("No path exists");

            return;
        }

        List<Integer> path = new ArrayList<>();

        int current = destination;

        while (current != -1) {

            path.add(current);

            current = parent[current];
        }

        Collections.reverse(path);

        System.out.println("Shortest Distance: " +
                           distance[destination]);

        System.out.println("Shortest Path: " + path);
    }

    public static void main(String[] args) {

        int vertices = 4;

        ArrayList<ArrayList<Edge>> graph =
            new ArrayList<>();

        for (int i = 0; i < vertices; i++) {

            graph.add(new ArrayList<>());
        }

        graph.get(0).add(new Edge(1, 4));

        graph.get(0).add(new Edge(2, 2));

        graph.get(1).add(new Edge(3, 5));

        graph.get(2).add(new Edge(3, 3));

        shortestPath(graph, 0, 3, vertices);
    }
}
Output

```text
Shortest Distance: 5

Shortest Path: [0, 2, 3]
Important: If multiple shortest paths have equal costs, the particular path printed depends on the order in which vertices and edges are processed.

6. BELLMAN-FORD ALGORITHM
6.1 What is Bellman-Ford?
Bellman-Ford is a single-source shortest path algorithm.

It finds shortest distances from one source vertex to every other reachable vertex.

Unlike Dijkstra, Bellman-Ford supports negative edge weights.

It can also detect negative weight cycles reachable from the source.

Important:

Bellman-Ford is slower than Dijkstra but supports negative edge weights.

6.2 Why Do We Need Bellman-Ford?
Consider:


```text
0 -----> 1
 \       |
  \      | -3
   \     |
    ---->2
Suppose:


```text
0 -> 1 = 4

0 -> 2 = 5

1 -> 2 = -3
Possible paths from 0 to 2:

Direct:


```text
0 -> 2

Cost = 5
Through vertex 1:


```text
0 -> 1 -> 2

Cost = 4 + (-3)

Cost = 1
Therefore, shortest distance:


```text
Distance[2] = 1
Dijkstra cannot be relied upon for graphs containing negative edges.

Bellman-Ford handles this situation.

6.3 Main Concept: Relax Every Edge V-1 Times
The most important rule of Bellman-Ford is:

For a graph containing V vertices, relax all edges V-1 times.

Why?

A shortest simple path can contain at most V-1 edges.

For example:


```text
0 -> 1 -> 2 -> 3
There are four vertices and three edges.

Therefore, after V-1 relaxation rounds, all shortest distances are established if there is no relevant negative cycle.

6.4 Bellman-Ford Algorithm

```text
1. Initialize all distances to infinity.

2. Set source distance to 0.

3. Repeat V-1 times:

       For every edge (u, v, weight):

           If distance[u] + weight < distance[v]:

               Update distance[v].

4. Perform one additional relaxation check.

5. If any distance can still be reduced:

       Negative weight cycle exists.
7. BELLMAN-FORD JAVA IMPLEMENTATION
We will use an Edge class.


java
import java.util.*;

public class BellmanFord {

    static class Edge {

        int source;
        int destination;
        int weight;

        Edge(int source, int destination, int weight) {

            this.source = source;

            this.destination = destination;

            this.weight = weight;
        }
    }

    static void bellmanFord(
        Edge[] edges,
        int vertices,
        int source
    ) {

        int[] distance = new int[vertices];

        Arrays.fill(distance, Integer.MAX_VALUE);

        distance[source] = 0;

        // V-1 relaxation rounds

        for (int i = 1; i < vertices; i++) {

            boolean updated = false;

            for (Edge edge : edges) {

                int u = edge.source;

                int v = edge.destination;

                int weight = edge.weight;

                if (distance[u] != Integer.MAX_VALUE &&
                    distance[u] + weight < distance[v]) {

                    distance[v] = distance[u] + weight;

                    updated = true;
                }
            }

            // Optimization

            if (!updated) {

                break;
            }
        }

        // Negative cycle detection

        for (Edge edge : edges) {

            int u = edge.source;

            int v = edge.destination;

            int weight = edge.weight;

            if (distance[u] != Integer.MAX_VALUE &&
                distance[u] + weight < distance[v]) {

                System.out.println(
                    "Negative Weight Cycle Detected"
                );

                return;
            }
        }

        System.out.println("Shortest Distances:");

        for (int i = 0; i < vertices; i++) {

            if (distance[i] == Integer.MAX_VALUE) {

                System.out.println(i + " : INF");

            } else {

                System.out.println(i + " : " + distance[i]);
            }
        }
    }

    public static void main(String[] args) {

        int vertices = 3;

        Edge[] edges = {

            new Edge(0, 1, 4),

            new Edge(0, 2, 5),

            new Edge(1, 2, -3)
        };

        bellmanFord(edges, vertices, 0);
    }
}
Output

```text
Shortest Distances:

0 : 0
1 : 4
2 : 1
7.1 Understanding Negative Cycle Detection
After V-1 iterations, we perform one more relaxation check.

If any distance can still be reduced, a reachable negative weight cycle exists.

Example:


```text
0 -> 1 = 1

1 -> 2 = -2

2 -> 0 = -2
Total cycle weight:


```text
1 + (-2) + (-2) = -3
Every time we travel around this cycle, the total cost decreases by 3.

Therefore, a finite shortest path does not exist for vertices reachable from this cycle.

7.2 Bellman-Ford Complexity

```text
Time Complexity = O(V × E)

Auxiliary Space = O(V)
If the algorithm stores all edges, the edge list itself takes O(E) space.

7.3 Dijkstra vs Bellman-Ford

Feature	Dijkstra	Bellman-Ford
Algorithm	Greedy	Repeated relaxation
Negative Edges	Not supported	Supported
Negative Cycle Detection	No	Yes
Time Complexity	O((V+E) log V) with binary heap	O(V × E)
Shortest Path	Single source	Single source
Performance	Generally faster	Generally slower
8. FLOYD-WARSHALL ALGORITHM
8.1 What is Floyd-Warshall?
Floyd-Warshall is an All-Pairs Shortest Path algorithm.

It finds shortest distances between every pair of vertices.

Unlike Dijkstra and Bellman-Ford, which calculate distances from one source, Floyd-Warshall calculates distances between all pairs.

It supports negative edge weights but does not produce meaningful finite shortest paths when a relevant negative cycle exists.

8.2 Example
Consider:


```text
0 ----5----> 1

1 ----3----> 2

0 ----10---> 2
Initially:


```text
0 -> 2 = 10
But through vertex 1:


```text
0 -> 1 -> 2

Cost = 5 + 3 = 8
Therefore:


```text
Shortest Distance from 0 to 2 = 8
Floyd-Warshall checks whether travelling through an intermediate vertex produces a shorter route.

8.3 Dynamic Programming Concept
The main formula is:


```text
dist[i][j] =
    min(
        dist[i][j],
        dist[i][k] + dist[k][j]
    )
Meaning:

The shortest distance from i to j is the minimum of:

Existing distance from i to j.
Distance from i to k plus distance from k to j.
Here k is the intermediate vertex.

This process is repeated for every possible intermediate vertex.

8.4 Initial Distance Matrix
For the graph:


```text
0 -> 1 = 5

1 -> 2 = 3

0 -> 2 = 10
Initial matrix:


```text
     0    1    2

0    0    5   10

1   INF   0    3

2   INF  INF   0
After considering vertex 1 as an intermediate vertex:


```text
Distance[0][2] = min(10, 5 + 3)

Distance[0][2] = 8
Final matrix:


```text
     0    1    2

0    0    5    8

1   INF   0    3

2   INF  INF   0
9. FLOYD-WARSHALL JAVA IMPLEMENTATION

java
import java.util.*;

public class FloydWarshall {

    static final int INF = 999999;

    static void floydWarshall(int[][] dist) {

        int vertices = dist.length;

        for (int k = 0; k < vertices; k++) {

            for (int i = 0; i < vertices; i++) {

                for (int j = 0; j < vertices; j++) {

                    if (dist[i][k] != INF &&
                        dist[k][j] != INF) {

                        if (dist[i][k] + dist[k][j]
                            < dist[i][j]) {

                            dist[i][j] =
                                dist[i][k] + dist[k][j];
                        }
                    }
                }
            }
        }

        // Negative cycle detection

        for (int i = 0; i < vertices; i++) {

            if (dist[i][i] < 0) {

                System.out.println(
                    "Negative Weight Cycle Detected"
                );

                return;
            }
        }

        System.out.println("Shortest Distance Matrix:");

        for (int i = 0; i < vertices; i++) {

            for (int j = 0; j < vertices; j++) {

                if (dist[i][j] == INF) {

                    System.out.print("INF ");

                } else {

                    System.out.print(dist[i][j] + " ");
                }
            }

            System.out.println();
        }
    }

    public static void main(String[] args) {

        int[][] graph = {

            {0, 5, 10},

            {INF, 0, 3},

            {INF, INF, 0}
        };

        floydWarshall(graph);
    }
}
Output

```text
Shortest Distance Matrix:

0 5 8

INF 0 3

INF INF 0
9.1 Floyd-Warshall Complexity

```text
Time Complexity = O(V³)

Auxiliary Space = O(1)
The algorithm modifies the supplied distance matrix in place.

The matrix itself requires O(V²) space.

9.2 Comparison of Shortest Path Algorithms

Algorithm	Problem	Negative Edges	Negative Cycle Detection	Time
BFS	Single source, unweighted	No weighted edges	No	O(V+E)
Dijkstra	Single source	No	No	O((V+E) log V)
Bellman-Ford	Single source	Yes	Yes	O(VE)
Floyd-Warshall	All pairs	Yes	Yes	O(V³)
10. MINIMUM SPANNING TREE (MST)
10.1 What is a Spanning Tree?
A Spanning Tree is a subgraph of a connected, undirected graph that:

Contains all vertices.
Is connected.
Contains no cycles.
A spanning tree of a graph with V vertices always contains:


```text
V - 1 edges
10.2 Example
Consider:


```text
       0
      / \
     1---2
      \ /
       3
A spanning tree connects all four vertices without creating a cycle.

One possible spanning tree:


```text
0
|
1
|
2
|
3
It contains:


```text
4 - 1 = 3 edges
10.3 What is a Minimum Spanning Tree?
A Minimum Spanning Tree is a spanning tree having the minimum possible total edge weight.

Example:


```text
       10
    0 ----- 1
     \       |
    6 \      | 5
       \     |
         2 --3
           4
Suppose the edges are:


```text
0 - 1 = 10

0 - 2 = 6

1 - 3 = 5

2 - 3 = 4
To construct an MST, select the minimum-weight edges while ensuring that:

No cycle is created.
All vertices become connected.
Select:


```text
2 - 3 = 4

1 - 3 = 5

0 - 2 = 6
Total MST cost:


```text
4 + 5 + 6 = 15
Number of selected edges:


```text
V - 1 = 3
10.4 Applications of MST
Designing minimum-cost computer networks.
Connecting cities using minimum road construction cost.
Electrical wiring networks.
Telecommunications infrastructure.
Pipeline network design.
Clustering algorithms.
Two major MST algorithms are:

Prim's Algorithm.
Kruskal's Algorithm.
11. PRIM'S ALGORITHM
11.1 What is Prim's Algorithm?
Prim's Algorithm is a greedy algorithm used to construct a Minimum Spanning Tree of a connected, undirected, weighted graph.

It starts from an arbitrary vertex and repeatedly adds the minimum-weight edge connecting the current tree to a vertex outside the tree.

Important:

Prim's Algorithm selects edges that expand one growing tree.

11.2 Working Example
Consider:


```text
       10
    0 ----- 1
     \       |
    6 \      | 5
       \     |
         2 --3
           4
Start from vertex 0.

Step 1
Start with:


```text
MST = {0}
Available edges:


```text
0 - 1 = 10

0 - 2 = 6
Select minimum:


```text
0 - 2 = 6
Step 2
Current MST vertices:


```text
{0, 2}
Available edges connecting to outside vertices:


```text
0 - 1 = 10

2 - 3 = 4
Select:


```text
2 - 3 = 4
Step 3
Current MST vertices:


```text
{0, 2, 3}
Available edges:


```text
0 - 1 = 10

3 - 1 = 5
Select:


```text
3 - 1 = 5
Final MST
Selected edges:


```text
0 - 2 = 6

2 - 3 = 4

3 - 1 = 5
Total weight:


```text
15
12. PRIM'S ALGORITHM USING JAVA
We will use:

Adjacency List.
PriorityQueue.
Visited Array.

java
import java.util.*;

public class PrimsAlgorithm {

    static class Edge {

        int destination;
        int weight;

        Edge(int destination, int weight) {

            this.destination = destination;

            this.weight = weight;
        }
    }

    static class Node {

        int vertex;
        int weight;

        Node(int vertex, int weight) {

            this.vertex = vertex;

            this.weight = weight;
        }
    }

    static void prim(
        ArrayList<ArrayList<Edge>> graph,
        int vertices
    ) {

        boolean[] visited = new boolean[vertices];

        PriorityQueue<Node> pq =
            new PriorityQueue<>(
                Comparator.comparingInt(n -> n.weight)
            );

        int totalWeight = 0;

        int edgesSelected = 0;

        pq.add(new Node(0, 0));

        while (!pq.isEmpty() &&
               edgesSelected < vertices) {

            Node current = pq.poll();

            int u = current.vertex;

            if (visited[u]) {

                continue;
            }

            visited[u] = true;

            totalWeight += current.weight;

            if (current.weight != 0) {

                edgesSelected++;

                System.out.println(
                    "Vertex: " + u +
                    ", Edge Weight: " + current.weight
                );
            }

            for (Edge edge : graph.get(u)) {

                if (!visited[edge.destination]) {

                    pq.add(
                        new Node(
                            edge.destination,
                            edge.weight
                        )
                    );
                }
            }
        }

        int visitedCount = 0;

        for (boolean value : visited) {

            if (value) {
                visitedCount++;
            }
        }

        if (visitedCount != vertices) {

            System.out.println(
                "Graph is disconnected. MST does not exist."
            );

            return;
        }

        System.out.println(
            "Total MST Weight: " + totalWeight
        );
    }

    public static void main(String[] args) {

        int vertices = 4;

        ArrayList<ArrayList<Edge>> graph =
            new ArrayList<>();

        for (int i = 0; i < vertices; i++) {

            graph.add(new ArrayList<>());
        }

        // Undirected edges

        graph.get(0).add(new Edge(1, 10));
        graph.get(1).add(new Edge(0, 10));

        graph.get(0).add(new Edge(2, 6));
        graph.get(2).add(new Edge(0, 6));

        graph.get(1).add(new Edge(3, 5));
        graph.get(3).add(new Edge(1, 5));

        graph.get(2).add(new Edge(3, 4));
        graph.get(3).add(new Edge(2, 4));

        prim(graph, vertices);
    }
}
Output

```text
Vertex: 2, Edge Weight: 6

Vertex: 3, Edge Weight: 4

Vertex: 1, Edge Weight: 5

Total MST Weight: 15
12.1 Prim's Algorithm Complexity
Using adjacency list and binary heap:


```text
Time Complexity = O(E log V)

Auxiliary Space = O(V + E)
The graph representation requires O(V + E) space.

13. KRUSKAL'S ALGORITHM
13.1 What is Kruskal's Algorithm?
Kruskal's Algorithm is a greedy algorithm used to find a Minimum Spanning Tree.

Unlike Prim's Algorithm, Kruskal's Algorithm considers all edges in increasing order of weight.

It selects the minimum-weight edge if adding it does not create a cycle.

Important:

Kruskal's Algorithm = Sorting + Disjoint Set Union (DSU).

13.2 Working Example
Consider:


```text
0 --10-- 1

0 --6--- 2

1 --5--- 3

2 --4--- 3
All edges:


Edge	Weight
0 - 1	10
0 - 2	6
1 - 3	5
2 - 3	4
Step 1: Sort Edges

```text
2 - 3 = 4

1 - 3 = 5

0 - 2 = 6

0 - 1 = 10
Step 2: Select Minimum Edge
Select:


```text
2 - 3 = 4
No cycle is formed.

Step 3
Select:


```text
1 - 3 = 5
No cycle is formed.

Step 4
Select:


```text
0 - 2 = 6
No cycle is formed.

Now all four vertices are connected.

We need exactly:


```text
V - 1 = 3 edges
Therefore, stop.

Final MST:


```text
2 - 3 = 4

1 - 3 = 5

0 - 2 = 6
Total:


```text
15
14. DISJOINT SET UNION (DSU)
DSU is also called:

Union-Find.
Disjoint Set Data Structure.
It is used to manage a collection of non-overlapping sets.

It supports two main operations:

Find.
Union.
DSU is particularly important in Kruskal's Algorithm because it helps detect whether adding an edge will create a cycle.

14.1 Find Operation
Find determines which set a particular element belongs to.

It returns the representative or root of that set.

Example:


```text
Set 1 = {0, 1, 2}

Set 2 = {3, 4}
Find:


```text
find(1) = 0

find(2) = 0

find(4) = 3
Here 0 and 3 are representative elements.

14.2 Union Operation
Union combines two different sets.

Initially:


```text
{0}

{1}

{2}

{3}
Perform:


```text
union(0, 1)
Result:


```text
{0, 1}

{2}

{3}
Perform:


```text
union(1, 2)
Result:


```text
{0, 1, 2}

{3}
14.3 DSU Without Optimization
A basic implementation uses a parent array.


java
int[] parent = new int[n];

for (int i = 0; i < n; i++) {

    parent[i] = i;
}
Initially, every element is its own parent.

Basic Find:


java
static int find(int x, int[] parent) {

    if (parent[x] == x) {

        return x;
    }

    return find(parent[x], parent);
}
Basic Union:


java
static void union(
    int a,
    int b,
    int[] parent
) {

    int rootA = find(a, parent);

    int rootB = find(b, parent);

    if (rootA != rootB) {

        parent[rootB] = rootA;
    }
}
However, this implementation can create highly unbalanced trees.

We improve it using:

Path Compression.
Union by Rank.
15. UNION-FIND WITH PATH COMPRESSION
Path Compression optimizes the Find operation.

Whenever we search for a representative, we make every visited node point directly to the root.

Before:


```text
0
|
1
|
2
|
3
After Find(3):


```text
    0
  / | \
 1  2  3
This reduces the depth of the tree.

Java Implementation

java
public class DSU {

    int[] parent;

    DSU(int n) {

        parent = new int[n];

        for (int i = 0; i < n; i++) {

            parent[i] = i;
        }
    }

    int find(int x) {

        if (parent[x] != x) {

            parent[x] = find(parent[x]);
        }

        return parent[x];
    }

    void union(int a, int b) {

        int rootA = find(a);

        int rootB = find(b);

        if (rootA != rootB) {

            parent[rootB] = rootA;
        }
    }

    public static void main(String[] args) {

        DSU dsu = new DSU(5);

        dsu.union(0, 1);

        dsu.union(1, 2);

        dsu.union(3, 4);

        System.out.println(dsu.find(2));

        System.out.println(dsu.find(4));

        System.out.println(dsu.find(0) ==
                           dsu.find(2));
    }
}
Output

```text
0

3

true
The exact representative depends on the order of union operations.

16. UNION BY RANK
Union by Rank is another optimization.

Rank approximately represents the height of the tree.

When combining two sets:

Attach the smaller-rank tree below the larger-rank tree.
If both ranks are equal, attach one root below the other and increase the resulting root's rank.
This prevents unnecessarily tall trees.

Complete Optimized DSU

java
public class OptimizedDSU {

    int[] parent;

    int[] rank;

    OptimizedDSU(int n) {

        parent = new int[n];

        rank = new int[n];

        for (int i = 0; i < n; i++) {

            parent[i] = i;

            rank[i] = 0;
        }
    }

    int find(int x) {

        if (parent[x] != x) {

            parent[x] = find(parent[x]);
        }

        return parent[x];
    }

    void union(int a, int b) {

        int rootA = find(a);

        int rootB = find(b);

        if (rootA == rootB) {

            return;
        }

        if (rank[rootA] < rank[rootB]) {

            parent[rootA] = rootB;
        }

        else if (rank[rootA] > rank[rootB]) {

            parent[rootB] = rootA;
        }

        else {

            parent[rootB] = rootA;

            rank[rootA]++;
        }
    }

    public static void main(String[] args) {

        OptimizedDSU dsu =
            new OptimizedDSU(5);

        dsu.union(0, 1);

        dsu.union(1, 2);

        dsu.union(3, 4);

        System.out.println(
            dsu.find(0) == dsu.find(2)
        );

        System.out.println(
            dsu.find(0) == dsu.find(4)
        );
    }
}
Output

```text
true

false
16.1 DSU Complexity
With path compression and union by rank:


```text
Amortized Time Complexity = O(α(V))
Here α(V) is the inverse Ackermann function.

For practical input sizes, it is extremely small.

17. KRUSKAL'S ALGORITHM USING JAVA
Now that we understand DSU, we can implement Kruskal's Algorithm.

Java Implementation

java
import java.util.*;

public class KruskalAlgorithm {

    static class Edge {

        int source;

        int destination;

        int weight;

        Edge(int source, int destination, int weight) {

            this.source = source;

            this.destination = destination;

            this.weight = weight;
        }
    }

    static class DSU {

        int[] parent;

        int[] rank;

        DSU(int n) {

            parent = new int[n];

            rank = new int[n];

            for (int i = 0; i < n; i++) {

                parent[i] = i;

                rank[i] = 0;
            }
        }

        int find(int x) {

            if (parent[x] != x) {

                parent[x] = find(parent[x]);
            }

            return parent[x];
        }

        boolean union(int a, int b) {

            int rootA = find(a);

            int rootB = find(b);

            if (rootA == rootB) {

                return false;
            }

            if (rank[rootA] < rank[rootB]) {

                parent[rootA] = rootB;
            }

            else if (rank[rootA] > rank[rootB]) {

                parent[rootB] = rootA;
            }

            else {

                parent[rootB] = rootA;

                rank[rootA]++;
            }

            return true;
        }
    }

    static void kruskal(
        Edge[] edges,
        int vertices
    ) {

        Arrays.sort(
            edges,
            Comparator.comparingInt(e -> e.weight)
        );

        DSU dsu = new DSU(vertices);

        int totalWeight = 0;

        int selectedEdges = 0;

        System.out.println("MST Edges:");

        for (Edge edge : edges) {

            if (dsu.union(
                edge.source,
                edge.destination
            )) {

                System.out.println(
                    edge.source + " - " +
                    edge.destination +
                    " : " + edge.weight
                );

                totalWeight += edge.weight;

                selectedEdges++;

                if (selectedEdges == vertices - 1) {

                    break;
                }
            }
        }

        if (selectedEdges != vertices - 1) {

            System.out.println(
                "Graph is disconnected. No spanning tree exists."
            );

            return;
        }

        System.out.println(
            "Total MST Weight: " + totalWeight
        );
    }

    public static void main(String[] args) {

        int vertices = 4;

        Edge[] edges = {

            new Edge(0, 1, 10),

            new Edge(0, 2, 6),

            new Edge(1, 3, 5),

            new Edge(2, 3, 4)
        };

        kruskal(edges, vertices);
    }
}
Output

```text
MST Edges:

2 - 3 : 4

1 - 3 : 5

0 - 2 : 6

Total MST Weight: 15
17.1 Understanding Cycle Detection in Kruskal
Consider:


```text
0 ---- 1

|      /

|     /

2 ----
Suppose we already selected:


```text
0 - 1

1 - 2
Now consider adding:


```text
0 - 2
Before adding the edge, check:


java
dsu.find(0) == dsu.find(2)
If true, both vertices already belong to the same set.

Adding the edge would create a cycle.

Therefore, reject the edge.

17.2 Kruskal Complexity
Sorting all edges:


```text
O(E log E)
DSU operations:


```text
O(E × α(V))
Overall:


```text
O(E log E)
Auxiliary DSU space:


```text
O(V)
The edge list requires O(E) space.

18. PRIM'S VS KRUSKAL'S ALGORITHM
This is a very important CDAC examination and interview question.


Feature	Prim's Algorithm	Kruskal's Algorithm
Type	Greedy	Greedy
Approach	Expands one tree	Selects globally smallest eligible edges
Starting Vertex	Required	Not required
Data Structure	PriorityQueue	Sorting + DSU
Cycle Detection	Visited array	DSU
Edge Selection	Minimum edge connecting tree to outside	Minimum edge that doesn't create cycle
Time	O(E log V)	O(E log E)
Suitable For	Often useful for dense graphs with suitable implementations	Often useful for sparse graphs
Disconnected Graph	Can construct a minimum spanning forest with a component-wise version	Naturally constructs a minimum spanning forest
Most Important Difference
Prim:


```text
Start from a vertex.

Grow one tree.

Add the minimum connecting edge.
Kruskal:


```text
Sort all edges.

Select the smallest valid edge.

Reject edges that create cycles.
19. IMPORTANT SHORTEST PATH AND MST CONCEPTS
19.1 Shortest Path Tree vs Minimum Spanning Tree
These two concepts are frequently confused.

Shortest Path Tree
A Shortest Path Tree contains shortest paths from one source vertex to all reachable vertices.

Minimum Spanning Tree
An MST connects all vertices with minimum total edge weight.

Example:

A graph can have a shortest path tree that is different from its MST.

Important:

A Minimum Spanning Tree does not necessarily provide the shortest path between every pair of vertices.

19.2 Can MST Be Applied to Directed Graphs?
Prim's and Kruskal's standard MST algorithms operate on undirected weighted graphs.

For directed graphs, a related but different problem is the Minimum Spanning Arborescence.

19.3 Can an MST Contain Negative Edge Weights?
Yes.

Prim's and Kruskal's algorithms can work with negative edge weights.

Negative edge weights do not create a problem for MST construction.

19.4 Can a Graph Have Multiple MSTs?
Yes.

If several edges have equal weights, multiple different spanning trees may have the same minimum total weight.

However, if all edge weights are distinct, the MST is unique.

19.5 What if the Graph Is Disconnected?
A disconnected graph does not have a spanning tree covering all its vertices.

However, we can construct a Minimum Spanning Forest, consisting of one MST for each connected component.

20. COMPLETE ALGORITHM COMPARISON TABLE

Algorithm	Purpose	Technique	Time Complexity
BFS	Unweighted shortest path	Queue	O(V+E)
DFS	Graph traversal	Stack	O(V+E)
Dijkstra	Single-source shortest path	Greedy + PriorityQueue	O((V+E) log V)
Bellman-Ford	Single-source shortest path	Relaxation	O(VE)
Floyd-Warshall	All-pairs shortest path	Dynamic Programming	O(V³)
Prim	Minimum Spanning Tree	Greedy + PriorityQueue	O(E log V)
Kruskal	Minimum Spanning Tree	Greedy + DSU	O(E log E)
DSU	Set management	Union-Find	O(α(V)) amortized per operation

24. INTERVIEW CODING PRACTICE
    Solve these problems independently using Java.

No. Problem Algorithm
1 Shortest distance using Dijkstra Dijkstra
2 Print shortest path Dijkstra + Parent
3 Shortest path with negative weights Bellman-Ford
4 Detect negative weight cycle Bellman-Ford
5 All-pairs shortest paths Floyd-Warshall
6 Construct Minimum Spanning Tree Prim
7 Construct Minimum Spanning Tree Kruskal
8 Implement DSU Union-Find
9 Implement Path Compression DSU
10 Implement Union by Rank DSU
11 Find minimum cost to connect cities MST
12 Check whether a graph is connected BFS/DFS
13 Find minimum cost network Prim/Kruskal
14 Find shortest path between two cities Dijkstra
15 Detect redundant connection DSU
Additional Practice Problem
Problem: Connecting Cities with Minimum Cost

You are given N cities and a list of possible connections between them. Each connection has a construction cost.

Find the minimum total cost required to connect all cities.

If all cities cannot be connected, return -1.

Expected approach:

Model cities as vertices.
Model connections as weighted edges.
Apply Kruskal's Algorithm.
Use DSU to prevent cycles.
Count selected edges.
Return total cost if exactly N-1 edges were selected; otherwise return -1. 25. FINAL REVISION SHEET

Concept Important Fact
Shortest Path Minimum total path weight
Relaxation Improves distance estimate
Dijkstra Greedy algorithm
Dijkstra Data Structure PriorityQueue
Dijkstra Negative Edges Not supported
Bellman-Ford Supports negative edges
Bellman-Ford Iterations V-1
Negative Cycle Additional relaxation check
Floyd-Warshall All-pairs shortest paths
Floyd-Warshall Technique Dynamic Programming
Floyd-Warshall Complexity O(V³)
Spanning Tree Connected and acyclic
MST Edge Count V-1
MST Algorithms Prim and Kruskal
Prim Grows one tree
Kruskal Sorts edges
Kruskal Data Structure DSU
DSU Find Finds representative
DSU Union Combines sets
Path Compression Reduces tree depth
Union by Rank Maintains balanced trees
Optimized DSU O(α(V)) amortized
Negative MST Edges Allowed
Multiple MSTs Possible
````

---

# PART 4 – ADVANCED GRAPH ALGORITHMS AND FINAL EXAM PREPARATION

1. DIRECTED ACYCLIC GRAPH (DAG)
   1.1 What is a DAG?
   DAG stands for Directed Acyclic Graph.

It is a directed graph that does not contain any directed cycle.

There are two important properties:

Every edge has a direction.
It is impossible to start from a vertex and return to the same vertex by following directed edges.
Example:

````text
A ---> B ---> D
|
|
v
C ---> D
This is a DAG because there is no directed cycle.

Consider another graph:


```text
A ---> B
^      |
|      |
|      v
D <--- C
Here:


```text
A -> B -> C -> D -> A
This graph contains a directed cycle.

Therefore, it is not a DAG.

1.2 Applications of DAG
DAGs are commonly used in:

Task scheduling.
Course prerequisites.
Software dependency management.
Build systems.
Project planning.
Workflow management.
Dependency resolution.
Data processing pipelines.
Example:

Suppose a student must complete the following subjects:


```text
Programming Basics
       |
       v
Data Structures
       |
       v
Algorithms
       |
       v
Advanced Programming
The subjects have dependencies.

You cannot study Algorithms before completing Data Structures.

Such dependencies can be represented using a DAG.

1.3 Important Property of DAG
Every DAG has at least one vertex with indegree zero.

A DAG also has at least one vertex with outdegree zero.

Where:

Indegree = Number of incoming edges.
Outdegree = Number of outgoing edges.
These properties are important in topological sorting.

2. TOPOLOGICAL SORTING
2.1 What is Topological Sorting?
Topological Sorting is a linear ordering of vertices in a Directed Acyclic Graph such that for every directed edge:


```text
u -> v
Vertex u appears before vertex v in the ordering.

In simple words:

If one task depends on another task, the prerequisite task must appear first.

Important:

Topological sorting is possible only for Directed Acyclic Graphs.

2.2 Example
Consider:


```text
A ---> C
|
|
v
B ---> D
Possible topological order:


```text
A, B, C, D
Another possible order:


```text
A, C, B, D
Both are valid because all directed dependencies are respected.

Topological sorting is not necessarily unique.

2.3 Real-Life Example
Suppose software installation has dependencies:


```text
Operating System
       |
       v
Java Runtime
       |
       v
Java Application
The Java Runtime must be installed before the Java Application.

A valid topological order is:


```text
Operating System

Java Runtime

Java Application
2.4 Applications
Course prerequisite scheduling.
Compilation order.
Build dependency resolution.
Project scheduling.
Task execution.
Package installation.
There are two common approaches:

DFS-based Topological Sorting.
Kahn's Algorithm using BFS.
3. TOPOLOGICAL SORTING USING DFS
DFS can be used to generate a topological ordering.

3.1 Working Principle
The algorithm follows these steps:

Create a visited array.
Start DFS from an unvisited vertex.
Visit all unvisited neighbours.
After visiting all neighbours, push the current vertex into a stack.
Continue for every unvisited vertex.
Pop elements from the stack to obtain the ordering.
The important concept is:

Push a vertex into the stack only after all its outgoing neighbours have been processed.

This is called finishing-time ordering.

3.2 Example
Consider:


```text
0 ---> 2
|
|
v
1 ---> 3
Suppose DFS begins from vertex 0.

Possible processing:


```text
Visit 0

Visit 1

Visit 3

Push 3

Push 1

Visit 2

Push 2

Push 0
Stack insertion order:


```text
3, 1, 2, 0
Stack removal order:


```text
0, 2, 1, 3
This is a valid topological ordering.

3.3 Java Implementation

java
import java.util.*;

public class TopologicalDFS {

    static void dfs(
        int vertex,
        ArrayList<ArrayList<Integer>> graph,
        boolean[] visited,
        Stack<Integer> stack
    ) {

        visited[vertex] = true;

        for (int neighbour : graph.get(vertex)) {

            if (!visited[neighbour]) {

                dfs(neighbour, graph, visited, stack);
            }
        }

        stack.push(vertex);
    }

    static void topologicalSort(
        ArrayList<ArrayList<Integer>> graph,
        int vertices
    ) {

        boolean[] visited = new boolean[vertices];

        Stack<Integer> stack = new Stack<>();

        for (int i = 0; i < vertices; i++) {

            if (!visited[i]) {

                dfs(i, graph, visited, stack);
            }
        }

        System.out.println("Topological Order:");

        while (!stack.isEmpty()) {

            System.out.print(stack.pop() + " ");
        }
    }

    public static void main(String[] args) {

        int vertices = 4;

        ArrayList<ArrayList<Integer>> graph =
            new ArrayList<>();

        for (int i = 0; i < vertices; i++) {

            graph.add(new ArrayList<>());
        }

        graph.get(0).add(1);

        graph.get(0).add(2);

        graph.get(1).add(3);

        graph.get(2).add(3);

        topologicalSort(graph, vertices);
    }
}
Possible Output

```text
Topological Order:

0 2 1 3
Another valid result is:


```text
0 1 2 3
Both are correct.

Important: This basic DFS implementation assumes the graph is a DAG. It does not independently detect cycles.

3.4 Complexity

```text
Time Complexity = O(V + E)

Space Complexity = O(V)
The graph representation additionally requires O(V + E) space.

4. KAHN'S ALGORITHM
4.1 What is Kahn's Algorithm?
Kahn's Algorithm is a BFS-based approach to topological sorting.

It uses the concept of indegree.

Instead of using DFS and a stack, it uses a Queue.

4.2 Main Concept: Indegree
Indegree is the number of incoming edges to a vertex.

Consider:


```text
0 ---> 2
|
|
v
1 ---> 3
Indegree values:


Vertex	Indegree
0	0
1	1
2	1
3	2
Vertex 0 has indegree zero because no edge enters it.

Kahn's Algorithm begins by inserting all vertices having indegree zero into a queue.

4.3 Algorithm Steps
Calculate the indegree of every vertex.
Insert all vertices with indegree zero into a queue.
Remove one vertex from the queue.
Add it to the topological ordering.
Reduce the indegree of all its outgoing neighbours.
If a neighbour's indegree becomes zero, insert it into the queue.
Repeat until the queue becomes empty.
If the number of processed vertices is less than V, the graph contains a directed cycle.

4.4 Java Implementation

java
import java.util.*;

public class KahnsAlgorithm {

    static void topologicalSort(
        ArrayList<ArrayList<Integer>> graph,
        int vertices
    ) {

        int[] indegree = new int[vertices];

        // Calculate indegree

        for (int i = 0; i < vertices; i++) {

            for (int neighbour : graph.get(i)) {

                indegree[neighbour]++;
            }
        }

        Queue<Integer> queue = new LinkedList<>();

        // Insert zero-indegree vertices

        for (int i = 0; i < vertices; i++) {

            if (indegree[i] == 0) {

                queue.add(i);
            }
        }

        int count = 0;

        System.out.println("Topological Order:");

        while (!queue.isEmpty()) {

            int current = queue.poll();

            System.out.print(current + " ");

            count++;

            for (int neighbour : graph.get(current)) {

                indegree[neighbour]--;

                if (indegree[neighbour] == 0) {

                    queue.add(neighbour);
                }
            }
        }

        if (count != vertices) {

            System.out.println(
                "\nGraph contains a cycle."
            );
        }
    }

    public static void main(String[] args) {

        int vertices = 4;

        ArrayList<ArrayList<Integer>> graph =
            new ArrayList<>();

        for (int i = 0; i < vertices; i++) {

            graph.add(new ArrayList<>());
        }

        graph.get(0).add(1);

        graph.get(0).add(2);

        graph.get(1).add(3);

        graph.get(2).add(3);

        topologicalSort(graph, vertices);
    }
}
Possible Output

```text
Topological Order:

0 1 2 3
4.5 Kahn's Algorithm Complexity

```text
Time Complexity = O(V + E)

Space Complexity = O(V)
4.6 DFS vs Kahn's Algorithm

Feature	DFS Approach	Kahn's Algorithm
Technique	DFS	BFS
Data Structure	Stack	Queue
Uses Indegree	No	Yes
Topological Sorting	Yes	Yes
Cycle Detection	With additional color states	Naturally detects cycles
Time Complexity	O(V+E)	O(V+E)
5. CYCLE DETECTION IN DAG
A Directed Acyclic Graph must not contain a directed cycle.

Kahn's Algorithm provides a simple cycle detection technique.

5.1 Using Kahn's Algorithm
Suppose:


```text
0 ---> 1
^      |
|      |
|      v
3 <--- 2
Every vertex belongs to a cycle.

There is no vertex with indegree zero.

Therefore, the queue is initially empty.

No vertex can be processed.

Since:


```text
Processed Vertices < Total Vertices
A cycle exists.

5.2 Java Implementation

java
import java.util.*;

public class DirectedCycle {

    static boolean hasCycle(
        ArrayList<ArrayList<Integer>> graph,
        int vertices
    ) {

        int[] indegree = new int[vertices];

        for (int i = 0; i < vertices; i++) {

            for (int neighbour : graph.get(i)) {

                indegree[neighbour]++;
            }
        }

        Queue<Integer> queue = new LinkedList<>();

        for (int i = 0; i < vertices; i++) {

            if (indegree[i] == 0) {

                queue.add(i);
            }
        }

        int processed = 0;

        while (!queue.isEmpty()) {

            int current = queue.poll();

            processed++;

            for (int neighbour : graph.get(current)) {

                indegree[neighbour]--;

                if (indegree[neighbour] == 0) {

                    queue.add(neighbour);
                }
            }
        }

        return processed != vertices;
    }

    public static void main(String[] args) {

        int vertices = 3;

        ArrayList<ArrayList<Integer>> graph =
            new ArrayList<>();

        for (int i = 0; i < vertices; i++) {

            graph.add(new ArrayList<>());
        }

        graph.get(0).add(1);

        graph.get(1).add(2);

        graph.get(2).add(0);

        System.out.println(hasCycle(graph, vertices));
    }
}
Output

```text
true
6. COURSE SCHEDULE PROBLEM
This is a very popular graph interview problem.

6.1 Problem Statement
Suppose there are N courses.

Some courses have prerequisites.

You are given prerequisite pairs:


```text
[course, prerequisite]
You must determine whether it is possible to complete all courses.

Example:


```text
Courses = 4

Prerequisites:

[1, 0]

[2, 1]

[3, 2]
Meaning:

Course 0 must be completed before course 1.
Course 1 must be completed before course 2.
Course 2 must be completed before course 3.
The dependency graph is:


```text
0 ---> 1 ---> 2 ---> 3
All courses can be completed.

Now consider:


```text
[1, 0]

[2, 1]

[0, 2]
Graph:


```text
0 ---> 1 ---> 2
^             |
|_____________|
There is a cycle.

Therefore, completing all courses is impossible.

6.2 Java Implementation

java
import java.util.*;

public class CourseSchedule {

    static boolean canFinish(
        int courses,
        int[][] prerequisites
    ) {

        ArrayList<ArrayList<Integer>> graph =
            new ArrayList<>();

        for (int i = 0; i < courses; i++) {

            graph.add(new ArrayList<>());
        }

        int[] indegree = new int[courses];

        for (int[] pair : prerequisites) {

            int course = pair[0];

            int prerequisite = pair[1];

            graph.get(prerequisite).add(course);

            indegree[course]++;
        }

        Queue<Integer> queue = new LinkedList<>();

        for (int i = 0; i < courses; i++) {

            if (indegree[i] == 0) {

                queue.add(i);
            }
        }

        int completed = 0;

        while (!queue.isEmpty()) {

            int current = queue.poll();

            completed++;

            for (int neighbour : graph.get(current)) {

                indegree[neighbour]--;

                if (indegree[neighbour] == 0) {

                    queue.add(neighbour);
                }
            }
        }

        return completed == courses;
    }

    public static void main(String[] args) {

        int[][] prerequisites = {

            {1, 0},

            {2, 1},

            {3, 2}
        };

        System.out.println(
            canFinish(4, prerequisites)
        );
    }
}
Output

```text
true
Time Complexity:


```text
O(V + E)
7. TASK SCHEDULING AND CRITICAL PATH
7.1 What is Task Scheduling?
Task scheduling means arranging tasks in an order that satisfies their dependencies.

Example:


```text
Task A: 2 days

Task B: 4 days

Task C: 3 days

Task D: 2 days
Dependencies:


```text
A ---> B ---> D

A ---> C ---> D
A must finish before B and C.

Both B and C must finish before D.

This dependency graph is a DAG.

7.2 What is Critical Path?
The Critical Path is the longest-duration path through a project dependency network.

It determines the minimum possible time needed to complete the entire project, assuming tasks can run in parallel whenever dependencies allow.

Possible paths:


```text
A -> B -> D

2 + 4 + 2 = 8 days

```text
A -> C -> D

2 + 3 + 2 = 7 days
Critical Path:


```text
A -> B -> D
Minimum project completion time:


```text
8 days
7.3 Important Difference

Shortest Path	Critical Path
Minimizes total path weight	Maximizes project duration
Used in routing	Used in project planning
Dijkstra/Bellman-Ford etc.	DAG Dynamic Programming
Minimum cost	Longest dependency path
For a DAG, the longest path can be calculated using topological ordering and dynamic programming.

8. STRONGLY CONNECTED COMPONENTS (SCC)
8.1 What is a Strongly Connected Component?
In a directed graph, a Strongly Connected Component is a maximal group of vertices in which every vertex is reachable from every other vertex.

Example:


```text
0 ---> 1
^      |
|      v
2 <----+
All three vertices can reach each other.

Therefore:


```text
SCC = {0, 1, 2}
Consider:


```text
0 ---> 1 ---> 2

^           |
|___________|
These three vertices form one SCC.

Now consider:


```text
0 <----> 1

2 <----> 3

1 -----> 2
SCCs are:


```text
{0, 1}

{2, 3}
There is a connection from the first component to the second, but not a return path.

8.2 Applications of SCC
Social network analysis.
Dependency analysis.
Web page link analysis.
Compiler optimization.
Deadlock analysis.
Communication networks.
Two important SCC algorithms:

Kosaraju's Algorithm.
Tarjan's Algorithm.
9. KOSARAJU'S ALGORITHM
9.1 What is Kosaraju's Algorithm?
Kosaraju's Algorithm finds all Strongly Connected Components in a directed graph.

It uses two DFS passes and a reversed graph.

9.2 Three Main Steps
Step 1: First DFS

Perform DFS on the original graph.

After completing all neighbours of a vertex, push it into a stack.

This records finishing order.

Step 2: Reverse the Graph

Reverse every directed edge.

For example:


```text
Original:

0 ---> 1

1 ---> 2
Reversed:


```text
0 <--- 1

1 <--- 2
Step 3: Second DFS

Process vertices by popping them from the finishing-order stack.

Perform DFS on the reversed graph.

Each DFS traversal identifies one SCC.

9.3 Java Implementation

java
import java.util.*;

public class Kosaraju {

    static void dfs1(
        int vertex,
        ArrayList<ArrayList<Integer>> graph,
        boolean[] visited,
        Stack<Integer> stack
    ) {

        visited[vertex] = true;

        for (int neighbour : graph.get(vertex)) {

            if (!visited[neighbour]) {

                dfs1(
                    neighbour,
                    graph,
                    visited,
                    stack
                );
            }
        }

        stack.push(vertex);
    }

    static void dfs2(
        int vertex,
        ArrayList<ArrayList<Integer>> reversed,
        boolean[] visited
    ) {

        visited[vertex] = true;

        System.out.print(vertex + " ");

        for (int neighbour : reversed.get(vertex)) {

            if (!visited[neighbour]) {

                dfs2(neighbour, reversed, visited);
            }
        }
    }

    static void findSCC(
        ArrayList<ArrayList<Integer>> graph,
        int vertices
    ) {

        boolean[] visited = new boolean[vertices];

        Stack<Integer> stack = new Stack<>();

        // First DFS

        for (int i = 0; i < vertices; i++) {

            if (!visited[i]) {

                dfs1(i, graph, visited, stack);
            }
        }

        // Reverse graph

        ArrayList<ArrayList<Integer>> reversed =
            new ArrayList<>();

        for (int i = 0; i < vertices; i++) {

            reversed.add(new ArrayList<>());
        }

        for (int i = 0; i < vertices; i++) {

            for (int neighbour : graph.get(i)) {

                reversed.get(neighbour).add(i);
            }
        }

        Arrays.fill(visited, false);

        int component = 1;

        // Second DFS

        while (!stack.isEmpty()) {

            int vertex = stack.pop();

            if (!visited[vertex]) {

                System.out.print(
                    "SCC " + component + ": "
                );

                dfs2(vertex, reversed, visited);

                System.out.println();

                component++;
            }
        }
    }

    public static void main(String[] args) {

        int vertices = 5;

        ArrayList<ArrayList<Integer>> graph =
            new ArrayList<>();

        for (int i = 0; i < vertices; i++) {

            graph.add(new ArrayList<>());
        }

        graph.get(0).add(1);

        graph.get(1).add(2);

        graph.get(2).add(0);

        graph.get(1).add(3);

        graph.get(3).add(4);

        graph.get(4).add(3);

        findSCC(graph, vertices);
    }
}
Output

```text
SCC 1: 0 2 1

SCC 2: 3 4
9.4 Complexity

```text
Time Complexity = O(V + E)

Space Complexity = O(V + E)
10. TARJAN'S ALGORITHM – INTRODUCTION
Tarjan's Algorithm is another algorithm for finding Strongly Connected Components.

Unlike Kosaraju's Algorithm, it uses a single DFS traversal.

It maintains two important arrays:

Discovery Time
The discovery time indicates when a vertex is first visited during DFS.

Low-Link Value
The low-link value represents the earliest discovery time reachable from a vertex using DFS tree edges and at most one back edge.

Tarjan also uses a stack to track active vertices.

When:


```text
low[u] == discovery[u]
Vertex u is the root of an SCC.

The algorithm pops vertices from the stack until u is removed.

10.1 Kosaraju vs Tarjan

Feature	Kosaraju	Tarjan
DFS Passes	Two	One
Reversed Graph	Required	Not required
Stack	Yes	Yes
Discovery Array	No	Yes
Low-Link Array	No	Yes
Complexity	O(V+E)	O(V+E)
Important:

Tarjan's algorithm also has applications in finding bridges and articulation points, although the exact low-link conditions differ between these problems.

11. BRIDGES IN A GRAPH
11.1 What is a Bridge?
A Bridge is an edge whose removal increases the number of connected components in an undirected graph.

Example:


```text
0 ---- 1 ---- 2 ---- 3
If we remove edge:


```text
1 ---- 2
The graph becomes disconnected.

Therefore, edge (1,2) is a bridge.

11.2 Another Example

```text
    0
   / \
  1---2
      |
      3
Edges:


```text
0-1

0-2

1-2

2-3
The triangle provides alternate paths between 0, 1 and 2.

But removing edge 2-3 disconnects vertex 3.

Therefore:


```text
Bridge = (2,3)
11.3 Discovery and Low Arrays
Bridge detection uses DFS.

Maintain:


```text
disc[u] = Discovery time of vertex u

low[u] = Earliest reachable discovery time
For a DFS tree edge from u to v:


```text
If low[v] > disc[u]
Then:


```text
(u, v) is a bridge.
Why?

Because the subtree rooted at v cannot reach u or any ancestor of u through a back edge.

11.4 Java Implementation

java
import java.util.*;

public class Bridges {

    static int timer = 0;

    static void dfs(
        int u,
        int parent,
        ArrayList<ArrayList<Integer>> graph,
        boolean[] visited,
        int[] disc,
        int[] low
    ) {

        visited[u] = true;

        disc[u] = low[u] = ++timer;

        for (int v : graph.get(u)) {

            if (v == parent) {

                continue;
            }

            if (!visited[v]) {

                dfs(
                    v,
                    u,
                    graph,
                    visited,
                    disc,
                    low
                );

                low[u] = Math.min(low[u], low[v]);

                if (low[v] > disc[u]) {

                    System.out.println(
                        "Bridge: " + u + " - " + v
                    );
                }

            } else {

                low[u] = Math.min(low[u], disc[v]);
            }
        }
    }

    static void findBridges(
        ArrayList<ArrayList<Integer>> graph,
        int vertices
    ) {

        boolean[] visited = new boolean[vertices];

        int[] disc = new int[vertices];

        int[] low = new int[vertices];

        timer = 0;

        for (int i = 0; i < vertices; i++) {

            if (!visited[i]) {

                dfs(
                    i,
                    -1,
                    graph,
                    visited,
                    disc,
                    low
                );
            }
        }
    }

    public static void main(String[] args) {

        int vertices = 4;

        ArrayList<ArrayList<Integer>> graph =
            new ArrayList<>();

        for (int i = 0; i < vertices; i++) {

            graph.add(new ArrayList<>());
        }

        graph.get(0).add(1);
        graph.get(1).add(0);

        graph.get(0).add(2);
        graph.get(2).add(0);

        graph.get(1).add(2);
        graph.get(2).add(1);

        graph.get(2).add(3);
        graph.get(3).add(2);

        findBridges(graph, vertices);
    }
}
Output

```text
Bridge: 2 - 3
Time Complexity:


```text
O(V + E)
Note: This implementation assumes a simple undirected graph without parallel edges. For multigraphs, track the parent edge ID rather than skipping every edge to the parent vertex.

12. ARTICULATION POINTS
12.1 What is an Articulation Point?
An Articulation Point is a vertex whose removal increases the number of connected components in an undirected graph.

Example:


```text
0 ---- 1 ---- 2 ---- 3
If vertex 1 is removed:


```text
0

2 ---- 3
The graph becomes disconnected.

Therefore, vertex 1 is an articulation point.

12.2 Difference Between Bridge and Articulation Point

Bridge	Articulation Point
An edge	A vertex
Removing it disconnects the graph further	Removing it disconnects the graph further
Edge-based concept	Vertex-based concept
Uses discovery and low arrays	Uses discovery and low arrays
12.3 Articulation Point Conditions
For a non-root DFS vertex u:

If there exists a DFS child v such that:


```text
low[v] >= disc[u]
Then u is an articulation point.

For the DFS root:

If the root has more than one DFS child, it is an articulation point.

The root rule is different from the non-root rule.

13. GRAPH COLORING
13.1 What is Graph Coloring?
Graph Coloring is the process of assigning colors to vertices so that no two adjacent vertices have the same color.

Example:


```text
    A
   / \
  B---C
Since every vertex is connected to the other two, we need three different colors.


```text
A = Red

B = Blue

C = Green
13.2 Chromatic Number
The Chromatic Number is the minimum number of colors required to color a graph so that adjacent vertices have different colors.

Notation:


```text
χ(G)
Examples:


Graph	Chromatic Number
Empty graph with vertices	1, if at least one vertex
Bipartite graph with at least one edge	2
Triangle	3
Complete graph K4	4
Even cycle	2
Odd cycle	3
13.3 Applications
Exam timetable scheduling.
Register allocation in compilers.
Frequency assignment.
Map coloring.
Resource allocation.
13.4 Greedy Graph Coloring
A simple greedy algorithm assigns the first available color to each vertex.


java
import java.util.*;

public class GraphColoring {

    static void colorGraph(int[][] graph, int vertices) {

        int[] color = new int[vertices];

        Arrays.fill(color, -1);

        color[0] = 0;

        for (int u = 1; u < vertices; u++) {

            boolean[] available = new boolean[vertices];

            Arrays.fill(available, true);

            for (int v = 0; v < vertices; v++) {

                if (graph[u][v] == 1 &&
                    color[v] != -1) {

                    available[color[v]] = false;
                }
            }

            for (int c = 0; c < vertices; c++) {

                if (available[c]) {

                    color[u] = c;

                    break;
                }
            }
        }

        for (int i = 0; i < vertices; i++) {

            System.out.println(
                "Vertex " + i +
                " : Color " + color[i]
            );
        }
    }

    public static void main(String[] args) {

        int[][] graph = {

            {0, 1, 1},

            {1, 0, 1},

            {1, 1, 0}
        };

        colorGraph(graph, 3);
    }
}
Output

```text
Vertex 0 : Color 0

Vertex 1 : Color 1

Vertex 2 : Color 2
Important: Greedy coloring does not always find the minimum possible number of colors. Its result can depend on vertex ordering.

14. BIPARTITE GRAPH APPLICATIONS
14.1 What is a Bipartite Graph?
A graph is bipartite if its vertices can be divided into two disjoint sets such that every edge connects vertices belonging to different sets.

Example:


```text
Set A          Set B

  0 ------------ 1

  2 ------------ 3

  0 ------------ 3

  2 ------------ 1
No edge connects two vertices in the same set.

14.2 Important Property
An undirected graph is bipartite if and only if it contains no odd-length cycle.

Examples:


Graph	Bipartite?
Tree	Yes
Even cycle	Yes
Odd cycle	No
Triangle	No
Square	Yes
14.3 Applications
Job assignment.
Student-project allocation.
Matching problems.
Employee-task assignment.
Dating and recommendation systems.
14.4 Maximum Bipartite Matching
A matching is a collection of edges in which no two selected edges share a vertex.

Example:

Suppose:


```text
Students = {S1, S2}

Projects = {P1, P2}
Connections:


```text
S1 -> P1

S1 -> P2

S2 -> P1
A maximum matching is:


```text
S1 -> P2

S2 -> P1
Both students get different projects.

This is a maximum bipartite matching of size 2.

A common algorithm for bipartite matching is the Kuhn algorithm, which uses DFS to find augmenting paths.

15. GRAPH CLONING
15.1 Problem Statement
You are given a reference to a node in a connected undirected graph.

Create a deep copy of the entire graph.

A deep copy means:

Every original vertex must have a new corresponding vertex.
Connections must be preserved.
No cloned vertex should reference an original vertex.
15.2 Why Do We Need a HashMap?
A graph may contain cycles.

If we recursively clone neighbours without remembering already cloned nodes, the algorithm can enter an infinite recursion.

We use:


java
HashMap<Node, Node>
The map stores:


```text
Original Node -> Cloned Node
15.3 Java Implementation

java
import java.util.*;

class Node {

    int val;

    List<Node> neighbors;

    Node(int val) {

        this.val = val;

        neighbors = new ArrayList<>();
    }
}

public class CloneGraph {

    HashMap<Node, Node> map = new HashMap<>();

    Node cloneGraph(Node node) {

        if (node == null) {

            return null;
        }

        if (map.containsKey(node)) {

            return map.get(node);
        }

        Node clone = new Node(node.val);

        map.put(node, clone);

        for (Node neighbour : node.neighbors) {

            clone.neighbors.add(
                cloneGraph(neighbour)
            );
        }

        return clone;
    }
}
Time Complexity:


```text
O(V + E)
Space Complexity:


```text
O(V)
16. WORD LADDER PROBLEM
16.1 Problem Statement
You are given:


```text
beginWord = "hit"

endWord = "cog"
Dictionary:


```text
hot

dot

dog

lot

log

cog
Find the shortest transformation sequence from hit to cog.

Only one character can be changed at a time.

Every intermediate word must exist in the dictionary.

16.2 Example Transformation

```text
hit
 |
 v
hot
 |
 v
dot
 |
 v
dog
 |
 v
cog
Total number of words:


```text
5
The shortest transformation length is 5.

16.3 Which Algorithm Is Used?
BFS.

Why?

Every transformation has equal cost.

Therefore, BFS finds the minimum number of transformations.

16.4 Java Implementation

java
import java.util.*;

public class WordLadder {

    static int ladderLength(
        String beginWord,
        String endWord,
        List<String> wordList
    ) {

        Set<String> dictionary =
            new HashSet<>(wordList);

        if (!dictionary.contains(endWord)) {

            return 0;
        }

        Queue<String> queue = new LinkedList<>();

        queue.add(beginWord);

        int level = 1;

        while (!queue.isEmpty()) {

            int size = queue.size();

            for (int i = 0; i < size; i++) {

                String word = queue.poll();

                if (word.equals(endWord)) {

                    return level;
                }

                char[] chars = word.toCharArray();

                for (int j = 0; j < chars.length; j++) {

                    char original = chars[j];

                    for (char c = 'a'; c <= 'z'; c++) {

                        chars[j] = c;

                        String next =
                            new String(chars);

                        if (dictionary.contains(next)) {

                            queue.add(next);

                            dictionary.remove(next);
                        }
                    }

                    chars[j] = original;
                }
            }

            level++;
        }

        return 0;
    }

    public static void main(String[] args) {

        List<String> words = Arrays.asList(

            "hot",

            "dot",

            "dog",

            "lot",

            "log",

            "cog"
        );

        System.out.println(
            ladderLength("hit", "cog", words)
        );
    }
}
Output

```text
5
Time Complexity of this straightforward implementation:


```text
O(N × L × 26 × L)
Where:

N = Number of dictionary words.
L = Length of each word.
This can be improved using wildcard-pattern indexing or bidirectional BFS.

17. NETWORK DELAY TIME
17.1 Problem Statement
Suppose a network contains N nodes.

A signal starts from node K.

Every edge has a transmission time.

Find the minimum time required for the signal to reach every node.

If any node cannot receive the signal, return -1.

17.2 Which Algorithm Is Used?
Dijkstra's Algorithm.

Why?

All edge weights are non-negative.
We need shortest distances from one source.
We need the maximum of the shortest distances.
Formula:


```text
Answer = Maximum shortest distance
17.3 Example

```text
0 -> 1 = 2

0 -> 2 = 5

1 -> 2 = 1
Source:


```text
K = 0
Shortest distances:


```text
0 = 0

1 = 2

2 = 3
Network delay:


```text
Maximum = 3
18. ADVANCED GRAPH ALGORITHM COMPARISON

Algorithm	Purpose	Technique	Complexity
Topological Sort	Dependency ordering	DFS/BFS	O(V+E)
Kahn's Algorithm	Topological sort and cycle detection	BFS	O(V+E)
Kosaraju	Strongly connected components	Two DFS passes	O(V+E)
Tarjan	Strongly connected components	One DFS	O(V+E)
Bridge Detection	Find critical edges	DFS + Low Link	O(V+E)
Articulation Points	Find critical vertices	DFS + Low Link	O(V+E)
Graph Coloring	Assign valid colors	Greedy/backtracking	Depends on algorithm
Graph Cloning	Deep copy graph	DFS + HashMap	O(V+E)
Word Ladder	Shortest word transformation	BFS	O(N × L × 26 × L) for basic implementation
Network Delay	Minimum signal propagation time	Dijkstra	O((V+E) log V)
````
