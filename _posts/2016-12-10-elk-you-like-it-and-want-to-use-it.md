---
layout: post
published: false
mathjax: false
featured: false
comments: false
title: 'ELK - you like it and want to use it ? '
description: Some thoughts on using ELK in a real application
tags: elk
---
Starting out with ELK is pretty straightforward and it usually highlights the benefits of a centralised loggic solution. Once you see this you will want to consider a few of the implications of running ELK in a scalable production environment.

Here's what I tried.

## We can forward logs directly to elasticsearch

I used NLog and a custom elasticsearch target to get things going. This allows you to quickly view and explore the logs with Kibana. NLog can be substituted with Log4Net or you can interact directly with ES through the Elasticsearch.net or even write your own client.
In this scenario the application is responsible of sending logs directly to Elasticsearch and we have coupling between the application and the ES cluser. 


Will the ES cluster availability impact application performance?

It depends on the actual implementation for the log sending, this must be non-blocking.
The NLog target seems to handle this well, it will not block your application when it can not deliver logs, provides configuration for exception management and a failover mechanism.

// config here

Data integrity

Beside application perfromance, another aspect is data integrity, more exactly not loosing logs when ES is down.
This can be prevented by using another target to store logs in a different location, maybe a file.
The FallBackGroup target can also be used, details here https://github.com/nlog/NLog/wiki/FallbackGroup-target.

Here is a link to a SO discussion about the NLog elasticsearch target:
http://stackoverflow.com/questions/40807162/application-logging-with-elk-stack/40810299#40810299


Data quality

If you want a real time web dashboard to access logs this can be a good solution, but if you want to take advantage of all the visualisations Kibana provides you will need to structure and process the logs you send.
The default option with ELK for log processing is Logstash.

## Why use Logstash

Processing logs

With application logs you have a level of control over the format or you can use structured logging to help with data analysis. In other cases you will want to analyse logs from external systems, let's say ELB, and at that point log processing (Logstash) will be required.


Logstash

So it helps with logs processing, but what exactly does that mean ?

Sample ELB log entry:

This type of log entry is not ideal for data analysis, the long text message needs to be broken down in different parts. This is where Logstash filters come in.

The grok filter can be used to parse the log message.

Sample Logstash config:
***



## Logstash at scale


Here is a good article with a few different options:
https://www.elastic.co/guide/en/logstash/current/deploying-and-scaling.html




## Elasticsearch cluster availability considerations

I've used Elasticsearch as a service from AWS with a single node and for a production environment this needs to be a multi node cluster. The master node is responsible for cluster management and limiting the writes to this mode is a good ideea.

AWS takes care of a lot of the complexity that comes with managing ES but we will still need to manage the data.
AWS provides automatic and manual snapshots that can be restored but this are high level backups and more related to the infrastucture than the data.
To back up the data and clean old indexes we can schedule jobs that work with the elasticsearch APIs.

eg:




## Managed ELK services







