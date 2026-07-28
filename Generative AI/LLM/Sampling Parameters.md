# 1. Temperature
Temperature controls **randomness**.
![[Pasted image 20260624225418.png]]
Temp =0 means deterministic and 1 = Balanced, >1 Random

# 2. Top-K Sampling
Instead of considering all tokens keep only top K most probable tokens.

# 3. Top-P (Nucleus Sampling)
Top-P is smarter than Top-K.
Instead of choosing a fixed number of tokens:
Choose enough tokens to reach probability P.

Choose the top tokens whose sum adds to probability(P).
This makes it adaptive to the confidence of the model.


## Together
temperature = 0.7
top_p = 0.9
top_k = 50

"Model outputs probabilities" -> "Apply Temperature" -> "Apply Top-K" -> "Apply Top-P" ->"Sample token"