# JP Heroku Notes

## Heroku with Dash from Plotly
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





## Other Heroku Apps

### Flask
https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-xviii-deployment-on-the-heroku-cloud

From above: web: gunicorn app:app


### Dash on Flask
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
