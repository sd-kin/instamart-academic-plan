---
layout: post
title:  "Решения"
subject: "Упражнения hackerrank"
date:   2018-11-11 20:55:22 +0300
---

## BASIC SELECT

#### Revising the Select Query I

>Query all columns for all American cities in CITY with populations larger than 100000. The CountryCode for America is USA.

{% highlight sql %}
SELECT * FROM CITY WHERE COUNTRYCODE='USA' AND POPULATION >= 100000;
{% endhighlight %}

#### Revising the Select Query II

> Query the names of all American cities in CITY with populations larger than 120000. The CountryCode for America is USA.

{% highlight sql %}
SELECT name FROM CITY WHERE COUNTRYCODE='USA' AND POPULATION >= 120000;
{% endhighlight %}
#### Select All

> Query all columns (attributes) for every row in the CITY table.
{% highlight sql %}
SELECT * FROM CITY;
{% endhighlight %}

#### Select By ID

> Query all columns for a city in CITY with the ID 1661.

{% highlight sql %}
SELECT * FROM CITY WHERE ID=1661;
{% endhighlight %}

#### Japanese Cities' Attributes

> Query all attributes of every Japanese city in the CITY table. The COUNTRYCODE for Japan is JPN.

{% highlight sql %}
SELECT * FROM CITY WHERE COUNTRYCODE='JPN';
{% endhighlight %}

#### Japanese Cities' Names

> Query the names of all the Japanese cities in the CITY table. The COUNTRYCODE for Japan is JPN.

{% highlight sql %}
SELECT name FROM CITY WHERE COUNTRYCODE='JPN';
{% endhighlight %}
#### Weather Observation Station 1
> Query a list of CITY and STATE from the STATION table.

{% highlight sql %}
SELECT CITY, STATE FROM STATION;
{% endhighlight %}

#### Weather Observation Station 3
> Query a list of CITY names from STATION with even ID numbers only. You may print the results in any order, but must exclude duplicates from your answer.

{% highlight sql %}
SELECT DISTINCT CITY FROM STATION WHERE (ID % 2) = 0;
{% endhighlight %}

#### Weather Observation Station 4

>Let N be the number of CITY entries in STATION, and let N1 be the number of distinct CITY names in STATION; query the value of N - N1  from STATION. In other words, find the difference between the total number of CITY entries in the table and the number of distinct CITY entries in the table.

{% highlight sql %}
SELECT COUNT(CITY) - COUNT(DISTINCT CITY) FROM STATION
{% endhighlight %}

#### Weather Observation Station 5
> Query the two cities in STATION with the shortest and longest CITY names, as well as their respective lengths (i.e.: number of characters in the name). If there is more than one smallest or largest city, choose the one that comes first when ordered alphabetically.

{% highlight sql %}
SELECT CITY, LENGTH(CITY) FROM STATION ORDER BY LENGTH(CITY) DESC, CITY ASC LIMIT 1;
SELECT CITY, LENGTH(CITY) FROM STATION ORDER BY LENGTH(CITY) ASC, CITY ASC LIMIT 1;
{% endhighlight %}

#### Weather Observation Station 6
> Query the list of CITY names starting with vowels (i.e., a, e, i, o, or u) from STATION. Your result cannot contain duplicates.

{% highlight sql %}
SELECT CITY FROM STATION WHERE CITY REGEXP '^[aeiou].*';
{% endhighlight %}

#### Weather Observation Station 7
> Query the list of CITY names ending with vowels (a, e, i, o, u) from STATION. Your result cannot contain duplicates.

{% highlight sql %}
SELECT DISTINCT CITY FROM STATION WHERE CITY REGEXP '.*[aeiou]$';
{% endhighlight %}

#### Weather Observation Station 8
> Query the list of CITY names from STATION which have vowels (i.e., a, e, i, o, and u) as both their first and last characters. Your result cannot contain duplicates.

{% highlight sql %}
SELECT DISTINCT CITY FROM STATION WHERE CITY REGEXP '^[aeiou].*[aeiou]$';
{% endhighlight %}

#### Weather Observation Station 9
> Query the list of CITY names from STATION that do not start with vowels. Your result cannot contain duplicates.

