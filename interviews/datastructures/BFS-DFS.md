## DFS/BFS

- Algorithms (Time Complexity) (recursive vs iterative)
- Problem spaces where this is applicable
- When to use BFS over DFS and vice versa

### DFS

#### Applications
1. *Pathfinding in Weighted Graphs*:
DFS can be useful when you want to explore all possible paths, especially in combination with techniques like backtracking. For example, finding all paths from a source to a destination.
Example:
In puzzles like the N-Queens problem, you use DFS to explore all possible arrangements.

2. *Topological Sorting*:
DFS is ideal for topological sorting in Directed Acyclic Graphs (DAGs). You explore each node fully before moving to another, making it suitable for dependency resolution problems.

3. *Solving Problems on Trees*:
DFS is commonly used for solving problems on trees, such as finding the maximum depth of a binary tree, checking if two trees are identical, or finding paths that sum to a certain value.

4. *Backtracking*:
DFS is the basis for backtracking algorithms where you explore one path deeply before backtracking and trying another. This is useful for problems like solving Sudoku, generating permutations, or finding combinations.

5. *Cycle Detection in Graphs*:
DFS is often used to detect cycles in directed or undirected graphs. In directed graphs, you can keep track of the recursion stack to identify cycles.

*Advantages of DFS*:
Space Efficiency: DFS generally requires less memory compared to BFS because it only needs to store the current path. For a graph with depth d, DFS typically requires O(d) space.
DFS can be more efficient when searching in deep or infinite graphs.

*Disadvantages of DFS*:
Not Guaranteed to Find the Shortest Path: DFS explores deeply first, so it may find a solution that is not the shortest. In weighted graphs, this might lead to suboptimal solutions unless combined with other techniques (e.g., iterative deepening DFS).

#### Recursive

```
  dfs(root) {  
    if (root == null) {
      return;
    }
    // condition e.g. sum+=root.value;
    dfs(root.left);
    dfs(root.right);
  }

  // dfs Graph
  dfs (Graph graph) {
    if (graph == null) {
      return;
    }
    Map<Node, Boolean> visitedMap = new HashMap<>();
    for (Node node = graph.nodes()) {
      if (!visitedMap.containsKey(node)) {
        dfs(node, graph, visitedMap);
      }
    }
  }

  dfs(Node node, Graph graph, Map<Node, Boolean> visitedMap) {
    if (node == null || graph == null) {
      return;
    }
    visitedMap.put(node, True);
    for (Node adjNode = graph.get(node).adjList()) {
        if (!visitedMap.containsKey(adjNode)) {
          dfs(adjNode, graph, visitedMap);
        }
    }
  }
```

#### Iterative
```
  dfs(root) {
    if (root == null) {
      return;
    }
    Stack<TreeNode> stack = new Stack<>();
    stack.push(root);
    while(!stack.isEmpty()) {
      TreeNode curNode = stack.pop();
      //condition e..g sum+=curNode.value;
      if (root.right) {
        stack.push(root.right);
      }
      if (root.left) {
        stack.push(root.left);
      }
    }
  }
```

### BFS

#### Applications/When to use BFS
1. *Finding the Shortest Path in an Unweighted Graph*:
- BFS explores nodes level by level, so the first time it reaches the target node, it is guaranteed to have found the shortest path in an unweighted graph.
Example:
- Finding the shortest path between two points in an unweighted grid or maze.
Network broadcast where each node represents a computer, and you want to reach every other computer in the minimum number of steps.

2. *Level-Order Traversal*:
When you need to process nodes level by level, BFS is ideal.
Example:
- Level-order traversal of a binary tree.

3. *Connected Components in an Unweighted Graph*:
BFS is useful for exploring all nodes connected to a starting node and marking them as visited.

4. *Finding the Minimum Moves or Steps*:
In problems like finding the minimum moves required in a board game (like snakes and ladders), BFS is typically used because it explores all possible moves at each step before moving on to deeper levels.

*Advantages of BFS*:
Guarantees finding the shortest path in an unweighted graph.
Explores nodes level by level, ensuring you find the solution with the minimum number of steps if one exists.

*Disadvantages of BFS*:
Space Complexity: BFS requires storing all nodes at the current level in the queue, which can lead to high memory usage, especially for wide graphs or deep trees. 
For example, BFS on a binary tree can use up to O(2^d) space, where d is the depth of the tree.

#### Recursive
```

  // BFS with Graph
  bfs(Graph graph, Map<Node,Boolean> discoveredMap, Queue queue) {
    if (queue.isEmpty()) {
      return;
    }
    Node curNode = queue.poll();
    for (Node node : graph.get(curNode).getAdjList()) {
      if (!discoveredMap.get(node)) {
       
        queue.offer(node);
        discoveredMap.put(node, True);
      }
    }
  }
```

#### Iterative
```
  bfs(root) {
    if (root == null) {
      return;
    }
    Queue<TreeNode> queue = new Queue<>();
    queue.offer(root);
    while(!queue.isEmpty()) {
      int qLength = queue.size();
      for (int i=0;i<qLength;i++) {
        TreeNode curNode = queue.poll();
        // process curNode
        if (curNode.left) {
          queue.offer(curNode.left);
        }
        if (curNode.right) {
          queue.offer(curNode.right);
        }
      }
    }
  }
```

### When to Use BFS over DFS:
- *Shortest Path in an Unweighted Graph*: If you need the shortest path, BFS is preferable because it finds the shortest path to the target node in an unweighted graph.
- *Exploring Nodes by Levels*: When the problem requires exploration of nodes level by level (e.g., solving a word ladder problem), BFS is more appropriate.
- *Guaranteed Solution in Shallow Trees*: When the solution is likely to be close to the root, BFS will find it faster since it explores nodes level by level.

### When to Use DFS over BFS:
- *Deep Exploration or Backtracking*: If your problem requires exploring paths deeply or trying all possible solutions (e.g., finding all possible permutations), DFS is the better choice.
- *Topological Sorting and Dependency Resolution*: For problems where you need to visit all nodes before visiting certain other nodes (e.g., scheduling tasks based on dependencies), DFS is preferable.
- *Memory Constraints*: If memory usage is a concern, DFS might be a better option, especially for large or infinite graphs where BFS would consume too much memory.

### When Not to Use BFS or DFS:
- *Weighted Graphs for Shortest Path Problems*: Neither BFS nor DFS is optimal for finding the shortest path in a weighted graph. Instead, you should use algorithms like Dijkstra's or A* for such problems.
- *Heuristic-Based Search Problems*: If the problem benefits from heuristics (e.g., solving puzzles like the 8-puzzle problem), algorithms like A* or Greedy Best-First Search are more suitable because they use heuristic information to guide the search.
- *Complex Optimization Problems*: For complex optimization problems like the Traveling Salesman Problem, BFS and DFS are generally not suitable due to their exhaustive nature. Dynamic programming, branch and bound, or heuristic-based methods like genetic algorithms might be more appropriate.
