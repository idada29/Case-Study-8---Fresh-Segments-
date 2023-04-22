# <p align="center" style="margin-top: 0px;"> Fresh Segments 🍊
## <p align="center"> Index Analysis

## Question 1
*What is the top 10 interests by the average composition for each month?*

 ```sql
 WITH cte as 
  (SELECT 
    C.interest_name,
	  B.Month_Year,
	  B.composition / B.index_value AS index_composition,
    RANK() OVER ( PARTITION BY B.Month_Year ORDER BY B.composition / B.index_value DESC) AS index_rank
FROM  interest_metrics B
INNER JOIN interest_map C
ON B.interest_id = C.id 
WHERE B.Month_Year IS NOT NULL
GROUP BY 1,2,3
ORDER BY B.Month_Year)

SELECT *
FROM CTE
WHERE index_rank <= 10;

```

## Question 2
*For all of these top 10 interests - which interest appears the most often?*
 
 ```sql 
WITH cte as (SELECT 
    C.interest_name,
	B.Month_Year,
	ROUND(B.composition / B.index_value,2) AS index_composition,
    RANK() OVER ( PARTITION BY B.Month_Year ORDER BY ROUND(B.composition / B.index_value,2) DESC) AS index_rank
FROM  interest_metrics B
INNER JOIN interest_map C
ON B.interest_id = C.id 
WHERE B.Month_Year IS NOT NULL
GROUP BY 1,2,3
ORDER BY B.Month_Year)

SELECT 
    interest_name, COUNT(*) AS most_loved
FROM
    CTE
WHERE
    index_rank <= 10
GROUP BY interest_name
ORDER BY most_loved DESC
LIMIT 10;

```
interest_name                            | most_loved
|----------------------------------------|-----------|
Alabama Trip Planners                   | 10
Luxury Bedding Shoppers                  | 10
Solar Energy Researchers                 | 10
Readers of Honduran Content              | 9
Nursing and Physicians Assistant Journal Researchers | 9
New Years Eve Party Ticket Purchasers    | 9
Work Comes First Travelers               | 8
Teen Girl Clothing Shoppers              | 8
Christmas Celebration Researchers        | 7
Las Vegas Trip Planners                  | 5




 ## Question 3
*What is the average of the average composition for the top 10 interests for each month?*

 ```sql
WITH cte as (SELECT 
    C.interest_name,
	B.Month_Year,
	ROUND(B.composition / B.index_value,2) AS index_composition,
    RANK() OVER ( PARTITION BY B.Month_Year ORDER BY ROUND(B.composition / B.index_value,2) DESC) AS index_rank
FROM  interest_metrics B
INNER JOIN interest_map C
ON B.interest_id = C.id 
WHERE B.Month_Year IS NOT NULL
GROUP BY 1,2,3
ORDER BY B.Month_Year)

SELECT 
    Month_Year, ROUND(AVG(index_composition),2) AS Avg_composition
FROM
    CTE
WHERE
    index_rank <= 10
GROUP BY Month_Year
ORDER BY Month_Year ASC;
 ```
Month_Year  |  Avg_composition
|----------- | ----------------|
2018-07-01  |  6.04
2018-08-01  |  5.94
2018-09-01  |  6.89
2018-10-01  |  7.07
2018-11-01  |  6.62
2018-12-01  |  6.65
2019-01-01  |  6.32
2019-02-01  |  6.58
2019-03-01  |  6.12
2019-04-01  |  5.75
2019-05-01  |  3.54
2019-06-01  |  2.43
2019-07-01  |  2.76
2019-08-01  |  2.63




## Question 4
*What is the 3 month rolling average of the max average composition value from September 2018 to August 2019 and include the previous top ranking interests in the same output shown below.*

