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



## 证书和CA

### PKI与数字证书

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



## 客户端模拟


