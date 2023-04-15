**`Case-Study-8-Fresh-Segments`**

<details>
<summary>About Project</summary>
Fresh Segments is a digital marketing agency that helps other businesses analyse trends in online ad click behaviour for their unique customer base.
Clients share their customer lists with the Fresh Segments team who then aggregate interest metrics and generate a single dataset worth of metrics for further analysis. In particular - the composition and rankings for different interests are provided for each client showing the proportion of their customer list who interacted with online assets related to each interest for each month.
</details>


<details>
<summary> Datasets - Interest Metrics & Interest Map </summary>
The Interest Metrics table contains information about aggregated interest metrics for a specific major client of Fresh Segments which makes up a large proportion of their customer base. Each record in this table represents the performance of a specific interest_id based on the client’s customer base interest measured through clicks and interactions with specific targeted advertising content. 
 <br>

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
 
</details>


![8](https://user-images.githubusercontent.com/22597020/232243814-f080ffd6-c026-4bc0-85f9-4452be1cc991.png)
