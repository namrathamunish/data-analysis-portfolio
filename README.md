# Python Data Analysis Learning Journey

Overview
This repository documents my hands-on learning of Python data analysis libraries: Pandas, NumPy, Matplotlib/Seaborn, and SciPy. All concepts were practiced using the Superstore Sales dataset.

---

## What I Learned

### 1. Pandas - Data Manipulation
Working with tables (DataFrames) - loading CSVs, filtering rows, grouping data, and performing aggregations.

**Key functions:**
- `pd.read_csv()` - Load data
- `df.groupby()` - Group and aggregate
- `df[df['column'] > value]` - Filter rows
- `.pivot_table()` - Cross-tabulations

### 2. NumPy - Numerical Computing
Fast math operations on arrays through broadcasting (applying operations to entire arrays without loops) and vectorization.

**Key functions:**
- `np.array()` - Create arrays
- `array * 0.9` - Broadcasting (apply to all elements)
- `array[array > 1000]` - Boolean indexing (filter)
- `np.mean()`, `np.sum()`, `np.std()` - Aggregations

### 3. Matplotlib & Seaborn - Visualization
Creating charts and statistical plots. Matplotlib is the foundation; Seaborn builds on it for easier, prettier statistical visualizations.

**Key plots:**
- Bar charts - Comparisons
- Heatmaps - Correlations and pivot tables
- Violin plots - Distribution comparisons

### 4. SciPy - Statistical Computing
Scientific functions including probability distributions and optimization.

**Key concepts:**
- `stats.norm.cdf()` - Calculate probabilities (cumulative distribution function)
- `minimize()` - Find optimal values (min/max)

### 5. Machine Learning Pre-processing
Preparing data for ML models through scaling and handling missing values.

**Key techniques:**
- `StandardScaler()` - Scale to mean=0, std=1
- `MinMaxScaler()` - Scale to range 0-1
- `SimpleImputer()` - Handle missing values

---


### Application 1: Product Profitability Analysis

**Business Question:** Which products are losing money?

**Code:**
```python
import pandas as pd
import numpy as np

df = pd.read_csv("Sample - Superstore.csv")

# Group by product and sum profit
product_profit = df.groupby("Product Name")["Profit"].sum().sort_values()

# Filter unprofitable products
losing_products = product_profit[product_profit < 0]

print(f"Unprofitable products: {len(losing_products)}")
print(f"Total loss: ${-losing_products.sum():,.2f}")
print("\nTop 5 biggest losers:")
print(losing_products.head())
```

**Results:**
- 1,178 unprofitable products
- Total loss: $156,131.29
- Biggest loser: Cubify CubeX 3D Printer (-$8,879.99)

**Business Insight:** Losses concentrated in high-ticket technology items with heavy discounting. Recommend discontinuing or repricing these products.

**Statistical Understanding:** Used aggregation (sum) grouped by product, then boolean indexing to filter negatives. The distribution shows most losses come from a small number of expensive items.

---

### Application 2: Regional Performance Analysis

**Business Question:** Which region-category combinations should we prioritize for marketing investment?

**Code:**
```python
import seaborn as sns
import matplotlib.pyplot as plt

# Create pivot table
regional_performance = df.pivot_table(
    values='Sales', 
    index='Region', 
    columns='Category', 
    aggfunc='mean'
)

# Visualize as heatmap
sns.heatmap(regional_performance, annot=True, fmt='.0f', cmap='YlGnBu', linewidths=0.5)
plt.title("Average Sales by Region and Category")
plt.ylabel("Region")
plt.xlabel("Category")
plt.tight_layout()
plt.show()


**Results:**
- South-Technology: Highest avg sale ($508)
- Central-Office Supplies: Lowest avg sale ($117)

**Business Insight:** Technology products perform best in South region (premium pricing accepted). Office Supplies underperform in Central (price sensitivity or poor market fit). Focus marketing budget on high-performing combinations.

**Statistical Understanding:** Used pivot table to create two-way summary, visualized as heatmap to identify patterns. Color gradient highlights outliers (opportunities and underperformers).

---

### Application 3: Discount Impact Analysis

**Business Question:** Are aggressive discounts actually profitable?

**Code:**
```python
# Calculate correlation
correlation = df[['Discount', 'Profit']].corr()
print(f"Discount-Profit correlation: {correlation.iloc[0,1]:.3f}")

# Create discount brackets
df['Discount_Bracket'] = pd.cut(df['Discount'], 
                                  bins=[0, 0.1, 0.2, 0.3, 1.0],
                                  labels=['0-10%', '10-20%', '20-30%', '30%+'])

# Analyze by bracket
discount_analysis = df.groupby('Discount_Bracket').agg({
    'Profit': 'mean',
    'Sales': 'mean',
    'Order ID': 'count'
}).round(2)

print(discount_analysis)


**Results:**
- Discount-Profit correlation: -0.219 (moderate negative)
- Discounts >20%: Negative average profit
- Discounts >30%: Average loss of $89.45 per sale

                   Profit   Sales  Order ID
Discount_Bracket                          
0-10%              96.06  578.40        94
10-20%             24.74  213.58      3709
20-30%            -45.68  454.74       227
30%+             -107.21  222.59      1166

**Business Insight:** Discounts above 20% destroy margins faster than they increase volume. Recommend capping discounts at 15% to maintain profitability while staying competitive.

**Statistical Understanding:** Correlation analysis identified negative relationship. Binning into brackets revealed threshold where discounts become harmful. Mean profit by bracket shows clear inflection point at 20%.

---

### Application 4: Customer Segment Behavior

