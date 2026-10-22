## Part 0: Data Issues Log

Row counts: orders 2,412 | payments 2,228 | customers 405 | products 152 | stores 15 | increment 160.

Every issue below was found by running the same rules the pipelines use. The full row-by-row list is in `data_issues_log_full.csv` (file, row key, CSV line, column, problem, dimension, severity).

| File | Column | Problem | Rows | Example keys | Dimension | Proposed rule | Severity |
|---|---|---|---|---|---|---|---|
| orders | customer_id | customer_id is missing | 18 | ORD002255, ORD000701, ORD000997, ORD000580 ... | Completeness | `customer_id_not_null` | Critical |
| orders | order_amount | order_amount != quantity x unit_price x (1 - discount_pct/100) | 18 | ORD002103, ORD002305, ORD002004, ORD000845 ... | Accuracy | `order_amount_matches_formula` | Critical |
| orders | order_id | duplicate order_id (repeat of an earlier row) | 12 | ORD001462, ORD001441, ORD001692, ORD002129 ... | Uniqueness | `order_id_unique` | Critical |
| orders | payment_mode | payment_mode spelling/casing will be standardised | 10 | ORD001812, ORD001414, ORD000159, ORD001364 ... | Validity | `payment_mode_needs_standardising` | Warning |
| orders | quantity | quantity is zero, negative or not a number | 10 | ORD000469, ORD001296, ORD001459, ORD000867 ... | Accuracy | `quantity_positive` | Critical |
| orders | product_id | product_id not found in products | 9 | ORD000986, ORD001058, ORD001685, ORD000070 ... | Consistency | `product_id_in_master` | Critical |
| orders | delivery_minutes | DELIVERED order has no delivery_minutes | 8 | ORD001554, ORD001800, ORD000731, ORD001277 ... | Completeness | `delivered_has_delivery_minutes` | Critical |
| orders | order_ts | order_ts is in the future | 7 | ORD002119, ORD001693, ORD001089, ORD001121 ... | Timeliness | `order_ts_not_future` | Critical |
| orders | order_ts | order_ts is missing or not a valid timestamp | 6 | ORD000403, ORD001610, ORD000534, ORD000362 ... | Validity | `order_ts_valid` | Critical |
| orders | delivery_minutes | delivery_minutes is 120 or more (impossible for a 10-minute service) | 6 | ORD001037, ORD002101, ORD000455, ORD001104 ... | Accuracy | `delivery_minutes_below_120` | Critical |
| orders | store_id | store_id not found in stores | 5 | ORD001799, ORD001046, ORD001492, ORD001275 ... | Consistency | `store_id_in_master` | Critical |
| orders | order_status | order_status is not DELIVERED/CANCELLED/RETURNED | 3 | ORD000500, ORD001970, ORD001637 | Validity | `order_status_valid` | Critical |
| orders | discount_pct | discount_pct is outside 0-100 | 3 | ORD002103, ORD001468, ORD001237 | Validity | `discount_pct_in_range` | Critical |
| orders | order_status | order_status casing/whitespace will be standardised | 2 | ORD001066, ORD000153 | Validity | `order_status_needs_standardising` | Warning |
| payments | (multiple) | SUCCESS amount_paid differs from the order's order_amount (see reconciliation) | 14 | PAY0001568, PAY0000354, PAY0000779, PAY0000323 ... | Consistency | `success_amount_matches_order` | Warning |
| payments | payment_mode | payment_mode spelling/casing will be standardised | 10 | PAY0002188, PAY0001850, PAY0001954, PAY0000817 ... | Validity | `payment_mode_needs_standardising` | Warning |
| payments | amount_paid | amount_paid is zero, negative or not a number | 10 | PAY0001568, PAY0002175, PAY0000116, PAY0000751 ... | Accuracy | `amount_paid_positive` | Critical |
| payments | order_id | order_id not found in orders (orphan payment) | 8 | PAY0002219, PAY0002220, PAY0002223, PAY0002221 ... | Consistency | `order_id_in_orders` | Critical |
| payments | payment_ts | payment_ts is in the future | 7 | PAY0000396, PAY0000127, PAY0001908, PAY0001037 ... | Timeliness | `payment_ts_not_future` | Critical |
| payments | payment_id | duplicate payment_id (repeat of an earlier row) | 4 | PAY0000509, PAY0001651, PAY0002000, PAY0000755 | Uniqueness | `payment_id_unique` | Critical |
| customers | email | email is missing or malformed | 12 | C00337, C00307, C00063, C00045 ... | Validity | `email_valid` | Warning |
| customers | loyalty_tier | loyalty_tier casing will be standardised | 8 | C00199, C00171, C00133, C00032 ... | Consistency | `loyalty_tier_needs_standardising` | Warning |
| customers | phone | phone is not exactly 10 digits | 7 | C00151, C00240, C00095, C00357 ... | Validity | `phone_is_10_digits` | Warning |
| customers | city | city is missing | 6 | C00355, C00394, C00233, C00142 ... | Completeness | `city_not_null` | Warning |
| customers | customer_id | duplicate customer_id (repeat of an earlier row) | 5 | C00103, C00252, C00226, C00036 ... | Uniqueness | `customer_id_unique` | Critical |
| customers | pincode | pincode is not exactly 6 digits | 3 | C00132, C00083, C00166 | Validity | `pincode_is_6_digits` | Warning |
| customers | signup_date | signup_date is in the future | 3 | C00400, C00398, C00038 | Timeliness | `signup_date_not_future` | Critical |
| products | cost_price | cost_price is higher than mrp | 5 | P0019, P0073, P0074, P0109 ... | Accuracy | `cost_price_not_above_mrp` | Critical |
| products | category | category is missing | 3 | P0051, P0060, P0068 | Completeness | `category_not_null` | Critical |
| products | product_id | duplicate product_id (superseded or repeated row) | 2 | P0030, P0140 | Uniqueness | `product_id_unique` | Critical |
| products | mrp | mrp is zero, negative or not a number | 1 | P0109 | Accuracy | `mrp_positive` | Critical |
| stores | zone | zone is missing | 1 | S07 | Completeness | `zone_not_null` | Critical |
| stores | (multiple) | capacity_orders_per_day is zero, negative or not a number | 1 | S12 | Accuracy | `capacity_positive` | Warning |

### Increment file (`orders_2026-10-01_increment.csv`)
No format, key or amount problems. 10 corrections (DELIVERED -> RETURNED, no other column changes) and 150 new orders (ORD002401 to ORD002550). Expected after MERGE: 10 rows updated, 150 inserted, 0 duplicates. September net revenue falls by Rs 4,193.70.

### Cross-file findings
- 71 DELIVERED orders have no SUCCESS payment (27 with no payment at all, 44 with only a FAILED one).
- 3 `Pending??` orders already have a SUCCESS payment.
- 8 payments belong to orders that do not exist (ORD97xxxx and similar).
- 14 SUCCESS payments differ from their order_amount.
- Order unit_price matches the current P0030/P0140 product rows, never the "(old)" rows, so the "(old)" rows are stale.
- All 11 orders on P0109 use unit_price 203 while the catalogue says -203, so the sign is likely a catalogue error.
- Store S07 has a blank zone and S12 has capacity 0 (stores.csv).

### Questions for the client
1. Is a negative or zero quantity a data-entry error or a return? The negative-quantity orders carry matching negative payments.
2. Can a DELIVERED order have only a FAILED payment (for example COD collected later), or is it always an error? This affects 44 orders.
3. For a customer_id that appears twice with different phone and area, which record is current?