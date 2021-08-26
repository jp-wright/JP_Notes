# JP Pandas and Numpy Notes:

## Pandas

#### Suppress Scientific Notation:
`pd.set_option('display.float_format', lambda x: '%.3f' % x)`

[Great article on optimizing Pandas performance](https://tomaugspurger.github.io/modern-4-performance.html)


***
When using importing a .csv in pandas, the first steps you should take to ensure things will work are:

1. df.head()
2. df.describe()
3. df.info()
4. check col names for spaces, and clean them up with:

```python
cols = [col.replace(' ', '_') for col in df.columns.tolist()]
df.columns = cols
```

check out `pd.melt(df)` -- neat function for reorganizing a DF by "ungrouping" or "melting" a DF and reorganizing by a specified column and value.

```python
np.bincount([iterable_name])
```


#### Categorical Data for Performance
`pd.Categorical` -- special dtype for huge speed boost (converts col to a C-array for Pandas) for categorical data; obviously applies best to columns with many repeated entries, such as gender.  Stores in single bytes instead of KB+ for Python's "object" dtypes.  Learn and use.  See [this write up](https://www.continuum.io/content/pandas-categoricals)
(`.astype('category')`)
`df['CategoryCol'].cat.codes`
`df['CategoryCol'].cat.categories`




When running into "index not unique" issues, converting DF to a MultiIndex is commonly a sensible solution (since having repeated entries in the index suggest there are groups of repeated data/categories/fields).  Using `df.set_index` to specific columns to create a hierarchy will result in a unique index.  Of course, a brute force (and possibly unhelpful/unwise choice is to simply use `df.reset_index()` which gives a new integer range index, but this might confuse your results!)


`df.sort_index(axis=)`
`df.index.is_unique`

`pd.isnull(df).any(1).nonzero()[0]`



`pd.MultiIndex`
+ `pd.IndexSlice` (jeff reback video @ 38:49)
    Assume a MultiIndex DF of 3 index levels and 2 col levels.
    ``` python
        idx = pd.IndexSlice
        dfmi.loc[idx[:, :, ['C1', 'C3']], idx[:, 'foo']]
        ## This returns all of Index level 0, 1, and only C1 and C3 from level 2.
        ## It also returns Col level 0 and only 'foo' from Col level 1.
    ```


`df.where()`
`df.mask()`
`df.assign` -- create new col, but can do it _in line_, which is great for chained methods.
`df.select_dtypes()`
`df.get_level_values()`
`df.reindex()`
`df.droplevel()`
`df.unstack()`
`df.isnull()` / `pd.isnull(df)`
`df.notnull()`...


`df.rename(columns={})`


Access or change the `name` attribute of both columns and index:
`df.index.name`
`df.columns.name`

### Create New Column Conditionally in One Go
One way, not vectorized, of creating new DF cols is to churn through the DF row by row, making your evaluations, and assign a value to the new col based on these results, like this:
```python
for idx in df.index:
    if df.loc[idx, 'Col_A'] == <some value>:
        df.loc[idx, 'New_Col'] = <new value>
    else:
        df.loc[idx, 'Col_A'] = ....    #etc.
```
This is obviously not a vectorized approach and is inefficient. It is also somewhat ugly and has needless code. It is very much _non-pythonic_.  

There are multiple ways to shorten this -- though not vectorize it, unfortunately. Four of the most common, and flexible, methods are discussed here. We can use `np.where()`, a list comp, `df.map`, or a conditional boolean mask. Note however that the comparison target (i.e. the conditional value) in these methods is constant.  To compare one cell in a DF to another, changing value (row by row, for example), you have to index into each one.  This variation is shown at the bottom of this section.

##### 1. `np.where()`
```python
df = pd.DataFrame({'Col_A': list('ZZXYW'), 'Col_B': list('ABBCD')})
df['New_Col'] = np.where(df['Col_A']=='Z', 'green', 'red')
print(df)

# gives us:
  Col_A Col_B   New_Col
0   Z    A      green
1   Z    B      green
2   X    B      red
3   Y    X      red
4   W    D      red

# Can also use multiple conditionals via '&' and parenthesis:
df['New_Col'] = np.where((df['Col_A'] == 'Z') & (df['Col_B'] == 'B'), 'green', 'red')
# gives us:
  Col_A Col_B   New_Col
0   Z    A      red
1   Z    B      green
2   X    B      red
3   Y    X      red
4   W    D      red
```

##### 2. List Comps
```python
df['New_Col'] = ['red' if a == 'Z' else 'green' for a in df['Col_A']]
# Need to double check the multiple conditional list comp:
df['New_Col'] = ['red' if (a == 'Z') and (b == 'B') else 'green' for a in df['Col_A']] for b in df['Col_B']]
```

Can use `np.ravel` to flatten multiple columns into a single column for parsing/summing, like so:

```python
total = np.sum([1 if cell == 'X' else 0 for cell in np.ravel([df['Col_A'], df['Col_B']])])
>>> 2
```
Take note of how a nested loop is handled in a list comp. The outermost loop is the first loop in the comp, and so on:

```python
a = [1 if x > 0 else 0 for x in [0, 0, 22] for x in [1, 1, 1, 1]]
>>> 12
# Nested loop:
a=0
for x in [0, 0, 22]:
    for x in [1, 1, 1, 1]:
        if x > 0:
            a += 1
>>> 12
```

##### 3. `df.map()`
```python
df['color'] = df['Col_A'].map(lambda x: 'red' if x == 'Z' else 'green')
```


##### 4. Conditional Boolean Mask
Note how this method works.  We mask into the df by using a `.loc` indexer (which is not usually how we mask) to give us only the desired rows to _assign_ to.  The `index` argument (i.e. the `i` argument inside the `.loc[i, c]` indexer) is the conditional bool mask. The second, `column`, argument is the new column we are creating.  To achieve the results above, we would do the following:

```python
df.loc[df['Col_A'] == 'Z', 'New_Col'] = 'green'
df.loc[df['Col_A'] != 'Z', 'New_Col'] = 'red'
```

Compared to the other methods, this approach seems more wasteful (2 lines) for a simple binary assignment (either `green` or `red`).  However, the value in this method becomes apparent when we have more than a binary assignment.  As shown above, using list comps or `np.where` can become a bit tedious when we have more than two conditions to juggle, in any number of columns.  This approach handles that easily, as we see now:

```python
df.loc[df['Col_A'] == 'Z', 'New_Col'] = 'green'
df.loc[df['Col_A'] == 'X', 'New_Col'] = 'red'
df.loc[df['Col_A'] == 'Y', 'New_Col'] = 'yellow'
df.loc[df['Col_B'] == 'D', 'New_Col'] = 'purple'

# gives us:
Col_A Col_B   New_Col
0   Z    A      green
1   Z    B      green
2   X    B      red
3   Y    X      yellow
4   W    D      purple
```

See how easy and specific that was?  It is not the most concise, necessarily, but it allows exact specificity without prohibitively complicated syntax that the same statement would involve if you tried to accomplish it in a list comp.  This is also a way to create _dummy vars_ by hand in a specific manner. (Just create the new col and set it `= 0`, then assign `1` via the masks above.  Thus you have `0` by default and `1` for the positive matches)

##### Variable comparisons
In this example we see a variable comparison -- testing whether the value in `df_test['Association']` is equal to the value in `df_test['Pred_Assoc']` for each row of the DF and assigning '`Yes`' if so, else '`No`'.  Note we are using a list comp here, but the other methods are also valid.  I personally just find list comps to be the most natural in this case.

```python
df_test['Pred_Correct?'] = ['Yes' if df_test.loc[idx, 'Association'] == df_test.loc[idx, 'Pred_Assoc'] else 'No' for idx in df_test.index]
```  


Of the three, List Comps and `map` are usually the fastest but not extremely so.  However, in a DF of a given size all are faster than the for loops, and much nicer to see.


<BR><BR>

### Selecting Cols based on dtype
```python
df.select_dtypes(include=['float64']).apply(your_function)
df.select_dtypes(exclude=['string','object']).apply(your_other_function)
```

<BR><BR>

### `.str.contains(...)`
Very useful Series-level method which returns a bool mask.  Here, we get a mask for all rows that have a capital 'P', such as 'Packers'.
```python
df_eda['Home_Team'].str.contains('P')
```

### GREAT Pandas Function - show all DF for just one time
```python
from IPython.display import display
def show_more(df, lines=None):
    """
    Function to display specified length of a DataFrame on a per-call basis.  Default number of lines is full length of DataFrame.  Can be specified as second argument in function call.
    """
    default_rows = pd.get_option("display.max_rows")
    if lines == None:
        lines = df.shape[0]

    pd.options.display.max_rows = lines
    display(df)
    pd.options.display.max_rows = default_rows

```

### `pd.qcut` and `pd.cut`
Divide and bin an array/Series into N bins for set quantile (5 = quintiles, etc.)
`pd.cut` makes the cuts based on _value_ of the data points (so, outliers cause extreme results with one item in a top or bottom bin and the rest clustered opposite it).  This means the membership of each bin is not balanced.
`pd.qcut` makes the cuts based on _frequency_ of the data points, returning equally sized bins as much as possible.

Use argument `retbins=True` to return the bin ranges.


#### .size() and .value_counts()
We can use `.size()` (instead of `.counts()`) as an aggregator when we use `.groupby()` to return the number of appearances of the grouped-by items.
We can also simply use `.value_counts()` on any Series, but it will sort them (descending) by default, and we may not want the series to be sorted necessarily.

Pandas can [read HTML directly](http://pandas.pydata.org/pandas-docs/stable/io.html#io-read-html) into a DF.  When possible, this is the best option for simplicity.  The HTML file it reads can be an actual URL, a local file, or a single string of HTML.  One thing to note if using a URL to pass to the `read_html()` method is that this will send a request to the server for this URL.  Depending on how many HTML tables you need, pinging the server for each table is likely a bad idea, leading to disconnection, temporary site-wide 404s, or even IP blocking.  If you need many tables per page and on many pages, you would like be better off using *BeautifulSoup* to get the HTML contents (HTML strings included) and use the `read_html()` method on each part of the HTML soup you need.

### Comparing DFs to singletons like `None`
From [this Stack Overflow post](http://stackoverflow.com/questions/36217969/how-to-compare-pandas-dataframe-against-none-in-python)  
Use `is` or `is not`, do not use equality operators `==` or `!=`:

```python
if self.pandas_df is not None:
    print('Do stuff')
```
[PEP 8](https://www.python.org/dev/peps/pep-0008/) says:

> Comparisons to singletons like `None` should always be done with `is` or `is not`, never the equality operators.  

There is also a nice [explanation](http://jaredgrubb.blogspot.de/2009/04/python-is-none-vs-none.html) why.

### Index


Resetting the Index via `reset_index()` can be a bit tricky.  As I understand it, we need to `set_index('col_name')` first, else we get all `NaN`.

`df.set_index('col_A').reset_index()` resets the index, starting at 0, to the len of `col_A`.  Technically I believe this does set the index itself to `col_A` and then the `reset_index()` method makes a new column from the existing index and then reindexes along that col.

##### Get index of a row if val in column matches arg:
Using the `[0]` index of index returns the first match only.
`df[df['Date'] == 'Playoffs'].index[0]`
can use `.tolist()` or `.values` as well for multiple matches.   

`df.isnull().sum()` -- counts of all NaNs in all cols.
`df.isin([container])` or `df['col'].isin()` are how you check for matching in a container or val.  Returns bool, so can index into DF via this: `df[df['Job_ID'].isin(id_list)]` returns only the rows whose Job_IDs are in the specified ID list.  Note this means it can also be inverted with the `~` operator.
use `argsort()` in numpy to sort by a column in a matrix



Reading from Excel files is much slower than reading a CSV file (Excel files have much overhead and have to be parsed differently.  As a result, it is always best to convert the .xlsx file to a .csv file if possible for reading into a Pandas dataframe. When reading large spreadsheets, the difference can be staggering.  For example, reading in the DVOA_Big_List, a spreadheet with 89,414 cells, as an .xlsx file takes around 34 seconds, while the same data as a .csv takes only 18.5 ms!)




When adding new rows, must use `df.loc` and not `df.iloc`.  `df.iloc` could potentially misstep on which index to add.  If you want to add multiple values in one go to a single row, you can use a list of values exactly as long as the number of columns in the DF.  Otherwise, a single value will be assigned to all columns.  You can also add values to multiple rows in a single statement by passing a list of the desired rows to be assigned to.  

```python
In [1]: df
Out[1]:
            A       B
Hungry      No      Yes                        
Thirsty     No      No

# Create a new row via the following
df.loc['Dessert'] = 'Yay'               # Fills A and B with 'Yay'
df.loc['Dessert'] = ['Yay', 'Never!']   # A=Yay, B=Never, must be same len as len(df.columns)
df.loc['Dessert', 'A'] = 'Yay'          # Fills A with 'Yay', rest of row = NaN

```
Note that all the new row assignments above assign values to the entire row.  The row itself does not exist prior to the assignment statement above, but upon its creation we are filling the entire row in a blanket assignment statement.  If we want to create multiple new rows in one statement, even if we assign the same value to all cols, then Pandas needs every row to already exist in the index.  So, you can assign to multiple rows in a single statement, but we need to create these rows in a separate statement (likely a loop). It is easiest to assign them as `None` just to make an empty cell we can fill in later with whatever data we want.

```python
for row in ['Dessert', 'Wine']:
    df.loc[row] = None

# Create TWO new rows in single statement via list
df.loc[['Dessert', 'Wine']] = 'Always'
```
The same rules apply as above, including the ability to assign to either only a certain col (the rest will fill with `NaN`) or to use a list to choose each col's val.

### Datetime

#### Datetime Accessor
See [Datetime properties documentation](http://pandas.pydata.org/pandas-docs/stable/api.html#datetimelike-properties).  

`pd.to_datetime()` has a sort of "hidden" attribute, `.dt`, that allows you to snag only the part of the time that you want, like minutes or the day of the week.  The following example takes a full `Timestamp` or `DateTimeIndex` object (typically in the format of `2017-02-23 00:46:02`) and returns only the minutes from it.  

```python
df['Minutes'] = pd.to_datetime(df['Duration'], format='%H:%M:%S').dt.minute
```



Roll back time by one Month
```python
df[‘date’] - pd.offsets.MonthEnd() - pd.offsets.MonthBegin()
```


```python
#clean up col names by replacing spaces with underscores -- needs to handle float
cols = df14.columns.tolist()
cols = [str(col) for col in cols] #convert all header names to str first\n",
cols = [col.replace(' ', '_') for col in cols]
cols = [''.join([i if 32 < ord(i) < 126 else "" for i in col]) for col in cols] #remove non-ASCII chars in headers
df14.columns = cols
```

find max value in a dict:
max(dict, key=lambda key: dict[key])





##### Get indices of cols based on which cols match a data type:
Here are two ways to get the indices of desired columns which, in this case, are either ints for floats.  
1. This way is possibly inefficient for large DFs because it doesn't take advantage of Pandas-specific methods, but is accomplished in one run.  

```python
for ind, col in enumerate(df.columns):
    if df[col].dtype in ['int64', 'float64']:
        print(ind, col)

# Here is the list comp:
inds = [ind for (ind, col) in enumerate(df.columns) if df[col].dtype in ['int64', 'float64']]
```

2. Uses Pandas `select_dtypes` and `get_loc` methods to get same result.

```python
cols = df.select_dtypes(include=['int64', 'float64']).columns
for col in cols:
    print(df.columns.get_loc(col))

# List Comp
inds = [df.columns.get_loc(col) for col in df.select_dtypes(include=['int64', 'float64']).columns]
```




Use `ipython --pylab` (or `%gui` ?) to enable interactive mode with IPython and MPL.  You can continue to work in IPy while a plot is displayed.


sort
df3.sort_values(['PA', 'MA'], ascending=[0,0])


`pd.to_csv()` exports to a csv, can pass the name you wish to save it under as a string; saves in dir the script was run in by default, put desired path in front of name to change destination.
Convert a series to datetime and return only the date by appending `.date()`:
`df['date'] = pd.to_datetime(df['real_dates'])[0].date()`


How to rename a column(s).  Note this function is used b/c it preserves column data instead of simply changing the names w/o also keeping the correct data:
`df = df.rename(columns = {'Job':'Jobe', 'OldCol2': 'NewCol2'})`

How to rename all cols by using a row:
`df.columns = df.iloc[0]` or `df.loc['row_name']`

How to reorganize cols:
cols = df.columns.tolist()
cols = cols[-2:] + cols[:-2] #or whatever order you need
cols.pop(whatever index you need to remove)
df = df[cols] (or df = df.ix[:, cols]) #remake df with right order


look into using `.eval()` in pandas
`dfl[dfl['visitor_ml']>=500].eval('Home - Visitor')`
The above command wouldn't work with spaces in the column names ('Home Score - Visitor Score' would fail). **update: don't use eval**


using aggregation then transform (VanderPlas) in Pandas
`df['PF/G_vs_Avg'] = df.groupby('Year')['PF/G'].transform(lambda x: x - x.mean())`
This creates a new column that is the normalized difference between the given data point (PF/G) and that year's mean PF/G.  Really nice.



Two steps to get date from a datetime series:
df['date'] = pd.to_datetime(df['date'])
df['date'] = pd.DatetimeIndex(df['date']).date

**note**: Don't have to convert `to_datetime` in order to get the `.date` but if you don't (in other words if you snag the `.date` from a series that is a string ('object'), it is much slower, and will result in frozen dataframes if the dataframe is large.  So, just convert first.)


How to drop by COLS:
`df.drop(['Col A', 'Col B', 'Col C'], axis=1, inplace=True)`

Note: axis = 1 means drop the column, and `inplace=True` makes the change permanent on this copy of the df.


How to drop by ROWS using a column criterion:
`df = df[df['date'] != '2015-07-10']`


Note you can't (so far as I have found) drop specific rows based on given critera (i.e. drop all rows where date = '2015-07-10') using the `drop` command.  If you can, I've not found how to do it correctly, and both Stack Overflow and an online book use the method above instead. So, we just make a new dataframe excluding the rows we don't want.

You CAN drop rows by specified indices, though:
`df3.drop(df3.index[4323:4333], inplace=True)`

Drop by the index name (whatever the data type is, and if it is an int it will drop based on the int displayed -- its 'name' -- and not the actual row-index integer.  So, if row 280 is labeled '7', it will be dropped if you use `df.drop(7)`, as will all other rows named '7'):
`df.drop(7, inplace=True)`


```python
df = pd.DataFrame({'a':[0,0,1,1], 'b':[0,1,0,1]})
df = df[(df.T != 0).any()]
df
   a  b
1  0  1
2  1  0
3  1  1
```

reset index:
df.reset_index(drop=True, inplace=True)


append
df.append(df2, ignore_index=True)


wrs = wrs.groupby('PNAME').agg(np.mean)










df.loc[] gives the index based on name passed (even if name is int, which could be anything, say 5000, in a df of only 20 rows.  )
df.iloc[] gives the actual integer position of the index (which goes from 0 to the len of the df)
df.ix[] takes either argument




index into a specific cell and change its value:
(This changes the value of HP to 777 for the row where Job == Monk)
`dfm.loc[dfm['Job'] == 'Monk', 'HP'] == 777`

For using multiple criteria:
`dfa.loc[((dfa['Job'] == 'Knight') & (dfa['MP'] == 80)), 'Job'] = 'dood'`

**note** there are other ways to do this, but this is the preferred method and doesn't trip up on a caveat or deprecation.

How to merely mask into (not change) a df based on multiple criteria:
`df3[(df3['exercise'] == 'Pec Deck') & (df3['date'] <= '2013-06-01')]`


**Pandas and Excel**
All row/col are zero-indexed in pd.read_excel()  
+ Sheetname: pick sheet to read in from Excel file. Can use index or string with exact name.
+ index_col: use a col to set as index for the DF (same as df.set_index())
+ skip_footer: number of rows from the bottom to skip when importing, 0=ignore no rows (default), 10=ignore last 10 rows of the spreadsheet, etc. Might be tough to use on some huge spreadsheets
+ header: explicitly set a row to be the header row, default 0th row.
+ parse_cols: set which cols to import into DF. Can be an int (col to stop at) or a list of ints. Can use string of col index such as "A:C, F" for Excel col refs. If None, import all calls (no parsing.)
+ skiprows: how many rows to skip from the top of the sheet when importing, 0=import all

Note: all of these operations can be done after importing via pandas, however if possible it is easier to do upon DF creation, and can potentially save memory if using big spreadsheets.

### String Manipulation
Pandas is actually quite adept at string manipulation, having some nice functions in addition to the standard `str` class.  One example is `extract`, which uses Regular Expressions to return matched text.  Here is an example of how we can use `extract` to split a string into separate parts and create new columns from those split parts. In the following string we have a compound word blob which actually represents a sex and age range, lower age to higher.  "m2435" represents male from 24-35:
```python
string = 'm2435'
words = pd.Series(string)
tmp_df = words.str.extract("(\D)(\d+)(\d{2})") #RegEx for match single char, then the next chars until the final two, then the final two.  See RegEx for more info.  

# Name columns
tmp_df.columns = ["sex", "age_lower", "age_upper"]

# Create `age` column based on `age_lower` and `age_upper`
tmp_df["age"] = tmp_df["age_lower"] + "-" + tmp_df["age_upper"]

# Merge
df = pd.concat([df, tmp_df], axis=1)

```


### Subset sorting
Don't sort an entire DF if you're only interested in the n-largest or n-smallest values in the DF.  Instead use Pandas' built-in `nlargest` and `nsmallest` methods.
```python
# Bad:
In [1]: delays.sort_values().tail(5)
>>> 63.3 ms per loop

# Good:
In [2]: delays.nlargest(5).sort_values()
>>> 12.3 ms per loop
```


### Append, Concat, Join, and Merge
Pandas can be tricky with these four options.  From my experience, they break down as follows: `append` and `concat` are similar, and `join` and `merge` are similar.  By default, `merge()` looks to join on common columns, `join()` on common indices, and `concat()` by just appending on a given axis.

#### Append and Concat
After much tinkering, I find that `append` and `concat` only stack the second DF onto the bottom of the first DF for `axis=0`, or the the right side for `axis=1`. `Concat` can intelligently combine the DFs if they share the same columns or same index. Append is more forceful in that it just stacks the new DF onto the bottom of the old DF, even if they share an index.  Because of its nature, `append` is also commonly used when adding a single series instead of an entire DF (though you will likely need to ignore the index, `ignore_index=True`).

(You can transpose both DFs in the function call to stack them sideways, then re-transpose the final product to have it back in normal order.  Doing so risks losing all columnar dtypes as the columns now have varied dtypes do to them actually being the rows of the original DFs. You also lose all column headers, so you will have to address both these issues if this is, for some strange reason, the method you need.  Can't imagine a situation in which it would be if you want to preserve that information.)  

As such, these two fit well with manipulation and combination of smaller tables or DFs (in my mind) for use largely in functions or whatnot.  One notable exception is that if you have data that is all in the same table structure (i.e. have the exact same columns in the exact same order), and you simply need to add to a "running list" of a DF, then these (`append` in particular) would be right to use.  

Example: you have Week 1 of NFL results for whatever columns (say the columns are points, turnovers, and punts).  After Week 2, you have the new week's data in the exact same structure.  You could just `append` Week 2 to Week 1, and then have a resulting DF of both weeks.  Etc.

#### Join and Merge

[Join](http://pandas.pydata.org/pandas-docs/version/0.19.2/generated/pandas.DataFrame.join.html) and
[Merge](http://pandas.pydata.org/pandas-docs/version/0.19.2/generated/pandas.DataFrame.merge.html#pandas.DataFrame.merge) behave much like the traditional SQL `join` function, but `merge` seems to be a slightly more focused, narrower sub-function of `join`.  See below.  
Here is a great [overview with examples](https://pythonprogramming.net/join-merge-data-analysis-python-pandas-tutorial/?completed=/concatenate-append-data-analysis-python-pandas-tutorial/) of these two functions.

##### Join
`join` in particular can be fussy.  If the two DFs have any of the same columns, the `join` function will use them both in the new DF.  To distinguish between them, Pandas appends a suffix to each to let you know which original DF the duplicate columns came from. As such you will have to provide the suffix which it appends, in the `lsuffix` and `rsuffix` arguments. So, `lsuffix = 'leftside'` would return the duplicate column from the left table in the join with "leftside" appended to the column name as its differentiator. (use `_[df_name]` to access the DF's index for this).

Pandas is strict about this, and Failure to provide suffix names gives a *ValueError*:  
`ValueError: columns overlap but no suffix specified: Index([u'Job_ID', u'Job', u'Identity', u'Role', u'Association'], dtype='object')`.  
Anytime you get this error, you have extra duplicate columns and must give a suffix for each (note the suffix can be the same word, if you want).

Use `caller.join(other, lsuffix='_caller', rsuffix='_other')` to join them based on their indices. Realize you will get duplicate columns, obvi.

If you want to join two DFs and use a shared column as the index for the new DF, then use: `caller.set_index('colA').join(other.set_index('colA'))`

So, if you want to join DF2 to DF1, on the same index, and only join the columns that are unique to them (probably the most commonly desired type of join...), then, as far as I can tell, using an `outer` join you will have to manually specify which columns from DF2 to join to DF1 (do this either ahead of time by creating a new, trimmed DF3 from DF2, or by passing DF2 in with a range of columns listed).  

Here is an example:  
`df1.join(df2.iloc[:, 3:8], how='outer')`  

This will get you all of `df1` and will add columns 3, 4, 5, 6, 7 from `df2` to `df1`, with the shared index (assuming the indices match, I believe.)  So, it is doable.  But it's touchy and frustrating.  Which leads us to a much better solution in `merge`.

##### Merge
Here is how easy it is to perform the final, commonly desired join using `merge` described at the end of the `join` section:  
`df1.merge(df2, how='outer')`

Boom. Done.  

`merge` has many other options, like what to join on and what types of joins, as well as separate indices to join on.  Read about it.  But in general, if you are trying to take parts of one DF and just slot them into a different DF because they both have the same indices, then you are going to use `merge`.  



dfm3.merge(dfc3, how='outer')






##### Stack/Unstack
Here is a [great overview blog post](http://nikgrozev.com/2015/07/01/reshaping-in-pandas-pivot-pivot-table-stack-and-unstack-explained-with-pictures/)
> Let us assume we have a DataFrame with MultiIndices on the rows and columns. Stacking a DataFrame means moving (also rotating or pivoting) the innermost column index to become the innermost row index. The inverse operation is called unstacking. It means moving the innermost row index to become the innermost column index.

Example using FFT DF:
```python
In [1]: df = pd.read_csv('~/Desktop/fft_class_stats.csv')
In [2]: df = df[df['Association'] == 'Generic']
In [3]: df.shape
Out [3]: (23, 20)
In [4]: df.head()
Out [4]: >>>
```

```bash
   Job_ID      Job Identity  Gender  Level     Role Association  HPm  MPm  \
70     4A   Squire   Squire  Either      1   Squire     Generic  100   75   
71     4B  Chemist  Chemist  Either      1  Chemist     Generic   80   75   
72     4C   Knight   Knight  Either      1   Squire     Generic  120   80   
73     4D   Archer   Archer  Either      1   Squire     Generic  100   65   
74     4E     Monk     Monk  Either      1   Squire     Generic  135   80   

    PAm  MAm  SPm  Move  Jump     CEV  HPc  MPc  PAc  MAc  SPc  
70   90   80  100     4     3    0.05   11   15   60   50  100  
71   75   80  100     3     3    0.05   12   16   75   50  100  
72  120   80  100     3     3    0.10   10   15   40   50  100  
73  110   80  100     3     3    0.10   11   16   45   50  100  
74  129   80  110     3     4    0.20    9   13   48   50  100
```

In order to `stack` or `unstack`, we need a MultiIndex (technically we can do it on a "flat" (2D, no nested indices) DF, but the result is just a `Series` that has either the rows or cols nested into the other, depending upon whether you stack or unstack).  If your DF is currently flat the primary way to create a hierarchy of indices (a MultiIndex)  is to use `set_index()`.  The flow of this operation is to choose columns in the existing DF to become part of the index (rows), and then choose which of those to become part of the columns nested structure.  

Using the example DF above, we would choose some columns (commonly the categorical ones) to be in our index.  Then we will choose to unstack either a specific index level or many of them.  By default `unstack` moves the _innermost_ row index to become _innermost_ column index.  Index levels are ordered outside-in, meaning the outermost index is level=0, the one immediately to the right of level 0 is level=1, and so on as the nesting of indices increases.  When calling `unstack` or `stack`, we can specify which level of index we want to move by either the number of the level or giving the name of the index itself.  By default the methods use `-1` as the level, which means the _innermost_ level (i.e. most nested).  

Here's an example using the FFT DF:  

1. Choose two (categorical) columns to be the index.  Order matters, here, as the first column named becomes the outermost index and they nest progressively after that. Then unstack the inner one (Role) to now be the new innermost column index.  Note the base `df` has no column (or row) nesting, so by unstacking the inner row index here (Role) we are going to place it as the inner column index under the existing columns, leaving 'Job' as the index for the rows.

2. The default `.unstack()` call is equivalent to `.unstack(1)` or `.unstack('Role')` in this case because 'Role' is level 1, and this is the innermost index level here.  This is also equivalent to `.unstack(-1)` which means the innermost level.  If we had three or more indices to begin with we could choose any sub-selection of them to unstack/stack.
    ```python
    In [1]: df_unstack = df.set_index(['Job', 'Role']).unstack('Role')
    In [2]: df_unstack.head()
    Out [2]: >>>
    ```
    ```bash
                Job_ID                    Identity                    Gender                    
    Role       Chemist Generic Squire      Chemist Generic  Squire   Chemist Generic  ...
    Job                                                                             
    Archer        None    None     4D         None    None  Archer      None    None  ...
    Bard            5B    None   None         Bard    None    None    Either    None  ...
    Calculator      5A    None   None   Calculator    None    None    Either    None  ...
    Chemist         4B    None   None      Chemist    None    None    Either    None  ...
    Dancer        None    None     5C         None    None  Dancer      None    None  ...
    ...
    ...

    ```

    We see that the new, unstacked DF indeed has 'Job' as the index for the rows and the 'Role' index is now the innermost column index.  This means for every column in the original DF ('Identity', 'Gender', 'Level', ... etc.) now have their own nested columns of 'Role' (which itself has three categories: 'Chemist', 'Generic', 'Squire').

    If we want to see the names of the levels for the rows or columns we can just use the `.names` attribute:  
    ```python
    In [1]: df_unstack.index.names
    Out [1]: FrozenList(['Job'])

    In [2]: df_unstack.columns.names
    Out [2]: FrozenList([None, 'Role'])     # None is original set of cols, unnamed upon original import.
    ```


### Groupby vs. Partitioning upstream
So, by partitioning the DF upfront into the 401 separate companies with the columns stacked/nested to provide the other groups made by the remaining three columns used in groupby, we were able to limit the overall groups to 401 and have a wider/fatter DF.  We did this by unstacking the original DF into the nested columns desired with this command:  

`df = df.set_index(['companyId', 'raw_date', 'fe_item', 'fe_per_rel', 'periodicity']).unstack(['fe_item', 'fe_per_rel', 'periodicity'])`
Much faster is many groups originally.












`df.swaplevel` and `df.reorder_levels` can be used to rearrange a multiindex.

`df.columns.get_level_values(0)`


### Append v. Concat
It's pretty common to have many similar sources (say a bunch of CSVs) that need to be combined into a single DataFrame. There are two routes to the same end:

1. Initialize one DataFrame and append to that
2. Make many smaller DataFrames and concatenate at the end

For pandas, the second option is faster. DataFrame appends are expensive relative to a list append. Depending on the values, pandas might have to recast the data to a different type. And indexes are immutable, so each time you append pandas has to create an entirely new one.





example from pellucid using `large_df_trial.csv`:  
`df = df.set_index(['companyId', 'raw_date', 'fe_item', 'fe_per_rel', 'periodicity']).unstack(['fe_item', 'fe_per_rel', 'periodicity'])`









### 10 Unknown Pandas Tricks
From this [Real Python article](https://realpython.com/python-pandas-tricks/)  

1. Configure Options & Settings at Interpreter Startup
You may have run across Pandas’ rich options and settings system before.

It’s a huge productivity saver to set customized Pandas options at interpreter startup, especially if you work in a scripting environment. You can use pd.set_option() to configure to your heart’s content with a Python or IPython startup file.

The options use a dot notation such as pd.set_option('display.max_colwidth', 25), which lends itself well to a nested dictionary of options:

import pandas as pd

def start():
    options = {
        'display': {
            'max_columns': None,
            'max_colwidth': 25,
            'expand_frame_repr': False,  # Don't wrap to multiple pages
            'max_rows': 14,
            'max_seq_items': 50,         # Max length of printed sequence
            'precision': 4,
            'show_dimensions': False
        },
        'mode': {
            'chained_assignment': None   # Controls SettingWithCopyWarning
        }
    }

    for category, option in options.items():
        for op, value in option.items():
            pd.set_option(f'{category}.{op}', value)  # Python 3.6+

if __name__ == '__main__':
    start()
    del start  # Clean up namespace in the interpreter
If you launch an interpreter session, you’ll see that everything in the startup script has been executed, and Pandas is imported for you automatically with your suite of options:

>>>
>>> pd.__name__
'pandas'
>>> pd.get_option('display.max_rows')
14
Let’s use some data on abalone hosted by the UCI Machine Learning Repository to demonstrate the formatting that was set in the startup file. The data will truncate at 14 rows with 4 digits of precision for floats:

>>>
>>> url = ('https://archive.ics.uci.edu/ml/'
...        'machine-learning-databases/abalone/abalone.data')
>>> cols = ['sex', 'length', 'diam', 'height', 'weight', 'rings']
>>> abalone = pd.read_csv(url, usecols=[0, 1, 2, 3, 4, 8], names=cols)

>>> abalone
     sex  length   diam  height  weight  rings
0      M   0.455  0.365   0.095  0.5140     15
1      M   0.350  0.265   0.090  0.2255      7
2      F   0.530  0.420   0.135  0.6770      9
3      M   0.440  0.365   0.125  0.5160     10
4      I   0.330  0.255   0.080  0.2050      7
5      I   0.425  0.300   0.095  0.3515      8
6      F   0.530  0.415   0.150  0.7775     20
# ...
4170   M   0.550  0.430   0.130  0.8395     10
4171   M   0.560  0.430   0.155  0.8675      8
4172   F   0.565  0.450   0.165  0.8870     11
4173   M   0.590  0.440   0.135  0.9660     10
4174   M   0.600  0.475   0.205  1.1760      9
4175   F   0.625  0.485   0.150  1.0945     10
4176   M   0.710  0.555   0.195  1.9485     12
You’ll see this dataset pop up in other examples later as well.

2. Make Toy Data Structures With Pandas’ Testing Module
Hidden way down in Pandas’ testing module are a number of convenient functions for quickly building quasi-realistic Series and DataFrames:

>>>
>>> import pandas.util.testing as tm
>>> tm.N, tm.K = 15, 3  # Module-level default rows/columns

>>> import numpy as np
>>> np.random.seed(444)

>>> tm.makeTimeDataFrame(freq='M').head()
                 A       B       C
2000-01-31  0.3574 -0.8804  0.2669
2000-02-29  0.3775  0.1526 -0.4803
2000-03-31  1.3823  0.2503  0.3008
2000-04-30  1.1755  0.0785 -0.1791
2000-05-31 -0.9393 -0.9039  1.1837

>>> tm.makeDataFrame().head()
                 A       B       C
nTLGGTiRHF -0.6228  0.6459  0.1251
WPBRn9jtsR -0.3187 -0.8091  1.1501
7B3wWfvuDA -1.9872 -1.0795  0.2987
yJ0BTjehH1  0.8802  0.7403 -1.2154
0luaYUYvy1 -0.9320  1.2912 -0.2907
There are around 30 of these, and you can see the full list by calling dir() on the module object. Here are a few:

>>>
>>> [i for i in dir(tm) if i.startswith('make')]
['makeBoolIndex',
 'makeCategoricalIndex',
 'makeCustomDataframe',
 'makeCustomIndex',
 # ...,
 'makeTimeSeries',
 'makeTimedeltaIndex',
 'makeUIntIndex',
 'makeUnicodeIndex']
These can be useful for benchmarking, testing assertions, and experimenting with Pandas methods that you are less familiar with.

3. Take Advantage of Accessor Methods
Perhaps you’ve heard of the term accessor, which is somewhat like a getter (although getters and setters are used infrequently in Python). For our purposes here, you can think of a Pandas accessor as a property that serves as an interface to additional methods.

Pandas Series have three of them:

>>>
>>> pd.Series._accessors
{'cat', 'str', 'dt'}
Yes, that definition above is a mouthful, so let’s take a look at a few examples before discussing the internals.

.cat is for categorical data, .str is for string (object) data, and .dt is for datetime-like data. Let’s start off with .str: imagine that you have some raw city/state/ZIP data as a single field within a Pandas Series.

Pandas string methods are vectorized, meaning that they operate on the entire array without an explicit for-loop:

>>>
>>> addr = pd.Series([
...     'Washington, D.C. 20003',
...     'Brooklyn, NY 11211-1755',
...     'Omaha, NE 68154',
...     'Pittsburgh, PA 15211'
... ])

>>> addr.str.upper()
0     WASHINGTON, D.C. 20003
1    BROOKLYN, NY 11211-1755
2            OMAHA, NE 68154
3       PITTSBURGH, PA 15211
dtype: object

>>> addr.str.count(r'\d')  # 5 or 9-digit zip?
0    5
1    9
2    5
3    5
dtype: int64
For a more involved example, let’s say that you want to separate out the three city/state/ZIP components neatly into DataFrame fields.

You can pass a regular expression to .str.extract() to “extract” parts of each cell in the Series. In .str.extract(), .str is the accessor, and .str.extract() is an accessor method:

>>>
>>> regex = (r'(?P<city>[A-Za-z ]+), '      # One or more letters
...          r'(?P<state>[A-Z]{2}) '        # 2 capital letters
...          r'(?P<zip>\d{5}(?:-\d{4})?)')  # Optional 4-digit extension
...
>>> addr.str.replace('.', '').str.extract(regex)
         city state         zip
0  Washington    DC       20003
1    Brooklyn    NY  11211-1755
2       Omaha    NE       68154
3  Pittsburgh    PA       15211
This also illustrates what is known as method-chaining, where .str.extract(regex) is called on the result of addr.str.replace('.', ''), which cleans up use of periods to get a nice 2-character state abbreviation.

It’s helpful to know a tiny bit about how these accessor methods work as a motivating reason for why you should use them in the first place, rather than something like addr.apply(re.findall, ...).

Each accessor is itself a bona fide Python class:

.str maps to StringMethods.
.dt maps to CombinedDatetimelikeProperties.
.cat routes to CategoricalAccessor.
These standalone classes are then “attached” to the Series class using a CachedAccessor. It is when the classes are wrapped in CachedAccessor that a bit of magic happens.

CachedAccessor is inspired by a “cached property” design: a property is only computed once per instance and then replaced by an ordinary attribute. It does this by overloading the .__get__() method, which is part of Python’s descriptor protocol.

Note: If you’d like to read more about the internals of how this works, see the Python Descriptor HOWTO and this post on the cached property design. Python 3 also introduced functools.lru_cache(), which offers similar functionality. There are examples all over the place of this pattern, such as in the aiohttp package.
The second accessor, .dt, is for datetime-like data. It technically belongs to Pandas’ DatetimeIndex, and if called on a Series, it is converted to a DatetimeIndex first:

>>>
>>> daterng = pd.Series(pd.date_range('2017', periods=9, freq='Q'))
>>> daterng
0   2017-03-31
1   2017-06-30
2   2017-09-30
3   2017-12-31
4   2018-03-31
5   2018-06-30
6   2018-09-30
7   2018-12-31
8   2019-03-31
dtype: datetime64[ns]

>>>  daterng.dt.day_name()
0      Friday
1      Friday
2    Saturday
3      Sunday
4    Saturday
5    Saturday
6      Sunday
7      Monday
8      Sunday
dtype: object

>>> # Second-half of year only
>>> daterng[daterng.dt.quarter > 2]
2   2017-09-30
3   2017-12-31
6   2018-09-30
7   2018-12-31
dtype: datetime64[ns]

>>> daterng[daterng.dt.is_year_end]
3   2017-12-31
7   2018-12-31
dtype: datetime64[ns]
The third accessor, .cat, is for Categorical data only, which you’ll see shortly in its own section.

4. Create a DatetimeIndex From Component Columns
Speaking of datetime-like data, as in daterng above, it’s possible to create a Pandas DatetimeIndex from multiple component columns that together form a date or datetime:

>>>
>>> from itertools import product
>>> datecols = ['year', 'month', 'day']

>>> df = pd.DataFrame(list(product([2017, 2016], [1, 2], [1, 2, 3])),
...                   columns=datecols)
>>> df['data'] = np.random.randn(len(df))
>>> df
    year  month  day    data
0   2017      1    1 -0.0767
1   2017      1    2 -1.2798
2   2017      1    3  0.4032
3   2017      2    1  1.2377
4   2017      2    2 -0.2060
5   2017      2    3  0.6187
6   2016      1    1  2.3786
7   2016      1    2 -0.4730
8   2016      1    3 -2.1505
9   2016      2    1 -0.6340
10  2016      2    2  0.7964
11  2016      2    3  0.0005

>>> df.index = pd.to_datetime(df[datecols])
>>> df.head()
            year  month  day    data
2017-01-01  2017      1    1 -0.0767
2017-01-02  2017      1    2 -1.2798
2017-01-03  2017      1    3  0.4032
2017-02-01  2017      2    1  1.2377
2017-02-02  2017      2    2 -0.2060
Finally, you can drop the old individual columns and convert to a Series:

>>>
>>> df = df.drop(datecols, axis=1).squeeze()
>>> df.head()
2017-01-01   -0.0767
2017-01-02   -1.2798
2017-01-03    0.4032
2017-02-01    1.2377
2017-02-02   -0.2060
Name: data, dtype: float64

>>> df.index.dtype_str
'datetime64[ns]
The intuition behind passing a DataFrame is that a DataFrame resembles a Python dictionary where the column names are keys, and the individual columns (Series) are the dictionary values. That’s why pd.to_datetime(df[datecols].to_dict(orient='list')) would also work in this case. This mirrors the construction of Python’s datetime.datetime, where you pass keyword arguments such as datetime.datetime(year=2000, month=1, day=15, hour=10).

5. Use Categorical Data to Save on Time and Space
One powerful Pandas feature is its Categorical dtype.

Even if you’re not always working with gigabytes of data in RAM, you’ve probably run into cases where straightforward operations on a large DataFrame seem to hang up for more than a few seconds.

Pandas object dtype is often a great candidate for conversion to category data. (object is a container for Python str, heterogeneous data types, or “other” types.) Strings occupy a significant amount of space in memory:

>>>
>>> colors = pd.Series([
...     'periwinkle',
...     'mint green',
...     'burnt orange',
...     'periwinkle',
...     'burnt orange',
...     'rose',
...     'rose',
...     'mint green',
...     'rose',
...     'navy'
... ])
...
>>> import sys
>>> colors.apply(sys.getsizeof)
0    59
1    59
2    61
3    59
4    61
5    53
6    53
7    59
8    53
9    53
dtype: int64
Note: I used sys.getsizeof() to show the memory occupied by each individual value in the Series. Keep in mind these are Python objects that have some overhead in the first place. (sys.getsizeof('') will return 49 bytes.)

There is also colors.memory_usage(), which sums up the memory usage and relies on the .nbytes attribute of the underlying NumPy array. Don’t get too bogged down in these details: what is important is relative memory usage that results from type conversion, as you’ll see next.
Now, what if we could take the unique colors above and map each to a less space-hogging integer? Here is a naive implementation of that:

>>>
>>> mapper = {v: k for k, v in enumerate(colors.unique())}
>>> mapper
{'periwinkle': 0, 'mint green': 1, 'burnt orange': 2, 'rose': 3, 'navy': 4}

>>> as_int = colors.map(mapper)
>>> as_int
0    0
1    1
2    2
3    0
4    2
5    3
6    3
7    1
8    3
9    4
dtype: int64

>>> as_int.apply(sys.getsizeof)
0    24
1    28
2    28
3    24
4    28
5    28
6    28
7    28
8    28
9    28
dtype: int64
Note: Another way to do this same thing is with Pandas’ pd.factorize(colors):

>>>
>>> pd.factorize(colors)[0]
array([0, 1, 2, 0, 2, 3, 3, 1, 3, 4])
Either way, you are encoding the object as an enumerated type (categorical variable).
You’ll notice immediately that memory usage is just about cut in half compared to when the full strings are used with object dtype.

Earlier in the section on accessors, I mentioned the .cat (categorical) accessor. The above with mapper is a rough illustration of what is happening internally with Pandas’ Categorical dtype:

“The memory usage of a Categorical is proportional to the number of categories plus the length of the data. In contrast, an object dtype is a constant times the length of the data.” (Source)
In colors above, you have a ratio of 2 values for every unique value (category):

>>>
>>> len(colors) / colors.nunique()
2.0
As a result, the memory savings from converting to Categorical is good, but not great:

>>>
>>> # Not a huge space-saver to encode as Categorical
>>> colors.memory_usage(index=False, deep=True)
650
>>> colors.astype('category').memory_usage(index=False, deep=True)
495
However, if you blow out the proportion above, with a lot of data and few unique values (think about data on demographics or alphabetic test scores), the reduction in memory required is over 10 times:

>>>
>>> manycolors = colors.repeat(10)
>>> len(manycolors) / manycolors.nunique()  # Much greater than 2.0x
20.0

>>> manycolors.memory_usage(index=False, deep=True)
6500
>>> manycolors.astype('category').memory_usage(index=False, deep=True)
585
A bonus is that computational efficiency gets a boost too: for categorical Series, the string operations are performed on the .cat.categories attribute rather than on each original element of the Series.

In other words, the operation is done once per unique category, and the results are mapped back to the values. Categorical data has a .cat accessor that is a window into attributes and methods for manipulating the categories:

>>>
>>> ccolors = colors.astype('category')
>>> ccolors.cat.categories
Index(['burnt orange', 'mint green', 'navy', 'periwinkle', 'rose'], dtype='object')
In fact, you can reproduce something similar to the example above that you did manually:

>>>
>>> ccolors.cat.codes
0    3
1    1
2    0
3    3
4    0
5    4
6    4
7    1
8    4
9    2
dtype: int8
All that you need to do to exactly mimic the earlier manual output is to reorder the codes:

>>>
>>> ccolors.cat.reorder_categories(mapper).cat.codes
0    0
1    1
2    2
3    0
4    2
5    3
6    3
7    1
8    3
9    4
dtype: int8
Notice that the dtype is NumPy’s int8, an 8-bit signed integer that can take on values from -127 to 128. (Only a single byte is needed to represent a value in memory. 64-bit signed ints would be overkill in terms of memory usage.) Our rough-hewn example resulted in int64 data by default, whereas Pandas is smart enough to downcast categorical data to the smallest numerical dtype possible.

Most of the attributes for .cat are related to viewing and manipulating the underlying categories themselves:

>>>
>>> [i for i in dir(ccolors.cat) if not i.startswith('_')]
['add_categories',
 'as_ordered',
 'as_unordered',
 'categories',
 'codes',
 'ordered',
 'remove_categories',
 'remove_unused_categories',
 'rename_categories',
 'reorder_categories',
 'set_categories']
There are a few caveats, though. Categorical data is generally less flexible. For instance, if inserting previously unseen values, you need to add this value to a .categories container first:

>>>
>>> ccolors.iloc[5] = 'a new color'
# ...
ValueError: Cannot setitem on a Categorical with a new category,
set the categories first

>>> ccolors = ccolors.cat.add_categories(['a new color'])
>>> ccolors.iloc[5] = 'a new color'  # No more ValueError
If you plan to be setting values or reshaping data rather than deriving new computations, Categorical types may be less nimble.

6. Introspect Groupby Objects via Iteration
When you call df.groupby('x'), the resulting Pandas groupby objects can be a bit opaque. This object is lazily instantiated and doesn’t have any meaningful representation on its own.

You can demonstrate with the abalone dataset from example 1:

>>>
>>> abalone['ring_quartile'] = pd.qcut(abalone.rings, q=4, labels=range(1, 5))
>>> grouped = abalone.groupby('ring_quartile')

>>> grouped
<pandas.core.groupby.groupby.DataFrameGroupBy object at 0x11c1169b0>
Alright, now you have a groupby object, but what is this thing, and how do I see it?

Before you call something like grouped.apply(func), you can take advantage of the fact that groupby objects are iterable:

>>>
>>> help(grouped.__iter__)

        Groupby iterator

        Returns
        -------
        Generator yielding sequence of (name, subsetted object)
        for each group
Each “thing” yielded by grouped.__iter__() is a tuple of (name, subsetted object), where name is the value of the column on which you’re grouping, and subsetted object is a DataFrame that is a subset of the original DataFrame based on whatever grouping condition you specify. That is, the data gets chunked by group:

>>>
>>> for idx, frame in grouped:
...     print(f'Ring quartile: {idx}')
...     print('-' * 16)
...     print(frame.nlargest(3, 'weight'), end='\n\n')
...
Ring quartile: 1
----------------
     sex  length   diam  height  weight  rings ring_quartile
2619   M   0.690  0.540   0.185  1.7100      8             1
1044   M   0.690  0.525   0.175  1.7005      8             1
1026   M   0.645  0.520   0.175  1.5610      8             1

Ring quartile: 2
----------------
     sex  length  diam  height  weight  rings ring_quartile
2811   M   0.725  0.57   0.190  2.3305      9             2
1426   F   0.745  0.57   0.215  2.2500      9             2
1821   F   0.720  0.55   0.195  2.0730      9             2

Ring quartile: 3
----------------
     sex  length  diam  height  weight  rings ring_quartile
1209   F   0.780  0.63   0.215   2.657     11             3
1051   F   0.735  0.60   0.220   2.555     11             3
3715   M   0.780  0.60   0.210   2.548     11             3

Ring quartile: 4
----------------
     sex  length   diam  height  weight  rings ring_quartile
891    M   0.730  0.595    0.23  2.8255     17             4
1763   M   0.775  0.630    0.25  2.7795     12             4
165    M   0.725  0.570    0.19  2.5500     14             4
Relatedly, a groupby object also has .groups and a group-getter, .get_group():

>>>
>>> grouped.groups.keys()
dict_keys([1, 2, 3, 4])

>>> grouped.get_group(2).head()
   sex  length   diam  height  weight  rings ring_quartile
2    F   0.530  0.420   0.135  0.6770      9             2
8    M   0.475  0.370   0.125  0.5095      9             2
19   M   0.450  0.320   0.100  0.3810      9             2
23   F   0.550  0.415   0.135  0.7635      9             2
39   M   0.355  0.290   0.090  0.3275      9             2
This can help you be a little more confident that the operation you’re performing is the one you want:

>>>
>>> grouped['height', 'weight'].agg(['mean', 'median'])
               height         weight
                 mean median    mean  median
ring_quartile
1              0.1066  0.105  0.4324  0.3685
2              0.1427  0.145  0.8520  0.8440
3              0.1572  0.155  1.0669  1.0645
4              0.1648  0.165  1.1149  1.0655
No matter what calculation you perform on grouped, be it a single Pandas method or custom-built function, each of these “sub-frames” is passed one-by-one as an argument to that callable. This is where the term “split-apply-combine” comes from: break the data up by groups, perform a per-group calculation, and recombine in some aggregated fashion.

If you’re having trouble visualizing exactly what the groups will actually look like, simply iterating over them and printing a few can be tremendously useful.

7. Use This Mapping Trick for Membership Binning
Let’s say that you have a Series and a corresponding “mapping table” where each value belongs to a multi-member group, or to no groups at all:

>>>
>>> countries = pd.Series([
...     'United States',
...     'Canada',
...     'Mexico',
...     'Belgium',
...     'United Kingdom',
...     'Thailand'
... ])
...
>>> groups = {
...     'North America': ('United States', 'Canada', 'Mexico', 'Greenland'),
...     'Europe': ('France', 'Germany', 'United Kingdom', 'Belgium')
... }
In other words, you need to map countries to the following result:

>>>
0    North America
1    North America
2    North America
3           Europe
4           Europe
5            other
dtype: object
What you need here is a function similar to Pandas’ pd.cut(), but for binning based on categorical membership. You can use pd.Series.map(), which you already saw in example #5, to mimic this:

from typing import Any

def membership_map(s: pd.Series, groups: dict,
                   fillvalue: Any=-1) -> pd.Series:
    # Reverse & expand the dictionary key-value pairs
    groups = {x: k for k, v in groups.items() for x in v}
    return s.map(groups).fillna(fillvalue)
This should be significantly faster than a nested Python loop through groups for each country in countries.

Here’s a test drive:

>>>
>>> membership_map(countries, groups, fillvalue='other')
0    North America
1    North America
2    North America
3           Europe
4           Europe
5            other
dtype: object
Let’s break down what’s going on here. (Sidenote: this is a great place to step into a function’s scope with Python’s debugger, pdb, to inspect what variables are local to the function.)

The objective is to map each group in groups to an integer. However, Series.map() will not recognize 'ab'—it needs the broken-out version with each character from each group mapped to an integer. This is what the dictionary comprehension is doing:

>>>
>>> groups = dict(enumerate(('ab', 'cd', 'xyz')))
>>> {x: k for k, v in groups.items() for x in v}
{'a': 0, 'b': 0, 'c': 1, 'd': 1, 'x': 2, 'y': 2, 'z': 2}
This dictionary can be passed to s.map() to map or “translate” its values to their corresponding group indices.

8. Understand How Pandas Uses Boolean Operators
You may be familiar with Python’s operator precedence, where and, not, and or have lower precedence than arithmetic operators such as <, <=, >, >=, !=, and ==. Consider the two statements below, where < and > have higher precedence than the and operator:

>>>
>>> # Evaluates to "False and True"
>>> 4 < 3 and 5 > 4
False

>>> # Evaluates to 4 < 5 > 4
>>> 4 < (3 and 5) > 4
True
Note: It’s not specifically Pandas-related, but 3 and 5 evaluates to 5 because of short-circuit evaluation:

“The return value of a short-circuit operator is the last evaluated argument.” (Source)
Pandas (and NumPy, on which Pandas is built) does not use and, or, or not. Instead, it uses &, |, and ~, respectively, which are normal, bona fide Python bitwise operators.

These operators are not “invented” by Pandas. Rather, &, |, and ~ are valid Python built-in operators that have higher (rather than lower) precedence than arithmetic operators. (Pandas overrides dunder methods like .__ror__() that map to the | operator.) To sacrifice some detail, you can think of “bitwise” as “elementwise” as it relates to Pandas and NumPy:

>>>
>>> pd.Series([True, True, False]) & pd.Series([True, False, False])
0     True
1    False
2    False
dtype: bool
It pays to understand this concept in full. Let’s say that you have a range-like Series:

>>>
>>> s = pd.Series(range(10))
I would guess that you may have seen this exception raised at some point:

>>>
>>> s % 2 == 0 & s > 3
ValueError: The truth value of a Series is ambiguous.
Use a.empty, a.bool(), a.item(), a.any() or a.all().
What’s happening here? It’s helpful to incrementally bind the expression with parentheses, spelling out how Python expands this expression step by step:

s % 2 == 0 & s > 3                      # Same as above, original expression
(s % 2) == 0 & s > 3                    # Modulo is most tightly binding here
(s % 2) == (0 & s) > 3                  # Bitwise-and is second-most-binding
(s % 2) == (0 & s) and (0 & s) > 3      # Expand the statement
((s % 2) == (0 & s)) and ((0 & s) > 3)  # The `and` operator is least-binding
The expression s % 2 == 0 & s > 3 is equivalent to (or gets treated as) ((s % 2) == (0 & s)) and ((0 & s) > 3). This is called expansion: x < y <= z is equivalent to x < y and y <= z.

Okay, now stop there, and let’s bring this back to Pandas-speak. You have two Pandas Series that we’ll call left and right:

>>>
>>> left = (s % 2) == (0 & s)
>>> right = (0 & s) > 3
>>> left and right  # This will raise the same ValueError
You know that a statement of the form left and right is truth-value testing both left and right, as in the following:

>>>
>>> bool(left) and bool(right)
The problem is that Pandas developers intentionally don’t establish a truth-value (truthiness) for an entire Series. Is a Series True or False? Who knows? The result is ambiguous:

>>>
>>> bool(s)
ValueError: The truth value of a Series is ambiguous.
Use a.empty, a.bool(), a.item(), a.any() or a.all().
The only comparison that makes sense is an elementwise comparison. That’s why, if an arithmetic operator is involved, you’ll need parentheses:

>>>
>>> (s % 2 == 0) & (s > 3)
0    False
1    False
2    False
3    False
4     True
5    False
6     True
7    False
8     True
9    False
dtype: bool
In short, if you see the ValueError above pop up with boolean indexing, the first thing you should probably look to do is sprinkle in some needed parentheses.

9. Load Data From the Clipboard
It’s a common situation to need to transfer data from a place like Excel or Sublime Text to a Pandas data structure. Ideally, you want to do this without going through the intermediate step of saving the data to a file and afterwards reading in the file to Pandas.

You can load in DataFrames from your computer’s clipboard data buffer with pd.read_clipboard(). Its keyword arguments are passed on to pd.read_table().

This allows you to copy structured text directly to a DataFrame or Series. In Excel, the data would look something like this:

Excel Clipboard Data

Its plain-text representation (for example, in a text editor) would look like this:

a   b           c       d
0   1           inf     1/1/00
2   7.389056099 N/A     5-Jan-13
4   54.59815003 nan     7/24/18
6   403.4287935 None    NaT
Simply highlight and copy the plain text above, and call pd.read_clipboard():

>>>
>>> df = pd.read_clipboard(na_values=[None], parse_dates=['d'])
>>> df
   a         b    c          d
0  0    1.0000  inf 2000-01-01
1  2    7.3891  NaN 2013-01-05
2  4   54.5982  NaN 2018-07-24
3  6  403.4288  NaN        NaT

>>> df.dtypes
a             int64
b           float64
c           float64
d    datetime64[ns]
dtype: object
10. Write Pandas Objects Directly to Compressed Format
This one’s short and sweet to round out the list. As of Pandas version 0.21.0, you can write Pandas objects directly to gzip, bz2, zip, or xz compression, rather than stashing the uncompressed file in memory and converting it. Here’s an example using the abalone data from trick #1:

abalone.to_json('df.json.gz', orient='records',
                lines=True, compression='gzip')
In this case, the size difference is 11.6x:

>>>
>>> import os.path
>>> abalone.to_json('df.json', orient='records', lines=True)
>>> os.path.getsize('df.json') / os.path.getsize('df.json.gz')
11.603035760226396
Want t























<BR><BR><BR><BR><BR>


## NumPy
One thing to note at the start: Numpy does _not_ have vectorized string operations by default.  

A `.char` class has been added for some operations, in particular checking sub-string membership within all elements of an array.  Otherwise, use the already excellent built-in Python string operations.  [Here's a good post](https://stackoverflow.com/questions/8089940/applying-string-operations-to-numpy-arrays) about this.

+ `np.convolve` -  
+ `np.percentile(a, q)` -- `a` is the array you are finding the percentiles of, `q` is the quantile you are searching for (i.e. `q=50` returns the value that is at the 50th percentile of `a`)

+ `np.repeat(iter, reps)` - repeats the iterable by element for N reps.  Ex: `np.repeat([55,66,77], 2)` gives [55, 55, 66, 66, 77, 77], each element of the iterable is repeated twice.

+ `np.tile(iter, reps)` - cousin to `np.repeat`, instead of repeating an iterable by each element, it repeats the entire iterable by N reps, "tiling" it.

+ `np.vectorize(func, output_type)` - vectorizes a function to work with arrays:
    ```python
    def maxx(x, y):
        """Get the maximum of two items"""
        if x >= y:
            return x
        else:
            return y

    pair_max = np.vectorize(maxx, otypes=[float])

    a = np.array([5, 7, 9, 8, 6, 4, 5])
    b = np.array([6, 3, 4, 8, 9, 7, 1])

    pair_max(a, b)
    #> array([ 6.,  7.,  9.,  8.,  9.,  7.,  5.])
    ```

+ Once you have an array, you can reorder columns or rows just by specifying the desired order as a slice indexer.
    ```python
    arr = np.arange(9).reshape(3,3)
    In [24]: arr
    Out[24]: array([[0, 1, 2],
                    [3, 4, 5],
                    [6, 7, 8]])

    In [25]: arr[:, [1,0,2]]
    Out[25]: array([[1, 0, 2],
                    [4, 3, 5],
                    [7, 6, 8]])
    ```

Same for simple reversal of a 2D matrix/array:
to reverse rows, just use `arr[::-1, :]`


+ `np.set_printoptions()` -- All arguments optional, many options to modify display of Numpy objects.  Really handy.  
    + `precision` is most common argument, limits decimals.
    + `suppress=True` is also very common as it suppresses sometimes bulky scientific notation. `5.434049e-04` becomes `0.0005434` etc.
    + `threshold` limits total elements displayed.


+ `np.genfromtxt()` - maintain str types inside array structure during import
    ```python
    url = 'https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data'
    iris = np.genfromtxt(url, delimiter=',', dtype='object')
    names = ('sepallength', 'sepalwidth', 'petallength', 'petalwidth', 'species')

    # Print the first 3 rows
    iris[:3]
    #> array([[b'5.1', b'3.5', b'1.4', b'0.2', b'Iris-setosa'],
    #>        [b'4.9', b'3.0', b'1.4', b'0.2', b'Iris-setosa'],
    #>        [b'4.7', b'3.2', b'1.3', b'0.2', b'Iris-setosa']], dtype=object)
    ```
    Since we want to retain the species, a text field, we've have set the `dtype` to object. Had we set `dtype=None`, a 1d array of tuples would have been returned.

+ `np.ptp` - peak to peak.  Max - min along an axis.
+ `np.percentile`

##### Value Counts a'la Pandas
+ `np.unique(arr, return_counts=True)` - returns counts per unique element, like `df.value_counts()`





How to convert a numeric to a categorical (text) array?
Difficulty Level: L2

Q. Bin the petal length (3rd) column of iris_2d to form a text array, such that if petal length is:

```python
Less than 3 --> 'small'
3-5 --> 'medium'
'>=5 --> 'large'
# Input
url = 'https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data'
iris = np.genfromtxt(url, delimiter=',', dtype='object')
names = ('sepallength', 'sepalwidth', 'petallength', 'petalwidth', 'species')
Show Solution
# Input
url = 'https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data'
iris = np.genfromtxt(url, delimiter=',', dtype='object')
names = ('sepallength', 'sepalwidth', 'petallength', 'petalwidth', 'species')

# Bin petallength
petal_length_bin = np.digitize(iris[:, 2].astype('float'), [0, 3, 5, 10])

# Map it to respective category
label_map = {1: 'small', 2: 'medium', 3: 'large', 4: np.nan}
petal_length_cat = [label_map[x] for x in petal_length_bin]

# View
petal_length_cat[:4]
# ['small', 'small', 'small', 'small']
```
