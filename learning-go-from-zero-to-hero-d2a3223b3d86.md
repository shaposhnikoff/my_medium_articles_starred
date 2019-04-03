
# Learning Go — from zero to hero

Pic: Gopher mascot and old logo

Let’s start with a small introduction to Go (or Golang). Go was designed by Google engineers Robert Griesemer, Rob Pike, and Ken Thompson. It is a statically typed, compiled language. The first version was released as open source in March 2012.
> # “Go is an open source programming language that makes it easy to build simple, reliable, and efficient software”.
> # — GoLang

In many languages, there are many ways to solve a given problem. Programmers can spend a lot of time thinking about the best way to solve it.

Go, on the other hand, believes in fewer features — with only one right way to solve the problem.

This saves developers time and makes the large codebase easy to maintain. There are no “expressive” features like maps and filters in Go.
> # “When you have features that add expressiveness it typically adds expense”
> # — Rob Pike

![Recently published new logo of go lang: [https://blog.golang.org/go-brand](https://blog.golang.org/go-brand)](https://cdn-images-1.medium.com/max/2124/1*AUiSG5Gqz8MzaGCvGpckGA.png)*Recently published new logo of go lang: [https://blog.golang.org/go-brand](https://blog.golang.org/go-brand)*

## Getting Started

Go is made of packages. The package main tells the Go compiler that the program is compiled as an executable, rather than a shared library. It is the entry point for an application. The main package is defined as:

<iframe src="https://medium.com/media/6a035572d880679c50d4cc0dff20ec2b" frameborder=0></iframe>

Let’s move ahead by writing a simple hello world example by creating a file main.go in the Go workspace.

### **Workspace**

A workspace in Go is defined by the environment variable** **GOPATH.

Any code you write is to be written inside the workspace. Go will search for any packages inside the GOPATH directory, or the GOROOT directory, which is set by default when installing Go. GOROOT** **is the path where the go is installed.

Set GOPATH to your desired directory. For now, let’s add it inside a folder ~/workspace.

    # export env
    export GOPATH=~/workspace

    # go inside the workspace directory
    cd ~/workspace

Create the file main.go with the following code inside the workspace folder we just created.

### Hello World!

<iframe src="https://medium.com/media/b9e8f2df7de66f01956682384c2c9585" frameborder=0></iframe>

In the above example, fmt is a built-in package in Go which implements functions for formatting I/O.

We import a package in Go by using the import** **keyword. func main is the main entry point where the code gets executed.** **Println is a function inside the package fmt which prints “hello world” for us.

Let’s see by running this file. There are two ways we can run a Go command. As we know, Go is a compiled language, so we first need to compile it before executing.

    > go build main.go

This creates a binary executable file main which now we can run:

    > ./main
     # Hello World!

There is another, simpler, way to run the program. The go run command helps abstract the compilation step. You can simply run the following command to execute the program.

    go run main.go
     # Hello World!

***Note**: To try out the code that is mentioned in this blog you can use [https://play.golang.org](https://play.golang.org/)*

## Variables

Variables in Go are declared explicitly. Go is a statically typed language. This means that the variable type is checked at the time of variable declaration. A variable can be declared as:

<iframe src="https://medium.com/media/0455c4e3517e72d33d553f2887c55879" frameborder=0></iframe>

In this case, the value will be set as 0. Use the following syntax to declare and initialize a variable with a different value:

<iframe src="https://medium.com/media/11ec1c1bf2b163fd84e6f090aa5263a7" frameborder=0></iframe>

Here the variable is automatically assigned as an int. We can use a shorthand definition for the variable declaration as:

<iframe src="https://medium.com/media/f61ce9e4dd9c75c59141614ca5934b1b" frameborder=0></iframe>

We can also declare multiple variables in the same line:

<iframe src="https://medium.com/media/78cff73523690c604c045dbc7d4bc7dc" frameborder=0></iframe>

## Data types

Like any other programming language, Go supports various different data structures. Let’s explore some of them:

### **Number, String, and Boolean**

Some of the supported** **number store types are int, int8, int16, int32, int64,
uint, uint8, uint16, uint32, uint64, uintptr…

The string type stores a sequence of bytes. It is represented and declared with keyword string.

A** **boolean** **value is stored using the keyword bool.

Go also supports complex number type data types, which can be declared with complex64 and complex128.

<iframe src="https://medium.com/media/38ddaaa54246759eb05f32736c3c60d6" frameborder=0></iframe>

### **Arrays, Slices, and Maps**

An array is a sequence of elements of the same data type. Arrays have a fixed length defined at declaration, so it cannot be expanded more than that. An array is declared as:

<iframe src="https://medium.com/media/34836fa0145a955aa9809b02eef29039" frameborder=0></iframe>

Arrays can also be multidimensional. We can simply create them with the following format:

<iframe src="https://medium.com/media/5687de6ee75d1eda6337da7699c2ca50" frameborder=0></iframe>

Arrays are limiting for cases when the values of array changes in runtime. Arrays also do not provide the ability to get a subarray. For this, Go has a data type called slices.

Slices store a sequence of elements and can be expanded at any time. Slice declaration is similar to array declaration — without the capacity defined:

<iframe src="https://medium.com/media/9453bf62c0d0476ad93920853d0dde57" frameborder=0></iframe>

This creates a slice with zero capacity and zero length. Slices can also be defined with capacity and length. We can use the following syntax for it:

<iframe src="https://medium.com/media/99d9612b824d837db8fabeba1a86b94f" frameborder=0></iframe>

Here, the slice has an initial length of 5 and has a capacity of 10.

Slices are an abstraction to an array. Slices use an array as an underlying structure. A slice contains three components: capacity, length, and a pointer to the underlying array as shown in the diagram below:

![image src: [https://blog.golang.org/go-slices-usage-and-internals](https://blog.golang.org/go-slices-usage-and-internals)](https://cdn-images-1.medium.com/max/2000/1*P0lNCO0sQwIYHLEX_mfSOQ.png)*image src: [https://blog.golang.org/go-slices-usage-and-internals](https://blog.golang.org/go-slices-usage-and-internals)*

The capacity of a slice can be increased by using the append or a copy function. An append function adds value to the end of the array and also increases the capacity if needed.

<iframe src="https://medium.com/media/fe86b8a1e28bef91ddfce115ebd1b794" frameborder=0></iframe>

Another way to increase the capacity of a slice is to use the copy** **function. Simply create another slice with a larger capacity and copy the original slice to the newly created slice:

<iframe src="https://medium.com/media/6f623df38dcc025ab60d749895a19212" frameborder=0></iframe>

We can create a sub-slice of a slice. This can be done simply using the following command:

<iframe src="https://medium.com/media/567bcbf0241699355cb17b317ded5e92" frameborder=0></iframe>

Maps are a data type in Go, which maps keys to values. We can define a map using the following command:

<iframe src="https://medium.com/media/72adaf1313b01eda7072d43af8a8cb9d" frameborder=0></iframe>

Here m is the new map variable, which has its keys as string and values are integers. We can add keys and values easily to a map:

<iframe src="https://medium.com/media/b667113c477f62370bee4b2e520234a0" frameborder=0></iframe>

## **Typecasting**

One type of data type can be converted into another using type casting. Let’s see a simple type conversion:

<iframe src="https://medium.com/media/6791cecd87f510aa7762b96f4e279548" frameborder=0></iframe>

Not all types of data type can be converted to another type. Make sure that the data type is compatible with the conversion.

## Conditional Statements

### if else

For conditional statements, we can use if-else statements as shown in the example below. Make sure that the curly braces are in the same line as the condition is.

<iframe src="https://medium.com/media/abae1bfaa845f4608c07a115be57056b" frameborder=0></iframe>

### switch case

Switch cases helps organize multiple condition statements. The following example shows a simple switch case statement:

<iframe src="https://medium.com/media/06a8474af2d7a17f443f06d2ac181777" frameborder=0></iframe>

## Looping

Go has a single keyword for the loop. A sngle for loop command help achieve different kinds of loops:

<iframe src="https://medium.com/media/9360b4ed0e2b29de6f4f0cc3faff3064" frameborder=0></iframe>

The above example is similar to a while loop in C. The same for statement can be used for a normal for loop

<iframe src="https://medium.com/media/5e06d623daecdf385a5fd2b68d44ce9e" frameborder=0></iframe>

Infinite loop in Go:

<iframe src="https://medium.com/media/1a5c57ce2a669b8c81b0f9b3323cf735" frameborder=0></iframe>

## Pointers

Go provides pointers. Pointers are the place to hold the address of a value. A pointer is defined by *. A pointer is defined according to the type of data. Example:

<iframe src="https://medium.com/media/f5db72d979c99365787260b5b24ac6b4" frameborder=0></iframe>

Here ap is the pointer to an integer type. The & operator can be used to get the address of a variable.

<iframe src="https://medium.com/media/38baa83cc18dec82f02123311919eaeb" frameborder=0></iframe>

The value pointed by the pointer can be accessed using the * operator:

<iframe src="https://medium.com/media/1be98d5ed6b798a0832883a43775130b" frameborder=0></iframe>

Pointers are usually preferred while passing a struct as an argument or while declaring a method for a defined type.

1. While passing value the value is actually copied which means more memory

1. With the pointer passed, the value changed by the function is reflected back in the method/function caller.

Example:

<iframe src="https://medium.com/media/52ee195b93ef2339edb9a5ff5cfa52ba" frameborder=0></iframe>

Note: While you are trying out the example code in the blog, do not forget to include it with package main and import fmt or other packages when needed as shown in the first main.go example above.

## Functions

The main function defined in the main package is the entry point for a go program to execute. More functions can be defined and used. Let’s look into a simple example:

<iframe src="https://medium.com/media/b2ad3658b2669853156a2b2c000ddf23" frameborder=0></iframe>

As we can see in the above example, a Go function is defined using the **func **keyword followed by the function name. The **arguments** a function takes needs to be defined according to its data type, and finally the data type of the return.

The return of a function can be predefined in function as well:

<iframe src="https://medium.com/media/89ac5e4471ce4dd10f2dc0fd2d00e9f7" frameborder=0></iframe>

Here c is defined as the return variable. So the variable c defined would be automatically returned without needing to be defined at the return statement at the end.

You can also return multiple return values from a single function separating return values with a comma.

<iframe src="https://medium.com/media/76c6fa32e7ea9a95131d2bc279a9f365" frameborder=0></iframe>

## Method, Structs, and Interfaces

Go is not a completely object-oriented language, but with structs, interfaces, and methods it has a lot of object-oriented support and feel.

### Struct

A struct is a typed, collection of different fields. A struct is used to group data together. For example, if we want to group data of a Person type, we define a person’s attribute which could include name, age, gender. A struct can be defined using the following syntax:

<iframe src="https://medium.com/media/6facb196ba25003d91760921b2b7c5fb" frameborder=0></iframe>

With a person type struct defined, now let’s create a person:

<iframe src="https://medium.com/media/fcf369ce0ca1d03abde3921381f51fd3" frameborder=0></iframe>

We can easily access these data with a dot(.)

<iframe src="https://medium.com/media/db8110e678e6e64b36e812184d070459" frameborder=0></iframe>

You can also access attributes of a struct directly with its pointer:

<iframe src="https://medium.com/media/9c7728396a13a5718697b4637e07517b" frameborder=0></iframe>

### Methods

Methods are a special type of function with a *receiver. *A receiver can be both a value or a pointer. Let’s create a method called describe which has a receiver type person we created in the above example:

<iframe src="https://medium.com/media/46df7df209d24c6c02aed032d989b632" frameborder=0></iframe>

As we can see in the above example, the method now can be called using a dot operator as pp.describe. Note that the receiver is a pointer. With the pointer we are passing a reference to the value, so if we make any changes in the method it will be reflected in the receiver pp. It also does not create a new copy of the object, which saves memory.

Note that in the above example the value of age is changed, whereas the value of name is not changed because the method setName is of the receiver type whereas setAge is of type pointer.

### Interfaces

Go interfaces are a collection of methods. Interfaces help group together the properties of a type. Let’s take the example of an interface animal:

<iframe src="https://medium.com/media/c976d5547ccc0bfeb3cd7230804adea2" frameborder=0></iframe>

Here animal is an interface type. Now let’s create 2 different type of animals which implement the animal interface type:

<iframe src="https://medium.com/media/3a93ba3ecf9570532cf63a03361bf5cf" frameborder=0></iframe>

In the main function, we create a variable a of type animal. We assign a snake and a cat type to the animal and use Println to print a.description. Since we have implemented the method describe in both of the types (cat and snake) in a different way we get the description of the animal printed.

## Packages

We write all code in Go in a package. The **main **package is the entry point for the program execution. There are lots of built-in packages in Go. The most famous one we have been using is the **fmt **package.
> # “Go packages in the main mechanism for programming in the large that go provides and they make possible to divvy up a large project into smaller pieces.”
> # — Robert Griesemer

### Installing a package

    go get <package-url-github>
    // example
    go get [github.com/satori/go.uuid](https://github.com/satori/go.uuid)

The packages we installed are saved inside the GOPATH env which is our work directory. You can see the packages by going inside the pkg folder inside our work directory cd $GOPATH/pkg.

### Creating a custom package

Let’s start by creating a folder custom_package:

    > mkdir custom_package
    > cd custom_package

To create a custom package we need to first create a folder with the package name we need. Let’s say we are building a package person. For that let’s create a folder named person inside custom_package folder:

    > mkdir person
    > cd person

Now let’s create a file person.go inside this folder.

<iframe src="https://medium.com/media/9d07edf81faa78d4d83b8e1ef70ca20c" frameborder=0></iframe>

We now need to install the package so that it can be imported and used. So let’s install it:

    > go install

Now let’s go back to the custom_package folder and create a main.go file

<iframe src="https://medium.com/media/19277beaee3160edbbb215e402445727" frameborder=0></iframe>

Here we can now import the package person we created and use the function Description. Note that the function secretName we created in the package will not be accessible. In Go, the method name starting without a capital letter will be private.

### **Packages Documentation**

Go has built-in support for documentation for packages. Run the following command to generate documentation:

    godoc person Description

This will generate documentation for the Description function inside our package person. To see the documentation run a web server using the following command:

    godoc -http=":8080"

Now go to the URL [http://localhost:8080/pkg/](http://localhost:6060/pkg/) and see the documentation of the package we just created.

### Some built-in packages in Go

**fmt**

The package implements formatted I/O functions. We have already used the package for printing out to stdout.

**json**

Another useful package in Go is the json package. This helps to encode/decode the JSON. Let’s take an example to encode/decode some json:

Encode

<iframe src="https://medium.com/media/9b11a050eb626fcf911882136fd841be" frameborder=0></iframe>

Decode

<iframe src="https://medium.com/media/1ae49895d765b7b5c55558717593f6e8" frameborder=0></iframe>

While decoding the json byte using unmarshal, the first argument is the json byte and the second argument is the address of the response type struct where we want the json to be mapped to. Note that the json:”page” maps page key to PageNumber key in the struct.

## Error Handling

Errors are the undesired and unexpected result of a program. Let’s say we are making an API call to an external service. This API call may be successful or could fail. An error in a Go program can be recognized when an error type is present. Let’s see the example:

<iframe src="https://medium.com/media/97e03fba6837f709aafa65408305f552" frameborder=0></iframe>

Here the API call to the error object may pass or could fail. We can check if the error is nil or present and handle the response accordingly:

<iframe src="https://medium.com/media/63e7767946e9bf2704531e5c135286c8" frameborder=0></iframe>

### Returning custom error from a function

When we are writing a function of our own, there are cases when we have errors. These errors can be returned with the help of the error object:

<iframe src="https://medium.com/media/675fb3f47622a3757d88a3370c1cc752" frameborder=0></iframe>

Most of the packages that are built in Go, or external packages we use, have a mechanism for error handling. So any function we call could have possible errors. These errors should never be ignored and always handled gracefully in the place we call these functions, as we have done in the above example.

### Panic

Panic is something that is unhandled and is suddenly encountered during a program execution. In Go, panic is not the ideal way to handle exceptions in a program. It is recommended to use an error object instead. When a panic occurs, the program execution get’s halted. The thing that gets executed after a panic is the defer.

### Defer

Defer is something that will always get executed at the end of a function.

<iframe src="https://medium.com/media/608bdf6c791a36bbdeeb7496cc3f6969" frameborder=0></iframe>

In the above example, we panic the execution of the program using panic(). As you notice, there is a defer statement which will make the program execute the line at the end of the execution of the program. Defer can also be used when we need something to be executed at the end of the function, for example closing a file.

## Concurrency

Go is built with concurrency in mind. Concurrency in Go can be achieved by Go routines which are lightweight threads.

**Go routine**

Go routines are the function which can run in parallel or concurrently with another function. Creating a Go routine is very simple. Simply by adding a keyword Go in front of a function, we can make it execute in parallel. Go routines are very lightweight, so we can create thousands of them. Let’s look into a simple example:

<iframe src="https://medium.com/media/8f5a9bb3f9872f9ffaa6cfdc006c1e54" frameborder=0></iframe>

As you can see in the above example, the function c is a Go routine which executes in parallel with the main Go thread. There are times we want to share resources between multiple threads. Go prefers not sharing the variables of one thread with another because this adds a chance of deadlock and resource waiting. There is another way to share resources between Go routines: via go channels.

**Channels**

We can pass data between two Go routines using channels. While creating a channel it is necessary to specify what kind of data the channel receives. Let’s create a simple channel with string type as follows:

<iframe src="https://medium.com/media/0459c92c12ea8ff03cb42627d284223b" frameborder=0></iframe>

With this channel, we can send string type data. We can both send and receive data in this channel:

<iframe src="https://medium.com/media/f5430efc99af069b8341011cc0bc3875" frameborder=0></iframe>

The receiver Channels wait until the sender sends data to the channel.

**One way channel**

There are cases where we want a Go routine to receive data via the channel but not send data, and also vice versa. For this, we can also create a **one-way channel**. Let’s look into a simple example:

<iframe src="https://medium.com/media/18861e21557898fe580c37ec2649c29d" frameborder=0></iframe>

In the above example, sc is a Go routine which can only send messages to the channel but cannot receive messages.

**Organizing multiple channels for a Go routine using select**

There may be multiple channels that a function is waiting on. For this, we can use a select statement. Let us take a look at an example for more clarity:

<iframe src="https://medium.com/media/5fd5dce1fd7ffd5e88c4e20645376f53" frameborder=0></iframe>

In the above example, the main is waiting on two channels, c1 and c2. With select case statement the main function prints, the message sends from the channel whichever it receives first.

**Buffered channel**

There are cases when we need to send multiple data to a channel. You can create a buffered channel for this. With a buffered channel, the receiver will not get the message until the buffer is full. Let’s take a look at the example:

<iframe src="https://medium.com/media/e4e974de0c4316e7c0228c4dd079e7bb" frameborder=0></iframe>

### Why is Golang Successful?
> # Simplicity… — Rob-pike

## Great!

We learned some of the major components and features of Go.

1. Variables, Datatypes

1. Array slices and maps

1. Functions

1. Looping and conditional statements

1. Pointers

1. Packages

1. Method, Structs, and Interfaces

1. Error Handling

1. Concurrency — Go routines and channels

Congratulations, you now have a decent understanding of Go.
> # One of my most productive days was throwing away 1,000 lines of code.
> # — Ken Thompson

Do not stop here. Keep moving forward. Think about a small application and start building.

[LinkedIn](https://www.linkedin.com/in/milap-neupane-99a4b565/), [Github](http://github.com/milap-neupane), [Twitter](https://twitter.com/_milap)
