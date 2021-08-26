# JP Python Optimization Notes
(From _High Performance Python_ by Gorelick and Oszvald)

## Arrays
#### Searching
For any array (list, tuple, dictionary, set in Python), if we know that an object is a member of the array we can look it up directly using Python's indexing with an algorithmic runtime of O(1) (ex: `my_list[3]`).  This, of course, is the best possible runtime for any command.  If we don't know that the desired object is in the array, we have to search for it.  There are various ways to do this and they have different runtime complexities associated with them.  Generally speaking, these searches break down into three different categories: linear, sorted, hash tables.


##### Linear
+ Complexity: O(n)  

As its name implies, linear search simply goes through an array from start to finish and checks each item.  This is the basic search type and it is almost always possible to avoid resorting to this method and its O(n) complexity.  For small arrays, it is fine (the difference between O(n) and O(log n) runtimes, for example, when searching over 10 items is minimal).  But for larger arrays, it is always best to sort then search in order to reduce the runtime.


##### Sorted
+ Sorting Complexity: O(n * log n) to O(n)  
+ Searching Complexity: O(log n) (avg. binary search complexity)  

By sorting an array we learn about the ordering of its contents.  This allows us the modify our search space, reducing our runtime.  There are many types of sorting algorithms and their applicability varies depending on the data and the task.  

Python lists' built-in sorting method uses the Tim sort, which is a hybrid sort that can range from O(n \* log n) at worst to O(n) at best.  Note this is just the _sorting_ component.  Once sorted, we still have to search for the desired value.  Since our list is sorted we can search for our value using binary search: If we know that all numbers in a list are in ascending order, for example, we can simply start halfway into the list and see if the halfway value is larger or smaller than the number we are searching for.  If it is larger, we can search to the left of the halfway point since all those numbers are smaller than the halfway point number, and so is our desired searched-for value.  Vice versa if the halfway point is smaller than our desired value.  This binary search algorithm has an average complexity of O(log n).  


##### Hash Tables
+ Conversion Complexity: O(n)  
+ Search Complexity: O(1)  

When a container is hashed the exact location in memory is stored in a hash table and is used to lookup the value desired directly.  These tables disregard the ordering of the array itself, resulting in a runtime of O(1), which is the best possible result.  However, due to the nature of hash tables knowing the specific location of an object, only dictionaries and sets are hashable in Python.  Thus converting an array into a set or dictionary will allow us to locate an item within it in the optimum runtime of O(1), but the conversion process itself is O(n) at best, as we must convert every item in the array.  In addition, the limits inherent in the set and dictionary data types, such as no repeating keys in a dictionary, might make the choice less wieldy than desired.


##### Bisect Module
The `bisect` module is made for binary searches within a list.  It allows to find the closest element in a list to our desired value in addition to having many methods that making searching a list simple and optimized.  I will expand this section once I can toy around with the methods in the module and write up how they best work.


### Lists vs. Tuples
Apart from the basic mutability of lists and immutability of tuples, and the use cases these factors present, it is important to realize that the immutability of tuples means they are cached at runtime and "don't need to talk to the kernel to reserve memory every time we want to use one."


##### .append()
+ Complexity: O(1)

The `.append()` method is list-only, since tuples don't support direct resizing.  When we `.append()` to a list, Python anticipates that "one append is the beginning of many appends" and instead of appending only a single new item, it appends the specified item and some empty blocks of memory as headroom for future appends.  The amount of headroom is typically small, but still adds up.  In general, the amount of additional headroom added is around ~1/8 of the size of the list (this is my approximation from the equation and __Figure 3-2__ in the book, p. 69).

It is important to note that the `.append()` method only creates a new surplus of headroom once the existing headroom is used up.  For all the `.append()` calls that add an item to the list without filling the existing headroom, the "allocate/copy operations" do not have to be used, thus making the runtime essentially O(1).  For the appends that fill the available headroom, a entirely new list (of size "current list + 1/8 headroom") is actually created in memory, the initial smaller list copied into the new larger one, and the previous list deleted.

Obviously the most deleterious result of this is that there is an excess of memory used in creating new blocks to a list.  It is most apparent in the case of having many small lists or one super large list.  The speed of the actual `.append()` method is O(1), but the memory issues associated with the implementation of the headroom expansion make it suboptimal in many cases.  Again, note that this "headroom" creation occurs __only with the `.append()` method__.



##### Concatenation (`+` operator)
+ Complexity: O(n)

Tuples can't be "appended" to, but they can be subtly resized using operator concatenation.  
```python
In [1]: t1 = tuple(1, 2, 3)
In [2]: t2 = tuple(4, 5, 6)
In [3]: t1 + t2
Out [3]: (1, 2, 3, 4, 5, 6)
```

In this way we have a comparable ability to add new tuples to an existing one.  This tuple concatenation always results in a new tuple object in memory.  But it also means we don't have the "headroom" memory issue we do with lists.  For example, if we have a list of 100 million items, it will take 112.5 million items worth of memory due to its headroom:

Array | Elements    | Memory (elements)
------|-------------|------------------
List  | 100,000,000 | 112,500,007
Tuple | 100,000,000 | 100,000,000

Technically speaking, lists always use more memory than tuples because a list has one additional element of memory allocated to meta information about its shape to allow for faster resizing.  This means if you have 1 million lists and 1 million tuples of the same content, the lists will have an additional 1 million elements of memory used compared to the tuples.

Another small but neat thing to note is that since Python is a Garbage Collected language, when objects are no longer defined / in use, the blocks of memory they used to occupy are freed up to be used by some other object in the future.  One exception is tuples that have 20 elements or fewer.  These tuples will have their space in memory preserved, which means using them repeatedly is even faster and should be an objective when possible.
