Fluentd automatically appends timestamp at time of ingestion, but often you want to leverage the timestamp in existing log records for accurate time keeping. When 
ingesting, if your timestamp is in some standard format, you can use the **time_format** option in in_tail, parser plugins to extract it.

In my use cases, I often have logs written directly in epoch time as either seconds or milliseconds. Parsing seconds is straightforward, using the **%s** flag in time_format. Parsing milliseconds is trickier, and no straightforward way to parse it in fluentd currently. 

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
  tag raw.json.log
  format json
  # don't set any timekey here
</source>


<filter raw.json.log>
  @type record_transformer
  <record>
    timesecond "${some_time_key/1000}"
  </record>
</filter>

<match raw.json.log>
type parser
key_name timesecond
format  /(?<time>\d*)/
time_format %s
reserve_data yes
tag parsed.json.log
</match>

{% endhighlight %}

First you parse the json while ignoring the time_key. Then divide the time_key field by 1000, and lastly run the parser plugin again to set the time key. I think this is a long-winded hack, hopefully it can be improved in future versions. 
