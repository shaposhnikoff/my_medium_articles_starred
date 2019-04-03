
# Flask Routing & Sessions: A Subtle Symphony

With great flexibility comes great responsibility

![](https://cdn-images-1.medium.com/max/3000/1*VLbmX0fZ1MJPJq1aYWCdCA.jpeg)

It isn’t often you find somebody sad or miserable enough to detail the inner workings of web framework features, such as sessions or routing. This is understandably so; we use frameworks because presumably hate dealing with these things from scratch. This is especially so when it comes to Flask, which only released version 1.0 a few months ago, introducing breaking changes rendering previous documentation more-or-less worthless.

Googling some of Flask’s critical features mostly returns one-liners from the app’s authors (half of which are useless, as they are for older versions of Flask). Stack Overflow threads mostly sit in silence, and even [Kite](https://kite.com/), AKA *“The smart copilot for programmers” *returns blank pages of documentation, akin to the blank stare of a clueless Golden Retriever.

***In retrospect, it was probably a poor choice for me to pick up 4 separate Flask-based projects during this time.***

We’re in a historic place in time where a team of developers put together something beautiful, yet somehow feels undersold. It seems as though the niche market of “those who can’t do, teach” remains untapped for Flask, as the usual suspects have yet to “do”. This leaves newcomers like myself to hack away for their own survival in the meantime. I’ve only just turned that mental corner where Flask’s quirks are as comforting as home-cooked meal, as opposed to frustrating single-word methods containing 6 words of documentation on average.

The good news is I am still technically alive, after spending weeks building Flask applications mostly through trial and error. The bad news is that I’ve become *Mr. Robot* in the process. That said, if there will ever be an ideal moment in my life to write about Flask, now is the time. As reality slowly slips away in 1s and 0s, I may as well pass along what I’ve learned.

## Broad Strokes

It only takes a couple minutes into explaining what Flask is when you realize that Flask, at its core, is overwhelmingly just the “V” in “MVC”. Jinja handles the templates, and something like SQLAlchemy will likely handle your models. Flask has an abundance of third-party libraries to handle business logic, but it is the core Flask package that we all agreed to gather around. This speaks volumes about the quality of Flask’s simple yet powerful request handling.

I’ll break down as many of Flask’s out-of-the-box features, focusing on what matters most (in my opinion). Take a look at some of the Flask libraries we’ll be playing around with:

    # app.py 

    from flask import Flask, render_template, request, redirect, Session, g 
    import os

## Configuring Our App

As always, we create our app with app = Flask(name)*. *Equally uninteresting is our configuration setup, which we'll import via a class in config.py:

    # app.py 

    from flask import Flask, render_template, request, redirect, Session, g 
    import config 
    import os 

    # Our app 
    app = Flask(__name__) 

    # Load our config variables app.config.from_object('config.ProductionConfig')

A number of things in our config are absolutely essential for sessions to work. Below is an example config file:

    # config.py 

    import os 
    class ProductionConfig(): 
        """Set app config vars.""" 
         SECRET_KEY = os.urandom(24) 
         SESSION_TYPE = null 
         SESSION_COOKIE_NAME = 'session name' 
         SESSION_PERMANENT = True PERMANENT_SESSION_LIFETIME = timedelta(days=31) (2678400 seconds)

**SECRET_KEY** is critical: this variable needs to exist in out config for sessions to function properly. The best way to handle this is by generating a key as seen above.

**SESSION_TYPE** allows us to specify where our session data should be stored. This is set null by default, but Flask supports a number of options:

There are plenty more variables you can set if you want to take a look [here](http://flask.pocoo.org/docs/1.0/config/).

## Sessions and Contexts

Unlike cookie-based sessions, Flask sessions are handled and stored on the server-side. A session object is simply a dict which is accessible throughout the application at a global level, referring to a single ‘logged-in user’. As long as the session is active, any context of our app will be able to retrieve, modify, or delete the values held in this session object,

    # Save a value to the user's session. 
    session['username'] = 'MyUsername' 

    # IMPORTANT: "True" forces our changes to be recognized. session.modified = True: 

    # Retrieve session values at any time, anywhere session.get('username') = True

Seeing as how sessions are accessible globally, it is also important to note that sessions can last a very long time; pretty much self-explanatory given the SESSION_PERMANENT = True configuration option. It's a good idea to set a session timeout period, or better yet, close them by the user's own request. Clearing a session is as simple as resetting the session dictionary values back to *None* by using the **pop** method: session.pop('value', None).

## The Application Context

Besides undying global sessions, Flask also provides us with a feature with an object more suitable for storing and passing temporary values between app contexts. This object known simply as g. While** **technically an abbreviation for "global", g is really just a convenient place to store temporarily store values which you can always depend on to be by your side.

    # app.py

    from Flask import g

It’s important to note that values assigned to g *only exist within the context they were created *by default. For example, if we store information to the object due to some user interaction on the dashboard, these values are lost once the user moves to another part of our app. That said, values assigned to g can technically be passed between contexts if we return g.value. This distinction between always-alive *sessions* and every dying *g* should be indicative of what reach respective object does. Spoiler alert: sensitive (or contextually useless) data should be stored temporarily with g, where values which will continuously be useful in determining the functionality of our should reside in session*.*

Interestingly enough, Flask has a *decorator* specifically for terminating values saved to g in the case we'd want to ensure the swift and total annihilation of such data. For instance, if we were to assign a database connection to g using g.db = connect_to_database(), we'd want to make sure that connection is closed as fast as possible before we forget:

    # app.py 

    def db_stuff(): 
        g.db = database_connection() 
        g.db.somequeryorwhatever 
        return g.db 

    @app.teardown_appcontext 
    def teardown_db(): 
        db = g.pop('db', None)

## Routes & Decorators

We’re surely familiar with the concepts behind routing users to deserved views by now. Before we look at the juicy stuff, consider this boring route for a boring product, where the homepage is a dashboard:

    # app.py 

    @app.route('/', methods=['GET', 'POST', 'OPTIONS']) 
    def dashboard(): 
        """Boring route.""" 
        return render_template('/dashboard.html')

Oh snap, our landing page is a /*dashboard?* How will we know which user’s dashboard to display when they visit the dashboard, or any other page for that matter? If only there were a way to intercept every request a user makes to our app?

Flask comes with a bunch of insanely useful ***decorators***. Python decorators are functions which either ‘wrap’ other functions in additional logic, or in our case, intercept functions to do with them what we what. Flask has a vast plethora of logic decorators, ranging from detecting first-time visitors, handling exceptions, executing logic before/after page loads, etc. Even the route we set above is a decorator!

## @flask.before_request

Adding **before_request** to our app allows us to run logic prior to the aforementioned request. With this power, we can do things like treat users differently (such as recognized or anonymous users), or just execute some sort of unique logic upon page load.

In this simple case, we check to see if a visitor has an active session every time they hit a route. This way, if a user’s session expired between before hitting a route in our app, we can prompt them to log in… or whatever.

**before_request **doesn’t accept any value parameters — the handler is mostly intended to perform tasks such as making a database query necessary for our app to run, or make sure users are still logged in.

    # app.py 
    @app.before_request def before_request(): 
         """Handle multiple users.""" 
         if 'username' in session: 
             return render_template('/dashboard.html') 
         else: 
             return render_template('/login.html')

## @flask.url_value_preprocessor

Unlike *before_request***, url_value_preprocessor** *does* accept incoming data. This allows us to handle data being posted to any part of our app before we even bother serving up views. Not only does this provide a convenient separation of concerns, but also helps us avoid *callback hell, *which yes, can happen in Python too.

Let’s say we’re accepting a POST request, where we create a view for our user’s personal details. When the user passes us their email address, we decide to retrieve the user’s records by hitting an API, and passing the results to the view.

Without modularizing our code, we’d have to handle things like waiting on API calls in the same functions as our routes. Not only is this shitty repetitive code, but running numerous API calls and rendering a view all at once is going to eventually break. Go ahead and ask the NodeJS guys. They’ll know.

    # app.py 

    @app.url_value_preprocessor 
    def url_value_preprocessor(endpoint, values): 
        """Handle data sent to any route.""" 
         if request.args: 
             email = request.args.get('email') 
             r = requests.post(endpoint, headers=headers, data=email)     
             session['usermetadata'] = r 
             session.modified = True 
             return session

## You’re Only Getting Started

We’ve only covered a small percentage of convenient tools Flask offers us. Go ahead and see [how many decorators](http://flask.pocoo.org/docs/1.0/api/) you can fuck with. Yeah dude, shit is legit — and we haven’t even talked about the Flask-Login package yet.

The beauty of lightweight frameworks is that they focus on the problems that drive us to web frameworks in the first place. Flask is clearly designed to handle serving views, standing up APIs, and handling user management effectively. Contrast this with frameworks like **Django**, which forces rigid app setup in what can commonly be an hour-long setup or greater. I’ll truthfully always have a place in my heart for Django as the fathers of Python MVC: I would say with confidence that without the creation of Django (as well as the official $10 dollar intro book from Barnes and Noble) I never would have transitioned from an obnoxious product manager personality to the kind of guy who owns multiple Python t-shirts. Hmm. Now that I think about it, maybe I should’ve stayed an office tool as opposed to solving all these complex problems. oh well.

Flask is indicative of a new direction of framework design — or rather lack thereof. Programmers who *know what they’re doing* can express themselves outside of traditional boundaries set by other frameworks, surely designed to keep idiots from ruining everything. There’s nothing wrong with being a worker drone repeating the same worthless projects, using same libraries, and essentially contributing nothing to humanity. I’d personally prefer to take the freedom and speed of Flask any day.

*Originally published at [hackersandslackers.com](https://hackersandslackers.com/the-art-of-building-flask-routes/) on September 19, 2018.*
