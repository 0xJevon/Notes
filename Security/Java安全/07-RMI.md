---
type: 查阅
status: active
created: 2026-11-13
updated: 2026-11-13
tags: [Java安全, RMI, 反序列化]
本线: 只查不读
源: JAVA安全-全本（2026-11-13 拆）
---

# 07 · RMI

> 查阅区 · **只查不读**，不进学习路径。正文拆自 `JAVA安全-全本`，未改写；仅合并 RMI 基础与 RMI 攻击两段、调标题与图片路径。
> 什么时候翻：看清 RMI 三部分与九个过程，判断反序列化发生在哪一段。

## RMI

`Remote Method Invocation`，Java远程方法调用，用于构建分布式应用程序，实现了`Java`程序之间跨`JVM`的远程通信。将整个远程调用的复杂网络通信流程封装为代理如Stub

![Snipaste_2025-03-26_12-43-26](_图/Snipaste_2025-03-26_12-43-26.png)

### 具体工作原理

![Routine](_图/Routine.png)

分为三部分

- Server
- Registry
- Client

涉及各自的创建(3)以及三部分之间两两通信(6)总计**九个**过程，以下会从RMI的实现然后是具体的代码流程进行分析



流程图

```java
sequenceDiagram
    participant Client
    participant Registry
    participant Stub
    participant Server

    Client->>Registry: lookup("ServiceName")
    Registry->>Client: 返回Stub（代理对象）
    Client->>Stub: 调用方法（如process()）
    Stub->>Server: 发送序列化请求（方法名、参数）
    Server->>Stub: 返回序列化结果
    Stub->>Client: 反序列化结果
```

### 服务端实现

1. 首先要创建一个远程接口，其中定义了要调用的方法，用于客户端远程调用

   ```java
   import java.rmi.Remote;
   import java.rmi.RemoteException;
   
   //继承Remote接口
   public interface IRemoteObj extends Remote {
       public String sayHello(String keywords) throws RemoteException;
   }
   ```

   - 接口必须继承`java.rmi.Remote`
   - 声明的方法必须抛出`java.rmi.RemoteException`

2. 然后要实现接口中的方法

   ```java
   import java.rmi.RemoteException;
   import java.rmi.server.UnicastRemoteObject;
   
   public class RemoteObjImpl extends UnicastRemoteObject implements IRemoteObj {
       public RemoteObjImpl() throws RemoteException{
   //        UnicastRemoteObject.exportObject(this, 0);
       }
   
       @Override
       public String sayHello(String keywords) throws RemoteException {
           String upKeywords = keywords.toUpperCase();
           System.out.println(upKeywords);
           return upKeywords;
       }
   }
   ```

   - 实现类必须继承`UnicastRemoteObject`，或手动导出对象（注释处）

3. 最后创建服务端对象

   ```java
   import java.rmi.AlreadyBoundException;
   import java.rmi.RemoteException;
   import java.rmi.registry.LocateRegistry;
   import java.rmi.registry.Registry;
   
   public class RMIServer {
       public static void main(String[] args) throws RemoteException, AlreadyBoundException {
           //实例化对象
           RemoteObjImpl remoteObj = new RemoteObjImpl();
           //创建注册中心
           Registry registry = LocateRegistry.createRegistry(1099);
           //将对象绑定到注册中心
           registry.bind("remoteObj", remoteObj);
       }
   }
   ```

   - Registry中存储Stub(客户端代理)，Stub是整个远程对象，包括方法的执行

### 客户端实现

1. 同样要创建远程接口用于获取远程调用的对象

   ```java
   import java.rmi.Remote;
   import java.rmi.RemoteException;
   
   public interface IRemoteObj extends Remote {
       public String sayHello(String keywords) throws RemoteException;
   }
   ```