{% highlight sql %}
SELECT DISTINCT CITY FROM STATION WHERE CITY REGEXP '^[^aeiou].*';
{% endhighlight %}

#### Weather Observation Station 10

>Query the list of CITY names from STATION that do not end with vowels. Your result cannot contain duplicates.

{% highlight sql %}
SELECT DISTINCT CITY FROM STATION WHERE CITY REGEXP '.*[^aeiou]$';
{% endhighlight %}

#### Weather Observation Station 11

> Query the list of CITY names from STATION that either do not start with vowels or do not end with vowels. Your result cannot contain duplicates.

{% highlight sql %}
SELECT DISTINCT CITY FROM STATION WHERE CITY REGEXP '(^[^aeiou].*)|(.*[^aeiou]$)';
{% endhighlight %}

#### Weather Observation Station 12
> Query the list of CITY names from STATION that do not start with vowels and do not end with vowels. Your result cannot contain duplicates.

{% highlight sql %}
SELECT DISTINCT CITY FROM STATION WHERE CITY REGEXP '^[^aeiou].*[^aeiou]$';
{% endhighlight %}

#### Higher Than 75 Marks
> Query the Name of any student in STUDENTS who scored higher than  Marks. Order your output by the last three characters of each name. If two or more students both have names ending in the same last three characters (i.e.: Bobby, Robby, etc.), secondary sort them by ascending ID.

{% highlight sql %}
SELECT NAME FROM STUDENTS WHERE MARKS > 75
ORDER BY SUBSTRING(NAME, -3, 3), ID ASC;
{% endhighlight %}

#### Employee Names
> Write a query that prints a list of employee names (i.e.: the name attribute) from the Employee table in alphabetical order.

{% highlight sql %}
SELECT NAME FROM Employee ORDER BY NAME;
{% endhighlight %}

#### Employee Salaries

>Write a query that prints a list of employee names (i.e.: the name attribute) for employees in Employee having a salary greater than 2000 per month who have been employees for less than 10  months. Sort your result by ascending employee_id.

{% highlight sql %}
SELECT NAME FROM Employee WHERE salary > 2000 AND months < 10 ORDER BY employee_id ASC;
{% endhighlight %}

## BASIC JOIN

#### Ollivander's Inventory
> Harry Potter and his friends are at Ollivander's with Ron, finally replacing Charlie's old broken wand. Hermione decides the best way to choose is by determining the minimum number of gold galleons needed to buy each non-evil wand of high power and age. Write a query to print the id, age, coins_needed, and power of the wands that Ron's interested in, sorted in order of descending power. If more than one wand has same power, sort the result in order of descending age.

{% highlight sql %}
SELECT w.id, p.age, w.coins_needed, w.power
FROM Wands as w JOIN Wands_Property as p on (w.code = p.code)
WHERE p.is_evil = 0 AND w.coins_needed = 
  (SELECT MIN(coins_needed)
  FROM Wands as w1 
    join Wands_Property as p1 ON (w1.code = p1.code)
  WHERE w1.power = w.power AND p1.age = p.age)
ORDER BY w.power DESC, p.age DESC
{% endhighlight %}

#### Challenges

> Julia asked her students to create some coding challenges. Write a query to print the hacker_id, name, and the total number of challenges created by each student. Sort your results by the total number of challenges in descending order. If more than one student created the same number of challenges, then sort the result by hacker_id. If more than one student created the same number of challenges and the count is less than the maximum number of challenges created, then exclude those students from the result.

{% highlight sql %}
SELECT h.hacker_id, 
       h.name, 
       COUNT(c.challenge_id) AS c_count
FROM Hackers h
JOIN Challenges c ON c.hacker_id = h.hacker_id
GROUP BY h.hacker_id, h.name
HAVING c_count = 
    (SELECT COUNT(c2.challenge_id) AS c_max
     FROM challenges as c2 
     GROUP BY c2.hacker_id 
     ORDER BY c_max DESC limit 1)
