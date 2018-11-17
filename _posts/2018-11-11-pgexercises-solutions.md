---
layout: post
title:  "Решения"
subject: "Упражнения pgexercises"
date:   2018-11-11 20:55:21 +0300
---

## Simple SQL Queries 

#### Retrieve everything from a table

**Question:**
> How can you retrieve all the information from the cd.facilities table?

{% highlight sql %}
SELECT * FROM cd.facilities
{% endhighlight %}

#### Retrieve specific columns from a table

**Question:**
> You want to print out a list of all of the facilities and their cost to members. How would you retrieve a list of only facility names and costs?

{% highlight sql %}
SELECT name, membercost from cd.facilities
{% endhighlight %}

#### Control which rows are retrieved

**Question:**
> How can you produce a list of facilities that charge a fee to members?

{% highlight sql %}
SELECT * FROM cd.facilities
WHERE membercost > 0
{% endhighlight %}


#### Basic string searches

**Question:**
> How can you produce a list of all facilities with the word 'Tennis' in their name?

{% highlight sql %}
SELECT * FROM cd.facilities WHERE name LIKE '%Tennis%'
{% endhighlight %}

#### Matching against multiple possible values

**Question:**
> How can you retrieve the details of facilities with ID 1 and 5? Try to do it without using the OR operator.

{% highlight sql %}
SELECT * FROM cd.facilities WHERE facid IN (1, 5)
{% endhighlight %}

#### Classify results into buckets

**Question:**
> How can you produce a list of facilities, with each labelled as 'cheap' or 'expensive' depending on if their monthly maintenance cost is more than $100? Return the name and monthly maintenance of the facilities in question.

{% highlight sql %}
SELECT name,
CASE
  WHEN monthlymaintenance > 100 THEN 'expensive'
  ELSE 'cheap'
END
FROM cd.facilities
{% endhighlight %}

#### Working with dates

**Question:**
> How can you produce a list of members who joined after the start of September 2012? Return the memid, surname, **firstname, and joindate of the members in question.:**

{% highlight sql %}
SELECT memid, surname, firstname, joindate
FROM cd.members
WHERE joindate > '2012-09-01'
{% endhighlight %}


#### Removing duplicates, and ordering results

**Question:**
> How can you produce an ordered list of the first 10 surnames in the members table? The list must not contain duplicates.

{% highlight sql %}
SELECT DISTINCT surname
FROM cd.members
ORDER BY surname
LIMIT 10
{% endhighlight %}

#### Combining results from multiple queries

**Question:**
> You, for some reason, want a combined list of all surnames and all facility names. Yes, this is a contrived example :-). Produce that list!

{% highlight sql %}
SELECT surname FROM cd.members
UNION
SELECT name FROM cd.facilities
{% endhighlight %}

#### Simple aggregation

**Question:**
> You'd like to get the signup date of your last member. How can you retrieve this information?

{% highlight sql %}
SELECT MAX(joindate) FROM cd.members
{% endhighlight %}

#### More aggregation

**Question:**
> You'd like to get the first and last name of the last member(s) who signed up - not just the date. How can you do that?

{% highlight sql %}
SELECT firstname, surname, joindate FROM cd.members
WHERE joindate = (SELECT MAX(joindate) FROM cd.members)
{% endhighlight %}


## Joins and Subqueries 

#### Retrieve the start times of members' bookings

**Question:**
> How can you produce a list of the start times for bookings by members named 'David Farrell'?

{% highlight sql %}
SELECT starttime FROM cd.bookings b
  JOIN cd.members m ON b.memid = m.memid
WHERE firstname = 'David' AND surname =  'Farrell'
{% endhighlight %}

#### Work out the start times of bookings for tennis courts

**Question:**
> How can you produce a list of the start times for bookings for tennis courts, for the date '2012-09-21'? Return a list of start time and facility name pairings, ordered by the time.

{% highlight sql %}
SELECT starttime, name
FROM cd.bookings b
  JOIN cd.facilities f ON f.facid = b.facid
