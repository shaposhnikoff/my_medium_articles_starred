Unknown markup type 10 { type: [33m10[39m, start: [33m98[39m, end: [33m103[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m29[39m, end: [33m38[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m57[39m, end: [33m61[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m98[39m, end: [33m111[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m27[39m, end: [33m44[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m63[39m, end: [33m67[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m51[39m, end: [33m61[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m85[39m, end: [33m98[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m4[39m, end: [33m21[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m29[39m, end: [33m33[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m100[39m, end: [33m112[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m149[39m, end: [33m163[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m41[39m, end: [33m65[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m102[39m, end: [33m108[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m25[39m, end: [33m33[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m41[39m, end: [33m50[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m57[39m, end: [33m73[39m }

# How to install Go Lang on Windows, macOS & Linux

Learn about what is Go Lang & how to get it up and running on your Windows, Mac or Linux systems.

![](https://cdn-images-1.medium.com/max/3600/1*ZZEHGJDQvkOeW7QTas3ULg.png)

## What exactly is Go Lang?

The Go programming language (also known as GoLang) is Google’s general-purpose programming language developed for the multi-core reality of today’s computers.

Concurrent, garbage-collected, and designed for scale, Go is a programming language made for building large-scale, complex software.

## Why use Go Lang? +1’s of GoLang?

Go is expressive, concise, clean, and efficient. Its concurrency mechanisms make it easy to write programs that get the most out of multicore and networked machines, while it's novel type system enables flexible and modular program construction. Go compiles quickly to machine code yet has the convenience of garbage collection and the power of run-time reflection. It’s a fast, statically typed, compiled language that feels like a dynamically typed, interpreted language.

## Is Go Lang the future? or not?

As Go Lang is made by **Google**, the project has regular financing. So, my answer is **Maybe or Maybe not**. The language is quite popular among guys but it’s still not in a position to bring up a revolution, it’s quite young.
> # **Hail Google, Hail GoLang**. ✈️

## Firing GoLang on your system

Setting up GoLang involves three basic steps as follows:

1. Downloading the installation files

1. Installing the packages

1. Verifying the up & running GoLang installation

## Minimum System Requirements

![Minimum System Requirements](https://cdn-images-1.medium.com/max/2306/1*KSFb47INand-6B-S8-uZwA.png)*Minimum System Requirements*
> Go Lang is currently supported only on x64 based processors.

## Downloading Go Lang Installation Files

Visit Go Lang’s official download page to download the installer according to your operating system.
[**Downloads**
*Go is an open-source programming language that makes it easy to build simple, reliable, and efficient software.*golang.org](https://golang.org/dl/)

![Downloadable options available](https://cdn-images-1.medium.com/max/2346/1*5PtbPMdaHEJ_vAzEUWUdRw.png)*Downloadable options available*

## Setting up on Windows

* Double click the MSI installer, you just downloaded to start the Go Lang installation on your Windows system.

* Follow the prompts to install the Go tools. By default, the installer puts the Go distribution in **c:\Go**.

![Running MSI Installer for Go Lang](https://cdn-images-1.medium.com/max/7680/1*8l12V96ORfs2fTwYukKAWQ.png)*Running MSI Installer for Go Lang*

* The installer should put the **c:\Go\bin** directory in your **PATH** environment variable. You may need to restart any open command prompts for the change to take effect.

## Setting up on macOS 🍎

* Double click the PKG installer, you just downloaded to start the Go Lang installation on your macOS system.

* Follow the prompts to install the Go tools. By default, the installer puts the Go distribution in **/usr/local/go**.

* The package should put the **/usr/local/go/bin** directory in your **PATH** environment variable. You may need to restart any open Terminal sessions for the change to take effect.

## Setting up on Linux

* Extract the **.tar.gz** archive you just downloaded to **/usr/local**, creating a Go tree in **/usr/local/go**

    **$** sudo tar -C /usr/local -xzf *go1.13.4.linux-amd64.tar.gz*
> Change the archive name as per requirement. ***go1.13.4.linux-amd64.tar.gz*** is the latest version available at the time of this writing.

* Add **/usr/local/go/bin** to the **PATH** environment variable. You can do this by adding this line to your **/etc/profile** (for a system-wide installation) or **$HOME/.profile**:

    **$** export PATH=$PATH:/usr/local/go/bin

## Verifying your installation
> The steps here are tested on a Windows system while being almost 99% similar for macOS & Linux. Follow along even if you have a macOS or Linux based system.

Check that Go is installed correctly by setting up a workspace and building a simple program, as follows.

* Launch a terminal window and navigate to **C:/Users/{%USERPROFILE%}**

    **$** cd C:/Users/*{%USERPROFILE%}*
> Here **{%USERPROFILE%}** refers to your username on your machine, mine is **shiva** here. 🔥 Do remember to replace it with yours.

* Now create the following directory structure inside the current directory and change the directory to **/hello**.

    **$** mkdir go/src/hello
    **$** cd go/src/hello

* Now, create a file named **hello.go** using the text editor of your choice, and copy-paste the following content into it.

    **package **main

    **import **"fmt"

    **func **main() **{**
        fmt.Printf("**Hello, World..!!**\n")
    **}**

* Save the file and go back to the terminal and run the following command to compile the code into a binary.

    **$** go build

* The previous step creates a binary named **hello.exe** in the same directory. Simply run the executable to run the code.

* If the installation had been correct, the **exe** will print **Hello, World..!!**

**The steps in Terminal will look somewhat like this**

![Live commands on Windows Terminal](https://cdn-images-1.medium.com/max/2000/1*KfaF_4yXpA_go1Sr_RTqTg.png)*Live commands on Windows Terminal*
> **ProTip:** I recommend using [Windows Terminal](https://www.microsoft.com/store/productId/9N0DX20HK701) with PS Core 6 enabled.

## Congratulations!

YaY..!! 🥳 Welcome to the world of **Go Lang**. The installation is finished and you are now ready to show up your mettle in programming.

## 🔍 Read more of my Articles
[**How to install Flutter on Mac & Windows**
*Learn about Flutter and how to set it up on Windows and Mac systems*medium.com](https://medium.com/enappd/install-flutter-on-windows-and-mac-1fd1dde453ba)
[**Guide to integrate Google Maps with Flutter**
*Learn how to build a mobile app featuring a Google Map using the Flutter SDK.*medium.com](https://medium.com/@ShivamGoyal1899/guide-to-integrate-google-maps-with-flutter-db86d033ea25)
[**Building a Flutter DateTime Picker in just 15 minutes**
*Learn implementation of an iOS-style DateTime Picker in Flutter*medium.com](https://medium.com/enappd/building-a-flutter-datetime-picker-in-just-15-minutes-6a4b13d6a6d1)

## 🎯 That’s all for today.
> # *If you got any queries hit me up in the comments or ping me over on [*hi@itsshivam.com](mailto:hi@itsshivam.com) 📧
> # *If you learned even a thing or two, clap your hands👏 as many times as you can to show your support! It really motivates me to contribute towards the community.*
> # *Feeling too generous? [Buy me a Drink](https://www.paypal.me/shivamgoyal1899/) 🍺*
> # *Wanna collaborate? [Let’s talk some tech](mailto:hi@itsshivam.com) 😊*
> # *Stalk me over on [itsshivam.com](https://itsshivam.com/), [GitHub](https://github.com/ShivamGoyal1899), or [LinkedIn](https://linkedin.com/in/shivamgoyal1899/). 👀*

![](https://cdn-images-1.medium.com/max/2000/0*Piks8Tu6xUYpF4DU)

**Follow us on [Twitter](https://twitter.com/joinfaun) **🐦** and [Facebook](https://www.facebook.com/faun.dev/) **👥** and join our [Facebook Group](https://www.facebook.com/groups/364904580892967/) **💬**.**

**To join our community Slack **🗣️ **and read our weekly Faun topics **🗞️,** click here⬇**

![](https://cdn-images-1.medium.com/max/3200/0*oSdFkACJxs5iy1oR)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇
