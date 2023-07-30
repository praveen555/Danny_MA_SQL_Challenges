                                                                    Challenge #6 

1. Using the following DDL schema details to create an ERD for all the Clique Bait datasets.

Click here to access the DB Diagram tool to create the ERD.
![image](https://github.com/praveen555/Danny_MA_SQL_Challenges/assets/23379996/cc401f25-81a9-4056-8d07-35701a34e566)

2. Digital Analysis

1. How many users are there?
   ```
      select count(distinct(user_id)) as count_users from users;
   ```
![image](https://github.com/praveen555/Danny_MA_SQL_Challenges/assets/23379996/7cb3cdc7-7468-4bd4-a1e5-379044a316f7)

2. How many cookies does each user have on average?

   ```
   select user_id,count(cookie_id) as cookie_count from users
   group by user_id
   order by user_id;
   ```
   ![image](https://github.com/praveen555/Danny_MA_SQL_Challenges/assets/23379996/93f0ef93-4986-44b4-ac2a-30e2a93c1021)

3. What is the unique number of visits by all users per month?
   