WHERE starttime::date = '2012-09-21' AND name LIKE 'Tennis Court%'
ORDER BY starttime
{% endhighlight %}

#### Produce a list of all members who have recommended another member

**Question:**
> How can you output a list of all members who have recommended another member? Ensure that there are no duplicates in the list, and that results are ordered by (surname, firstname).

{% highlight sql %}
SELECT DISTINCT recommender.firstname, recommender.surname
FROM cd.members member
  JOIN cd.members recommender ON member.recommendedby = recommender.memid 
ORDER BY recommender.surname, recommender.firstname
{% endhighlight %}

#### Produce a list of all members, along with their recommender

**Question:**
> How can you output a list of all members, including the individual who recommended them (if any)? Ensure that results are ordered by (surname, firstname).

{% highlight sql %}
SELECT member.firstname, 
  member.surname,
  recommender.firstname,
  recommender.surname
FROM cd.members member
  LEFT JOIN cd.members recommender
    ON member.recommendedby = recommender.memid 
ORDER BY member.surname, member.firstname
{% endhighlight %}

#### Produce a list of all members who have used a tennis court

**Question:**
> How can you produce a list of all members who have used a tennis court? Include in your output the name of the court, and the name of the member formatted as a single column. Ensure no duplicate data, and order by the member name.

{% highlight sql %}
SELECT DISTINCT firstname || ' ' || surname mname, name
FROM cd.members m
  JOIN cd.bookings b ON m.memid = b.memid
  JOIN cd.facilities f ON b.facid = f.facid
WHERE name LIKE 'Tennis Court%'
ORDER BY mname
{% endhighlight %}

#### Produce a list of costly bookings


**Question:**
> How can you produce a list of bookings on the day of 2012-09-14 which will cost the member (or guest) more than $30? Remember that guests have different costs to members (the listed costs are per half-hour 'slot'), and the guest user is always ID 0. Include in your output the name of the facility, the name of the member formatted as a single column, and the cost. Order by descending cost, and do not use any subqueries.

{% highlight sql %}
SELECT * FROM (SELECT firstname || ' ' || surname,
  name,
  CASE
    WHEN m.memid = 0 THEN guestcost * slots
  ELSE membercost * slots
  END AS cost
FROM cd.members m
  JOIN cd.bookings b ON m.memid = b.memid
  JOIN cd.facilities f ON f.facid = b.facid
WHERE starttime::date = '2012-09-14') a
  WHERE cost > 30
ORDER BY cost DESC
{% endhighlight %}

#### Produce a list of all members, along with their recommender, using no joins.

**Question:**
> How can you output a list of all members, including the individual who recommended them (if any), without using any joins? Ensure that there are no duplicates in the list, and that each firstname + surname pairing is formatted as a column and ordered.

{% highlight sql %}
SELECT *
FROM (SELECT DISTINCT
  firstname || ' ' || surname member,
  (
    SELECT firstname || ' ' || surname
    FROM cd.members mm
    WHERE m.recommendedby = mm.memid
  ) recommender
FROM cd.members m) a
ORDER BY member
{% endhighlight %}

#### Produce a list of costly bookings, using a subquery

**Question:**
> The Produce a list of costly bookings exercise contained some messy logic: we had to calculate the booking cost in both the WHERE clause and the CASE statement. Try to simplify this calculation using subqueries. For reference, thequestion was:
> How can you produce a list of bookings on the day of 2012-09-14 which will cost the member (or guest) more than $30? Remember that guests have different costs to members (the listed costs are per half-hour 'slot'), and the guest user is always ID 0. Include in your output the name of the facility, the name of the member formatted as a single column, and the cost. Order by descending cost.

{% highlight sql %}
SELECT * FROM (SELECT firstname || ' ' || surname,
  name,
  CASE
    WHEN m.memid = 0 THEN guestcost * slots
  ELSE membercost * slots
  END AS cost
FROM cd.members m
  JOIN cd.bookings b ON m.memid = b.memid
  JOIN cd.facilities f ON f.facid = b.facid
WHERE starttime::date = '2012-09-14') a
  WHERE cost > 30
