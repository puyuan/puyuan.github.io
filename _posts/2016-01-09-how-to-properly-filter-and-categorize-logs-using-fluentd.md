---
layout: post
title: Properly Filtering and Categorizing Logs using Fluentd
categories:
- fluentd
---

The myth that big data is unstructured data (Variety, Veracity) is well-known. But when doing analytics, unstructured data is not effective, some sort of structure is needed for processing. Rather than pushing the categorization and filtering of data at the final stage (e.g map reduce), categorizing data as it comes through the log pipeline (even better, create structure) is beneficial in the long run.  

---

## Background

In my past projects, analytics was an important requirement, but could't be clearly defined until real data was available to be explored. Everyone knew logs have to be written for debugging and analytics later on, but to be written in a analytics-ready structure was usually an afterthought.  

Often, Logs from various servers and clients were collected (in a huge mess) and dumped on to Hadoop, and everyone hoped MapReduce will work its magic.

The myth that big data is unstructured data (Variety, Veracity) is well-known. But when doing analytics, unstructured data is not effective, some sort of structure is needed. Rather than pushing the categorization and filtering of data at the final stage (e.g map reduce), categorizing data as it comes through the log pipeline (even better, create structure) is beneficial in the long run.  

## Fluentd
I used Flume extensively to move log data to S3. Flume is no doubt a robust, fault tolerant log transport framework, but when it comes to tagging, filtering and structuring logs, its certainly not the best tool. 

So, Fluentd to the rescue. To leverage existing Flume framework, I connected Flume to Fluentd to take advantage of its filtering plugins. Flume's fluentd connector made that easy, so I could use Fluentd's extensive parser filtering plugin. 

## Grep Plugin

The grep plugin filters out messages like in linux grep, and is the first thing someone may look at for filtering messages. But apparently, it removed remaining messages which I would love to keep. Below, I explain the misconception. 

Input: 
{% highlight javascript %}
foo.bar { "host": "server01", "message": "login failure for user xxxx"}
foo.bar { "host": "server03", "message": "create account for user xxxx"}
{% endhighlight %}

Config:
{% highlight xml%}
<match foo.bar.**>
  type grep
  input_key message
  regexp login
  add_tag_prefix greped
</match>
{% endhighlight %}

Output:

{% highlight javascript %}
greped.foo.bar.logs { "host": "server01", "message": "login failure for user xxxx"}
{% endhighlight %}

When a record is dismissed by grep, say,  the 2nd entry above,  "create account for user xxx", it will be removed from the pipeline. It won't even be kept in foo.bar.logs event. Grep keeps the records you are interested, while removing all other records from the pipeline. 

Below we will talk about how to keep those dismissed entries as well using the rewrite tag filter plugin. 

## Rewrite Tag Filter
Rewrite Tag Filter is not written in Flume's official documentation, but to me, it plays an important role.
https://github.com/fluent/fluent-plugin-rewrite-tag-filter

Its categorizes log events into different tags, so you have a way of keeping all the entries but applying different tags. 

Input:
{% highlight javascript %}
foo.bar.logs { "host": "server01", "message": "login failure for user xxxx"}
foo.bar.logs { "host": "server03", "message": "create account for user xxxx"}
{% endhighlight %}

Config:
{% highlight xml %}
<match foo.bar.logs>
  type rewrite_tag_filter
  rewriterule1  login  login.foo.bar
  rewriterule2   *     remaining.foo.bar
</match>
{% endhighlight %}

Output:
{% highlight javascript  %}
login.foo.bar{ "host": "server01", "message": "login failure for user xxxx"}
remaining.foo.bar{ "host": "server03", "message": "create account for user xxxx"}
{% endhighlight %}

From above, all logs are kepts but applied different tags. That is useful when you have uncategorized logs coming through the pipeline. I use this extensively to mark different apache events coming through the pipeline, which can later be parsed.

## Conclusion
 Pick grep filter when you are interested only in certain events. Use rewrite filter for categorization. 




 

