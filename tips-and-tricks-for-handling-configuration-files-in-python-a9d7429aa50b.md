Unknown markup type 10 { type: [33m10[39m, start: [33m157[39m, end: [33m169[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m208[39m, end: [33m220[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m4[39m, end: [33m16[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m115[39m, end: [33m118[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m117[39m, end: [33m131[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m208[39m, end: [33m209[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m213[39m, end: [33m214[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m367[39m, end: [33m368[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m372[39m, end: [33m373[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m49[39m, end: [33m61[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m7[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m22[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m177[39m, end: [33m189[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m12[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m4[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m3[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m28[39m, end: [33m36[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m12[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m4[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m26[39m, end: [33m33[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m192[39m, end: [33m199[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m6[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m10[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m8[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m13[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m159[39m, end: [33m162[39m }

# Tips and Tricks for Handling Configuration Files in Python

Utilizing ConfigParser to read and write config files in Python

![Photo by [Mr Cup / Fabien Barral](https://unsplash.com/@iammrcup?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/file?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)](https://cdn-images-1.medium.com/max/8576/1*Ikkiar2teCizr06fDxN7ig.jpeg)*Photo by [Mr Cup / Fabien Barral](https://unsplash.com/@iammrcup?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/file?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)*

By reading this article, you’ll learn the basic steps required to manage your configuration files in Python. This tutorial focuses on a Python module called ConfigParser. Based on the official [documentation](https://docs.python.org/3/library/configparser.html), ConfigParser is a:
> “ … class which implements a basic configuration language which provides a structure similar to what’s found in Microsoft Windows INI files. You can use this to write Python programs which can be customized by end users easily.”

There are four sections in this tutorial:

1. Setup

1. File format

1. Basic API

1. Conclusion

Let’s move on to the next section to install the module.

## 1. Setup

The ConfigParser module is part of the Python 3 library, and you should have it when you install Python 3. You can easily import it using the following code:

    import configparser

If you’re using Python 2.6–3.5, you can try the following import statement:

    from backports import configparser

Other than that, if you’d like to code something that works for both Python 2/3, kindly use the following code:

    try:
        import configparser
    except:
        from six.moves import configparser

## 2. File Format

You can name the configuration based on your own preferences, but it’s highly recommended to keep the extension as ini.

A configuration file consists of one or multiple sections identified based on the opening and closing bracket header [section_name]. Each section will contain key-value entries that are separated by either a = or : sign. I’ll be using the equal sign for this tutorial. Whitespaces in-between the key-value entries are allowed. You can even put in a comment using the # or ; prefix.

Let’s have a look at one example of the configuration file:

    [default]
    host = 192.168.1.1
    port = 22
    username = username
    password = password

    [dev_database]
    port = 22
    forwardx11 = no
    name = db_test

We have two sections in the configuration file, and each section has its own key-value entries. You can create your own configuration file or generate it via code. Let’s move on to the next section to find out more about it.

## 3. Basic API

The first thing we need to do is to initialize a ConfigParser object.

    config = configparser.ConfigParser()

### Write

We can easily create the default section used by initializing the section with a dictionary.

    config['default'] = {
            "host" : "192.168.1.1",
            "port" : "22",
            "username" : "username",
            "password" : "password"
        }

* default — The name of the section

* “host” : “192.168.1.1” — The key-value pairs entry for the config file

Besides, you can initialize it as an empty dict and add the entry line by line later on. This provides us with a lot more flexibility. Let’s have a look at how to do it for the dev_database section.

    config['dev_database'] = {}
        config['dev_database']['port'] = "22"
        config['dev_database']['forwardx11'] = "no"
        config['dev_database']['name'] = "db_test"

* dev_database — Name of the section

* port — Key for an entry

* 22 — Value for an entry

Once you’re done, you can start to write the content to a config file.

    with open('test.ini', 'w') as configfile:
            config.write(configfile)

You should be able to see a test.ini file generated once you run the code.

### Read

Let’s try to read the configuration file you’ve just generated.

    config.read('test.ini')

You can identify the sections that are present in the configuration file using the following API call.

    config.sections()

Next, we’ll try to get the name of the database.

    config['dev_database']['name']

* dev_database — Name of the section

* name — Key for an entry

You should be able to get db_test as an output. What if you wanted to get all the key-value entries of a section? Let’s try it out with the following code to print out all the values into the default section.

    for key in config['default']:
            print(config['default'][key])

### Data type

By default, the module will parse the data internally as a string. You need to convert them either manually or using the getter methods.

    config['default'].getint('port')

There are three getter methods that can be utilized for conversion:

* getint

* getboolean

* getfloat

Let’s try it out in another way:

    config.getboolean('dev_database', 'forwardx11')

Please be noted the second parameter is the key of an entry and not a fallback result.

### Fallback

If you’d like to have a fallback result, you can define it as follows:

    config.get('default', 'database', fallback='prod_database')

prod_database will be returned as the result if the database key isn’t found or the section isn’t found.

## 4. Conclusion

Let’s recap what we’ve learned today. First, we started with the setup by writing a few lines of import code.

Next, we explored in detail the format of the configuration files, which consists of sections and key-value entries. It’s recommended to keep the extension as ini.

Then, we tested out a few basic APIs available in the module. This includes writing the configurations to a file and reading an existing configuration file. We can even convert the data to a particular data type using the getter methods. The fallback parameter can be used to provide a fallback response in case the section or key isn’t found.

Thanks for reading, and I hope to see you again in the next article.

## Reference

1. [https://docs.python.org/3/library/configparser.html](https://docs.python.org/3/library/configparser.html)

1. [https://github.com/jaraco/configparser](https://github.com/jaraco/configparser)
