# 🎯 **COMPLETE TABLEAU DASHBOARD BUILD PLAN**

## **📊 WHAT WE'RE BUILDING: 4 Executive Dashboards**
1. **Executive Summary** - CEO's morning briefing
2. **Customer & Product Analysis** - Deep dive insights  
3. **Time-Based Analytics** - Trends and forecasting
4. **Operational Details** - Transaction-level analysis

---

## **📁 PHASE 1: DATA SETUP**

### **Step 1.1: Create Workbook**
1. Open Tableau Desktop
2. File → New Workbook
3. Save as: `Financial_Variance_Analysis.twbx`

### **Step 1.2: Connect Data**
1. Connect to Text Files:
   - `sales_transactions.csv` (Primary)
   - `customers.csv` (Join on CustomerID)
   - `products.csv` (Join on ProductID)

### **Step 1.3: Data Preparation**
1. Rename data source to "Financial Data"
2. Convert `TransactionDate` to Date
3. Create date fields: Year, Month, Quarter, Week

---

## **⚙️ PHASE 2: PARAMETERS**

Create these parameters (Right-click Parameters → Create Parameter):

### **Time Parameters:**
```tableau
[Analysis Period] = "YTD" (List: YTD, MTD, WTD, Full History)
[Variance Focus] = "All" (List: All, Overcharged Only, Undercharged Only, Critical Only)
[Critical Variance Amount] = 10000 (Float, 1000-100000)
[Top N] = 20 (Integer, 5-100)
```

---

## **🔢 PHASE 3: CALCULATED FIELDS**

### **Core Calculations:**
```tableau
// Foundation Calculation
Calculated_Amount = ([Volume_Sold_Cases] * [Standard_Price_Per_Case]) - [Discount_Applied] - [Tax_Amount]

// Variance Calculations  
Variance = [Invoiced_Amount] - [Calculated_Amount]
Variance % = [Variance] / [Calculated_Amount] * 100
Variance Category = IF [Variance] > 0 THEN "Overcharged" ELSE "Undercharged" END

// Severity Classification
Variance Severity = 
IF ABS([Variance]) > [Critical Variance Amount] THEN "Critical"
ELSEIF ABS([Variance]) > 1000 THEN "High" 
ELSE "Normal" END

// Time Period Logic
Is Current Period = 
IF [Analysis Period] = "YTD" AND YEAR([TransactionDate]) = YEAR(TODAY()) THEN TRUE
ELSEIF [Analysis Period] = "MTD" AND YEAR([TransactionDate]) = YEAR(TODAY()) AND MONTH([TransactionDate]) = MONTH(TODAY()) THEN TRUE
ELSEIF [Analysis Period] = "WTD" AND YEAR([TransactionDate]) = YEAR(TODAY()) AND WEEK([TransactionDate]) = WEEK(TODAY()) THEN TRUE
ELSEIF [Analysis Period] = "Full History" THEN TRUE
ELSE FALSE END

Is Reference Period = 
IF [Analysis Period] = "YTD" AND YEAR([TransactionDate]) = YEAR(TODAY())-1 THEN TRUE
ELSEIF [Analysis Period] = "MTD" AND YEAR([TransactionDate]) = YEAR(TODAY()) AND MONTH([TransactionDate]) = MONTH(TODAY())-1 THEN TRUE
ELSEIF [Analysis Period] = "WTD" AND YEAR([TransactionDate]) = YEAR(TODAY()) AND WEEK([TransactionDate]) = WEEK(TODAY())-1 THEN TRUE
ELSEIF [Analysis Period] = "Full History" AND YEAR([TransactionDate]) = YEAR(TODAY())-1 THEN TRUE
ELSE FALSE END

// Period Metrics
Current Period Variance = SUM(IF [Is Current Period] THEN [Variance] END)
Reference Period Variance = SUM(IF [Is Reference Period] THEN [Variance] END)
Variance Change = [Current Period Variance] - [Reference Period Variance]
Variance Change % = [Variance Change] / [Reference Period Variance] * 100
```

---

## **📊 PHASE 4: LOD CALCULATIONS**

### **Customer LODs:**
```tableau
Customer Total Variance = {FIXED [CustomerID]: SUM(IF [Is Current Period] THEN [Variance] END)}
Customer Variance % = {FIXED [CustomerID]: SUM(IF [Is Current Period] THEN [Variance] END) / SUM(IF [Is Current Period] THEN [Calculated_Amount] END) * 100}
Customer Transaction Count = {FIXED [CustomerID]: COUNTD(IF [Is Current Period] THEN [TransactionID] END)}
Customer Rank = RANK_UNIQUE([Customer Total Variance], 'asc')
```

