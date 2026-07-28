**Bias**: It measures how wrong models assumptions are.
High bias means: Model is too simple, Can't learn true patterns.
High Training, Testing Error.
>Bias is the error introduced by overly simplistic assumptions in the learning algorithm.
Under fitting :- High Bias, Low Variance

**Variance**
How sensitive model is to changes in training data.
It memorises noise, Training error low; Test error high.
> Variance is the amount by which a model's predictions would change if it were trained on a different dataset.
> Overfitting :- Low Bias, High Variance

# How Do We Reduce Variance?
If model overfits:
Reduce complexity.
Examples:
```
Regularization
Pruning
Dropout
More data
Cross-validation
Ensemble methods
```

>Error= Bias^2 + Variance + noise



## Cheat Sheet
```Bias
→ Model too simple
→ Underfitting
→ High training error

Variance
→ Model too complex
→ Overfitting
→ Large train-test gap

Underfitting
→ High Bias, Low Variance

Overfitting
→ Low Bias, High Variance

Increase Complexity
→ Bias ↓
→ Variance ↑

Reduce Variance
→ More data
→ Regularization
→ Ensembles

Error
=
Bias² + Variance + Noise
```


To actively prevent overfitting.
Methods:
```
Max Depth
Learning Rate
L1
L2
Subsampling
```
Learning rate to prevent model from overshooting/ undershooting also not taking very high time to learn.
L1:  add a parameter to loss like:
```
Loss = Loss + λ Σ|w|
Penalises large weights.
```
>L1 regularisation adds the absolute value of weights to the loss function. It encourages sparse models by driving some coefficients exactly to zero.

L2:
```
Loss = Loss + λ Σw²
It shrinks Large weight does not make them 0
```
> L2 regularisation adds the squared magnitude of weights to the loss function. It reduces overfitting by shrinking weights while retaining all features.
