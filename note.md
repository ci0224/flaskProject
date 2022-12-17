# Flask

## Installation

Recommend to use a virtual environment

``` shell
# in terminal 
pip install Flask
```

## Quick Start


```python
# app.py
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello_world():
    return "<p>Hello, World!</p>"
```
```shell
# in terminal
flask --app minimal run
# the minimal is consist with the file name
# if the file name is app.py or wsgi.py
#   this command can be shortened as 
flask run
```
```shell
# debug mode
flask --app minimal --debug run
```
```shell
# external visible server

flask run --host=0.0.0.0
```
### HTML Escaping
When returning HTML (the default response type in Flask), any user-provided values rendered in the output must be escaped to protect from injection attacks. HTML templates rendered with Jinja, introduced later, will do this automatically.

escape(), shown here, can be used manually. It is omitted in most examples for brevity, but you should always be aware of how you’re using **untrusted data**.
```python

from markupsafe import escape

@app.route("/<name>")
def hello(name):
    return f"Hello, {escape(name)}!"
```
### Routing 
```python
@app.route('/')
def index():
    return 'Index Page'

@app.route('/hello')
def hello():
    return 'Hello, World'
```

### Variable rules 

```python
from markupsafe import escape

@app.route('/user/<username>')
def show_user_profile(username):
    # show the user profile for that user
    return f'User {escape(username)}'

@app.route('/post/<int:post_id>')
def show_post(post_id):
    # show the post with the given id, the id is an integer
    return f'Post {post_id}'

@app.route('/path/<path:subpath>')
def show_subpath(subpath):
    # show the subpath after /path/
    return f'Subpath {escape(subpath)}'
```

| rule | meaning  |
| ------- | --- | 
| string | (default) accepts any text without a slash | 
| int | accepts positive integers | 
| float | accepts positive floating point values | 
| path | string text with slashes | 
| uuid | accepts UUID strings | 

### Unique URLs / Redirection Behavior

```python
@app.route('/projects/')
def projects():
    return 'The project page'

@app.route('/about')
def about():
    return 'The about page'
```
The canonical URL for the projects endpoint has a trailing slash. It’s similar to a folder in a file system. If you access the URL without a trailing slash (/projects), Flask redirects you to the canonical URL with the trailing slash (/projects/).

The canonical URL for the about endpoint does not have a trailing slash. It’s similar to the pathname of a file. Accessing the URL with a trailing slash (/about/) produces a 404 “Not Found” error.

This helps keep URLs unique for these resources, which helps search engines avoid indexing the same page twice.

### URL Building

```python
from flask import url_for

@app.route('/')
def index():
    return 'index'

@app.route('/login')
def login():
    return 'login'

@app.route('/user/<username>')
def profile(username):
    # the next link has one backslash only.
    # Markdown code chunk might display 2 ''
    return f'{username}\'s profile' 

with app.test_request_context():
    print(url_for('index'))
    print(url_for('login'))
    print(url_for('login', next='/'))
    print(url_for('profile', username='John Doe'))
```
```
# output
/
/login
/login?next=/
/user/John%20Doe
```

### HTTP Methods

Web applications use different HTTP methods when accessing URLs. You should familiarize yourself with the HTTP methods as you work with Flask. 

By default, a route only answers to GET requests. You can use the methods argument of the route() decorator to handle different HTTP methods.
```python
from flask import request

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        return do_the_login()
    else:
        return show_the_login_form()
```

### Rendering Templates

```python
from flask import render_template

@app.route('/hello/')
@app.route('/hello/<name>')
def hello(name=None):
    return render_template('hello.html', name=name)
```
Flask will look for templates in the templates folder. So if your application is a module, this folder is next to that module, if it’s a package it’s actually inside your package:
```html
Case 1: a module:

/application.py
/templates
    /hello.html
Case 2: a package:

/application
    /__init__.py
    /templates
        /hello.html
```
For templates, you can use the full power of Jinja2 templates. Head over to the official [Jinja2 Template Documentation](https://jinja.palletsprojects.com/templates/) for more information.

Here is an example template:
```html
<!doctype html>
<title>Hello from Flask</title>
{% if name %}
  <h1>Hello {{ name }}!</h1>
{% else %}
  <h1>Hello, World!</h1>
{% endif %}
```

### Context Locals (for unit test)

You notice that code which depends on a request object will suddenly break because there is no request object. The solution is creating a request object yourself and binding it to the context. The easiest solution for unit testing is to use the *test_request_context()* context manager. In combination with the with statement it will bind a test request so that you can interact with it. Here is an example:

```python
from flask import request

with app.test_request_context('/hello', method='POST'):
    # now you can do something with the request until the
    # end of the with block, such as basic assertions:
    assert request.path == '/hello'
    assert request.method == 'POST'
```
The other possibility is passing a whole WSGI environment to the request_context() method:
```python
with app.request_context(environ):
    assert request.method == 'POST'
```

[stopped at here](https://flask.palletsprojects.com/en/2.2.x/quickstart/#the-request-object)