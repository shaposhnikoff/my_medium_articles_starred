Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m16[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m22[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m44[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m227[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m44[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m47[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m241[39m }

# Deploying Flask with Gunicorn 3



Application deployments often come with unexpected consequences and bugs. Deploying can be the step that makes or breaks a deadline, or makes or breaks an application. Many Data-Scientists dread this deployment, as it can be quite tedious, and requires skills that many of us aren’t necessarily experts in (Dev-Ops.) In my[ “ Soft Skills of Data Science” article](https://towardsdatascience.com/the-unspoken-data-science-soft-skills-cc836d51b73d?source=your_stories_page---------------------------), I mentioned how Dev-Ops skills are certainly practiced in some regard by a Data-Scientist. This is for good reason, as I find myself getting more excited about *“ SysAdmin activities.”*

One service I’m truly in love with is [**Linode** for Linux-based VPS hosting](https://towardsdatascience.com/linode-might-be-the-best-deployment-solution-ad8991282c32?source=your_stories_page---------------------------). Linode is great because it has the minimalism I want with the security and features that I love. So we’re going to start this off by assuming you have a domain you would like to host at (or an IP,) and have a Linode or similar virtual private server up and running, ready to SSH into.

## Get in there!

Step one of course is to know your host and root password. Your host will be an IPV4 address given to you by Linode, for example:

    48.87.54.01

In order to SSH into our server, we can use an SSH tool like PuTTy (mostly for Windows users,) or just use SSH through our terminal. Then it’s as simple as SSHing to your root user. In the off-chance that “ ssh: command not found” returns from this command, you might need to install openssh.

    ssh root@your.ip.address

Great! Now we’re in, and we have to create a new user! Of course, this will depend on your distro…

![](https://cdn-images-1.medium.com/max/2000/1*KhFrObx3QU2_i-wMPsxoeg.jpeg)

I’m going to be on Red Hat:

    useradd *username*

But if you chose Ubuntu:

    adduser username

Now if you’d like to make your new account have access to super user commands with their password:

    usermod -aG sudo username

## Setup Directories (NGINX)

After setting up your user account, you can login by using the “ login” command from root, or you can SSH into your user account by replacing root with your username:

    ssh username@your.ip.address

Step one is to be grabbing NGINX. You can think of NGINX as a big router for any incoming signals, NGINX makes it super easy to serve more than one domain off of a single website, and listen for ports. So open up your terminal and install it via your package manager.

    sudo dnf install nginx

The package managers for your operating systems (if you don’t know) are also here below. (Just run sudo {package manager} install nginx)

    Distro      |     Package Manager
    ---------------------------------
    Ubuntu      |        apt-get
    RedHat      |      dnf / yum
    Opensuse    |          man
    Arch        |         pacman

Now we need to have an established domain or location for www in an HTTP protocol. Whether that’s a purchased domain, or just the IP for your server doesn’t really matter all that much. Next, we’ll have to edit our configuration file for NGINX:

    sudo nano /etc/nginx/sites-enabled/flask_app

This will bring up a text editor, there is a chance that the file will be old, but most likely it will be new. The result should be something like this:

    **server** {
        **listen** 80;
        **server_name** your.ip.here;
    
        **location** / {
            **proxy_pass** http://127.0.0.1:8000;
            **proxy_set_header** Host $host;
            **proxy_set_header** X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }

Now we just need to unlink the default configuration file:

    sudo unlink /etc/nginx/sites-enabled/default

Now move up into the root directory (not your user home directory.)

    cd ..

Now we need to make a directory inside of var/www/your.ip.here.

    sudo mkdir var/www/your.ip.here

And then give ourselves superuser priveleges so we don’t need to use sudo to access our filesystem.

    sudo chown var/www/your.ip.here

And that’s basically it! Congratulations, your static files will officially work inside of this directory! But what about Flask? Well we’re one quick server setup away!

## Setting up the server (Gunicorn)

Clone, SSH, or Secure copy your files into the var/www directory we created in the filesystem setup. Now we need to pop back in and install Gunicorn 3.

    sudo dnf install gunicorn3

And do the same with supervisor (refer to the package manager chart like last time if you need it!)

    sudo dnf install supervisor

Now we have to make a supervisor, so get ready to open up some more text in Nano!

    sudo nano /etc/supervisor/conf.d/flask_app.conf

And configure it with your settings!:

    [program:flask_app]
    directory=/var/
    command=gunicorn3 -workers=3 flask_app:app
    autostart=true
    autorestart=true
    stopasgroup=true
    killasgroup=true
    stderr_logfile=/var/log/appname/lognameerr.log
    stdout_logfile=/var/log/appname/lognamestdout.log

Then make the directory for your logs (of course this isn’t entirely necessary, but is definitely good practice.)

    sudo mkdir /path/to/logs (/var/log for me)
    sudo touch outputpath
    sudo touch stderrpath

## Congratulations!

You’ve deployed your Flask app “ The old fashioned way!” This allows for a lot more expand ability, and versatility, especially compared to other alternatives like Heroku. I think this is a much fun way, but some might say it can be a little tedious. Of course these are not usually DS responsibilities, but if you ever find your team in a pickle, this will definitely be something you’re glad you learned!
