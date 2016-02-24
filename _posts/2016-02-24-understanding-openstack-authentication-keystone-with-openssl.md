---
layout: post
title: "understanding authentication keystone with openssl"
categories: [openstack]
tags: [openSSL,keystone]
description: 理解keystone中利用openssl实现的身份认证
---



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


