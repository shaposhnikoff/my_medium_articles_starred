
# Raspberry Pi 3 — Shell Scripting — Door Monitor (an IoT Device)

I recently worked on a fun project and we have different hardware and servers set up locally. I set up a door monitor system that would keep track of every entrance and exist in the room for security purpose.

Even though there are several door monitor devices, services, and gadget out there that I could have bought online, I decided to use my electronic and programming background to built one from scratch with a Raspberry Pi.

![**Raspberry Pi 3 Model B**](https://cdn-images-1.medium.com/max/4096/1*A_I9CHNsFv1EMX2LL1lQ8g.png)***Raspberry Pi 3 Model B***

So, I’m going to share some of the problems I ran into and how I fixed them while building a fully functional door monitor with a Raspberry Pi 3 Model B, mini breadboard, magnetic switch door sensor, terminal block, 3 meters of bell wire, 2 resistors(10K and 1K), and 3x IDC connection wires to the Pi. All of these package kits can be purchased online on Amazon.

![connection diagram — photo credit: Tim Rustige](https://cdn-images-1.medium.com/max/2380/1*nuYR1fgQPV_E50KEqEahSw.jpeg)*connection diagram — photo credit: Tim Rustige*

After making all the connections, the next step is coding, creating a script that would monitor the door status and send a notification. I used shell-scripting for my project. The scripting can be done in python too.

![**IoT Device**](https://cdn-images-1.medium.com/max/8064/1*OKc_T5yX1UEysIF8a7OsKw.jpeg)***IoT Device***

On the picture above, my Pi is connected and powered through a battery pack expansion board power supply. This practice solves power loss failure and keeps the Pi running in case of electricity loss which allows us to check the door status log file when the Pi cannot send email notifications.

![**Door Sensors**](https://cdn-images-1.medium.com/max/2000/1*edyQyS5t7JsfuYlX5ytDNg.jpeg)***Door Sensors***

The magnetic switch door sensors should be placed on the door to be monitored.

### What is a script? “In Simple English”

Computers are strict logic machines with zero common sense. That means if we want them to do something, we have to provide them with detailed, step-by-step instructions on exactly what to do. This is what a script is. A list of steps to be completed in a specific order, which is easy enough if we’re doing something really basic, like simple math equations, but gets much harder when we’re running more complex operations.
> **How to make an executable shell script**

At this point, I assumed that you have made all the connection necessary and have your Pi set up and running on the latest version of Raspbian OS for the Raspberry Pi.

Open you terminal and type the following command:

    nano test.sh

That will open nano, a basic text editor and allow you to enter a series of commands to be executed when your script is executed, forming the basis of the script. Let’s make a script that will display a message when executed. Enter the following on test.sh:

    echo "Hello! this is a test script"

Then save the file & exit. If you aren’t familiar with nano:
> To save the changes you’ve made, press Ctrl + O. To exit nano, press Ctrl + X. If you ask nano to exit from a modified file, it will ask you if you want to save it. Just press N in case you don’t, or Y in case you do. It will then ask you for a filename. Just type it in or if u already have a file like test.sh in our case. Just press Enter.
> If you accidentally confirmed that you want to save the file but you actually don’t, you can always cancel by pressing Ctrl + C when you’re prompted for a filename.

To make the *test.sh* executable type:

    chmod u+x test.sh

and then to run the script:

    sudo ./test.sh

The following is the door motored shell script I used for my project. It sends an email when the door is opened or closed and write a the door status to a log file that can be checked in case the internet connection is down or a power loss failure.

<iframe src="https://medium.com/media/34b86d8894af8bc28097f3a65f831d1e" frameborder=0></iframe>

If you are working on a similar project and would like to use the script, you can simply create a shell file, copy and paste the code above, make the script executable, and run it on your Pi.

Before you can run the script, you need to connect the Pi to the internet, have or create a Gmail account (using your PC) which will be used to send an email notification from the Pi, and install-configure several mail applications from the internet.

Install the following package:

    sudo apt-get install ssmtp

    sudo apt-get install mailutils

    sudo apt-get install mpack

Setup default settings SSMTP so the Pi can send emails. Open the ssmtp.config file:

    sudo nano /etc/ssmtp/ssmtp.conf

Here is how I configured my ssmtp.config file:

<iframe src="https://medium.com/media/cf6b03e2e4b597dd8cb9622fb0c790ec" frameborder=0></iframe>

save file & exit.

Make sure the Pi is connected to the internet. Send a test email with this command, substituting the email address below with your own:

    echo “This is a test email” | mail -s “Subject” example2@gmail.com

Assuming that worked okay, you can now run your script.

    sudo ./yourDoorScript.sh

Assuming all the hardware connections were made correctly and the script was well written. An email notification will be sent to the email address specified in the script every time the door is opened.

## Raspberry pi 3 Hints

The following are few tips about setting up the Pi.
> **Checking script status / detecting script crash**

The door monitor script is meant to run continuously (runs in an infinite loop) that check door state and send a notification when the door is opened. I created another script named scriptStatus.sh that check if the door monitor script is running fine and restart it if It crashes for some reasons.

<iframe src="https://medium.com/media/d52bf1f9777989ccf2c0a2bce911276d" frameborder=0></iframe>
> **How to run a script or a program on the RPi3 at start up**

They are five ways or methods that are available to run a program or script at boot on the Raspberry Pi:

* rc.local

* .bashrc

* init.d tab

* systemd

* crontab

In this article, I’m going to explore the **rc.local **method. This method is especially useful if the Pi is not gonna be connected to a monitor, you want to power up your Pi in headless mode, and have it run a program without configuration or a manual start.

In order to have a command or program run when the Pi boots, edit the file /etc/rc.local and add commands to the **rc.local** file. I’m using nano in this article, you can use an editor of your choice.

    sudo nano /etc/rc.local

Add commands to run the shell script using the complete file path or *absolute referencing* of the file location. Make sure the line **exit 0** is at the end, then save the file and exit.
> **Hints**

* Make sure to reference absolute file names rather than relative to your home folder. For example use “/home/pi/Desktop/myScript.sh” instead of “myScript.sh”.

* If your command runs continuously (perhaps runs an infinite loop) or is likely not to exit, you must be sure to fork the process by adding an ampersand to the end of the command, like so:

    sudo /home/pi/Desktop/myScript.sh &

Otherwise, the script will not end and the Pi will not boot. The ampersand allows the command to run in a separate process and continue booting with the process running. Here is a copy of my edited rc.local file:

<iframe src="https://medium.com/media/2b408122899fe7558b60811b28fe7ffb" frameborder=0></iframe>

Reboot the Pi to have the script executed startup:

    sudo reboot
> For more details, check out the project code on my [**GitHub](https://github.com/YannMjl/RaspberryPi-Scripts)**
> If you enjoyed this article, you might also like “[**How to build a bidirectional app for Internet of Things/Chat with Python](https://medium.com/coinmonks/how-to-built-a-bidirectional-app-for-internet-of-thing-chat-with-python-fc926e605b0f)**”
> # Cheers!!!!
