### Quintiq Forecast, Statistical Forecast (Baseline) and Historical Consensus Forecast
Columns present:
1. Forecast Series (e.g., 100000/01-BD: Consensus Forecast Weekly_Volume (MT)): Identifies the product/planning node/market combination for which the forecast is created.
2. Aug-26, Sep-26, Oct-26 ... Mar-29: Forecast month.
3. Values under each month: Forecast quantity for that month in Metric Tons (MT).

What this dataset contains:
- Future demand forecast
- Product/planning combination
- Market/customer/region code (BD, BH, BI, etc.)
- Monthly forecast volume (MT)
- Consensus forecast (final agreed forecast used for planning)

### Invoice / Sales History (Monthly) and Product and country 

#### Sales History (2024-2026)
1. **Super Region (Ship To)**: Highest geographical region (e.g., Asia, Europe, Middle East).
2. **Super Region 1 (Ship To)**: Sub-region within the Super Region.
3. **DP (Mat Sales)**: Product family/business product category.
4. **Mobility (unnamed column)**: Likely indicates the Mobility segment/category for all records.
5. **Market Centre**: Sales market or commercial region.
6. **Material**: Product code.
7. **Grade**: Product grade name.
8. **Country (Ship To)**: Customer destination country.
9. **Calendar (Year/Month)**: Sales period (e.g., Jun-2023).
10. **Invoiced (TO): Numeric Value**: Actual invoiced sales quantity, typically in Metric Tons (MT).
**In brief**
This dataset tells:
11. **Which product/grade was sold**
12. **In which country/region**
13. **In which month**
14. **How much was invoiced**
#### Sap- Sales Invoice Transaction 2024-2026
**Main columns and meaning**
1. **Sales Organization / Sales Office / Sales District / Sales Group**: Sales team responsible for the sale.
2. **Purchase Order No.**: Customer's PO reference.
3. **Sales Document / Order Item**: Sales order number and line item.
4. **SO It Created On**: Order creation date.
5. **Document Currency / Local Currency**: Currency used.
6. **Req. Dlv. Dt / Delivery Date**: Requested and actual delivery dates.
7. **Incoterms / Inco. 2**: Delivery terms and destination.
8. **Payment Terms**: Customer payment conditions.
9. **Billing Document / Billing Date**: Invoice number and invoice date.
10. **Sold-To-Party / Ship-To-Party / Payer**: Customer identifiers.
11. **Name1 / Ship-To-Party Name**: Customer name.
12. **Country / Ship To Country**: Customer country.
13. **Material Number / Material Description**: Product sold.
14. **Plant**: Manufacturing or supplying plant.
15. **Material Group 1 / Material Group 3**: Product classifications.
16. **End Use**: Application/end-use segment.
17. **Market Area**: Sales market region.
18. **Sales Unit**: Unit of measure (MT).
19. **Delivery Number**: Delivery document number.
20. **Customer Group 9 / CG9 Description**: Customer segment/category.
21. **Order Quantity**: Quantity ordered.
22. **Billed Quantity**: Quantity invoiced.
23. **Created By**: User who created the billing document.

**In brief**

This dataset tells you:
1. **Who bought** (customer)
2. **What was bought** (product)
3. **How much was bought** (quantity)
4. **Where it was sold** (country/market)
5. **When it was delivered and billed** (dates)
6. **Which sales team handled it**

### Freight Rate
1. Mass Upload - Land
   Contains Columns:
	1. Plant.
	2. Route.
	3. Cnty Type.
	4. Vendor.
	5. Fr. Rates.
	6. Currency (mostly in AED).
	7. No. of Units
	8. Val Start
	9. Val End
	10. Vendor_
	11. Cnty Type_
	12. Type
	13. Source Code
	14. Plant Code
	15. Dest Code
	16. Country
	17. Dest Name
