# JP SQLAlchemy Notes
Good documentation on their website.

Using Vtown fantasy football as an example because I have the vtown.db file handy.

### Basic stuff, sort and flesh out later
```python
from sqlalchemy import select, and_, or_, Table, MetaData, create_engine
from sqlalchemy.sql import column, text
from sqlalchemy.sql.expression import case

# Use a local SQLite DB (created from football data using Postgres)
# Can use MySQL or PostgreSQL as well, if have those file types
# 4 slashes before path on Unix, 3 otherwise
engine = create_engine('sqlite://///Users/jpw/Desktop/vtown.db')

# Make connection object
conn = engine.connect()

# Create a Table object, giving the matching name for the actual table in the DB and a MetaData object
# I am unsure if the table name 'vtown' defaults to the .db name?  Unsure if table inside vtown.db is actually named 'vtown'
vtable = Table('vtown', MetaData())
```

Create a (very basic) SELECT statement
```python
s = select([column('Team'), column('Owner')])
```

This works by simply naming a column that we will pass to the query statement below which _names the table_ we are selecting from.  If the columns in this select statement are not in that table, I believe we will get an error.  This is a more flexible way using the Select statement with the table name in it.  A more basic way is to simply name the table you're selecting from in the select statement itself, such as:

```python
s = select(vtown.c.Team, vtown.c.Owner)

# print the select statement to see what's in it:
print(s)
SELECT "Team", "Owner"
```

Here we name the table in the select statement, `vtown`, and the exact column we want with the column attribute `c`.  This query is only applicable to the `vtown` table though.  If we had another table of fantasy football league table and we wanted to access the `Team` and `Owner` cols in it, this select statement would not work.  Hence the first way, naming only the columns using the column object is the more flexible and intelligent way, since we could send that select statement to any fantasy football table that had those columns and it would work.

##### Make the query object
```python
# Use that SELECT statement in a query
query = s.select_from(vtable)
```

We now have a query.  We can do various things with it.  Two common actions are to either use it as it is, which is an n-tuple container, in our script or to load it into a Pandas dataframe.  Both of these require that we actually _connect_ to the database and execute this query


##### As a Python Data Structure
```python
# connect to DB and execute our query
result = conn.execute(query)

# We can iterate through it as any container/iterable:
for row in result:
    print(row)  # Gives an n-tuple for n columns

# Print the query to see the full query
In [1]: print(query)
SELECT "Team", "Owner"
FROM vtown
```

This also let's us use all the methods that come with this SQLAlchemy class object.  We can access them for the query or the result.  For example, we can take our query and grab only the distinct results by using the `.distinct()` method on our query (which just adds a SQL `DISTINCT` keyword to our query internally):

```python
result = conn.execute(query.distinct())
for row in result:
    print(row)
```


##### As a Pandas DataFrame
```python
# Use the result and connection object to create a DF!
pd.read_sql(query, conn)
```


##### Adding a col to the select statement
`s.append_column(column('Final_Wins'))`




##### Always Close the Connection
`result.close()`







##### IPython Testing Block
```python
from sqlalchemy import select, and_, or_, Table, MetaData, create_engine
from sqlalchemy.sql import column, text
from sqlalchemy.sql.expression import case
engine = create_engine('sqlite://///Users/jpw/Desktop/vtown.db')
conn = engine.connect()
vtable = Table('vtown', MetaData())

s = select([column('Team'), column('Owner')])
s.append_column(column('Final_Wins'))
# print(s)
s = s.where(and_(column('Owner').in_(['Brad', 'Ben', 'JP', 'Zack']), column('Final_Wins') >= 8))

s = s.where(column('Year').in_(['2014', '2015']))
# s = s.where(column('Owner').in_(['Zack']))
# s.append_whereclause(column('Owner').in_(['Zack']))
# whereclause = text("where(column('Owner').in_(['Brad', 'Ben']))")
# s.append_whereclause(whereclause)
# print(s)
query = s.select_from(vtable)
print(query)
result = conn.execute(query)
for row in result:
    print(row)
```


```sql
SELECT factset_entity_id as companyId, period_end as period_end_date,
       calendar_year, calendar_quarter, location_flag,
       rank() over (partition by factset_entity_id order by period_end desc) as rel_rank
FROM pviews.canonical_quarters_loc_v vfp
WHERE period_end <= :end_date
AND factset_entity_id IN :company_ids
AND fiscal_quarter % :period_factor = 0
```

s = select([column('Team'), column('Owner'),func.rank().over(func.partition_by('Year'))])