```sql
WITH Monthly_Interest as (
    SELECT 
        C.interest_name AS interest_name,
        B.Month_Year,
        ROUND(B.composition / B.index_value,2) AS index_composition,
        RANK() OVER (PARTITION BY B.Month_Year ORDER BY ROUND(B.composition / B.index_value,2) DESC) AS index_rank
    FROM interest_metrics B
    INNER JOIN interest_map C ON B.interest_id = C.id 
    WHERE B.Month_Year IS NOT NULL
    GROUP BY 1,2,3
    ORDER BY B.Month_Year
), 
Interest_Ranking AS (
    SELECT
        month_year,
        interest_name,
        MAX(index_composition) AS max_index_composition,
        ROUND(AVG(index_composition) OVER (ORDER BY month_year ROWS BETWEEN 2 PRECEDING AND CURRENT ROW), 2) AS `3_month_moving_avg`,
		CONCAT(LAG(interest_name) OVER (ORDER BY month_year), ': ', LAG(index_composition) OVER (ORDER BY month_year)) AS `1_month_ago`,
        CONCAT(LAG(interest_name) OVER (ORDER BY month_year), ': ', LAG(index_composition,2) OVER (ORDER BY month_year)) AS `2_months_ago`
    FROM Monthly_Interest
    WHERE index_rank = 1
    GROUP BY month_year, interest_name,index_composition
)
SELECT 
    month_year,
    interest_name,
    max_index_composition,
    `3_month_moving_avg`,
    `1_month_ago`,
    `2_months_ago`
FROM
    Interest_Ranking
WHERE
    `2_months_ago` IS NOT NULL
ORDER BY month_year;
```	
interest_name | Month_Year  | ranking | pc_ranking | Composition
|--------------|-------------|---------|------------|------------|
TV Junkies    | 2018-07-01  | 49      | 93.28      | 5.3
Techies       | 2018-07-01  | 97      | 86.69      | 5.41
Android Fans  | 2018-07-01  | 182     | 75.03      | 5.09
Blockbuster Movie Fans | 2018-07-01 | 287 | 60.63 | 5.27
TV Junkies    | 2018-08-01  | 481     | 37.29      | 1.7
Techies       | 2018-08-01  | 530     | 30.9       | 1.9
Android Fans  | 2018-08-01  | 684     | 10.82      | 1.77
Techies       | 2018-09-01  | 594     | 23.85      | 1.6
TV Junkies    | 2018-10-01  | 430     | 49.82      | 2.34
Oregon Trip Planners | 2018-11-01 | 163 | 82.44   | 4.44
Oregon Trip Planners | 2018-12-01 | 410 | 58.79   | 3.61
TV Junkies    | 2018-12-01  | 619     | 37.79      | 1.72
Oregon Trip Planners | 2019-01-01 | 357 | 63.31   | 3.31
Oregon Trip Planners | 2019-02-01 | 243 | 78.32   | 4.55
Techies       | 2019-02-01  | 1015    | 9.46       | 1.89
Android Fans  | 2019-02-01  | 1058    | 5.62       | 1.85
Oregon Trip Planners | 2019-03-01 | 881 | 22.45   | 2.93
Techies       | 2019-03-01  | 1026    | 9.68       | 1.91
Android Fans  | 2019-03-01  | 1081    | 4.84       | 1.72
Oregon Trip Planners | 2019-04-01 | 849 | 22.75   | 2.33
Oregon Trip Planners | 2019-05-01 | 628 | 26.72   | 1.95
Oregon Trip Planners | 2019-06-01 | 701 | 14.93   | 1.92
Oregon Trip Planners | 2019-07-01 | 845 | 2.2     | 1.84
Oregon Trip Planners | 2019-08-01 | 857 | 25.41   | 2.53
TV Junkies    | 2019-08-01  | 1034    | 10.01      | 1.94
Techies       | 2019-08-01  | 1058    | 7.92       | 1.

	
	
## Question 5
*Provide a possible reason why the max average composition might change from month to month? Could it signal something is not quite right with the overall business model for Fresh Segments?*

	
