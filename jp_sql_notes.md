# JP SQL, SQLAlchemy, PyMongo, and PostgreSQL Notes

<BR>

Good reference sites:
1. [W3 School - SQL](https://www.w3schools.com/sql/default.asp)
2. [Mode Analytics - SQL](https://community.modeanalytics.com/sql/tutorial/introduction-to-sql) With beginner, intermediate, and advanced tutorials
3. [PostgreSQL Manual](https://www.postgresql.org/docs/9.6/static/index.html) Very good examples of SQL unrelated to PostgreSQL itself.
4. [SQL Window Functions (Group By -> Transforms)](https://www.sqltutorial.org/sql-window-functions/)



## SQL General Notes
***
**SQL** stands for **Structured Query Language** and has been around since the 1970s.  It's a language used to interact with databases and is industry standard, with various "flavors" being used in a given industry or company.  Python supports many of these flavors, with the two most common likely being **SQLite3** and **MySQL**.  Certain frameworks or modules for utilizing SQL in Python, such as **SQLAlchemy** and **PostgreSQL**, are popular and more powerful/convenient than vanilla SQL. There are other database access languages and management systems, such as **NoSQL** or **MongoDB** (**PyMongo**).  For these notes we will be focusing on SQL.

<BR>

#### Entity Relationship Diagram (ERD)
+ **Primary Key (PK)**
  + The main column header, identifies all the data in the column beneath it, in a specific table.   
  + When a PK is referenced in a separate table it is known as a Foreign Key (FK) to the separate table.
    + Note: when a PK is used in another table that has its own PK with the exact same name as the FK, it must be specified explicitly in reference by using the `table-name.FK` method.  
  + The most common relationship between multiple tables (databases?) is a "one-to-many" relationship.  The notion of "many-to-many" is highly complicated and is, for most practical purposes, going to be avoided.
<BR><BR>

#### Aggregate functions  
When using an aggregate function, such as `COUNT()` or `AVERAGE()`, you have to include the remaining columns in the `SELECT` statement that aren't part of the aggregate functions in a `GROUP BY` statement.  Period.  

Note you can do an `ORDER BY`, a `GROUP BY`, or any other aggregate wrap-up function on a column you aren't `SELECT`ing for.  I'm not sure why you'd want to do this, but it is valid.

`HAVING` is used basically as a `WHERE` statement for aggregate functions.  For example, you can't do a query with a `WHERE` clause asking for cells that have a `COUNT` of 5, say.  It will crash (unless there is a Primary Key column of the count already listed).  You can't do a `WHERE` on an aggregate calculation made in the `SELECT` statement.  So, use `HAVING` a `COUNT` = 5, instead.

<BR>

#### Creating SQL databases
You can use the free [SQLite DB Browser](http://sqlitebrowser.org/) to create a database from SQL (or export to SQL).

[SQL Fiddle](www.sqlfiddle.com) to create databases.  You can also copy and paste entire files, like .csv files, into a database builder and it will create a SQL database from the stuff you pasted.  Neat!  Great for me and NFL spreadsheets...

<BR><BR>




## SQLite
***
This section is taken from the [Python-Course page](http://www.python-course.eu/sql_python.php) on SQLite.  

SQLite is a simple relational database system, which saves its data in regular data files or even in the internal memory of the computer, i.e. the RAM. It was developed for embedded applications, like Mozilla-Firefox (Bookmarks), Symbian OS or Android. SQLITE is "quite" fast, even though it uses a simple file. It can be used for large databases as well. If you want to use SQLite, you have to import the module sqlite3. To use a database, you have to create first a Connection object. The connection object will represent the database. The argument of connection - in the following example "`company.db`" - functions both as the name of the file, where the data will be stored, and as the name of the database. If a file with this name exists, it will be opened. It has to be a SQLite database file of course! In the following example, we will open a database called company. The file does not have to exist.:

```python
>>> import sqlite3
>>> connection = sqlite3.connect("company.db")
```  

We have now created a database with the name "company". It's like having sent the command "`CREATE DATABASE company;`" to a SQL server. If you call "`sqlite3.connect('company.db')`" again, it will open the previously created database.  

After having created an empty database, you will most probably add one or more tables to this database. The SQL syntax for creating a table "employee" in the database "company" looks like this:  

```sql
CREATE TABLE employee (
staff_number INT NOT NULL AUTO_INCREMENT,
fname VARCHAR(20),
lname VARCHAR(30),
gender CHAR(1),
joining DATE,
birth_date DATE,  
PRIMARY KEY (staff_number) );
```

This is the way, somebody might do it on a SQL command shell. Of course, we want to do this directly from Python. To be capable to send a command to "SQL", or SQLite, we need a cursor object. Usually, a cursor in SQL and databases is a control structure to traverse over the records in a database. So it's used for the fetching of the results. In SQLite (and other Python DB interfaces)it is more generally used. It's used for performing all SQL commands.

We get the cursor object by calling the cursor() method of connection. An arbitrary number of cursors can be created. The cursor is used to traverse the records from the result set. We can define a SQL command with a triple quoted string in Python:

```python
sql_command = """
CREATE TABLE employee (
staff_number INTEGER PRIMARY KEY,
fname VARCHAR(20),
lname VARCHAR(30),
gender CHAR(1),
joining DATE,
birth_date DATE);"""
```

Concerning the SQL syntax: You may have noticed that the `AUTOINCREMENT` field is missing in the SQL code within our Python program. We have defined the staff_number field as "`INTEGER PRIMARY KEY`" A column which is labelled like this will be automatically auto-incremented in SQLite3. To put it in other words: If a column of a table is declared to be an `INTEGER PRIMARY KEY`, then whenever a `NULL` will be used as an input for this column, the `NULL` will be automatically converted into an integer which will one larger than the highest value so far used in that column. If the table is empty, the value 1 will be used. If the largest existing value in this column has the 9223372036854775807, which is the largest possible INT in SQLite, an unused key value is chosen at random.

Now we have a database with a table but no data included. To populate the table we will have to send the "`INSERT`" command to SQLite. We will use again the execute method. The following example is a complete working example. To run the program you will either have to remove the file *company.db* or uncomment the "`DROP TABLE`" line in the SQL command:

```python
import sqlite3
connection = sqlite3.connect("company.db")

cursor = connection.cursor()

delete
cursor.execute("""DROP TABLE employee;""")

sql_command = """
CREATE TABLE employee (
staff_number INTEGER PRIMARY KEY,
fname VARCHAR(20),
lname VARCHAR(30),
gender CHAR(1),
joining DATE,
birth_date DATE);"""

cursor.execute(sql_command)

sql_command = """INSERT INTO employee (staff_number, fname, lname, gender, birth_date)
    VALUES (NULL, "William", "Shakespeare", "m", "1961-10-25");"""
cursor.execute(sql_command)


sql_command = """INSERT INTO employee (staff_number, fname, lname, gender, birth_date)
    VALUES (NULL, "Frank", "Schiller", "m", "1955-08-17");"""
cursor.execute(sql_command)

# never forget this, if you want the changes to be saved:
connection.commit()

connection.close()
```

Of course, in most cases, you will not literally insert data into a SQL table. You will rather have a lot of data inside of some Python data type e.g. a dictionary or a list, which has to be used as the input of the insert statement.

The following working example, assumes that you have already an existing database company.db and a table employee. We have a list with data of persons which will be used in the `INSERT` statement:

```python
import sqlite3
connection = sqlite3.connect("company.db")

cursor = connection.cursor()

staff_data = [ ("William", "Shakespeare", "m", "1961-10-25"),
               ("Frank", "Schiller", "m", "1955-08-17"),
               ("Jane", "Wall", "f", "1989-03-14") ]

for p in staff_data:
    format_str = """INSERT INTO employee (staff_number, fname, lname, gender, birth_date)
    VALUES (NULL, "{first}", "{last}", "{gender}", "{birthdate}");"""

    sql_command = format_str.format(first=p[0], last=p[1], gender=p[2], birthdate = p[3])
    cursor.execute(sql_command)
```

The time has come now to finally query our employee table:

```python
import sqlite3
connection = sqlite3.connect("company.db")

cursor = connection.cursor()

cursor.execute("SELECT * FROM employee")
print("fetchall:")
result = cursor.fetchall()
for r in result:
    print(r)
cursor.execute("SELECT * FROM employee")
print("\nfetch one:")
res = cursor.fetchone()
print(res)
```

If we run this program, saved as "sql_company_query.py", we get the following result, depending on the actual data:

```python
$ python3 sql_company_query.py
fetchall:
(1, 'William', 'Shakespeare', 'm', None, '1961-10-25')
(2, 'Frank', 'Schiller', 'm', None, '1955-08-17')
(3, 'Bill', 'Windows', 'm', None, '1963-11-29')
(4, 'Esther', 'Wall', 'm', None, '1991-05-11')
(5, 'Jane', 'Thunder', 'f', None, '1989-03-14')

fetch one:
(1, 'William', 'Shakespeare', 'm', None, '1961-10-25')
```



<BR><BR><BR>


## SQL Syntax and Functions Notes
### Queries for Data
***
For strings, some versions of SQL require single quotes.
Statements end in semi-colons.

```sql
SELECT [column] FROM [table]
WHERE [row entry/name] = ‘string’ or val
WHERE [col] LIKE ‘xxx%’    …xxx are characters, % is wildcard.  So ‘G%’ would choose all names starting with G
WHERE [col] IN (‘item A’, ‘item B’, ‘item C’)   …gives all matches listed in IN
WHERE [col] OR [col]
WHERE [col] AND [col]
WHERE [col] BETWEEN [val] AND [val]
```

Can do calculations in syntax (e.g. `SELECT gdp/pop FROM [table]` gives the gdp per person)
Can return length of entry with `SELECT LENGTH(row)`

#### WHERE


#### BETWEEN and IN


#### CASE:  
The `CASE` statement shown is used to substitute North America for Caribbean
`CASE` is used before the column of choice in this case, giving only the modified column and not the original

```sql
SELECT name, 		--[note the trailing comment before use of CASE]
CASE WHEN continent='Caribbean' THEN 'North America'
ELSE continent END
FROM world
```

#### OR:  
Note you have to write name of `column` (continent, here) and its definition for each `OR` when using `OR` statements.

```sql
WHEN continent = 'Europe' OR continent = 'Asia' THEN 'Eurasia'
```

Example with `CASE` and `OR`:

```sql
SELECT name,
CASE
WHEN continent = 'Europe' OR continent = 'Asia' THEN 'Eurasia'
WHEN continent = 'North America' OR continent = 'South America' OR continent = 'Caribbean' THEN 'America'
ELSE continent END
FROM world

WHERE name LIKE 'A%' OR name LIKE ('B%')
```

#### Exclusive OR (XOR):
Show the countries that are big by area or big by population but not both. Show name, population and area.
```sql
SELECT name, population, area FROM world
WHERE population > 250000000 XOR area > 3000000
```


Note that logic operators like `AND` and `OR` can be grouped via parenthesis to form a single argument, just like in normal code or math.

```sql
WHERE name = ‘joe’ AND age < 25 OR age > 40     
--is different than..
WHERE name = ‘joe’ AND (age < 25 OR age > 40)
```


#### DISTINCT and LIMIT:  
Used to return distinct results
```sql
SELECT DISTINCT City FROM Customers;
```




#### Select All Columns
There are two primary ways to select all columns from a table.
1. List the column name of each column.
2. Use the symbol \*.  
For example, to select all columns from `Store_Information`, we issue the following SQL:

```sql
SELECT * FROM Store_Information;
```


#### AGGREGATE FUNCTIONS
##### GROUP BY
```sql
AVG(), COUNT(), MAX(), MIN(), SUM() [FIRST(), LAST()]
```

If using the above funcs, must use `GROUP BY` at end of overall query with the columns queried for in the original `SELECT` query
i.e. Every field we `SELECT` needs to be either in the `GROUP BY` clause (which `name_length` is) or an aggregate computation

Example:

```sql
SELECT matchid, mdate, COUNT(teamid)
FROM game JOIN goal ON (game.id = goal.matchid)
WHERE teamid = 'GER'
GROUP BY matchid, mdate
```

##### HAVING



#### ORDER BY

<BR>

#### JOIN
You can combine two steps into a single query with a `JOIN`.

```sql
SELECT *
FROM game JOIN goal ON (game.id = goal.matchid)
```


The `FROM` clause says to merge data from the `goal` table with that from the `game` table. The ON says how to figure out which rows in `game` go with which rows in `goal` - the `id` from `goal` must match `matchid` from `game`.


##### Self JOIN
One 'trick' when performing joins is to do a _self join_.  In effect we create a copy of the given table and join one copy to the other in order to do certain operations to one while preserving the original information in the other, usually.  A basic example of why we'd do this is if we had a table of sales made and divisions they were in, and the managers of the various groups making sells in each division.  If we want to find the maximum amount sold by a group _per division_, and then also report the name of the manager for the group that was the highest, we can do a self join.

It would work roughly like this:
1. `SELECT` for the aggregated info you wanted, such as `MAX`(_sell_) value (aliased as "Max_Sell" or what have you) and the _division_, do a `GROUP BY` on _division_ (with this `SELECT` statement aliased as __t1__ or whatever alias you want for the first copy of the table).  This returns the `MAX` sell value per division.  This is good, but we also want to know which group had this maximum per division and who their manager was.  

2. So we perform a straightforward second `SELECT` of _group_ and _manager_ from the same table but alias it as __t2__, etc.

3. Now we `JOIN` these `SELECT` results from the same original table together on the columns from __t1__ only (i.e. the cols in __t1__ equal their counterpart in __t2__), forming a `SELF JOIN`.  We do not `JOIN` on the columns we selected from __t2__, since that was the whole point of using a Self `JOIN` to begin with -- to perform operations on copy of the table while also being able to select information from the original, un-operated-on table.

4. Last we make an overall "umbrella" `SELECT` statement from the now-joined tables, grabbing only the info we want.  Technically speaking, the `SELECT` statements in \#1 and \#2 are called _sub-queries_, but when I think of the process for doing a Self `JOIN`, I think of them as two independent selections we just join together and make a final selection from.

5. This example query would look like this:
    ```sql
    SELECT t1.division, t1.max_sell, t2.group, t2.manager
    FROM (
        SELECT division, max(sell) AS max_sell
        FROM div_sales
        GROUP BY division ) AS t1
    INNER JOIN (
        SELECT group, manager
        FROM div_sales ) AS t2
        ON t1.division = t2.division
        AND t1.max_sell = t2.sell
    ORDER BY t1.division
    ```

One important point to make is that when using a Self `JOIN` since we are using sub-queries (sub-`SELECT`) we _must_ return each as an alias (here, `t1` and `t2`) and enclose it in parenthesis.

Another important thing to note when using a Self `JOIN` is that it allows multiple answers of a same value.  So, in this case if multiple groups had sold the exact same amount within a division, the Self `JOIN` would return them all.  Window functions, shown below, do not.  They return only one result per value (which can be determined by order of the partition they use).

<BR>

#### Window Functions: OVER
Window functions are basically equivalent to `.transform()` in Pandas.  They require a function, which can be an aggregate function like `AVG()`, `RANK()`, or `SUM()` but doesn't have to be, and then apply that function to the groups specified in the table.  How the table is grouped into separate divisions is specified in the `PARTITION BY` list (this is technically optional, though very commonly used; without it, the entire table is used which sort of obviates the need for the window partition to begin with).  They can also aliased using the `WINDOW` clause if multiple Window Functions are needed in a single query.

Two good guides for Window Functions:
1. [PostgreSQL - Window Functions](https://www.postgresql.org/docs/9.1/static/tutorial-window.html)
2. [Mode Analytics - Window Function](https://community.modeanalytics.com/sql/tutorial/sql-window-functions/)

Here is how we would use a Window Function to achieve the same result we got in the Self JOIN query above:

1. `SELECT` the columns we want to return.

2. Use a sub-query in our `FROM` statement which is where we use our Window Function.

3. Choose the columns we want to be calculated for `OVER` the partitions. Here, _sell_, _group_, _manager_.

4. `PARTITION BY` these columns on _division_, ordering them in descending order.  Each window function must be returned as an alias we are selecting for in our outer `SELECT` statement (I believe).

5. Use `FIRST_VALUE()` to say we want the top value returned for our descending-sorted partitions.

6. `GROUP BY` all four columns in our `SELECT` statement.

7. Here's example code (have not tested this because this is a theoretical database; `jp_sql_practice.md` has real code used in FFT databases).

    ```sql
    SELECT division, sell AS max_sell, group, manager
    FROM (
        SELECT
        division,
        FIRST_VALUE(sell) OVER (PARTITION BY division ORDER BY sell DESC) AS sell,
        FIRST_VALUE(group) OVER (PARTITION BY division ORDER BY sell DESC) AS group
        FIRST_VALUE(manager) OVER (PARTITION BY division ORDER BY sell DESC) AS manager
        FROM div_sales
        ) AS sales
    GROUP BY division, sell, group, manager
    ORDER BY division;
    ```







#### UNION


#### ALIASES

#### EXISTS


<BR><BR>

### Creating and Modifying Tables
#### CREATE DATABASE

#### DROP DATABASE


#### CREATE TABLE


#### ALTER TABLE


#### DROP TABLE




#### UPDATE


#### DELETE















<BR><BR><BR>
<BR><BR><BR>

#### SQL ZOO EXAMPLES:
***
Show the countries which have a name that includes the word 'United'
```sql
SELECT name
FROM world
WHERE name LIKE 'United%'
```

Show the countries that are big by area or big by population. Show name, population and area.
```sql
SELECT name, population, area
FROM world
WHERE area > 3000000 OR population > 250000000
```

Exclusive OR (XOR).  
Show the countries that are big by area or big by population but not both. Show name, population and area.
```sql
SELECT name, population, area
FROM world
WHERE population > 250000000 XOR area > 3000000
```

Show the name and population in millions and the GDP in billions for the countries of the continent 'South America'. Use the `ROUND` function to show the values to two decimal places.
```sql
SELECT name, ROUND(population/1000000, 2), ROUND(gdp/1000000000, 2)
FROM world
WHERE continent = 'South America'
```

Show the name and per-capita GDP for those countries with a GDP of at least one trillion (1000000000000; that is 12 zeros). `ROUND` this value to the nearest $1000. (note negative decimals for `ROUND` func)
```sql
SELECT name, ROUND(gdp/population, -3)
FROM world
WHERE gdp >= 1000000000000
```


Show the name - but substitute Australasia for Oceania - for countries beginning with N. (Note the trailing comma after name!)
```sql
SELECT name,
CASE WHEN continent = 'Oceania' THEN 'Australasia'
ELSE continent END
FROM world
WHERE name LIKE 'N%'
```


Show the name - but substitute Eurasia for Europe and Asia; substitute America - for each country in North America or South America or Caribbean. Show countries beginning with A or B
```sql
SELECT name,
CASE WHEN continent = 'Asia' OR continent = 'Europe' THEN 'Eurasia'
WHEN continent = 'North America' OR continent = 'South America' or continent = 'Caribbean' THEN 'America'
ELSE continent END
FROM world
WHERE name LIKE 'A%' OR name LIKE 'B%'
```


List the winners, year and subject where the winner starts with Sir. Show the winners by name order.
```sql
SELECT *
FROM nobel
WHERE winner LIKE 'Sir%'
ORDER BY winner
```

List the winners, year and subject where the winner starts with Sir. Show the the most recent first, then by name order.
```sql
SELECT winner, yr, subject
FROM nobel
WHERE winner LIKE 'Sir%'
ORDER BY yr DESC, winner
```

The expression subject IN ('Chemistry','Physics') can be used as a value - it will be 0 or 1.
Show the 1984 winners and subject ordered by subject and winner name; but list Chemistry and Physics last.
```sql
SELECT winner, subject
FROM nobel
WHERE yr=1984
ORDER BY subject IN ('Chemistry', 'Physics'), subject, winner
```

Select the code which shows the years when a Medicine award was given but no Peace or Literature award was
```sql
SELECT DISTINCT yr
FROM nobel
WHERE subject='Medicine'
AND yr NOT IN
    (SELECT yr
    FROM nobel
    WHERE subject='Literature')
AND yr NOT IN
    (SELECT yr
    FROM nobel
    WHERE subject='Peace')
```


##### Subquery, DISTINCT
Pick the code that shows the amount of years where no Medicine awards were given
```sql
SELECT COUNT(DISTINCT yr)
FROM nobel
WHERE yr NOT IN
    (SELECT DISTINCT yr
    FROM nobel
    WHERE subject = 'Medicine')
```


##### Using Subqueries (nested SELECTs)
Select the code which would show the year when neither a Physics or Chemistry award was given
```sql
SELECT yr
FROM nobel
WHERE yr NOT IN
    (SELECT yr
    FROM nobel
    WHERE subject IN ('Chemistry','Physics'))
```

Show the population of China as a multiple of the population of the United Kingdom
```sql
SELECT population /
    (SELECT population
    FROM world
    WHERE name='United Kingdom')
FROM world
WHERE name = 'China'
```

Show each country that has a population greater than the population of ALL individual countries in Europe.
```sql
SELECT name
FROM world
WHERE population > ALL
    (SELECT population
    FROM world
    WHERE continent='Europe')
```

List each country name where the population is larger than that of 'Russia'.
```sql
SELECT name
FROM world
WHERE population >
    (SELECT population
    FROM world
    WHERE name = 'Russia')
```

List the name and continent of countries in the continents containing eitherArgentina or Australia. Order by name of the country.
```sql
SELECT name, continent
FROM world
WHERE continent IN
    (SELECT continent
    FROM world
    WHERE name = 'Argentina' OR name = 'Australia')
ORDER BY name
```


Germany (population 80 million) has the largest population of the countries in Europe. Austria (population 8.5 million) has 11% of the population of Germany.
Show the name and the population of each country in Europe. Show the population as a percentage of the population of Germany.
```sql
SELECT name, CONCAT(ROUND(population /
    (SELECT population
    FROM world
    WHERE name = 'Germany') * 100, 0), '%')
FROM world
WHERE continent = 'Europe'
```

For each continent show the continent and number of countries.
```sql
SELECT continent, COUNT(name)
FROM world
GROUP BY continent
```

List the continents that have a total population of at least 100 million.  (`HAVING` always comes after `GROUP BY`)
```sql
SELECT continent
FROM world
GROUP BY continent
HAVING SUM(population) > 100000000
```

Show the player, teamid, stadium and mdate and for every German goal.
```sql
SELECT player, teamid, stadium, mdate
FROM game
JOIN goal ON (game.id = goal.matchid)
WHERE teamid = "GER"
```


Show the name of all players who scored a goal against Germany.
```sql
SELECT DISTINCT player
FROM goal
JOIN game ON (game.id = goal.matchid)
WHERE (team1 = 'GER' OR team2 = 'GER') AND teamid NOT IN ('GER')
```


Show teamname and the total number of goals scored.
```sql
SELECT teamname AS 'Country', COUNT(player) AS 'Goals Scored'
FROM goal
JOIN eteam ON (goal.teamid = eteam.id)
GROUP BY teamname
```

For every match involving 'POL', show the matchid, date and the number of goals scored.
```sql
SELECT matchid, mdate, COUNT(matchid)
FROM game
JOIN goal ON (game.id = goal.matchid)
WHERE team1 = 'POL' OR team2 = 'POL'
GROUP BY matchid, mdate
```


List every match with the goals scored by each team as shown. This will use `CASE WHEN` which has not been explained in any previous exercises.
```html
        mdate       team1   score1  team2   score2
1       July 2012	ESP     4	    ITA	    0
10      June 2012	ESP	    1	    ITA	    1
10      June 2012	IRL	    1	    CRO	    3
...
```

Notice in the query given every goal is listed. If it was a `team1` goal then a 1 appears in `score1`, otherwise there is a 0. You could `SUM` this column to get a count of the goals scored by `team1`. Sort your result by `mdate`, `matchid`, `team1` and `team2`.

```sql
SELECT mdate, team1,
SUM(CASE WHEN goal.teamid = game.team1 THEN 1 ELSE 0 END) Score1, team2,
SUM(CASE WHEN goal.teamid = game.team2 THEN 1 ELSE 0 END) Score2
FROM game
LEFT JOIN goal ON (game.id = goal.matchid)
GROUP BY mdate, matchid, team1, team2
```


Which were the busiest years for 'John Travolta', show the year and the number of movies he made each year for any year in which he made more than 2 movies.
```sql
SELECT yr, COUNT(title)
FROM movie
JOIN casting ON (movie.id = movieid)
JOIN actor ON (actorid = actor.id)
WHERE name = 'John Travolta'
GROUP BY yr

HAVING COUNT(title) =
    (SELECT MAX(c)
    FROM
        (SELECT yr, COUNT(title) AS c
        FROM movie
        JOIN casting ON movie.id=movieid
        JOIN actor ON actorid=actor.id
        WHERE name='John Travolta'
        GROUP BY yr) AS t
    )
```


List the film title and the leading actor for all of the films 'Julie Andrews' played in.
```sql
SELECT DISTINCT title, name
FROM movie
JOIN casting ON (casting.movieid = movie.id)
JOIN actor ON (casting.actorid = actor.id)
WHERE movieid IN
    (SELECT movieid
    FROM casting
    WHERE actorid IN
        (SELECT id
        FROM actor
        WHERE name='Julie Andrews')
    )
AND ord=1
```


Obtain a list, in alphabetical order, of actors who've had at least 30 starring roles.
```sql
SELECT name
FROM movie
JOIN casting ON (casting.movieid = movie.id)
JOIN actor ON (casting.actorid = actor.id)
WHERE ord=1
GROUP BY name
HAVING COUNT(name) >= 30
```


List the films released in the year 1978 ordered by the number of actors in the cast.
```sql
SELECT title, COUNT(actorid)
FROM movie
JOIN casting ON (casting.movieid = movie.id)
JOIN actor ON (casting.actorid = actor.id)
WHERE yr = 1978
GROUP BY title
ORDER BY COUNT(actorid) DESC
```

List all the people who have worked with 'Art Garfunkel'.
```sql
SELECT actor.name
FROM movie JOIN casting ON (casting.movieid = movie.id)
JOIN actor ON (casting.actorid = actor.id)
WHERE actor.name !='Art Garfunkel' AND movie.id IN
    (SELECT movie.id
    FROM movie
    JOIN casting ON (casting.movieid = movie.id)
    JOIN actor ON (casting.actorid = actor.id)
    WHERE actor.name='Art Garfunkel')
```
