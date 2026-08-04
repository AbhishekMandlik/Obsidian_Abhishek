The intermittent demand problem....

1. Percentage of time their is no demand.
2. Average Demand Interval: On average how much do we wait before sales happen.
3.  CV squared: Size of the order jumps around... how much is the change in demand when the demand does come.

Simple smoothing method that we use:
Single exponential smoothing, It updated every period including the zeroes.
Failures:
1. Always lagging as the forecast chases the last event instead of leading it.
2. Time-dependent: The answer changes based on the time you ask it, so it is not real demand.
3. Long gaps between the orders might decay the demand to zero which can potentially starve the stock and harm the prediction.


Split the signal into two parts and separate the demand into two part:
1. How big is the order when the order occurs?
2. How many periods between the order?
 Per-period forecast:
 forecast for each month is: Expected size/Expected Gap.

Two smoothing equations:
```
## When Demand Occurs:
1. Z(t)=œ·Y(t) + (1-œ)·Z(t-1)
2. pt = œ·q + (1-œ)·p(t-1)
   
## When demand is 0:
Y(t)=0
Z(t)=Z(t-1) and p(t)=p(t-1) and q=q+1
Nothing updated estimates are frozen untill the next order.

Z= smoothened demand size.
p smoothened inter-demand interval.
q periods since the last demand.
œ smoothing constant, typically 0.05-0.2
```
Known bias and one-line curve
1. Croston over-forecast: The average of z/p is not the same as dividing of the average which un-intentionally creates a positive bias....
2. Their is a one line fix to this Syntetos-Boylan Analysis; it is used to remove the positive bias that is present in itself.
![[Pasted image 20260730131812.png]]
3. TSB: when parts stop selling for good, its forecast decays to zero instead of being stuck up like the Crostons.  It is for those items that may stop selling in the future. 

We classify these items before deciding weather to use classical, SBA or TSB:
![[Pasted image 20260730132137.png]]
Classify and then forecast for the best results...
Slow movers or spare parts it is the most important to look at it.
Its lead-time forecasting and not the current forecast.









