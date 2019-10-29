Unknown markup type 10 { type: [33m10[39m, start: [33m50[39m, end: [33m55[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m79[39m, end: [33m91[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m334[39m, end: [33m354[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m28[39m, end: [33m39[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m90[39m, end: [33m110[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m25[39m, end: [33m36[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m31[39m, end: [33m43[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m101[39m, end: [33m112[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m81[39m, end: [33m96[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m130[39m, end: [33m141[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m8[39m, end: [33m19[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m57[39m, end: [33m70[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m128[39m, end: [33m139[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m38[39m, end: [33m49[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m47[39m, end: [33m67[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m136[39m, end: [33m147[39m }

# Vagrant Provisioning

Vagrant Provisioning

### Using Shell Provisioner to Configure Virtual Systems

One of the coolest and core features of [**Vagrant](https://www.vagrantup.com/)** is the ability to provision systems, or in other words configure systems to an initial state. [**Vagrant](https://www.vagrantup.com/)** can do this with through shell scripts, popular change configuration systems, or through containers with [**Docker](https://www.docker.com/)**.

This is a small overview of this system using the shell provisioner. We’ll run hello_web.sh installs a web server with some files on an Ubuntu 16.04 system.

## Prerequisites

This small tutorial requires that [**Vagrant](https://www.vagrantup.com/)** is installed along with virtual system [**VirtualBox](https://www.virtualbox.org/)**. These instructions use [**Curl](https://curl.haxx.se/) **and** [Bash](https://www.gnu.org/software/bash/)**, so if you do not have those, you’ll use whatever is equivalent in your shell environment. For example, Windows users would need to find the equivalent PowerShell commands, and utilize something like the System.Net.WebClient object in place of [**Curl](https://curl.haxx.se/)**.

![](https://cdn-images-1.medium.com/max/2658/1*f8l3kw0qKuOuGpC97HgirQ.png)

I created some previous guides on installing [**Vagrant](https://www.vagrantup.com/)** and other tools, that may be useful depending on your operating system:

### Installing Vagrant on macOS (using Homebrew)

A small guide that runs through using [**Homebrew](https://brew.sh/)** to install [**Vagrant](https://www.vagrantup.com/)** and [**VirtualBox](https://www.virtualbox.org/).**
[**VirtualBox and Friends on macOS**
*VirtualBox, Vagrant, Test Kitchen, Docker Machine, Minikube*medium.com](https://medium.com/@Joachim8675309/virtualbox-and-friends-on-macos-fd0b78c71a32)

### Installing Vagrant on Windows (using Chocolatey)

A small guide that runs through using [**Chocolatey](https://chocolatey.org/)** to install [**Vagrant](https://www.vagrantup.com/)** and [**VirtualBox](https://www.virtualbox.org/).**
[**VirtualBox and Friends on Windows 8.1**
*VirtualBox, Vagrant, Test Kitchen, Docker Machine, Minikube*medium.com](https://medium.com/@Joachim8675309/virtualbox-and-friends-on-windows-8-1-3c691460698f)

### Installing Vagrant on Fedora

A small guide that runs through installing [**Vagrant](https://www.vagrantup.com/)** and [**VirtualBox](https://www.virtualbox.org/)** on [**Fedora](https://getfedora.org/)**.
[**Vagrant and Friends on Fedora 28**
*VirtualBox, Vagrant, Test Kitchen, Docker Machine, Minikube*medium.com](https://medium.com/@Joachim8675309/vagrant-and-friends-on-fedora-28-37b8cbc47e47)

## Part I: Provisioning Ubuntu Systems

![**Ubuntu 16.04 Xenial Xerus**](https://cdn-images-1.medium.com/max/2000/1*iT9vgEiG0OaGM1o2QekfVQ.png)***Ubuntu 16.04 Xenial Xerus***

For this part, we’ll create provisioning script for Ubuntu 16.04 Xenial Xerus.

### Vagrant Structure

To get started we need to create a working directory and support files:

    **WORKAREA**=**${WORKAREA**:-"**${HOME}**/vagrant-shell"**}**

    mkdir -p **${WORKAREA}**/scripts
    touch **${WORKAREA}**/{Vagrantfile,scripts/hello_web.sh}

    cd **${WORKAREA}**

This will create a base line structure that we’ll use to create a virtual development environment, in which to develop provisioning scripts.

    ~/vagrant-shell
    ├── Vagrantfile
    └── scripts
        └── hello_web.sh

### Vagrant Configuration

Now we need to fill out the Vagrantfile configuration script to get started. Here’s a configuration for Ubuntu 16.04 Xenial Xerus:

    script_path = './scripts'

    Vagrant.configure("2") **do** |config|
      config.vm.box = "ubuntu/xenial64"
      config.vm.network "forwarded_port", guest: 80, host: 8080

    *  ####### Provision #######
    *  config.vm.provision "shell" **do** |script|
        script.path = "#{script_path}/hello_web.sh"
        script.args = %w(apache2 apache2 /var/www/html)
      **end**
    **end**

With this in place, we can bring up our [**Vagrant](https://www.vagrantup.com/)** system without provisioning:

    vagrant up --no-provision

### Provisioning Script

Our provisioning script will install [**Apache HTTP server](https://httpd.apache.org/)** and create some content. Edit the scripts/hello_web.sh with this:

    *#!/usr/bin/env bash*

    *#### Set variables with intelligent defaults
    ***APACHE_PACKAGE**=**${1**:-'apache2'**}**
    **APACHE_SERVICE**=**${2**:-'apache2'**}**
    **APACHE_DOCROOT**=**${3**:-'/var/www/html'**}**

    *#### Download and Install Package
    *apt-get update
    apt-get install -y **${APACHE_PACKAGE}**

    *#### Start, Enable Service*
    systemctl start **${APACHE_SERVICE}**.service
    systemctl enable **${APACHE_SERVICE}**.service

    *#### Create Content
    *cat <<-'**HTML'** > **${APACHE_DOCROOT}**/index.html
    <**html**>
     <**body**>
     <**h1**>Hello World!</**h1**>
     </**body**>
    </**html**>
    **HTML**

We can test this provisioning script with:

    vagrant provision

### Testing the Solution

We can test the solution out using by running:

    curl -i [http://127.0.0.1:8080](http://127.0.0.1:8080)

This will return something like:

    HTTP/1.1 200 OK
    **Date**: Wed, 08 Aug 2018 08:41:30 GMT
    **Server**: Apache/2.4.18 (Ubuntu)
    **Last-Modified**: Wed, 08 Aug 2018 08:39:09 GMT
    **ETag**: "37-572e871b6e929"
    **Accept-Ranges**: bytes
    **Content-Length**: 55
    **Content-Type**: text/html

    <**html**>
     <**body**>
     <**h1**>Hello World!</**h1**>
     </**body**>
    </**html**>

## Part II: Adapting to Other Operating Systems

![**Debian • Ubuntu • CentOS • Fedora • Gentoo • ArchLinux**](https://cdn-images-1.medium.com/max/2658/1*WCezdMegMlMyfG8hitOWDQ.png)***Debian • Ubuntu • CentOS • Fedora • Gentoo • ArchLinux***

We can expand our shell provisioning system to support other operating system distributions (or distros for short).

We must first update our Vagrantfile to target the desired Vagrant image for the distro we want, and also pass in the correct arguments to our shell script for the ① *package name*, ② *service name*, and ③ *docroot*, that correspond to the distro we are targeting.

Further, we have to update our hello_web.sh provisioning script to support each of these distros.

### Update Vagrantfile

As we added the ability to pass arguments to the shell script from [**Vagrant](https://www.vagrantup.com/)**, we can easily modify our Vagrantfile to support other operating systems.

If you are using the same environment, first remove the current environment with vagrant destroy, then we can modify the existing Vagrantfile to support CentOS 7 for example:

    script_path = './scripts'

    Vagrant.configure("2") **do** |config|
      config.vm.box = "centos/7"
      config.vm.network "forwarded_port", guest: 80, host: 8080

    *  ####### Provision #######
    *  config.vm.provision "shell" **do** |script|
        script.path = "#{script_path}/hello_web.sh"
        script.args = %w(
          httpd 
          httpd 
          /var/www/html
        )
      **end**
    **end**

In this Vagrantfile configuration, we simply swapped the config.vm.box to point to the desired distro, and pass the appropriate script.args for the *package name*, *service name*, and *docroot*.

These values can vary depending on operating system distro you are using (and also version of the distro). Here’s a chart of the differences from popular distros:

![**Vagrant box name and arguments for package, service, docroot**](https://cdn-images-1.medium.com/max/3364/1*gKJbyifVQ79xIIobLZTczQ.png)***Vagrant box name and arguments for package, service, docroot***

As another example, using [**Gentoo](https://www.gentoo.org/)**, the Vagrantfile would be changed to this:

    script_path = './scripts'

    Vagrant.configure("2") **do** |config|
      config.vm.box = "generic/gentoo"
      config.vm.network "forwarded_port", guest: 80, host: 8080

    *  ####### Provision #######
    *  config.vm.provision "shell" **do** |script|
        script.path = "#{script_path}/hello_web.sh"
        script.args = %w(
          www-servers/apache 
          apache2 
          /var/www/localhost/htdocs
        )
      **end**
    **end**

### Updating Provisioning Script

We also need to evolve our provisioning script scripts/hello_web.sh to support different distros. This can get quite complex, as different distros have different *package management* and *service management* systems.

    *#!/usr/bin/env bash*

    #### Set variables with intelligent defaults
    **APACHE_PACKAGE**=**${1**:-'apache2'**}**
    **APACHE_SERVICE**=**${2**:-'apache2'**}**
    **APACHE_DOCROOT**=**${3**:-'/var/www/html'**}**

    *#### Download and Install Package
    ***DISTRO**=$(
     awk -F= '/^ID=/{print tolower($2) }' /etc/os-release \
      | tr -d '"'
    )

    **case** "**${DISTRO}**" **in**
      centos|rhel)
        yum install -y **${APACHE_PACKAGE}**
        ;;
      fedora)
        dnf install -y **${APACHE_PACKAGE}**
        ;;
      debian|ubuntu)
        apt-get update
        apt-get install -y **${APACHE_PACKAGE}**
        ;;
      gentoo)
        emerge **${APACHE_PACKAGE}**
        ;;
      arch)
        pacman -Syu --noconfirm
        pacman -S --noconfirm **${APACHE_PACKAGE}**
        ;;
      *)
        echo "Distro '**${DISTRO}**' not supported" 2>&1
        exit 1
        ;;
    **esac**

    *#### Start, Enable Service
    ***case** "**${DISTRO}**" **in**
      centos|rhel|fedora|debian|ubuntu|arch)
        systemctl start **${APACHE_SERVICE}**.service
        systemctl enable **${APACHE_SERVICE}**.service
        ;;
      gentoo)
        rc-update add **${APACHE_SERVICE}** default
        /etc/init.d/**${APACHE_SERVICE}** start
        ;;
      *)
        echo "Distro '**${DISTRO}**' not supported" 2>&1
        exit 1
        ;;
    **esac**

    *#### Create Content
    *cat <<-'**HTML**' > **${APACHE_DOCROOT}**/index.html
    <html>
      <body>
        <h1>Hello World!</h1>
      </body>
    </html>
    **HTML**

## Shell Provisioning is Complex — Don’t do it

As you can see, wiring in all these variations can get quite complex to script in shell or other scripting language, as you have to handle all these variations.

As an alternative, a change configuration solutions like [**CFEngine](https://cfengine.com/)** or CAPS ([**Chef](https://www.chef.io/chef/)**, [**Ansible](https://www.ansible.com/)**, [**Puppet](https://puppet.com/)**, [**Salt Stack](https://saltstack.com/)**) have the ability to abstract the specific operating system resources, conceptually, you just do something conceptually like this:

    **package**(**$package_name**).install()
    **service**(**$service_name**).enable().start()
    **file**("**$docroot**/index.html", $source).create()

With a good change configuration, you don’t have to worry about the unique package system (*yum*, *dnf*, *apt-get*, *emerge*, or *pacman*) and instead worry about a generic *package* resource.

Similarly, with change configuration, you don’t have to worry about the underlying service management system ([**System V init system](https://en.wikipedia.org/wiki/Init)** with /etc/init.d scripts, [**Upstart](http://upstart.ubuntu.com/)** with Ubuntu 14 and CentOS 6, or newer [**SystemD](https://www.freedesktop.org/wiki/Software/systemd/)** init system), and instead just worry about a generic *service* resource.

## Wrapping Up

That is essentially all there is to it, the take away is that you can use [**Vagrant](https://www.vagrantup.com/)** to configure your system with a variety of solutions. This can be used to rapidly develop and test on virtual systems at no cost other than the time it takes to initially download the image from [**VagrantCloud](https://app.vagrantup.com/boxes/search)**.

Shell scripting is great for quickly learning how to configure a new operating system or learning a new service, but can get quite complex, and for this reason, long term, consider moving toward proper change configuration solution that will have abstractions for system resources.
