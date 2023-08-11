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
set week_number=weekofyear(week_date);

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

This more of a lenghtly solution but the first naive one I could come up with. I will update this as I progress with a better one. The temporary table was generated with the help of ChatGPT was not written manually. 

```
CREATE TEMPORARY TABLE temp_numbers (
    number INT
);

INSERT INTO temp_numbers (number)
VALUES
    (1), (2), (3), (4), (5), (6), (7), (8), (9), (10),
    (11), (12), (13), (14), (15), (16), (17), (18), (19), (20),
    (21), (22), (23), (24), (25), (26), (27), (28), (29), (30),
    (31), (32), (33), (34), (35), (36), (37), (38), (39), (40),
    (41), (42), (43), (44), (45), (46), (47), (48), (49), (50),
    (51), (52);
    

select distinct(number) as week_number from temp_numbers
left join weekly_sales on temp_numbers.number=weekly_sales.week_number
where week_number is null
order by number;
```
![image](https://github.com/praveen555/Danny_MA_SQL_Challenges/assets/23379996/79dc8921-9eef-41f1-9608-3cd3d7c63261)


3. How many total transactions were there for each year in the dataset?

```
select calendar_year,sum(transactions) as total_transaction from weekly_sales
group by calendar_year;
```
![image](https://github.com/praveen555/Danny_MA_SQL_Challenges/assets/23379996/a50d2c91-e3c8-439d-98f3-c7ec24eaaf2f)


4.What is the total sales for each region for each month?

```
select calendar_year,month_number,region,sum(transactions) as monthly_sales from weekly_sales
group by region,calendar_year,month_number
order by calendar_year desc,month_number desc,monthly_sales desc;
```
![image](https://github.com/praveen555/Danny_MA_SQL_Challenges/assets/23379996/e01c027b-12d8-44ca-954c-75cbdfff06da)


5. What is the total count of transactions for each platform

```
select platform,sum(transactions) as total_count from weekly_sales
group by platform
order by total_count desc;
```
![image](https://github.com/praveen555/Danny_MA_SQL_Challenges/assets/23379996/b186cf43-467e-4a57-bfa9-59e3c925d728)


6. What is the percentage of sales for Retail vs Shopify for each month?

```
with cte1 as 
(
select calendar_year, month_number,platform,sum(sales) as monthly_amount from weekly_sales
group by calendar_year,month_number,platform
),
cte2 as 
(
select calendar_year,month_number as mnt, sum(sales) as total_month_sales from weekly_sales
group by calendar_year,month_number
)
select cte2.calendar_year,cte2.mnt,platform, round((monthly_amount/total_month_sales)*100,2) as percentage_sales from cte2 
inner join cte1 on cte1.calendar_year=cte2.calendar_year and cte1.month_number=cte2.mnt;
```
![image](https://github.com/praveen555/Danny_MA_SQL_Challenges/assets/23379996/bb51fdd5-3069-4819-8dfc-9cb3b49efe35)


7.What is the percentage of sales by demographic for each year in the dataset?

```
with cte1 as 
( select calendar_year, sum(sales) as amount from weekly_sales
group by calendar_year
),
cte2 as 
( select calendar_year,demographics,sum(sales) as cat_sales from weekly_sales
group by calendar_year,demographics
)
select cte1.calendar_year,demographics,round((cat_sales/amount)*100,2) as percentage_sales from cte2
inner join cte1 on cte1.calendar_year=cte2.calendar_year;
```
![image](https://github.com/praveen555/Danny_MA_SQL_Challenges/assets/23379996/0a5553a9-e2f0-4082-a36f-9da30ab5d5b3)

8. Which age_band and demographic values contribute the most to Retail sales?

```
select age_band,demographics,sum(sales) as total_sales from weekly_sales
where platform="Retail"
group by age_band,demographics
order by total_sales DESC
LIMIT 1;
```
![image](https://github.com/praveen555/Danny_MA_SQL_Challenges/assets/23379996/a9df329b-d0fc-4a67-80e7-c8d9d3ab1167)

9. Can we use the avg_transaction column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?

Averaging the already average value will not present accurate results. More accurate would be to do group by sum for each platform year wise and find the average per year. 

```
select calendar_year, platform, sum(sales) as total_sales,sum(transactions) as total_transactions, round(sum(sales)/sum(transactions),2) as avg_transactions from weekly_sales
group by calendar_year, platform

```

![image](https://github.com/praveen555/Danny_MA_SQL_Challenges/assets/23379996/5f72cfd5-3538-4139-98cd-40e0a024ca40)




## 3. Before & After Analysis

1. What is the total sales for the 4 weeks before and after 2020-06-15? What is the growth or reduction rate in actual values and percentage of sales?

Initially I did interval +4 week until i had to look into the question again understand. Take away lesson is the spend some time understanding the questions as it is not always straightforward as it seems. 

Also i think I love CTE. This query break down everything from a beginners presepctive. I will try to optimize it in future. 

```
-- What is the total sales for the 4 weeks before and after 2020-06-15? What is the growth or reduction rate in actual values and percentage of sales?
with cte1 as 
(
select sum(sales) as after_sales from weekly_sales
where  (week_date>='2020-06-15' and week_date <= DATE_ADD('2020-06-15',INTERVAL 3 WEEK))
),
cte2 as 
(select sum(sales) as before_sales from weekly_sales
where (week_date<'2020-06-15' and week_date >= date_sub('2020-06-15', interval 4 week))
),
cte3 as 
(
select * from cte1
join cte2 on 1=1
)
select before_sales,after_sales,after_sales-before_sales as difference_in_sales,round(((after_sales-before_sales)/before_sales)*100,2) as pct_change from cte3;
```
![image](https://github.com/praveen555/Danny_MA_SQL_Challenges/assets/23379996/5297493e-89fa-423f-869e-8e0b3eceae5f)


2. What about the entire 12 weeks before and after?

Simply change the interval duration from the previous query 

```
with cte1 as 
(
select sum(sales) as after_sales from weekly_sales
where  (week_date>='2020-06-15' and week_date <= DATE_ADD('2020-06-15',INTERVAL 11 WEEK))
),
cte2 as 
(select sum(sales) as before_sales from weekly_sales
where (week_date<'2020-06-15' and week_date >= date_sub('2020-06-15', interval 12 week))
),
cte3 as 
(
select * from cte1
join cte2 on 1=1
)
select before_sales,after_sales,after_sales-before_sales as difference_in_sales,round(((after_sales-before_sales)/before_sales)*100,2) as pct_change from cte3;
```
![image](https://github.com/praveen555/Danny_MA_SQL_Challenges/assets/23379996/2807dc51-25f1-4fe0-a3c2-34ed2c888876)


3. How do the sale metrics for these 2 periods before and after compare with the previous years in 2018 and 2019?

Note- While cross checking my answers I found several github repos using week_number as way to filter it and using the same 
week_number to filter across the years of 2019 & 2018. However a quick analysis below reveals that the week_number for the 
years in 2018 and 2019 change to 24 instead of 25. 

```
select weekofyear('2018-06-15'), weekofyear('2019-06-15'),weekofyear('2020-06-15')
```
![image](https://github.com/praveen555/Danny_MA_SQL_Challenges/assets/23379996/6c328d1b-f19f-4eba-ab8e-87891a09d4ae)

hence using 25 as base week_number if we do +12 we get 37 week_number which is right for year 2020. However for year 2018 and 2019
the week_number is 24 and doing +12 weeks will give us 36 week_number for after sales. 

for before sales 25-12= 13 is right for 2020 however for 2018/19 it should be 24-12=12. The same goes for 4 week period. 

Hence the correct results I believe is this :
![image](https://github.com/praveen555/Danny_MA_SQL_Challenges/assets/23379996/bd1cd002-7b52-4476-b105-081474094c6b)

```
-- How do the sale metrics for these 2 periods before and after compare with the previous years in 2018 and 2019?
with cte1 as 
(
select calendar_year,sum(sales) as after_sales from weekly_sales
 where  (week_date>='2018-06-15' and week_date <= DATE_ADD('2018-06-15',INTERVAL 3 WEEK))
 or (week_date>='2019-06-15' and week_date <= DATE_ADD('2019-06-15',INTERVAL 3 WEEK))
 or  (week_date>='2020-06-15' and week_date <= DATE_ADD('2020-06-15',INTERVAL 3 WEEK))
 group by calendar_year
),
cte2 as 
(select calendar_year as yr, sum(sales) as before_sales from weekly_sales
where (week_date<'2018-06-15' and week_date >= date_sub('2018-06-15', interval 4 week)) 
or (week_date<'2019-06-15' and week_date >= date_sub('2019-06-15', interval 4 week)) 
or (week_date<'2020-06-15' and week_date >= date_sub('2020-06-15', interval 4 week))
group by calendar_year 
),
cte3 as 
(
select * from cte1
join cte2 on cte1.calendar_year=cte2.yr
)
select calendar_year,before_sales,after_sales,after_sales-before_sales as difference_in_sales,round(((after_sales-before_sales)/before_sales)*100,2) as pct_change from cte3; 
```
If we change the values of interval to 11 and 12 we get the 4 week period
![image](https://github.com/praveen555/Danny_MA_SQL_Challenges/assets/23379996/52939378-f81e-4443-a726-d866c6232ab1)

My thoughts on this :

1. The general trend observed is the the 4 week period more sales happen in the first half of year (though we technically looking 1 month before and after)
2. A general observation again from the table is that a sales happen more in the first quater of the year than in the second quater. e.g. we are looking
   for range between mid Feb (15th Feb) to mid june (15th June). After that the sales decrease for the next quater.

## Final thoughts
## 3. Introducing sustainable packaging in June 2020 lead to increased after sales due to which the difference in sales got reduced. If the company did not do the change there might as well a high difference in year 2020 leading to decreased sales as we might expect the difference to be close to  -25% as indicated in the previous 2 years of 2018 & 2019. 

## Feedback and thoughts on this is welcome :). Hope you had a good learning and fun session as I did. 





   









