
# Build Simple Restful Api With Python and Flask Part 1



I’m going to divide this series into 3 or 4 articles. At the end of the series you would understand how easy to build restful API with flask. In this article we’ll setting our environment and create endpoint that will show “Hello World”.

![](https://cdn-images-1.medium.com/max/2000/1*k9dkRUxUNwk2FB9iXZZkxg.jpeg)

I assumed that you have python 2.7 and pip installed on your computer. I have tested the code presented in this article on python 2.7, though it likely be okay on python 3.4 or newer.

**a. Installing flask**

Flask is microframework for python. The “micro” in microframework means Flask aims to keep the core simple but extensible ([http://flask.pocoo.org/docs/0.12/foreword/#what-does-micro-mean](http://flask.pocoo.org/docs/0.12/foreword/#what-does-micro-mean)). You could installing flask with the following command:

$ pip install Flask

**b. Prepare your IDE**

Actually you could using all kind of text editor to build python app, but it would much easier if your are using IDE. Personally I prefer using Pycharm from jetbrains([https://www.jetbrains.com/pycharm/](https://www.jetbrains.com/pycharm/)).

**c. Create “Hello World” in flask**

First you need to create your project folder, for this tutorial i will named it “flask_tutorial”. If you are using pycharm you could create project folder by choosing File and New Project from menus.

![](https://cdn-images-1.medium.com/max/2732/1*gcwvCdMPZTfuk1CHJPWwVw.png)

After that you could set your project location and interpreter. Your computer could have some python interpreter anyway.

After setting your project, on pycharm right click on your project folder then choose New -> Python File and named it “app.py”.

![](https://cdn-images-1.medium.com/max/2732/1*sH4l8sfvoAfdrUx1EL-93g.png)

Write down code below on app.py.

    **from **flask **import **Flask
    
    app = Flask(__name__)
    
    @app.route(**"/"**)
    **def **hello():
        **return "Hello World!"
    
    
    if **__name__ == **'__main__'**:
        app.run(debug=True)

Run it from terminal. You could using command line or from pycharm you can click Terminal tab that located on bottom left and write code below.

$ python app.py

![](https://cdn-images-1.medium.com/max/2732/1*gVxJULj-kOXq3LKOadQsrg.png)

Open your browser and access localhost:5000. And voila now you have your first flask application :)

![](https://cdn-images-1.medium.com/max/2732/1*qq8iXGKqxPWYMm1KQd1MFw.png)

OK now let’s take a look to the code.

    **from **flask **import **Flask

This line ask the application to import Flask module from flask package. Flask used to create instances of web application.

    app = Flask(__name__)

This line create an instance of your web application. __name__ is a special variable in python, it will equal to “__main__” if the module(python file) being executed as the main program.

    @app.route(**"/"**)

This line define the routes. For example if we set route to “/” like above, the code will be executed if we access localhost:5000/. You could set the route to “/hello” and our “hello world” will be shown if we access localhost:5000/hello.

    **def **hello():
        **return "Hello World!"**

This line define function that will be executed if we access route.

    **if **__name__ == **'__main__'**:
        app.run(debug=True)

This line mean that your flask app will being run if we run from app.py. Notice also that we are setting the debug parameter to true. That will print out possible Python errors on the web page helping us trace the errors.

Ok that’s all for part 1, next we will try CRUD operation on SQLLite with flask.
