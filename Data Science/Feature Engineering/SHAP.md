### Overview
SHAP explains a prediction using:

$$\text{Prediction} = \text{Base Value} + \text{Feature Contributions} $$

For example:

$$1250 = 1000 + 150 + 80 - 30 \+ 50$$

Base Value: Models starting point before considering the specific feature. It is related to model's average prediction over a background dataset.

1. First We will train our model
2. Then we give it to SHAP explainer
```
model = xgboost.XGBRegressor().fit(X,y)
explainer = shap.Explainer(model)
shap_values = explainer(X)
shap_values.shape
```
3. After this we do is plotting
	1. Waterfall plot it will look something like this:
	2. This is only for one row we can do is aggregate the mean of absolute values to get the contribution of each of the values.![[Pasted image 20260911080628.png]]
	3. Force plot: Rotates the above plot by 90 degrees:
	4. It give median pricing at the centre of blue and red.![[Pasted image 20260911080925.png]]
	5. We can also understand in this plot how every feature interacts with other feature.
	6. Beeswarm : This will give overview of features that are most important for the model. It uses SHAP values to show the distribution of the impacts each feature has on the model output.
	7. Here they are sorted based on which are more important to low important;Latitude is most important, Population least important.![[Pasted image 20260911081607.png]]

### Basic Mathematics Behind it
It relies on Shapley values from cooperative game theory, where the "game" is predicting an outcome, and the "players" are the feature values of that specific record
The fundamental equation of SHAP is the **efficiency property**, which states that the sum of the SHAP values of all features plus a base value must equal the model's actual prediction for that record.
$$
\sum _{i=1}^{M}\phi _{i}(x)=f(x)-E[f(x)]
$$
Alternatively, it is written as an additive explanation model:  
$$ g(z^{\prime })=\phi _{0}+\sum _{i=1}^{M}\phi _{i}z_{i}^{\prime }$$

**What the variables mean:**

- f(x): The actual **prediction of the model**
- E(f(x)): The **base value** (the expected/average prediction of the model across the entire training dataset).
- phi_i(x) or phi _{i}: The **SHAP value** (attribution) for feature \(i\).
- M: The **total number of features**.
- z_{i}^{\prime }: A binary variable \(\{0, 1\}\) representing whether feature \(i\) is "present" or "observed" (for a specific record analysis, these are all \(1\)).

### Why is SHAP analysis Better?
1. It gives you local importance unlike macro-level ranking
2. **It obeys mathematical property of Consistency,** It will always be consistent with the way we have trained our model unlike Gini/ Permutation importance that have actually shown mathematical flaw.
	1. Gini = 1- (sumof)p^2 / It is for classification problem.
	2. **Permutation feature importance** measures the importance of a feature by calculating how much the model's score drops after randomly shuffling (permuting) that specific feature's values. Shuffling breaks the relationship between the feature and the target outcome
3. It also tell us how is one feature impacting, either positively or negatively unlike regular feature engineering.
4. It helps us in understanding feature interactions cleanly unlike traditional importance which blurs the lines when features depend on each other. If `Feature A` only matters when `Feature B` is present..
5. For us specifically it will give us something like this:
   In **Germany**, within the **Automotive Manufacturing** sector, the demand for **Polypropylene (PP)** went down by **1,200 metric tons** this month because the **Global Political Risk (GPR) Index** spiked by 25%, causing raw **Crude Oil feedstock prices** to increase by **$15/barrel**, which heavily suppressed local purchasing power. 
6. This will mainly help the business people understand what is happening for each use case and help them make better decision.


### 1. SHAP: It gives an explanation for Every row
Suppose you have this dataset:

|Month|GDP|Oil|Inflation|Prediction|
|---|---|---|---|---|
|Jan|6%|80|4%|1200|
|Feb|5%|90|6%|1050|
|Mar|7%|75|3%|1350|

SHAP produces something conceptually like this:
#### Row 1

| Feature   | SHAP Value |
| --------- | ---------- |
| GDP       | +120       |
| Oil       | +50        |
| Inflation | +30        |
The Same features can have different importance and different effects for different rows.
Across all possible combinations of features, how much did this features fairly contribute to this prediction? This comes from game-theory.
In above GDP receives more credit because considering different feature combinations as it contributed more to increasing the prediction.

