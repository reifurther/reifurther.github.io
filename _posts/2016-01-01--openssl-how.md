---
layout: post
title: "通过openssl理解网络信息安全"
categories: [openSSL, RSA, CA, PKI]
tags: [openSSL]
description: 透过openssl学习加密算法、签名、证书等基本知识.
---



## 密码学背景

密码学要解决：

* 机密性问题：主要是分为对称加密算法和公开密钥算法
* 完整性问题：主要是采用散列函数和数据签名相结合的算法
* 鉴别和防抵赖问题：依靠严格执行的密码协议或网络协议

所以现代密码学主要分为两大部分： 密码算法和密码协议

> **Tips:**
> 算法无法建立将特定密钥对跟具体的个人身份联系起来的可信任关系
> 

## 对称加密算法

又称为传统密码算法、单密钥算法 或 秘密密钥算法。

对称加密是最快速，最便捷的一种加密方式，它有很多种算法，由于效率很高，所以被广泛的应用在很多加密协议的核心当中。绝大多数情况下，加密密钥和解密密钥是相同的。但是也可以不相同，但是解密密钥能够从加密密钥中推算出来。



根据加密方式不同，可以分为：

* 流加密算法（序列加密算法）：每次只对明文中的单个bit或单个Byte进行加密，优点是能够实时进行数据传输和解密，缺点是抗攻击能力比较弱。
* 块加密算法（分组加密算法）：每次对明文中的一组数据位（典型长度64位）进行加密，优点反之。

> **Tips:**
> 
> OpenSSL实现了8种对称加密算法，其中包括1种流加密算法RC4，7种块加密算法：AES,DES,Blowfish,CAST,IDEA,RC2,RC5
>

### 不进行加密操作

OpenSSL是将所有的对称加密算法指令集成到一个指令程序中，就是enc。

enc可以支持不对文件进行任何加解密操作，它支持复制或编解码操作。

简单复制功能，如：

```vim
OpenSSL> enc -none -in money.txt -out money_enc.txt  
```

BASE64编解码功能，如：

```vim
OpenSSL> base64 -in money.txt -out money_base64_encoding.txt
OpenSSL> base64 -in money_base64_encoding.txt -out money_base64_decoding.txt -d
```

### 加解密文件

使用3DES的CBC模式进行加解密操作，如：

```vim
OpenSSL> des-ede3-cbc -in money.txt -out money_3des.txt -k 12345678
OpenSSL> des-ede3-cbc -in money_3des.txt -out money_3des_decpt.txt -k 12345678 -d
```
如果需要对加密后的密文再进行BASE64编码，可采用如下指令进行加解密：

```vim
OpenSSL> des-ede3-cbc -in money.txt -out money_3des_base64.txt -k 12345678 -e -a 
OpenSSL> des-ede3-cbc -in money_3des_b64.txt -out money_3des_b64_decpt.txt -k 12345678 -d -a 
```

除了用 -k 直接显示输入口令外，还有一些方法，如：

```vim
OpenSSL> des-cbc -in money.txt -out money_des.txt
OpenSSL> des-cbc -in money.txt -out money_des.txt -pass stdin
OpenSSL> des-cbc -in money.txt -out money_des.txt -pass pass:12345678

#使用环境变量
$export mypass=12345678
OpenSSL> des-cbc -in money.txt -out money_des.txt -pass env:mypass

#使用文件
$echo 12345678 > passfile.txt
OpenSSL> des-cbc -in money.txt -out money_des.txt -pass file:passfile.txt
```

对于块加密算法的某些模式，还需要初始向量，这个时候还可以通过参数选择是否使用盐值，如：

```vim
OpenSSL> des-cbc -in money.txt -out money_des.txt -k 12345678 -iv CEFASDF
OpenSSL> des-cbc -in money.txt -out money_des.txt -k 12345678 -nosalt
OpenSSL> des-cbc -in money.txt -out money_des.txt -k 12345678 -S ABDF23A
```

综上：
对称加密的最大缺点是密钥的管理与分配，换句话说，如何安全的把密钥发送到需要解密你的消息的人手里是一个问题。

如果在线上交易中使用，还存在一些问题，如：

