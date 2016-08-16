---
layout: post
title: "how to configure haproxy"
categories: [paas]
tags: [haproxy]
description: 介绍什么是haproxy,如何使用及配置示例
---


## Haproxy介绍

文档中心： [http://cbonte.github.io/haproxy-dconv/](http://cbonte.github.io/haproxy-dconv/) 



## 安装


```vim
$yum install haproxy -y 
```

配置文件： /etc/haproxy/haproxy.cfg


## 配置文件说明

 haproxy配置中分成五大部分：
 
   ***global***: 全局配置参数，参数是进程级别的，通常这些参数是与操作系统有关。
        
   ***defaults***: 配置默认参数，这里的默认参数是相对于后面的frontend 、backend、listen组件而言。
        
   ***frontend***: 接收客户端发起请求，建立socket监听。可根据规则直接指定具体的后端服务器。
   
   ***backend***: 后端服务器集群配置，一个backend对应一个或者多个后端服务器。
   
   ***listen***: 用于建立虚拟主机，是frontend和backend的结合体，可直接取代frontend和backend
   
### global


```vim
global        # 对全局参数的说明

            log         127.0.0.1 local2    # 全局的日志配置，使用log关键字，此日志需要借助rsyslog来进行配置，默认等级为info

            chroot      /var/lib/haproxy       # 改变当前工作目录，基于安全性的考虑
            pidfile     /var/run/haproxy.pid  # 当前进程pid文件
            maxconn     4000                # 当前进程最大的连接数，很重要的一个参数，后面有详细的讲解。
            user        haproxy             # 启动服务所属用户
            group       haproxy             # 启动服务所属组
            daemon                            # 开启守护进程运行模式

            # turn on stats unix socket
            stats socket /var/lib/haproxy/stats        # haproxy socket文件

```

> **Tips:**
> 
> **maxconn**: 启动服务后，haproxy进程的最大连接数，也就是该服务器haproxy支持的最大并发数。
> 
> **nbproc**: 开启进程数，默认配置是开启1个进程，建议开启一个进程。如开启2个进程定义： nbproc 2
> 
> 

### defaults


```vim
defaults    # 开始定义 defaults段
            mode                    http          # haproxy支持两种工作模式tcp、http、health，默认是http
            log                     global        # 应用全局配置的日志
            option                  httplog        # 启用日志记录HTTP请求
            option                  dontlognull # 启用该项，日志中将不会记录关于对后端服务器的检测情况。
            option 					   http-server-close            # 每次请求完毕后主动关闭http通道
            option 					   forwardfor except 127.0.0.0/8    # 这里是转发客户端ip地址至后端服务器上。根据测试，无需修改此项，在后端服务器上设置{X-Forwarded-For}i就能实现。
            option                  redispatch    # 此项是对backend server内容中的serverID作用的
            retries                 3             # 定义尝试连接后端服务器的失败次数，尝试3次失败后，后端该服务器将标记为不可用状态
            timeout http-request    10s            # http请求超时时间
            timeout queue           1m            # 一个请求在队列里的超时时间
            timeout connect         10s            # 连接超时时间
            timeout client          1m            # 客户端超时时间
            timeout server          1m            # 服务端超时时间
            timeout http-keep-alive 10s            # http keepalived保持时间
            timeout check           10s            # 检测超时时间
            maxconn                 3000        # 每个frontend定义的最大连接数

```


> **Tips:**
> 
> mode: 工作模式, 后面会有详细介绍
> 
> timeout http-keep-alive: http keepalived 超时时间，具体需要测试，一般保持默认
> 
> maxconn：defaults中出现的maxconn值是为frontend做限制的。后面有详细的讲解
> 
> redispatch: 是对backend server内容中的serverID作用的，当backend中定义的server启用了cookie时，haproxy会将请求到的后端服务器的serverID插入到cookie中，以保证session持久性，如果此时后端服务器宕机，但是客户端的cookie是不会刷新的，如果开启了cookie，将会使客户端请求强制定向到另一台后端server上，保证了服务的正常运行。
> 
> 


### listen


```vim
listen stats        # 申明定义一个listen段，命名为stats
            bind *:1080        # bind设置socket 格式：ip:port
            mode http         # 指定工作模式
            stats refresh 30     # 刷新时间，这里30秒刷新一次
            stats enable        # 启用haproxy监控状态
            stats hide-version    # 隐藏haproxy的版本号
            stats uri /haproxy?wsd    # 设置haproxy监控页的uri，可任意设置
            stats auth admin:admin    # 验证用户名密码
            stats admin if TRUE        # 此项是实现haproxy监控页的管理功能的。

```


### frontend & backend


```vim

frontend www.test.com    # 定一个frontend 段，命名为www.test.com 命名随意。
            mode http             # 定义工作模式
            bind *:8000         # 定义监听地址和端口 格式：ip:port
            option forwardfor   # 将客户端ip转发到后端真实服务器
            use_backend web check    # 使用web后端服务器群组，并进行健康检查

backend web             # 定义一个backend 段，命名为web，该命名在frontend中 use_backend使用
            balance roundrobin    # 主机集群的调度算法，此参数很重要，下面详细解释。
            # server 定义中可以使用的参数：{weight | check | inter | rise | fall | maxconn }
            server web1 127.0.0.1:8080 check    # 具体定义的后端主机 格式：server 主机名 ip:port check
            server web2 127.0.0.1:80 check

```

> **Tips:**
> 
> server: 定义中可以使用的参数
> 
> weight -- 调整服务器权重
>  
> check -- 允许对该服务器进行健康检查
> 
> inter -- 设置连续两次健康检查之间的时间，单位为毫秒(ms)，默认是2000(ms) = 2秒
> 
> rise -- 指定多少次连续成功的健康检查后，即可认定该服务器处于可操作状态，默认为2次
> 
> fall -- 地址多少次失败的检查后，即认为服务宕机，并标识为不可用状态，默认为3次
> 
> maxconn -- 指定可被送到后端服务器的最大连接并发数
> 
> 

## 高阶配置

### 代理模式 mode

 设定实例的运行模式或协议，当实现内容交换时，前端和后端必须工作在同一种模式，否则无法开启实例。
 
 支持： { **http** | **tcp** | **health** }

 1. tcp：实例运行于纯tcp模式，在客户端和服务器端建立一个tcp通道，且不会对7层报文做任何的操作，通常用于ssh、smtp等
           
 2. http：实例运行于http模式，客户端在转发至后端服务器之前将被深度分析，所有与规则不符合的格式都将被拒绝。
 
 3. health：实例运行于health模式，其对入站请求仅响应ok信息并关闭连接，且不会记录任何日志，此模式将用于响应外部组件的健康状态检查请求，目前该模式已经被废弃。
 

### 负载均衡算法 balance

定义负载均衡算法，可作用于 defaults、listen、backend中，其作用是将一个连接通过对应的规则重新派发至后端服务器。支持的算法如下：

1.  **roundrobin**: 基于权重进行轮询，在服务器处理时间保持平均分配，这是最常见的负载均衡算法，但是在haproxy中此算法是动态的，这表示其权重可动态调整，每个后端服务器仅能最多并发4128个连接，切记！


2.  **static-rr**: 基于权重进行轮询，与roundrobin类似，但是static-rr为静态方法，在运行时，调整权重是不会生效的，不过后端服务器并发数是没有限制的，因此如果要使用haproxy做轮叫一定要修改 balance 为 static-rr 算法


3. **leastconn**: 新的连接请求被派发至具有最少连接数目的后端主机，在有较长时间会话的场景中推荐使用此算法，如LDAP、SQL等，其并不太适用于较短会话的应用层协议，如http，此算法是动态的，可以运行时调整权重；


4. **source**: 将请求的源地址进行hash运算，并由后端服务器的权重总和相除后派发至某匹配的服务器，这可以使得同一个客户端ip的请求始终被派发至特定主机，不过当某台服务器权重发生变化时，如某台后端服务器宕机或新添加了新的机器，许多客户端的请求可能会被派发至与此前请求不同的主机，因此这种算法常用与负载均衡无cookie功能的基于TCP的协议，其默认为静态，但是可以通过hash-type 来修改此特性


5.  **uri**: 对uri左半部分或整个URI进行hash运算，并由服务器的总权重相除后派发至某匹配的服务器，这可以使得对同一个uri的访问并定向至后端同一台主机，除非服务器的权重总数发生了变化，次算法常用与代理缓存或反病毒代理以提高缓存命中率，需要注意的是，此算法仅应用与HTTP后端服务器场景，其默认为静态算法，不过可以使用hash-type 来修改此特性。


6. **url_param**: 通过<argument>为URL指定的参数在每个HTTP GET请求中将会被检索；如果找到了指定的参数且其通过等于号“=”被赋予了一个值，通过<argument>为URL指定的参数在每个HTTP GET请求中将会被检索；如果找到了指定的参数且其通过等于号“=”被赋予了一个值，此算法可以通过追踪请求中的用户标识进而确保同一个用户ID的请求将被送往同一个特定的服务器，除非服务器的总权重发生了变化；如果某请求中没有出现指定的参数或其没有有效值，则使用轮叫算法对相应请求进行调度；此算法默认为静态的，不过其也可以使用hash-type修改此特性


7. **hdr**: 对于每个HTTP请求，通过<name>指定的HTTP首部将会被检索；如果响应的首部没有出现或者没有有效值，则使用轮叫算法对响应的请求进行调度；此算法为静态，可以通过hash-type修改。


8. **rdp-cookie**: 远程桌面协议，该算法几乎没有被使用到。



### 性能参数配置

### ACL





## 示例

SSO通过代理支持http与tcp进行授权与鉴权操作。

```vim

#---------------------------------------------------------------------
# Example configuration for a possible web application.  See the
# full configuration options online.
#
#   http://haproxy.1wt.eu/download/1.4/doc/configuration.txt
#
#---------------------------------------------------------------------

#---------------------------------------------------------------------
# Global settings
#---------------------------------------------------------------------
global
    # to have these messages end up in /var/log/haproxy.log you will
    # need to:
    #
    # 1) configure syslog to accept network log events.  This is done
    #    by adding the '-r' option to the SYSLOGD_OPTIONS in
    #    /etc/sysconfig/syslog
    #
    # 2) configure local2 events to go to the /var/log/haproxy.log
    #   file. A line like the following can be added to
    #   /etc/sysconfig/syslog
    #
    #    local2.*                       /var/log/haproxy.log
    #
    log         127.0.0.1 local2

    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon

    # turn on stats unix socket
    stats socket /var/lib/haproxy/stats

#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#---------------------------------------------------------------------
defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 3000

#---------------------------------------------------------------------
# main frontend which proxys to the backends
#---------------------------------------------------------------------

listen stats
    mode http
    bind 0.0.0.0:1080
    stats enable
    stats hide-version
    stats uri     /haproxyadmin?stats
    stats realm   Haproxy\ Statistics
    stats auth    admin:admin
    stats admin if TRUE
frontend http-in
    bind *:17300
    mode http
    log global
    option httpclose
    option logasap
    option dontlognull
    capture request  header Host len 20
    capture request  header Referer len 60
    default_backend servers
frontend healthcheck
    bind :1099
    mode http
    option httpclose
    option forwardfor
    default_backend servers
backend servers
    balance source
    server  sso1 172.17.252.64:17300 check inter 2000 fall 3 weight 30
    server  sso2 172.17.252.65:17300 check inter 2000 fall 3 weight 30

frontend tcp-in
    bind *:6666
    mode tcp
    default_backend tcp-back
backend tcp-back
    mode tcp
    balance leastconn
    server  ssomagpie1 172.17.252.64:6666
    server  ssomagpie2 172.17.254.65:6666


```


## 与LVS及Nginx对比


Ref : [http://www.cnblogs.com/hukey/p/5586765.html](http://www.cnblogs.com/hukey/p/5586765.html)