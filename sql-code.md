# 🛫 SQL Technical Reference

This file houses the SQL code utilized to uncover insights and analytics for this project. The SQL scripts provided here serve as a record of the queries and transformations performed on the dataset and provided in my Data Analysis Report.

## Table of Contents
  - [Section 4.1 Demographic Analysis](#41-demographic-analysis)
      - [Provinces](#provinces)
      - [Cities](#cities)
      - [Marital Status and Education](#marital-status-and-education)
      - [Income](#income)
  - [Section 4.3 Loyalty Points](#42-loyalty-points)
  
## 4.1 Demographic Analysis

### Provinces

- The code shown below is used to find the percentage of loyalty members that reside in each province in the dataset and sorted by highest to lowest percentage.

```sql

SELECT 
	f.province,
	COUNT(DISTINCT f.loyalty_number) AS number_of_members,
	ROUND((COUNT(DISTINCT f.loyalty_number) * 100.0 /
(SELECT 
		 	COUNT(*)
		FROM			
flight_loyalty_history f)), 2) AS percentage	
FROM
	flight_loyalty_history f
GROUP BY
	f.province
ORDER BY
	percentage DESC

```

### Cities

- The code shown below is used to find the percentage of loyalty members that reside in each city in the dataset and sorted by highest to lowest percentage.

```sql

SELECT 
	f.city,
	COUNT(DISTINCT f.loyalty_number) AS number_of_members,
	ROUND((COUNT(DISTINCT f.loyalty_number) * 100.0 /
	 	(SELECT 
		 	COUNT(*)
		FROM
			flight_loyalty_history f)), 2) AS percentage	
FROM 
	flight_loyalty_history f
GROUP BY
	f.city
ORDER BY
	percentage DESC

```

### Marital Status and Education

- The code shown below is used to find the highest percentage of loyalty members that are of a similar marital status and education background and sorted by highest to lowest percentage.

```sql

SELECT
	f.education,
	f.marital_status,
	ROUND((COUNT(DISTINCT f.loyalty_number) * 100.0 /
	 	(SELECT 
		 	COUNT(*)
		FROM
			flight_loyalty_history f)), 2) AS percentage
FROM
	flight_loyalty_history f
GROUP BY
	f.education,
	f.marital_status
ORDER BY
	percentage DESC

```
- The code shown below calculates the percentage of loyalty members who traveled with companions, categorized by marital status. The CTE selects members who had at least 1 flight with a companion, and the main query lists each marital status type, the total number of accompanied flights, and the percentage of members out of the total member count and groups all results by marital status.

```sql

WITH flights_with_companions AS (
	SELECT
		c.loyalty_number,
		c.flights_with_companions AS accompanied_flights
	FROM
		customer_flight_activity c
	WHERE
		c.flights_with_companions != 0
)

--- end of cte ---

SELECT
	f.marital_status,
	SUM(fc.accompanied_flights) AS total_accompanied_flights,
	ROUND((COUNT(DISTINCT f.loyalty_number) * 100.0/
		   	(SELECT
  				COUNT(*)
  			FROM
  				flight_loyalty_history f)), 2) AS percentage
FROM
	flight_loyalty_history f
		JOIN flights_with_companions fc
			ON f.loyalty_number=fc.loyalty_number
GROUP BY
	f.marital_status
ORDER BY
	percentage DESC

```

### Income

- The code shown below finds the percentages of members in different salary brackets, using a CTE to define the different levels of the salary bracket using a `CASE WHEN` expression. The main query lists each salary bracket and the percentage of members out of the total member count in each group, ordering them from the highest percentage to the lowest.

```sql

WITH salary_bracket AS (
	SELECT
		f.loyalty_number,
		CASE
			WHEN f.salary < 50000 THEN 'Under $50k'
			WHEN f.salary >=50000 AND f.salary <= 75000 THEN 'Between $50k and $75k'
			WHEN f.salary > 75000 AND f.salary <= 100000 THEN 'Between $75k and $100k'
			WHEN f.salary > 100000 THEN 'Over $100k'
			ELSE 'Unknown Salary'
		END AS salary_bracket
	FROM
		flight_loyalty_history f
)

--- end of cte

SELECT
	s.salary_bracket,
	ROUND((COUNT(DISTINCT f.loyalty_number) * 100.0 /
	 	(SELECT 
		 	COUNT(*)
		FROM
			flight_loyalty_history f)), 2) AS percentage
FROM
	flight_loyalty_history f
		JOIN salary_bracket s
			ON f.loyalty_number=s.loyalty_number
GROUP BY
	s.salary_bracket
ORDER BY
	percentage DESC


```
## 4.2 Flight Booking Patterns

- The code below calculates the percentage of accompanied flights by dividing the number of them by the total flights after casting the data type as a decimal to ensure the division involves at least one decimal value so the result returns as a decimal as well.

```sql

SELECT
	ROUND(SUM(c.flights_with_companions) / CAST(SUM(c.total_flights) AS DECIMAL(18,2))*100, 1) AS percentage
FROM
  customer_flight_activity c;

```

## 4.3 Loyalty Points

- The code below calculates the percentage of points redeemed out of the total points accumulated and groups the percentages by loyalty card type.

```sql

SELECT
	f.loyalty_card,
	ROUND((SUM(c.points_redeemed)/SUM(c.points_accumulated))*100, 1)	
FROM
	flight_loyalty_history f
		JOIN
			customer_flight_activity c
				ON f.loyalty_number=c.loyalty_number
GROUP BY
	f.loyalty_card

```