ORDER BY cost DESC
{% endhighlight %}


## Modifying data 

#### Insert some data into a table
**Question:**
>The club is adding a new facility - a spa. We need to add it into the facilities table. Use the following values: facid: 9, Name: 'Spa', membercost: 20, guestcost: 30, initialoutlay: 100000, monthlymaintenance: 800.

{% highlight sql %}
INSERT INTO cd.facilities
VALUES (9, 'Spa', 20, 30, 100000, 800)
{% endhighlight %}

#### Insert multiple rows of data into a table

**Question:**
>In the previous exercise, you learned how to add a facility. Now you're going to add multiple facilities in one command. Use the following values: facid: 9, Name: 'Spa', membercost: 20, guestcost: 30, initialoutlay: 100000, monthlymaintenance: 800.
facid: 10, Name: 'Squash Court 2', membercost: 3.5, guestcost: 17.5, initialoutlay: 5000, monthlymaintenance: 80.

{% highlight sql %}
INSERT INTO cd.facilities
VALUES ( 9, 'Spa', 20, 30, 100000, 800),
  (10,'Squash Court 2', 3.5, 17.5, 5000, 80)
{% endhighlight %}
 
#### Insert calculated data into a table

**Question:**
> Let's try adding the spa to the facilities table again. This time, though, we want to automatically generate the value for the next facid, rather than specifying it as a constant. Use the following values for everything else: Name: 'Spa', membercost: 20, guestcost: 30, initialoutlay: 100000, monthlymaintenance: 800.

{% highlight sql %}
INSERT INTO cd.facilities
SELECT MAX(facid) + 1, 'Spa', 20, 30, 100000, 800
  FROM cd.facilities
{% endhighlight %}
  
#### Update some existing data

**Question:**
> We made a mistake when entering the data for the second tennis court. The initial outlay was 10000 rather than 8000: you need to alter the data to fix the error.

{% highlight sql %}
UPDATE cd.facilities f
SET initialoutlay = 10000
WHERE f.name = 'Tennis Court 2'
{% endhighlight %}

#### Update multiple rows and columns at the same time

**Question:**
> We want to increase the price of the tennis courts for both members and guests. Update the costs to be 6 for members, and 30 for guests.

{% highlight sql %}
UPDATE cd.facilities
SET membercost = 6, guestcost = 30
WHERE name LIKE 'Tennis Court%'
{% endhighlight %}

#### Update a row based on the contents of another row

**Question:**
> We want to alter the price of the second tennis court so that it costs 10% more than the first one. Try to do this without using constant values for the prices, so that we can reuse the statement if we want to.

{% highlight sql %}
UPDATE cd.facilities
SET membercost = ( SELECT membercost FROM cd.facilities WHERE facid = 0) * 1.1,
  guestcost = ( SELECT guestcost FROM cd.facilities WHERE facid = 0) * 1.1
WHERE facid = 1
{% endhighlight %}

#### Delete all bookings

**Question:**
> As part of a clearout of our database, we want to delete all bookings from the cd.bookings table. How can we accomplish this?

{% highlight sql %}
DELETE FROM cd.bookings
{% endhighlight %}

#### Delete a member from the cd.members table

**Question:**
> We want to remove member 37, who has never made a booking, from our database. How can we achieve that?

{% highlight sql %}
DELETE FROM cd.members
WHERE memid = 37
{% endhighlight %}

#### Delete based on a subquery

**Question:**
> In our previous exercises, we deleted a specific member who had never made a booking. How can we make that more general, to delete all members who have never made a booking?

{% highlight sql %}
DELETE FROM cd.members m
WHERE NOT EXISTS (SELECT * FROM cd.bookings WHERE memid = m.memid)
{% endhighlight %}

## Aggregation 

#### Count the number of facilities

**Question:**
> For our first foray into aggregates, we're going to stick to something simple. We want to know how many facilities exist - simply produce a total count.

