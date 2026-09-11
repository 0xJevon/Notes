# Python for Pentesters

此处是为了实验渐构课程而做的一次尝试，目的是掌握学习最基本的划分的能力（也就是对目标教程100天python从新手到大师教程中的结构做基本的区分，也就是下层和上层材料），并做到短时间掌握一门编程语言（虽然之前接触过不同的编程语言，但应用场景基本为应付考试，并没有真的做到掌握），总之这是一次修改自己底层学习模型的尝试。

鉴于Python是一门语言，而语言的学习更偏向于内隐知识，内隐知识的构建更偏向于在实际运用通过输入输出的即时反馈来构建，所以文字材料的作用偏弱，所以此下只做部分对文字材料结构划分的尝试，而更多的学习还是直接对代码模仿修改来学习。

## 初识Python

第一节课是对Python的基本介绍如历史、优缺点、应用领域等，没有概念的构建。

Python有强大的库和海量的模块

然后是Python编译器和环境的构建以及注释的写法

## 语言元素

此处提到了计算机处理指令和程序的基础部件和发展历史，但与我的学习目的-》python不符，故跳过不管

### 变量和类型

出现第一个概念，以下是对原文字材料的划分以及简化

[变量]

"存储数据的载体" -》实际存在的数据或者说是存储器中存储数据的一块内存空间，变量的值可以被读取和修改

查找顺序：

“局部作用域” -》“嵌套作用域” -》“全局作用域” -》“内置作用域”

### 数字类型

- 实数int

- 浮点数

  - ```py
    import decimal
    x = 0.1  + 0.2
    print(x)
    
    c = decimal.Decimal('0.1')
    d = decimal.Decimal('0.2')
    print(c + d)
    
    """
    0.30000000000000004
    0.3
    
    """
    ```

- 复数

  - ```py
    x = 1 + 2j
    x.real #1.0
    x.imag #2.0
    ```

- 整数算法

### 布尔类型

- bool()
  - 值为0的数字类型
  - 空的序列和集合
  - 定义为Flase的对象

### 逻辑运算符

- not
- and

- or

#### 短路逻辑

(not 1) or (0 and 1) or (3 and 4) or (5 and 6) or (7 and 8 and 9) == 4

```py
print(3 and 4)
print(3 or 4)
print(0 and 1)
print(0 or 3)

"""
4 ture
3 ture
0 false
3 ture

"""
```

not 1 or 0 and 1 or 3 and 4 or 5 and 6 or 7 and 8 and 9 == 4

0 or 0 or 4 or 6 or 9

## 语言结构

### 顺序

### 分支

### 循环

## 序列

有序

- \+ \- * \

- id

  - 对象id值

- is/is not

- in/not in

- del

  - 删除对象

- min()/max()

- len()

- sum()

- sorted()

  - 排序
  - reverse 逆向排序

- all()

  - 是否全为真

- any()

  - 是否部分为真

- enumerate()

  - 逐个枚举对象和序列，也可指定开始序列号

  - ```py
    y = "YJW"
    enum = enumerate(y)
    print(list(enum)) #[(0, 'Y'), (1, 'J'), (2, 'W')]
    ```

- **zip()**

  - 逐个聚合

  - ```py
    x = [5, 2, 0]
    y = "YJW"
    zipped = zip(x, y)
    list(zipped) #[(5, 'Y'), (2, 'J'), (0, 'W')]
    ```

- **map()**

  - 以函数逐个迭代返回

  - ```py
    mapped = map(ord, "FishC") #返回ascii值
    list(mapped) #[70, 105, 115, 104, 67]
    
    ```

- **filter()**

  - 以函数过滤迭代

  - ```py
    y = "Yjw"
    print(list(filter(str.islower, y))) #过滤小写字母
    #['j', 'w']
    ```

    

### 字符串

**有序可变序列**

使用\转义，可在字符串中表示 ' and \ 如

```python
'\'hello, world!\''
```

- 原始字符串

  - 如\t，\n等，正确使用是\\\t，\\\n或

  - 若不需要\转义可使用

```python
		r'\'hello, world!\''
```

#### 基本用法

- 转化为字符串
  - str()

- 形式： ''
- 元素的重复
  - '元素1， 元素2，...  ' * n
- 元素拼接
  - +
- 索引
  - str[]
- 切片
  - str2[2:5]	左闭右开
  - str2[2::2]   自索引2开始，步长2
  - str2[::-1]    反转
- 长度
  - len()
- 首字母大写
  - 字符串首个字母
    - capitalize()
  - 单词首字母
    - title()
