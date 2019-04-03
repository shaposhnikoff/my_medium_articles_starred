
# Hi, Dan. I wanted to follow up with you on this.

In the process of reading up on this I ran into the following posts, which might be useful to others interested in this topic:

[http://www.innervoice.in/blogs/2013/12/02/linux-bridge-virtual-networking/](http://www.innervoice.in/blogs/2013/12/02/linux-bridge-virtual-networking/)

[http://www.innervoice.in/blogs/2013/12/08/tap-interfaces-linux-bridge/](http://www.innervoice.in/blogs/2013/12/08/tap-interfaces-linux-bridge/)

[http://backreference.org/2010/03/26/tuntap-interface-tutorial/](http://backreference.org/2010/03/26/tuntap-interface-tutorial/)

A high level summary would be: the cbr0 bridge is created and attached to the eth0 physical nic, and for each pod a virtual ethernet interface is created in the pod’s namespace and attached to a tap interface created on the bridge.

Thanks again for reading and responding!