* 通讯双方在首次通信时协商一个共同的密钥，这个通道必须是安全可靠的
* 密钥的数目太大，不同的通讯者之间都不一样，这很难适应开放式大量的信息交流
* 对称加密算法本身不能验证发送者和接受者的身份

1976年，美国学者Dime和Henman为解决信息公开传送和密钥管理问题，提出了一种新的密钥交换协议，即允许在不安全的媒体上的通讯双方交换信息，安全的达成一致的密钥。就是『公开密钥系统』。相对于「对称加密算法」也叫做「非对称加密算法」，这也意味着现代密码学的重大突破。

## 非对称加密算法

又称为公开密钥算法。其基本特点如下：

* 加密密钥与解密密钥不相同
* 密钥对中的一个密钥可以公开 （称为公开密钥）
* 根据公开密钥很难推算出私人密钥

因为其算法较对称加密算法要慢很多，所以一般不直接用于大量数据的加密，主要有两种典型的应用：

* 密钥交换：使用公钥进行加密，而使用私钥进行解密。
* 数字签名：使用私钥进行加密，而使用公钥进行解密。

> **Tips:**
> 
> OpenSSL实现了4种非对称加密算法，包括：DH, RSA, DSA, EC。
> 
> 其中RSA既可用于密钥交换，也可用于数字签名。DH只能用于密钥交换，DSA专用于数字签名。
> 


### 生成、管理、使用RSA密钥

利用genrsa指令可以生成并输出一个RSA私钥，如下：

```vim
# 生成一个1024位的RSA私钥，对输出密钥不加密
$openssl genrsa -out rsa_privatekey.pem 1024

# 生成一个1024位的RSA密钥，并采用DES3算法进行加密
$openssl genrsa -out rsa_privatekey2.pem -passout pass:12345678 1024
```

利用rsa指令可以对密钥进行一些处理，如重新设置加密口令或加密算法、从私钥中输出公钥参数、进行格式转换等。

```vim
# 将采用DES3算法进行加密的RSA私钥 转换成使用256位AES算法加密
OpenSSL> rsa -in rsa_privatekey2.pem -passin pass:12345678 -out rsa_pk_aes.pem -passout pass:12345678 -aes256
```

利用rsautl可以用来进行数据的加解密：

```vim
# 首先根据私钥生成对应的公钥
OpenSSL> rsa -in rsaprivatekey2.pem -pubout -out rsapublickey2.pem

# 利用公钥对数据进行加密
OpenSSL> rsautl -in money.txt -out money_encrypt.txt -inkey rsapublickey2.pem -pubin -encrypt

# 利用私钥对数据进行解密
OpenSSL> rsautl -in  money_encrypt.txt -out money_decrypt.txt -inkey rsaprivatekey2.pem -decrypt
```

DH,DSA的操作与上类似。

另外，如上可见不管是哪种加密算法，都是可逆的。



## 信息摘要和数字签名

信息摘要算法一般用于保证数据的完整性， 它几乎是不可逆的。它是将数据量较小的数据（固定长度的摘要信息）与原数据量较大的文件建立一种特定的一一对应关系。

对于一个大文件来说使用非对称算法加解密太慢，所以通常是将信息摘要算法和非对称算法结合使用。

> **Tips:**
> 
> OpenSSL实现了5种信息摘要算法，包括：MD2, MD5, MDC2, SHA, DSS.
> 

信息摘要操作可以直接使用算法名指令md5之类，也可用统一指令dgst，如：

```vim
# 使用md5对文件进行信息摘要操作
$openssl dgst -md5 money.txt

# 可规整信息
$openssl dgst -md5 -c -hex money.txt

# 可指定随机种子文件
$openssl dgst -md5 -rand file1:file2 money.txt

```


### 执行数字签名

一个实用的数字签名操作流程如下：

* 产生一个RSA(或DSA)密钥对
* 对要签名的原始文件File做信息摘要操作，得到摘要信息Mf。
* 使用RSA私钥对Mf进行加密得到Sf。
* Sf就是原始文件的签名信息，可跟文件一起保存，或发送给接收人。


