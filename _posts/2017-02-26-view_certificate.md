---
layout: post
title: 查看证书
categories: HTTPS相关
---

查看证书的命令是

```shell
openssl x509 -in server.crt -text -noout
```

-in 指定要查看证书的名称

-text 以文本形式输出

-noout不输出证书的原始内容