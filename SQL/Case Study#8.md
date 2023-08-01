1. Update the fresh_segments.interest_metrics table by modifying the month_year column to be a date data type with the start of the month

```
update interest_metrics
set _month= cast(substring(month_year,1,2) as unsigned),
_year=cast(substring(month_year,4,7) as unsigned);
```

2.What is count of records in the fresh_segments.interest_metrics for each month_year value sorted in chronological order (earliest to latest) with the null values appearing first?


```
select count(month_year) as counts,count(*) as total_count from interest_metrics
order by month_year 
```
![image](https://github.com/praveen555/Danny_MA_SQL_Challenges/assets/23379996/2e39fdff-5d2c-4ed4-8bc8-344c87bfd581)