OR c_count IN 
    (SELECT DISTINCT c_compare AS c_unique
     FROM (SELECT h2.hacker_id, 
                  h2.name, 
                  COUNT(challenge_id) AS c_compare
           FROM Hackers h2
           JOIN Challenges c ON c.hacker_id = h2.hacker_id
           GROUP BY h2.hacker_id, h2.name) counts
     GROUP BY c_compare
     HAVING COUNT(c_compare) = 1)
ORDER BY c_count DESC, h.hacker_id;
{% endhighlight %}

#### Contest Leaderboard

>You did such a great job helping Julia with her last coding contest challenge that she wants you to work on this one, too! The total score of a hacker is the sum of their maximum scores for all of the challenges. Write a query to print the hacker_id, name, and total score of the hackers ordered by the descending score. If more than one hacker achieved the same total score, then sort the result by ascending hacker_id. Exclude all hackers with a total score of 0 from your result.

{% highlight sql %}
SELECT  A.HACKER_ID, A.NAME, SUM(SCORE)
FROM
  (SELECT H.HACKER_ID, H.NAME, CASE WHEN MAX(S.SCORE) > 0 THEN MAX(S.SCORE) ELSE NULL END SCORE FROM Hackers H
    JOIN Submissions S ON S.HACKER_ID = H.HACKER_ID
  GROUP BY H.HACKER_ID, H.NAME, S.CHALLENGE_ID) A
GROUP BY A.HACKER_ID, A.NAME
HAVING SUM(SCORE) IS NOT NULL
ORDER BY SUM(SCORE) DESC, A.HACKER_ID ASC
{% endhighlight %}

#### Asian Population

>Given the CITY and COUNTRY tables, query the sum of the populations of all cities where the CONTINENT is 'Asia'.

Note: CITY.CountryCode and COUNTRY.Code are matching key columns.

{% highlight sql %}
SELECT SUM(C.POPULATION) FROM CITY C
  JOIN COUNTRY CT ON C.COUNTRYCODE = CT.CODE
WHERE CONTINENT = 'Asia'
{% endhighlight %}

#### African Cities

> Given the CITY and COUNTRY tables, query the names of all cities where the CONTINENT is 'Africa'.

Note: CITY.CountryCode and COUNTRY.Code are matching key columns.

{% highlight sql %}
SELECT C.NAME
FROM CITY C
  JOIN COUNTRY CR ON CR.CODE = C.COUNTRYCODE
WHERE CONTINENT = 'Africa'
{% endhighlight %}

#### Average Population of Each Continent

> Given the CITY and COUNTRY tables, query the names of all the continents (COUNTRY.Continent) and their respective average city populations (CITY.Population) rounded down to the nearest integer.

Note: CITY.CountryCode and COUNTRY.Code are matching key columns.

{% highlight sql %}
SELECT CR.CONTINENT, FLOOR(AVG(C.POPULATION))
FROM CITY C
  JOIN COUNTRY CR ON CR.CODE = C.COUNTRYCODE
GROUP BY CR.CONTINENT
{% endhighlight %}

#### The Report

> You are given two tables: Students and Grades. Students contains three columns ID, Name and Marks. Ketty gives Eve a task to generate a report containing three columns: Name, Grade and Mark. Ketty doesn't want the NAMES of those students who received a grade lower than 8. The report must be in descending order by grade -- i.e. higher grades are entered first. If there is more than one student with the same grade (8-10) assigned to them, order those particular students by their name alphabetically. Finally, if the grade is lower than 8, use "NULL" as their name and list them by their grades in descending order. If there is more than one student with the same grade (1-7) assigned to them, order those particular students by their marks in ascending order.Write a query to help Eve.

{% highlight sql %}
SELECT CASE WHEN g.Grade >= 8
  THEN s.Name
  ELSE NULL
  END,
  g.Grade,
  s.Marks
FROM Students s
  JOIN Grades g ON s.Marks BETWEEN g.Min_Mark AND g.Max_Mark
ORDER BY g.Grade DESC, s.Name ASC
{% endhighlight %}

#### Top Competitors

> Julia just finished conducting a coding contest, and she needs your help assembling the leaderboard! Write a query to print the respective hacker_id and name of hackers who achieved full scores for more than one challenge. Order your output in descending order by the total number of challenges in which the hacker earned a full score. If more than one hacker received full scores in same number of challenges, then sort them by ascending hacker_id.

