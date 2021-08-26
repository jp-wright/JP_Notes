# JP SQL Practice with FFT
This is being used for learning __PostgreSQL__.  All queries are made in PostgreSQL using the `fft_split.sql` data, which is imported into a PostgreSQL database using the method found in my `jp_postgres_notes.md` file from the parent file of `fft_all_class_stats.csv`.  The next step is to learn how to use pgAdmin4 using the same data.  The outputs listed below each query are usually truncated if they're longer than ~10 rows just for brevity in this notes file.

## Materials
1. `fft_split.sql` is the primary file, but it has been split into two separate tables below to practice `JOIN` methods.  Both tables share the Primary Key of `Job_ID`.
    + `fft_names.csv` -- the nominal data for all FFT chars
        + `Job_ID`
        + `Job`
        + `Identity`
        + `Gender`
        + `Level` (numeric)
        + `Role`
        + `Association`
    + `fft_stats.csv` -- the numerical data for all FFT chars
        + `Job_ID`
        + `HPm`
        + `MPm`
        + `PAm`
        + `MAm`
        + `SPm`
        + `M`
        + `J`
        + `CEV`
        + `HPc`
        + `MPc`
        + `PAc`
        + `MAc`
        + `SPc`
    + `fft_all` -- the parent table which combines both of the above tables

## Practice Questions
#### Single Table Nominal Queries -- fft_names Table
1. __Select all the entries that are Squires:__
    ```sql
    SELECT
        *
    FROM
        fft_names
    WHERE
        Job = 'Squire';
    ```    

    ###### Output:
    job_id |  job   | identity | gender | level |  role  | association
    -------|--------|----------|--------|-------|--------|-------------
    1      | Squire | Ramza1   | Male   |     1 | Story  | Heroes
    2      | Squire | Ramza2   | Male   |    10 | Story  | Heroes
    3      | Squire | Ramza3   | Male   |    26 | Story  | Heroes
    4      | Squire | Delita1  | Male   |     1 | Story  | Lion_War
    7      | Squire | Algus    | Male   |    10 | Story  | Lion_War
    4A     | Squire | Squire   | Either |     1 | Squire | Generic

<BR>

2. __Select all entries that have 'Knight' in their Job:__
    ```sql
    SELECT
        *
    FROM
        fft_names
    WHERE
        Job LIKE '%Knight';    -- '%' is the wildcard char
    ```

    ###### Output:
    job_id |      job      |  identity   | gender | level |  role  |  association   
    -------|---------------|-------------|--------|-------|--------|-------------
    5      | Holy_Knight   | Delita2     | Male   |    12 | Story  | Lion_War
    6      | Arc_Knight    | Delita3     | Male   |    30 | Story  | Lion_War
    8      | Arc_Knight    | Zalbag      | Male   |    42 | Story  | Heroes
    9      | Rune_Knight   | Dycedarg    | Male   |    43 | Story  | Lion_War
    11     | Dark_Knight   | Gafgarion   | Male   |    17 | Story  | Lion_War
    12     | Hell_Knight   | Dead_Malak  | Male   |     0 | Story  | Unknown
    17     | Dark_Knight   | Gafgarion   | Male   |    18 | Story  | Lion_War
    19     | Heaven_Knight | Rafa        | Female |    22 | Story  | Heroes
    1A     | Hell_Knight   | Malak       | Male   |    23 | Story  | Heroes
    1B     | Arc_Knight    | Elmdor      | Male   |    41 | Story  | Shrine_Knights

<BR>

