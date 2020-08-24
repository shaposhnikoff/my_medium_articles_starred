Unknown markup type 10 { type: [33m10[39m, start: [33m149[39m, end: [33m160[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m184[39m, end: [33m197[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m202[39m, end: [33m206[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m227[39m, end: [33m249[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m59[39m, end: [33m63[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m89[39m, end: [33m103[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m107[39m, end: [33m125[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m38[39m, end: [33m42[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m78[39m, end: [33m84[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m77[39m, end: [33m81[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m193[39m, end: [33m197[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m239[39m, end: [33m243[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m45[39m, end: [33m48[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m135[39m, end: [33m137[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m77[39m, end: [33m91[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m123[39m, end: [33m131[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m27[39m, end: [33m34[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m261[39m, end: [33m265[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m150[39m, end: [33m155[39m }

# Scraping URLs with BeautifulSoup

Use Python’s BeautifulSoup library to assist in the honest act of systematically stealing data without permission.

![](https://cdn-images-1.medium.com/max/3000/1*ISjP6mDJKdG3D4KPcUbPdQ.jpeg)

Whether it be [Kaggle](https://www.kaggle.com/), [Google Cloud](https://console.cloud.google.com/bigquery), or the [federal government](https://www.data.gov/), there’s plenty of reliable open-sourced data on the web. While there are plenty of reasons to hate being alive in our current chapter of humanity, open data is one of the few redeeming qualities of life on Earth today. But what is the opposite of “open” data, anyway?

Like anything free and easily accessible, the only data inherently worth anything is either harvested privately or stolen from sources that would prefer you didn’t. This is the sort of data business models can be built around, as social media platforms such as [LinkedIn](https://techcrunch.com/2018/11/24/linkedin-ireland-data-protection/) have shown us as our personal information is [bought and sold by data brokers](https://securityboulevard.com/2018/06/data-brokers-you-are-being-packaged-and-sold/). These companies [attempted to sue individual programmers](https://techcrunch.com/2016/08/15/linkedin-sues-scrapers/) like ourselves for scraping the data they collected via the same means, and epically lost in a court of law:

[https://www.forbes.com/sites/emmawoollacott/2019/09/10/linkedin-data-scraping-ruled-legal/#28997b001b54](https://www.forbes.com/sites/emmawoollacott/2019/09/10/linkedin-data-scraping-ruled-legal/#28997b001b54)

The topic of scraping data on the web tends to raise questions about the ethics and legality of scraping, to which I plea: *don’t hold back*. If you aren’t personally disgusted by the prospect of your life being transcribed, sold, and frequently leaked, the court system has ruled that you legally have a right to scrape data. The name of this publication is not **People Who Play It Safe And Slackers**. We’re a home for those who fight to take power back, and we’re going to scrape the shit out of you.

## Tools for the Job

Web scraping in Python is dominated by three major libraries: [**BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/bs4/doc/)**, [**Scrapy](https://scrapy.org/)**, and [**Selenium](https://selenium-python.readthedocs.io/)**. Each of these libraries intends to solve for very different use cases. Thus it’s essential to understand what we’re choosing and why.

* **BeautifulSoup** is one of the most prolific Python libraries in existence, in some part having shaped the web as we know it. BeautifulSoup is a lightweight, easy-to-learn, and highly effective way to programmatically isolate information on a single webpage at a time. It’s common to use BeautifulSoupin conjunction with the **requests** library, where *requests* will fetch a page, and *BeautifulSoup* will extract the resulting data.

* **Scrapy **has an agenda much closer to mass pillaging than BeautifulSoup. Scrapy is a tool for building crawlers: these are absolute monstrosities unleashed upon the web like a swarm, loosely following links, and haste-fully grabbing data where data exists to be grabbed. Because Scrapy serves the purpose of mass-scraping, it is much easier to get in trouble with Scrapy.

* **Selenium** isn’t exclusively a scraping tool as much as an automation tool that can be used to scrape sites. Selenium is the nuclear option for attempting to navigate sites programmatically, and should be treated as such: there are much better options for simple data extraction.

We’ll be using BeautifulSoup, which should genuinely be anybody’s default choice until the circumstances ask for more. BeautifulSoup is more than enough to steal data.

## Preparing Our Extraction

Before we steal any data, we need to set the stage. We’ll start by installing our two libraries of choice:

    $ pip3 install beautifulsoup4 requests

As mentioned before, **requests** will provide us with our target’s HTML, and **beautifulsoup4** will parse that data.

We need to recognize that a lot of sites have precautions to fend off scrapers from accessing their data. The first thing we can do to get around this is spoofing the headers we send along with our requests to make our scraper look like a legitimate browser:

    import requests

    headers = {
            'Access-Control-Allow-Origin': '*',
            'Access-Control-Allow-Methods': 'GET',
            'Access-Control-Allow-Headers': 'Content-Type',
            'Access-Control-Max-Age': '3600',
            'User-Agent': 'Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0'
        }

This is only a first line of defense (or offensive, in our case). There are plenty of ways sites can still keep us at bay, but setting headers works shockingly well to fix most issues.

Now let’s fetch a page and inspect it with BeautifulSoup:

    import requests
    from bs4 import BeautifulSoup

    ...
    url = "[http://example.com](http://example.com)"
    req = requests.get(url, headers)
    soup = BeautifulSoup(req.content, 'html.parser')
    print(soup.prettify())

We set things up by making a request to [http://example.com](http://example.com). We then create a BeautifulSoup object which accepts the raw content of that response via req.content. The second parameter, 'html.parser', is our way of telling BeautifulSoup that this is an HTML document. There are other parsers available for parsing stuff like XML, if you're into that.

When we create a BeautifulSoup object from a page’s HTML, our object contains the HTML structure of that page, which can now be easily parsed by all sorts of methods. First, let’s see what our variable soup looks like by using print(soup.prettify()):

    <html class="gr__example_com"><head>
        <title>Example Domain</title>
        <meta charset="utf-8">
        <meta http-equiv="Content-type" content="text/html; charset=utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <meta property="og:site_name" content="Example dot com">
        <meta property="og:type" content="website">
        <meta property="og:title" content="Example">
        <meta property="og:description" content="An Example website.">
        <meta property="og:image" content="[http://example.com/img/image.jpg](http://example.com/img/image.jpg)">
        <meta name="twitter:title" content="Hackers and Slackers">
        <meta name="twitter:description" content="An Example website.">
        <meta name="twitter:url" content="[http://example.com/](http://example.com/)">
        <meta name="twitter:image" content="[http://example.com/img/image.jpg](http://example.com/img/image.jpg)">
    </head>

    <body data-gr-c-s-loaded="true">
      <div>
        <h1>Example Domain</h1>
          <p>This domain is established to be used for illustrative examples in documents.</p>
          <p>You may use this domain in examples without prior coordination or asking for permission.</p>
        <p><a href="[http://www.iana.org/domains/example](http://www.iana.org/domains/example)">More information...</a></p>
      </div>
    </body>
    </html>

## Targeting HTML Elements

There are many methods available to us for pinpointing and grabbing the information we’re trying to get out of a page. Finding the *exact *information we want out of a web page is a bit of an art form: effective scraping requires us to recognize patterns in document’s HTML that we can take advantage of to ensure we only grab the pieces we need. This is especially the case when dealing with sites that actively try to prevent us from doing just that.

Understanding the tools we have at our disposal is the first step to developing a keen eye for what’s possible. We’ll start with the meat and potatoes.

### Using find() & find_all()

The most straightforward way to finding information in our soup variable is by utilizing soup.find(...) or soup.find_all(...). These two methods work the same with one exception: **find** returns the first HTML element found, whereas **find_all** returns a list of all elements matching the criteria (even if only one element is found, **find_all** will return a list of a single item).

We can search for DOM elements in our soup variable by searching for certain criteria. Passing a positional argument to **find_all** will return *all* anchor tags on the site:

    soup.find_all("a")
    # <a href="[http://example.com/elsie](http://example.com/elsie)" class="boy" id="link1">Elsie</a>
    # <a href="[http://example.com/lacie](http://example.com/lacie)" class="boy" id="link2">Lacie</a> 
    # <a href="[http://example.com/tillie](http://example.com/tillie)" class="girl" id="link3">Tillie</a>

We can also find all anchor tags which have the class name *“boy”*. Passing the class_ argument allows us to filter by class name. Note the underscore!

    soup.find_all("a" class_="boy")
    # <a href="[http://example.com/elsie](http://example.com/elsie)" class="boy" id="link1">Elsie</a>
    # <a href="[http://example.com/lacie](http://example.com/lacie)" class="boy" id="link2">Lacie</a>

If we wanted to get *any* element with the class name *“boy”* besides anchor tags, we can do that too:

    soup.find_all(class_="boy")
    # <a href="[http://example.com/elsie](http://example.com/elsie)" class="boy" id="link1">Elsie</a>
    # <a href="[http://example.com/lacie](http://example.com/lacie)" class="boy" id="link2">Lacie</a>

We can search for elements by id in the same way we searched for classes. Remember that we should only expect a single element to be returned with an id, so we should use **find** here:

    soup.find("a", id="link1")
    # <a href="[http://example.com/elsie](http://example.com/elsie)" class="boy" id="link1">Elsie</a>

Often times we’ll run into situations where elements don’t have reliable class or id values. Luckily we can search for DOM elements with any attribute, including non-standard ones:

    soup.find_all(attrs={"data-args": "bologna"})

### CSS Selectors

Searching HTML using CSS selectors is one of the most powerful ways to find what you’re looking for, especially for sites trying to make your life difficult. Using CSS selectors enables us to find and leverage highly-specific patterns in the target’s DOM structure. This is the best way to ensure we’re grabbing *exactly* the content we need. If you’re rusty on CSS selectors, I highly recommend becoming reacquainted. Here are a few examples:

    soup.select(".widget.author p")

In this example, we’re looking for an element that has a “widget” class, as well as an “author” class. Once we have that element, we go deeper to find any paragraph tags held within that widget. We could also modify this to get only the *second* paragraph tag inside the author widget:

    soup.select(".widget.author p:nth-of-type(2)")

To understand why this is so powerful, imagine a site that intentionally has no identifying attributes on its tags to keep people like you from scraping their data. Even without names to select by, we could observe the DOM structure of the page and find a unique way to navigate to the element we want:

    soup.select("body > div:first-of-type > div > ul li")

A specific pattern like this is likely unique to only a single collection of <li> tags on the page we're exploiting. The downside of this method is we're at the whim of the site owner, as their HTML structure could change.

## Get Some Attributes

Chances are we’ll almost always want the contents or the attributes of a tag, as opposed to the entirety of a tag’s HTML. If we’re scraping anchor tags, for instance, we probably just want the href value, as opposed to the entire tag. The .get method can be used here to retrieve values of attributes on a tag:

    soup.find_all('a').get('href')

The above finds the destination URLs for all <a> tags on a page. Another example can have us grab a site's logo image:

    soup.find(id="logo").get('src')

Sometimes it’s not attributes we’re looking for, but just the text within a tag:

    soup.find('p').get_text()

### Pesky Tags to Deal With

In our example of creating link previews, a good first source of information would obviously be the page’s meta tags: specifically the og tags they've specified to openly provide the bite-sized information we're looking for. Grabbing these tags are a bit more difficult to deal with:

    soup.find("meta", property="og:description").get('content')

Now that’s ugly. Meta tags are an especially interesting case; they’re all uselessly dubbed ‘meta’, thus we need a second identifier (in addition to the tag name) to specify *which *meta tag we care about. Only then can we bother to *get* the actual content of said tag.

## Realizing Something Will Always Break

If we were to try the above selector on an HTML page that did not contain an og:description, our script would break unforgivingly. Not only do we miss this data, but we miss out on everything entirely - this means we always need to build in a plan B, and at the very least deal with a lack of tag altogether.

It’s best to break out this logic one tag at a time. First, let’s look at an example for a base scraper with all the knowledge we have so far:

    import requests
    from bs4 import BeautifulSoup

    def scrape_page_metadata(url):
        """Scrape target URL for metadata."""
        headers = {
            'Access-Control-Allow-Origin': '*',
            'Access-Control-Allow-Methods': 'GET',
            'Access-Control-Allow-Headers': 'Content-Type',
            'Access-Control-Max-Age': '3600',
            'User-Agent': 'Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0'
        }
        pp = pprint.PrettyPrinter(indent=4)
        r = requests.get(url, headers=headers)
        html = BeautifulSoup(r.content, 'html.parser')
        metadata = {
            'title': get_title(html),
            'description': get_description(html),
            'image': get_image(html),
            'favicon': get_favicon(html, url),
            'sitename': get_site_name(html, url),
            'color': get_theme_color(html),
            'url': url
            }
        pp.pprint(metadata)
        return metadata

This function lays the foundation for snatching a given URL’s metadata. The result we’re looking for is a dictionary named metadata, which contains the data we manage to scrape successfully.

Each key in our dictionary has a corresponding function which attempts to scrape the corresponding information. Here’s what we have for fetching a page’s **title**, **description**, and** social image** values:

    ...

    def get_title(html):
        """Scrape page title."""
        title = None
        if html.title.string:
            title = html.title.string
        elif html.find("meta", property="og:title"):
            title = html.find("meta", property="og:title").get('content')
        elif html.find("meta", property="twitter:title"):
            title = html.find("meta", property="twitter:title").get('content')
        elif html.find("h1"):
            title = html.find("h1").string
        return title

    def get_description(html):
        """Scrape page description."""
        description = None
        if html.find("meta", property="description"):
            description = html.find("meta", property="description").get('content')
        elif html.find("meta", property="og:description"):
            description = html.find("meta", property="og:description").get('content')
        elif html.find("meta", property="twitter:description"):
            description = html.find("meta", property="twitter:description").get('content')
        elif html.find("p"):
            description = html.find("p").contents
        return description

    def get_image(html):
        """Scrape share image."""
        image = None
        if html.find("meta", property="image"):
            image = html.find("meta", property="image").get('content')
        elif html.find("meta", property="og:image"):
            image = html.find("meta", property="og:image").get('content')
        elif html.find("meta", property="twitter:image"):
            image = html.find("meta", property="twitter:image").get('content')
        elif html.find("img", src=True):
            image = html.find_all("img").get('src')
        return image

* **get_title **tries to get the <title> tag, which has a very low chance of failing. Just in case the target page actually *is* missing this tag, we fall back to Facebook and Twitter meta tags. If *all of this still fails*, we finally resort to trying to pull the first <h1> tag on the page (if we get to this point, we're probably scraping a garbage site).

* **get_description** is nearly identical to our method for scraping page titles. The last resort is a desperate attempt to pull the first paragraph on the page.

* **get_image **looks for the page’s “share” image, which is used to generate link previews on social media platforms. Our last resort is to pull the first <img> tag containing a source image.

## What Did We Build?

This simple script we just threw together is the basis for how most services generate “link previews”: an embedded widget containing a synopsis of a site before clicking in (think Facebook, Slack, Discord, etc.). There are even some services which charge monthly fees of ~$10/month to provide the service we’ve just built. Instead of paying for something like that, feel free to take my source code and use it as you please:

    """Scrape metadata from target URL."""
    import requests
    from bs4 import BeautifulSoup
    import pprint

    def scrape_page_metadata(url):
        """Scrape target URL for metadata."""
        headers = {
            'Access-Control-Allow-Origin': '*',
            'Access-Control-Allow-Methods': 'GET',
            'Access-Control-Allow-Headers': 'Content-Type',
            'Access-Control-Max-Age': '3600',
            'User-Agent': 'Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0'
        }
        pp = pprint.PrettyPrinter(indent=4)
        r = requests.get(url, headers=headers)
        html = BeautifulSoup(r.content, 'html.parser')
        metadata = {
            'title': get_title(html),
            'description': get_description(html),
            'image': get_image(html),
            'favicon': get_favicon(html, url),
            'sitename': get_site_name(html, url),
            'color': get_theme_color(html),
            'url': url
            }
        pp.pprint(metadata)
        return metadata

    def get_title(html):
        """Scrape page title."""
        title = None
        if html.title.string:
            title = html.title.string
        elif html.find("meta", property="og:title"):
            title = html.find("meta", property="og:title").get('content')
        elif html.find("meta", property="twitter:title"):
            title = html.find("meta", property="twitter:title").get('content')
        elif html.find("h1"):
            title = html.find("h1").string
        return title

    def get_description(html):
        """Scrape page description."""
        description = None
        if html.find("meta", property="description"):
            description = html.find("meta", property="description").get('content')
        elif html.find("meta", property="og:description"):
            description = html.find("meta", property="og:description").get('content')
        elif html.find("meta", property="twitter:description"):
            description = html.find("meta", property="twitter:description").get('content')
        elif html.find("p"):
            description = html.find("p").contents
        return description

    def get_image(html):
        """Scrape share image."""
        image = None
        if html.find("meta", property="image"):
            image = html.find("meta", property="image").get('content')
        elif html.find("meta", property="og:image"):
            image = html.find("meta", property="og:image").get('content')
        elif html.find("meta", property="twitter:image"):
            image = html.find("meta", property="twitter:image").get('content')
        elif html.find("img", src=True):
            image = html.find_all("img").get('src')
        return image

    def get_site_name(html, url):
        """Scrape site name."""
        if html.find("meta", property="og:site_name"):
            sitename = html.find("meta", property="og:site_name").get('content')
        elif html.find("meta", property='twitter:title'):
            sitename = html.find("meta", property="twitter:title").get('content')
        else:
            sitename = url.split('//')[1]
            return sitename.split('/')[0].rsplit('.')[1].capitalize()
        return sitename

    def get_favicon(html, url):
        """Scrape favicon."""
        if html.find("link", attrs={"rel": "icon"}):
            favicon = html.find("link", attrs={"rel": "icon"}).get('href')
        elif html.find("link", attrs={"rel": "shortcut icon"}):
            favicon = html.find("link", attrs={"rel": "shortcut icon"}).get('href')
        else:
            favicon = f'{url.rstrip("/")}/favicon.ico'
        return favicon

    def get_theme_color(html):
        """Scrape brand color."""
        if html.find("meta", property="theme-color"):
            color = html.find(
                "meta",
                property="theme-color"
            ).get('content')
            return color
        return None

I’ve uploaded the source code for this tutorial to Github, which contains instructions on how to download and run this script yourself. Enjoy, and join us next time when we up the ante with more nefarious scraping tactics!

[https://github.com/hackersandslackers/beautifulsoup-tutorial](https://github.com/hackersandslackers/beautifulsoup-tutorial)

*Originally published at [hackersandslackers.com](https://hackersandslackers.com/scraping-urls-with-beautifulsoup/) on November 11, 2018.*
