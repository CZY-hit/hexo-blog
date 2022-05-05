---
title: 利用apache 2搭建一个简单的网站
categories:
- 网站搭建
tags: [apache2]
---
通过apache2部署静态网页非常简单，下面介绍一下ubuntu环境通过apache2配置简单静态网页的流程。
环境：Ubuntu 20.04 LTS, bash
<!-- more -->
**1、Apache 2的安装**

	sudo apt update
	sudo apt install apache2
**2、确认安装是否成功**

	apache2 --version
**3、验证Apache2 是否正确运行**

	sudo systemctl status apache2
![检查apache2运行状态](/img/apache2-check-status.png)

**4、Apache2各个组成部分**

Apache2的各个组成部分被安装在五个地方

-  配置文件 `/etc/apache2`
- lib文件 `/usr/lib`
- service启动文件 `/etc/init.d/apache2`
- 网页存放位置 `/var/www`
- 软件所在位置 `/usr/share/apache2`

**5、Apache的启动与关闭**

- 启动： `/etc/init.d/apache2 start ` 
	
或者 

	systemctl start apache2
	
- 关闭：`/etc/init.d/apache2 stop `

或者
	
	systemctl stop apache2

- 重启：`/etc/init.d/apache2 restart` 

 或者  
 
 	systemctl restart apache2
**6、网页部署**
使用Apache2部署网页非常简单，首先打开配置文件目录

	cd /etc/apache2
在此目录下，`apache2.conf`文件包含了`apache2`的主要参数，比如可配置的线程数量」进程数量、用户数量、服务器数量等。
**7、链接网页**
想要链接网页首先要在`/var/www/html`中创建自己的网页，默认情况下此时已经有一个默认的网页`index.html`，它会在你本地访问`http://127.0.0.1`时默认呈现
，想要把自己的网站挂载到服务器上，只需要用自己的`index.php`或者`index.html`替换原始`index.html`即可。

