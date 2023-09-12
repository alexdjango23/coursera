# Hi student, the reason i am not using Mode, is because as free user i cannot enable external sharing. So you wont be able to open the link.
Please see answers below. You can copy and paste the code into Mode to validate the result. Obviously i wont be able to include graphs that i created into Github. Please be kind in your review. I did created two bar charts. I just cannot include it due to the shortcomings of this course assignment set-up.


1. We are running an experiment at an item-level, which means all users who visit will see the same page, but the layout of different item pages may differ.
Compare this table to the assignment events we captured for user_level_testing.
Does this table have everything you need to compute metrics like 30-day view-binary?

```
Answer: The table itself is missing event_id (an identifier for that event) and a date column - event_time. 
Without date, i cannot compare whether the view item event is after the assignment or not.
```
SELECT 
  * 
FROM 
  dsv1069.final_assignments_qa 


2. Reformat the final_assignments_qa to look like the final_assignments table, filling in any missing values with a placeholder of the appropriate data type.
```
SELECT 
  item_id,
  test_a       AS test_assignment, 
  'test_a'     AS test_number, 
  '2023-01-01' AS test_start_date
FROM 
  dsv1069.final_assignments_qa
```

3. Use this table to compute order_binary for the 30 day window after the test_start_date for the test named item_test_2
```
SELECT 
test_assignment,
 COUNT ( item_id) AS trials,
 SUM (order_binary) AS successes

FROM 
 (
  SELECT 
   final_assignments.item_id,
   final_assignments.test_assignment,
   MAX(CASE WHEN (orders.created_at > final_assignments.test_start_date AND
              DATE_PART('day', orders.created_at - final_assignments.test_start_date)<=30)
              THEN 1 ELSE 0 END) AS order_binary
  FROM 
    dsv1069.final_assignments
  LEFT JOIN 
    dsv1069.orders
  ON orders.item_id = final_assignments.item_id 
  WHERE
    final_assignments.test_number = 'item_test_2'
  GROUP BY 
   final_assignments.item_id,
   final_assignments.test_assignment
  ORDER BY
    item_id
) AS binaryorders

GROUP BY 
test_assignment
```

4. Use this table to compute view_binary for the 30 day window after the test_start_date for the test named item_test_2

```
SELECT 
  test_assignment,
  COUNT (DISTINCT item_id) AS trials,
  SUM(view_binary) AS successes

FROM 
  (
  SELECT 
   final_assignments.item_id,
   final_assignments.test_assignment, 
   MAX(CASE WHEN ( views.event_time >=  final_assignments.test_start_date AND
                    DATE_PART('day',  views.event_time -  final_assignments.test_start_date)<=30)
               THEN 1 ELSE 0 END) AS view_binary
  FROM 
    dsv1069.final_assignments
  LEFT JOIN 
    (
    SELECT 
      event_id,
      event_time,
      user_id,
      CAST(parameter_value AS INT) AS item_id
    FROM
      dsv1069.events
    WHERE 
      event_name = 'view_item'
    AND
      parameter_name = 'item_id'
    ) AS views
  ON 
    views.item_id = final_assignments.item_id 
  WHERE
    final_assignments.test_number = 'item_test_2'
  GROUP BY 
  final_assignments.item_id,
  final_assignments.test_assignment 
  ) AS binaryviews 
  
GROUP BY 
  test_assignment
```

5. --Use the https://thumbtack.github.io/abba/demo/abba.html to compute the lifts in metrics and the p-values for the binary metrics ( 30 day order binary and 30 day view binary) using a interval 95% confidence. 
```
Order binary
The p-value is 0.88., and the lift is -1%. With 95% confidence interval the range of improvement is between-14% and 12%.
There is no confidence in proving that the control and variance have difference chances of success.

View binary
P value for this lift 0.2, and test group has 2.6% more viewed items with a 95% confidence interval being -1.4% – 6.5%.

For both experiments, there is no statistically significant change to the metrics viewed percent as a result of the treatment. 

```

6. Use Mode’s Report builder feature to write up the test. Your write-up should include a title, a graph for each of the two binary metrics you’ve calculated. The lift and p-value (from the AB test calculator) for each of the two metrics, and a complete sentence to interpret the significance of each of the results.

```
Order binary
The p-value is 0.88., and the lift is -1%.
There is no confidence in proving that the control and variance have difference chances of success.

View binary
P value for this lift 0.2, and test group has 2.6% more viewed items.
For both experiments, there is no statistically significant change to the metrics viewed percent as a result of the treatment. 


Obviously i wont be able to include graphs that i created into Github. Please be kind in your review. I did created two bar charts.
I just cannot include it due to the shortcomings of this course assignment set-up.
Note that in order to share a Mode link with external views, you have to pay to be a premium account. This is a trap.
Therefore, i cannot share my Mode link.

```
