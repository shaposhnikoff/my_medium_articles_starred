
# Generate Wildcard SSL certificate using Let’s Encrypt/Certbot

From past few days or months, everyone on the World Wide Web is talking about SSL certificates and rushing to implement them. But Why? Cause recently google has announced, if your website, webpages or web-applications does not have SSL certificate then Chrome will label them as Non-Secured.

![](https://cdn-images-1.medium.com/max/2304/1*pFuY5Isg7EOGdYyHr0PZ1A.png)

In this blog will cover, how to generate a wildcard SSL certificate for your domain using Certbot. I am generating certificate for the domain **erpnext.xyz**

![](https://cdn-images-1.medium.com/max/2000/1*u3H8gEJcWCqmcAGEUS6frA.png)

## Step 1: Setup Pre-requisites

If you already have a droplet or a system then make sure your system have **Python 2.7 or 3** and **git** installed on it. As I am starting on fresh Ubuntu droplet, we have to setup the above pre-requisites.

    apt-get update
    apt-get install python-minimal
    python --version
    apt-get install git-core
    git --version

## Step 2: Setup Certbot

After setting up the pre-requisites, now will setup the Certbot via github.

    cd /opt
    git clone [https://github.com/certbot/certbot.git](https://github.com/certbot/certbot.git)
    cd certbot && ./certbot-auto

While installing the Certbot, I came across the error

    Traceback (most recent call last):
      File "/usr/lib/python3/dist-packages/virtualenv.py", line 2363, in <module>
        main()
      File "/usr/lib/python3/dist-packages/virtualenv.py", line 719, in main
        symlink=options.symlink)
      File "/usr/lib/python3/dist-packages/virtualenv.py", line 988, in create_environment
        download=download,
      File "/usr/lib/python3/dist-packages/virtualenv.py", line 918, in install_wheel
        call_subprocess(cmd, show_stdout=False, extra_env=env, stdin=SCRIPT)
      File "/usr/lib/python3/dist-packages/virtualenv.py", line 812, in call_subprocess
        % (cmd_desc, proc.returncode))
    OSError: Command /opt/eff.org/certbot/venv/bin/python2.7 - setuptools pkg_resources pip wheel failed with error code

After googling, I came to know, the error triggered due to improper locale variables. Set the locale variables and re-run.

    export LC_ALL="en_US.UTF-8"
    export LC_CTYPE="en_US.UTF-8"

You can also install the Certbot via apt installer.

    apt-get install letsencrypt

## Step 3: Generate The Wildcard SSL Certificate

Now with the help of Certbot will generate wildcard certificate for our test domain **erpnext.xyz**

![./certbot-auto certonly — manual — preferred-challenges=dns — email [saurabh@erpnext.com](mailto:saurabh@erpnext.com) — server [https://acme-v02.api.letsencrypt.org/directory](https://acme-v02.api.letsencrypt.org/directory) — agree-tos -d *.erpnext.xyz](https://cdn-images-1.medium.com/max/2832/1*9vI0_g8LzgmsJffnfyIIDw.png)*./certbot-auto certonly — manual — preferred-challenges=dns — email [saurabh@erpnext.com](mailto:saurabh@erpnext.com) — server [https://acme-v02.api.letsencrypt.org/directory](https://acme-v02.api.letsencrypt.org/directory) — agree-tos -d *.erpnext.xyz*

**Note: As we are generating wildcard ssl certificate, mention domain with * i.e. *.erpnext.xyz**

## Step 4: Authenticate The Domain’s Ownership

For wildcard certificates, the only challenge method Let’s Encrypt accepts is the DNS challenge, which we can invoke via the **preferred-challenges=dns** flag.

After executing the above command, the Certbot will share a text record to add to your DNS.

    Please deploy a DNS TXT record under the name
    _acme-challenge.erpnext.xyz with the following value:

    J50GNXkhGmKCfn-0LQJcknVGtPEAQ_U_WajcLXgqWqo

**Record Name**: _acme-challenge
**Record Value**: J50GNXkhGmKCfn-0LQJcknVGtPEAQ_U_WajcLXgqWqo

Create TXT record via DNS console and setup key and value

![](https://cdn-images-1.medium.com/max/4032/1*5RkAPwZnWoCbjGbmY2-u2w.png)

## Step 5: Get The Certificate

Once you authenticate the domain ownership; by cleaning up dns challenges, Certbot generates the ssl certificate and required keys.

![Congratulations!!! You have wildcard SSL certificate](https://cdn-images-1.medium.com/max/2072/1*fQdFw-Tf-5rfERXBbJYgPA.png)*Congratulations!!! You have wildcard SSL certificate*

Congratulations!!! You have successfully generated wildcard SSL certificate for your domain.

## Step 6: Cross Verify The Certificate

To cross verify certificate’s validity via command line run

    ./certbot-auto certificates

![](https://cdn-images-1.medium.com/max/2320/1*CaHZSurriKEbcnco4bMHvA.png)
