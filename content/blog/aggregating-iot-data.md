---
title: "Aggregating IOT data"
date: 2021-10-24T12:00:00+00:00
featureImage: https://miro.medium.com/max/1400/0*qwpfdDzTL0rcjSxD
postImage: https://miro.medium.com/max/1400/0*qwpfdDzTL0rcjSxD
# tags: c
categories: blog
toc: true
mediumLink: https://tommcn.medium.com/aggregating-iot-data-2b6dcf541d5f
---
# Aggregating IOT data

Introduction

Data runs the world. Each day, **2.5 million terabytes** of data are created by humans ([source](https://seedscientific.com/how-much-data-is-created-every-day)). The Internet Of Things (IOT) accounts for a huge portion of this statistic. The IOT is the collection of devices with embedded sensors and that collect and exchange data over the internet or similar protocols. IOT accounts for a large portion of the statistic mentioned above.

![Photo by [BENCE BOROS](https://unsplash.com/@benceboros?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/13440/0*qwpfdDzTL0rcjSxD)*Photo by [BENCE BOROS](https://unsplash.com/@benceboros?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

Making an IOT device is trivial. Using a Raspberry Pi or an Arduino and some sensors, anyone can build a device that checks temperature, humidity, movement… These types of project are cheap and are a great way for people to learn about programming and networking. But the power of IOT doesn’t stop there. IOT devices are used everywhere: boats, smart cars, farms, labs, oil rigs… With all this data being produced, we need to find a way to aggregate and analyze all of this data.

## My Solution

Some solutions exist to aggregate such data, the most notable of which is [ThingSpeak](https://thingspeak.com/). I set out to build a simplified clone of this service to learn about servers and data aggregation. I chose to only keep a subset of features which I found most useful.

My project (which I called ThingTotal) has a single goal: to receive and store data in a way that is easy to analyse and does so in an efficient way. I do so by providing users with “streams”, which are endpoints users can send their data too. “Streams” can be customized to verify the data is in an acceptable format using JSON schema, a tool that allows to annotate and validate JSON data. Each “stream” can have multiple “entries”, which are the individual pieces of data sent to the endpoint.

![Photo by [Chris Liverani](https://unsplash.com/@chrisliverani?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/7262/0*blKq0yIpGN7nT3pb)*Photo by [Chris Liverani](https://unsplash.com/@chrisliverani?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

## How I built it

To build this project I used many amazing open source technologies. The server is built using [Django](https://github.com/django/django). Django provides many features, the most notable of which is its ORM (Object Relational Mapping). Django’s ORM provides a layer of abstraction over the database, allowing us to use python classes and function to interact with the database without having to worry about the queries at a database level. When used properly, ORMs allow for greater confidence in the code and remove many security risks, the most notable of which is SQL injections attacks.

![Django logo](https://cdn-images-1.medium.com/max/2400/1*HVKOLLX7wprRbHTl2IPDcQ.png)*Django logo*

On top of Django, I used the [Django Rest Framework](https://github.com/encode/django-rest-framework) (DRF), a framework that helps turn your Django server into a REST API. A REST (Representational State Transfer) API (Application Programming Interface) is, to put it simply, each entry in the database (called ressource) has its own unique URL (ex. https://example.com/articles/12) and operations on these ressources are done using very specific types of requests. DRF makes it very easy to keep track of what's going on and keep clean urls:
```txt
GET <https://example.com/api/streams> -> List all the streams
POST <https://example.com/api/streams> -> Create a new stream

GET <https://example.com/api/streams/619b> -> List the stream with id 619b
PUT <https://example.com/api/streams/619b> -> Edit the stream with id 619b

GET <https://example.com/api/streams/619b/entries> -> List all the entries for the stream with id 619b
POST <https://example.com/api/stream/619b/entries> -> Create a new entry for the stream with id 619b
```

An other unique feature of DRF is that it is able to automatically create what’s called an [OpenAPI](https://www.openapis.org/) documentation file, which is a standard for documenting APIs. This makes it incredibly easy for teams to work together, where we could have one person working on the server (backend) and someone else working on the web page (frontend) and the designer would be able to see exactly how they should interact with the backend to get a specific ressource or utilize a certain feature.

Finally, for the database (the thing that stores all of the data) I decided to use a NoSQL database, namely [MongoDB](https://www.mongodb.com/). MongoDB is a NoSQL database, meaning the data isn’t stored in rows and columns (think excel spreadsheet). MongoDB stores data in “documents” which allows the program to store data as key-value pairs, which is closely linked to the way that the IOT information is received and sent.

![MongoDB logo](https://cdn-images-1.medium.com/max/2000/1*bQ9ODgAHWc3GVHTl_lw1DA.png)*MongoDB logo*

## My code

Here are the most important pieces of code. I wont go into all details of my code as they get somewhat complex, all of my code is available on [GitHub](https://github.com/tommcn/thingtotal).

Without questions the most important pieces of code are my models. A model is the class that defines how information is stored in the database.

I have 2 models. The first one is the model that stores my streams.

![Django model for Streams](https://cdn-images-1.medium.com/max/2484/1*W2yewbBGsv2Fy1MJEYl_jA.png)*Django model for Streams*

It has many fields. The _id field is used to uniquely represent each stream. The name and description fields are self-explanatory, they each allow for strings of up to 100 characters to be stored. The fields property is where users can define the data that they will be sending to the server. This allows the program to validate the data sent. Finally, the created_at field store the date and time the stream for created.

And I have a model for entries:

![Django model for Entries](https://cdn-images-1.medium.com/max/2752/1*H4X1oE03ecD_jr9sbSBp3g.png)*Django model for Entries*

This model is a bit more complex. The `_id` and `created_at` fields have the same purpose as in the Stream model. The stream field is what allows each entry to be linked to one Stream, it basically states that this entry is linked to that streams. It also allows us to then search the database for all entries that are linked to a specific Stream with ease and speed. The data field is where we store the data sent to the server from IOT devices. We also had a little piece of information, telling the program that when it needs to use the plural of Entry (for example in documentation), it should use "entries", instead of the naive "entrys".

## Examples

![Create a stream](https://cdn-images-1.medium.com/max/2000/1*O9P0sNmMitx0KPj4ScKgKQ.png)*Create a stream*

![List streams](https://cdn-images-1.medium.com/max/2000/1*9zv3OKayVEiEzSyo3vzNtA.png)*List streams*

![Stream detail](https://cdn-images-1.medium.com/max/2000/1*xF_i_TtfLoV0dnGpNPk4_A.png)*Stream detail*

![Add an entry to a stream](https://cdn-images-1.medium.com/max/2000/1*GnuI7i4k6zQ1BpesjYzGlA.png)*Add an entry to a stream*

![List entries for a specific stream](https://cdn-images-1.medium.com/max/2000/1*MucfMWk4IcgkT7tmYOvVTQ.png)*List entries for a specific stream*

## Conclusion

Through this project, I learnt about the power and NoSQL databases when used correctly. I also discovered the true power of many tools I have used in the past, especially MongoDB and Django Rest Framework. This is a (very) simplified clone of [ThingSpeak](https://thingspeak.com/), a great tool that I have used many times in the past. [Here’s a video a made about this project](https://youtu.be/1G8PSPte9xk).

*I hoped you liked this article. If you found this interesting, consider following me on Medium, subscribe to my [newsletter](https://tommcn.substack.com/) or connect with me on [LinkedIn](https://www.linkedin.com/in/tomas-mc/).*
