## Data Mart Challenge #5

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
   