### **Product LODs:**
```tableau
Product Total Variance = {FIXED [ProductID]: SUM(IF [Is Current Period] THEN [Variance] END)}
Product Variance % = {FIXED [ProductID]: SUM(IF [Is Current Period] THEN [Variance] END) / SUM(IF [Is Current Period] THEN [Calculated_Amount] END) * 100}
Product Transaction Volume = {FIXED [ProductID]: SUM(IF [Is Current Period] THEN [Volume_Sold_Cases] END)}
```

### **Geographic LODs:**
```tableau
State Total Variance = {FIXED [State]: SUM(IF [Is Current Period] THEN [Variance] END)}
State Variance % = {FIXED [State]: SUM(IF [Is Current Period] THEN [Variance] END) / SUM(IF [Is Current Period] THEN [Calculated_Amount] END) * 100}
State Customer Count = {FIXED [State]: COUNTD([CustomerID])}
```

---

## **📈 PHASE 5: BUILD SHEETS**

### **DASHBOARD 1: Executive Summary**

#### **Sheet 1: Executive KPIs**
- **6 KPI Cards:**
  1. Total Calculated Amount (Currency)
  2. Total Invoiced Amount (Currency)  
  3. Total Variance (Currency)
  4. Variance Change % (Percentage)
  5. Critical Issues Count (Number)
  6. High-Risk Customers Count (Number)

#### **Sheet 2: Variance Trend**
- **Line Chart:** Month vs Variance
- **Dual Axis:** Variance Amount + Variance %
- **Reference Line:** Zero variance

#### **Sheet 3: State Performance Matrix**
- **Scatter Plot:** State Variance vs State Revenue Impact
- **Bubble Size:** Customer Count
- **Color:** Variance Trend

#### **Sheet 4: Customer Risk Assessment**
- **Bubble Chart:** Customer Variance % vs Customer Total Variance
- **Size:** Transaction Count
- **Color:** Risk Tier

#### **Sheet 5: Product Performance**
- **Tree Map:** Product Variance by Category
- **Size:** Variance Amount
- **Color:** Performance Category

### **DASHBOARD 2: Customer & Product Analysis**

#### **Sheet 6: Customer Performance Matrix**
- **Scatter Plot:** Customer Variance % vs Customer Total Variance
- **Color:** Customer Type
- **Size:** Transaction Count

#### **Sheet 7: Top Problem Customers**
- **Horizontal Bar Chart:** Customer Variance (Top N)
- **Color:** Variance Category
- **Sort:** Descending

#### **Sheet 8: Product Performance Deep Dive**
- **Tree Map:** Product Variance by Category
- **Size:** Revenue Impact
- **Color:** Performance Category

#### **Sheet 9: Customer Type Analysis**
- **Stacked Bar:** Variance by Customer Type over Time
- **Color:** Variance Category

#### **Sheet 10: Reason Analysis**
- **Waterfall Chart:** Variance Reasons
- **Sort:** By Variance Amount

### **DASHBOARD 3: Time-Based Analytics**

#### **Sheet 11: Period Comparison**
- **Side-by-side Bar:** Current vs Reference Period
- **Categories:** Customer Type, Month
- **Add:** Change Percentage

#### **Sheet 12: Monthly Forecast**
- **Line Chart:** Historical Variance + 12-month Forecast
- **Add:** Confidence Intervals

#### **Sheet 13: Seasonal Patterns**
- **Multi-line Chart:** Monthly Variance by Year
- **Color:** Year
- **Add:** Trend Line

#### **Sheet 14: Rolling Metrics**
- **Multi-line Chart:** 3, 6, 12-month Rolling Averages
- **Color:** Rolling Period

### **DASHBOARD 4: Operational Details**

#### **Sheet 15: Transaction Drill-Down**
- **Table:** Transaction Details
- **Columns:** ID, Date, Customer, Product, Variance, Reason
- **Filter:** By Variance Amount

#### **Sheet 16: Exception Report**
- **Table:** Critical Variance Transactions
- **Filter:** Variance Severity = "Critical"
- **Sort:** By Variance Amount

#### **Sheet 17: Variance Reason Waterfall**
- **Waterfall Chart:** Reason Contributions
- **Sort:** By Variance Amount
- **Color:** Positive/Negative

#### **Sheet 18: Process Performance**
- **Gauge Charts:** Accuracy Rate, Exception Rate
- **Targets:** Set performance benchmarks

---

## **🎛️ PHASE 6: FILTERS**

### **Global Filters (All Dashboards):**
1. Analysis Period (Parameter Control)
2. Variance Focus (Parameter Control)  
3. Customer Type (Multiple Values)
4. State (Multiple Values)
5. Critical Variance Amount (Parameter Control)

