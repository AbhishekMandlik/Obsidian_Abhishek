### Cross-selling
Complementary products that go well what the customer is already buying.
Check for confidence(A->B)
```
Confidence(A->B)=Support(A intersection B)/Support B
```
Business Goal:
Increase
1. Average Order value
2. Revenue
That too without acquiring new customers.

### Upselling
Encouraging customers to buy a better, more premium version or more expensive version of what they intended to purchase.
It often uses:
1. Customer segmentation.
2. Purchase history.
3. Predicted willingness to pay/
4. Recommendation System.
rather than only association rules.

### Product Recommendation
What the customer is likely to purchase next.
Types:
1. Collaborative Filtering: People similar to you liked these things.
2. Content-Based: Product similar to what you viewed.
3. Market Basket Analysis: People buying coffee also buy some other things, for that you have metrics like support, confidence, lift(removes bias from confidence)
All have the same goal that is to increase clicks, purchases, engagement.

### Store Layout Optimisation
Which products should be kept where in a the supermarket, sometime things that are essentially bought together are kept far so that customer intentionally walks the whole supermarket looking for it.
This is done by looking for High confidence and High support.

### Bundle Creation
Instead of selling the products separately, sell them together.
Example:
Laptop Bundle:
- Laptop
- Bag
- Mouse
- Charger
Benefits of this is 
1. Customer Convenience.
2. Larger Sales.
3. Clears inventory.
4. Better margins and better preparation for the next order.

### Coupon Targeting
Instead of giving coupons to everyone give them to people who are likely to buy.
It has benefits like:
- Higher coupon redemption
- Lower marketing cost
- Higher ROI

It uses 
- Association rules.
- Customer Segmentation.
- Purchase history.
- Propensity models.

### Inventory Planning
If product A sells what else should we be stocking.
1. MBA helps estimate.
2. Associated demand by using support or confidence, or lift.
3. Time series forecasting.
4. Seasonal demand prediction.
5. Demand forecasting models
To ensure complimentary products are also available together.

This is connected to [[Conditional Probability-Bayesian]],[[Common business applications]],[[Customer Analytics, CRM Analytics, Recommendation Systems, and Marketing Data Science.]]