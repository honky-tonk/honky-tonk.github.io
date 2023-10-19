---
layout: post
title: Prometheus Series 3 --- prometheus federated
subtitle: 
gh-repo: honky-tonk/honky-tonk.github.io 
gh-badge: [star, fork, follow]
tags: [DevOps]
comments: true
---
# Preface
With a multiple cluster environment, to prevent single point failure, we highly recommend using Prometheus Federation to set up a monitoring cluster. The best Prometheus Federation solution involves having one Prometheus slave server per cluster. Each Prometheus slave server gathers metrics from its respective cluster, which are then pulled by the master Prometheus server with the desired metrics, also Prometheus federated have a lot of disadvantage we will discuss later.
# Topology
In our environment, a kubernetes cluster is existed, so we have one slave prometheus in kubernetes to collect metric which from kubernetes cluster, and a master prometheus server which exist in linux node directly
![Prometheus_arch](https://honky-tonk.github.io/assets/img/post3_1.jpg)

The master prometheus directly installed above Kubenetes node  

# Install prometheus in kubenetes with prometheus-operator
first we install prometheus-oprator in kubenetes 

```
root@k8s-master:~# kubectl create -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/master/bundle.yaml
customresourcedefinition.apiextensions.k8s.io/alertmanagerconfigs.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/alertmanagers.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/podmonitors.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/probes.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/prometheusagents.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/prometheuses.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/prometheusrules.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/scrapeconfigs.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/servicemonitors.monitoring.coreos.com created
customresourcedefinition.apiextensions.k8s.io/thanosrulers.monitoring.coreos.com created
clusterrolebinding.rbac.authorization.k8s.io/prometheus-operator created
clusterrole.rbac.authorization.k8s.io/prometheus-operator created
deployment.apps/prometheus-operator created
serviceaccount/prometheus-operator created
service/prometheus-operator created
```
before you Configure RBAC privileges, please confirm apiVersion
```
root@k8s-master:~/prometheus-operator# kubectl api-resources | grep clusterrole
clusterrolebindings                                  rbac.authorization.k8s.io/v1           false        ClusterRoleBinding
clusterroles                                         rbac.authorization.k8s.io/v1           false        ClusterRole
```
config RBAC privileges
```
root@k8s-master:~# mkdir prometheus-operator
root@k8s-master:~# cd prometheus-operator/
root@k8s-master:~/prometheus-operator# vim prome-rbac.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: prometheus
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: prometheus
rules:
- apiGroups: [""]
  resources:
  - nodes
  - nodes/metrics
  - services
  - endpoints
  - pods
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources:
  - configmaps
  verbs: ["get"]
- apiGroups:
  - networking.k8s.io
  resources:
  - ingresses
  verbs: ["get", "list", "watch"]
- nonResourceURLs: ["/metrics"]
  verbs: ["get"]
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: prometheus
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: prometheus
subjects:
- kind: ServiceAccount
  name: prometheus
  namespace: default

root@k8s-master:~/prometheus-operator# kubectl apply -f prome-rbac.yaml
serviceaccount/prometheus unchanged
```
> different between role and clusterroles:
>- A Role can only be used to grant access to resources within a single namespace.
>
>- A ClusterRole can be used to grant the same permissions as a Role, but because they are cluster-scoped, they can also be used to grant access to:
>- - cluster-scoped resources (like nodes)
>- - namespaced resources (like pods) across all namespaces


deploy two prometheus server(HA)
```
root@k8s-master:~/prometheus-operator# vim prometheus.yaml
apiVersion: monitoring.coreos.com/v1
kind: Prometheus
metadata:
  name: prometheus
  labels:
    app: prometheus
spec:
  image: quay.io/prometheus/prometheus:v2.22.1
  nodeSelector:
    kubernetes.io/os: linux
  replicas: 2
  resources:
    requests:
      memory: 400Mi
  securityContext:
    fsGroup: 2000
    runAsNonRoot: true
    runAsUser: 1000
  serviceAccountName: prometheus
  version: v2.22.1
  serviceMonitorSelector: {}
root@k8s-master:~/prometheus-operator# kubectl apply -f prometheus.yaml
prometheus.monitoring.coreos.com/prometheus created
```

deploy prometheus svc for prometheus login and service other monitor target

```yaml
root@k8s-master:~/prometheus-operator# vim prom_svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: prometheus
  labels:
    app: prometheus
spec:
  ports:
  - name: web
    port: 9090
    targetPort: web
  selector:
    app.kubernetes.io/name: prometheus
  sessionAffinity: ClientIP
root@k8s-master:~/prometheus-operator# kubectl apply -f prom_svc.yaml
```
the golang code we wanna deployment to 
```go
package main

import (
	"fmt"
	"log"
	"net/http"
	"os"

	"github.com/prometheus/client_golang/prometheus"
	"github.com/prometheus/client_golang/prometheus/promhttp"
)

var http_request_success_total = prometheus.NewCounter(prometheus.CounterOpts{
	Name: "http_request_success_total", //metric name
	Help: "access success count",
},
)

var http_request_total = prometheus.NewCounter(prometheus.CounterOpts{
	Name: "http_request_total",
	Help: "access count total",
})

var http_request_fail_total = prometheus.NewCounter(prometheus.CounterOpts{
	Name: "http_request_fail_total",
	Help: "access faile count",
})

func handler_for_hello_get(rw http.ResponseWriter, r *http.Request) {
	file, _ := os.OpenFile("access_log", os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0666)
	infologger := log.New(file, "INFO", log.Ldate|log.Ltime|log.Lshortfile)

	if r.Method != "GET" {
		fmt.Fprintf(rw, "handle get method only!!!\n")
		infologger.Println("connect from ", r.RemoteAddr, " Failed!!!")
		file.Close()
		http_request_fail_total.Inc()
		http_request_total.Inc()
		return
	}

	fmt.Fprintf(rw, "hello!!!\n")
	infologger.Println("connect from ", r.RemoteAddr, " Success!!!")
	http_request_success_total.Inc() // metric value add 1
	http_request_total.Inc()
	file.Close()
}

//func NewCountCollector * ()

func main() {
	prometheus.MustRegister(http_request_success_total)
	prometheus.MustRegister(http_request_total)
	prometheus.MustRegister(http_request_fail_total)

	http.Handle("/metrics", promhttp.Handler()) //for prometheus
	add := ":8080"
	fmt.Printf("start server at %v\n", add)
	http.HandleFunc("/hello", handler_for_hello_get)
	log.Fatal(http.ListenAndServe(add, nil))
}

```
the deployment yaml file 
```yaml
root@k8s-master:~/k8s_demo_config/golang_web_application# vim server_deplyment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: golang-server
  name: golang-server
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: golang-server
  strategy:
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: golang-server
        group: database
    spec:
      containers:
      - image: honkytonkman/server_in_k8s:1.2
        name: golang-server
        ports:
        - containerPort: 8080
          protocol: TCP
root@k8s-master:~/k8s_demo_config/golang_web_application# kubectl apply -f server_deplyment.yaml
```
the service file we have
```yaml
root@k8s-master:~/k8s_demo_config/golang_web_application# vim server_service.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: web
    type: service
  name: golang-server
  namespace: default
spec:
  ports:
  - name: service-web
    port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: golang-server
  type: NodePort
root@k8s-master:~/k8s_demo_config/golang_web_application# kubectl apply -f server_service.yaml
```

Finally, we should configure the monitoring configuration file for the Kubernetes service we just created."
```yaml
root@k8s-master:~/k8s_demo_config/promethues_monitor# vim service-monitor.yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: golang-service-monitor
spec:
  endpoints:
    - port: service-web
      path: /metrics
      interval: 15s
  namespaceSelector:
    any: true
  selector:
    matchLabels:
      app: web
root@k8s-master:~/k8s_demo_config/promethues_monitor# kubectl apply -f service-monitor.yaml
```



 

# Set federate in master prometheus
add follow part in  prometheus master server's config file
```yaml
  - job_name: "federate"
    scrape_interval: 15s

    honor_labels: true
    metrics_path: '/federate'

    params:
      'match[]':
        - '{__name__=~"job:.*"}'
        - '{__name__=~".+"}'

    static_configs:
      - targets:
        - "192.168.152.131:9091"

```
In above configuration params part,  ```'{__name__=~"job:.*"}'``` mean match all metrics which name start with jobs, ```'{__name__=~".+"}'``` mean match all metric but not match empty string(if use ```*``` instead of ```+```you will match empty string), or you can write ```'{job="golang-server"}'``` mean match matric which job name is ```golang-server``` 

Restart prometheus service
```
root@k8s-master:~# systemctl restart prometheus
```

Finally, we check  slave prometheus service status in master prometheus federated if up
![Prometheus_arch](https://honky-tonk.github.io/assets/img/post3_2.png)
![Prometheus_arch](https://honky-tonk.github.io/assets/img/post3_3.png)

Check the metric which Pulled from slave prometheus Server
![Prometheus_arch](https://honky-tonk.github.io/assets/img/post3_4.png)


# Disadvantage
- inefficient query:when a client query metrics, which metrics available in "leaf" or slaver cluster, we must proxy the request from client to "leaf" or slaver cluster's prometheus, this inefficient, and query must send to "root" master cluster's prometheus first, and proxy it. 
- inefficient for scaling:  you could set "root" or master cluster's prometheus gather slaver's metric together in memory, but it's not benefit for scaling. cause "root" or master's memory is limit