{% highlight sql %}
SELECT h.hacker_id, h.name
FROM Hackers h
  JOIN Submissions s ON s.hacker_id = h.hacker_id
  JOIN Challenges ch ON s.challenge_id = ch.challenge_id
  JOIN Difficulty d ON d.difficulty_level = ch.difficulty_level
WHERE s.score = d.score
GROUP BY h.hacker_id, h.name
HAVING COUNT(s.hacker_id) > 1
ORDER BY COUNT(distinct s.challenge_id) DESC, h.hacker_id ASC
{% endhighlight %}

## Advanced select

#### Type of Triangle

>Write a query identifying the type of each record in the TRIANGLES table using its three side lengths. Output one of the following statements for each record in the table:

Equilateral: It's a triangle with  sides of equal length.
Isosceles: It's a triangle with  sides of equal length.
Scalene: It's a triangle with  sides of differing lengths.
Not A Triangle: The given values of A, B, and C don't form a triangle.

{% highlight sql %}
SELECT 
  CASE
    WHEN NOT( A + B > C AND A + C > B AND B + C > A) THEN 'Not A Triangle'
    WHEN A = B AND A = C AND B = C THEN 'Equilateral'
    WHEN A = B OR A = C OR B = C THEN 'Isosceles'
    ELSE 'Scalene'
  END
FROM TRIANGLES
{% endhighlight %}

#### The PADS

> Generate the following two result sets:
Query an alphabetically ordered list of all names in OCCUPATIONS, immediately followed by the first letter of each profession as a parenthetical (i.e.: enclosed in parentheses). For example: AnActorName(A), ADoctorName(D), AProfessorName(P), and ASingerName(S).
Query the number of ocurrences of each occupation in OCCUPATIONS. Sort the occurrences in ascending order, and output them in the following format: There are a total of [occupation_count] [occupation]s.
where [occupation_count] is the number of occurrences of an occupation in OCCUPATIONS and [occupation] is the lowercase occupation name. If more than one Occupation has the same [occupation_count], they should be ordered alphabetically.

Note: There will be at least two entries in the table for each type of occupation.

{% highlight sql %}
SELECT CONCAT(Name, '(', SUBSTRING(Occupation, 1, 1), ')')
FROM OCCUPATIONS
ORDER BY Name;

SELECT CONCAT('There are a total of ', COUNT(Occupation), ' ', LOWER(Occupation), 's.')
FROM OCCUPATIONS
GROUP BY Occupation
ORDER BY COUNT(Occupation) ASC, Occupation ASC;
{% endhighlight %}

#### Occupations

> Pivot the Occupation column in OCCUPATIONS so that each Name is sorted alphabetically and displayed underneath its corresponding Occupation. The output column headers should be Doctor, Professor, Singer, and Actor, respectively.

Note: Print NULL when there are no more names corresponding to an occupation.

{% highlight sql %}
SET @doc_number = 0, @prof_number = 0, @sign_number = 0, @act_number = 0;
SELECT MIN(doctors), MIN(professors), MIN(signers), MIN(actors)
FROM (SELECT 
  (CASE WHEN occupation = 'Doctor' THEN (@doc_number:=@doc_number + 1)
        WHEN occupation = 'Professor' THEN (@prof_number:=@prof_number + 1)
        WHEN occupation = 'Singer' THEN (@sign_number:=@sign_number + 1)
        WHEN occupation = 'Actor' THEN (@act_number:=@act_number + 1)
  END) AS rank,
  CASE WHEN occupation = 'Doctor' THEN name END AS doctors,
  CASE WHEN occupation = 'Professor' THEN name END AS professors,
  CASE WHEN occupation = 'Singer' THEN name END AS signers,
  CASE WHEN occupation = 'Actor' THEN name END AS actors 
FROM OCCUPATIONS
ORDER BY name) ranked
GROUP BY rank
{% endhighlight %}

#### Binary Tree Nodes

>You are given a table, BST, containing two columns: N and P, where N represents the value of a node in Binary Tree, and P is the parent of N. Write a query to find the node type of Binary Tree ordered by the value of the node. Output one of the following for each node:

