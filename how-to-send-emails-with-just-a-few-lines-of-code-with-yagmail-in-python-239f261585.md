
# How to send emails with just a few lines of code with Yagmail in Python



On our last lesson, [How to send beautiful emails with attachments using only Python](https://letslearnabout.net/tutorial/how-to-send-beautiful-emails-with-attachments-using-only-python/), we built a script to send an HTML-based email with attachments.

If you are using a Gmail account, we can significantly simplify our code with [Yagmail](https://pypi.org/project/yagmail/), an STMP client for Gmail.

<center><iframe width="560" height="315" src="https://www.youtube.com/embed/D4dX1pueV54" frameborder="0" allowfullscreen></iframe></center>

## Setting up everything

We are going to send emails using a Gmail account. Before anything, create a Gmail account (or use your own) and enable the “Less secure app access” in your account. You can do it here: [https://myaccount.google.com/u/0/security?hl=en&pli=1](https://myaccount.google.com/u/0/security?hl=en&pli=1)

![](https://cdn-images-1.medium.com/max/2000/0*PPnXcsErGNG6W_sK)

After that, you are set! Create an environment with Python (I use pipenv, so I create one with* shell*), install yagmail with *pip install yagmail *and you are ready to go!

## Our basic script

When I said that Yagmail simplifies our work I wasn’t kidding. Let me show it to you.

First, create a Python file. Mine is *yagmail_sender.py*.

Now, write the following:

<iframe src="https://medium.com/media/cd96cd163725a4a6f4441f219ea02edd" frameborder=0></iframe>

This is so clear that it doesn’t need explanation.

But in case it does:

* As usual, we state our sender and receiver email. Also a subject and a password typed by the user.

* Then, we create a yagmail instance with the user and the password to log in.

* The last variable, *contents*, is a list of strings. This is the body of our email

* Then we send to the receiver *receiver_email* with the subject the email with the body that is listed on *contents*.

That’s it. That’s all you need:

![](https://cdn-images-1.medium.com/max/2000/0*t8JYukmfJj_UDukC)

## Sending an email to a list of people

Can you imagine how to send that email to a list of email address?

Replace the old lines with the new ones:

<iframe src="https://medium.com/media/d71e6117fd066af3bfc207ab3d9f064b" frameborder=0></iframe>

Notice the change from singular to plural.

Time to run the code again:

![](https://cdn-images-1.medium.com/max/2000/0*vbVYVxOeiTfeX5SI)

Every receiver on the list has received the email!
> Disclaimer: Even if it’s a list, it behaves like a set in Python. Repeated emails won’t be sent an email twice.

## Adding attachments

Sending attachments is incredible easy too. I don’t want people fighting on the comments, so in this one I’ll send two pictures: One of cats and another of dogs.

Add the desired files on the same root of your program and add the absolute path. Here’s my *contents*:

<iframe src="https://medium.com/media/6fa67cac1f10580882fced08735f61bd" frameborder=0></iframe>

I’m using a Windows OS, so I escaped the route with the backslash ( ‘\’).

That’s all we need. Seriously. Run the code and you’ll have two files attached to the email:

![](https://cdn-images-1.medium.com/max/2000/0*vgcX4sdSlgJm9Qud)

In case the code fails (we lose the internet connection, our password is wrong, etc), let’s wrap everything with a nice try/catch:

<iframe src="https://medium.com/media/08b978c09bca5315f711c149c3a6ea1d" frameborder=0></iframe>

Let’s introduce a wrong password:

![](https://cdn-images-1.medium.com/max/2000/0*7SnhQSd8bRWiYtBn)

Nice, now we get a text informing us that something went wrong and the message.

If you didn’t kn o w, this message is the same we can get when using the : Yagmail is just a wrapper around that library that simplifies the code greatly, as you just saw.

## Conclusion

Yagmail is a wrapper around the smtplib library that help us to write short, good code. The catch is that you can only use it with Gmail addresses.

But as everybody uses them, that won’t be a problem, right?

In case you are still using Hotmail or other email service, you can still use the *smtplib* Python library. Learn how to do it here:

[How to send beautiful emails with attachments (yes, cat pics too) using only Python](https://letslearnabout.net/tutorial/how-to-send-beautiful-emails-with-attachments-using-only-python/)

And yes, I know that after looking at the cats and dogs pics on the email attachment you want to see the whole picture, not the thumbnail. You deserved it:

![](https://cdn-images-1.medium.com/max/2000/0*cZYXztfRnj4lPRQh)

![](https://cdn-images-1.medium.com/max/2000/0*ty_2byKoWOtN9F6Y)

Yagmail package: [https://pypi.org/project/yagmail/](https://pypi.org/project/yagmail/)

Yagmail docs: [https://buildmedia.readthedocs.org/media/pdf/yagmail/latest/yagmail.pdf](https://buildmedia.readthedocs.org/media/pdf/yagmail/latest/yagmail.pdf)

[My Youtube tutorial videos](https://www.youtube.com/channel/UC9OLm6YFRzr4yjlw4xNWYvg?sub_confirmation=1) [Final code on Github](https://github.com/david1707/yagmail_sender) [Reach to me on Twitter](https://twitter.com/DavidMM1707) [Read more tutorials](https://letslearnabout.net/category/tutorial/)

*Originally published at [https://letslearnabout.net](https://letslearnabout.net/tutorial/how-to-send-easily-emails-with-yagmail/) on September 26, 2019.*
