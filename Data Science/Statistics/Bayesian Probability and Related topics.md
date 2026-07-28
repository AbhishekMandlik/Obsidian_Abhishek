### 1. Conditional Probability:
What is probability of A occurring given B has occurred.
$$P(A|B)= P(A ∩ B) / P(B)$$
Joint probability divided by probability of the condition.
$$P(A|B) != P(B|A)$$
Multiplication Rule:
$$
\begin{aligned}
P(A \cap B) &= P(A|B) \cdot P(B)  \\
P(A \cap B) &= P(B|A) \cdot P(A) \\
\end{aligned}
$$

So we get

$$
\begin{aligned}
P(A|B) \cdot P(B) &= P(B|A) \cdot P(A)
\end{aligned}
$$
This identity is the foundation for:
1. Bayes Theorem.
2. Bayesian Networks.
3. Hidden Markov Models.
4. Chain Rule of Probability.
5. Many generative models.

### 2. Independence
Definition: Two events are independent if knowing if probability of one event does not change probability of the other.
that means that:
$$
P(A|B)=P(A)
$$
Equivalent test to this would be that:
$$
P(A ∩ B)= P(A) *P(B)
$$


### Bayesian Probability: A Different Way of thinking
#### Two major schools of probability
#### 1. Frequentist Probability
Probability based on long-run frequencies
#### 2. Bayesian Probability
It represents how strongly we believe in something given the information we currently have and as new evidence arrives we update that belief.
It has bayesian update cycle: keep on updating the belief as and when new information comes to light.


### Bayes's Theorem
#### The Formula
$$
P(A|B)=\frac{P(B|A)\,P(A)}{P(B)}​
$$
Let's translate each term into plain English.
$$
{ \text{Posterior} = \frac{\text{Likelihood}\times\text{Prior}} {\text{Evidence}} }​​
$$
This is often the easiest way to remember it.
$$
{ \text{P(B|A)} = \frac{\text{P(A|B)}\times\text{P(B)}} {\text{P(A|B)}\times\text{P(B)}+ \text{P(A|Bº)}\times\text{P(B°)}} }​​
$$
```if P(B) = probability of not having diseases, p(B°)= probability of not having diseases```
>This is true for n types of items not only having and not having....

 