{% highlight sql %}
SELECT COUNT(*) FROM cd.facilities
{% endhighlight %}

#### Count the number of expensive facilities

**Question:**
> Produce a count of the number of facilities that have a cost to guests of 10 or more.

{% highlight sql %}
SELECT COUNT(*)
FROM cd.facilities
WHERE guestcost >= 10
{% endhighlight %}

#### Count the number of recommendations each member makes.

**Question:**
> Produce a count of the number of recommendations each member has made. Order by member ID.

{% highlight sql %}
SELECT recommendedby, COUNT(recommendedby)
FROM cd.members
GROUP BY recommendedby
HAVING COUNT(recommendedby) > 0
ORDER BY recommendedby
{% endhighlight %}

#### List the total slots booked per facility

**Question:**
> Produce a list of the total number of slots booked per facility. For now, just produce an output table consisting of facility id and slots, sorted by facility id.

{% highlight sql %}
SELECT facid, SUM(slots)
FROM cd.bookings
GROUP BY facid
ORDER BY facid
{% endhighlight %}

#### List the total slots booked per facility in a given month

**Question:**
> Produce a list of the total number of slots booked per facility in the month of September 2012. Produce an output table consisting of facility id and slots, sorted by the number of slots.

{% highlight sql %}
SELECT facid, SUM(slots)
FROM cd.bookings
WHERE starttime >= '2012-9-01' AND starttime < '2012-10-01'
GROUP BY facid
ORDER BY SUM(slots)
{% endhighlight %}

#### List the total slots booked per facility per month

**Question:**
> Produce a list of the total number of slots booked per facility per month in the year of 2012. Produce an output table consisting of facility id and slots, sorted by the id and month.

{% highlight sql %}
SELECT facid, DATE_PART('month', starttime), SUM(slots)
FROM cd.bookings
WHERE  DATE_PART('year', starttime) = '2012'
GROUP BY facid, DATE_PART('month', starttime)
ORDER BY facid, DATE_PART('month', starttime)
{% endhighlight %}

#### Find the count of members who have made at least one booking

**Question:**
> Find the total number of members who have made at least one booking.

{% highlight sql %}
SELECT COUNT(DISTINCT memid) FROM cd.bookings
{% endhighlight %}

#### List facilities with more than 1000 slots booked

**Question:**
> Produce a list of facilities with more than 1000 slots booked. Produce an output table consisting of facility id and hours, sorted by facility id.

{% highlight sql %}
SELECT facid, SUM(slots)
FROM cd.bookings
GROUP BY facid
HAVING SUM(slots) > 1000
ORDER BY facid
{% endhighlight %}

#### Find the total revenue of each facility

**Question:**
> Produce a list of facilities along with their total revenue. The output table should consist of facility name and revenue, sorted by revenue. Remember that there's a different cost for guests and members!

{% highlight sql %}
SELECT name, SUM(cost)
FROM (SELECT  name,
  CASE
    WHEN memid = 0 THEN slots * guestcost
  ELSE slots * membercost
  END as cost
FROM cd.facilities f
  JOIN cd.bookings b ON b.facid = f.facid ) a
GROUP BY NAME
ORDER BY  SUM(cost)
{% endhighlight %}

#### Find facilities with a total revenue less than 1000

**Question:**
> Produce a list of facilities with a total revenue less than 1000. Produce an output table consisting of facility name and revenue, sorted by revenue. Remember that there's a different cost for guests and members!

{% highlight sql %}
SELECT name, SUM(cost)
FROM (SELECT  name,
  CASE
    WHEN memid = 0 THEN slots * guestcost
  ELSE slots * membercost
  END as cost
FROM cd.facilities f
  JOIN cd.bookings b ON b.facid = f.facid ) a
GROUP BY NAME
HAVING SUM(cost) < 1000
ORDER BY  SUM(cost)
{% endhighlight %}

#### Output the facility id that has the highest number of slots booked

**Question:**
> Output the facility id that has the highest number of slots booked. For bonus points, try a version without a LIMIT clause. This version will probably look messy!

