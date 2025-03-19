# Data-driven Unit-Economics
![](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTLGn3uOCKBul_hYG0_rOUBI3RrZwNJebfl_Q&s)
## I. Context
The company is called "TechStream Solutions", and the product is a Software as a Service (SaaS) platform named "Streamline Pro". This platform provides comprehensive project management and collaboration tools for businesses of all sizes.

TechStream Solutions has been operating for several years and has gathered significant data on their costs and revenues. They are now looking to analyze their unit economics to understand the profitability of Streamline Pro on a per-customer basis.

The dataset are in the the shared folder on Google Drive: [link](https://drive.google.com/drive/folders/1qhOW9Y2orRXuzbX-kXEmuJ7TMQiRs2Uv?usp=drive_link)

By performing these calculations, TechStream Solutions aims to:
  - Identify the profitability of acquiring and retaining customers.
  - Assess the efficiency of their marketing and sales strategies.
  - Make informed decisions on scaling their operations and optimizing their resources allocation.
  - This information will guide TechStream Solutions in refining their business strategies, ensuring sustainable growth, and maximizing profitability.
    
## II. Objective
**Background:** Streamline Pro is a comprehensive project management and collaboration tool designed to help businesses manage projects, track progress, and collaborate efficiently. Understanding the unit economics of Streamline Pro is crucial for evaluating its financial health and sustainability. This involves analysing key metrics such as Customer Acquisition Cost (CAC), Average Revenue Per User (ARPU), Cost of Goods Sold (COGS), Gross Margin, Customer Lifetime Value (LTV), and the LTV/CAC ratio.

**Objective:** Your task is to calculate the unit economics for Streamline Pro for the month of March 2023. This will help us assess the profitability and efficiency of our customer acquisition strategies and operational expenses.

## III. Methodology
**1. Write a function to read the dataset on Google Drive**
```python
def read_file_ggsheet(file_id, month_col=None):
  df_ggsheet = pd.read_excel('https://docs.google.com/spreadsheets/d/'+ file_id + '/export?format=xlsx')
  # If the data has datetime columns, we need to filter it and only keep the data for March 2023
  if month_col is not None:
    df_202303 = df_ggsheet[(df_ggsheet[month_col].dt.month == 3) & (df_ggsheet[month_col].dt.year == 2023)]
    return df_202303
  else:
    return df_ggsheet
```
```python
customer_lifespan = read_file_ggsheet(file_id = '1by8tPHwOnq3uKYK2E7sA9VBUYoPM4p1Rnrm_Ss9cyHI')
daily_spendings = read_file_ggsheet(file_id='1AZOIThOV4P-0eYDge53ZwumVkfkHoYPWxst3k3Bv87c', month_col='date')
monthly_expenses = read_file_ggsheet(file_id='10OGbaywwMIqKgnPGy8VDvpBVtjyqln47iYa2lFhI9Mw', month_col='month')
payroll = read_file_ggsheet(file_id='1c_WihqTZCQvNgxzmd-OwhR9i5diwtfxXVLyMn8R-Lp4', month_col='month')
receipts_history = read_file_ggsheet(file_id='1qayqML1zCKdmtzutkcy9LWvE6xFRm6TGBEVkHHJKIuE', month_col='date')
```
After running this function, now I got all the necessary information to calculate the required unit economics.

**2. CAC**

It is the cost of attracting customers. And in the context of this company, it is related to the expenses of online advertising, sales staff salaries and commissions, marketing software
```python
# Total sales and marketing expenses
online_ads_cost = daily_spendings['spending'].sum() 
sales_salary_cost = payroll[payroll['department'].isin(['Sales','Marketing'])]['paid'].sum()
mkt_software_cost = monthly_expenses[monthly_expenses['item'] == 'Salesforce']['amount'].sum()
total_sale_mkt_expenses = online_ads_cost + sales_salary_cost + mkt_software_cost

# Number of new customers acquired
new_cust_count = len(receipts_history[receipts_history['new_customer']==1]['customer_id'])
new_cust_count
```
```python
cac = total_sale_mkt_expenses / new_cust_count
```
The result is **1213.9683**

**3. ARPU**

Average revenue per user (or per unit), also known as ARPU, is a metric that helps companies of all types understand how much money they earn from an average customer over a given period of time.
```python
#ARPU
revenue_total = receipts_history['receipt_amount'].sum()
num_cust = len(receipts_history['customer_id'].unique())

arpu = (revenue_total)/(num_cust)
```
And I got the ARPU of **284.3596**

**4.COGs**

Cost of Goods Sold (COGs) represents the direct costs of producing the goods or services a company sells in a specific period. This includes expenses like raw materials, direct labor and manufacturing overhead.
```python
#COGS
infrastructure_costs = monthly_expenses[monthly_expenses['item'].isin(['AWS Hosting','Google Cloud Storage','Atlassian Jira','Slack','Zoom'])]['amount'].sum()
salary_employees_involved = payroll[payroll['department'] == 'Engineering']['paid'].sum()

cogs = infrastructure_costs + salary_employees_involved
```
And the result is **20840**

**5. Gross Margin**

The difference between sales revenue and the cost of goods sold (COGs), expressed as a percentage of sales revenue.
```python
gross_margin = ((revenue_total-cogs)/revenue_total)*100
```
And the result is **74.9016 %**

**6. LTV**

Customer Lifetime Value (LTV) is a metric that estimates the total revenue a business can expect from a customer over the entire duration of their relationship.
Its formula: *LTV = ARPU * CustomerLifeSpan * Gross Margin*
```python
# Firstly, we need to create a new column to calculate the customer lifespan
customer_lifespan['cust_lifespan(months)'] = (((customer_lifespan['churn_date'] - customer_lifespan['start_date']).dt.days )/30).astype('int')

# Calculate the average of customer lifespan
a = customer_lifespan['cust_lifespan(months)'].mean()

# Find the LTV
LTV = arpu * customer_lifespan['cust_lifespan(months)'].mean() * (gross_margin/100)
```
And we got the result is **1985.0643**

**7. LTV/CAC**

This is a ratio that compares the Customer Lifetime Value (LTV) to the Customer Acquisition Cost (CAC). This metric is used to assess the efficiency and profitability of a company's customer acquisition strategies.
```python
ltv_cac = LTV / cac
```
And here is the result **1.6352**

## IV. Conclusions
**Insights**
- LTV/CAC = 1.6352 (Not optimal). Although this metric is greater than 1, but not ideal (usually >= 3). This shows that customers give more revenue than the cost of acquiring them, but the gross margin is not really optimal.

- High CAC ($1213.97). The cost of acquiring a customer is quite high, reducing the profit per customer.

- Gross margin = 74.90% (Good). The gross profit margin is high, indicating that the business model is profitable, but the total operating costs need to be considered to ensure that the final profit remains sustainable.

- ARPU = $284.36 (Low compared to CAC). The average revenue per customer is not high enough compared to the cost of acquiring customers (CAC). This can affect the ability to recover capital quickly.

- High COGs ($20,840). Cost of goods sold is very high, which can affect overall profitability.

**Actionable insights**
- Increase LTV: Retain customers longer, encourage ugrades to higher packages, improve service.
- Reduce CAC: Leverage content marketing, optimize advertising campaigns, promote referrals.
- Increase ARPU: Sell more add-ons, expand premium packages, more flexible pricing.
- Optimize COGs: Cut cloud costs, automate backend, optimize engineer performance.
- Improve Gross Margin: Increase revenue from B2B customers, improve self-service support.


