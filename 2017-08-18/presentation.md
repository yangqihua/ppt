<!--markpress-opt

{
	"layout": "horizontal",
	"autoSplit": true,
	"sanitize": false,
	"theme": "light",
	"noEmbed": false
}

markpress-opt-->
# 开发规范（JAVAEE篇）

## 1.基本规范
> 本规范基于 [阿里巴巴Java开发手册](https://yq.aliyun.com/articles/69327?spm=5176.100239.topwz.1.om5dRN)，所有开发规范都建立在该规范之上。

## 2.开发工具使用规范
- 统一使用 IntelliJ IDEA 作为 JavaEE 后台开发工具
- 统一使用 Maven 进行开发包管理，并使用阿里云国内镜像仓库
- 统一使用 maven.wagon 插件进行自动化部署
- 统一使用私有 git 服务进行代码版本控制
- 统一使用 teambiton 工具进行团队协作

## 3.框架使用规范
- 统一使用SpringBoot作为驱动框架，优点如下：
 + 零配置
 + 易部署
 + 易集成微服务
 + ...

- 统一使用 lombok 对代码进行简洁化处理
- 统一使用 spring-data-jpa 进行数据持久层开发
- 统一使用 apache commons-lang 或 google guava 工具包提高开发效率
- 统一使用 logback 进行日志记录

## 4.项目开发规范
- 每个项目必须有日志（开发环境日志和生产环境日志）
- 每个项目必须有统一异常处理（ Http 协议的状态码不要报500 ）
- 每个项目必须有安全性处理（常见的SQL注入、XSS攻击、CSRF、Dos攻击等等)

