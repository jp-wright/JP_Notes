# JP Dash (Plotly) Notes

## Deploying on Heroku
Copied from [Dash's site](https://dash.plot.ly/deployment).

Heroku is one of the easiest platforms for deploying and managing public Flask applications.
[View the official Heroku guide to Python](https://devcenter.heroku.com/articles/getting-started-with-python#introduction).
Here is a simple example. This example requires a Heroku account, `git`, and `virtualenv`.

##### Step 1. Create a new folder for your project:

```bash
$ mkdir dash_app_example
$ cd dash_app_example
```

##### Step 2. Initialize the folder with git and a virtualenv

```bash
$ git init        # initializes an empty git repo
$ virtualenv venv # creates a virtualenv called "venv"
$ source venv/bin/activate # uses the virtualenv
```

`virtualenv` creates a fresh Python instance. You will need to reinstall your app's dependencies with this virtualenv:

```bash
$ pip install dash
$ pip install plotly
```

You will also need a new dependency, gunicorn, for deploying the app:

```bash
$ pip install gunicorn
```

##### Step 3. Initialize the folder with a sample app (`app.py`), a `.gitignore` file, `requirements.txt`, and a `Procfile` for deployment

Create the following files in your project folder:

__app.py__

```python
import os

import dash
import dash_core_components as dcc
import dash_html_components as html

external_stylesheets = ['https://codepen.io/chriddyp/pen/bWLwgP.css']

app = dash.Dash(__name__, external_stylesheets=external_stylesheets)

server = app.server

app.layout = html.Div([
    html.H2('Hello World'),
    dcc.Dropdown(
        id='dropdown',
        options=[{'label': i, 'value': i} for i in ['LA', 'NYC', 'MTL']],
        value='LA'
    ),
    html.Div(id='display-value')
])

@app.callback(dash.dependencies.Output('display-value', 'children'),
              [dash.dependencies.Input('dropdown', 'value')])
def display_value(value):
    return 'You have selected "{}"'.format(value)

if __name__ == '__main__':
    app.run_server(debug=True)
```


__.gitignore__
```bash
venv
*.pyc
.DS_Store
.env
```

__Procfile__
```bash
web: gunicorn app:server
```
(Note that `app` refers to the filename `app.py`. `server` refers to the variable `server` inside that file).

__requirements.txt__  
`requirements.txt` describes your Python dependencies. You can fill this file in automatically with:

```bash
$ pip freeze > requirements.txt
```

##### Step 4. Initialize Heroku, add files to Git, and deploy

```bash
$ heroku create my-dash-app # change my-dash-app to a unique name
$ git add . # add all files to git
$ git commit -m 'Initial app boilerplate'
$ git push heroku master # deploy code to heroku
$ heroku ps:scale web=1  # run the app with a 1 heroku "dyno"
```

Activate your app with by running it in the command line `python app.py`.  Debug notes will stream to the terminal window with `debug=True` set.

You should be able to view your app at https://my-dash-app.herokuapp.com (changing `my-dash-app` to the name of your app).

##### Step 5. Update the code and redeploy
When you modify `app.py` with your own code, you will need to add the changes to git and push those changes to heroku.

```bash
$ git status # view the changes
$ git add .  # add all the changes
$ git commit -m 'a description of the changes'
$ git push heroku master
```
This workflow for deploying apps on heroku is very similar to how deployment works with the Plotly Enterprise's Dash Deployment Server. [Learn more](https://plot.ly/dash/pricing/).



## Misc Heroku notes
From https://community.plot.ly/t/dash-app-deployment-on-heroku-successful-but-app-not-loading/7918/2

Change above to: web: gunicorn index:server --> No, this is wrong. JP, 11/2019

Also seen: web: gunicorn app
Or web: gunicorn app:server --> this one is right, jp.

These are Included in an extensionless Procfile




My output...:
requirements.txt:
dash==0.30.0
dash-core-components==0.38.0
dash-html-components==0.13.2
dash-renderer==0.15.1
dash-table==3.1.6
plotly==3.4.1
pandas==0.23.3
gunicorn==19.9.0






## Dash on Flask
### Flask Mega Tutorial
https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-xviii-deployment-on-the-heroku-cloud

From above: web: gunicorn app:app

### Deploying Dash on Flask
https://dash.plot.ly/deployment
Dash apps are web applications. Dash uses Flask as the web framework. The underlying Flask app is available at app.server, that is:
import dash

app = dash.Dash(__name__)

server = app.server # the Flask app
You can also pass your own flask app instance into Dash:
import flask

server = flask.Flask(__name__)
app = dash.Dash(__name__, server=server)
By exposing this server variable, you can deploy Dash apps like you would any Flask app. For more, see the official Flask Guide to Deployment. Note that
While lightweight and easy to use, Flask's built-in server is not suitable for production as it doesn't scale well and by default serves only one request at a time














## Enhancing Performance
From [this](https://dash.plot.ly/performance) Dash page

The main performance limitation of dash apps is likely the callbacks in the application code itself. If you can speed up your callbacks, your app will feel snappier.

##### Memoization

Since Dash's callbacks are functional in nature (they don't contain any state), it's easy to add memoization caching. Memoization stores the results of a function after it is called and re-uses the result if the function is called with the same arguments.
To better understand how memoization works, let's start with a simple example.

```python
import time
import functools32

@functools32.lru_cache(maxsize=32)
def slow_function(input):
    time.sleep(10)
    return 'Input was {}'.format(input)
```

Calling `slow_function('test')` the first time will take 10 seconds. Calling it a second time with the same argument will take almost no time since the previously computed result was saved in memory and reused.

Dash apps are frequently deployed across multiple processes or threads. In these cases, each process or thread contains its own memory, it doesn't share memory across instances. This means that if we were to use `lru_cache`, our cached results might not be shared across sessions.

Instead, we can use the Flask-Caching library which saves the results in a shared memory database like Redis or as a file on your filesystem. Flask-Caching also has other nice features like time-based expiry. Time-based expiry is helpful if you want to update your data (clear your cache) every hour or every day.

Here is an example of Flask-Caching with Redis:

```python
import datetime
import os

import dash
import dash_core_components as dcc
import dash_html_components as html
from dash.dependencies import Input, Output
from flask_caching import Cache

external_stylesheets = ['https://codepen.io/chriddyp/pen/bWLwgP.css']

app = dash.Dash(__name__, external_stylesheets=external_stylesheets)
cache = Cache(app.server, config={
    # try 'filesystem' if you don't want to setup redis
    'CACHE_TYPE': 'redis',
    'CACHE_REDIS_URL': os.environ.get('REDIS_URL', '')
})
app.config.suppress_callback_exceptions = True

timeout = 20
app.layout = html.Div([
    html.Div(id='flask-cache-memoized-children'),
    dcc.RadioItems(
        id='flask-cache-memoized-dropdown',
        options=[
            {'label': 'Option {}'.format(i), 'value': 'Option {}'.format(i)}
            for i in range(1, 4)
        ],
        value='Option 1'
    ),
    html.Div('Results are cached for {} seconds'.format(timeout))
])


@app.callback(
    Output('flask-cache-memoized-children', 'children'),
    [Input('flask-cache-memoized-dropdown', 'value')])
@cache.memoize(timeout=timeout)  # in seconds
def render(value):
    return 'Selected "{}" at "{}"'.format(
        value, datetime.datetime.now().strftime('%H:%M:%S')
    )


if __name__ == '__main__':
    app.run_server(debug=True)
```

Here is an example that caches a dataset instead of a callback. It uses the `FileSystem` cache, saving the cached results to the filesystem.

This approach works well if there is one dataset that is used to update several callbacks.

```python
import datetime as dt
import os
import time

import dash
import dash_core_components as dcc
import dash_html_components as html
import numpy as np
import pandas as pd
from dash.dependencies import Input, Output
from flask_caching import Cache

external_stylesheets = ['https://codepen.io/chriddyp/pen/bWLwgP.css']

app = dash.Dash(__name__, external_stylesheets=external_stylesheets)
cache = Cache(app.server, config={
    'CACHE_TYPE': 'filesystem',
    'CACHE_DIR': 'cache-directory'
})

TIMEOUT = 60

@cache.memoize(timeout=TIMEOUT)
def query_data():
    # This could be an expensive data querying step
    df =  pd.DataFrame(
        np.random.randint(0,100,size=(100, 4)),
        columns=list('ABCD')
    )
    now = dt.datetime.now()
    df['time'] = [now - dt.timedelta(seconds=5*i) for i in range(100)]
    return df.to_json(date_format='iso', orient='split')


def dataframe():
    return pd.read_json(query_data(), orient='split')

app.layout = html.Div([
    html.Div('Data was updated within the last {} seconds'.format(TIMEOUT)),
    dcc.Dropdown(
        id='live-dropdown',
        value='A',
        options=[{'label': i, 'value': i} for i in dataframe().columns]
    ),
    dcc.Graph(id='live-graph')
])


@app.callback(Output('live-graph', 'figure'),
              [Input('live-dropdown', 'value')])
def update_live_graph(value):
    df = dataframe()
    now = dt.datetime.now()
    return {
        'data': [{
            'x': df['time'],
            'y': df[value],
            'line': {
                'width': 1,
                'color': '#0074D9',
                'shape': 'spline'
            }
        }],
        'layout': {
            # display the current position of now
            # this line will be between 0 and 60 seconds
            # away from the last datapoint
            'shapes': [{
                'type': 'line',
                'xref': 'x', 'x0': now, 'x1': now,
                'yref': 'paper', 'y0': 0, 'y1': 1,
                'line': {'color': 'darkgrey', 'width': 1}
            }],
            'annotations': [{
                'showarrow': False,
                'xref': 'x', 'x': now, 'xanchor': 'right',
                'yref': 'paper', 'y': 0.95, 'yanchor': 'top',
                'text': 'Current time ({}:{}:{})'.format(
                    now.hour, now.minute, now.second),
                'bgcolor': 'rgba(255, 255, 255, 0.8)'
            }],
            # aesthetic options
            'margin': {'l': 40, 'b': 40, 'r': 20, 't': 10},
            'xaxis': {'showgrid': False, 'zeroline': False},
            'yaxis': {'showgrid': False, 'zeroline': False}
        }
    }


if __name__ == '__main__':
    app.run_server(debug=True)
```
