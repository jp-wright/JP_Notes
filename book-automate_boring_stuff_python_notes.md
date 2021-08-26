# "Automate the Boring Stuff with Python" by Al Sweigart

#### Printing and Output
When printing the contents of the dictionary we can use the `pprint()` function to see the items listed in an easily readable column. If you import the pprint module into your programs, you’ll have access to the `pprint()` and `pformat()` functions that will “*pretty print*” a dictionary’s values. This is helpful when you want a cleaner display of the items in a dictionary than what `print()` provides.

```python
import pprint message = 'It was a bright cold day in April, and the clocks were striking thirteen.'
count = {}

for character in message:
    count.setdefault(character, 0)
    count[character] += 1

pprint.pprint(count)
```

This time, when the program is run, the output looks much cleaner, with the keys sorted.

```python
{' ': 13,
',': 1,
'.': 1,
'A': 1,
'I': 1,
'a': 4,
'b': 1,
'c': 3,
'd': 3,
'e': 5,
[etc...]}
```

The `pprint.pprint()` function is especially helpful when the dictionary itself contains nested lists or dictionaries.
If you want to obtain the prettified text as a string value instead of dis- playing it on the screen, call `pprint.pformat()` instead. These two lines are equivalent to each other:

```python
pprint.pprint(someDictionaryValue)
print(pprint.pformat(someDictionaryValue))
```
<br>

#### Object-oriented programming
Like many languages, Python allows you to define classes that encapsulate data and the functions that operate on them. We’ll use them sometimes to make our code cleaner and simpler. It’s probably simplest to explain them by constructing a heavily annotated example. Imagine we didn’t have the built-in Python set. Then we might want to create our own `Set` class.

What behavior should our class have? Given an instance of `Set`, we’ll need to be able to add items to it, remove items from it, and check whether it contains a certain value. We’ll create all of these as member functions, which means we’ll access them with a dot after a `Set` object:

```python
# by convention, we give classes PascalCase (CamelCase) names
class Set:
    # these are the member functions
    # every one takes a first parameter "self" (another convention)
    # that refers to the particular Set object being used

def __init__(self, values=None):
    """This is the constructor.
    It gets called when you create a new Set.
    You would use it like
    s1 = Set()          # empty set
    s2 = Set([1,2,2,3]) # initialize with values"""

self.dict = {}
    # each instance of Set has its own dict property
    # which is what we'll use to track memberships
    if values is not None:
        for value in values:
            self.add(value)

def __repr__(self):
    """this is the string representation of a Set object
    if you type it at the Python prompt or pass it to str()"""
    return "Set: " + str(self.dict.keys())

# we'll represent membership by being a key in self.dict with value True
def add(self, value):
    self.dict[value] = True

# value is in the Set if it's a key in the dictionary
def contains(self, value):
    return value in self.dict

def remove(self, value):
    del self.dict[value]
```

Which we could then use like:

```python
s = Set([1,2,3])
s.add(4)
print s.contains(4)     # True s.remove(3)
print s.contains(3)     # False
```

#### Functional Tools: Map, Reduce, Filter, and Currying
We will also occasionally use `map`, `reduce`, and `filter`, which provide *functional* alternatives to list comprehensions:

```python
def double(x):
    return 2 * x

xs = [1, 2, 3, 4]
twice_xs = [double(x) for x in xs]        # [2, 4, 6, 8]
twice_xs = map(double, xs)                # same as above
list_doubler = partial(map, double)       # *function* that doubles a list
twice_xs = list_doubler(xs)               # again [2, 4, 6, 8]
```

You can use `map` with multiple-argument functions if you provide multiple lists:

```python
def multiply(x, y):
    return x * y

products = map(multiply, [1, 2], [4, 5])  # [1 * 4, 2 * 5] = [4, 10]
```

Similarly, `filter` does the work of a list-comprehension if:

```python
def is_even(x):
    """True if x is even, False if x is odd"""
    return x % 2 == 0

x_evens = [x for x in xs if is_even(x)]    # [2, 4]
x_evens = filter(is_even, xs)              # same as above
list_evener = partial(filter, is_even)     # *function* that filters a list
x_evens = list_evener(xs)                  # again [2, 4]
```

And `reduce` combines the first two elements of a list, then that result with the third, that result with the fourth, and so on, producing a single result:

```python
x_product = reduce(multiply, xs)           # = 1 * 2 * 3 * 4 = 24
list_product = partial(reduce, multiply)   # *function* that reduces a list
x_product = list_product(xs)               # again = 24
```

