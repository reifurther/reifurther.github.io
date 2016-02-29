---
layout: post
title: "keystone architecture and API"
categories: [openstack]
tags: [keystone]
description: 简要理解openstack认证组件keystone的一些概念模型，包括API介绍
---


## Openstack认证服务包含的组件

Openstack认证服务组件keystone提供如下核心功能：

* 用户身份认证、鉴权
* 目录服务 （对某个服务是否有访问权限）


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
* API v3 引入更多操作，如Domains、Projects、Groups、Policy等。
* v3的验证/auth/tokens,相比v2.0的/tokens，token的ID不再在body中包含，而是在返回header中的X-Subject-Token

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

* domains 是 users, groups, projects 之上的集合抽象
* projects 包含一个或多个 users
* groups 是user的集合, 与 project 或 domains 是 多对多 的关系
* users 用户, 与 project 或 domains 是 多对多 的关系
* roles 角色, 描述 user-project 对应关系

#### Service catalog and endpoints

 openstack身份管理是通过「服务」的形式对外服务，
 
 这里的Services就是指服务目录，指具体的Openstack服务，比如nova服务、glance服务等等。
 
 它是一个Web服务，具体体现是通过endpoint访问。
 
 endpoint可分为三类：
 
 * adminurl : 给 admin用户作管理用
 * internalurl : 给 openstack 内部组件间的服务
 * publicurl : 给其它用户可以访问的地址
 

各概念之间的关系如下：
![relationship](/assets/media/identity_relationship.jpg)

一些 v3 API 的例子，

[http://172.17.254.218/openstack-docs/liberty/keystone/api_curl_examples.html](http://172.17.254.218/openstack-docs/liberty/keystone/api_curl_examples.html) 

#### 创建的基本流程

1. **创建service服务目录**
	* 指定类型，默认type就是identity

2. **创建endpoint**
	* 必须指明使用哪个Service
	* 必须指明使用region
	* 一般建3套API：public,internal(默认端口5000)；admin（默认端口35357）
	
3. **创建project**
	* 类似tanant概念
	* 必须指明属哪个domain

4. **创建user**
	* 必须指明属哪个domain
	
5. **创建role**
	* 角色的实际定义是在Policy.json文件中
	
6. **将role赋给project 和 user**

> **Tips**
> 
> 在创建endpoint时，需要指明region，这里region是指：
> 
> 比如A、B中心都建有openstack集群，独立提供nova、swift等服务，但希望用户权限管理集中，这时就需要通过region进行区分。
> 