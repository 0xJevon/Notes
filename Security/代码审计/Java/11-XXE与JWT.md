---
type: 查阅
status: active
created: 2026-11-13
updated: 2026-11-13
tags: [代码审计, Java, XXE, JWT]
本线: 按需查
源: JAVA安全-全本（2026-11-13 拆）
---

# 11 · XXE 与 JWT

> 查阅区 · **只查不读**，不进学习路径。原文拆自 `JAVA安全-全本`，正文一字未改，仅调标题层级与图片相对路径。
> 什么时候翻：XXE 找解析器类名；JWT 看算法与 kid 的处理方式

## XXE注入

-XMLReader

-SAXReader

-SAXBuilder

-Unmarshaller

-DocumentBuilder

/**

 \* 审计的函数

 \* 1. XMLReader

 \* 2. SAXReader

 \* 3. DocumentBuilder

 \* 4. XMLStreamReader

 \* 5. SAXBuilder

 \* 6. SAXParser

 \* 7. SAXSource

 \* 8. TransformerFactory

 \* 9. SAXTransformerFactory

 \* 10. SchemaFactory

 \* 11. Unmarshaller

 \* 12. XPathExpression

 */

## JWT（身份认证）

### 结构

由三段base64加密数据通过**.**连接构成token，第一段（header）包含加密算法和token类型，第二段包含数据（由加密算法+密匙共同加密），第三段是签名（包含**密匙**）

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```



### 利用

#### 空加密攻击

**无密匙**，也就是没有第三段数据，前提是要**后端代码支持空加密算法**，比较鸡肋

#### 密匙爆破

#### KID攻击

在header头中可以添加kid数据，用于指定使用的加密算法的文件路径，可能造成任意文件读取、SQLl注入、RCE