- 字母大写
  - upper()
- 字母索引
  - find()
  - 未找到报错
    - index()
- 指定字符，布尔返回
- 开始
  - startswith()
- 结束
  - endswith()
- 字符串指定位置，指定位置填指定字符
  - 居中，两侧

    - center()
  - 居右，左侧
    - rjust()
  - 居左，右侧
    - ljust()
- 判断构成
  - 纯数字

    - isdigit()
  - 纯字母
    - isalpha()
  - 混合
    - isalnum()
- 去除两侧空格
  - strip()

#### 格式化字符串

```python
a, b = 5, 10

print('{0} * {1} = {2}'.format(a, b, a * b))

print(f'{a} * {b} = {a * b}')
```

### 列表

**与字符串一样是有序可变序列**

- 转换为列表
  - list()

- 列表的形式：[元素1， 元素2，...  ]

- 列表元素的重复

  - [元素1， 元素2，...  ] * n

- 列表长度

  - len()

- 索引(含n个元素的列表)

  - 正向
    - 0 -> n-1
  - 反向
    - -1 -> -n 

- 循环遍历元素

  - for
  - 索引 + 元素
    - enumerate()

- 添加元素

  - append()
  - insert()

- 合并列表

  - +=
  - extend()

- 删除元素

  - 指定删除元素
    - remove()
  - 指定删除位置
    - pop()
  - 清空
    - clear()

- 切片

  - 切片赋值

    - ```py
      y = [1, 2, 3, 4, 5]
      y[1:4] = []
      print(y) #[1, 5]
      ```

- 排序

  - 首字母

    - sorted

  - 长度

    - ```python
      key=len
      ```

  - 任意规则的倒序

    - ```python
      reverse=True
      ```

#### 生成式

- 单变量
  - f = [x for x in range(1, 10)]
- 多变量
  - f = [x + y for x in 'ABCDE' for y in '1234567']

#### 生成器

- 生成元素
  - f = [x ** 2 for x in range(1, 1000)]
- 生成对象
  - f = (x ** 2 for x in range(1, 1000))
- yield

### 元组

**有序不可变序列**，用于打包解包

- 转化为元组
  - tuple()

- 元组的形式：[元素1， 元素2，...  ]
  - 元素不支持修改、添加、删除
- 索引
  - 获取元素
- 性能更优
  - 占用空间、创建时间



## 集合

- 创建语法

  - 字面量
    - {}
  - 构造器
    - set()
  - 推导式
    - {num for num in range(1, 100) if num % 3 == 0}

- 修改集合

  - 添加元素
    - add()
    - update()
  - 删除
    - discard()
    - remove()
    - 首元素出栈
      - pop()

- 集合性质

  - 交

    - &

    - ```python
      intersection()
      ```

  - 并

    - |

    - ```
      union()
      ```

  - 差

    - -

    - ```
      difference
      ```

  - 对称差

    - ^

    - ```
      symmetric_difference
      ```

  - 子集

    - \<=

    - ```
      issubset()
      ```

  - 超集

    - \>=

    - ```
      issuperset()
      ```

      

## 字典

- 键值对
  - 字面量
    - {键1:值1， 键2:值2, ...}
  - 构造器
    - dict(键1=值1， 键2=值2, ...)
    - dict(zip(['键1', '键2', ...], '值1值2...'))
  - 推导器
    - {num: num ** 2 for num in range(1, 10)}
- 键
  - 获取值
    - dict[]
    - get()
  - 修改键的对应值
    - 更新
      - update()
    - 清空
      - clear()
    - 删除
      - popitem()
        - 从后往前挨个输出键值对
      - pop()

## 函数

- def

- 形参和实参

- 位置参数

  - 位置固定
  - 在关键字参数前

- 关键字参数

  - 指定参数名

  - ```py
    def abc(a, b, c):
    	print(a, b, c)
        
    abc(a = 1, c = 2, b = 3)
    ```

- 默认参数

  - 定义时指定的参数值

  ```py
  sum(iterable, /, start = 0) #/左侧只能使用位置参数而不能使用关键字参数
  def abc(a, *, b, c): # *右侧只能使用关键字参数
  	print(a, b, c)
      
  abc(1, 2, 3) #会报错
  ```

