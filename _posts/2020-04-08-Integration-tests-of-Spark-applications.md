---
layout: post
title: Integration tests of Spark applications
excerpt: Writing Apache Spark applications is no different than writing any other code - you should test it with both unit and integration tests. Unfortunately, even with the huge value they provide, integration tests of big data application are often skipped...
---

When you implement big data applications, for example using Apache Spark, you usually rely on existing data to create a proper topology of transformations. It helps to quickly create the code that runs on production well. But how can you be sure that next modification will not break the existing business logic? Having a nice set of unit tests (for example for UDFs) is not always enough and integrations tests set is the way to keep your code working.


[Read an article](https://getindata.com/blog/integration-tests-spark-applications-big-data) on how to implement classic `given/when/then` scenario for Spark applications. It is not that hard when done properly and brings a lot of value to existsing application.

Check it out! [https://getindata.com/blog/integration-tests-spark-applications-big-data](https://getindata.com/blog/integration-tests-spark-applications-big-data)