2. Consolidated_Data_output
   Contains Columns:
	 - **Country of Origin**: The country from which the shipment starts.
	- **Port / Place of Loading**: The port where the container is loaded onto the vessel.
	- **Country of Destination**: The country where the shipment is being sent.
	- **Port / Place of Discharge**: The port where the container is unloaded.
	- **Borouge Route Description**: Borouge's description/name for the shipping route.
	- **Source Code**: Internal code representing the origin location.
	- **Destination Code**: Internal code representing the destination location.
	- **Trade**: Trade lane or shipping corridor (e.g., GCC-Europe, Middle East-Asia).
	- **Bunker Trade Factor**: Fuel-related surcharge factor used in freight calculations.
	- **DTHC included in AIFR**: Indicates whether Destination Terminal Handling Charges are included in the freight rate.
	- **FREE DAYS at Destination**: Number of days containers can stay at destination without incurring storage charges.
	- **Forecast1**: First forecasted freight rate/value.
	- **Forecast2**: Second forecasted freight rate/value.
	- **Transit Time (In Days)**: Expected shipping duration from origin to destination.
	- **All-In-Rate (AIR) 20' US$**: Total freight charge for a 20-foot container.
	- **All-In-Rate (AIR) 40' US$**: Total freight charge for a 40-foot container.
	- **NOR All-In-Rate (AIR) 20' US$**: Freight rate for a 20-foot NOR (Non-Operating Reefer) container.
	- **NOR All-In-Rate (AIR) 40' US$**: Freight rate for a 40-foot NOR container.
	- **Surcharge Rate for 20' (USD)**: Additional charges applicable to a 20-foot container.
	- **Surcharge Rate for 40' (USD)**: Additional charges applicable to a 40-foot container.
	- **Total with Surcharges for 20' (USD)**: Final cost for a 20-foot container after adding surcharges.
	- **Total with Surcharges for 40' (USD)**: Final cost for a 40-foot container after adding surcharges.
	- **CHECK**: Validation field used to verify calculations or data quality.
	- **Comment**: User comments or notes.
	- **Remarks**: Additional observations or explanations.
	- **SourceFile**: Original file from which the record was extracted.
	- **SheetName**: Excel sheet/tab from which the record was extracted.


### Customer Regularity & Segmentation and Customer Domain Classification 


**Main column groups**

**Sales & Order Details**
1. Sales Organization, Sales Office, Sales Group
2. Sales Document, Order Item
3. Purchase Order No.
4. Order Quantity, Billed Quantity
5. Billing Date, Delivery Date
→ Actual sales transactions.

**Customer Details**
1. Sold-To-Party
2. Ship-To-Party
3. Customer Name
4. Country
5. Market Area
6. Customer Group 9
→ Who bought and from where.

**Product Details**

1. Material Number
2. Material Description
3. Grade
4. Material Group 1
5. Material Group 3
6. End Use
→ What product was sold.

**Customer Regularity Metrics**
1. SO_Min / SO_Max
2. DaysFrom_Last_Trans
3. Cust_Seg
4. Dist_SO_Orders_By_Cust
5. Cust_Orders_Category
6. Regular/NonRegular
7. Repete_Monthly
8. Repete_Qtr

**Monthly Purchase History**
1. Month-1 to Month-12
2. month1_Qty to month12_Qty
3. Total_month_Qty_Trans
4. Total_month_Qty_last3mnths
→ Customer purchase quantities over the last 12 months.

**Quarterly Analysis**
1. Q1_Qty_Trans
2. Q2_Qty_Trans
3. Q3_Qty_Trans
4. Q4_Qty_Trans
→ Quantity purchased in each quarter.

**Predictive Columns**
1. Next_Expected_SO_MinRng
2. Next_Expected_SO_MaxRng
3. Probability_Score-NED
→ Predicted next order date range and likelihood of ordering.

**Customer Classification**

1. Mapping_CLTV
2. Billed_Qty_cat
3. CustomerGrp
4. Material_Type
5. CRP
→ Customer value and segmentation categories.

**In brief**

This dataset tells:

1. **Who bought what**
2. **How much they bought**
3. **How often they buy**
4. **Whether they are regular or irregular customers**
5. **When they are likely to place the next order**
6. **Customer segment and loyalty/profile information**

