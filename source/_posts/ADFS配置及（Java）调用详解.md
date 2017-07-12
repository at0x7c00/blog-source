---
title: ADFS配置及（Java）调用详解
date: 2017-06-22
categories: 运维
tags: 
- ADFS
- java-saml

---

# 配置ADFS
基本可以按照[http://www.web520.cn/archives/41572](http://www.web520.cn/archives/41572)来配置ADFS，但是其中的web服务器证书模板添加ADFS主机名称这一步我就没有操作成功，但也没管，后来还是通了。

# Java-Saml
这是一个开源的SAML客户端，还有[其他语言版本](https://github.com/onelogin)的，很强大，并且更新很频繁。
[https://github.com/onelogin/java-saml](https://github.com/onelogin/java-saml)

# 重要说明
* 所有证书里的通信证书才是最关键的，客户端都是通过这个证书来通信的。
![](/images/note-images/certs.png)
* 信赖方配置时各个URL都应该配什么，参考这里[https://www.vidbeo.com/support/users/how-do-i-configure-single-sign-on-using-adfs](https://www.vidbeo.com/support/users/how-do-i-configure-single-sign-on-using-adfs)
*  java-saml需要pkcs#8格式的密钥，可以通过如下方式来转换得到：
> How to convert a .PFX format private key to PKCS#8
> 1. Make an back up copy from IIS
> 2. Convert .PFX to .PEM
> bin/openssl pkcs12 -in YourCertName.pfx -nocerts -nodes -out NewName.pem
> (When prompted, enter the .PFX Private Key Password)
> 3. Convert PEM to PKCS8
> bin/openSSL pkcs8 -in NewName.pem -topk8 -nocrypt -out YourCertName.pk8

参考:[https://knowledge.symantec.com/kb/index?page=content&id=SO12643&pmv=print&actp=PRINT&viewlocale=en_US](https://knowledge.symantec.com/kb/index?page=content&id=SO12643&pmv=print&actp=PRINT&viewlocale=en_US)

* 信赖方的加密属性：可以不选，如果选一定要选之前提到的“通信证书”
![](/images/note-images/relay.png)
* Windows Server 2016服务器在Logout时有bug，无法正常跳转。
[https://social.technet.microsoft.com/Forums/windows/en-US/acbf767a-2758-49bc-b3ab-45a8420af780/logout-when-adfs-is-bridge-between-wsfederation-rp-and-saml-claims-provider-external-saml-idp?forum=ADFS](https://social.technet.microsoft.com/Forums/windows/en-US/acbf767a-2758-49bc-b3ab-45a8420af780/logout-when-adfs-is-bridge-between-wsfederation-rp-and-saml-claims-provider-external-saml-idp?forum=ADFS)





