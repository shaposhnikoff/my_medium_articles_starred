
# How to connect Google Cloud Platform networks via VPN

When multiple teams are working on a software development project it makes sense to have multiple Google Cloud Platform projects for more fine-grained access and permission control over project resources. With this approach, however, you need a way to connect services without having these services exposed to the public. That’s where a VPN helps. You can use a VPN to securely connect the networks in which your services are running. In this blog post we’ll set up a 1:1 connection between two networks. The Google VPN setup also allows for 1:n and n:m VPN setups between networks.

### Quick summary of steps needed

* In each of your projects create a dedicated network for your instances that you want to connect via VPN

* Set up the VPN connection in each of your projects with a dedicated endpoint IP address

* Add Firewall rules so that instances can “talk” across their dedicated network boundaries

* Test the VPN setup

### Detailed steps of connecting Google Cloud Platform networks via VPN

Create a dedicated network for Google Compute Engine instances you want to connect via VPN in project A. This will be our project-a-network. Choose an address range in one of the [private IP address ranges](https://en.wikipedia.org/wiki/Private_network). We will use a class C network with over 65k addresses for our example. Note the address range, you’ll need it later for the VPN connection setup. Last but not least, select one address in your address range as the “Gateway”.

![](https://cdn-images-1.medium.com/max/2000/1*Es0eFNhVcxMU0_JM0fW7BQ.png)

Repeat the steps above to create a dedicated network for the instances you want to connect via VPN in project B. This will be our project-b-network. Important note: Choose an address range distinct from the one you chose for the project A setup. Again, note the address range and set a valid “Gateway”.

![](https://cdn-images-1.medium.com/max/2000/1*_UDSQNKlYhFqVz0ppvGjog.png)

The next step is to create a VPN Connection for each of your projects pointing to each other. Go to Networks->VPN and start creating a new VPN connection in project A. Give it a name, select the project-a-network you created in the prior step, and create an IP address for the VPN. At this point you don’t know the remote IP address just yet, so leave it blank for now.

Choose a strong shared secret (the shared secret is very sensitive as it allows access into your network -> **keep it very very safe!**) and set the “Remote network IP range” to the “Address range” you selected for your project-b-network. The shared secret is only to be used for the VPN connection setup, don’t share it anywhere else.

Now only the “Remote peer IP address” is still missing. Keep the setup page open and open a new browser tab. Go to the Developer Console and repeat the same steps for setting up a VPN connection in your project B. Once you have created an “IP address” for your project B VPN setup, note it, go back to your project A VPN setup page, and insert this “IP address” in the “Remote peer IP address” field.

In your project B VPN setup, set the “Remote peer IP address” to the “IP address” you created for the project A setup. Use the same shared secret for both setups.

![](https://cdn-images-1.medium.com/max/2000/1*lnEIf8rpDw_GYWTJNi6DSQ.png)

After a short wait you should see a green checkmark for both of your VPN setups indicating a successful configuration. If you don’t see these checkmarks, check the logs for information on what went wrong when establishing the connection.

![](https://cdn-images-1.medium.com/max/2000/1*sHrxNxZhk0lGV8Ik5_eCDg.png)

### Testing the VPN

When you see the green checkmarks for both your VPN setups you should be good to go. Let’s verify that the connection really works.

What do we need to do to test the connection?

* Create Compute Engine instances in project A and project B that are in the networks we created for the VPN

* Create Firewall rules to allow SSH and ping

* SSH into the instances, lookup the IP address, and ping the machine in the “other” network

First we create a Google Compute Engine instance in each of our projects. Select the respective dedicated networks (project-a-network and project-b-network) in the “Management, disk, networking, access & security” options when you create the instances.

![](https://cdn-images-1.medium.com/max/2000/1*lD2ZTf9j_3bJ3yddMzVyFg.png)

Second, we need to add two firewall rules to the dedicated networks (project-a-network and project-b-network) we created. Go to Networking-> Networks and click the project-[a|b]-network. Click “Add firewall rule”. The first rule we create allows SSH traffic from the public so that we can SSH into the instances we just created. The second rule allows icmp traffic (ping uses the icmp protocol) between the two networks.

![](https://cdn-images-1.medium.com/max/2000/1*GqZS0EAaMD0m391cx4doVA.png)

![](https://cdn-images-1.medium.com/max/2000/1*68B6Yg8in0y78H8pHPNAjg.png)

Now you can SSH into each instance and ping the other.

![](https://cdn-images-1.medium.com/max/3602/1*EzNfp_qoSyvHlT2SpyEm2w.png)

In this blog post I have demonstrated how to set up a 1:1 VPN connection between two Google Cloud Platform networks. You can extend these 1:1 connections to 1:n or even n:m links between networks by adding additional “Tunnels” to your VPN connection(s).

Additional information about setting up VPN connections can be found at [https://cloud.google.com/compute/docs/vpn](https://cloud.google.com/compute/docs/vpn)**.**
