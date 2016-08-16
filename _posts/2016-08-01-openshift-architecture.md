---
layout: post
title: "OpenShfit Architecture"
categories: [paas]
tags: [docker,kubernetes]
description: 理解Openshift产品的框架、设计、组件等
---


## OpenShfit Architecture


### 核心组件

目前使用的是 Openshift Enterprise 3.2， 使用的主要组件如下：


| 名称			  | 使用版本   | 目前最新稳定版本   | 开发语言  | 备注 |
|:-------------|:---------:|:---------------:|:--------:|:----|
| Docker       | 1.10.3    | 1.12.0          | Go       ||
| Kubernetes   | 1.2       | 1.3.4           | Go       ||
| etcd         | 2.2.5     | 3.0.4           | Go       ||
| haproxy      | 1.5.14    | 1.6.0           | C        | router|
| dnsmasq      | 2.66      | 2.76            | C        | 内部dns服务|
| webCosole    |           | 1.6.0           | Nodejs   | Openshift管理控制台|
| elasticsearch| 1.5.2     | 2.3.5           | Java     | EFK，应用和中间件的日志搜索引擎|
| fluentd      | 0.12.20   | 0.14.2          | Ruby     | EFK，应用和中间件的数据收集|
| kibana       | 4.1.2     | 4.5.4           | JS       | EFK，应用和中间件的数据展示|
| heapster     | 1.1.0     | 1.1.0           | Go       | k8s的子项目，对容器集群作监控与性能分析|
| cadvisor     | 0.4.1     | 0.23.8          | Go       | 对容器运行的性能指标与使用情况进行收集、聚合、处理和输出|
| hawkular     | 0.8.2     | 0.9.0           | Java     | 监控工具，based on eap6.4.4 |
| cassandra    | 2.2.1     | 3.7             | Java     | 监控的后端存储|


批量安装是使用脚本 ansible 安装。


### To Do ...