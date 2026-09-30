# Question 6: Social Network using Adjacency Matrix and Adjacency List

## 1. Problem Statement
A small social network has 6 people (vertices) A to F with these connections (edges):

`A-B, A-C, B-D, B-E, C-F, E-F`

- (a) Implement the network using an Adjacency Matrix and an Adjacency List. Perform BFS and DFS from vertex A.
- (b) Implement a search operation to locate a vertex using both representations and record the operations required.
- (c) Compare the two representations (space, traversal, search/edge checking, time complexity) and decide which is more suitable for a sparse social network.

The graph is undirected, so every edge is stored in both directions.

## 2. Input Data
Vertices: `A B C D E F` (index 0 to 5)

| Edge No. | Edge |
|---|---|
| 1 | A - B |
| 2 | A - C |
| 3 | B - D |
| 4 | B - E |
| 5 | C - F |
| 6 | E - F |

V = 6, E = 6.
Vertex searches: A, B, C, D, E, F. Edge checks: A-B, E-F, D-F.

## 3. Source Code (C)
Operation counting: one operation = one matrix cell check, one list node visit, or one vertex comparison (in search).

```c
#include <stdio.h>
#include <stdlib.h>

#define V 6
#define E 6

char names[V] = {'A', 'B', 'C', 'D', 'E', 'F'};
int edges[E][2] = {{0,1}, {0,2}, {1,3}, {1,4}, {2,5}, {4,5}};

int matrix[V][V], visited[V], ops, mode;
struct Node { int v; struct Node *next; } *head[V], *tail[V];

void addToList(int a, int b) {
    struct Node *n = calloc(1, sizeof(struct Node));
    n->v = b;
    if (head[a]) tail[a]->next = n;
    else head[a] = n;
    tail[a] = n;
}

int neighbours(int u, int out[]) {
    int n = 0;
    if (mode == 0) {
        for (int k = 0; k < V; k++) {
            ops++;
            if (matrix[u][k]) out[n++] = k;
        }
    } else {
        for (struct Node *p = head[u]; p; p = p->next) {
            ops++;
            out[n++] = p->v;
        }
    }
    return n;
}

int bfs(int target) {
    int queue[V], nb[V], front = 0, rear = 0;
    for (int i = 0; i < V; i++) visited[i] = 0;
    ops = 0;
    visited[0] = 1;
    queue[rear++] = 0;
    while (front < rear) {
        int u = queue[front++];
        if (target < 0) printf("%c ", names[u]);
        else {
            ops++;
            if (u == target) return ops;
        }
        int n = neighbours(u, nb);
        for (int i = 0; i < n; i++)
            if (!visited[nb[i]]) {
                visited[nb[i]] = 1;
                queue[rear++] = nb[i];
            }
    }
    return ops;
}

void dfs(int u) {
    int nb[V];
    visited[u] = 1;
    printf("%c ", names[u]);
    int n = neighbours(u, nb);
    for (int i = 0; i < n; i++)
        if (!visited[nb[i]]) dfs(nb[i]);
}

int hasEdge(int a, int b) {
    if (mode == 0) { ops = 1; return matrix[a][b]; }
    ops = 0;
    for (struct Node *p = head[a]; p; p = p->next) {
        ops++;
        if (p->v == b) return 1;
    }
    return 0;
}

int main() {
    char *label[2] = {"Matrix: ", "List:   "};

    for (int i = 0; i < E; i++) {
        int a = edges[i][0], b = edges[i][1];
        matrix[a][b] = matrix[b][a] = 1;
        addToList(a, b);
        addToList(b, a);
    }

    printf("Adjacency Matrix\n   ");
    for (int i = 0; i < V; i++) printf("%c ", names[i]);
    for (int i = 0; i < V; i++) {
        printf("\n%c  ", names[i]);
        for (int j = 0; j < V; j++) printf("%d ", matrix[i][j]);
    }

    printf("\n\nAdjacency List\n");
    for (int i = 0; i < V; i++) {
        printf("%c -> ", names[i]);
        for (struct Node *p = head[i]; p; p = p->next) printf("%c ", names[p->v]);
        printf("\n");
    }

    printf("\nBFS from A\n");
    for (mode = 0; mode < 2; mode++) {
        printf("%s", label[mode]);
        bfs(-1);
        printf("| operations = %d\n", ops);
    }

    printf("\nDFS from A\n");
    for (mode = 0; mode < 2; mode++) {
        for (int i = 0; i < V; i++) visited[i] = 0;
        ops = 0;
        printf("%s", label[mode]);
        dfs(0);
        printf("| operations = %d\n", ops);
    }

    printf("\nSearch a vertex (operations required)\n");
    for (int t = 0; t < V; t++) {
        mode = 0; int m = bfs(t);
        mode = 1; int l = bfs(t);
        printf("Search %c: Matrix = %2d, List = %2d\n", names[t], m, l);
    }

    printf("\nEdge check (operations required)\n");
    int pairs[3][2] = {{0,1}, {4,5}, {3,5}};
    for (int i = 0; i < 3; i++) {
        int a = pairs[i][0], b = pairs[i][1];
        mode = 0; int rm = hasEdge(a, b), om = ops;
        mode = 1; int rl = hasEdge(a, b), ol = ops;
        printf("Edge %c-%c: Matrix = %d (%d op), List = %d (%d ops)\n",
               names[a], names[b], rm, om, rl, ol);
    }

    printf("\nSpace used\nMatrix: %d cells\nList:   %d nodes\n", V * V, 2 * E);
    return 0;
}
```

