---
layout: post
title: Prometheus Series 1 --- Installation
subtitle: 
gh-repo: honky-tonk/honky-tonk.github.io
gh-badge: [star,follow]
tags: [DevOps]
comments: true
---
# Preface
Promethues is funded in 2012, and widly used since Cloud-Native become popular, in this article, we install prometheus above Ubuntu 21.04

why i didn't install prometheus in kubernetes use prometheus-operator? because i want to build a scalable monitor cluster which can scale as the number of cluster increase, so with first stage i install prometheus in ubuntu directly as master prometheus server

# Environment
- OS:Ubuntu 20.04.6
- IP:192.168.152.131
- hostname:k8s-master

# Start the journey
first of all we need an account for promethues service, this account have not home directory(```--no-create-home```), login shell do nothing(```--shell /bin/false```),and this account is system account(```-r```normally system account UID start at 1000)
```shell
root@k8s-master:~# useradd --system --no-create-home --shell /bin/false prometheus
```

after we create promethues account download promethues package is what we need , and extract

```
root@k8s-master:~# wget https://github.com/prometheus/prometheus/releases/download/v2.47.1/prometheus-2.47.1.linux-amd64.tar.gz
root@k8s-master:~# tar -zxvf prometheus-2.47.1.linux-amd64.tar.gz
prometheus-2.47.1.linux-amd64/
prometheus-2.47.1.linux-amd64/LICENSE
prometheus-2.47.1.linux-amd64/NOTICE
prometheus-2.47.1.linux-amd64/prometheus.yml
prometheus-2.47.1.linux-amd64/consoles/
prometheus-2.47.1.linux-amd64/consoles/prometheus.html
prometheus-2.47.1.linux-amd64/consoles/prometheus-overview.html
prometheus-2.47.1.linux-amd64/consoles/node-cpu.html
prometheus-2.47.1.linux-amd64/consoles/index.html.example
prometheus-2.47.1.linux-amd64/consoles/node.html
prometheus-2.47.1.linux-amd64/consoles/node-disk.html
prometheus-2.47.1.linux-amd64/consoles/node-overview.html
prometheus-2.47.1.linux-amd64/promtool
prometheus-2.47.1.linux-amd64/console_libraries/
prometheus-2.47.1.linux-amd64/console_libraries/prom.lib
prometheus-2.47.1.linux-amd64/console_libraries/menu.lib
prometheus-2.47.1.linux-amd64/prometheus
```


create promethues configure directory and data store directory
```
root@k8s-master:~# mkdir /etc/prometheus
root@k8s-master:~# mkdir /etc/prometheus/data
```

move promethues binary file under ```/usr/local/bin/```
```
root@k8s-master:~# cd prometheus-2.47.1.linux-amd64/
root@k8s-master:~/prometheus-2.47.1.linux-amd64# ls
console_libraries  consoles  LICENSE  NOTICE  prometheus  prometheus.yml  promtool
root@k8s-master:~/prometheus-2.47.1.linux-amd64#
root@k8s-master:~/prometheus-2.47.1.linux-amd64# mv promtool prometheus /usr/local/bin/
```

move promethues configuration file to ```/etc/promethues```
```
root@k8s-master:~/prometheus-2.47.1.linux-amd64# mv prometheus.yml /etc/prometheus/
```

set privilege of ```/etc/promethues```
```
root@k8s-master:~/prometheus-2.47.1.linux-amd64# chown  -R prometheus:prometheus /etc/prometheus
```

verification
```
root@k8s-master:~/prometheus-2.47.1.linux-amd64# prometheus --version
prometheus, version 2.47.1 (branch: HEAD, revision: c4d1a8beff37cc004f1dc4ab9d2e73193f51aaeb)
  build user:       root@4829330363be
  build date:       20231004-10:31:16
  go version:       go1.21.1
  platform:         linux/amd64
  tags:             netgo,builtinassets,stringlabels
```

