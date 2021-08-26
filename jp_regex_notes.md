# JP RegEx Notes



#### Note we precede all RegEx string queries with an `r` to indicate that our query is using raw string so we don't honor special meanings for certain characters outside of RegEx (in other words, we are preventing confusing double meanings for certain chars).
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