2. 然后创建客户端对象

   ```java
   import java.rmi.NotBoundException;
   import java.rmi.RemoteException;
   import java.rmi.registry.LocateRegistry;
   import java.rmi.registry.Registry;
   
   public class RMIClient {
       public static void main(String[] args) throws RemoteException, NotBoundException {
           //从注册中心中获取stub用于处理远程对象的网络请求
           Registry registry = LocateRegistry.getRegistry("127.0.0.1", 1099);
           //确定要调用的网络接口
           IRemoteObj remoteObj = (IRemoteObj) registry.lookup("remoteObj");
           //调用网络接口下的方法
           remoteObj.sayHello("Hello");
       }
   }
   ```

   - Client与Registry通信后首次获取Stub，其后便直接通过Stub调用方法，也就是Stub是实际处理Client与Server之间网络通信而封装的代理
   - Stub中封装了服务端的网络地址(IP:Port)和对象标识(ObjID)，还负责处理序列化与反序列化的网络通信
   - 客户端获取的是Stub的序列化数据，然后在本地重建代理对象。

### 服务端创建

先分析服务端的具体创建，在实例化对象处打断点调试

![Snipaste_2025-04-03_13-48-49](_图/Snipaste_2025-04-03_13-48-49.png)

- 然后一直跟到`UnicastRemoteObject`里面，在这里面封装并发布了具体实现网络请求的代理，这里默认端口为0，若没有具体给定端口则后面会自行赋值一个随机端口，然后具体网络服务的发布是通过`exportObject()`实现，继续跟进

  ![Snipaste_2025-04-03_12-00-16](_图/Snipaste_2025-04-03_12-00-16.png)

  - 这里的obj是实现远程方法调用的代码逻辑的，那处理网络请求的就是`UnicastServerRef()`

    ![Snipaste_2025-04-03_12-00-54](_图/Snipaste_2025-04-03_12-00-54.png)

  - 然后跟进可以看到它`new LiveRef()`，而这个`LiveRef`就是非常重要的一个类了，算是一个网络引用的类，里面封装了很多涉及网络处理的对象

    ![Snipaste_2025-04-03_12-03-12](_图/Snipaste_2025-04-03_12-03-12.png)

  - ObjID就是对象标识，主要看`LiveRef`的构造函数，跟进this

    ![Snipaste_2025-04-03_12-03-26](_图/Snipaste_2025-04-03_12-03-26.png)

    - 前面封装了怎么多层，终于出现了涉及具体网络请求的类`TCPEndpoint`，可以看到传入host和port就能具体处理网络请求

    ![Snipaste_2025-04-03_12-04-21](_图/Snipaste_2025-04-03_12-04-21.png)

  - 回到`LiveRef`的构造，可以看到host和port是赋值到endpoint中的，而endpoint又在LiveRef里面，所以涉及网络处理的数据都在`LiveRef`中，而且后面会看到不管是客户端还是服务端的`LiveRef`自始至终都是同一个

    ![Snipaste_2025-04-03_12-05-18](_图/Snipaste_2025-04-03_12-05-18.png)

- 回到最初的`LiveRef`，会发现在它的父类`UnicastRef`进行了一个赋值操作，而这个类对应的客户端，之前的`UnicastServerRef`对应的是服务端，这也说明`LiveRef`始终没变

  ![Snipaste_2025-04-03_12-06-48](_图/Snipaste_2025-04-03_12-06-48.png)

- 一路跟最终又调用了一个新的`exportObject()`，这里可以发现整个创建过程就是不停的将网络处理过程赋值封装再调用`exportObject()`，最终的目的就是发表网络服务

  ![Snipaste_2025-04-03_12-07-19](_图/Snipaste_2025-04-03_12-07-19.png)

