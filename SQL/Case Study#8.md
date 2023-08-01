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

3. month wise count per year

   ![image](https://github.com/praveen555/Danny_MA_SQL_Challenges/assets/23379996/b51f161d-b194-4f7d-bc40-fc8bd20fc43d)

On the primary note, interest id is what would matter the most. While I did try to find ways using the ranking there wasn't any specific pattern i obsereved. Hence it is better to drop the null data. Overall there 1194 rows with null data account to 8% data loss. 

4. How many interest_id values exist in the fresh_segments.interest_metrics table but not in the fresh_segments.interest_map table? What about the other way around?

   Do a left join

   ```
   select count(interest_id),count(*) from interest_metrics
   left join interest_map on interest_metrics.interest_id=interest_map.id
   ```
   ![image](https://github.com/praveen555/Danny_MA_SQL_Challenges/assets/23379996/99fba271-119c-4a17-8a4d-15f11d7648e1)

   out of 14273 records 13080 records have common keys from interest_metrics. The reamining 1193 are null values of interest_id.

   Do right join
   ```
   select count(id),count(*) from interest_metrics
   right join interest_map on interest_metrics.interest_id=interest_map.id
   ```
   ![image](https://github.com/praveen555/Danny_MA_SQL_Challenges/assets/23379996/94dd7f4f-1fd1-4905-870c-da93ea56bbf3)

   all the keys present in interest_map are also present in interest_metrics as we see no difference.

5. Summarise the id values in the fresh_segments.interest_map by its total record count in this table

   ```
   select id,count(id) as total_count from interest_map
   group by id
   order by total_count ASC, id;
   ```
   ![image](https://github.com/praveen555/Danny_MA_SQL_Challenges/assets/23379996/1cdd0190-9e58-4e98-860c-bc3e2d59c653)

   All the records have exactly one count, which means there is no duplication. This could also be checked using a where clause to see if the total count >1 for       any record


