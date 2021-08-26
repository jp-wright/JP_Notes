# JP Datetime Notes


#### Datetime Module
***
Quick and dirty usage, with `.strftime()`:  

```python
>>> from datetime import datetime
>>> datetime.now().strftime('%Y-%m-%d %H:%M:%S')
```
`datetime.year()`
`datetime.date()`
etc.


Note that Pandas `pd.to_datetime()` method is more flexible in what arguments it handles than the standard `datetime.datetime()` Python module.  `datetime.datetime()` can only accept integers in the correct argument positions (or an unzipped list with the `*` command).  Pandas can accept a string in the format of `'2017 September 7 12:33:00'`.  Note that the delimiters must be spaces and that the time must also be included. In another example of the flexibility of the Pandas function, the time can be represented as `4:19PM ET` or something similar -- including the "AM" or "PM" designation as well as a timezone abbreviation -- and Pandas will still parse it correctly.  Nice!
    Technically the Pandas method will return a `timestamp`, but the individual attributes such as `.hour` or `.date()` are still viable.

<BR><BR>

The following is taken from [this excellent post](http://www.marinamele.com/2014/03/13-useful-tips-about-python-datetime.html) on how to use the `datetime` module effectively.  

The Python `datetime` library provides several useful objects to manipulate times and dates. I’ve been using them a lot lately, and I want to share some useful operations that might be useful to you as well 😉

You can find a video version of this post at the bottom of the page 🙂

1. First, let’s import the `datetime` library and create three different kind of objects:
    + `date` object: stores the date
    + `time` object: stores the time
    + `datetime` object: stores both the date and the time

    If we create the `datetime` object first, we can extract its date and time and create the respective objects:

    ```python
    >>> import datetime
    >>> now = datetime.datetime.now()
    >>> today = now.date()
    >>> moment = now.time()
    ```

    If you print each of these items you’ll get something like:

    ```python
    >>> now
    datetime.datetime(2014, 3, 23, 16, 38, 46, 271475)
    >>> today
    datetime.date(2014, 3, 23)
    >>> moment
    datetime.time(16, 38, 46, 271475)
    ```

    Where you can see that the time is 16h 46min and 46.171475 seconds, and today is March 23rd, 2014.  

2. You can also create a date and time objects and obtain a `datetime` object using the `combine` method:

    ```python
    >>> today = datetime.date.today()
    >>> moment = datetime.datetime.now().time()
    >>> now = datetime.datetime.combine(today, moment)
    ```

3. Another interesting object is the `timedelta` object, which can be used to sum or subtract a number of days:

    ```python
    >>> yesterday = today - datetime.timedelta(1)
    ```

    Or it can store a datetime difference between two `datetime` objects:

    ```python
    >>> delta = yesterday - today
    ```

4. `Date` objects have three mandatory arguments (you can change its order by using keys):

    ```python
    >>> my_date = datetime.date(1984, 6, 24)
    >>> my_date = datetime.date(day=24, year=1984, month=6)
    ```

5. `Time` objects don’t have mandatory arguments. These tree statements are equivalent:

    ```python
    >>> my_time = datetime.time()
    >>> my_time = datetime.time(0,0)  # first argument hour, second minute
    >>> my_time = datetime.time(hour=0, minute=0)
    ```

6. `Datetime` objects have the same mandatory arguments as the date objects:

    ```python
    >>> my_datetime = datetime.datetime(year=1984, month=6, day=24)  # Time is set to 0:00
    >>> my_other_datetime = datetime.datetime(1984, 6, 24, 18, 30)
    >>> my_other_datetime = datetime.datetime(year=1984, month=6, day=24, hour=18, minute=30)
    ```

7. Change one `datetime` object to obtain another using the `replace` method:

    ```python
    >>> another_datetime = my_datetime.replace(year=2014, month=1)
    ```

8. Obtain a `datetime` object representing the epoch: 01-01-1970:

    ```python
    >>> epoch = datetime.datetime.utcfromtimestamp(0)
    ```

9. Obtain the number of days and seconds between the epoch and now, or the total number of seconds that have passed:

    ```python
    >>> delta = now - epoch
    >>> days = delta.days

    >>> seconds = delta.seconds
    >>> total_seconds = delta.total_seconds()
    ```

10. Recover now using the number of seconds since epoch using the `utcfromtimestamp` method:

    ```python
    >>> now = datetime.datetime.utcfromtimestamp(seconds)
    ```

11. Write a `date` object as “1984-06-24”:

    ```python
    >>> string_date = str(my_date)
    ```

12. Recover a `date` object from a string like “1984-06-24”:

    ```python
    >>> my_date = datetime.date(*[int(i) for i in string_date.split("-")])
    ```

13. Write a `date` object with a **custom string format** – the `strftime` method:

    ```python
    >>> string_date =  my_date.strftime('%m/%d/%Y')  # This writes "06/24/1984"
    ```

You can also check this video version made by Webucator. Don’t forget to visit their website to check their [Python courses](https://www.webucator.com/programming/python.cfm) 🙂.





## Datetime Abbreviations in `strftime`

Code	| Meaning	                                                         | Example
--------|--------------------------------------------------------|---------
%a      | Weekday as locale’s abbreviated name.	                             | Mon
%A      | Weekday as locale’s full name.                                     | Monday
%w      | Weekday as a decimal number, where 0 is Sunday and 6 is Saturday.  | 1
%d      | Day of the month as a zero-padded decimal number.                  | 30
%-d     | Day of the month as a decimal number. (Platform specific)          | 30
%b      | Month as locale’s abbreviated name.	                             | Sep
%B      | Month as locale’s full name.	                                     | September
%m      | Month as a zero-padded decimal number.                             | 09
%-m     | Month as a decimal number. (Platform specific)                     | 9
%y      | Year without century as a zero-padded decimal number.              | 13
%Y      | Year with century as a decimal number.	                         | 2013
%H      | Hour (24-hour clock) as a zero-padded decimal number.              | 07
%-H     | Hour (24-hour clock) as a decimal number. (Platform specific)      | 7
%I      | Hour (12-hour clock) as a zero-padded decimal number.              | 07
%-I     | Hour (12-hour clock) as a decimal number. (Platform specific)      | 7
%p      | Locale’s equivalent of either AM or PM.                            | AM
%M      | Minute as a zero-padded decimal number.                            | 06
%-M     | Minute as a decimal number. (Platform specific)                    | 6
%S      | Second as a zero-padded decimal number.                            | 05
%-S     | Second as a decimal number. (Platform specific)                    | 5
%f      | Microsecond as a decimal number, zero-padded on the left.	         | 000000
%z      | UTC offset in the form +HHMM or -HHMM (empty string if naive).     | -
%Z      | Time zone name (empty string if the object is naive).              | - 	
%j      | Day of the year as a zero-padded decimal number.	                 | 273
%-j     | Day of the year as a decimal number. (Platform specific)	         | 273
%U      | Week number of the year (Sunday as the first day of the week) <br> as a zero padded decimal number. All days in a new year <br> preceding the first  Sunday are considered to be in week 0.	                                                                     | 39
%W      | Week number of the year (Monday as the first day of the week) <br> as a decimal number. All days in a new year preceding the <br> first Monday are considered to be in week 0.                                                                       | 39
%c      | Locale’s appropriate date and time representation.	             | Mon Sep 30 07:06:05 2013
%x      | Locale’s appropriate date representation.	                         | 09/30/13
%X      | Locale’s appropriate time representation.	                         | 07:06:05
%%      | A literal '%' character.                                           | %
