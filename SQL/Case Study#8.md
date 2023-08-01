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
   ![image](https://github.com/praveen555/Danny_MA_SQL_Challenges/assets/23379996/b82c114a-55b4-4894-9336-fc1924e26a4c)


   out of 14273 records 13080 records have common keys from interest_maps. The reamining 1193 are null values of interest_id. Otherwise is it safe to say all keys     from interest_id are present in id also.

   Do right join
   ```
    select count(id),count(interest_id),count(*) from interest_metrics
    right join interest_map on interest_metrics.interest_id=interest_map.id
   ```
     ![image](https://github.com/praveen555/Danny_MA_SQL_Challenges/assets/23379996/f2030c4e-630a-491e-83c1-4aec1cdaf65c)

      There are 7 ids in interest_map which are not present in interest_id in interest_metrics


6. Summarise the id values in the fresh_segments.interest_map by its total record count in this table

   ```
   select id,count(id) as total_count from interest_map
   group by id
   order by total_count ASC, id;
   ```
   ![image](https://github.com/praveen555/Danny_MA_SQL_Challenges/assets/23379996/1cdd0190-9e58-4e98-860c-bc3e2d59c653)

   All the records have exactly one count, which means there is no duplication. This could also be checked using a where clause to see if the total count >1 for       any record

7. What sort of table join should we perform for our analysis and why? Check your logic by checking the rows where interest_id = 21246 in your joined output and include all columns from fresh_segments.interest_metrics and all columns from fresh_segments.interest_map except from the id column.

   ```
    select _month, _year, month_year, interest_id, composition, index_value, ranking, percentile_ranking, interest_name, interest_summary, created_at, 
    last_modified from interest_metrics
    inner join interest_map on interest_metrics.interest_id=interest_map.id
    where interest_id=21246;
    ```

![image](https://github.com/praveen555/Danny_MA_SQL_Challenges/assets/23379996/9c39d924-9870-42c6-8453-118bb9f6cfcc)

Since we dedicded to do away with null values doing a inner join seems a good option as this would only keep the keys which are present in both the tables. For the interest_id=21246 we see _month , _year_ columns are bad and contain null values. Going by the pattern we could make it 05-2019 or leave it all together.


8. Are there any records in your joined table where the month_year value is before the created_at value from the fresh_segments.interest_map table? Do you think these values are valid and why?

```
select _month, _year, month_year, interest_id, composition, index_value, ranking, percentile_ranking, interest_name, interest_summary, created_at, last_modified from interest_metrics
inner join interest_map on interest_metrics.interest_id=interest_map.id
where  _year < year(created_at)
```
Strictly speaking I was trying to filter out by year first and then month however i could not find any values which are having _year less than the year of create_at
however when I do where  _year <= year(created_at) and _month<= month(created_at) i get some values returned. What makes it difficult to judge is we do not the day column in interest_metrics table to compare these. 

Even if the create_at value is greater than _year _month table the values would have been valid. Let's say you indetify a new customer segment which did not exist earlier. You create a new id for this segment with on this day. However the events of clicking the ads have happened in the past. Once you assign this id you can recalcualte your table with the new ids and get the results. New shoppings catergories or new trends come and go in months/weeks times the faster you are in identifying these trends the more up to date your records will be. Otherwise it is fine if you have a lag to observe if something is trending and then assigning the id for this segement and then redoing your calculations. 





 


