# <p align="center" style="margin-top: 0px;"> Case-Study-8-Fresh-Segments 🍊
## <p align="center"> Data Exploration and Cleansing


*1. Update the fresh_segments.interest_metrics table by modifying the month_year column to be a date data type with the start of the month*

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

*2. What is count of records in the fresh_segments.interest_metrics for each month_year value sorted in chronological order (earliest to latest) with the null values appearing first?*
 
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
|35964 | 1        | People in this audience participate and/or spectate in eSports (Competitive/Professional video gaming), follow various ...|
|40185 | 1        | People researching news and trends in astronomy.                                                                          |
|40186 | 1        | Consumers watching and reading about WWE and pro wrestling.                                                               |
|42010 | 1        | People reading Minecraft news and following gaming trends.                                                               |
|42400 | 1        | People reading Diablo news and following gaming trends.                                                                  |
|47789 | 1        | People researching attractions and accommodations in Israel. These consumers are more likely to spend money on travel...|

	
	
	
*Summarise the id values in the fresh_segments.interest_map by its total record count in this table*
  
*What sort of table join should we perform for our analysis and why? Check your logic by checking the rows where interest_id = 21246 in your joined output and include all columns from fresh_segments.interest_metrics and all columns from fresh_segments.interest_map except from the id column.*
  
*Are there any records in your joined table where the month_year value is before the created_at value from the fresh_segments.interest_map table? Do you think these values are valid and why?*
