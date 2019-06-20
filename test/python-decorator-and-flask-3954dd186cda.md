Unknown markup type 10 { type: [33m10[39m, start: [33m13[39m, end: [33m14[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m18[39m, end: [33m30[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m40[39m, end: [33m51[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m74[39m, end: [33m96[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m23[39m, end: [33m33[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m10[39m, end: [33m19[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m47[39m, end: [33m52[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m81[39m, end: [33m82[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m121[39m, end: [33m126[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m42[39m, end: [33m52[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m203[39m, end: [33m208[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m89[39m, end: [33m94[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m115[39m, end: [33m136[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m304[39m, end: [33m309[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m28[39m, end: [33m38[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m75[39m, end: [33m80[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m18[39m, end: [33m69[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m71[39m, end: [33m76[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m95[39m, end: [33m110[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m431[39m, end: [33m446[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m48[39m, end: [33m62[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m160[39m, end: [33m169[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m10[39m, end: [33m24[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m50[39m, end: [33m59[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m63[39m, end: [33m88[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m117[39m, end: [33m127[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m165[39m, end: [33m179[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m197[39m, end: [33m206[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m213[39m, end: [33m222[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m199[39m, end: [33m219[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m242[39m, end: [33m256[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m35[39m, end: [33m55[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m77[39m, end: [33m91[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m108[39m, end: [33m114[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m122[39m, end: [33m136[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m23[39m, end: [33m40[39m }

# Python Decorator and Flask



Decorator is one of the my favorite Python features. While seemingly confused at first, it is really just *a function that takes another function as parameter and return another function*. This is possible because in Python, function is a first-class citizen, meaning that function can be used as a parameter, a return value and can be assigned to variable.

Once understood, decorator can be used in various situations and some of which will be discussed below.

By definition,
> a decorator is a function that takes another function and extends the behavior of the latter function *without* explicitly modifying it

Concretely the code below:

    [@decorator](http://twitter.com/decorator)
    def f(argument):
        ….

will replace f by decorator(f): calling f(argument) is then equivalent to decorator(f)(argument).

Flask, a popular and lightweight web framework uses decorator extensively that results in a elegant and intuitive code (and that also results in a huge popularity of this micro framework recently). Decorator uses can be found throughout the framework, from defining routes to adding hooks to application/request lifecycle. In this small post, some use cases of decorators in Flask will be discussed. Please note that this post is intended for users who have experiences with Flask and decorators. The code examples are also mostly pseudocode to illustrate ideas rather than be immediately usable. If you want to learn about Flask or decorator, you can find some ressources at the end of the post.

**First thing first, how [@app](http://twitter.com/app).route is implemented?**

The first example of Flask usually begins with something like:

    [@app](http://twitter.com/app).route(“/”)
    def index():
        return “Hello world”
    

By having app.route as decorator, the function index is registered for the route / so that when that route is requested, index is called and its result “Hello world” is returned back to the client (be it web browser, curl, etc).

Have you ever wondered how the ubiquitous [@app](http://twitter.com/app).route is implemented? As the decorator is usually used to modify a function behavior, its effect can normally be seen when this function is *called*. Whereas index is not called anywhere in the code.

An unpopular fact about decorator is that it is **definition-time and not runtime**, that is index will be replaced by [app](http://twitter.com/app).route('/')(index) when the module is imported. The fact that this registering is done in definition time is important here, as if it wasn’t the case, Flask would have no way to know of index existence.

The implementation logic of [@app](http://twitter.com/app).route is actually quite simple, basically index is added into a global url-function mapping variable that will be searched upon when a request comes in:

    class App:
        route_functions = {}
        def route(url_pattern):
            def wrap(f):
                route_functions[url_pattern] = f
                return f
            return wrap

so when replacing index = app.route(“/”)(index) = wrap(index) = index, index is added into the route_functions that contains route-function matching.

**Login decorator**

Let’s say you have a Flask application that allows user to add tweets. Each tweet belongs to a user and only logged in user can add tweet. Therefore you want to limit some routes to only logged in users and if user is not logged in, they will be redirected to the login page. Adding the login check for every route is quite cumbersome and violates the DRY principle. It would be very handy if we can just decorate such routes with @login_required and all the login checks will be done automatically.

    @app.route("/add_tweet")
    @login_required
    def add_tweet():
       ...

Suppose that the user verification uses cookie, login_required first checks the cookie, if the cookie is not valid, redirect user to login page. Otherwise call add_tweet:

    from functools import wraps

    def login_required(f):
        **@wraps(f)**
        def wrap(*args, **kwargs):

            # if user is not logged in, redirect to login page      
            if not request.headers["authorization"]:
                return redirect("login page")

            # get user via some ORM system
            user = User.get(request.headers["authorization"])

            # make user available down the pipeline via flask.g
            g.user = user

            # finally call f. f() now haves access to g.user
            return f(*args, **kwargs)
       
        return wrap

Note that login_required needs to be placed **after** app.route so login_required(add_tweet) will be called and not just add_tweet when the route matches. Otherwise, if login_required is placed **before** app.route, only add_tweet is called when the route is requested and therefore the login requirement check has no effect.

**Roles based decorator**

The previous login decorator can be extended to take into account various user roles. Let’s say we have admin users that can call routes that other users cannot. It would be nice to have a decorator admin_login_required, that, when used with login_required marks a route only available for admin users.

    def admin_login_required():
        def wrap(*args, **kwargs):
            # user is available from [@login_required](http://twitter.com/login_required)
            if not g.user.is_admin:
                return "you need to be admin", 401

    return f(*args, **kwargs)

So to decorate a route that requires admin access, one can just add these 2 decorators:

    [@app](http://twitter.com/app).route("/admin/delete_user")
    [@login_required](http://twitter.com/login_required)
    [@admin_login_required](http://twitter.com/admin_login_required)
    def admin_delete_user():
        ...

The order here is again important: admin_login_required must be placed after login_required to benefit from g.user set in login_required.

**Page caching**

Let’s say you have some routes returning html pages that are rendered using some query parameters. The same URL will then result in a identical page. That is inefficient to render the same page each time, caching would be useful. We can implement a quick caching mechanism that consists of a url-response dictionary as follow:

[**Other uses](http://twitter.com/admin_login_required)**

Making use of decorator can resolve various DRY problems in a elegant way:

* Get input from request query, json, form with type checking, so that if the input is missing or not correctly-typed, return 400

* Add elapsed time logging to know what routes is slow

* Etc

**Some excellent ressources IMO to learn Flask and decorators:**

* [http://flask.pocoo.org/docs/0.12/tutorial/](http://flask.pocoo.org/docs/0.12/tutorial/)

* [https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world](https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world)

* [http://simeonfranklin.com/blog/2012/jul/1/python-decorators-in-12-steps/](http://simeonfranklin.com/blog/2012/jul/1/python-decorators-in-12-steps/)

* [https://www.thecodeship.com/patterns/guide-to-python-function-decorators/](https://www.thecodeship.com/patterns/guide-to-python-function-decorators/)
