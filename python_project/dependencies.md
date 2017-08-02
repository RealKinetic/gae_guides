Dependency Management, 3rd Party Libraries, etc
===============================================

We like to use as many standard Python practices as possible. This is why we're using tools such as `pip` and `virtualenv` however there's some tricks we need to do to ensure they work correctly with GAE.

By default when installing libraries with pip they will be installed into your `site-packages` directory. If you're not using `virutalenv` this will be the global site-packages directory that sits within your Python directory.

Deactive your virtualenv and you can see where your system Python site-packages directory is by running this command:

```
$ python -c "from distutils.sysconfig import get_python_lib; print(get_python_lib())"
```

You'll see a result similar to this:

```
/usr/local/Cellar/python/2.7.13/Frameworks/Python.framework/Versions/2.7/lib/python2.7/site-packages
```

If you activate your virtualenv (venv) and do the same you'll see somthing like:

```
/Users/username/.virtualenvs/gaestarter/lib/python2.7/site-packages
```

This is how virtual environments keep your libraries separate. A note virtual env creates a global variable for your virtual environment. In an active virtual environment run:

```
$ echo $VIRTUAL_ENV
```

The problem with storing our libraries in site-packages is when deploying our GAE project it only pulls files from our directory up. So it will not bundle our libraries in site-packages and deploy them. You will end up with import errors when trying to run the deployed application.

To mitigate this problem pip gives us a way to override where packages are install when running the pip command. In this case we're going to create a `vendor` directory to store the libraries we want deployed with our application.

```
mkdir vendor
```

We could then install a single library like so:

```
$ pip install flask -t vendor
```

If you run that command you'll see many directories now exist in our vendor directory. This includes Flask all of the libraries that Flask is dependent upon. Unfortunately if we want to uninstall Flask we can't just use `pip uninstall flask` as it looks only in site-directories and as of the moment the `-t` flag is not supported by uninstall. However for now we can just clear all of the libraries out of our vendor directory.

```
$ rm -rf vendor/*
```

Now this would be annoying if we had to do this for all packages. But thankfully pip comes with support for using a file that lists all our libraries which we generally call `requirements.txt`. So create a file named `requirements.txt` and add the following line

```
Flask>=0.12,<0.13
```

By adding the `>=0.12,<0.13` to the entry we're ensuring that when install Flask it will always use the most recent version of Flask above version `0.12` but below version `0.13`. This is generally called pinning a version. This is often critical in managing our dependencies to avoid pulling in broken or non-tested changes from upstream providers. Or often you'll hit collisions with multiple libraries not supporting the same versions of dependencies.

To install the libraries from our requirements.txt file you run the following command:

```
$ pip install -Ur requirements.txt -t vendor
```

The `-r` says we're passing in a file of libraries instead of a library name. The `-U` says to check for updates to any of the libraries and install those if there's a newer version than we currently have installed.

If you run that command now you'll see the same install steps we witnessed when installing Flask directly.

* FYI for those using git you will want to add `vendor` to your `.gitignore` file as there's no need to commit these files into your repo.

### Library Configuration

