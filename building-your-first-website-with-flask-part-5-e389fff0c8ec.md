Unknown markup type 10 { type: [33m10[39m, start: [33m75[39m, end: [33m78[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m109[39m, end: [33m118[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m179[39m, end: [33m188[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m198[39m, end: [33m207[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m307[39m, end: [33m319[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m381[39m, end: [33m392[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m89[39m, end: [33m101[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m34[39m, end: [33m44[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m87[39m, end: [33m98[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m60[39m, end: [33m71[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m28[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m17[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m54[39m, end: [33m66[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m204[39m, end: [33m208[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m57[39m, end: [33m72[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m135[39m, end: [33m142[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m114[39m, end: [33m123[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m177[39m, end: [33m179[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m21[39m, end: [33m29[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m108[39m, end: [33m119[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m162[39m, end: [33m169[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m49[39m, end: [33m56[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m14[39m }

# Building Your First Website with Flask: Part 5

Creating forms and databases with Flask

![Photo by [Hal Gatewood](https://unsplash.com/@halgatewood?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/websites?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)](https://cdn-images-1.medium.com/max/3900/1*Gku0rkmFowUOYBXl5i9oNg.jpeg)*Photo by [Hal Gatewood](https://unsplash.com/@halgatewood?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/websites?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)*

Our team has been writing several posts recently focused on developing a website using Flask.

* [Part 1 — Hello World and Beyond](https://medium.com/better-programming/building-your-first-website-with-flask-part-1-903a8b44e806)

* [Part 2 — HTML Templates and CSS](https://medium.com/better-programming/building-your-first-website-with-flask-part-2-6324721be2ae)

* [Part 3 — Keeping Track of Sessions and Intro to Forms](https://medium.com/better-programming/building-your-first-website-with-flask-part-3-99df7d589078)

For Part 4 of this series, we’ll be covering creating a database to handle various data. After creation, a set of features available to database management in Flask gets considered as well. So for the fourth part, we plan to achieve the following from our simple website:

[· Inserting data from a form]()

· Incorporating a database

· Handling basic user profile

## Inserting Data From a Form

With Flask, form data can be sent to a template.** **The form data received will trigger a function that forwards the collected data to a template for rendering on selected web pages.

In the example below, we will use the home page, which can be found at the **‘/’** URL. This renders a web page (user.html) with a form. The data filled in by a user is posted to the ‘/result’ URL. The results() function triggers the output on the resulting URL. The function also collects form data present in request.form using a dictionary style (dictionary object) and sends it to result.html for rendering.

The template renders the form data dynamically into an HTML table. Below is the code for FormTable.py*.*

<iframe src="https://medium.com/media/73e3e4c49e96d7738d05dc21fef33c5d" frameborder=0></iframe>

Given below is the HTML script of users.html.

<iframe src="https://medium.com/media/097713715538858c2e4cbe547a8ab0f4" frameborder=0></iframe>

Once you submit the form, it will call the result HTML page. The code of the template (result.html) is given below:

<iframe src="https://medium.com/media/537b91804dfff3edfd4c63709fb24e73" frameborder=0></iframe>

Run the Python script, and enter the URL [http://localhost:5000/](http://localhost:5000/) in the browser.

The output of going to the form will look like this (minus what we typed in):

![](https://cdn-images-1.medium.com/max/2000/1*_NAXrsHw6Es8AUI86YSJQw.png)

When the submit button is clicked, form data is rendered on result.html in the form of an HTML table.

![](https://cdn-images-1.medium.com/max/2000/1*AMp0zocWTFiIxq361JEhmA.png)

## Incorporating a Database

If you recall, in our comparison of [Flask vs. Django](https://www.coriers.com/python-backends-flask-vs-django/), Flask is a micro-web framework in Python. This definition implies it does not feature an object-relational mapper (ORM) (whereas Django does).

So if you wish to incorporate an interactive database, then you require the additional installation of an extension. [SQLAlchemy](http://www.sqlalchemy.org/) has become a popular choice to install. In Flask, the ready-made extension for the addition SQLAlchemy is called [Flask-SQLAlchemy](http://flask-sqlalchemy.pocoo.org/).

## Installing Flask-SQLAlchem

The installation of Flask-SQLAlchemy requires the use of pip in your active virtual environment created for Flask. To install the extension, enter the following into your active environment for Flask in Python:

pip install flask-sqlalchemy

After the successful installation of Flask-SQLAlchemy, the next installation needed is MySQL. This installation can be achieved using:

pip install mysql

After the successful installation of Flask-SQLAlchemy alongside other dependencies, it is time to create our interactive database.

## Creating a Database

The use of SQLAlchemy in creating a database is pretty straightforward. SQLAlchemy supports several approaches to working with database. A common approach is the use of declarative syntax that permits you to create classes for modeling the database.

So for our simple Flask tutorial, we would make use of SQLite to serve as the back end. Other back-end solutions, like MySQL or Postgres, could be employed. To begin, we will be looking at how to create:

· A database file through the normal SQLAlchemy

· A separate script that uses the different Flask-SQLAlchemy syntax

So let’s start by putting the code into a file called db_create.py.

<iframe src="https://medium.com/media/a79b00b7ec2bac904ff0293fc0c0a846" frameborder=0></iframe>

The early part of the code may appear similar, as we are asking Python to import SQLAlchemy and other essential extensions needed to make the different parts of the code work.

Subsequently, the code tries to create SQLAlchemy’s engine object, which essentially connects Python to the selected database. Also, we connect the SQLite and create a database file in place of creating a database inside the memory.

Additionally, the base class created is meant to serve the creation of declarative-class definitions. This class definition defines the database table. For example, we might want to define a class called user that is attached to a user table.

Also note the name given to the table is done through the** __tablename__ **class attribute. The creation of the table’s columns is set using the data types we require.

You can read more about it and get in-depth details from the well-written [documentation](http://docs.sqlalchemy.org/en/latest/orm/extensions/declarative/basic_use.html). On running the code above, an error in the output terminal is obtained. Now let’s make all this work in Flask.

## Using Flask-SQLAlchemy to Create a Database

The first thing we require in using Flask-SQLAlchemy is the creation of a simple application script. Let’s call the application script** **apps.py. Now let’s add the code below into the Python file and save it.

Also, a simple secret is to set up a database object at the same time, which permits the integration of SQLAlchemy into Flask.

<iframe src="https://medium.com/media/3c766b7083a4797961224c2108ca1eb2" frameborder=0></iframe>

Next is to create the Flask app object and provide a location for the SQLAlchemy database file. The creation of a models.py file can do the trick. This would contain the following code:

<iframe src="https://medium.com/media/fbf786427de7315cca9fe2a856c36afe" frameborder=0></iframe>

Flask-SQLAlchemy does not necessitate all the imports as required by the plain SQLAlchemy. All we need is the database (db) object created in the app script. From there, we add db to all classes using the original SQLAlchemy code.

Also, the predefined db.Model also serves as the base class. To help initialize the database, we create the db_setup.py*.*

<iframe src="https://medium.com/media/e93be47c0d949ed4c55669f128e892b1" frameborder=0></iframe>

The code above would initialize the database using the table created within the model’s script. To visualize the initialization, let’s make some additions to our test.py script:

<iframe src="https://medium.com/media/720a609d2a58df5a2046acc0bfeed7da" frameborder=0></iframe>

Here we only import our app object alongside the init_db function. Now let’s run the code in the terminal, using the following:

python test.py

While this code is running, you can go to the link below to see what the page looks like:

[http://127.0.0.1:5000/](http://127.0.0.1:5000/)

![](https://cdn-images-1.medium.com/max/2000/1*Ftj7AgYh8VQsaaV8cvQUWw.png)

## Conclusion

We have successfully created a simple database entry using a form as well as a web app that has an empty database.

The database created for the web app cannot accept any input yet, but it offers an introduction to the use of a database.

In the next part, we will integrate the database with the Flask front end so you can start creating new users.