- 接着就是一路跟踪一直到又一个重要的封装对象Stub处，会发现Stub这个动态代理是在Server创建的，然后发布到Registry，最后再被Client获取

  ![Snipaste_2025-04-03_12-08-43](_图/Snipaste_2025-04-03_12-08-43.png)

  - 跟进看具体的创建过程，这里封装了远程对象的实现类以及IP:Port

    ![Snipaste_2025-04-03_12-14-00](_图/Snipaste_2025-04-03_12-14-00.png)

  - 这里就看到了通过类加载创建动态代理的过程

    ![Snipaste_2025-04-03_12-18-05](_图/Snipaste_2025-04-03_12-18-05.png)

- 最后创建好了动态代理Stub

  ![Snipaste_2025-04-03_12-19-40](_图/Snipaste_2025-04-03_12-19-40.png)

- 再一路跟到下一个重要的封装对象Target，这里的Target就是将所有封装对象再做了一次总的封装，最后也是将Taget发布到Registry上

  ![Snipaste_2025-04-03_12-20-16](_图/Snipaste_2025-04-03_12-20-16.png)

  - 跟进查看一下Taget都封装了什么

    ![Snipaste_2025-04-03_12-22-04](_图/Snipaste_2025-04-03_12-22-04.png)

  - 这里比较一下disp和stub，分别对应Server和Client，可以看到ref是同一个

    ![Snipaste_2025-04-03_12-23-47](_图/Snipaste_2025-04-03_12-23-47.png)

- 跳出去就到了Target的发布

  ![Snipaste_2025-04-03_12-25-28](_图/Snipaste_2025-04-03_12-25-28.png)

  - 看一下具体的发布逻辑，这里又是在不同类中反复调用`exportObject`，最后到了`TCPTransport`中，这个类也就是真正的执行类

    ![Snipaste_2025-04-03_12-27-21](_图/Snipaste_2025-04-03_12-27-21.png)

  - 这里服务端开了监听端口

    ![Snipaste_2025-04-03_12-27-55](_图/Snipaste_2025-04-03_12-27-55.png)

  - 再到下面的`newServerSocket()`

    ![Snipaste_2025-04-03_12-29-03](_图/Snipaste_2025-04-03_12-29-03.png)

    ![Snipaste_2025-04-03_12-33-15](_图/Snipaste_2025-04-03_12-33-15.png)

  - 这里创建了新的socket等待连接，并且若默认port为0这里还进行了端口的随机赋值

    ![Snipaste_2025-04-03_12-40-03](_图/Snipaste_2025-04-03_12-40-03.png)

- 之后就是在Thread完成网络连接的过程，整个创建过程也就完成了，将Target发布到Registry后就是等待Client获取Stub连接通信

  ![Snipaste_2025-04-03_12-37-22](_图/Snipaste_2025-04-03_12-37-22.png)

  ![Snipaste_2025-04-03_12-37-52](_图/Snipaste_2025-04-03_12-37-52.png)

- 在发布完成后在Server中还会有记录

  ![Snipaste_2025-04-03_14-42-53](_图/Snipaste_2025-04-03_14-42-53.png)

  - `target.setExportedTransport(this)`就是一些赋值，记录发生在`ObjectTable`中，跟进`ObjectTable.*putTarget*(target)`

    ![Snipaste_2025-04-03_12-42-24](_图/Snipaste_2025-04-03_12-42-24.png)

    ![Snipaste_2025-04-03_12-45-00](_图/Snipaste_2025-04-03_12-45-00.png)

    - 这里将信息保存到了*objTable*和*implTable*两个表中

- 总结一下整个创建过程就是通过`exportObject()`将远程服务发布到指定的IP和Port上，并且还创建了用于客户端操作的网络代理Stub，只是不停的赋值封装，然后又在不同的类中反复调用`exportObject()`，使整个过程显得复杂，最后的记录保存到静态的HashMap中，可以理解为日志

### 注册中心创建

在createRegistry处下断点

![Snipaste_2025-04-03_16-32-07](_图/Snipaste_2025-04-03_16-32-07.png)

- 然后跟进`createRegistry()`，Registry默认端口就是1099，所以这里写与不写都可

  ![Snipaste_2025-04-03_16-33-27](_图/Snipaste_2025-04-03_16-33-27.png)