- 收集参数

  - 传入参数数量不定

  - ```py
    def func1(*args): #打包为元组
        print("有{}个参数".format(len(args)))
        print(f'第2个参数是：{args[1]}')
    func1(1, 22, 3, 4)
    
    """
    有4个参数
    第2个参数是：22
    
    """
    def func2(**kwargs): #打包为字典，要指出键值对
        print(f'参数是：{kwargs}')
    func2(a = 1, b = 22, c = 3, d = 4)
    
    # 参数是：{'a': 1, 'b': 22, 'c': 3, 'd': 4}
    
    def test(a, *b, **c):
        print(a, b, c)
    test(1, 3, 5, y = 7, j = 9, w = 10)
    
    # 1 (3, 5) {'y': 7, 'j': 9, 'w': 10}
    
    help(str.format)
    ```

- 解包参数

  - ```py
    args = (1, 2, 3, 4)
    kwargs = {'a':1, 'b':2, 'c':3, 'd':4}
    def func(a, b, c, d):
    	print(a, b, c, d)
    func(args) # 报错，只有一个元素
    func(*args) # 解包元组
    func(**kwargs) # 同理解包字典
    ```

    

## 面向对象编程

### 三大特性

- 封装（黑箱）

- 继承

  - super()

  - 重写

  - 抽象类

    - ```python
      from abc import ABCMeta, abstractmethod
      ```

    - metaclass=ABCMeta
    - @abstractmethod

- 多态
  - 子类可重写父类方法

### 定义类

#### 动态方法

对对象的方法，给对象发消息

- 初始化对象(\__init__)

- 打印属性(\__str__)

- getter和setter

- 自定义方法

#### 静态方法

在对象创建前，给类发消息

- @staticmethod
- 可用if来判断

#### 类方法

获取和类相关的信息，并创建类的对象

- @classmethod

- 约束名为cls，代表当前类相关的信息的对象

  

### 访问可见性

Python中对属性和方法的访问权限只有public和private，一般以双下划线开头如__foo，来命名private，但仍可通过一定的规则访问，所以一般只以\_开头来暗示私人属性或方法。、

#### @property装饰器

为达到不将属性直接暴露给外界可使用，来包装getter和setter方法

### \__shots__

Python作为一门动态语言可在运行时对对象解绑或绑定新的属性和方法，可通过\_\_slots\_\_对当前类限制（对子类无效）

```
__slots__ = ('_name', '_age', '_gender')
```

### 类的关系

使用UML对类的关系进行统一的建模

- is-a
  - 继承
- has-a
  - 关联
- use-a
  - 依赖



## 文件和异常

### 读写文本文件

- 读写方式

  - 一次性读写

    - ```python
      f = open('致橡树.txt', 'r', encoding='utf-8')
      	print(f.read())
      ```

  - for-in单行循环读写

    - ```python
      with open('致橡树.txt', mode='r'， encoding='utf-8') as f:
              for line in f:
                  print(line, end='')
                  time.sleep(0.5)
          print()
      ```

  - readlines列表读写

    - ```python
      with open('致橡树.txt'， encoding='utf-8') as f:
              lines = f.readlines()
              print(lines)
      ```

- 文件写入

  - ```python
    open(filename, 'w', encoding='utf-8')
    with open('致橡树.txt', 'r', encoding='utf-8') as f:
    ```


- 路径处理 pathlib

- pickle序列化

  - 序列化dump

    - ```python
      with open("data.pkl", "wb") as f:
      	pickle.dump(x, y)
      ```

  - 反序列化load

    - ```python
      with open("data.pkl", "rb") as f:
      	z = pickle.dump(x, y)
      ```

