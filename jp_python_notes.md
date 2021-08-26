# JP Python Notes and Tips </p>

## <p style="text-align: right;"> File Input and Output </p>
***
To read or write to a file, we have to open it first.
When opening a file, we can specify the exact (full) file path, which will locate the file regardless of the directory we are operating our script from, or we can use a relative file path, which requires the file to be in the same directory as the script we are running.  If no file of the given name exists, the file will be created in the referenced directory.

```python
filename = raw_input("Enter file:")         # Enter filename
try:
    if len(filename) < 1:
        filename = "de-tab-me.txt"          # Relative path
        filename = '/Users/jpw/desktop/.de-tab-me.txt'      # Full path
        print "Using", filename
except:
    print 'File not found; creating file.'

handle = open(filename)
```

To open the file we use the `open()` command.  By default, Python makes the open command read-only. If  we want to do anything besides simply read the file, we have to enter a second argument to the `open()` function.

<br>

#### Command Line
Use `htop` in a new window to monitor computer performance, such as memory and CPU%.

Use `python -c "import sys; print('\n'.join(sys.path))"` to see which paths are included in your PYTHONPATH.

Or this in a script/ipython:
```python
import os
try:
    user_paths = os.environ['PYTHONPATH'].split(os.pathsep)
except KeyError:
    user_paths = [])
```



#### if / elif / else
Can make single line statements for brevity.  Best to only use these when the statement is simple and short, otherwise readability diminishes and the purpose of the `if` block becomes confusing.
Here's a good example of when to use the single line approach:
```python
if idx < 0:
    z = 5
else:
    z = 1
```
This very basic statement takes up four lines and has much dead space.  Let's turn it into a single line `if` statement.  Note that there are no colons and only one assignment (`=`) for `z`.
```python
z = 5 if idx < 0 else 1
```

#### Truthy and Falsey
You are checking it against the literal value of the boolean `False`. The same as `'A' == False` will not be true.

If you cast it, you'll see the difference:

```python
>>> l = []
>>> l is True
False
>>> l is False
False
>>> l == True
False
>>> l == False
False
>>> bool(l) == False
True
```

The reason `False == False` is true is because you are comparing the same objects. It is the same as `2 == 2` or `'A' == 'A'`.

The difficulty comes when you see things like if l: and this check never passes. That is because you are checking against the truth value of the item. By convention, all these items will fail a boolean check - that is, their boolean value will be `False`:

+ `None`
+ `False` (obviously)
+ Any empty sequence: `''`, `[]`, `()`
+ Any "zero" value: `0`, `0.0`, etc.
+ Any empty collection: `{}` (an empty `dict`)
+ Anything whose `len()` returns a 0  

These are called "falsey" values. **Everything else is "true"**. Which can lead to some strange things like:

```python
>>> def foo():
...   pass
...
>>> bool(foo)
True
```

It is also good to note here that methods that don't return an explicit value, always have `None` as their return type, which leads to this:

```python
>>> def bar():
...   x = 1+1
...
>>> bool(bar)
True
>>> bool(bar())
False
```



##### **Open File Arguments**

Command | Function
------- | --------
`open(filename, 'r')` | open text file in read-only mode, cursor positioned at beginning of the file.
`open(filename, 'w')` | open text file in write-only mode, truncate file to zero length by erasing existing file! Cursor positioned at beginning of the file.
`open(filename, 'a')` | open text file in write-only mode, cursor positioned at end of existing file (no erasure).
`open(filename, 'r+')` | open text file in read-and-write mode, cursor positioned at beginning of the file.
`open(filename, 'w+')` | open text file in read-and-write mode, truncate file to zero length by erasing existing file! Cursor positioned at beginning of the file.
`open(filename, 'a+')` | open text file in read-and-write mode, cursor positioned at end of existing file (no erasure).
`open(filename, 'rb')` | open file to read in binary mode, ideal for .csv files, among others.
`open(filename, 'wb')` | open file to write in binary mode, ideal for .csv files, among others
`open(filename, 'ab')` | open file to append to in binary mode, ideal for .csv files, among others





If you are to use a dual-mode such as `r+` or `w+`, you need to familiarize yourself with the `.seek()` method too, as using both reading and writing operations will move the current position in the file and you'll most likely want to move that current file position explicitly between such operations.

<br>

##### CSV files
One issue that may arise when reading from and writing to files is that the `open()` command treats every file as a text file by default.  Obviously, this is not always the case.  

