---
layout: post
title: Apache2 配置https
categories: [https]
keywords: apache, https
---

第一步创建自签名证书

参考：这篇[文档](http://www.liuchungui.com/blog/2015/09/25/zi-jian-zheng-shu-pei-zhi-httpsfu-wu-qi/)的创建证书部分

第二步配置Apache服务器

参考：这篇[文档](http://www.jianshu.com/p/ae047991d40b)的配置部分

第三步是执行下面的命令，我就是因为没有执行这条命令导致ssl 协议错误，搞了半天才解决

```Shell
sudo a2ensite default-ssl
```

如果有问题可以参考这个[链接](http://stackoverflow.com/questions/119336/ssl-error-rx-record-too-long-and-apache-ssl)

免费证书申请可以参考这个[链接](https://www.freehao123.com/startssl-ssl-apache-ngnix/)