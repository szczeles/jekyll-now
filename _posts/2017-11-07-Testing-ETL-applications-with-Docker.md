---
layout: post
title: Testing ETL applications with Docker (Part 1)
excerpt: here here TODO
---

You just completed an ETL application in [Apache Hadoop](TODOapachehadoop) environment. Your application is moving files around HDFS, implements some business logic to place them properly. You used python that executes shell `hadoop` commands or some pythonic library (like [snakebite](TODOsnakebitelink) or [hdfslib3](TODOotherpythonlibrarylink)), maybe this is a Java application using `FileSystem` API or even a plain UNIX shell script with a bunch of greps and condition statements. Or maybe you used a high-level library like [pyspark](https://spark.apache.org/docs/latest/quick-start.html) and wrote the code in [Jupyter](http://jupyter.org/) as I [used to do](https://www.slideshare.net/MariuszStrzelecki/one-jupyter-to-rule-them-all) in Allegro. You package the application, next you schedule it to be triggered by [Oozie](TODOoozielink) or [Airflow](TODOairflowlink). You want to monitor all executions, so your applications puts some metrics into [Graphite](TODOgraphitelink), [Elasticsearch](TODOelasticlink) or [OpenTSBD](opentsdblink). In the next days you observe the job execution, it works as expected and deals smoothly with a large number of files. You are happy, the big data are simpler than you imagined! :-)

After a month or two you get a task to slighly change a business logic inside your application. On the first look it looks pretty straightforward. But then you realize that you developed the application on green field. If execution went wrong, you were able to remove resulting directories and re-run. Nobody used the data you produced during initial development, so you were free to do anything. Now, when data are widely used in another projects and ETL is executed once a hour, you just cannot simply validate your modifications at any time. The first idea is to copy data somewhere else and perform code change on these. But even if the change you are going to implement looks easy, it may interfere with the whole application flow, so you probaby need to re-test all assumptions and conditions. Now you know, it will not be as easy as you thought.

After a year you receive an e-mail from Hadoop ecosystem maintainers that they are going to upgrade whole distribution. You are kindly asked to check if all your applications works without any issue on new Hadoop. You implemented most of them some time ago, so probably you cannot easily recall all conditions and exceptions you covered. And now you have four weeks to test all ETLs against new environment.

If neither of above scenarios happened to you - trust me - sooner or later you will face them. There are some techniques that ease ETLs maintanence. One of them is creating unit tests for classes that implement the logic inside application. But if the loogic looks like "move this file", "rename this file", "join the two datasets" - testing it require a lot of mocking and - believe me or not - you will not be satisfied looking and resulting code ;-) Another method is to keep in-sync separate "staging" cluster, where ETLs can be perfomed without any harm, because nobody is using the results. But this is manual way and requires to keep up-to-date set of how application should behave on different datasets. The third method I like the most and I would liek to tell you about is...

## Integration testing in Docker Environment

Start with listing dependencies of your ETL. These might be:

* HDFS, as you use it to read or write the data
* hadoop client library -  




{% highlight python %}
from pyspark.sql.types import *
{% endhighlight %}


 - developing locally
 - integration testing
 - hadoop version upgrade
