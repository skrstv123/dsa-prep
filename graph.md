# 🕸️ Graph Theory & BFS/DFS Patterns

Notes on common patterns for technical interviews involving grids, dependencies, and pathfinding.

---

## 1. Grid Traversal (Islands Pattern) 🏝️
**Problem:** *Number of Islands*
**Pattern:** Treat a 2D matrix as a graph. Each cell $(r, c)$ is a node; its 4 neighbors (Up, Down, Left, Right) are edges.

### 🔑 Key Concepts
* **Visited Set:** Essential to prevent infinite loops. Use a `set()` or a `visited[r][c]` boolean grid.
* **Boundary Checks:** Always verify $0 \le r < R$ and $0 \le c < C$ before accessing the grid.
* **DFS Strategy:** Useful for "coloring" or counting connected components.

### 💻 Solution
```python
class Solution:
    def numIslands(self, grid: List[List[str]]) -> int:
        if not grid: return 0
        rows, cols = len(grid), len(grid[0])
        visited = set()
        moves = [(1, 0), (-1, 0), (0, 1), (0, -1)]

        def dfs(r, c):
            if (r < 0 or r >= rows or c < 0 or c >= cols or 
                grid[r][c] == '0' or (r, c) in visited):
                return
            visited.add((r, c))
            for dr, dc in moves:
                dfs(r + dr, c + dc)

        islands = 0
        for r in range(rows):
            for c in range(cols):
                if grid[r][c] == '1' and (r, c) not in visited:
                    islands += 1
                    dfs(r, c)
        return islands

```

---

## 2. Multi-Source BFS (Time-Based Expansion) 🍊

**Problem:** *Rotting Oranges*
**Pattern:** When multiple sources expand at the same time, use **BFS**. Each "level" of the BFS represents one unit of time.

### 🔑 Key Concepts

* **Initial Queue:** Add ALL initial sources (rotten oranges) to the queue at once.
* **Layer Processing:** Use the size of the queue (or swapping lists) to process everything happening at  before moving to .
* **Final Check:** Use a counter for "fresh" items to see if any are unreachable.

### 💻 Solution

```python
class Solution:
    def orangesRotting(self, grid: List[List[int]]) -> int:
        rows, cols = len(grid), len(grid[0])
        fresh = 0
        queue = []
        
        for r in range(rows):
            for c in range(cols):
                if grid[r][c] == 1: fresh += 1
                if grid[r][c] == 2: queue.append((r, c))
        
        minutes = 0
        moves = [(1, 0), (-1, 0), (0, 1), (0, -1)]
        
        while queue and fresh > 0:
            minutes += 1
            next_level = []
            for r, c in queue:
                for dr, dc in moves:
                    nr, nc = r + dr, c + dc
                    if 0 <= nr < rows and 0 <= nc < cols and grid[nr][nc] == 1:
                        grid[nr][nc] = 2
                        fresh -= 1
                        next_level.append((nr, nc))
            queue = next_level
            
        return minutes if fresh == 0 else -1

```

---

## 3. Topological Sort (Dependency Ordering) 🛸

**Problem:** *Course Schedule / Alien Dictionary*
**Pattern:** For directed graphs where  means "A must come before B."

### 🔑 Key Concepts

* **In-degree Map:** Tracks how many prerequisites a node has.
* **Adjacency List:** Tracks which nodes are "unlocked" after a specific node is finished.
* **Kahn's Algorithm:**
1. Add all nodes with `In-degree == 0` to a queue.
2. While queue is not empty:
* Pop node, add to result.
* Decrement neighbors' in-degrees.
* If a neighbor hits 0, add to queue.
* **Cycle Detection:** If `len(result) != total_nodes`, there is a cycle (deadlock).

```python
from collections import deque, defaultdict

class Solution:
    def canFinish(self, numCourses: int, prerequisites: List[List[int]]) -> bool:
        adj = defaultdict(list)
        in_degree = [0] * numCourses
        for dest, src in prerequisites:
            adj[src].append(dest)
            in_degree[dest] += 1
            
        queue = deque([i for i in range(numCourses) if in_degree[i] == 0])
        count = 0
        while queue:
            curr = queue.popleft()
            count += 1
            for neighbor in adj[curr]:
                in_degree[neighbor] -= 1
                if in_degree[neighbor] == 0:
                    queue.append(neighbor)
        return count == numCourses
```

## 4. Advanced Topo Sort (Graph Building) 🛸
**Problem:** Alien Dictionary
**Pattern:** Build the graph manually by comparing adjacent sorted words.

### 🔑 Key Concepts
* First Difference: Only the first differing character between words gives ordering info.
* Prefix Check: If word1 is longer than word2 and contains it as a prefix, it's invalid.

### 💻 Solution (Alien Dictionary)

```python
from collections import defaultdict, deque

class Solution:
    def alienOrder(self, words: List[str]) -> str:
        # 1. Init maps
        adj = defaultdict(set)
        in_degree = {c: 0 for w in words for c in w}
        
        # 2. Build Graph (Compare adjacent pairs)
        for i in range(len(words) - 1):
            w1, w2 = words[i], words[i+1]
            min_len = min(len(w1), len(w2))
            if len(w1) > len(w2) and w1[:min_len] == w2[:min_len]:
                return "" # Invalid prefix order
            
            for j in range(min_len):
                if w1[j] != w2[j]:
                    if w2[j] not in adj[w1[j]]:
                        adj[w1[j]].add(w2[j])
                        in_degree[w2[j]] += 1
                    break
        
        # 3. BFS (Kahn's)
        queue = deque([c for c in in_degree if in_degree[c] == 0])
        res = []
        while queue:
            c = queue.popleft()
            res.append(c)
            for neighbor in adj[c]:
                in_degree[neighbor] -= 1
                if in_degree[neighbor] == 0:
                    queue.append(neighbor)
                    
        return "".join(res) if len(res) == len(in_degree) else ""

```
---