For example, when working with a .csv file, commonly found in Excel, we need to add an argument to the `open()` function to specify that it is not a text file, else we might encounter unexpected issues such as skipping lines/cells when writing data. (Following info on .csv use is from this StackOverflow post: http://stackoverflow.com/questions/4249185/using-python-to-append-csv-files)

> CSV is a binary format, believe it or not. The csv module is writing the misleadingly-named "lineterminator (should be "rowseparator") as `\r\n` as expected but then the Windows C runtime kicks in and replaces the `\n` with `\r\n` so that you have `\r\r\n` between rows. When you "open" the csv file with Excel it becomes confused

> Always open your CSV files in binary mode (`'rb'`, `'wb'`, `'ab'`), whether you are operating on Windows or Linux. That way, you will get the expected rowseparator (CR LF) even on \*x boxes, your code will be portable, and any linefeeds embedded in your data won't be changed into something else (on writing) or cause dramas (on input, provided of course they're quoted properly).

> Other problems:

> + Don't put your data in your root directory (`C:\`). Windows inherited a hierarchical file system from MS-DOS in the 1980s. Use it.

> + If you must embed hard-wired filenames in your code, use raw strings `r"c:\test.csv"` ... if you had `"c:\test.csv"` the `'\t'` would be interpreted as a TAB character; similar problems with `\r` and `\n`

> + The examples in the Python manual are aligned more towards brevity than robust code.

> Don't do this:

> ```python
w = csv.writer(open('foo.csv', 'wb'))
```

> Do this:

> ```python
f = open('foo.csv', 'wb')
w = csv.writer(f)
```

> Then when you are finished, you have `f` available so that you can do `f.close()` to ensure that your file contents are flushed to disk. Even better: read up on the new `with` statement.





##### Seeking
**Terry says generally that using any sort of byte seeking is a dangerous proposition and he suggests avoiding it.  Says there are almost always other ways which are more reliable and better implemented to achieve same goals.**  

Regarding seek() there's not too much to worry about. First of all, it is useful when operating over an open file. It's important to note that its syntax is as follows:

```python
fp.seek(offset, from_what)
```

where `fp` is the file pointer you're working with; `offset` means how many positions you will move; `from_what` defines your point of reference:

0: means your reference point is the beginning of the file
1: means your reference point is the current file position
2: means your reference point is the end of the file
if omitted, from_what defaults to 0.

Never forget that when managing files, there'll always be a position inside that file where you are currently working on. When just open, that position is the beginning of the file, but as you work with it, you may advance. `seek` will be useful to you when you need to walk along that open file, just as a path you are traveling into.

<BR><BR>

#### Variable-length Arguments - \*args and \*\*kwargs

