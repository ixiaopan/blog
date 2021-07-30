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



## Questions



Q1 [weather-observation-station-5](https://www.hackerrank.com/challenges/weather-observation-station-5/problem)

Query the two cities in **STATION** with the shortest and longest *CITY* names, as well as their respective lengths (i.e.: number of characters in the name). If there is more than one smallest or largest city, choose the one that comes first when ordered alphabetically.



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
  - combine multiple result sets
  - removes one duplicate row; To retain the duplicate row, use `UNION ALL`
- `CHAR_LENGTH()` 
  - return the length of a string, measured in characters
- `MIN(expr)/MAX(expr)` 
  - return the maximum/minimum value of `expr`



Q2 [weather-observation-station-6](https://www.hackerrank.com/challenges/weather-observation-station-6/problem)



Query the list of *CITY* names starting with vowels (i.e., `a`, `e`, `i`, `o`, or `u`) from **STATION**.



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



Write a query calculating the amount of error (i.e.: average monthly salaries), and round it up to the next integer.



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

Generate a report containing three columns: *Name*, *Grade* and *Mark*.

- in descending order by grade
- oder by name alphabetically if there is more than one students with the same grade
- Print "NULL" as the name if the grade is less than 8 (1-7)
- oder by marks if there is more than one students with the same grade (1-7)



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





## References

- https://stackoverflow.com/questions/16470942/how-to-use-mysql-join-without-on-condition/16471286



