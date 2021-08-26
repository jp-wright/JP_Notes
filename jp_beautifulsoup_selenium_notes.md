# JP BeautifulSoup and Selenium Notes

<BR>

## BeautifulSoup
##### *---IMPORTANT NOTE---*
If you have tried multiple parsers (lxml, html5lib, html.parser) and are still finding that your `soup` object is missing tags/objects, then the most likely explanation is that the tags/objects you are looking for are not actually part of the orginal, hard-coded HTML for the web page but rather are generated via JavaScript after the page loads.  BeautifulSoup can only parse the actual HTML file that forms the page itself and is blind to all objects created via JavaScript.   
##### *---END NOTE---*

<BR>

BeautifulSoup is an HTML and XML parsing package which is used to access and scrape data from web sites.  It is a fairly robust package but does not handle dealing with the UI of websites, such as entering text in search fields, logging in with passwords, or clicking web buttons, very well.  There are other packages which specialize in interacting with the UI, such as **Selenium**, which we discuss below. Though BeautifulSoup can handle XML, there is also another package called **ElementTree** for Python which specializes in parsing XML.  For more info, see BeautifulSoup's exceptional [documentation](https://www.crummy.com/software/BeautifulSoup/bs4/doc/).
Import into Python, along with `requests` and `time` via:

```python
from bs4 import BeautifulSoup
import requests
import time
```

What follows below is a basic structure I use to scrape web pages:

```python
def get_soup(url):
    r = requests.get(url)
    html = r.content
    soup = BeautifulSoup(html, 'html.parser')
    return soup

# Nice func to group things
def get_tags(url, tag_type):
    tags = soup(tag_type)
    for tag in tags:
        print('TAG:', tag)
        print('URL:', tag.get('href', None))
        print('Contents:', tag.contents)
        print('Attrs:', tag.attrs)

if __name__ == '__main__':

    url = "http://www.great-site-to-scrape.com"
    soup = get_soup(url)
```
<BR><BR>

#### JP's Commonplace BS
```python
tables = soup.find_all('table')
links = soup.find_all('a')


```

<BR><BR>

