---
type: 判定
status: active
created: 2026-11-13
updated: 2026-11-13
tags: [代码审计, Java, 框架, 入口]
本线: 每次接新项目过一遍
---
# 02 · 项目形态（Servlet · SpringBoot · Maven）

> **为什么排在第 2 篇**：不知道项目形态，就不知道路由从哪来、拦截在哪、依赖有哪些。这一步答不出来，后面所有 sink 都跟不到入口。
> 依据：Java 代码审计最重要的是**开发模式**，最常见两类——基于 Servlet 的项目、基于 SpringBoot 的项目；两者都大概率用 Maven 管依赖。三样必须熟。
> 细节已并入本篇与 [[05-SpringBoot-Web路由与MVC]]、[[06-MyBatis与SQL审计]]；原来的 4,857 行「框架底座」是 Java 教材，2026-11-13 移出主库。

---

## 一、拿到源码先答六问（答不出就别往下跟）

| # | 问题 | 在哪看 | 答不出会怎样 |
|---|---|---|---|
| 1 | Servlet 还是 SpringBoot（还是 SSM 混合） | 有无 `web.xml`；有无 `@SpringBootApplication` | 找不到路由表 |
| 2 | 路由从哪个文件/注解来 | `web.xml` 的 `url-pattern` / `@RequestMapping` 家族 | 入口清单为空 |
| 3 | 拦截器与过滤器有哪几个、顺序如何 | `filter-mapping` 顺序 / `WebMvcConfigurer` / Shiro 配置 | 把已拦的洞当成可利用 |
| 4 | 依赖与版本（尤其 fastjson / shiro / log4j / commons-collections） | `pom.xml` / `build.gradle` / `WEB-INF/lib` | 漏掉 1day |
| 5 | 是单体还是多模块 | `pom.xml` 的 `modules` / 目录并排多个 `pom.xml` | 只看了一个子模块 |
| 6 | 有没有 Actuator / Swagger / Druid 监控页 | 依赖 + `application.yml` | 漏掉最省事的那条路 |

---

## 二、Servlet 项目

**特征**：有 `WEB-INF/web.xml`；`WEB-INF/lib` 一堆 jar；源码是 `*.java` + `*.jsp`；没有启动类。

### 2.1 路由的三个来源

| 来源 | 写法 | 怎么提 |
|---|---|---|
| `web.xml` | `<servlet><servlet-name>x</servlet-name><servlet-class>...</servlet-class></servlet>` + `<servlet-mapping><url-pattern>/a/b</url-pattern>` | 先导 `servlet-name` ↔ `servlet-class` ↔ `url-pattern` 三列表 |
| 注解 | `@WebServlet("/a/b")`、`@WebFilter` | 全局搜 `@WebServlet`、`@WebFilter`、`@WebListener` |
| JSP 直连 | 文件路径即 URL（`/admin/list.jsp`） | 列 `*.jsp` 文件树；重点看无 `session` 校验的页 |

> 老项目常见混用：一部分在 `web.xml`，一部分用注解。**两处都要提，不能只看一处。**

### 2.2 `web.xml` 是统御全局的文件

按顺序读三块：

1. `<context-param>` / `<listener>`：启动时加载什么（如 Spring 的 `ContextLoaderListener`、`IntrospectorCleanupListener`）。
2. `<filter>` + `<filter-mapping>`：**顺序即执行顺序**，`<url-pattern>` 决定拦哪些路径。`/*` 之外的范围就是缺口。
3. `<servlet>` + `<servlet-mapping>`：真正的入口表。注意 `load-on-startup` 与 `<init-param>`。

**判定动作**：把 `filter-mapping` 的 `url-pattern` 与 `servlet-mapping` 的 `url-pattern` 做交集/差集——差集里就是"未被过滤器覆盖的入口"，这是最快的一批候选。

### 2.3 Filter 的审计点

- 逻辑在 `init()` / `doFilter()`；`chain.doFilter()` 之前的判断才是拦截。
- 常见绕过：只判 `getRequestURI()` 前缀（`/admin` 可被 `/admin/../admin`、`/admin;.js`、`//admin` 绕）；只判后缀（`.js` 放行）；`doFilter` 里 return 但没 `return` 到底。
- 对应的判定卡记法见 [[20-漏洞清单与挖掘思路]] §身份验证。

---

## 三、SpringBoot 项目

**特征**：无 `web.xml`；有 `@SpringBootApplication` 启动类；`application.yml` / `application.properties`；`src/main/resources` 下有 `static`、`templates`。

### 3.1 MVC 与路由

| 层 | 常见类 | 审计关心什么 |
|---|---|---|
| C 控制器 | `@Controller` / `@RestController` | **路由来源**：类上 `@RequestMapping` + 方法上 `@GetMapping/@PostMapping/@PutMapping/@DeleteMapping` 拼接 |
| S 服务 | `@Service` | 数据流中间段，看参数是否被清洗 |
| M 模型/DAO | `@Repository` / Mapper 接口 | sink 所在：SQL、文件、命令 |
| 视图 | `templates/*.html`（Thymeleaf）、`static/*` | 模板注入、静态资源直读 |

**路由提取要点**：

- 类级路径与方法级路径**拼接**；`@RequestMapping` 不写 `method` = 全方法放行。
- 路径变量 `@PathVariable`、`@RequestParam`、`@RequestBody` 都是可控入参，区别只在绑定方式。
- `@RestController` = 所有方法加 `@ResponseBody`，注意别把返回值当"无风险"。
- 全局前缀 `server.servlet.context-path` 会让所有路由多一段，提路由时要带上。

