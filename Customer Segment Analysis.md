# <p align="center" style="margin-top: 0px;"> Fresh Segments 🍊
## <p align="center"> Customer Segment Analysis

## Question 1
*Which interests have been present in all month_year dates in our dataset?Using our filtered dataset by removing the interests with less than 6 months worth of data, which are the top 10 and bottom 10 interests which have the largest composition values in any month_year? Only use the maximum composition value for each interest but you must keep the corresponding month_year*

 ```sql
  
WITH Composition_Ranking AS (
SELECT
	B.Month_Year,
    A.interest_name,
    any_value(B.composition) AS Composition,
    RANK() OVER (PARTITION BY A.interest_name ORDER BY any_value(B.composition) DESC) AS interest_rank
  FROM
    interest_metrics B
  JOIN interest_map A 
  ON A.id = B.interest_id
  WHERE B.interest_id != 0 AND B.Month_Year > 6
  GROUP BY
    1,2
),
  
-- Select the top 10 interests by composition
TOP_10 AS(
SELECT 
	Month_Year, Interest_name, Composition
FROM Composition_Ranking
WHERE interest_rank = 1
ORDER BY Composition DESC
LIMIT 10),
  
-- Select the bottom 10 interests by composition
BOTTOM_10 AS(
	SELECT 
		Month_Year, Interest_name, Composition
	FROM Composition_Ranking
	WHERE interest_rank = 1
	ORDER BY Composition ASC
	LIMIT 10)

-- Combine the top and bottom 10 interests and order them by composition
SELECT * FROM TOP_10
UNION 
SELECT * FROM BOTTOM_10
ORDER BY Composition DESC;
  
```

| Month_Year |          Interest_name          |   Composition    |
|------------|---------------------------------|------------------|
| 2018-12-01 |    Work Comes First Travelers    |      21.2       |
| 2018-07-01 |        Gym Equipment Owners      |     18.82       |
| 2018-07-01 |         Furniture Shoppers       |     17.44       |
| 2018-07-01 |      Luxury Retail Shoppers      |     17.19       |
| 2018-10-01 | Luxury Boutique Hotel Researchers |     15.15       |
| 2018-12-01 |      Luxury Bedding Shoppers     |     15.05       |
| 2018-07-01 |           Shoe Shoppers          |     14.91       |
| 2018-07-01 |  Cosmetics and Beauty Shoppers   |     14.23       |
| 2018-07-01 |         Luxury Hotel Guests      |      14.1       |
| 2018-07-01 |     Luxury Retail Researchers    |     13.97       |
| 2018-07-01 |      Readers of Jamaican Content |      1.86       |
| 2019-02-01 |      Automotive News Readers     |      1.84       |
| 2018-07-01 |            Comedy Fans           |      1.83       |
| 2019-08-01 |   World of Warcraft Enthusiasts  |      1.82       |
| 2018-08-01 |         Miami Heat Fans          |      1.81       |
| 2018-07-01 | Online Role Playing Game Enthusiasts |     1.73       |
| 2019-08-01 |   Hearthstone Video Game Fans    |      1.66       |
| 2018-09-01 |   Scifi Movie and TV Enthusiasts |      1.61       |
| 2018-09-01 |   Action Movie and TV Enthusiasts|      1.59       |
| 2019-03-01 |     The Sims Video Game Fans     |      1.57       |


## Question 2
*Which 5 interests had the lowest average ranking value?*
 
 ```sql 
SELECT 
    A.interest_name,
    ROUND(AVG(B.ranking), 1) AS ranking_average,
    COUNT(A.interest_name) AS records
FROM
    interest_metrics B
        JOIN
    interest_map A ON A.id = B.interest_id
WHERE
    B.interest_id != 0
GROUP BY 1
ORDER BY ranking_average ASC
LIMIT 5;

```

interest_name                 | ranking_average | records
------------------------------|----------------|--------
Winter Apparel Shoppers       | 1.0            | 9
Fitness Activity Tracker Users| 4.1            | 9
Mens Shoe Shoppers            | 5.9            | 14
Elite Cycling Gear Shoppers   | 7.8            | 5
Shoe Shoppers                 | 9.4            | 14


 ## Question 3
*Which 5 interests had the largest standard deviation in their percentile_ranking value?*

 ```sql
-- Select the interest_id, interest_name, standard deviation of percentile_ranking, max percentile_ranking, min percentile_ranking, and count of records for each interest
SELECT 
	B.interest_id,
    A.interest_name,
    ROUND(STDDEV(B.percentile_ranking), 1) AS stddev_ranking,
	MAX(B.percentile_ranking) AS max_pc_ranking,
    MIN(B.percentile_ranking) AS min_pc_ranking,
    COUNT(A.interest_name) AS records
FROM
    interest_metrics B
JOIN
    interest_map A 
ON B.interest_id =  A.id
GROUP BY 1,2
ORDER BY stddev_ranking DESC
LIMIT 5;
 ```
interest_id | interest_name             | stddev_ranking | max_pc_ranking | min_pc_ranking | records
|----------- | ------------------------ | -------------- | -------------- | -------------- | --------|
6260        | Blockbuster Movie Fans    | 29.2           | 60.63          | 2.26           | 2
131         | Android Fans              | 27.5           | 75.03          | 4.84           | 5
23          | Techies                   | 27.5           | 86.69          | 7.92           | 6
150         | TV Junkies                | 27.2           | 93.28          | 10.01          | 5
38992       | Oregon Trip Planners      | 26.9           | 82.44          | 2.2            | 10
   



## Question 4
*For the 5 interests found in the previous question - what was minimum and maximum percentile_ranking values for each interest and its corresponding year_month value? Can you describe what is happening for these 5 interests?*

```sql
WITH std_rank AS
(SELECT 
	B.interest_id,
    A.interest_name,
    ROUND(STDDEV(B.percentile_ranking), 1) AS stddev_ranking,
	MAX(B.percentile_ranking) AS max_pc_ranking,
    MIN(B.percentile_ranking) AS min_pc_ranking,
    COUNT(A.interest_name) AS records
FROM
    interest_metrics B
JOIN
    interest_map A 
ON B.interest_id =  A.id
GROUP BY 1,2
ORDER BY stddev_ranking DESC
LIMIT 5)

SELECT 
    C.interest_name,
	B.Month_Year,
    ROUND(B.ranking, 0) AS ranking,
    ROUND(B.percentile_ranking, 2) AS pc_ranking,
	MAX(B.composition) AS Composition
FROM  interest_metrics B
JOIN std_rank C
ON B.interest_id = C.interest_id 
GROUP BY 1,2,3,4;
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
*How would you describe our customers in this segment based off their composition and ranking values? What sort of products or services should we show to these customers and what should we avoid?*
	



