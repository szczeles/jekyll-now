---
layout: post
title: Running Apache Spark on AWS
excerpt: If you use AWS and deploy big data jobs there with Apache Spark, you probably use AWS EMR. But, it is not the only maganed service you can use! For one of my clients I deployed a few jobs on AWS Glue and AWS Fargate, and they work much more efficiently than EMR-based ones. There is no "one tool fits all" for Big Data on AWS.
---

![Spark on EMR, Glue and Fargate](/images/spark-aws.png)

When you browse the Internet looking for methods of running [Apache Spark](https://spark.apache.org/) on [AWS](https://aws.amazon.com/) infrastructure you are most likely to be redirected to the documentation of AWS EMR (Elastic Map Reduce) service, which is Amazon's Hadoop distribution suited to run in AWS cloud environment. It's quite an easy way to deploy your data pipelines, but sometimes bootstrapping a huge cluster to perform simple ad-hoc analysis it's a cumbersome task.

They say *to a man with a hammer everything looks like a nail* and I felt into this trap with EMR once. I wrote an article, describing two other ways of running Apache Spark jobs on AWS-managed infrastructure - AWS Glue and AWS Fargate - that I use in my projects for [Acast](https://www.acast.com). You will find there the key differences between these methods when it comes to flexibility and pricing, showing why there is no place for "one service fits all" approach in AWS world.

Check out! [https://medium.com/acast-tech/running-apache-spark-on-aws-81a5f766d3a6](https://medium.com/acast-tech/running-apache-spark-on-aws-81a5f766d3a6)
