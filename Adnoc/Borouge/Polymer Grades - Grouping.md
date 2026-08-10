## Naive Approach

**1. Group by chemical family alone (PE, PP, PVC, PET...)**  
This is almost always the first instinct, because it's how the plant, the sales catalog, and the ERP already organize SKUs. The problem: chemical family says nothing about how demand _behaves_. A PE film grade for FMCG packaging and a PE pipe grade for construction end up in the same "PE model," even though one is smooth and consumer-seasonal and the other is lumpy and capex-driven. You get a model that's mediocre for both instead of good for either.

**2. Group by application/end-use only (packaging, automotive, construction...)**  
Better than pure chemistry, but still naive if stopped there — it mixes your top-volume, smooth grade with a niche, order-once-a-quarter grade in the same application bucket just because they're both "used in packaging." The application tells you the _right driver_, but not whether the series is even forecastable with a standard model.

**3. Group purely by volume (top sellers vs. rest, or ABC only)**  
Common because it's a one-line sort in Excel. It captures "where the money is" but ignores demand _shape_ entirely — a group could contain both a perfectly smooth top-seller and an erratic one, which again need different methods even though they're both high volume.

**4. Group by business unit / sales team / plant that produces it**  
Purely organizational, has nothing to do with demand statistics or causal drivers at all — it's grouping by who owns the P&L, not by what would make a model accurate. Very common in practice because it matches how forecasts get _reviewed_, even though it has no bearing on how they should be _built_.

**5. Group alphabetically / by SKU code / by pack size**  
The most naive version — essentially random with respect to demand behavior, but surprisingly common when someone just needs to split 200 SKUs into "5 buckets" for workload division rather than for modeling accuracy.

**6. Eyeball the historical charts and cluster "by feel"**  
A step up from the above because it's at least looking at the data, but it doesn't scale past a handful of SKUs, isn't reproducible, and tends to overweight whichever 2-3 years happen to be visually memorable (e.g. a COVID spike) rather than the underlying statistical pattern.

**Why these all fall short, in one sentence:** none of them separate the two questions that actually determine forecast accuracy — _is this series statistically forecastable in a regular way_ (ADI/CV², what we plotted earlier) and _what causes its demand_ (application/driver) — so they either group on the wrong axis entirely (org chart, alphabet) or on only one of the two right axes (chemistry, application, volume) and call it done.

## Formal Approach
![[Pasted image 20260810124817.png]]

| Group        | Who's in it                                                            | Why separate                                                                                           | Method                                                                                       |
| ------------ | ---------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------- |
| 1            | Top-20% grades, smooth/seasonal, consumer-driven demand                | Highest value, needs precision, distinct seasonal driver (e.g. festive/consumer cycle)                 | SARIMA / ETS / Prophet + seasonal regressors                                                 |
| 2            | Top-20% grades, smooth/seasonal, industrial-driven demand              | Same volume tier, different driver (construction/auto capex cycles)                                    | Regression/ETS with sector-index regressors                                                  |
| 3            | Top-20% grades, erratic (frequent but volatile)                        | Volume too high to ignore, but classic time series models will underperform here                       | LightGBM/XGBoost with lag + rolling features, or a per-group global model                    |
| 4            | Mid-tail, intermittent but consistent order size                       | Distinct enough pattern to need Croston-family methods, but pool across grades to offset short history | Croston / SBA, partial-pooled across the group                                               |
| 5            | Long-tail (the ~80% of grades sharing ~20% of volume), lumpy/irregular | Not worth individual models — pool everything, forecast at family/application level, disaggregate down | TSB or a single pooled ML model (grade as categorical feature) + hierarchical reconciliation |
| 6 (optional) | New or <12-month-history grades                                        | Not enough history for any of the above                                                                | Analogous forecasting from a similar existing grade's early-life curve                       |

![[Pasted image 20260810124819.png]]
![[Pasted image 20260810125043.png]]


Here we can classify the material grades into such things based on ADI and CV^2
This will allow different models for different categories, Instead of naively classifying them based on Order volume and Sales Data.