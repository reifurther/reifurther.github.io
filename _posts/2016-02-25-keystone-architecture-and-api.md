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

>  **tips:**
> 
> 在API v2.0中，所有的操作其实在一个 'default' domain里。当创建一个user同时也创建了一个default domain。
> 

keystone的Libery版API [这里](http://172.17.254.218/openstack-docs/liberty/keystone/http-api.html) 也说明了 v2.0 和 v3 的一些关键不同点。 

### API v2.0

to be continue ....

### API v3

[API v3](http://specs.openstack.org/openstack/keystone-specs/api/v3/identity-api-v3.html) 官方手册详细说明了其定义、资源、核心方法等。

**一些重要概念：**

#### tokens

在前一篇文章中已细述。

#### credentials

用户凭据。证明用户的信息，API中可进行CRUD，有三种形式存在：

* user/password
* user/api key
* token

#### domains, projects, groups, users, roles

* domains 包含 users, groups, projects
* projects 包含一个或多个 users
* groups 是user的集合, 与 project 或 domains 是 多对多 的关系
* users 用户, 与 project 或 domains 是 多对多 的关系
* roles 角色, 描述 user-project 对应关系

#### Service catalog and endpoints

 这里的Services就是指服务目录，指具体的Openstack服务，比如nova服务、glance服务等等。
 
 它是一个Web服务，可以通过URL 或是 endpoint访问。
 
 endpoint可分为三类：
 
 * adminurl : 给 admin用户作管理用
 * internalurl : 给 openstack 内部组件间的服务
 * publicurl : 给其它用户可以访问的地址
 

各概念之间的关系如下：
![relationship](/assets/media/identity_relationship.jpg)

一些 v3 API 的例子，

[http://172.17.254.218/openstack-docs/liberty/keystone/api_curl_examples.html](http://172.17.254.218/openstack-docs/liberty/keystone/api_curl_examples.html) 

