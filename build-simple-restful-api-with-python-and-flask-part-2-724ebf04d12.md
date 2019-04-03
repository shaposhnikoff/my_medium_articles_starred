
# Build Simple Restful Api With Python and Flask Part 2



In this article I will show you how to build simple restful api with flask and SQLite that have capabilities to create, read, update, and delete data from database.

Similar with my article on part 1([https://medium.com/python-pandemonium/build-simple-restful-api-with-python-and-flask-part-1-fae9ff66a706](https://medium.com/python-pandemonium/build-simple-restful-api-with-python-and-flask-part-1-fae9ff66a706)) I have tested the code presented in this article on python 2.7, though it likely be okay on python 3.4 or newer. Also I assume you have installed flask on your computer, I have explained this step on part 1.

**a. Installing flask-sqlalchemy and flask-marshmallow**

SQLAlchemy is python SQL toolkit and ORM that gives developer the full power and flexibility of SQL. Where flask-sqlalchemy is flask extension that adds support for SQLAlchemy to flask application([http://flask-sqlalchemy.pocoo.org/2.1/](http://flask-sqlalchemy.pocoo.org/2.1/)).

In other hand flask-marshmallow is flask extension to integrate flask with marshmallow(an object serialization/deserialization library). In this article we use flask-marshmallow to rendered json response.

You could installing flask-sqlalchemy and flask-marshmallow easily using pip, with the following command:

$ pip install flask_sqlalchemy
$ pip install flask_marshmallow
$ pip install marshmallow-sqlalchemy

**b. Prepare your code**

Create new python file on flask_tutorial folder named it crud.py. Write down following code in crud.py.

    **from **flask **import **Flask, request, jsonify
    **from **flask_sqlalchemy **import **SQLAlchemy
    **from **flask_marshmallow **import **Marshmallow
    **import **os
    
    app = Flask(__name__)
    basedir = os.path.abspath(os.path.dirname(__file__))
    app.config[**'SQLALCHEMY_DATABASE_URI'**] = **'sqlite:///' **+ os.path.join(basedir, **'crud.sqlite'**)
    db = SQLAlchemy(app)
    ma = Marshmallow(app)
    
    
    **class **User(db.Model):
        id = db.Column(db.Integer, primary_key=True)
        username = db.Column(db.String(80), unique=True)
        email = db.Column(db.String(120), unique=True)
    
        **def **__init__(self, username, email):
            self.username = username
            self.email = email
    
    
    **class **UserSchema(ma.Schema):
        **class **Meta:
            *# Fields to expose
            *fields = (**'username'**, **'email'**)
    
    
    user_schema = UserSchema()
    users_schema = UserSchema(many=True)
    
    
    *# endpoint to create new user
    *@app.route(**"/user"**, methods=[**"POST"**])
    **def **add_user():
        username = request.json[**'username'**]
        email = request.json[**'email'**]
        
        new_user = User(username, email)
    
        db.session.add(new_user)
        db.session.commit()
    
        **return **jsonify(new_user)
    
    
    *# endpoint to show all users
    *@app.route(**"/user"**, methods=[**"GET"**])
    **def **get_user():
        all_users = User.query.all()
        result = users_schema.dump(all_users)
        **return **jsonify(result.data)
    
    
    *# endpoint to get user detail by id
    *@app.route(**"/user/<id>"**, methods=[**"GET"**])
    **def **user_detail(id):
        user = User.query.get(id)
        **return **user_schema.jsonify(user)
    
    
    *# endpoint to update user
    *@app.route(**"/user/<id>"**, methods=[**"PUT"**])
    **def **user_update(id):
        user = User.query.get(id)
        username = request.json[**'username'**]
        email = request.json[**'email'**]
    
        user.email = email
        user.username = username
    
        db.session.commit()
        **return **user_schema.jsonify(user)
    
    
    *# endpoint to delete user
    *@app.route(**"/user/<id>"**, methods=[**"DELETE"**])
    **def **user_delete(id):
        user = User.query.get(id)
        db.session.delete(user)
        db.session.commit()
    
        **return **user_schema.jsonify(user)
    
    
    **if **__name__ == **'__main__'**:
        app.run(debug=True)

For short code above will have 5 endpoints with capabilities to create new record, get all records from database, get record detail by id, update selected record, and delete selected record. Also on this code we define model for our database.

Let’s take a look to the code part by part

    **from **flask **import **Flask, request, jsonify
    **from **flask_sqlalchemy **import **SQLAlchemy
    **from **flask_marshmallow **import **Marshmallow
    **import **os

On this part we import all module that needed by our application. We import Flask to create instance of web application, request to get request data, jsonify to turns the JSON output into a [**Response](http://flask.pocoo.org/docs/0.12/api/#flask.Response)** object with the *application/json* mimetype, SQAlchemy from flask_sqlalchemy to accessing database, and Marshmallow from flask_marshmallow to serialized object.

    app = Flask(__name__)
    basedir = os.path.abspath(os.path.dirname(__file__))
    app.config[**'SQLALCHEMY_DATABASE_URI'**] = **'sqlite:///' **+ os.path.join(basedir, **'crud.sqlite'**)

This part create an instances of our web application and set path of our SQLite uri.

    db = SQLAlchemy(app)
    ma = Marshmallow(app)

On this part we binding SQLAlchemy and Marshmallow into our flask application.

    **class **User(db.Model):
        id = db.Column(db.Integer, primary_key=True)
        username = db.Column(db.String(80), unique=True)
        email = db.Column(db.String(120), unique=True)
    
        **def **__init__(self, username, email):
            self.username = username
            self.email = email

After import SQLAlchemy and bind it to our flask app, we can declare our models. Here we declare model called User and defined its field with it’s properties.

    **class **UserSchema(ma.Schema):
        **class **Meta:
            *# Fields to expose
            *fields = (**'username'**, **'email'**)
    
    
    user_schema = UserSchema()
    users_schema = UserSchema(many=True)

This part defined structure of response of our endpoint. We want that all of our endpoint will have JSON response. Here we define that our JSON response will have two keys(username, and email). Also we defined user_schema as instance of UserSchema, and user_schemas as instances of list of UserSchema.

    *# endpoint to create new user
    *@app.route(**"/user"**, methods=[**"POST"**])
    **def **add_user():
        username = request.json[**'username'**]
        email = request.json[**'email'**]
        
        new_user = User(username, email)
    
        db.session.add(new_user)
        db.session.commit()
    
        **return **jsonify(new_user)

On this part we define endpoint to create new user. First we set the route to “/user” and set HTTP methods to POST. After set the route and methods we define function that will executed if we access this endpoint. On this function first we get username and email from request data. After that we create new user using data from request data. Last we add new user to data base and show new user in JSON form as response.

    *# endpoint to show all users
    *@app.route(**"/user"**, methods=[**"GET"**])
    **def **get_user():
        all_users = User.query.all()
        result = users_schema.dump(all_users)
        **return **jsonify(result.data)

On this part we define endpoint to get list of all users and show the result as JSON response.

    *# endpoint to get user detail by id
    *@app.route(**"/user/<id>"**, methods=[**"GET"**])
    **def **user_detail(id):
        user = User.query.get(id)
        **return **user_schema.jsonify(user)

Like on previous part on this part we define endpoint to get user data, but instead of get all the user here we just get data from one user based on id. If you look carefully at the route, you can see different pattern on the route of this endpoint. Patern like “**<id>” **is parameter, so you can change it with everything you want. This parameter should put on function parameter(in this case **def **user_detail(id)) so we can get this parameter value inside function.

    *# endpoint to update user
    *@app.route(**"/user/<id>"**, methods=[**"PUT"**])
    **def **user_update(id):
        user = User.query.get(id)
        username = request.json[**'username'**]
        email = request.json[**'email'**]
    
        user.email = email
        user.username = username
    
        db.session.commit()
        **return **user_schema.jsonify(user)

On this part we define endpoint to update user. First we call user that related with given id on parameter. Then we update username and email value of this user with value from request data.

    *# endpoint to delete user
    *@app.route(**"/user/<id>"**, methods=[**"DELETE"**])
    **def **user_delete(id):
        user = User.query.get(id)
        db.session.delete(user)
        db.session.commit()
    
        **return **user_schema.jsonify(user)

Last we define endpoint to delete user. First we call user that related with given id on parameter. Then we delete it.

**c. Generate SQLite database**

In the previous step you have write code to handle CRUD operation to SQLite, but if you run this python file and try to access the endpoint(you can try to access localhost:5000/user) you will get the error message similar with
> OperationalError: (sqlite3.OperationalError) no such table: user [SQL: u’SELECT user.id AS user_id, user.username AS user_username, user.email AS user_email \nFROM user’]

This error message occurred because you trying to get data from SQLite, but you don’t have SQLite database yet. So In this step we will generate SQLite database first before running our application. You can generate SQLite database based on your model in crud.py using following step.

1. enter into python interactive shell

First you need to enter into python interactive shell using following command in your terminal:

$ python

2. import db object and generate SQLite database

Use following code in python interactive shell

>>> from crud import db
>>> db.create_all()

And crud.sqlite will be generated inside your flask_tutorial folder.

**d. Run flask application**

Now after generate sqlite database, we are ready to run our flask application. Run following command from your terminal to run your application.

$ python crud.py

And we are ready to try our flask application. To try endpoint in our flask application we can using tools for API development, like curl or postman. Personally I like postman for api development([https://www.getpostman.com/postman](https://www.getpostman.com/postman)). In this article I will use postman for accessing the endpoint.

1. Create new user

![](https://cdn-images-1.medium.com/max/2732/1*PzaKpUO819ADQTNLk--O3A.png)

2. Get all user

![](https://cdn-images-1.medium.com/max/2732/1*ZORHZnLJgX2m--x8UqIP5w.png)

3. Get user by id

![](https://cdn-images-1.medium.com/max/2732/1*Key6eoZquYaFr0sMx7ANsg.png)

4. Update user

![](https://cdn-images-1.medium.com/max/2732/1*BsOlON0NoHmdmaw1Fc94GQ.png)

![](https://cdn-images-1.medium.com/max/2732/1*-296lkba6Rd_00OkXDGN_Q.png)

5. Delete user

![](https://cdn-images-1.medium.com/max/2732/1*IbU54bo4GAqxKqYESwVXcg.png)

![](https://cdn-images-1.medium.com/max/2732/1*ToKKImn3aFEzQuoiEUbaZw.png)

That’s all for this article. Next I plan to write small test using pytest.
