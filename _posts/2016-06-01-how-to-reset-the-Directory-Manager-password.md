---
layout: post
title: "How to reset the cn=Directory Manager's password"
categories: [ldap]
tags: [opendj, opends]
description: 在OpenDJ、OpenDS或OUD中有时需要修改管理员密码，或者忘记需要重置。
---

建议步骤如下，供参考。

### 停止OpenDJ服务



```vim
$ ./bin/stop-ds
```



### 生成新的加密密钥


```vim
$ ./bin/encode-password -s SSHA512 -c NewPassword
```

其中 **NewPassword** 是新密码明文，输出类似如下：

Encoded Password:  "{SSHA512}0zpEjr7RaC8wrAneQRGAVyLNUJb8QeLvKNpiUi1USga4MS8fLF1enyy+8SLFdGLGLBQZd8LF6YehViJeuwE04Ig1mXyJPSdi"


### 替换密文

修改`config/config.ldif` 文件，找到如下节点：

```vim
dn: cn=Directory Manager,cn=Root DNs,cn=config
objectClass: person
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: ds-cfg-root-dn-user
objectClass: top
userpassword: {SSHA512}HJ89gggPnKDL7AyNmQdPx6xo8FulMIUPK916VUMQ9rHZ1zvEDgw/PaXddMMTQY13uDYG4B5CRMcpCHzow9qT3zQitD2Mr5gn

```

用刚生成的密文替掉 userpassword的值。


### 重启OpenDJ服务


```vim
$ ./bin/start-ds &
```

至此，修改完毕。

因为应用访问是长连接，可能需要重启生效。