**Business Question:** Do different customer segments have different purchasing patterns?

**Code:**
```python
# Segment analysis
segment_analysis = df.groupby('Segment').agg({
    'Sales': ['mean', 'sum', 'count'],
    'Profit': 'mean',
    'Discount': 'mean',
    'Quantity': 'mean'
}).round(2)

segment_analysis.columns = ['Avg_Sale', 'Total_Sales', 'Transactions', 
                            'Avg_Profit', 'Avg_Discount', 'Avg_Quantity']
print(segment_analysis)

# Visualize
segment_analysis['Avg_Sale'].plot(kind='bar', color='steelblue')
plt.title("Average Sale by Customer Segment")
plt.ylabel("Average Sale ($)")
plt.xlabel("Segment")
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()
```

**Results:**
- Consumer: Most transactions, lowest avg sale ($229)
- Corporate: Highest total sales, moderate profit
- Home Office: Smallest segment, highest avg profit margin

**Business Insight:** Corporate segment drives volume but Home Office has better margins. Implement segment-specific strategies: volume pricing for Corporate, premium offerings for Home Office.

**Statistical Understanding:** Multi-level aggregation (mean, sum, count) across segments. Compared averages to identify behavioral differences rather than just volume.

---

### Application 5: Sales Driver Analysis

**Business Question:** What factors most strongly predict high sales?

**Code:**
```python
# Correlation matrix for numeric variables
correlations = df[['Sales', 'Quantity', 'Discount', 'Profit']].corr()

# Visualize
sns.heatmap(correlations, annot=True, cmap='coolwarm', center=0, 
            vmin=-1, vmax=1, square=True, linewidths=1)
plt.title("Correlation Matrix: Sales Drivers")
plt.tight_layout()
plt.show()

print("\nCorrelation with Sales:")
print(correlations['Sales'].sort_values(ascending=False))
```

**Results:**
- Quantity → Sales: +0.62 (strong positive)
- Profit → Sales: +0.48 (moderate positive)
- Discount → Sales: +0.03 (weak/no relationship)

**Business Insight:** Quantity is the strongest driver of sales (bulk orders = higher revenue). Surprisingly, discounts barely affect sales volume, suggesting we're leaving money on the table with unnecessary discounting.

**Statistical Understanding:** Correlation matrix reveals relationships between variables. Strong Sales-Quantity correlation (+0.62) indicates bulk purchasing is key driver. Weak Sales-Discount correlation (+0.03) challenges assumption that discounts drive volume.

---

### Application 6: Shipping Cost-Benefit Analysis

**Business Question:** Does shipping mode affect profitability?

**Code:**
```python
# Shipping mode analysis
shipping_analysis = df.groupby('Ship Mode').agg({
    'Profit': ['mean', 'sum'],
    'Sales': 'mean',
    'Order ID': 'count'
}).round(2)

shipping_analysis.columns = ['Avg_Profit', 'Total_Profit', 'Avg_Sale', 'Count']
shipping_analysis = shipping_analysis.sort_values('Avg_Profit', ascending=False)
print(shipping_analysis)

# Visualize
shipping_analysis['Avg_Profit'].plot(kind='barh', color='green')
plt.title("Average Profit by Shipping Mode")
plt.xlabel("Average Profit ($)")
plt.ylabel("Shipping Mode")
plt.tight_layout()
plt.show()
```

**Results:**
- Standard Class: Highest avg profit ($30.67), most transactions
- Same Day: Lowest avg profit ($8.71), premium pricing insufficient

**Business Insight:** Standard shipping is most profitable. Premium shipping options (Same Day) don't generate enough margin to justify costs. Either increase Same Day pricing or promote Standard Class more aggressively.

**Statistical Understanding:** Grouped by categorical variable (Ship Mode) and calculated mean profit. Horizontal bar chart makes ranking immediately clear.

---

### Application 7: Outlier Detection & Investigation

**Business Question:** What are the extreme cases (very high profit/loss) that need investigation?

**Code:**
```python
# Calculate profit margin
df['Profit_Margin'] = (df['Profit'] / df['Sales']) * 100

# Find extreme cases
high_profit = df.nlargest(10, 'Profit')[['Product Name', 'Sales', 'Profit', 'Discount', 'Profit_Margin']]
high_loss = df.nsmallest(10, 'Profit')[['Product Name', 'Sales', 'Profit', 'Discount', 'Profit_Margin']]

print("Top 10 Most Profitable Sales:")
print(high_profit)

print("\nTop 10 Biggest Losses:")
print(high_loss)

# Discount analysis for losses
print(f"\nAverage discount on top 10 losses: {high_loss['Discount'].mean():.1%}")
print(f"Average discount on top 10 profits: {high_profit['Discount'].mean():.1%}")
```

**Results:**
- Biggest loss: Canon imageCLASS 2200 (-$6,599.98, 80% discount)
- Top losses average 62% discount
- Top profits average 3% discount

**Business Insight:** Extreme losses caused by excessive discounting on high-value items. Implement approval workflow for discounts >30% on items >$500 to prevent margin destruction.

**Statistical Understanding:** Used `.nlargest()` and `.nsmallest()` to identify outliers. Calculated derived metric (profit margin %) to normalize across different sale sizes. Compared average discount between extreme groups to identify root cause.


---

## Tools Used
- Python 
- pandas 
- NumPy
- Matplotlib 
- Seaborn 
- SciPy 
- scikit-learn
- Jupyter Notebook

---

## Next Steps
- Complete ML preprocessing (categorical encoding)
- Build predictive models (regression, classification)
- Deploy automated reporting pipeline
- Apply skills to new datasets