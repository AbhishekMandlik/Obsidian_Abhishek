A **Transformer** is a neural network architecture introduced in 2017 in the paper **"Attention Is All You Need."** It revolutionized natural language processing and is now widely used for text, images, speech, and even protein analysis. Models like GPT, BERT, and Vision Transformers (ViTs) are all based on the Transformer architecture.

### The Big Idea

Instead of processing data one word at a time (like RNNs or LSTMs), Transformers process **all tokens in parallel** and use an **attention mechanism** to determine which parts of the input are most relevant to each other.

For example, in the sentence:

> _"The animal didn't cross the road because **it** was too tired."_

The Transformer learns that **"it"** refers to **"the animal"**, even though they are separated by several words.

---

## Architecture Overview

```
		Input Text
			 │
		Tokenization
			 │
		Token Embeddings
			 │
		Positional Encoding
			 │
 ┌────────────────────────────┐
 │ Transformer Layers(N times)│
 │                            │
 │ Multi-Head Self-Attention  │
 │           │                │
 │   Add & LayerNorm          │
 │           │                │
 │ Feed-Forward Network       │
 │           │                │
 │   Add & LayerNorm          │
 └────────────────────────────┘
		     │
		Output Layer
		     │
		Predictions
```

---

## Key Components

### 1. Token Embeddings

Each word (or token) is converted into a dense numerical vector.

Example:

```
"cat" → [0.12, -0.56, 1.23, ...]
"sat" → [0.91, 0.45, -0.77, ...]
```

These vectors capture semantic meaning.

---

### 2. Positional Encoding

Since Transformers process all words simultaneously, they need information about word order.

Positional encoding adds position information:

```
Word      Position
The          1
cat          2
sat          3
```

Without this, the model would treat:

```
Dog bites man
```

the same as

```
Man bites dog
```

---

### 3. Self-Attention (The Heart of the Transformer)

Each word looks at every other word and decides how much attention to pay to it.

Suppose we have:

```
"The cat chased the mouse."
```

When processing **"chased"**, the model attends strongly to:

- cat
    
- mouse
    

and less to:

- the
    

---

### Query, Key, and Value

Every token is transformed into three vectors:

- **Query (Q)** – what the token is looking for.
    
- **Key (K)** – what the token offers.
    
- **Value (V)** – the information passed along.
    

Attention score:

# [  
\text{Attention}(Q,K,V)

\text{softmax}\left(  
\frac{QK^T}{\sqrt{d_k}}  
\right)V  
]

Steps:

1. Compare Query with every Key.
    
2. Compute similarity scores.
    
3. Apply softmax to obtain attention weights.
    
4. Compute a weighted sum of the Value vectors.
    

---

### 4. Multi-Head Attention

Instead of computing one attention pattern, the Transformer computes several in parallel.

Each head can learn different relationships, such as:

- grammar
    
- subject–verb agreement
    
- long-range dependencies
    
- semantic similarity
    

Example:

```
Head 1 → syntax
Head 2 → pronouns
Head 3 → sentiment
Head 4 → entities
```

Their outputs are concatenated and projected into a single representation.

---

### 5. Feed-Forward Network (FFN)

Each token independently passes through a small neural network:

```
Linear
   ↓
ReLU/GELU
   ↓
Linear
```

This adds nonlinearity and increases the model's expressive power.

---

### 6. Residual Connections and Layer Normalization

Each sublayer uses:

```
Output = LayerNorm(Input + Sublayer(Input))
```

This helps:

- stabilize training,
    
- preserve information,
    
- enable very deep networks.
    

---

## Encoder vs. Decoder

The original Transformer has two parts:

### Encoder

```
Input
 ↓
Encoder Layer × N
 ↓
Encoded Representation
```

Used for understanding text.

Examples:

- BERT
    
- Sentence embeddings
    
- Classification
    
- Translation (encoding source text)
    

---

### Decoder

```
Previous Output
       │
Masked Self-Attention
       │
Cross-Attention
       │
Feed Forward
       │
Next Token Prediction
```

The decoder generates text one token at a time.

Examples:

- GPT
    
- ChatGPT
    
- Language generation
    

The decoder uses **masked self-attention**, which prevents it from "seeing" future tokens during generation.

---

## Types of Transformer Models

|Model|Architecture|Purpose|
|---|---|---|
|Encoder-only|Understanding|Classification, search, embeddings|
|Decoder-only|Generation|Chatbots, code generation, text completion|
|Encoder–decoder|Both|Translation, summarization, question answering|

---

## Why Transformers Are Better Than RNNs

|RNN/LSTM|Transformer|
|---|---|
|Sequential processing|Parallel processing|
|Struggles with long dependencies|Captures long-range relationships effectively|
|Slow training|Faster training on GPUs/TPUs|
|Limited context|Can attend to all tokens in the context window|

---

## Example: Attention in Action

Sentence:

> "The trophy doesn't fit into the suitcase because **it** is too big."

When predicting what **"it"** refers to:

- Attention to **trophy**: 0.82
    
- Attention to **suitcase**: 0.12
    
- Attention to other words: 0.06
    

The model infers that **"it"** most likely refers to the **trophy**.

---

## Why Transformers Power Modern AI

Transformers excel because they:

- **Model long-range dependencies** using self-attention.
    
- **Train efficiently** by processing sequences in parallel.
    
- **Scale effectively** to billions or even trillions of parameters.
    
- **Generalize across domains**, enabling applications in language (GPT), vision (Vision Transformers), speech, and biology.
    

These properties have made the Transformer architecture the foundation of most state-of-the-art AI systems today.