>Root: If node is root node.
Leaf: If node is leaf node.
Inner: If node is neither root nor leaf node.

{% highlight sql %}
SELECT N,
  CASE
    WHEN P IS NULL THEN 'Root'
    WHEN NOT EXISTS (SELECT * FROM BST WHERE P = z.N) THEN 'Leaf'
    ELSE 'Inner'
END
FROM BST z
ORDER BY N
{% endhighlight %}

#### New Companies

>Amber's conglomerate corporation just acquired some new companies. Each of the companies follows this hierarchy: Founder -> Lead Manager -> Senior Manager -> Manager -> Employee
Given the table schemas below, write a query to print the company_code, founder name, total number of lead managers, total number of senior managers, total number of managers, and total number of employees. Order your output by ascending company_code.

Note:

The tables may contain duplicate records.
The company_code is string, so the sorting should not be numeric. For example, if the company_codes are C_1, C_2, and C_10, then the ascending company_codes will be C_1, C_10, and C_2.

{% highlight sql %}
SELECT 
  company_code, 
  founder,
  (SELECT COUNT(DISTINCT lead_manager_code ) FROM Lead_Manager LM WHERE LM.company_code = C.company_code ),
  (SELECT COUNT(DISTINCT senior_manager_code ) FROM Senior_Manager sm WHERE sm.company_code = C.company_code ),
  (SELECT COUNT(DISTINCT manager_code) FROM Manager m  WHERE m.company_code = C.company_code ),
  (SELECT COUNT(DISTINCT employee_code) FROM Employee e  WHERE e.company_code = C.company_code )
  
FROM COMPANY C
ORDER BY company_code
{% endhighlight %}

## Advanced join

#### Projects

> You are given a table, Projects, containing three columns: Task_ID, Start_Date and End_Date. It is guaranteed that the difference between the End_Date and the Start_Date is equal to 1 day for each row in the table. If the End_Date of the tasks are consecutive, then they are part of the same project. Samantha is interested in finding the total number of different projects completed.Write a query to output the start and end dates of projects listed by the number of days it took to complete the project in ascending order. If there is more than one project that have the same number of completion days, then order by the start date of the project.

{% highlight sql %}
SELECT a.Start_Date, min(b.End_Date) FROM ( 
  (SELECT Start_Date FROM Projects p WHERE NOT EXISTS ( SELECT * FROM Projects WHERE End_Date = p.Start_Date)) a, 
  (SELECT End_Date FROM Projects p WHERE NOT EXISTS ( SELECT * FROM Projects WHERE Start_Date = p.End_Date)) b)
WHERE a.Start_Date < b.End_Date
GROUP BY a.Start_Date
ORDER BY DATEDIFF(MIN(b.End_Date), a.Start_Date) ASC, a.Start_Date ASC;
{% endhighlight %}

#### Placements
> You are given three tables: Students, Friends and Packages. Students contains two columns: ID and Name. Friends contains two columns: ID and Friend_ID (ID of the ONLY best friend). Packages contains two columns: ID and Salary (offered salary in $ thousands per month).Write a query to output the names of those students whose best friends got offered a higher salary than them. Names must be ordered by the salary amount offered to the best friends. It is guaranteed that no two students got same salary offer.

{% highlight sql %}
SELECT s.name FROM Students s
  JOIN Friends f ON s.id = f.id
  JOIN Students sf ON sf.id = f.Friend_ID
  JOIN Packages ps ON s.id = ps.id
  JOIN Packages pf ON sf.id = pf.id
WHERE pf.Salary > ps.Salary
ORDER BY pf.Salary
{% endhighlight %}

#### Symmetric Pairs

> You are given a table, Functions, containing two columns: X and Y. Two pairs (X1, Y1) and (X2, Y2) are said to be symmetric pairs if X1 = Y2 and X2 = Y1.Write a query to output all such symmetric pairs in ascending order by the value of X.

{% highlight sql %}
SELECT f1.X, f1.Y FROM Functions f1
  JOIN Functions f2 ON f1.X=f2.Y AND f1.Y=f2.X
GROUP BY f1.X, f1.Y
HAVING COUNT(f1.X)>1 or f1.X<f1.Y
ORDER BY f1.X 
{% endhighlight %}

