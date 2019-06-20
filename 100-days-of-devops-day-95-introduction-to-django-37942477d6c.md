
# 100 Days of DevOps — Day 95-Introduction to Django

Welcome to Day 95 of 100 Days of DevOps, Focus for today is Introduction to Django

*Django is a popular Open Source Web Framework and being a Web framework it majorly solves our two major problem*

* *It maps a requested URL from a user to the code that actually meant to handle it*

* *It also allows us to create that requested HTML dynamically*

***Installing Django***

*I am installing Django via Anaconda and then try to create a virtual env. Using Virtual Env we can test new features without breaking the existing app. The reason I am doing it via Anaconda as it includes the virtual environment handler.*

    *conda create — name myvirtualenv django # package we want to initiate env*

*To activate this environment, use:*

    *# > source activate myvirtualenv*

*Once again the advantage of using Virtual Env*

* *Now anything installed via pip or conda when this environment is activated will only be installed for this environment.*

* *It helps projects to keep your project self-contained and not run into issues when we roll new packages*

*Now let’s go back to Django, when we install Django it actually installed a command line tool called django-admin*

*Let’s create our first project*

    *$ django-admin startproject myfirst_project*

*This will create a directory structure like this*

    *$ tree myfirst_project/
    myfirst_project/
    ├── manage.py
    └── myfirst_project
     ├── __init__.py
     ├── settings.py
     ├── urls.py
     └── wsgi.py*

* ***manage.py:** It will be associated with many commands as we build our web app.*

* ***__init__.py :** This is a blank Python script, that due to its special name let’s Python know that this directory can be treated as a package*

* ***settings.py:** This is where we are going to store our project settings*

* ***urls.py: **It stores all the URL patterns for our project. Basically different pages of our web application*

* ***wsgi.py:** Python script that acts as the web server gateway interface, it helps us to deploy our web app to Production*

*Let’s use manage.py now*

    *$ python manage.py runserver
    Performing system checks…*

    *System check identified no issues (0 silenced).*

    *You have 13 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
    Run ‘python manage.py migrate’ to apply them.*

    *May 27, 2017–00:09:46
    Django version 1.10.5, using settings ‘myfirst_project.settings’
    **Starting development server at [http://127.0.0.1:8000/](http://127.0.0.1:8000/)**
    Quit the server with CONTROL-C.
    [27/May/2017 00:09:59] “GET / HTTP/1.1” 200 1767*

*On the web gui, we will see something like this [**http://127.0.0.1:8000/](http://127.0.0.1:8000/)***

![](https://cdn-images-1.medium.com/max/5748/1*CflZNoJ728XxaxXYKRBBDw.png)

*NOTE: For the time being ignore migration warning*

*Let’s create our first web application with our Django Project*

    ***$ python manage.py startapp myfirst_app***

    *$ tree
    .
    ├── db.sqlite3
    ├── manage.py
    ├── myfirst_app
    │ ├── __init__.py
    │ ├── admin.py
    │ ├── apps.py
    │ ├── migrations
    │ │ └── __init__.py
    │ ├── models.py
    │ ├── tests.py
    │ └── views.py
    └── myfirst_project
     ├── __init__.py
     ├── __pycache__
     ├── settings.py
     ├── urls.py
     └── wsgi.py*

    *4 directories, 17 files*

* ***_init__.py :** This is a blank Python script, that due to its special name lets Python know that this directory can be treated as a package*

* ***admin.py:** We can register our models here which Django will then use them with Django admin interface*

* ***apps.py: **We can place application-specific configuration here*

* ***models.py:** To store the application’s data models*

* ***tests.py: **We can store test functions to test our code*

* ***views.py: **We can have functions that handle requests and return responses*

* ***Migrations folder:** This folder stores database specific information as it relates to the model*

*Now we need to let Django know that we created this application*

    *$ pwd
    /Users/plakhera/django/myfirst_project/myfirst_project*

    *vim settings.py*

    *# Application definition*

    *INSTALLED_APPS = [
     ‘django.contrib.admin’,
     ‘django.contrib.auth’,
     ‘django.contrib.contenttypes’,
     ‘django.contrib.sessions’,
     ‘django.contrib.messages’,
     ‘django.contrib.staticfiles’,
    ** ‘myfirst_app’   ---> Add This entry about your application**
    ]*

*Once this time we need to create view*

    */Users/plakhera/django/myfirst_project/myfirst_app*

    *vim views.py*

    *from django.shortcuts import render
    from django.http import HttpResponse*

    *# Create your views here.*

    *def index(request):
     return HttpResponse(”Hello World”)*

*In order to see this view we need to map it via urls.py*

    */Users/plakhera/django/myfirst_project/myfirst_project/*

    *vim urls.py*

    *from django.conf.urls import url
    from django.contrib import admin
    from myfirst_app import views*

    *urlpatterns = [
     url(r’^admin/’, admin.site.urls),
     url(r’^$’, views.index,name=’index’ ),
    ]*

*Now try to run the server*

    *$ python manage.py runserver
    Performing system checks…*

    *System check identified no issues (0 silenced).*

    *You have 13 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
    Run ‘python manage.py migrate’ to apply them.*

    *May 27, 2017–17:00:51
    Django version 1.10.5, using settings ‘myfirst_project.settings’
    Starting development server at [http://127.0.0.1:8000/](http://127.0.0.1:8000/)
    Quit the server with CONTROL-C.
    [27/May/2017 17:00:58] “GET / HTTP/1.1” 200 11*

![](https://cdn-images-1.medium.com/max/3792/1*m1-gfktQePaTeTCcuq3aCQ.png)

*Looking forward from you guys to join this journey and spend a minimum an hour every day for the next 100 days on DevOps work and post your progress using any of the below medium.*

* *Twitter: [@100daysofdevops](http://twitter.com/100daysofdevops) OR @[lakhera2015](https://twitter.com/lakhera2015)*

* *Facebook: [https://www.facebook.com/groups/795382630808645/](https://www.facebook.com/groups/795382630808645/)*

* *Medium: [https://medium.com/@devopslearning](https://medium.com/@devopslearning)*

* *Slack: [https://devops-myworld.slack.com/messages/CF41EFG49/](https://devops-myworld.slack.com/messages/CF41EFG49/)*

* *GitHub Link:[https://github.com/100daysofdevops](https://github.com/100daysofdevops)*

***Reference***
[**100 Days of DevOps — Day 0**
*D-day is just one day away and finally, this is a continuation of the post(I posted a month earlier)*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-0-4f2c9750542d)
[**100 Days of DevOps**
*Motivation*medium.com](https://medium.com/@devopslearning/100-days-of-devops-81faf13bf772)
[**100 Days of DevOps — Day 94-Introduction to Numpy for Data Analysis**
*Welcome to Day 94 of 100 Days of DevOps, Focus for today is Introduction to Numpy for Data Analysis*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-94-introduction-to-numpy-for-data-analysis-127561af9e1d)
