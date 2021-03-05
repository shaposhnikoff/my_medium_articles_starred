
# Filtering HTTP and HTTPS traffic using Squid proxy in GCP

Recently, we have configured Squid proxy for one of our customers on GCP. During the installation and configuration, We faced a lot of issues and most of the posts on the internet did not have enough information and it was not in one place. So, we thought of writing this post. In this post, we are going to explain, how to install squid proxy on the ubuntu 18.04 VM and configure it to filter the HTTP and HTTPS requests. Let begin!!!

## What is Squid-Proxy

Squid is a Unix-based **proxy server** that caches Internet content closer to a requestor than its original point of origin as well as it will filter the traffic based on the configuration. Squid supports the caching of many different kinds of Web objects, including those accessed through **HTTP** and **FTP**. Caching frequently requested Web pages, media files, and other content accelerates response time and reduces bandwidth congestion.

![](https://cdn-images-1.medium.com/max/2560/1*U10n1efAU8YR18CI4bGvJg.png)

## Squid-proxy Installation steps.

Squid-proxy Version 3.5.27 does not come with OpenSSL dependencies and it’s not enabled SSL by default. We need to compile it manually in order to enable SSL.

## Step -1: Install the necessary dependencies for Squid proxy.

Run the below commands to install dependencies

    apt-get install dpkg-dev libldap2-dev libpam0g-dev libdb-dev cdbs libsasl2-dev debhelper libcppunit-dev libkrb5-dev comerr-dev libcap2-dev libecap3-dev libexpat1-dev libxml2-dev autotools-dev libltdl-dev pkg-config libnetfilter-conntrack-dev nettle-dev libgnutls28-dev libssl1.0-dev dh-apparmor ssl-cert libcrypt-openssl-x509-perl

## Step -2: enable the apt source repo (/etc/apt/sources.list)

Generally, The Source repository will be commented on sources file in /etc/apt/sources.list

In order to get the source package, we need to enable the source repository. The source repository highlighted in the below example file with bold.

This is an example file.

    ## Note, this file is written by cloud-init on first boot of an instance

    ## modifications made here will not survive a re-bundle.

    ## if you wish to make changes you can:

    ## a.) add ‘apt_preserve_sources_list: true’ to /etc/cloud/cloud.cfg

    ## or do the same in user-data

    ## b.) add sources in /etc/apt/sources.list.d

    ## c.) make changes to template file /etc/cloud/templates/sources.list.tmpl

    # See [http://help.ubuntu.com/community/UpgradeNotes](http://help.ubuntu.com/community/UpgradeNotes) for how to upgrade to

    # newer versions of the distribution.

    deb [http://asia-south1-c.gce.clouds.archive.ubuntu.com/ubuntu/](http://asia-south1-c.gce.clouds.archive.ubuntu.com/ubuntu/) bionic main restricted

    **deb-src [http://asia-south1-c.gce.clouds.archive.ubuntu.com/ubuntu/](http://asia-south1-c.gce.clouds.archive.ubuntu.com/ubuntu/) bionic main restricted**

    ## Major bug fix updates produced after the final release of the

    ## distribution.

    deb [http://asia-south1-c.gce.clouds.archive.ubuntu.com/ubuntu/](http://asia-south1-c.gce.clouds.archive.ubuntu.com/ubuntu/) bionic-updates main restricted

    **deb-src [http://asia-south1-c.gce.clouds.archive.ubuntu.com/ubuntu/](http://asia-south1-c.gce.clouds.archive.ubuntu.com/ubuntu/) bionic-updates main restricted**

    ## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu

    ## team. Also, please note that software in universe WILL NOT receive any

    ## review or updates from the Ubuntu security team.

    deb [http://asia-south1-c.gce.clouds.archive.ubuntu.com/ubuntu/](http://asia-south1-c.gce.clouds.archive.ubuntu.com/ubuntu/) bionic universe

    **deb-src [http://asia-south1-c.gce.clouds.archive.ubuntu.com/ubuntu/](http://asia-south1-c.gce.clouds.archive.ubuntu.com/ubuntu/) bionic universe**

    deb [http://asia-south1-c.gce.clouds.archive.ubuntu.com/ubuntu/](http://asia-south1-c.gce.clouds.archive.ubuntu.com/ubuntu/) bionic-updates universe

    **deb-src [http://asia-south1-c.gce.clouds.archive.ubuntu.com/ubuntu/](http://asia-south1-c.gce.clouds.archive.ubuntu.com/ubuntu/) bionic-updates universe**

    ## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu

    ## team, and may not be under a free licence. Please satisfy yourself as to

    ## your rights to use the software. Also, please note that software in

    ## multiverse WILL NOT receive any review or updates from the Ubuntu

    ## security team.

    deb [http://asia-south1-c.gce.clouds.archive.ubuntu.com/ubuntu/](http://asia-south1-c.gce.clouds.archive.ubuntu.com/ubuntu/) bionic multiverse

    **deb-src [http://asia-south1-c.gce.clouds.archive.ubuntu.com/ubuntu/](http://asia-south1-c.gce.clouds.archive.ubuntu.com/ubuntu/) bionic multiverse**

    deb [http://asia-south1-c.gce.clouds.archive.ubuntu.com/ubuntu/](http://asia-south1-c.gce.clouds.archive.ubuntu.com/ubuntu/) bionic-updates multiverse

    **deb-src [http://asia-south1-c.gce.clouds.archive.ubuntu.com/ubuntu/](http://asia-south1-c.gce.clouds.archive.ubuntu.com/ubuntu/) bionic-updates multiverse**

    ## N.B. software from this repository may not have been tested as

    ## extensively as that contained in the main release, although it includes

    ## newer versions of some applications which may provide useful features.

    ## Also, please note that software in backports WILL NOT receive any review

    ## or updates from the Ubuntu security team.

    deb [http://asia-south1-c.gce.clouds.archive.ubuntu.com/ubuntu/](http://asia-south1-c.gce.clouds.archive.ubuntu.com/ubuntu/) bionic-backports main restricted universe multiverse

    **deb-src [http://asia-south1-c.gce.clouds.archive.ubuntu.com/ubuntu/](http://asia-south1-c.gce.clouds.archive.ubuntu.com/ubuntu/) bionic-backports main restricted universe multiverse**

    ## Uncomment the following two lines to add software from Canonical’s

    ## ‘partner’ repository.

    ## This software is not part of Ubuntu, but is offered by Canonical and the

    ## respective vendors as a service to Ubuntu users.

    # deb [http://archive.canonical.com/ubuntu](http://archive.canonical.com/ubuntu) bionic partner

    **deb-src [http://archive.canonical.com/ubuntu](http://archive.canonical.com/ubuntu) bionic partner**

    deb [http://security.ubuntu.com/ubuntu](http://security.ubuntu.com/ubuntu) bionic-security main restricted

    **deb-src [http://security.ubuntu.com/ubuntu](http://security.ubuntu.com/ubuntu) bionic-security main restricted**

    deb [http://security.ubuntu.com/ubuntu](http://security.ubuntu.com/ubuntu) bionic-security universe

    **deb-src [http://security.ubuntu.com/ubuntu](http://security.ubuntu.com/ubuntu) bionic-security universe**

    deb [http://security.ubuntu.com/ubuntu](http://security.ubuntu.com/ubuntu) bionic-security multiverse

    **deb-src [http://security.ubuntu.com/ubuntu](http://security.ubuntu.com/ubuntu) bionic-security multiverse**

## Step — 3: Source the squid proxy in the opt folder.

Moving to the opt folder

    cd /opt/

Sourcing the Squid proxy from the repo.

    apt-get source squid

## Step — 4: Edit the Rules and control files to make the squid proxy to use SSL support

Once you source the squid proxy in /opt folder it will be creating the squid3–3.x.x folder.

The rules and control files are located inside that folder.

    /opt/squid3–3.x.x/debian/rules

    /opt/squid3–3.x.x/debian/control

Add the following lines in the rule file.

    — sbindir=/usr/sbin/ \
    — enable-ssl \
    — enable-ssl-crtd \
    — with-openssl

Take a look at the screenshot below

![](https://cdn-images-1.medium.com/max/2878/0*LrD7I4Akkcw6K1bg)

Kindly add the Build dependency in the control file on the 1st block.

![](https://cdn-images-1.medium.com/max/2886/0*77fKFeCodyxWVH7B)

Once updated come back to the /opt/squid3–3.x.x folder

## Step — 5: Compile the squid proxy

Run the following command to compile the squid proxy. It will take 20 min to compile the squid proxy. Recommended running the command on the screen.

    dpkg-buildpackage -rfakeroot -b

**Note: — Ignore the warning during the compilation.**

The command will create the following file in the /opt/ folder.

    squid_3.5.27–1ubuntu1.4_amd64.deb

    squid-common_3.5.27–1ubuntu1.4_all.deb

## Step — 6: Install the Squid proxy using dpkg

Run the following command to install the squid proxy with ssl.

    dpkg -i squid_3.5.27–1ubuntu1.4_amd64.deb

    squid-common_3.5.27–1ubuntu1.4_all.deb

## Step — 7: Run the following command to install the additional dependency (based on the requirement)

    apt-get install -f

## Step — 8: Generate the Self-signed Certificate.

Once the installation is done. We need to generate the Self-signed certificate to server the HTTPS request from the client machine.

Move the squid proxy installed folder.

    cd /etc/squid/

Create the SSL folder

    mkdir -p /etc/squid/ssl

Change the permission for ssl folder

    chmod -R 700 ssl

Move the SSL folder and generate the SSL

    cd ssl

    openssl req -new -newkey rsa:2048 -sha256 -days 365 -nodes -x509 -extensions v3_ca -keyout xxxxx.pem -out xxxxx.pem

    chmod 744 xxxx.pem

**Note: Kindly change -keyout and -out value based on your requirement.**

It will ask you to enter the company details and the domain details. Provide relevant details.

## Step -9 : Generate the .der file from .pem file

Once you create the SSL. Make out the .der file. Kindly run the following command for the same.

    openssl x509 -in xxxx.pem -outform DER -out xxxx.der

**Note: Change the -in and -out the value based on your requirement.**

**We will be importing this to our client machine browsers on the Root and Trusted Authentication block.**

## Step — 10: Generate the SSL_DB

In order to handle the HTTPS request, We need to create the SSL DB using ssl_cdtd tool

    /usr/lib/squid/ssl_crtd -c -s /var/lib/ssl_db -M 4MB

## Step — 11: Change the SSL_DB file owner

Squid proxy runs with its own user (Proxy). So In order to access the ssl_db file, we need to change the owner of the file

    chown -R proxy:proxy /var/lib/ssl_db

## Step — 12: Back the original squid proxy config and update our custom config

In order to handle the HTTPS request, we need to customize the squid proxy config. Kindly find the config file below.

    ### Backing up the Original file of squid proxy

    cp /etc/squid/squid.conf /etc/squid/squid.conf.org

    ### Null the squid.conf file

    > /etc/squid/squid.conf

Squid.conf

<iframe src="https://medium.com/media/8e174c953e72e64c3dee1e0567ae1cbc" frameborder=0></iframe>

**Note: Update the cert location and the side which you need to whitelist.**

**Create the whitelist.txt file in /etc/squid/ to whitelist the HTTP sides.**

**You can update the https side in the config itself.**

## Step 13: Validating the squid proxy config and restart

Run the following command to check the squid proxy configuration.

validate the config

    squid -k parse

restart the squid proxy

    service squid restart

Status check for squid proxy

    Service squid status

## Step 14: Adding the Iptables NAT rule to route the traffic via squid proxy to the internet(optional)

Our Client is on a private network and we need to route the traffic via NAT for the internet. What we have done is to make the Squid proxy instance as NAT instance.

Adding the NAT rule in the Iptables to route all incoming HTTP and https traffic to squid proxy port.

    sudo iptables -t nat -A PREROUTING -i ens4 -p tcp — dport 80 -j REDIRECT — to-port 3128 -m comment — comment “squid transparent https proxy”

    sudo iptables -t nat -A PREROUTING -i ens4 -p tcp — dport 443 -j REDIRECT — to-port 3129 -m comment — comment “squid transparent https proxy”

**Other Traffic**

    sudo iptables -t nat -A POSTROUTING -o ens4 -j MASQUERADE

Check the logs in the following location

    /var/log/squid/access.log — — Access log

    /var/og/squid/cache.log. — — Error log

That’s it. Now your squid proxy is live and handling the HTTP and HTTPS requests.

I hope you found this useful!
