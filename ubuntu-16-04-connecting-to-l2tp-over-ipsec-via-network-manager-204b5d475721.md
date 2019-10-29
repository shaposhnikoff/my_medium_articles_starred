
# Ubuntu 16.04 Connecting to L2TP over IPSEC via Network Manager



![](https://cdn-images-1.medium.com/max/2000/1*VMaXdWQG637tblq6ZkwCcA.png)

This is more or less notes for myself but I am also hoping to save some of you from spending too much time setting up something that should be a simple functionality available out-of-the-box in my opinion.

While I am pretty fluid with editing config files (VIM as my choice of poison) and executing shell commands for what I need done, I feel like in this day-in-age, people should just be able to fill in the blanks on a GUI to connect back to their work places via VPN when using any latest flavor of GNU/Linux that comes with a standard graphical desktop (Gnome, Unity, KDE, Etc) interface by default.

This is of course not the case for my favorite distro, Ubuntu (LTS 16.04). So since the ideal scenario is not available to us, my next logical hunt was for a quick fix. During the process of identifying my quick fix, I stumbled upon quite a bit of misinformation and outdated guides that really comes no where near helping me accomplish my simple little task of connecting to my work L2TP over IPSEC VPN server at work. So in order to save myself and whoever reads this the next time connecting to a L2TP over IPSEC vpn server with Ubuntu 16.04 is needed, here are the actual steps that finally worked very well for me:

    sudo add-apt-repository ppa:nm-l2tp/network-manager-l2tp
    sudo apt-get update
    sudo apt-get install network-manager-l2tp-gnome
    sudo service xl2tpd stop
    sudo update-rc.d xl2tpd disable

![](https://cdn-images-1.medium.com/max/2000/1*0-mYBTxxnDlG0ySEhKIT_Q.png)

And that’s about it. Things were as easy as they should be from that point on.
