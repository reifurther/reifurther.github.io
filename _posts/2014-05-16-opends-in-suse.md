---
layout: post
title: "suse下安装openDS遇到的问题"
categories: [suse, openDS]
tags: [suse, openDS]
description: suse下安装openDS2.2遇到如下问题，供参考.
---

suse下安装openDS2.2遇到如下问题，供参考。

## 安装完后无法启动服务器

提示：
`The administration connector self-signed certificate cannot be generated
`

解决方法，在/etc/hosts里添加一行：
{% highlight ruby %}
127.0.1.1       linux-5tzq
{% endhighlight %}
其中linux-5tzq 是安装机的主机名。

## 双向复制数据不能同步问题

在两台主机间作双向复制:  172.18.64.42:5444 **\<-\>** 172.18.64.39:5444  , 
此时并未配置schema，建立不成功，
表现：

 - 99user.ldif未完全复制过去，00-core.ldif未修改
 - 复制链路未建立

> `报错信息如下: 
> `将服务器 172.18.64.42:5444 上的注册信息初始化为服务器 172.18.64.39:5444 的内容 ..... 在使用服务器 172.18.64.39:5444 中的内容进行初始化期间出现错误。最新日志详细信息: [30/七月/2012:23:37:17 +0800]severity="NOTICE" msgCount=0 msgID=9896349 message="通过副本初始化 任务quicksetup-initialize1 已开始执行"。任务状态: STOPPED\_BY\_ERROR。有关详细信息，请检查172.18.64.39:5444 的错误日志。请参见 /tmp/opends-replication-6498757972750616075.log 以了解有关此操作的详细日志。

### 初始解决方法
将42机器上的schema以及config.xml都修改一致，并发数据也导入。
但还是报错。

> `报错信息如下: 
> `在使用服务器 linux-5tzq:5444 中的内容进行初始化期间出现错误。最新日志详细信息: [31/七月/2012:02:26:33 +0800] severity="NOTICE" msgCount=0 msgID=9896349 message="通过副本初始化 任务 quicksetup-initialize2 已开始执行"。任务状态: STOPPED\_BY\_ERROR。有关详细信息，请检查linux-5tzq:5444 的错误日志。

### 最后解决
执行：
{% highlight ruby %}
$getent hosts linux-5tzq
{% endhighlight %}

`发现 两台主机ip均一致，为127.0.1.1 ，显然错误。 
`

把172.18.64.42的主机ip地址更改为 127.0.1.2, 配置双向复制成功。