### ASP Forecast Material Pricing
**Column Descriptions**

1. **ASP_Forecast_Aug2026-Feb2028**: Name of the forecasting model/dataset. Contains forecasted ASP values from Aug-2026 to Feb-2028.
    
2. **Mapping**: Product or planning hierarchy identifier (e.g., `AE100000/01`). Identifies the product, grade, or forecast node for which ASP is being forecasted.
    
3. **Scenario**: Forecast assumption used.
    
    - **Base**: Most likely forecast.
        
    - **Severe**: Alternative scenario (often higher/lower depending on business assumptions).
        
4. **2026-08, 2026-09, ..., 2028-02**: Monthly forecasted ASP values representing the expected selling price for that month (e.g., `Aug-26 = 1048.1`, `Sep-26 = 1097.2`).
    
5. **Grand Total**: Aggregate or average ASP across the forecast horizon (depends on how the report was configured), used as a summary measure.
    
6. **Latest_MA_3_Ref**: Latest 3-period Moving Average reference. Historical ASP benchmark used to generate or validate the forecast.
    
7. **Latest_Qty_MA_3_Ref**: Latest 3-period Moving Average quantity reference. Historical sales volume benchmark used alongside ASP forecasting.
    
8. **Rich_Sparse**: Data quality / forecasting classification indicating how much historical data exists:
    
    - **Rich**: Sufficient historical data available.
        
    - **Sparse**: Limited historical data available.
        

**What is Present in This Dataset?**

For each product/planning node, the dataset contains:

1. Product / forecast mapping (`AE100000/01`)
    
2. Different forecast scenarios (Base, Severe, etc.)
    
3. Monthly future ASP forecasts
    
4. Historical moving-average reference prices
    
5. Historical moving-average reference quantities
    
6. Data richness classification (Rich/Sparse)
    

**Example Interpretation**

For:

Plaintext

```
Mapping: AE100000/01
Scenario: Base
Aug-26: 1048.1
Sep-26: 1097.2
Oct-26: 1133.5
...
```

It means:

> The expected average selling price for product/planning node **AE100000/01** is forecasted to be approximately **1048** in Aug-26, **1097** in Sep-26, and **1134** in Oct-26 under the **Base** scenario.

For:

Plaintext

```
Scenario: Severe
Aug-26: 1089.7
Sep-26: 1153.1
...
```
It shows the ASP forecast under an alternative business scenario.
**In Layman's Terms**
This dataset answers:
> _"At what price do we expect to sell each product in future months?"_
Unlike the Quintiq forecast dataset, which forecasts **volume (MT)**, this dataset forecasts **price (ASP)**.
1. **Quintiq Forecast** → "How much will we sell?" (Volume)
2. **ASP Forecast** → "At what price will we sell?" (Price)
    
Together they can be used to estimate future revenue:
- **Revenue Forecast = Forecast Volume × Forecast ASP**

### Inventory – Current On-hand Snapshot,
Present in Sales update_Availability priority
- **Super Region**
- **Super Region 1**
- **Country**
- **Plant**
- **MaterialGroup 3**
- **Product Quality**
- **Polymer Type**
- **Material**
- **Sales Target Adjusted_Volume**
- **Invoiced Qty (filtered)**
- **Orders Qty (Filtered)**
- **Allocation Plan**
- **SR**
- **SR1**
- **IHP Type** (appears twice)
- **MC**
- **Check**
- **Grade**
- **Poly type**

### Order Intake / Open Orders
**AFO Dataset - What is present?**

This is an **Allocation Forecast Orders (AFO) dataset**. It contains detailed sales order lines showing how customer order quantities are allocated across multiple items/splits.

**Main columns**

1. **Sales Organization / Sales Office / Sales District / Sales Group** → Sales team handling the order.
    
2. **Purchase Order No.** → Customer PO reference.
    
3. **Sales Document / Order Item** → Order number and line item.
    
4. **SO It Created On** → Order creation date.
    
