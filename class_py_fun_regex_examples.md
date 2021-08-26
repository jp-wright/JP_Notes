# Regex Examples - Python Fundamentals
By Jonpaul Wright


These notes are not meant to be an exhaustive overview of Regex.  Instead, they are just showing you a very brief example of how Regex can be used to parse string data that is sometimes messy, repetitive, or atypical.  For full information please see [this excellent tutorial](http://www.regular-expressions.info/tutorial.html) which I have used multiple times myself.


## Using Regex
There are countless cases for usage of Regex.  It's flexible and powerful.  The commands used in it are actually not too complex nor numerous.  When using it, you will want to test it.  Usually this is done in IPython for easier stuff, such as the things in this note file, or you can use really snazzy sites like [this awesome one](http://regexr.com/) for dynamic syntax highlighting showing you the results of your Regex queries.



##### Basic Regex Expressions
Note these definitions are meant to be the most universal available.  Different flavors of Regex will have different additional characters included in their usages.
Expression  | Meaning
------------|-------
`\d`        | Includes *all digits*; equal to `[0-9]`
`\w`        | Stands for *word* character; equal to `[A-Za-z0-9_]` (i.e. all letters, numbers, and underscore)
`\s`        | Stands for *whitespace* character; equal to [ \t\r\n\f]. That is: `\s` matches a *space*, a *tab*, a *line break*, or a *form feed*.
`\D`        | Negated `\d`; matches any *non-digit* character (i.e. the exact opposite of `\d`). Equal to `[^\d]`
`\W`        | Negated `\w`; matches any *non-word* character  (i.e. the exact opposite of `\w`). Equal to `[^\w]`
`\S`        | Negated `\s`; matches any *non-whitespace* character  (i.e. the exact opposite of `\s`). Equal to `[^\s]`
`^`         | Matches the *beginning* of a line.
`$`         | Matches the *end* of a line.
`.`         | Matches *any* character.
`*`         | *Repeats* a character *zero or more* times; `a*` matches "any number of `a` characters"
`*?`        | *Repeats* a character *zero or more* times (non-greedy)
`+`         | *Repeats* a character *one or more* times; `a+` matches "at least one `a` character"
`+?`        | *Repeats* a character *one or more* times (non-greedy)
`[aeiou]`   | Matches a single character in the list *set* (here, any of `a`, `e`, `i`, `o`, `u`)
`[^aeiou]`  | Matches a single character *not in* the listed *set* (here, any character not a vowel...)
`()`        | A *capture group*, where string *extraction* occurs



##### Brief Notes
Shorthand character classes can be used both inside and outside the square brackets. `\s\d` matches a whitespace character followed by a digit. `[\s\d]` matches a single character that is either whitespace or a digit. When applied to 1 + 2 = 3, the former regex matches ` 2` (space two), while the latter matches 1 (one). `[\da-fA-F]` matches a hexadecimal digit, and is equivalent to `[0-9a-fA-F]` if your flavor only matches ASCII characters with `\d`.

Be careful when using the negated shorthands inside square brackets. `[\D\S]` is not the same as `[^\d\s]`. The latter matches any character that is neither a digit nor whitespace. It matches `x`, but not `8`. The former, however, matches any character that is either not a digit, or is not whitespace. Because all digits are not whitespace, and all whitespace characters are not digits, `[\D\S]` matches any character; digit, whitespace, or otherwise.


## Common Use Cases
Regex can do many things that Python's built in string methods can't.  Here are some good examples of Regex use cases.  Note that there are different "flavors" of Regex, and the Regex used in Python is slightly different than other languages usages.  Sometimes the syntax is a bit different, and in some cases new abilities are enabled for Regex in different types.  See [this site](http://www.regular-expressions.info/refflavors.html) for more about these differences.  (One common thing you'll see is a forward slash `/` at the beginning and end of a Regex query, which isn't necessary in Python).

#### Matching a Password or Username
Pretend a username or password had to be alphanumeric, could also contain an underscore or hyphen, and had to be 5 to 21 characters long.  We could find it using Regex.
```python
In [1]: string = 'omg_b3st_us3rn8me'
In [2]: re.findall(r'^[a-z0-9_-]{5,21}', string)
```

Let's break down the expression above.

Characters | Meaning
-----------|--------
`^` | the carat (or 'hat') character says "start our Regex query from the beginning of the string/line"
`[...]` | square brackets match all characters inside of them
`[a-z0-9_-]` | the characters inside the square brackets we are matching with.  All letters, all digits, an underscore, and a hyphen.
`{3,18}` | the number of characters we are accepting, anywhere from 5 to 21.


<BR>

This is good if our data or input is clean, but what if our data is in a jumble or has parts to it that we don't want to include?  For example, pretend our input had the username and password and email all in one line, as follows:
```python
In [1]: data = 'user:omg_b3st_us3rn8me,pass:am8zing_passwurd_dood-jimmy.johnson@hbtc.com%'
```

Using built in Python string methods we can't just split on "`:`" because there isn't a colon in the email.  There isn't a comma separating the password and email, so we can't split on it either.  One thing we do is to use Regex to pull each of these out of the string.

```python
# every re.findall command returns a list, even if it only has one entry
In [1]: re.findall(r'(?<=:)[a-z0-9_-]{5,21}', data)
Out [1]: ['omg_b3st_us3rn8me', 'am8zing_passwurd_dood']

# Assign the results to 'user' and 'pw' variables via unpacking
In [2]: user, pw = re.findall(r'(?<=:)[a-z0-9_-]{5,21}', data)
In [3]: user
Out [3]: 'omg_b3st_us3rn8me'

In [4]: pw
Out [4]: 'am8zing_passwurd_dood'

# Now get email
In [5]: re.findall(r'(?<=-)\S+(?=%)', data)
Out [5]: ['jimmy.johnson@hbtc.com']
```

You'll notice we used two new components in our queries, both inside of parentheses:  
`(?<=-)`  
`(?=%)`  

I discuss these below.



## Advanced Techniques
In non-JS we can use a *look-behind* technique to start a match at certain characters, but not include them in the returned result.
`(?<=chars)`

Pretend we want to grab only the words inside the parentheses below. We can use a *look-behind* to start matching on the open-parenthesis but not return the parenthesis itself.  We must escape the parentheses with a backslash (`\(` and `\)`) inside the RegEx query because the parentheses have a special RegEx meaning themselves.  (Note: there are multiple ways to do this query, this is just to demonstrate the *look-behind* method).

```python
In [1234]: phrase = "I just love chocolate and eggs (only green ones)!"
In [1235]: re.findall(r'(?<=\()[^\)]+', phrase)
Out[1235]: ['only green ones']
```  


Match before certain char(s):  
```python
In [1]: game = 'Atlanta Falcons at Denver Broncos - October 9th, 2016'
In [2]: re.findall(r'\w+(?=\sat)', game)
Out[2]: ['Falcons']
```