3. __Select all entries that have 'Minotaur' or 'Dryad' in their Identity:__
    ```sql
    SELECT
        *
    FROM
        fft_names
	WHERE
        Identity LIKE 'Minotaur%' OR Identity LIKE 'Dryad%';
    ```

    ###### Output:
    job_id |    job     | identity  | gender  | level |   role   | association
    -------|------------|-----------|---------|-------|----------|------------
    7C     | Woodman    | Dryad1    | Monster |     1 | Dryad    | Monster
    7D     | Trent      | Dryad2    | Monster |     1 | Dryad    | Monster
    7E     | Taiju      | Dryad3    | Monster |     1 | Dryad    | Monster
    7F     | Bull_Demon | Minotaur1 | Monster |     1 | Minotaur | Monster
    80     | Minitaurus | Minotaur2 | Monster |     1 | Minotaur | Monster
    81     | Sacred     | Minotaur3 | Monster |     1 | Minotaur | Monster

<BR>

4. __Select all entries that are Male and Lucavi:__
    ```sql
    SELECT
        *
    FROM
        fft_names
    WHERE
        Gender = 'Male' AND Association = 'Lucavi';
    ```

    ###### Output:
    job_id |      job       | identity  | gender | level | role  | association
    -------|----------------|-----------|--------|-------|-------|------------
    3E     | Angel_of_Death | Zalera    | Male   |    44 | Story | Lucavi
    40     | Regulator      | Hashmalum | Male   |    59 | Story | Lucavi
    43     | Impure_King    | Queklain  | Male   |    20 | Story | Lucavi

<BR>

5. __Select all entries that are Female, Shrine Knights, and between Level 35-45:__
    ```sql
    SELECT
        *
    FROM
        fft_names
    WHERE
        Gender = 'Female' AND Association = 'Shrine_Knights' AND Level BETWEEN 35 AND 45;
    ```

    ###### Output:
    job_id |      job      | identity  | gender | level | role  |  association   
    -------|---------------|-----------|--------|-------|-------|---------------
    2A     | Divine_Knight | Meliadoul | Female |    35 | Story | Shrine_Knights
    2D     | Assassin      | Celia     | Female |    38 | Story | Shrine_Knights
    2E     | Assassin      | Lede      | Female |    37 | Story | Shrine_Knights
    2F     | Divine_Knight | Meliadoul | Female |    40 | Story | Shrine_Knights

<BR>

