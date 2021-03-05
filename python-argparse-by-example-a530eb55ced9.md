
# Python Argparse by Example

Build flexible scripts with command line arguments

The argparse module is part of the Python standard library, and lets your code accept command line arguments. This makes your code easy to configure at run-time. There are [multiple ways](https://stackabuse.com/command-line-arguments-in-python/) to do this in Python, but argparse is the most powerful with minimal additional code required. This article is a collection of example code snippets — I wrote it because the [official documentation](https://docs.python.org/3/howto/argparse.html) is quite heavy-going, and couldn’t find a suitable quick reference.

![Simpsons — 20th Century Fox](https://cdn-images-1.medium.com/max/2000/1*Q-4f272V9CJG5Af0VmCVUg.png)*Simpsons — 20th Century Fox*

### Basic usage

There are four basic steps to using argparse in your code:

1. import the argparse module;

1. create the parser;

1. add arguments;

1. parse the arguments.

<iframe src="https://medium.com/media/407fe7606b68dfd145b728361f9dbc1e" frameborder=0></iframe>

This adds a single positional, mandatory (required) argument. If we run our script but forget to add the argument, we will get an error. If we supply an argument, it gets printed to the console:

    $ python argparse_basic.py
    usage: argparse_basic.py [-h] argument1
    argparse_basic.py: error: the following arguments are required: argument1
    

    $ python argparse_basic.py "Hello, world!"
    Hello, world!
    

    $ python argparse_basic.py 3.14159
    3.14159

We can also run the script and print the help message, which will list the arguments and any constraints:

    $ python basic.py --help
    usage: basic.py [-h] argument1

    positional arguments:
      argument1   This is the first argument

    optional arguments:
      -h, --help  show this help message and exit

The following sections expand on the code snippet above — they all use a similar structure, although we don’t reproduce all of the code unless it adds value.

### Positional arguments

These are read from the command line in the order that they appear after the script name, and are required unless otherwise specified:

<iframe src="https://medium.com/media/51bd037a45e3aad3c7c7da6475e5315f" frameborder=0></iframe>

### Optional (flag) arguments

Flag arguments are specified on the command line by name, using single and/or double dashes (e.g. -h and/or --help) and are optional unless otherwise specified. They can be added on the command line in any order:

<iframe src="https://medium.com/media/e88b77c5b39296f3bbf29d23ea0a78b6" frameborder=0></iframe>

### Mutually exclusive arguments

Some arguments don’t make sense to be both set at the same time. The mutually exclusive group allows you to specify this: an exception is raised if more than one argument in the group is provided:

<iframe src="https://medium.com/media/ada71efba0306d68592ef1eec0f95246" frameborder=0></iframe>

arg1 is True if the flag is included on the command line, otherwise False. arg2 is the opposite.

## Advanced functionality

The examples above should cover more than 90% of the scenarios that you are are likely to use argparse for. It has some rather powerful additional features, however, and in the following sections we’ll see how it can be used to perform more specific input validation as well as managing input and output streams.

### Parsing custom data

As well as using the predefined types such as str, int, float etc, the type= parameter can accept any callable that takes a single string argument and returns a value. This can be used to do custom validation or conversion of the input:

<iframe src="https://medium.com/media/6cb477d18d1137d22e87f12b1a9645e0" frameborder=0></iframe>

In the example above, an exception is raised if the input contains one or more space characters:

    $ python argparse_custom_type.py abc123
    abc123
    

    $ python argparse_custom_type.py "Hello, world!"
    usage: argparse_custom_type.py [-h] argument1
    error: argument1: "Hello, world!" is not a single word

### Opening and closing files

It is relatively common to use command line arguments to specify paths to input and output files, such as for source data and results summaries. argparse can handle that for you: FileType gives you a more flexible way of specifying that an argument should be a file, and can handle encoding, access mode (read/write/append etc) and other hyperparameters:

<iframe src="https://medium.com/media/98baab3b28503100c611e8cb66a76d56" frameborder=0></iframe>

When using the FileType type, argparse takes care of opening the file(s) for you. If the input file name is not found, it throws an error that references the associated argument; if the optional output file argument is supplied, argparse creates the output stream:

    $ python argparse_FileType.py source_dta.csv
    usage: argparse_FileType.py [-h] [--output OUTPUT] infile
    argparse_FileType.py.py: error: argument infile: can't open 'source_dta.csv': [Errno 2] No such file or directory: 'source_dta.csv'
    

    $ python basic.py source_dta.csv --output sum.csv

What are your thoughts? Feel free to leave your comments or any argparse code snippets you use below!
> [*Rupert Thomas](https://twitter.com/rupertthomas) is a technology consultant specialising in machine learning, machine vision, and data-driven products. [Rupert Thomas](undefined)*

### References

[Python official documentation — argparse](https://docs.python.org/3/library/argparse.html)

[Python official argparse tutorial](https://docs.python.org/3/howto/argparse.html)
