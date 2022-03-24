---
title: "Building an MQTTv5 server in python from scratch"
date: 2022-01-20T12:00:00+00:00
featureImage: https://miro.medium.com/max/1400/0*BYu40zHdQL8jymUb
postImage: https://miro.medium.com/max/1400/0*BYu40zHdQL8jymUb
# tags: c
categories: blog
toc: true
mediumLink: https://tommcn.medium.com/building-an-mqttv5-server-in-python-from-scratch-99da4de4337c
---

# Building an MQTTv5 server in python from scratch

Introduction

The *Internet Of Things* (IOT) is the collection of physical devices which have sensors, processing ability and a way to connect to other devices, often over the Internet. For example, a weather station which collected humidity, temperature and wind speed/direction and then reported this data to a server could be considered an IOT device. Communication between these devices and a server, also called *peers* or *nodes*, is an interesting problem as we need to balance speed, computational cost and reliability.

## The Problem

![Photo by [Erik Mclean](https://unsplash.com/@introspectivedsgn?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/10944/0*cSdoY1DItoBfpZZy)*Photo by [Erik Mclean](https://unsplash.com/@introspectivedsgn?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

Let’s say you have a thermometer hooked up to a micro-controller. You want to send the current temperature to a server for the purpose of analyzing the data. Temperature can be represented using a single byte which gives a range of [-127; 127]. Which means we need a minimum of 1 byte for our payload.

Now we need to decide how this payload will navigate the internet. The most obvious solution would be to use TCP/IP (Transmission Control Protocol over Internet Protocol, aka Internet protocol suite). TCP’s strength is that it guarantees that (if possible) packets will arrive to the destination and that they will be correctly ordered.

*More technical note: While UDP could be used here instead of TCP because of its speed, UDP does not guarantee that the packets will arrive to the destination (no acknowledgement). In an environment where decisions need to be taken based off the sent data as soon as possible (think of pipelines, data-centers, solar farms…), missing a message could be costly. However, in our example, the data is not necessarily immediately critical, so UDP could be acceptable. Also, it would technically be possible to write a custom TCP implementation that would remove unneeded features (ie. packet ordering is useless if we have small enough packets and maximum packet size guarantees), this is out of the scope of this project and just a bad idea overall.*

![Photo by [NASA](https://unsplash.com/@nasa?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/8512/0*VExVWeEoW_1GzyD_)*Photo by [NASA](https://unsplash.com/@nasa?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

But let’s go back to the payload. While it may seem that seem that sending just the temperature will work fine, it will not scale. Let’s say that you want to add a second temperature sensor, how would you differentiate between both temperature sensors? The easy and obvious solution would be to add an identifier to the payload to uniquely identify each sensor. The UID (Unique Identifier) could be a short with 2 bytes for up to ~32 000 devices or, to future-proof your design, you could use an int with 4 bytes for up to about 2 billion devices. The choice would depend on how you expect your project to scale. We are now up to 3-5 bytes for our payload.

The choice of protocol is important. Different protocols have different levels of overhead. Overhead is the minimum number of bytes that has to be sent no matter what the actual data is. For example, the protocol might want to send the time the message was sent, while this information could be interesting for different use-cases, for our use this information isn’t important and thus a waste of bandwidth. If we choose a protocol with a lot of overhead, we could potentially be sending dozens of bytes over the network and only have 5 bytes worth of information. This would be a huge waste of bandwidth, especially if you start scaling to thousands or hundreds of thousands of devices.

![Photo by [Denny Müller](https://unsplash.com/@redaquamedia?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/6834/0*BYu40zHdQL8jymUb)*Photo by [Denny Müller](https://unsplash.com/@redaquamedia?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

## The Solution

We are looking for a protocol that has a high level of reliability while keeping overhead costs low. Fortunately, such a standard exists today: [MQTT](https://mqtt.org/) (Message Queuing Telemetry Transport) is the standard for IoT Messaging. MQTT follows a publish-subscribe design, meaning that clients would subscribe to “topics” and are able to publish to those topics the information they want to relay to all “subscribers”.

![The MQTT pub-sub architecture](https://cdn-images-1.medium.com/max/2048/0*fzrY-RkZq-FIl6GM.png)*The MQTT pub-sub architecture*

MQTT has **very low overhead**, only 4 bytes plus the topic name (which could be as small as you want) for its most basic publish operation on top of TCP. It also comes with ways for the client to have some sort of guaranty that the message has arrived, called “acknowledgements” (ACK). Most write operations (CONNECT, SUBSCRIBE, PUBLISH…) have an acknowledgement equivalent (namely CONNACK, SUBACK and PUBACK). The broker would send those packet to indicate it has received and acted on the packet.

## An MQTT packet

An MQTT packet contains up to 3 parts: a fixed header (present in all packets), a variable header (present in some packets) and a payload (present in some packets).

**Fixed Header**

The fixed headers is only 2 bytes long. It contains three pieces of information: the packet type, a number between 1 and 15 (4 bits), indicates the type of packet this is (ie. CONNECT, PUBLISH, SUBACK…), special flags which specify certain behaviours that the broker should take (4 bits). The second byte is used to denote the remaining length of the packet in bytes, which allow us to know where the packet ends and where a new one begins.

![Fixed Header format for all packet types. *Adapted from the MQTTv5 specification*](https://cdn-images-1.medium.com/max/2000/1*cSxffHdKbt5YvceDRNpfkA.png)*Fixed Header format for all packet types. *Adapted from the MQTTv5 specification**

**Variable Header**

When its presence is mandated, the variable header contains “properties” of the request, stored in a key-value format. Those properties depend of the type of packet can be used to indicate the presence (or lack thereof) of features, human-readable error reason codes, how the client should interpret the data. It will also contain some packet specific data that would not be appropriate to put in the Payload.

**Payload**

The payload of an MQTT packet is the where the raw user data is stored. Most times, it will contain the data that is communicated (in PUBLISH packets) to and from the broker and nothing else.

## Writing a MQTT Broker in Python from Scratch

*Note: The code showed here is incomplete and for reference. The full code is available on my [GitHub](https://github.com/tommcn/python-mqtt).*

**The TCP server**

We first open a threaded TCP socket server. This allows us to receive raw TCP packets for us to handle in TCPHandler. The server is threaded to allow us to handle many connections at the same time meaning that the I can run multiple instance of the handler at the same time, instead of having to wait for the program to finish handling one connection before moving on to the other (*however one needs to be careful with race conditions when using threads*).
```python
import socketserver

class TCPHandler(socketserver.BaseRequestHandler):
    def handle(self):
        # Handle incoming TCP packets
        pass 

class Server(socketserver.ThreadingTCPServer):
    daemon_threads = True
    allow_reuse_address = True

def runserver(host, port):
    with Server((host, port), TCPHandler) as server:
        try:
            log.info("Starting server on %s:%s", host, port)
            server.serve_forever()
        except KeyboardInterrupt:
            server.shutdown()
            print()
            log.info("Server stopped")
```
<iframe src="https://medium.com/media/1cc26ad14f2d9eb202e75a659352f2b9" frameborder=0></iframe>

**Parsing Fixed Header**

Once we receive a TCP packet, we need to get the first two bytes to get the Fixed Header. Using byte manipulation we are able to extract the packet type and flags from the first byte.

Once we have the packet type, we can send the data to a function made to parse the packet type.

**Handling packet**

Once we know how to handle the packet, we can get the remainder of the packet and start parsing the variable header and payload.
```py
def handle_CONNECT(self, data):
    MQTTRemaingLength = data[1]
    remaining = self.request.recv(MQTTRemaingLength)
    protocolName = remaining[:6]
    # Continue handling the packet
```
<iframe src="https://medium.com/media/296d39e9732258b752987d4f59031ec2" frameborder=0></iframe>

*I won’t show the entirety of the handling of the packet because it contains a lot of checking and making sure formats are correct. All my code is available on my [GitHub](https://github.com/tommcn/python-mqtt).*

**Answering packets**

I wrote python classes which allow me to write the code to build and send packets in a streamlined manner. Notice that the Variable Header and Payload are kept separate and the use of bytearray to help represent the data going over the wire.
```py
import constants as c
from packet import MQTTFixedHeader, MQTTPacket, MQTTPayload, MQTTVariableHeader


class ConnackVariableHeader(MQTTVariableHeader):
    def __init__(self, connectReason):
        self.returnCode = connectReason

    def toBytes(self):
        sessionPresent = False
        out = bytearray()
        out.append(sessionPresent << 7)
        out.append(self.returnCode)
        out.append(0)
        return out


class ConnackPayload(MQTTPayload):
    def __init__(self):
        self.data = bytearray()

    def toBytes(self):
        return self.data


class ConnackPacket(MQTTPacket):
    def __init__(self, variableHeader=None, payload=None):
        header = MQTTFixedHeader(c.MQTTControlPacketType.CONNACK, 0x00)
        super().__init__(header)
        self.variableHeader = variableHeader
        self.payload = payload

    def build(self):
        variableHeader = ConnackVariableHeader(0x00)
        payload = ConnackPayload()
        packet = ConnackPacket(variableHeader, payload)
        return packet
```
<iframe src="https://medium.com/media/4a57f8f72b539809a07b2af4e595006b" frameborder=0></iframe>

*Again, I can’t show all parents classes due to their size and many of them contain more complex code to allow me to extend the classes easily. As always, all my code is available on my [GitHub](https://github.com/tommcn/python-mqtt).*

This format allows me to write code for all packets in a succinct and consistent manner.

## Conclusion

The MQTT standard, which was used to monitor pipelines in 1999 and bandwidth on a satellite link, was opened by IBM on version 3.1 in 2010. In 2013, IBM gave control of the standard to [OASIS](https://www.oasis-open.org/), a conglomerate of open standard that solve complex technical problem. In 2019, MQTTv5 was released. It balances the need for reliability and efficiency in IoT environments where ressources are scarce and efficacy is critical. Using python without any packages, we are able to parse MQTT packets over a raw TCP socket. MQTT alternatives exists that are either faster, have smaller weight or provide a better developer experience but this is by far the most common one and was very interesting to implement.

## Further reading

I would recommend reading the [original technical specification](https://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html) and visiting the [OASIS website](https://www.oasis-open.org/). For those looking for a more robust and efficient implementation, [Eclipse Mosquitto](https://github.com/eclipse/mosquitto) (written in C) is one of the more popular ones. If you want to see my python implementation you can do this [here](https://github.com/tommcn/python-mqtt).

*If you liked this article, consider following me on Medium, or connect with me on [LinkedIn](https://www.linkedin.com/in/tomas-mc/).*
