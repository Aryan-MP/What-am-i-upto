# Part 2: Word2Vec and Neural Network Embeddings - Complete Deep Dive

## Table of Contents
1. [Why Neural Embeddings?](#why-neural-embeddings)
2. [The Distributional Hypothesis](#the-distributional-hypothesis)
3. [Word2Vec Architecture](#word2vec-architecture)
4. [Skip-Gram vs CBOW](#skip-gram-vs-cbow)
5. [Negative Sampling Explained](#negative-sampling-explained)
6. [Training Process](#training-process)
7. [Properties of Learned Embeddings](#properties-of-learned-embeddings)

---

## Why Neural Embeddings?

### Limitations of Count-Based Methods

#### 1. Sparsity Problem
- Co-occurrence matrices are **HUGE** and mostly zeros
- For 100K vocabulary: 100K × 100K = **10 billion entries**!
- Most word pairs never co-occur

#### 2. Computational Cost
- SVD (dimension reduction) is **O(n³)** - very slow
- Hard to update when new data arrives

#### 3. Frequency Bias
- Common words dominate the matrix
- Rare but meaningful words get lost

#### 4. Fixed Representations
- No learning from data structure
- Can't capture complex patterns

### Neural Network Solution

Instead of counting co-occurrences, we **LEARN** embeddings by:

1. Define a prediction task (predict context from word)
2. Use a neural network with an embedding layer
3. Train the network on lots of text
4. **Extract the learned embeddings!**

**Key insight**: We don't actually care about the prediction task. We care about what the network learns to solve it!

---

## The Distributional Hypothesis

### J.R. Firth (1957):
> "You shall know a word by the company it keeps"

### More Formally

- Words that appear in similar contexts have similar meanings
- **Context** = surrounding words

### Example

```
"The [cat] sat on the mat"
→ context: [the, sat, on, the, mat]

"The [dog] sat on the rug"
→ context: [the, sat, on, the, rug]
```

cat and dog have similar contexts → they should have similar embeddings!

### Mathematical Formulation

We want to find embeddings such that:

```
P(context | word) is high for words that actually appear together
```

Or equivalently:
```
P(word | context) is high
```

This is a **CONDITIONAL PROBABILITY** problem!

---

## Word2Vec Architecture

### Two Architectures

#### 1. Skip-Gram
**Predict context words from target word**

```
Input: "cat" 
Output: ["the", "sat", "on", "mat"]
```

#### 2. CBOW (Continuous Bag of Words)
**Predict target from context**

```
Input: ["the", "sat", "on", "mat"]
Output: "cat"
```

**We'll focus on Skip-Gram** as it works better for most tasks.

---

## Skip-Gram Architecture Deep Dive

### Network Structure

```
Input Layer (one-hot):     [0, 0, 1, 0, 0, ...] (size: vocab_size)
                                    ↓
Embedding Layer (weights):  W_embed (vocab_size × embed_dim)
                                    ↓
Hidden Layer:              [0.2, 0.8, -0.3, ...]  (size: embed_dim)
                                    ↓
Output Layer (weights):     W_output (embed_dim × vocab_size)
                                    ↓
Output (probabilities):    [0.01, 0.3, 0.5, ...]  (size: vocab_size)
```

### Key Insight

**The embedding layer weights ARE the word embeddings!**

After training, we throw away the output layer and keep only the embeddings.

### Training Objective

Maximize:
```
log P(context | word)
```

For a word w and context c:
```
P(c | w) = exp(v_c · v_w) / Σ_i exp(v_i · v_w)
```

This is called the **SOFTMAX** function.

### The Problem

Computing the denominator requires summing over **ALL words**!

For 100K vocabulary, that's **100K operations per training example**!

---

## Negative Sampling - The Key Trick

### The Problem with Full Softmax

For each training example, we need to:
1. Compute scores for ALL vocab_size words
2. Compute softmax over ALL vocab_size words
3. Calculate gradients for ALL vocab_size words

For vocab_size = 100,000, this is **incredibly slow**!

### Negative Sampling Solution

Instead of predicting ALL words, we:

1. Take the actual context word (**POSITIVE** example)
2. Sample k random words (**NEGATIVE** examples, typically k=5-20)
3. Train only on these **k+1 words**!

### Objective Change

**Full softmax:**
```
maximize log P(context | word) for all words
```

**Negative sampling:**
```
maximize log σ(v_pos · v_word)    ← positive example
+ Σ log σ(-v_neg_i · v_word)      ← negative examples
```

where σ(x) = 1 / (1 + exp(-x)) is the **sigmoid function**.

### Intuition

- Make dot product **HIGH** for word-context pairs that appear together
- Make dot product **LOW** for word-random_word pairs

This is a **BINARY CLASSIFICATION** problem (context vs non-context) instead of a **MULTI-CLASS** problem (which context word?)!

### Sampling Strategy

Not all negative samples are equal! Common words should be sampled more often.

**Probability of sampling word w:**
```
P(w) = count(w)^0.75 / Σ count(w')^0.75
```

#### Why 0.75?

- Makes sampling more uniform than raw frequency
- Common words still sampled more, but not dominantly
- Rare words get a better chance

### Example

Suppose we're training on: **"the cat sat"**

**Positive pair**: (cat, sat)

**Negative sampling** (k=5):
- Sample 5 random words: [car, python, quantum, tree, house]

**Training:**
- ✓ Increase similarity(cat, sat)
- ✗ Decrease similarity(cat, car)
- ✗ Decrease similarity(cat, python)
- ✗ Decrease similarity(cat, quantum)
- ✗ Decrease similarity(cat, tree)
- ✗ Decrease similarity(cat, house)

**Speedup**: Only 6 words updated instead of 100,000!
Speedup factor: **100,000/6 ≈ 16,667x faster!**

---

## Training Process Step-by-Step

### Step 1: Initialize Embeddings

```
W_embed = random matrix (vocab_size × embedding_dim)
W_context = random matrix (vocab_size × embedding_dim)
```

Typically initialized with small random values (e.g., × 0.01).

### Step 2: Generate Training Pairs

From sentence: "the cat sat on the mat"

With window_size=2, for word "sat" (index 2):
- Contexts: "the" (0), "cat" (1), "on" (3), "mat" (4)
- Pairs: (sat, the), (sat, cat), (sat, on), (sat, mat)

### Step 3: For Each Pair (target, context)

1. **Get embeddings**
   ```
   target_embed = W_embed[target_idx]
   context_embed = W_context[context_idx]
   ```

2. **Positive example**
   ```
   score_pos = target_embed · context_embed
   pred_pos = σ(score_pos)
   loss = -log(pred_pos)
   ```

3. **Sample negative examples**
   ```
   Sample k words according to P(w) ∝ count(w)^0.75
   ```

4. **Negative examples**
   ```
   For each negative word:
       score_neg = target_embed · neg_embed
       pred_neg = σ(score_neg)
       loss += -log(1 - pred_neg)
   ```

5. **Update embeddings**
   ```
   Compute gradients
   W_embed -= learning_rate × gradient
   W_context -= learning_rate × gradient
   ```

### Step 4: Repeat

- Process all training pairs
- Multiple epochs (typically 5-10)
- Gradually decrease learning rate

---

## Properties of Learned Embeddings

### 1. Semantic Similarity

Words with similar meanings have similar embeddings:

```
cosine_similarity("cat", "dog") ≈ 0.8
cosine_similarity("cat", "car") ≈ 0.1
```

### 2. Analogical Reasoning

Famous example:
```
king - man + woman ≈ queen
```

**How it works:**
```
vec("king") - vec("man") ≈ "royalty + male"
"royalty + male" - "male" ≈ "royalty"
"royalty" + vec("woman") ≈ "royalty + female"
≈ vec("queen")
```

### 3. Compositionality

Embeddings can be combined:
```
vec("New York") ≈ vec("New") + vec("York")
```

### 4. Clustering

Similar words cluster in embedding space:
- Animals: cat, dog, bird, fish
- Countries: France, Germany, Italy, Spain
- Colors: red, blue, green, yellow

### 5. Dimensionality Captures Relationships

Different dimensions capture different aspects:
- Dimension 1: alive vs inanimate
- Dimension 2: abstract vs concrete
- Dimension 3: size (small vs large)
- etc.

---

## Skip-Gram vs CBOW Comparison

| Aspect | Skip-Gram | CBOW |
|--------|-----------|------|
| **Task** | Predict context from word | Predict word from context |
| **Speed** | Slower | Faster |
| **Data efficiency** | Better for small datasets | Better for large datasets |
| **Rare words** | Works better | Works worse |
| **Typical use** | When quality matters | When speed matters |

**Generally preferred: Skip-Gram** for most applications.

---

## Training Tips

### Hyperparameters

- **Embedding dimension**: 50-300 (typical: 100-200)
- **Window size**: 2-10 (typical: 5)
- **Negative samples**: 5-20 (typical: 5-10)
- **Learning rate**: 0.025 (with decay)
- **Min count**: Remove words appearing < 5 times
- **Epochs**: 5-10

### Data Requirements

- **Minimum**: 1M words for basic quality
- **Good**: 10M-100M words
- **Best**: 1B+ words (like Google's Word2Vec)

### Training Time

For 1M words, 100 dimensions:
- On CPU: ~10-30 minutes
- On GPU: ~1-5 minutes

---

## Key Formulas

### Sigmoid Function
```
σ(x) = 1 / (1 + exp(-x))
```

### Negative Sampling Objective
```
J = log σ(v_c · v_w) + Σ(i=1 to k) log σ(-v_ni · v_w)
```

where:
- v_c = context word embedding
- v_w = target word embedding
- v_ni = negative sample i embedding
- k = number of negative samples

### Sampling Probability
```
P(w) = (count(w)^0.75) / Σ(count(w')^0.75)
```

---

## Historical Context

### Timeline

- **2013**: Word2Vec introduced by Mikolov et al. at Google
- **2014**: GloVe introduced by Pennington et al. at Stanford
- **2018**: ELMo introduces contextual embeddings
- **2019**: BERT revolutionizes NLP with transformers

### Why Word2Vec Was Revolutionary

1. **Simple**: Easy to understand and implement
2. **Fast**: Can train on billions of words
3. **Effective**: Captures semantic relationships
4. **Scalable**: Works for massive vocabularies
5. **Foundation**: Inspired all modern embeddings

---

## Comparison with Other Methods

| Method | Year | Type | Pros | Cons |
|--------|------|------|------|------|
| **One-hot** | - | Sparse | Simple | No semantics, huge |
| **LSA/SVD** | 1990s | Count | Captures co-occurrence | Slow, static |
| **Word2Vec** | 2013 | Neural | Fast, semantic | No context, static |
| **GloVe** | 2014 | Hybrid | Combines count + neural | Static |
| **BERT** | 2018 | Contextual | Dynamic, context-aware | Slow, large |

---

## Limitations of Word2Vec

1. **No context sensitivity**
   - "bank" (river) vs "bank" (finance) have same embedding

2. **Out-of-vocabulary**
   - New words can't be embedded

3. **Fixed after training**
   - Can't adapt to new meanings

4. **Requires large data**
   - Needs millions of words for good quality

**Solution**: Contextual embeddings (BERT, GPT, etc.)

---

## Next Steps

Now that you understand Word2Vec, you're ready for:
- **Sentence embeddings**: Averaging, weighted averaging
- **Transformers**: BERT, GPT attention mechanisms
- **Contextual embeddings**: Dynamic representations
- **Fine-tuning**: Adapting to specific domains

---

## Exercises

1. **Implement**:
   - Skip-Gram with negative sampling from scratch
   - Training loop with learning rate decay
   - Similarity search using learned embeddings

2. **Experiment**:
   - Train on different corpus sizes
   - Vary window size, negative samples, dimensions
   - Compare Skip-Gram vs CBOW

3. **Analyze**:
   - Find analogies in your trained embeddings
   - Cluster words by similarity
   - Visualize embeddings in 2D (using t-SNE)

4. **Compare**:
   - Word2Vec vs GloVe
   - Different training corpora (news vs social media)
   - Effect of corpus size on quality
