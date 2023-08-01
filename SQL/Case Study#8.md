1. Update the fresh_segments.interest_metrics table by modifying the month_year column to be a date data type with the start of the month

```
update interest_metrics
set _month= cast(substring(month_year,1,2) as unsigned),
_year=cast(substring(month_year,4,7) as unsigned);
```
Addtion you could also do alter table interest_metric modify _month interger and same for _year to convert these values into integers. 

2.What is count of records in the fresh_segments.interest_metrics for each month_year value sorted in chronological order (earliest to latest) with the null values appearing first?


```
select count(month_year) as counts,count(*) as total_count from interest_metrics
order by month_year 
```
![image](https://github.com/praveen555/Danny_MA_SQL_Challenges/assets/23379996/2e39fdff-5d2c-4ed4-8bc8-344c87bfd581)

3.What do you think we should do with these null values in the fresh_segments.interest_metrics

Let us analyse few things before we make comment
1. number of records per year
   ![image](https://github.com/praveen555/Danny_MA_SQL_Challenges/assets/23379996/4eae8d37-794b-4ef9-8302-364b84f9fc74)

2. month wise count per year

![image](https://github.com/praveen555/Danny_MA_SQL_Challenges/assets/23379996/b51f161d-b194-4f7d-bc40-fc8bd20fc43d)

On the primary note, interest id is what would matter the most. While I did try to find ways using the ranking there wasn't any specific pattern i obsereved. Hence it is better to drop the null data. Overall there 1194 rows with null data account to 8% data loss. 