{% highlight sql %}
WITH sslots AS ( 
  SELECT DISTINCT facid, SUM(slots) OVER (PARTITION BY facid) sm
  FROM cd.bookings)
SELECT * FROM sslots
WHERE sm = (SELECT MAX(sm) FROM sslots)
{% endhighlight %}

#### List the total slots booked per facility per month, part 2

**Question:**
> Produce a list of the total number of slots booked per facility per month in the year of 2012. In this version, include output rows containing totals for all months per facility, and a total for all months for all facilities. The output table should consist of facility id, month and slots, sorted by the id and month. When calculating the aggregated values for all months and all facids, return null values in the month and facid columns.

{% highlight sql %}
SELECT facid, DATE_PART('month', starttime) as month, SUM(slots)
FROM cd.bookings
WHERE DATE_PART('year', starttime) = '2012' AND slots IS NOT NULL
GROUP BY ROLLUP (facid, month)
ORDER BY facid
{% endhighlight %}

#### List the total hours booked per named facility

**Question:**
> Produce a list of the total number of hours booked per facility, remembering that a slot lasts half an hour. The output table should consist of the facility id, name, and hours booked, sorted by facility id. Try formatting the hours to two decimal places.

{% highlight sql %}
SELECT b.facid, f.name, trunc(SUM(slots) / 2.0, 2)
FROM cd.bookings b
  JOIN cd.facilities f ON f.facid = b.facid
GROUP BY b.facid, f.name
ORDER BY b.facid
{% endhighlight %}

#### List each member's first booking after September 1st 2012


**Question:**
> Produce a list of each member name, id, and their first booking after September 1st 2012. Order by member ID.

{% highlight sql %}
SELECT DISTINCT surname, firstname, m.memid, MIN(starttime)
FROM cd.members m
  JOIN cd.bookings b ON m.memid = b.memid
WHERE starttime > '2012-09-01'
GROUP BY surname, firstname, m.memid
ORDER BY memid
{% endhighlight %}

#### Produce a list of member names, with each row containing the total member count

**Question:**
> Produce a list of member names, with each row containing the total member count. Order by join date.

{% highlight sql %}
SELECT COUNT(*) OVER (), firstname, surname
FROM cd.members
order by joindate
{% endhighlight %}

#### Produce a numbered list of members

**Question:**
> Produce a monotonically increasing numbered list of members, ordered by their date of joining. Remember that member IDs are not guaranteed to be sequential.

{% highlight sql %}
SELECT ROW_NUMBER() OVER(order by joindate), firstname, surname
FROM cd.members
{% endhighlight %}

#### Output the facility id that has the highest number of slots booked, again

**Question:**
> Output the facility id that has the highest number of slots booked. Ensure that in the event of a tie, all tieing results get output.

{% highlight sql %}
SELECT f, s FROM (SELECT facid f, sum(slots) s, rank() over(order by sum(slots) desc) rank
FROM cd.bookings
GROUP BY facid) a
where rank = 1
{% endhighlight %}

#### Rank members by (rounded) hours used

**Question:**
> Produce a list of members, along with the number of hours they've booked in facilities, rounded to the nearest ten hours. Rank them by this rounded figure, producing output of first name, surname, rounded hours, rank. Sort by rank, surname, and first name.

{% highlight sql %}
SELECT firstname, surname, ROUND(SUM(slots)/2, -1) hours,
  RANK() OVER(ORDER BY ROUND(SUM(slots)/2, -1) DESC) rank
FROM cd.members m
  JOIN cd.bookings b ON m.memid = b.memid
GROUP BY m.memid
ORDER BY rank, surname, firstname
{% endhighlight %}

#### Find the top three revenue generating facilities

**Question:**
> Produce a list of the top three revenue generating facilities (including ties). Output facility name and rank, sorted by rank and facility name.

