
# Introduction to object-oriented programming in Python

Duomly — programming online courses

This article was originally published at: [https://www.blog.duomly.com/object-oriented-programming-in-python/](https://www.blog.duomly.com/object-oriented-programming-in-python/)

Python allows writing programs following several programming paradigms (like procedural programming, functional programming, object-oriented programming) and to combine them.

Object-oriented programming is one of the most widely used paradigms today. It is based on the use of *objects* — entities that contain data members called **attributes** and bounded functions (routines, procedures) called **methods**.

Objects are instances of classes. In other words, classes mostly define the structure of objects and serve as templates for creating them. Classes have methods definitions but can also contain data common for all their instances.

This article is about **object-oriented programming in Python**. It explains how to create classes and use them to instantiate their objects. In particular, it covers the following:

* Creating Python classes

* Data attributes

* Instance methods

* Properties

* Class and static methods

* Inheritance

This article doesn’t cover all the details on these topics. There are also many other aspects of object-oriented programming in Python. Hopefully, it can provide a good foundation to start learning and implementing object-oriented programs with Python.

## Creating Python Classes

We define a Python class with the keyword class, followed by the name of the class, semicolon, and the implementation of the class:

    >>> class MyClass:
    ...     pass
    ...

By convention, Python classes are named using ThePascalCase.
Let’s now create an instance of our new class called MyClass:

    >>> a = MyClass()
    >>> a
    <__main__.MyClass object at 0x7f32ef3deb70>

The statement a = MyClass() creates an instance of MyClass and assigns the reference to it to a new variable a.

We can get the type, that is the class of an object with the Python built-in function type() or directly with the attribute .__class__. Once we have the class (type), we can get its name with the attribute .__name__:

    >>> type(a)
    <class '__main__.MyClass'>
    >>> a.__class__
    <class '__main__.MyClass'>
    >>> a.__class__.__name__
    'MyClass'

By the way, let’s mention that Python classes are also objects. They are the instances of the class type:

    >>> type(MyClass)
    <class 'type'>

Let’s now define one method.

Each **instance method** in Python must have the first parameter that corresponds to the instance, that is the object itself. By convention, this parameter is called self. It’s followed with other parameters if any at all. When we call a method, we *don’t* explicitly provide the argument that corresponds to the parameter self.

One of the most important methods we usually define is .__init__(). This method is called after an instance of the class is created. It initializes the class members. Let’s make it look like this:

    >>> class MyClass:
    ...     def __init__(self, arg_1, arg_2, arg_3):
    ...         print(f'an instance of {type(self).__name__} created')
    ...         print(f'arg_1: {arg_1}, arg_2: {arg_2}, arg_3: {arg_3}')
    ...

We’ll create an instance of MyClass to see what’s going to happen. Our .__init__() method expects three arguments (arg_1, arg_2, and arg_3; remember that we don’t pass the first argument that corresponds to self). Thus, we’ll give it three arguments when we instantiate the object:

    >>> a = MyClass(2, 4, 8)
    an instance of MyClass created
    arg_1: 2, arg_2: 4, arg_3: 8

This is what just happened as the consequence of the statement above:

* An instance that is an object of the type MyClass is created.

* The method .__init__() of this instance is invoked automatically.

* The arguments we’ve passed to MyClass() (2, 4, and 8) are passed to .__init__().

* .__init__() executes and prints what we’ve requested. It gets the name of the class with a type(self).__name__.

Now we have one class, its method .__init__(), and one instance of this class.

## Data Attributes

Let’s modify MyClass and make it have some data attributes.

We initialize and define, and also change a data attribute by assigning it a value in .__init__() or any other instance method:

    >>> class MyClass:
    ...     def __init__(self, arg_1, arg_2, arg_3):
    ...         self.x = arg_1
    ...         self._y = arg_2
    ...         self.__z = arg_3
    ...

Now MyClass has three data attributes:

* .x that gets the value of arg_1

* ._y that gets the value of arg_2

* .__z that gets the value of arg_3

This can be written in a more compact form thanks to the Python unpacking mechanism:

    >>> class MyClass:
    ...     def __init__(self, arg_1, arg_2, arg_3):
    ...         self.x, self._y, self.__z = arg_1, arg_2, arg_3
    ...

The purpose of all these underscores (_) in the names of the attributes is to indicate the level of “privacy”:

* The attributes without leading underscores (like .x) can be normally called and modified from outside the object.

* The attributes with a single leading underscore (like ._y) can also be normally called and modified from outside the object. However, the underscore is the conventional sign that the creator of the class strongly advises against such use of the variable. It should be called and modified only via the functional members of the class (like methods and properties).

* The attributes with a double leading underscore (like .__z) will have the name changed (in this case to ._MyClass__z) in the process called **name mangling**. They could also be called and modified from outside the object with the new name. However, there is a strong recommendation against this practice. It should also be called and modified with its original name only via the functional members of the class.

The data attributes of Python objects are usually stored in the dictionary called .__dict__ that’s also the attribute of the object. It’s possible to store data in other places, however. We can get .__dict__ either by calling it directly or with the Python built-in function vars():

    >>> a = MyClass(2, 4, 8)
    >>> vars(a)
    {'x': 2, '_y': 4, '_MyClass__z': 8}
    >>> a.__dict__
    {'x': 2, '_y': 4, '_MyClass__z': 8}

The key ‘_MyClass__z’ is there instead of ‘__z’ because of name mangling.

We can use .__dict__ as any other Python dictionary.

This is the conventional way how to get and change the values associated with the data attributes:

    >>> a.x
    2
    >>> a._y
    4
    >>> a.__z
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    AttributeError: 'MyClass' object has no attribute '__z'
    >>> a.x = 16
    >>> a.x
    16
    >>> vars(a)
    {'x': 16, '_y': 4, '_MyClass__z': 8}

Note that we can’t access a.__z since .__dict__ doesn’t have the key ‘__z’.

## Instance Methods

Now, we’ll create two instance methods:

* .set_z() that modifies .__z

* .get_z() that returns the value of .__z

Remember that the first parameter of each instance method (called self by convention) refers to the object itself, but we don’t provide it when invoking the method:

    >>> class MyClass:
    ...     def __init__(self, arg_1, arg_2, arg_3):
    ...         self.x, self._y, self.__z = arg_1, arg_2, arg_3
    ...     
    ...     def set_z(self, value):
    ...         self.__z = value
    ...     
    ...     def get_z(self):
    ...         return self.__z
    ...
    >>> b = MyClass(2, 4, 8)

The methods .get_z() and .set_z() provide a conventional interface to retrieve and modify the value of .__z:

    >>> b.get_z()
    8
    >>> b.set_z(16)
    >>> vars(b)
    {'x': 2, '_y': 4, '_MyClass__z': 16}

.get_z() and .set_z() can bring additional functionality like checking the validity of data. Such methods enable encapsulation, one of the main concepts in object-oriented programming.

## Properties

The alternative (and perhaps a more Pythonic) way to access and modify the data attributes is using **properties**. They encapsulate the methods — getters, setters, and deleters — but behave like ordinary data attributes.

This is the implementation of the property .z that has the same functionality as .get_z() and .set_z():

    >>> class MyClass:
    ...     def __init__(self, arg_1, arg_2, arg_3):
    ...         self.x, self._y, self.__z = arg_1, arg_2, arg_3
    ...     
    ...     @property
    ...     def z(self):
    ...         return self.__z
    ...     
    ...     @z.setter
    ...     def z(self, value):
    ...         self.__z = value
    ...
    >>> b = MyClass(2, 4, 8)

This is how we can access and modify the data attribute .__z with the corresponding property .z:

    >>> b.z
    8
    >>> b.z = 16
    >>> vars(b)
    {'x': 2, '_y': 4, '_MyClass__z': 16}

This code is shorter and arguably more elegant than in the previous example.

## Class and Static Methods

In addition to instance methods and properties, classes can have **class methods** and **static methods**.

Let’s add three methods to MyClass:

    >>> class MyClass:
    ...     def __init__(self, arg_1, arg_2, arg_3):
    ...         self.x, self._y, self.__z = arg_1, arg_2, arg_3
    ...     
    ...     def f(self, arg):
    ...         print('instance method f called')
    ...         print(f'instance: {self}')
    ...         print(f'instance attributes:\n{vars(self)}')
    ...         print(f'class: {type(self)}')
    ...         print(f'arg: {arg}')
    ...     
    ...     @classmethod
    ...     def g(cls, arg):
    ...         print('class method g called')
    ...         print(f'cls: {cls}')
    ...         print(f'arg: {arg}')
    ...     
    ...     @staticmethod
    ...     def h(arg):
    ...         print('static method h called')
    ...         print(f'arg: {arg}')
    ...
    >>> c = MyClass(2, 4, 8)

The method .f() is an instance method. Instance methods must have the first argument referring to the object itself. They can access the object with self, the data attributes of the object with vars(self) or self.__dict__, the class that corresponds to the object with a type(self) or self.__class__, as well as their own arguments.

The method .g() is decorated with @classmethod. That makes it a class method. Each class method must have the first parameter that refers to the class, called cls by convention. As in the case of instance methods, we don’t explicitly provide the argument that corresponds to cls. Class methods can access the class with cls and own arguments.

The method .h() is decorated with @staticmethod. That makes it a static method. Static methods can access just their own arguments.

This is how instance methods are usually invoked in Python:

    >>> c.f('my-argument')
    instance method f called
    instance: <__main__.MyClass object at 0x7f32ef3def98>
    instance attributes:
    {'x': 2, '_y': 4, '_MyClass__z': 8}
    class: <class '__main__.MyClass'>
    arg: my-argument

Class methods and static methods are usually called directly with the class instead of the instance:

    >>> MyClass.g('my-argument')
    class method g called
    cls: <class '__main__.MyClass'>
    arg: my-argument
    >>> MyClass.h('my-argument')
    static method h called
    arg: my-argument

Remember that we don’t pass the argument that corresponds to the first parameter cls of a class method.

However, class methods and static methods can be called like this:

    >>> c.g('my-argument')
    class method g called
    cls: <class '__main__.MyClass'>
    arg: my-argument
    >>> c.h('my-argument')
    static method h called
    arg: my-argument

When we call c.g or c.h and there aren’t instance members with such names, Python will search for a class and static members.

## Inheritance

Inheritance is another important feature of object-oriented programming. It’s a concept where one class (called the subclass or derived class) obtains, that is *inherits* the data and function members of some other class (called the superclass or base class).

In Python, all classes implicitly inherit the built-in Python class object. However, we can define the inheritance hierarchy of our own classes as suitable.

For example, we’ll create a new class called MyOtherClass that inherits MyClass:

    >>> class MyOtherClass(MyClass):
    ...     def __init__(self, u, v, w, x, y, z):
    ...         super().__init__(x, y, z)
    ...         self.__u, self.__v, self.__w = u, v, w
    ...     
    ...     def f_(self, arg):
    ...         print('instance method f_ called')
    ...         print(f'instance: {self}')
    ...         print(f'instance attributes:\n{vars(self)}')
    ...         print(f'class: {type(self)}')
    ...         print(f'arg: {arg}')
    ...
    >>> d = MyOtherClass(1, 2, 4, 8, 16, 32)

MyOtherClass has the members of MyClass: .x, ._y, .__z, and .f(). The data members of the base class .x, ._y, and .__z are initialized with the statement super().__init__(x, y, z) that invokes the .__init__() method of the base class.

MyOtherClass also has own members: .__u, .__v, .__w, and .f_().

We’ll get the data members with vars():

    >>> vars(d)
    {'x': 8,
     '_y': 16,
     '_MyClass__z': 32,
     '_MyOtherClass__u': 1,
     '_MyOtherClass__v': 2,
     '_MyOtherClass__w': 4}

We can call the methods from both bases and derived classes:

    >>> d.f('some-argument')
    instance method f called
    instance: <__main__.MyOtherClass object at 0x7f32ef3e7048>
    instance attributes:
    {'x': 8,
     '_y': 16,
     '_MyClass__z': 32,
     '_MyOtherClass__u': 1,
     '_MyOtherClass__v': 2,
     '_MyOtherClass__w': 4}
    class: <class '__main__.MyOtherClass'>
    arg: some-argument
    >>> d.f_('some-argument')
    instance method f_ called
    instance: <__main__.MyOtherClass object at 0x7f32ef3e7048>
    instance attributes:
    {'x': 8,
     '_y': 16,
     '_MyClass__z': 32,
     '_MyOtherClass__u': 1,
     '_MyOtherClass__v': 2,
     '_MyOtherClass__w': 4}
    class: <class '__main__.MyOtherClass'>
    arg: some-argument

However, if a derived class contains the member with the same name as its base class, the member of the derived class has precedence.

## Conclusions

Object-oriented programming is one of the programming paradigms offered by Python. It can be very useful to make appropriate abstractions and represent real-world behavior. However, sometimes it can be counter-intuitive and bring unnecessary overhead to the development process.

This article illustrates how to use Python classes and make basic object-oriented programs. There’s much more about classes and object-oriented programming in Python like:

* Methods .__repr__() and .__str__()

* Method .__new__()

* Operators

* Methods .__getattribute__(), .__getattr__(), .__setattr__(), and .__delattr__()

* Generators

* Callability

* Creating sequences

* Descriptors

* Context managers

* Abstract classes and members

* Multiple inheritance

* Use of super()

* Copying

* Pickling

* Slots

* Class decorators

* Data classes and even more

Object-oriented programming is certainly one of the most popular approaches today. It’s worth learning if one wants to be a Python developer. But remember that Python supports other programming paradigms, like procedural and functional, that might be more appropriate in some scenarios.

Happy coding!

![[Duomly — programming online courses](https://www.duomly.com)](https://cdn-images-1.medium.com/max/2000/0*Z9K1_jKzQkWM109O.png)*[Duomly — programming online courses](https://www.duomly.com)*

Thank you for reading.

This article was provided by our teammate Mirko.
