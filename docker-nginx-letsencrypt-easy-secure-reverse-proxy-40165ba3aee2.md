
# How to set up an easy and secure reverse proxy with Docker, Nginx & Letsencrypt

Perfect score on SSL Labs

## Introduction

Ever tried setting up some sort of server at home? Where you have to open a new port for every service? And have to remember what port goes to which service, and what your home ip is? This is definitely something that works, and people have been doing it for the longest time.

However, wouldn’t it be nice to type *plex.example.com*, and have instant access to your media server? This is exactly what a reverse proxy will do for you, and combining it with Docker, it’s easier than ever.

## Prerequisites

### **Docker & Docker-Compose**

You should have Docker version 17.12.0+, and Compose version 1.21.0+.

### Domain

You should have a domain set up, and have an SSL Certificate associated with it. If you don’t have one, then [follow my guide here](https://medium.com/devopslinks/docker-letsencrypt-dns-validation-75ba8c08a0d) on how to get a free one with LetsEncrypt.

## What This Article Will Cover

I’m a firm believer in understanding what you are doing. There was a time where I would follow guides, and have no clue on how to troubleshoot failures. If that’s how you want to do it, [here’s a great tutorial](https://medium.freecodecamp.org/docker-compose-nginx-and-letsencrypt-setting-up-website-to-do-all-the-things-for-that-https-7cb0bf774b7e), which covers how to set it up. While my articles are lengthy, you should end up with an understanding of how it all works.

What you will learn here, is what a reverse proxy is, how to set it up, and how you can secure it. I do my best to divide the subject into sections, divided by headers, so feel free to jump over a section, if you feel like it. I recommend reading the entire article one time first, before starting to set it up.

## What is a Reverse Proxy?

### Regular Proxy

Let’s start with the concept of a regular proxy. While this is a term that’s very prevalent in the tech community, it is not the only place it’s used. A proxy means that information is going through a third party, before getting to the location.

Say that you don’t want a service to know your IP, you can use a proxy. A proxy is a server that has been set up specifically for this purpose. If the proxy server you are using is located in, for example, Amsterdam, the IP that will be shown to the outside world is the IP from the server in Amsterdam. The only ones who will know your IP are the ones in control of the proxy server.

### Reverse Proxy

To break it into simple terms, a proxy will add a layer of masking. It’s the same concept in a reverse proxy, except instead of masking outgoing connections (you accessing a webserver), it’s the incoming connections (people accessing your webserver) that will be masked. You simply provide a URL like *example.com*, and whenever people access that URL, your reverse proxy will take care of where that request goes.

Let’s say you have two servers set up on your internal network. Server1 is on *192.168.1.10*, and Server2 is on *192.168.1.20.* Right now your reverse proxy is sending requests coming from *example.com* to Server1. One day you have some updates to the webpage. Instead of taking the website down for maintenance, you just make the new setup on Server2. One you’re done, you simply change a single line in your reverse proxy, and now requests are sent to Server2. Assuming the reverse proxy is setup correctly, you should have absolutely no downtime.

But perhaps the biggest advantage of having a reverse proxy, is that you can have services running on a multitude of ports, but you only have to open ports 80 and 443, HTTP and HTTPS respectively. All requests will be coming into your network on those two ports, and the reverse proxy will take care of the rest. All of this will make sense once we start setting the proxy up.

## Setting Up the Container

### What to Do

<iframe src="https://medium.com/media/c722f328df40e2fac7a0662e20277784" frameborder=0></iframe>

First of all, you should add a new service to your docker-compose file. You can call it whatever you prefer, in this case I’ve chosen *reverse*. Here I’ve just chosen *nginx* as the image, however in a production environment, it’s usually a good idea to specify a version in case there are ever any breaking changes in future updates.

Then you should volume bind two folders. */etc/nginx* is where all your configuration files are stored, and */etc/ssl/private* is where your SSL certificates are stored. It is VERY important that your config folder does NOT exist on your host first time you’re starting the container. When you start your container through docker-compose, it will automatically create the folder and populate it with the contents of the container. If you have created an empty config folder on your host, it will mount that, and the folder inside the container will be empty.

### Why it Works

There isn’t much to this part. Mostly it’s like starting any other container with docker-compose. What you should notice here is that you are binding port 80 and 443. This is where all requests will come in, and they will be forwarded to whatever service you will specify.

## Configuring Nginx

### What to Do

Now you should have a config folder on your host. Changing to that directory, you should see a bunch of different files and a folder called conf.d. It’s inside conf.d that all your configuration files will be placed. Right now there’s a single default.conf file, you can go ahead and delete that.

Still inside conf.d, create two folders: sites-available and sites-enabled. Navigate into sites-available and create your first configuration file. Here we’re going to setup an entry for [Plex](https://plex.tv), but feel free to use another service that you have set up if you like. It doesn’t really matter what the file is called, however I prefer to name it like plex.conf.

Now open the file, and enter the following:

<iframe src="https://medium.com/media/2d5cab287394c0b04bae1fa5adf716d9" frameborder=0></iframe>

Go into the sites-enabled directory, and enter the following command:

    ln -s ../sites-available/plex.conf .

This will create a symbolic link to the file in the other folder. Now there’s only one thing left, and that is to change the nginx.conf file in the config folder. If you open the file, you should see the following as the last line:

    include /etc/nginx/conf.d/*.conf;

Change that to:

    include /etc/nginx/conf.d/sites-enabled/*.conf;

In order to get the reverse proxy to actually work, we need to reload the nginx service inside the container. From the host, run docker exec <container-name> nginx -t. This will run a syntax checker against your configuration files. This should output that the syntax is ok. Now run docker exec <container-name> nginx -s reload. This will send a signal to the nginx process that it should reload, and congratulations! You now have a running reverse proxy, and should be able to access your server at *plex.example.com *(assuming that you have forwarded port 80 to your host in your router).

Even though your reverse proxy is working, you are running on HTTP, which provides no encryption whatsoever. The next part will be how to secure your proxy, and get a perfect score on [SSL Labs](https://www.ssllabs.com/ssltest/analyze.html).

### Why it Works

**The Configuration File**

As you can see, the plex.conf file consists of two parts. An upstream part and a server part. Let’s start with the server part. This is where you are defining the port receiving the incoming requests, what domain this configuration should match, and where it should be sent to.

The way this server is being set up, you should make a file for each service that you want to proxy requests to, so obviously you need some way to distinguish which file to receive each request. This is what the server-name directive does. Below that we have the location directive.

In our case we only need one location, however you can have as many location directives as you want. Imagine you have a website with a frontend and a backend. Depending on the infrastructure you’re using, you’ll have the frontend as one container and the backend as another container. You could then have location / {} which will send requests to the frontend, and location /api/ {} which will send requests to the backend. Suddenly you have multiple services running on a single memorable domain.

As for the upstream part, that can be used for load-balancing. If you’re interested in learning more about how that works, you can look at the [official docs here](http://nginx.org/en/docs/http/ngx_http_upstream_module.html). For our simple case, you just define the hostname or ip address of the service you want to proxy to, and what port is should be proxied to, and then refer to the upstream name in the location directive.

**Hostname Vs. IP Address**

To understand what a hostname is, let’s make an example. Say you are on your home network *192.168.1.0.* You then set up a server on *192.168.1.10* and run Plex on it. You can now access Plex on *192.168.1.10:32400*, as long as you are still on the same network. Another possibility is to give the server a hostname. In this case we’ll give it the hostname *plex*. Now you can access Plex by entering *plex:32400 *in your browser!

This same concept was introduced to docker-compose in version 3. If you look at the docker-compose file earlier in this article, you’ll notice that I gave it a hostname: reverse directive. Now all other containers can access my reverse proxy by its hostname. One thing that’s very important to note, is that the service name has to be the same as the hostname. This is something that the creators of docker-compose chose to impose.

Another really important thing to remember, is that by default docker containers are put on their own network. This means that you won’t be able to access your container by it’s hostname, if you’re sitting on your laptop on your host network. It is only the containers that are able to access each other through their hostname.

So to sum it up and make it really clear. In your docker-compose file, add the hostname directive to your services. Most of the time your containers will get a new IP every time you restart the container, so referring to it via hostname, means it doesn’t matter what IP your container is getting.

**Sites-available & Sites-enabled**

Why are we creating the sites-available and sites-enabled directories? This is not something of my creation. If you install Nginx on a server, you will see that it comes with these folders. However because Docker is built with microservices in mind, where one container should only ever do one thing, these folders are omitted in the container. We’re recreating them again, because of how we’re using the container.

And yes, you could definitely just make a sites-enabled folder, or directly host your configuration files in conf.d. Doing it this way, enables you to have passive configuration laying around. Say that you are doing maintenance, and don’t want to have the service active; you simply remove the symbolic link, and put it back when you want the service active again.

**Symbolic Links**

Symbolic links are a very powerful feature of the operating system. I had personally never used them before setting up an Nginx server, but since then I’ve been using them everywhere I can. Say you are working on 5 different projects, but all these projects use the same file in some way. You can either copy the file into every project, and refer to it directly, or you can place the file in one place, and in those 5 projects make symlinks to that file.

This gives two advantages: you take up 4 times less space than you otherwise would have, and then the most powerful of them all; change the file in one place, and it changes in all 5 projects at once! This was a bit of a sidestep, but I think it’s worth mentioning.

## Securing Nginx Proxy

### What to Do

Go to your config folder, and create 3 files: ssl.conf common.conf common_location.conf. Fill them with the following input:

<iframe src="https://medium.com/media/da0061a8c9bcc7e5981957ed4c7ed181" frameborder=0></iframe>

<iframe src="https://medium.com/media/397280f159346bd80945768f678ac528" frameborder=0></iframe>

<iframe src="https://medium.com/media/aa3bfba55e69daf851bb1ddc8ac47085" frameborder=0></iframe>

Now open the plex.conf file, and change it to the following (notice lines 6, 9, 10 & 14):

<iframe src="https://medium.com/media/f01589b8da7775122c581d87c3d917d5" frameborder=0></iframe>

Now go back to the root of your config folder, and run the following command:

    openssl dhparam -out dhparams.pem 4096

This will take a long time to complete, even up to an hour in some cases.

If you followed my article on getting a LetsEncrypt SSL Certificate, your certificates should be located in </path/to/your/letsencrypt/config>/etc/letsencrypt/live/<domain>/ .

When I helped a friend set this up on his system, we ran into some problems where it couldn’t open the files when they were located in that directory. Most likely the cause of some permissions problems. The easy solution to this is to make an SSL directory, like </path/to/your/nginx/config>/certs, and then mount that to the Nginx container’s /etc/ssl/private folder. In the newly created folder, you should then make symbolic links, to the certs in your LetsEncrypt’s config folder.

When the openssl command is done running, you should run the docker exec <container-name> nginx -t to make sure that all the syntax is correct, and then reload it by running docker exec <container-name> nginx -s reload. At this point everything should be running, and you now have a working and perfectly secure reverse proxy!

### Why it Works

Looking in the plex.conf file, there is only one major change, and that is what port the reverse proxy is listening on, and telling it that it’s an ssl connection. Then there are 3 places where we’re including the 3 other files we made. While SSL is kind of secure by itself, these other files make it even more secure. However if for some reason you don’t want to include these files, you need to move the ssl-certificate and ssl-certificate-key inside the .conf file. These are required to have, in order for an HTTPS connection to work.

**Common.conf**

Looking in the common.conf file, we add 4 different headers. Headers are something that the server sends to the browser on every response. These headers tell the browser to act a certain way, and it is then up to the browser to enforce these headers.

[**Strict-Transport-Security (HSTS)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security)**

This header tells the browser that connections should be made over HTTPS. When this header has been added, the browser won’t let you make plain HTTP connection to the server, ensuring that all communication is secure.

[**X-Frame-Options](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options)**

When specifying this header, you are specifying whether or not other sites can embed your content into their sites. This can help avoid [clickjacking](https://en.wikipedia.org/wiki/Clickjacking) attacks.

[**X-Content-Type-Options](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Content-Type-Options)**

Say you have a site where users can upload files. There’s not enough validation on the files, so a user successfully uploads a php file to the server, where the server is expecting an image to be uploaded. The attacker may then be able to access the uploaded file. Now the server responds with an image, however the file’s MIME-type is text/plain. The browser will ‘sniff’ the file, and then render the php script, allowing the attacker to do RCE (Remote Code Execution).

With this header set to ‘nosniff’, the browser will not look at the file, and simply render it as whatever the server tells the browser that it is.

[**X-XSS-Protection](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-XSS-Protection)**

While this header was more necessary in older browsers, it’s so easy to add that you might as well. Some XSS (Cross-site Scripting) attacks can be very intelligent, while some are very rudimentary. This header will tell browsers to scan for the simple vulnerabilities and block them.

**Common_location.conf**

**X-Real-IP**

Because your servers are behind a reverse proxy, if you try to look at the requesting IP, you will always see the IP of the reverse proxy. This header is added so you can see which IP is actually requesting your service.

**X-Forwarded-For**

Sometimes a users request will go through multiple clients before it reaches your server. This header includes an array of all those clients.

**X-Forwarded-Proto**

This header will show what protocol is being used between client and server.

**Host**

This ensures that it’s possible to do a reverse DNS lookup on the domain name. It’s used when the server_name directive is different than what you are proxying to.

**X-Forwarded-Host**

Shows what the real host of the request is instead of the reverse proxy.

**X-Forwarded-Port**

Helps identify what port the client requested the server on.

**Ssl.conf**

SSL is a huge topic in and of itself, and too big to start explaining in this article. There are many great tutorials out there on how SSL handshakes work, and so on. If you want to look into this specific file, I suggest looking at the protocols and ciphers being used, and what difference they make.

## Redirecting HTTP to HTTPS

The observant ones have maybe noticed that we are only ever listening on port 443 in this secure version. This would mean that anyone trying to access the site via *https://** would get through, but trying to connect through *http://** would just get an error. Luckily there’s a really easy fix to this. Make a file with the following contents:

<iframe src="https://medium.com/media/7f3184f0360487097423ed04dd876158" frameborder=0></iframe>

Now just make sure that it appears in your sites-enabled folder, and when you’ve reloaded the Nginx process in the container, all requests to port 80 will be redirected to port 443 (HTTPS).

## Final Thoughts

Now that your site is up and running, you can head over to [SSL Labs](https://www.ssllabs.com/ssltest/analyze.html) and run a test to see how secure your site is. At the time of writing this, you should get a perfect score. However there is a big thing to notice about that.

There will always be a balance between security and convenience. In this case the weights are heavily on the side of security. If you run the test on SSL Labs and scroll down, you will see there are multiple devices that won’t be able to connect with your site, because they don’t support new standards.

So have this in mind when you are setting this up. Right now I am just running a server at home, where I don’t have to worry about that many people being able to access it. But if you do a scan on Facebook, you’ll see they won’t have as great a score, however their site can be accessed by more devices.
