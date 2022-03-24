---
title: "Simulating a Smart City using TypeScript"
date: 2022-02-12T12:00:00+00:00
featureImage: https://miro.medium.com/max/1400/0*Za8jUfreX6UYzu36
postImage: https://cdn-images-1.medium.com/max/10944/0*yWNrVaZ3J_gUe2u8
# tags: c
categories: blog
toc: true
mediumLink: https://tommcn.medium.com/simulating-a-smart-city-using-typescript-2f7b87d53089
---

# Simulating a Smart City using TypeScript

Introduction

As the urban population of the earth continues to grow, the current city model needs to evolve. Climate change has underlined the necessity to rethink the way we interact with our city. From the way we commute to how we use the power grid, there are definitively things we can do to improve our carbon footprint. But the environment isn’t the only problem plaguing our cities. The way we are using our cities is evolving, they need to evolve with us. This is where the concept of smart cities comes into play.

![Photo by [Hugh Han](https://unsplash.com/@hughhan?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/12032/0*Za8jUfreX6UYzu36)*Photo by [Hugh Han](https://unsplash.com/@hughhan?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

A smart city is like a smart house, but on a much larger scale. Using sensors and other embedded devices at the scale of a city, we can save cost, increase accessibility and convenience for all citizens. This is like the way that smart homes can decrease energy consumption and assist you in day-to-day activities.

## What Did I Build?

I have built a series of programs that, when put together, could simulate how an actual smart city could look like. My solution is split up into three main components: devices, servers, and the database.

### Devices

In my solution, devices are the collection of sensors (like a temperature sensor) and actors (like a LED or motor), usually on the same physical device (Arduino, Raspberry Pi, ESP8266…) with a connection to the Internet (or some network). The device can make decisions and can control its actors without necessarily needing to “consult” with a central server. An example of a device could be a streetlamp that would turn on if presence is detected. The device would use its presence sensors to detect the presence of individuals and turn on the streetlamp with LED actors. All this processing happens on the device itself to reduce load on the central server.

Another example of a device would be a weather station that collects information (like temperature, humidity, wind speed, air quality…) and reports it to some server for storage and analysis.

![Photo by [NOAA](https://unsplash.com/@noaa?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/8000/0*FoAOOr0UJPw-T-SL)*Photo by [NOAA](https://unsplash.com/@noaa?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

### Servers

Devices are nice for small self-contained use-cases, but one will obviously want to create more complex systems, not limited to a single device. For example, let’s take a smart parking garage. You would have proximity sensors on top of every parking spot, with a LED to show drivers which spots in the row are free. You might also want to have a display at the entrance that shows the number of open spots for each floor at the entrance. To do this, you would have a server to which all your parking spots devices would connect, the server would then calculate the number of spots available per floor.

![Photo by [Jordan Harrison](https://unsplash.com/@jordanharrison?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/12000/0*exxoEYGDCU0Ba1wa)*Photo by [Jordan Harrison](https://unsplash.com/@jordanharrison?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

You can also have many servers that connect to each other. You might want a central server for all your parkings to display on a website. This allows us to have more distribution computing, like edge computing, which reduces load on a single server and helps save bandwidth.

### Database

Obviously, with all this data you collect, you will need to store it somewhere. You might think that some standard SQL database (SQLite and PostgreSQL often come to mind) will be fine to hold all your data. However, this will not scale well, if you have hundreds or thousands of data points that need to be committed to a database. Relational Database Management Systems (most SQL databases) are optimized to hold relationships between records and to perform complex queries. However, with our data, we only want to store a small amount of data with a timestamp, no fancy relationships needed. This is where time series databases (TSDB) come in. They can manage hundreds of thousands of points at a time with an extra-precise resolution (down to nanoseconds) and most of them will come with built-in tools for data analysis, aggregation, and visualization.

![Photo by [benjamin lehman](https://unsplash.com/@benjaminlehman?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/11520/0*3aroV_pxIXzl9cHC)*Photo by [benjamin lehman](https://unsplash.com/@benjaminlehman?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

In my project, I have decided to use [InfluxDB](https://www.influxdata.com/products/influxdb/), but the choice of DB is dependent on your use-case and specific requirements. InfluxDB also comes with built-in data visualization tools, alerting and dashboards. One of the things to keep in mind thought is that the open-source version of InfluxDB doesn’t include a straightforward way to shard the database, meaning that the database will need to be on a single device, which could be dangerous in case of an outage. InfluxDB will do its best to not corrupt or lose data in case of an outage, but downtime will still be a problem in case the database goes down.

### Docker

I have decided to run all those services on [Docker](https://www.docker.com/), which, simply put, creates light-weight virtual machines for all the services we run (in my context, a service denotes the code that powers a server or a device). Docker allows us to configure our services easily and run multiple services on a single physical device without worrying about a process hogging ressources (CPU, memory, storage, network…) or interacting with processes it shouldn’t be interacting with. Since Docker runs on top of a virtualization software, should your code be compromised, an attacker would not be able to access the machine and sensitive information (obviously, nothing is 100% secure, running an outdated version of Docker or using less secure options will make it possible for an attacker to escape the sandbox).

![Photo by [Ian Taylor](https://unsplash.com/@carrier_lost?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/9976/0*jVV3ehhe8gGoOzLt)*Photo by [Ian Taylor](https://unsplash.com/@carrier_lost?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

### MQTT

Finally, communication between devices and servers is important. It is important to have protocols that are both efficient and reliable. I will not go into details about the protocol as I have already gone into detail in a [previous article](https://tommcn.medium.com/building-an-mqttv5-server-in-python-from-scratch-99da4de4337c).

## Code

### Typescript

As the programming language, I decided to use TypeScript (TS). TS is a superset of the JavaScript (JS) language, and both are some of the most used languages in the world. The appeal of TS when compared to JS is that TS is a strongly typed language, meaning that all the variables have specific types and functions denote their return types. This adds some level of certainty that the code will not run into some of the more common programming errors (Attribute Error, Type Error…).

### ES6 Asynchronous Programming

ECMAScript 6 (ES6) is the JS standard that ensures interoperability on different devices and browsers. ES6 is the ES update that came out in 2015, it introduced new ways to use asynchronous programming features. Asynchronous programming is the capability to run functions or statements separately from the main thread. This means that you can start the execution of a function and continue with the rest of the code, essentially running multiple pieces of code at the same time, increasing performance without having to deal with threads.

![Photo by [Stephane Gagnon](https://unsplash.com/@metriics?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/10944/0*yWNrVaZ3J_gUe2u8)*Photo by [Stephane Gagnon](https://unsplash.com/@metriics?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

### Object Oriented Programming

Finally, using classes, I can easily reuse code that would be the same for all devices. For example, the way the data is sent to the server, or the monitoring code will be the same regardless of the device the code is being deployed on.

## Examples

*Note: the following code samples are incomplete and will not compile on their own. For the full code, see my [GitHub repo](https://github.com/tommcn/city-simulator).*

### Device Abstract Class

![The device abstract class](https://cdn-images-1.medium.com/max/2688/1*5JYuhdZ5yjqVfCWg2gsBHA.png)*The device abstract class*

This class is an abstract class from which all devices are inherited. As stated earlier, it contains all the code shared by devices.

**Send function**

The send function is responsible for sending data over the MQTT client (client). It publishes data to the things/data/:type/:id channel, which makes it easy for the receiving end to filter which type of data it wants to receive (for example subscribing to the topic things/data/streetlamp/+allows the subscriber to only listen to streetlamp data).

It knows what data to send by calling the getDataToSendmethod, which returns an object with the data to send. It is up to the implementation of the class to implement this method as the data is specific to the device.

**Tick function**

The tick function is where the code will a function that should be run at a regular interval (like 5 seconds). This piece of code should be responsible for collecting data, sending it to the server and making sure no errors have been thrown.

**Logic function**

This is where the main logic for the device will lie. With our streetlamp example, it’s there that you would check if presence was detected and in there that you would turn the lights on.

### Streetlamp Device

![Some of the code for the streetlamp device](https://cdn-images-1.medium.com/max/3260/1*NSuw83neGRIT_gP5AdmkDg.png)*Some of the code for the streetlamp device*

This is what an example device would look like. Notice the implementation of the logic function. The heavy use of Promise.all(...) allows me to scale the number of lights without heavily degrading performance.

### InfluxDB data

![InfluxDB visualisation](https://cdn-images-1.medium.com/max/3830/1*OPzaXFGbsw0d6cUhOKwd8A.png)*InfluxDB visualisation*

As mentioned above, InfluxDB comes with a nice-looking interface for data visualization and analysis. It contains many useful features, most notably the ability to aggregate data using many distinct functions and see multiple lines on a single chart while allowing us to filter data easily. The InfluxData team also creates their own query language called *Flux*, which is incredibly powerful when working with lots of data.

## Next Steps

While I’m happy about what I built there are a lot of features I could add to the project. Firstly, I could add more devices and functionality to my program to create a more realistic simulation. This should be easy because of the way I built the device classes. It would also be cool to test it in the real world with a GPIO library. Another important part of production deployment is monitoring the physical devices the code is deployed on. To monitor the devices and docker container, I could use the [Telegraf](https://www.influxdata.com/time-series-platform/telegraf/) agent (built by the same team from InfluxDB) which collects lots of data including number of processes running, disk I/O, RAM, CPU and battery usage, CPU temperature… All of this is important data to ensure that downtime is minimal and perform predictive maintenance.

## Further Reading

If this was interesting to you, here are some extra ressources:

* [My code on GitHub](https://github.com/tommcn/city-simulator)

* [Google’s Sidewalk Labs, who are also creating smarter cities](https://www.sidewalklabs.com/)

* [InfluxData website](https://www.influxdata.com/)

* [An O’Reilly chapter on Time Series tools](https://www.oreilly.com/library/view/time-series-databases/9781491920909/ch04.html)

*If you liked this article, consider following me on Medium, or connect with me on [LinkedIn](https://www.linkedin.com/in/tomas-mc/).*