Compile and run:
```
gcc graph.c -o graph
./graph
```

## 4. Output
```
Adjacency Matrix
   A B C D E F 
A  0 1 1 0 0 0 
B  1 0 0 1 1 0 
C  1 0 0 0 0 1 
D  0 1 0 0 0 0 
E  0 1 0 0 0 1 
F  0 0 1 0 1 0 

Adjacency List
A -> B C 
B -> A D E 
C -> A F 
D -> B 
E -> B F 
F -> C E 

BFS from A
Matrix: A B C D E F | operations = 36
List:   A B C D E F | operations = 12

DFS from A
Matrix: A B D E F C | operations = 36
List:   A B D E F C | operations = 12

Search a vertex (operations required)
Search A: Matrix =  1, List =  1
Search B: Matrix =  8, List =  4
Search C: Matrix = 15, List =  8
Search D: Matrix = 22, List = 11
Search E: Matrix = 29, List = 13
Search F: Matrix = 36, List = 16

Edge check (operations required)
Edge A-B: Matrix = 1 (1 op), List = 1 (1 ops)
Edge E-F: Matrix = 1 (1 op), List = 1 (2 ops)
Edge D-F: Matrix = 0 (1 op), List = 0 (1 ops)

Space used
Matrix: 36 cells
List:   12 nodes
```

## 5. Graph Representations

Adjacency Matrix:

| | A | B | C | D | E | F |
|---|---|---|---|---|---|---|
| A | 0 | 1 | 1 | 0 | 0 | 0 |
| B | 1 | 0 | 0 | 1 | 1 | 0 |
| C | 1 | 0 | 0 | 0 | 0 | 1 |
| D | 0 | 1 | 0 | 0 | 0 | 0 |
| E | 0 | 1 | 0 | 0 | 0 | 1 |
| F | 0 | 0 | 1 | 0 | 1 | 0 |

Adjacency List:

| Vertex | Neighbours |
|---|---|
| A | B, C |
| B | A, D, E |
| C | A, F |
| D | B |
| E | B, F |
| F | C, E |

## 6. Trace Tables

### 6.1 BFS from A (uses a queue)

| Step | Dequeued | Added to queue | Queue after step | Visited order |
|---|---|---|---|---|
| 0 | - | A | A | - |
| 1 | A | B, C | B, C | A |
| 2 | B | D, E | C, D, E | A B |
| 3 | C | F | D, E, F | A B C |
| 4 | D | none | E, F | A B C D |
| 5 | E | none | F | A B C D E |
| 6 | F | none | empty | A B C D E F |