[《总结：Python中的异常处理》](https://segmentfault.com/a/1190000007736783)

- 处理异常

  - try-except

    ```py
    try：
    	1 / 0
        5 + "five"
    except (ZeroDivisionError, ValueError, TypeError)：
    ```

  - try-except-else

- 异常与否都会执行

  - finally

- 抛出异常

  - raise


### 读写二进制文件

- 读写关键词
  - rb
  - wb

```python
<class 'bytes'>
```

### 读写JSON文件

- `dump` - 将Python对象按照JSON格式序列化到文件中
- `dumps` - 将Python对象处理成JSON格式的字符串
- `load` - 将文件中的JSON数据反序列化成对象
- `loads` - 将字符串的内容反序列化成Python对象

## 正则表达式 

[《正则表达式30分钟入门教程》](https://deerchao.net/tutorials/regex/regex.htm)

- #t 爬虫
  - Beautiful Soup
  - Lxml

## 模块和包

### 引用模块

```py
import 模块名
from 模块名 import 对象 #注意不同模块的对象命名冲突
import 模块名 as 关联名
```

- if \_\_name\__ == '\__main__':
  - 作为模块代入时，\_\_name\__为模块名，即其下代码不会作为模块代码导入执行

- \__all__属性
  - 对于模块若没有定义\__all__则from ... import * 语法将导入模块内所有对象
  - 对于包若没有定义\__all__则from ... import * 语法不导入包内任何模块

## 进程和线程

让程序同时执行多个任务

### 进程

#c 操作系统中执行的一个程序，以进程为单位分配存储空间，子进程有自己独立的内存空间，通过IPC实现数据共享。

### 线程

#c 一个进程拥有的多个并发的执行线索，即多个可以获得CPU调度的执行单元，多个线程共享进程的内存空间

### Python多进程

#c 使用multiprocessing模块的`Process`类来创建子进程，该模块还提供进一步的封装调用，如如批量启动进程的进程池（`Pool`）、用于进程间通信的队列（`Queue`）和管道（`Pipe`）等。

#t 使用multiprocessing模块中的`Queue`类解决子进程间的通信问题

### Python多线程

#c 1.使用threading模块的`Thread`类来创建线程

​	 2.通过继承`Thread`类的方式来创建自定义的线程类，然后再创建线程对象并启动线程。

- #t 临界资源

  - #c 多个线程共享一个变量（资源）可能造成不可控结果

  - #t 通过锁保护临界资源，得“锁”的线程才能访问“临界资源”

- #t “全局解释器锁”（GIL）
  - 存在于Python的解释器中，任何线程执行前必须先获得GIL锁，然后每执行100条字节码，解释器就自动释放GIL锁，让别的线程有机会执行

### 多进程/线程的局限

- 执行任务数量多时，可能会将资源浪费在任务的切换，使系统性能下降。

- 计算密集型任务主要消耗CPU资源也不适用Python

### 协程

- #c 单线程+异步I/O编程模型
  - 无线程切换消耗
  - 无多线程锁机制
  - 无临界资源冲突

## 网络编程

### 套接字

- #c 用[C语言](https://zh.wikipedia.org/wiki/C%E8%AF%AD%E8%A8%80)写成的应用程序开发库，主要用于实现进程间通信和网络编程

- 语法

  - ```python
    server = socket(family=, type=)
    ```

  - ```python
    from socket import socket, SOCK_STREAM, AF_INET
    # 需从库中引入
    family=AF_INET - IPv4地址
    family=AF_INET6 - IPv6地址
    type=SOCK_STREAM - TCP套接字
    type=SOCK_DGRAM - UDP套接字
    type=SOCK_RAW - 原始套接字
    ```

#### TCP套接字

- #c 使用TCP协议提供的传输服务来实现网络通信的编程接口

#### UDP套接字

#### 数据报套接字

#### 原始套接字

### 网络应用开发

#### 电子邮件

- SMTP协议

- ```python
  from smtplib import SMTP
  ```

#### 短信



### 图像和文档

#### Pillow操作图像

- 读取
- 裁剪 crop
- 缩略 thumbnail
- 缩放
- 粘贴 paste
- 旋转 rotate
- 翻转 transpose
- 操作像素 putpixel
- 滤镜 ImageFilter

#### Excel

```python
from openpyxl import Workbook
```

#### Word

```python
from docx import Document
```

## Python进阶

### 知识补充

- 生成式

- 嵌套列表

- heapq

  - 找出列表中最大和最小的N个元素

  - ```python
    heapq.nsmallest(N, list, key=lambda )
    ```

  - ```python
    heapq.nlargest(N, list, key=lambda )
    ```

- itertools

  - 迭代工具

- collections

  - namedtuple
  - deque
  - Counter
    - most_common()
  - OrderedDict
  - defaultdict
    - setdefault()

### 数据结构与算法

- 算法：解决问题的方法和步骤
  - 排序和查找
  - 常用算法
- 好坏的评判标准：渐近时间复杂度和渐近空间复杂度

### 函数的使用

- 高阶函数的用法

  ```python
  items1 = list(map(lambda x: x ** 2, filter(lambda x: x % 2, range(1, 10))))
  items2 = [x ** 2 for x in range(1, 10) if x % 2]
  ```

- 作用域

  - Python搜索变量的LEGB顺序或是优先级（Local >>> Embedded >>> Global >>> Built-in）

  - `global`和`nonlocal`关键字的作用

    `global`：声明或定义全局变量（要么直接使用现有的全局作用域的变量，要么定义一个变量放到全局作用域）。

    `nonlocal`：声明使用嵌套作用域的变量（嵌套作用域必须存在该变量，否则报错）。

- 闭包

  - 外部作用域可保存

- 位置参数、可变参数、关键字参数、命名关键字参数

- 参数的元信息（代码可读性问题）

- 匿名函数和内联函数的用法（`lambda`函数）

- 函数装饰器
  - 在不修改函数代码的情况下，动态地为函数或方法添加额外的功能。

```python
from functools import wraps

def log(func):
	@wraps()
	def wrapper(*args, **kwargs):
   		print(f"Calling {func.__name__} with arguments {args}, {kwargs}")
    	return func(*args, **kwargs)
	return wrapper

#添加装饰器功能
@log
def add(a, b):
    return a + b

#调用函数
add(2, 3)

# 输出：Calling add with arguments (2, 3), {}
```

### 面向对象

- 对象的复制（深复制/深拷贝/深度克隆和浅复制/浅拷贝/影子克隆）

- 垃圾回收、循环引用和弱引用

- 魔法属性和方法
  - 适用自定义魔法方法实现对象的运算
  - 适用hash和eq去重将对象放到`set`中或作为`dict`的键
  - 定义`__enter__` 和 `__exit__` 方法使用上下文语法`with`


- 混入类（Mixin）

- 元编程和元类

  对象是通过类创建的，类是通过元类创建的，元类提供了创建类的元信息。所有的类都直接或间接的继承自`object`，所有的元类都直接或间接的继承自`type`。

- 面向对象设计原则

  - 单一职责原则 （**S**RP）- 一个类只做该做的事情（类的设计要高内聚）
  - 开闭原则 （**O**CP）- 软件实体应该对扩展开发对修改关闭
  - 依赖倒转原则（DIP）- 面向抽象编程（在弱类型语言中已经被弱化）
  - 里氏替换原则（**L**SP） - 任何时候可以用子类对象替换掉父类对象
  - 接口隔离原则（**I**SP）- 接口要小而专不要大而全（Python中没有接口的概念）
  - 合成聚合复用原则（CARP） - 优先使用强关联关系而不是继承关系复用代码
  - 最少知识原则（迪米特法则，Lo**D**）- 不要给没有必然联系的对象发消息

  > **说明**：上面加粗的字母放在一起称为面向对象的**SOLID**原则。

- GoF设计模式

  - 创建型模式：单例、工厂、建造者、原型
  - 结构型模式：适配器、门面（外观）、代理
  - 行为型模式：迭代器、观察者、状态、策略

### 迭代器和生成器

- **迭代器是一次性的可迭代对象**
  - 其中元素只可提取一次

- 迭代器是实现了迭代器协议的对象。

- `__iter__`和`__next__`魔术方法是迭代器协议。

- 迭代器转化

  - iter()

- 提取迭代器元素

  - next()

- 生成器是语法简化版的迭代器。

- 生成器进化为协程。

  生成器对象可以使用`send()`方法发送数据，发送的数据会成为生成器函数中通过`yield`表达式获得的值。这样，生成器就可以作为协程使用，协程简单的说就是可以相互协作的子程序

### 并发编程

- 多线程

  - 线程调度

    - 多个线程竞争一个资源 - 保护临界资源 - 锁（Lock/RLock）

    - 多个线程竞争多个资源（线程数>资源数） - 信号量（Semaphore）
    - 多个线程的调度 - 暂停线程执行/唤醒等待中的线程 - Condition

  - GIL限制

    - 不能发挥CPU的多核特性。

- 多进程

  - 有效的解决了GIL的问题，能发挥CPU的多核特性。

- **多线程和多进程的比较**。
  - 以下情况需要使用多线程：


1. 程序需要维护许多共享的状态（尤其是可变状态），Python中的列表、字典、集合都是线程安全的，所以使用线程而不是进程维护共享状态的代价相对较小。
2. 程序会花费大量时间在I/O操作上，没有太多并行计算的需求且不需占用太多的内存。
   - 以下情况需要使用多进程：


1. 程序执行计算密集型任务（如：字节码操作、数据处理、科学计算）。
2. 程序的输入可以并行的分成块，并且可以将运算结果合并。
3. 程序在内存使用方面没有任何限制且不强依赖于I/O操作（如：读写文件、套接字等）。

- 异步I/O

  - 从调度程序的任务队列中挑选任务，该调度程序以交叉的形式执行这些任务

  - 关键词
    - `asyncio`模块
      - `await`
      - `async`

  - 利用点
    - 当程序不需要真正的并发性或并行性，而是更多的依赖于异步处理和回调时

