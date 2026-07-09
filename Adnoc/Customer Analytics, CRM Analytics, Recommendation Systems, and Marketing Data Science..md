 

### 1. Affinity
How strongly two products, services or behaviours are associated with each other. It tries to capture a meaningful relationship between two products.

### 2. Propensity Modelling
> How likely is a customer to perform a specific action.

eg. Buy, Upgrade, Churn, Renew, Click, Respond to email etc.
Instead of classifying customers as yes or no, It estimates a probability and return the value.

This is a supervised learning problem:
Input features:
- Age..
- Income.
- Purchase history.
- Number of websites visits.
- Average order value.
- Time since last purchase.
- Device type.

Models commonly used are:
1.  Logistic Regression.
2. XGBoost.
3. LightGBM.
4. Random Forest.
5. Neural Networks.

> Propensity Model ranks customers so business can target the most promising users first.

### 3. Market Basket Analysis
What combination of products appear in the same shopping basket?
Common Algorithms:
1. Apriori
2. FP-Growth
3. ECLAT

### 4. Association Rules
Support, Confidence, Lift
This is covered in [[Market Basket Analysis]]
Please look into it for better information.

### 5. Recommendation System
It suggests products or content that users are likely to prefer.
Business Goal:
Increase
1. Revenue.
2. Engagement.
3. Retention.

```
Types of recommendation models:
1. Collaborative Filtering: Uses behavior of similar users.
2. Content-Based Filtering: Uses item characteristics.
3. Hybrid Systems: Combines both.
```

### 6. Next Best Offer (NBO)
A business decision problem:
Choose the single best product or service to recommend next.
It combines:
1. Propensity score.
2. Customer value.
3. Business rules.
4. product eligibility.

### 7. Next Best Action (NBA)
Bit broader than NBO it recommends the best action that has to be done.
Possible actions:
- Offer discount
- Send reminder
- Escalate to support
- Do nothing
- Recommend a product
- Schedule a call
If a customer is likely to churn then offer a retention discount.

### 8. Customer Segmentation
Divide customers into groups with similar characteristics as different customers require different marketing strategies.
Methods:
1. Rule-based:
	1. Age
	2. Income
	3. Region
2. Machine Learning
	1. K-means
	2. Hierarchical Clustering
	3. DBSCAN
	4. Gaussian Mixture Models
Important for targeted campaigns

### 9. RFM Analysis
One of the simplest and most effective segmentation methods
1. Recency
2. Frequency
3. Monetary
Typical Segments
- Champions
- Loyal Customers
- Potential loyalists
- At-risk
- Lost customers

### 10. Customer Lifetime Value
The expected total profit or revenue from a customer over the entire relationship.
```
CLV = Average purchase Value x Purchases per year x Customer Lifetime
```
Businesses often prioritise high CLV customers

### 11. Uplift Modelling
Who will buy because of the campaign?
This distinction matters because some customers would have purchased anyway.
Common approaches:
- Two-model approach
- Uplift trees
- Causal forests
- Meta-learners (e.g., T-learner, X-learner)


### 12. Personalisation
Tailoring content, offers or experiences to an individual user.
Personalisation can use:
- Demographics
- Past purchases
- Search history
- Device
- Location
- Time of day
- Context
- Recommendation models
- Large language models for personalised content


### How these concepts connect
```
                Customer Data
                     │
                     ▼
          Customer Segmentation
                     │
                     ▼
               RFM Analysis
                     │
                     ▼
             Propensity Modeling
                     │
         ┌───────────┴───────────┐
         ▼                       ▼
   Affinity Analysis      Recommendation System
         │                       │
         ▼                       ▼
   Market Basket         Personalized Suggestions
         │
         ▼
Support • Confidence • Lift
         │
         ▼
   Next Best Offer (NBO)
         │
         ▼
   Next Best Action (NBA)
         │
         ▼
  Measure Uplift & Business Impact
         │
         ▼
 Increase CLV through Personalization
```



### Cheat Sheet:

| Term                                  | Meaning                                                     | Example                                                  |
| ------------------------------------- | ----------------------------------------------------------- | -------------------------------------------------------- |
| **Affinity**                          | Likelihood that two products are purchased together         | Customers buying an iPhone often buy AirPods.            |
| **Propensity**                        | Probability that a customer will perform a specific action  | Customer has an 82% propensity to buy insurance.         |
| **Purchase Propensity Score**         | Model output estimating purchase probability                | Customer A: 0.91, Customer B: 0.15                       |
| **Next Best Offer (NBO)**             | Best product to recommend next                              | Recommend a credit card to a savings account holder.     |
| **Next Best Action (NBA)**            | Best action to take, not necessarily a sale                 | Offer a discount, send an email, or recommend a product. |
| **Recommendation Engine**             | ML system suggesting products                               | Amazon's "Customers also bought..."                      |
| **Association Rule Mining**           | Finds products frequently bought together                   | Bread → Butter                                           |
| **Market Basket Analysis**            | Discovers purchasing patterns across transactions           | Diapers and baby wipes are commonly bought together.     |
| **Lift**                              | Measures how much stronger an association is than chance    | Lift > 1 indicates a meaningful association.             |
| **Confidence**                        | Probability of buying B given A                             | 70% of bread buyers also buy butter.                     |
| **Support**                           | Frequency of an itemset in transactions                     | 15% of baskets contain milk and cereal.                  |
| **Collaborative Filtering**           | Recommends based on similar users or items                  | Netflix movie recommendations.                           |
| **Content-Based Filtering**           | Recommends based on product features                        | Suggest more action movies to an action movie fan.       |
| **Customer Segmentation**             | Groups customers with similar behavior                      | High-value, budget-conscious, occasional shoppers.       |
| **Customer Lifetime Value (CLV/LTV)** | Expected total value from a customer                        | Premium customers have high LTV.                         |
| **RFM Analysis**                      | Segments customers using Recency, Frequency, Monetary value | Identify loyal vs. at-risk customers.                    |
| **Lookalike Modeling**                | Finds customers similar to your best customers              | Target new users resembling top spenders.                |
| **Lead Scoring**                      | Predicts likelihood of conversion                           | Sales prioritizes leads with scores above 80.            |
| **Churn Prediction**                  | Predicts which customers may leave                          | Offer retention incentives to high-risk users.           |
| **Personalization**                   | Tailors recommendations to individuals                      | Personalized homepage recommendations.                   |
| **Conversion Rate**                   | Percentage of users who purchase after a recommendation     | 8% conversion from recommendations.                      |
| **Acceptance Rate**                   | Percentage of recommendations accepted                      | 35% accepted the cross-sell offer.                       |
| **Uplift Modeling**                   | Predicts incremental impact of an intervention              | Identify customers who buy _because_ of the campaign.    |
| **Response Modeling**                 | Predicts who will respond to a campaign                     | Estimate email campaign responders.                      |
This is related to [[Customer Analytics, CRM Analytics, Recommendation Systems, and Marketing Data Science.]], [[Market Basket Analysis]], [[Common business applications]]



## Example: End-to-end e-commerce scenario

Imagine an online electronics store:

1. **Customer Segmentation** groups users into "students", "professionals", and "gamers".
2. **RFM Analysis** identifies a professional who shops frequently and spends a lot.
3. A **Propensity Model** predicts an 88% chance they'll buy a wireless mouse after purchasing a laptop.
4. **Market Basket Analysis** shows laptops and wireless mice have high **Affinity**, with strong **Support**, **Confidence**, and **Lift**.
5. The **Recommendation System** surfaces three compatible mice.
6. The **Next Best Offer** is a premium wireless mouse with a 10% discount.
7. The **Next Best Action** is to send that offer via email within 24 hours of the laptop purchase.
8. The email content is **Personalized** using the customer's browsing history.
9. An **Uplift Model** determines this customer is likely to buy _because_ of the discount, making the campaign worthwhile.
10. The successful cross-sell increases the customer's **Customer Lifetime Value (CLV)**.