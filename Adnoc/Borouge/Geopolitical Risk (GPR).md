### What do we Do?
1. Construct text-based indicator of geopolitical risk -- GPR Index -- measuring frequency of articles in leading newspapers discussing adverse geopolitical events.
2. Separate threats of adverse geopolitical events from their realisation and escalation.


### Empirical Evidence
1. High GPR --> BAD.
2. Idiosyncratic GPR reduces firm-level investment
> **Idiosyncratic GPR** means **geopolitical risks that are unique to a specific company** (not risks that affect the whole economy or industry, but risks that are specific to one firm). For example, a company operating in a politically unstable country might face unique risks that others do not. 
> **Reduces firm-level investment** means that when a company faces these unique geopolitical risks, it is likely to **spend less on new projects, equipment, or expansion**.

$$
GPR=\frac{G\,}{U}​
$$
```
G: articles mentioning adverse geopolitical events.
U: total number of articles.
```
### Selecting terms in set G
- Bag 1: Threat words, Act words.
- Bag 2: War words, nuclear words, terrorism words.

We break these indexes into GPT and GPA, Threats and acts:
Both are different as some movements in GPT may happen when no underlying act materialises or attacking without giving any warning(Terrorism)

### Geopolitical Risk and Economic Activity
-  VAR Evidence for US suggests that threats matter as much as acts.
- Panel Regressions: GPR predicts economic disasters across countries.
- Panel Quantile Regressions: How GPR affects distribution of economic variables across countries.


### Firm - Level  Effects of Geopolitical Risk

Conceptual Framework:
$$
GPR(I,t)=GPR(t)+{GPR(t)}\times{ð(k)} + Z(I,t)
$$
```
ð(k): Industry-Exposure tp aggregate GPR.
Z(I,t): idiosyncratic geopolitical risk.
Goal: To measurse effect of idiosyncratic and industry-exposure to aggregate GPR on the firm investment
```

$$
R(k,t) = α(k) + β(k)∆GPR(t) + ε(k,t)
$$
```
Λk = −sign (ß(k)-ß̐)
1. More exposed (negative beta): Entertain, Transportation, Textiles.
2. Less exposed (positive beta): Gold, Oil, Defense.
```
We Estimate:
```
log iki,t+2 = αi[+αt] + βh (Λk∆ log GPRt) + d Xi,t + εi,t+2 

αi and αk,t: firm and industry-time fixed effects
γ: response of log ik in t + 2 to change in firm-level GPR (Zi,t) in
quarter t
Xi,t: firm cash flows and Tobin’s Q, log iki,t−1
```