# Part 5: HNSW (Hierarchical Navigable Small World) - Complete Deep Dive

## Table of Contents
1. [Small World Networks](#small-world-networks)
2. [HNSW Design](#hnsw-design)
3. [Construction Algorithm](#construction-algorithm)
4. [Search Algorithm](#search-algorithm)
5. [Performance Analysis](#performance-analysis)

---

## Small World Networks

### The Foundation

**Small World Phenomenon**: "Six degrees of separation"
- Any two people connected by ~6 social links
- Despite 7 billion people on Earth!

### Graph Types

**1. Regular Lattice:**
```
O—O—O—O
|  |  |  |
O—O—O—O
```
- High clustering
- High diameter (many hops)

**2. Random Graph:**
```
O  ⟋O⟍  O
 ⟍ | ⟋
   O
```
- Low clustering
- Low diameter

**3. Small World:**
```
O—O—O—O
|  |⟋ |⟍
O—O  O—O
```
- High clustering + Low diameter
- Mostly local connections
- Few long-range shortcuts!

### Key Insight for HNSW

Small world graphs enable **EFFICIENT GREEDY SEARCH**:
1. Start at any node
2. Move to neighbor closer to target
3. Repeat until can't improve
4. **Arrive at target quickly!**

---

## HNSW Design

### The Breakthrough Idea

Instead of ONE graph, create **MULTIPLE LAYERS**:

```
Layer 2:  A ——————— D           ← Few nodes, long edges (HIGHWAY)
          |         |
Layer 1:  A — B — C — D         ← More nodes, medium edges
          |   |   |   |
Layer 0:  A—B—C—D—E—F—G—H      ← All nodes, local edges (STREETS)
```

### Analogy: Finding Address in City

1. **Highway** (top layer) - travel fast, far
2. **Main roads** (middle layers) - get closer
3. **Local streets** (bottom layer) - precise location

### Layer Assignment

Each node assigned layer l with probability:

```
P(l) = (1/M)^l × (1 - 1/M)
```

**Exponential decay:**
- Most nodes in layer 0 (bottom)
- Few nodes in higher layers
- Creates hierarchy automatically!

Expected number of layers: **O(log N)**

### Node A vs Node E

```
Node A: Appears in ALL layers (lucky!)
Node E: Only in layer 0 (most nodes)
```

---

## Construction Algorithm

### For New Node with Layer l

```
1. Start at entry point in top layer
2. Search from top layer down to layer l
3. At each layer:
   - Find M nearest neighbors
   - Connect bidirectionally
4. Update entry point if this node is highest
```

### Detailed Steps

**Step 1: Determine Random Layer**
```python
level = 0
while random() < 1/M and level < 50:
    level += 1
```

**Step 2: Search from Top to Target Layer**
```python
current_nearest = [entry_point]
for layer in range(max_layer, level, -1):
    current_nearest = search_layer(vector, current_nearest, 1, layer)
```

**Step 3: Connect at Each Layer**
```python
for layer in range(min(level, max_layer), -1, -1):
    candidates = search_layer(vector, current_nearest, ef_construction, layer)
    neighbors = select_neighbors_heuristic(candidates, M)
    
    for neighbor in neighbors:
        add_bidirectional_connection(node, neighbor, layer)
```

### Neighbor Selection Heuristic

Don't just take M closest! Instead:
- Prefer **diverse** neighbors
- Avoid clustering
- Better graph structure

```python
def select_neighbors_heuristic(candidates, M):
    selected = []
    for dist, idx in sorted(candidates):
        if len(selected) >= M:
            break
        # Add if far enough from already selected
        if not_too_close_to_selected(idx, selected):
            selected.append(idx)
    return selected
```

---

## Search Algorithm

### Goal: Find K Nearest Neighbors

```
1. Start at entry point in top layer
2. Greedy search to bottom of layer
3. Move down one layer
4. Repeat until layer 0
5. At layer 0, search for ef candidates
6. Return k closest
```

### Layer Search (Core Algorithm)

```python
def search_layer(query, entry_points, num_closest, layer):
    visited = set(entry_points)
    candidates = [(distance(query, p), p) for p in entry_points]
    best = list(candidates)
    
    while candidates:
        current_dist, current = pop_closest(candidates)
        
        if current_dist > worst_in_best(best):
            break  # Can't improve
        
        # Explore neighbors
        for neighbor in graph[layer][current]:
            if neighbor not in visited:
                visited.add(neighbor)
                dist = distance(query, neighbor)
                
                if dist < worst_in_best(best) or len(best) < num_closest:
                    push(candidates, (dist, neighbor))
                    push(best, (dist, neighbor))
                    keep_only_best_k(best, num_closest)
    
    return best
```

### Complete Search

```python
def search(query, k, ef=None):
    if ef is None:
        ef = max(k, ef_construction)
    
    # Start from entry point at top
    current_nearest = [entry_point]
    
    # Search from top layer down to layer 1
    for layer in range(max_layer, 0, -1):
        current_nearest = search_layer(query, current_nearest, 1, layer)
    
    # At layer 0, search for ef candidates
    candidates = search_layer(query, current_nearest, ef, 0)
    
    # Return top k
    return sorted(candidates, key=distance)[:k]
```

---

## Parameters

### M (max connections)

**Typical value: 16**

**Higher M:**
- ✓ Better recall
- ✓ More robust
- ✗ More memory
- ✗ Slower construction

**Lower M:**
- ✓ Less memory
- ✓ Faster construction
- ✗ Lower recall

### M_max (layer 0 connections)

**Typical value: 2×M = 32**

Layer 0 has more connections for better search quality.

### ef_construction

**Typical value: 200**

Size of dynamic candidate list during construction.

**Higher ef_construction:**
- ✓ Better quality index
- ✗ Slower construction

### ef (search)

**Typical value: ef ≥ k**

Size of dynamic candidate list during search.

**Higher ef:**
- ✓ Better recall
- ✗ Slower search

**Typical range: k to 3k**

---

## Performance Analysis

### Complexity

**Construction:**
```
Time: O(N × log N × M × dimension)
Space: O(N × M × log N)
```

**Search:**
```
Time: O(log N × M × dimension)
Space: O(1)
```

### Why O(log N)?

1. **O(log N) layers** (exponential decay)
2. Each layer: check **O(M) neighbors**
3. Total: **O(log N × M)**

Compare to:
- Naive search: **O(N)**
- LSH: **O(N^ρ)** where ρ < 1

**HNSW achieves O(log N)!** Like binary search!

---

## Comparison with Other Methods

| Algorithm | Construction | Query | Space | Accuracy |
|-----------|--------------|-------|-------|----------|
| **Exact** | O(1) | O(N) | O(N) | 100% |
| **LSH** | O(N) | O(N^ρ) | O(NL) | 90-95% |
| **HNSW** | O(N log N) | **O(log N)** | O(N log N) | **95-99%** |
| **IVF** | O(N) | O(N/k) | O(N) | 85-95% |

**HNSW is the WINNER for most use cases!**

---

## Why HNSW is Best

1. **Logarithmic search time** (as good as binary search!)
2. **High recall** (95-99% accuracy typical)
3. **No training required** (unlike IVF)
4. **Scales to billions** of vectors
5. **Supports incremental updates**

---

## Production Use

### Used By

- **Pinecone**: HNSW-based
- **Qdrant**: Pure HNSW
- **Weaviate**: HNSW
- **Milvus**: Supports HNSW
- **Vespa**: HNSW option

This is **THE algorithm** used in production!

---

## Parameter Tuning Guide

### For Indexing Speed

```
M = 8-12
ef_construction = 100
```

### For Search Quality

```
M = 16-32
ef_construction = 200-400
```

### For Memory Efficiency

```
M = 8
M_max = 16
```

### For Search Speed

```
ef = k  (minimum)
```

### For Search Quality

```
ef = 2k to 3k
```

---

## Key Formulas

### Layer Probability

```
P(layer = l) = (1/M)^l × (1 - 1/M)
```

### Expected Layers

```
E[max_layer] = log_M(N)
```

### Search Complexity

```
O(log N) layers × O(M) neighbors = O(M log N)
```

---

## Algorithm in One Picture

```
Query arrives
     ↓
Start at top layer (few nodes)
     ↓
Greedy search → zoom in
     ↓
Move down one layer
     ↓
Greedy search → zoom in more
     ↓
Move down to layer 0 (all nodes)
     ↓
Final search with larger ef
     ↓
Return top k neighbors
```

---

## Common Misconceptions

### ❌ "HNSW is just a skip list"

While similar in spirit (hierarchy), HNSW is fundamentally different:
- **Skip list**: 1D, deterministic
- **HNSW**: Multi-dimensional, graph-based

### ❌ "Higher layers are redundant"

No! Higher layers are crucial for O(log N) complexity. Without them, degrades to O(N).

### ❌ "More connections = always better"

No! Too many connections:
- Waste memory
- Slow down search
- Diminishing returns

M=16-32 is sweet spot.

---

## Tips for Implementation

### 1. Use Max-Heaps

For efficient candidate management:
```python
import heapq
candidates = []  # min-heap
best = []  # max-heap (use negative distances)
```

### 2. Prune Aggressively

Don't keep more than M connections:
```python
if len(neighbors) > M:
    neighbors = select_best_M(neighbors)
```

### 3. Batch Construction

Add multiple points before searching:
```python
for batch in batches:
    for point in batch:
        add_without_search(point)
    reindex_batch()
```

### 4. Normalize Vectors

For cosine similarity:
```python
vector = vector / np.linalg.norm(vector)
```

Then use dot product (faster than cosine).

---

## Exercises

1. **Implement HNSW from scratch**
   - Start with basic NSW
   - Add hierarchy
   - Add pruning heuristics

2. **Experiment with parameters**
   - Vary M, ef_construction, ef
   - Measure recall vs speed tradeoff
   - Find optimal for your data

3. **Compare with LSH**
   - Same dataset
   - Measure: construction time, search time, recall
   - Which is better when?

4. **Study production implementations**
   - hnswlib (C++, Python bindings)
   - Faiss (Facebook)
   - Compare approaches

---

## Summary

**HNSW Key Ideas:**
1. Hierarchical layers (highway to streets)
2. Exponential layer assignment
3. Greedy search with zoom-in
4. O(log N) search complexity!

**When to Use HNSW:**
- ✓ Medium to large scale (>10K vectors)
- ✓ Need high recall (>95%)
- ✓ Can afford O(N log N) construction
- ✓ Production systems

**This is the STATE-OF-THE-ART algorithm!**

Understanding HNSW deeply gives you the foundation for:
- Building production vector databases
- Reading research papers
- Optimizing for your use case
- Contributing to open source

**You now understand the algorithm powering modern AI!**
