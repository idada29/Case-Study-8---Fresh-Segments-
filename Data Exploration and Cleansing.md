# <p align="center" style="margin-top: 0px;"> Case-Study-8-Fresh-Segments 🍊
## <p align="center"> Data Exploration and Cleansing

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
  
  
*What is count of records in the fresh_segments.interest_metrics for each month_year value sorted in chronological order (earliest to latest) with the null values appearing first?*
  
*What do you think we should do with these null values in the fresh_segments.interest_metrics*
  
*How many interest_id values exist in the fresh_segments.interest_metrics table but not in the fresh_segments.interest_map table? What about the other way around?*
  
*Summarise the id values in the fresh_segments.interest_map by its total record count in this table*
  
*What sort of table join should we perform for our analysis and why? Check your logic by checking the rows where interest_id = 21246 in your joined output and include all columns from fresh_segments.interest_metrics and all columns from fresh_segments.interest_map except from the id column.*
  
*Are there any records in your joined table where the month_year value is before the created_at value from the fresh_segments.interest_map table? Do you think these values are valid and why?*
