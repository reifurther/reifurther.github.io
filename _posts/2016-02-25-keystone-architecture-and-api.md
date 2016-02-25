---
layout: post
title: "keystone architecture and API"
categories: [openstack]
tags: [keystone]
description: 简要理解openstack认证组件keystone的一些概念模型，包括API介绍
---


## Openstack认证服务包含的组件

Openstack认证服务组件keystone提供如下核心功能：

* 用户身份认证
* 鉴权
* 目录服务


### Data Model



### HTTP API

keystone实现了2个API版本, [Identity API v2.0](http://specs.openstack.org/openstack/keystone-specs/#v2-0-api)  和 [Identity API v3](http://specs.openstack.org/openstack/keystone-specs/#v3-api)

为避免冲突，2个版本的模型、实现和文档都完全不同。按照社区的发展，v2.0版本将逐渐淡出。

### API v2.0 & API v3

概念上不同

* API v2.0 基于多租户授权(multi-tenant)模型，tenant租户作为最高级
* API v3 基于命名空间(namespace)模型，domains域作为最高级

接口上的不同

* API v2.0 只对 Tokens、Users、Tenants 操作。
* API v3 引入Domains、Projects、Groups、Policy等操作。

keystone的Libery版API [这里](http://172.17.254.218/openstack-docs/liberty/keystone/http-api.html) 也说明了 v2.0 和 v3 的一些关键不同点。 

本篇只重点介绍 API v3 版。

### API v3

[API v3](http://specs.openstack.org/openstack/keystone-specs/api/v3/identity-api-v3.html) 官方手册详细说明了其定义、资源、核心方法等。

一些重要概念：

* Users: /v3/users 
* Groups: /v3/groups
* Credentials: /v3/credentials
* Projects: /v3/projects
* Domains: /v3/domains
* Roles: /v3/roles/
* Regions: /v3/regions
* Services: /v3/services
* Endpoints: /v3/endpoints
* Tokens
* Policy

各概念之间的关系如下：
![relationship](/assets/media/identity_relationship.jpg)



```

