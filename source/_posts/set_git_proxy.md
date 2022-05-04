---
title: 如何给git设置代理
data: 2022-05-04
categories:
- ubuntu20.04配置
tags: [git,代理]
---
国内在使用使用git获取github上代码时往往速度很慢，甚至总会连接失败，此时就需要给git设置代理。
<!--more-->
**如何给git设置代理**

	git config --global http.proxy http://proxyuser:proxypwd@proxy.server.com:porynumber
	
- proxyuser和proxypwd为登入代理服务器的帐号密码
-  proxy.server.com为代理服务器URL地址
- portnumber为代理服务器所在端口号


**如何取消git代理设置**

	git config --global --unset http.proxy
	
**如何检查现git现在的代理设置**

	git config --global --get http.proxy