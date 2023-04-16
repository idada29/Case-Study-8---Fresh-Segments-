# <p align="center" style="margin-top: 0px;"> Case-Study-8-Fresh-Segments 🍊
## <p align="center"> Data Exploration and Cleansing

<details>
<summary> Question 1 </summary>
*Update the fresh_segments.interest_metrics table by modifying the month_year column to be a date data type with the start of the month*

 ```sql
-- You have to first change the length of the varcha to allow for the new addition of month beginning
ALTER TABLE interest_metrics MODIFY month_year VARCHAR(15);

-- Next update with a concatenation, TO ADD MONTH beginning at 01
UPDATE fresh_segments.interest_metrics
SET month_year = CONCAT("01-",month_year);

UPDATE fresh_segments.interest_metrics
SET month_year = REPLACE(month_year, '-', '/')
WHERE month_year LIKE '%-%';

UPDATE interest_metrics
SET month_year = STR_TO_DATE(month_year,"%d/%m/%Y");  
```
<details>  
  
*What is count of records in the fresh_segments.interest_metrics for each month_year value sorted in chronological order (earliest to latest) with the null values appearing first?*
 
 ```sql 
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
  
*How many interest_id values exist in the fresh_segments.interest_metrics table but not in the fresh_segments.interest_map table? What about the other way around?*
  
*Summarise the id values in the fresh_segments.interest_map by its total record count in this table*
  
*What sort of table join should we perform for our analysis and why? Check your logic by checking the rows where interest_id = 21246 in your joined output and include all columns from fresh_segments.interest_metrics and all columns from fresh_segments.interest_map except from the id column.*
  
*Are there any records in your joined table where the month_year value is before the created_at value from the fresh_segments.interest_map table? Do you think these values are valid and why?*
