---
layout: post
title: Kubenetes network 1
subtitle:
gh-repo: honky-tonk/honky-tonk.github.io 
gh-badge: [star,follow]
tags: [DevOps]
comments: true
---
# Kubenetes Network Overview 
The network is often referred to as the  "dark side" of kubernetes, Actully is NOT hard to understand if you have knowledge about linux bridge and basic netowrk.

So what is bridge network in linux? 
>A bridge is a way to connect two Ethernet segments together in a protocol independent way. Packets are forwarded based on Ethernet address, rather than IP address (like a router).
>
>--linux foundation 

This seem not hard to understand, In a straitforward way, You could imagine a bridge in a linux as a virtual switch, the interface of bridge could seen as interface of switch, Of course bridge support vlan stp etc... You could use ```brctl show``` or ```ip link bridge ```  to show or set or del or add bridge in linux.

## Intra Node(Communication Intra Node)
In nowaday, Linux bridge is most common used in docker, containerd etc..., In this container engine linux bridge solve the intra node communication(between the Pod/container in a the same node), As previous section mention, The bridge could seem as virtual switch, each bridge's veth could seem as interface of switch, Now two Pod/container in a same node communication is not mystery, the veth is existed in each Pod/container, so they can communication with each other via bridge!

![cidr](https://honky-tonk.github.io/assets/img/20231031_1.png)

In my enviroment, I have 3 node(one master two worker), my CNI is flannel, in my master node i have two bridge list below
```
root@k8s-master:~# brctl show
bridge name     bridge id               STP enabled     interfaces
cni0            8000.06fa58e2aaae       no              veth9391b9a3
                                                        vethaa0e92ac
docker0         8000.024284edead3       no
```
All my Pod(which use pod cidr network **NOT** node ip and forword traffic via port of node) talk each other via bridge which name is cni0, Also we could know how many veth the bridge have(each veth exist in a pod)  
```
root@k8s-master:~# brctl show cni0
bridge name     bridge id               STP enabled     interfaces
cni0            8000.06fa58e2aaae       no              veth9391b9a3
                                                        vethaa0e92ac
```

```
root@k8s-master:~# ip a
...
...
...
5: cni0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default qlen 1000
    link/ether 06:fa:58:e2:aa:ae brd ff:ff:ff:ff:ff:ff
    inet 10.244.0.1/24 brd 10.244.0.255 scope global cni0
       valid_lft forever preferred_lft forever
    inet6 fe80::4fa:58ff:fee2:aaae/64 scope link
       valid_lft forever preferred_lft forever
6: vethaa0e92ac@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master cni0 state UP group default
    link/ether ba:17:34:4e:18:6e brd ff:ff:ff:ff:ff:ff link-netns cni-0874ce43-fb7d-9350-e4ba-5e52003f209c
    inet6 fe80::b817:34ff:fe4e:186e/64 scope link
       valid_lft forever preferred_lft forever
7: veth9391b9a3@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master cni0 state UP group default
    link/ether a6:49:90:0d:68:c0 brd ff:ff:ff:ff:ff:ff link-netns cni-789ef80d-204f-00ea-a4d3-9c1f4d0dbc84
    inet6 fe80::a449:90ff:fe0d:68c0/64 scope link
       valid_lft forever preferred_lft forever
```
Then we list all the Pod my master node exist, There is 8 Pod exist but why we only have 2 veth? cause in master node only two pod exist with pod-cidr network(my pod-cidr network is 10.244.0.74/16), Other Pod work with node ip and different port 
```
root@k8s-master:~# kubectl get pod --all-namespaces -o wide | grep master
kube-flannel   kube-flannel-ds-8px9l                               1/1     Running            17 (53m ago)   84d   192.168.152.131   k8s-master    <none>           <none>
kube-system    coredns-7bdc4cb885-6kzwr                            1/1     Running            11 (53m ago)   33d   10.244.0.74       k8s-master    <none>           <none>
kube-system    coredns-7bdc4cb885-g5n5w                            1/1     Running            11 (53m ago)   33d   10.244.0.75       k8s-master    <none>           <none>
kube-system    etcd-k8s-master                                     1/1     Running            11 (53m ago)   84d   192.168.152.131   k8s-master    <none>           <none>
kube-system    kube-apiserver-k8s-master                           1/1     Running            12 (53m ago)   84d   192.168.152.131   k8s-master    <none>           <none>
kube-system    kube-controller-manager-k8s-master                  1/1     Running            11 (53m ago)   84d   192.168.152.131   k8s-master    <none>           <none>
kube-system    kube-proxy-rzd8d                                    1/1     Running            11 (53m ago)   84d   192.168.152.131   k8s-master    <none>           <none>
kube-system    kube-scheduler-k8s-master                           1/1     Running            11 (53m ago)   84d   192.168.152.131   k8s-master    <none>           <none>
```
 

# Flannel

As we mention above, In our enviorment we got 3 node one master two slaver, My pod-clustere-cidr is 10.244.0.0/16, In Kubernetes, Each node have their own subnet, If we pod-clustere-cidr is 10.244.0.0/16, the subnet of master node may as 10.244.0.0/24, work1 node may as 10.244.1.0/24, work2 node may as 10.244.2.0/24, The Pod exist in master node may assigned 10.244.0.1/24 ip, The Pod exist in work1 node may assigned 10.244.1.1/24 ip, The Pod exist in work2 node may assigned 10.244.2.1/24 ip

![cidr](https://honky-tonk.github.io/assets/img/20231031_3.png)

We can vertify that via follow command 

```
root@k8s-master:~# kubectl cluster-info dump | grep cidr
                            "--allocate-node-cidrs=true",
                            "--cluster-cidr=10.244.0.0/16",
I1101 10:30:38.543018       1 shared_informer.go:311] Waiting for caches to sync for cidrallocator
I1101 10:30:38.543064       1 shared_informer.go:318] Caches are synced for cidrallocator
root@k8s-master:~# kubectl cluster-info dump | grep -i cidr
                "podCIDR": "10.244.0.0/24",
                "podCIDRs": [
                "podCIDR": "10.244.1.0/24",
                "podCIDRs": [
                "podCIDR": "10.244.2.0/24",
                "podCIDRs": [
                            "--allocate-node-cidrs=true",
                            "--cluster-cidr=10.244.0.0/16",
```
So now, we Know the Pod communicate Cross Node is via layer 3 network, In this section we will talking about Kubernetes CNI Flannel.

>Flannel is a simple and easy way to configure a layer 3 network fabric designed for Kubernetes.

![cidr](https://honky-tonk.github.io/assets/img/20231031_2.png)

All Flannel responsibilities is for layer 3 network and **NOT** Support  network policy, If you wanna network policy feature you could looking for other CNI plug, like Cilium, Calico

Flannel support multi-way of layer 3 network, Like VXLAN, host-gw, UDP, Once set, the backend should **NOT** be changed at runtime
>VXLAN is the recommended choice. host-gw is recommended for more experienced users who want the performance improvement and whose infrastructure support it (typically it can't be used in cloud environments). UDP is suggested for debugging only or for very old kernels that don't support VXLAN. 

## VXLAN
We will not unfold the detail of VXLAN, But highly recommand reader to read the VXLAN document from [Arista Network](https://www.arista.com/assets/data/pdf/Whitepapers/Arista_Networks_VXLAN_White_Paper.pdf)

In Kubernetes, the VTEP is hosts kernel, The VTEP is in charge of encapsulate VXLAN header and UDP header

**NOTICE!**

Although Multicast is Most choice of VXLAN implement of first arp, BUT flannel NOT support Multicast, because flannel work in ip layer, multicast work in link layer and "most cloud environments don't support multicast so it would require emulation by replicating the packet at the source, which is very inefficient. " -- eyakubovich

So How flannel implement VXLAN? We could learn from [github](https://github.com/flannel-io/flannel/blob/master/pkg/backend/vxlan/vxlan.go)

>// How it works:
>
>// Create the vxlan device but don't register for any L2MISS or L3MISS messages
>
>// Then, as each remote host is discovered (either on startup or when they are added), do the following
>
>// 1) Create routing table entry for the remote subnet. It goes via the vxlan device but also specifies a next hop (of the remote flannel host).
>
>// 2) Create a static ARP entry for the remote flannel host IP address (and the VTEP MAC)
>
>// 3) Create an FDB entry with the VTEP MAC and the public IP of the remote flannel daemon.


When a node start, Flannel will store create static ARP entry, Forward Table, routing Table for the node and  store this entry in database(etcd?)

Once a kubernetes Node is Up, the route to the new Node is create in each other node in the cluster, We could vertify by this command
```
root@k8s-master:~# ip route
default via 192.168.152.2 dev ens32 proto static metric 100
10.244.0.0/24 dev cni0 proto kernel scope link src 10.244.0.1
10.244.1.0/24 via 10.244.1.0 dev flannel.1 onlink
10.244.2.0/24 via 10.244.2.0 dev flannel.1 onlink
169.254.0.0/16 dev ens32 scope link metric 1000
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown
192.168.152.0/24 dev ens32 proto kernel scope link src 192.168.152.131 metric 100
```

When inter Node communicate hapen, every package send by pod will go through flannel.1 interface, then send to ens32 interface

We can use tcpdump or tshark tool to capture package

How to know what kind of backend your flannel use?
get configmap of flannel you will got it
```
root@k8s-master:~# kubectl get configmaps -n kube-flannel  kube-flannel-cfg -o yaml | grep Type
        "Type": "vxlan"
```

# How K8S Service Work?
In the nutshell, K8S Service Power By Kube-proxy(Most of CNI Use kube-proxy), and kube-proxy Power by Iptables or ipvs, in this section we will explain how kube-proxy work(Iptables mode).

First of all kube-proxy exist as daemonsets in namespace kube-system
```
root@k8s-master:~# kubectl get daemonsets -n kube-system
NAME         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-proxy   3         3         3       3            3           kubernetes.io/os=linux   91d
``` 

Which mean kube-proxy will exist in each k8s node, When i create a kubenetes services, the kube-proxy each node will create forward rule to service backend(loadbalance), and loadbalance implement by iptables, However iptables did not support loadbalance native, So this implement is complex and low proformence, To address this issue we will discuss Cillium in next article, which use  new technology Ebpf to achive fast high proformence even not involved linux kernel network stack(not skb struct) and observability  






