## NAME - Ajay Singh
## ID - Bitsom_Ba_2511011
===========================================

## 1. PROBLEM SUMMARY
===========================================

You are working as a business analyst for a retail company.

The company has exported order-level sales data from multiple internal systems. The raw dataset contains issues such as inconsistent text formatting, date format problems, duplicate records, missing values, invalid discounts, calculation mismatches, and order status inconsistencies.

Your manager wants a clean, validated, analysis-ready version of the data along with summary reports that can be used for business review.

Your task is to clean the dataset, validate the business rules, document all issues found, and create summary reports using Excel.


===========================================

## 2. DATASET DESCRIPTION
===========================================

File Name   : cleaned_orders.xlsx

### Columns in Dataset:
1.  order_id         - Unique order identifier
2.  order_date       - Date order was placed
3.  ship_date        - Date order was shipped
4.  customer_id      - Unique customer ID
5.  customer_name    - Name of customer
6.  segment          - Customer segment
7.  region           - Geographic region
8.  state            - State of delivery
9.  city             - City of delivery
10. category         - Product category
11. sub_category     - Product sub category
12. product_name     - Name of product
13. ship_mode        - Shipping method used
14. quantity         - Number of items ordered
15. unit_price       - Price per unit
16. discount         - Discount applied
17. sales            - Total sales amount
18. cost             - Total cost amount
19. profit           - Profit earned
20. payment_status   - Payment status
21. order_status     - Order status


===========================================

## 3. TOOLS USED
===========================================

- Microsoft Excel 
  - Formulas: IF, TRIM, PROPER, SUBSTITUTE
  - ISNUMBER, COUNTIF, ABS, MONTH, YEAR
  - VALUE, SUBSTITUTE, DATEVALUE
  - PivotTables for summary reports
  - Conditional Formatting for flagging
  - Flash Fill for date standardization
  - Find and Replace for text cleaning
  - Data Validation for rule checking


===========================================

## 4. CLEANING STEPS PERFORMED
===========================================

Step 1: Text Standardization
- Removed leading and trailing spaces
  using TRIM() function
- Fixed case inconsistencies
  using PROPER() function
- Removed double spaces
  using SUBSTITUTE() function
- Columns cleaned: customer_name, segment,
  region, state, city, category,
  sub_category, ship_mode,
  payment_status, order_status

Step 2: Date Standardization
- Identified 4 different date formats
- Created clean_order_date column
- Created clean_ship_date column
- Standardized all to DD-MM-YYYY format
- Created shipping_delay_days column

Step 3: Discount Standardization
- Created cleaned_discount column
- Fixed negative discounts → set to 0
- Fixed text % values → converted to desimal
- Fixed missing discounts → set to 0

Step 4: Missing Values
- Missing region → filled as Unknown
- Missing ship_mode → filled as Unknown
- Missing discount → filled as 0

Step 5: Calculated Columns Added
- cleaned_discount
- calculated_sales
- calculated_profit
- profit_margin
- shipping_delay_days
- order_month
- order_year
- data_quality_flag (CLEAN/WARNING/INVALID)


===========================================

## 5. BUSINESS RULES APPLIED
===========================================

| Rule                    | Action Taken          |
|-------------------------|-----------------------|
| Missing region          | Fill as Unknown       |
| Missing ship_mode       | Fill as Unknown       |
| Missing discount        | Treat as 0            |
| Negative discount       | Flag as INVALID       |
| Discount above range    | Flag as INVALID (0)       |
| Cancelled orders        | Exclude from summary  |
| Failed payments         | Exclude from summary  |
| Refunded orders         | Summarize separately  |
| Ship before order date  | Flag as INVALID       |

===========================================

## 6. SUMMARY OF DATA QUALITY ISSUES
===========================================

| Issue Type              | Count |
|-------------------------|-------|
| Missing region          |   25 |
| Missing ship_mode       |   21|
| Missing discount        |   18  |
| Negative discount       |   15  |
| Discount as text %      |   8  |
| Ship before order date  |   93 |

### Final Data Quality Flags:
| Flag     | Count | 
|----------|-------|
| CLEAN    | 703  |  
| WARNING  |  37 | 
| INVALID  |  172 |
| Total    | 912  |  

===========================================

## 7. SUMMARY OF FINAL PIVOT REPORTS
===========================================

Pivot 1: Sales and Profit by Region
- Shows which region generates most revenue
- Sorted by sales highest to lowest

Pivot 2: Sales and Profit by Category
- Shows performance by product category
- Includes sub-category breakdown
- Sorted by sales highest to lowest

Pivot 3: Order Count by Ship Mode
- Shows most popular shipping methods
- Count of orders per ship mode

Pivot 4: Profit Margin by Segment
- Shows profitability by customer type
- Compares Consumer vs Corporate etc

Pivot 5: Issues by Region
- Filtered by region
- Helps identify problem areas

Pivot 6: Monthly Sales Trend
- Shows sales pattern over months
- Covers 2024 and 2025
- Sorted by month order

===========================================

## 8. KEY BUSINESS INSIGHTS
===========================================

1. DISCOUNT ISSUES
   - rows had invalid discount values
   - Negative discounts inflated costs
   - High discounts caused negative profit
     in some orders

2. SHIPPING ISSUES
   - orders shipped before order date
   - This indicates data entry errors
   - Needs process improvement

3. PAYMENT ISSUES
   - orders have failed payments
   - These should not count in revenue
   - Follow up needed with customers

4. REFUND ANALYSIS
   - orders were refunded
   - Separate summary created
   - Needs investigation by category

5. MISSING DATA
   -  rows have missing region/shipmode
   - Filled as Unknown for analysis
   - Source system needs fixing

6. PROFIT MARGINS
   - Some orders show negative profit
   - Caused by high discount values
   - Pricing strategy review needed

===========================================

## 9. ASSUMPTIONS AND LIMITATIONS
===========================================

### Assumptions Made:
1. Discount must be between 0 and 1
2. Negative discount = data entry error
3. Missing region/ship_mode = Unknown
4. Text % discount = formatting error
5. Discount above 50% = possible but unusual
6. No records deleted without clear reason

### Limitations:
1. Mixed date formats could not be fully
   auto-converted in all cases
2. High discounts kept as is - cannot
   confirm without business context
3. Missing values filled with Unknown -
   actual values cannot be recovered
4. Near-duplicate detection not performed
5. Status inconsistencies need manual review
6. Data dictionary not available for
   complete validation

===========================================

## 10. SCREENSHOTS INCLUDED
===========================================

Screenshots taken of:
- Raw dataset before cleaning
- Cleaned dataset with calculated columns
- Any major pivot summary
- Another major pivot summary
===========================================

