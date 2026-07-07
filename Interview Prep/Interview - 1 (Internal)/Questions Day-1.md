## 1 Why did Transformers replace RNNs/LSTMs?

### Short Interview Answer

Transformers replaced RNNs and LSTMs because they can process all tokens in parallel, capture long-range dependencies more effectively using self-attention, and scale much better to large datasets and models.
Issues with RNN:
1. Sequential computation (slow training)
2. Hard to parallelise on GPUs
3. Vanishing/exploding gradients
4. Struggle with long-range 
### LSTM Improvement
LSTMs introduced:
- Forget gate
- Input gate
- Output gate
to preserve long-term memory.

However:
- Still sequential
- Training remains slow
- Difficult to scale

### Transformer Solution

Transformers use self-attention:
```
Every word attends to every other word.
```
Benefits:
✅ Parallel processing
✅ Better long-range context
✅ Faster training
✅ Better scalability

## 2. What is Self-Attention?
Self-attention allows each token in a sequence to determine which other tokens are important for understanding its meaning.

## 3. What is Multi-Head Attention?

Multi-head attention allows the model to learn multiple types of relationships simultaneously by running several self-attention mechanisms in parallel.
### Benefit
Different heads learn:
- Syntax
- Grammar
- Entity relationships
- Context
- Semantic meaning
simultaneously.

## 4. What are Embeddings?
Embeddings are dense numerical vector representations of text that capture semantic meaning.
## 5. What are Positional Encodings?
Transformers process all tokens in parallel and therefore have no inherent understanding of word order. Positional encodings provide information about the position of each token in a sequence.