```vim
# 1. 首先生成一个RSA密钥，加密保存
OpenSSL> genrsa -out money_rsa_privkey.pem -passout pass:12345678 -des3 1024
# 2. 根据RSA私钥导出一个相应的RSA公钥
OpenSSL> rsa -in money_rsa_privkey.pem -passin pass:12345678 -out money_rsa_pubkey.pem -pubout
# 3. 使用sha1算法对原文件做信息摘要操作
OpenSSL> dgst -sha1 money.txt -out money_sgn.txt
# 4. 使用rsa私钥对摘要信息做加密
OpenSSL> rsautl -in money_sgn.txt -out money_encrypt.txt -inkey money_rsa_privkey.pem -pubin -encrypt

# 上面的3，4步其实可以简单通过dgst可以合为1步完成
OpenSSL> dgst -sha1 -sign money_rsa_privkey.pem -out money_sgn.txt money.txt
```

将 money.txt, money_sgn.txt, money_rsa_pubkey.pem 一起发送给接收方，完成整个签名流程。


### 验证数字签名

对上述数字签名的验证过程如下：

* 验证者收到File和Sf后，首先对文件File采用相同的信息摘要算法，得到摘要信息Mfn。
* 使用公钥对Sf进行解密得到Mfo。
* 比较Mfn 和 Mfo，如果相同，则验证成功，证明文件file没有更改，并且数字签名Sf有效。

验证RSA数字签名很简单，一个指令即可：

```vim
# 签名算法必须指定
OpenSSL> dgst -sha1 -verify money_rsa_pubkey.pem -signature money_sgn.txt money.txt
```


但是：
签名、验签虽然近乎完美，但仍不能证明这个私钥持有者的真实身份。

## 证书和CA

数字证书： 建立实体跟密钥对之间的联系

CA: 所用用户都信任，确认特定实体与密钥对一致。

证书按照其在证书链的位置分类，可分为：**终端用户证书** 和 **CA证书**。

* 终端用户证书：在证书链中处于最末端，只能用于具体应用程序或协议中
* CA证书：可以签发别的用户证书，即可有下级证书

对于一般用户来说申请的都是终端用户证书，也可以申请CA证书。对于OpenSSL来说是在配置文件openssl.cnf中通过[v3_req]节的***basicConstraints***字段指定的，keystone默认关。

### 申请证书

申请证书包含很多步骤，如生成密钥对，填写用户信息、签名等。

* 用户先生成私钥
* 根据私钥生成证书请求文件

下面命令可同时生成1024位的私钥和证书请求文件：

```vim
OpenSSL> req -new -newkey rsa:1024 -keyout privkey.pem -passout pass:12345678 -out req.pem
```

上面的命令虽然简单，但保护RSA私钥是采用默认DES3-CBC方式（可直接查看privkey.pem文件头），如果想采用其它的算法就必须分开了，如下：

```vim
# 先产生私钥，采用256位的AES算法
OpenSSL> genrsa -aes256 -passout pass:12345678 -out rsakey.pem 1024
# 根据私钥生成证书请求文件
OpenSSL> req -new -key rsakey.pem -passin pass:12345678 -out req.pem
```


当然，也可以对已经签发的证书请求文件进行验证：

```vim
OpenSSL> req -in req.pem -verify -noout
```


### 颁发证书

签发证书的过程，应该是利用CA服务器中CA的私钥进行加密的过程。

具体操作如下，其中req.pem是用户的证书审请文件：


```vim
OpenSSL> ca -in req.pem -out mycert.cer -notext
```

> **Tips:**
> 
> 需要注意的是，用户生成申请证书请求文件时输入的commonName应与CA本身产生私钥的commonName不能相同，否则会出现：
> ```TXT_DB error number 2``` 错误。
> 

有一些基本命令可以查看已颁发证书的内容：

```vim
# 将证书内容直接输出到屏幕
OpenSSL> x509 -in mycert.cer -noout -text
# 显示证书的序列号
OpenSSL> x509 -in mycert.cer -noout -serial
# 显示证书HASH值
OpenSSL> x509 -in mycert.cer -noout -hash

```


### 证书吊销

证书吊销共分为4步：

