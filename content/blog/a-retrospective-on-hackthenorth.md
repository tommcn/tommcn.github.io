---
title: "A Look Back on HTN 2021"
date: 2021-10-06T12:00:00+00:00
featureImage: https://cdn-images-1.medium.com/max/2000/1*U5hvHglkpcyfaccate0ABg.png
postImage: https://cdn-images-1.medium.com/max/13824/1*Fsi3DWNjDMyUHHAEp6-uFw.jpeg
# tags: c
categories: blog
toc: true
mediumLink: https://tommcn.medium.com/a-retrospective-on-hackthenorth-2021-130f570461c6
---

# A Look Back on HTN 2021

TL;DR

We built an app that tracked people using security cameras in stores and won the **Best Entrepreneurship Hack** and **Best Use of Google Cloud** prizes. More information is available on our [devpost](https://devpost.com/software/hive-hq).

## **Introduction**

![](https://cdn-images-1.medium.com/max/2000/1*U5hvHglkpcyfaccate0ABg.png)

[HackTheNorth](https://hackthenorth.com/) is the biggest hackathon in Canada, with over 3500 participants and 447 projects submitted in 2021 on their [devpost](https://hackthenorth2021.devpost.com/). Being the biggest Canadian hackathon, HTN provides participants with a plethora of mentors and workshops. HTN is the perfect place to experiment with new technologies and build something innovative.

## **Ideation**

When looking for project ideas, we wanted to make something relevant to our current situation, be it the election or the reopening of the economy. We used news sites to look for current issues in our communities. When brainstorming hackathon ideas, we came up with a bunch of ideas that we decided not to pursue for multiple reasons:

![Photo by [CDC](https://unsplash.com/@cdc?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/8000/0*0LjM2RU9OLFgO78_)*Photo by [CDC](https://unsplash.com/@cdc?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

* A COVID school cases tracker: scrapped because of the lack of complexity (Ontario provides the raw data and people have already built pretty cool spreadsheets with them.

* A vaccine passport: Too many requirements regarding security and privacy, which our team members didn’t have experience with.

* An app calculating maximum room occupancy: Although Ontario’s guideline only talks about room size, true calculations require additional information such as airflow and room arrangement.

The following ideas were not pursued due to a lack of interest from team members:

* A news aggregator

* A news bias detector (someone built it [here](https://devpost.com/software/bias-detector-xkptnu))

* A political platform aggregator (someone built it [here](https://devpost.com/software/poliviews))

* A campaign promises tracker

* An online voting platform

## **Our project**

We finally decided to go with an application that would help businesses optimize their floorplan to be more COVID friendly, allowing them to open their stores safely. We achieved this by using machine learning to analyse security camera feeds and detect bottlenecks and high-traffic areas on the store’s premises. We called it “Hive-HQ”.

![](https://cdn-images-1.medium.com/max/13824/1*Fsi3DWNjDMyUHHAEp6-uFw.jpeg)

This idea didn’t come to fruition instantly, it took us a couple of hours to go from “helping stores reopen safely” to “detecting bottlenecks and high-traffic areas”. To staart, we jumped on the idea of using cameras to detect violations of COVID’s 2 meter “bubble”.

![Camera view showing a 2 meter radius around everyone](https://cdn-images-1.medium.com/max/2000/1*hqBohWVWFYB99AOHc1_Jvg.png)*Camera view showing a 2 meter radius around everyone*

After realizing this idea would only take a couple of hours, we decided to translate the detected people into an uploaded 2D floor plan. This is when the idea of adding a heatmap of high-traffic areas came into play. Since we already had the position of everyone in view of the camera, we were able to quickly add this feature to our project. From there, it was trivial for our team to add features which allowed users to connect multiple cameras and floor plans to create one big heatmap of the entire premise.

![The floor plan for the above camera view showing our feature allowing to stich together multiple camera shots](https://cdn-images-1.medium.com/max/2000/1*nCuvHck7cxKD7GmwYt1RtA.png)*The floor plan for the above camera view showing our feature allowing to stich together multiple camera shots*

Finally, we wanted to add some little statistics to our web app such as the number of people in the store and the average stay in the store, but we ran out of time and had to focus on other parts of the submission.

![Our UI design, implemented in vanilla React](https://cdn-images-1.medium.com/max/3840/1*jrvnFA8Nyfv6Ck1OwMtsRQ.png)*Our UI design, implemented in vanilla React*

One of the prizes we were running for was for the project with the best business viability (presented by Contrary). To qualify for it, we had to explain why our project could be a good business venture. For this reason, we decided to make our project cloud-friendly so businesses didn’t need strong servers to run our product, while allowing for privacy-weary business owners to run the code on-premise. With this in mind, we allowed ourselves to use some of Google Cloud most powerful servers and utilized their GPUs to their full extent.

## **Technical Side**

![The YOLOv5 splash picture](https://cdn-images-1.medium.com/max/4960/1*SMTbRfSSsPNXWcryLet-8g.jpeg)*The YOLOv5 splash picture*

For our backend, we opted to use the brand new [YOLOv5](https://github.com/ultralytics/yolov5) (You Only Look Once) arhcitecture, tweaked to only detect people and used custom algorithms to track individuals over multiple cameras. We tested [multiple](https://github.com/arvganesh/Multi-Camera-Object-Tracking) [different](https://github.com/JunweiLiang/Object_Detection_Tracking) [options](https://github.com/samihormi/Multi-Camera-Person-Tracking-and-Re-Identification) [to](https://github.com/KunalArora/multiple-camera_multiple-people_tracking) track individual users on security cameras, but they all required a specific environment, which would take too long to set up and iterate with, or they weren’t easily compatible with a Google Cloud’s GPU, which was a big requirement if we wanted good performance with multiple cameras. As for the cloud platform, we toyed around with Azure, but ran into several issues, mostly regarding Active Directory, since we didn’t have time to experiment longer, we chose to switch to Google Cloud because most of our team members had experience with it and free credits were provided for the competition.

Our frontend was built with React and vanilla CSS. Before we actually started coding, we built a high-fidelity design using Figma, which we were able to replicate fairly easily afterwards, while keeping a distinctive look and feel to our web app. We did run into problems when we wanted to preserve state in the app. Because of the way the information we wanted to preserve, we couldn’t simply store it in localStorage, so, in the interest of time, we decided to not replicate state between server and client, and instead just fetch state on every page reload and stream the various camera streams directly from our AI server.

<iframe src="https://medium.com/media/8af87ba8a92c3f7d96c1a6cc1c7b1b4c" frameborder=0></iframe>

## **Lessons learned**

As with any project, we learnt a lot during this hackathon. The first big lesson we learned was time management. Although we were allowed to start brainstorming before the hackathon started, we only had 36 hours to write our implementation. Of those 36 hours, we spent a good chunk of time troubleshooting problems with the servers, team creation on google cloud and fighting with Azure. We also left the creation of the video to the last minute, which left us scrambling to record, edit and upload it to the devpost. Thankfully we were able to send our submission literally 1 minute before the deadline (but as is customary in many hackathons, the submission deadline was pushed by 1 hour. One problem we also faced when connecting the front and back end was documentation. The backend was implemented as an API server, however, we didn’t have a document outlining how to use it and all the different endpoints. This meant the frontend team had to read the server’s source code to understand how it worked. Thankfully, our frontend team did have some experience with Flask, the webserver we were running.

However, we believe that we did multiple things right. We used the live share feature of Visual Studio Code, which allowed us to all edit code on one person’s computer. This allowed us to iterate extremely quickly and not have to deal with git merge conflicts. Once we got our VM working, we immediately ran the VSCode live share server off of it. We also separated our work very efficiently among ourselves. We also had our features laid out in incremental steps. This allowed us to work on features starting with a minimum viable product and slowly add features to our app. This helped us stay on track and not add features just for the sake of it when we didn’t even have a working proof of concept.

## **What now**

Although our project was built with COVID as the primary concern, our app can actually be useful in the post-COVID world. Since we are able to track a customer’s journey across the store, we are able to see which products do better and suggest moving items to different locations to optimize revenue in the store. We know there are already products that do that, but our solution would be significantly cheaper as it would be fully automatic.

The code we wrote is understandably not optimized. We have plenty of ideas to make our product easier to deploy, maintain and use. Currently, all of the information stored on the server is stored in a single dictionary, which doesn’t allow for horizontal scaling. An improvement could be to use an in-memory database such as Redis, which would allow for multiple hosts to connect to it. On the topic of scalability, with our current code structure, it wouldn’t be very difficult to make the server a container that could be deployed on Docker or on Kubernetes. Some of our front-end code may need some refactoring but is overall pretty well structured.

## **Conclusion**

During this hackathon, we learnt a lot about project/time management during this hackathon, mainly how to divide tasks in a team and the importance of good communication and documentation. We also got to experiment with some of the newest AI technologies out there and implemented complex mathematics in an efficient manner.

Check out the other project submitted to HackTheNorth [here](https://hackthenorth2021.devpost.com/).

Connect with us on LinkedIn:

* Tomas: [https://www.linkedin.com/in/tomas-mc/](https://www.linkedin.com/in/tomas-mc/)

* Edward: [https://www.linkedin.com/in/edwardjxli/](https://www.linkedin.com/in/edwardjxli/)

* Victoria: [https://www.linkedin.com/in/victoriadarosa/](https://www.linkedin.com/in/victoriadarosa/)

* Lefan: [https://www.linkedin.com/in/lefan-hu-579591204/](https://www.linkedin.com/in/lefan-hu-579591204/)
