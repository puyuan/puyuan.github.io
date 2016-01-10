Fluentd automatically appends timestamp at time of ingestion, but often you want to leverage the timestamp in existing log records for accurate time keeping. When 
ingesting. In your timestamp is properly formated, you can use the time_format option in in_tail, parser plugins etc.. to extract it.

In my use cases, I have epoch time written in seconds or milliseconds. Parsing seconds is straightforward, just use the %s in time_format. 

## Parsing Seconds
{%highlight xml%}
<source>
  @type tail
  path /var/log/json.log
  tag some.json.log
  format json
  time_key some_time_key
  time_format %s  # here is the second format
</source>
{% endhighlight %}

time_key chooses the field that holds the datetime format. time_format is epoch time in seconds. 

What if your format is milliseconds. Fluentd currently doesn't have a format string to process it. Some record transformation is needed. 

## Parsing in milliseconds

{%highlight xml%}
<source>
  @type tail
  path /var/log/json.log
  tag some.json.log
  format json
  # don't set any timekey here
</source>


<filter foo.bar>
  @type record_transformer
  <record>
    time "${some_time_key/1000}"
  </record>
</filter>


{% endhighlight %}
