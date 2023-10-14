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
when we want to monitor target we shoud export their metric path ```/metrics``` and set metric it self there are lot of exporter available in [github](https://github.com/prometheus/node_exporter) but in this tutorial we will write your own exporter by golang, and use in prometheus and visualization in grafana

# write your own node expoter with golang
the metric we wanna monitor is 
- 1 min cpu avg
- 5 min cpu avg
- 15 min cpu avg

code as follow
```golang
package main

import (
    "fmt"
    "log"
    "net/http"
    "os/exec"
    "strconv"
    "sync"
    "time"

    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promhttp"
)

var cpu_1_min_avg_load = prometheus.NewGauge(
    prometheus.GaugeOpts{
      Name: "cpu_1_min_avg_load", //metric name
      Help: "cpu 1 min avg load ",
    },
)

var cpu_5_min_avg_load = prometheus.NewGauge(
    prometheus.GaugeOpts{
      Name: "cpu_5_min_avg_load", //metric name
      Help: "cpu 5 min avg load ",
    },
)

var cpu_15_min_avg_load = prometheus.NewGauge(
    prometheus.GaugeOpts{
      Name: "cpu_15_min_avg_load", //metric name
      Help: "cpu 15 min avg load ",
    },
)

func get_cpu_1_min_avg_load_metric(wg *sync.WaitGroup) {
    wg.Done()
    fmt.Println("start collect 1 min avg cpu load each sec")
    for {
      cmd := "/usr/bin/uptime | /usr/bin/awk -F ':' '{ print $NF }' | awk -F ',' '{print $1}'| sed 's/ //'"
      result, err := exec.Command("bash", "-c", cmd).Output()
      if err != nil {
        fmt.Println("get metric error: ", err.Error())
      }
      s := string(result) // convert []byte to string, attention string contain '\n' in last character
      s = s[:len(s)-1]    // cut last '\n' otherwise yout got error when you parse to float
      f, err := strconv.ParseFloat(s, 64)
      if err != nil {
        fmt.Println("convert err: ", err.Error())
      }
      cpu_1_min_avg_load.Set(f)
      time.Sleep(1 * time.Second)
    }
}

func get_cpu_5_min_avg_load_metric(wg *sync.WaitGroup) {
    wg.Done()
    fmt.Println("start collect 5 min avg cpu load each sec")
    for {
      cmd := "/usr/bin/uptime | /usr/bin/awk -F ':' '{ print $NF }' | awk -F ',' '{print $2}'| sed 's/ //'"
      result, err := exec.Command("bash", "-c", cmd).Output()
      if err != nil {
        fmt.Println("get metric error: ", err.Error())
      }
      s := string(result) // convert []byte to string, attention string contain '\n' in last character
      s = s[:len(s)-1]    // cut last '\n' otherwise yout got error when you parse to float
      f, err := strconv.ParseFloat(s, 64)
      if err != nil {
        fmt.Println("convert err: ", err.Error())
      }
      cpu_5_min_avg_load.Set(f)
      time.Sleep(1 * time.Second)
    }
}

func get_cpu_15_min_avg_load_metric(wg *sync.WaitGroup) {
    wg.Done()
    fmt.Println("start collect 15 min avg cpu load each sec")
    for {
      cmd := "/usr/bin/uptime | /usr/bin/awk -F ':' '{ print $NF }' | awk -F ',' '{print $3}'| sed 's/ //'"
      result, err := exec.Command("bash", "-c", cmd).Output()
      if err != nil {
        fmt.Println("get metric error: ", err.Error())
      }
      s := string(result) // convert []byte to string, attention string contain '\n' in last character
      s = s[:len(s)-1]    // cut last '\n' otherwise yout got error when you parse to float
      f, err := strconv.ParseFloat(s, 64)
      if err != nil {
        fmt.Println("convert err: ", err.Error())
      }
      cpu_15_min_avg_load.Set(f)
      time.Sleep(1 * time.Second)
    }
}

func main() {
    //registe all metric
    prometheus.MustRegister(cpu_1_min_avg_load)
    prometheus.MustRegister(cpu_5_min_avg_load)
    prometheus.MustRegister(cpu_15_min_avg_load)

    wg := sync.WaitGroup{}
    wg.Add(3)
    go get_cpu_1_min_avg_load_metric(&wg)
    go get_cpu_5_min_avg_load_metric(&wg)
    go get_cpu_15_min_avg_load_metric(&wg)

    http.Handle("/metrics", promhttp.Handler())
    fmt.Println("service starts listening on port 8080")
    log.Fatal(http.ListenAndServe(":8080", nil))

}
```
there are serval thing we should notice in above code
- in line 42, 62, 82, we use ```bash -C``` to excution our cmd, cause the cmd will be excuted in a subshell enviroment
- ```exec.Command().Output``` return a byte slice, i can't find any way to convert byte to float64 direct, so in the first convert the byte slice to string, then we use ```strconv.ParseFloat()```convert to float64, and be  attention, the string we get after convert, there exist ```'\n'``` end of string, so we should cut ```'\n'``` off before convert to float64 
# test
TODO


