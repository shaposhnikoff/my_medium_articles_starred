Unknown markup type 10 { type: [33m10[39m, start: [33m66[39m, end: [33m77[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m41[39m, end: [33m42[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m29[39m, end: [33m35[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m36[39m, end: [33m45[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m165[39m, end: [33m177[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m71[39m, end: [33m81[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m126[39m, end: [33m133[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m73[39m, end: [33m77[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m146[39m, end: [33m152[39m }

# Learn Guitar with Python

Revealing the musical scales with lists, dicts, and matplotlib

![Photo by [Andrew Yardley](https://unsplash.com/@andrewyardley?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/2000/1*OEQDaZHb7MWdNjGukYP-_Q.png)*Photo by [Andrew Yardley](https://unsplash.com/@andrewyardley?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

What can we do with Python to help us play an instrument?

In this tutorial, we will start off from scratch and see how we can use lists, dictionaries, and matplotlib to build a tool that can help us play the guitar. But before we start, let’s have a crash course in some principles of music we will need later on. If you’re curious, check out [this interactive colab notebook](https://colab.research.google.com/github/diegopenilla/PythonGuitar/blob/master/How_to_learn_guitar_with_Python.ipynb) to try it out yourself.

## Background Information

Music theory is a complex subject made up of a lot of components. For the purposes of this post, we will be simplifying things here quite a bit, so if you are a musician don’t take it too seriously.

Pretend that you can imagine the spectrum of audible sound, and try to divide it into a musical system that makes sense for you and everyone around you. This is not easy! After centuries of struggle, humans have come up with a system that looks like this (but there are still disputes going on):

![Notes and their frequencies](https://cdn-images-1.medium.com/max/2000/1*8FC4spQheCISvBkqwhtbdQ.png)*Notes and their frequencies*

If you inspect the rows, you will notice that the notes are the same but the frequencies are different. What you see this way are called octaves: the [interval](https://en.wikipedia.org/wiki/Interval_(music)) between one musical [pitch](https://en.wikipedia.org/wiki/Pitch_(music)) and another with double its frequency. It’s amazing to hear how this proportion make these different sounds feel so similar and related to one another. In fact:
> The human ear tends to hear these as being essentially “the same” due to closely related harmonics.

For this post, we will not bother about implementing frequencies but will only take into account the different notes within an octave—those that you see in every column:

![](https://cdn-images-1.medium.com/max/2000/1*v8AF8o9-J9NIDndtEibNEg.png)

At this point, there are some things I must mention:

* The interval between C and C# or between C# and D (right next to each other) is called a **semitone **or** half step.**

* The interval between C and D or between F and G is called a** tone **or **step.**

But why I am saying this? Turns out the music you are used to hear is based on a set of [musical scales](https://en.wikibooks.org/wiki/Music_Theory/Scales_and_Intervals). And these are nothing more than* intervals between the notes that you choose to play*. Or in more harsh terms, they are just patterns.

The easiest scale possible is the [chromatic scale](https://en.wikipedia.org/wiki/Chromatic_scale), in which you take a*ll the semitones* between an octave (literally all the different notes). Let’s create a variable to hold the A chromatic scale:

    A = ['A','A#', 'B', 'C','C#','D','D#','E','F','F#' ,'G','G#']

The first note of a scale is called the **root **and** **gives the scale its name. It is also the place where you rest your fingers on when you are playing (as it feels like the base sound that holds everything together). Let’s see another chromatic scale starting with a root in C (the C chromatic scale):

    C = ['C','C#','D','D#','E','F','F#' ,'G','G#','A','A#', 'B']

Interestingly, even though these two scales have exactly the same notes, because they start with a different root the intervals between the individual notes and the root is completely different, which makes these scales feel nothing like each other.

But enough about chromatic scales (taking all semitones doesn’t sound so pleasant). Guided by their sense of beauty, humans have come up with many ways of playing these notes. After a long selection process, [some scales ](https://en.wikipedia.org/wiki/List_of_musical_scales_and_modes)have stuck and flourished into the music we know today. If you hear any of these legendary scales you will notice they sound [very distinct](https://en.wikipedia.org/wiki/Scale_(music)). Let’s take a look at the C major scale and compare it to the C chromatic scale shown above.

    C_major_scale = ['C', 'D', 'E', F', 'G', 'A', 'B']

By taking only specific intervals, we create the major scale as shown below:

![Intervals of the major scale: t*one, tone, semitone, tone, tone, tone, semitone.*](https://cdn-images-1.medium.com/max/2000/1*0ongukHsqYx5Mv534qhmBw.png)*Intervals of the major scale: t*one, tone, semitone, tone, tone, tone, semitone.**

If we start with the C chromatic scale and follow this pattern, we get the C major scale. If we start with the A chromatic scale and follow these intervals, we get the A major scale and so forth.

*By transforming these intervals to indexes*, we can implement this in Python fairly easily. All we have to do it is access the values of a list using indexes. Let’s see how to get the notes of C major:

    C = ['C','C#','D','D#','E','F','F#' ,'G','G#','A','A#', 'B']

    # intervals of the major scale as indexes:
    major_scale = [0, 2, 4, 5, 7, 9, 11]

    # select elements with these indexes into a list
    notes = [C[i] for i in major_scale]

    print(notes)

![](https://cdn-images-1.medium.com/max/2000/1*-7GsMv3f-u3-fhVhhutEBQ.png)

**In summary**: using the intervals of a given scale as indexes, we can retrieve the notes that make up the scale. This is all you need to understand. We will be applying this principle over and over again to extract the notes that compose a given scale. Using this information, we will then plot where these notes are located in the guitar.

## Overview

The purpose of this tutorial is to create a program with which you can find the position of the notes of a given scale in the guitar. For that, we will have to figure out methods to:

* Extract the notes composing a given scale.

* Locate where they are positioned in the strings of the guitar.

* Plot the strings and frets of the guitar along the notes of the scale.

Let’s start!

### Extracting notes

An easy way to create chromatic scales (without typing them explicitly) is to concatenate a list with all the notes within itself and slice the desired elements. Let’s see how:

    whole_notes = ['C','C#','D','D#','E','F',
                   'F#','G','G#','A','A#','B']*2

We concatenate two octaves worth of notes together and save it as whole_notes. Having a list like this allows us to easily slice chromatic scales, starting with any root. Let’s see an example with B:

    # index where element 'B' is located.
    root = whole_notes.index('B')

    # starting from this index, slice twelve elements
    B = whole_notes[root:root+12]
    print(B)

![B chromatic scale](https://cdn-images-1.medium.com/max/2000/1*i_28sCr8oeS6fjgyBF1ecQ.png)*B chromatic scale*

Now that we have an octave starting from B, we can use the major scale intervals we defined to retrieve the B major scale (just as we did before):

    notes = [B[i] for i in major_scale]
    print(notes)

![](https://cdn-images-1.medium.com/max/2000/1*gMQugXzbxDiKyrwS7NSAVg.png)

If we want to retrieve the notes contained in another scale, then:

    another_scale = [0, 2, 5, 10, 11]
    notes = [B[i] for i in another_scale]
    print(notes)

![](https://cdn-images-1.medium.com/max/2000/1*0L-vtv84RkqGQQ6LetKPRQ.png)

Hopefully now you can see where this is all going. We’d better write this a function for convenience (this summarizes all the code we’ve implemented so far):

<iframe src="https://medium.com/media/2135bb5520e415f3454c33c26db818b4" frameborder=0></iframe>

* We create a chromatic scale (octave) starting at a given root.

* Then extract the notes specified by intervals(the indexes of notes to be played) and return them in a list.

Now let’s create a dictionary with common musical scales to make this more useful:

<iframe src="https://medium.com/media/ce62882899e985b38e54ffe28705492d" frameborder=0></iframe>

Using this dictionary, we now have an easy way of accessing the notes of any scale, starting at any root as seen below:

    get_notes('A', scales['minor'])

![A minor scale](https://cdn-images-1.medium.com/max/2000/1*wYFfco9Naib4Btj421rOBA.png)*A minor scale*

    get_notes('E', scales['harmonic_minor'])

![E harmonic minor scale](https://cdn-images-1.medium.com/max/2000/1*bs4Sp_njbdA27NOFDXqCpQ.png)*E harmonic minor scale*

    get_notes('F', scales['minor_blues'])

![F minor blues scale](https://cdn-images-1.medium.com/max/2000/1*cXt6qwDcUreQegcB-LFEjQ.png)*F minor blues scale*

and so forth…

<iframe src="https://medium.com/media/3c98d3f9a4bf5faa433932362263473e" frameborder=0></iframe>

## Creating the Guitar

![[Guitar notes ](http://www.simplifyingtheory.com/guitar-notes-piano/)for the first 15 frets. We see the six strings of the guitar: low E, A, D, G, B and high E.](https://cdn-images-1.medium.com/max/2000/1*-ULyT6axpA-DiJWSWhBq1w.png)*[Guitar notes ](http://www.simplifyingtheory.com/guitar-notes-piano/)for the first 15 frets. We see the six strings of the guitar: low E, A, D, G, B and high E.*

We’re almost done! Now that we have a way of accessing the notes of a scale, let’s create a guitar to visualize them. This will involve representing the strings of the guitar as a dictionary. Let’s see how we can implement this:

<iframe src="https://medium.com/media/696e94215a9424b271240948b1c0fc19" frameborder=0></iframe>

    print(string.keys())

![](https://cdn-images-1.medium.com/max/2000/1*vQnm7OfUkkmXD89Y_Rxi_Q.png)

    print(string['E'])

![Notes of the 20 first frets in the E string](https://cdn-images-1.medium.com/max/2000/1*dI6y0cBSQKGTU_Pa-qLH0Q.png)*Notes of the 20 first frets in the E string*

By having the notes in the strings in this manner, we are now able to match them with the notes of our scales to find their position.

### Finding notes in the guitar

To record the positions of the notes, we will be creating yet another dictionary. For each note in a given scale we will add the position (index) where it is located in each string of the guitar. For this we will be doing the same we’ve been doing so far: accessing elements of lists using indexes.

<iframe src="https://medium.com/media/84653d793daa13fb7b25e1ffbc22b163" frameborder=0></iframe>

The only little detail we have to pay attention to is that because the our strings contain 20 and not 12 notes, some notes are repeated. As we saw before by printingstrings['E'], all the notes from E to B are repeated twice. The last note to be repeated, B, is at indexes 7 and 19. So, for all notes found in the strings at an index 7 or less we must add its duplicate index.

Calling the function returns a dictionary with the name of the strings as keys and the position of notes in the specified scale as values.

    # finding notes in a scale:
    C_minor_blues = get_notes('C', scales['minor_blues'])

    # finding positions of these notes in the guitar, as a dict
    positions = find_notes(C_minor_blues)
    print(positions)

![Position of notes of the C minor blues scale in the guitar](https://cdn-images-1.medium.com/max/2000/1*ax2jmIvmuefJJwOHSkCWEw.png)*Position of notes of the C minor blues scale in the guitar*

    # accessing note positions in string E
    print(positions['E'])

![](https://cdn-images-1.medium.com/max/2000/1*INDiSI1RNdoJ7lOJbdAwuA.png)

Now that we have easy access to this information, we are finally able to plot this in the guitar!

## Plotting the Guitar

Displaying a guitar involves several steps:

* Plotting the strings E, A,D,G,B and high E as horizontal lines in a 2D plot at different heights y=1, 2, 3, 4, 5, 6. (For loop line 9 ).

* Plotting the frets of the guitar as vertical lines. (For loop line 12).

* Finding the position of notes for our desired scale using the function find_notes from before and saving them as a dictionary to_plot.

* Creating circles with labels inside to represent the notes and their location in the guitar. (Nested for loop lines 28, 29).

I also added some non-essential code to make it look cooler. Make sure to check the comments if you are curious.

<iframe src="https://medium.com/media/4923fdec49b33f3947c74dd751862527" frameborder=0></iframe>

Here is the end result of this little adventure: by calling the function plot and specifying a root and a scale, we can display a *matplotlib guitar* with the info we need to play along.

    plot('C', scales['major'])

![The C major scale](https://cdn-images-1.medium.com/max/2144/1*ADwZ9TzkAlyfSRIBc8C6sA.png)*The C major scale*

    plot('F', scales['lydian'])

![F lydian scale](https://cdn-images-1.medium.com/max/4328/1*Hlct5l6d6mNVbIPzZiIbLw.png)*F lydian scale*

    plot('D', scales['minor_blues'])

![D minor blues scale](https://cdn-images-1.medium.com/max/4344/1*ycTwoGG0A0dsbuk9wlJz4Q.png)*D minor blues scale*

Now we can play any scale we want without having to google it all the time. If you feel like playing your own custom scales, just add them to the scalesdictionary.

<iframe src="https://medium.com/media/8b57b274c9df3bb0d37628f55f4e58a0" frameborder=0></iframe>

### Thanks for reading!

That’s it! I hope you had some fun reading these thoughts—and that you actually use this to play the guitar!

[In the collab notebook](https://colab.research.google.com/github/diegopenilla/PythonGuitar/blob/master/How_to_learn_guitar_with_Python.ipynb), I implemented some dropdown menus to make it easier to use, so feel free to check that out.

(For all musicians out there, forgive me for not bothering about ascending or descending scales or using only sharps and not a single flat♭to represent the notes, but it made things more simple!)
