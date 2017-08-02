A GAE Python Project
====================


While Google has a decent amount of documentation for Google App Engine and the standard getting started guides and tutorials there isn't much material out there on setting up a proper, production ready GAE project. This post is a walk-through of laying out how we structure our Python based Google App Engine - Standard projects. This includes packaging dependencies for deployment as well as ensuring libraries are accessible in the local environment and tests. 

A note this guide focus on Python 2.7 on the GAE standard environment. In future posts we hope to show how to use other languages on the Flexible environment.

[Getting Started](start.md)
[Dependencies and Libraries](dependencies.md)


### Makefile

At Real Kinetic we're fans of using Makefiles to give us shortcuts to the commands we often run. We like Make as it's simple and is supported on OSX and Linux distributions. Create a file named `Makefile` in your project root.

Then add the following:

```
install:
    pip install -Ur requirements.txt -t vendor
```

Now you can run the following command as shorthand for the pip command:

```
make install
```

And while we're at it let's add a command to run our development server. Add the following to your Makefile:

```
run:
    dev_appserver.py .
```
