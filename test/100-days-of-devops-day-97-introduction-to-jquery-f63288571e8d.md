
# 100 Days of DevOps — Day 97-Introduction to JQuery

Welcome to Day 97 of 100 Days of DevOps, Focus for today is Introduction to JQuery

* *jQuery is a Javascript Library.*

* *It is just a large single .js file that has many pre-built methods and objects that simplify our workflow.*

* *jQuery was created as a way to help simplify interactions with the DOM(when jQuery was created the conventional methods such as .querySelector() didn’t exist).*

* *One of the main features is the use of the $ sign.*

**Download [**http://jquery.com/download/](http://jquery.com/download/)

*If you don’t want to download jQuery, you can use the direct CDN link [https://code.jquery.com/](https://code.jquery.com/)*

*Actual jQuery Code [*https://code.jquery.com/jquery-3.1.1.js](https://code.jquery.com/jquery-3.1.1.js)

*Difference between using Vanilla vs jQuery*
> # //Vanilla
> # var testvar = document.querySelectorAll(‘p’)
> # //jQuery
> # var testvar = $(‘p’)

*So as we can see it saves lot of our typing works*

To confirm jQuery is loaded(type $ sign)

![](https://cdn-images-1.medium.com/max/2708/1*id6FmDWk-In7hzO6dTDE1g.png)

    <!DOCTYPE html>
    <html>
      <head>
        <meta charset="utf-8">
        <title>jQuery Introduction</title>
        <script
      src="https://code.jquery.com/jquery-3.1.1.js"
      integrity="sha256-16cdPddA6VdVInumRGo6IbivbERE8p7CQR3HzTBuELA="
      crossorigin="anonymous"></script>
      </head>
      <body>
     <h1>This is an Introduction to jQuery</h1>
     <ol>
       <li>First</li>
       <li>Second</li>
       <li>Third</li>
     </ol>
     <a href="https://code.jquery.com/">jQuery</a>
    

     <input type="text" name="Name" value="What is your name">
     <input type="button" name="ClickMe" value="ClickMe">
      </body>
    </html>

![Selecting via jQuery](https://cdn-images-1.medium.com/max/2740/1*sMU_Zais7GhXX8GBx500nw.png)*Selecting via jQuery*

*Now if I need to change the heading color/background it’s pretty straightforward in case of jQuery*

![](https://cdn-images-1.medium.com/max/2728/1*xJn8BTT5ANk1FaKoDk9Ipw.png)

*Now if I want to change multiple CSS properties at the same time, rather then passing one property at a time we can pass an object(which act as a dictionary).*

![](https://cdn-images-1.medium.com/max/5748/1*EtOhhD658ajwiSl7hBCGXw.png)

*Now as we see earlier we can grab multiple objects at once and if want to change color to red it will be applied to all items but what would be the case if I want to apply it only to a single item, to make it work we need to use **eq***

*NOTE: To grab the last item using negative indexing **eq(-1)***

![](https://cdn-images-1.medium.com/max/2712/1*V_uyAO24Rs0POnE7XOFEMg.png)

*Now if I want to change the text, it is pretty straightforward*

![](https://cdn-images-1.medium.com/max/2852/1*ETfz3ttV-zvichDk17CV5Q.png)

*If I want to change the style of my contents*

![](https://cdn-images-1.medium.com/max/2844/1*_Be__Pfx-lFzgCbTpam11w.png)

*Now if I want to change attributes*

![](https://cdn-images-1.medium.com/max/2876/1*NJQJjhbz9svQpfGXJ-ikjQ.png)

*Same way we can change values*

![](https://cdn-images-1.medium.com/max/2856/1*csKER0T6zVvS4M3q-7u_4Q.png)

*Adding and Removing a class*

![](https://cdn-images-1.medium.com/max/2864/1*qayh_bCb2KdyrzjgCdXMjw.png)

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
[**100 Days of DevOps — Day 96-*Document Object Model(DOM)***
Document Object Model(DOM)medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-96-document-object-model-dom-8860ea8018f7)
