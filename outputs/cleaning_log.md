### Task 9: Write Cleaning Log


## 1. Issues Found


### Issue 1 — Negative Shipping Delays (Date Logic Violations)
- **Count:** 93 records
- **Flag:** `date_flag = INVALID`
- **Description:** The `shipping_delay_days` field shows negative values, meaning the ship date falls *before* the order date. This is physically impossible and indicates either a data entry error, system timestamp bug, or an incorrect date field mapping.

### Issue 2 — Negative Discount Values
- **Count:** 15 records
- **Flag:** `Dis_flag = INVALID-NEGATIVE`
- **Description:** The `discount` column contains negative percentage values (e.g., −0.19, −0.14, −0.24). A negative discount implies the customer was *charged extra*, which is not a valid discount concept and is likely a data entry error or system export anomaly.

### Issue 3 — Excessive Discounts Entered as Percentages (Format Error)
- **Count:** 8 records
- **Flag:** `Dis_flag = INVALID-ABOVE`
- **Description:** Discount values entered as whole percentages (e.g., `70%`, `85%`) instead of decimal fractions (e.g., `0.70`, `0.85`). When applied to sales calculations, these produce extreme negative profits. Some records (e.g., discount = `85%`) yield a Profit Margin of −342%, which is clearly erroneous.

### Issue 4 — Failed Payment Status
- **Count:** 69 records
- **Flag:** `Data_Quality_Flag = INVALID`
- **Description:** Orders with `payment_status = Failed` are flagged as invalid because revenue cannot be recognized for transactions where payment was not successfully collected. These represent real business risk — goods may have been shipped but payment was not received.

### Issue 5 — Unknown Ship Mode
- **Count:** 21 records
- **Flag:** `Data_Quality_Flag = WARNING`
- **Description:** The `ship_mode` field contains the value `Unknown` for 21 orders. Without knowing the shipping method, logistics cost analysis and shipping performance reporting are impaired. These are flagged as WARNING rather than INVALID because the core financials may still be valid.

### Issue 6 — Unknown Region
- **Count:** 25 records
- **Flag:** `Data_Quality_Flag = WARNING`
- **Description:** The `region` field is `Unknown` for 25 records. Regional sales analysis and territory-based performance tracking cannot be performed for these orders. The state and city are typically populated, so region can potentially be inferred, but no automated mapping was applied.

### Issue 7 — Conflicting Duplicate Records
- **Count:** 12 duplicate pairs (24 total rows)
- **Flag:** `Duplicate_flag = CONFLICTING DUPLICATE - REVIEW`
- **Description:** 12 order IDs appear twice with conflicting values in `sales`, `order_status`, and occasionally `payment_status`. These are not simple duplicates — they carry different business states, suggesting either a system sync error, order amendment without proper versioning, or a return/re-order recorded on the same ID.


### Issue 8 — Shipping Delays 
- **Flag:** `date_flag = OK` (not flagged invalid)

## 2. Cleaning Actions Performed

| # | Action | ---|
|---|--------|-----------------|
| 1 | Flagged all records with negative `shipping_delay_days` as `date_flag = INVALID` | 
| 2 | Flagged all negative discount values as `Dis_flag = INVALID-NEGATIVE` | 
| 3 | Flagged discounts entered as whole-percentage strings (e.g., `70%`) as `Dis_flag = INVALID-ABOVE` |
| 4 | Standardized invalid negative discounts to 0.0 in `Standardized_dis` column | 
| 5 | Converted percentage-formatted discounts to decimal in `Standardized_dis` column | 
| 6 | Recalculated `Calculated_sales` and `Calculated_Profit` using standardized discount | 
| 7 | Flagged `payment_status = Failed` records as `INVALID` in `Data_Quality_Flag` | 
| 8 | Flagged `ship_mode = Unknown` records as `WARNING` in `Data_Quality_Flag` | 
| 9 | Flagged `region = Unknown` records as `WARNING` in `Data_Quality_Flag` | 
| 10 | Tagged duplicate original rows with `Duplicate_flag = ORIGINAL` | 
| 11 | Tagged conflicting duplicate rows with `Duplicate_flag = CONFLICTING DUPLICATE - REVIEW` | 
| 12 | Added `Data_Quality_Flag` column with values: `CLEAN`, `INVALID`, `WARNING` | 
| 13 | Added `Profit_Margin` column (`Calculated_Profit / Calculated_sales`) | 
| 14 | Added `Order_Month` and `Order_year` columns from `order_date` | 

---

## 3. Business Rules Applied

| Rule ID | Business Rule | Rationale |
|---------|--------------|-----------|
| BR-01 | `ship_date` must be ≥ `order_date` (shipping delay ≥ 0) | Goods cannot be dispatched before an order is placed |
| BR-02 | `discount` must be in range [0.0, 1.0] | Discounts represent a fraction; negative means surcharge; >1 means full price waived or data error |
| BR-03 | `payment_status = Failed` orders are excluded from revenue reporting | Revenue can only be recognised when payment is successfully received |
| BR-04 | `ship_mode` must be one of: Standard Class, First Class, Second Class, Same Day | `Unknown` ship mode makes logistics cost attribution impossible |
| BR-05| `region` must be one of: North, South, East, West | `Unknown` region prevents geographic performance reporting |
| BR-06 | Each `order_id` must be unique with consistent field values | Conflicting duplicates indicate a data integrity problem requiring human review |
| BR-07 | `Standardized_dis` replaces invalid discount values with 0.0 for computation | Negative or percentage-format discounts are corrected to prevent calculation errors |
| BR-08 | `Calculated_sales = unit_price × quantity × (1 − Standardized_dis)` | Standard sales formula using corrected discount value |
| BR-9 | `Calculated_Profit = Calculated_sales − cost` | Profit is calculated from corrected (not raw) sales figure |

