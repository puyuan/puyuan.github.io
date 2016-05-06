---
layout: post
title: Big data architecture
categories:
- data mining
---

After working on big data pipelines for more than a year, I want to writeup a lesson learnt of what I have done right and done wrong. 

## Done wrong

### Not defining any data formats
 Literally, allowing any component owner to define their data schema, even unstructured data and parse data at the very end. It is very hard to fix data schema once they go into production, especially timestamp written in wrong timezone, missing fields, number precisions, lack of deduplication features. Unstructured data is a characteristic of big data, but doesn't mean it has to be that way. 
 
### Not focused on Dev-Ops
 Data collection components are very customized for each project--python scripts, configurations, parsers. They sort of get ignored by dev-ops because of their diverse nature.   Having to constantly tweak settings per project is a no-no for devops tool implementation. For for the same reason, its very important to auto-deployment of such complexity. 
 
### Using too many new frameworks
 We were introducing new frameworks like Druid very early on. Its hard to find people who know all new technologies. In fact, if we stick with Hive, maybe Spark, it would keep us in good shape. With the right data structure, you don't need very complicated new technology. 
 
### Not understanding big data vs average data
 Some projects are small, and can be analyzed using SQL. In fact SQL and Tableau is not a bad option. When data is huge, run Map-Reduce type of jobs, and consolidate. Don't run everything with Hive , even for small sized data. Its slow, and occupies your cluster. Unless you have unlimited cluster, consider using SQL for ordinary tasks. 
 
## Improvements
 
### Define formats early on, thinking about how you would process it
 
 If you know what metrics to analyze, define early on with component owners the schema and output. It takes little time for component owner to define json structure, timestamp formats to save big for the data team. Pick a nice data transport layer like fluentd, logstash, which can tag events that can be separated when it hits storage. 
 
### Data pipeline
 Some component owners think its a bad idea to dump everything to the data pipeline to be separated later on. Wouldn't it be less effort for me to write in a seperate specialized database to be analyzed. The downside is that eventually consolidation with other types of data will happen, i.e. data warehouse, and component owner have to think about how to organize historical data. Just write in the pipeline and have the data team take care of it. Write with good schema and tagged events. 
 
 Data pipeline is connected to Hadoop, S3, which acts as a nice backup storage. Recovery of data can be done easily. 
 
### Architecture
 Traditionally data job is ad-hoc with Spaghetti scripts. With newer frameworks like Flume, Fluentd, Logstash, its possible to do architecture and deploy configurations in a nice fashion. Same principle of refactoring code can be applied to data pipelines. Pull out repetitive common function into a new common configuration. e.g. similar types of parsing code, data schema shared across projects component. Some projects spit out new formats as they evolve, so define customized configurations for that. There is always common and customized configuration that can be refactored as the project evolve. 
 
 Draw an architecure of your data components. Don't think spaghetti. 
 
### Devops
 Again, use tool like puppet, Ansible to deploy frameworks and configuration. Never patch server by hand, alot of data folks like patching because of the ad-hoc nature of their work. Always patch the devop code e.g. Ansible, and deploy on to server. 
 
### Bulk loading
 We love streaming right to our target sink. Who hates realtime.  But sometimes database crash, pipeline fails. Bulk loading allows you to recover data from Hadoop S3, so have that in place. 
 
### EMR
 - run EMR when data gets too large and need consolidation and aggregation. 
 - if data is small, SQL, why not?
 - if you are a large company with unlimited scalability, run hadoop, presto, spark etc.. for all your data task. But if you are not, conserve resources, pick wisely. 
 
 
 
 
 
 
 
 
 
 