1. Hypothesis Testing:
   A statistical method used to determine whether an observed effect is likely real or just due to a random chance.
	1. Null Hypothesis (h°): Nothing changed.
	2. Alternative Hypothesis (h¡): Something changed.
   Goal is to check if there is enough evidence to reject h°.
   >It is statistical procedure used to determine whether the observed differences are statistically significant or could have occurred by random chance.
   
2.  p- value:
   Probability of observing the results at least this extreme assuming that the NULL hypothesis is true. It does not mean probability that the hypothesis is true.
   if we get =="p=0.02"== If there was no improvement that means that null hypothesis  says that there is only 2 % chance of seeing results this good. That is why we reject h°.
   _typical threshold-> œ=0.05_. If p≤5% then reject the null hypothesis.
   >A p-value measures how likely the observed data would occur if the null hypothesis were true.

3. Confidence Interval: 
   Instead of predicting one exact number we predict a range of numbers.
   Meaning that we are reasonably confident that the true average lies within this interval.
   >A confidence interval provides a range of plausible values for an unknown population parameter. Correct statement to describe CI of 95% is that: If we repeated the experiment many times, 95% of those intervals would contain the true parameter.
   
4. A / B Testing:
   Comparing two versions of something.
   Common parameters to measure:
	1. CTR
	2. Revenue
	3. Session Length
   Common A/B Testing pitfalls:
	- Pitfall 1 - Stopping too early, we need a sufficient sample size.
	- Pitfall 2 - Multiple comparisons (Need correction e.g. Bonferroni)
	- Pitfall 3 - Selection bias 
	  e.g. if desktop version uses one thing and mobile version uses the same thing so for that we need random assignment.
	- Pitfall 4 - Novelty effect:
	  Users click on it more simply because it is new, so check the retention of users.
	- Pitfall 5 - Seasonality
5.  BIas - Variance Tradeoff: Look at [[Foundations]].
6. Regularisation: L1 useful for feature selection as it can reduce the coefficients to exactly zero and L2 penalises large weights so it keeps all features but weights become smaller.
   > Regularisation reduces overfitting by penalising model complexity during training.
7. Cross Validation:
   Instead of one train-test split we can do is 5-fold CV. If we have sample split it into 5 folds and then train on 4 and test on 1, do this multiple times until all are tested and then do the averaging of all.
