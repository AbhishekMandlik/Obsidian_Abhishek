If we have got a lot of data, MBA does association analysis which means that it answers one simple question that is "what products tend to be bought together?"
Real world example is that Amazon's Frequently Bought together sections or Netflix's user also watched these shows.
It's a recommendation model. 
What do companies use it for:
1. Cross-selling.
2. Upselling.
3. Product recommendations.
4. Store layout optimisation.
5. Bundle creation.
6. Coupon targeting.
7. Inventory planning.
eg. Buy diapers and get wipes a 10% off.


###  1. Association Rule Mining:
Meaning
People buying bread frequently buy butter.
We are trying to discover relationships in items.
This is called as Association Rule Mining.
A->B. If A is bought, B is often bought as well.

Not causation. It means that people who buy bread also buy butter it does not mean that that bread causes butter purchases this is a wrong notion.

### 2. Frequent Itemsets:
An itemset like for eg. Milk and Bread appear together frequently.
It is important because generating rules for every possible combination is impossible and time/space consuming. So generating rules for only those itemsets drastically reduce the search space.
### 3. Support:
It asks how frequently does this itemset appear?
Formula 
```
Support(A)=(Transactions containing A)/ (Total transactions)
# It can be for a single item or it can be for multiple items like support for items containing A and B, A or B.
```
> Why is support important:
> 	Imagine this that Confidence(Champagne-> Caviar) =100% but people buying those things are only 2, so this rule is based on almost no data which is statistically insignificant. Support introduces the statistical significance  in it.


### 4. Confidence:
If someone buys A how likely are they to buy item B,
Formula
```
Confidence(A->B)=Support(A intersection B)/Support A
```
> Limitation of it is that it ignores if Item A is already popular and its confidence with anything would be very high, so if you have to do cross-selling or sell it  with any other thing Confidence will not give you correct answer regarding it.


### 5. Lift:
If Item B is bought quite frequently then Confidence(A->B) will be naturally high which can be misleading in many scenario's. So to correct this we use Lift parameter:
```
Lift = Confidence(A->B)/Support(B)
Lift = Support(A intersection B)/Support(A)*Support(B)
```

```
if(lift==1){
	return Independent;
}
else if(lift>1){
	return Positive association;
}
else return Negative Association;
```


### 6. Apriori Algorithm:
It finds frequent itemsets efficiently.
> Principle If an itemset isn't frequent none of it's supersets can be frequent.
> It results in a huge speedup
#### How Apriori algorithm works:
1. Find frequent individual items, remove the rare one
2. Generate pairs- Remove the infrequent pairs.
3. Generate triplets from the remaining survivor pairs- and at each step similarly prunes as many impossible condition as possible.
Drawbacks:
- It scans the database repeatedly and generates many candidate itemsets, which become expensive on large datasets.
### 7. FP - Growth
Avoids repeated database scans which makes it faster than that of Apriori.
It uses FP Tree. (Frequent Pattern Tree)
This is a trie data structure:
> FP-Growth is generally faster than Apriori because it compresses transactions into a tree instead of generating many candidate itemsets.

Intuition: Suppose many customers buy: one itemset these prefixes are stored in a tree rather than repeated for every transaction, Compressions reduce memory usage and the number of database scans.

Advantages:
1. Much faster on larger datasets.
2. Does not generate huge number of candidate itemsets.
3. Requires only 2 scans on the database.
Disadvantages:
4. More Complex to implement.
5. It can me memory intensive for highly diverse datasets

### 8. ECLAT:
### 9. Rule Pruning
Thousands of rules get generated so we need to keep only the useful ones. So we use thresholds. Keep only rules where:
1. Support > 5%
2. Confidence > 60%
3. Lift > 1.5
4. Removing redundant rules:
   ```
   Example:
   1] Bread-> Butter
   2] Bread,Milk -> Butter
   ## second rule does not add much value to it if they have nearly the same confidence, so prune the second rule.
   ```

### 10. Choosing Thresholds
```
Support too low, Millions of rules, Lots of noise
Support too high, Miss useful niche products, Need balance.
```
### Business Trade-off:
- Large supermarket: Higher support may be reasonable.
- Luxury retailer: Lower support may still uncover profitable relationships.

This is related to [[Customer Analytics, CRM Analytics, Recommendation Systems, and Marketing Data Science.]], [[Advanced Bayesian Concepts]],[[Common business applications]].
