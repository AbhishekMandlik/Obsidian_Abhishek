# Decision Trees
A Decision Tree is a model that repeatedly asks questions to split data into smaller and purer groups.
They try to maximise purity after every split.
Which means decrease in Entropy.
Entropy measures the randomness or impurity of a node. Lower entropy means purer data
Formula:
```
Entropy = -Σ p log₂(p)
#p: The probability of a specific outcome occurring (0<p<1).
```
Information Gain
The most important Decision Tree concept
After splitting how much info did we gain?
```
Information Gain = Parent Entropy - Weighted Child Entropy
```

How does a Decision Tree choose a split?

> The tree evaluates candidate splits and selects the one with the highest Information Gain (or lowest impurity).

Gini ImpurityUsed by Scikit - learn.
```
Gini = 1 - Σ p²
```

**Decision Stump**
> A Decision Stump is a one-level decision tree containing only a single split.


# Random Forest
Many Decision Trees
For the data sets we will be doing random sampling with replacement. (Bootstrapping)
Reduce overfitting.
Also we will be doing random feature selection.
Reduce correlation between trees <-> Reduce variance.
Then we will combine the result. (Aggregation)
Bootstrapping+Aggregation=Bagging

>Random Forest reduces overfitting by averaging predictions from multiple de-correlated decision trees trained on bootstrap samples.

# XG - Boost
Very Fast, Very Accurate, Takes care of bias-variance tradeoff (Immune to the curse of dimensionality)
```
Trees built sequentially instead of like Random Forest where they are built independently.
```
Tree 1 Predicts, Train Tree2 on mistakes
#### Regularisation
Why XGBoost became famous.
It actively prevents overfitting.
Methods:
```
Max Depth
Learning Rate
L1
L2
Subsampling
```

>Random Forest builds many independent trees using bagging and combines them through voting. XGBoost builds trees sequentially, where each new tree learns the residual errors of previous trees. XGBoost generally achieves higher accuracy but requires more tuning and training time.


```
Decision Tree
→ Splits data recursively

Entropy
→ Measure of impurity

Information Gain
→ Reduction in impurity after split

Gini
→ Alternative impurity metric

Tree Depth
→ Number of levels

Deep Trees
→ Overfit (High Variance)

Decision Stump
→ Tree depth = 1

Random Forest
→ Many trees + voting

Bagging
→ Bootstrap + Aggregation

Bootstrap Sampling
→ Sample with replacement

Feature Randomization
→ Random subset of features

XGBoost
→ Sequential boosting

Residual
→ Actual - Prediction

Boosting
→ New trees correct old errors

Random Forest
→ Reduces variance

XGBoost
→ Reduces bias and variance

AdaBoost 
→ Sequentially trains weak learners, increasing the weight of misclassified samples so future learners focus on difficult examples.
```


# Key Difference from AdaBoost and XG Boost

AdaBoost:

```
Increase sample weights
```

XGBoost:

```
Predict residual errors
```

# AdaBoost vs XGBoost

| AdaBoost            | XGBoost                  |
| ------------------- | ------------------------ |
| Uses sample weights | Uses residuals/gradients |
| Usually stumps      | Usually deeper trees     |
| Less accurate       | More accurate            |
| Less regularization | Strong regularization    |
| Sensitive to noise  | More robust              |
| Older algorithm     | Industry standard        |
