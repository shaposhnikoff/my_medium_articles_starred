Unknown markup type 10 { type: [33m10[39m, start: [33m94[39m, end: [33m98[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m12[39m, end: [33m16[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m12[39m, end: [33m16[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m31[39m, end: [33m33[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m33[39m, end: [33m37[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m113[39m, end: [33m117[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m85[39m, end: [33m101[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m290[39m, end: [33m294[39m }

# A Simple Productive Trick for Your Mac

A helpful every-day trick to automate opening your windows and apps

![Photo by [Radek Grzybowski](https://unsplash.com/@rgrzybowski?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/6330/0*o0BzIQ8KkTLPkWQU)*Photo by [Radek Grzybowski](https://unsplash.com/@rgrzybowski?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

Reading through stuff on the internet, I found an interesting command for the terminal called open. In the beginning, it didn’t feel like anything special. But I find myself using it all the time now. I find it so useful I thought of writing a whole short article about it!

So here it goes, (assuming you have some familiarity with the shell) go to your terminal and run the command below to see what we are dealing with.

    $ open

![Some of the options for the command in your terminal](https://cdn-images-1.medium.com/max/2904/1*Abk9ITNMVGKNkoChMQ_KKg.png)*Some of the options for the command in your terminal*

As you see, open has a lot of options that could be interesting to try out, but we will be jumping straight to what I find really useful.

### Opening your apps

You can use open with the flag -a to specify that you want to open an application. This means you can open any *.app* you have installed with this command!

Let’s first find out what you can open in this manner running:

    $ ls /Applications/

You should see a list of the apps that you have. Now try opening some of them from the command line.

For example, launch iTunes using open like this:

    $ open -a "iTunes.app"

![](https://cdn-images-1.medium.com/max/3352/1*7o6341jqo5LBx7fs17accQ.png)

Or launching Safari running:

    $ open -a "Safari.app"

![](https://cdn-images-1.medium.com/max/2596/1*qQfGjcWU0CqteusFrcZrmg.png)

Unsurprisingly, this does exactly what you’d expect. But why go through all this effort?

<iframe src="https://medium.com/media/2e7b3a500f0b0398e99b99410d49174e" frameborder=0></iframe>

That’s a good question! And it was only after a while that I realize that some of these applications can be opened with additional arguments.

In particular, you can open web browsers like Safari or Google Chrome specifying the URLs that you want, and have them all opened.

For example:

    $ open -a "Google Chrome.app" [https://www.youtube.com/](https://www.youtube.com/) [https://www.wikipedia.org/](https://www.wikipedia.org/) [https://www.reddit.com/](https://www.reddit.com/)

![](https://cdn-images-1.medium.com/max/3116/1*xcUiiEA55Jjf0aa_bOO0bA.png)

For opening code editors like Visual Studio Code or Atom, you can specify the file path that you want. For example, this will launch the editor at the specified folder:

    $ open -a "Visual Studio Code.app" <folder_name>

![](https://cdn-images-1.medium.com/max/3592/1*uA4vKa4jQhmRKURodhqKDQ.png)

And of course, you can also use normal shell expressions. For example, this will open the editor in your current working directory:

    $ open -a "Visual Studio Code.app" .

— Okay, cool. But having to manually enter URLs or file paths to open apps one by one surely doesn’t seem any helpful!

And I agree! It is only when you start chaining these commands into scripts that all of this becomes meaningful. open allows you to write handy scripts to automate your workflow. Just make a list of all the apps and websites that you need for a project, we’ll see how to make a script that launches them.

### Writing a handy script

Let’s pretend that you are writing a script to start working quickly in the morning (start_working.sh ). Make sure to write down all the programs and websites that you are constantly using. I put here an example just to give you an idea. It’ s really simple, just call them one by one with open.

<iframe src="https://medium.com/media/270e02303f4f53837b858f356cc7ac8b" frameborder=0></iframe>

Now save it and change the permissions so you can run the file as a program with:

    $ chmod +x start_working.sh

And you are ready to open all your stuff at once! Writing a couple of scripts in this way, allows you to move around your projects and mind settings with ease. Without all the pain of having to reopen all the windows and applications every time. Starting right where you left off, just by running your handy program:

    $ ./start_working

![Starting all your apps in a couple of seconds](https://cdn-images-1.medium.com/max/2444/1*0H6PPbSNSCnx45yiqj74Kw.png)*Starting all your apps in a couple of seconds*

### Thanks for reading

Sure this isn’t practical at all if you just need to open an app or two, but I think it’s a good trick if you constantly reopening a lot of windows and switching between different projects. Hopefully, you can find this useful.
