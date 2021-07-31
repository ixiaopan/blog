---
title: "Daily SQL Practice"
date: "2021-07-30"
description: ""
# tags: []
categories: [
    "Computer Science",
]
series: ["Computer Science"]
katex: true
---



Practice! Practice! Practice!



<!--more-->

## Order of execution



- from
- where
- group by
- having
- select
- order by
- limit



## Leetcode





## Hackerrank



Q1 [weather-observation-station-5](https://www.hackerrank.com/challenges/weather-observation-station-5/problem)



> Query the two cities in **STATION** with the shortest and longest *CITY* names, as well as their respective lengths (i.e.: number of characters in the name). If there is more than one smallest or largest city, choose the one that comes first when ordered alphabetically.



```sql
/*
CITY: DEF, ABC, PQRS and WXY.
outputs:
	ABC 3
	PQRS 4
*/

-- v1
(SELECT city, CHAR_LENGTH(city)
from station
ORDER by CHAR_LENGTH(city), city
LIMIT 1)

UNION

(SELECT city, CHAR_LENGTH(city)
from station
ORDER by CHAR_LENGTH(city) desc, city
LIMIT 1)

-- v2
(SELECT city, CHAR_LENGTH(city)
from station
where CHAR_LENGTH(city) = (SELECT MAX(CHAR_LENGTH(city)) from station)
ORDER BY city asc
LIMIT 1)

UNION

(SELECT city, CHAR_LENGTH(city)
from station
where CHAR_LENGTH(city) = (SELECT MIN(CHAR_LENGTH(city)) from station)
ORDER BY city asc
LIMIT 1);

```



What I learned

- `UNION`
  - combine multiple result sets, the selected columns must have the same size, data type, and order
  - removes one duplicate row; To retain the duplicate row, use `UNION ALL`
- `CHAR_LENGTH()` 
  - return the length of a string, measured in characters
- `MIN(expr)/MAX(expr)` 
  - return the maximum/minimum value of `expr`



Q2 [weather-observation-station-6](https://www.hackerrank.com/challenges/weather-observation-station-6/problem)



>  Query the list of *CITY* names starting with vowels (i.e., `a`, `e`, `i`, `o`, or `u`) from **STATION**.



```SQL
-- v1
select distinct city
from station
where city LIKE 'a%' or city LIKE 'e%' or city LIKE 'i%' or city LIKE 'o%' or city LIKE 'u%'
where city LIKE '%a%' or city LIKE '%e%' or city LIKE '%i%' or city LIKE '%o%' or city LIKE '%u%'


-- v2
select distinct city
from station
where city REGEXP '^[aeiou]' -- start with vowels
where city REGEXP 'a|e|i|o|u' -- contains vowels
where city NOT REGEXP '^[aeiou]' -- not start with vowels
where city REGEXP '^[aeiou].*[aeiou]$' -- start&end with vowels


-- v3
select distinct city
from station
where LEFT(city, 1) IN ('a', 'e', 'i', 'o', 'u') -- start with vowels
where LEFT(city, 1) IN ('a', 'e', 'i', 'o', 'u') and RIGHT(city, 1) IN ('a', 'e', 'i', 'o', 'u' -- start&end with vowels

```



What I learned

- `REG_EXP` / `NOT REG_EXP`
  - match substring
- `LEFT(str, n)`
  - return the leftmost `n` characters



Q3 [The Blunder](https://www.hackerrank.com/challenges/the-blunder/problem)



> Write a query calculating the amount of error (i.e.: average monthly salaries), and round it up to the next integer.



```sql
SELECT CEIL(AVG(o.salary) - AVG(t.salary_wo_zero))
from 
    (SELECT id, REPLACE(salary, 0, '') as salary_wo_zero from employees) t
JOIN employees as o on o.id = t.id

```



What I learned

- `Every derived table must have its own alias`
- `REPLACE()`
  - return the string with all occurences of `from_str` replaced by `to_str`
- Specify table name when two tables have the same columns



Q4 [The Score Report](https://www.hackerrank.com/challenges/the-report/problem?isFullScreen=true&h_r=next-challenge&h_v=zen&h_r=next-challenge&h_v=zen)

> Generate a report containing three columns: *Name*, *Grade* and *Mark*.
>
> - in descending order by grade
> - oder by name alphabetically if there is more than one students with the same grade
> - Print "NULL" as the name if the grade is less than 8 (1-7)
> - oder by marks if there is more than one students with the same grade (1-7)



```sql
SELECT 
CASE 
    WHEN grade > 7 THEN name
    ELSE NULL
END name,
grade, marks
FROM Students JOIN Grades
WHERE Marks between Min_Mark and Max_Mark
ORDER BY grade desc, name, marks
```



What I learned

- `join/inner join` without `on` is the same as `cross join`
- `on` is required for `left/right join` but optional for `join/inner join` 
- `BETWEEN`
  - used in `where` clause to find values within a range
- `CASE WHEN`
  - evaluate a list of conditions and return one of the possible results



Q5 [earnings-of-employees](https://www.hackerrank.com/challenges/earnings-of-employees/problem?isFullScreen=true)



> find the *maximum total earnings* for all employees as well as the total number of employees who have maximum total earnings. 



```sql
SELECT months * salary as earnings, COUNT(*)
from employee
GROUP BY 1
ORDER BY 1 desc
LIMIT 1
```



What I learned

- In MySQL, you can use the column alias in the `group by/order by/having` to refer to that column



Q6 [draw-the-triangle](https://www.hackerrank.com/challenges/draw-the-triangle-1/problem?isFullScreen=true)



```SQL
--- * * * * * 
--- * * * * 
--- * * * 
--- * * 
--- *

create table Test(id integer, title varchar(10));
insert into Test(id, title) values(1, "Hello");
insert into Test(id, title) values(2, "Hello");
insert into Test(id, title) values(3, "Hello");
insert into Test(id, title) values(4, "Hello");
insert into Test(id, title) values(5, "Hello");

SELECT *, @NUMBER:=@NUMBER-1
FROM Test, (SELECT @NUMBER:=5) t ;

--- id	title	@NUMBER:=5 @NUMBER:=@NUMBER-1
--- 1	  Hello	  5          4
--- 2	  Hello	  5          3
--- 3	  Hello	  5          2
--- 4	  Hello	  5          1
--- 5	  Hello	  5          0

--- solution 1
SET @counter = 3;

SELECT *, REPEAT('* ', @counter := @counter - 1)
FROM Test
WHERE @counter > 2;


--- id	title	REPEAT('* ', @number := @number - 1)
--- 1	Hello	* *

--- solution 2
SET @counter = 3;

SELECT *, REPEAT('* ', @counter := @counter - 1)
FROM Test
LIMIT 1

--- solution 3
SELECT REPEAT('* ', @NUMBER := @NUMBER - 1) 
FROM 
  information_schema.tables, 
  (SELECT @NUMBER:=21) t 
LIMIT 20;

```

What I learned

- `SELECT` a number or string without `FROM` clause, the query will return one row with that value
- `SELECT` a number or string from a table, the query will return that value for every row in the table
- `:=` is an assignment operator
- `SET @var = val` declar a variable



Q7 [median](https://www.hackerrank.com/challenges/weather-observation-station-20)



```sql
SELECT ROUND(AVG(lat_n), 4)
from (
    SELECT 
      lat_n, 
      ROW_NUMBER() OVER(ORDER BY lat_n) AS row_num,
      COUNT(lat_n) OVER() AS t_row_num
    from station
) t2
WHERE row_num in (FLOOR((t_row_num-1)/2+1), CEIL((t_row_num-1)/2+1))

```



What I learned

- `median = ( floor(N/2) + ceil(N/2) ) / 2` where `N` is a `0-based` counter
- `OVER()`
  - `ROW_NUMBER()` to obtain the positional index for each row
  - `COUNT()` to calculate total number of a subset



Q8 [symmetric-pairs](https://www.hackerrank.com/challenges/symmetric-pairs/problem)



> Two pairs *(X1, Y1)* and *(X2, Y2)* are said to be *symmetric* *pairs* if *X1 = Y2* and *X2 = Y1*.
>
> Write a query to output all such *symmetric* *pairs* in ascending order by the value of *X*. List the rows such that *X1 ≤ Y1*.



```sql

SELECT f1.X, f1.Y
from Functions f1
JOIN Functions f2 on f1.Y = f2.X and f1.X = f2.Y

GROUP BY f1.X, f1.Y
HAVING COUNT(f1.X) > 1 or f1.X < f1.Y

ORDER BY f1.X
```



What I learned

- `HAVING COUNT(f1.X) > 1` to filter own mirrored pairs - there is only one row `(2, 2)`, which should be excluded
- `f1.X < f1.Y` to filter duplicated pairs - `(2, 5), (5, 2)`, then `(5, 2)` should be discarded



Q9 [Pivot](https://www.hackerrank.com/challenges/occupations/problem)



> [Pivot](https://en.wikipedia.org/wiki/Pivot_table) the *Occupation* column in **OCCUPATIONS** so that each *Name* is sorted alphabetically and displayed underneath its corresponding *Occupation*. The output column headers should be *Doctor*, *Professor*, *Singer*, and *Actor*, respectively.



```SQL
--- Doctor,  Professor  Singer, Actor
--- Jenny    Ashley     Meera  Jane
--- Samantha Christeen  Priya  Julia
--- NULL     Ketty      NULL   Maria


SELECT MIN(Doctor), MIN(Professor), MIN(Singer), MIN(Actor)
FROM (SELECT Name, Occupation,
       RANK() OVER(PARTITION BY Occupation ORDER BY Name) AS row_num,
       CASE WHEN Occupation = 'Doctor' THEN Name ELSE NULL END AS Doctor,
       CASE WHEN Occupation = 'Professor' THEN Name ELSE NULL END AS Professor,
       CASE WHEN Occupation = 'Singer' THEN Name ELSE NULL END AS Singer,
       CASE WHEN Occupation = 'Actor' THEN Name ELSE NULL END AS Actor
      FROM OCCUPATIONS) t
GROUP BY row_num

```



What I learned

- `MIN(expr)` can be used with numeric, **char**, datetime datatypes, ignoring `NULL` 
  - e.g. `MIN(['AA', 'AB', 'AC']) = 'AA'`
- The idea is to 
  - construct a one-hot encoding table
  - group by occupation and rank by name within each group
  - aggregate each row by grouping by within-group row number using `MIN(str)`



## References

- [join without on](https://stackoverflow.com/questions/16470942/how-to-use-mysql-join-without-on-condition/16471286)
- [SQL Alias](https://www.mysqltutorial.org/mysql-alias/#:~:text=In%20MySQL%2C%20you%20can%20use,to%20refer%20to%20the%20column.&text=The%20following%20statement%20selects%20the,GROUP%20BY%20and%20HAVING%20clauses.&text=Notice%20that%20you%20cannot%20use%20a%20column%20alias%20in%20the%20WHERE%20clause.)
- [sql-using-alias-in-group-by](https://stackoverflow.com/questions/3841295/sql-using-alias-in-group-by)
- [SQL-Hackerrank-Draw-the-triangle](https://nifannn.github.io/2017/10/24/SQL-Notes-Hackerrank-Draw-The-Triangle-1/)
- [SQL :=](https://discuss.codecademy.com/t/could-someone-explain-me-why-this-code-creates-automatic-20-lines/558795)

