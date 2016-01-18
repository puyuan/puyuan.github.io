---
layout: post
title: InfluxDB Tag Cardinality and Memory Performance
categories:
- influxdb memory
---

Recently I began surveying the new era time-series databases for implementing real-time analytics at my workplace. Most of these support horizontal sharding, fullfilling a new demand for time-series big data--data with high volume, having some loosely defined schema. Druid.io is one of those, and so is influxDB. 

InfluxDB is written in GO, does not require external dependencies so installation was a breeze. It's SQL like syntax is easy to grasp. 

InfluxDB has an intriguing concept of tags (much like an index). In its [official docs](https://docs.influxdata.com/influxdb/v0.9/concepts/key_concepts/), it gives a sense of optimism for using tags.

> Tags are optional. You don’t need to have tags in your data structure, but it’s generally a **good idea** to make use of them because, unlike fields, tags are indexed. This means that queries on tags are faster and that tags are ideal for storing commonly-queried metadata.

However, there is a **huge caveat** here. In my implementation of a 200MB dataset, influxDB quickly consumed my entire 12GB of memory. Memory usage stayed at 95% and never seemed to be declining, not until I killed the process.  There was some discussion about memory leak and one on WAL log not flushed [fast enough](https://github.com/influxdata/influxdb/issues/3967). 

After several tests, it appeared to be another issue.  Memory consumption was immediately noticeable after loading the Grafana dashboard, which issued a couple of queries.  Mem usage shot from 10% to 90% almost crashing the machine. 
It appears that the problem is not memory leak, nor the WAL log not flushed fast enough. The real problem was due to tags, especially its cardinality. 

Here is a post on this [issue](https://groups.google.com/forum/?utm_medium=email&utm_source=footer#!searchin/influxdb/cardinality$20memory/influxdb/iyCFGavNaeU/zrfOqf6nCAAJ)
> If you have highly dynamic data, that should be stored as a field, not a tag. That makes it impossible to use with GROUP BY clauses, however, which I suspect is a non-starter for you.

>The system simply can't handle an ever-growing index without ever-growing RAM usage.

Voila, if this has been clearly written in the documentation next to tag usage, this could have saved a lot time. Basically, influxdb constructs an inverted index in memory, growing with the cardinality of the tags. 

In essence, I was defining sessionIds as tags, which is highly dynamic. In another measurement, I had three tags useragent fields, e.g. browser, os, device. 

When influxdb counts cardinality in a Measurement, it counts the combination of all tags. For example, if my measurement has the following tags: 3 os, 200 devices, 3 browsers, then the cardinality is  
  > 3x200x3=1800.  

Ofcourse 1800 is not huge, but if you had more devices, OSes and browsers, that number can grow over time. I now think twice before defining tags in my data, unless I am certain I will be using grouping, filtering, and that cardinality is low and predictable. 

In the case of user agents, you can break the os, device, browser into 3 measurements, and set the tags individually. This way the tags won't be counted as a combination of the 3. Most of the time, when analyzing user agents, you don't need to group 3 tags together, you just need to group them individually and plot them on separate graphs.   

InfluxDB is not that hard to use, but performance limit is real if you don't optimize. If somebody stumbled upon the same memory problems after a time-bound query, be sure to check the cardinality of your tags. Hope this helps. 
