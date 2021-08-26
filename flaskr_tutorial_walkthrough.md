# Introducing ["Flaskr"](http://flask.pocoo.org/docs/0.12/tutorial)
This tutorial will demonstrate a blogging application named Flaskr, but feel free to choose your own less Web-2.0-ish name ;) Essentially, it will do the following things:

1. Let the user sign in and out with credentials specified in the configuration. Only one user is supported.
2. When the user is logged in, they can add new entries to the page consisting of a text-only title and some HTML for the text. This HTML is not sanitized because we trust the user here.
3. The index page shows all entries so far in reverse chronological order (newest on top) and the user can add new ones from there if logged in.

SQLite3 will be used directly for this application because it’s good enough for an application of this size. For larger applications, however, it makes a lot of sense to use [SQLAlchemy](http://www.sqlalchemy.org/), as it handles database connections in a more intelligent way, allowing you to target different relational databases at once and more. You might also want to consider one of the popular NoSQL databases if your data is more suited for those.

<br><br>

## Step 0: [Creating the Folders](http://flask.pocoo.org/docs/0.12/tutorial/folders/#tutorial-folders)

Before getting started, you will need to create the folders needed for this application:

```html
/flaskr
    /flaskr
        /static
        /templates
```

The application will be installed and run as Python package. This is the recommended way to install and run Flask applications. You will see exactly how to run `flaskr` later on in this tutorial. For now go ahead and create the applications directory structure. In the next few steps you will be creating the database schema as well as the main module.

As a quick side note, the files inside of the `static` folder are available to users of the application via HTTP. This is the place where CSS and JavaScript files go. Inside the `templates` folder, Flask will look for [Jinja2](http://jinja.pocoo.org/) templates. You will see examples of this later on.



<BR><BR>
## Step 1: [Database Schema](http://flask.pocoo.org/docs/0.12/tutorial/schema/#tutorial-schema)

In this step, you will create the database schema. Only a single table is needed for this application and it will only support SQLite. All you need to do is put the following contents into a file named `schema.sql` in the `flaskr/flaskr` folder:

```sql
drop table if exists entries;
create table entries (
  id integer primary key autoincrement,
  title text not null,
  'text' text not null
);
```

This schema consists of a single table called `entries`. Each row in this table has an `id`, a `title`, and a `text`. The `id` is an automatically incrementing integer and a primary key, the other two are strings that must not be null.



<BR><BR>
## Step 2: [Application Setup Code](http://flask.pocoo.org/docs/0.12/tutorial/setup/#tutorial-setup)

Now that the schema is in place, you can create the application module, `flaskr.py`. This file should be placed inside of the `flaskr/flaskr` folder. The first several lines of code in the application module are the needed import statements. After that there will be a few lines of configuration code. For small applications like `flaskr`, it is possible to drop the configuration directly into the module. However, a cleaner solution is to create a separate `.ini` or `.py` file, load that, and import the values from there.

Here are the import statements (in `flaskr.py`):

```python
# all the imports
import os
import sqlite3
from flask import Flask, request, session, g, redirect, url_for, abort, \
     render_template, flash
```

The next couple lines will create the actual application instance and initialize it with the config from the same file in `flaskr.py`:

```python
app = Flask(__name__) # create the application instance :)
app.config.from_object(__name__) # load config from this file , flaskr.py

# Load default config and override config from an environment variable
app.config.update(dict(
    DATABASE=os.path.join(app.root_path, 'flaskr.db'),
    SECRET_KEY='development key',
    USERNAME='admin',
    PASSWORD='default'
))
app.config.from_envvar('FLASKR_SETTINGS', silent=True)
```

The **Config** object works similarly to a dictionary, so it can be updated with new values.

##### Database Path  
Operating systems know the concept of a current working directory for each process. Unfortunately, you cannot depend on this in web applications because you might have more than one application in the same process.

For this reason the app.root_path attribute can be used to get the path to the application. Together with the os.path module, files can then easily be found. In this example, we place the database right next to it.

For a real-world application, it’s recommended to use [Instance Folders](http://flask.pocoo.org/docs/0.12/config/#instance-folders) instead.


Usually, it is a good idea to load a separate, environment-specific configuration file. Flask allows you to import multiple configurations and it will use the setting defined in the last import. This enables robust configuration setups. **from_envvar()** can help achieve this.

```python
app.config.from_envvar('FLASKR_SETTINGS', silent=True)
```

Simply define the environment variable **FLASKR_SETTINGS** that points to a config file to be loaded. The silent switch just tells Flask to not complain if no such environment key is set.

In addition to that, you can use the **from_object()** method on the config object and provide it with an import name of a module. Flask will then initialize the variable from that module. Note that in all cases, only variable names that are uppercase are considered.

The `SECRET_KEY` is needed to keep the client-side sessions secure. Choose that key wisely and as hard to guess and complex as possible.

Lastly, you will add a method that allows for easy connections to the specified database. This can be used to open a connection on request and also from the interactive Python shell or a script. This will come in handy later. You can create a simple database connection through SQLite and then tell it to use the **sqlite3.Row** object to represent rows. This allows the rows to be treated as if they were dictionaries instead of tuples.

```python
def connect_db():
    """Connects to the specific database."""
    rv = sqlite3.connect(app.config['DATABASE'])
    rv.row_factory = sqlite3.Row
    return rv
```

In the next section you will see how to run the application.

For issues with *environment variables* and *access keys*, see this page about [Flask settings](http://flask.pocoo.org/docs/0.12/tutorial/setup/#tutorial-setup).




<BR><BR>
## Step 3: [Installing flaskr as a Package](http://flask.pocoo.org/docs/0.12/tutorial/packaging/#tutorial-packaging)

Flask is now shipped with built-in support for [Click](http://click.pocoo.org/). Click provides Flask with enhanced and extensible command line utilities. Later in this tutorial you will see exactly how to extend the `flask` command line interface (CLI).

A useful pattern to manage a Flask application is to install your app following the [Python Packaging Guide](https://packaging.python.org/). Presently this involves creating two new files; `setup.py` and `MANIFEST.in` in the projects root directory. You also need to add an `__init__.py` file to make the `flaskr/flaskr` directory a package. After these changes, your code structure should be:

```html
/flaskr
    /flaskr
        __init__.py
        /static
        /templates
        flaskr.py
        schema.sql
    setup.py
    MANIFEST.in
```

The content of the `setup.py` file for `flaskr` is:

```python
from setuptools import setup

setup(
    name='flaskr',
    packages=['flaskr'],
    include_package_data=True,
    install_requires=['flask',],
)
```

When using setuptools, it is also necessary to specify any special files that should be included in your package (in the `MANIFEST.in`). In this case, the static and templates directories need to be included, as well as the schema. Create the `MANIFEST.in` and add the following lines:

```html
graft flaskr/templates
graft flaskr/static
include flaskr/schema.sql
```

To simplify locating the application, add the following import statement into this file, `flaskr/__init__.py`:

```python
from .flaskr import app
```

This import statement brings the application instance into the top-level of the application package. When it is time to run the application, the Flask development server needs the location of the app instance. This import statement simplifies the location process. Without it the export statement a few steps below would need to be `export FLASK_APP=flaskr.flaskr`.

At this point you should be able to install the application. As usual, **it is recommended to install your Flask application within a [vanilla ](https://virtualenv.pypa.io/) or [Anaconda](https://conda.io/docs/using/envs.html) virtualenv**. (I have installed *flaskr* in the `flaskr_app` virtualenv (Python 3.6), via `conda`, for example).  With that said, go ahead and install the application with (don't forget the period at the end of the following command):

```html
pip install --editable .
```

The above installation command **assumes that it is run within the projects root directory, _flaskr/_**. The *editable* flag allows editing source code without having to reinstall the Flask app each time you make changes. The flaskr app is now installed in your virtualenv (see output of `pip freeze`).

With that out of the way, you should be able to start up the application. When you `export` the **flaskr** app to the `FLASK_APP` name, you must be in the same directory as the `flaskr.py` file.  Once done, you can **run the application with the following commands**:

```html
export FLASK_APP=flaskr
export FLASK_DEBUG=true
flask run
```

##### JP Note:
If you don't import the app as a package, you can simply run `python flaskr.py` from its correct directory and it will **run the app!**  

You will lose control of the terminal window you run your flask app from. You end a `flask run [app]` session with the standard keyboard interrupt command, `ctrl-c`, in the terminal.  You can disconnect from the server itself with `server.terminate` in your script, but more on this later.
##### End JP Note

(In case you are on Windows you need to use *set* instead of *export*). The **FLASK_DEBUG** flag enables or disables the interactive debugger. Never leave debug mode activated in a production system, because it will allow users to execute code on the server!

You will see a message telling you that server has started along with the address at which you can access it.

When you head over to the server in your browser, you will get a 404 error because we don’t have any views yet. That will be addressed a little later, but first, you should get the database working.

*References:*  
Want your server to be publicly available? Check out the [externally visible server](http://flask.pocoo.org/docs/0.12/quickstart/#public-server) section for more information.





<BR><BR>
## Step 4: [Database Connections](http://flask.pocoo.org/docs/0.12/tutorial/dbcon/#tutorial-dbcon)

You currently have a function for establishing a database connection with connect_db, but by itself, it is not particularly useful. Creating and closing database connections all the time is very inefficient, so you will need to keep it around for longer. Because database connections encapsulate a transaction, you will need to make sure that only one request at a time uses the connection. An elegant way to do this is by utilizing the application context.

Flask provides two contexts: the application context and the request context. For the time being, all you have to know is that there are special variables that use these. For instance, the request variable is the request object associated with the current request, whereas g is a general purpose variable associated with the current application context. The tutorial will cover some more details of this later on.

For the time being, all you have to know is that you can store information safely on the g object.

So when do you put it on there? To do that you can make a helper function. The first time the function is called, it will create a database connection for the current context, and successive calls will return the already established connection:

```python
def get_db():
    """Opens a new database connection if there is none yet for the
    current application context.
    """
    if not hasattr(g, 'sqlite_db'):
        g.sqlite_db = connect_db()
    return g.sqlite_db
```

Now you know how to connect, but how can you properly disconnect? For that, Flask provides us with the teardown_appcontext() decorator. It’s executed every time the application context tears down:

```python
@app.teardown_appcontext
def close_db(error):
    """Closes the database again at the end of the request."""
    if hasattr(g, 'sqlite_db'):
        g.sqlite_db.close()
```

Functions marked with teardown_appcontext() are called every time the app context tears down. What does this mean? Essentially, the app context is created before the request comes in and is destroyed (torn down) whenever the request finishes. A teardown can happen because of two reasons: either everything went well (the error parameter will be None) or an exception happened, in which case the error is passed to the teardown function.

Curious about what these contexts mean? Have a look at the The Application Context documentation to learn more.

*References:*  
Where do I put this code? If you’ve been following along in this tutorial, you might be wondering where to put the code from [this step](http://flask.pocoo.org/docs/0.12/tutorial/dbcon/#tutorial-dbcon) and [the next](http://flask.pocoo.org/docs/0.12/tutorial/dbinit/#tutorial-dbinit). A logical place is to group these module-level functions together, and put your new get_db and close_db functions below your existing connect_db function (following the tutorial line-by-line).  If you need a moment to find your bearings, take a look at how the [example source](https://github.com/pallets/flask/tree/master/examples/flaskr/) is organized. In Flask, you can put all of your application code into a single Python module. You don’t have to, and if your app [grows larger](http://flask.pocoo.org/docs/0.12/patterns/packages/#larger-applications), it’s a good idea not to.

Read about Flask's [application context](http://flask.pocoo.org/docs/0.12/appcontext/#app-context) states (*requests* and *general*) to better understand how Flask is processing your connection and data operations.




<BR><BR>
## Step 5: [Creating the Database](http://flask.pocoo.org/docs/0.12/tutorial/dbinit/#tutorial-dbinit)

As outlined earlier, Flaskr is a database powered application, and more precisely, it is an application powered by a relational database system. Such systems need a schema that tells them how to store that information. Before starting the server for the first time, it’s important to create that schema.

Such a schema can be created by piping the schema.sql file into the sqlite3 command as follows (*note:* do not actually do this):

```html
sqlite3 /tmp/flaskr.db < schema.sql
```

The downside of this is that it requires the sqlite3 command to be installed, which is not necessarily the case on every system. This also requires that you provide the path to the database, which can introduce errors. It’s a good idea to add a function that initializes the database for you, to the application.

To do this, you can create a function and hook it into a flask command that initializes the database. For now just take a look at the code segment below. A good place to add this function, and command, is just below the connect_db function in flaskr.py:

```python
def init_db():
    db = get_db()
    with app.open_resource('schema.sql', mode='r') as f:
        db.cursor().executescript(f.read())
    db.commit()

@app.cli.command('initdb')
def initdb_command():
    """Initializes the database."""
    init_db()
    print('Initialized the database.')
```

The app.cli.command() decorator registers a new command with the flask script. When the command executes, Flask will automatically create an application context which is bound to the right application. Within the function, you can then access flask.g and other things as you might expect. When the script ends, the application context tears down and the database connection is released.

You will want to keep an actual function around that initializes the database, though, so that we can easily create databases in unit tests later on. (For more information see [Testing Flask Applications](http://flask.pocoo.org/docs/0.12/testing/#testing)).

The open_resource() method of the application object is a convenient helper function that will open a resource that the application provides. This function opens a file from the resource location (the flaskr/flaskr folder) and allows you to read from it. It is used in this example to execute a script on the database connection.

The connection object provided by SQLite can give you a cursor object. On that cursor, there is a method to execute a complete script. Finally, you only have to commit the changes. SQLite3 and other transactional databases will not commit unless you explicitly tell it to.

Now, it is possible to create a database with the flask script:
```html
flask initdb
Initialized the database.
```



<BR><BR>
## Step 6: [The View Functions](http://flask.pocoo.org/docs/0.12/tutorial/views/#tutorial-views)

Now that the database connections are working, you can start writing the view functions. You will need four of them:

#### Show Entries  
This view shows all the entries stored in the database. It listens on the root of the application and will select title and text from the database. The one with the highest id (the newest entry) will be on top. The rows returned from the cursor look a bit like dictionaries because we are using the sqlite3.Row row factory.

The view function will pass the entries to the show_entries.html template and return the rendered one:

```python
@app.route('/')
def show_entries():
    db = get_db()
    cur = db.execute('select title, text from entries order by id desc')
    entries = cur.fetchall()
    return render_template('show_entries.html', entries=entries)
```

#### Add New Entry  
This view lets the user add new entries if they are logged in. This only responds to POST requests; the actual form is shown on the show_entries page. If everything worked out well, it will flash() an information message to the next request and redirect back to the show_entries page:

```python
@app.route('/add', methods=['POST'])
def add_entry():
    if not session.get('logged_in'):
        abort(401)
    db = get_db()
    db.execute('insert into entries (title, text) values (?, ?)',
                 [request.form['title'], request.form['text']])
    db.commit()
    flash('New entry was successfully posted')
    return redirect(url_for('show_entries'))
```

Note that this view checks that the user is logged in (that is, if the logged_in key is present in the session and True).

##### Security Note
Be sure to use question marks when building SQL statements, as done in the example above. Otherwise, your app will be vulnerable to SQL injection when you use string formatting to build SQL statements. See [Using SQLite 3 with Flask](http://flask.pocoo.org/docs/0.12/patterns/sqlite3/#sqlite3) for more.

#### Login and Logout  
These functions are used to sign the user in and out. Login checks the username and password against the ones from the configuration and sets the logged_in key for the session. If the user logged in successfully, that key is set to True, and the user is redirected back to the show_entries page. In addition, a message is flashed that informs the user that he or she was logged in successfully. If an error occurred, the template is notified about that, and the user is asked again:

```python
@app.route('/login', methods=['GET', 'POST'])
def login():
    error = None
    if request.method == 'POST':
        if request.form['username'] != app.config['USERNAME']:
            error = 'Invalid username'
        elif request.form['password'] != app.config['PASSWORD']:
            error = 'Invalid password'
        else:
            session['logged_in'] = True
            flash('You were logged in')
            return redirect(url_for('show_entries'))
    return render_template('login.html', error=error)
```

The logout function, on the other hand, removes that key from the session again. There is a neat trick here: if you use the pop() method of the dict and pass a second parameter to it (the default), the method will delete the key from the dictionary if present or do nothing when that key is not in there. This is helpful because now it is not necessary to check if the user was logged in.

```python
@app.route('/logout')
def logout():
    session.pop('logged_in', None)
    flash('You were logged out')
    return redirect(url_for('show_entries'))
```

*References:*  
##### Security Note
Passwords should never be stored in plain text in a production system. This tutorial uses plain text passwords for simplicity. If you plan to release a project based off this tutorial out into the world, passwords should be both [hashed and salted](https://blog.codinghorror.com/youre-probably-storing-passwords-incorrectly/) before being stored in a database or file.

Fortunately, there are Flask extensions for the purpose of hashing passwords and verifying passwords against hashes, so adding this functionality is fairly straight forward. There are also many general python libraries that can be used for hashing.

You can find a list of recommended Flask extensions [here](http://flask.pocoo.org/extensions/)



<BR><BR>
## Step 7: [The Templates](http://flask.pocoo.org/docs/0.12/tutorial/templates/#tutorial-templates)

Now it is time to start working on the templates. As you may have noticed, if you make requests with the app running, you will get an exception that Flask cannot find the templates. The templates are using Jinja2 syntax and have autoescaping enabled by default. This means that unless you mark a value in the code with Markup or with the |safe filter in the template, Jinja2 will ensure that special characters such as < or > are escaped with their XML equivalents.

We are also using template inheritance which makes it possible to reuse the layout of the website in all pages.

Put the following templates into the templates folder:

#### layout.html  
This template contains the HTML skeleton, the header and a link to log in (or log out if the user was already logged in). It also displays the flashed messages if there are any. The {% block body %} block can be replaced by a block of the same name (body) in a child template.

The session dict is available in the template as well and you can use that to check if the user is logged in or not. Note that in Jinja you can access missing attributes and items of objects / dicts which makes the following code work, even if there is no 'logged_in' key in the session:

```html
<!doctype html>
<title>Flaskr</title>
<link rel=stylesheet type=text/css href="{{ url_for('static', filename='style.css') }}">
<div class=page>
  <h1>Flaskr</h1>
  <div class=metanav>
  {% if not session.logged_in %}
    <a href="{{ url_for('login') }}">log in</a>
  {% else %}
    <a href="{{ url_for('logout') }}">log out</a>
  {% endif %}
  </div>
  {% for message in get_flashed_messages() %}
    <div class=flash>{{ message }}</div>
  {% endfor %}
  {% block body %}{% endblock %}
</div>
```

#### show_entries.html  
This template extends the layout.html template from above to display the messages. Note that the for loop iterates over the messages we passed in with the render_template() function. Notice that the form is configured to to submit to the add_entry view function and use POST as HTTP method:

```html
{% extends "layout.html" %}
{% block body %}
  {% if session.logged_in %}
    <form action="{{ url_for('add_entry') }}" method=post class=add-entry>
      <dl>
        <dt>Title:
        <dd><input type=text size=30 name=title>
        <dt>Text:
        <dd><textarea name=text rows=5 cols=40></textarea>
        <dd><input type=submit value=Share>
      </dl>
    </form>
  {% endif %}
  <ul class=entries>
  {% for entry in entries %}
    <li><h2>{{ entry.title }}</h2>{{ entry.text|safe }}
  {% else %}
    <li><em>Unbelievable.  No entries here so far</em>
  {% endfor %}
  </ul>
{% endblock %}
```

#### login.html  
This is the login template, which basically just displays a form to allow the user to login:

```html
{% extends "layout.html" %}
{% block body %}
  <h2>Login</h2>
  {% if error %}<p class=error><strong>Error:</strong> {{ error }}{% endif %}
  <form action="{{ url_for('login') }}" method=post>
    <dl>
      <dt>Username:
      <dd><input type=text name=username>
      <dt>Password:
      <dd><input type=password name=password>
      <dd><input type=submit value=Login>
    </dl>
  </form>
{% endblock %}
```

<BR><BR>
## Step 8: [Adding Style](http://flask.pocoo.org/docs/0.12/tutorial/css/#tutorial-css)

Now that everything else works, it’s time to add some style to the application. Just create a stylesheet called style.css in the static folder:

```html
body            { font-family: sans-serif; background: #eee; }  
a, h1, h2       { color: #377ba8; }  
h1, h2          { font-family: 'Georgia', serif; margin: 0; }  
h1              { border-bottom: 2px solid #eee; }  
h2              { font-size: 1.2em; }  
.page           { margin: 2em auto; width: 35em; border: 5px solid #ccc;
                  padding: 0.8em; background: white; }  
.entries        { list-style: none; margin: 0; padding: 0; }  
.entries li     { margin: 0.8em 1.2em; }  
.entries li h2  { margin-left: -1em; }  
.add-entry      { font-size: 0.9em; border-bottom: 1px solid #ccc; }  
.add-entry dl   { font-weight: bold; }  
.metanav        { text-align: right; font-size: 0.8em; padding: 0.3em;
                  margin-bottom: 1em; background: #fafafa; }  
.flash          { background: #cee5F5; padding: 0.5em;
                  border: 1px solid #aacbe2; }  
.error          { background: #f0d6d6; padding: 0.5em; }  
```

<BR><BR>
## Step 9: [Testing the Application](http://flask.pocoo.org/docs/0.12/tutorial/testing/#tutorial-testing)

Now that you have finished the application and everything works as expected, it’s probably not a bad idea to add automated tests to simplify modifications in the future. The application above is used as a basic example of how to perform unit testing in the [Testing Flask Applications](http://flask.pocoo.org/docs/0.12/testing/#testing) section of the documentation. Go there to see how easy it is to test Flask applications.

#### Adding tests to flaskr  
Assuming you have seen the [Testing Flask Applications](http://flask.pocoo.org/docs/0.12/testing/#testing) section and have either written your own tests for flaskr or have followed along with the examples provided, you might be wondering about ways to organize the project.

One possible and recommended project structure is:

```html
flaskr/
    flaskr/
        __init__.py
        static/
        templates/
    tests/
        test_flaskr.py
    setup.py
    MANIFEST.in
```

For now go ahead a create the tests/ directory as well as the test_flaskr.py file.

#### Running the tests  
At this point you can run the tests. Here pytest will be used.

##### Note
Make sure that pytest is installed in the same virtualenv as flaskr. Otherwise pytest test will not be able to import the required components to test the application:

```html
pip install -e .
pip install pytest
```

Run and watch the tests pass, within the top-level flaskr/ directory as:

```html
py.test
```

#### Testing + setuptools  
One way to handle testing is to integrate it with setuptools. Here that requires adding a couple of lines to the setup.py file and creating a new file setup.cfg. One benefit of running the tests this way is that you do not have to install pytest. Go ahead and update the setup.py file to contain:

```python
from setuptools import setup

setup(
    name='flaskr',
    packages=['flaskr'],
    include_package_data=True,
    install_requires=[
        'flask',
    ],
    setup_requires=[
        'pytest-runner',
    ],
    tests_require=[
        'pytest',
    ],
)
```

Now create setup.cfg in the project root (alongside setup.py):

```html
[aliases]
test=pytest
```

Now you can run:

```html
python setup.py test
```

This calls on the alias created in setup.cfg which in turn runs pytest via pytest-runner, as the setup.py script has been called. (Recall the setup_requires argument in setup.py) Following the standard rules of test-discovery your tests will be found, run, and hopefully pass.

This is one possible way to run and manage testing. Here pytest is used, but there are other options such as nose. Integrating testing with setuptools is convenient because it is not necessary to actually download pytest or any other testing framework one might use.



<BR>
<BR>
<BR>

##### Reference Material  
+ Testing a Flask application is an important part of your development process. Read [this page](http://flask.pocoo.org/docs/0.12/testing/#testing) on how to do it.



<BR>
<BR>
<BR>
<BR>
<BR>
<BR>
<BR>

# [Testing Flask Applications](http://flask.pocoo.org/docs/0.12/testing/#testing)


Something that is untested is broken.
The origin of this quote is unknown and while it is not entirely correct, it is also not far from the truth. Untested applications make it hard to improve existing code and developers of untested applications tend to become pretty paranoid. If an application has automated tests, you can safely make changes and instantly know if anything breaks.

Flask provides a way to test your application by exposing the Werkzeug test Client and handling the context locals for you. You can then use that with your favourite testing solution. In this documentation we will use the unittest package that comes pre-installed with Python.

### The Application
First, we need an application to test; we will use the application from the Tutorial. If you don’t have that application yet, get the sources from the examples.

### The Testing Skeleton
In order to test the application, we add a second module (flaskr_tests.py) and create a unittest skeleton there:

```python
import os
import flaskr
import unittest
import tempfile

class FlaskrTestCase(unittest.TestCase):

    def setUp(self):
        self.db_fd, flaskr.app.config['DATABASE'] = tempfile.mkstemp()
        flaskr.app.config['TESTING'] = True
        self.app = flaskr.app.test_client()
        with flaskr.app.app_context():
            flaskr.init_db()

    def tearDown(self):
        os.close(self.db_fd)
        os.unlink(flaskr.app.config['DATABASE'])


if __name__ == '__main__':
    unittest.main()
```    

The code in the setUp() method creates a new test client and initializes a new database. This function is called before each individual test function is run. To delete the database after the test, we close the file and remove it from the filesystem in the tearDown() method. Additionally during setup the TESTING config flag is activated. What it does is disable the error catching during request handling so that you get better error reports when performing test requests against the application.

This test client will give us a simple interface to the application. We can trigger test requests to the application, and the client will also keep track of cookies for us.

Because SQLite3 is filesystem-based we can easily use the tempfile module to create a temporary database and initialize it. The mkstemp() function does two things for us: it returns a low-level file handle and a random file name, the latter we use as database name. We just have to keep the db_fd around so that we can use the os.close() function to close the file.

If we now run the test suite, we should see the following output:

```html
$ python flaskr_tests.py

----------------------------------------------------------------------
Ran 0 tests in 0.000s

OK
```

Even though it did not run any actual tests, we already know that our flaskr application is syntactically valid, otherwise the import would have died with an exception.

####The First Test
Now it’s time to start testing the functionality of the application. Let’s check that the application shows “No entries here so far” if we access the root of the application (/). To do this, we add a new test method to our class, like this:

```python
class FlaskrTestCase(unittest.TestCase):

    def setUp(self):
        self.db_fd, flaskr.app.config['DATABASE'] = tempfile.mkstemp()
        self.app = flaskr.app.test_client()
        flaskr.init_db()

    def tearDown(self):
        os.close(self.db_fd)
        os.unlink(flaskr.app.config['DATABASE'])

    def test_empty_db(self):
        rv = self.app.get('/')
        assert b'No entries here so far' in rv.data
```

Notice that our test functions begin with the word test; this allows unittest to automatically identify the method as a test to run.

By using self.app.get we can send an HTTP GET request to the application with the given path. The return value will be a response_class object. We can now use the data attribute to inspect the return value (as string) from the application. In this case, we ensure that 'No entries here so far' is part of the output.

Run it again and you should see one passing test:

```html
$ python flaskr_tests.py
.
----------------------------------------------------------------------
Ran 1 test in 0.034s

OK
```

### Logging In and Out
The majority of the functionality of our application is only available for the administrative user, so we need a way to log our test client in and out of the application. To do this, we fire some requests to the login and logout pages with the required form data (username and password). And because the login and logout pages redirect, we tell the client to follow_redirects.

Add the following two methods to your FlaskrTestCase class:

```python
def login(self, username, password):
    return self.app.post('/login', data=dict(
        username=username,
        password=password
    ), follow_redirects=True)

def logout(self):
    return self.app.get('/logout', follow_redirects=True)
```

Now we can easily test that logging in and out works and that it fails with invalid credentials. Add this new test to the class:

```python
def test_login_logout(self):
    rv = self.login('admin', 'default')
    assert b'You were logged in' in rv.data
    rv = self.logout()
    assert b'You were logged out' in rv.data
    rv = self.login('adminx', 'default')
    assert b'Invalid username' in rv.data
    rv = self.login('admin', 'defaultx')
    assert b'Invalid password' in rv.data
```

### Test Adding Messages
We should also test that adding messages works. Add a new test method like this:

```python
def test_messages(self):
    self.login('admin', 'default')
    rv = self.app.post('/add', data=dict(
        title='<Hello>',
        text='<strong>HTML</strong> allowed here'
    ), follow_redirects=True)
    assert b'No entries here so far' not in rv.data
    assert b'&lt;Hello&gt;' in rv.data
    assert b'<strong>HTML</strong> allowed here' in rv.data
```

Here we check that HTML is allowed in the text but not in the title, which is the intended behavior.

Running that should now give us three passing tests:

```html
$ python flaskr_tests.py
...
----------------------------------------------------------------------
Ran 3 tests in 0.332s

OK
```

For more complex tests with headers and status codes, check out the [MiniTwit Example](https://github.com/pallets/flask/tree/master/examples/minitwit/) from the sources which contains a larger test suite.

### Other Testing Tricks
Besides using the test client as shown above, there is also the test_request_context() method that can be used in combination with the with statement to activate a request context temporarily. With this you can access the request, g and session objects like in view functions. Here is a full example that demonstrates this approach:

```python
import flask

app = flask.Flask(__name__)

with app.test_request_context('/?name=Peter'):
    assert flask.request.path == '/'
    assert flask.request.args['name'] == 'Peter'
```

All the other objects that are context bound can be used in the same way.

If you want to test your application with different configurations and there does not seem to be a good way to do that, consider switching to application factories (see Application Factories).

Note however that if you are using a test request context, the `before_request()` and `after_request()` functions are not called automatically. However `teardown_request()` functions are indeed executed when the test request context leaves the with block. If you do want the before_request() functions to be called as well, you need to call `preprocess_request()` yourself:

```python
app = flask.Flask(__name__)

with app.test_request_context('/?name=Peter'):
    app.preprocess_request()
    ...
```

This can be necessary to open database connections or something similar depending on how your application was designed.

If you want to call the after_request() functions you need to call into process_response() which however requires that you pass it a response object:

```python
app = flask.Flask(__name__)

with app.test_request_context('/?name=Peter'):
    resp = Response('...')
    resp = app.process_response(resp)
    ...
```

This in general is less useful because at that point you can directly start using the test client.

### Faking Resources and Context
(New in version 0.10.)

A very common pattern is to store user authorization information and database connections on the application context or the flask.g object. The general pattern for this is to put the object on there on first usage and then to remove it on a teardown. Imagine for instance this code to get the current user:

```python
def get_user():
    user = getattr(g, 'user', None)
    if user is None:
        user = fetch_current_user_from_database()
        g.user = user
    return user
```

For a test it would be nice to override this user from the outside without having to change some code. This can be accomplished with hooking the `flask.appcontext_pushed` signal:

```python
from contextlib import contextmanager
from flask import appcontext_pushed, g

@contextmanager
def user_set(app, user):
    def handler(sender, **kwargs):
        g.user = user
    with appcontext_pushed.connected_to(handler, app):
        yield
```

And then to use it:

```python
from flask import json, jsonify

@app.route('/users/me')
def users_me():
    return jsonify(username=g.user.username)

with user_set(app, my_user):
    with app.test_client() as c:
        resp = c.get('/users/me')
        data = json.loads(resp.data)
        self.assert_equal(data['username'], my_user.username)
```

### Keeping the Context Around
(New in version 0.4.)

Sometimes it is helpful to trigger a regular request but still keep the context around for a little longer so that additional introspection can happen. With Flask 0.4 this is possible by using the test_client() with a with block:

```python
app = flask.Flask(__name__)

with app.test_client() as c:
    rv = c.get('/?tequila=42')
    assert request.args['tequila'] == '42'
```

If you were to use just the test_client() without the with block, the assert would fail with an error because request is no longer available (because you are trying to use it outside of the actual request).

### Accessing and Modifying Sessions
(New in version 0.8.)

Sometimes it can be very helpful to access or modify the sessions from the test client. Generally there are two ways for this. If you just want to ensure that a session has certain keys set to certain values you can just keep the context around and access flask.session:

```python
with app.test_client() as c:
    rv = c.get('/')
    assert flask.session['foo'] == 42
```

This however does not make it possible to also modify the session or to access the session before a request was fired. Starting with Flask 0.8 we provide a so called “session transaction” which simulates the appropriate calls to open a session in the context of the test client and to modify it. At the end of the transaction the session is stored. This works independently of the session backend used:

```python
with app.test_client() as c:
    with c.session_transaction() as sess:
        sess['a_key'] = 'a value'

    # once this is reached the session was stored
```

Note that in this case you have to use the sess object instead of the flask.session proxy. The object however itself will provide the same interface.
