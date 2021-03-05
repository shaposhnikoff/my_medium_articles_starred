
# Different Ways to Handle  JSON in a Linux Shell

Photo by Ferenc Almasi on Unsplash

Who doesn’t love a good JSON blob? That’s the way most of our data comes these days. Whether you’re interacting with an API or gathering some data from a database, you’re probably getting back JSON. Most popular programming languages have built-in ways to manipulate and parse JSON into different objects and formats. However, if you’re working with data at the shell level in Bash, things aren’t always so simple.

By default shells like Bash do not have a standard JSON parser included. You would either have to drop into a programming language interpreter or install a small dedicated utility. If you’re trying to stay within the confines of the shell, then adding a program is arguably the most straightforward option. This way you don’t have to deal with setting up a different language (and its dependencies) or handle moving back and forth between the two.

Most dedicated JSON manipulation programs are also exceptionally fast, have a small footprint and don’t need additional dependencies whatsoever.

In this article, we’ll explore some popular applications that add this crucial functionality to your shell. This will let you handle JSON data in shell scripts and make on-the-fly manipulations without having to switch to a more robust programming language. Let’s take a look.

### [jq](https://stedolan.github.io/jq/)

Without a doubt this is one of the more popular dedicated programs for dealing with JSON. jq is primarily used for filtering and parsing incoming JSON data. This is useful if you’re working with a shell script that is interacting with an API. jq is also small and super fast. Let’s look at how to use a simple jq [filter](https://stedolan.github.io/jq/manual/#Basicfilters) with an API call:

    $ curl --silent [https://randomuser.me/api](https://randomuser.me/api) | jq '.results[0] .name'

If you were to inspect the raw output of the randomuser.me call it would be a blob of unreadable JSON. Kind of like this:

    {"results":[{"gender":"female","name":{"title":"Mrs","first":"Alice","last":"Williams"},"location":{"street":{"number":3138,"name":"Parliament St"},"city":"Elgin","state":"Newfoundland and Labrador","country":"Canada","postcode":"E3P 6W1","coordinates":{"latitude":"-64.8528","longitude":"132.5197"},"timezone":{"offset":"-12:00","description":"Eniwetok, Kwajalein"}},"email":"[alice.williams@example.com](mailto:alice.williams@example.com)","login":{"uuid":"0fa05a45-68c5-458e-8949-a902909c0366","username":"organicmouse652","password":"desert","salt":"FWwxI4cL","md5":"2ac513eba7e262d9a4a337f7612f1ccd","sha1":"3be657547a30a48baa880157870c611471fa5f64","sha256":"b9624ef417a792266c38901d7d60b3333ea0b500bd92765e3a231b5ad72f6559"},"dob":{"date":"1947-01-20T09:49:10.987Z","age":74},"registered":{"date":"2004-09-26T02:43:22.681Z","age":17},"phone":"057-014-3165","cell":"950-360-9030","id":{"name":"","value":null},"picture":{"large":"[https://randomuser.me/api/portraits/women/59.jpg](https://randomuser.me/api/portraits/women/59.jpg)","medium":"[https://randomuser.me/api/portraits/med/women/59.jpg](https://randomuser.me/api/portraits/med/women/59.jpg)","thumbnail":"[https://randomuser.me/api/portraits/thumb/women/59.jpg](https://randomuser.me/api/portraits/thumb/women/59.jpg)"},"nat":"CA"}],"info":{"seed":"a86aaf53eb0eae8c","results":1,"page":1,"version":"1.3"}}

When you pipe the output of that to jq you can immediately start performing simple queries on it. If you simply pipe the output to jq then it will pretty print it to the console, making it much easier to read. But lets assume all we want is a random name back from the API.

The output of the results key is an array with approximately one profile entry in it. In this filter we get the first item of the array and then get the name key from that. This should display something similar to this:

    $ curl --silent [https://randomuser.me/api](https://randomuser.me/api) | jq '.results[0] .name'

    {
      "title": "Miss",
      "first": "Charline",
      "last": "Legrand"
    }

Now we have an easy way to manipulate API data all without ever leaving the shell or shell script.

The functionality of jq doesn’t stop at simple filters like this though. Let’s say we wanted to create a whole new key called fullname that was both the first and last names together? That’s quite easy with jq:

    $ curl --silent [https://randomuser.me/api](https://randomuser.me/api) | \
    jq '.results[0] | {fullname: (.name.first + " " + .name.last)}'

    {
      "fullname": "Charline Legrand"
    }

Just like you can pipe commands together in the shell, you can also pipe jq filters together as well. The list of built-in functions, operators and other extras are extensive. Check out the [manual](https://stedolan.github.io/jq/manual/) for details on how to squeeze the most out of this awesome utility.

### [jo](https://github.com/jpmens/jo)

While jq is awesome for parsing through and manipulating existing JSON data, jo is even better at *creating* JSON data. This handy little program let’s us build JSON data structures in the shell *way* easier than if we tried to by hand ourselves. Building up a JSON string and battling escape characters is no picnic and wastes time.

Let’s take a look at how easy it is to use jo to create some data:

    $ jo name=bob creation_date="$(date +%m-%d-%y)"

    {"name":"bob","creation_date":"02-23-21"}

Rather than build up the string manually and handle placing each curly brace and quote we simply declare each item like a normal variable. One of the really nice things about jo is that you can assign the output of sub-shells just like you would with any other variable. If we tried to handle escaping the arguments to something like date it could quickly become a cumbersome endeavor.

We can also pretty print larger blobs of data using the -p option with jo. Let’s see what this looks like when we create an array of data:

    $ jo -p arr=$(jo -a one two three four five six)

    {
       "arr": [
          "one",
          "two",
          "three",
          "four",
          "five",
          "six"
       ]
    }

Read more about the creation of jo and some of the really cool things it does with JSON below:

[https://jpmens.net/2016/03/05/a-shell-command-to-create-json-jo/](https://jpmens.net/2016/03/05/a-shell-command-to-create-json-jo/)

### [json_pp](https://github.com/deftek/json_pp)

This is a simple utility that let’s you display larger JSON blobs in a more readable format and convert between different formats too. Just like any other “pretty print” utility, json_pp will format JSON neatly with the proper spacing and indentation so you can track down the information you want quicker.

    $ curl --silent [https://randomuser.me/api](https://randomuser.me/api) | json_pp

    {
       "info" : {
          "page" : 1,
          "results" : 1,
          "seed" : "83b8f82090d14bfb",
          "version" : "1.3"
       },
       "results" : [
          {
             "cell" : "076 515 26 35",
             "dob" : {
                "age" : 28,
                "date" : "1993-03-05T14:37:55.246Z"
             },
             "email" : "[annelise.durand@example.com](mailto:annelise.durand@example.com)",
    ...

This utility lacks the same robust set of features as something like jq or jo but if you’re in a pinch and this is the only thing available it will at least allow you to produce more readable JSON.

### [jshon](http://kmkeen.com/jshon/)

Another extremely lightweight JSON parser available is jshon. This program has functionality similar to jq and has been around for quite some time. There are similar filters to jq and it also has handy built-in functions too. Some of the core functions are great for quickly sifting through large amounts of data. Let’s look at how we would grab all the key names from an object:

    $ echo '{"key1":"value1","key2":"value2","key3":"value3"}' | jshon -k

    key1
    key2
    key3

You can also obtain the keys using jq, but the result is an actual array object, like this:

    $ echo '{"key1":"value1","key2":"value2","key3":"value3"}' | jq keys

    [
      "key1",
      "key2",
      "key3"
    ]

If you’re looking for raw key output you can just use jshon. On top of this, jshon is fast, *really* fast. Benchmarking the keys function of jq against jshon -k shows jshon being over *7 times* faster:

    $ time echo '{"key1":"value1","key2":"value2","key3":"value3"}' | jshon -k

    key1
    key2
    key3

    real 0m0.005s
    user 0m0.002s
    sys 0m0.004s

    $ time echo '{"key1":"value1","key2":"value2","key3":"value3"}' | jq keys

    [
      "key1",
      "key2",
      "key3"
    ]

    real 0m0.039s
    user 0m0.035s
    sys 0m0.005s

*Thanks for reading! Parsing JSON in shell scripts or right in the console doesn’t have to be clunky when you leverage these awesome utilities. Check out [6 Terminal Commands You Should Know](https://betterprogramming.pub/6-terminal-commands-you-should-know-8e9767bdfec) to keep expanding your shell expertise even further.*
