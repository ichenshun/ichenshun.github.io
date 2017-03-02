---
layout: post
title: Android使用NanoHttpd部署https服务器
categories: HTTPS相关
keywords: NanoHttpd, 手机, HttpServer
---

按照这个[链接](https://github.com/NanoHttpd/nanohttpd) 在代码中include NanoHttpd库

但需要自己定义Keystore的加载

```Java
KeyStore keystore = KeyStore.getInstance("BKS");
InputStream keystoreStream = mAssets.open("keystore.bks");
if (keystoreStream == null) {
    throw new IOException("Unable to load keystore from classpath: " + "keystore.bks");
}

char[] passphrase = "password".toCharArray();
keystore.load(keystoreStream, passphrase);
KeyManagerFactory keyManagerFactory = KeyManagerFactory.getInstance(KeyManagerFactory.getDefaultAlgorithm());
keyManagerFactory.init(keystore, passphrase);
```

按照文档中的方法，创建的keystore格式为jks，需要用[keystore explorer](http://keystore-explorer.org/)将格式转换为BKS。

如果想使用openssl创建的私钥和证书创建一个KeyStore，可以使用如下操作：

1. 打开KeyStore Explorer，创建一个KeyStore，指定类型为BKS
2. 导入Key Pair，指定openssl类型，指定私钥和证书文件
3. 输入私钥存储密码和KeyStore文件存储密码，导入完成
4. 保存KeyStore文件

在代码中，如下语句的第二个参数，指定了keystore文件的存储密码。

```java
keystore.load(keystoreStream, passphase);
```

如下语句的第二个参数，指定了私密的存储密码

```java
KeyManagerFactory.init(keystore, passphrase);
```