## Data Mart Challenge #5

## Data Cleaning

This is a more step by step guide instead of single query. 

1. Convert the week_date to a DATE format

```
alter table weekly_sales
add formatted_date DATE;

update weekly_sales
set formatted_date=str_to_date(week_date,"%d/%m/%Y");

alter table weekly_sales
drop column week_date;

alter table weekly_sales
change column formatted_date week_date DATE;
```

![image](https://github.com/praveen555/Danny_MA_SQL_Challenges/assets/23379996/9024a773-6491-4b3e-b5e0-a18971cf1d09)

2. Add a week_number as the second column for each week_date value, for example any value from the 1st of January to 7th of January will be 1, 8th to 14th will be 2 etc

```
alter table weekly_sales
add week_number int ;

update weekly_sales
set week_number=week(week_date);

```
![image](https://github.com/praveen555/Danny_MA_SQL_Challenges/assets/23379996/d2d727bd-7265-4ced-a856-138ba2943140)


3. Add a month_number with the calendar month for each week_date value as the 3rd column

```
alter table weekly_sales
add month_number int ;

update weekly_sales
set month_number=month(week_date);

```
![image](https://github.com/praveen555/Danny_MA_SQL_Challenges/assets/23379996/1cf29b83-2863-4b6e-a69c-366b074bff5f)

4. Add a calendar_year column as the 4th column containing either 2018, 2019 or 2020 values

   ```
   alter table weekly_sales
   add calendar_year int ;
   update weekly_sales
   set calendar_year=year(week_date);
   ```
   ![image](https://github.com/praveen555/Danny_MA_SQL_Challenges/assets/23379996/7ba050ec-f47d-4a70-aa13-b0eef9546fed)

5. Add a new column called age_band after the original segment column using the following mapping on the number inside the segment value
   
```
alter table weekly_sales
add age_band varchar(50) ;

update weekly_sales
set age_band= case
when segment="null" then NULL
when substring(segment,2,2)='1' then "Young Adults"
when substring(segment,2,2)='2' then "Middle Age"
when substring(segment,2,2)='3' or substring(segment,2,2)=4 then "Retriees"
else NULL
end;
```
![image](https://github.com/praveen555/Danny_MA_SQL_Challenges/assets/23379996/e6d5c09b-0ffe-4e8f-a310-c12d30bbab8d)

6. Add a new demographic column using the following mapping for the first letter in the segment values

```
alter table weekly_sales
add demographics varchar(50) ;

update weekly_sales
set demographics= case
when segment="null" then NULL
when substring(segment,1,1)='C' then "Couples"
when substring(segment,1,1)='F' then "Family"
else NULL
end;
```
![image](https://github.com/praveen555/Danny_MA_SQL_Challenges/assets/23379996/6c3ceb1f-015f-4381-9323-659f5406b43f)

7. Ensure all null string values with an "unknown" string value in the original segment column as well as the new age_band and demographic columns

```
   alter table weekly_sales
   modify segment varchar(20);
   
   update weekly_sales
   set segment= case
   when segment="null" then "unkown"
   else segment
   end,
   demographics=case
   when demographics is null then "unkown"
   else demographics
   end,
   age_band=case
   when age_band is null then "unkown"
   else age_band
   end;
```
![image](https://github.com/praveen555/Danny_MA_SQL_Challenges/assets/23379996/cbc87b4b-9eab-4c3d-88b5-31f856ab89d8)

Note- Make sure to increase the column length of segment as it was initially in the size of only 4. 

8. Generate a new avg_transaction column as the sales value divided by transactions rounded to 2 decimal places for each record

```
alter table weekly_sales
add  avg_transaction float;

update weekly_sales
set avg_transaction= round(sales/transactions,2)
```
![image](https://github.com/praveen555/Danny_MA_SQL_Challenges/assets/23379996/48e9c8cd-d876-4633-ad88-3d1a29a2307e)


Note : I initally did not bother to order the columns as it does not matter much in my opinion however for visibility sake and to follow the question instructions I created a temporary table to order it as it seems easiers. In Pandas you choose the columns in the order to reorganize the dataframe.
You can chose to persist this temporary table in this ordered format if you want to. I however chose not to as it does not make a difference in the further analysis. Since I have broken down steps my table persists even when the connection is broken or I quit the application. While it is always recommended to create a backup table and work on the new table. 

```
create temporary table temp_weekly_sales as 
select week_date,week_number,month_number,calendar_year,segment,age_band,demographics,region
platform,customer_type,transactions,sales,avg_transaction from weekly_sales;

select * from temp_weekly_sales;
```

![image](https://github.com/praveen555/Danny_MA_SQL_Challenges/assets/23379996/2badc1fa-8994-453a-9587-508f59b80769)


## 2. Data Exploration 

1.What day of the week is used for each week_date value?
```
select week_date, dayofweek(week_date) as day_of_week from weekly_sales;
```
![image](https://github.com/praveen555/Danny_MA_SQL_Challenges/assets/23379996/3423b898-1eaf-4700-8964-1021baa012cd)

2. What range of week numbers are missing from the dataset?

```
```
3. How many total transactions were there for each year in the dataset?

```
select calendar_year,count(transactions) as count_transaction from weekly_sales
group by calendar_year;
```
![image](https://github.com/praveen555/Danny_MA_SQL_Challenges/assets/23379996/286f5d12-d2b4-4e2c-984f-dce0f0e8294a)

4.What is the total sales for each region for each month?

```
select calendar_year,month_number,region,sum(transactions) as monthly_sales from weekly_sales
group by region,calendar_year,month_number
order by calendar_year desc,month_number desc,monthly_sales desc;
```
![image](https://github.com/praveen555/Danny_MA_SQL_Challenges/assets/23379996/e01c027b-12d8-44ca-954c-75cbdfff06da)


5. What is the total count of transactions for each platform

```
select platform,sum(transactions) as total_amount from weekly_sales
group by platform
order by total_amount desc;
```
![image](https://github.com/praveen555/Danny_MA_SQL_Challenges/assets/23379996/6fb62cbe-b760-4563-98e0-96c117bef914)

6. What is the percentage of sales for Retail vs Shopify for each month?

```
with cte1 as 
(
select calendar_year, month_number,platform,sum(transactions) as monthly_amount from weekly_sales
group by calendar_year,month_number,platform
),
cte2 as 
(
select calendar_year,month_number as mnt, sum(transactions) as total_month_sales from weekly_sales
group by calendar_year,month_number
)
select cte2.calendar_year,cte2.mnt,platform, round((monthly_amount/total_month_sales)*100,2) as percentage_sales from cte2 
inner join cte1 on cte1.calendar_year=cte2.calendar_year and cte1.month_number=cte2.mnt;
```
![image](https://github.com/praveen555/Danny_MA_SQL_Challenges/assets/23379996/be50df25-b5cc-4bae-983c-efb63c3c3552)

7.What is the percentage of sales by demographic for each year in the dataset?

```
with cte1 as 
( select calendar_year, sum(transactions) as amount from weekly_sales
group by calendar_year
),
cte2 as 
( select calendar_year,demographics,sum(transactions) as cat_sales from weekly_sales
group by calendar_year,demographics
)
select cte1.calendar_year,demographics,round((cat_sales/amount)*100,2) as percentage_sales from cte2
inner join cte1 on cte1.calendar_year=cte2.calendar_year;
```
![image](https://github.com/praveen555/Danny_MA_SQL_Challenges/assets/23379996/8396f5fb-27ad-4b30-9cc3-76c6d9d955b0)














