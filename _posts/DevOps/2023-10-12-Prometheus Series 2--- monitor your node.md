---
layout: post
title: Prometheus Series 2 --- monitor your node!!!
subtitle: 
gh-repo: daattali/beautiful-jekyll
gh-badge: [star, fork, follow]
tags: [DevOps]
comments: true
---
# How Prometheus Work
![Prometheus_arch](https://honky-tonk.github.io/assets/img/architecture.png)

The image above shows the Arch of Prometheus. In this section, we will discuss how Prometheus works.

Firstly, a complete monitoring system architecture consists of several parts:
- monitor server
- monitor target
- alert
- visualization

The monitor server is the main component of the monitoring system. It is responsible for pulling/receiving monitor metrics from monitor targets and persistently storing them. If there are a large number of monitor targets, the monitor server needs the ability to discover these targets automatically. To store metrics, a professional database is required. In Zabbix, MySQL can be used as the metrics persistent store, although it is an RDBMS


in prometheus prometheus-server as monitor server, prometheus-server also have ability to collect data from monitor target, service discovery, most importent, prometheus-server use TSDB to store time series data, of course promethues-server support  a special Query langurage PromQL to query data for visualization tool

A monitor target can vary depending on the system being monitored. It could be a Pod, a node, an API, etc. In Prometheus, all you need to do is export the ```/metrics``` path and set the metric format. Afterward, Prometheus-server will collect the data from these targets. in prometheus there are four primary metric types PromQL can query
- **counters**:track **how often** an event occurs,Do not use a counter to expose a value that can decrease(counters metrics always up or clean to zero). For example, do not use a counter for the number of currently running processes
- **gauges**: as their name, gauges metrics type represents a single numerical value that can arbitrarily go up and down, so if we wanna counter the number of currently running processes, we should gauges type metric
- **histograms**:in prometheus histograms is slightly different with [histogram](https://en.wikipedia.org/wiki/Histogram) we know, cause all data prometheus get is time series, so the main different is bucket format, in prometheus bucket is **cumulative** it's very important, for example, we wanna stastistic http request as histogram use prometheus, bucket set as 1, 5, 10, then we get first bucket is http request under 1min, second bucket is http request under 5min(**cumulative** include number of bucket 1) etc..., in mathematic historam each bucket just only stastistic their own range, like bucket 1 is stastistic how many person height in range [150cm, 160cm], bucket 2 is stastistic how many person height in range [175cm, 185cm], in mathematic histogram is **NOT** cumulative 
- **summaries**: summaries is similar with cumulative, stastistic **cumulative** metric value, but summaries have a concept **Quantile function**, 


in prometheus architecture alert is support by alertmanager, alertmanager can use email, pagerduty etc... to send alert message out

after brief introduction, let get start monitor your node!!!


# node exporter
TODO

# write your own node expoter with golang
TODO
# test
TODO


