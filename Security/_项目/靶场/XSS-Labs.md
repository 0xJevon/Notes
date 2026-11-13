# XSS-Labs

万能检测语句 ：<SCRscriptIPT>()Oonnjavascript

## Level-1

使用JS弹窗函数

## Level-2

双引号闭合绕过

## Level-3

htmlspecialchars函数（把预定义的字符转换为HTML实体。）针对**<>**进行html实体化

> ```javascript
> 预定义的字符是：
> &（和号） 成为&amp;
> " （双引号） 成为 &quot;
> ' （单引号） 成为 &apos;
> < （小于） 成为 &lt;
> > （大于） 成为 &gt;
> ```

htmlspecialchars默认不对`'`进行处理

onfocus事件获得焦点时触发，最常与 <input>、<select> 和 <a> 标签一起使用，以上面图片的html标签<input>为例，<input>标签是有输入框的

onfocus事件就是当输入框被点击的时候，就会触发myFunction()函数，然后我们再配合javascript伪协议来执行javascript代码
使用  ' onfocus=javascript:alert() '

onclick等

## Level-4

同上

## Level-5

strtolower()：转换为小写字母

<a>的href属性：当标签<a>被点击的时候，就会触发执行转跳，上面是转跳到一个网站，我们还可以触发执行一段js代码

"> <a href=javascript:alert()> xss </a> <"

## Level-6

大小写绕过

## Level-7

双写绕过

<img>

<i f r a m e>

## Level-8

编码绕过

 href属性自动解析Unicode编码

可通过链接将脚本语言传给href

## Level-9

代码审计，if判断是否有http://

## Level-10

可修改/添加参数名

对于onfocus隐藏的input标签可以插入**type**="text"显示 

## Level-11

http头传参

添加Referer参数

注意参数位置

## Level-12

http-user-agent传参

## Level-13

Cookie传参

## Level-14

exif xss

exif（可交换图像文件格式）：记录数码照片的属性信息和拍摄数据

上传带xss代码的图片

## Level-15

ng-include文件包含，可包含之前任意一关并传参以达到弹窗的效果

不能包含直接弹出的代码，但可以有标签

## Level-16

回车代替空格绕过检测，回车的url编码是%0a

不用/的标签：<img>、<details>、<svg>

## Level-17

## Level-18

## Level-19

## Level-20

以上四关都是flash xss，19，20涉及swf反编译