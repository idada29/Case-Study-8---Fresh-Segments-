# <p align="center" style="margin-top: 0px;">  **`Case-Study-8-Fresh-Segments`**

<details>
<summary>About Project</summary>
Fresh Segments is a digital marketing agency that helps other businesses analyse trends in online ad click behaviour for their unique customer base.
Clients share their customer lists with the Fresh Segments team who then aggregate interest metrics and generate a single dataset worth of metrics for further analysis. In particular - the composition and rankings for different interests are provided for each client showing the proportion of their customer list who interacted with online assets related to each interest for each month.
</details>


<details>
<summary> Datasets - Interest Metrics & Interest Map </summary>
The Interest Metrics table contains information about aggregated interest metrics for a specific major client of Fresh Segments which makes up a large proportion of their customer base. Each record in this table represents the performance of a specific interest_id based on the client’s customer base interest measured through clicks and interactions with specific targeted advertising content. 
 
 ---
 | Month | Year | Month_Year | Interest_ID | Composition | Index_Value | Ranking | Percentile_Ranking |
|------:|-----:|-----------|------------:|------------:|------------:|--------:|-------------------:|
|     7 | 2018 |    07-2018 |       32486 |       11.89 |        6.19 |       1 |              99.86 |
|     7 | 2018 |    07-2018 |        6106 |        9.93 |        5.31 |       2 |              99.73 |
|     7 | 2018 |    07-2018 |       18923 |       10.85 |        5.29 |       3 |              99.59 |
|     7 | 2018 |    07-2018 |        6344 |       10.32 |         5.1 |       4 |              99.45 |
|     7 | 2018 |    07-2018 |         100 |       10.77 |        5.04 |       5 |              99.31 |
|     7 | 2018 |    07-2018 |          69 |       10.82 |        5.03 |       6 |              99.18 |
|     7 | 2018 |    07-2018 |          79 |       11.21 |        4.97 |       7 |              99.04 |
|     7 | 2018 |    07-2018 |        6111 |       10.71 |        4.83 |       8 |               98.9 |
|     7 | 2018 |    07-2018 |        6214 |        9.71 |        4.83 |       8 |               98.9 |
|     7 | 2018 |    07-2018 |       19422 |       10.11 |        4.81 |      10 |              98.63 |
 
 
In July 2018, the composition metric is 11.89, meaning that 11.89% of the client’s customer list interacted with the interest interest_id = 32486 - we can link interest_id to a separate mapping table to find the segment name called “Vacation Rental Accommodation Researchers”

The index_value is 6.19, means that the composition value is 6.19x the average composition value for all Fresh Segments clients’ customer for this particular interest in the month of July 2018.

The ranking and percentage_ranking relates to the order of index_value records in each month year. 
 
</details>

```sql
DESCRIBE interest_metrics;
```
 
 
| Field          | Type         | Null | Key |
|----------------:|--------------|------|-----|
| _month         | varchar(4)   | YES  |     |
| _year          | varchar(4)   | YES  |     |
| month_year     | date         | YES  |     |
| interest_id    | varchar(5)   | YES  | PRI |
| composition    | float        | YES  |     |
| index_value    | float        | YES  |     |
| ranking        | int          | YES  |     |
| percentile_ranking | float   | YES  |     |
 



![8](https://user-images.githubusercontent.com/22597020/232243814-f080ffd6-c026-4bc0-85f9-4452be1cc991.png)
