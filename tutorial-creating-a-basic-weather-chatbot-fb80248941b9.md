
# Tutorial: Creating a Basic Weather Chatbot



## Do you want to develop a ChatBot but don’t know where to start?

## Sometimes the best way it’s just trying out,

You can start with something simple but functional and at the same time learn about the topic and also get ideas for your future projects.

## So let’s get started!

On this tutorial we are going to use Telegram as the chatbot platform and Python for a fast, powerful and easy development.

Ok so first we need to get a Token for our bot, to do that go to telegram and search for the “BotFather” and send the command /newbot

![](https://cdn-images-1.medium.com/max/2000/1*e1IOHmcUk1OIMwgSaZflVA.png)

It will ask you for a name and an Username for the Bot (You can’t repeat a username so you have to create a unique one, usually with Bot at the end), in the last question it will answer with the Token and a Url that takes you to the chat with your bot.

The token looks something like **123456:ABC-DEF1234ghIkl-zyx57W2v1u123ew11**, but we’ll simply use **<token>** to refer to it.

Now, copy the Token on a safe place, we will use it later.

If you don’t have Python installed on you computer then get it on [https://www.python.org/ ](https://www.python.org/)and then install it.

There are a lot of useful Python Telegram-Bot APIs you can find more information about it on:
[**python-telegram-bot/python-telegram-bot**
*python-telegram-bot - We have made you a wrapper you can't refuse*github.com](https://github.com/python-telegram-bot/python-telegram-bot)

But we are going to use this one since its very complete and easy to use:
[**datamachine/twx.botapi**
*twx.botapi - Unofficial Telegram Bot API Client*github.com](https://github.com/datamachine/twx.botapi)

First we have to install it with pip.

**pip install twx.botapi**

now we need a Weather API, we are going to use Open Weather Map:
[**csparpa/pyowm**
*pyowm - PyOWM - A Python wrapper around the OpenWeatherMap web API*github.com](https://github.com/csparpa/pyowm)

**pip install pyowm**

you need to get an api key here:
[**OpenWeatherMap. Application ID- OpenWeatherMap**
*Edit description*openweathermap.org](http://openweathermap.org/appid)

from now on we are going to refer to this api key as** <OWMKEY>**
 This Api will show us the weather, wind, temperature and other details about the location we’ll get from the user.

## Now we are ready to develop it!

    # -*- coding: utf-8 -*-
    import sys
    from time import sleep
    from twx.botapi import TelegramBot, ReplyKeyboardMarkup **#Telegram BotAPI**
    import traceback
    from pyowm import OWM **#Weather API**

    **"""
    Setup the bot and the Weather API
    """**
    TOKEN = <token>
    OWMKEY = <OWMKEY>

    bot = TelegramBot(TOKEN) 
    bot.update_bot_info().wait()  **#wait for a message**
    print bot.username 
    last_update_id = 0 
    def process_message(bot, u): **#This is what we'll do when we get a message** 
       ** #Use a custom keyboard **
        keyboard = [['Get Weather']] **#Setting a Button to Get the Weather **
        reply_markup = ReplyKeyboardMarkup.create(keyboard) **#And create the keyboard **
        if u.message.sender and u.message.text and u.message.chat: **#if it is a text message then get it **
            chat_id = u.message.chat.id 
            user = u.message.sender.username
            message = u.message.text 
            print chat_id 
            print message 
            if message == 'Get Weather': **#if the user is asking for the weather then we ask the location **
                bot.send_message(chat_id, 'please send me your location') 
            else: 
                bot.send_message(chat_id, 'please select an option', reply_markup=reply_markup).wait() **#if not then just show the options**
     
        elif u.message.location: **#if the message contains a location then get the weather on that latitude/longitude **
            print u.message.location 
            chat_id = u.message.chat.id 
            owm = OWM(OWMKEY) **#initialize the Weather API **
            obs = owm.weather_at_coords(u.message.location.latitude, u.message.location.longitude) **#Create a weather observation **
            w = obs.get_weather() **#create the object Weather as w **
            print(w) **# <Weather - reference time=2013-12-18 09:20, status=Clouds> **
            l = obs.get_location() **#create a location related to our already created weather object And send the parameters **
            status = str(w.get_detailed_status()) 
            placename = str(l.get_name()) 
            wtime = str(w.get_reference_time(timeformat='iso')) 
            temperature = str(w.get_temperature('celsius').get('temp'))
            bot.send_message(chat_id, 'Weather Status: ' +status +' At '+placename+' ' +wtime+' Temperature: '+ temperature+ 'C') **#send the anwser**
            bot.send_message(chat_id, 'please select an option', reply_markup=reply_markup).wait() **#send the options again**
        else: 
            print u bot.send_message(chat_id, 'please select an option', reply_markup=reply_markup).wait() 

    while True: **#a loop to wait for messages**
        updates = bot.get_updates(offset = last_update_id).wait() **#we wait for a message**
        try: 
            for update in updates: **#get the messages**
                if int(update.update_id) > int(last_update_id): **#if it is a new message then get it**
                    last_update_id = update.update_id 
                    process_message(bot, update) **#send it to the function** 
                    continue 
            continue 
        except Exception: 
            ex = None 
            print traceback.format_exc() 
            continue

You can now save it and run it

if everything went right you’ll see something like this:

![](https://cdn-images-1.medium.com/max/2000/1*GliZRAxNNBFP654H0eJePQ.jpeg)

Great! now is time to read a little bit more and add some pictures and interaction to it,** but for now we have our brand new bot working and being useful!**

If you want to dig a little bit more on this Bot API here the link to the documentation:
[**twx.botapi - Unofficial Telegram Bot API Client - TWX 1.0b3 documentation**
*Conversation the message belongs to - user in case of a private message, GroupChat in case of a group*pythonhosted.org](https://pythonhosted.org/twx/twx/botapi/botapi.html)

also some examples for the weather API:
[**csparpa/pyowm**
*pyowm - PyOWM - A Python wrapper around the OpenWeatherMap web API*github.com](https://github.com/csparpa/pyowm/blob/master/pyowm/docs/usage-examples.md)

Original post:
[**Tutorial: Creating a Basic Weather Chatbot**
*You can start with something simple but functional and at the same time learn about the topic and also get ideas for…*guatebot.com](http://guatebot.com/blog/?p=141)

<iframe src="https://medium.com/media/7078d8ad19192c4c53d3bf199468e4ab" frameborder=0></iframe>

![](https://cdn-images-1.medium.com/max/2000/1*bQlRSzFHJEmF4Q7PyrLgng.gif)

![](https://cdn-images-1.medium.com/max/2000/1*6XUspT9JOSq0w0Fi35HIaA.png)

![](https://cdn-images-1.medium.com/max/2000/1*c1LDMH5vbnIz9rmAka8Hwg.png)

![](https://cdn-images-1.medium.com/max/2000/1*D0Jf3dI6ZThtqcfwDYY7mg.png)