The following is taken from this [SaltyCrane post](http://www.saltycrane.com/blog/2008/01/how-to-use-args-and-kwargs-in-python/)  
1. \*args = **list** of arguments
2. \*\*kwargs = **dictionary** of arguments

The special syntax, **\*args** and **\*\*kwargs** in function definitions is used to pass a variable number of arguments to a function. The single asterisk form (\*args) is used to pass a *non-keyworded*, variable-length argument **list**, and the double asterisk form (\*\*kwargs) is used to pass a *keyworded*, variable-length argument list, most commonly thought of as a **dictionary**. Here is an example of how to use the non-keyworded form. This example passes one formal (positional) argument, and two more variable length arguments.

```python
def test_var_args(farg, \*args):
    print "formal arg:", farg
    for arg in args:
        print "another arg:", arg

test_var_args(1, "two", 3)
```

Results:

```python
formal arg: 1
another arg: two
another arg: 3
```  

Here is an example of how to use the *keyworded* form. Again, one formal argument and two *keyworded* variable arguments are passed.

```python
def test_var_kwargs(farg, \*\*kwargs):
    print "formal arg:", farg
    for key in kwargs:
        print "another keyword arg: %s: %s" % (key, kwargs[key])

test_var_kwargs(farg=1, myarg2="two", myarg3=3)
```  

Results:

```python
formal arg: 1
another keyword arg: myarg2: two
another keyword arg: myarg3: 3
```  

**Using \*args and \*\*kwargs when calling a function**  

This special syntax can be used, not only in function definitions, but also when calling a function.

```python
def test_var_args_call(arg1, arg2, arg3):
    print "arg1:", arg1
    print "arg2:", arg2
    print "arg3:", arg3

args = ("two", 3)
test_var_args_call(1, *args)
```  

Results:

```python
arg1: 1
arg2: two
arg3: 3
```

Here is an example using the *keyworded* form when calling a function:

```python
def test_var_args_call(arg1, arg2, arg3):
    print "arg1:", arg1
    print "arg2:", arg2
    print "arg3:", arg3

kwargs = {"arg3": 3, "arg2": "two"}
test_var_args_call(1, **kwargs)
```

Results:

```python
arg1: 1
arg2: two
arg3: 3
```

<BR><BR>

<BR><BR><BR>

#### Exiting a Script
***
From the [StackOverflow post](http://stackoverflow.com/questions/19747371/python-exit-commands-why-so-many-and-when-should-each-be-used) on the ways to exit a script.  

1. `quit` raises the `SystemExit` exception behind the scenes.   
    Furthermore, if you print it, it will give a message:
    ```python
    >>> print (quit)
    Use quit() or Ctrl-Z plus Return to exit
    >>>
    ```
    This functionality was included to help people who do not know Python. After all, one of the most likely things a newbie will try to exit Python is typing in `quit`.

    Nevertheless, `quit` should not be used in production code. This is because it only works if the `site` module is loaded. Instead, this function should only be used in the interpreter.  

2. `exit` is an alias for `quit` (or vice-versa). They exist together simply to make Python more user-friendly.  
    Furthermore, it too gives a message when printed:
    ```python
    >>> print (exit)
    Use exit() or Ctrl-Z plus Return to exit
    >>>
    ```
    However, like `quit`, `exit` is considered bad to use in production code and should be reserved for use in the interpreter. This is because it too relies on the `site` module.  

3. `sys.exit` raises the `SystemExit` exception in the background. This means that it is the same as `quit` and `exit` in that respect.

    Unlike those two however, `sys.exit` is considered good to use in production code. This is because the sys module will always be there.  

4. `os._exit` exits the program **without calling cleanup handlers, flushing stdio buffers, etc.** Thus, it is not a standard way to exit and should only be used in special cases. The most common of these is in the child process(es) created by `os.fork`.

    Note that, of the four methods given, only this one is unique in what it does.  

Summed up, all four methods exit the program. However, the first two are considered bad to use in production code and the last is a non-standard, dirty way that is only used in special scenarios. So, if you want to exit a program normally, go with the third method: `sys.exit`.

Or, even better in my opinion, you can just do directly what `sys.exit` does behind the scenes and run:
```python
raise SystemExit
```
This way, you do not need to import `sys` first.
However, this choice is simply one on style and is purely up to you.  
<BR>  

Further note about `sys.exit()`:  
`sys.exit()` will terminate all python scripts, but `quit()` only terminates the script which spawned it. Must use `import sys` for the `sys.exit()` command

> Exit from Python. This is implemented by raising the  `SystemExit` exception, so cleanup actions specified by finally clauses of `try` statements are honored, and it is possible to intercept the exit attempt at an outer level.
>
> The optional argument arg can be an integer giving the exit status (defaulting to zero), or another type of object. If it is an integer, zero is considered “successful termination” and any nonzero value is considered “abnormal termination” by shells and the like. Most systems require it to be in the range 0-127, and produce undefined results otherwise. Some systems have a convention for assigning specific meanings to specific exit codes, but these are generally underdeveloped; Unix programs generally use 2 for command line syntax errors and 1 for all other kind of errors. If another type of object is passed, None is equivalent to passing zero, and any other object is printed to `stderr` and results in an exit code of 1. In particular,  `sys.exit("some error message")` is a quick way to exit a program when an error occurs.
>
> Since `exit()` ultimately “only” raises an exception, it will only exit the process when called from the main thread, and the exception is not intercepted.



#### Module Imports
**For importing from within the same, or a subfolder of the,  directory as the `.py` script:**  

When trying to import a module in a different directory than the script you're trying to run, navigate to the directory of the module itself in the terminal and see if there is a `__init__.py` file.  If there isn't, execute `touch __init__.py` in that directory to create it.  Then in your python script you can drill down into the correct directory (only down from the script's directory) in your import statement by doing a dot-command.   

Example:  
Your script file is called `my_script.py` and is in `~/main` dir and your module is called `my_module.py` and is in `~/main/data` dir, you would go into the `data` dir in terminal, issue `touch __init__.py` command, and then use the following import command atop your python script:  

```python
from data.my_module import [class]  
```

...or whatever you need to import, or `*`.  You can use `..` ahead of the dir to extend the pathname:

```python
df = pd.read_csv('../data/bigfile.csv')
```

**For importing from _anywhere_ on your system:**  
You must similarly have/create a `__init__.py` file in the directory housing the module itself, then do the following for your import at the top of your actual script.  We use `sys.path.insert()` to ensure this path is the first one evaluated when importing in case there is a name conflict with another module in a different path.  Using this, we can have many scripts in many projects import from the same parent file.  I'm not sure if this is considered standard practice, but when you have a file of, say, dictionaries or classes that will be changing frequently, it would be an extreme hassle to have to change 15 copies of the module (or to copy and replace all 15).  Using the `sys.path.insert()` import, one change is disseminated to all 15 projects using the same module.  Of course, as soon as projects start needing variations on a module and specialized for them, it is likely best to give the project its own copy in the `/data` folder for each project.

```python
import sys
sys.path.insert(0, '~/directory/to/your_module')
import your_module
# or
from your_module import object/etc.
```



### <p style="text-align: right;"> Data Structures and Modules </p>
***

'Containers' such as lists, dictionaries, sets, and tuples are called *iterables*.

#### Dictionaries
<br>


#### Generators

When joining the items of a list, it is commonly best to use a generator instead of a list.  Generators store and return items one at a time, regardless of how large the containing list is.  Lists, however, construct and store all items in memory at once.  When you have a list of only 100 items, there is little benefit to using a generator over a list, but when you are dealing with large data sets which might have one million -- or more -- data points, for example, then you are well served not to commit all one million to memory and instead churn through them one at a time.  We create a generator in two ways: the first is to use parenthesis instead of brackets in a 'list' comprehension.  The second is to use the `yield` command (in lieu of the `return` command) in a function, causing the function itself to become a generator.  Here is a great explanation of Python generators: https://jeffknupp.com/blog/2013/04/07/improve-your-python-yield-and-generators-explained/

```python
# Constructing a 'list' of any size via a generator
new_data = (' '.join(word) for word in word_list)

# Make a generator function by using 'yield' instead of 'return'
def get_primes(number):
    while True:
        if is_prime(number):
            yield number
        number += 1
```
<br>

#### Sorting

To sort an iterable, usually a list or dictionary, we can use the `sorted()` function.

```python
liz = [1, 2, 6, 8, 4, 5, 9, 7, 3, 10]
sorted_liz = sorted(liz)
print sorted_liz
# [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

To sort in descending order, add the `reverse=True` argument to the `sorted()` call.
```python
liz = [1, 2, 6, 8, 4, 5, 9, 7, 3, 10]
sorted_liz = sorted(liz, reverse=True)
print sorted_liz
# [10, 9, 8, 7, 6, 5, 4, 3, 2, 1]
```

To sort a dictionary or a multi-dimensional list (one that is composed of other lists or tuples), we have to add a `key` argument using a `lambda` function to our `sorted()` call.

```python
liz = [(1, 'a'), (2, 'c'), (43, 'e'), (65, 'g'), (6, 'r'), (28, 'f'), (9, 'z'), (3, 'y'), (87, 'a')]
# This will sort by the first sub-index, the integers in this case, of each tuple in this list.
sorted_liz = sorted(liz, key=lambda item: item[0], reverse=False)
print sorted_liz
# [(1, 'a'), (2, 'c'), (3, 'y'), (6, 'r'), (9, 'z'), (28, 'f'), (43, 'e'), (65, 'g'), (87, 'a')]
```

If we wanted to sort by the second sub-index, the letters, we'd use item[1] above.  And so on for as many n-dimensions of the iterable which you'd like to sort by.

For a dictionary, we need to add the `.items()` method to the dictionary name in the `sorted()` call.

```python
diction = {'bones': 1, 'c': 1, 'dude': 1, 'ramza': 1, 'lite': 1, 'Trisection': 1, 'ff': 2, 'Y': 3}
# This will sort diction by the second sub-index, in this case the count, in descending order.
sorted_diction = sorted(diction.items(), key=lambda item: item[1], reverse=True)
print sorted_diction
# [('Y', 3), ('ff', 2), ('bones', 1), ('c', 1), ('dude', 1), ('ramza', 1), ('lite', 1), ('Trisection', 1)]
```



#### Counter (from collections module)
```python
c = Counter('abcdeabcdabcaba')  # count elements from a string

c.most_common(3)                # three most common elements
[('a', 5), ('b', 4), ('c', 3)]

sorted(c)                       # list all unique elements
['a', 'b', 'c', 'd', 'e']

''.join(sorted(c.elements()))   # list elements with repetitions
'aaaaabbbbcccdde'

sum(c.values())                 # total of all counts
15

c['a']                          # count of letter 'a'
5

 for elem in 'shazam':           # update counts from an iterable
...     c[elem] += 1                # by adding 1 to each element's count

c['a']                          # now there are seven 'a'
7

 del c['b']                      # remove all 'b'
 c['b']                          # now there are zero 'b'
0

d = Counter('simsalabim')       # make another counter
c.update(d)                     # add in the second counter
c['a']                          # now there are nine 'a'
9

c.clear()                       # empty the counter
c
Counter()
```

Note: If a count is set to zero or reduced to zero, it will remain
in the counter until the entry is deleted or the counter is cleared:

```python
c = Counter('aaabbc')
c['b'] -= 2                     # reduce the count of 'b' by two

c.most_common()                 # 'b' is still in, but its count is zero
[('a', 3), ('c', 1), ('b', 0)]

c = Counter()                          # a new, empty counter
c = Counter('gallahad')                # a new counter from an iterable
c = Counter({'a': 4, 'b': 2})          # a new counter from a mapping
c = Counter(a=4, b=2)                  # a new counter from keyword args

```




### IPython
`%config` will show all options open to configuration.  
See this [list of `%magic` commands](http://ipython.readthedocs.io/en/stable/interactive/magics.html) for options.    

**Important:**  
Using the `%config` command allows you to set config values.
Example: `%config TerminalInteractiveShell.colors = 'LightBG'`  
To see what the current setting is, similarly use: `%config TerminalInteractiveShell.colors`

A useful magic command is `%pylab` (find in via `%pylab?`).  It imports *numpy* and *matplotlib* into your IPython session, and takes an optional `[gui]` argument which allows you to specify the MPL backend you'd like to use (e.g. `qt5`).  It also has a `--no-import-all` optional argument which limits what exactly it imports to help avoid namespace collisions.  You can also initialize IPython with pylab by adding it to the starting command like this: `ipython --pylab`

Many customizations can be saved to a user-made [IPython profile](http://ipython.readthedocs.io/en/stable/development/config.html#config-overview) so they can be quickly loaded for use in the future.

IPython 5.0 brought about some changes to the default colors, both for the shell I/O and for syntax highlighting.  You can change both, of course.  

<br>


##### I/O and Prompt
To change I/O and prompt (such as tracebacks) colors:
`%colors?` will list the available choices.  Use `%colors [color name]` to choose one.  
Currently the four supplied I/O color choices are:  
+ `Neutral`
+ `NoColor`
+ `LightBG`  *
+ `Linux`  

`Neutral` is the default.  For my personal standard soft-blue terminal, `Neutral` clashes badly.  I always use `LightBG`, as it is what I was used to before the change in 5.0 anyway.  

**I have a TextExpander shortcut set up to change both I/O and syntax highlighting colors in Ipython: in the interactive shell, type `ipc` and the commands below will pop up ready:**

`%colors LightBG`
`%config TerminalInteractiveShell.highlighting_style = 'xcode'`


[Official docs](http://ipython.readthedocs.io/en/stable/config/details.html#terminal-colors) on changing terminal colors. And another [documentation](http://ipython.readthedocs.io/en/stable/config/details.html#termcolour).


<br>

##### Syntax Highlighting
For changing syntax highlighting, you can choose from the bundled `pygments` (themes):
This command will show you the available pygments, some of which are replicas of popular text editors like *vim* or *emacs*.  

```python
import pygments
list(pygments.styles.get_all_styles())
```

The currently supplied choices are:  
`default` | `emacs` | `friendly` | `colorful` | `autumn` | `murphy` | `manni` | `monokai` | `perldoc` | `pastie` | `borland` | `trac` | `native` | `fruity` | `bw` | `vim` | `vs` | `tango` | `rrt` | `xcode` | `igor` | `paraiso-light` | `paraiso-dark` | `lovelace` | `algol` | `algol_nu`

A special option is `legacy`.  This option attempts to match the syntax highlighting to your choice of I/O color above.  In this way, your I/O color theme dictates your syntax highlighting.  I believe the legacy option for `LightBG` color theme above is `pastie`, which I do not care for.  

Of these, I find `autumn` and `xcode` to be the most pleasing, with different highlights for different objects, but not the garish background highlighting of strings. `borland`, `murphy`, `igor`, and `paraiso-light` are also all feasible.  Many others are designed for dark backgrounds.  

If you wish to create your own custom pygments for IPython, read this [short how-to](http://pygments.org/docs/styles/#creating-own-styles).


Filesystem navigation, via a magic `%cd` command, along with a persistent bookmark system (using `%bookmark`) for fast access to frequently visited directories




# JP Data Science Projects Random Notes
```python
random.choice # with replacement
random.sample # without replacement
```


##### Dynamically Assign Variable to Instantiated Class
```python
class C:
    pass

obj = C()
setattr(obj, 'xyz', 42)
print(obj.xyz)
```

##### Accessing all attributes in a class Instance
```python
class Car(object):
    def __init__.(self, Year, Color, Make, Horsepower):
        ...

HDN = Car(1999, 'Green', 'Honda', 462)

HDN.__dict__.items()

#output below
[('Year', 1999),
 ('Color', 'Green'),
 ('Make', 'Honda'),
 ('Horsepower', 462)]
```


Here's how to get a list of all the attributes in a given instance of a class.
We can't just loop through it like a list or dictionary, because the attributes are methods inside the class.  But we can do something very similar, using the `getattr()` built-in method.  

```python
# Makes a list of string literals of the attribute *names*
HDN_attribute_list = [att[0] for att in HDN.__dict__.items()]
```

Note that class attributes are accessible by the `__dict__` magic method, which grants access to all the methods of the `dict` class itself (such as dict.items(), dict.keys(), dict.values(), etc.)

```python
# Access the actual values of an instance for all attributes
for itm in HDN_attribute_list:
    print getattr(A1, itm)

# Or access attributes without the list
for itm in HDN.__dict__.keys():
    print getattr(A1, itm)
```

Note that all of these are accessing the *instance* (`HDN`) of the class, not the parent class itself (`Car`).  


##### Dynamically Assign Variable to Objects in List
**This is unsafe, unwise, and inefficient**, but this is how you do it. If in main namespace, can use `locals()`. If you are inside a function, you must use `globals()` because the namespace inside a function is local (there are a couple exceptions).  Either way, this is somewhat pointless (no need to unload the variables from the container you are iterating through!) and can be both unsafe and open to memory leaks and program instability.

```python
liz = ['aa', 'bb', 'cc']
for val in liz:
    locals()["%s" % val] = val

# aa = 'aa'
# bb = 'bb'
# cc = 'cc'
```

The correct approach is to use a *dictionary* (this is basically exactly what they are made for) or even a `class` with the `setattr()` function.  Shown below.

```python
# Use a dictionary, or set an attribute on an object:
d = {}
d['xyz'] = 42
print(d['xyz'])
or if you prefer, use a class:

class C: pass

obj = C()
setattr(obj, 'xyz', 42)
print(obj.xyz)
```

For further reading on why this dynamic assignment is bad, see:
+ [StackOverflow - Dynamically Set Local Variables](http://stackoverflow.com/questions/8028708/dynamically-set-local-variable)
+ [Keep Data Out Of Your Variable Names](http://nedbatchelder.com/blog/201112/keep_data_out_of_your_variable_names.html)


##### How to Select Random Colors for a Plot
```python
import matplotlib.colors as colors
clr_list = np.array(list(colors.cnames)).astype(str)
hue_data = random.sample(clr_list, len(num_data_pts))
```
### NFL Projects
Reading from Excel files is much slower than reading a CSV file (Excel files have much overhead and have to be parsed differently.  As a result, it is always best to convert the .xlsx file to a .csv file if possible for reading into a Pandas dataframe. When reading large spreadsheets, the difference can be staggering.  For example, reading in the DVOA_Big_List, a spreadheet with 89,414 cells, as an .xlsx file takes around 34 seconds, while the same data as a .csv takes only 18.5 ms!)

##### Euclidean Distance and Skew
I used this code to find the Euclidean Distance in a 2D scatter plot of data points in FFT and NFL from a y=x diagonal line to show that the point on the diagonal line which any given data point is closest to (via a right angle with the diagonal line) is exactly 1/2 the sum of the two x-y components for the data point itself.  So, for a data point with `x = 14` and `y = 8`, which gives a sum of `x + y = 22`, then the point on the diagonal line itself which is closest in cartesian coords is exactly 1/2 the sum, so `11`.  This was done in an effort to quantify `skew_dist` for each data point, which measures how far the data point favors the x or y component.  This is directly related to the `skew` itself, which is simply `x - y`, with `0` being perfectly balanced.  Note this comparison of `skew` only makes sense when both the `x` and `y` components of the scatter plot are measured in the same degree.

From `don.py`:
```python
def euc_dist_skew(df):
    df['Off'] = df['Off'] * 2  #Change x-y values and still ratio = 2 for diagonal as long as the values are above 1
    df['Def'] = df['Def']
    df['Skew'] = df['Off'] + df['Def']
    df['Sum'] = df['Off'] - df['Def']
    skew_dic = {}
    df['Skew_Dist'] = None
    # df['pt'] = None
    skew_ind = None

    df.set_index(df['Team'], drop=False, inplace=True)
    for team in df['Team']:
        # point = df.loc[team, 'Sum'] / 2.
        # df.loc[team, 'Skew_Dist'] = euc([df.loc[team, 'Off'], df.loc[team, 'Def']], [point, -point])
        if df.loc[team, 'Off'] > df.loc[team, 'Def']:
            for val in np.arange(0, max(df['Off']), .1):
                skew_dic.setdefault(team, []).append(euc([df.loc[team, 'Off'], df.loc[team, 'Def']], [val, -val]))
                skew_dic.setdefault(team + 'pt', []).append(val)
        else:
            for val in np.arange(min(df['Off']), 0, .1):
                skew_dic.setdefault(team, []).append(euc([df.loc[team, 'Off'], df.loc[team, 'Def']], [val, -val]))
                skew_dic.setdefault(team + 'pt', []).append(val)





    for team in df['Team']:
        if df.loc[team, 'Skew'] == 0:
            df.loc[team, 'Skew_Dist'] = 0
        elif df.loc[team, 'Skew'] < 0:
            df.loc[team, 'Skew_Dist'] = min(skew_dic[team]) * -1
        else:
            df.loc[team, 'Skew_Dist'] = min(skew_dic[team])

        skew_ind = np.array(skew_dic[team]).argmin()
        df.loc[team, 'pt'] = skew_dic[team + 'pt'][skew_ind]


    df['ratio'] = df['Sum'] / df['pt']
    df['Skew_Dist'] = df['Skew_Dist'].astype(float).round(decimals=1)
    df['pt'] = df['pt'].astype(float)
    df['ratio'] = df['ratio'].astype(float).round(decimals=2)

    df.index = [num for num in xrange(32)]

    print df[['Team', 'Sum', 'Skew', 'Skew_Dist', 'pt', 'ratio']]

df_skew = df.sort_values('Skew_Dist', ascending=False)[['Team', 'Skew_Dist', 'Skew', 'pt']]
```


##### Build a dict of lists of words using defaultdict

I saw this question this morning:

> I'm adding words to lists depending on what character they begin with. This seems a silly way to do it, though it works:

```python
        nouns = open('nouns.txt', 'r')
        for word in nouns:
            word = word.rstrip()
            if word[0] == 'a':
                a.append(word)
            elif word[0] == 'b':
                b.append(word)
            elif word[0] == 'c':
                c.append(word)
            # etc...
```

Naturally, the answer here is to make a dictionary keyed by first letter:

```python
words = defaultdict(list)
for word in nouns:
    words[word[0]].append(word)
```





#### String Wildcard Matching
```python
>>> from fnmatch import fnmatch, fnmatchcase
>>> fnmatch('foo.txt', '*.txt')
True
>>> fnmatch('foo.txt', '?oo.txt')
True
>>> fnmatch('Dat45.csv', 'Dat[0-9]*')
True
>>> names = ['Dat1.csv', 'Dat2.csv', 'config.ini', 'foo.py']
>>> [name for name in names if fnmatch(name, 'Dat*.csv')]
['Dat1.csv', 'Dat2.csv']
>>>
```

#### Standardize and Z-score
When to use the sample or population standard deviation?
> We are normally interested in knowing the population standard deviation because our population contains all the values we are interested in. Therefore, you would normally calculate the population standard deviation if:
>  1. you have the entire population
>  2. you have a sample of a larger population, but you are only interested in this sample and do not wish to generalize your findings to the population.

**Important Note About Degrees of Freedom**
As mentioned above, the StDev for an entire population is calculated differently than the StDev for a sample of that population.  For a Sample, in the denominator we use degrees of freedom (i.e. number of samples) minus 1, commonly displayed as "N-1". For a Population, we simply use N.  Since the denominator is smaller in the equation used for a sample, the resulting number is larger.  This is correct as it implies a larger spread (i.e. uncertainty) when we only have a sample rather than the entire population.  (If we have the entire population, there is no data we don't have, so our certainty is higher; or, like in the case of \#2 above, you are only concerned with the sample you have and do not want to extrapolate your findings to a greater population).  
This is important because different Python packages use different defaults for the degrees of freedom, almost always specified with the `ddof` argument.  

Since \#1 is rare, #\2 is likely the more common use case.  Either way, here are two ways to standardize data in Pandas.  



##### \#1 - SciPy Z-Score
Scipy offers a z-score module.  The default is for a *population*, which is `ddof = 0`.  Since this is much less common, you will likely want to set `ddof = 1` for a *sample* StDev.
```python
from scipy.stats import zscore
df.apply(zscore, ddof=1)
```
Use on cols as desired.  

##### \#2 - Pandas calculation
Pandas uses `ddof = 1` by default, but I pass it as an argument below just for clarity.
For an entire DF:  
```python
stdev = (classes - classes.mean()) / classes.std(ddof=1)
```

For a specific col:
```python
classes.loc[:, 'HPm_sig'] = (classes['HPm'] - classes['HPm'].mean()) / classes['HPm'].std(ddof=1)
```


from Jake VanderPlas:
Use the following to print out an excellent confusion matrix of a classification algorithm result.  The trickiest part is the labels for x and y ticks.  Note that here y is simply a pandas series (from a larger DF).  The confusion matrix apparently takes all the unique classes you are trying to predict for given data (binary or multi-class) and sorts it alphabetically (or numerically, though I am unsure how it sorts when the classes you are predicting are string representations of numbers.  First guess is as a string.) So, the best way I found to handle this is to make y_name a sorted list of the unique classes you're predicting.  Also note certain datasets, such as those provided by SKLearn, have a `target_names` attribute which you can access and avoid this hassle.

```python
import seaborn as sns
print 'Confusion_matrix: '
# mat = confusion_matrix(y_test, y_pred)
mat = confusion_matrix(y, y)
y_name = sorted(y.unique())
ax = sns.heatmap(mat.T, square=True, annot=True, fmt='d', cbar=False, xticklabels=y_name, yticklabels=y_name, robust=True, vmax=None, vmin=None)

plt.xlabel('True Label')
plt.ylabel('Predicted Label')
plt.xticks(rotation='vertical')
plt.yticks(rotation='horizontal')
```


### The "print" statement (and related material)
--------------------------------------------
In class 2, we spent a lot of time using the "print" statement.
I wanted to fill in the details on "print" in case people are
confused or wondering.

The basic form of print is
```python
print something
```

Where something is an item or expression of _any_ data type (most
often a string, but potentially anything).  Some data types don't make
sense to print directly (we'll see examples when we get to
object-oriented programming, where a single variable can represent a
_huge_ collection of data and state information), but most that we've
encountered can be printed out by the print statement in a
straightforward way.

So for example

```python
print "hello"              # a string
print 7.5                  # a number
print ["E", "F", "F"]      # a list of strings
print [2, 4, 6, 8]         # a list of numbers
print [[], "", [], "", ""] # a list of empty strings and empty lists
print "That's funny."      # a string again
print len("Shikaripura")   # the result of a function, in this case
                           # the len function, whose results ("return
                           # type") are numbers
print raw_input()          # the result of a function, in this case
                           # the raw_input function, whose results
                           # are strings
print 3.14159*(r*r)        # the result of a calculation, which will
                           # be performed by Python and then printed
                           # out (assuming that some number has been
                           # assigned to the variable r)
```

Remember that `raw_input()`, which reads a line of input
from the user and returns that line as a string, _always returns a
string_.  (That's actually why it's called "raw".)

So sometimes this is good:

```python
print "What is your name?"
name = raw_input()
name_length = len(name)
print "Your name is " + name + " and has " + str(name_length) + " letters."
```

But sometimes it's annoying:

```python
this_year = 2002
print "How old are you?"
age = raw_input()
print "You were probably born around " + str(this_year-age)
```

Oops!  That doesn't work!  How come?

That's right, we have to do

```python
this_year = 2002
print "How old are you?"
age = int(raw_input())
print "You were probably born around " + str(this_year-age)
```

or

```python
this_year = 2002
print "How old are you?"
age = raw_input()
print "You were probably born around " + str(this_year-int(age))
```

in order to be able to do arithmetic on the age (because it has to be
a number type in order to do arithmetic on it).

Does all this make sense?  If not, please experiment and then ask me
questions about it.

One last thing about `print`: if you wanted to have several
successive print statements give output all on one line, you
can put a comma at the _end_ of the print statement to indicate
that the next print statement should output on the same line.

```python
print 2,
print 4,
print 6,
print 8,
print 10
```

Note that you still get a space in between items (but now the items
can be printed with subsequent statements).  Of course, you can do
other things in between:

```python
x = 2
print x,
x = x * 2
print x,
x = x * 2
print x,
x = x * 2
print x,
```



```python
new_dic = sorted(dic.items(), key=lambda item: item[...] reverse=T/F)
```




### iterate through functions in a module
https://stackoverflow.com/questions/21885814/how-to-iterate-through-a-modules-functions






## View function definition
Use double question marks.  
`function_name??` will print the source code.

For something imported you can use
```python
import inspect
print(inspect.getsource(function_name))
```








#### Copy module
The copy module contains a function called copy that can duplicate any object!
`copy()` is a shallow copy (only the object itself, not the objects it refers to) and `deepcopy()` is, yes, a deep copy that also copies all the objects the copied object refers to.

```python
>>> import copy
>>> p2 = copy.copy(p1)
>>> p1 is p2
False

>>> p1.some_data is p2.some_data
True

p3 = copy.deepcopy(p1)
>>> p1 is p3
True
```




#### Decorators
http://simeonfranklin.com/blog/2012/jul/1/python-decorators-in-12-steps/
> Python supports a feature called function closures which means that inner functions defined in non-global scope remember what their enclosing namespaces looked like at definition time.



#### `locals()`
This is foolish, unnecessary, and potentially unsafe, but this is technically how you would put all the Jobs into the global namespace (i.e. outside of the container) from a list or dict for FFT.
```python
for job, obj in job_dic.iteritems():
    # locals()["%s" % job[0:3]] = obj
    locals()[f"{job[:3]}"] = obj

A1 = Job('Archer')
A2 = Job('Archer')
M1 = Job('Monk')
```



#### `itertools`
Use `.chain(*iterable)` to flatten a list of lists efficiently
Flatten a list of lists via list comp with [y for x in list_of_lists for y in x]
