Unknown markup type 10 { type: [33m10[39m, start: [33m29[39m, end: [33m36[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m111[39m, end: [33m118[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m105[39m, end: [33m112[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m77[39m, end: [33m84[39m }

# How to Hash in Python

Encrypt, decrypt, checksum, and more

![Photo by [Vincentiu Solomon](https://unsplash.com/@vincentiu?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/10368/0*ZXzU561zLq1calah)*Photo by [Vincentiu Solomon](https://unsplash.com/@vincentiu?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

Hashing is a key part of most programming languages. Large amounts of data can be represented in a fixed buffer. Key-value structures use hashes to store references.

Hashes are used to secure. Hashes can be deterministic or non-deterministic. Hashes can be significantly different with small changes to data or very similar.

This article will review the most common ways to hash data in Python.

## 1. Built-In Hashing

Python provides the built-in .hash() function as shown below.

    >>> hash("test")
    2314058222102390712

The above was run in Python 2.7, let’s try Python 3.7.

    >>> hash("test")
    5946494221830395164

The result is different and will be different for each new Python invocation. Python has never guaranteed that .hash() is deterministic.

In Python 2.x, it would be deterministic most of the time but not always. Python 3.x added randomness to .hash() to improve security. The default sort order of dictionaries, sets, and lists is backed by built-in hashing.

I have a [whole project](https://github.com/neuml/py27hash) covering Python 2.x hashing in Python 3.x. Generally, .hash() shouldn’t be relied on for anything across Python invocations.

## 2. Checksums

Checksums are used to validate data in a file. ZIP files use checksums to ensure a file is not corrupt when decompressing. Unlike Python’s built-in hashing, it’s deterministic. The same data will return the same result each time.

Below is an example using [zlib](https://www.zlib.net/)’s alder32 and crc32. Alder32 is usually a better choice as it’s much faster and almost as reliable as crc32.

    >>> import zlib
    >>> zlib.adler32(b"test")
    73204161
    >>> zlib.crc32(b"test")
    3632233996

For a small database, adler32 could be used as a simple ID hash. But collisions will quickly become a concern as data grows.

## 3. Secure Hashing

Secure hashes and message digests have evolved over the years. From MD5 to SHA1 to SHA256 to SHA512.

Each method grows in size, improving security and reducing the risk of hash collisions. A collision is when two different arrays of data resolve to the same hash.

Hashing can take a large amount of arbitrary data and build a digest of the content. Open-source software builds digests of their packages to help users know that they can trust that files haven’t been tampered with. Small changes to the file will result in a much different hash.

Look at how different two MD5 hashes are after changing one character.

    >>> import hashlib
    >>> hashlib.md5(b"test1").hexdigest()
    '5a105e8b9d40e1329780d62ea2265d8a'
    >>> hashlib.md5(b"test2").hexdigest()
    'ad0234829205b9033196ba818f7a872b'

Let’s look at some common secure hash algorithms.

### MD5– 16 bytes/128 bit

MD5 hashes are 16 bytes or 128 bits long. See the example below, note that a hex digest is representing each byte as a hex string (i.e. the leading 09 is one byte). MD5 hashes are no longer commonly used.

    >>> import hashlib
    >>> hashlib.md5(b"test").hexdigest()
    '098f6bcd4621d373cade4e832627b4f6'
    >>> len(hashlib.md5(b"test").digest())
    16

### SHA1–20 bytes/160 bit

SHA1 hashes are 20 bytes or 160 bits long. SHA1 hashes are also no longer commonly used.

    >>> import hashlib
    >>> hashlib.sha1(b"test").hexdigest()
    'a94a8fe5ccb19ba61c4c0873d391e987982fbbd3'
    >>> len(hashlib.sha1(b"test").digest())
    20

### SHA256–32 bytes/256 bit

SHA256 hashes are 32 bytes or 256 bits long. SHA256 hashes are commonly used.

    >>> import hashlib
    >>> hashlib.sha256(b"test").hexdigest()
    '9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08'
    >>> len(hashlib.sha256(b"test").digest())
    32

### SHA512–64 bytes/512 bit

SHA512 hashes are 64 bytes or 512 bits long. SHA512 hashes are commonly used.

    >>> import hashlib
    >>> hashlib.sha512(b"test").hexdigest()
    'ee26b0dd4af7e749aa1a8ee3c10ae9923f618980772e473f8819a5d4940e0db27ac185f8a0e1d5f84f88bc887fd67b143732c304cc5fa9ad8e6f57f50028a8ff'
    >>> len(hashlib.sha512(b"test").digest())
    64

## 4. Near-Duplicate Detection

Up to this point, all the methods above generate significantly different hashes when data changes.

It’s a good property of a hash function to generate significantly different hashes with small changes to the data, especially for message digests. Think about if someone could tamper with a file and the hash would only be slightly different, not good.

But what if the objective is finding *similar* content. Duplicate or near-duplicate detection helps reduce the amount of data stored.

Some use cases require identifying subtle data differences such as plagiarism detection. The example below will install the [Simhash](https://github.com/leonsim/simhash) Python library to demonstrate.

    pip install simhash

After the install is finished, the following code is run.

<iframe src="https://medium.com/media/c99acf37ef8c6958c698482155177925" frameborder=0></iframe>

Notice how similar the hashes are for data that is close. But it’s much farther away for data that is different.

## 5. Perceptual Hashing

The last type of hashing we’ll cover is perceptual hashing. This hashing method is used to detect differences in images and video.

One example of this would be detecting near-duplicate frames in video. Algorithms can be used to eliminate storing duplicate content in a video or determining an image is close enough to consider it a duplicate, saving space.

The example below shows two near-duplicate images and how close their perceptual hashes are. The [ImageHash](https://github.com/JohannesBuchner/imagehash) Python library is used to demonstrate.

    pip install ImageHash

The following images are used:

![Original Image](https://cdn-images-1.medium.com/max/2000/1*Vdp6EeaPuYX-FnZM0wstJA.png)*Original Image*

![Modified Image — Removed “4 min read (894 words) so far”](https://cdn-images-1.medium.com/max/2000/1*KH3lu2igRbcfKDKvqXbttg.png)*Modified Image — Removed “4 min read (894 words) so far”*

Notice how the bottom image is almost identical except it removed text in the bottom-right. If a secure hash like MD5 is used, the hashes will be significantly different as designed. Let’s see.

<iframe src="https://medium.com/media/6431e6acd990d9cab2b80986181f9127" frameborder=0></iframe>

As expected, the hashes are completely different. Now let’s try ImageHash.

<iframe src="https://medium.com/media/0abd96049d29244a294a082205aea05d" frameborder=0></iframe>

The hashes are different but they are very close. Only one byte at the end is different. ImageHash allows subtracting two hashes to get the difference.

In some cases, it may be desired for this to be a duplicate, or detecting subtle differences in an image could be important. Perceptual hashing gives developers an option to detect this.

## **Conclusion**

This article covered a number of different ways to hash data in Python. Depending on the use case, these methods provide a number of options for building hashes. Hopefully, this helps with a current or future project.