#### Interviews

> Samantha interviews many candidates from different colleges using coding challenges and contests. Write a query to print the contest_id, hacker_id, name, and the sums of total_submissions, total_accepted_submissions, total_views, and total_unique_views for each contest sorted by contest_id. Exclude the contest from the result if all four sums are 0.

Note: A specific contest can be used to screen candidates at more than one college, but each college only holds 1 screening contest.

{% highlight sql %}
SELECT contests.*, col.s, col.asb, col.v, col.uv FROM
contests
JOIN
(SELECT colleges.contest_id, sum(subs) s, sum(asabs) asb, sum(views) v, sum(uviews) uv
FROM colleges
JOIN
(SELECT college_id, sum(ss.ts) subs, sum(ss.tas) asabs, sum(vs.tv) views, sum(vs.tuv) uviews
FROM challenges
LEFT JOIN 
  (SELECT challenge_id chid, sum(total_submissions) ts, sum(total_accepted_submissions) tas
  FROM submission_stats GROUP BY challenge_id) ss
    ON ss.chid = challenges.challenge_id
LEFT JOIN 
  (SELECT challenge_id chid, sum(total_views) tv, sum(total_unique_views) tuv
  FROM view_stats GROUP BY challenge_id) vs
    ON vs.chid = challenges.challenge_id
GROUP BY college_id) cr
ON colleges.college_id = cr.college_id
GROUP BY colleges.contest_id
HAVING s>0 AND asb > 0 AND v > 0 AND uv > 0) col
  ON contests.contest_id = col.contest_id
ORDER BY contests.contest_id
{% endhighlight %}

#### 15 Days of Learning SQL

>Julia conducted a 15 days of learning SQL contest. The start date of the contest was March 01, 2016 and the end date was March 15, 2016.Write a query to print total number of unique hackers who made at least 1 submission each day (starting on the first day of the contest), and find the hacker_id and name of the hacker who made maximum number of submissions each day. If more than one such hacker has a maximum number of submissions, print the lowest hacker_id. The query should print this information for each day of the contest, sorted by the date.

{% highlight sql %}
SET @day_rank := 0;
SELECT v.sd, v.uc, b.mhid, h.name  FROM
(SELECT c.sd, COUNT(DISTINCT b.hid) uc FROM (
    SELECT a.sd, @day_rank := @day_rank + 1 rank
    FROM (
        SELECT s.submission_date sd
        FROM submissions s
     GROUP BY s.submission_date
    ) a
) c

JOIN (
    SELECT s2.submission_date sd, s2.hacker_id hid, COUNT(DISTINCT s3.submission_date) cnt
    FROM submissions s2
    JOIN submissions s3
        ON s2.hacker_id = s3.hacker_id AND s2.submission_date >= s3.submission_date
    GROUP BY s2.submission_date, s2.hacker_id
) b
    ON b.sd = c.sd AND b.cnt = c.rank
GROUP BY c.sd) v

JOIN (
SELECT submission_date, MIN(hacker_id) mhid
FROM (
SELECT submission_date, hacker_id, COUNT(submission_id) cnt
FROM submissions s
GROUP BY submission_date, hacker_id
HAVING cnt = (
    SELECT MAX(cnt) FROM (
    SELECT submission_date, hacker_id, COUNT(submission_id) cnt
    FROM submissions
    GROUP BY submission_date, hacker_id
) d
GROUP BY submission_date
HAVING submission_date = s.submission_date
) )x
GROUP BY submission_date
) b

ON b.submission_date = v.sd
JOIN hackers h ON b.mhid = h.hacker_id
ORDER BY v.sd
{% endhighlight %}

## Aggregation

#### Revising Aggregations - The Count Function

> Query a count of the number of cities in CITY having a Population larger than 100 000.

{% highlight sql %}
SELECT COUNT(*) FROM CITY
WHERE POPULATION > 100000
{% endhighlight %}

#### Revising Aggregations - The Sum Function

> Query the total population of all cities in CITY where District is California.

{% highlight sql %}
SELECT SUM(POPULATION) FROM CITY WHERE DISTRICT = 'California'
{% endhighlight %}

