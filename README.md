# Hi student, the reason i am not using Mode, is because as free user i cannot enable external sharing. So you wont be able to open the link.
Please see answers below. You can copy and paste the code into Mode to validate the result. Obviously i wont be able to include graphs that i created into Github. Please be kind in your review. I did created two bar charts. I just cannot include it due to the shortcomings of this course assignment set-up.


1. We are running an experiment at an item-level, which means all users who visit will see the same page, but the layout of different item pages may differ.
Compare this table to the assignment events we captured for user_level_testing.
Does this table have everything you need to compute metrics like 30-day view-binary?

```
Answer: The table itself is missing event_id (an identifier for that event) and event_time. 
Without event_time, i cannot compare whether the view item event is after the assignment or not.
```
SELECT 
  * 
FROM 
  dsv1069.final_assignments_qa 


2. Reformat the final_assignments_qa to look like the final_assignments table, filling in any missing values with a placeholder of the appropriate data type.
```
CREATE TABLE 
	dsv1069.final_assignments 
	(
	item_id INT(10) NOT NULL PRIMARY KEY,
	test_assignment INT(10),
	test_number VARCHAR(11),
	test_start_date DATETIME
	);

INSERT INTO 
	dsv1069.final_assignments
SELECT
	item_id,
	test_assignment,
	test_number, 
	test_start_date -- note this column is missing from the table
FROM
	(
	SELECT 
  		item_id,
  		test_a AS test_assignment,
 		'test_a' AS test_number,
		test_start_date -- note this column is missing from the table
	FROM 
 		dsv1069.final_assignments_qa
  
	UNION ALL 
	SELECT 
  		item_id,
  		test_b AS test_assignment,
 		'test_b' AS test_number,
		test_start_date -- note this column is missing from the table
	FROM 
 		dsv1069.final_assignments_qa
  
	UNION ALL 
	SELECT 
  		item_id,
  		test_c AS test_assignment,
 		'test_c' AS test_number,
		test_start_date -- note this column is missing from the table
	FROM 
 		dsv1069.final_assignments_qa
  
	UNION ALL 
	SELECT 
  		item_id,
  		test_d AS test_assignment,
 		'test_d' AS test_number,
		test_start_date -- note this column is missing from the table
	FROM 
 		dsv1069.final_assignments_qa
  
	UNION ALL 
	SELECT 
  		item_id,
  		test_e AS test_assignment,
 		'test_e' AS test_number,
		test_start_date -- note this column is missing from the table
	FROM 
 		dsv1069.final_assignments_qa
  
	UNION ALL 
	SELECT 
  		item_id,
  		test_f AS test_assignment,
 		'test_f' AS test_number,
		test_start_date -- note this column is missing from the table
	FROM 
 		dsv1069.final_assignments_qa
  
	) AS final_assignments_pivot
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
The p-value is 0.88., and the lift is -1%. There is no confidence in proving that the control and variance have difference chances of success. 
With 95% confidence interval the range of improvement is between-14% and 12%, with a success rate of -1% 

View binary
P value 0.2, and lift is 2.6% with a 95% confidence interval being -1.4% – 6.5%. The p value shows some good confidence that the experiment has a influence on the success rate. 
However the success rate is 2.6% and insignificant
```
