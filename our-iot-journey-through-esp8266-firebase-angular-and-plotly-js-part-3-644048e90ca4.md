Unknown markup type 10 { type: [33m10[39m, start: [33m107[39m, end: [33m115[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m58[39m, end: [33m66[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m175[39m, end: [33m195[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m83[39m, end: [33m103[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m65[39m, end: [33m73[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m44[39m, end: [33m50[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m34[39m, end: [33m44[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m49[39m, end: [33m58[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m80[39m, end: [33m86[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m10[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m10[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m10[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m10[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m178[39m, end: [33m229[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m246[39m, end: [33m267[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m10[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m27[39m, end: [33m33[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m56[39m, end: [33m61[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m10[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m32[39m, end: [33m41[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m9[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m9[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m23[39m, end: [33m28[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m35[39m, end: [33m41[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m294[39m, end: [33m299[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m316[39m, end: [33m325[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m348[39m, end: [33m357[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m369[39m, end: [33m378[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m381[39m, end: [33m391[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m48[39m, end: [33m57[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m9[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m169[39m, end: [33m177[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m236[39m, end: [33m244[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m299[39m, end: [33m300[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m305[39m, end: [33m306[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m19[39m, end: [33m27[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m209[39m, end: [33m217[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m40[39m, end: [33m60[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m89[39m, end: [33m102[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m32[39m, end: [33m45[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m235[39m, end: [33m245[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m41[39m, end: [33m49[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m42[39m, end: [33m50[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m55[39m, end: [33m75[39m }

# Post 3 of 3. Our IoT journey through ESP8266, Firebase and Plotly.js

Science moves forward :-)

## TL;DR

My Medium home page is [here](https://medium.com/@o.lourme).

This post stands in a series of 3 as detailed below. These 3 posts reflect the Project Architecture we chose to achieve a simple luminosity logging/live plotting project. We hope they will **help developers or students discovering ESP8266 chip and Firebase platform**. Moreover, we believe that the solution developed here can be an interesting alternative to the comprehensive **Google, AWS and Azure IoT solutions** [[link](https://cloud.google.com/solutions/iot/), [link](https://docs.aws.amazon.com/fr_fr/iot/latest/developerguide/what-is-aws-iot.html) and [link](https://docs.microsoft.com/fr-fr/azure/iot-hub/iot-hub-device-management-overview)] when we have to **manage in a simple way only a few connected devices**. (For instance, we won’t set up a MQTT broker neither deal with a registry.)

![Project Architecture — Numbers 1, 2, 3 follow posts numerotation](https://cdn-images-1.medium.com/max/2000/1*J0IGNrEAq3n8zxCZsLO5dQ.png)*Project Architecture — Numbers 1, 2, 3 follow posts numerotation*

**Post 1** [[link](https://medium.com/@o.lourme/our-iot-journey-through-esp8266-firebase-angular-and-plotly-js-part-1-a07db495ac5f)]: We investigate on how an **ESP8266** can regularly make acquisition of an analog data and push its 10-bit equivalent value to a **Firebase Realtime Database**. We finish with considerations on **ESP8266 power saving.**

**Post 2** [[link](https://medium.com/@o.lourme/our-iot-journey-through-esp8266-firebase-angular-and-plotly-js-part-2-14b0609d3f5e)]: We write a **Firebase Cloud Function** in Typescript language to **timestamp **the data just pushed to Firebase Realtime Database.

**Post 3** (this post): With a** **web app using **plotly.js library**, we lively plot in a browser the analog data versus time. The web app is hosted with **Firebase Hosting**.

***Post 3. A web app hosted by Firebase Hosting susbscribes to the data stream coming from a Firebase Realtime Database and plot it.***

## A) Where we left

At the end of Post 1 [[link](https://medium.com/@o.lourme/our-iot-journey-through-esp8266-firebase-angular-and-plotly-js-part-1-a07db495ac5f)], an ESP8266 regularly pushed the 10-bit luminosity measure it just made to the measures node of a Firebase Realtime Database:

![Firebase Console — Each pushed value of luminosity is associated with a unique PushID](https://cdn-images-1.medium.com/max/2000/1*6aFAQyXJo3nQ-0BUjf45Rw.gif)*Firebase Console — Each pushed value of luminosity is associated with a unique PushID*

At the end of Post 2 [[link](https://medium.com/@o.lourme/our-iot-journey-through-esp8266-firebase-angular-and-plotly-js-part-2-14b0609d3f5e)], each time a push was made to measures node, a Firebase Cloud Function associated a timestamp to that push. The obtained object was then pushed to timestamped_measures node of the Firebase Realtime Databse:

![Firebase Console — Cloud Function at work!](https://cdn-images-1.medium.com/max/2000/1*MSioPaOnDex_OP_0qWbS0w.gif)*Firebase Console — Cloud Function at work!*

Now, that would be nice if a **web app**, each time a timestamped measure is pushed to timestamped_measures node, could get the last 200 (for instance) timestamped measures in that node in order to replot measures versus timestamps. **Firebase SDK** makes that subscription really easy and [plotly](undefined) ( [https://plot.ly/javascript/](https://plot.ly/javascript/)) plots data with no pain!

Indeed, at the end of this third and last post describing our ESP8266-based project, we will get this:

![What we will achieve at the end of this post!](https://cdn-images-1.medium.com/max/2000/1*VqqpujaI-nPgRzWCquzO5w.gif)*What we will achieve at the end of this post!*

## B) Angular web app or “classical” web app?

When we hacked this small project (5 months ago), before writing the 3 related Medium posts, we used **Angular** framework [[link](https://angular.io/)] to write the web app. We believed we would use it in this last post of the series, that’s why it is mentioned in the 3 posts URLs. But, as we published our first (4 months ago) and second (2 months ago) Medium posts [[link](https://medium.com/@o.lourme/our-iot-journey-through-esp8266-firebase-angular-and-plotly-js-part-1-a07db495ac5f), [link](https://medium.com/@o.lourme/our-iot-journey-through-esp8266-firebase-angular-and-plotly-js-part-2-14b0609d3f5e)], it turned out that this project could be **a basis for students discovering IoT**, even if they were not initially developers. In that context, we cannot ask them to learn Angular before diving into the IoT topic, that would really be discouraging. So, as our project went eventually more newbies oriented, we will explain here how to write the web app “classically”, *i.e. *with **html**, **javascript** and **css** files. It’s not possible to change Medium posts URLs afterwards, that’s why “Angular” remains in them and we’re sorry for that. That’s not bad marketing!

*Note: *If you want to write the web app with Angular all the same, follow the excellent resource mentioned in the **Acknowledgments** section at the end of this post. We ourselves started with it.

## C) Project description and setup

*Note:* Each time there is an action to perform, it is preceeded by a lowercase letter like **a. b. c. **etc.

**a.** Head to the **project GitHub repository** to see the full source code of our web app: [https://github.com/olivierlourme/esp8266webapp](https://github.com/olivierlourme/esp8266webapp). There is even a link to a live demo.

**b.** Clone the repository in your development folder (or download it if you don’t have git installed):

    **c:\_APP>**git clone https://github.com/olivierlourme/esp8266webapp.git

**c. **Move to the just generated project directory and launch VS code over it:

    **c:\_APP>**cd esp8266webapp
    **c:\_APP\esp8266webapp>**code .

*Note:* In Post 2 [[link](https://medium.com/@o.lourme/our-iot-journey-through-esp8266-firebase-angular-and-plotly-js-part-2-14b0609d3f5e)], we had developed the cloud function in a esp8266 folder. We could develop our web app in the same folder as it is just another feature of the project. But to keep presentation clear and posts independent, we chose another folder for the web app.

*Note: *The files of the web app are all in a public folder in order to prepare **Firebase Hosting** detailed at the end of this post.

Let’s briefly explain the role of index.html and script.js files present in the public folder. We won’t go much into details as **the source code is fully commented on the github repository**.

index.html:

* index.html includes the **Firebase SDK** scripts that allows manipulation of the Firebase Realtime Database of the project. For details, see [https://firebase.google.com/docs/web/setup](https://firebase.google.com/docs/web/setup).

    <!-- Firebase App is always required and must be first -->
    <script src="[https://www.gstatic.com/firebasejs/5.5.9/firebase-app.js](https://www.gstatic.com/firebasejs/5.5.9/firebase-app.js)"></script>
    <!-- Add additional services that you want to use -->
    <script src="[https://www.gstatic.com/firebasejs/5.5.9/firebase-database.js](https://www.gstatic.com/firebasejs/5.5.9/firebase-database.js)"></script>

* index.html includes the **javascript plotting library** named **plotly.js**. This powerful library, built on top of [d3.js](https://d3js.org/) and [stack.gl](https://github.com/stackgl), makes insertion of sophisticated charts inside webpages really easy. See [https://plot.ly/javascript/](https://plot.ly/javascript/)

    <!-- Include Plotly.js -->
    <script src="[https://cdn.plot.ly/plotly-latest.min.js](https://cdn.plot.ly/plotly-latest.min.js)"></script>

* index.html includes **moment.js library**. This library can convert timestamps expressed in Epoch time (milliseconds) to human readable dates. See [https://momentjs.com](https://momentjs.com/). For example: moment(1543658683000).format('YYYY-MM-DD HH:mm:ss') would produce 
 '2018–12–01 11:04:43' for a computer with a GMT+01:00 timezone.

    <!-- Include the moment.js library -->
    <script src="[https://momentjs.com/downloads/moment.js](https://momentjs.com/downloads/moment.js)"></script>

* index.html has then in its <body> section an identified <div> where the plot will be placed.

    <!-- Plotly chart will be drawn inside this div. -->
    <div id="myPlot" style="width: 100vw; max-height:75vh"></div>

* index.html includes at last the script.js script containing the essential part of the work.

    <!-- Include the script describing the work to do -->
    <script src="script.js"></script>

script.js:

* script.js defines in a const named config the **Firebase Project Configuration**. It doesn’t contain any sensitive information, it is a way for the web app to identify the Firebase project on Google servers. Anyway, it shouldn’t be exposed unnecessarily (for instance, if we use git, we place this const in a file named config.js, we reference it from script.js and we add config.jsto .gitignore).

**d.** To grab this Firebase Project Configuration, follow the 4 steps below and copy what’s in “5”:

![Firebase Console — Obtaining Firebase Project Configuration](https://cdn-images-1.medium.com/max/2000/1*MbKLLdH9Uus_BT79UV4hxQ.png)*Firebase Console — Obtaining Firebase Project Configuration*

**e.** Paste these informations to the beginning of script.js. You should end up with something like this:

    const config = {
      apiKey: "<API_KEY>",
      authDomain: "<PROJECT_ID>.firebaseapp.com",
      databaseURL: "https://<DATABASE_NAME>.firebaseio.com",
      projectId: "<PROJECT_ID>",
      storageBucket: "<BUCKET>.appspot.com",
      messagingSenderId: "<SENDER_ID>",
    };

* script.js then “subscribes” to the value changes in the Firebase Realtime Database node we want to plot: When a change occurs (*i.e.* a push to the node), an array of the nbOfElts **last values** of timestamp is made and also an array of the nbOfElts **last values** of luminosity. Those arrays then feed the x and y part of the plotly trace.

    firebase.database()
      .ref('timestamped_measures')
      .limitToLast(nbOfElts) // gives the sliding effect!
      .on('value', ts_measures => {...

*Note: *The value of nbOfElts is up to you. For instance, if you have a delay of 5 minutes between measures, it makes 12 measures per hour. If you want a sliding plot covering a whole day, you need to assign to nbOfElts the value 24*12 = 288.

*Note:* Our Firebase web app is made synced with the database since it connects to it through what we call **WebSockets **[[link](https://en.wikipedia.org/wiki/WebSocket)]** **and not normal HTTP.

**f. **Adapt **Firebase Realtime Database rules**

It’s time to adapt rules that grant access to nodes of our database. At the end of Post 2 [[link](https://medium.com/@o.lourme/our-iot-journey-through-esp8266-firebase-angular-and-plotly-js-part-2-14b0609d3f5e)], we had this:

    {
      "rules": {
        "measures": {
          ".read": "false",
          ".write": "false" 
        },
        "timestamped_measures": {
          ".read": "false",
          ".write": "false" 
        }
      }
    }

But now, as we want our web app to read timestamped_measures node, with no kind of authentication, the rules have to be changed to this:

    {
      "rules": {
        "measures": {
          ".read": "false",
          ".write": "false" 
        },
        "timestamped_measures": {
         ** ".read": "true",**
          ".write": "false" 
        }
      }
    }

## D) Starting a development server, configuring Firebase Hosting

### D) 1) Description

We’ll cover two topics here:

* start a **development server** to test locally our web app while improving it,

* configure **Firebase Hosting** in order to have quickly our app hosted on a Google server.

In fact, these topics are linked: As we will configure the use of a Firebase service (here the hosting one), we will benefit from the development server.

![Firebase Hosting logo](https://cdn-images-1.medium.com/max/2000/1*0QDB-Fz8wElUwT41nSRLQQ.png)*Firebase Hosting logo*

**a. **To be sure to log-in to Firebase with the right account (*i.e. *the one who created the esp8266-rocksproject), we type first:

    **c:\_APP\esp8266webapp>**firebase logout

Followed by:

    **c:\_APP\esp8266webapp>**firebase login

**b.** Once logged-in, we associate esp8266webapp folder with the Firebase services we want to use. Here we just want Firebase Hosting. Of course, we use also Firebase Realtime Database in this project, but it has already been included in index.html. So the command is:

    **c:\_APP\esp8266webapp>**firebase init hosting

**c.** A few questions are asked. Below, we put our responses in **bold**:

    ? Are you ready to proceed? **Yes
    **? Select a default Firebase project for this directory: **esp8266-rocks
    **? What do you want to use as your public directory? **public
    **? Configure as a single-page app (rewrite all urls to /index.html)? **No
    **? File public/index.html already exists. Overwrite? **No**

A few files are generated (and a 404 page as well):

![](https://cdn-images-1.medium.com/max/2000/1*BtZts2frIrFMEpRGOX7Pig.png)

### D) 2) Use of Firebase development server

**d.** Having initiated Firebase allows us to launch a **local development server**. We do it that way:

    **c:\_APP\esp8266webapp>**firebase serve --only hosting

**e.** We get the URL to test our web app and we paste it in a browser.

![](https://cdn-images-1.medium.com/max/2000/1*IZGgyD7yhTPCGKlNUSVvPQ.png)

Fantastic! The app works! And each time a new measure arrives, the plot moves to the left.

![The web app, locally tested](https://cdn-images-1.medium.com/max/2000/1*-hwA13v8hz0xnk8RjFEf6g.png)*The web app, locally tested*

*Note:* Now, we cannot have access anymore to the terminal that launched the Firebase development server because it is monopolized by this latter, displaying messages about the incoming connections and requests. If we need to stop the server, we hit CTRL + C. We can also open another terminal if we still need the server to work.

*Note:* **After saving a change made in the source code, we have to refresh the browser to test it** because Firebase development server is not a live server like this one : [[link](https://www.npmjs.com/package/live-server)].

### D) 3) Use of Firebase Hosting

When we’re satisfied with our development, it’s time to host the web app.

**f. **After hitting CTRL+C to stop Firebase development server, we just have to type this command:

    **c:\_APP\esp8266webapp>**firebase deploy --only hosting

After a few seconds, deploy is complete and we get the public URL of our web app: [https://esp8266-rocks.firebaseapp.com](https://esp8266-rocks.firebaseapp.com) . We test it in a browser (you too!), it works perfectly!

![Hosting URL of the web app](https://cdn-images-1.medium.com/max/2000/1*77Q0WEWSfKYKJJNOdrcUig.png)*Hosting URL of the web app*

*Note: *If you have your own domain, you can connect your Firebase web app to it. See [[link](https://firebase.google.com/docs/hosting/custom-domain)].

### D) 4 ) How much data can be downloaded from the database?

With **Firebasebase “Spark” pricing plan** (the free one), up to 10 GB/month can be downloaded. This can be quickly reached if:

* We have many points in the plot, *i.e.* if nbOfElts is high.

* We have a high refresh rate for it.

* Many people are connected to our web app!

To know the current downloaded amount of data, we hit **Project Overview** in Firebase Console:

![Firebase Console — Project Overview](https://cdn-images-1.medium.com/max/2000/1*iJHD5-c9eccbYKSC_B-tXQ.png)*Firebase Console — Project Overview*

## Conclusion

**About this post**

That’s all for today! We now have a web app basis to monitor data. But it can be improved in many ways. The first thing to do is to implement **Firebase Authentication** [[link](https://firebase.google.com/docs/auth/)] and roles to handle user/admin access to the web app. Database rules should then be updated adequatly. With authentication feature, the web app could:

* have a form element for the **admin **to **choose the refresh period** (that could be a Firebase Realtime Database node written by the web app and read by ESP8266 after each push),

* have a button for the **admin** to **delete the** measures **and **timestamped_measures **nodes**,

* have a form element for **users **to **set the number of last elements to plot.**

Then we could consider the **sending of notifications **to some registered users. For instance a user could receive a message if luminosity goes below a personalized threshold. This is possible with **Firebase Cloud Messaging**.

We have already implemented both aspects (authentication/notifications) in other contexts and this was really immediate.

**Acknowledgments for this post: **We’d like to thank here @angularfirebase ([Jeff Delaney](undefined)) whose videos-lessons are great quality content. Especially, **Episode #39 “Realtime Charts With Plot.ly”** [[link](https://angularfirebase.com/lessons/realtime-charts-with-plot-ly/)] made us discover plotly.js in an Angular context and gave us the idea to lively plot measures made by an ESP8266.

**About this 3-post series**

With this third and last post about ESP8266-Firebase topic, we now have the full picture…

* Personally, our system (ESP8266 — Cloud Function — Web App) has been working like a charm for days and days. ESP8266 is supplied with a 10.000 mAh powerbank and just pushes a measure once in a while. It remains in deep sleep mode the rest of the time. Moreover, as timestamps are delegated to a cloud function, at the end ESP8266 **consumes really little power**.

* Concerning **security**, though “secret” authentication to Firebase is not perfect, it does the job. The absence of an easy flash memory encryption is the only serious drawback we will regret (cf. post 1, [[link](https://medium.com/@o.lourme/our-iot-journey-through-esp8266-firebase-angular-and-plotly-js-part-1-a07db495ac5f)]).

* UPDATE (March 15, 2019). To go further and build a more professional, scalable and secure solution, read our next Medium post called [**GCP-Cloud IoT Core with ESP32 and Mongoose OS](https://medium.com/@o.lourme/gcp-cloudiotcore-esp32-mongooseos-1st-5c88d8134ac7?source=---------4------------------)**.

**About writing for Medium**

**Writing for Medium **for the first time was a positive experience. We didn’t realize at first the long process it was to write, choose pictures, refactor source code and comments, etc. to seamlessly present our project. Hacking the whole project took us less than one day. Writing just **one **post for Medium was a matter of at least 2 full days! But it’s really rewarding when you see that people from many countries, even France :-), enjoyed it and give you positive feedback. And it opened interesting discussions with our job colleagues as well. So, for sure we’ll write some other posts in 2019. Stay tuned, and try yourself writing!

*Tip: *By default your Medium post title will influence the post URL. If your title is long, you’ll have a long URL for your post provoking sometimes display disagreements due to long URLs. To avoid that, before publishing you can personalize your post URL by selecting **Customize story link **after clicking on the 3 dots … (close to the bell).

We’ve been really happy to share this newly learned knowledge with you.

**Links to the posts of this series: [**Post 1](https://medium.com/@o.lourme/our-iot-journey-through-esp8266-firebase-angular-and-plotly-js-part-1-a07db495ac5f), [Post 2](https://medium.com/@o.lourme/our-iot-journey-through-esp8266-firebase-angular-and-plotly-js-part-2-14b0609d3f5e), Post 3, [GitHub](https://github.com/olivierlourme/esp8266webapp) and [Live Demo](https://esp8266-rocks.firebaseapp.com/).

**Check also this post:** [GCP-Cloud IoT Core with ESP32 and Mongoose OS](https://medium.com/@o.lourme/gcp-cloudiotcore-esp32-mongooseos-1st-5c88d8134ac7?source=---------4------------------).
