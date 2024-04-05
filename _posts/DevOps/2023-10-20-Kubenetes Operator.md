---
layout: post
title: Kubenetes Operator
subtitle: 
gh-repo: honky-tonk/honky-tonk.github.io 
gh-badge: [star,follow]
tags: [DevOps]
comments: true
---
# What is Kubenetes Operator

If we want to deploy our code to Kubernetes cluster, in a traditional way we would write code, and package it into container images, then we push image to image repository, finally we use kubernetes resource type like ```Deployment```, like ```Service``` etc... to deploy our code onto kubernetes cluster.

However for some complex project code that use resource which kubernetes supply is not efficient and flexible, such as a job configuration section that has it own lifecycle, but only we can use is Kubernetes's build-in ```ConfigMaps``` resource to manage it, once the configuretion section finish their job we should exit graceful, but this seem hard to implement in ```ConfigMaps```

To address the issue, we can using kubernetes operator create a custom defined resource based on our project code, that is tailored to our project code's needs, and allows Kubernetes(sub-controller existed when you create operator, and sub-controller charge of the lifecycle of operator) to manage the lifecycle of the custom resource. This approach provides a more flexible and efficient way to deploy and manage our project code on Kubernetes.    

# Difference Between Kubenetes Operator and Helm

Helm like a package manager for kubernetes cluster, which are declear as yaml file, and helm allow user to template resource config, and  direct deploy to kubernetes cluster

operator is more flexible, you could create self define resource base kubernetes exention api and manage lifecycle of your application

# How to Create Kubenetes Operator
In the article, we use **KubeBuilder**

the KubeBuilder is a set of tooling about how to structre custom controllers and operators

>the custom controller charge of the operator we create and manage it lifecycle

before we get start with kubenetes operator, we should dive into apiVersion, Kind etc. if you deploy application via yaml file you must familiar with ```kind```, ```apiVersion``` keyword, but what actually hack is that? in generally you could treat ```apiVersion``` as **group** of api with version, so what is api in a api group? it a kind(one api is mapped to one ```kind```) 

# TODO



