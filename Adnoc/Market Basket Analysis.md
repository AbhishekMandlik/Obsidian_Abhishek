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


### Association Rule Mining:
Meaning
People buying bread frequently buy butter.
Not causation.

### Frequent Itemsets:
An itemset like for eg. Milk and Bread appear together frequently.

### Support:
It asks how frequently does this itemset appear?
Formula 
```
Support(A)=(Transactions containing A)/ (Total transactions)
# It can be for a single item or it can be for multiple items like support for items containing A and B, A or B.
```

### Confidence:
If someone buys A how likely are they to buy item B,
Formula
```
Confidence(A->B)=Support(A intersection B)/Support A
```

### Lift:
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


### Apriori Algorithm:
It finds frequent itemsets efficiently.
> Principle If an itemset isn't frequent none of it's supersets can be frequent.
> It results in a huge speedup

## FP - Growth
Avoids repeated database scans which makes it faster than that of Apriori.
It uses FP Tree
> FP-Growth is generally faster than Apriori because it compresses transactions into a tree instead of generating many candidate itemsets.

### Rule Pruning
Thousands of rules get generated so we need to keep only the useful ones. So we use thresholds. Keep only rules where:
1. Support > 5%
2. Confidence > 60%
3. Lift > 1.5

### Choosing Thresholds
```
Support too low, Millions of rules, Lots of noise
Support too high, Miss useful niche products, Need balance.
```

