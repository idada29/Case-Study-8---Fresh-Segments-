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
-- Replace the null value in the newly concatenated month_year with unknown then count records
 SELECT 
    COALESCE(month_year, 'Unknown') AS Dates,
    COUNT(*) AS Records
FROM
    interest_metrics
GROUP BY Dates
ORDER BY 1 DESC , 2; 
```

 | Dates       | Records |
|-------------|---------|
| Unknown     | 1194    |
| 2019-08-01  | 1149    |
| 2019-03-01  | 1136    |
| 2019-02-01  | 1121    |
| 2019-04-01  | 1099    |
| 2018-12-01  | 995     |
| 2019-01-01  | 973     |
| 2018-11-01  | 928     |
| 2019-07-01  | 864     |
| 2019-05-01  | 857     |
| 2018-10-01  | 857     |
| 2019-06-01  | 824     |
| 2018-09-01  | 780     |
| 2018-08-01  | 767     |
| 2018-07-01  | 729     |


 ## Question 3
*If we were to remove all interest_id values which are lower than the total_months value we found in the previous question - how many total data points would we be removing?*

 ```sql  
-- Select all records in the month_year that is null
WITH records AS (
  SELECT COALESCE(month_year, 'Unknown') AS Dates,
    COUNT(*) AS Records
  FROM interest_metrics
  GROUP BY Dates
  ORDER BY 1 DESC , 2
)
-- Count the number of null/unknown value and others
SELECT 
	CASE WHEN Dates = 'Unknown' THEN 'Unknown'
    ELSE 'Other' END AS Record_dates,
    SUM(Records) AS Total_Record
FROM records
GROUP BY 
	CASE WHEN Dates = 'Unknown' THEN 'Unknown' ELSE 'Other' END;
 ```
**In this case, it was impossible to trace the ads interaction to any date or interest_id. Hence we will drop it**
 | Record_dates | Total_records |
|--------------|---------------|
| Unknown      | 1194          |
| Other        | 13079       |


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