last we use systemd manage prometheus service
```
root@k8s-master:~/prometheus-2.47.1.linux-amd64# vim /etc/systemd/system/prometheus.service
[Unit]
Description=Prometheus
Wants=network-oline.target
After=network-oline.target

StartLimitIntervalSec=300
StartLimitBurst=5

[Service]
User=prometheus
Group=prometheus
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/prometheus \
        --config.file=/etc/prometheus/prometheus.yml \
        --storage.tsdb.path=/etc/prometheus/data \
        --web.listen-address=0.0.0.0:9090 \
        --web.enable-lifecycle

[Install]
WantedBy=multi-user.target
root@k8s-master:~/prometheus-2.47.1.linux-amd64# systemctl enable prometheus
Created symlink /etc/systemd/system/multi-user.target.wants/prometheus.service → /etc/systemd/system/prometheus.service.
root@k8s-master:~/prometheus-2.47.1.linux-amd64# systemctl start prometheus
root@k8s-master:~/prometheus-2.47.1.linux-amd64# systemctl status prometheus
● prometheus.service - Prometheus
     Loaded: loaded (/etc/systemd/system/prometheus.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2023-10-11 20:17:48 CST; 7s ago
   Main PID: 17966 (prometheus)
      Tasks: 9 (limit: 6968)
     Memory: 16.4M
     CGroup: /system.slice/prometheus.service
             └─17966 /usr/local/bin/prometheus --config.file=/etc/prometheus/prometheus.yml --storage.tsdb.path=/etc/prometheus/data --web.listen-address=0.0>

10月 11 20:17:48 k8s-master prometheus[17966]: ts=2023-10-11T12:17:48.753Z caller=head.go:681 level=info component=tsdb msg="On-disk memory mappable chunks r>
10月 11 20:17:48 k8s-master prometheus[17966]: ts=2023-10-11T12:17:48.753Z caller=head.go:689 level=info component=tsdb msg="Replaying WAL, this may take a w>
10月 11 20:17:48 k8s-master prometheus[17966]: ts=2023-10-11T12:17:48.753Z caller=head.go:760 level=info component=tsdb msg="WAL segment loaded" segment=0 ma>
10月 11 20:17:48 k8s-master prometheus[17966]: ts=2023-10-11T12:17:48.753Z caller=head.go:797 level=info component=tsdb msg="WAL replay completed" checkpoint>
10月 11 20:17:48 k8s-master prometheus[17966]: ts=2023-10-11T12:17:48.755Z caller=main.go:1045 level=info fs_type=EXT4_SUPER_MAGIC
10月 11 20:17:48 k8s-master prometheus[17966]: ts=2023-10-11T12:17:48.755Z caller=main.go:1048 level=info msg="TSDB started"
10月 11 20:17:48 k8s-master prometheus[17966]: ts=2023-10-11T12:17:48.755Z caller=main.go:1229 level=info msg="Loading configuration file" filename=/etc/prom>
10月 11 20:17:48 k8s-master prometheus[17966]: ts=2023-10-11T12:17:48.756Z caller=main.go:1266 level=info msg="Completed loading of configuration file" filen>
10月 11 20:17:48 k8s-master prometheus[17966]: ts=2023-10-11T12:17:48.756Z caller=main.go:1009 level=info msg="Server is ready to receive web requests."
10月 11 20:17:48 k8s-master prometheus[17966]: ts=2023-10-11T12:17:48.756Z caller=manager.go:1009 level=info component="rule manager" msg="Starting rule mana>
```

# Install Grafana

now we install grafana and set to connection prometheus.

first we need the install dependencies

```
root@k8s-master:~# sudo apt-get install -y apt-transport-https software-properties-common
```

then add the apt-key we will use apt-key during grafana installnation
```
root@k8s-master:~# wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
```

get the grafana source list file
```
root@k8s-master:~# echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
``` 
then we update 
```
root@k8s-master:~# sudo apt-get update
```

final install grafana
```
root@k8s-master:~# apt-get install grafana
```
