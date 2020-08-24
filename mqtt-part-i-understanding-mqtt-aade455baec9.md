
# MQTT — Part I: Understanding MQTT

Here, I have another article, this time with MQTT.

![“A satellite view of the United States lighting up at night” by [NASA](https://unsplash.com/@nasa?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/13292/0*lzVHktUpQ6tSb041)*“A satellite view of the United States lighting up at night” by [NASA](https://unsplash.com/@nasa?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

I think, writing such articles seals information into my mind, so I wanted to share my latest work on MQTT with a quick PoC smart city design over MQTT network.

I hope this article helps people to understand MQTT and how to use MQTT for message transport in their IoT solutions.

A quick note; I am a software engineer, so I have only looked for high level design of MQTT for message delivery, have no experience of required physical network infrastructure for MQTT, however data size and details of MQTT packages can make network engineers to understand, IoT network requirements to support MQTT.

**MQTT : A Quick Introduction**

**MQTT **(Message Queuing Telemetry Transport) is an ISO standard ([ISO/IEC PRF 20922](https://www.iso.org/standard/69466.html)) connectivity protocol.

* MQTT has been developed to be used on low-bandwidth and high latency networks in late 1990s, such as delivering data over satellite connection.

* MQTT is one of the most popular M2M communication protocol nowadays for Internet of Things (IoT) where latency and low-bandwidth is not a problem.

* MQTT has been evolved to implement Publish/Subscribe messaging pattern to allow decoupled applications to run independently over the network and enable scale-able solutions for IoT applications.

* MQTT is a messaging protocol and messages are transferred on TCP/IP stack. Applications either sending or receiving messages use specified TCP ports for MQTT message transport.

More detailed information about MQTT and its history can be found from following resources:

* [http://mqtt.org/documentation](http://mqtt.org/documentation)

* [https://thenewstack.io/mqtt-protocol-iot/](https://thenewstack.io/mqtt-protocol-iot/)

* [https://www.hivemq.com/blog/mqtt-essentials-part-1-introducing-mqtt](https://www.hivemq.com/blog/mqtt-essentials-part-1-introducing-mqtt)

Before getting into more details with MQTT protocol, I want to make a brief introduction to *publish/subscribe messaging pattern*.

**Publish/Subscribe Messaging Pattern**

Publish/Subscribe messaging pattern defines a message/data transportation model to/from clients, one-to-many message distribution and decoupling of applications for large scale connected systems.

Below image shows a general overview of design of publish and subscribe messaging model to visualize the general overview.

![Publish Subscribe Model](https://cdn-images-1.medium.com/max/2000/1*UCkArzMw-UxOBmeONcIINQ.png)*Publish Subscribe Model*

Publish/Subscribe or pub/sub in short, is used when there are multiple clients generates data to send without knowing the receiver to message broker which, coordinates the incoming and outgoing messages.

Final component, subscriber, which are the clients who tries to get certain type of messages.

Pub/sub pattern allows decoupled working model, none of the workers is tightly connected with other clients so any of the failure won’t cause a total system crash, but certain malfunction may happen according to used data or client’s working model. It is also flexible to extend overall system for new clients without effecting overall system.

Clients are either be publishers or subscribers. Publishers are the clients sending messages with certain topic (message identifier, tag). Subscribers are the clients subscribes for a certain topic to receive relevant message/data.

RSS feeds can be small example, publishers are the websites and subscribers gets updates from the subscribed web sites.

![](https://cdn-images-1.medium.com/max/2390/1*Gad15t9Vpzmbvs-YTFE7GA.png)

**MQTT **implements pub/sub messaging model with defined standards and specifications on top of TCP/IP.

Pub/sub pattern fits into IoT because of the need for continuous meta data transport, like sensor data; which is generated continuously all the time and being consumed by many other applications such as data analytics, visualization etc.

**Note **that, pub/sub messaging is not a fit for real time applications because they are sensitive solutions and any delay or loss of data can create signification problems for real time systems, where that much sensitivity is ignored on MQTT messaging structure.

**MQTT Terminology**

Here are some terms which are widely used when reading, talking about MQTT from outer resources and in this article.

* **MQTT Client**: A client application which is connected to internet and implements MQTT on top of TCP/IP in order to send or receive messages.

* **MQTT Topic**: A message identifier used to tag, classify messages in a hierarchical concept.

* **Publisher**: MQTT Client which sends data over network.

* **Subscriber**: MQTT Client which subscribes to certain topic on the MQTT network.

* **MQTT Broker**: Broker’s mainly are application servers which controls the MQTT clients with their connectivity, authentication, message delivery, message storage.

* **Application Message**: Actual message, data which is being published or subscribed.

**MQTT Message Structure**

MQTT message structure sets specification and data structure for sending bytes and bits from/to clients and servers.

I will only cover some of the fundamental message types to understand how messages are structured in MQTT. You can always refer to MQTT specification for detailed explanations according to your needs.

**MQTT Specification**: [http://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html](http://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html)

**MQTT Application Message**

MQTT Application Message is a network package which has a certain structure defined by MQTT specification to be encoded by clients to read over to identify message type and encode incoming data.

TCP/IP package for MQTT is being constructed with three layers:

* Fixed header

* Variable header

* Payload

![MQTT Network Package Structure](https://cdn-images-1.medium.com/max/2000/1*wW0VvF3d7cRDt5Fr5DJkJA.png)*MQTT Network Package Structure*

**Fixed Header**

MQTT fixed header contains information about the type of message, if message is a connection message, published message, or an acknowledgement.

Fixed header has 2 bytes of information. 1st byte defines the type of package and its configuration by bits.

First 4 bits of 1st byte, shows the package type. 0000 and 1111 are reserved, not used, so there are 14 types of messages defined in MQTT specification.

* **CONNECT **package defined with 0001

* **CONNACK **package defined with 0010

* **PUBLISH **package defined with 0011

* And followed with **PUBACK, PUBREC, PUBREL, PUBCOMP, SUBSCRIBE, SUBACK, UNSUBSCRIBE, UNSUBACK, PINGREQ, PINGRESP, DISCONNECT**

Next 4 bits of 1st byte, used as flags used only for PUBLISH message types to set 3 configurations. Flags are DUP (Duplicated message) 5th bit, QoS (Quality of Service) 6th and 7th bit and RETAIN 8th bit

* **DUP**: Indicates current message is the duplicate of previous message

* **QoS**: 2 bits to indicate QoS Level. 0, 1 or 2. 00 ensures at most once delivery, 01 ensures at least once delivery, 10 ensures exactly one delivery of message.

* **RETAIN**: 1 bit to configure that message to be retained after its publish.

Next 1 byte has the information about remaining bytes including the data in Payload section. This helps the client to encode rest of the bytes.

**Variable Header**

Variable header is being used in some of the packages according to needs of the package type.

Variable header starts with package identifier bytes and followed with optional fields according to required packages.

**CONNECT** package type don’t have any information in Variable Header however, **PUBLISH **package type, includes Protocol Name (**MQTT**), Protocol Level, Connect Flags and Keep Alive bytes and bits. Topics are also send inside the variable header for **PUBLISH **messages.

**Payload**

Payload includes rest of the information carried along with the package. This information can be username and password for CONNECT package or application message for PUBLISH or SUBSCRIBE package types.

**CONNECT**

CONNECT package is used to set a connection between clients and servers to get authentication, check if connection exists to configure network and start message publish or subscribe. Below are the quick information about this package.

* **Package Type**: defined with 0001 bits.

* **Flags**: Disabled for this package type.

* Variable header consists of 10 bytes, including CONNECT flags

* **Username **and **password **can be send with this package when bits are set. They are being send in Payload section.

* Server responds to client with **CONNACK **package type.

**PUBLISH**

PUBLISH package is used to send messages to message broker.

* **Package Type**: defined with 0010 bits.

* **Flags**: Enabled only for PUBLISH, see above for definitions.

* **Variable header**: includes Topic name in variable header.

* Application message is put into Payload section.

* Server responds to client with **PUBACK **package type.

**SUBSCRIBE**

SUBSCRIBE package is used to attach itself to server to receive messages with requested tag to message broker.

* Package Type: defined with 1000 bits.

* Flags: Disabled

* Variable header: has package identifier information/

* Payload: Topic filter and QoS info send within Payload section.

* Server responds to client with **SUBACK (**1001**) **package type.

SUBSCRIBE package is used to let message broker know about the topics request of client. When a subscribe message send to message broker, message broker sends publish type messages with determined topic to subscribed client.

For more detailed explanation please refer to MQTT specs. This much information might be too low level if you are not implementing a new MQTT library but I think it is important to understand underlying logic of such protocols to get all benefits and use systems effectively with existing MQTT libraries.

* [http://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html](http://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html)

* [http://mqtt.org/documentation](http://mqtt.org/documentation)

**MQTT Topic/s**

![Photo by [rawpixel](https://unsplash.com/@rawpixel?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/12000/0*-awbyMJ9qmyrnhC6)*Photo by [rawpixel](https://unsplash.com/@rawpixel?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

MQTT Topic is an UTF-8 encoded string send with MQTT TCP/IP packages. A MQTT topic can be 64KB size at maximum.

Topics mainly used to classifying, or tagging messages. Subscriber client makes requests of certain messages belong to certain topic and publishers sends messages with their topic. Topics allow a large system to control over messages and their flow between clients.

Topics are one of the most important concept of MQTT messaging infrastructure to keep overall system simple or complex.

For large systems, including thousands of clients publishing millions of messages should be managed in a very well hierarchical manner to handle removing or adding new clients or new data types to system seamlessly. Therefore, one should be aware of certain principles of topics and

**Topic Level**

Topic string can be any string but since we talked about designing a hierarchical messaging structure, adding levels will make system to separate certain message subjects from each other to isolate messages.

Forward slash **“/” **is used for level separation. For example; you would have sensor data and location data of a certain client. How would you separate it?

location/client_id

sensor/client_id

**or**

client_id/location

client_id/sensor

Either way, you would separate data and make it easier for clients and servers to parse topics easier and create a global standard with using ‘/’ instead of a custom level separator.

**Special Characters for subscription**

There are two special characters used by subscriber clients to generalize subscription multiple topics at once:

**‘#’**

Hash or pound sign will allow you subscribe all levels coming after # sign.

In order to subscribe level4 and level5 messages, client need to subscribe below strings.

level1/level2/level3/level4

level1/level2/level3/level5

If **#** used, **level1/level2/level3/#** , it would be subscribed to both level4 and level5 at once.

# must be used right after ‘/’ char if its not used as the first level. If # char, used right after level identifier, you may encounter an error.

**‘+’**

‘+’ char used to define all chars at a level.

If there would be topics as below:

level1/level2/level3

level4/level2/level3

using ‘+’ at the beginning like +/level2/level3 will make client to subscribe for both topics at once.

Please see: [http://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html#_Toc398718106](http://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html#_Toc398718106) for mode details.

**MQTT Broker**

MQTT Broker is actually another MQTT Client which is mainly deployed as a server application to mediate subscriptions and published messages within the network.

You can think MQTT broker at the center of the system where all packages are coming and going through. It is also the main center to store messages, manage authentication of clients and all other functions which you can think of as a system manager.

MQTT broker or server uses TCP/IP port and 1883 is the default port for MQTT server which is assigned by IANA.

1883 used for non-TLS (Transport Layer Security) connections. 8883 port is assigned for TLS connections.

All security, client management is being done on MQTT broker and these requirements can be easily implemented over your existing user management or database systems.

There isn’t more specific information about MQTT broker rather than knowing:

* MQTT broker is a server which implements MQTT TCP/IP communication layer on TCP/IP ports 1883 or 8883.

* MQTT broker can be any operating system according to your choice of libraries, implementations and IT system requirements.

In this part of the article, I wanted to go over all important information about MQTT and in the next section of MQTT blog you can find how I utilize MQTT within a Smart City System.

**Next**: [MQTT — Part II: Smart City Design with MQTT](https://medium.com/@onur.dundar1/mqtt-part-ii-smart-city-design-over-mqtt-da41018a45c2)
