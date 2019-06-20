
# 100 Days of DevOps — Day 82- Python Object Oriented Programming(OOP)

Welcome to Day 82 of 100 Days of DevOps, Focus for today is Python Object Oriented Programming(OOP)

*We can think OOP as a **paradigm** or a way to organize our code*

*In most of the cases, we store our data in a temporary variable and then we call this variable via a function.*

*OOP uses specially designed entities called **object** to organize data and functionalities to process this data via **methods***

*The design of these objects and methods is specified in a **blueprint** called **classes**.*

*Everything in Python is an **object(instance)** even number or in another way, we can say Instance is a constructed object of the class.*

*An object is an entity that holds data in the form of attributes of a particular **class or type** with associated functionality in the form of methods.*

*So as you can see here every object in Python has a type even None value has a type NoneType*

    *>>> myint = 1*

    *>>> type(myint)*

    ***<class ‘int’>***

    *>>> mystr = “hello”*

    *>>> type(mystr)*

    ***<class ‘str’>***

    *>>> mynone = None*

    *>>> type(mynone)*

    ***<class 'NoneType'>***

    *>>> def myfun():*

    *...     print("hello")*

    *...*

    *>>> type(myfun)*

    ***<class 'function'>***

    *>>> l = [1,2,3,4]*

    *>>> type(l)*

    ***<class 'list'>***

*Now looking at the last example of list do we ever think how the list is represented in memory(**linked list of cells**, **follow pointer to the next index**)*

