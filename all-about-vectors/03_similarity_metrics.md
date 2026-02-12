# Part 3: Similarity Search and Distance Metrics - Complete Deep Dive

## Table of Contents
1. [What is Similarity?](#what-is-similarity)
2. [Distance vs Similarity](#distance-vs-similarity)
3. [All Distance Metrics Explained](#all-distance-metrics-explained)
4. [All Similarity Metrics Explained](#all-similarity-metrics-explained)
5. [When to Use Which Metric](#when-to-use-which-metric)
6. [Performance Characteristics](#performance-characteristics)

---

## What is Similarity?

### Fundamental Question
How do we measure how "close" two things are?

### In the Real World
- **People**: Similar if they share interests, beliefs, behaviors
- **Products**: Similar if they have same features, uses, price range
- **Texts**: Similar if they discuss same topics, use same words

### In Vector Space
- Vectors are **points** in high-dimensional space
- "Close" means **small distance** OR **large similarity score**
- Different metrics capture different notions of "close"

---

## Distance vs Similarity

### Distance Metrics
- Measure how **FAR APART** vectors are
- **Range**: [0, ∞) where 0 = identical
- **Smaller is MORE similar**
- Examples: Euclidean, Manhattan, Chebyshev

### Similarity Metrics
- Measure how **CLOSE** vectors are
- **Range**: typically [-1, 1] or [0, 1]
- **Larger is MORE similar**
- Examples: Cosine similarity, Jaccard, Pearson correlation

### Conversion Between Them

```
similarity = 1 / (1 + distance)
distance = 1 / similarity - 1
```

(there are other conversions too)

---

## Properties of Distance Metrics

For a function to be a valid "metric", it must satisfy:

### 1. Non-Negativity
```
d(x, y) ≥ 0
```
Distance is never negative.

### 2. Identity
```
d(x, y) = 0 if and only if x = y
```
Distance is zero only for identical points.

### 3. Symmetry
```
d(x, y) = d(y, x)
```
Distance from x to y equals distance from y to x.

### 4. Triangle Inequality
```
d(x, z) ≤ d(x, y) + d(y, z)
```
Direct path is never longer than indirect path.

**Note**: Not all "distance functions" satisfy all properties! For example, KL divergence violates symmetry.

---

## All Distance Metrics Explained

### 1. Euclidean Distance (L2)

**The most common distance metric.** "As the crow flies" distance.

#### Formula
```
d(v1, v2) = sqrt(Σ(v1[i] - v2[i])²)
```

#### Geometric Interpretation
Length of the straight line connecting two points.

#### When to Use
- When all dimensions have same scale and units
- When magnitude matters (pixel intensities, measurements)
- When you want "true" geometric distance
- General purpose, works well in most cases

#### Pros
- ✓ Intuitive geometric meaning
- ✓ Satisfies all metric properties
- ✓ Works well in low dimensions

#### Cons
- ✗ Sensitive to scale (large values dominate)
- ✗ Curse of dimensionality in high dimensions
- ✗ Affected by outliers (due to squaring)

#### Computational Complexity
O(n) where n = dimensions

---

### 2. Manhattan Distance (L1, Taxicab Distance)

**Sum of absolute differences.** Like walking on a city grid.

#### Formula
```
d(v1, v2) = Σ|v1[i] - v2[i]|
```

#### Geometric Interpretation
Distance if you can only move along axes (like Manhattan streets).

#### When to Use
- Sparse data (many zeros)
- When dimensions are not interchangeable
- When you want robustness to outliers
- Discrete/categorical features

#### Pros
- ✓ Less sensitive to outliers than Euclidean
- ✓ Works well with high-dimensional sparse data
- ✓ Faster to compute (no square root)

#### Cons
- ✗ Less intuitive than Euclidean
- ✗ Overestimates distance in diagonal directions

#### Example
```
Manhattan from (0,0) to (3,4) = 3 + 4 = 7
Euclidean from (0,0) to (3,4) = √(9+16) = 5
```
Manhattan is always ≥ Euclidean.

#### Computational Complexity
O(n)

---

### 3. Chebyshev Distance (L∞, Maximum Distance)

**Maximum absolute difference** across all dimensions.

#### Formula
```
d(v1, v2) = max_i |v1[i] - v2[i]|
```

#### Geometric Interpretation
Minimum number of moves to get from one point to another if you can move one unit in any direction (including diagonally). Like a **king's move in chess**.

#### When to Use
- When worst-case dimension matters
- Grid-based problems (pathfinding, games)
- When one bad dimension makes items dissimilar

#### Pros
- ✓ Very simple and fast to compute
- ✓ Useful for worst-case analysis

#### Cons
- ✗ Ignores all but one dimension
- ✗ Very coarse metric

#### Example
```
Chebyshev from (0,0) to (3,4) = max(3,4) = 4
```
Only the largest difference matters!

#### Computational Complexity
O(n)

---

### 4. Minkowski Distance (Generalized Lp)

**A family of distance metrics** parameterized by p.

#### Formula
```
d(v1, v2) = (Σ|v1[i] - v2[i]|^p)^(1/p)
```

#### Special Cases
- p = 1: **Manhattan distance**
- p = 2: **Euclidean distance**
- p = ∞: **Chebyshev distance**

#### Properties
- **Higher p**: More influenced by large differences
- **Lower p**: More influenced by overall differences

#### When to Use
- When you want to interpolate between Manhattan and Euclidean
- p > 2: When you want to penalize large differences more
- p < 2: When you want more uniform weighting

#### Typical Values
- p = 1.5: Good for some high-dimensional data
- p = 3 or 4: When large differences are very important

#### Computational Complexity
O(n)

---

### 5. Canberra Distance

**Weighted version of Manhattan distance.** Normalizes by magnitude.

#### Formula
```
d(v1, v2) = Σ |v1[i] - v2[i]| / (|v1[i]| + |v2[i]|)
```

#### When to Use
- When dealing with non-negative data
- When small values are as important as large values
- Data with different scales across dimensions
- Comparing probability distributions

#### Pros
- ✓ Normalizes for magnitude (scale-invariant)
- ✓ Sensitive to changes near zero

#### Cons
- ✗ Undefined when both values are zero
- ✗ Not a true metric (violates triangle inequality)

#### Example
Difference between (1000, 1000) and (1001, 1001) vs (1, 1) and (2, 2):
```
Manhattan: first pair = 2, second pair = 2 (same)
Canberra: first pair ≈ 0.001, second pair ≈ 0.67 (very different!)
```

#### Computational Complexity
O(n)

---

### 6. Hamming Distance

**Number of positions where vectors differ.**

#### Formula
```
d(v1, v2) = Σ I(v1[i] ≠ v2[i])
```
where I is the indicator function (1 if true, 0 if false).

#### When to Use
- Binary vectors (0s and 1s)
- Categorical data (discrete values)
- Error detection (comparing bit strings)
- DNA sequences

#### Pros
- ✓ Very simple and interpretable
- ✓ Efficient for binary data
- ✓ Natural for discrete/categorical data

#### Cons
- ✗ Only works for discrete values
- ✗ Doesn't capture magnitude of differences

#### Example
```
v1 = [1, 0, 1, 1, 0]
v2 = [1, 1, 1, 0, 0]
Hamming distance = 2 (positions 1 and 3 differ)
```

#### Computational Complexity
O(n)

---

### 7. Mahalanobis Distance

**Distance that accounts for correlations** between dimensions.

#### Formula
```
d(v1, v2) = sqrt((v1-v2)ᵀ Σ⁻¹ (v1-v2))
```
where Σ is the covariance matrix.

#### Intuition
- Euclidean distance assumes dimensions are independent
- Mahalanobis accounts for correlations
- Normalizes by variance in each direction

#### When to Use
- Dimensions are correlated
- Different dimensions have different variances
- Detecting outliers
- Multivariate statistical analysis

#### Pros
- ✓ Accounts for data distribution
- ✓ Scale-invariant
- ✓ Statistically principled

#### Cons
- ✗ Requires covariance matrix (needs lots of data)
- ✗ Computationally expensive
- ✗ Assumes Gaussian distribution

#### Example
If height and weight are correlated, Mahalanobis distance between (tall, heavy) and (short, light) is **SMALL** even if Euclidean distance is large, because the difference follows the expected correlation.

#### Computational Complexity
O(n²) for matrix inversion + O(n²) for multiplication

---

## All Similarity Metrics Explained

### 1. Cosine Similarity ⭐

**THE most important metric for text embeddings!**

#### Formula
```
cos(θ) = (v1 · v2) / (||v1|| × ||v2||)
```

#### Range
**[-1, 1]**
- 1: Same direction (very similar)
- 0: Perpendicular (unrelated)
- -1: Opposite direction (opposite meaning)

#### Geometric Interpretation
Cosine of the angle between vectors. Measures **direction, not magnitude**.

#### When to Use
- **Text embeddings** (THIS IS THE DEFAULT!)
- When magnitude doesn't matter
- When vectors might have different scales
- High-dimensional sparse data

#### Why It's Perfect for Text

1. **Long vs short documents**:
   ```
   "AI is great" vs "AI is great. AI is great."
   ```
   Should be similar even though second is 2x longer!

2. **Direction captures meaning**:
   Words with similar meanings point in similar directions.

3. **Normalized**:
   Not affected by document length or word frequency.

#### Pros
- ✓ Scale-invariant (perfect for text)
- ✓ Bounded [-1, 1] (easy to interpret)
- ✓ Fast to compute for normalized vectors

#### Cons
- ✗ Ignores magnitude completely
- ✗ Not a true distance metric

#### Optimization
If vectors are pre-normalized (||v|| = 1), then:
```
cosine_similarity(v1, v2) = v1 · v2  (just dot product!)
```

#### Computational Complexity
- O(n) if vectors normalized
- O(n) + O(n) + O(1) = O(n) if not normalized

---

### 2. Dot Product (Inner Product)

Not technically a similarity metric, but **widely used**!

#### Formula
```
v1 · v2 = Σ v1[i] × v2[i]
```

#### Range
**[-∞, ∞]**
- Larger = more similar (for normalized vectors)

#### Geometric Interpretation
Projection of v1 onto v2, times length of v2.
Or: ||v1|| × ||v2|| × cos(θ)

#### When to Use
- When vectors are normalized (unit length)
- In neural networks (attention mechanisms)
- When magnitude carries information

#### Relationship to Cosine
For normalized vectors:
```
dot_product = cosine_similarity
```

#### Pros
- ✓ Extremely fast to compute
- ✓ Natural for neural networks
- ✓ Captures both direction and magnitude

#### Cons
- ✗ Not bounded
- ✗ Affected by vector magnitudes
- ✗ Not scale-invariant

#### Use Case in Transformers
In attention mechanisms:
```
attention(Q, K) = softmax(Q · Kᵀ / √d)
```
Dot product measures query-key similarity!

#### Computational Complexity
O(n)

---

### 3. Pearson Correlation

**Measures LINEAR correlation** between vectors.

#### Formula
```
r = Σ((v1[i] - mean(v1)) × (v2[i] - mean(v2))) / 
    (std(v1) × std(v2) × n)
```

#### Range
**[-1, 1]**
- 1: Perfect positive correlation
- 0: No linear correlation
- -1: Perfect negative correlation

#### Interpretation
- Centers data (subtracts means)
- Normalizes by standard deviations
- **Cosine similarity of centered vectors!**

#### When to Use
- Measuring linear relationships
- When absolute values don't matter, only trends
- Comparing time series
- User ratings (similar taste)

#### Pros
- ✓ Scale and shift invariant
- ✓ Statistically interpretable
- ✓ Robust to linear transformations

#### Cons
- ✗ Only captures LINEAR relationships
- ✗ Sensitive to outliers
- ✗ Requires sufficient variance

#### Example
```
v1 = [1, 2, 3, 4, 5]
v2 = [10, 20, 30, 40, 50]  (v2 = 10×v1)
Pearson correlation = 1.0 (perfect correlation!)
```
Even though values are very different.

#### Computational Complexity
O(n)

---

### 4. Jaccard Similarity (Jaccard Index)

**Intersection over union** for sets.

#### Formula
```
J(A, B) = |A ∩ B| / |A ∪ B|
```

For vectors (treating as sets of non-zero elements):
```
J(v1, v2) = (# both non-zero) / (# at least one non-zero)
```

#### Range
**[0, 1]**
- 1: Identical sets
- 0: No overlap

#### When to Use
- Binary vectors (presence/absence)
- Sets (tags, keywords, categories)
- Sparse vectors
- Document similarity (bag of words)

#### Pros
- ✓ Intuitive for sets
- ✓ Simple to compute and understand
- ✓ Good for sparse data

#### Cons
- ✗ Ignores frequency/magnitude
- ✗ Only considers presence/absence

#### Example
```
Document 1 words: {cat, dog, bird}
Document 2 words: {cat, dog, fish}
Intersection: {cat, dog} = 2 words
Union: {cat, dog, bird, fish} = 4 words
Jaccard = 2/4 = 0.5
```

#### Computational Complexity
O(n)

---

### 5. Dice Coefficient (Sørensen-Dice Index)

**Similar to Jaccard** but gives more weight to intersection.

#### Formula
```
Dice = 2 × |A ∩ B| / (|A| + |B|)
```

#### Range
**[0, 1]**

#### Relationship to Jaccard
```
Dice = 2×Jaccard / (1 + Jaccard)
```

#### When to Use
- When you want to emphasize overlap more than Jaccard
- Image segmentation evaluation
- Set similarity with more weight on common elements

#### Difference from Jaccard
```
Example: A = {1,2,3}, B = {1,2,4,5}
Intersection = 2, |A| = 3, |B| = 4, Union = 5

Jaccard = 2/5 = 0.4
Dice = 2×2/(3+4) = 0.571

Dice is always ≥ Jaccard
```

#### Computational Complexity
O(n)

---

## When to Use Which Metric

### Decision Guide

```
┌─────────────────────────────────────────┐
│       WHAT TYPE OF DATA?                │
└─────────────────────────────────────────┘
                  │
        ┌─────────┴─────────┐
        │                   │
    TEXT/NLP           OTHER DATA
        │                   │
        │         ┌─────────┴─────────┐
        │         │                   │
        │    BINARY/SETS       CONTINUOUS
        │         │                   │
        │    Jaccard           Euclidean
        │    Hamming          Manhattan
        │    Dice             Minkowski
        │                         │
        │                 ┌───────┴────────┐
        │                 │                │
        │          MAGNITUDE         MAGNITUDE
        │           MATTERS         IRRELEVANT
        │                 │                │
        │           Euclidean        Cosine
        │           Dot Product
        │
    COSINE ⭐
    (almost always!)
```

### By Use Case

| Use Case | Best Metric | Why |
|----------|-------------|-----|
| **Text embeddings** | Cosine | Direction = meaning, scale-invariant |
| **Image embeddings** | Euclidean (L2) | Pixel values, magnitude matters |
| **Binary data** | Hamming, Jaccard | Presence/absence |
| **Sparse high-dim** | Manhattan (L1), Cosine | Robust to zeros |
| **Outlier-prone** | Manhattan | More robust than Euclidean |
| **Correlated dims** | Mahalanobis | Accounts for correlations |
| **Time series** | Pearson, DTW | Trends matter |
| **When uncertain** | Cosine (text), Euclidean (other) | Good defaults |

---

## Performance Characteristics

### Computational Complexity

| Metric | Time Complexity | Notes |
|--------|----------------|-------|
| Dot Product | O(n) | Fastest |
| Cosine | O(n) | O(n) if normalized |
| Euclidean | O(n) | Includes sqrt |
| Manhattan | O(n) | No sqrt, faster than Euclidean |
| Chebyshev | O(n) | Just max operation |
| Minkowski | O(n) | Includes power operations |
| Mahalanobis | O(n²) | Matrix operations |

### Memory Requirements

| Metric | Space Complexity | Notes |
|--------|-----------------|-------|
| Most metrics | O(1) | Only need input vectors |
| Mahalanobis | O(n²) | Needs covariance matrix |

---

## Key Formulas Reference

### For Normalized Vectors (||v|| = 1)

```
cosine_similarity(v1, v2) = v1 · v2
```

### Relationship Between Metrics

```
For normalized vectors:
euclidean_dist² = 2 × (1 - cosine_similarity)
```

### Converting Distance to Similarity

```
similarity = 1 / (1 + distance)
similarity = exp(-distance)
similarity = 1 - (distance / max_distance)
```

---

## Common Pitfalls

### ❌ Using Euclidean for Text
```python
# BAD: Euclidean on text embeddings
distance = euclidean(doc1_embedding, doc2_embedding)
```
**Why**: Document length affects magnitude unfairly.

**Fix**: Use cosine similarity.

### ❌ Not Normalizing
```python
# BAD: Assuming vectors are normalized
similarity = dot_product(v1, v2)  # Wrong if not normalized!
```
**Fix**: Normalize first or use cosine.

### ❌ Wrong Metric for Data Type
```python
# BAD: Cosine on categorical data
similarity = cosine(one_hot1, one_hot2)
```
**Fix**: Use Hamming or Jaccard for categorical.

---

## Exercises

1. **Implement all metrics** from scratch (no libraries)

2. **Compare metrics** on your data:
   - Plot similarity distributions
   - Find which captures your notion of "similar"

3. **Prove mathematically**:
   - Cosine similarity is bounded [-1, 1]
   - Euclidean satisfies triangle inequality
   - For unit vectors: dot = cosine

4. **Experiment**:
   - When does Manhattan beat Euclidean?
   - How does dimensionality affect metrics?
   - Visualize metric behavior in 2D/3D

---

## Summary

**Most Important Takeaway**:
- **Text**: Use **Cosine Similarity** ⭐
- **Images**: Use **Euclidean Distance**
- **Binary**: Use **Hamming** or **Jaccard**
- **When unsure**: Start with Cosine or Euclidean

Understanding these metrics deeply is **crucial** for vector databases and similarity search!
