Unknown markup type 10 { type: [33m10[39m, start: [33m126[39m, end: [33m129[39m }

# Transform Data in Pure Python

No external libraries required

![Photo by [Chris Ried](https://unsplash.com/@cdr6934?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/12032/0*Y-yj3iI9UQHfUMur)*Photo by [Chris Ried](https://unsplash.com/@cdr6934?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

Python is often selected by developers to write data transformations. With its vast ecosystem of third-party packages, functionality can quickly be integrated.

In working with big data, libraries like [pandas](https://pandas.pydata.org/), [PySpark](https://spark.apache.org/docs/latest/api/python/index.html), [NumPy](https://numpy.org/), [TensorFlow](https://www.tensorflow.org/), and [PyTorch](https://pytorch.org/) allow you to quickly manipulate and transform data.

But what if you have small data, a couple of hundred records or even a thousand. Do you need to use external libraries?

Not all the time, Python has simple built-in methods to help with transforming data.

## List Comprehensions

List comprehensions are a nice built-in feature with a concise syntax for transforming data.

Without list comprehensions, let’s say you wanted to generate a five row by five column data structure, you would need to use for loops as follows.

<iframe src="https://medium.com/media/a7799f15a2176418d4c562312f4efe2f" frameborder=0></iframe>

That is six lines of code, not too bad. What if you could do the same in one line?

    [[randint(0, 10) for y in range(5)] for x in range(5)]

The above line is much more concise and removes the need to collect and append items to a list. It’s actually two list comprehensions:

    # Collects rows
    [<row> for x in range(5)]

    # Builds each row
    [randint(0, 10) for y in range(5)]

## Filtering Data

The above code is an example of transforming data. It converted two range statements into a 5x5 matrix of data. List comprehensions can also use conditionals to further transform data.

    # Select columns 0, 1, 3
    [[column for x, column in enumerate(row) if x in (0, 1, 3)] for row in data]

The above line selects the 0th, 1st, and 3rd column from each row and builds a new matrix. Once again, it’s actually two list comprehensions.

    # Row iterator
    [<row> for row in data]

    # Transforms row
    [[column for x, column in enumerate(row) if x in (0, 1, 3)]

## Dictionary Comprehensions

Dictionaries can also be constructed using comprehensions. The following logic will create a column-based view of the data, with the key being the column number and value being the column values.

    # Assumes all rows have the same number of columns
    columns = {i: [row[i] for row in data] for i in range(len(data[0]))}

This example iterates through a list like the previous examples but it adds a new twist. Instead of having a single expression, each iteration builds a key-value pair.

## Running an Example

The following is a full example of the functionality discussed.

<iframe src="https://medium.com/media/6a7eba556156a4fbc06d8d10b2837225" frameborder=0></iframe>

Which will generate the following output:

    Row view:
    [[1, 2, 4, 7, 8],
     [1, 0, 0, 5, 2],
     [0, 0, 2, 3, 12],
     [1, 0, 0, 5, 14],
     [1, 1, 1, 7, 3]]

    Column view:
    {0: [1, 1, 0, 1, 1],
     1: [2, 0, 0, 0, 1],
     2: [4, 0, 2, 0, 1],
     3: [7, 5, 3, 5, 7],
     4: [8, 2, 12, 14, 3]}

    Stats:
    Column 0, min=0.00, max=1.00, mean=0.80, stdev=0.45
    Column 1, min=0.00, max=2.00, mean=0.60, stdev=0.89
    Column 2, min=0.00, max=4.00, mean=1.40, stdev=1.67
    Column 3, min=3.00, max=7.00, mean=5.40, stdev=1.67
    Column 4, min=2.00, max=14.00, mean=7.80, stdev=5.31

## Conclusion

Python has much already built into the language.

All of the discussed functionality is definitely made simple with libraries like pandas and NumPy. But it’s not always necessary if you have a small project and/or wish to limit third-party dependencies.

It’s not a bad choice to use pure Python.
