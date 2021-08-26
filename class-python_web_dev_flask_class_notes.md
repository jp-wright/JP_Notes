# Python Web Dev with Flask - Class Notes (taught by Michael Herman from RealPython)

### Day 1 -- Jan 24, 2017
#### Which objects live on the client side or the server?
**Client**
+ Browser
+ JavaScript (AJAX)
  + jQuery
  + Angular
  + React
+ HTML

**Server**
+ Data Store
+ RDMS
+ NoSQL (Mongo)
+ Python
+ Web Framework
  + Flask (low level, very flexible)
  + Django (high level, enforces MVC/MTV design pattern)
  + Pyramid/Pylons
  + Bottle
  + REST lite
<br><br>

#### Tasks Performed by the Client Side:
+ Authentication and Authorization
+ Templating
+ REST API
+ Session / Management Cookies
+ Email
+ ORM (Database Access, like SQLAlchemy)
+ Chat Window
+ Testing
+ Admin Interface
+ Web Blog and Comments
<br><br>

#### MVC: Model, View, Controller
**Model**
+ Data and Data Access
+ In memory (Redis)
+ External API call

**View**
+ Representation of the desired data on the client side (e.g. if you google "Flowers" you only get data that is related to your query, not the entirety of Google's database).  Whatever data you actually get to view.
+ Templates

**Controller**
+ Whatever manages the data between your model and your view.  The logical part of your app.  The glue that holds it all together.
+ In general, keep the controller "fat" with the gnarly logic, and keep everything else "skinny" with little logic processing.


#### Virtual Environments
Look at "auto env" for Python, which triggers a specified env whenever you run something from a specified folder, etc.  **Note**: I installed, via Anaconda, Python 3.5 virtual environment globally in the `Macintosh HD/anaconda/` directory.  I named it simply **"python3"**. This is perhaps the more "advanced" or more "data science" way of doing it, as it is accessible system-wide, instead of being nested into separate folders for each project, etc.  In order to achieve this global access, you must "source" the virtual environment from this base location.  To do this, use the following:

`source activate [environ name]`
`source deactivate [environ name]`

I am not very clear on how to create more virtual environments with different names, yet.

<br><br><br>


### Day 2 -- Jan 24, 2017




### Day 3 -- Feb 21, 2017
Steps to get a Flask app going:
1. mkdir
2. virtual env
  + add to git (optional)
3. install flask
4. create app.py
5. start making web page.

Steps to create a DB via SQLite for Flask:
1. create db.py file
2. import sqlite3
3. create actual db
  + create table
4. seed with data


#### Day 4 -- Mar 7, 2017
Check out "cookie cutter" for Python -- code generator for boilerplate stuff, like setting up the process below!
Steps to getting our project going:
1. create proj directory:
  + `mkdir 2017-03-07_web_dev_class_day4_heroku`  -- project dir
  + `touch app.py` -- main script
  + `touch db.py`
  + `touch snag_data.py`
  + `touch scheduler.py`
  + `mkdir static` -- create static folder
  + `mkdir templates` -- create *Jinja* templates
  + Make sure *Heroku* is installed **globally** (not in venv): `brew install heroku`

2. set up virtual env, install modules
  + `conda create -n realpy_day4 python=3` -- create "realpy_day4" venv
  + `source activate realpy_day4` -- activate venv
  + `conda install flask` -- install Flask into our venv (alternatively use `pip install flask` if desired or if not using a conda env.)
  + `conda install requests` -- install `requests` module
  + `pip install schedule` -- module for scheduling requests repeatedly (investigate this module and make notes)
  +

3. create basic `app.py` structure (i.e. *Flask* boilerplate)
    ```python
    from flask import Flask, render_template
    import sqlite3
    app = Flask(__name__)

    def drop_table():
        with sqlite3.connect("bitcoin.db") as connection:
            c = connection.cursor()
            c.execute("""DROP TABLE IF EXISTS currency""")
        return True

    def create_db():
        with sqlite3.connect("bitcoin.db") as connection:
            c = connection.cursor()
            c.execute("""CREATE TABLE currency (
                    exchange text,
                    price REAL,
                    tiempo DATETIME DEFAULT CURRENT_TIMESTAMP)"""
                    )
        return True

    @app.route('/')
    def index():
        return "OMG, so much hello world."

    if __name__ == '__main__':
        app.run(debug=True)
        drop_table()
        create_db()
    ```  
    + Note that the current time, labeled `tiempo`, is automatically inserted into the table whenever we `add_data()`.  We do not insert it ourselves. Neat.

4. Use `requests` to get the "bitstamp" API from https://www.bitstamp.net/api/
  + find the API for the USD currency of bitcoin: https://www.bitstamp.net/api/v2/ticker/btcusd/
    ```python
    url = 'https://www.bitstamp.net/api/v2/ticker/btcusd/'
    r = requests.get(url)
    print(r.json())
    ```
  + Put this in db.py
  + Run db.py to get the current bitstamp data   

5. insert current bitstamp data into our `currency` table in the `bitcoin` db.
    ```python
    def get_data():
        url = 'https://www.bitstamp.net/api/v2/ticker/btcusd/'
        r = requests.get(url)
        return r.json()

    def add_data(data):
        print(data)
        with sqlite3.connect('bitcoin.db') as connection:
            c = connection.cursor()
            values = ['bitstamp', data['last']]
            c.execute("INSERT INTO currency (exchange, price) VALUES(?, ?)", values)
        return True
    ```

6. Set up a schedule to automatically get new data.  We are importing the `get_data` and `add_data` functions from our `snag_data` module, then using the `schedule` module to make them run every X period of time (hour in this case).  So, we run our *scheduler* which then runs our `snag_data` for us at set intervals.  We ideally want to run it as a daemon in the background which always gathers our data.
    ```python
    import schedule
    import time

    from snag_data import get_data, add_data

    def test():
        data = get_data()
        if data:
            add_data(data)
            print('heya')

    schedule.every().hour.do(get_data)

    # Not the most efficient way, but a simple way that works for now
    while True:
        schedule.run_pending()
        time.sleep(2)
    ```

7. Heroku - purpose of Heroku is to host your interactive web sites/products.  It has two big perks: it integrates directly with Git/Github, and has its own copy of Python already installed and ready to roll.
  + `git push heroku master` -- this command is used at some point to push our content from github to heroku, which will display our site/stuff/code/db.
  + Once we push up our code from git to Heroku, we still have to tell Heroku where to find things and how to run things, such as `app.py`
  + If a `ProcFile` is an issue, look at the Heroku docs (it will likely also give you a direct URL for it.)  It is the mechanism that determines the order of operations, if you will, for Heroku.  If `app.py` is what runs our *Flask* and data code, then we have to use `web: python app.py`, just as if on a CLI (command line interface) to tell it to actually run `python app.py`.  
  + Also, something about using *Flask*'s development built-in server (accessed by using `debug=True` in `app.run`) on Heroku = BAD.  Use **g unicorn** server as *production* server and not the basic *Flask* server.  In this case, `python app.py` runs our *Flask* commands, which uses the *Flask* dev server which is NOT SAFE for live use.  So, we would actually NEVER use `web: python app.py` in our ProcFile for Heroku without employing some production level server.
  + Debug Heroku errors by looking at Heroku logs:
    + `heroku logs [--tail]` -- `--tail` makes the log live.
  + Heroku will give us a port and we need to grab it, put it into our app.run in our app.py Flask script. Note we would use `debug=False` for actual live production for the reason mentioned above.
        ```python
        app.run(debug=False, port=int(os.environ.get('PORT', 5000)))
        ```
  + `heroku config`
  + free tier of Heroku is markedly slower than paid tier, FYI.
  + There's more on Heroku but we will deal with it next class.


Follow up note about Heroku from Michael Herman (teacher) on Slack:
>We'll have to switch to postgres. i don't think i am going to go down that route. it's too many moving pieces. i may outline the steps for it, but i am not going to show in class.
>
>1. issue last night was that **heroku does not support sqlite**. on that note, you never want to store static files on heroku since they do periodically flush them. s3 is the way to go.
>2. updated code -> https://github.com/realpython/flask-bitcoin-example (this works fine locally)
>3. HW -> https://github.com/realpython/web-dev-for-data-scientists/blob/master/lessons/04-retrieval.md#homework



### Files to add to gitignore
These are files that we do NOT want to ever upload to github.
1. dependencies.  Use `pip freeze` to save all the depencies and then do a `pip install requirements.txt` to install all the modules from the frozen reqs
2. System files like `DS.Store`, any `.` files, any binary files, and frequently `.db` files (since it changes so much it is like a binary file). However in our example today we are going to upload it since it is not private or big, or something.  
3. Sensitive information like API keys, passwords, and auth tokens!



#### Day 5 -- Mar 21, 2017
Use [Bootstrap CDN templates](www.bootstrapcdn.com) for good styles.

Some basic Bokeh imports:
```python
from bokeh.plotting import figure
from bokeh.resources import Inline
from bokeh.embed import components

# prepare some data
x = [1, 2, 3, 4, 5]
y = [6, 7, 2, 4, 5]

# create a new plot with a title and axis labels
p = figure(title="simple line example", x_axis_label='x', y_axis_label='y')

# add a line renderer with legend and line thickness
p.line(x, y, legend="Temp.", line_width=2)
```


Flask-bitcoin template:
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css">
    <title>Flask Bitcoin</title>
    {{ css_resources | safe }}
    {{ js_resources | safe }}
    {{ plot_script | safe }}
  </head>
  <body>
    {{ plot_div | safe }}
  </body>
</html>
```



Karl's shit for displaying some Bokeh shit:
```python
def chart():
    all_data = defaultdict(list)
    query = models.Currency.query.all()
    p = figure(
        width=1080,
        height=600,
        x_axis_type="datetime",
    )

    for row in query:
        all_data[row.exchange].append((row.horah, row.price))


    for i, (exchange, points) in enumerate(all_data.items())
            color = bokeh.palettes.Category20[20][i]
            X,Y = zip(*sorted(points))
            p.line(X,Y, line_width=2, alpha=0.7, legend=exchange, color=color)

    js_resources = INLINE.render_js()
    css_resources = INLINE.render_css()
    script, div = components(p)

    return render_template(
        'index.html',
        js_resources = js_resources,
        css_resources = css_resources,
        plot_script = script,
        plot_div = div,
```
