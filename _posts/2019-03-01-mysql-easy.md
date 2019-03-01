- 1 combine two tables

  ```mysql
  select FirstName,LastName,city,State from Person  left join 
  Address on Person.PersonId = Address.PersonId;
  ```

  

- 2 second hightest salary

  ```mysql
  select ifnull((select distinct Salary from Employee
  order by Salary desc limit 1 offset 1),NUll) as SecondHighestSalary;
  ```

  

- 3 employees earning more than their managers

  ```mysql
  select a.NAME as Employee from Employee as a join Employee as b
  on a.ManagerId = b.Id and a.Salary > b.Salary;
  ```

  

- 4 Duplicate Emails

  ```mysql
  select Email from Person group by Email
  having count(Email) > 1;	
  
  ---------------
  mysql> select distinct(a.Email) from Person as a join Person as b on a.Email=b.Email and a.Id!=b.Id;
  +---------+
  | Email   |
  +---------+
  | a@b.com |
  +---------+
  1 row in set (0.00 sec)
  ```

  

- 5 customers who never order

  ```mysql
  select Name as Customers from Customers where Customers.id not in (select CustomerId from Orders); 
  ```

  

- 6 Delete Duplicate Emails

  ```mysql
  delete p1.* from Person as p1 join Person as p2
  where p1.Email = p2.Email and p1.Id > p2.Id;
  ```

  

- 7 Rising Temperature

  ```mysql
  select a.Id from Weather as a join Weather as b
  on a.Temperature > b.Temperature and DATEDIFF(a.RecordDate,b.RecordDate) = 1;
  
  -----------------------------------------
  mysql> select * from Weather as a join Weather as b on a.Temperature > b.Temperature and DATEDIFF(a.RecordDDate,b.RecordDate)=1;
  +----+------------+-------------+----+------------+-------------+
  | Id | RecordDate | Temperature | Id | RecordDate | Temperature |
  +----+------------+-------------+----+------------+-------------+
  |  2 | 2015-01-02 |          25 |  1 | 2015-01-01 |          10 |
  |  4 | 2015-01-04 |          30 |  3 | 2015-01-03 |          20 |
  +----+------------+-------------+----+------------+-------------+
  2 rows in set (0.00 sec)
  ```

  

- 8 not boring movies

  ```mysql
  select * from cinema where mod(id, 2) != 0 
  and description != 'boring'
  order by rating desc;
  ---------------------------
  mysql> select * from cinema where mod(id,2)!=0 and description!='boring' 	
      -> order by rating desc;
  +----+------------+-------------+--------+
  | id | movie      | description | rating |
  +----+------------+-------------+--------+
  |  5 | House card | Interesting |    9.1 |
  |  1 | War        | great 3D    |    8.9 |
  +----+------------+-------------+--------+
  2 rows in set (0.00 sec)
  ```

  

- 9 swap salary

  ```mysql
  update salary set sex = case sex when 'm' then 'f' else 'm' end;
  
  
  mysql> select * from Salary;
  +----+------+------+--------+
  | id | name | sex  | salary |
  +----+------+------+--------+
  |  1 | A    | m    |   2500 |
  |  2 | B    | f    |   1500 |
  |  3 | C    | m    |   5500 |
  |  4 | D    | f    |    500 |
  +----+------+------+--------+
  4 rows in set (0.00 sec)
  
  
  ------------------------------------
  
  mysql> update Salary set sex=case sex when 'm' then 'f' else 'm' end;
  Query OK, 4 rows affected (0.00 sec)
  Rows matched: 4  Changed: 4  Warnings: 0
  
  mysql> select * from Salary;
  +----+------+------+--------+
  | id | name | sex  | salary |
  +----+------+------+--------+
  |  1 | A    | f    |   2500 |
  |  2 | B    | m    |   1500 |
  |  3 | C    | f    |   5500 |
  |  4 | D    | m    |    500 |
  +----+------+------+--------+
  4 rows in set (0.00 sec)
  ```

  

- 10 big countries

  ```mysql
  select name,population,area from World 
  where population > 25000000
  or area > 3000000;
  ---------------------------
  mysql> select * from World where
      -> population > 25000000 or area > 3000000;
  +----+--------------+----------+---------+------------+-----------+
  | Id | name         | contient | area    | population | gdp       |
  +----+--------------+----------+---------+------------+-----------+
  |  1 |  Afghanistan | Asia     |  652230 |   25500100 |  20343000 |
  |  3 | Algeria      | Africa   | 2381741 |   37100000 | 188681000 |
  +----+--------------+----------+---------+------------+-----------+
  2 rows in set (0.00 sec)
  ```

  

- 11 Classes More Than 5 Students

  ```mysql
  mysql> select class from courses
      -> group by class;
  +----------+
  | class    |
  +----------+
  | Biology  |
  | Computer |
  | English  |
  | Math     |
  +----------+
  4 rows in set (0.00 sec)
  
  mysql> select count(class) from courses group by class;
  +--------------+
  | count(class) |
  +--------------+
  |            1 |
  |            1 |
  |            1 |
  |            6 |
  +--------------+
  4 rows in set (0.00 sec)
  
  mysql> select class from courses group by class
      -> having count(class)>=5;
  +-------+
  | class |
  +-------+
  | Math  |
  +-------+
  1 row in set (0.00 sec)
  ```

  