{% highlight sql %}
SELECT * FROM (SELECT name, RANK() OVER(ORDER BY SUM(cost) DESC) rank
FROM (SELECT  name,
  CASE
    WHEN memid = 0 THEN slots * guestcost
  ELSE slots * membercost
  END as cost
FROM cd.facilities f
  JOIN cd.bookings b ON b.facid = f.facid ) a
GROUP BY NAME) b
WHERE rank <= 3
{% endhighlight %}

#### Classify facilities by value

**Question:**
> Classify facilities into equally sized groups of high, average, and low based on their revenue. Order by classification and facility name.

{% highlight sql %}
WITH revenues AS (SELECT name, SUM(cost) rev
FROM (SELECT  name,
  CASE
    WHEN memid = 0 THEN slots * guestcost
  ELSE slots * membercost
  END as cost
FROM cd.facilities f
  JOIN cd.bookings b ON b.facid = f.facid ) a
GROUP BY NAME),

ranked_facilities AS (SELECT name, DENSE_RANK() OVER(ORDER BY REV DESC) rank FROM revenues),
classified_facilities AS (SELECT name, (MAX(rank) OVER() - rank)/3 as class FROM ranked_facilities)

SELECT name, 
  CASE
    WHEN class = 2 THEN 'high'
  WHEN class = 0 THEN 'low'
  ELSE 'average'
  END
FROM classified_facilities
ORDER BY class DESC, name 
{% endhighlight %}

#### Calculate the payback time for each facility

**Question:**
> Based on the 3 complete months of data so far, calculate the amount of time each facility will take to repay its cost of ownership. Remember to take into account ongoing monthly maintenance. Output facility name and payback time in months, order by facility name. Don't worry about differences in month lengths, we're only looking for a rough value here!

{% highlight sql %}
WITH revenues AS (SELECT name, SUM(cost) rev
FROM (SELECT name,
  CASE
    WHEN memid = 0 THEN slots * guestcost
  ELSE slots * membercost
  END as cost
FROM cd.facilities f
  JOIN cd.bookings b ON b.facid = f.facid ) a
GROUP BY NAME)

SELECT r.name, initialoutlay / ((AVG(rev) OVER(PARTITION BY r.name)/3) - monthlymaintenance)
FROM revenues r
  JOIN cd.facilities f ON f.name = r.name 
{% endhighlight %}
  
#### Calculate a rolling average of total revenue

**Question:**
> For each day in August 2012, calculate a rolling average of total revenue over the previous 15 days. Output should contain date and revenue columns, sorted by the date. Remember to account for the possibility of a day having zero revenue. This one's a bit tough, so don't be afraid to check out the hint!

{% highlight sql %}
WITH revenues AS (SELECT starttime::DATE dt, SUM(cost) rev
FROM (SELECT starttime,
  CASE
    WHEN memid = 0 THEN slots * guestcost
  ELSE slots * membercost
  END as cost
FROM cd.facilities f
  JOIN cd.bookings b ON b.facid = f.facid ) a
GROUP BY starttime::DATE)

SELECT * FROM (SELECT dt, avg(rev) over(order by dt rows between 14 PRECEDING AND CURRENT ROW) FROM revenues

ORDER BY dt) z
WHERE dt >= '2012-08-01' AND dt < '2012-09-01'
{% endhighlight %}

## Working with Timestamps 

#### Produce a timestamp for 1 a.m. on the 31st of August 2012

**Question:**
> Produce a timestamp for 1 a.m. on the 31st of August 2012.

{% highlight sql %}
SELECT CAST('2012-08-31 01:00:00' as timestamp)
{% endhighlight %}

#### Subtract timestamps from each other

**Question:**
> Find the result of subtracting the timestamp '2012-07-30 01:00:00' from the timestamp '2012-08-31 01:00:00'

{% highlight sql %}
SELECT '2012-08-31 01:00:00'::timestamp - '2012-07-30 01:00:00'::timestamp
{% endhighlight %}


#### Generate a list of all the dates in October 2012

**Question:**
> Produce a list of all the dates in October 2012. They can be output as a timestamp (with time set to midnight) or a date.