详见 [[05-SpringBoot-Web路由与MVC]]。

### 3.2 拦截在哪

| 机制 | 配置位置 | 判什么 |
|---|---|---|
| Filter | `@WebFilter` / `FilterRegistrationBean` | `urlPatterns` 覆盖范围 |
| Interceptor | `WebMvcConfigurer#addInterceptors` 的 `addPathPatterns` / `excludePathPatterns` | **`exclude` 列表就是白名单**，逐个看里面有没有敏感接口 |
| Spring Security | `SecurityFilterChain` / `WebSecurityConfigurerAdapter` | `permitAll()` 的路径 |
| Shiro | `ShiroFilterFactoryBean` 的 `filterChainDefinitionMap` | `anon` 的路径；注意 `/**` 与具体路径的先后顺序 |

`excludePathPatterns` 与 `anon` 是最高频的越权来源——**先看白名单，再看逻辑**。详见 [[04-SpringBoot-配置与校验]]。

### 3.3 启动与扫描范围

- 启动类所在包是默认扫描根，`@ComponentScan` 可扩大/缩小。控制器在扫描包外 = 该路由不生效（但也可能被别的配置类扫进来）。
- 自动配置：`spring.factories`（Boot 2.x）/ `AutoConfiguration.imports`（3.x）决定默认加载哪些组件。看到不认识的端点，先查是不是某个 starter 带的。
- 静态资源与 Actuator：`/actuator/*`、`/druid/*`、`/swagger-ui.html` 常被漏在鉴权外。

详见 [[03-SpringBoot-自动配置与启动]]。

---

## 四、Maven（依赖判版本）

### 4.1 `pom.xml` 读法

| 节点 | 作用 | 审计用法 |
|---|---|---|
| `<parent>` | 继承父 POM（常是 `spring-boot-starter-parent`） | 版本被父管，子模块常不写版本号，别以为没依赖 |
| `<properties>` | 版本变量（`<fastjson.version>`） | 搜变量名才能拿到真实版本 |
| `<dependencies>` | 直接依赖 | 逐个对 1day |
| `<dependencyManagement>` | 只声明版本不引入 | 真实版本可能来自这里 |
| `<modules>` | 多模块列表 | 挨个进子目录看，别漏 |

**多模块的坑**：根 `pom.xml` 里没有业务依赖，全部在子模块。看到根 POM 干净就收工 = 漏审。

### 4.2 版本对 1day 的入口

先按组件搜，再看版本落点；**只对本地依赖里真实出现的版本做匹配，不建 CVE 清单**。

| 组件 | 搜索词 | 危险版本特征 |
|---|---|---|
| Fastjson | `fastjson` | 1.2.24 / 1.2.47 / ≤1.2.68 / ≤1.2.80（分界不同，判法见 Java 安全） |
| Shiro | `shiro-core` / `shiro-spring` | < 1.2.4 硬编码 key；1.2.4–1.7.x 仍可打 |
| Log4j2 | `log4j-core` | 2.0–2.14.1 |
| Commons-Collections | `commons-collections` | 3.x 有链，4.x 需特定 gadget |
| Spring | `spring-core` / `spring-beans` | 5.3.x < 5.3.18、< 5.2.20（Spring4Shell 相关） |
| XStream / Jackson / SnakeYAML | 对应包名 | 反序列化入口 |

> 版本命中了也**只算"待验证"**：能不能打还看入口是否可控、组件是否被调用。判前置交给 [[Security/Java安全/00-索引|Java 安全]]，本目录只负责"把它标红"。

### 4.3 只有 jar 没有源码

- `WEB-INF/lib` 或 `BOOT-INF/lib` 里逐个看文件名与版本。
- 反编译（CFR / Fernflower / jadx 视情况），先看 `META-INF/MANIFEST.MF` 与 `spring.factories` 认框架，再看 controller 包。
- 混淆过的先找字符串常量（路由、SQL 片段），比追类结构快。

---

## 五、一页分型表

| 看到什么 | 判定 | 下一步 |
|---|---|---|
| `WEB-INF/web.xml` + `lib` 一堆 jar | Servlet 项目 | 提 `url-pattern` 表，做 filter 差集 |
| `@SpringBootApplication` + `application.yml` | SpringBoot | 搜 `@RequestMapping` 家族，看 `excludePathPatterns` |
| 两者都有 | SSM 老项目迁 Boot | 路由两处都提，别只提注解 |
| 只有 `pom.xml` 和 `src/main/java`，无 web 目录 | 可能是 jar 包/定时任务/API 服务 | 找 `main` 方法与 `@Scheduled`、`MQ` 监听器 |
| 目录下多个 `pom.xml` | 多模块 | 从 parent 的 `modules` 逐个进 |
| 有 `*.jsp` 且能直接访问 | JSP 直连 | 列 jsp 文件树，重点看无鉴权页 |

---

## 六、这一步的产物

一张**项目画像**（可贴在入口建模卡 `E-xxx` 顶部）：

1. 形态：Servlet / SpringBoot / 混合
2. 路由来源与总数
3. 拦截机制与白名单路径
4. 依赖风险清单（组件 + 版本 + 是否被调用）
5. 未覆盖的入口（filter 差集 / `exclude` 差集）
6. 明显外露的管理面（actuator / druid / swagger / 上传目录）

**没有这张画像就开始跟 sink，等于闭眼走。**
