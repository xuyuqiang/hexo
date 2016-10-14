---
title: tomcat实现SSL认证
date: 2016-10-14 11:08:40
tags:
    - tomcat
    - ssl
categories: ios
---
## openssl生成ssl证书
### 1.概述
> SSL 证书通过在客户端浏览器和 Web 服务器之间建立一条 SSL 安全通道(Secure socket layer(SSL),安全协议是由 Netscape Communication 公司设计开发。该安全协议主要用来提供对用户和服务器的认证;对传送的数据进行加密和隐藏;确保数据在传送中不被改变,即数据 的完整性,现已成为该领域中全球化的标准。由于 SSL 技术已建立到所有主要的浏览器和 WEB 服务器程序中,因此,仅需安装服务器证书就可以激活该功能了)。即通过它可以激活 SSL 协 议,实现数据信息在客户端和服务器之间的加密传输,可以防止数据信息的泄露。保证了双方传递信息的安全性,而且用户可以通过服务器证书验证他所访问的网站是否是真实可靠。
> SSL 网站不同于一般的 Web 站点,它使用的是“HTTPS”协议,而不是普通的“HTTP”协议。 因此它的 URL(统一资源定位器)格式为“https://www.etsec.com.cn”。

<!-- more -->
### 2.什么是x509证书链
> x509 证书一般会用到三类文件,key,csr,crt。 
> Key 是私用密钥,openssl 格式,通常是 rsa 算法。 
> csr是证书请求文件,用于申请证书。在制作csr文件的时候,必须使用自己的私钥来签署申请, 还可以设定一个密钥。 
> crt 是 CA 认证后的证书文件(windows 下面的 csr,其实是 crt),签署人用自己的 key 给你签署 的凭证。

### 3.key的生成
```bash
openssl genrsa -des3 -out server.key 2048 
```
> 这样是生成 rsa 私钥,des3 算法,openssl 格式,2048 位强度。server.key 是密钥文件名。为了生成这样的密钥,需要一个至少四位的密码。可以通过以下方法生成没有密码的 key: 
> openssl rsa -in server.key -out server.key 
> server.key 就是没有密码的版本了。

### 4.生成CA的crt
```bash
openssl req -new -x509 -key server.key -out ca.crt -days 3650
```
> 生成的 ca.crt 文件是用来签署下面的 server.csr 文件。 
> 需要依次输入国家,地区,组织,email。最重要的是,有一个 common name,可以写你的名字或 者域名(例如: www.etsec.com.cn)。如果为了 https 申请,这个必须和域名吻合,否则会引发 浏览器警报。

### 5.csr 的生成方法
```bash
openssl req -new -key server.key -out server.csr
```
> 需要依次输入国家,地区,组织,email。最重要的是,有一个 common name,可以写你的名字或 者域名(例如: www.etsec.com.cn)。如果为了 https 申请,这个必须和域名吻合,否则会引发 浏览器警报。生成的 csr 文件交给 CA 签名后形成服务端自己的证书。

### 6.crt生成方法
> CSR 文件必须有 CA 的签名才可形成证书.可将此文件发送到 CA 厂商 Entrust 等地方由它验证,要 支付一笔费用,测试证书可以自己做 CA 啦. 
```bash
openssl x509 -req -days 3650 -in server.csr -CA ca.crt -CAkey server.key -CAcreateserial -out server.crt
```
> 输入 key 的密钥后,完成证书生成。-CA 选项指明用于被签名的 csr 证书,-CAkey 选项指 明用于签名的密钥。-CAserial 指明序列号文件,而-CAcreateserial指明文件不存在时自动生成。最后生成了私用密钥:server.key 和自己认证的 SSL 证书:server.crt

### 7.生成.p12文件
```bash
openssl pkcs12 -export -clcerts -in server.crt -inkey server.key -out server.p12
```

### 8.tomcat实现SSL认证
> 按照以上方法证书生成后,我们再配置 tomcat。首先将以上创建的证书 p12文件拷贝到tomcat 主目录下的 conf 文件夹下。
> 1.双击 server.crt 证书导入信任证书库。 
> 2.配置 Tomcat 支持 HTTPS 双向认证(服务器将认证客户端证书): 修改 tomcat 的 conf 目录里的 server.xml 文件($TOMCAT_HOME/conf/server.xml),找到类似下 面内容的配置处,添加配置如下:
```bash
<Connector port="8443" maxThreads="150" minSpareThreads="25" maxSpareThreads="75" enableLookups="false" disableUploadTimeout="true" acceptCount="100" debug="0" scheme="https" secure="true" clientAuth="false" sslProtocol="TLS" keystoreFile="conf/tomcat.p12" keystorePass="111111" keystoreType="PKCS12"/>
```
> 其中,clientAuth 指定是否需要验证客户端证书,如果该设置为“false”,则为单向 SSL 验证,SSL 配置可到此结束。如果 clientAuth 设置为“true”,表示强制双向 SSL 验证,必须验证客户端证书, 但服务器不会颁发证书必须由客户端导入。如果 clientAuth 设置为“want”,则表示可以验证客户端 证书,但如果客户端没有有效证书,也不强制验证。 
注意:浏览器的高级选型中要启用 TLS 加密方式。

### 9.最后
> tomcat的ssl访问端口为8443。如果是手机游览器想访问，可将ca.crt证书放到tomcat外部访问目录中，供手机游览器访问ca.crt文件然后安装即可。 
> 这里配置这个tomcat ssl认证，主要是为了实现iOS中通过plist文件安装ipa程序，以供内部测试人员测试使用。