BFS order: **A B C D E F**

### 6.2 DFS from A (uses recursion / stack)

| Step | Current vertex | Action | Stack | Visited order |
|---|---|---|---|---|
| 1 | A | Visit A, go to B | A | A |
| 2 | B | Visit B, go to D | A B | A B |
| 3 | D | Visit D, no new neighbour, go back | A B D | A B D |
| 4 | B | Next unvisited neighbour is E | A B | A B D |
| 5 | E | Visit E, go to F | A B E | A B D E |
| 6 | F | Visit F, go to C | A B E F | A B D E F |
| 7 | C | Visit C, no new neighbour, go back | A B E F C | A B D E F C |
| 8 | - | Go back through F, E, B, A. All done | empty | A B D E F C |

DFS order: **A B D E F C**

### 6.3 Search Operation Trace (BFS from A, stop when found)

Search for F:

| Representation | Work done | Total operations |
|---|---|---|
| Matrix | Compare A + scan row A (6), compare B + scan row B (6), compare C + scan row C (6), compare D + scan row D (6), compare E + scan row E (6), compare F | 6 + 5 x 6 = **36** |
| List | Compare A + 2 nodes, compare B + 3 nodes, compare C + 2 nodes, compare D + 1 node, compare E + 2 nodes, compare F | 6 + 10 = **16** |

All searches:

| Target | Matrix operations | List operations |
|---|---|---|
| A | 1 | 1 |
| B | 8 | 4 |
| C | 15 | 8 |
| D | 22 | 11 |
| E | 29 | 13 |
| F | 36 | 16 |

### 6.4 Edge Check Operations

| Edge | Exists | Matrix operations | List operations |
|---|---|---|---|
| A-B | Yes | 1 | 1 |
| E-F | Yes | 1 | 2 |
| D-F | No | 1 | 1 |

## 7. Complexity Analysis
V = number of vertices, E = number of edges, deg(u) = number of neighbours of u.

| Operation | Adjacency Matrix | Adjacency List |
|---|---|---|
| Space complexity | O(V^2) | O(V + E) |
| BFS / DFS time | O(V^2) | O(V + E) |
| Search a vertex (traversal based) | O(V^2) | O(V + E) |
| Edge check (u, v) | O(1) | O(deg(u)) |
| Find all neighbours of u | O(V) | O(deg(u)) |

Reasons:
- Matrix traversal scans one full row of V cells for each of the V vertices, so V x V = 36 checks.
- List traversal visits every edge from both ends, so 2E = 12 node visits.
- The matrix stores V x V = 36 cells. The list stores 2E = 12 nodes.

## 8. Comparison Table

| Criterion | Adjacency Matrix | Adjacency List |
|---|---|---|
| Space used | 36 cells | 12 nodes |
| Space complexity | O(V^2) | O(V + E) |
| BFS operations | 36 | 12 |
| DFS operations | 36 | 12 |
| BFS order | A B C D E F | A B C D E F |
| DFS order | A B D E F C | A B D E F C |
| Search for F | 36 operations | 16 operations |
| Search for D | 22 operations | 11 operations |
| Edge check E-F | 1 operation | 2 operations |
| Traversal time complexity | O(V^2) | O(V + E) |

Both representations give the same BFS and DFS order because neighbours are visited in the same alphabetical order.

## 9. Final Conclusion
A social network is sparse: each person is connected to only a few others, so E is much smaller than V^2. In the execution results, the adjacency list needed 12 operations for a full BFS or DFS, while the matrix needed 36. Searching for vertex F took 16 operations with the list and 36 with the matrix. The list also uses O(V + E) space instead of O(V^2).

The matrix is only better for checking a single edge (1 operation), but this small advantage does not make up for the wasted space and time spent scanning zero entries.

**Therefore, the Adjacency List is the more suitable representation for a sparse social network.**
