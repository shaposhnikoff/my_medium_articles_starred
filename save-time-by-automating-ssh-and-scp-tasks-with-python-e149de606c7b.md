
# Save Time by Automating SSH and SCP Tasks with Python

Photo by Aron Visuals on Unsplash

I recently helped a friend who had to flash software on more than 100 devices. For each device, he knew the MAC-address and had a corresponding password to log in via ssh. He had this information stored in an excel sheet. His manual process of flashing was

1. Find out the device’s IP-address given its MAC-address.

1. SSH into the device using the IP-address and the password.

1. Upload the software installer via SCP.

1. Execute some shell commands to install the software.

Doing this manually for one device is fine, for two you start thinking wtf and after three you think **WTF**! In software, it is always a goal to reduce the number of WTFs, so I thought maybe I can help him by automating his process using a nice and sweet Python script. In this article, I like to show you what we ended up with.

### Find IP-Address given a MAC-Address

The first thing we needed to address is finding out the IP-addresses of our devices inside our local network. For that to work, both your PC and your devices must be connected to the *same* network. Given an IP-address, you can use the Python module [scapy](https://scapy.readthedocs.io/) (Note that I have used a different package in a previous version of this article. However, this only worked when I already had a connection to or pinged another device) to find out which MAC-address uses the IP. In code, this looks like

    **from** scapy.all **import** ARP, Ether, srp
    **from** typing **import** Dict

    **def** ip_sweep() -> Dict[str,str]:
        target_ip = "192.168.1.1/24"
        arp = ARP(pdst=target_ip)
        ether = Ether(dst="ff:ff:ff:ff:ff:ff")
        packet = ether / arp
        result = srp(packet, timeout=3)[0]
        **return** {rec.hwsrc.lower(): rec.psrc for _, rec in result}

    mac_to_ip = ip_sweep()

So, we just test every IP address and store the ones that gave back a MAC — address. When using scapy, you need to execute your script as root. Cool, now we know all MAC-address to IP-address connections. One problem down three left.

### Establish an SSH Connection

Next problem was, how do we establish an SSH connection using Python. There is a neat library called [paramiko](http://www.paramiko.org/#) that allows you doing that fairly simple. At the very basic, you need the IP-address you want to connect to, a username, and a password. The code looks like

    **import** paramiko
    **# To allow connection without having to accept the host connection
    # We need to use this policy. I've just pulled it here due to 
    # mediums way of displaying code ...
    **policy** = **paramiko.client.AutoAddPolicy

    **# Iterate over all our found mac, ip connections and SSH into
    # the clients
    for** mac, ip **in** mac_to_ip.items():
        **# Assume that to be given**
        pwd = mac_to_pwd[mac]

    **with** paramiko.SSHClient() **as** client:
            client.set_missing_host_key_policy(policy)
            **try**:
                client.connect(ip, username=**'root'**, password=pwd)
            **except** paramiko.ssh_exception.NoValidConnectionsError:
                print("Connection failed")

Now, we can use the connected client to execute shell commands, and you will see that in a bit.

### Upload Data via SCP

Before executing commands, we need to upload the installer to our device. To that end, we make use of the library SCP, which goes perfectly hand in hand with paramiko. In the following example, assume we’ve already established a connection using the paramiko SSHClient.

    **from** scp **import** SCPClient 
    **import** sys

    **def** progress4(f, size, sent, p):
        prog = sent / size * 100.
        sys.stdout.write(**f"(**{p[0]}**:**{p[1]}**) **{f}\'**s progress: **{prog}\r**"**)

    local_path = "/home/me/installer.tar.gz"
    remote_path = "/home/root/inst/"
    **with** SCPClient(client.get_transport(), progress4=progress4) **as** scp:
        scp.put(local_path, remote_path=remote_path)

The *progress4* function prints out the upload progress. To me, this is helpful to get an estimate of how long I still have to wait and not become impatient, which might create another WTF :). Now we are almost there, just one step left.

### Execute Shell commands via SSH

Now, as we have a running ssh connection and uploaded the installer, we only have to unpack it and run it. This can be easily done through shell commands that we can send out using the SSH Client. In code, this looks like

    **def** exec_blocking(cmd, client):
        print(**f"Executing **{cmd}**"**)
        _, stdout_, stderr_ = client.exec_command(cmd)
        status = stdout_.channel.recv_exit_status()
        print(**f"STATUS **{status}**"**)
        **for** line **in** stdout_.readlines():
            print(line)

        **if** status != 0:
            errors = **"**\n**"**.join(list(stderr_.readlines))
            **raise** Exception(**f"**{cmd}** failed with **{errors}**"**)
    **# Unpack the installer** 
    exec_blocking("tar xzvf /home/root/inst/installer.xzvf", client)
    **# Run the installer**
    exec_blocking("./installer/install.sh", client)

The command gets sent to the device with *exec_command*. This is a non-blocking function that immediately returns. To wait until the command has finished execution you need to call *recv_exit_status()*. With this, you can make sure that your command was successful or not.

### Conclusion

Combining all the code snippets into a single script allowed us to flash/execute our commands on all the devices we had on our list with a single command. This saved us a lot of time and WTFs :)

Hopefully, you’ve learned something new. Thank you for following along and feel free to contact me for questions, comments, or suggestions.
