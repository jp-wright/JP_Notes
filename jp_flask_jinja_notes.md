# JP Flask and Jinja Notes
<BR>


## Flask
***


## Flask Mega Tutorial by the Flask Guru
https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-xviii-deployment-on-the-heroku-cloud




### ** Debug Mode and Safety! **
**_NEVER_** enable `debug=True` on live production machines / scripts!  It allows any user on the network to arbitrarily run any Python code in the script.  This is intended to allow for you to debug any issues that arise, and is quite convenient.  But for anyone with malicious intent, it allows them to do basically anything they choose with your server and your local, if they're good enough at hacking.  

```python
if __name__ == '__main__':
    flask_app.run(debug=True) # only enable debugger when running on local and testing yourself.
```
<BR>

### Using Jinja Templates in Flask
Simply use the following import in your Flask `.py` scripts to interact with Jinja templates:
```python
from flask import Flask, render_template
```
<BR>

### Dynamic URLs
Flask can handle [dynamic URLs](http://flask.pocoo.org/docs/0.12/quickstart/#variable-rules), where different code is run based on the URL the user enters (or follows a link to).  We designate which parts of the URL will be dynamic with angled brackets (aka "alligator brackets"), `<[dynamic_portion]>`.  (A "dynamic" URL is really just a variable passed in from a URL's name -- that's all).  By default, this returns a string object, but since the purpose of dynamic URLs is to frequently do something with them in code, we can convert (i.e. cast) them into a few different object types.  Below is an example of using dynamic URL variables to perform a basic calculation and `return` the data to a Jinja template for further use.

```python
@app.route('/calc/<operator>/<int:num1>/<int:num2>')
def operation(operator, num1, num2):
    try:
        if operator == 'add':
            tot = num1 + num2
        elif operator == 'subtract':
            tot = num1 - num2
        elif operator == 'multiply':
            tot = num1 * num2
        elif operator == 'divide':
            tot = num1 / num2
    except:
        ValueError: "Unrecognized operator"

    data = {
    'operator': operator,
    'num1': num1,
    'num2': num2,
    'solution': tot
    }

    return render_template('index.html', data=data)
```
<BR>

Here's a list of Flask Dynamic URL converters:  

Converter | Definition
----------|-----------
`string`| accepts any text without a slash (the default)
`int`	| integers
`float`	| floating point values
`path`	| string with slashes (`<path:some/great/path/>`)
`any`	| matches one of the items provided
`uuid`	| accepts UUID strings

<BR><BR>

### Passing Variables/Objects to a Jinja Template
Here is some code showing how to pass different types of variables / objects from Flask to a Jinja template.  

```python
from flask import Flask, render_template
import pandas as pd

@app.route('/')
def landing():
    x = 462
    car_dic = {'Honda': 'Accord', 'Porsche': '911'}
    df = pd.read_csv('/party_time.csv')
    return render_template('home_page.html', var1=x, cars=car_dic, df=df)
    # objects will now be accessible inside the Jinja template "home_page.html".  
```
<BR>

### Passing JSON to a Jinja Template
Here is some code showing how to pass JSON from Flask to a Jinja template.  

```python
from flask import Flask, jsonify, render_template

@app.route('/')
def landing():
    x = 462
    car_dic = {'Honda': 'Accord', 'Porsche': '911'}
    return jsonify(car_dic)

    # or
    data = jsonify(car_dic)
    return render_template('home_page.html', data=data)
```

There are two methods to pass JSON to a Jinja template from Flask (from [Stack Overflow](http://stackoverflow.com/questions/34077761/what-is-the-difference-between-jsonify-and-tojson-in-flask)):
+ `jsonify` - returns a `Response` object to be returned from the Flask view as a JSON response to the client.
+ `tojson` - If you want to use JSON but not directly return it to the client by using, you can use the `tojson` filter to convert an object to JSON in the template.

When you need to have JSON in your template, such as to use it in a JavaScript variable, you should use `tojson`. When you need to return a JSON response to the client you should use `jsonify`.

<BR>

### Redirecting and Referencing URLs
`url_for`

<BR>

### Accessing Databases
There are multiple ways to access a database in Flask.  For this example, let's say there is a database named `fft_classes.db` in the same directory as the main **flask** app `.py` script. Most common is SQL and its variants.  Flask has good integration with SQL and is especially designed with using **SQLAlchemy** in mind, which requires a bit more overhead and will be covered further down.  Some basic SQL examples are below.   

1. Here are the imports we will use in various database access methods.  

    ```python
    from flask import Flask, render_template
    import sqlite3
    import os
    import pandas as pd
    app = Flask(__name__)
    app.config.update(DATABASE=os.path.join(app.root_path, 'fft_classes.db')) # for #3 below
    ```
<BR>

2. A basic way to get info from a database is simply to pass it as a `.db` file directly.  This is fine for very small applications, but since the database is named in this file / module only (the main `flask_app.py` controller file), it won't be accessible to other modules as easily (or maybe not as securely, which is important).  We examine an alternative, more globally-minded approach in \#3 below.

    ```python
    @app.route('/db_query')
    def db_query():
        # Make Connection object with DB
        with sqlite3.connect("fft_classes.db") as conn:

            # Create Cursor
            c = conn.cursor()

            # execute SQL statement selecting all data from the table "fft_class_stats" inside the `fft_classes.db` database:
            c.execute("""SELECT * FROM fft_class_stats""")

            # save all data from our SQL query to a variable using the cursor method `fetchall()`; now we can access and iterate through this in HTML template!
            rows = c.fetchall()
            return render_template('db_query.html', rows=rows)    
    ```
<BR>

3. In this example we use the environment variable for the Flask app we've created in this script via the built-in Flask app configuration method shown in the `import` statements in \#1 above.  The code is exactly the same save for the `connection` statement, which now references the `app.config` variable we assigned with our imports above.

    ```python
    @app.route('/db_query')
    def db_query():
        with sqlite3.connect(app.config['DATABASE']) as conn:
            c = conn.cursor()
            c.execute("""SELECT * FROM fft_class_stats""")
            rows = c.fetchall()
            return render_template('db_query.html', rows=rows)    
    ```
<BR>

Once returned to the Jinja template, we can interact with the database using Jinja's syntax.  
For example:  

```html
<body>
    {% if rows %}
    {% for row in rows %}
    <li>{{row[2]}} has a job class of {{row[1]}} and is {{row[3]}} gender.</li>
    {% endfor %}
```

For the first row in the databse, this would return the following:

```html
'Ramza1' has a job class of 'Squire' and is 'Male' gender.
```

...and so on for each row.

<BR><BR><BR>







## Jinja
***
Jinja is a web page templating engine designed for use with Python, which means Python web frameworks such as Flask integrate with Jinja very well.  In short, Jinja allows you to make templates of HTML web pages to streamline replication and allow modular modification based on the exact URL of a given domain.  Jinja also allows you to pass it variables and data structures which you can interact with in a largely Pythonic manner in any given template.  Jinja is extremely well documented in a very user-friendly way.  Here is the primary [documentation on templating](http://jinja.pocoo.org/docs/2.9/templates/), and is the source of much of what is below.

Here are the default delimiters used in templates:
Delimiter | Purpose
----------|--------
`{% ... %}` | for Statements, such as `if` and `for` logic.
`{{ ... }}` | for Expressions to print to the template output, such as variables.
`{# ... #}` | for Comments not included in the template output
`#  ... ##` | for Line Statements

<BR>

### Accessing Variables in Templates
When we pass a [variable](http://jinja.pocoo.org/docs/2.9/templates/#variables) to a Jinja template, we can access the attributes and methods of these objects inside the template by using the familiar *dot operator*, `.`, method.  For example, if we pass in a dictionary called `car_dic = {'Honda': 'Accord', ...}` with a list of cars, we can access a value for a given key by using the dot operator where we name the key as an attribute. In this case, `{{car_dic.Honda}}` returns `Accord`.

<BR>

### Filters
Variables can be modified by [filters](http://jinja.pocoo.org/docs/2.9/templates/#filters). Filters are separated from the variable by a pipe symbol (`|`) and may have optional arguments in parentheses. Multiple filters can be chained. The output of one filter is applied to the next.  

For example, `{{ name|striptags|title }}` will remove all HTML Tags from variable name and title-case the output `(title(striptags(name)))`.  We will

Filters that accept arguments have parentheses around the arguments, just like a function call. For example: `{{ listx|join(', ') }}` will join a list with commas `(str.join(', ', listx))`.

Jinja has an extensive list of built-in filters ready to use.  See their great [documentation](http://jinja.pocoo.org/docs/2.9/templates/#list-of-builtin-filters) for full examples.

<BR>

### Escaping HTML in Templates
When printing / displaying HTML from variables passed into the template via certain objects, such as the object created with the `df.to_html()` function, you must include the `|safe` keyword filter, as shown below, in order to designate the variable as *safe* to parse for HTML.  This is known as passing, or encasing a variable, in **Markup**. Otherwise Jinja templates will auto-escape variables containing HTML, meaning they will be displayed as one long, continuous string without formatting (i.e. a complete mess).  Below is a snippet showing the basic functioning of this filter.

We use the `df.to_html()` method to generate the raw HTML that describes our DF.  We pass this HTML string object from our controller Python script to our template `db_view.html` file.  Clearly we want to render the HTML product when we display our web page, and not the actual HTML code.  (Note that the `df.to_html()` method also allows for passing CSS styling to the resulting HTML via the `classes` argument).

```python
# This is in the controller.py script for Flask, which feeds info to Jinja for templating
@app.route('/db_view')
def db_print():
    df = pd.read_csv('/Users/jpw/Dropbox/Data_Science/jp_projects/2017-01-25_fft_class_stats/data/fft_class_stats_jp.csv', header=0)

    df_html = df.to_html(header=True, index=False, bold_rows=True, border=0, justify='left', formatters=None) # returns a string object of raw HTML of the DF

    return render_template('db_view.html', df=df_html) # Passes our HTML of the DF to our template as `df`
```

Now in our template, `db_view.html`:
We've passed our HTML string object in as df.  In Jinja2 we access variables by enclosing them in double curly brackets.  

Recall that Jinja2 templates enable auto-escaping by default, which means our template won't see our df variable as actual HTML, but instead will see it as a long blob of a string that will be displayed literally, instead of interpreted as HTML to be rendered.  If we just write `{{df}}`, Jinja2 treats it like any normal variable and auto-escapes it fully. Here's a snippet of what the user actually sees on the web page when we do this with a standard HTML table:

```html
<html>
    <body>
        {{df}}  <!-- access variable, fully auto-escaped by Jinja. //-->
        ...     
```

...gives the user this messy string.  

```bash
<table border="0" class="dataframe"> <thead> <tr style="text-align: left;"> <th>Job_ID</th> <th>Job</th> <th>Identity</th> <th>Gender</th> <th>Level</th> <th>Role</th> <th>Association</th> <th>HPm</th> <th>MPm</th> <th>PAm</th> <th>MAm</th> <th>SPm</th> <th>M</th> <th>J</th> <th>CEV</th> <th>HPc</th> <th>MPc</th> <th>PAc</th> <th>MAc</th> <th>SPc</th> </tr> </thead> <tbody> <tr> <td>1</td> <td>Squire</td> <td>Ramza1</td> <td>Male</td> <td>1</td> <td>Story</td> <td>Heroes</td> <td>125</td> <td>105</td> <td>111</td> <td>102</td> <td>107</td> <td>4</td> <td>3</td> <td>0.10</td> <td>11</td> <td>11</td> <td>50</td> <td>48</td> <td>95</td> </tr> <tr>
```

Really great looking web page... not.  
Clearly, this is not what we are intending.  
To resolve this and actually display the interpreted HTML table instead of the HTML code, we add the `| safe` filter (after a pipe `|`) to our variable in the template, as so:

```html
<html>
    <body>
        {{df | safe}}  <!-- tells Jinja to not escape this variable; render the contents //-->
        ...    
```

...now gives the user the following table, as desired.  

```html
Job_ID  Job     Identity    Gender  Level	Role	Association	HPm	MPm	PAm	MAm	SPm	M	J	CEV	 HPc MPc PAc MAc SPc
1       Squire	Ramza1      Male	1       Story	Heroes	    125	105	111	102	107	4	3	0.10 11	 11	 50	 48	 95
2       Squire	Ramza2	    Male	10      Story	Heroes      125	105	111	102	107	4	3	0.10 11	 11	 50	 48	 95
3       Squire	Ramza3	    Male	26      Story	Heroes      125	105	111	102	107	4	3	0.10 11	 11	 50	 48	 95
4       Squire	Delita1     Male	1       Story	Lion_War    130	100	120	100	100	4	3	0.05 10	 11	 50	 50	 100
```
(Note that if the table isn't alined properly in these notes, it is when it is rendered as HTML by Jinja and displayed on the web page).

For more information on escaping HTML in Jinja, see the official Jinja [documentation](http://jinja.pocoo.org/docs/2.9/templates/#html-escaping).  

Here is an excerpt from Flask's documentation regarding Jinja templating as well:

> ### Controlling Autoescaping
> Autoescaping is the concept of automatically escaping special characters for you. Special characters in the sense of HTML (or XML, and thus XHTML) are `&`, `>,` `<,` `"` as well as `'`. Because these characters carry specific meanings in documents on their own you have to replace them by so called “entities” if you want to use them for text. Not doing so would not only cause user frustration by the inability to use these characters in text, but can also lead to security problems. (see Cross-Site Scripting (XSS)).  
>
> Sometimes however you will need to disable autoescaping in templates. This can be the case if you want to explicitly inject HTML into pages, for example if they come from a system that generates secure HTML like a markdown to HTML converter.  
>
> There are three ways to accomplish that:  
>
> 1. In the Python code, wrap the HTML string in a **Markup** object before passing it to the template. This is in general the recommended way.
> 2. Inside the template, use the `|safe` filter to explicitly mark a string as safe HTML (`{{ myvariable|safe }}`)
> 3. Temporarily disable the autoescape system altogether.  
>
> To disable the autoescape system in templates, you can use the `{% autoescape %}` block:
>
> ```html
> {% autoescape false %}
>     <p>autoescaping is disabled here
>     <p>{{ will_not_be_escaped }}
> {% endautoescape %}
> ```
>
> Whenever you do this, please be very cautious about the variables you are using in this block.
