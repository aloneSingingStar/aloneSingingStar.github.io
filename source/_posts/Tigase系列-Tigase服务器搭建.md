---
title: Tigase系列--Tigase服务器搭建
date: 2017-09-26 09:12:02
tags:
- tigase
- xmpp
- tigase服务
description: tigase,tigase服务器,xmpp
categories: tigase
---
## 1.下载安装包

下载地址：https://projects.tigase.org/projects/tigase-server/files
安装包说明：不同后缀的安装包对应不同操作系统、或者不同的安装方式，比如tigase-server-7.1.1-b4457-javadoc.jar适用与GUI方式安装(一步步都有图像界面那种)、同样也可以用于ssh远程到服务器，控制台安装。tigase-server-7.1.1-b4457-dist-max.tar.gz适用与WEB页面方式安装等
安装文档说明：一切安装步骤以官方文档为准，可以将下载好的包解压（jar包可以将.jar改为.zip，解压即可看到文档），文档目录在docs目录，比如GUI方式安装的文档路径如下：docs/Administration_Guide/html/index.html#guiinstall
## 2.安装前准备
安装数据库：MySQL、PostgreSQL、SQL Server、Derby选一种你想用的
基本环境:jdk安装
## 3. 安装过程需要注意的问题
安装步骤：根据你选择的安装方式，查看对应的官方文档，我选择的是最简单的GUI方式，文档路径:docs/Administration_Guide/html/index.html#guiinstall
注意点1：Basic Server Configuration（基本服务配置）
![serverConfiguration](https://github.com/aloneSingingStar/aloneSingingStar.github.io/blob/master/img/serverConfiguration.gif)
Your XMPP(Jabber) domains:自定义xmpp域名，比如veloci.tigase.org
Server administrators:管理员账号（在数据库注册的第一个用户），格式为：名称@自定义的域名，比如admin@veloci.tigase.org
账户名称必须是：名称@自定义的域名,自定义的域名部门必须与Your XMPP(Jabber) domains中定义的一致，否则登录会报authentication相关的错

注意点2：Tigase数据库安装
![tigase数据库安装](https://github.com/aloneSingingStar/aloneSingingStar.github.io/blob/master/img/tigasedb.jpg)
成功后，会在本地mysql数据库生成tigasedb数据库，其中可以在tig_users表中看到之前注册的Server administrators:管理员账号(在这个数据库中存在的用户，可以用来登录)

## 4.启动Tigase服务，验证安装是否成功
启动MySQL服务，确保MySQL数据库可连接
进入安装Tigase后生成的目录，执行：./scripts/tigase.sh start etc/tigase.conf，启动服务
网页中访问：http://localhost:8080/ui,如果不能访问，可能是因为安装时，少装了HTTP相关组件，这样的话，就可以安装一个客户端，比如psi来登录验证，在页面中输入：Server administrators:管理员账号、密码，如果能登录进入，说明安装成功，如果不能，请检查安装配置

## 5.关于SSL握手证书及Android客户端信任库证书
安装目录下certs目录中存放着证书，当我们使用第一个用户（一般是管理员）登录时，用户发起SSL连接，服务器会首先检查你是否在certs目录放入【你自定义的域名.pem】格式的证书，如果没有，则生成一个[你自定义的域名.pem]格式的证书，然后发送给客户端，然后由客户端决定是否信任这个证书，比如用PSI客户端登录，会弹出框体，让你查看该证书的详细信息，并且让你决定是否信任证书。而在Android中，则需要我们先在本地生成一个信任库，然后将该服务器证书导入到信任库中，每次登录时，Android客户端先会在信任库中检查是否有该证书，有则信任，不信任，则拒绝连接，这个需要代码来实现。

自签名证书制作、客户端信任库制作：
官方文档：docs/Administration_Guide/html/index.html#ServerCertificates
```
1.Generate local private key.
openssl genrsa -out[自定义域名.key] 1024

2.Generate a certificate request:
openssl req -new -key 自定义域名.key -out 自定义域名.csr

这一步会让你填写相关信息，比如所在国家、城市、公司信息等，其中最重要的一点是Common Name,这里填上你自定义的域名，不然客户端登录时会报：Cerificate hostname doesn't match domain name you want to connect

3.Sign the Certificate Request（self-signing）
openssl x509 -req -days 365 -in 自定义域名.csr -signkey 自定义域名.key -out 自定义域名.crt

4.Generate PEM file
cat 自定义域名.crt 自定义域名.key > 自定义域名.pem
如果是第三方授权的证书，则需要导入证书链，格式如下，例如，如果您有来自XMPP联盟的证书，则需要下载StartCom根证书和中间ICA证书。
cat 自定义域名.crt 自定义域名.key sub.class1.xmpp.ca.crt ca.crt > 自定义域名.pem

5.Android客户端生成truststore，并导入受信任的服务器证书
1.要生成bks证书，需要bcprov-ext-jdk15on-151.jar。且将该文件放到/Library/Java/JavaVirtualMachines/jdk1.8.0_65.jdk/Contents/Home/lib目录下。

2.执行如下命令：
keytool -importcert -trustcacerts -keystore client_trust.bks -file 自定义域名.crt -storetype BKS -provider org.bouncycastle.jce.provider.BouncyCastleProvider

6.将生成的bks文件放入Android代码的res/raw文件夹下，然后写代码进行ssl握手

```