#### BeautifulSoup Methods and Attributes
For full examples of how to navigate HTML trees with BeautifulSoup, see the following docs:
1. [Going Down](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#going-down)
    + [tag names](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#navigating-using-tag-names)
    + [`.contents`](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#contents-and-children) and [`.children`](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#contents-and-children)
    + [`.descendants`](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#descendants)
    + [`.string`](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#string), [`.strings`](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#strings-and-stripped-strings), [`.stripped_strings`](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#strings-and-stripped-strings)

2. [Going Up](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#going-up)
    + [`.parent`](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#parent) and [`.parents`](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#parents)

3. [Going Sideways](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#going-sideways)
    + [`.next_sibling`](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#next-sibling-and-previous-sibling) and [`.previous_sibling`](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#next-sibling-and-previous-sibling)
    + [`.next_siblings`](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#next-siblings-and-previous-siblings) and [`.previous_siblings`](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#next-siblings-and-previous-siblings)

4. [Going Back and Forth](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#going-back-and-forth)
    + [`.next_element`](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#next-element-and-previous-element) and [`.previous_element`](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#next-element-and-previous-element)
    + [`.next_elements`](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#next-elements-and-previous-elements) and [`.previous_elements`](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#next-elements-and-previous-elements)

5. [Searching the Tree](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#searching-the-tree)
    + [A string](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#a-string)
    + [A regular expression](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#a-regular-expression)
    + [A list](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#a-list)
    + [A function](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#a-function)
    + [`find_all()`](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#find-all)
    + [`name`](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#the-name-argument)
      + This is the same as `find(name='<tag>')`, BS just takes the first positional argument of a `find()` call as the `name` attribute since it is the most common way to find tags. Hence, `find('a')` == `find(name='a')`.  
      + This is the name of the `<tag>`, not the `name=[blah]` attribute of the tag.  So `<a name='happy'>` will have be found with `find('a')` but not with a `find('happy')`,
    + [Keyword](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#the-keyword-arguments)
    + [CSS Classes](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#searching-by-css-class)
      + It’s very useful to search for a tag that has a certain CSS class, but the name of the CSS attribute, “class”, is a reserved word in Python. Using `class` as a keyword argument will give you a syntax error. As of Beautiful Soup 4.1.2, you can search by CSS class using the keyword argument `class_`
    + [`string`](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#the-string-argument)
    + [`limit`](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#the-limit-argument)
    + [`recursive`](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#the-recursive-argument)
    + [`find()`](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#the-recursive-argument)
    + [`find_parent(s)()`](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#find-parents-and-find-parent)
    + [`find_next_sibling(s)()`](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#find-next-siblings-and-find-next-sibling)
    + [`find_previous_sibling(s)()`](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#find-previous-siblings-and-find-previous-sibling)
    + [`find_next()`](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#find-all-next-and-find-next) and [`find_all_next()`](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#find-all-next-and-find-next)
    + [`find_previous()`](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#find-all-previous-and-find-previous) and [`find_all_previous()`](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#find-all-previous-and-find-previous)
    + [CSS Selectors, like `id`](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#css-selectors)
      + `soup.find(id='link1')` (or `find_all()`) returns item with `id=link1`
      + `soup.select("#link1")` returns item with `id=link1`

6. [Modifying the Tree](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#modifying-the-tree)  
<BR>

#### In-Depth Examples of Common BS Uses

To find *all* of a given HTML tags, such as anchor tags (`a`) or tables (`table`), we use `find_all()`.  To find the first one encountered, we just use the `.find()` method.  If we want to churn through an entire list of all the given tags, we usually use the `find.all()` method to get the entire list, and then in a `for` loop we churn through each entry.  Note an equivalent method for `find_all()` is simply to use `soup('<tag>')`, which returns all of the given attribute.  The use of `find_all()` is preferred as it is explicit.  We can also use the dot operator to access tags as `soup` attributes and then sub-tags as we go down the HTML tree, returning the first occurrence of each.  For more specific queries, we can pass in a dictionary of attributes that we'd like to match as well.

Examples:  
```python
soup = get_soup(url)
soup.find('table')          # returns first <table> in soup
soup.find_all('table')      # returns list of all <table>s in soup
soup.find_all('table', {'id': 'games'})     # finds only table tags with id='games'
soup('table')               # calling a tag; same list as find_all('table')
soup.table                  # same as soup.find('table')
soup.table.tr               # drill down to <tr> in <table>
```

<BR>

To access different *attributes* of tags, such as *class*, *id*, *href*, or *name*, we index into the tag object created above using bracket notation.  We can also access all the attributes of a given tag as a dictionary using the `.attrs` attribute.  Note we can modify these attributes in BeautifulSoup and these changed will be shown when the HTML is next rendered:
```python
a_link = soup.find('a')     # returns first <a> tag found
a_link['href']              # returns the "href" attribute of the <a> tag.
a_link['name']              # returns the "name" attr
a_link.name                 # same as a_link['name']
a_link.attrs                # {'href': "www.happyface.com", 'name': "smile so big"}
a_link['name'] = "sadness"  # assign new name for this <a> tag.
del a_link['name']          # deletes this attr
```

<BR>

The [contents](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#contents-and-children) of tag are available as a list with the `.contents` attribute.  This applies to any object, including the `soup` itself.  We can then index into the contents of any tag for a specific attribute, and repeat this step to drill down through each object's children for a specific attribute deeper in the tree.  Depending on the size and complexity of the web page, the contents might be rather large and iterating over a list this size might be prohibitive and inefficient.  For just this purpose, BeautifulSoup has a pair of splendid *generator* attributes, `.children` and `.descendants`.   

Pretend this was our HTML, saved/passed as "jp_html" for the following examples:  
```html
<html>
<head>
    <title>Jonpaul, How Do I Love Thee? Let Me Count The Ways.</title>
</head>
<body>
    <p class="title"><b>Jonpaul's Amazing Qualities</b></p>
    <div class="description">
        <p name="list">
            <table name="qualities">
                <tr name="cols">
                    <td name="1st">1st</td>
                    <td name="2nd">2nd</td>
                    <td name="3rd">3rd</td>
                </tr>
                <tr name="feats">
                    <td>Brain</td>
                    <td>Braun</td>
                    <td>Manly Laugh</td>
                </tr>
            </table>
        </p>
    </div>
</body>
</html>
```

Now let's look at how BeautifulSoup can parse this.

```python
soup = BeautifulSoup(jp_html, 'html.parser')
head_tag = soup.head
head_tag
>>> <head><title>Jonpaul, How Do I Love Thee? Let Me Count The Ways.</title></head>
```  

Here is the output in HTML syntax, just to emphasize the nested tags:  

```html
<head><title>Jonpaul, How Do I Love Thee? Let Me Count The Ways.</title></head>
```
Notice how the output of `head_tag` is still a nested object, with `<title>` nested inside of `<head>`.  If we want to access the `<title>` tag itself, we can use the `.contents` attribute and index in to pluck `<title>` from the resulting list.

```python
head_tag.contents                                   # returns a list
>>> [<title>Jonpaul, How Do I Love Thee? Let Me Count The Ways.</title>]         

title_tag = head_tag.contents[0]                    # title is 0th element
title_tag
>>> <title>Jonpaul, How Do I Love Thee? Let Me Count The Ways./title>

title_tag.contents                                  # gives actual text of <title>
>>> [u'Jonpaul, How Do I Love Thee? Let Me Count The Ways.']
```  

As mentioned above, we can iterate through the list returned by the `.find_all()` method or `.contents` attribute, but for larger pages we can use the `.children` and `.descendants` generators.  `.children` only yields the direct children **only once**, while `.descendants` yields *all* children for *every* tag **recursively** (i.e. repeated for each child which, depending where you start the function call, can be every element on the entire page times the number of total elements. Yikes).  We will use the lone `<table>` tag from the beautifully titled HTML sample code above to demonstrate the difference.

```python
table_tag = soup.table

# using the .children generator to iterate through direct children only ONCE
for child in table_tag.children:
    print(child)
# Output below in HTML
```
`.children` output:
```html
<tr name="cols">
<td name="1st">1st</td>
<td name="2nd">2nd</td>
<td name="3rd">3rd</td>
</tr>
<tr name="feats">
<td>Brain</td>
<td>Braun</td>
<td>Manly Laugh</td>
</tr>
```

```python
# using the .descendants generator to iterate through all children RECURSIVELY
for child in table_tag.descendants:
    print(child)
# Output below
```
`.descendants` output below.  Note the recursive (repeated results).
```html
<tr name="cols">
<td name="1st">1st</td>
<td name="2nd">2nd</td>
<td name="3rd">3rd</td>
</tr>
<td name="1st">1st</td>
1st
<td name="2nd">2nd</td>
2nd
<td name="3rd">3rd</td>
3rd
<tr name="feats">
<td>Brain</td>
<td>Braun</td>
<td>Manly Laugh</td>
</tr>
<td>Brain</td>
Brain
<td>Braun</td>
Braun
<td>Manly Laugh</td>
Manly Laugh
```

<BR>
If a tag has only one child and it is a string, or only one sub-tag whose child is a string, then we can access it with the `.string` attribute.  Otherwise `.string` is set to `None`.  This was formerly `.text` in BeautifulSoup.  Very commonly used attribute.

```python
head_tag.string
>>> 'Jonpaul, How Do I Love Thee? Let Me Count The Ways.'
```

<BR><BR>

When printing the content of a BS object, such as `soup`, we can use the `.prettify()` command to print HTML in a properly indented and spaced format which makes reading it much easier:
```python
print(soup.prettify())
# <html>
#  <head>
#   <title>
#    The Dormouse's story
#   </title>
#  </head>
```
<BR>

The following examples are from the official BeautifulSoup [Quick Start docs](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#quick-start):

Here’s an HTML document we’ll be using as an example throughout this document. It’s part of a story from Alice in Wonderland:

```html
html_doc = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title"><b>The Dormouse's story</b></p>

<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>

<p class="story">...</p>
"""
```  

Running the “three sisters” document through Beautiful Soup gives us a BeautifulSoup object, which represents the document as a nested data structure:

```python
from bs4 import BeautifulSoup
soup = BeautifulSoup(html_doc, 'html.parser')

print(soup.prettify())
# <html>
#  <head>
#   <title>
#    The Dormouse's story
#   </title>
#  </head>
#  <body>
#   <p class="title">
#    <b>
#     The Dormouse's story
#    </b>
#   </p>
#   <p class="story">
#    Once upon a time there were three little sisters; and their names were
#    <a class="sister" href="http://example.com/elsie" id="link1">
#     Elsie
#    </a>
#    ,
#    <a class="sister" href="http://example.com/lacie" id="link2">
#     Lacie
#    </a>
#    and
#    <a class="sister" href="http://example.com/tillie" id="link2">
#     Tillie
#    </a>
#    ; and they lived at the bottom of a well.
#   </p>
#   <p class="story">
#    ...
#   </p>
#  </body>
# </html>
```

Here are some simple ways to navigate that data structure:

```python
soup.title
# <title>The Dormouse's story</title>

soup.title.name
# u'title'

soup.title.string
# u'The Dormouse's story'

soup.title.parent.name
# u'head'

soup.p
# <p class="title"><b>The Dormouse's story</b></p>

soup.p['class']
# u'title'

soup.a
# <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>

soup.find_all('a')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.find(id="link3")
# <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>

```

One common task is extracting all the URLs found within a page’s <a> tags:

```python
for link in soup.find_all('a'):
    print(link.get('href'))
# http://example.com/elsie
# http://example.com/lacie
# http://example.com/tillie
```

Another common task is extracting all the text from a page:

```python
print(soup.get_text())
# The Dormouse's story
#
# The Dormouse's story
#
# Once upon a time there were three little sisters; and their names were
# Elsie,
# Lacie and
# Tillie;
# and they lived at the bottom of a well.
#
# ...
```
<BR><BR>

### Parsing Only Part of a Document
Let’s say you want to use Beautiful Soup look at a document’s *<a>* tags. It’s a waste of time and memory to parse the entire document and then go over it again looking for *<a>* tags. It would be much faster to ignore everything that wasn’t an *<a>* tag in the first place. The `SoupStrainer` class allows you to choose which parts of an incoming document are parsed. You just create a `SoupStrainer` and pass it in to the `BeautifulSoup` constructor as the `parse_only` argument.

(Note that *this feature won’t work if you’re using the html5lib parser*. If you use html5lib, the whole document will be parsed, no matter what. This is because html5lib constantly rearranges the parse tree as it works, and if some part of the document didn’t actually make it into the parse tree, it’ll crash. To avoid confusion, in the examples below I’ll be forcing Beautiful Soup to use Python’s built-in parser.)

#### `SoupStrainer`
The `SoupStrainer` class takes the same arguments as a typical method from Searching the tree: `name`, `attrs`, `string`, and \*\*kwargs. Here are three `SoupStrainer` objects:

```python
from bs4 import SoupStrainer
only_a_tags = SoupStrainer("a")
only_tags_with_id_link2 = SoupStrainer(id="link2")

def is_short_string(string):
    return len(string) < 10

only_short_strings = SoupStrainer(string=is_short_string)
```

<BR><BR>

### HTML Parsers
BeautifulSoup includes the option to use different HTML/XML parsers when creating its `soup`.  This is only important when either speed is an issue or when the HTML you are parsing is invalid (i.e. written with errors in it).  If an HTML file is perfectly valid (has no errors), then all parsers will return the same `soup` object.  However, if the HTML is invalid, each parser handles errors differently, with some being more lenient than others.  If you are having issues with reading an incomplete amount of a web page in, or with certain tags and attributes not appearing correctly, it might be because the original HTML file you're parsing has errors (very common) and trying a different parser might help.  

#### Installing a parser
Beautiful Soup supports the HTML parser included in Python’s standard library, but it also supports a number of third-party Python parsers. One is the lxml parser. Depending on your setup, you might install lxml with one of these commands:  

`$ apt-get install python-lxml`
`$ easy_install lxml`
`$ pip install lxml`  

Another alternative is the pure-Python html5lib parser, which parses HTML the way a web browser does. Depending on your setup, you might install html5lib with one of these commands:  

`$ apt-get install python-html5lib`
`$ easy_install html5lib`
`$ pip install html5lib`  

This table summarizes the advantages and disadvantages of each parser library:

Parser	         | Typical usage	        | Advantages	   | Disadvantages
-----------------|--------------------------|------------------|--------------
Python’s html.parser | `BeautifulSoup(markup, "html.parser")` | + Batteries included <BR> + Decent speed <BR> + Lenient (as of Python 2.7.3 and 3.2.) | Not very lenient (before Python 2.7.3 or 3.2.2) |
lxml’s HTML parser	| `BeautifulSoup(markup, "lxml")` | + Very fast <BR> + Lenient | External C dependency
lxml’s XML parser	| `BeautifulSoup(markup, "lxml-xml")` <BR> `BeautifulSoup(markup, "xml")` | + Very fast <BR> + The only currently supported XML parser | External C dependency
html5lib	| `BeautifulSoup(markup, "html5lib")` | + Extremely lenient <BR> + Parses pages the same way a web browser does <BR> + Creates valid HTML5 | Very slow <BR> External Python dependency  

If you can, I recommend you install and use lxml for speed. If you’re using a version of Python 2 earlier than 2.7.3, or a version of Python 3 earlier than 3.2.2, it’s essential that you install lxml or html5lib–Python’s built-in HTML parser is just not very good in older versions.

Note that if a document is invalid, different parsers will generate different Beautiful Soup trees for it. See [Differences between parsers](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#differences-between-parsers) for details.




















<BR><BR><BR>

## Selenium
A great way to download tables from pages is to use Selenium and Pandas together.  
High level: use Selenium to find the enclosing element around the table.  Grab the HTML from this element, then use Pandas to search that HTML for a <table> tag, and it will auto convert it to a Pandas DF.

Ex:
[after setting up webdriver]

```python
elem = driver.find_element_by_tag_name('div')   ## Find by whatever is right for page (XPath, class, etc)
html = elem.get_attribute('innerHTML')
df = pd.read_html(html)
```


If only one table on the page, you can likely use `elem = driver.find_element_by_tag_name('html')`

If grabbing the correct element proves difficult, you can grab the whole HTML page (above) and parse it with BeautifulSoup, getting the elements you need as well.

Conveniently, `pd.read_html()` should grab ALL tables on a page and return them all in a list.  This allows you to simply grab the entire page, turn into html, and take all tables at once, leaving you to sort them out from the resulting list.

The key is getting the HTML from the Selenium element. By default, Selenium elements do not return HTML; they're objects.  Must use the `get_attribute('innerHTML')` method to grab the HTML from the Selenium element.  

One catch is that depending on the web page itself, the use of 'innerHTML' might not work (according to StackOverflow, with web-based stuff that I'm not familiar with).







#### Download automatically with Firefox, trying with Chrome
from selenium import webdriver
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.chrome.options import Options as chrOptions
from selenium.webdriver.firefox.options import Options as ffxOptions
from selenium.webdriver.firefox.firefox_binary import FirefoxBinary
from selenium.webdriver.common.by import By
from selenium.webdriver.support import expected_conditions as EC


Download file with Selenium
Great guide 1: http://allselenium.info/file-downloads-python-selenium-webdriver/
StackOverflow: https://stackoverflow.com/questions/18439851/how-can-i-download-a-file-on-a-click-event-using-selenium
Try to make automatic download in FireFox
options = ffxOptions()
options.set_preference("browser.download.folderList", 2)
options.set_preference("browser.download.dir", '/Users/jpw/Desktop')
options.set_preference("browser.download.manager.showWhenStarting", False)
options.set_preference("browser.download.useDownloadDir", True)
options.set_preference("browser.helperApps.neverAsk.saveToDisk", 'image/jpeg,image/gif,video/mpeg,video/mp4,video/x-flv')
options.add_argument("User-Agent: research_app:v0.0.1 (by username")
binary = FirefoxBinary("/Applications/Firefox.app/Contents/MacOS/firefox-bin")
browser = webdriver.Firefox(firefox_options=options, firefox_binary=binary)
browser = webdriver.Firefox(firefox_options=options)
browser.get(url)
browser.wait = WebDriverWait(browser, 2)


Try to download in Chrome
opts = chrOptions()
These won't work with Chrome, only Firefox
opts.to_capabilities("browser.download.folderList",2)
opts.to_capabilities("browser.download.dir", '/Users/Desktop')
opts.to_capabilities("browser.download.manager.showWhenStarting", False)
opts.to_capabilities("browser.helperApps.neverAsk.saveToDisk", 'image/jpeg, image/gif, video/mpeg, video/mp4, video/x-flv')
opts.add_argument("User-Agent: research_app:v0.0.1 (by username")
browser = webdriver.Chrome(chrome_options=opts)

url = '<some_url'
browser.get(url)
urllib.urlretrieve(url, url.split('/')[-1])
browser.wait = WebDriverWait(browser, 2)

browser.find_element_by_id('exportpt').click()
browser.find_element_by_id('exporthlgt').click()



opts.add_argument("Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/61.0.3163.100 Safari/537.36")
opts.add_argument("User-Agent: research_app:v0.0.1 (by username)")
Selenium apparently works w/o specifying path to chromedriver executeable..
driver = webdriver.Chrome(’/path/to/chromedriver’, chrome_options = opts)
driver = webdriver.Chrome(chrome_options=opts)
driver.wait = WebDriverWait(driver, 2)

url = '<some_url>'
driver.get(url)
User Name field is called 'edit-name' on FO
user = driver.find_element_by_id('edit-name')
user.click()



### Updating Chromedriver
To use Selenium with Chrome, you must use the Chromedriver version that matches the version of Chrome you have.  Chrome by default is automatically updated as Google releases new versions; depending on the update, this might make your installed Chromedriver out of date.  When this happens, a Chrome Selenium browser will launch but will fail to function properly.  (A common error is: `WebDriverException: Message: unknown error: call function result missing 'value'`).  

To install a new chromedriver binary, download the correct binary version from the following URL:

https://chromedriver.storage.googleapis.com/index.html

As of 10/10/18 the 'correct' version is 2.36.
There _are_ newer versions available, and I'm not exactly sure how to know which version of chromedriver is the correct one for which version of Chrome (for what it's worth, the current installed version of Chrome on my local is "Version 69.0.3497.100").

Once downloaded, unzip the file and move the binary into the `/usr/local/bin` dir, replacing any existing `chromedriver` file.

And, while it should obviously already be included, if it isn't you'll need to add this dir to your PATH variable.  `export PATH=$PATH:/usr/local/bin` in terminal, or add it in your `.bash_profile`.  Technically speaking, you can add the `chromedriver` binary to whatever dir you execute your binaries from.  The `usr/local/bin` is the default.  If you already have a `chromedriver` file installed somewhere, you can locate it with `sudo find -H / -type f -iname chromedriver` in the terminal.

To test that the new version has been installed, restart the terminal and run `chromedriver --version`

Googling the error listed above is a good way to find more information as this is a common and recurring issue.

[Here](https://stackoverflow.com/questions/49162667/unknown-error-call-function-result-missing-value-for-selenium-send-keys-even/49193599) is a good SO post about it.





## Scrapy

add later








## Other Modules and Tricks
### Requests Module
See [docs here](http://docs.python-requests.org/en/master/user/quickstart/)
Note all `headers` values must be a *string*!


### fake-useragent
Deep in the depths of the Python lair there lives a module called [`fake-useragent`](https://pypi.python.org/pypi/fake-useragent) which uses up-to-date browser information to fake as if a browser is sending a request to a server via manipulating the `user agent` field in HTTP requests.
`pip install fake-useragent`

```python
from fake_useragent import UserAgent
ua = UserAgent()

ua.chrome
# Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.2 (KHTML, like Gecko) Chrome/22.0.1216.0 Safari/537.2'

headers = {
    'User-Agent': ua.chrome,
    'From': 'youremail@domain.com' #optional
}
```


However, doing this simply overwrites the existing default headers dict, which is usually undesirable (especially as it might incorporate compression modules like `gzip`, if you've it installed, which greatly reduce the amount of bandwidth each request to the server takes).  Understanding this, we might want to simply update the default headers to include our fake browser tag as the UA.

Use the following to find current defaults `requests.utils.default_user_agent()`.
In Python, it is something like: `'python-requests/2.12.4'`


### gzip
This compresses data on the server-side.  As a user sending requests from a browser/script, we can merely notify the server that we are equipped to handle compressed data (i.e. we can decompress it once we receive it), so please send us the compressed data if you have that ability.  
Many servers simply don't have any compressed data / compression ability, so it won't matter if we request it or not.  But for those that do, it is a nice feature to reduce the bandwidth used by us when scraping, helping reduce our impact on them.  
Gzip has many variations available in many packages, such as for Flask.
`pip search gzip` shows the lot of them.

`gzip` was already installed when I tested it (`import gzip`).  So, that means it is either installed on the MacOS as a common web utility, or part of Python via Anaconda, or part of another package (most likely `requests`).  Here's the link for the zip file if you have to install separately.  

Install gzip:
[zip file](http://www.gzip.org/gz124src.zip)

Add to `requests` module `headers`:
```python
headers = {
    'User-Agent': ua.chrome,
    'From': 'youremail@domain.com' #optional
    'Accept-Encoding': 'gzip', 'deflate'
}
```


### `GET` vs. `POST`
See this post on difference between using `params` for `GET` query string and `data` for `POST` datahttps://github.com/kennethreitz/requests/issues/171