```vim
# 执行吊销证书指令，如指明吊销原因是因为证书私钥泄漏
OpenSSL> ca -revoke mycert.cer -crl_reason keyCompromise
# 吊销完了还要生成一个CRL，可以设定CRL的更新时间为7天7小时
OpenSSL> ca -gencrl -crldays 7 -crlhours 7 -out crl.crl
# 查询序列号为1的证书状态
OpenSSL> ca -status 1
# 更新文本证书数据库
OpenSSL> ca -updatedb
```


## 建立CA服务器

CA服务器本质上是一个应用程序，技术上实现了符合PKI和X.509等标准的证书签发和管理功能。

CA服务器的基本功能包括：接受申请证书请求、审核证书请求、签发证书、发布证书、吊销证书、生成和发布证书吊销列表（CRL）、证书库管理

CA的目录结构说明：

* newcerts目录: 存放新证书
* certs目录：存放签发者证书
* private目录：存放私钥文件
* crl目录：存放CA的CRL
* cacert.pem：CA证书
* cakey.pem：CA私钥
* index.txt：文本证书库文件
* serial：序列号文件 

这些目录文件也可以通过openssl.cnf修改。

### 手动创建

CA的目录结构可以手工创建，步骤如下：

1. 创建名为「demoCA」的目录。
2. 在该目录下创建 newcerts, private, crl 和 certs 子目录。
3. 在该目录下创建 一个空的index.txt文本文件。
4. 在该目录下创建 一个空的serial文件，文件中填01
5. 生成一个自签的根证书cacert.pem 放到该目录下
6. 生成一个私钥cakey.pem 放到 demoCA/private 目录下

其中5，6步用到的证书和私钥文件，可以通过如下命令创建：

```vim
OpenSSL> req -x509 -newkey rsa:2048 -keyout cakey.pem -out cacert.pem
```

这里是采用自签的根证书，也可以是向另一个CA申请的证书。


### 自动创建

openssl的贴心服务， 可自动创建一个CA目录结构，这里是利用openssl默认自带的ca.pl脚本，如下：

```vim
# 将脚本文件放到想创建的一个目录下
$cp /System/Library/OpenSSL/misc/CA.pl .

# 生成demoCA目录 
$perl CA.pl -newca
```

脚本中使用的是openssl默认配置文件：/System/Library/OpenSSL/openssl.cnf。

当然也可以手动指定配置文件，设置 **OPENSSL_CONF** 环境变量即可。

### PKI

PKI（公钥基础设施）：是一种基于非对称密钥算法的安全基础标准，提供一个框架，建立特定密钥对与具体的个人身份联系的可信任关系，而建立这种联系的主要形式就是颁发可信任的数字证书。

PKI包括一系列的组件：

* 验证机构(CA)：是PKI中的核心机构，负责确认身份和创建数字证书，建立一个身份和一对密钥之间的联系。
* 注册机构(RA)：可选组件，最主要的职责是接收申请人的申请请求，确认申请人的身份，然后将确认好的身份的申请请求递交给CA。
* 证书服务器：将用户的公钥和其它一些信息形成证书结构并用CA的私钥进行签名，从而生成正式的数字证书。
* 证书库：存储可以公开发布的证书的设施，通常是以目录的形式组成PKI的证书库
* 证书验证：通常是对证书链的验证

## SSL协议
SSL(Security Socket Layer)：是由NetScape提出的一种基于传输层安全协议。目前已发展到SSL v3，标准化版本为TLS。最初是为了Web的安全性。

SSL协议中的重要概念：

* 连接：是两台主机之间提供特定类型服务的传输，是点对点的关系，每个连接都与一个会话相关联。
* 会话：是客户端和服务器之间的关联，通过握手协议进行创建，多个连接可共享一个会话。

在openssl中提供了一些指令来模拟一个SSL客户端或是服务端

**连接一个HTTPS服务器**
		
```vim
OpenSSL> s_client -connect 113.107.107.80:443

# 如果要显示出其所使用的证书链 
OpenSSL> s_client -connect 113.107.107.80:443 -showcerts

```

**测试10秒内的连接数，这里指定使用RC4-MD5算法**

```vim
OpenSSL> s_time -connect 113.107.107.80:443 -www portal/login.jsp -time 10 -cipher RC4-MD5

```
 		
更详细的内容后面补充....

