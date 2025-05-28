# Sales Strategy Optimisation Analysis using Python

## Project Overview
This project explores the effectiveness of three sales strategies—Email, Call, and Email + Call at Pens and Printers Limited who launched a new line of office stationery aimed at boosting workplace creativity.  To support the success of this innovative product line, the primary business goal is to identify which strategy yields the highest return in revenue and customer engagement. You can find the comprehensive report on Medium [here.](You can find the comprehensive report and in-depth documentation on [Medium here.](https://medium.com/@UjuEmmanuella/sales-strategy-analysis-for-a-new-product-line-a-data-driven-case-study-in-sales-strategy-fd386a02e46d)

---

## Problem Statement

The sales team needed help understanding which method performs best:

* How many customers were reached per method?
* What does the revenue spread look like?
* How did revenue trend over time for each method?
* Which method should be prioritized going forward?

---

## Data Source

The dataset `product_sales.csv` from DataCamp  contains sales records from the first six weeks post-launch, including: `week,` `sales_method`, `customer_id`, `nb_sold`, `revenue` etc.

---

## Objective

* Clean and validate raw data.
* Perform exploratory analysis.
* Develop a metric to measure performance.
* Recommend the most effective sales strategy.

---

## Tools Used & Skills Demonstrated

* **Tools**: Python, Pandas, Seaborn, Matplotlib, Jupyter Notebook, Power Point
* **Skills**: Data cleaning, EDA, metric design, visualization, business insight

---

## Methodology

1. Data cleaning and validation
2. Exploratory data analysis (EDA)
3. Revenue trend analysis by week and method
4. Metric development (ARPSE)
5. Insight generation and recommendation

---

## Data Cleaning & Preparation

- **Sales Method**  
  Standardized inconsistent labels (e.g., `"em + call"`) into three clean categories:  
  - Email  
  - Call  
  - Email + Call

- **Revenue**  
  Missing values were imputed using the **mean revenue** per sales method.

- **Years as Customer**  
  Two entries (47 and 63 years) exceeded the company’s founding year constraints. These were corrected by capping at **39 years**.

- **Duplicates**  
  No duplicate rows were found.

- **Other Columns**  
  All other columns were valid and required no changes.


```python
# Load libraries
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt

# Load data
sales_data = pd.read_csv('product_sales.csv')

## Displays the first few rows and summary info of the datasets.
sales_data.head()
sales_data.info()
```

```python
# Check unique sales methods
df['sales_method'].value_counts()

# instead of 3, there are 5 unique values in the sales method column
# Email           7456
# Call            4962
# Email + Call    2549
# em + call         23
# email             10
```

```python
# Define a mapping dictionary to correct the inconsistent values
sales_method_mapping = {
    'Email': 'Email',
    'Call': 'Call',
    'Email + Call': 'Email + Call',
    'em + call': 'Email + Call',
    'email': 'Email'
}

# Apply the mapping to the 'sales_method' column
sales_data['sales_method'] = sales_data['sales_method'].map(sales_method_mapping)

# Check the value counts to ensure there are only 3 unique values
print(sales_data['sales_method'].value_counts())
# After cleaning, the 'sales_method' column now contains 3 unique, standardized values:
# Email, Call, Email + Call
```

```python
# Find mean revenue for each sales method
mean_revenue_by_sales_method = sales_data.groupby('sales_method')['revenue'].mean()
print(mean_revenue_by_sales_method)

def replace_null_revenue(row):
    """
    Replaces null (NaN) values in the 'revenue' column of a pandas DataFrame with the mean (or median) revenue
    for the corresponding 'sales_method' group.

    Parameters:
    -----------
    row : pandas Series
        A single row of a pandas DataFrame containing the 'revenue' and 'sales_method' columns.

    Returns:
    --------
    float
        The value of the 'revenue' column for the given row, either the original value if it is not null, or
        the mean (or median) revenue for the corresponding 'sales_method' group if it is null.
    """
    if pd.isnull(row['revenue']):
        return mean_revenue_by_sales_method[row['sales_method']]
    else:
        return row['revenue']
    
    # apply function to the revenue column
sales_data['revenue'] = sales_data.apply(replace_null_revenue, axis=1)

# check for any null values in the revenue column
print(sales_data['revenue'].isnull().sum())
# After standardizing, no nulls founded.
```

```python
#check for values > 39
sales_data[sales_data['years_as_customer'] > 39]
# Two rows had invalid entries (47 and 63), exceeding the maximum possible value of 39 (as the company was founded in 1984)

# find all values > 39 and replace with 39
sales_data.loc[sales_data['years_as_customer'] > 39, 'years_as_customer'] = 39

# check to see if replacement worked
sales_data[sales_data['years_as_customer'] > 39]
# there no more inavlid values
```

```python
# check number of unique values for state
sales_data['state'].nunique()
# No missing or invalid values, records are clean
```

```python
# check for any duplicate rows
duplicate_rows = sales_data[sales_data.duplicated()]
duplicate_rows
# no duplicates
```

```python
# view changes
sales_data.info()
# Data is cleaned and validated
```

---

## 📊 Exploratory Data Analysis

### A. Number of Customers by Sales Method

```python
# find number of customers for each sales method
customers_by_sales_method = sales_data['sales_method'].value_counts()
print(customers_by_sales_method)

# Email: 7,466 customers
# Call: 4,962 customers
# Email + Call: 2,572 customers

# Creates a bar chart showing the number of customers for each sales
method using Seaborn and Matplotlib

customers_by_sales_method = sales_data['sales_method'].value_counts()

ax = sns.barplot(x=customers_by_sales_method.index, y=customers_by_sales_method.values)

plt.title("Number of Customers by Sales Method")
plt.xlabel("Sales Method")
plt.ylabel("Number of Customers")

# Add value labels to each bar
for i, v in enumerate(customers_by_sales_method.values):
    ax.text(i, v + 0.5, str(v), ha='center')

plt.show()

```

### B. What is the spread of revenue overall and for each sales method

```python
# Histogram for overall revenue
plt.boxplot(sales_data['revenue'])
plt.xlabel('Revenue')
plt.ylabel('Frequency')
plt.title('Overall Revenue Distribution')
plt.show()
plt.show()

# Displays a boxplot to compare revenue distribution across different sales methods.
sales_data.boxplot(column='revenue', by='sales_method')
plt.xlabel('Sales Method')
plt.ylabel('Revenue')
plt.title('Revenue Distribution by Sales Method')
plt.suptitle('')  # Remove auto-generated sup-title
plt.show()

# Call: Mostly between $50–$75
# Email: $75–$125, with outliers beyond $150
# Email + Call: $100–$175, with outliers beyond $200

```

### C. Revenue over Time For Each Sales Method
```python

# Plots the total weekly revenue for each sales method to visualize trends and compare performance over time
revenue_over_time = sales_data.groupby(['week', 'sales_method'])['revenue'].sum().unstack()
revenue_over_time.plot()
plt.xlabel('Week')
plt.ylabel('Revenue')
plt.title('Revenue over Time by Sales Method')
plt.legend(title='Sales Method')
plt.show()

# Email: Steady decline after week 1
# Call: Fluctuates, with a drop in week 6
# Email + Call: Consistent upward trend
```

### D. Investigating other differences between customers in each group

```python
# Example: Boxplot for years_as_customer by sales_method
sales_data.boxplot(column='years_as_customer', by='sales_method')
plt.xlabel('Sales Method')
plt.ylabel('Years as Customer')
plt.title('Years as Customer Distribution by Sales Method')
plt.suptitle('')  # Remove auto-generated sup-title
plt.show()

# Other comparisons can be performed similarly (e.g., nb_site_visits, state, etc.)
```

```python
# Example: Boxplot for years_as_customer by sales_method
sales_data.boxplot(column='nb_sold', by='sales_method')
plt.xlabel('Sales Method')
plt.ylabel('Number Sold')
plt.title('Number of Sales Distribution by Sales Method')
plt.suptitle('')  # Remove auto-generated sup-title
plt.show()

# Other comparisons can be performed similarly (e.g., nb_site_visits, state, etc.)
```

```python
# Example: Boxplot for years_as_customer by sales_method
sales_data.boxplot(column='nb_site_visits', by='sales_method')
plt.xlabel('Sales Method')
plt.ylabel('# of Site Visits')
plt.title('# of Site Visits Distribution by Sales Method')
plt.suptitle('')  # Remove auto-generated sup-title
plt.show()

# Other comparisons can be performed similarly (e.g., nb_site_visits, state, etc.)
```

```python

## Calculates and plots the average revenue per customer each week for every sales method to reveal efficiency trends over time
grouped_data = sales_data.groupby(['week', 'sales_method']).agg({'revenue': 'sum', 'customer_id': 'count'}).reset_index()
grouped_data['average_revenue_per_customer'] = grouped_data['revenue'] / grouped_data['customer_id']
pivot_data = grouped_data.pivot_table(index='week', columns='sales_method', values='average_revenue_per_customer')
pivot_data.plot(kind='line', marker='o')
plt.xlabel('Week')
plt.ylabel('Average Revenue per Customer')
plt.title('Average Revenue per Customer by Sales Method over Time')
plt.legend(title='Sales Method')
plt.grid()
plt.show()
```

Beyond basic EDA, further segmentation as in the above **revealed**:

- Email + Call customers had more website visits, bought more items on average, and showed higher revenue per customer.

- Customer tenure (years as a customer) was consistent across groups, suggesting behavior patterns were influenced by strategy rather than loyalty.

---

### Metric Design — ARPSE (Average Revenue Per Sales Effort) = (Total Revenue for Method) / (Number of customers * Sales effort)

```python
# Define the sales effort for each sales method
sales_effort = {
    'Email': 0.5,
    'Call': 3,
    'Email + Call': 1
}

# Group the data by sales_method and aggregate the total revenue and number of customers
grouped_data = sales_data.groupby('sales_method').agg({'revenue': 'sum', 'customer_id': 'count'}).reset_index()

# Calculate ARPSE for each sales method
grouped_data['ARPSE'] = grouped_data.apply(lambda row: row['revenue'] / (row['customer_id'] * sales_effort[row['sales_method']]), axis=1)

# Display the ARPSE for each sales method
print(grouped_data[['sales_method', 'ARPSE']])

# Metric Results:
# Email: 194.25
# # Email + Call: 183.65
# Call: 15.86
# Although Email has the highest ARPSE, Email + Call showed better engagement and long-term growth potential.
```

---

## 💡 Insights

* **Email**: Most efficient in terms of revenue per effort but declined over time.
* **Call**: Labor-intensive and less consistent.
* **Email + Call**: Balanced strategy with steady growth, high engagement, and strong revenue.

---

## 📈 Recommendation

* Focus on **Email + Call** for long-term campaigns.
* Use **Email-only** for low-effort, short-term outreach.
* Deprioritize **Call** due to high effort and diminishing returns.
* Track future performance using **ARPSE** metric.

---

## 📂 Files & Resources

📜 Python Script: [Sales_Strategy_Optimisation_Analysis](Sales_Strategy_Analysis/python_notebook.ipynb)

✍🏽 Medium Write-Up : [Read article here](https://medium.com/@UjuEmmanuella/sales-strategy-analysis-for-a-new-product-line-a-data-driven-case-study-in-sales-strategy-fd386a02e46d)