- 接着往下创建了`RegistryImpl`，这里的if进行了安全检查，不重要

  ![Snipaste_2025-04-03_16-35-54](_图/Snipaste_2025-04-03_16-35-54.png)

- 然后这里创建了`LiveRef`

  ![Snipaste_2025-04-03_16-37-05](_图/Snipaste_2025-04-03_16-37-05.png)

  - 可以看到和Server创建的ref基本一致

    ![Snipaste_2025-04-03_16-37-48](_图/Snipaste_2025-04-03_16-37-48.png)

  - 但是跟进`setup()`就有不同的了，这里的第三个参数为true，也就是创建了一个永久对象，而下面那张图也就是Server中的参数为flase，所以创建的是临时对象

    ![Snipaste_2025-04-03_16-38-59](_图/Snipaste_2025-04-03_16-38-59.png)

    ![Snipaste_2025-04-03_12-07-19](_图/Snipaste_2025-04-03_12-07-19.png)

    - 但二者的逻辑基本一致都是先赋值再用`exportObject()`发布

- 接着往下有到了创建`Stub`，这里用于Registry与Client或Server进行通信

  ![Snipaste_2025-04-03_16-46-18](_图/Snipaste_2025-04-03_16-46-18.png)

  - 但跟进`createProxy()`后会发现Registry创建Stub与Server创建的逻辑不同

    ![Snipaste_2025-04-03_16-48-21](_图/Snipaste_2025-04-03_16-48-21.png)

    ![Snipaste_2025-04-03_16-48-47](_图/Snipaste_2025-04-03_16-48-47.png)

  - 这里判断是否存在所创建对象且后缀为_Stub的类，也就是`RegistryImpl_Stub`类，若存在则会创建Stub，而这个类是封装在JDK中的，所以Registry的Stub由JDK直接提供，可以看到后面Skeleton的创建也是相同的方法

    ![Snipaste_2025-04-03_16-50-29](_图/Snipaste_2025-04-03_16-50-29.png)

  - 直接通过反射获取自带类创建

    ![Snipaste_2025-04-03_16-52-01](_图/Snipaste_2025-04-03_16-52-01.png)

- 接着往下则是创建Skeleton

  ![Snipaste_2025-04-03_16-54-50](_图/Snipaste_2025-04-03_16-54-50.png)

  ![Snipaste_2025-04-03_16-56-50](_图/Snipaste_2025-04-03_16-56-50.png)

  ![Snipaste_2025-04-03_16-57-15](_图/Snipaste_2025-04-03_16-57-15.png)

- Skeleton也是通过反射加载的相同逻辑不多赘述，接下来也是Target封装全部对象

  ![Snipaste_2025-04-03_16-58-59](_图/Snipaste_2025-04-03_16-58-59.png)

#### Registry封装的内容

- 然后发布的逻辑都一样直接到`ObjectTable`处看都封装了些什么

  ![Snipaste_2025-04-03_17-08-00](_图/Snipaste_2025-04-03_17-08-00.png)

  - 可以看到这里共有三个对象，分别查看三个对象中disp下的ref和stub，第一个对象是`$Proxy`

    ![Snipaste_2025-04-03_17-17-45](_图/Snipaste_2025-04-03_17-17-45-17436732751891.png)

  - 然后第二个是`DGCImpl_Stub`，它是自动创建的，是分布式垃圾回收的一个对象

    ![Snipaste_2025-04-03_17-18-21](_图/Snipaste_2025-04-03_17-18-21.png)

  - 然后第三个才是这次创建的`RegistryImpl_Stub`

    ![Snipaste_2025-04-03_17-19-01](_图/Snipaste_2025-04-03_17-19-01.png)

  - 可以看到三者中的ref和stub都相同，所以都是一个东西，但代码只创建`RegistryImpl`对象，但实际上却有三个对象，这里之后讨论

### 绑定

