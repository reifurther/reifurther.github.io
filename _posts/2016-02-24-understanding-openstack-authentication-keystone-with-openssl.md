---
layout: post
title: "understanding keystone authentication with openssl"
categories: [openstack]
tags: [openSSL,keystone]
description: 理解keystone中的token,及如何利用openssl实现的身份认证
---


## Kenstone中的Tokens

token即我们通常意义上所说的令牌，相对于每次请求都需要验证用户名/密码 更方便和更安全。

它具有临时性和短周期性。在openstack中一般如下：

* 用户端发送用户名/密码给keystone
* keystone通过UUID或PKI生成加密后的token,返回给client端
* client端将缓存的token注入到Openstack API的请求中
* Openstack API endpoints获取并校验token，确定调用的合法性
* 若合法则处理请求，返回请求结果，若不合法则拒绝请求

产生token的方式有**UUID**和**PKI**两种，具体可参考该[blog](https://www.mirantis.com/blog/understanding-openstack-authentication-keystone-pki/)，还是很清晰。


### UUID tokens (Folsom and older)

具体可参考上面的blog，因为用于openstack较旧的版本，这里不重点分析了。

一般来说，UUID的方式是每次都要经过keystone才能认证，token中没有其它额外信息，网络开销小。

### PKI tokens (Grizzly and on)

Openstack G版及以上通常采用证书签名token的方式，称之为PKI tokens。

keystone这里作为了一个CA，用于生成用户证书，并利用用户的私钥和用户证书对token进行签名。

Openstack服务中的每个API endpoint都有一份keystone签发的证书、失效列表和根证书。API不用每次都去keystone认证token是否合法，只需根据keystone的证书和失效列表就可以确定。但更新失效列表不可避免。

PKI的方式，主要是利用标准算法来检查签名，不需要每次都经过keystone进行验证，所以keystone不会成为整个openstack的瓶颈. 但是PKI token包含了很多元数据信息，网络开销大。

PKI tokens产生并校验的具体的流程如下：

![PKI tokens](https://www.mirantis.com/wp-content/uploads/2013/07/PKI-token-validation-flow-1.png)

在上图中，每个API endpoint都会持有keystone的如下信息：

* 签发的用户证书
* 证书吊销列表（CRL）
* CA证书

API endpoint只需要根据证书验签就可以了，可以做到离线认证。


## Openstack的用户认证

其基本操作如下：

1. 每个用户的相关信息发送给keystone
2. keystone的CA的私钥通过cms进行加密生成token
3. 用户请求服务时同时将token发送给swift之类的组件
4. swift组件利用keystone签发的证书和CA的证书将token解密，得到具体信息


> **Tips:**
> 
> openssl中的cms指令 主要用来对S/MIME消息进行加解密、签名验签、压缩等。
> 
> keystone有自己的CA服务器，所以必须有CA的私钥和自签根证书
> 

## 使用的Openssl指令

之前的文章也提到了具体操作，这里是将keystone中的源码提出来，简化分析：

### 对于keystone的CA端

CA端很简单，生成私钥和自签证书即可，

* 生成私钥

```vim
$openssl genrsa -out ca_private_key.pem 1024  
```

* 对于CA生成自签的证书，也是根证书

```vim
$openssl req -new -x509 -days 3650 -key ca_private_key.pem -out cacert.crt  
```

上面也可以一条命令搞掂，可参考之前的文章『自建CA章节』。

* 根据用户reifu的申请文件，生成用户证书

```vim
OpenSSL> x509 -md5 -days 3650 -req -CA cacert.crt -CAkey ca_private_key.pem -CAcreateserial -CAserial ca.srl -in swift_reifu_req.csr -out swift_reifu.crt
```


### openstack其它组件端

这里以swift组件的用户**reifu**为例，

* 首先reifu用户生成自己的私钥

```vim
OpenSSL> genrsa -out swift_reifu_key.pem 1024  
```

* 利用私钥生成证书请求文件

```vim
OpenSSL> req -new -key swift_reifu_key.pem -out wift_reifu_req.csr  
```

将证书文件发给CA，生成用户证书。拿到CA颁发的用户证书swfit_reifu.crt之后，

* token加密

利用**用户证书和私钥**将相关信息加密，得到所谓token加密文件。

```vim
OpenSSL> cms -sign -signer swfit_reifu.crt -inkey swift_reifu_key.pem -outform PEM -nosmimecap -nodetach -nocert -noattr < reifu_info.txt > reifu_info_sec.txt
```

* token解密

利用用户证书和CA证书 解密token文件，得到用户信息

```vim
OpenSSL> cms -verify -certfile swfit_reifu.crt -CAfile cacert.crt -inform PEM -nosmimecap -nodetack -nocerts -noattr < reifu_info_sec.txt 
```


