# "Data Science From Scratch in Python" by Joel Grus


#### Dictionaries
#### defaultdict

Imagine that you’re trying to count the words in a document. An obvious approach is to create a dictionary in which the keys are words and the values are counts. As you check each word, you can increment its count if it’s already in the dictionary and add it to the dictionary if it’s not:

```python
word_counts = {}
for word in document:
    if word in word_counts:
        word_counts[word] += 1
    else:
        word_counts[word] = 1
```

You could also use the “forgiveness is better than permission” approach and just handle the exception from trying to look up a missing key:

```python
word_counts = {}
for word in document:
try:
    word_counts[word] += 1
except KeyError:
    word_counts[word] = 1
```

A third approach is to use `get`, which behaves gracefully for missing keys:

```python
word_counts = {}
for word in document:
    previous_count = word_counts.get(word, 0)
    word_counts[word] = previous_count + 1
```

A fourth approach is to use the `setdefault()` method of the dictionary class:

```python
count = {}

for word in document:
    word_counts.setdefault(word, 0)
    word_counts[word] += 1
```

Every one of these is slightly unwieldy, which is why defaultdict is useful. A defaultdict is like a regular dictionary, except that when you try to look up a key it doesn’t contain, it first adds a value for it using a zero-argument function you provided when you created it. In order to use defaultdicts, you have to import them from collections:

```python
from collections import defaultdict
word_counts = defaultdict(int)          #int() produces 0
for word in document:
    word_counts[word] += 1

```

They can also be useful with list or dict or even your own functions:

```python
dd_list = defaultdict(list)             # list() produces an empty list
dd_list[2].append(1)                    # now dd_list contains {2: [1]}
dd_dict = defaultdict(dict)             # dict() produces an empty dict
dd_dict["Joel"]["City"] = "Seattle"     # {"Joel": {"City" : Seattle"}}
dd_pair = defaultdict(lambda: [0, 0])
dd_pair[2][1] = 1                       # now dd_pair contains {2: [0,1]}
```

These will be useful when we’re using dictionaries to “collect” results by some key and don’t want to have to check every time to see if the key exists yet.


<br>

#### Counter
A `Counter` turns a sequence of values into a `defaultdict(int)`-like object mapping keys to counts. We will primarily use it to create histograms:

```python
from collections import Counter
c = Counter([0, 1, 2, 0])          # c is (basically) {0 : 2, 1 : 1, 2 : 1}
```

This gives us a very simple way to solve our word_counts problem:

```python
word_counts = Counter(document)
```

A `Counter` instance has a most_common method that is frequently useful:

```python
#print the 10 most common words and their counts
for word, count in word_counts.most_common(10):
    print word, count
```

<br>
<br>

#### Sets
Another data structure is the set, which represents a collection of distinct elements:
```python
s = set()
s.add(1)       # s is now { 1 }
s.add(2)       # s is now { 1, 2 }
s.add(2)       # s is still { 1, 2 }
x = len(s)     # equals 2
y = 2 in s     # equals True
z = 3 in s     # equals False
```

We’ll use sets for two main reasons. The first is that in is a very fast operation on sets. If we have a large collection of items that we want to use for a membership test, a set is more appropriate than a list:

```python
stopwords_list = ["a","an","at"] + hundreds_of_other_words + ["yet", "you"]
"zip" in stopwords_list     # False, but have to check every element

stopwords_set = set(stopwords_list)
"zip" in stopwords_set      # very fast to check
```

The second reason is to find the distinct items in a collection:

```python
item_list = [1, 2, 3, 1, 2, 3]
num_items = len(item_list)                # 6
item_set = set(item_list)                 # {1, 2, 3}
num_distinct_items = len(item_set)        # 3
distinct_item_list = list(item_set)       # [1, 2, 3]
```

We’ll use sets much less frequently than dicts and lists.

<br>
<br>

#### Boolean
Python lets you use any value where it expects a Boolean. The following are all `False`:
```python
False
None
[] (an empty list)
{} (an empty dict)
""
set()
0
0.0
```

Pretty much anything else gets treated as `True`. This allows you to easily use `if` statements to test for empty lists or empty strings or empty dictionaries or so on. It also sometimes causes tricky bugs if you’re not expecting this behavior:

```python
s = some_function_that_returns_a_string()
if s:
    first_char = s[0]
else:
    first_char = ""
```

A simpler way of doing the same is:

```python
first_char = s and s[0]
```

Since `and` returns its second value when the first is `True`, the first value when it’s not. Similarly, if `x` is either a number or possibly `None`:

```python
safe_x = x or 0
```

is definitely a number.

<br>
<br>

#### All, Any
Python has an all function, which takes a list and returns True precisely when every element is truthy, and an any function, which returns True when at least one element is `True`:

```python
all([True, 1, {3}])     # True
all([True, 1, {}])      # False, {} is False
any([True, 1, {}])      # True, True is True
all([])                 # True, no False elements in the list
any([])                 # False, no True elements in the list
```

<br>
<br>

#### Sorting
Every Python list has a sort method that sorts it in place. If you don’t want to mess up your list, you can use the sorted function, which returns a new list:

```python
x = [4,1,2,3]
y = sorted(x)     # is [1,2,3,4], x is unchanged
x.sort()          # now x is [1,2,3,4]
```

By default, `sort` (and `sorted`) sort a list from smallest to largest based on naively comparing the elements to one another.
If you want elements sorted from largest to smallest, you can specify a `reverse=True` parameter. And instead of comparing the elements themselves, you can compare the results of a function that you specify with `key`:

```python
# sort the list by absolute value from largest to smallest
x = sorted([-4,1,-2,3], key=abs, reverse=True)  # is [-4,3,-2,1]

# sort the words and counts from highest count to lowest
wc = sorted(word_counts.items(),
            key=lambda (word, count): count,
            reverse=True)
```

<br>
<br>

#### List Comprehensions

Frequently, you’ll want to transform a list into another list, by choosing only certain elements, or by transforming elements, or both. The Pythonic way of doing this is list comprehensions:

```python
even_numbers = [x for x in range(5) if x % 2 == 0]  # [0, 2, 4]
squares      = [x * x for x in range(5)]            # [0, 1, 4, 9, 16]
even_squares = [x * x for x in even_numbers]        # [0, 4, 16]
```

You can similarly turn lists into dictionaries or sets:

```python
square_dict = {x : x * x for x in range(5)}  # { 0:0, 1:1, 2:4, 3:9, 4:16 }
square_set  = {x * x for x in [1, -1]}       # { 1 }
```

If you don’t need the value from the list, it’s conventional to use an underscore as the variable:

```python
zeroes = [0 for _ in even_numbers]      # has the same length as even_numbers
```

A list comprehension can include multiple `fors`:

```python
pairs = [(x, y)
         for x in range(10)
         for y in range(10)]   # 100 pairs (0,0) (0,1) ... (9,8), (9,9)
```

and later `fors` can use the results of earlier ones:

```python
increasing_pairs = [(x, y)                       # only pairs with x < y,
                    for x in range(10)           # range(lo, hi) equals
                    for y in range(x + 1, 10)]   # [lo, lo + 1, ..., hi - 1]
```

<br>
<br>

#### Generators and Iterators
A problem with lists is that they can easily grow very big. `range(1000000)` creates an *actual* list of 1 million elements. If you only need to deal with them one at a time, this can be a huge source of inefficiency (or of running out of memory). If you potentially only need the first few values, then calculating all 1 million is a massive waste. A generator is something that you can iterate over (for us, usually using `for` loops) but whose values are produced only as needed (*lazily*). One way to create generators is with functions and the `yield` operator:

```python
def lazy_range(n):
    """a lazy version of range"""
    i = 0
    while i < n:
        yield i
        i += 1
```

The following loop will consume the yielded values one at a time until none are left:

```python
for i in lazy_range(10):
    do_something_with(i)
```

Python actually comes with a lazy_range function called `xrange`, and in Python 3, range itself is lazy.  In general, you should always use `xrange` instead of `range` when iterating. This means you could even create an infinite sequence:

```python
def natural_numbers():
    """returns 1, 2, 3, ..."""
    n = 1
    while True:
        yield n
        n += 1
```

Although you probably shouldn’t iterate over it without using some kind of break logic.

**TIP:** The flip side of laziness is that you can only iterate through a generator once. If you need to iterate through something multiple times, you’ll need to either recreate the generator each time or use a list.

A second way to create generators is by using for comprehensions wrapped in parentheses:

```python
lazy_evens_below_20 = (i for i in lazy_range(20) if i % 2 == 0)
```

**TIP:** Recall also that every dict has an `items()` method that returns a list of its key-value pairs. More frequently we’ll use the `iteritems()` method, which lazily yields the key-value pairs one at a time as we iterate over it.

<br>
<br>


#### Randomness
As we learn data science, we will frequently need to generate random numbers, which we can do with the `random` module:

```python
import random

four_uniform_randoms = [random.random() for _ in range(4)]

#  [0.8444218515250481,      # random.random() produces numbers
#   0.7579544029403025,      # uniformly between 0 and 1
#   0.420571580830845,       # it's the random function we'll use
#   0.25891675029296335]     # most often
```

The random module actually produces pseudorandom (that is, deterministic) numbers based on an internal state that you can set with `random.seed` if you want to get reproducible results:

```python
random.seed(10)         # set the seed to 10
print random.random()   # 0.57140259469
random.seed(10)         # reset the seed to 10
print random.random()   # 0.57140259469 again
```

We’ll sometimes use `random.randrange`, which takes either 1 or 2 arguments and returns an element chosen randomly from the corresponding range():

```python
random.randrange(10)    # choose randomly from range(10) = [0, 1, ..., 9]
random.randrange(3, 6)  # choose randomly from range(3, 6) = [3, 4, 5]
```

There are a few more methods that we’ll sometimes find convenient. `random.shuffle` randomly reorders the elements of a list:

```python
up_to_ten = range(10)
random.shuffle(up_to_ten)
print up_to_ten
# [2, 5, 1, 9, 7, 3, 8, 6, 4, 0]  (your results will probably be different)
```

If you need to randomly pick one element from a list you can use `random.choice`:

```python
my_best_friend = random.choice(["Alice", "Bob", "Charlie"])     # "Bob" for me
```

And if you need to randomly choose a sample of elements without replacement (i.e., with no duplicates), you can use `random.sample`:

```python
lottery_numbers = range(60)
winning_numbers = random.sample(lottery_numbers, 6)  # [16, 36, 10, 6, 25, 9]
```

To choose a sample of elements with replacement (i.e., allowing duplicates), you can just make multiple calls to `random.choice`:

```python
four_with_replacement = [random.choice(range(10))
                         for _ in range(4)]
# [9, 4, 4, 2]
```

<br>
<br>
