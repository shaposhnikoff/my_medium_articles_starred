
# 100 Days of DevOps — Day 96-Document Object Model(DOM)

Welcome to Day 96 of 100 Days of DevOps, Focus for today is Introduction to Document Object Model(DOM)

## Document Object Model(DOM)

*DOM act as an interface between JavaScript and HTML/CSS code on a webpage*

*Most Modern Web browsers will construct the DOM which basically means storing all the HTML tags as Javascript objects*

*To check the DOM of any website*
> # Go to console(In Chrome Browser → Right Click on WebPage → Inspect → Console)and type document

![document](https://cdn-images-1.medium.com/max/5756/1*2UFzCKQxzehHwSRMrBcULw.png)*document*

*This will return the HTML text of the page*

*To see actual objects*
> # console.dir(documents)

![console.dir(documents)](https://cdn-images-1.medium.com/max/5676/1*zROGodKqRNzq1oSzxs23JA.png)*console.dir(documents)*

As you can see DOM is enormous.

*How to grab HTML elements from the DOM*

*HTML elements are properties of the DOM*

*Now let see some important document attributes*
> # document.URL: Returns actual URL of the website
> # document.body: Returns everything inside body
> # document.head: Returns everything in the head of page
> # document.links: Return all the links on the page

*Let see some methods for grabbing elements from the DOM*
> # document.getElementById()
> # Returns elements based on whatever Id we passed
> # document.getElementByClassName()
> # Returns elements based on whatever classname we passed, and then it return lists of all the items belonging to that class name
> # document.getElementByTagName()
> # Return lists of all the items belonging to that tag name
> # document.querySelector()
> # Returns the first element matches the css style selector
> # document.querySelectorAll()
> # Returns all the elements matches the css style selector

*Now taking this code as an example let try to grab HTML elements*

    <!DOCTYPE html>
    <html>
      <head>
        <meta charset="utf-8">
        <title>DOM Interaction</title>
      </head>
      <body>
    <h1>Document Object Model(DOM) Interaction</h1>
    <p>How to grab HTML elements from the DOM</p>
    <p>HTML elements are properties of the DOM</p>
    <h5>Now let see some important document attributes</h5>
    <ul class="mytest">
      <li id="test">document.URL: Returns actual URL of the website</li>
      <li>document.body: Returns everything inside body</li>
      <li>document.head: Returns everything in the head of page</li>
      <li>document.links: Return all the links on the page</li>
    </ul>

    <h5>Let see some methods for grabbing elements from the DOM</h5>
    <ul class="mytest">
      <li>document.getElementById()
      Returns elements based on whatever Id we passed</li>
      <li>document.getElementByClassName()
      Returns elements based on whatever classname we passed, and then it return lists of all the items belonging to that class name</li>
      <li>document.getElementByTagName()
      Return lists of all the items belonging to that tag name</li>
      <li>document.querySelector() 
      Returns the first element matches the css style selector</li>
      <li>document.querySelectorAll()
      Returns all the elements matches the css style selector</li>
    </ul>
      </body>
    </html>

![document.URL](https://cdn-images-1.medium.com/max/2988/1*OqdrpZnK_3Kabie2baHivA.png)*document.URL*

*As you see it document.url returns the actual URL of the website*

*Now let see document.body*

![document.body](https://cdn-images-1.medium.com/max/2972/1*etX5_6qXYq8d1zaQdSbLNw.png)*document.body*

![document.head](https://cdn-images-1.medium.com/max/2980/1*QvPA_DYhzDZC2lUrst4J9Q.png)*document.head*

*Returns everything on the head page*

![document.links](https://cdn-images-1.medium.com/max/2952/1*hGRdRH4R-lPbfxv7dZ9IJA.png)*document.links*

*In case if there are links in our page*

*These **are all attributes, not methods** so we don’t need to put parentheses in front of it*

*Now looks at important/useful methods*

![document.getElementById](https://cdn-images-1.medium.com/max/2996/1*fsuoe_wDQew8rtV1RPU9MQ.png)*document.getElementById*

![document.getElementsByClassName](https://cdn-images-1.medium.com/max/2992/1*onfJ7SFhiZcd5xy3LoOabQ.png)*document.getElementsByClassName*

*NOTE: It’s spelled as Element**s(extra s)***

![document.getElememtsByTagName](https://cdn-images-1.medium.com/max/2996/1*5Zlfgu_AycQ8adi8EkUnMw.png)*document.getElememtsByTagName*

![document.querySelector/querySelectorAll](https://cdn-images-1.medium.com/max/2996/1*sI83bT9sXSZTxg8kB-WOJw.png)*document.querySelector/querySelectorAll*
> ***# for id***
> ***. for class***

*As mentioned earlier query selector allows us to grab things based on CSS selectors and save a lot of our time. So querySelectorAll returns all the objects whereas querySelector returns all object.*

*Now let see how it's helpful for us*

*Take a simple example where I want to change the Header color*

![Change Header Color](https://cdn-images-1.medium.com/max/2716/1*7dnTyyl_RhPUooqQjy3kpA.png)*Change Header Color*

*Let’s Dig further how to interact with the HTML from the DOM and change text, HTML code and attributes*

    <!DOCTYPE html>
    <html>
      <head>
        <meta charset="utf-8">
        <title>DOM Interaction</title>
      </head>
      <body>
    <h1>Document Object Model(DOM) Interaction</h1>
    <p>How to grab HTML elements from the DOM</p>
    <p>HTML elements are properties of the DOM</p>
    <h5>Now let see some important document attributes</h5>
    <ul>
      <li>document.URL: Returns actual URL of the website</li>
      <li>document.body: Returns everything inside body</li>
      <li>document.head: Returns everything in the head of page</li>
      <li>document.links: Return all the links on the page</li>
    </ul>

    <h5>Let see some methods for grabbing elements from the DOM</h5>
    <ul class="test">
      <li>document.getElementById()
      Returns elements based on whatever Id we passed</li>
      <li>document.getElementByClassName()
      Returns elements based on whatever classname we passed, and then it return lists of all the items belonging to that class name</li>
      <li>document.getElementByTagName()
      Return lists of all the items belonging to that tag name</li>
      <li>document.querySelector() 
      Returns the first element matches the css style selector</li>
      <li>document.querySelectorAll()
      Returns all the elements matches the css style selector</li>
      <a href="http://www.google.com">google</a>
    </ul>
      </body>
    </html>

![textContent](https://cdn-images-1.medium.com/max/5748/1*tIUpixw3aqmwUwYsjjvLTA.png)*textContent*

*What **textContent** will do is just return the **textContent***

![innerHTML](https://cdn-images-1.medium.com/max/2852/1*6m4JW7dsAQ3fE_mn4MoblA.png)*innerHTML*

*So now we need to change the style of content we need to use **innerHTML** which will return the actual html*

![getAttribute/setAttribute](https://cdn-images-1.medium.com/max/5740/1*GjajVEScl4oIsFS3WiHnvg.png)*getAttribute/setAttribute*

***getAttribute:** Returns the original attributes*

***setAttribute:** Allowed us to set attribute*

*So in the above example out of the class .test, first I want to grab URL(which is a part of anchor tag) and then I can update it using setAttribute.*

*Looking forward from you guys to join this journey and spend a minimum an hour every day for the next 100 days on DevOps work and post your progress using any of the below medium.*

* *Twitter: [@100daysofdevops](http://twitter.com/100daysofdevops) OR @[lakhera2015](https://twitter.com/lakhera2015)*

* *Facebook: [https://www.facebook.com/groups/795382630808645/](https://www.facebook.com/groups/795382630808645/)*

* *Medium: [https://medium.com/@devopslearning](https://medium.com/@devopslearning)*

* *Slack: [https://devops-myworld.slack.com/messages/CF41EFG49/](https://devops-myworld.slack.com/messages/CF41EFG49/)*

* *GitHub Link:[https://github.com/100daysofdevops](https://github.com/100daysofdevops)*

***Reference***
[**100 Days of DevOps — Day 0**
*D-day is just one day away and finally, this is a continuation of the post(I posted a month earlier)*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-0-4f2c9750542d)
[**100 Days of DevOps**
*Motivation*medium.com](https://medium.com/@devopslearning/100-days-of-devops-81faf13bf772)
[**100 Days of DevOps — Day 95-Introduction to Django**
*Welcome to Day 95 of 100 Days of DevOps, Focus for today is Introduction to Django*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-95-introduction-to-django-37942477d6c)