Now that we have our vendor library we need to let GAE know about it. The best way to do this is to [add a reference to the appengine configuration](https://cloud.google.com/appengine/docs/standard/python/getting-started/python-standard-env). 

Create a file name `appengine_config.py` in the root project directory. once created add the following code:

```
from google.appengine.ext import vendor

# Add the libraries installed in our vendor folder.
vendor.add('vendor')
```

Now if we run our application again and open the page http://localhost:8080 you should see:

```
Hello Flask World!
```

### Built-in Libraries

Now we've seen how we can add our own libraries to be deployed but there's another option as well. Google provides a list of 3rd party libraries that you can use without needing to deploy them.  [Here](https://cloud.google.com/appengine/docs/standard/python/tools/built-in-libraries-27) is the list of libraries (and they're versions) that Google includes with GAE. 

Let's walkthrough a quick example of how to use one of these libraries. In this case we'll use `yaml`. We don't currently have a need for this but it's good to understand this process.  To tell GAE that you want to use `yaml` we need to [update our `app.yaml` file](https://cloud.google.com/appengine/docs/standard/python/tools/using-libraries-python-27). To add `yaml` we'll add a `libraries` entry to the end of our `app.yaml` file like so:

```
runtime: python27
api_version: 1
threadsafe: true

handlers:
- url: /.*
  script: main.app

libraries:
- name: yaml
  version: "3.10"
```

* Note: If you want the latest version of the library you're using you can replace the version with `latest`. Although we prefer to pin as we do in our requirements file.

Now if we can just use the `yaml` lib in our app and we don't have to anything else. However there is a gotcha that we need to be aware of. If you read through Google's docs on ["Requesting a library"](https://cloud.google.com/appengine/docs/standard/python/tools/using-libraries-python-27#requesting_a_library) you'll see this bullet point `Some libraries must be installed locally.` So how do we know what libraries are included or not. This is where things get fun. If you remember where you installed your Google Cloud SDK you can head there now but if not we'll use the trusty `dev_appserver.py` file to help us out. Run this command:

```
$ which dev_appserver.py
```

Which should give you something like:

```
/Users/username/programs/google-cloud-sdk/bin/dev_appserver.py
```

Now we know where our sdk is: `/Users/username/programs/google-cloud-sdk`

Within our sdk we have a `platform` directory which should have a `google_appengine` directory within it. That's where our GAE SDK lives. Within that directory we see a directory named `lib`. If we `ls` that directory we'll see that we've found the folder storing the libraries that Google gives us for local development.

```
ls ~/programs/google-cloud-sdk/platform/google_appengine/lib
```

So what do we do? If we put it back in our vendor directory then it will be deployed. So we once again hit the issue of unnecessary code in our runtime. Beyond less files there are other potential advantages to using Google's provided libraries. Many Python libraries included `clang` modules for performance. However GAE does not allow us to push up native code such as c modules as they are trying to keep a protected sandbox. This makes complete sense from their standpoint and why something like GAE Flex is appealing for those that do want their own native code. We'll cover that later. For now this means there's a potential performance benefit to using Google provided libraries.

This means for local development we'll want to have our libraries accessible but not allow them to be deployed. 

One solution is to just create another folder maybe named something like `local_libs`. Then we could have a requirements file for these libraries and pip install to that folder. Then we can have them vendored. We'll need to make one small tweak to our `appengine_config.py` file. We don't want to try and load the `local_libs` folder if we're on a deployed appspot. Only attempt to load it when we're running locally. So we'll need to check if we're local. To do that we would add something like this to the `appengine_config.py` file:

```
if not os.getenv('SERVER_SOFTWARE', '').startswith('Google App Engine/'):
    vendor.add('local_libs')
```

While this is a viable soltuion it's not one that we generally use. Mostly because it's another requirements file to management. And as you'll see later we'll already be creating a `requirements_dev.txt` file for libraries specific to testing, etc. So how could we leverage using the same `requirements_dev.txt` file here?

First we'll create the separate requirements file named that we name: `requirements_dev.txt`. 

Let's then add a new entry to our Makefile:

```
install-dev:
	pip install -Ur requirements_dev.txt
```

And let's update our install entry while we're at it:

```
install: install-dev
	pip install -Ur requirements.txt -t vendor
```

This will run our `install-dev` command as part of our `install` command.

Now we need to hack GAE a bit. Google runs their standard SDK in a sandbox which wipes out many of our path and environment variables. Why they are so aggressive locally I do not know. Even then it would be fantastic if they'd provide a way to pass in a list of paths to use locally only.

So what can we do? Let's check out what options GAE provides us. If you pass in `-h` to `dev_appserver.py` (`dev_appserver.py -h`) to view the help menu and commands for the server you will see a very long list of options that we can pass into our dev server. The flag we are interested in is `--python_startup_script`. It's description is:

  the script to run at the startup of new Python runtime instances (useful for tools such as debuggers.  (default: None))

We're going to take advantage of the fact that GAE let's us pass in a script to run prior to it executing our application. So we're going to create a startup script that we're going to call: `startup.py`. So go ahead and create that file in the project root folder.

Add this chunk of code to that file:

```
#!/usr/bin/env python

import os
import sys

# Add our Virtual Environment site packages path to our system path on bootup of
# the dev server. This will allow you to install things to your local venv like
# flask, pycrypto, etc and actually get them to be used by the dev_server.
venv = os.environ.get("VIRTUAL_ENV")
if venv:
    sys.path.append("{}/lib/python2.7/site-packages".format(venv))
```

This little hack of a script takes advantage of that `$VIRTUAL_ENV` global variable that we mentioned earlier. We take the path that is store in that variable and we add it to our system path. This puts our venv's site-packages back into the path as it should be.

Now we just need to update our `run` command

```
run:
        dev_appserver.py . --python_startup_script=startup.py
```