过程很简单就是`hashTable.put(IP, port)`，检查name是否存在，不存在则put进去

![Snipaste_2025-04-03_17-54-50](_图/Snipaste_2025-04-03_17-54-50.png)

### 客户端请求注册中心

#### 获取注册中心

通过`getRegistry()`获取，在此处下断点

![Snipaste_2025-04-04_09-56-48](_图/Snipaste_2025-04-04_09-56-48.png)

- 进入方法，通过host和port获取

  ![Snipaste_2025-04-04_09-57-14](_图/Snipaste_2025-04-04_09-57-14.png)

- 往下跟同样的创建`LiveRef`，然后封装ip和端口，随后建立Stub进行通信

  ![Snipaste_2025-04-04_09-57-47](_图/Snipaste_2025-04-04_09-57-47.png)

  - 可以跟进看一下`Stub`的创建

    ![Snipaste_2025-04-04_09-59-54](_图/Snipaste_2025-04-04_09-59-54.png)

    ![Snipaste_2025-04-04_10-00-55](_图/Snipaste_2025-04-04_10-00-55.png)

    - 可以看到和Registry建立`Stub`是一样的逻辑都是JDK自带的

#### 查找远程对象

与Registry建立通信后便是使用`lookup()`查找远程对象的`Stub`

![Snipaste_2025-04-04_10-02-17](_图/Snipaste_2025-04-04_10-02-17-17437344863001.png)

- 进入该方法，可以看到序列化要请求的远程对象，Registry会通过反序列化读取

  ![Snipaste_2025-04-04_10-06-59](_图/Snipaste_2025-04-04_10-06-59.png)

- 漏洞点出现在下面的`invoke()`中，通过代码逻辑可以看出该方法是处理网络请求的，跟进去是一个接口，然后在`UnicastRef`找到接口的实现

  ![Snipaste_2025-04-04_10-08-23](_图/Snipaste_2025-04-04_10-08-23.png)

  ![Snipaste_2025-04-04_10-09-02](_图/Snipaste_2025-04-04_10-09-02.png)

  - 代码会从`executeCall()`调用到`out.getDGCAckHandler()`，而这个方法下面的异常2本意是将报错信息整个拿出来，但是若注册中心返回恶意的对象此处便可作为漏洞点利用，而且Client的每次通信都会触发该方法。

    ![Snipaste_2025-04-04_10-13-46](_图/Snipaste_2025-04-04_10-13-46.png)

- 回到原本的逻辑还有一处漏洞点，可以看到Client和Server通信的Stub同样以序列化的形式传输，也意味着可以构造恶意对象进行利用

  ![Snipaste_2025-04-04_10-10-41](_图/Snipaste_2025-04-04_10-10-41.png)

- 最后便是获取到了远程对象也就是Stub动态代理

  ![Snipaste_2025-04-04_10-19-16](_图/Snipaste_2025-04-04_10-19-16.png)

### 客户端请求服务端

此处便是客户端请求服务端调用远程对象其下的方法，也就是远程方法调用

![Snipaste_2025-04-04_10-20-16](_图/Snipaste_2025-04-04_10-20-16.png)

- 跟进invoke()中间就是一堆判断然后抛出异常，到尾部才是方法调用处

  ![Snipaste_2025-04-04_10-21-33](_图/Snipaste_2025-04-04_10-21-33-17437359384253.png)

  - 进入`invokeRemoteMethod()`，这里有个重载的`invoke()`，漏洞点就在该方法中

    ![Snipaste_2025-04-04_10-22-53](_图/Snipaste_2025-04-04_10-22-53.png)

  - 往下跟到循环中有一个`marshalValue()`，这里将参数序列化

    ![Snipaste_2025-04-04_10-24-54](_图/Snipaste_2025-04-04_10-24-54.png)

    ![Snipaste_2025-04-04_11-14-12](_图/Snipaste_2025-04-04_11-14-12.png)

  - 再往后又调用了`call.executeCall()`之前看到过该方法中有个异常报错漏洞点

    ![Snipaste_2025-04-04_10-26-05](_图/Snipaste_2025-04-04_10-26-05.png)

  - 再往下是另一个漏洞点，此处会将远程方法调用的结果返回

    ![Snipaste_2025-04-04_10-27-32](_图/Snipaste_2025-04-04_10-27-32.png)

    ![Snipaste_2025-04-04_10-29-23](_图/Snipaste_2025-04-04_10-29-23.png)

