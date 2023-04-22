# <p align="center" style="margin-top: 0px;"> Fresh Segments 🍊
## <p align="center"> Data Exploration and Cleansing

## Question 1
*Update the fresh_segments.interest_metrics table by modifying the month_year column to be a date data type with the start of the month*

 ```sql
-- You have to first change the length of the varchar to allow for the new addition of month beginning
ALTER TABLE interest_metrics MODIFY month_year VARCHAR(15);

-- Next update with a concatenation, TO ADD MONTH beginning at 01
UPDATE fresh_segments.interest_metrics
SET month_year = CONCAT("01-",month_year);

-- Replace the - symbol to allow for further date manipulation
UPDATE fresh_segments.interest_metrics
SET month_year = REPLACE(month_year, '-', '/')
WHERE month_year LIKE '%-%';

UPDATE interest_metrics
SET month_year = STR_TO_DATE(month_year,"%d/%m/%Y");  
```
## Question 2
*What is count of records in the fresh_segments.interest_metrics for each month_year value sorted in chronological order (earliest to latest) with the null values appearing first?*
 
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
*What do you think we should do with these null values in the fresh_segments.interest_metrics*
 A good data quality step in handling null values is:
 - Identifying the reason for missing data.
 - Assessing the extent of missing data.
 - Assessing the data type of the missing data, to determine suitable statistical techniques to apply.
 - Considering using imputation techniques if you have sufficient information to create new data points without making the data bias.
 
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
*How many interest_id values exist in the fresh_segments.interest_metrics table but not in the fresh_segments.interest_map table? What about the other way around?*
**The results shows certain ads interst which never had any customer interact through out the captured period in the interest_metric table. Also all click interest were properly summarized in the mapping table as there was no interest_id on metrics that isnt present in the interest_map**
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
*Summarise the id values in the fresh_segments.interest_map by its total record count in this table*
	
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

 ## Question 6
*What sort of table join should we perform for our analysis and why? Check your logic by checking the rows where interest_id = 21246 in your joined output and include all columns from fresh_segments.interest_metrics and all columns from fresh_segments.interest_map except from the id column.*
	
```sql
	
--  Select all record for 21246
WITH CTE_1 AS 
(SELECT * FROM interest_metrics
 WHERE interest_id = 21246)

-- Join the information from the metric table to find all matching info from the mapping table, with all interest_id in the metrics table only      
SELECT 
    CTE_1.*,
    B.interest_name,
    B.interest_summary,
    B.created_at,
    B.last_modified
FROM CTE_1
LEFT JOIN interest_map B 
ON CTE_1.interest_id = B.id
WHERE CTE_1.month_year IS NOT NULL;
```
	
| _month | _year | month_year  | interest_id | composition | index_value | ranking | percentile_ranking | interest_name                        | interest_summary                          | created_at | last_modified |
| ------ | ----- | -----------| -----------| ----------- | -----------| ------- | ----------------- | ------------------------------------ | ----------------------------------------- | ---------- | ------------- |
| 7      | 2018  | 2018-07-01 | 21246       | 2.26        | 0.65       | 722     | 0.96              | Readers of El Salvadoran Content    | People reading news from El Salvadoran media sources. |            |               |
| 8      | 2018  | 2018-08-01 | 21246       | 2.13        | 0.59       | 765     | 0.26              | Readers of El Salvadoran Content    | People reading news from El Salvadoran media sources. |            |               |
| 9      | 2018  | 2018-09-01 | 21246       | 2.06        | 0.61       | 774     | 0.77              | Readers of El Salvadoran Content    | People reading news from El Salvadoran media sources. |            |               |
| 10     | 2018  | 2018-10-01 | 21246       | 1.74        | 0.58       | 855     | 0.23              | Readers of El Salvadoran Content    | People reading news from El Salvadoran media sources. |            |               |
| 11     | 2018  | 2018-11-01 | 21246       | 2.25        | 0.78       | 908     | 2.16              | Readers of El Salvadoran Content    | People reading news from El Salvadoran media sources. |            |               |
| 12     | 2018  | 2018-12-01 | 21246       | 1.97        | 0.7        | 983     | 1.21              | Readers of El Salvadoran Content    | People reading news from El Salvadoran media sources. |            |               |
| 1      | 2019  | 2019-01-01 | 21246       | 2.05        | 0.76       | 954     | 1.95              | Readers of El Salvadoran Content    | People reading news from El Salvadoran media sources. |            |               |
| 2      | 2019  | 2019-02-01 | 21246       | 1.84        | 0.68       | 1109    | 1.07              | Readers of El Salvadoran Content    | People reading news from El Salvadoran media sources. |            |               |
| 3      | 2019  | 2019-03-01 | 21246       | 1.75        | 0.67       | 1123    | 1.14              | Readers of El Salvadoran Content    | People reading news from El Salvadoran media sources. |            |               |
| 4      | 2019  | 2019-04-01 | 21246       | 1.58        | 0.63       | 1092    | 0.64              | Readers of El Salvadoran Content    | People reading news from El Salvadoran media sources. |            |

## Question 7  
*Are there any records in your joined table where the month_year value is before the created_at value from the fresh_segments.interest_map table? Do you think these values are valid and why?*
```sql 
-- Solution - involves two steps:
-- I used step one to confirm records with metric date lesser than mapping date, next the step two was used to do a more indepth search of drilling down in terms of month and year rather than just full date as we formated the month_year to the beginning of the month
-- More explanation: The first thing to check is how many rows are there per unique identifier in our table 
-- in this case it’s going to be the id column from our fresh_segments.interest_map table. 
-- So there are definitely rows which show this characteristic - however, when we think about this from a deeper perspective
-- all of our metrics look like they are created monthly! Having the beginning of the month may just be a proxy for a 
-- summary version of all of our aggregated metrics throughout the month - so in this case we need to be wary that the 
-- month_year column might well be before our created_at column - but it shouldn’t be from an earlier month.
-- Let’s confirm this by comparing the truncatated beginning of month for each created_at value with the month_year column again. 
WITH CTE_1 AS (
SELECT interest_id, month_year AS metrics_year_months
FROM  interest_metrics
), 
CTE_2 AS (
SELECT id, created_at, created_at AS map_year_months
FROM interest_map
)
SELECT 
    COUNT(*) AS Records
FROM
    CTE_1 INNER JOIN CTE_2 
    ON CTE_1.interest_id = CTE_2.id
WHERE
    CTE_1.metrics_year_months < CTE_2.map_year_months;

-- STEP 2
WITH CTE_1 AS (SELECT 
    interest_id,
    DATE_FORMAT(month_year, '%Y-%m') AS metrics_year_months
FROM
    interest_metrics)
        
SELECT 
    CTE_1.*,
    B.created_at,
    DATE_FORMAT(B.created_at, '%Y-%m') AS month_created
FROM CTE_1
	LEFT JOIN interest_map B 
    ON CTE_1.interest_id = B.id
WHERE
    metrics_year_months < DATE_FORMAT(B.created_at, '%Y-%m');	
```	
**Indeed we can see that there are no rows for this query - so all of our data points seem to be valid for our case study!**
| interest_id | metrics_year_months | created_at  | month_created |
| ----------- | ------------------ | ----------- | ------------- |


