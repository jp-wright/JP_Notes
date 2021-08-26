# JP Postgres Notes

### Quick interjection
Good web articles on how to use PostgreSQL with pgAdmin4:

[The best](https://www.codementor.io/@engineerapart/getting-started-with-postgresql-on-mac-osx-are8jcopb)
[Using pgAdmin4 to connect to PostgreSQL](https://www.enterprisedb.com/postgres-tutorials/connecting-postgresql-using-psql-and-pgadmin)







__PostgreSQL__ is a full-featured open source SQL-based database management system, if you will.  It can directly interface with Python using the __psycopg2__ library (yay!).  This means we can import or create a database in Postgres and access it as-is through Python, as shown in [this short tutorial](https://wiki.postgresql.org/wiki/Psycopg2_Tutorial), which is a huge convenience.  

On its own, Postgres has two interfaces, one in the terminal, called __psql__, and a GUI called __pgAdmin4__.  The notes below cover psql first.  I have yet to learn pgAdmin4, so that will follow later.  To my understanding, both have the same capabilities.  

I have a book in my _Python Books_ directory (/Users/jpw/Dropbox/Data_Science/Python/Python_Books) called __PostgreSQL: Up and Running__ that covers all the info needed to effectively use Postgres.  I haven't churned through it yet, so what follows below is a barebones "getting started" guide.  Postgres itself is a separate local server application from psql and pgAdmin4 and must be running (in the background, denoted by an elephant icon in the menubar) for either of those two to work.  In practice, it is common to use these terms interchangeably, where Postgres means psql, etc.

<BR>

## Postgres and PSQL
Note that **Homebrew** and **Postgres** are *not* part of the Anaconda distribution, so you will have to install these independently using **brew** even if you are using an Anaconda distribution of Python.

### Installing Postgres
*JP Note*: According to an interview, I think Postgres is available via **Conda** (it may not be in the main distribution)

1. Install **homebrew** and **brew cask**.
    + Install homebrew. Instructions [here](http://brew.sh/)
    + Install brew cask with: `brew install caskroom/cask/brew-cask`
    + If you already have them, update: `brew update && brew upgrade brew-cask`

2. Install your **Postgres** database.
    + The easiest way is to install the pre-built application (with an elephant icon) using the following command: `brew cask install postgres`

After the installation is complete, use *Spotlight* to search for `postgres` and open the application. It will ask you if you want to move it to the Applications folder, say "Move it"

<BR>
<BR>

### Setting up PSQL on a Mac
1. In terminal go to your home directory (`cd ~/`)

2. Open terminal configurations:
    + `atom ~/.bash_profile` (This will vary somewhat based on OS. If `.bashrc` or `.profile` exist, edit those rather than using `.bash_profile`, which creates a new file).
    + If you are using **zsh**: `atom .zshrc` (if you are on your personal computer and don't know what this is, you are probably using bash in \#1).

3. Insert the following line at the end of the file and save the file: <br>
    `export PATH=/Applications/Postgres.app/Contents/Versions/latest/bin:$PATH`

4. Open a new terminal and run `psql`

<BR>
<BR>

### Using Postgres (see [documentation](http://www.postgresql.org/docs/9.3/interactive/)):
For a brief walkthrough, follow instructions in `/DSI_Camp/Week1/SQL/individual.md` and/or ...`/pair.md`.  Some of the below steps are culled from these files.  The documentation on the Postgres website above is really excellent.  There is also a __pyscopg2__ assignment in there.  In general, using psycopg2 is likely the best option if you know you're going to be operating primarily in Python.  However, databases are used by all languages and frameworks in some extent or another, and Postgres as a standalone application allows us to do everything needed to make data accessible for any application.  The instructions here are for using Postgres in this application-agnostic way.  Psycopg2 instructions will follow further below.

Overall, the big picture is relatively the same for using Postgres to construct, manipulate, and return databases and their data for use in other applications like the backend of a website or dynamic queries for your company or clients.  It commonly follows some pattern like this:

1. Have a spreadsheet (`.csv` usually) of data.
2. Import .`csv` into a Postgres database.
3. Manipulate, trim, or sub-select from this database.
4. Export this desired data into an application-agnostic database (commonly `.db` file).
5. Use this exported `.db` file in your website or client-relations, etc.

These steps are covered below.  As noted above, if you're using Python you can simplify this process by using psycopg2 in your Python scripts in lieu of messing with exporting a Postgress database to a `.db` file.  

<BR>

#### Creating a Database:  
There are two common ways that databases are created in Postgres.
1. Import a __.sql__ file
2. Import a __.csv__ file

We can also export any database we are working with in Postgres to a .sql file or a .db file using `pg_dump [postgres database name] > new_database_name[.sql/.db]`.  The export will take place in the current working directory.  This is covered further below in these notes.

1. __Import a .sql file__  
    psql allows you to interface with an existing .sql file to create a new database.  Commonly we will use psql to investigate, modify, and return certain information from an existing .sql database.  In these cases we have to create an empty psql database first and then populate it with the existing .sql database.  The instructions below are written for this scenario.  

    1. Open Postgres by doing a command-space search for Postgres or finding it in the `/applications/` folder.  In terminal, type `psql` to begin a psql session.  Opening psql without having Postgres running will fail to open psql.  (Also, you can click the Postgres elephant menubar icon and open psql from there).

    2. From the command line run `psql`.  Once in psql, run this command to create the database.

        ```sql
        CREATE DATABASE database_name;
        \q      -- quits psql upon creation of db
        ```

    3. Navigate in terminal to where your .sql database exists on your hard drive and run the following commands to import the database:

        ```bash
        cd <wherever .sql file is>
        psql -U postgres database_name < database_name.sql
        ```

        You should see a bunch of SQL commands flow through the terminal.
        You don't have to name the psql database name the same as the .sql database name -- it can be anything you want.  The `-U` modifier grants access to the database to the current user.  It isn't necessary but is smart in case of other use cases down the road.

    4. To use the interactive Postgres prompt enter the following to be connected to your database in the terminal.

        ```bash
        psql -U postgres database_name
        ```

    5. After you are done with all your database work in psql, use `\q` to exit.

    Assuming you didn't `\q` quit, we are now in a command line client. This is how we will explore the database to gain an understanding of what data we are playing with.

<BR>

2. __Import a .csv file__  
    We can import a .csv file directly into a psql database by first creating an empty database, creating an empty table in that database, and then copying the .csv file data directly into this table.  See the official [Postgres documentation](https://www.postgresql.org/docs/9.2/static/sql-copy.html) or a good [summary how-to guide](http://www.postgresqltutorial.com/import-csv-file-into-posgresql-table/).

    1. Create an empty database in psql just like we did for the .sql file above.  (As always, you can import data into an existing Postgres database, of course.  This just covers how to do it from scratch).  We'll use a made up "football" database.

        ```sql
        CREATE DATABASE football_db;
        \q      -- quits psql upon creation of db.  Don't do this if you use #1 below.
        ```

    2. Create an empty table to import the .csv data into.  One table per .csv file.  When we create a new table we have to name it ("nfl_table"), list each column and the data type of the column (consult my `jp_sql_notes.md` file for more detail on this).  Note I believe that the column names we want to fill with data here must match the column names in the .csv file, since they will be imported directly.  I'm not positive of this, but I've never seen it done differently.  Continuing with the football database, here's an example table we'd see.  

        ```sql
        DROP TABLE IF EXISTS nfl_table;
        CREATE TABLE nfl_table (
            ID              SERIAL NOT NULL PRIMARY KEY,
            Year            INT,
            Num_Teams       INT,
            Playoff_Pos     INT,
            SeasonRk        INT,
            Team            VARCHAR(255),
            Owner           VARCHAR(255)
        );
        ```

        Note the "ID" column which is a `SERIAL` datatype and serves as a unique identifier for each row. This simply starts at "1" and counts up for each row.  In Postgres it adds a bit of coding overhead to the import/export log but functions as expected.

        There are now two options to actually create the table to insert data into.

        1. __In psql terminal:__  
            Create the target table from inside psql by entering the SQL statements directly into the terminal.  This requires multiple steps and can be hard to verify and edit should editing be needed.  It will also have to be redone every time you wish to import this same data in the future.  So, do not `\q` out of Postgres.

        2. __From .sql file:__  
            _This is certainly the preferred method_.  Import the command to create a table as well as import all the data in one go from an external .sql file.  We can reuse this file if ever needed as well.    We will need to `\q` out of Postgres.

        The SQL and Postgres commands shown below in \#2 and \#3 will be used for either method of importing.  If you are using the "in terminal" method above, then you'd simply enter these commands into the psql terminal.  

        If you are using the smarter ".sql file" method, then you'd make a new `.sql` file in a text editor with these commands in it, save it (say, as `football_data.sql`), and import this file in one go with the following command in the bash terminal (_not_ in the psql terminal):  

        `psql -U postgres football_db < football_data.sql`


    3. Import the .csv data using the `COPY` command.  We name the table we are copying into ("nfl_table") and the columns we are inserting into.  Then we specify where the `.csv` file we are copying from is located, that its delimiter is a comma (we'd use a tab or semi-colon or whatever if the file used that instead), and that we'd like to use the `.csv`'s headers.

        ```sql
        COPY nfl_table (Year, Num_Teams, Playoff_Pos, SeasonRk, Team, Owner)
        FROM '<directory of csv file/filename.csv>' DELIMITER ',' CSV HEADER;
        ```

        So, for what I consider the most sensible and reusable approach, we'd have a single .sql file created in a text editor with the following in it:

        ```sql
        DROP TABLE IF EXISTS nfl_table;     -- removes old table if exists; can't insert data if old table exists
        CREATE TABLE nfl_table (            -- create the new table we are inserting into
            ID              SERIAL NOT NULL PRIMARY KEY,
            Year            INT,
            Num_Teams       INT,
            Playoff_Pos     INT,
            SeasonRk        INT,
            Team            VARCHAR(255),
            Owner           VARCHAR(255)
        );

        COPY nfl_table (Year, Num_Teams, Playoff_Pos, SeasonRk, Team, Owner)
        FROM '<directory of csv file/some_nfl_data.csv>' DELIMITER ',' CSV HEADER;
        ```

        And that's it.  

<BR>
<BR>

#### Exporting a Database:
Postgres has a very versatile bash command line (_not_ in psql terminal) export function, `pg_dump`.  By default using `pg_dump` will create the dumped database file in the current working directory.  It will also export the data in a way that is meant to be restored into a Postgres database specifically (i.e. not useable by other programs nor ready to `INSERT INTO` other .db files).  [Here is the documentation](https://www.postgresql.org/docs/9.6/static/app-pgdump.html).  A quick look shows there are a lot of optional commands to choose from.  I'll cover some of the more useful cases below.

1. `--data-only`  
    > Dump only the data, not the schema (data definitions). Table data, large objects, and sequence values are dumped.

    This is probably the most common non-Postgres use.  We want to take the data in a Postgres database (usually a specific table) and export it for another application or database usage.  This command will give just the data and not the table schema.

2. `--column-inserts`  
    > Dump data as INSERT commands with explicit column names (INSERT INTO table (column, ...) VALUES ...). This will make restoration very slow; it is mainly useful for making dumps that can be loaded into non-PostgreSQL databases. However, since this option generates a separate command for each row, an error in reloading a row causes only that row to be lost rather than the entire table contents.

    This option allows us to export our table data as `INSERT INTO` SQL commands just like if we were using `CREATE TABLE` in SQL and wanted to fill our table with data.  This is how we will use Postgres to prepare data for use in a .db file on server backends if needed.

3. `--table=<table>`  
    > Dump only tables with names matching table. For this purpose, "table" includes views, materialized views, sequences, and foreign tables. Multiple tables can be selected by writing multiple -t switches. Also, the table parameter is interpreted as a pattern according to the same rules used by psql's \d commands (see Patterns), so multiple tables can also be selected by writing wildcard characters in the pattern. When using wildcards, be careful to quote the pattern if needed to prevent the shell from expanding the wildcards; see Examples.
    >
    > The -n and -N switches have no effect when -t is used, because tables selected by -t will be dumped regardless of those switches, and non-table objects will not be dumped.

    This option specifies the table we want to export from the database.  Multiple tables can be listed.

Here's an example of how to use these arguments together to create a .sql file that has data only and the accompanying `INSERT INTO` statements (note any file extension can be used, technically, but `.sql` is the most common and the one we want here, since we are exporting a SQL insert data command):  

`pg_dump --table=nfl_table --data-only --column-inserts football_db > nfl_data.sql`

The key point here is that we now have a .sql file with all the _actual data_ (and accompanying SQL insert statements) in it.  Previously we only had a .sql file with the Postgres `COPY` method linking to a .csv file, which is unusable by anything but Postgres.

This export .sql file will usually have some Postgres-specific `SET` commands at the top which require you to open the file in a text editor and comment out or delete them to be usable in other databases.  If you have a `SERIAL` column, like an "ID" column, then Postgres might also include the following line in your exported .sql file which you'll likely have to comment out / erase:

```sql
SELECT pg_catalog.setval('vtown_id_seq', 42, true);
```

Finally, in order to achieve our goal to have a `.db` SQLite database (different from Postgres database!) in which to use with any number of languages or web frameworks, we can create it using SQLite3's command-line creation function utilizing our newly exported .sql data-only insert file.

`sqlite3 nfl.db < nfl_data.sql`  

 _Note:_ this assumes the `.db` database already has the matching table, `nfl_table` in this case, inside of it and ready to be inserted into.  If it doesn't you'll receive an error and need to add that table to it.  There are a couple of ways to do this, all of which involve using standard SQL `CREATE TABLE` syntax.  We'll use our football database example to demonstrate this now.

<BR>

#### Creating a .db File For Our Postgres Data
The two primary methods of doing this are detailed below.  Also, as always, if the table already exists in the `.db` file, we will need to drop it before we can re-create it below (if we want to start fresh; if we simply want to update it by adding new data, that is a different task) by using `DROP TABLE IF EXISTS <table name>` before the following commands to create it.

1. __Create it at the command line using `SQLite3`.__  
    This is the simplest way, but might actually be burdensome for large datasets.  Perform the following steps in the terminal.
    1. `sqlite3 nfl.db`  
        This loads the nfl.db database into sqlite3 in the terminal.  If no nfl.db file exists, it creates it anew, empty.

    2. `CREATE TABLE nfl_table (<columns and data types...>);`  
        That's it.  Note two things:
        1. The table name we create here has to match exactly the table name in the `nfl_data.sql` file we made above by using `pg_dump`, otherwise SQLite3 won't understand which table to insert the data into.

        2. Like all SQL commands the `CREATE TABLE` command won't be completed until we enter a semi-colon `;` into the terminal.  This means we can open the parenthesis and paste our columns and data types directly into the terminal if we have them elsewhere in a different document.  

        Once done use `<CTRL>-D` to exit SQLite3.  The entire creation example is shown below.

    ```bash
    $ sqlite3 nfl.db                            <-- Enter or create the database
    SQLite version 3.13.0 2016-05-18 10:57:30
    Enter ".help" for usage hints.
    sqlite> CREATE TABLE nfl_table (            <-- start the creation process, press <return>
    ...> ID             INT PRIMARY KEY,        <-- paste/write the table data
    ...> Year           INT,
    ...> Num_Teams      INT,
    ...> Playoff_Pos    INT,
    ...> SeasonRk       INT,
    ...> Team           VARCHAR(255),
    ...> Owner          VARCHAR(255)
    ...> );                                     <-- commit the created table
    ```

    Our `nfl.db` database file will now have an empty table called `nfl_table` in it and will be ready to have the data in our `nfl_data.sql` file inserted into it when we use the `sqlite3 nfl.db < nfl_data.sql` command.  Party.

2. __Create it inside the .sql file we exported from Postgres.__  
    We essentially do the same thing we did in \#1 except we add it to the .sql file we created via the `pg_dump` command earlier.  This way all the commands we need are included in a single file.  There is a benefit and a drawback to this approach.  

    + Benefit: consolidation of commands and data.  Don't have to enter anything on the command line each time we want to make a new database of this data.  It's already taken care of in the .sql file.

    + Downside: If you have a database that already has this table in it and you merely want to add this data to that table, depending on the applications you're using (and how they process a given flavor of SQL) you'll encounter either an error or could overwrite the existing table.  

    Thus this approach is best if you are going to be using this data in all-new databases.  As a side note, you can always simply comment out (`/* <text> */`) the `CREATE TABLE` statement in the .sql file.  To add the actual statement, open the .sql file you made with `pg_dump` (such as `nfl_data.sql`) in a text editor and add it before the `INSERT INTO ()` statements.  The statement itself will be identical to the `CREATE TABLE` statement we used in Postgres itself when we imported the .csv file into our Postgres database; as such we can just copy and paste from that file.

    ```sql
    CREATE TABLE nfl_table (
        ID              INT PRIMARY KEY,
        Year            INT,
        Num_Teams       INT,
        Playoff_Pos     INT,
        SeasonRk        INT,
        Team            VARCHAR(255),
        Owner           VARCHAR(255)
    );
    ```

    Now when we use `sqlite3 nfl.db < nfl_data.sql` to create our `nfl.db` file the data from `nfl_data.sql` will be automatically inserted into it since the `CREATE TABLE` and `INSERT INTO` statements are both in the `nfl_data.sql` file.  

<BR>
<BR>

#### Basic Exploration in Postgres (From DSI)
First, we will want to see what the available tables are. Remember, now that we are in the database, all of our commands or actions will be on the database you created `database_name`.  Please see below for a full list of PSQL commands.

1. What are the tables in our database? Run `\d` to find out.
2. What columns does each table have? Run `\d <tablename>` to find out.
3. `q` will end any query, but not the program (`\q` ends the program)
4. **ALL** queries must end in a `;`

<BR>
<BR>

#### Create a Temporary Table
To create a temporary table, do the following:
```sql
SELECT userid, date_7d
INTO TEMPORARY TABLE logins_mob
FROM logins_7d JOIN logins ON logins_7d.userid = logins.userid
WHERE logins.type = "mobile"
```

...or what have you. Enter this *into psql terminal*, where it will live as long as the session is open, allowing you to reference it as you need, including creating permanent tables from info taken from the temp tables.

<BR>

### psycopg2
##### Editing SQL within Python via 'Pipelines'
You will edit stuff all you want in psql terminal and then once ready you will enter it into terminal (or IPython).  You must create a connection using psycopg2, then a cursor for that connection.  You must enclose the SQL query as a `cursor.execute()` argument, then once done you have to actually `commit()` the query before the actual database will be modified.  At the end you must close the connection.   If you have *ANY ERROR AT ALL* in any `c.execute()` command/query, you MUST use `conn.rollback()` to clear the cursor, as it has been 'tainted' by the error.

See "**postgres_script.py**" and **/sql-python/individual.md** for more info.

```python
import psycopg2
from datetime import datetime

conn = psycopg2.connect(dbname='socialmedia', user='postgres', host='/tmp')
c = conn.cursor()

today = '2014-08-14'

timestamp = datetime.strptime(today, '%Y-%m-%d').strftime("%Y%m%d")

c.execute(
    '''
    CREATE TABLE logins_7d AS
    SELECT
        userid,
        COUNT(*) AS cnt,
        type,
        timestamp %(timestamp)s AS date_7d
    FROM
        logins
    WHERE
        logins.tmstmp > timestamp %(timestamp)s - interval '7 days'
    GROUP BY
        userid;
    ''',
    {'timestamp': timestamp}
    )

conn.commit()
conn.close()
```
<BR>
<BR>
<BR>

### PSQL Commands
***
##### GENERAL OPTIONS:
Command | Action
--------|-------
`-c COMMAND` | run only single command (SQL or internal) and exit
`-d, --dbname=NAME` | specify database name to connect to (default: "logged in username here")
`-f, --file=FILENAME` | execute commands from file, then exit
`--help` | show this help, then exit
`-l, --list` | list available databases, then exit
`-v NAME=VALUE` | set psql variable NAME to VALUE
`--version` | output version information, then exit
`-X` | do not read startup file (~/.psqlrc)
 <BR>

##### TYPES:
Command | Action
--------|-------
`\h` | for help with SQL commands
`\?` | for help with psql commands
`\g` | or terminate with semicolon (`;`) to execute query
`\q` | to quit
<BR>

##### GENERAL:
Command | Action
--------|-------
`\c[onnect] [DBNAME\|- USER\|- HOST\|- PORT\|-] |` | connect to new database
`\cd [DIR]` | change the current working directory
`\encoding [ENCODING]` | show or set client encoding
`\h [NAME]` | help on syntax of SQL commands, `*` for all commands
`\set [NAME [VALUE]]` | set internal variable, or list all if no parameters
`\timing` | toggle timing of commands (currently off)
`\unset NAME` | unset (delete) internal variable
`\prompt [TEXT] NAME` | prompt user to set internal variable
`\! [COMMAND]` | execute command in shell or start interactive shell
<BR>

##### QUERY BUFFER:
Command | Action
--------|-------
`\e [FILE]` | edit the query buffer (or file) with external editor
`\g [FILE]` | send query buffer to server (and results to file or `|pipe`)
`\p` | show the contents of the query buffer
`\r` | reset (clear) the query buffer
`\w FILE` | write query buffer to file
<BR>

##### INPUT/OUTPUT:
Command | Action
--------|-------
`\echo [STRING]` | write string to standard output
`\i FILE` | execute commands from file
`\o [FILE]` | send all query results to file or `|pipe`
`\qecho [STRING]` | write string to query output stream (see `\o`)
<BR>

##### INFORMATIONAL:
Command | Action
--------|-------
`\d [NAME]` | describe table, index, sequence, or view
`\d{t|i|s|v|S} [PATTERN]` (add `+` for more detail) | list tables/indexes/sequences/views/system tables
`\da [PATTERN]` | list aggregate functions
`\db [PATTERN]` | list tablespaces (add `+`)
`\dc [PATTERN]` | list conversions
`\dC` | list casts
`\dd [PATTERN]` | show comment for object
`\dD [PATTERN]` | list domains
`\df [PATTERN]` | list functions (add `+`)
`\dF [PATTERN]` | list text search configurations (add `+`)
`\dFd [PATTERN]` | list text search dictionaries (add `+`)
`\dFt [PATTERN]` | list text search templates
`\dFp [PATTERN]` | list text search parsers (add `+`)
`\dg [PATTERN]` | list groups
`\dn [PATTERN]` | list schemas (add `+`)
`\do [NAME]` | list operators
`\dl` | list large objects (same as `\lo_list`)
`\dp [PATTERN]` | list table, view, and sequence access privileges
`\dT [PATTERN]` | list data types (add `+`)
`\du [PATTERN]` | list users
`\l` | list all databases (add `+`)
`\z [PATTERN]` | list table, view, and sequence access privileges (same as `\dp`)
<BR>

##### FORMATTING:
Command | Action
--------|-------
`\a` | toggle between unaligned and aligned output mode
`\C [STRING]` | set table title, or unset if none
`\f [STRING]` | show or set field separator for unaligned query output
`\H` | toggle HTML output mode (currently off)
`\pset NAME [VALUE]` | set table output option: (NAME := {format \| border \| expanded \| fieldsep \| footer \| null \| numericlocale \| recordsep \| tuples_only \| title \| tableattr \| pager})
`\t` | show only rows (currently off)
`\T [STRING]` | set HTML <table> tag attributes, or unset if none
`\x` | toggle expanded output (currently off)
<BR>

##### COPY, LARGE OBJECT:
Command | Action
--------|-------
`\copy ...` | perform SQL COPY with data stream to the client host
`\lo_export LOBOID FILE` | LOBOID FILE
`\lo_import FILE [COMMENT]` | FILE [COMMENT]
`\lo_list` | list
`\lo_unlink LOBOID` | large object operations
<BR>

##### Connection Options:
Command | Action
--------|-------
`-h, --host=HOSTNAME` | database server host or socket directory
`-p, --port=PORT `| database server port number
`-U, --username=NAME` | connect as specified database user
`-W, --password` | force password prompt (should happen automatically)
`-e, --exit-on-error` | exit on error, default is to continue
`-d DBNAME` | some database


<BR><BR><BR>











***
## From DSI Lecture

You work at a social media site. You have to write a lot of queries to get the desired data and stats from the database.

You will be working with the following tables:

* `registrations`: One entry per user when the register
* `logins`: One entry every time a user logs in
* `optout`: A list of users which have opted out of receiving email
* `friends`: All friend connections. Note that while friend connections are mutual, sometimes this table has both directions and sometimes it doesn't. You should assume that if A is friends with B, B is friends with A even if both directions aren't in the table.
* `messages`: All messages users have sent
* `test_group`: A table for an A/B test

## Connecting to the database

There's a sql dump of the database in `socialmedia.sql`.

Here are the steps to load it up:

1. First create the database by running `psql` and then the following command:

    ```sql
    CREATE DATABASE socialmedia;
    ```
    Use `\q` to quit `psql`.

2. You can load the sql dump with this command on the command line:

    ```shell
    psql -U postgres socialmedia < data/socialmedia.sql
    ```

3. Finally, to run the database:

    ```shell
    psql -U postgres socialmedia
    ```

If you're running into issues, make sure you're running `postgres.app`.


## Investigate your database

You can get a list of all the tables with this command: `\d`

You can get the info for a specific table with this command: `\d friends`

Start by looking at your tables. Even if someone tells you what a table is, you should look at it to verify that is what you expect. Many times they are not properly documented. You might wonder what the `type` field is. Well, take a look at the table:

```sql
SELECT * FROM registrations LIMIT 10;
```

It's also good to get an idea of all the possible values and the distribution:

```sql
SELECT type, COUNT(*) FROM registrations GROUP BY type;
```

Look at some rows from every table to get an idea of what you're dealing with.

## Tips:
* Take a look at the postgres [datetime documentation](http://www.postgresql.org/docs/8.4/static/datatype-datetime.html) if you're unsure how to work with the dates.
* Try and get practice adding newlines and whitespace so that the queries are easily readable.
* Sometimes it may be useful to create temporary tables like this: `CREATE TEMPORARY TABLE tmp_table AS SELECT ...` or use a [with query](http://www.postgresql.org/docs/9.1/static/queries-with.html).

### Questions


1. Get the number of users who have registered each day, ordered by date.

    ```sql
    SELECT tmstmp::DATE as reg_date, COUNT(*)
    FROM registrations
    GROUP BY reg_date
    ORDER BY reg_date;
    ```

2. Which day of the week gets the most registrations?

    ```sql
    WITH day_registrations as (
        SELECT date_part('dow', tmstmp) as weekday, COUNT(*) as reg_count
        FROM registrations
        GROUP BY weekday
        ORDER BY weekday)

    SELECT CASE
    WHEN weekday = '0' THEN 'Sunday'
    WHEN weekday = '1' THEN 'Monday'
    WHEN weekday = '2' THEN 'Tuesday'
    WHEN weekday = '3' THEN 'Wednesday'
    WHEN weekday = '4' THEN 'Thursday'
    WHEN weekday = '5' THEN 'Friday'
    ELSE 'Saturday' END, reg_count
    FROM day_registrations;
    ```

3. You are sending an email to users who haven't logged in in the week before '2014-08-14' and have not opted out of receiving email. Write a query to select these users.

    ```sql
    SELECT reg.userid
    FROM registrations AS reg
    WHERE reg.userid NOT IN (
        SELECT DISTINCT userid
        FROM logins
        WHERE tmstmp > '2014-08-07'::DATE
        GROUP BY userid)
    AND reg.userid NOT IN (
        SELECT DISTINCT userid
        FROM optout);
    ```

4. For every user, get the number of users who registered on the same day as them. Hint: This is a self join (join the registrations table with itself).

    `Solution 1:`
    ```sql
    WITH dailyr as (
        SELECT tmstmp::DATE as reg_date, COUNT(*) as reg_count
        FROM registrations
        GROUP BY reg_date
        ORDER BY reg_date
    )
    SELECT registrations.userid, dailyr.reg_count - 1 as cnt
    FROM registrations, dailyr
    WHERE registrations.tmstmp::DATE = dailyr.reg_date
    ORDER BY registrations.userid;
    ```
    `Solution 2:`
    ```sql
    SELECT r.userid, count(rr.userid)
    FROM registrations r
    JOIN registrations rr
    ON r.tmstmp::DATE = rr.tmstmp::DATE
    AND r.userid <> rr.userid
    GROUP BY r.userid
    ORDER BY r.userid;
    ```

5. You are running an A/B test and would like to target users who have logged in on mobile more times than web. You should only target users in test group A. Write a query to get all the targeted users.
    ```sql
    SELECT m_users.userid, tg.grp
    FROM (SELECT ml.userid
    FROM
        (SELECT userid, COUNT(type) as mobile_logins
        FROM logins
        WHERE type = 'mobile'
        GROUP BY userid, type
        ORDER BY userid) ml,
        (SELECT userid, COUNT(type) as web_logins
        FROM logins
        WHERE type = 'web'
        GROUP BY userid, type
        ORDER BY userid) wl
    WHERE ml.userid = wl.userid
    AND mobile_logins > web_logins) m_users, test_group tg
    WHERE m_users.userid = tg.userid
    AND tg.grp = 'A';
    ```
6. You would like to determine each user's most communicated with user. For each user, determine the user they exchange the most messages with (outgoing plus incoming).

```sql
    WITH user_msgs as (
        SELECT m1.sender, m1.recipient, COUNT(*) + m2.msg_rec AS num_msgs
        FROM messages m1,
            (SELECT sender, recipient, COUNT(*) msg_rec
            FROM messages
            GROUP BY sender, recipient
            ORDER BY sender, recipient) m2
            WHERE m1.sender = m2.recipient AND m2.sender = m1.recipient
        GROUP BY m1.sender, m1.recipient, m2.msg_rec
        ORDER BY m1.sender, m1.recipient),
    max_per_user as (
        SELECT um1.sender AS user1, MAX(um1.num_msgs) as msgs
        FROM user_msgs as um1
        JOIN user_msgs as um2
        ON um1.sender = um2.sender AND um1.recipient = um2.recipient
        GROUP BY user1)

    select um.sender as user1, um.recipient as user2, mpu.msgs
    from user_msgs as um, max_per_user as mpu
    where um.num_msgs = mpu.msgs and um.sender = mpu.user1
    order by user1, user2;

-- AND (m1.sender = '0' AND m1.recipient = '17'
-- OR m1.sender = '17' AND m1.recipient = '0')
```
***
















<BR>
<BR>
<BR>


#### ~~Important Note About Using "DB Browser for SQLite" to Export SQL DB~~
#### UPDATE: Never use this POS app.  Instead use `COPY` in Postgres. Detailed above.__
Steps to using DB Browser in order to make a SQL DB for Postgres
1. In DB Browser __Create New Database__, save to local.
2. Once in this new DB, go to __File__ and __Import Table From CSV File__.
3. It will create a poorly constructed table from this CSV file.
4. Go to __File__ and __Export Database as SQL DB__, save to local as [exported db filename].
    + Select table(s) you want to export
    + Check YES to "Keep column names in INSERT INTO"
    + Check YES to "Multiple rows (VALUES) per INSERT statement"
    + Export Everything
5. In terminal navigate to location of saved SB.
6. Enter Postgres by entering `psql`
7. Create new Postgres DB with `CREATE DATABASE [db name]`
8. Exit Postgres with `\q`
9. Load exported DB into Postgres with `psql -U postgres [db name] < [exported db file]`
10. Enter Postgres to explore loaded DB with `psql -U postgres [db name]`


#### There is a MAJOR catch
The DB Browser app does not do a good job of exporting a table to a SQL DB.  It omits some necessary information and also uses a strange and unsupported formatting for some components.  In order for the exported db file to be useable by Postgres, we have to open the exported DB file in Atom and make a few edits.  I will point out the problems first and then show a properly formatted version at the end.

1. The exported db (`[exported db file].sql`) will look like this upon opening in a text editor:

    ```sql
    BEGIN TRANSACTION;
    CREATE TABLE `fft_classes` (
    	`Job_ID`		
    	`Job`
        `Identity`
        `Level`
        ...etc			
    );
    INSERT INTO `fft_classes` (Job_ID,Job,Identity,Gender,Level,Role,Association,HPm,MPm,PAm,MAm,SPm,M,J,CEV,HPc,MPc,PAc,MAc,SPc) VALUES ('1','Squire','Ramza1','Male','1','Story','Heroes','125','105','111','102','107','4','3','0.1','11','11','50','48','95'),
        ...etc.
    COMMIT;
    ```

2. The first problem is the use of back ticks ("\`") to surround items.  These occur in three places and must be removed.
    + Surrounding the name of the new table in the `CREATE TABLE` statement.
    + Surrounding the name of each column within the `CREATE TABLE` statement.
    + Surrounding the name of the table in the `INSERT INTO` statement.

3. The second problem is the absence of data types for each column in the `CREATE TABLE` statement.  A data type must be assigned to each column.  Here is a [list of data types](https://www.w3schools.com/sql/sql_datatypes.asp) for SQL tables if you need a reference.  Common uses are shown below.

    ```sql
    VARCHAR(255): 'text up to 255 chars; fewer chars = less memory; if over 255, converted to' TEXT
    TEXT: 'text up to 65,535 chars (takes more memory)'
    INT(size): 'integer values, size argument optional (can be written as' INTEGER(size) ')'
    DECIMAL(size, d): 'number, size = total number of digits, d = number of digits to right of decimal'
    NUMERIC(size, d): 'same as' DECIMAL
    DATE
    TIME
    DATETIME
    TIMESTAMP
    ```

4. The third issue appears to not be a problem, but should be mentioned.  In the `INSERT INTO` statement, numeric values such as `INT` or `DECIMAL` are enclosed in single quotes (uncommon, undesirable) along with `TEXT`/`VARCHAR` variables (required).  I checked the result of this and apparently Postgres converts them to the specified data type listed in the `CREATE TABLE` command, thankfully.  If this was not the case, using DB Browser to convert .csv files into .sql databases wouldn't be viable, because we'd have to go through every single insert statement / line and remove the single quotes from all numeric values by hand.  Also, listing the

5. Here is the corrected version of the exported DB file in \#1.  Using this in the Postgres loading commands detailed above should work just fine.

    ```sql
    BEGIN TRANSACTION;
    CREATE TABLE fft_stats (
    Job_ID varchar,
    Job varchar,
    Identity varchar,
    Iteration int,
    Gender varchar,
    Level int,
    Role varchar,
    Association varchar,
    Immunities varchar,
    Innate varchar,
    HPm int,
    MPm int,
    PAm int,
    MAm int,
    SPm int,
    Move int,
    Jump int,
    CEV float,
    HPc int,
    MPc int,
    PAc int,
    MAc int,
    SPc int
    );

    INSERT INTO fft_classes (Job_ID,Job,Identity,Gender,Level,Role,Association,HPm,MPm,PAm,MAm,SPm,M,J,CEV,HPc,MPc,PAc,MAc,SPc) VALUES ('1','Squire','Ramza1','Male','1','Story','Heroes','125','105','111','102','107','4','3','0.1','11','11','50','48','95'),
        ...etc.
    COMMIT;
    ```



<BR>
<BR>
