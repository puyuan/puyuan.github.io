---
layout: post
title: How to Properly Filter and Categorize Logs using Fluentd
categories:
- fluentd
---


# Background

In the projects I have worked on, data analytics usually came at the final stage of the product design. Log processing wasn't really standardized until frameworks like Flume, Fluentd, and Scribe emerged (Very recently). Not many companies know how to design the analytics properly. 

In my past projects, log data was simply collected from various servers and clients and dumped on to Hadoop and S3. Different types of unstructured events were mingled together, and basically filtered and crunched through by hive or mapreduce like frameworks. 

The myth of big data is unstructured data is OK. But when doing analytics, unstructured data is not OK, some sort of structure is needed. Rather than pushing the categorization and filtering of data at the final stage (e.g map reduce), categorizing data as it comes through the log pipeline (even better, create structure) is beneficial in the long run.  

# Fluentd
I used Flume extensively to move log data to S3. Flume is no doubt a robust, fault tolerant log framework, but when it comes to tagging, filtering and structuring log, its configurations are complex. 

So, Fluentd to the rescue. I connected the flume pipeline to fluentd to take advantage of its filtering plugins. Initially, I had some misconceptions of filters like grep, so here I will discuss some plugins that are useful. 
 