### 2. The mathematical definition

The SHAP value for feature ii is:
$$
\phi_i = \sum_{S \subseteq F \setminus \{i\}} \frac{|S|!(M-|S|-1)!}{M!} \left[ f(S \cup \{i\}) - f(S) \right]
$$
#### F
All features.$$F = \{GDP, Oil, Inflation\}$$
#### S
```
Nothing
Oil only
Inflation only
Oil + Inflation
```
$$f(S∪{i})−f(S)f(S \cup \{i\}) - f(S)$$
> How much does the prediction change when feature ii is added?

SHAP calculates this contribution across **all possible feature combinations** and averages them fairly.

### 3. The most important property: Additivity
SHAP values satisfy:
$$f(x)=E[f(X)]+∑i=1Mϕif(x) = E[f(X)] + \sum_{i=1}^{M}\phi_i$$

Where:
- f(x)f(x) = prediction for this row
- E[f(X)]E[f(X)] = base value
- ϕi\phi_i = SHAP value of feature ii

In simple terms:
$${ Prediction = Base\ Value + Sum\ of\ SHAP\ Values }$$
> Positive SHAP value GDP: GDP pushed prediction higher, Negative means it pushed prediction lower.

### 4. Local Explanation vs Global Explanation
Local: It explains one prediction, Global explains the whole model.
$$ Global Importancei​=mean(∣SHAPi​∣)$$
### 5. Common SHAP plots
#### Waterfall Plot
This explains one predictions.
```
Base Value        1000
GDP              +150
Manufacturing    +100
Inflation         -50
Oil               +50
                 -----
Prediction       1250
```

This is probably the best plot for explaining a single forecast.
#### Summary Plot
This explains the entire model
Conceptually:

```
GDP              ●●●●●●●●●
Manufacturing    ●●●●●●●
Oil              ●●●●●
Inflation        ●●●
FX               ●●
```

It helps answer:

> Which features influence the model most?

#### Beeswarm plot
For each feature, you see:
- Every dot = one row
- X-axis = SHAP value
- Colour = Actual feature value

It reveals patterns like:
> High GDP values generally push predictions upwards, or other pushes predictions downwards.


#### SHAP values are NOT same as feature values.
It gives the features contribution to this specific prediction.

### 6. How SHAP work with different models.
1. Tree Models:
	1. Use shap.TreeExplainer()
	2. This is highly optimised
2. Linear Models:
	1. shap.LinearExplainer()
3. Any model
	1. shap.KernelExplainer()

### 7. Basic Python example

Suppose you have a trained XGBoost model.

```
import shap

# Create SHAP explainer
explainer = shap.TreeExplainer(model)

# Calculate SHAP values
shap_values = explainer.shap_values(X_test)
```

The output conceptually looks like:

```
Rows × Features
```

For example:

```
shap_values.shape
```

Could be:

```
(1000, 10)
```

Meaning:

- 1,000 predictions
- 10 features
- One SHAP contribution for every feature of every prediction

There is modern SHAP API which is new API and is generally very clean.
```
import shap
explainer = shap.Explainer(model, X_train)
shap_values = explainer(X_test)
```
Then:
#### Global importance

```
shap.plots.bar(shap_values)
```

#### Summary/beeswarm

```
shap.plots.beeswarm(shap_values)
```

#### Single prediction

```
shap.plots.waterfall(shap_values[0])
```


### 8. SHAP and feature interactions
Feature interactions can also be investigated by SHAP.
This is feature interaction.

If the features are highly co-related then:
SHAP then has a difficult question:
> If GDP and Industrial Production both contain similar information, how should the model's contribution be divided between them?

Different SHAP configurations and background assumptions can distribute attribution differently.
Therefore:
> **Do not blindly interpret SHAP values as absolute economic truth when features are highly correlated.**

### 9. SHAP vs traditional feature importance

|Feature Importance|SHAP|
|---|---|
|Usually global|Local and global|
|One value per feature|One value per feature per row|
|Shows importance|Shows direction and magnitude|
|Often model-specific|More unified framework|
|May not explain individual predictions|Explains individual predictions|
|Can be inconsistent|Has strong theoretical properties|

# The one-sentence definition

> **SHAP is a method for breaking a model's prediction into the individual contributions of each input feature, so you can understand why the model made that specific prediction.**