#### 小结

- Client远程方法调用的整个流程就是先获取注册中心，然后查找远程对象获取到Server发布的ref，然后与Server建立连接，进行通信，调用方法，获得返回值

### 注册中心处理客户端请求

客户端那边操作的是`Stub`，而服务端这边操作的是`Skel`，而`Skel`经过网络处理后会放在Traget里面，所以将断点打在处理Target的地方

![Snipaste_2025-04-04_18-16-06](_图/Snipaste_2025-04-04_18-16-06.png)

- 然后是将`skel`放进`disp`

  ![Snipaste_2025-04-04_18-20-24](_图/Snipaste_2025-04-04_18-20-24.png)

- 随后进入`dispatch()`

  ![Snipaste_2025-04-04_11-45-40](_图/Snipaste_2025-04-04_11-45-40.png)

  - 因为skel不为null，所以进入`oldDispatch()`

  ![Snipaste_2025-04-04_11-46-46](_图/Snipaste_2025-04-04_11-46-46.png)

  - 然后是`skel.dispatch()`，这里就是**客户端打注册中心**的攻击方式

    ![Snipaste_2025-04-04_11-47-42](_图/Snipaste_2025-04-04_11-47-42.png)

- 然后漏洞发生在`RegistryImpl_Skel`

  ![Snipaste_2025-04-04_11-48-41](_图/Snipaste_2025-04-04_11-48-41.png)

  - 会发现里面的`bind()`、`lookup()`、`rebind()`、`unbind()`都可以触发反序列化

    ![Snipaste_2025-04-04_18-44-28](_图/Snipaste_2025-04-04_18-44-28.png)

    ![Snipaste_2025-04-04_18-44-57](_图/Snipaste_2025-04-04_18-44-57.png)

    ![Snipaste_2025-04-04_18-45-27](_图/Snipaste_2025-04-04_18-45-27.png)

    ![Snipaste_2025-04-04_18-45-27](_图/Snipaste_2025-04-04_18-45-27.png)

### 服务端处理客户端请求

断点同样下在Target处，但这里又一个坑点，第一个Target的Stub是DGCImpl ，要的并不是这个，而是Proxy 动态代理的 Stub

![Snipaste_2025-04-04_17-25-57](_图/Snipaste_2025-04-04_17-25-57.png)

- 同样进入`dispatch()`方法下，因为`skel=null`，所以不会执行`oldDispatch ()`

  ![Snipaste_2025-04-04_17-26-37](_图/Snipaste_2025-04-04_17-26-37.png)

- 继续往下获取输入流以及远程对象的方法

  ![Snipaste_2025-04-04_17-27-45](_图/Snipaste_2025-04-04_17-27-45.png)

- 然后在下面的循环中通过`unmarshalValue()`，反序列化Client传过来的参数

  ![Snipaste_2025-04-04_17-31-49](_图/Snipaste_2025-04-04_17-31-49.png)

  ![Snipaste_2025-04-04_17-30-32](_图/Snipaste_2025-04-04_17-30-32.png)

- 继续往下就是Server调用方法获取结果

  ![Snipaste_2025-04-04_17-32-37](_图/Snipaste_2025-04-04_17-32-37-17437686299245.png)

  - 并且Server同样使用`marshaValue()`将结果序列化后返回给Client，Client同样使用`unmarshalValue()`反序列化获取结果

    ![Snipaste_2025-04-04_17-33-50](_图/Snipaste_2025-04-04_17-33-50.png)