When passing functions around, sometimes we’ll want to partially apply (or curry) functions to create new functions. As a simple example, imagine that we have a function of two variables:

```python
def exp(base, power):
    return base ** power
```

and we want to use it to create a function of one variable `two_to_the` whose input is a power and whose output is the result of `exp(2, power)`.

We can, of course, do this with `def`, but this can sometimes get unwieldy:

```python
def two_to_the(power):
    return exp(2, power)
```

A different approach is to use `functools.partial`:

```python
from functools import partial
two_to_the = partial(exp, 2)     # is now a function of one variable
print two_to_the(3)              # 8
```

You can also use partial to fill in later arguments if you specify their names:

```python
square_of = partial(exp, power=2)
print square_of(3)                 # 9
```

It starts to get messy if you curry arguments in the middle of the function, so we’ll try to avoid doing that.



#### enumerate
Not infrequently, you’ll want to iterate over a list and use both its elements and their indices:

```python
# not Pythonic
for i in range(len(documents)):
document = documents[i]
do_something(i, document)

# also not Pythonic
i = 0 for document in documents:
do_something(i, document)
i += 1
```

The Pythonic solution is `enumerate`, which produces tuples (index, element):

```python
for i, document in enumerate(documents):
    do_something(i, document)
```

Similarly, if we just want the indices:

```python
for i in range(len(documents)):
    do_something(i)         # not Pythonic

for i, _ in enumerate(documents):
    do_something(i)         # Pythonic
```

We’ll use this a lot.

#### zip and Argument Unpacking
Often we will need to `zip` two or more lists together. `zip` transforms multiple lists into a single list of tuples of corresponding elements:

```python
list1 = ['a', 'b', 'c']
list2 = [1, 2, 3] zip(list1, list2)        # is [('a', 1), ('b', 2), ('c', 3)]
```

If the lists are different lengths, `zip` stops as soon as the first list ends. You can also “unzip” a list using a strange trick:

```python
pairs = [('a', 1), ('b', 2), ('c', 3)]
letters, numbers = zip(*pairs)
```

The asterisk performs *argument unpacking*, which uses the elements of `pairs` as individual arguments to `zip`. It ends up the same as if you’d called:

```python
zip(('a', 1), ('b', 2), ('c', 3))
```

which returns [('a','b','c'), ('1','2','3')]. You can use argument unpacking with any function:

```python
def add(a, b):
    return a + b

add(1, 2)           # returns 3
add([1, 2])         # TypeError!
add(*[1, 2])        # returns 3
```

It is rare that we’ll find this useful, but when we do it’s a neat trick.

#### args and kwargs
Let’s say we want to create a higher-order function that takes as input some function `f` and returns a new function that for any input returns twice the value of `f`:

```python
def doubler(f):
    def g(x):
        return 2 * f(x)
    return g
```

This works in some cases:

```python
def f1(x):
    return x + 1

g = doubler(f1)
print g(3)          # 8 (== ( 3 + 1) * 2)
print g(-1)         # 0 (== (-1 + 1) * 2)
```

However, it breaks down with functions that take more than a single argument:

```python
def f2(x, y):
    return x + y

g = doubler(f2)
print g(1, 2)    # TypeError: g() takes exactly 1 argument (2 given)
```

What we need is a way to specify a function that takes arbitrary arguments. We can do this with argument unpacking and a little bit of magic:

```python
def magic(*args, **kwargs):
    print "unnamed args:", args
    print "keyword args:", kwargs

magic(1, 2, key="word", key2="word2")

# prints
#  unnamed args: (1, 2)
#  keyword args: {'key2': 'word2', 'key': 'word'}
```
*[JP Note: whoa, this seems like a very powerful ability, if confusing at first]*

That is, when we define a function like this, `args` is a tuple of its unnamed arguments and `kwargs` is a *dict* of its named arguments. It works the other way too, if you want to use a list (or tuple) and dict to supply arguments to a function:

```python
def other_way_magic(x, y, z):
    return x + y + z

x_y_list = [1, 2] z_dict = { "z" : 3 }
print other_way_magic(*x_y_list, **z_dict)   # 6
```

You could do all sorts of strange tricks with this; we will only use it to produce higher-order functions whose inputs can accept arbitrary arguments:

```python
def doubler_correct(f):
    """works no matter what kind of inputs f expects"""
    def g(*args, **kwargs):
        """whatever arguments g is supplied, pass them through to f"""
        return 2 * f(*args, **kwargs)
    return g

g = doubler_correct(f2)
print g(1, 2)       # 6
```