5. **Req. Dlv. Dt / Delivery Date** → Requested and actual delivery dates.
    
6. **Incoterms / Inco. 2** → Delivery terms and destination.
    
7. **Billing Document / Billing Date** → Invoice information.
    
8. **Sold-To-Party / Ship-To-Party / Payer** → Customer details.
    
9. **Name1 / Ship-To-Party Name** → Customer name.
    
10. **Country / Ship To Country** → Destination country.
    
11. **Material Number / Material Description** → Product sold.
    
12. **Plant** → Supplying plant.
    
13. **Material Group 1 / Material Group 3** → Product categories.
    
14. **End Use** → Application segment.
    
15. **Market Area** → Market/region.
    
16. **Sales Unit** → Unit of measure (MT).
    
17. **Delivery Number** → Delivery reference.
    
18. **Customer Group 9 / CG9 Description** → Customer segment.
    
19. **Order Quantity** → Quantity ordered.
    
20. **Billed Quantity** → Quantity billed/invoiced.
    

**In brief**

This dataset tells:

1. **Who ordered** (customer)
    
2. **What product was ordered**
    
3. **How much was ordered and billed**
    
4. **Where it was shipped**
    
5. **Order, delivery, and billing details**
    
6. **Allocation/split of quantities across order items**

### Sales Quotation
quote.csv
**Salesforce CPQ (Configure Price Quote) / Quote Management Dataset**

**Customer & Account Information**

- **AccountType__c**
    
- **CustomerID__c**
    
- **CustomerName__c**
    
- **CustomerNumber__c**
    
- **CustomerGroup__c**
    
- **HierarchyAccount__c**
    
- **Payer__c**
    
- **Payer1__c**
    
- **ShipToParty__c**
    
- **ShipToParty1__c**
    
- **ShipToName__c**
    
- **CustomerAddress__c**
    
- **ShipToAddress__c**
    
- **CustomerContact__c**
    
- **Primary_Contact__c**
    
- **SAPAccountNumberForSoldTo__c**
    
- **SAPAccountNumberForShipTo__c**
    
- **SAPAccountNumberForPayer__c** → Identify who the customer is, where products will be delivered, and the associated SAP customer master records.
    

**Pricing & Commercial Information**

- **ApprovedQuoteValue__c**
    
- **PortalAmount__c**
    
- **NetVolume__c**
    
- **TotalQuantity__c**
    
- **TotalQuantities__c**
    
- **TotalUnitPrice__c**
    
- **MinUnitPrice__c**
    
- **Component_Cost__c**
    
- **PTComponentofCosts__c**
    
- **FloorPriceDifferenceRollUp__c**
    
- **TargetPriceDifferenceRollUp__c**
    
- **ExchangeRate__c**
    
- **ExchangeCurrency__c**
    
- **CorporateCurrency__c**
    
- **HedgingRate__c**
    
- **All SBQQ__ pricing fields** → Contain quote value, selling price, costs, exchange rates, discounts, and financial calculations.
    

**Product & Volume Information**

- **GradeType__c**
    
- **MaxQuantity__c**
    
- **Material**
    
- **TotalProducts__c**
    
- **TotalQuantity__c**
    
- **NetVolume__c**
    
- **Quote line count fields** → Describe which polymer grades are being quoted, how much quantity is requested, and how many products are included in the quotation.
    

**Order & Quote Management**

- **Name**
    
- **Id**
    
- **OrderType__c**
    
- **SAPOrderNumber__c**
    
- **QuoteSource__c**
    
- **PrimaryQuote__c**
    
- **ClosedOpportunity__c**
    
- **CancelledOrder__c**
    
- **CarryOverOrder__c**
    
- **BackwardOrder__c**
    
- **Status_Reason__c**
    
- **SBQQ__Status__c**
    
- **SBQQ__Type__c**
    
- **SBQQ__Ordered__c**
    
- **SBQQ__DocumentStatus__c** → Track the lifecycle of the quotation and any resulting sales order.
    

**Approvals & Workflow**

