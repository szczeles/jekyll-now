---
layout: post
title: GPU-based workloads as a part of Airflow DAGs
excerpt: Apache Airflow allows you to orchestrate not only the classic ETL jobs, copying data from one place to another, but it is also extremely helpful on running part of your pipeline on the specific hardware. For one of my projects I used it to run compute-heavy job on GPU nodes in AWS.
---

![Airflow-GPu](/images/airflow-gpu.png)

[Apache Airflow](https://airflow.apache.org/) fits great as a Spark jobs orchestrator. It can easily start applications on existing Hadoop clusters, or even start a new cluster for each job with [AWS EMR](https://aws.amazon.com/emr/) or similar on-demand clusters cloud providers. But can it be useful if there is a need to add some GPU-based heavy ML model training?

It turns out that when connected properly, Airflow DAGs allow you to run part of your process on custom GPU-based nodes inside AWS with [AWS Batch](https://aws.amazon.com/batch/) as a middleware. In this setup you pay only for the time you actually use the machine so it is very cost efficient as well. Read an article describing how I implemented a pipeline with mixed:

* classic Big Data proceses with Spark and
* custom step that trained the model using GPUs

in one of my projects at [Fandom](https://www.fandom.com/). Link to the article: [https://medium.com/fandom-engineering/gpu-based-workloads-as-a-part-of-airflow-dags-1d856418b529](https://medium.com/fandom-engineering/gpu-based-workloads-as-a-part-of-airflow-dags-1d856418b529)