{% highlight sql %}
SELECT GENERATE_SERIES('2012-10-01', '2012-10-31', interval '1 day') as timestamp
{% endhighlight %}


#### Get the day of the month from a timestamp

**Question:**
> Get the day of the month from the timestamp '2012-08-31' as an integer.

{% highlight sql %}
SELECT EXTRACT('day' from  '2012-08-31'::timestamp)
{% endhighlight %}


#### Work out the number of seconds between timestamps

**Question:**
> Work out the number of seconds between the timestamps '2012-08-31 01:00:00' and '2012-09-02 00:00:00'

{% highlight sql %}
SELECT  extract( 'epoch' FROM ('2012-09-02 00:00:00'::timestamp - '2012-08-31 01:00:00'::timestamp))
{% endhighlight %}


#### Work out the number of days in each month of 2012

**Question:**
> For each month of the year in 2012, output the number of days in that month. Format the output as an integer column containing the month of the year, and a second column containing an interval data type.

{% highlight sql %}
WITH months AS (SELECT * FROM  GENERATE_SERIES('2012-01-01', '2013-01-01', INTERVAL '1 month') month)

SELECT extract('month' from month), lead(month::date) OVER (order by month) - month::date || ' days'  FROM months
limit 12
{% endhighlight %}

#### Work out the number of days remaining in the month

**Question:**
> For any given timestamp, work out the number of days remaining in the month. The current day should count as a whole day, regardless of the time. Use '2012-02-11 01:00:00' as an example timestamp for the purposes of making the answer. Format the output as a single interval value.

{% highlight sql %}
select date_trunc('month',  '2012-02-11 01:00:00'::date  + interval '1 month')
   - date_trunc('day',  '2012-02-11 01:00:00'::date)
{% endhighlight %}

#### Work out the end time of bookings

**Question:**
> Return a list of the start and end time of the last 10 bookings (ordered by the time at which they end, followed by the time at which they start) in the system.

{% highlight sql %}
SELECT starttime, starttime + (slots*(interval '0.5 hour')) endtime
FROM cd.bookings
ORDER BY endtime DESC, starttime desc
limit 10
{% endhighlight %}

#### Return a count of bookings for each month

**Question:**
> Return a count of bookings for each month, sorted by month

{% highlight sql %}
SELECT date_trunc('month', starttime) as month, count(*)
FROM cd.bookings
GROUP BY month
ORDER BY month
{% endhighlight %}

#### Work out the utilisation percentage for each facility by month

**Question:**
> Work out the utilisation percentage for each facility by month, sorted by name and month, rounded to 1 decimal place. Opening time is 8am, closing time is 8.30pm. You can treat every month as a full month, regardless of if there were some dates the club was not open.

{% highlight sql %}
WITH calendar AS (
  SELECT month,
    extract( 'epoch' from ((month + interval '1 month') - month))/(24*60*60) * 25 AS slots 
  FROM  GENERATE_SERIES('2012-01-01', '2013-01-01', INTERVAL '1 month') month)

SELECT name, bmonth, ROUND((sslots*100/slots)::numeric, 1)
FROM(
  SELECT f.name, date_trunc('month', b.starttime) AS bmonth, sum(slots) sslots
  FROM cd.facilities f
    JOIN cd.bookings b ON f.facid = b.facid
  GROUP BY f.name, bmonth
  ) a
  JOIN calendar c ON c.month = a.bmonth
ORDER BY name, bmonth
{% endhighlight %}

## String Operations 

#### Format the names of members

**Question:**
> Output the names of all members, formatted as 'Surname, Firstname'

{% highlight sql %}
SELECT CONCAT(surname, ', ', firstname) FROM cd.members
{% endhighlight %}

#### Find facilities by a name prefix

**Question:**
> Find all facilities whose name begins with 'Tennis'. Retrieve all columns.

{% highlight sql %}
SELECT * FROM cd.facilities WHERE name LIKE 'Tennis%'
{% endhighlight %}


#### Perform a case-insensitive search

**Question:**
> Perform a case-insensitive search to find all facilities whose name begins with 'tennis'. Retrieve all columns.

