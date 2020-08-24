Unknown markup type 10 { type: [33m10[39m, start: [33m44[39m, end: [33m49[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m159[39m, end: [33m164[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m9[39m, end: [33m14[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m17[39m, end: [33m22[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m122[39m, end: [33m127[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m145[39m, end: [33m149[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m70[39m, end: [33m74[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m59[39m, end: [33m69[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m120[39m, end: [33m124[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m145[39m, end: [33m155[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m35[39m, end: [33m39[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m41[39m, end: [33m51[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m125[39m, end: [33m128[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m45[39m, end: [33m55[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m106[39m, end: [33m110[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m37[39m, end: [33m42[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m62[39m, end: [33m66[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m62[39m, end: [33m72[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m105[39m, end: [33m108[39m }

# My Simple Python Development Setup

Keep it simple and keep your environments isolated

![Photo by [Max Nelson](https://unsplash.com/@maxcodes?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/python?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)](https://cdn-images-1.medium.com/max/12000/1*oJ6-SZXqS_BAAb8691MJ7Q.jpeg)*Photo by [Max Nelson](https://unsplash.com/@maxcodes?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/python?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)*

I love Python. My first real development job was developing [Django](https://www.djangoproject.com/) web applications and I always loved the expressiveness of the language and how it’s a decent tool for the job in almost all areas of software development that I’m interested in.

This is a brief explanation of the environment I use to develop Python applications and how to set it up.

## What Are My Setup Requirements?

* Keep the OS default Python installation untouched and pristine.

* Manage multiple isolated Python versions.

* Manage virtual environments to isolate project requirements.

## Keep Your OS Default Python Installation Untouched

I like to keep my operating system as clean as possible, everything runs more smoothly this way. When developing and especially when testing and learning new things, we all make mistakes and break things.

Isolate your development environment, so when you inevitably break something, the damage is also isolated.

The Python version of your operating system belongs to your OS. It’s usually not the Python version you want, in the case of macOS it’s Python 2.7, and by installing packages globally, you’ll sooner or later have problems when needing multiple versions of the same package.

The other reason why you should leave your operating system’s Python alone is that some processes of your operating system might use that Python version. If you change it, you could damage your operating system.

## Manage Multiple Isolated Versions of Python

For managing multiple Python versions I use [pyenv](https://github.com/pyenv/pyenv). Even though I always try to use the latest Python release, sometimes you need to work on an older codebase. pyenv helps you easily switch the Python version.

As I use macOS, I tend to install almost all the development tools I need using [Homebrew](https://docs.brew.sh/Homebrew-and-Python).

    brew install pyenv

Usually, pyenv will be added to your path after installation. To install any Python version, do the following:

    # Show the list of available python versions

    # If you don’t see the python version you may try updating pyenv to its latest version

    pyenv install — list

    # Install the version you want

    pyenv install -v 3.7.2

    # List your installed python versions (The * will indicate your current version)

    pyenv versions

    # Change your global python version

    pyenv global 3.6.8

    # Sets a location specific python version (creates a .python-version file)

    pyenv local 2.7.15

    # Uninstall any version

    pyenv uninstall 2.7.15

One advantage of pyenv is that it builds each Python version you install from the source and they are all located in your pyenv root directory:

    ~/.pyenv/versions/

## **Manage Virtual Environments**

The way you keep your project dependencies isolated and well-managed when developing in Python are virtual environments.

There are a lot of different tools to do this, if you want a more in-depth comparison between the most common tools check out this great Stack Overflow answer.
[**What is the difference between venv, pyvenv, pyenv, virtualenv, virtualenvwrapper, pipenv, etc?**
*virtualenv is a very popular tool that creates isolated Python environments for Python libraries. If you're not…*stackoverflow.com](https://stackoverflow.com/questions/41573587/what-is-the-difference-between-venv-pyvenv-pyenv-virtualenv-virtualenvwrappe)

Both of the tools we are going to discuss work similarly. They create a folder for your Python installation and dependencies and they modify the PATH environment variable to point it to the desired installation.

As I try to use the latest Python 3.x version for new projects, I use [venv](https://docs.python.org/3/library/venv.html) to manage virtual environments. It’s included in Python’s standard library since version 3.3, this means you don’t have to install any extra tools.

Imagine you want to start a new project using Python 3, you create a new folder for your project and run:

    # Create our virtual environment named “env”

    python3 -m venv env

    # A python installation in a new env folder will be created.

    # Activate your virtual env

    source env/bin/activate

    # You’ll be working in your virtual environment…. Happy hacking!

    # To leave your virtual environment

    deactivate

If you need to support Python 2 or versions below 3.3 then [virtualenv](https://virtualenv.pypa.io/) is the way to go. Actually, the implementation of venv is heavily based in virtualenv so the way they work is really similar.

One main difference is that unlike venv, virtualenv is not a part of Python’s standard library. You need to install it using [pip](https://pypi.org/project/pip/):

    pip install virtualenv

The way you create virtual environments with virtualenv is really similar to what we previously saw using venv. Navigate to your project folder and run:

    # Create our virtual environment named “env”

    virtualenv env

    # A python installation in a new "env" folder will be created.

    # Activate your virtual environment

    source env/bin/activate

    # You’ll be working in your virtual environment…. Again, happy hacking!

    # To leave your virtual environment

    deactivate

## **Recap**

* Don’t mess up your OS Python installation, you’ll regret it.

* Manage multiple Python versions with pyenv.

* Avoid installing project-specific packages globally, the solution is to use virtual environments.

* To manage virtual environments for Python versions >= 3.3 use venv, it comes in the standard library.

* To manage virtual environments for older Python versions, use virtualenv, it needs to be installed using pip.