6. __Select all entries that have multiple-word Jobs but aren't members of Glabados or Shrine Knights:__
    We have to escape the underscore "\_" character in the LIKE statement because the __underscore is a wildcard for a single character in SQL.__  (Note the ESCAPE keyword is not recognized by Markdown, apparently.  Also, the escaped character of a backslash "\\" is also the escape character in Markdown, so it has a double backslash to escape the escape character backslash.  Thus, it's a bit confusing to paste into this note file, but in the real SQL query we would only use one backslash inside the ESCAPE keyword argument, which accepts only a single argument).
    ```sql
    SELECT
        *
    FROM
        fft_names
    WHERE
        Job LIKE '%\_%' ESCAPE '\\' AND Association NOT IN ('Glabados', 'Shrine_Knights');
    ```

    ###### Output:
    job_id |       job       |    identity     | gender  | level |    role    | association
    -------|-----------------|-----------------|---------|-------|------------|------------
     5     | Holy_Knight     | Delita2         | Male    |    12 | Story      | Lion_War
     6     | Arc_Knight      | Delita3         | Male    |    30 | Story      | Lion_War
     8     | Arc_Knight      | Zalbag          | Male    |    42 | Story      | Heroes
     9     | Rune_Knight     | Dycedarg        | Male    |    43 | Story      | Lion_War
     0D    | Holy_Swordsman  | Orlandu         | Male    |    33 | Story      | Heroes
     11    | Dark_Knight     | Gafgarion       | Male    |    17 | Story      | Lion_War
     12    | Hell_Knight     | Dead_Malak      | Male    |     0 | Story      | Unknown
     17    | Dark_Knight     | Gafgarion       | Male    |    18 | Story      | Lion_War
     19    | Heaven_Knight   | Rafa            | Female  |    22 | Story      | Heroes
     1A    | Hell_Knight     | Malak           | Male    |    23 | Story      | Heroes

<BR>
<BR>

#### Single Table Numerical Queries -- fft_stats Table
1. __Select entries with an average Move + Jump over 4, and display this value:__  
    Note here the use of "2.0" as the decimal makes the dividend a float, giving us 5.5 for some entries.
    ```sql
    SELECT
        Job_ID, Identity,
        (Move + Jump) / 2.0 AS M_plus_J
    from
        fft_stats
	WHERE
        (Move + Jump) / 2 > 4
    ORDER BY
        M_plus_J;
    ```

    ###### Output:
    job_id |  identity   | m_plus_j      
    -------|-------------|---------
    75     | Ahriman3    | 5.0
    3C     | Velius      | 5.0
    73     | Ahriman1    | 5.0
    74     | Ahriman2    | 5.0
    60     | Chocobo3    | 5.5
    2D     | Celia       | 5.5
    2E     | Lede        | 5.5
    5E     | Chocobo1    | 5.5
    5F     | Chocobo2    | 5.5
    76     | Cockatrice1 | 6.0
    77     | Cockatrice2 | 6.0
    78     | Cockatrice3 | 6.0
    49     | Altima2     | 6.5

<BR>

2. __Find the top 10 entries in terms of combined 'XXm' stats (where bigger is better):__
    ```sql
    SELECT
        Job_ID,
        Identity,
        HPm + MPm + PAm + MAm + SPm AS Total_Multipliers
    FROM
        fft_stats
    ORDER BY
        Total_Multipliers DESC
    LIMIT 10;
    ```

    ###### Output:
    job_id |    identity     | total_multipliers
    -------|-----------------|-------------------
    87     | Behemoth3       |               741
    99     | Demon           |               725
    9A     | Demon           |               703
    10     | Zalmo           |               700
    86     | Behemoth2       |               662
    94     | Onion_Knight(8) |               660
    8A     | Dragon3         |               651
    8C     | Hydra2          |               644
    28     | Wiegraf2        |               640
    1B     | Elmdor          |               640

<BR>

3. __Find the top 10 entries in terms of combined 'XXc' stats (where smaller is better):__
    ```sql
    SELECT
        Job_ID,
        Identity,
        HPc + MPc + PAc + MAc + SPc AS Total_Multipliers
    FROM
        fft_stats
    ORDER BY
        Total_Multipliers ASC
    LIMIT 10;
    ```

    ###### Output:
    job_id |  identity  | total_multipliers
    -------|------------|-------------------
    1C     | Teta       |                 0
    12     | Dead_Malak |                 0
    2C     | (undead)   |                 0
    5E     | Chocobo1   |               154
    87     | Behemoth3  |               160
    86     | Behemoth2  |               162
    90     | Apanda     |               163
    85     | Behemoth1  |               163
    60     | Chocobo3   |               165
    6F     | Skeleton3  |               166

    Uh oh.  There are three entries that have zero for the sum of all their growth constants.  This means those are three unplayable and non-interacting entries in FFT, so we really should remove them from our query since we learn nothing by having them included.  

    We can't use the alias of "Total_Multipliers" in the `WHERE` clause because "Total_Multipliers" doesn't actually exist as a column in the database, so `WHERE` doesn't know where to look for it in order to return results of our `SELECT` statement.  We _can_ use the alias in the `ORDER BY` clause, however, because the "selecting" has already been done and we are now simply ordering the results, which now include a column called "Total_Multipliers" we just created.

4. __Same query, omitting zeros:__
    ```sql
    SELECT
        Job_ID,
        Identity,
        HPc + MPc + PAc + MAc + SPc AS Total_Multipliers
    FROM
        fft_stats
    WHERE
        HPc + MPc + PAc + MAc + SPc > 0
    ORDER BY
        Total_Multipliers ASC
    LIMIT 10;
    ```

    ###### Output:
    job_id | identity  | total_multipliers
    -------|-----------|------------------
    5E     | Chocobo1  |               154
    87     | Behemoth3 |               160
    86     | Behemoth2 |               162
    90     | Apanda    |               163
    85     | Behemoth1 |               163
    60     | Chocobo3  |               165
    6D     | Skeleton1 |               166
    8A     | Dragon3   |               166
    6E     | Skeleton2 |               166
    6F     | Skeleton3 |               166

<BR>

5. __Find the maximum HPm, grouped by Move rating, and show the Identity of these six entries:__  
    There are two ways to do this.  First, use a Self `JOIN`.  Second, use a Window Function (`OVER`).  A Self `JOIN` will allow multiple rows that meet the specified criteria, a Window Function will give only one, even if others are 'tied' with it or exactly equal.  You'll notice this below with Self `JOIN` having 7 rows and Window Function having only 6.  
    In this example, the self JOIN is the better approach because there are multiple characters with the same MOVE rating and the same HPm rating, so taking the MAX HPm rating per move-group does not return a single result per group.  Ideally we would want all results, and the self JOIN gives that.

    For more information on these methods, consult `jp_sql_notes.md`.

    ##### Self `JOIN`
    ```sql
    SELECT
        t1.Move,
        t1.max_HPm,
        t2.Identity
    FROM (
        SELECT
            Move,
            MAX(HPm) AS max_HPm
        FROM
            fft_stats
        GROUP BY
            Move
        ) AS t1         -- Alias aggregate subquery as t1
        INNER JOIN (
            SELECT          -- Can SELECT * too; see below
                Move,
                HPm,
                Identity     
            FROM
                fft_stats
            ) AS t2         -- Alias standard subquery as t2
            ON t1.Move = t2.Move
            AND t1.max_HPm = t2.HPm
    ORDER BY
        t1.Move;
    ```

    ###### Output:
    move | max_hpm | identity  
    -----|---------|-----------
    0    |       0 | Teta
    3    |     181 | Marlboro3
    4    |     170 | Zalbag-Z
    4    |     170 | Dycedarg
    5    |     157 | Dragon3
    6    |     108 | Chocobo1
    8    |     125 | Altima2
    (7 rows)

    A quick note on the explicit columns selected in the second sub-query.  We can simply `SELECT *` and get all columns in the table in this sub-query since we are ultimately only going to return the columns named in the outer-most ("umbrella") `SELECT` statement.  However, when dealing with very large databases, being explicit and selecting _only_ the columns we will ultimately need, as done above, uses less memory and is faster.  Plus, it makes the query itself clearer.  As such, it is considered standard good practice to be explicit in a sub-query even when you don't have to be.

    <BR>

    ##### Window Function
    ```sql
    SELECT
        Move,
        HPm AS max_HPm,
        Identity
    FROM (
        SELECT
            Move,
            FIRST_VALUE(HPm) OVER (PARTITION BY Move ORDER BY HPm DESC) AS HPm,
            FIRST_VALUE(Identity) OVER (PARTITION BY Move ORDER BY HPm DESC) AS Identity
        FROM
            fft_stats
        ) AS fft
    GROUP BY
        Move,
        HPm,
        Identity
    ORDER BY
        Move;
    ```

    ###### Output:
    move | max_hpm | identity  
    -----|---------|-----------
    0    |       0 | Teta
    3    |     181 | Marlboro3
    4    |     170 | Zalbag-Z
    5    |     157 | Dragon3
    6    |     108 | Chocobo1
    8    |     125 | Altima2
    (6 rows)


6. __Find the best PAm per CEV and the number of characters with a given CEV:__
    ```sql
    SELECT
        t2.Identity,
        t2.Job,
        t1.max_PAm,
        t1.CEV,
        t1.Num_Entries
    FROM (
        SELECT
            CEV,
            MAX(PAm) AS max_PAm,
            COUNT(Job_ID) AS Num_Entries
        FROM
            fft_stats
        GROUP BY
            CEV
        ) AS t1
        INNER JOIN (
            SELECT
                Identity,
                Job,
                PAm,
                CEV
            FROM
                fft_stats
            ) AS t2
            ON t1.max_PAm = t2.PAm
            AND t1.CEV = t2.CEV
    ORDER BY
        Num_Entries DESC;
    ```

    ###### Output:
    identity        | max_pam | cev  | num_entries
    ----------------|---------|------|-------------
    Demon           |     135 | 0.10 |          30
    Reis            |     147 | 0.05 |          21
    Hydra3          |     175 | 0.00 |          16
    Minotaur2       |     152 | 0.15 |           9
    Minotaur3       |     173 | 0.12 |           8
    Behemoth3       |     200 | 0.18 |           8
    Monk            |     129 | 0.20 |           6
    Ahriman3        |     126 | 0.11 |           5
    Chocobo2        |     150 | 0.25 |           5
    Onion_Knight(8) |     130 | 0.30 |           4
    Behemoth2       |     149 | 0.13 |           4
    Dragon3         |     147 | 0.08 |           4
    Dragon2         |     130 | 0.09 |           3
    Lede            |     120 | 0.28 |           3
    Zalera          |     145 | 0.24 |           3
    Adramelk        |     139 | 0.19 |           2
    Cockatrice3     |     152 | 0.33 |           2
    Panther2        |     116 | 0.26 |           2
    Beowulf         |     125 | 0.14 |           2
    Ghost2          |      93 | 0.27 |           1
    Elmdor          |     120 | 0.16 |           1
    Rofel           |     122 | 0.21 |           1
    Zalbag-Z        |     125 | 0.22 |           1
    Panther1        |      98 | 0.23 |           1
    Reis            |     120 | 0.07 |           1
    Boar1           |      70 | 0.42 |           1
    Boar2           |      80 | 0.36 |           1
    Boar3           |     160 | 0.39 |           1
    Elidibs         |      50 | 0.03 |           1

<BR>

7. __Find the worst PAm per CEV and the number of characters with a given CEV:__
    ```sql
    SELECT
        t2.Identity,
        t1.max_PAm,
        t1.CEV,
        t1.Num_Entries
    FROM (
        SELECT
            CEV,
            MIN(PAm) AS min_PAm,
            COUNT(Job_ID) AS Num_Entries
        FROM
            fft_stats
        GROUP BY
            CEV
        ) AS t1 INNER JOIN (
            SELECT
                Identity,
                PAm,
                CEV
            FROM
                fft_stats
            ) AS t2
            ON t1.max_PAm = t2.PAm
            AND t1.CEV = t2.CEV
    ORDER BY
        Num_Entries DESC;
    ```

    identity    | min_pam | cev  | num_entries
    ------------|---------|------|-------------
    Rafa        |      80 | 0.10 |          30
    Bard        |      30 | 0.05 |          21
    Teta        |       0 | 0.00 |          16
    Chocobo1    |      97 | 0.15 |           9
    Ahriman1    |      90 | 0.12 |           8
    (undead)    |       0 | 0.18 |           8
    Ovelia      |     100 | 0.20 |           6
    Bomb2       |      85 | 0.11 |           5
    Agrias      |     100 | 0.25 |           5
    Thief       |     100 | 0.25 |           5
    Agrias      |     100 | 0.25 |           5
    Skeleton3   |     125 | 0.13 |           4
    Cockatrice1 |     105 | 0.30 |           4
    Dead_Malak  |       0 | 0.08 |           4
    Alma?       |     100 | 0.24 |           3
    Zalmo       |      90 | 0.09 |           3
    Ghost3      |      97 | 0.28 |           3
    Kletian     |      80 | 0.14 |           2
    Goblin2     |     103 | 0.19 |           2
    Apanda      |     100 | 0.33 |           2
    Ghost1      |      90 | 0.26 |           2
    Elidibs     |      50 | 0.03 |           1
    Rofel       |     122 | 0.21 |           1
    Zalbag-Z    |     125 | 0.22 |           1
    Panther1    |      98 | 0.23 |           1
    Ghost2      |      93 | 0.27 |           1
    Reis        |     120 | 0.07 |           1
    Boar1       |      70 | 0.42 |           1
    Boar2       |      80 | 0.36 |           1
    Boar3       |     160 | 0.39 |           1
    Elmdor      |     120 | 0.16 |           1


<BR>

#### JOIN Table Queries
1. __Show the Job, Identity, and Association of Everyone With a MAm >= 110 and PAm >= 110:__
    ```sql
    SELECT
        fft_names.Job,
        fft_names.Identity,
        fft_names.Association,
        fft_stats.PAm,
        fft_stats.MAm
    FROM
        fft_stats INNER JOIN fft_names
        ON fft_names.Job_ID = fft_stats.Job_ID
    WHERE
        fft_stats.MAm >= 110 AND fft_stats.PAm >= 110
    ORDER BY
        fft_stats.PAm + fft_stats.MAm DESC;
    ```

    ###### Output:
    job             |    identity     |  association   | pam | mam
    ----------------|-----------------|----------------|-----|-----
    Tiamat          | Hydra3          | Monster        | 175 | 120
    Angel_of_Death  | Zalera          | Lucavi         | 145 | 140
    Warlock         | Velius          | Lucavi         | 141 | 140
    Archaic_Demon   | Demon           | Monster        | 130 | 145
    Impure_King     | Queklain        | Lucavi         | 144 | 130
    Wildbow         | Boar3           | Monster        | 160 | 110
    Regulator       | Hashmalum       | Lucavi         | 141 | 125
    Ultima_Demon    | Demon           | Monster        | 135 | 128
    Holy_Dragon     | Reis            | Heroes         | 147 | 110
    Onion_Knight(8) | Onion_Knight(8) | Generic        | 130 | 120
    Arch_Angel      | Altima2         | Lucavi         | 120 | 130
    Ghost_of_Fury   | Adramelk        | Lucavi         | 139 | 110
    Plague          | Ahriman3        | Monster        | 126 | 120
    Assassin        | Lede            | Shrine_Knights | 120 | 125
    Assassin        | Celia           | Shrine_Knights | 120 | 125
    Soldier         | Cloud           | Heroes         | 123 | 120
    Mime            | Mime            | Generic        | 120 | 115
    Holy_Angel      | Altima1         | Lucavi         | 113 | 120
    Dragoner        | Reis            | Heroes         | 120 | 110
    Arc_Knight      | Delita3         | Lion_War       | 120 | 110
    Ochu            | Marlboro2       | Monster        | 110 | 110




2. __Try to find top percent ranks for blah:__
    ```sql

    SELECT
    	t1.Association,
    	t1.Identity,
    	t1.Job,
    	t1.PAm,
    	t1.PAm / cast(t2.max_PAm as float) as pct_max
    FROM
    	(
    	SELECT
    		job_id,
    		Association,
    		Identity,
    		Job,
    		PAm,
    		MAX(PAm) OVER(PARTITION BY Association) as max_PAm
    	FROM
    		fft_stats
    	) as t2
    	JOIN fft_stats as t1
    	on t1.job_id = t2.job_id
    WHERE t1.PAm > 0
    ORDER BY t1.association, pct_max DESC
    ```




    ##### OUTPUT.......
    association   |     identity     |       job       | pam |       pct_max       
  ----------------|------------------|-----------------|-----|---------------------
   Generic        | Dark_Knight      | Dark_Knight     | 140 |                   1
   Generic        | Onion_Knight(8)  | Onion_Knight(8) | 130 |  0.9285714285714286
   Generic        | Monk             | Monk            | 129 |  0.9214285714285714
   Generic        | Samurai          | Samurai         | 128 |  0.9142857142857143
   Generic        | Mime             | Mime            | 120 |  0.8571428571428571
   Generic        | Knight           | Knight          | 120 |  0.8571428571428571
   Generic        | Lancer           | Lancer          | 120 |  0.8571428571428571
   Generic        | Ninja            | Ninja           | 120 |  0.8571428571428571
   Generic        | Dancer           | Dancer          | 110 |  0.7857142857142857
   Generic        | Geomancer        | Geomancer       | 110 |  0.7857142857142857
   Generic        | Archer           | Archer          | 110 |  0.7857142857142857
   Generic        | Thief            | Thief           | 100 |  0.7142857142857143
   Generic        | Priest           | Priest          |  90 |  0.6428571428571429
   Generic        | Squire           | Squire          |  90 |  0.6428571428571429
   Generic        | Chemist          | Chemist         |  75 |  0.5357142857142857
   Generic        | Mediator         | Mediator        |  75 |  0.5357142857142857
   Generic        | Wizard           | Wizard          |  60 | 0.42857142857142855
   Generic        | Oracle           | Oracle          |  50 | 0.35714285714285715
   Generic        | Time_Mage        | Time_Mage       |  50 | 0.35714285714285715
   Generic        | Calculator       | Calculator      |  50 | 0.35714285714285715
   Generic        | Summoner         | Summoner        |  50 | 0.35714285714285715
   Generic        | Onion_Knight(1)  | Onion_Knight(1) |  50 | 0.35714285714285715
   Generic        | Bard             | Bard            |  30 | 0.21428571428571427
   Glabados       | Draclau          | Cardinal        | 100 |                   1
   Glabados       | Funeral          | High_Priest     | 100 |                   1
   Glabados       | Balmafula        | Arc_Witch       | 100 |                   1
   Glabados       | Ajora            | Phony_Saint     | 100 |                   1
   Glabados       | Zalmo            | Holy_Priest     |  90 |                 0.9
   Heroes         | Reis             | Holy_Dragon     | 147 |                   1
   Heroes         | Worker_8         | Steel_Giant     | 140 |  0.9523809523809523
   Heroes         | Beowulf          | Temple_Knight   | 125 |  0.8503401360544217
   Heroes         | Cloud            | Soldier         | 123 |  0.8367346938775511
   Heroes         | Orlandu          | Holy_Swordsman  | 122 |  0.8299319727891157
   Heroes         | Zalbag           | Arc_Knight      | 120 |  0.8163265306122449
   Heroes         | Reis             | Dragoner        | 120 |  0.8163265306122449
   Heroes         | Ramza3           | Squire          | 111 |  0.7551020408163265
   Heroes         | Ramza1           | Squire          | 111 |  0.7551020408163265
   Heroes         | Ramza2           | Squire          | 111 |  0.7551020408163265
   Heroes         | Malak            | Hell_Knight     | 105 |  0.7142857142857143
   Heroes         | Rafa             | Heaven_Knight   | 100 |  0.6802721088435374
   Heroes         | Simon            | Bishop          | 100 |  0.6802721088435374
   Heroes         | Olan             | Astrologist     | 100 |  0.6802721088435374










<BR><BR><BR><BR>

Two ways to select a grouped result, one with WHERE on a sub-query, and the other with HAVING in-line.

```sql
SELECT *
FROM (
	SELECT Association, Gender, count(Gender) as ct_gender, sum(Level) / count(Level) as avg_lvl
	from fft_stats
	GROUP BY Association, Gender
	ORDER BY Association, Gender
	) as tmp
WHERE ct_gender > 5



SELECT Association, Gender, count(Gender) as ct_gender, sum(Level) / count(Level) as avg_lvl
from fft_stats
GROUP BY Association, Gender
HAVING count(Gender) > 5
ORDER BY Association, Gender
```


OUTPUT:
association    | gender  | ct_gender | avg_lvl
---------------|---------|-----------|---------
Generic        | Either  |        23 |       1
Heroes         | Female  |         6 |      22
Heroes         | Male    |        12 |      18
Lion_War       | Male    |        11 |      11
Monster        | Monster |        52 |       1
Shrine_Knights | Male    |         9 |      39
Unknown        | Male    |         7 |       0
