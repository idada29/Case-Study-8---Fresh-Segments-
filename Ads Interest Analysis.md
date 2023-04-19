# <p align="center" style="margin-top: 0px;"> Case-Study-8-Fresh-Segments 🍊
## <p align="center"> Ads Interest Analysis

## Question 1
*Which interests have been present in all month_year dates in our dataset?*

 ```sql
  -- In the CTE, each interest ID advert is selected along with how many months it has been interacted with.  
WITH Interest_Months AS (
SELECT 
    interest_id, COUNT(DISTINCT (month_year)) AS total_months
FROM
    interest_metrics
GROUP BY interest_id)

-- Within the aggregated months, the second step counts the number of adverts / interest IDs interacted with.
SELECT 
    total_months, COUNT(interest_id) AS Total_interest
FROM
    Interest_Months
GROUP BY total_months
HAVING total_months <> 0
ORDER BY Interest_Months.total_months DESC; 
```

| Total Months | Total Interest |
|---------------|----------------|
|     14       |      480      |
|     13       |       82      |
|     12       |       65      |
|     11       |       94      |
|     10       |       86      |
|      9       |       95      |
|      8       |       67      |
|      7       |       90      |
|      6       |       33      |
|      5       |       38      |
|      4       |       32      |
|      3       |       15      |
|      2       |       12      |
|      1       |       13      |

  
 
## Question 2
*Using this same total_months measure - calculate the cumulative percentage of all records starting at 14 months - which total_months value passes the 90% cumulative percentage value?*
 
 ```sql 
-- This aggregates the interest interaction by the number of months a user clicked
WITH interest_counts AS (
  SELECT
    interest_id,
    COUNT(DISTINCT month_year) AS total_months
  FROM
    interest_metrics
  GROUP BY
    interest_id
  HAVING 
    COUNT(DISTINCT (month_year)) <> 0
), 
-- This counts the number of interest appearing in multiple months
month_counts AS (
  SELECT
    total_months,
    COUNT(DISTINCT interest_id) AS interest_count
  FROM
    interest_counts
  GROUP BY
    total_months
), 
-- This step is very essential to establish the range of our data that doesn't represent the majority of the customer interactions. For this, it is set at 90%
-- cummulative percentage.
	
Cumulative AS (
  SELECT
    total_months,
    interest_count,
    SUM(interest_count) OVER (ORDER BY total_months DESC) AS cumulative_count
  FROM
    month_counts
)
  
SELECT
    total_months,
    interest_count, 
	ROUND(100 * cumulative_count / SUM(interest_count) OVER(), 2) AS cumulative_percentage
FROM Cumulative
GROUP BY total_months;

```

| Total Months | Interest Count | Cumulative % |
|--------------|----------------|---------------|
|      14      |      480      |     39.93    |
|      13      |       82      |     46.76    |
|      12      |       65      |     52.16    |
|      11      |       94      |     59.98    |
|      10      |       86      |     67.14    |
|       9      |       95      |     75.04    |
|       8      |       67      |     80.62    |
|       7      |       90      |     88.10    |
|       6      |       33      |     90.85    |
|       5      |       38      |     94.01    |
|       4      |       32      |     96.67    |
|       3      |       15      |     97.92    |
|       2      |       12      |     98.92    |
|       1      |       13      |    100.00    |

**This approach to cumulative metrics is really popular when we want to determine where we should cut off data which is “rare” and does not represent the majority of the other records in the analysis.**

 ## Question 3
*If we were to remove all interest_id values which are lower than the total_months value we found in the previous question - how many total data points would we be removing?*

 ```sql
-- If you compare this to the last question which states the cutoff of the cummulative percentage should be at 90 percent, 
-- that will fall just right on the 6 months, thus we will be removing -- all interest rate lower than 6 months. 
-- It means we would be removing any information for abouT 400 customers which really is alot.
	
WITH removed_interests AS (
  SELECT
    interest_id,
    COUNT(DISTINCT month_year) AS total_months
  FROM
    interest_metrics
  WHERE interest_id != 0
  GROUP BY
    interest_id
  HAVING 
    COUNT(DISTINCT (month_year)) < 6
)

SELECT 
  COUNT(*) AS removed_rows
FROM 
  interest_metrics m
  INNER JOIN removed_interests c 
  ON m.interest_id = c.interest_id;
 ```
 | removed_rows |
|--------------|
|400      |	       



## Question 4
*Does this decision make sense to remove these data points from a business perspective? Use an example where there are all 14 months present to a removed interest example for your arguments - think about what it means to have less months present from a segment perspective.*

```sql	
SELECT DISTINCT
    A.interest_id AS Ids, COUNT(A.interest_id) AS count_ids
FROM
    interest_metrics A
        LEFT JOIN
    interest_map B ON A.interest_id = B.id
WHERE B.id IS NULL
GROUP BY ids;

-- Viceversa, there are seven records in map table that is not in the metrics table 
SELECT DISTINCT
    B.id AS Ids, COUNT(B.id) AS count_ids, B.interest_summary
FROM
    interest_map B 
        LEFT JOIN
	interest_metrics A ON B.id = A.interest_id 
WHERE A.interest_id IS NULL
GROUP BY ids,3;	
```	
|Ids   | count_ids | interest_summary                                                                                                          |
|------|----------|---------------------------------------------------------------------------------------------------------------------------|
|19598 | 1        | People reading fan sites, promotional material, and news on the Doctor Who series.                                       |
|35964 | 1        | People in this audience participate and/or spectate in eSports (Competitive/Professional video gaming), follow various video game leagues, and keep up with video game tournaments that take place throughout the year.|
|40185 | 1        | People researching news and trends in astronomy.                                                                          |
|40186 | 1        | Consumers watching and reading about WWE and pro wrestling.                                                               |
|42010 | 1        | People reading Minecraft news and following gaming trends.                                                               |
|42400 | 1        | People reading Diablo news and following gaming trends.                                                                  |
|47789 | 1        | People researching attractions and accommodations in Israel. These consumers are more likely to spend money on travel|

	
	
## Question 5	
*After removing these interests - how many unique interests are there for each month?*
	
```sql
SELECT 
    CONCAT('There is ',
            COUNT(B.id),
            ' in interest_map table') AS Records
FROM
    interest_map B;	
```
|Records|
|------:|
|There is 1209 in interest_map table|