![](https://cdn-images-1.medium.com/max/2000/1*8TahVV5tTfyCKYmbOjnAug.png)

*But we don’t need to know, the only thing we need to know how to manipulate lists(slicing,len(), append/extend/pop/push) because these procedures takes care for us.*

*Objects can be destroyed*

* *Using **del***

* *Python itself reclaim destroyed or inaccessible objects — using **garbage collection***

***Module vs Classes***

* *Modules are files that contain Python code and can be executed/imported and can contain the class definition*

*The easiest way to understand OOP is via pictorial example*

![](https://cdn-images-1.medium.com/max/2000/1*29pI75HKtVar3AVNp7L2wQ.png)

* *Here we can think of Car as a **class of object***

* *Car class provides the **blueprint** for a car object*

* *We can think **Instance** as a widget coming out of the blueprint*

* *Each instance of a car does the same thing(**method**)*

* *Each instance of a car has own state(**attributes eg: speed, color**)*

*Let take one more example*

    *>>> var = “hello”*

    *>>> print (type(var))*

    *<class ‘str’>*

    *>>> print (var.upper())*

    *HELLO*

*Here **var** is an **Instance***

***Str** is a **class** of **Type string***

***upper** is the **method***

*Let’s define our first class and re-stress some of the points discussed earlier*

* *User-defined objects are created using the **class** keyword*

* *We can think class as a **blueprint** which defines the nature of future objects*

* *From classes, we can construct **instances***

* *An instance is a specific object created from a particular class*

* *By convention classes name is uppercase*

    *>>> class MyFirstClass(object):*

    *… pass **#pass means that object has no content***

    *…*

    *# Creating instance of **MyFirstClass(**x is now the reference to our new instance of a MyFirstClass class or in another word we can say we instantiate a MyFirstClass**)**
    >>> x = MyFirstClass()*

    *# It gives memory location where object is stored(0x107c98898)
    >>> print(x)*

    *<__main__.MyFirstClass object at 0x107c98898>*

* *Currently, inside the class, we just have passed but we can define class **attributes** and **methods***

* ***Attribute: **Characteristic of an object*

* ***Method: **Operation we can perform with the object*

* *The word **object **means** **that MyFirstClass is a python object and **inherits all its attributes(class parent)***

*For eg: As mentioned above in the car example same way, we can create **cat class**. The **Attribute** of a cat may be its **breed** or its **name, **while a method of a cat may be defined by a **.meow() **method which returns a **sound.***

***Attributes***

* *Attributes are the variables that belong to a class(can be of any data type)*

* *Can be accessed after the “.”*

*The syntax for creating attributes*

    *self.attribute = something*

***Special method:** It’s used to initialize the attributes of an object*

    *__init__()*

*The First example look like this*

    ***class Cat(**object**):
        def __init__(**self,breed**):
            **self.breed **= **breed # We are creating attributes here*

    *#Then we are creating instance of Cat
    x **= **Cat**(**breed**="Persian")
    **y **= **Cat**(**breed**="Sphynx")***

*Let’s break down this code*

* *Special method **__init__()** is called automatically right after the object has been created(initialize some data attributes)*

* *Each attribute in a class definition begins with a reference to the instance object. It’s by convention named **self(that’s by convention that we use a self keyword, it actually doesn’t have to be self but it’s not self :-) )**. Value is passed during the class instantiation*

    ***self.breed = breed***

* *When we invoke the creation of an instance, this will bind the variables to breed within that instance to the supplied value*

*Now we have created two instances of the Cat class, with two breed type, to access these attributes*

    *>>> x.breed*

    *‘Persian’*

    *>>> y.breed*

    *‘Sphynx’*

***NOTE**: We don’t have any parenthesis after breed and that’s because it’s an **attribute** and it doesn’t take any arguments.*

*While trying to create an instance of a class we can’t do it like this*

    *x **= **Cat**()***

*It will throw this error*

    *Traceback (most recent call last):
     File “/Users/plakhera/Downloads/salesforce-spring-2017/loggingcode.py”, line 5, in <module>
     x = Cat()
    TypeError: __init__() missing 1 required positional argument: ‘breed’*

*Here self by default is given and it lets the method know we are referencing the cat object.*

*Now let’s take a look at **class object attributes. **These class object attributes are the same for any instance of the class. In the below example cat is always a mammal irrespective of their name and breed*

    *>>> class Cat(object):*

    *… **species = “mammal”***

    *… def __init__(self,breed,name):*

    *… self.breed = breed*

    *… self.name = name*

    *…*

    *>>> x = Cat(“Persian”,”Luca”)*

    *>>> x.name*

    *‘Luca’*

    *>>> x.breed*

    *‘Persian’*

    *>>> x.species*

    *‘mammal’*

***NOTE:** Class Object attributes are defined outside of any methods in the class. Also by convention, we place them first before the init.*

***Methods***

*We can think of methods as functions defined inside the class*

    ***class Circle(**object**):
        **pi **= **3.14*

    *    **def __init__(**self,radius**=**1**):
            **self.radius **= **radius*

    *    **def area(**self**):
            return **self.radius *** **self.radius *** **Circle.pi*

    *    **def setRadius(**self,radius**):
            **self.radius **= **radius*

    *    **def getRadius(**self**):
            return **self.radius*

    *x **= **Circle**()
    **x.setRadius**(**3**)
    **print**(**x.getRadius**())
    **print**(**x.area**())***

*Let’s break down the code and see what’s going on*

* *We instantiated a circle with a default radius of 1*

* *We define the method to calculate the area*

* *Then we define the method to reset the radius*

* *Finally a method for getting a radius*

***Inheritance***

*Inheritance is a way to form new classes**(descendants)** using classes that have already been defined**(ancestors)**. The newly formed classes are called **derived classes** and the classes that we derive from are called base classes. In this way, we can take advantage of code reuse.*

    ***class Animal(**object**):
        def __init__(**self**):
            **print**("Welcome to the Animal World")***

    ***    def name(**self**):
            **print**("Animal here")***

    ***    def eat(**self**):
            **print**("Time to eat")***

    ***class Dog(Animal):
        def __init__(**self**):
            **Animal.__init__**(**self**)
            **print**("welcome to Dog World")***

    ***    def name(**self**):
            **print**("Dog here")***

    ***    def bark(**self**):
            **print**("Woof woof time")***

*Output*

    *x **= **Dog**()
    **Welcome to the Animal World
    welcome to Dog World*

    *x.name**()
    **Dog here*

    *x.eat**()
    **Time to eat*

    *x.bark**()
    **Woof woof time*

*Let’s break this code*

* *We have two classes Animal(base class)and Dog(derived class)*

* *The derived class modify the existing behavior of the base class(name())*

* *Finally derived class extends the functionality of the base class by defining new bark() method*

***Special Methods: **Let’s take a look at some special methods in Python*

    ***class Book(**object**):
        def __init__(**self,name,author,pages**):
            **print**("My first book")
            **self.name **= **name
            self.author **= **author
            self.pages **= **pages*

    *    **def __str__(**self**):
            return "Name:%s, author:%s, pages:%d" %(**self.name, self.author,self.pages**)***

    ***    def __len__(**self**):
            return **self.pages*

    *    **def __del__(**self**):
            **print**("Book no longer in use")***

*Output*

    *x **= **Book**("Python World"**,**"Prashant"**,100**)
    **My first book*

    *print**(**x**)
    **Name:Python World, author:Prashant, pages:100*

    *print**(**len**(**x**))***

    ***100***

    ***del **x
    Book no longer in use*

*Here __str__,__len__ and __del__ are special method(defined by the use of underscore). They allow us to use Python special functions on objects created through our class*

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
[**100 Days of DevOps — Day 81-Debugging Python Code**
*Welcome to Day 81 of 100 Days of DevOps, Focus for today is Debugging Python Code*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-81-debugging-python-code-a1e19b4011a8)
