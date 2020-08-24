Unknown markup type 10 { type: [33m10[39m, start: [33m122[39m, end: [33m128[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m11[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m4[39m, end: [33m13[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m111[39m, end: [33m118[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m202[39m, end: [33m208[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m79[39m, end: [33m85[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m157[39m, end: [33m175[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m193[39m, end: [33m199[39m }

# 10 Python Tricks and Scripts for Strings Transformation and Decomposing

Parse strings like a true Pythonista

![Photo by [Peter Neumann](https://unsplash.com/@peterneumann?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/10944/0*A0i0Il4bBbbpyHF7)*Photo by [Peter Neumann](https://unsplash.com/@peterneumann?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

My last piece about [Python idioms acceptable for data structures and containers](https://medium.com/better-programming/10-tiny-python-idioms-for-collections-and-data-structures-2f0d2923832) was very successful, so I decided to prepare a compilation of my scripts for strings parsing and tokenization.

Nobody can deny the importance of text analyses and strings parsing. It is applied in almost every software development direction, from URL parsing to natural language analysis. I won’t cover all possible applications of this method — that’s far beyond the scope of one piece. But I will introduce some basic techniques to work with strings and tokens within Python programs (since it’s one of my favorite languages).

These little scripts should be treated as building blocks for greater text analysis and data preprocessing applications. Knowledge of the basics is highly important for further development.

OK, let’s go to the list!

### 1. Translate and Replace

The first case is to replace or delete some symbols or some substring from the text. Python has built-in functions in the string module which perform the desired tasks.

translate() uses symbols map to delete or change specific symbols:

<iframe src="https://medium.com/media/e88750f67e3951f4bae562fe81038b33" frameborder=0></iframe>

And replace() works as its name implies — by changing a substring to the desired one:

<iframe src="https://medium.com/media/50c1742a2622375b1d9e98f5cca3da78" frameborder=0></iframe>

### 2. String sanitizing

Now we can apply the knowledge from the previous statement for string sanitizing. This is one of the most demanded processes in a data science project during data cleaning — the great example is raw text with whitespace symbols and line breaks. Here’s a simple script to clean such a string:

<iframe src="https://medium.com/media/46c1c8243edf0860f8da67b85fc76d63" frameborder=0></iframe>

### 3. Split in a way you want

Text analysis demands different metrics, like word count, symbol count, average sentence length. To calculate these values we need our text to be prepared — cleaned and split. Luckily for us, Python has several built-in functions for the text division:

* The default split by the whitespace symbol:

    test_string.split()

    Out[1]: ['The', 'quick', 'brown', 'fox', 'jumps', 'over', 'the', 'lazy', 'dog']

* Split by the determined number of tokens:

    test_string.split(' ', 2)

    Out[2]: ['The', 'quick', 'brown fox jumps over the lazy dog']

* Reversed split for the string:

    test_string.rsplit(' ', 2)

    Out[3]: ['The quick brown fox jumps over the', 'lazy', 'dog']

* Split by a custom symbol (of course, all functions support this feature):

    test_string.split('e')

    Out[4]: ['Th', ' quick brown fox jumps ov', 'r the lazy dog']

* Split the string into the desired token, with tokens before and after it:

    test_string.partition('fox')

    Out[5]: ('The quick brown ', 'fox', ' jumps over the lazy dog')

### 4. Strip and fill

Another important feature is the ability to delete extra leading and trailing symbols from the string. We have strip() functions family for this purpose:

* Cut spaces by default.

* Cut spaces from the left or the right.

* Cut custom symbols.

<iframe src="https://medium.com/media/7a597c60dba63986d3977f04ace12a3a" frameborder=0></iframe>

Additionally, there’s a useful feature for padding numbers with leading zeros:

<iframe src="https://medium.com/media/579b0b068a75e34a4a082647078d7493" frameborder=0></iframe>

### 5. Deconstruct and recreate

Text generation requires the construction of sentences and phrases from the dictionary of words. As a process, this is the reverse of string splitting. Python allows us to use a built-in string method, join(), to join words back into the sentence:

<iframe src="https://medium.com/media/0deef26767c1d72de43f3f096fabaecf" frameborder=0></iframe>

### 6. Remove punctuation

This is another use case in the set of instruments for text cleaning. Python’s string module [has a lot of built-in constants](https://docs.python.org/3/library/string.html) with a separate set of symbols. string.punctuation is one of them, so we will use it for the string sanitizing.

<iframe src="https://medium.com/media/aa333bd303c622c6c97fd29a97822ab7" frameborder=0></iframe>

### 7. Work with cases

Text formatting is every data scientist’s pain. Words that don’t have consecutive cases and sentences in different formats create a lot of trouble when data cleaning. However, there are other Python functions for these tasks:

<iframe src="https://medium.com/media/9c72f4167d236ac00dbadebead40b71d" frameborder=0></iframe>

### 8. The world of regex

Sometimes text cleaning is not easy to provide by determined symbols or phrases. We need to use some patterns instead. It is the perfect field for regular expression usage and [the appropriate Python module](https://docs.python.org/3/library/re.html).

We won’t discuss all the power of the regex, but we will focus on its applications instead — data splitting and replacing, for example. Yes, these tasks are described above, but here’s the more powerful alternative.

Split by pattern:

<iframe src="https://medium.com/media/ca98a9cae865ee82437dfb3c4c6ea196" frameborder=0></iframe>

Replace by pattern:

<iframe src="https://medium.com/media/f6e0fdbd3a36fddd384af45775ceb5c0" frameborder=0></iframe>

### 9. Tokenize the string

It is time to gather all the tricks we have learned earlier and apply them for the real tokenization. However, we won’t repeat all the code. Here’s an example of a pretty cool alternative with pandas usage. In our example, we should clean the string from the extra symbols, convert to a single case and split it into tokens.

<iframe src="https://medium.com/media/9141b2c988aa67867246b46d925c3319" frameborder=0></iframe>

### 10. (Bonus) Find the substring

Before any cleaning task, we have to determine whether we really need the cleaning. In most cases, the question narrows to the search of some symbol or phrase within the text. Python provides a lot of functions for our purposes.

Trailing substring:

    test_string.endswith('dog')

    Out[1]: True

Leading substring:

    test_string.startswith('dog')

    Out[2]: False

General check:

    'fox' **in** test_string

    Out[3]: True

Get the index of the substring:

    test_string.find('fox')

    Out[4]: 16

Of course, any task can be solved in a variety of ways, especially if we’re talking about Python. However, I think my vision of strings parsing will be useful for you.

You can find the Jupyter notebook with working code on my Github:
[**Midvel/medium_jupyter_notes**
*Permalink Dismiss GitHub is home to over 40 million developers working together to host and review code, manage…*github.com](https://github.com/Midvel/medium_jupyter_notes/blob/master/string_manip/string-tokenization-parsing.ipynb)

Also, make sure that you haven’t missed my previous article about Python idioms:
[**10 Tiny Python Idioms for Collections and Data Structures**
*Work with containers in the most Pythonic way*medium.com](https://medium.com/better-programming/10-tiny-python-idioms-for-collections-and-data-structures-2f0d2923832)

Please share your favorite Python techniques!
