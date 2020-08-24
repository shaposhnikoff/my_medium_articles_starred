Unknown markup type 10 { type: [33m10[39m, start: [33m41[39m, end: [33m51[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m56[39m, end: [33m61[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m10[39m, end: [33m20[39m }
Unknown markup type 10 {
  type: [33m10[39m,
  start: [33m30[39m,
  end: [33m46[39m,
  href: [32m''[39m,
  title: [32m''[39m,
  rel: [32m''[39m,
  name: [32m''[39m,
  anchorType: [33m0[39m,
  creatorIds: [],
  userId: [32m''[39m
}
Unknown markup type 10 { type: [33m10[39m, start: [33m20[39m, end: [33m26[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m39[39m, end: [33m57[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m4[39m, end: [33m10[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m23[39m, end: [33m48[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m155[39m, end: [33m179[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m24[39m, end: [33m30[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m26[39m, end: [33m52[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m12[39m, end: [33m20[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m25[39m, end: [33m41[39m }
Unknown markup type 10 {
  type: [33m10[39m,
  start: [33m48[39m,
  end: [33m64[39m,
  href: [32m''[39m,
  title: [32m''[39m,
  rel: [32m''[39m,
  name: [32m''[39m,
  anchorType: [33m0[39m,
  creatorIds: [],
  userId: [32m''[39m
}
Unknown markup type 10 { type: [33m10[39m, start: [33m213[39m, end: [33m237[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m46[39m, end: [33m53[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m58[39m, end: [33m65[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m67[39m, end: [33m74[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m114[39m, end: [33m121[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m21[39m, end: [33m25[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m128[39m, end: [33m139[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m195[39m, end: [33m199[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m20[39m, end: [33m24[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m52[39m, end: [33m56[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m58[39m, end: [33m62[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m64[39m, end: [33m69[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m74[39m, end: [33m79[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m4[39m }
Unknown markup type 10 {
  type: [33m10[39m,
  start: [33m0[39m,
  end: [33m4[39m,
  href: [32m''[39m,
  title: [32m''[39m,
  rel: [32m''[39m,
  name: [32m''[39m,
  anchorType: [33m0[39m,
  creatorIds: [],
  userId: [32m''[39m
}
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m5[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m5[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m76[39m, end: [33m101[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m114[39m, end: [33m138[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m84[39m, end: [33m89[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m147[39m, end: [33m152[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m156[39m, end: [33m171[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m27[39m, end: [33m31[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m73[39m, end: [33m82[39m }

# Tutorial: Amazon price tracker using Python and MongoDB (Part 1)

A two-part tutorial on how to create an Amazon price tracker.

![Photo by [M. B. M.](https://unsplash.com/@m_b_m?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/12000/0*9yrEnlgpzV0DWGOP)*Photo by [M. B. M.](https://unsplash.com/@m_b_m?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

Recently there was an Amazon sale and I wanted to buy a product that I had been checking on for a long time. But, as I was ready to buy it I noticed that the price had increased and I wondered if the sale was even legitimate. So I figured by creating this price tracker app, it would not only increase my fluency in python but I would also have my very own home-brewed app to track amazon prices.
> # While I have been programming for a long time, it is only recently that I have picked up python but it’s a bliss so far. If any of you python experts find my code to be not very “Pythonic”, my apologies, I will learn more :).
> This tutorial assumes that you have at least basic knowledge of python. Also that you have Python and MongoDB installed on your system.
> Note that this tutorial is meant to demonstrate on how to create an amazon price tracker and not to teach programming.

So, without further ado let us begin part 1 of this tutorial.

### Step 1: Creating files and folder for the project

* Open whichever directory you like and create a folder, name it amazon_price_tracker or just anything you want.

* Now open the folder and create two files scraper.py and db.py.

That’s all for the first step, now open the *terminal* in the projects directory and head to the next step.

### Step 2(Optional): Creating a virtual environment with virtualenv

This is an **Optional** step to isolate the packages that are being installed. You can find more about virtualenv [here](https://virtualenv.pypa.io/en/latest/userguide/).

Run this to create an environment.

    $ virtualenv ENV

And run this to activate the environment.

    $ source ENV/bin/activate

If you want to deactivate the environment then simply run the following.

    $ deactivate

Now, activate the environment if you haven’t already and head to step 3.

### Step 3: Installing the required packages.

* Run this command to Install **requests** (a library to make HTTP requests)

    $ pip install requests

* Run this command to Install **BeautifulSoup4** (a library to scrape information from web pages)

    $ pip install bs4

* Run this command to install **html5lib**(modern HTML5 parser)

    $ pip install html5lib

* Run this command to install **pymongo** (a driver to access MongoDB)

    $ pip install pymongo

### Step 4: Starting to code the extract_url(URL) function

Now, open scraper.py and we need to import a few packages that we had previously installed.

    import requests
    from bs4 import BeautifulSoup

Now, let us create a function** **extract_url(URL) to make the URL shorter and verify if the URL is valid [www.amazon.in](http://www.amazon.in) URL or not.

<iframe src="https://medium.com/media/3cfff667adedd16004e8f016ae7ff720" frameborder=0></iframe>

This function takes an Amazon India URL such as:
> [https://www.amazon.in/Samsung-Galaxy-M30-Gradation-Blue/dp/B07HGJJ58K/ref=br_msw_pdt-1?_encoding=UTF8&smid=A1EWEIV3F4B24B&pf_rd_m=A1VBAL9TL5WCBF&pf_rd_s=&pf_rd_r=VFJ98F93X80YWYQNR3GN&pf_rd_t=36701&pf_rd_p=9806b2c4-09c8-4373-b954-bae25b7ea046&pf_rd_i=desktop](https://www.amazon.in/dp/B07HGJJ58K/)”

and converts them to shorter URL [https://www.amazon.in/dp/B07HGJJ58K](https://www.amazon.in/dp/B07HGJJ58K/) which is more manageable. Also if the URL is not valid [www.amazon.in](http://www.amazon.in) URL then it would return a *None*

### Step 5: What we need for the next function

For the next function Google [“my user agent](https://www.google.co.in/search?q=my+user+agent)”, copy your user agent and assign it to variable headers.

    headers = { "User-Agent": "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.100 Safari/537.36"
    }

Before we create a function to scrap details from the page let us visit an Amazon product page like [this](https://www.amazon.in/dp/B07HGJJ58K) one and find the elements that have the name and the price of the product. We will need the element’s id to be able to find the elements when we extract the data.

![](https://cdn-images-1.medium.com/max/2732/1*SY2KtViywQoey2jKD0bqzQ.png)

Once the page is rendered, do a right-click on the name of the product and click on “inspect” which will show the element which has the name of the product.

![](https://cdn-images-1.medium.com/max/2000/1*KrYoUtGHlfknkcIJctI2TA.png)

We can see that the* *<span> element with id=“productTitle”* *now hold on to this id, we will use it later to scrap the name of the product.

![](https://cdn-images-1.medium.com/max/2732/1*uMz87W6Z9Nk8uelqYHKcwg.png)

We will do the same for the price, now right-click on the price and click on inspect.

![](https://cdn-images-1.medium.com/max/2000/1*4zTHFCsDzqMjB9l_1_bFVg.png)

The <span>element with id=“priceblock_dealprice”* *has the price that we need. But, this product is on sale so its id is different from a normal id which is id=“priceblock_ourprice”*.*

### Step 6: Creating the price converter function

If you look closely the <span> element has the price but it has many unwanted pieces of stuff like the ₹ rupee symbol, blank space, a comma separator, and decimal point.

We just want the integer portion of the price, so we will create a price converter function that will remove the unwanted characters and gives us the price in integer type.

Let us name this function get_converted_price(price)

<iframe src="https://medium.com/media/da89ec8b7a7a491be13759926d4750ee" frameborder=0></iframe>

With some simple string manipulations, this function will give us the converted price in integer type.
> ***UPDATE: **As mention by @[*oorjahalt](undefined) *we can simply use regex to extract price.*

<iframe src="https://medium.com/media/6e2a76c5ec2061c396a99b9f7568ba6f" frameborder=0></iframe>
> **NOTE**: While tracker is meant for [www.amazon.in](http://www.amazon.in), this may very well be used for [www.amazon.com](http://www.amazon.com) or other similar websites with very minor changes such as:

To make this compatible with the global version of amazon simply do this:

* change the ₹ to $

    stripped_price = price.strip("$ ,")

* we can skip find_dot and to_convert_price*** ***entirely and just do this

    converted_price = float(replace_price)

We would, however, will be converting the price to a float type.

* And changing [*www.amazon.in](http://www.amazon.in)* to* [www.amazon.com](http://www.amazon.com)* in extract_url(URL) function

This would make it compatible with [*www.amazon.com.](http://www.amazon.com.)*

Now, as we buckle up we can finally proceed towards creating the scraper function.

### Step 7: Onto the details scraper function

OK so let us create the function that would extract the details of the product such as its name, price and returns a dictionary that contains the name, price and the URL of the product. We will name this function get_product_details(URL).

The first two variables for this function are headers and details***,* **headers** **which** **will contain your user-agent and details is a dictionary that will contain the details for the product.

    headers = {
            "User-Agent": "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.100 Safari/537.36"
    }
    details = {"name": "", "price": 0, "deal": True, "url": ""}

The another variable _url will hold the extracted URL for us and we will check if the URL is valid or not. An invalid URL would return None, if*** ***the URL is invalid then we will set the details to None*** ***and return at the end so that we know something is wrong with the URL.

    _url = extract_url(url)
    if _url is None:
        details = None

Now, we come to the else part. This has 4 variables page, soup, title and price.

page*** ***variable*** ***will hold the requested product’s page.

soup variable will hold the HTML, with this we can do lots of stuff like finding an element with an id and extract its text, which is what we will do. You can find more about other BeautifulSoup’s function [here](https://www.crummy.com/software/BeautifulSoup/bs4/doc/).

title variable as the name suggests will hold the element that has the title of the product.

price variable will hold the element that has the price of the product.

    page = requests.get(url, headers=headers)
    soup = BeautifulSoup(page.content, "html5lib")
    title = soup.find(id="productTitle")
    price = soup.find(id="priceblock_dealprice")

Now that we have the elements for title and price, we will do some checks.

Let us begin with price, as mentioned earlier the id of price can be either id=”priceblock_dealprice” on deals or id=”priceblock_ourprice”*** ***on*** ***normal days.

    if price is None:
        price = soup.find(id="priceblock_ourprice")
        details["deal"] = False

Since we are first checking if there is any deal price or not, the code will change price from deal price to normal price and also set the deal to false in details[“deal”]*** ***if there is no deal price. This is done so that we know the price is normal.

Now, even then if we don’t get the price that means something is wrong with the page, maybe the product is out of stock or maybe the product is not released yet or some other possibilities. The following code will check if there are title and price or not.

    if title is not None and price is not None:
        details["name"] = title.get_text().strip()
        details["price"] = get_converted_price(price.get_text())
        details["url"] = _url

If there are price and title of the product then we will store them.

    details["name"] = title.get_text().strip()

This will store the name of the product but, we have to strip any unwanted blank leading and trailing spaces from the title. The strip() function remove any trailing and leading spaces.

    details["price"] = get_converted_price(price.get_text())

This will store the price of the product. With the help of the get_converted_price(price) function that we created earlier gives us the converted price in integer type.

    details["url"] = _url

This will store the extracted URL.

    else:
        details = None
    return details

We will set the details to None if the price or title doesn’t exist.

Finally, the function is complete and here is the complete code

<iframe src="https://medium.com/media/47a3f021faf6fd105bd90af214308f92" frameborder=0></iframe>
> **Note**: While this code does not work for books since books have different productid, you can make it work for books if you tweak the code.

### Step 8: Let us run scraper.py

At the end of the file add the following.

    print(get_product_details("Insert an Amazon URL"))

Open the terminal where you have your scraper.py file and run the scraper.py like so.

    $ python3 scraper.py

If you have done everything correctly you should get an output like this.

    {‘name’: ‘Nokia 8.1 (Iron, 4GB RAM, 64GB Storage)’, ‘price’: 19999, ‘deal’: False, ‘url’: ‘[https://www.amazon.in/dp/B077Q42J32'](https://www.amazon.in/dp/B077Q42J32')}

And voila we have completed Part 1 of the Amazon price tracker tutorial.

I will see you next week with the follow-up part 2 where we will explore **MongoDB** using P**yMongo** to store our data.
> # This was my first article on medium or blogging in general and I am excited to share my thoughts, experiments and stories here in the future. Hope you’ll like it.

*Find the complete source code for this article below.*
[**ArticB/amazon_price_tracker**
*amazon_price_tracker-An amazon price tracker using Python and MongoDB*github.com](https://github.com/ArticB/amazon_price_tracker)

**Follow the link below for part 2.**
[**Tutorial: Amazon price tracker using Python and MongoDB (Part 2)**
*A two-part tutorial on how to create Amazon price tracker*medium.com](https://medium.com/@deeprajpradhan/tutorial-amazon-price-tracker-using-python-and-mongodb-part-2-5a9107ed2204)
