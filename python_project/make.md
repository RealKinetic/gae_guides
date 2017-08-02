Makefile
========

At Real Kinetic we're fans of using Makefiles to give us shortcuts to the commands we often run. We like Make as it's simple and is supported on OSX and Linux distributions. If you'd prefer systems like `gulp`, `grunt`, etc feel free to use those. The Makefile is not required.

If you're going to use a Makefile here's how we set ours up.

Create a file named `Makefile` in your project root.  Then add the following:

```
install: install-dev
	pip install -Ur requirements.txt -t vendor

install-dev:
	pip install -Ur requirements_dev.txt

run:
    dev_appserver.py .
```

Now you can run the following commands:

Install your dependencies both local and vendored
```
make install
```

Run the dev server
```
make run
```
