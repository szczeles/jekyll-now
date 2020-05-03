---
layout: post
title: Anomaly detection in podcasting
excerpt: Detecting the anomalies in incoming data is one the main tasks of Data Engineer. Recently I implemented the detection and notification system for one clients, Acast. It turned out that simple logarithm curve fitting solves most of the challenges.
---

![Anomalies detection results](/images/acast-anomalies-detection.png)

Being a Data Engineer is not only about moving the data but also about extracting value from them. Read an article on how I implemented anomalies detection for one of our my clients, [Acast](https://www.acast.com), in the podcasting industry. With the mechanisms I implemented, Acast employees are notified on the podcasts that suddenly gain or lose the audience.

The results reveal trendsetting podcasts (the influencing ones, that grow episode after episode), hot topics (on shows that publish an episode that is listened all over the world) and also weird podcasts applications behaviour (sudden spikes in the listening pattern). All these anomalies notifications are based purely on the data and simple logarithm curve fitting - it's unbelievable what you can achieve with data&math!

Check out! [https://medium.com/acast-tech/anomaly-detection-in-podcasting-543f4f01f6a0](https://medium.com/acast-tech/anomaly-detection-in-podcasting-543f4f01f6a0)
