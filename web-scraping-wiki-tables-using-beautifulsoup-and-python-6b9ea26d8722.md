
# Web Scraping Wikipedia Tables using BeautifulSoup and Python

Source: SixFeetUp

## *‘Data is the new oil’*

As an aspiring data scientist, I do a lot of projects which involve scraping data from various websites. Some companies like Twitter do provide APIs to get their information in a more organized way while we have to scrape other websites to get data in a structured format.

The general idea behind web scraping is to retrieve data that exists on a website and convert it into a format that is usable for analysis. In this tutorial, I will be going through a detail but simple explanation of how to scrape data in Python using [BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/). I will be scraping [Wikipedia](https://en.wikipedia.org/wiki/List_of_Asian_countries_by_area) to find out all the countries in Asia.

![Table with names of Asian countries on Wiki](https://cdn-images-1.medium.com/max/2000/1*0Uls6YZNTUwapvAv4uCeLw.png)*Table with names of Asian countries on Wiki*

Firstly we are going to** import requests** library. Requests allows you to send *organic, grass-fed* HTTP/1.1 requests, without the need for manual labor.

import requests

Now we assign the link of the website through which we are going to scrape the data and assign it to variable named** website_url**.

**requests.get(url).text** will ping a website and return you HTML of the website.
> website_url = requests.get(‘[https://en.wikipedia.org/wiki/List_of_Asian_countries_by_area](https://en.wikipedia.org/wiki/List_of_Asian_countries_by_area)[’).text](https://en.wikipedia.org/wiki/Premier_League).text)

We begin by reading the source code for a given web page and creating a BeautifulSoup (soup)object with the [BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/bs4/doc/) function. Beautiful Soup is a Python package for parsing HTML and XML documents. It creates a parse tree for parsed pages that can be used to extract data from HTML, which is useful for web scraping. Prettify() function in BeautifulSoup will enable us to view how the tags are nested in the document.
> from bs4 import BeautifulSoup
soup = BeautifulSoup(website_url,’lxml’)
print(soup.prettify())

![](https://cdn-images-1.medium.com/max/2000/1*hsZ0QrwflHUjYkYJxhEjxQ.png)

If you carefully inspect the HTML script all the table contents i.e. names of the countries which we intend to extract is under class Wikitable Sortable.

![](https://cdn-images-1.medium.com/max/2000/1*NyaaGqqHnemKSWu8DQqUHQ.png)

So our first task is to find class ‘wikitable sortable’ in the HTML script.
> My_table = soup.find(‘table’,{‘class’:’wikitable sortable’})

Under table class ‘wikitable sortable’ we have links with country name as title.

![](https://cdn-images-1.medium.com/max/2000/1*s6f-dPOmoNQVMe0yOqLV-w.png)

Now to extract all the links within <a>, we will use **find_all().**

![](https://cdn-images-1.medium.com/max/2000/1*nVwRAyrF5LLqwqOrxdwtmw.png)

From the links, we have to extract the title which is the name of countries.

To do that we create a list **Countries** so that we can extract the name of countries from the link and append it to the list countries.

![](https://cdn-images-1.medium.com/max/2000/1*qNhJCThsfkxZWHKiPq1RkQ.png)

Convert the list countries into Pandas DataFrame to work in python.

![](https://cdn-images-1.medium.com/max/2000/1*Sn71ZZa_pal70zK3Neki5w.png)

Thank you for reading my first article on Medium. I will make it a point to write regularly about my journey towards Data Science. Thanks again for choosing to spend your time here — means the world.

You can find my code on [Github](https://github.com/stewync/Web-Scraping-Wiki-tables-using-BeautifulSoup-and-Python/blob/master/Scraping%2BWiki%2Btable%2Busing%2BPython%2Band%2BBeautifulSoup.ipynb).