---

## 4. Assumptions Made

1. **Negative Delays = Data Entry Error:** It was assumed that all negative `shipping_delay_days` values are errors, not intentional same-day or pre-staged shipments. No business scenario justifies a ship date prior to an order date.

2. **Negative Discounts = Entry Error, Not Surcharges:** The business does not apply surcharges labelled as discounts. Negative discount values are treated as a sign-flip error during data entry.

3. **Percentage Discounts = Format Error:** Discounts entered as `70%` or `85%` (string format) are treated as decimal entry errors (intended values: 0.70 and 0.85). These are corrected in `Standardized_dis` before recalculation.

4. **"Original" Row is Authoritative for Duplicates:** Where duplicate pairs exist, the row marked `ORIGINAL` is assumed to be the first-recorded (authoritative) version. The conflicting row is flagged for human review and *not* automatically deleted.

5. **Failed Payments = Uncollected Revenue:** Orders with `payment_status = Failed` are assumed to have been attempted but not completed. They may represent bad debt or retry scenarios. No adjustment was made to remove cost or inventory entries.

6. **High Discounts (>50%) are Valid Unless Also Format-Invalid:** Discounts of 55% or 65% (expressed as decimals: 0.55, 0.65) are retained as-is and not automatically flagged INVALID. Only those entered as `70%`/`85%` strings are corrected. Some high-discount cases result in negative calculated profits, which are a business concern but not a data error.

7. **Region Cannot Be Auto-Inferred:** Although state and city are available, region was not back-filled because the mapping logic was not provided. Records retain `region = Unknown` and are flagged as WARNING.

8. **Ship Mode Cannot Be Auto-Inferred:** No external shipping method reference was available to fill in `Unknown` ship mode values.

9. **Extreme Delays (>30 days) are Retained:** These are mathematically valid records. The business may have legitimate reasons for long delays (backorders, custom items). They are not flagged INVALID.

10. **Cost Field is Accurate:** The `cost` column was not validated against a product master. Profit calculations assume cost values are correct.

---

## 5. Records Removed

**20 Exect Duplicate records were permanently deleted from the dataset.**

The cleaning strategy adopted a *flag-and-retain* approach to preserve auditability. All problematic records remain in the dataset with clear quality flags, enabling:
- Business users to review INVALID records before exclusion
- Finance teams to decide on treatment of Failed Payments
- Operations teams to investigate duplicate pairs

---

## 6. Records Flagged

### Flagged as INVALID (172 records)
Records in this category have confirmed data quality errors affecting financial accuracy:

| Reason | Count |
|--------|-------|
| Negative shipping delay (ship before order date) | 93 |
| Failed payment status | 69 |
| Negative discount value | 15 |
| Discount entered as percentage string (e.g., 70%, 85%) | 8 |


### Flagged as WARNING (37 records)
Records in this category have incomplete descriptive data but may have valid financials:

| Reason | Count |
|--------|-------|
| Unknown region | 25 |
| Unknown ship mode | 21 |


### Flagged as CONFLICTING DUPLICATE – REVIEW (12 records)
These records share an `order_id` with another row but have different `sales`, `order_status`, or `payment_status` values. They require manual review by the operations or sales team:


## 7. Limitations of the Cleaning Process

1. **No Source System Access:** Cleaning was performed solely on the exported Excel file. The root cause of errors (e.g., whether negative delays are due to a timezone bug or manual entry) could not be verified against the originating order management system.

2. **Duplicate Resolution is Incomplete:** Conflicting duplicates were identified and flagged but not resolved. Determining which row is "correct" requires business context (e.g., was the order amended after a return?). The cleaning process does not merge or delete these rows.

3. **Region Cannot Be Back-Filled:** Without a state-to-region master table integrated into this process, the 25 `Unknown` region records could not be corrected automatically.

4. **Ship Mode Gap Unresolved:** The 21 `Unknown` ship mode records cannot be filled without access to shipping carrier data or the original shipping labels.

5. **High Discounts (0.55, 0.65) Not Blocked:** Discounts above 50% that are correctly formatted as decimals are not flagged as errors. From a data quality standpoint they are valid numbers, but the business may want to apply an approval policy review for orders with discounts > 30%.

6. **Cost Validation Not Performed:** The `cost` column was accepted as-is. If cost entries are inflated or understated, the `Calculated_Profit` and `Profit_Margin` columns will be inaccurate. A product cost master comparison was not within scope.

7. **Sales Discrepancy Between Raw and Calculated Fields:** For 59 records, `sales` and `Calculated_sales` differ by more than ₹1. This may reflect rounding, mid-order amendments, or the impact of incorrect discounts applied at source. For all analysis, `Calculated_sales` using `Standardized_dis` is the recommended field.

8. **No Time-Series Validation:** The cleaning process did not verify whether the order date distribution across months/years is reasonable. Potential gaps in data (missing months) were not detected.

9. **Manual Human Review Required for INVALID Records:** The final disposition of 172 INVALID records (particularly the 69 Failed Payments representing a significant revenue risk) must be decided by business stakeholders — this cleaning log documents the issues but cannot substitute for that business decision.

---


