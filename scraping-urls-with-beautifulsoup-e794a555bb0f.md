
# Scraping URLs with BeautifulSoup

The honest act of robotically stealing data

![](https://cdn-images-1.medium.com/max/3000/1*VzSOh793LaxwuxtRDX7Evg.jpeg)

There are plenty of reliable and open sources of data on the web. Datasets are freely released to the public domain by the likes of Kaggle, Google Cloud, and of course local & federal government. Like most things free and open, however, following the rules to obtain public data can be a bit… boring. I’m not suggesting we go and blatantly break some grey-area laws by stealing data, but this blog isn’t exactly called **People Who Play It Safe And Slackers**, either.

My personal Python roots can actually be traced back to an ambitious side-project: to aggregate all new music from across the web and deliver it the masses. While that project may have been abandoned (after realizing it already existed), **BeautifulSoup** was more-or-less my first ever experience with Python.

## The Tool(s) for the Job(s)

Before going any further, we’d be ill-advised to not at least mention Python’s other web-scraping behemoth, [**Scrapy](https://scrapy.org/)**. **BeautifulSoup** and **Scrapy** have two very different agendas. BeautifulSoup is intended to parse or extract data one page at a time, with each page being served up via the **requests** library or equivalent. **Scrapy,** on the other hand, is for creating crawlers: or rather absolute monstrosities unleashed upon the web like a swarm, loosely following links and haste-fully grabbing data where data exists to be grabbed. To put this in perspective, Google Cloud functions will not even let you import Scrapy as a usable library.

This isn’t to say that **BeautifulSoup** can’t be made into a similar monstrosity of its own. For now, we’ll focus on a modest task: generating link previews for URLs by grabbing their metadata.

## Step 1: Stalk Your Prey

Before we steal any data, we should take a look at the data we’re hoping to steal.

    import requests
    from bs4 import BeautifulSoup

    def scrape(url):
        """Scrape URLs to generate previews."""
        headers = requests.utils.default_headers()
        headers.update({
            'User-Agent': 'Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0',
        })
        r = requests.get(url, headers)
        raw_html = r.content
        soup = BeautifulSoup(raw_html, 'html.parser')
        print(soup.prettify())

The above is the minimum needed to retrieve the DOM structure of an HTML page. **BeautifulSoup** accepts the .content output from a request, from which we can investigate the contents.

Using BeauitfulSoup will often result in different results for your scaper than you might see as a human, such as 403 errors or blocked content. An easy way around this faking your headers into looking like normal browser agents, as we do here: 
headers.update({ 'User-Agent': 'Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0', })`

The result of print(soup.prettify()) will predictably output a "pretty" printed version of your target DOM structure:

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

After turning our request content into a BeautifulSoup object, we access items in the DOM via dot notation as such:

    title = soup.title.string

.string gives us the actual content of the tag which is Example Domain, whereas soup.title would return the entirety of the tag as <title>Example Domain</title>.

Dot notation is fine when pages have predictable hierarchies or structures, but becomes much less useful for extracting patterns we see in the document. soup.a will only return the first instance of a link, and probably isn't what we want.

If we wanted to extract *all* <a> tags of a page's content while avoiding the noise of nav links etc, we can use CSS selectors to return a list of all elements matching the selection. soup.select('body p > a') retrieves all links embedded in paragraph text, limited to the body of the page.

Some other methods of grabbing elements:

* **soup.find(id=”example”)**: Useful for when a single element is expected.

* **soup.find_all(‘a’)**:** **Returns a list of all elements matching the selection after searching the document recursively.

* **.parent **and **.child**: Relative selectors to a currently engaged element.

## Get Some Attributes

Chances are we’ll almost always want the contents or the attributes of a tag, as opposed to the entire <a> tag's HTML. A common example of going after a tag's attributes would be in the cases of img and a tags. Chances are we're most interested in the src and href attributes of such tags, respectively.

The .get method refers specifically to getting the value of attributes on a tag. For example, soup.find('.logo').get('href') would find an element with the class "logo", and return the URL to that image.

In our example of creating link previews, a good first source of information would obviously be the page’s meta tags: specifically the og tags they've specified to openly provide the bite-sized information we're looking for. Grabbing these tags are a bit more difficult to deal with:

    soup.find("meta", property="og:description").get('content')

Oh yeah, now that’s some ugly shit right there. Meta tags are especially interesting because they’re all uselessly dubbed ‘meta’, thus we need a second differentiator in addition to the tag name to specify *which *meta tag we care about. Only then can we bother to *get* the actual content of said tag.

## Step 3: Realizing Something Will Always Break

If we were to try the above selector on an HTML page which did not contain og:description, our script would break unforgivingly. Not only do we miss this data, but we miss out on everything entirely - this means we always need to build in a plan B, and at the very least deal with a lack of tag altogether.

It’s best to break out this logic one tag at a time. First, let’s look at an example for a base scraper with all the knowledge we have so far:

    import requests
    from bs4 import BeautifulSoup

    def scrape(url):
        """Scrape scheduled link previews."""
        headers = requests.utils.default_headers()
        headers.update({
            'User-Agent': 'Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0',
        })
        r = requests.get(url)
        raw_html = r.content
        soup = BeautifulSoup(raw_html, 'html.parser')
        links = soup.select('body p > a')
        previews = []
        for link in links:
            url = link.get('href')
            r2 = requests.get(url, headers=headers)
            link_html = r2.content
            embedded_link = BeautifulSoup(link_html, 'html.parser')
            link_preview_dict = {
                'title': getTitle(embedded_link),
                'description': getDescription(embedded_link),
                'image': getImage(embedded_link),
                'sitename': getSiteName(embedded_link, url),
                'url': url
                }
            previews.append(link_preview_dict)
            print(link_preview_dict)

Great — there’s a base function for snatching all links out of the body of a page. Ultimately we’ll create a JSON object for each of these links containing preview data, link_preview_dict.

To handle each value of our dict, we have individual functions:

    def getTitle(link):
        """Attempt to get a title."""
        title = ''
        if link.title.string is not None:
            title = link.title.string
        elif link.find("h1") is not None:
            title = link.find("h1")
        return title

    def getDescription(link):
        """Attempt to get description."""
        description = ''
        if link.find("meta", property="og:description") is not None:
            description = link.find("meta", property="og:description").get('content')
        elif link.find("p") is not None:
            description = link.find("p").content
        return description

    def getImage(link):
        """Attempt to get a preview image."""
        image = ''
        if link.find("meta", property="og:image") is not None:
            image = link.find("meta", property="og:image").get('content')
        elif link.find("img") is not None:
            image = link.find("img").get('href')
        return image

    def getSiteName(link, url):
        """Attempt to get the site's base name."""
        sitename = ''
        if link.find("meta", property="og:site_name") is not None:
            sitename = link.find("meta", property="og:site_name").get('content')
        else:
            sitename = url.split('//')[1]
            name = sitename.split('/')[0]
            name = sitename.rsplit('.')[1]
            return name.capitalize()
        return sitename

In case you’re wondering:

* **getTitle **tries to get the <title> tag, and falls back to the page's first <h1> tag (surprisingly enough some pages are in fact missing a title).

* **getDescription** looks for the OG description, and falls back to the content of the page’s first paragraph.

* **getImage **looks for the OG image, and falls back to the page’s first image.

* **getSiteName **similarly tries to grab the OG attribute, otherwise it does it’s best to extract the domain name from the URL string under the assumption that this is the origin’s name (look, it ain’t perfect).

## What Did We Just Build?

Believe it or not, the above is considered to be enough logic to be a paid service with a monthly fee. Go ahead and Google it; or better yet, just steal my source code entirely:

    import requests
    from bs4 import BeautifulSoup
    from flask import make_response

    def getTitle(link):
        """Attempt to get a title."""
        title = ''
        if link.title.string is not None:
            title = link.title.string
        elif link.find("h1") is not None:
            title = link.find("h1")
        return title

    def getDescription(link):
        """Attempt to get description."""
        description = ''
        if link.find("meta", property="og:description") is not None:
            description = link.find("meta", property="og:description").get('content')
        elif link.find("p") is not None:
            description = link.find("p").content
        return description

    def getImage(link):
        """Attempt to get image."""
        image = ''
        if link.find("meta", property="og:image") is not None:
            image = link.find("meta", property="og:image").get('content')
        elif link.find("img") is not None:
            image = link.find("img").get('href')
        return image

    def getSiteName(link, url):
        """Attempt to get the site's base name."""
        sitename = ''
        if link.find("meta", property="og:site_name") is not None:
            sitename = link.find("meta", property="og:site_name").get('content')
        else:
            sitename = url.split('//')[1]
            name = sitename.split('/')[0]
            name = sitename.rsplit('.')[1]
            return name.capitalize()
        return sitename

    def scrape(request):
        """Scrape scheduled link previews."""
        if request.method == 'POST':
            # Allows POST requests from any origin with the Content-Type
            # header and caches preflight response for an 3600s
            headers = {
                'Access-Control-Allow-Origin': '*',
                'Access-Control-Allow-Methods': 'POST',
                'Access-Control-Allow-Headers': 'Content-Type',
                'Access-Control-Max-Age': '3600'
            }
            request_json = request.get_json()
            target_url = request_json['url']
            headers.update({
                'User-Agent': 'Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0',
            })
            r = requests.get(target_url)
            raw_html = r.content
            soup = BeautifulSoup(raw_html, 'html.parser')
            links = soup.select('.post-content p > a')
            previews = []
            for link in links:
                url = link.get('href')
                r2 = requests.get(url, headers=headers)
                link_html = r2.content
                embedded_link = BeautifulSoup(link_html, 'html.parser')
                preview_dict = {
                    'title': getTitle(embedded_link),
                    'description': getDescription(embedded_link),
                    'image': getImage(embedded_link),
                    'sitename': getSiteName(embedded_link, url),
                    'url': url
                    }
                previews.append(preview_dict)
            return make_response(str(previews), 200, headers)
        return make_response('bruh pls', 400, headers)

*Originally published at [hackersandslackers.com](https://hackersandslackers.com/scraping-urls-with-beautifulsoup/) on November 11, 2018.*