### DGC

前面提到过在创建注册中心时会创建三个对象封装到`ObjectTable`中，而`DGCImpl_Stub`在传入前就自动创建了，可以看到在put前就已经存在了

![Snipaste_2025-04-04_17-43-56](_图/Snipaste_2025-04-04_17-43-56.png)

那么DGC这个类是在哪儿创建的呢？又是怎么创建的呢？断点打在前面的if处

![Snipaste_2025-04-04_17-41-05](_图/Snipaste_2025-04-04_17-41-05.png)

- 这里调用了`DGCImpl`的*dgcLog*变量，而这个变量是一个静态变量

  ![Snipaste_2025-04-04_17-46-29](_图/Snipaste_2025-04-04_17-46-29.png)

  - 调取一个类的静态会完成这个类的初始化，而在类加载中我们知道了类的初始化会调用类的静态代码块，而这个类的静态代码块中有`run()`，该方法的下方的创建对象处就是完整的创建ref、将ref放入disp并创建stub动态代理的过程

    ![Snipaste_2025-04-04_17-47-03](_图/Snipaste_2025-04-04_17-47-03.png)

  - 可以跟进`createProxy()`，可以发现`DGCImpl_Stub`的创建和Registry创建Stub时一致

    ![Snipaste_2025-04-04_17-55-16](_图/Snipaste_2025-04-04_17-55-16.png)

    - 同样判断有无`DGCImpl_Stub`类，很明显JDK中自带了，此处就是将它实例化了

      ![Snipaste_2025-04-04_17-56-22](_图/Snipaste_2025-04-04_17-56-22.png)

  - 开启随机端口用于远程回收，并便放进Target中

    ![Snipaste_2025-04-04_17-57-46](_图/Snipaste_2025-04-04_17-57-46.png)

  - 之后的`setSkeleton()`和Registry中的过程一致，创建skel放入disp中

- 漏洞同样存在于Stub和Skel中，先看Stub中同样调用了`ref.invoke()`，造成JRMP攻击

  ![Snipaste_2025-04-04_17-59-47](_图/Snipaste_2025-04-04_17-59-47.png)

  - 然后是`dirty()`还有`readObject()`，这里的`clean()`和`dirty()`分别是强清除内存和弱清除内存

    ![Snipaste_2025-04-04_18-00-17](_图/Snipaste_2025-04-04_18-00-17.png)

- 然后看Skel中`dispatch()`中同样存在漏洞点

  ![Snipaste_2025-04-04_18-04-31](_图/Snipaste_2025-04-04_18-04-31.png)

#### 小结

自动创建的清除内存的过程，在Stub和Skel中都存在漏洞点，也就是JRMP攻击


## RMI 攻击方式


根据基础中分析的RMI运行流程可知有以下几种攻击方式

- Client打Registry
- Client打Server
- Client

但在8u121版本后很多攻击方式以及被过滤了

- 在`RegistryImpl`中加了一个`registryFileter`函数，限制了可以反序列化的类

  ![Snipaste_2025-04-05_10-54-17](_图/Snipaste_2025-04-05_10-54-17.png)

- 同样在`DGCImpl`中也加入了`checkInput`函数，限制了更少的类可以反序列化

  ![Snipaste_2025-04-05_10-57-15](_图/Snipaste_2025-04-05_10-57-15.png)

总的来讲RMI的利用范围并不广，且利用的主要就是CC链再做一些变形，不做深入的代码分析，深入的攻击利用可以参考文章：

https://drun1baby.top/2022/07/23/Java%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E4%B9%8BRMI%E4%B8%93%E9%A2%9802-RMI%E7%9A%84%E5%87%A0%E7%A7%8D%E6%94%BB%E5%87%BB%E6%96%B9%E5%BC%8F/#1-%E6%94%BB%E5%87%BB-RMI-Registry