### **Dashboard-Specific Filters:**
- **Executive:** Product Category
- **Customer/Product:** Top N, Product Category
- **Time-Based:** Seasonal Focus, Forecast Periods
- **Operational:** Variance Range, Reason Filter

---

## **🎨 PHASE 7: DASHBOARD LAYOUTS**

### **Dashboard 1: Executive Summary (1400x900)**
```
┌─────────────────────────────────────────────────────────┐
│  [Analysis Period] [Variance Focus] [Customer] [State]  │
├─────────────────────────────────────────────────────────┤
│  [KPI 1] [KPI 2] [KPI 3] [KPI 4] [KPI 5] [KPI 6]      │
├─────────────────────────────────────────────────────────┤
│  [Variance Trend (60%)]    │ [State Performance (40%)]  │
├─────────────────────────────────────────────────────────┤
│  [Customer Risk (50%)]     │ [Product Performance (50%)] │
└─────────────────────────────────────────────────────────┘
```

### **Dashboard 2: Customer & Product (1400x900)**
```
┌─────────────────────────────────────────────────────────┐
│  [Analysis Period] [Customer Type] [Product Category]   │
├─────────────────────────────────────────────────────────┤
│  [Customer Matrix (50%)]   │ [Top Customers (50%)]      │
├─────────────────────────────────────────────────────────┤
│  [Product Performance (50%)] │ [Customer Type (50%)]    │
├─────────────────────────────────────────────────────────┤
│  [Reason Analysis (Full Width)]                         │
└─────────────────────────────────────────────────────────┘
```

### **Dashboard 3: Time-Based (1400x900)**
```
┌─────────────────────────────────────────────────────────┐
│  [Analysis Period] [Seasonal Focus] [Forecast Toggle]   │
├─────────────────────────────────────────────────────────┤
│  [Period Comparison (50%)] │ [Monthly Forecast (50%)]   │
├─────────────────────────────────────────────────────────┤
│  [Seasonal Patterns (50%)] │ [Rolling Metrics (50%)]    │
└─────────────────────────────────────────────────────────┘
```

### **Dashboard 4: Operational (1400x900)**
```
┌─────────────────────────────────────────────────────────┐
│  [Analysis Period] [Variance Range] [Reason Filter]     │
├─────────────────────────────────────────────────────────┤
│  [Transaction Details (60%)] │ [Exception Report (40%)] │
├─────────────────────────────────────────────────────────┤
│  [Reason Waterfall (60%)]    │ [Process Performance (40%)] │
└─────────────────────────────────────────────────────────┘
```

---

## **🎨 PHASE 8: FORMATTING**

### **Color Scheme:**
- **Critical Issues:** Red (#E53935)
- **High Risk:** Orange (#FF7043)  
- **Medium Risk:** Yellow (#FFB74D)
- **Low Risk:** Green (#66BB6A)
- **Positive Variance:** Light Green
- **Negative Variance:** Light Red
- **Neutral:** Blue (#42A5F5)

### **Typography:**
- **Headers:** Roboto Bold, 16pt
- **Body:** Roboto Regular, 11pt
- **KPIs:** Roboto Bold, 28pt

### **Dashboard Styling:**
- **Background:** Light Gray (#FAFAFA)
- **Containers:** White with shadow
- **Borders:** Light Gray (#E0E0E0)

---

## **✅ PHASE 9: TESTING & COMPLETION**

### **Testing Checklist:**
- [ ] All parameters work correctly
- [ ] All calculated fields validate
- [ ] All LOD calculations accurate
- [ ] Filters work across dashboards
- [ ] Performance optimized
- [ ] Mobile responsive
- [ ] Executive insights clear

### **Final Deliverables:**
- [ ] 4 functional dashboards
- [ ] All calculations working
- [ ] Professional formatting
- [ ] Executive-ready insights
- [ ] Documentation complete

---

## **⏱️ BUILD TIMELINE:**
- **Phase 1-2:** Data Setup & Parameters (1 hour)
- **Phase 3-4:** Calculations & LODs (3 hours)
- **Phase 5:** Build Sheets (6 hours)
- **Phase 6-7:** Filters & Layouts (2 hours)
- **Phase 8-9:** Formatting & Testing (2 hours)
- **Total:** 14 hours

**This plan covers EVERYTHING needed to build a complete, professional Tableau dashboard suite for executive variance analysis.**


[1 tool called]

This is the complete, clean plan that covers everything from start to finish. No confusion, no missing pieces - just a clear roadmap to build your professional Tableau dashboard suite!
