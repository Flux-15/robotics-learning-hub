# 📄 A Formal Basis for the Heuristic Determination of Minimum Cost Paths (A* Algorithm)

<div align="center">

[ ⬅️ Back to Papers Index ](README.md) &nbsp;•&nbsp; [ 🏠 Hub Home ](../README.md)

</div>

---

## 📌 Metadata
* **Authors:** Peter E. Hart, Nils J. Nilsson, Bertram Raphael (Stanford Research Institute)
* **Publication:** IEEE Transactions on Systems Science and Cybernetics (1968)
* **Paper Link:** [IEEE Xplore](https://ieeexplore.ieee.org/document/4082128)
* **Core Contribution:** Formulation and mathematical proof of the **A* Search Algorithm**, which guarantees finding the shortest path on a graph while evaluating the minimal number of nodes using an admissible heuristic.

---

## 🔑 Mathematical Foundation

A* evaluates candidate nodes $n$ in a graph using the evaluation function:
$$f(n) = g(n) + h(n)$$

* $g(n)$: Exact cost incurred traveling from the start node to candidate node $n$.
* $h(n)$: Estimated heuristic cost to travel from candidate node $n$ to the goal node.
* $f(n)$: Estimated total cost of the cheapest path constrained to pass through node $n$.

### 1. Admissibility & Optimality
* **Definition of Admissibility:** A heuristic $h(n)$ is **admissible** if it never overestimates the true remaining cost to the goal $h^*(n)$:
  $$h(n) \le h^*(n), \quad \forall n$$
* **Optimality Theorem:** If $h(n)$ is admissible and graph edge costs are positive, tree-search A* is guaranteed to return an optimal (minimum-cost) path.
* **Consistency (Monotonicity):** For any node $n$ and neighbor $n'$ connected by an edge with step cost $c(n, n')$:
  $$h(n) \le c(n, n') + h(n')$$
  If $h(n)$ is consistent, graph-search A* never needs to reopen closed nodes, ensuring maximum search efficiency.

### 2. Common Heuristics in Robotics
* **Euclidean Distance ($L_2$ norm):** For holonomic mobile bases moving freely in 2D/3D continuous space:
  $$h(n) = \sqrt{(x_n - x_{\text{goal}})^2 + (y_n - y_{\text{goal}})^2}$$
* **Manhattan Distance ($L_1$ norm):** For 4-connected grid maps:
  $$h(n) = |x_n - x_{\text{goal}}| + |y_n - y_{\text{goal}}|$$
* **Octile / Diagonal Distance:** For 8-connected grid maps where diagonal movements cost $\sqrt{2}$.

---

## 💻 Algorithmic Workflow & Pseudocode

```python
import heapq

def a_star_search(graph, start, goal, heuristic_fn):
    # Priority Queue ordered by f(n) = g(n) + h(n)
    open_set = []
    heapq.heappush(open_set, (0 + heuristic_fn(start, goal), 0, start, [start]))
    
    g_costs = {start: 0}
    visited = set()

    while open_set:
        f_cost, g_cost, current, path = heapq.heappop(open_set)

        if current == goal:
            return path, g_cost

        if current in visited:
            continue
        visited.add(current)

        for neighbor, edge_weight in graph.get_neighbors(current):
            new_g = g_cost + edge_weight
            if neighbor not in g_costs or new_g < g_costs[neighbor]:
                g_costs[neighbor] = new_g
                f = new_g + heuristic_fn(neighbor, goal)
                heapq.heappush(open_set, (f, new_g, neighbor, path + [neighbor]))

    return None, float('inf') # No path found
```

---

## 🦾 Robotics Applications & Modern Extensions
* **Hybrid A* (Stanford Junior & Nav2 Smac Planner):** Extends A* to non-holonomic car-like vehicles (Ackermann steering). Discretizes continuous $(x, y, \theta)$ space into kinematic Reeds-Shepp and Dubins curves while guaranteeing collision-free drivable paths.
* **D* and D* Lite:** Dynamic replanning extensions used by Mars rovers to repair existing paths in-place as new obstacles are detected by onboard cameras without resolving from scratch.