#### Revising Aggregations - Averages

> Query the average population of all cities in CITY where District is California.

{% highlight sql %}
SELECT AVG(POPULATION)
FROM CITY
WHERE DISTRICT = 'California'
{% endhighlight %}

#### Average Population

> Query the average population for all cities in CITY, rounded down to the nearest integer.

{% highlight sql %}
SELECT ROUND(AVG(POPULATION)) FROM CITY
{% endhighlight %}

#### Japan Population

> Query the sum of the populations for all Japanese cities in CITY. The COUNTRYCODE for Japan is JPN.

{% highlight sql %}
SELECT SUM(POPULATION)
FROM CITY
WHERE COUNTRYCODE = 'JPN'
{% endhighlight %}

#### Population Density Difference

> Query the difference between the maximum and minimum populations in CITY.

{% highlight sql %}
SELECT MAX(POPULATION) - MIN(POPULATION) FROM CITY
{% endhighlight %}

#### The Blunder
> Samantha was tasked with calculating the average monthly salaries for all employees in the EMPLOYEES table, but did not realize her keyboard's  key was broken until after completing the calculation. She wants your help finding the difference between her miscalculation (using salaries with any zeroes removed), and the actual average salary.Write a query calculating the amount of error (i.e.:  average monthly salaries), and round it up to the next integer.

{% highlight sql %}
SELECT CEIL(AVG(Salary) - AVG(CAST(REPLACE(Salary, '0', '') AS UNSIGNED))) FROM EMPLOYEES
{% endhighlight %}

#### Top Earners

> We define an employee's total earnings to be their monthly  worked, and the maximum total earnings to be the maximum total earnings for any employee in the Employee table. Write a query to find the maximum total earnings for all employees as well as the total number of employees who have maximum total earnings. Then print these values as  space-separated integers.

{% highlight sql %}
SELECT (SALARY * MONTHS) INC, COUNT(EMPLOYEE_ID)
FROM Employee E
GROUP BY INC
ORDER BY INC DESC
LIMIT 1
{% endhighlight %}

#### Weather Observation Station 2

>Query the following two values from the STATION table:

>The sum of all values in LAT_N rounded to a scale of  decimal places.
The sum of all values in LONG_W rounded to a scale of  decimal places.

{% highlight sql %}
SELECT ROUND(SUM(LAT_N), 2), ROUND(SUM(LONG_W), 2) FROM STATION
{% endhighlight %}

#### Weather Observation Station 13
> Query the sum of Northern Latitudes (LAT_N) from STATION having values greater than 38.7880 and less than 137.2345 . Truncate your answer to 4 decimal places.

{% highlight sql %}
SELECT TRUNCATE(SUM(LAT_N), 4) FROM STATION
WHERE LAT_N BETWEEN 38.7880 AND 137.2345
{% endhighlight %}

#### Weather Observation Station 14
> Query the greatest value of the Northern Latitudes (LAT_N) from STATION that is less than 137.2345 . Truncate your answer to 4 decimal places.

{% highlight sql %}
SELECT TRUNCATE(LAT_N, 4) FROM STATION
WHERE LAT_N < 137.2345
ORDER BY LAT_N DESC
LIMIT 1
{% endhighlight %}

#### Weather Observation Station 15

> Query the Western Longitude (LONG_W) for the largest Northern Latitude (LAT_N) in STATION that is less than 137.2345. Round your answer to 4 decimal places.

{% highlight sql %}
SELECT ROUND(LONG_W, 4) FROM STATION
WHERE LAT_N < 137.2345
ORDER BY LAT_N DESC
LIMIT 1
{% endhighlight %}

#### Weather Observation Station 16

>Query the smallest Northern Latitude (LAT_N) from STATION that is greater than 38.7780. Round your answer to  decimal 4 places.

{% highlight sql %}
SELECT ROUND(LAT_N, 4) FROM STATION
WHERE LAT_N > 38.7780
ORDER BY LAT_N ASC
LIMIT 1
{% endhighlight %}

#### Weather Observation Station 17
>Query the Western Longitude (LONG_W)where the smallest Northern Latitude (LAT_N) in STATION is greater than 38.7780. Round your answer to 4 decimal places.