{% highlight sql %}
SELECT * FROM cd.facilities WHERE LOWER(name) LIKE 'tennis%'
{% endhighlight %}


#### Find telephone numbers with parentheses

**Question:**
> You've noticed that the club's member table has telephone numbers with very inconsistent formatting. You'd like to find all the telephone numbers that contain parentheses, returning the member ID and telephone number sorted by member ID.

{% highlight sql %}
SELECT memid, telephone
FROM cd.members
WHERE telephone ~ '^\('
ORDER BY memid
{% endhighlight %}


#### Pad zip codes with leading zeroes
**Question:**
> The zip codes in our example dataset have had leading zeroes removed from them by virtue of being stored as a numeric type. Retrieve all zip codes from the members table, padding any zip codes less than 5 characters long with leading zeroes. Order by the new zip code.

{% highlight sql %}
SELECT LPAD(zipcode::text, 5, '0') FROM cd.members
{% endhighlight %}

#### Count the number of members whose surname starts with each letter of the alphabet

**Question:**
> You'd like to produce a count of how many members you have whose surname starts with each letter of the alphabet. Sort by the letter, and don't worry about printing out a letter if the count is 0.

{% highlight sql %}
SELECT substr(surname, 1,1) l, count(*) FROM cd.members
GROUP BY l
ORDER BY l
{% endhighlight %}

#### Clean up telephone numbers

**Question:**
> The telephone numbers in the database are very inconsistently formatted. You'd like to print a list of member ids and numbers that have had '-','(',')', and ' ' characters removed. Order by member id.

{% highlight sql %}
SELECT memid, REGEXP_REPLACE(telephone::text, '[-, \(, \)]', '', 'g') FROM cd.members
ORDER BY memid
{% endhighlight %}

## Recursive Queries 

#### Find the upward recommendation chain for member ID 27

**Question:**
> Find the upward recommendation chain for member ID 27: that is, the member who recommended them, and the member who recommended that member, and so on. Return member ID, first name, and surname. Order by descending member id.

{% highlight sql %}
WITH RECURSIVE recommender(id) as (
  SELECT recommendedby FROM cd.members where memid = 27
  UNION ALL
  SELECT recommendedby FROM cd.members m, recommender r where m.memid = r.id
  )
  
SELECT r.id, m.firstname, m.surname FROM recommender r
  JOIN cd.members m ON m.memid = r.id
order by memid desc 
{% endhighlight %}

#### Find the downward recommendation chain for member ID 1

**Question:**
> Find the downward recommendation chain for member ID 1: that is, the members they recommended, the members those members recommended, and so on. Return member ID and name, and order by ascending member id.

{% highlight sql %}
WITH RECURSIVE recommendations(ids) as (
  SELECT ARRAY[memid] FROM cd.members where recommendedby in (1)
  UNION
  SELECT ARRAY[memid] FROM cd.members m, recommendations r WHERE  m.recommendedby = ANY(ids)
  )
  
SELECT ids[1] id, firstname, surname FROM recommendations r
  JOIN cd.members ON ids[1] = memid
ORDER BY id
{% endhighlight %}

#### Produce a CTE that can return the upward recommendation chain for any member

**Question:**
> Produce a CTE that can return the upward recommendation chain for any member. You should be able to select recommender from recommenders where member=x. Demonstrate it by getting the chains for members 12 and 22. Results table should have member and recommender, ordered by member ascending, recommender descending.

{% highlight sql %}
WITH RECURSIVE recommenders(id, rid) AS (
  SELECT memid, recommendedby FROM cd.members
  UNION ALL
  SELECT r.id, m.recommendedby
  FROM cd.members m
    JOIN recommenders r ON r.rid = m.memid
  )
  
SELECT r.id, r.rid,m.firstname, m.surname FROM recommenders r
   JOIN cd.members m ON m.memid = r.rid
WHERE rid IS NOT NULL AND id IN (12,22)
ORDER BY id, rid DESC
{% endhighlight %}
