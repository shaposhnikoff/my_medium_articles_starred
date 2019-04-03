
# Structuring your Flask App Like an Adult

Leverage Blueprints, Flask-Assets, and the Application Factory.

![](https://cdn-images-1.medium.com/max/3000/1*EJk375yL2rZgpSZ_tCftQA.jpeg)

When we first started developing in Flask, most of us took the 5 lines of code in the quick-start guide and ran with it. Compared to every other web framework, getting a “Hello world” to flash on screen without being hassled with database configurations, template preferences, or reverse proxy setups felt a lot like robbing a bank.

At some point or another, we inevitably pause the party and take a look around. All of our views are smushed into a single file named something meaningless like app.py. All logic lives in the root directory. We're in our 30s and the app we've just created looks as terrible as our bathrooms. It's time to get our shit together.

## The Flask Application Factory

The overwhelming preference to start a Flask application is to use a structure dubbed the [Application Factory](http://flask.pocoo.org/docs/1.0/patterns/appfactories/). The gist is to keep the initialization preferences of our application in a single __init__.py file, sometimes borrowing help from peer files such as db.py or models.py. Either way, the gist is to keep global logic separated from the other parts.

A simple app using the application factory method might look something like this:

    [app] 
    ├── myapp/ 
    │   ├── __init__.py 
    │   ├── db.py 
    │   ├── forms.py 
    │   ├── models.py 
    │   ├── views.py 
    │   ├── static/ 
    │   └── templates/ 
    ├── config.py 
    ├── requirements.txt 
    ├── setup.py 
    ├── Pipfile 
    ├── Pipfile.lock 
    ├── README.md 
    ├── app.json 
    └── wsgi.py

The main takeaway here being the presence of the **myapp** directory which now houses our app logic and the presence of our good friend __init__.py.

An example of what might live in __init.py__ could be something like this:

    import os
    import sys
    from flask import Flask, g
    from config import BaseConfig
    from flask_login import LoginManager
    from flask_pymongo import PyMongo

    def create_app():
        app = Flask(__name__, instance_relative_config=True)
        app.config.from_object('config.BaseConfig')
        login = LoginManager()

    with app.app_context():
            from . import views
            from . import auth
            login.init_app(app)
            mongo = PyMongo(app, app.config['MONGO_URI'])
            app.register_blueprint(views.main)
            app.register_blueprint(admin.admin)

    return app

Another common practice is to keep libraries which need to run init_app in this file as well. This could be something like the LoginManager seen in the example above, or a global database configuration. Lastly, this is also where we would register any **Blueprints** as well.

## Using Blueprints in Flask

While the Application Factory is a good first step in structuring our app as a good self-respecting human being would, we haven’t solved the problem of organizing our app into standalone parts. **Blueprints** are a way for us to separate our app into parts which share very little with one another. Prime examples would include apps with an *admin* panel with an accompanying *client-facing* side, or apps where the *logged in *state is vastly different from the app’s *logged out* state. In these cases, it seems silly to mix both logic and static assets into a single lump, which is where Blueprints come in.

**NOTE:** If you’re a Django person and this is all starting to sound familiar, that’s because we can equate Flask’s **Blueprints** to Django’s concept of **apps.** There are differences and added flexibility, but the concept remains the same.

Registering a part of your app as a Blueprint begins with the following two lines:

    from flask import 

    Blueprint auth = Blueprint('auth', __name__)

When we registered the admin Blueprint previously in __init__.py, the line app.register_blueprint(admin.admin) is essentially saying "look for a **Blueprint** named *admin* in a module (either file or folder structure) called *admin."* It's important not to overlook the concept that Blueprints can either be single files or *entirely standalone file structures with their own templates and static files.* For instance, a Flask app with completely decoupled Blueprints might be structured as follows:

    [app]
    ├── myapp/
    │   ├── __init__.py
    │   ├── admin/
    │   │    ├── __init__.py
    │   │    ├── views.py
    │   │    ├── forms.py
    │   │    ├── static/
    │   │  └── templates/
    │   ├── frontend/
    │   │    ├── __init__.py
    │   │    ├── views.py
    │   │    ├── forms.py
    │   │    ├── static/
    │   │  └── templates/  
    │   ├── db.py
    │   ├── forms.py
    │   ├── models.py
    │   └── views.py
    ├── config.py
    ├── requirements.txt
    ├── setup.py
    ├── Pipfile
    ├── Pipfile.lock
    ├── README.md
    ├── app.json
    └── wsgi.py

In the case above, each **Blueprint** lives as its own Python module and contains its own templates and static files.

## Using Flask-Assets with Blueprints

We’ve already forced a lot of information down your throat, but there’s one last thing worth mentioning in this overview of working with **Blueprints, **which is working with assets. We’ve previously looked at the Flask-Static-Compress library for static asset management, but **Blueprints** lend themselves better to Flask-Assets way of thinking.

Flask-Assets is a library which creates a bundle (aka compressed) of assets upfront. Thus, the start of a Blueprint definition might now look something like this:

    from flask import Blueprint
    from flask_assets import Environment, Bundle, build
    import sass

    auth = Blueprint('admin', __name__)

    assets = Environment(admin)
    scss = Bundle('scss/main.scss', 'scss/forms.scss', filters='libsass', output='build/css/style.css')
    assets.register('scss_all', scss)
    scss.build()

Environment states the context of our asset bundle, which is *admin*, the current Blueprint.

Bundle takes any number of files to compressed together as arguments. Then we must pass the type of "filter" the assets are (typically a precompiler) and of course an output destination for the Bundle.

.register() registers the bundle we just created, not unlike the way we registered our Blueprint.

.build() must be called explicitly in order to build the bundle at runtime. Conversely, we could intentionally exclude .build() if we expect our assets are not to change.

## And Now You Know Everything

…or not, really. The most we should take from this post is:

* There’s a better way to structure our apps.

* There are many potential decisions we can make about the structure of our app.

* There’s way more stuff we need to Google or look up on StackOverflow.

Truthfully, there are plenty of resources within [Flask’s own documentation](http://flask.pocoo.org/docs/1.0/tutorial/views/#) or around the internet that cover the topic of Flask app organization and its granular topics more than this single post could ever hope to. Nonetheless, here’s to hoping you’re feeling a sense of direction in these crazy, adult lives of ours.

*Originally published at [hackersandslackers.com](https://hackersandslackers.com/structuring-your-flask-app/) on October 15, 2018.*
