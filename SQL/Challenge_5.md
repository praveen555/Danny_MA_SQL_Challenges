## Data Mart Challenge #5

## Data Cleaning

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
![image](https://github.com/praveen555/Danny_MA_SQL_Challenges/assets/23379996/7fc5959d-3f6a-4423-b9cd-5c4c9b557bf3)










