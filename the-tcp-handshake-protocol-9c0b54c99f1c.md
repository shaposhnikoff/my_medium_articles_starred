
# The TCP Handshake Protocol

The TCP Handshake Protocol

Initiating communications on a TCP/IP network uses what are called ***handshake protocols***. These are the rules that determine communications between dissimilar systems over a public domain network. It is like a l*ingua franca* among computers so that they can understand each other. It is actually a de facto standard that has evolved over the decades with the growth of the Internet. This is what allows any device with support for the TCP/IP protocol stack to access the Internet to send and receive data.

The TCP/IP protocol provides an abstraction layer for the application from the underlying hardware. The developer who builds the application does not need to know the specific physical details of the hardware. Otherwise, if that were the case, then the developer will need to build a different application that is compatible with each specific type of the hardware. That would be very difficult and very restrictive to device manufacturers who want to connect their devices to the network. Imagine if the developer had to build a different application for each manufacturer’s device just to connect to the Internet. That would be hard to manage when new manufacturers enter the scene.

The purpose of the TCP stack is to separate the concerns between the application developer and the hardware manufacturer. What the hardware manufacturer needs to focus on are the technical specifications on the **Physical Layer (PHY)**. This defines the electrical, signal and circuit specifications of the device. It is then up to the application developer to program access to the device using the **Application Layer (APP)**. This PHY and APP will interact using the standard protocol defined in TCP/IP.

### Sockets

In order for the PHY to communicate with the APP layer, endpoints called ***sockets*** must be established. The socket establishes a bi-directional communications channel either within the same machine or between many machines on the network. These are common transports that deal with network elements. Depending on the programming language used, the sockets use specific classes that are provided as library modules which developers can access.

In theory, the service running on a server is identified by a port number. Typically the [port number](https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers) can be assigned to any service that runs on a server on the network. Standardization committees like the IANA and IETF have come up with port assignments (i.e. Well Known Ports) for the most commonly used services to simplify the process. These ports were specified under **IETF RFC 1700**. Common ports that are used include HTTP(80), FTP(21), SMTP(25), DNS(53), Telnet(23), SSH(22) and HTTPS(443) to name a few. To access the services running on these ports, a socket is created to establish the connection.

### Three-Way Handshake

The method by which a TCP/IP client computer connects to a server is called the **Three-Way Handshake**. This involves the sending of three signals called a **message**.

1. The first message is the **SYN** or Synchronize message. The SYN data packet is a request to make a connection from the client to the server.

1. The server will reply back, if it is available, with a **SYN/ACK** or Synchronize/Acknowledge message. This means that the port on the server is open and available for connection. If the server is not available, it will timeout and no connection will be established.

1. The client will receive the SYN/ACK data packet and reply back to the server with the **ACK** or Acknowledge message. A communications channel is then established and the client can now connect to the server to transmit and receive data.

![A diagram showing the Three-Way Handshake](https://cdn-images-1.medium.com/max/2000/1*-jCZvPMYwE-03d2EPrAdDA.png)*A diagram showing the Three-Way Handshake*

For example, when a user connects to a website from their computer, they function as the client. The server is the computer that hosts the website. Every time that content is downloaded from the website, it undergoes the Three-Way Handshake. The client will request access on port 80 that serves up the website using the HTTP protocol. The server, if public, will open the connection for the client. Once the connection is established it will serve up the content and the client will download it to their Internet browser. Once that is completed, the connection remains open depending on the setting of the keep-alive header on the website. After the connection times out, usually after a moment of inactivity that is based on the website’s setting, the connection is terminated by the server.

### Network Socket Programming

This is a simple example of a client/server application which will implement sockets using **Python 3.6.5**. This is an example of running a service for a client to connect to. This implements everything locally, so the server and client are actually processes running on the same machine.

**SERVER**

First import the socket library module.

    import socket

Next, create the socket object, define the local machine, reserve a port for the service and bind the host to the port.

    s = socket.socket()         
    host = socket.gethostname() 
    port = 54321               
    s.bind((host, port))

The server then needs to open the port for client connections.

    s.listen(5)

This last part will establish the connection and confirm it with a message to the client. Then the connection will be closed.

    while True:
       c, addr = s.accept()     
       print('Got connection from', addr)
       c.close()                

**CLIENT**

Import the socket library module.

    import socket

Create the socket object.

    s = socket.socket()

Get the hostname of the server and port the service is running on to create the connection. No name is needed for this example since it is connecting locally to the service and the port must be the same as that on the server 54321.

    host = socket.gethostname() 
    port = 54321

Finally, establish the connection and close it after.

    s.connect((host, port))
    print(s.recv(1024))
    s.close()

Once you run the server and the client, you can confirm that it works when the following message appears:

    Got connection from (‘192.168.1.7’, 54001)

### TCP Connection Table

The TCP/IP protocol keeps a table of connections in the server’s memory. This tracks all the connections from clients who are connected to the server. The server listens only on 1 port, which multiplexes all incoming connections to the server. Multiplexing is the process of allowing multiple connections on one logical link. The server than responds back using a different port number. In this example the client used source port 54001 that established a connection on server port 54321.

![The server can open multiple connections using multiplexing techniques. This allows different clients to connect to the same destination port.](https://cdn-images-1.medium.com/max/2000/1*JaHmkNbcVOUp3aQB7EJYwA.png)*The server can open multiple connections using multiplexing techniques. This allows different clients to connect to the same destination port.*

The following is a summary of the fields in the connection table.

**State:** The connection status (e.g. waiting, listening, sending, closed, etc.)
**Local Address:** The IP address of the server. In a listening state it is set to 0.0.0.0.
**Local Port:** Port number on the server.
**Remote Address:** The connecting client’s IP address.
**Remote Port:** The port number the client is connected to.

### Summary

The Three-Way Handshake is the method used to describe client/server communications on a TCP/IP network. It is a standard way for computers to communicate regardless of their manufacture or brand. All that developers and manufacturers need to know are the specifications for their layer of concern. The rest is handled by the rules established in the protocols. This is what provides interoperability across the network.
