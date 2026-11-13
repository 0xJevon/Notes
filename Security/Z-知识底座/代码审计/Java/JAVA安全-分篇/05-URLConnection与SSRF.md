---
type: 查阅
status: active
created: 2026-11-13
updated: 2026-11-13
tags: [Java, 代码审计, SSRF]
本线: 只查不读
源: JAVA安全-全本（2026-11-13 拆）
---

# 05 · URLConnection 与 SSRF

> 查阅区 · **只查不读**，不进学习路径。原文拆自 `JAVA安全-全本`，正文一字未改，仅调标题层级与图片相对路径。
> 什么时候翻：审 SSRF 时，确认发起方是 URLConnection/URL，协议支持范围到哪

## URLConnection

表示应用程序以及与URL建立通信连接的所有类的超类，通过`URL`类中的`openConnection`方法获取到`URLConnection`的类对象。

支持以下协议:

```java
file ftp mailto http https jar netdoc gopher(jdk8以后被阉割)
```

### SSRF

- Java中SSRF漏洞利用
  - 利用`file`协议读取文件内容（仅限使用`URLConnection|URL`发起的请求）
  - 利用`http` 进行内网web服务端口探测
  - 利用`http` 进行内网非web服务端口探测(如果将异常抛出来的情况下)
  - 利用http进行`ntlmrelay`攻击(仅限`HttpURLConnection`或者二次包装`HttpURLConnection`并未复写`AuthenticationInfo`方法的对象)