- **RequiresApproval__c**
    
- **ApprovalAssignedQueue__c**
    
- **PaymentTermApproval__c**
    
- **SecretaryofCreditCommittee__c**
    
- **AssignToSalesClicked__c**
    
- **ShowAssignToSalesButton__c**
    
- **WorkFlowType__c**
    
- **CreatedByAgent__c**
    
- **CreatedByMDMUser__c**
    
- **Manager__c**
    
- **CSR__c**
    
- **OwnerId**
    
- **OwnerName__c**
    
- **OwnerRole__c**
    
- **OwnerManager__c**
    
- **OwnerProfile__c** → Control approvals, routing, user assignments, and workflow management.
    

**Dates & Audit Trail**

- **CreatedDate**
    
- **LastModifiedDate**
    
- **LastActivityDate**
    
- **LastReferencedDate**
    
- **LastViewedDate**
    
- **PricingDate__c**
    
- **ExpiresOn__c**
    
- **SBQQ__ExpirationDate__c**
    
- **RequestedDeliveryDate__c**
    
- **RequestedDeliveryMonth__c**
    
- **RequestedDeliveryMonthYear__c**
    
- **SBQQ__StartDate__c**
    
- **SBQQ__EndDate__c**
    
- **SystemModstamp** → Store creation dates, update dates, quote validity dates, and requested delivery dates.
    

**Sales Organization & Market Structure**

- **SalesOrganization__c**
    
- **SalesOrganizationCode__c**
    
- **SalesArea__c**
    
- **SalesChannel__c**
    
- **SalesChannelCode__c**
    
- **Market__c**
    
- **MarketCenter__c**
    
- **MarketCenterCode__c**
    
- **PerformanceCell__c**
    
- **PerformanceCellVP__c**
    
- **PerformanceCellSVP__c**
    
- **SuperRegion__c**
    
- **SuperRegion1__c**
    
- **SuperRegion2__c**
    
- **Country__c**
    
- **UserPerformanceCell__c** → Identify the geographical and organizational ownership of the quote.
    

**Logistics & Delivery Terms**

- **Incoterms__c**
    
- **Incoterm__c**
    
- **IncotermsLocation__c**
    
- **IncotermsLocationCode__c**
    
- **IncotermLocationText__c**
    
- **ContainerType__c**
    
- **TrailerType__c**
    
- **TransportType__c**
    
- **ShipmentRequired__c**
    
- **ShipmentCostPerTruck__c**
    
- **ShippingInstructions__c**
    
- **Show_Incoterms_Warming__c** → Specify delivery terms and transportation requirements.
    

**Payment Terms & Credit**

- **PaymentTerm1__c**
    
- **PaymentTerm2__c**
    
- **PaymentTermCondition__c**
    
- **PaymentTermDays__c**
    
- **PaymentTermNumberOfDays__c**
    
- **CreditLimitExceeded__c**
    
- **BankGuaranteeAmount__c** → Contain customer payment conditions, credit checks, and guarantee requirements.
    

**Salesforce CPQ (SBQQ) Fields**

- **All fields beginning with SBQQ__** → Standard Salesforce CPQ fields used for quote generation, discounts, templates, contracts, renewals, subscriptions, quote documents, billing information, and pricing calculations.
    

**System & Technical Fields**

- **CreatedById**
    
- **RecordTypeId**
    
- **CurrencyIsoCode**
    
- **IsDeleted**
    
- **SystemModstamp**
    
- **Ref__c**
    
- **Check**
    
- **SkipValidation__c**
    
- **Recalculate__c**
    
- **RequiresRecalculate__c**
    
- **Visibility-related fields** → Primarily used by Salesforce for system processing and UI behavior.
    

**In Layman's Terms**

This dataset tells you:

1. Who requested a quote
    
2. Which product they want
    
3. How much quantity they want
    
4. At what price
    
5. Under what payment terms
    
6. Where it will be shipped
    
7. Who approved it
    
8. Whether the quote was accepted, expired, or converted into an order
