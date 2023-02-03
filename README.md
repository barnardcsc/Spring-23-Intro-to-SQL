## Introduction to SQL and Relational Databases - Spring 2023


#### Prerequisites

- [Download Github repository:](https://github.com/Barnard-Computational-Science-Center/2022-Spring-Intro-to-SQL-and-Databases)
  - _Code > Download ZIP_
- Get access to a relational database system (preferably Postgres).
  - [The easiest way I've found is through bit.io](https://bit.io/), which is what I will use for this workshop. It's free to create an account with up to 3 databases. 
  - You can also [download Postgres](https://www.postgresql.org/download/) and install it on your computer.
  - Alternatively, you can create an instance on any of the major cloud providers (AWS, Microsoft Azure, Google Cloud, Digital Ocean). 
- If you signed up for [bit.io](bit.io), you can use the built-in editor. I'll use the built-in editor to keep our environments consistent. You can also access your database with an external GUI client:
  - [pgAdmin (free)](https://www.pgadmin.org)
  - [DBeaver (free)](https://dbeaver.io)
  - [Datagrip (free for students)](https://www.jetbrains.com/community/education/#students)
  - [Postico (discounted for students, Mac only)](https://eggerapps.at/postico2/buy.html)
  - [TablePlus (my preferred tool - discounted for students)](https://tableplus.com/blog/2018/04/education-plan.html)

#### Learning goals
- Why you need and should use a database
- How to set up the database that you need
- How to think about developing a sound schema
- How to implement and create the schema
- How to create, update, delete, and query your database

#### Resources

- [PostgreSQL Exercises](https://pgexercises.com/)
- [Postgres Tutorials](https://www.postgresqltutorial.com/)
- [Official documentation](https://www.postgresql.org/docs/current/index.html)
- ["Awesome" Postgres Github Page](https://github.com/dhamaniasad/awesome-postgres)
- [The Art of PostgreSQL by Dimitri Fontaine](https://theartofpostgresql.com/)
- [Official Postgres repository](https://git.postgresql.org/gitweb/?p=postgresql.git;a=summary)

---

#### 1. Introduction

##### Why databases?

- Scale & size (when Excel won't cut it as a datastore)
- Safety (data persistence, preventing corruption)
- Organized, structured, accessible
- The foundation of your app, project, API, etc.


##### Why SQL (structured query language)?
- Standardized
- Declarative (describe what you want, not how)
- Ubiquitous and a valuable skill


##### General Tips
- Lowercase all column names
- Don’t use spaces or special characters inside of column names. 
- Only use underscores as the separator (“first name” v. first_name)
- Don’t use keywords as table or column names. You can find the list of [keywords here](https://www.postgresql.org/docs/current/sql-keywords-appendix.html).
- Strive to use the correct column types:
    - Don’t store dates as strings, etc. 
    - For strings, don't use CHAR(N) or VARCHAR types, just use TEXT.
- Be extra careful with the DELETE command as it's very easy to accidentally delete data.


#### 1.1 The basics

Create a table called `states`:

```SQL
CREATE TABLE "public"."states" (
	"id" INT GENERATED BY DEFAULT AS IDENTITY,
	"name" TEXT NOT NULL UNIQUE,
	"abbreviation" TEXT NOT NULL UNIQUE,
	"population" INT NOT NULL,
	PRIMARY KEY ("id")
);
```

Insert a row:

```SQL
INSERT INTO states (name, abbreviation, population)
		VALUES('New York', 'NY', 20000000);
```

Insert another row, but keep the 'NY' abbreviation. What happens? 

```SQL
INSERT INTO states (name, abbreviation, population)
		VALUES('Virginia', 'NY', 8600000);
		
-- Unique constraint error
```

Fix the error: 

```SQL
INSERT INTO states (name, abbreviation, population)
		VALUES('Virginia', 'VA', 8600000);
```

Update the population: 

```SQL
UPDATE
	states
SET
	population = 8642000
WHERE
	name = 'Virginia';
```


#### 1.2 Relational tables, foreign keys, one-to-many

How would we store city data in our database? For example, suppose we wanted to store every city in NY and VA. One way is to expand our table:


```SQL
-- This is a bad idea:
ALTER TABLE states
	ADD COLUMN city1 TEXT,
		ADD COLUMN city2 TEXT,
		...;

```

Problems with this approach:

- Our columns grow with each additional city
- We have to add an additional column for each new attribute (ex. city1_population)

The solution is to create a new table, `cities`: 


```SQL
CREATE TABLE "public"."cities" (
	"id" INT GENERATED BY DEFAULT AS IDENTITY,
	"state_id" INT NOT NULL,
	"name" TEXT NOT NULL,
	CONSTRAINT "state_id_fkey" FOREIGN KEY ("state_id") REFERENCES "public"."states" ("id"),
	PRIMARY KEY ("id")
);
```

The `states` and `cities` tables are connected by the `state_id` foreign key in the `cities` table. They have a `one-to-many` relationship: while a state can have many cities, a city can only be in one state. 


Let's add some city data. First check your state ids:


```SQL
SELECT * FROM states
```

Then:


```SQL
INSERT INTO cities (state_id, name)
		VALUES (1, 'New York City'), (1, 'Buffalo'), (3, 'Alexandria'), (3, 'Virginia Beach')
```

#### 1.3 Joining data, aliasing, views

Often, you will want to combine data. Suppose you want the city data that includes the state:

```SQL
SELECT
	c.id,
	c.name AS "city",
	s.name AS "state"
FROM
	cities c
	JOIN states s ON s.id = c.state_id
```

When we join a table, we have access to all of its columns. But since some of the column names in our tables are identical (id, name), we have to specify which one we want. We can use aliases to do so: `cities c` and `states s`.

You can pick any string to be an alias, it just needs to come after the table name. A common convention is the first letter of the table name. If you omit the alias, you get an ambiguity error: 

```SQL
ERROR:  column reference "id" is ambiguous
```

Queries can get large and complicated fast, especially when joining tables. It can be difficult to manage large blocks of SQL. One way to deal with this is to create views of commonly used queries. Using the above query: 


```SQL
CREATE VIEW vi_cities_with_states AS (
	SELECT
		c.id,
		c.name AS "city",
		s.name AS "state"
	FROM
		cities c
		JOIN states s ON s.id = c.state_id
);
```
	
Then query the data:


```SQL
SELECT * FROM vi_cities_with_states
```

---

#### 2.0 Normalization exercise

[Download the data normalization exercise.](https://github.com/Barnard-Computational-Science-Center/2022-Spring-Intro-to-SQL-and-Databases/raw/main/data-norm-exercise.xlsx)

We want to structure our database to ensure data integrity and remove redundancy. We briefly performed some normalization earlier by creating a one-to-many relationship between our `states` and  `cities` tables. The exercise above will reinforce these concepts. 

Instructions:


1. Start by analyzing the unnormalized data. What do you notice? Can you think of better ways to organize it?  
2. Use the `Normalize customers` tab to get an idea of what we should do.
3. In the `Assignment` tab, try applying normalization concepts to the `item` column in the `sales` table.

---

#### 3. Build the tables in the normalization exercise

Customers:

```SQL
CREATE TABLE "public"."customers" (
	"id" INT GENERATED BY DEFAULT AS IDENTITY,
	"name" TEXT NOT NULL,
	"loyalty_member" INT NOT NULL DEFAULT 0, -- if no value supplied, defaults to 0.
	"postal_code" TEXT, -- nullable.
	"date_of_birth" DATE, -- nullable.
	PRIMARY KEY ("id") -- Assign primary key (non-null & unique)
);
```

Items:

```SQL
CREATE TABLE "public"."items" (
	"id" INT GENERATED BY DEFAULT AS IDENTITY,
	"name" TEXT NOT NULL,
	PRIMARY KEY ("id")
);
```

Sales:

```SQL
CREATE TABLE "public"."sales" (
	"id" INT GENERATED BY DEFAULT AS IDENTITY,
	"customer_id" INT NOT NULL,
	"item_id" INT NOT NULL,
	CONSTRAINT "customer_id_fkey" FOREIGN KEY ("customer_id") REFERENCES "public"."customers" ("id"),
	CONSTRAINT "item_id_fkey" FOREIGN KEY ("item_id") REFERENCES "public"."items" ("id"),
	PRIMARY KEY ("id")
);
```

Notice that `sales` was created last, as it has foreign key columns that reference the other two tables.


#### 3.1 Insert data

Populate `customers`:

```SQL
-- Populate customers
INSERT INTO "customers" (name, loyalty_member, postal_code, date_of_birth)
		VALUES('Mark', 0, '99524', '1980-02-01'), ('Nia', 1, '85055', '1965-11-05'), ('Fred', 0, '49036', '1977-06-15');
```


Populate `items`:

```SQL
INSERT INTO items (name)
		VALUES('shoes'), ('pants'), ('shirt'); 
```



Populate `sales`:

```SQL
INSERT INTO sales (customer_id, item_id)
		VALUES(1, 1), (2, 2), (1, 1), (2, 2), (3, 2), (3, 3), (1, 3), (2, 1), (3, 3), (3, 3);
```




#### 3.2 Querying the data

How many times has each item been purchased?

```SQL
SELECT
	item_id,
	COUNT(item_id)
FROM
	sales
GROUP BY
	item_id
```

Join the items table, and get the item name as well: 

```SQL
SELECT
	s.item_id,
	i.name,
	COUNT(s.item_id)
FROM
	sales s
	JOIN items i ON i.id = s.item_id
GROUP BY
	s.item_id,
	i.name
```

#### 3.3 Monetary values, alter table, alter column

We're going to add a new column to items: `price`. 

While there is a `money` column type, the better way to handle monetary values is to use integers that are equal to the smallest fractional amount in your base currency:

- 1c = 1
- $5.34 = 534


Add the new `price` column: 

```SQL
ALTER TABLE items ADD COLUMN price INT;
```
	
Double-check the table:

```SQL
SELECT * FROM items;
```

Update the prices: 

```SQL
UPDATE items SET price = 3500 WHERE id = 1;
UPDATE items SET price = 2200 WHERE id = 2;
UPDATE items SET price = 1300 WHERE id = 3;
```

After updating the prices, add a `NOT NULL` constraint to the `price` column. I couldn't do this earlier, because there weren't prices to begin with, so it would have thrown an error:

```SQL
ALTER TABLE items ALTER COLUMN price SET NOT NULL;
```

#### 3.4 More advanced queries

Calculate the total revenue (sum of sales) for each customer. Start with something simple and build on it: 

```SQL
SELECT s.item_id, s.customer_id FROM sales s;
```
We have customers and what they purchased. Now we need prices:

```SQL
SELECT
	s.item_id, -- do you need this column?
	s.customer_id,
	i.price
FROM
	sales s
	JOIN items i ON i.id = s.item_id
	ORDER BY customer_id
```

All the data is there, now some aggregation. We want to `SUM` over the prices, then `GROUP BY` the customer:

```SQL
SELECT
	s.customer_id,
	SUM(i.price) as "total_sales"
FROM
	sales s
	JOIN items i ON i.id = s.item_id
GROUP BY
	s.customer_id
ORDER BY
	customer_id
```


Create a view: 


```SQL
CREATE VIEW vi_total_sales_per_customer AS (
	SELECT
		s.customer_id,
		SUM(i.price) AS "total_sales"
	FROM
		sales s
		JOIN items i ON i.id = s.item_id
	GROUP BY
		s.customer_id
	ORDER BY
		customer_id)
```

Select your data: 

```SQL
SELECT * FROM vi_total_sales_per_customer
```

---

#### 4. Vehicle collision database

- Working raw data: [2022-01-vehicle-collisions](https://data.cityofnewyork.us/Public-Safety/Motor-Vehicle-Collisions-Crashes/h9gi-nx95)
- [CSV files to import into tables](https://github.com/Barnard-Computational-Science-Center/2022-Spring-Intro-to-SQL-and-Databases/tree/main/csv-data)
- Original source data: [NYC Motor Vehicle Collisions - Crashes](https://data.cityofnewyork.us/Public-Safety/Motor-Vehicle-Collisions-Crashes/h9gi-nx95)
- The original dataset is a two millions rows and spans many years. I dropped some columns, and only looked at January 2022 (~7.8K records).


#### 4.1 Inspect the data, develop a schema

Download and open the working raw data: [2022-01-vehicle-collisions](https://data.cityofnewyork.us/Public-Safety/Motor-Vehicle-Collisions-Crashes/h9gi-nx95). What issues do you see?

- Column for each vehicle involved in a collision
- Since most collisions involve only 1 or 2 vehicles, the majority of row values are empty.

Other things to consider:

- Column names should be renamed, lowercased, and separated by underscores
- Find the primary key - `COLLISION_ID` is a good candidate, but need to check that it's `NOT NULL` and `UNIQUE`. Use Excel filters and `Data > Remove Duplicates` to check.

One note on normalization here: we're only going to normalize the vehicles type code columns. You could also do it for boroughs and the street name columns.


#### 4.2 Create the tables


Collisions: 

```SQL
CREATE TABLE "public"."collisions" (
	"id" INT NOT NULL, -- Because we aren't starting from 1
	"date" DATE NOT NULL,
	"time" TIME NOT NULL,
	"borough" TEXT,
	"zip_code" TEXT,
	"latitude" DOUBLE PRECISION,  -- 15 decimal digits precision
	"longitude" DOUBLE PRECISION , -- 15 decimal digits precision
	"on_street_name" TEXT, 
	"cross_street_name" TEXT,
	"off_street_name" TEXT,
	PRIMARY KEY ("id")
);
```

Vehicles: 

```SQL
CREATE TABLE "public"."vehicles" (
	"id" INT GENERATED BY DEFAULT AS IDENTITY,
	"vehicle" TEXT NOT NULL,
	PRIMARY KEY ("id")
);
```


Vehicle collisions:

```SQL
CREATE TABLE "public"."vehicle_collisions" (
	"id" INT GENERATED BY DEFAULT AS IDENTITY,
	"collision_id" INT NOT NULL,
	"vehicle_id" INT NOT NULL,
	CONSTRAINT "collision_id_fkey" FOREIGN KEY ("collision_id") REFERENCES "public"."collisions" ("id"),
	CONSTRAINT "vehicle_id_fkey" FOREIGN KEY ("vehicle_id") REFERENCES "public"."vehicles" ("id"),
	PRIMARY KEY ("id")
);
```

#### 4.3 Import the CSV files

Import the CSV files into the respective tables. On bit.io, click the `Upload Data` tab on the toolbar, select the table, and upload the file.

> The import tool might throw a couple minor type errors on the `collisions` tables, but it should be okay. 


#### 4.4 Query the data

We have some interesting new column types here, like coordinates and time. 

Start with something simple, like collisions per day:

```SQL
SELECT
	date,
	COUNT(date)
FROM
	collisions
GROUP BY
	date
ORDER BY
	date
```

Can you get the number of collisions per hourly block?

```SQL
SELECT
	COUNT(id),
	time
FROM
	collisions
GROUP BY
	time,
	date_trunc('hour', time)
ORDER BY
	count DESC
```

Include the day-of-week (1: Monday, 7: Sunday):

```SQL
SELECT
	date,
	COUNT(date),
	extract(isodow FROM date) AS "day_of_week"
FROM
	collisions
GROUP BY
	date
ORDER BY
	date
```


Get the mean number of collisions per day of week. This query involves a subquery:

```SQL
SELECT
	AVG(daily_collisions),
	day_of_week AS avg_daily_collisions
FROM (
	SELECT
		COUNT(date) AS "daily_collisions",
		extract(isodow FROM date) AS "day_of_week"
	FROM
		collisions
	GROUP BY
		date
	ORDER BY
		date) a
GROUP BY
	day_of_week
```

Collisions per vehicle:

```SQL
SELECT
	vc.vehicle_id,
	v.vehicle,
	COUNT(vc.vehicle_id)
FROM
	vehicle_collisions vc
	JOIN vehicles v ON v.id = vc.vehicle_id
GROUP BY
	vc.vehicle_id,
	v.vehicle
ORDER BY
	count DESC
```

---

#### 5. Finishing up, ideas to explore...

This data includes lat/long coordinates and Postgres has a powerful GIS module called [PostGIS](https://postgis.net). You can use this module to: 

- Trivially compute distances between points
- Find clusters of points
- Combine with other geographic data

Other things to consider:

- Can you combine this data with other data, such as the distribution of vehicle types on the road, to find out which vehicles are most likely to end up in a collision? 
- Identify particularly dangerous thoroughfares 
- Promote solutions (better signage, adjust speed limits, etc). 
- You could save someone's life!
