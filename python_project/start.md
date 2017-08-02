Initial Skeleton
================

We're going to start with the same steps as the [Google provided Quickstart](https://cloud.google.com/appengine/docs/standard/python/quickstart). So go ahead and walk through that guide for the hello world as the first step. While it's not a requirement for you to have a deep understanding of GAE we do recommend understanding some of the basics. So we recommend stepping through the rest of the guide to develop a flask app. This will help you understand the basic structure and capabilities of GAE. We'll also be creating a Flask project so much of the material will translate.

For the first step of this guide we're going to start from the [hello world example provided by Google](https://github.com/GoogleCloudPlatform/python-docs-samples/tree/master/appengine/standard/hello_world). You can either clone the full examples repo and go to that directory or since they are only 3 files you could just copy them down. Once done you should end up with a single directory that holds 3 files: `app.yaml`, `main.py` and `main_test.py`.

Once you have those files in your directory you can test your app by running:

```
$ dev_appserver.py app.yaml
```

Or the shorthand version of:

```
$ dev_appserver.py .
```

Which will look for the app.yaml to run off of.

### Virtual Environments

Now we are proponents of Python Virtual Environments. If you're not familiar with [Python virtualenv](http://pypi.python.org/pypi/virtualenv) you can learn more in the excellent ["The Hitchhiker's Guide to Python"](http://python-guide-pt-br.readthedocs.io/en/latest/) and it's [section on Virtual Environments](http://python-guide-pt-br.readthedocs.io/en/latest/dev/virtualenvs/). Personally I'm also a fan of [virtualenvwrapper](https://virtualenvwrapper.readthedocs.io/en/latest/index.html) which is covered at the [bottom of the same guide](http://python-guide-pt-br.readthedocs.io/en/latest/dev/virtualenvs/#virtualenvwrapper). Virtualenv wrapper provides some nice UX improvements over vitrualenv.

Here's how I use virtualenv and virtualenvwrapper for my Python projects.

I first `cd` to the directory of the project I'm working on.

```
$ cd ~/projects/gae_starter
```

Once in the directory I then create the virtual environment.

```
$ mkvirtualenv -a $PWD gaestarter
```

The `-a $PWD` tells virtualenvwrapper that the current working directory is the directory to attach to this virtual environment. This will automatically switch you to that directory when activating your virtual environment. The `gaestarter` is the name of the virtual environment for the project. You can use any name you'd like but obviously a name specific to the project is ideal. The value of having virtual environments is supporting more than one project environment and keeping your dependencies isolated. So these environments should be scoped to the specific project.

By default when you create a virtual env with virtualenvwrapper it will activate the virtualenv. However to see who to interact with the virtualenv in the future you can follow these steps.

First you can deactivate the current virtualenv by ensuring your in an active virtualenv shell in your terminal by issuing the following command:

```
$ deactivate
```

To activate your virtualenv (if using virtualenvwrapper) you can be in any shell in any directory and issue the following command:

```
$ workon gaestarter
```

This will take you to the project directory and activate the virtualenv.

### Flask REST API Server

The first thing we're going to build out is a [REST](https://en.wikipedia.org/wiki/Representational_state_transfer) server to expose http endpoints via [Flask](http://flask.pocoo.org/).

  Flask is a microframework for Python based on Werkzeug, Jinja 2 and good intentions. And before you ask: It's BSD licensed!

Flask is our favorite Python web framework as it's relatively simple and provides the flexibility to be configured for many different use cases. As mentioned our first use case is as a REST API. We're going to send and receive JSON encoded data over http.

Let's update our `main.py` to be a Flask based request instead of webapp2. Remove all the existing code in `main.py` and replace it with the following:

```
from flask import Flask

app = Flask(__name__)

@app.route('/')
def home():
    return "Hello Flask World!"
```

Now if you run the app you'll get a 500 error in the page and if you look at your console you'll see an `ImportError` like the following:

```
Traceback (most recent call last):
  File "/Users/username/programs/google-cloud-sdk/platform/google_appengine/google/appengine/runtime/wsgi.py", line 240, in Handle
    handler = _config_handle.add_wsgi_middleware(self._LoadHandler())
  File "/Users/username/programs/google-cloud-sdk/platform/google_appengine/google/appengine/runtime/wsgi.py", line 299, in _LoadHandler
    handler, path, err = LoadObject(self._handler)
  File "/Users/username/programs/google-cloud-sdk/platform/google_appengine/google/appengine/runtime/wsgi.py", line 85, in LoadObject
    obj = __import__(path[0])
  File "/Users/username/projects/open/gae_guides/gae_starter/main.py", line 15, in <module>
    from flask import Flask
```

We don't have the Flask library installed in our project. So let's do that next.

Adding Flask to our project takes us to one of the under documented yet important part of your application. Dependency and package management. It's one thing to add a library to our project but how do we ensure it gets deployed correctly while running correctly locally both with the development server and via our unit tests.

[Onto Dependency Management and Libraries](dependencies.md)
