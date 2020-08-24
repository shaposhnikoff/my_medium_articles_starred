Unknown markup type 10 { type: [33m10[39m, start: [33m4[39m, end: [33m7[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m39[39m, end: [33m42[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m22[39m, end: [33m23[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m52[39m, end: [33m58[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m16[39m, end: [33m19[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m41[39m, end: [33m48[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m22[39m, end: [33m25[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m139[39m, end: [33m140[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m81[39m, end: [33m90[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m95[39m, end: [33m101[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m68[39m, end: [33m71[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m111[39m, end: [33m122[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m59[39m, end: [33m60[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m135[39m, end: [33m138[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m79[39m, end: [33m84[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m110[39m, end: [33m111[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m135[39m, end: [33m136[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m140[39m, end: [33m151[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m57[39m, end: [33m68[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m137[39m, end: [33m149[39m }

# Stop Using range() in Your Python for Loops

How to access the current index using the enumerate() function

![Photo by [bantersnaps](https://unsplash.com/@bantersnaps?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/range?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)](https://cdn-images-1.medium.com/max/12000/1*Xkji5B8NF7erM82B4i38nA.jpeg)*Photo by [bantersnaps](https://unsplash.com/@bantersnaps?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/range?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)*

The for loop. It is a cornerstone of programming — a technique you learn as a novice and one that you’ll carry through the rest of your programming journey.

If you’re coming from other popular languages such as PHP or JavaScript, you’re familiar with using a variable to keep track of your current index.

<iframe src="https://medium.com/media/2065a24abb498ecce993566fda3feef4" frameborder=0></iframe>

It’s critical to understand that these for loops do not actually iterate over the array; they manually iterate via an expression that serves as a proxy for referencing each array value.

In the example above, i has no explicit relation to scores, it simply happens to coincide with each necessary index value.

## The Old (Bad) Way

The traditional for loop as shown above does not exist in Python. However, if you’re like me, your first instinct is to find a way to recreate what you’re comfortable with.

As a result, you may have discovered the range() function and come up with something like this.

<iframe src="https://medium.com/media/75fdabe25e63481433c08b668fcbbc21" frameborder=0></iframe>

The problem with this for loop is that it isn’t very “Pythonic”. We’re not actually iterating over the list itself, but rather we’re using i as a proxy index.

In fact, even in JavaScript there are methods of directly iterating over arrays ([forEach()](https://medium.com/better-programming/stop-using-for-loops-to-iterate-over-arrays-5c46940e79d1) and [for…of](https://medium.com/better-programming/use-for-of-to-loop-through-your-javascript-arrays-57ebb900ab5a)).

## Using the enumerate() Function

If you want to properly keep track of the “index value” in a Python for loop, the answer is to make use of the enumerate() function, which will “count over” an iterable—yes, you can use it for other data types like strings, tuples, and dictionaries.

The function takes two arguments: the iterable and an optional starting count.

If a starting count is not passed, then it will default to 0. Then, the function will return tuples with each current count and respective value in the iterable.

    scores = [54,67,48,99,27]

    for i, score in enumerate(scores):
       print(i, score)

This code is so much cleaner. We avoid dealing with list indices, iterate over the actual values, and explicitly see each value in the for loop’s definition.

Here’s a bonus, have you ever wanted to print a numbered list but had to print i + 1 since the first index is 0? Simply pass the value 1 to enumerate() and watch the magic!

<iframe src="https://medium.com/media/a9e7595b39588c3d0ac5ff350eeeafa7" frameborder=0></iframe>

I hope this tutorial was helpful. What other uses of the enumerate() function have you found? Do you find the syntax easier to read than range(len())? Share your thoughts and experiences below!
