# Part 1: Embeddings Fundamentals - Complete Deep Dive

## Table of Contents
1. [What is an Embedding?](#what-is-an-embedding)
2. [Why Do We Need Embeddings?](#why-do-we-need-embeddings)
3. [The Mathematics Behind Embeddings](#the-mathematics-behind-embeddings)
4. [Different Types of Embeddings](#different-types-of-embeddings)
5. [How Embeddings Are Learned](#how-embeddings-are-learned)

---

## What is an Embedding?

### Fundamental Concept

An **embedding** is a mapping from discrete objects (like words, sentences, images) to continuous vector spaces.

**Discrete vs Continuous:**
- **Discrete**: Separate, distinct values (like words: "cat", "dog", "house")
- **Continuous**: Can take any value in a range (like numbers: 1.5, 2.73, -0.44)

### Why Map Discrete to Continuous?

1. **Computers understand numbers**, not words
2. We can do **math on numbers** (add, subtract, measure distance)
3. **Similar things should be close together** in number space
4. We can **capture relationships and meaning** through positions

### Simple Example

```
Word → Vector
"cat"   → [0.5, 0.8, -0.2]
"dog"   → [0.6, 0.7, -0.1]
"car"   → [-0.3, 0.1, 0.9]
```

Notice: cat and dog are close (similar values), car is far away.

---

## One-Hot Encoding: The Simplest Embedding

### Concept

Give each word a unique position in a vector. That position is 1, all others are 0.

**Example with vocabulary ["cat", "dog", "bird"]:**
```
"cat"  → [1, 0, 0]
"dog"  → [0, 1, 0]
"bird" → [0, 0, 1]
```

### Problems with One-Hot

1. **Every word is equally different** from every other word
   - Distance(cat, dog) = Distance(cat, car) = Distance(cat, quantum)
2. **Vectors are HUGE** for large vocabularies (100,000+ dimensions)
3. **Very sparse** (mostly zeros)
4. **No relationship** between similar words

**However**, it's important to understand because it's the **foundation**!

---

## Count-Based Embeddings

### Core Idea: Distributional Hypothesis

> "You shall know a word by the company it keeps" - J.R. Firth, 1957

**Words that appear in similar contexts have similar meanings.**

### Method: Co-occurrence Matrix

1. Count how often words appear near each other
2. Create a matrix where entry (i,j) = count of word_i near word_j
3. Each row becomes the embedding for that word

**Example Corpus:**
```
"the cat sat on the mat"
"the dog sat on the rug"
"the cat and the dog played"
```

**Co-occurrence matrix (window size = 1):**
```
           cat  dog  sat  mat  rug  played  and
cat      [  0,   1,   1,   0,   0,    1,    1 ]
dog      [  1,   0,   1,   0,   0,    1,    1 ]
sat      [  1,   1,   0,   2,   1,    0,    0 ]
...
```

Notice: **cat and dog have similar vectors** (appear in similar contexts)!

### PMI (Pointwise Mutual Information)

PMI measures how much more likely two words are to co-occur than if they were independent.

**Formula:**
```
PMI(w1, w2) = log(P(w1, w2) / (P(w1) × P(w2)))
```

This gives more weight to **meaningful associations** and reduces the impact of common words like "the", "and", "is".

---

## The Mathematics of Embeddings

### 1. Vector Spaces

A vector space is a mathematical structure where:
- We have **vectors** (ordered lists of numbers)
- We can **add vectors**: v1 + v2
- We can **multiply by scalars**: 3 × v1
- There's a **zero vector**: [0, 0, ..., 0]

**Example in 2D:**
```
v1 = [2, 3]
v2 = [1, 4]
v1 + v2 = [3, 7]
2 × v1 = [4, 6]
```

### 2. Dot Product (Inner Product)

The dot product of two vectors is:

```
v1 · v2 = v1[0]×v2[0] + v1[1]×v2[1] + ... + v1[n]×v2[n]
```

**Example:**
```
[2, 3, 1] · [4, 1, 2] = 2×4 + 3×1 + 1×2 = 8 + 3 + 2 = 13
```

**Geometric Interpretation:**
```
v1 · v2 = ||v1|| × ||v2|| × cos(θ)
```
where θ is the angle between the vectors.

**Why it measures similarity:**
- θ = 0° (same direction): cos(0°) = 1 (maximum similarity)
- θ = 90° (perpendicular): cos(90°) = 0 (no similarity)
- θ = 180° (opposite): cos(180°) = -1 (opposite meaning)

### 3. Vector Norm (Magnitude/Length)

The Euclidean norm (L2 norm) is:

```
||v|| = sqrt(v[0]² + v[1]² + ... + v[n]²)
```

This is the "length" of the vector (distance from origin).

**Example:**
```
||[3, 4]|| = sqrt(3² + 4²) = sqrt(9 + 16) = sqrt(25) = 5
```

### 4. Cosine Similarity

Cosine similarity normalizes the dot product:

```
cosine_sim(v1, v2) = (v1 · v2) / (||v1|| × ||v2||)
```

This removes the effect of magnitude, focusing only on **direction**.

**Range: -1 to 1**
- 1 = identical direction (very similar)
- 0 = perpendicular (unrelated)
- -1 = opposite direction (antonyms)

**This is THE most common similarity measure for text embeddings!**

### 5. Euclidean Distance

L2 distance between vectors:

```
dist(v1, v2) = ||v1 - v2|| = sqrt(sum((v1[i] - v2[i])²))
```

Measures straight-line distance in space.

**Range: 0 to infinity**
- 0 = identical vectors
- larger = more different

Used when magnitude matters (e.g., image embeddings).

### 6. Manhattan Distance (L1)

Sum of absolute differences:

```
dist(v1, v2) = sum(|v1[i] - v2[i]|)
```

Like walking on a city grid (hence "Manhattan").

**Less sensitive to outliers** than Euclidean.

---

## Key Takeaways

### 1. Embeddings Transform Text → Vectors
- Transforms text into numerical representations
- Similar meanings → Similar vectors
- Typically 128-1536 dimensions

### 2. Similarity Metrics: How to Compare Vectors
- **Cosine**: Best for text (direction matters)
- **Euclidean**: Best when magnitude matters
- **Dot Product**: Fast when vectors are normalized

### 3. Count-Based Methods
- Use co-occurrence statistics
- Foundation of LSA, GloVe
- Simple but effective

### 4. Why Dense Embeddings?
- **Sparse** (one-hot): Huge, no relationships
- **Dense**: Compact, captures meaning, relationships

---

## Mathematical Properties to Remember

### Properties of Distance Metrics

A valid distance metric must satisfy:

1. **Non-negativity**: d(x, y) ≥ 0
2. **Identity**: d(x, y) = 0 if and only if x = y
3. **Symmetry**: d(x, y) = d(y, x)
4. **Triangle Inequality**: d(x, z) ≤ d(x, y) + d(y, z)

### Important Formulas

**For normalized vectors (||v|| = 1):**
```
cosine_similarity(v1, v2) = v1 · v2  (just dot product!)
```

**Relationship between cosine and Euclidean:**
```
For normalized vectors:
euclidean_dist² = 2 × (1 - cosine_similarity)
```

---

## Next Steps

Now that you understand embeddings fundamentally, you're ready to learn:
- **Word2Vec**: How neural networks learn embeddings
- **Sentence embeddings**: BERT, transformers
- **Advanced techniques**: Contextualized embeddings

The foundation you've built here is crucial for understanding everything that follows!

---

## Exercises

1. **Implement from scratch:**
   - One-hot encoder
   - Co-occurrence matrix builder
   - All similarity metrics (without libraries)

2. **Experiment:**
   - Build embeddings from your own text corpus
   - Compare different similarity metrics
   - Visualize 2D embeddings

3. **Think about:**
   - Why does cosine similarity ignore magnitude?
   - When would you want magnitude to matter?
   - How many dimensions do you need to capture meaning?

4. **Prove:**
   - Cosine similarity is bounded [-1, 1]
   - Euclidean distance satisfies triangle inequality
   - For unit vectors: dot product = cosine similarity
