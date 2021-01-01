---
layout: post
title: GROUP BY and aggregate functions
published: true
author: alexander
comments: true
date: 2015-09-18 01:09:58
tags:
    - aggregate functions
    - group by
    - MySQL
    - postgres
    - SQL
categories:
    - uncategorized
permalink: /group-aggregate-functions
---
Imagine you have the following table &#8216;users&#8217;:

| id | username | level | points |
| -- | -------- | ----- | ------ |
| 1  | jsmith   | 10    | 1000   |
| 2  | fbloggs  | 10    | 2000   |
| 2  | jdoe     | 3     | 100    |
| 4  | mbloggs  | 3     | 1000   |
| 5  | bdavis   | 8     | 500    |

You can do some pretty interesting stuff with this data. You could determine the results of a contest for who can acquire the most points. Let&#8217;s try that.

```sql
SELECT username, MAX(points) FROM users;
```

<pre><samp>+----------+-------------+
| username | MAX(points) |
+----------+-------------+
| jsmith   |        2000 |
+----------+-------------+
</samp></pre>


Whoa, hang on a minute. That&#8217;s not right. What just happened here?

`MAX()` is an aggregate function. What does that mean? It means its job is to take in many rows, perform an operation to combine all the data, and return a single result. We got our single result. But what happened to username? It got aggregated. We asked for two pieces of information that didn&#8217;t necessarily have anything to do with each other.

It&#8217;s worth pointing out that Postgres would not let you make this mistake.

<pre><samp>ERROR:  column "users.username" must appear in the GROUP BY clause or be used in an aggregate function.</samp></pre>

Let&#8217;s use Postgres for now to be safe.

```sql
SELECT username, points FROM users ORDER BY points DESC;
```

<pre><samp> username | points
----------+--------
 fbloggs  |   2000
 jsmith   |   1000
 mbloggs  |   1000
 bdavis   |    500
 jdoe     |    100
</samp></pre>


jsmith and mbloggs are tied for second place. Listing users that are tied on the same line, separated by commas would be a better way to see this.

```sql
SELECT ARRAY_AGG(username), points FROM users GROUP BY points ORDER BY points DESC;
```

in Postgres or in MySQL,

```sql
SELECT GROUP_CONCAT(username), points FROM users GROUP BY points ORDER BY points DESC;
```

<pre><samp>    array_agg     | points
------------------+--------
 {fbloggs}        |   2000
 {jsmith,mbloggs} |   1000
 {bdavis}         |    500
 {jdoe}           |    100
</samp></pre>

Can we simplify the SQL at all? What if we want just the user with the most points? What about this:

```sql
SELECT ARRAY_AGG(username), MAX(points) from users;
```

The username column is now in an aggregate function too, so Postgres shouldn&#8217;t complain.


<pre><samp>              array_agg               | max
--------------------------------------+------
 {jsmith,fbloggs,jdoe,mbloggs,bdavis} | 2000
</samp></pre>

Wow. We&#8217;ve managed to shoot ourselves in the foot even with Postgres. Since this is the second time we&#8217;ve made a mistake like this, let&#8217;s take a moment to dig a bit deeper about why this doesn&#8217;t work the way we might think it should in this particular situation.

What is `MAX()`? It returns the maximum value for a particular column given a group of rows. It&#8217;s a piece of statistical information about a GROUP of rows. You don&#8217;t need statistics about a single row.

For instance, it&#8217;s more apparent that it wouldn&#8217;t make sense to do this:

```sql
SELECT id, COUNT(*) FROM users;
```

The number of rows doesn&#8217;t really have much to do with any of the ids.

## Picking a Winner Correctly

So how would you accomplish this? Like this:

```sql
SELECT MAX(points) FROM users;
```

<pre><samp> max
------
 2000
(1 row)
</samp></pre>

```sql
SELECT username, points FROM users WHERE points=2000;
```

-or-

```sql
SELECT username, points FROM users WHERE points = (SELECT MAX(points) FROM users);
```

Notice that we use one query (or subquery) to find the highest point value, and a second query to find all users (there could be a tie) with that value. Aggregates aren&#8217;t for working with single rows.

## Using Aggregate Functions Correctly

So what would be an appropriate way to use aggregate functions and `GROUP BY`? What if we wanted to look at the amount of participation of the various user levels?

```sql
SELECT level, SUM(points), MIN(points), MAX(points) FROM users GROUP BY level;
```


<pre><samp> level | sum  | min  | max
-------+------+------+------
     8 |  500 |  500 |  500
     3 | 1100 |  100 | 1000
    10 | 3000 | 1000 | 2000
</samp></pre>

Here we&#8217;re dealing with groups of rows that have something in common: level. We group all users of the same level into a single pseudo-row. Note that, just as it doesn&#8217;t make sense to use aggregate functions on a single row, it doesn&#8217;t make sense to talk about a single column of a group of rows. Users of level 10 don&#8217;t have a single points value, they have many. How do you want them aggregated? Do you want to add them up? Concatenate them into a list? Find the highest?

The one sort of gray area/exception in this case is level 8. Because there is only one row in the group, there is technically only one points value, only one id, etc. It might be helpful to think of these cases not as a single row, but as a set or array containing one row. [5] instead of 5.
