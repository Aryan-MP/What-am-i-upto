# Part 4: Locality Sensitive Hashing (LSH) - Complete Deep Dive

## Table of Contents
1. [The Problem: Curse of Dimensionality](#the-problem)
2. [What is LSH?](#what-is-lsh)
3. [Random Hyperplane LSH](#random-hyperplane-lsh)
4. [MinHash LSH](#minhash-lsh)
5. [Parameter Tuning](#parameter-tuning)

---

## The Problem: Curse of Dimensionality

### Naive Search is Too Slow
- **Naive search**: O(n) - check every vector
- For 1 million vectors: 1 million comparisons
- For 1 billion vectors: 1 billion comparisons
- **This is too slow!**

### Why Traditional Data Structures Fail

**Binary Search Tree**:
- Works in 1D: O(log n)
- In high dimensions: no clear ordering!

**K-D Tree**:
- Works up to ~20 dimensions
- Above that: degrades to O(n)

### The Curse in Action

**In high dimensions:**
1. All points become **equally far apart**!
2. Volume of space grows exponentially
3. Data becomes **sparse**
4. Distance metrics **lose meaning**

**Demonstration:**
```
1D:  Points can be close or far
     [----*---*----------*-----]

100D: Almost all points are equally far!
      Every point is in a "corner"
```

### The Solution: Approximate Search

Instead of **EXACT** nearest neighbors:
- Find **APPROXIMATE** nearest neighbors
- Allow some error
- Get **MASSIVE speedup**!

**LSH achieves this elegantly.**

---

## What is LSH?

### Core Idea

**Hash similar items to the same bucket** with high probability.

**Traditional hash** (SHA-256, MD5):
- Random distribution
- Goal: **avoid** collisions

**Locality Sensitive Hash**:
- Similar items → same hash
- Different items → different hash  
- Goal: **CREATE** useful collisions!

### Formal Definition

A family H of hash functions is (r, cr, P1, P2)-sensitive if:

```
For any two points x, y:
- If dist(x, y) ≤ r:  Pr[h(x) = h(y)] ≥ P1
- If dist(x, y) ≥ cr: Pr[h(x) = h(y)] ≤ P2
```

where:
- r = radius (threshold for "similar")
- c > 1 = approximation factor
- **P1 > P2** (similar items collide more often)

### Example

For similar items x, y:
```
Pr[h(x) = h(y)] = 0.9  (90% same bucket)
```

For different items x, z:
```
Pr[h(x) = h(z)] = 0.1  (10% same bucket)
```

### The Key Insight

1. Create MANY hash functions (h1, h2, ..., hk)
2. Hash each point with all functions
3. Combine: H(x) = (h1(x), h2(x), ..., hk(x))
4. Two points are candidates if ANY hash matches
5. Only compute exact distance for candidates

**This reduces O(n) search to O(n^ρ) where ρ < 1!**

---

## Random Hyperplane LSH

### For Cosine Similarity

This is the **most elegant** LSH scheme!

### The Idea

1. Generate random vector r (hyperplane normal)
2. For any point x:
   - If x · r ≥ 0: hash(x) = 1 (above hyperplane)
   - If x · r < 0: hash(x) = 0 (below hyperplane)

### Geometric Interpretation

```
      Points above
           ↑
    *      |      *
  *   *    |    *   *
-----------r----------  ← Hyperplane
    *      |      *
  *   *    |    *
           ↓
      Points below
```

A hyperplane divides space into two half-spaces.
Points on same side → same hash value!

### Mathematical Analysis

**Question**: What's Pr[h(x) = h(y)] for two points x, y?

Let θ = angle between x and y

**Answer**:
```
Pr[h(x) = h(y)] = 1 - θ/π
```

**Proof**:
- Random hyperplane r
- h(x) ≠ h(y) when r is between x and y
- Probability of this = θ/π (angle fraction)
- So Pr[h(x) = h(y)] = 1 - θ/π

### Beautiful Result!

Collision probability directly relates to angle!

For cosine similarity = cos(θ):
```
θ = 0°   (cos = 1):  Pr[collision] = 1.0
θ = 90°  (cos = 0):  Pr[collision] = 0.5
θ = 180° (cos = -1): Pr[collision] = 0.0
```

**This is EXACTLY what we want!**

### Using Multiple Hash Functions

Use k independent random hyperplanes: r1, r2, ..., rk

**Hash signature**: (h1(x), h2(x), ..., hk(x))

Example for k=8: `(1, 0, 1, 1, 0, 1, 0, 1)`

**Two points are candidates if signatures MATCH!**

Probability of matching:
```
Pr[all k bits match] = (1 - θ/π)^k
```

### Amplification with Multiple Tables

Use L hash tables with different random hyperplanes.

Points are candidates if they match in **ANY** table.

**This gives us control over precision/recall tradeoff!**

### Algorithm

**Construction:**
```
For each table t = 1 to L:
    Generate k random hyperplanes
    For each point p:
        Compute k-bit hash
        Add p to bucket[hash]
```

**Query:**
```
For each table t = 1 to L:
    Compute k-bit hash of query
    Retrieve all points in bucket[hash]
Return union of all candidates
```

---

## MinHash LSH

### For Jaccard Similarity

Used for **SET similarity** (Jaccard index).

Extremely important for:
- Document deduplication
- Finding similar users/items
- Plagiarism detection

### The Problem

Given two sets A and B, compute:
```
Jaccard(A, B) = |A ∩ B| / |A ∪ B|
```

For large sets, this is expensive!

### MinHash Solution

Create a "signature" that estimates Jaccard similarity.

### Algorithm

1. Create random permutations π1, π2, ..., πk
2. For each set S and permutation πi:
   ```
   minhash_i(S) = minimum element in πi(S)
   ```
3. Signature: (minhash_1(S), ..., minhash_k(S))

### Beautiful Theorem

```
Pr[minhash(A) = minhash(B)] = Jaccard(A, B)
```

**Proof sketch:**
- Consider any permutation π
- minhash(A) = minhash(B) iff minimum element is in A ∩ B
- This happens with probability |A ∩ B| / |A ∪ B| = Jaccard(A, B)

### Example

```
A = {1, 3, 5, 7, 9}
B = {2, 3, 5, 7, 8}

Jaccard = |{3, 5, 7}| / |{1, 2, 3, 5, 7, 8, 9}| 
        = 3/7 ≈ 0.43
```

Permutation π: [3, 7, 1, 5, 2, 9, 8]
```
π(A) = {3, 7, 1, 5, 9} → min = 3
π(B) = {2, 3, 7, 5, 8} → min = 2
minhash(A) ≠ minhash(B)
```

After many permutations:
```
Fraction of matches ≈ 0.43 ✓
```

---

## Parameter Tuning

### Parameters

- **k**: Number of hash functions per table
- **L**: Number of hash tables

### Tradeoff

**Higher k:**
- ✓ More precision (fewer false positives)
- ✗ Lower recall (miss some true positives)
- ✓ Faster search (smaller buckets)

**Higher L:**
- ✓ Higher recall (catch more true positives)
- ✗ More memory
- ✗ Slower search (check more tables)

### Probability Analysis

For two points at distance d:

Single hash match probability: p = f(d)
- For cosine: p = 1 - θ/π
- For Jaccard: p = J(A, B)

All k hashes match: **p^k**

Match in at least one of L tables: **1 - (1 - p^k)^L**

### Example

For p_similar = 0.9, p_different = 0.1:

```
k=1:  p_s^1 = 0.9,  p_d^1 = 0.1
k=5:  p_s^5 = 0.59, p_d^5 = 0.00001
k=10: p_s^10 = 0.35, p_d^10 ≈ 0
```

With L=10 tables:
```
Pr[retrieve similar]   = 1 - (1-0.35)^10 ≈ 0.98
Pr[retrieve different] = 1 - (1-0)^10 ≈ 0
```

**Great separation!**

### Rule of Thumb

- Start with k = 10-20, L = 5-10
- Increase k for better precision
- Increase L for better recall
- **Measure on your data!**

---

## Performance Comparison

| Method | Time | Space | Accuracy |
|--------|------|-------|----------|
| Exact | O(n) | O(n) | 100% |
| LSH | O(n^ρ) | O(nL) | 90-95% |
| HNSW | O(log n) | O(n log n) | 95-99% |

ρ < 1 (typically 0.5-0.8)

---

## Key Formulas

### Random Hyperplane LSH

```
Pr[h(x) = h(y)] = 1 - arccos(cos_sim(x,y))/π
```

### MinHash

```
Pr[minhash(A) = minhash(B)] = Jaccard(A, B)
```

### Sampling Probability

```
P(w) = count(w)^0.75 / Σcount(w')^0.75
```

---

## Summary

**LSH Key Ideas:**
1. Hash similar items to same bucket
2. Multiple hash functions for better coverage
3. Multiple tables for better recall
4. Approximate but FAST

**When to Use:**
- 10K-1M vectors
- Need simple implementation
- Can tolerate ~90% accuracy

**When NOT to Use:**
- Very large scale (>10M) → use HNSW
- Need >95% accuracy → use HNSW
- Dynamic updates important → use HNSW

**Next**: Learn about HNSW - the state-of-the-art!