{% highlight sql %}
SELECT ROUND(LONG_W, 4) FROM STATION
WHERE LAT_N > 38.7780
ORDER BY LAT_N ASC
LIMIT 1
{% endhighlight %}

#### Weather Observation Station 18

>Consider P1(A,B) and P2(C,D) to be two points on a 2D plane.

>A happens to equal the minimum value in Northern Latitude (LAT_N in STATION).
B happens to equal the minimum value in Western Longitude (LONG_W in STATION).
C happens to equal the maximum value in Northern Latitude (LAT_N in STATION).
D happens to equal the maximum value in Western Longitude (LONG_W in STATION).
Query the Manhattan Distance between points  and  and round it to a scale of  decimal places.

{% highlight sql %}
SELECT ROUND(ABS(MIN(LONG_W ) - MAX(LONG_W)) + ABS(MIN(LAT_N ) - MAX(LAT_N)), 4) FROM STATION
{% endhighlight %}

#### Weather Observation Station 19

> Consider P1(A,C) and P2(B,D) to be two points on a 2D plane where (A,B) are the respective minimum and maximum values of Northern Latitude (LAT_N) and (C,D) are the respective minimum and maximum values of Western Longitude (LONG_W) in STATION.Query the Euclidean Distance between points  and  and format your answer to display 4 decimal digits.

{% highlight sql %}
SELECT TRUNCATE(SQRT(POWER(MIN(LAT_N) - MAX(LAT_N), 2) + POWER(MIN(LONG_W) - MAX(LONG_W), 2)), 4) FROM STATION
{% endhighlight %}

#### Weather Observation Station 20

> A median is defined as a number separating the higher half of a data set from the lower half. Query the median of the Northern Latitudes (LAT_N) from STATION and round your answer to 4 decimal places.

{% highlight sql %}
SET @rownum = 0;
SELECT ROUND(l, 4) 
FROM
( SELECT @rownum := @rownum + 1 AS rank, LAT_N AS l FROM STATION ORDER BY LAT_N ) a
WHERE rank = CEIL((SELECT COUNT(*) FROM STATION)/2)
{% endhighlight %}

## Alternative queries

#### Draw The Triangle 1

> P(R) represents a pattern drawn by Julia in R rows. The following pattern represents P(5):
    *****
    ****
    ***
    **
    *
    Write a query to print the pattern P(20).

{% highlight sql %}
DELIMITER //
Begin 
Declare i INT Default 0; 
      While i < 10 DO 
            set i = i +1; 
      End while; 
End//
{% endhighlight %}

#### Draw The Triangle 2

> P(R) represents a pattern drawn by Julia in R rows. The following pattern represents P(5):
    *
    **
    ***
    ****
    *****
    Write a query to print the pattern P(20).
    
{% highlight sql %}
DECLARE @i INT = 1
WHILE (@i <= 20) 
BEGIN
   PRINT REPLICATE('* ', @i) 
   SET @i = @i + 1
END
{% endhighlight %}

#### Print prime numbers

> Write a query to print all prime numbers less than or equal to 1000. Print your result on a single line, and use the ampersand (&) character as your separator (instead of a space).
For example, the output for all prime numbers  <= 10 would be: 2&3&5&7

{% highlight sql %}
SET group_concat_max_len = 2048;
SELECT GROUP_CONCAT(nmb SEPARATOR '&') FROM
(
    SELECT @count3 := @count3 + 1 nmb
    FROM information_schema.tables t1, information_schema.tables t2 ,( SELECT @count3 := 1) f
    WHERE @count3 < 1000
) g
WHERE nmb NOT IN (
    SELECT DISTINCT c.n 
    FROM
        (
            SELECT @count := @count + 1 n
            FROM information_schema.tables t1, information_schema.tables t2,( SELECT @count := 1) a
            WHERE @count < 1000
        ) c,
        (
            SELECT @count2 := @count2 + 1 n
            FROM information_schema.tables t1, information_schema.tables t2,( SELECT @count2 := 1) b
            WHERE @count2 < 1000
        ) d
    WHERE c.n > d.n AND c.n % d.n = 0
    )
{% endhighlight %}


