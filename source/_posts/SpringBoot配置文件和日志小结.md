---
title: SpringBoot配置文件和日志小结
copyright: true
date: 2017-12-15 22:55:00
tags: [SpringBoot]
categories: SpringBoot入门
---

学习SpringBoot配置文件和日志文件

<!--more-->

# 搭建Spring Tool Suit环境

## 直接下载官方提供的集成环境

在[官网](https://spring.io/tools/sts/all/)下载对应系统版本的Spring Tool Suite

## 通过Eclipse插件的方式进行安装

1、下载工具

```shell
Eclipse：http://www.eclipse.org/downloads/packages/eclipse-ide-java-ee-developers/neonr
Spring Tool Suite：https://spring.io/tools/sts/all
```

2.使用版本为

```shell
Eclipse：eclipse-jee-neon-R-win32-x86_64.zip
Spring Tool Suite：springsource-tool-suite-3.8.0.RELEASE-e4.5.2-updatesite.zip
```

3.安装步骤

* 解压Eclipse，打开Help ---> Install New Sofware...
* 点击右上角位置的：Add...
* 点击右上角位置的：Loccal...，选中解压之后的springsource-tool-suite-3.8.0.RELEASE-e4.5.2-updatesite，，之后点击 OK
* 点击 Select All，之后确认，后面有多次需要确认，会花费一定的时间，请耐心等待，安装完成之后，会提示重启。



> 可以选中在线安装Spring Tool Suite 这个插件。Help ---> Eclipse Marketplce...之后搜索spring tool suite

***

# RESTfull API简单项目的快速搭建

## 加入对应的依赖

```xml
<!-- web依赖 -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!-- 单元测试依赖 -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-test</artifactId>
  <scope>test</scope>
</dependency>

<!-- 开发工具(项目自动重启) -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-devtools</artifactId>
</dependency>
```

## 新建实体类

```java
/**
 * 
 */
package com.jiangyx.demo.domain;

import java.util.Date;

/**
 * User实体类
 * @author jiangyx
 *
 */
public class User {
	
	private int id;
	private String name;
	private Date date;
	public int getId() {
		return id;
	}
	public void setId(int id) {
		this.id = id;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public Date getDate() {
		return date;
	}
	public void setDate(Date date) {
		this.date = date;
	}
}

```

## 新建控制器类

```shell
package com.jiangyx.demo.controller;

import java.util.Date;
import java.util.HashMap;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import com.jiangyx.demo.domain.User;

/**
 * @author jiangyx
 *
 */
@RestController
@RequestMapping(value="index")
public class IndexController {
	
	@GetMapping
	public String index() {
		return "Hello Spring";
	}
	
	@GetMapping(value="get")
	public HashMap<String, Object> get(@RequestParam String name) {
		HashMap<String, Object> map = new HashMap<>();
		map.put("title", "hello world");
		map.put("name", name);
		return map;
	}
	
	@GetMapping(value="/get/{id}/{name}")
	public User getUser(@PathVariable int id, @PathVariable String name) {
		User user = new User();
		user.setId(id);
		user.setName(name);
		user.setDate(new Date());
		return user;
	}
	
}
```

## 运行项目并测试

直接运行main方法或者使用maven命令: spring-boot:run 

```shell
测试： http://localhost:8080/index
带参数：http://localhost:8080/index/get?name=jiangyx
带参数有中文：http://localhost:8080/index/get?name=蒋蜀黍
url测试：http://localhost:8080/index/get/1/jiangyx
url测试：http://localhost:8080/index/get/1/蒋蜀黍
```

## 打包项目

运行maven命令

```shell
clean package
```

在项目下的target文件夹中可以找到项目的jar包，运行命令启动项目

```shell
java -jar xxx.jar
```

***

# 配置文件详解：Properties和YAML

## Spring Boot的配置文件生效顺序会对值进行覆盖

1. @TestPropertySource 注解
2. 命令行参数
3. Java系统属性（System.getProperties()）
4. 操作系统环境变量
5. 只有在random.*里包含的属性会产生一个RandomValuePropertySource
6. 在打包的jar外的应用程序配置文件（application.properties，包含YAML和profile变量）
7. 在打包的jar内的应用程序配置文件（application.properties，包含YAML和profile变量）
8. 在@Configuration类上的@PropertySource注解
9. 默认属性（使用SpringApplication.setDefaultProperties指定）

## 配置随机值

```properties
bookshop.secret=${random.value}
bookshop.number=${random.int}
bookshop.bignumber=${random.long}
bookshop.number.less.than.ten=${random.int(10)}
bookshop.number.in.range=${random.int[1024,65536]}
```

在项目中可以读取设置的随机值

```java
@Value(value = "${bookshop.secret}")
```

```java
/**
 * 
 */
package com.jiangyx.demo.controller;

import java.util.HashMap;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author mickle.jiang
 *
 */
@RestController
public class IndexController {
	
	@Value(value="${bookshop.secret}")
	private String secret;
	
	@Value(value="${bookshop.host}")
	private String host;
	
	@Value(value="${bookshop.desc}")
	private String desc;
	
	@Value(value="${bookshop.number.in.range}")
	private int number;
	
	@GetMapping
	public HashMap<String, Object> index() {
		HashMap<String, Object> map = new HashMap<>();
		map.put("secret", secret);
		map.put("host", host);
		map.put("desc", desc);
		map.put("number", number);
		return map;
	}
}
```

得到结果

```json
{
	number: 65032,
	host: "www.bookshop.com",
	secret: "409aed7221ecaf7b73e8fd68024577f3",
	desc: "www.bookshop.com is a domain name"
}
```



> 注：出现黄点提示，是要提示配置元数据，可以不配置

## 属性占位符

当application.properties里的值被使用时，它们会被存在的Environment过滤，所以你能够引用先前定义的值（比如，系统属性）。

```properties
bookshop.host=www.bookshop.com
bookshop.desc=${bookshop.host} is a domain name
```

## Application属性文件，按优先级排序，位置高的将覆盖位置低的

1. 当前目录下的一个/config子目录
2. 当前目录
3. 一个classpath下的/config包
4. classpath根路径（root）



> 这个列表是按优先级排序的（列表中位置高的将覆盖位置低的）

## 配置应用端口和其他配置的介绍

```properties
#端口配置：
server.port=8090
#时间格式化
spring.jackson.date-format=yyyy-MM-dd HH:mm:ss
#时区设置
spring.jackson.time-zone=Asia/Shanghai
```

## 可以使用yaml替代properties

```yaml
server:
	port: 8090
```

***

# 配置文件-多环境配置

## 多环境配置的好处

1. 不同环境配置可以配置不同的参数
2. 便于部署，提高效率，减少出错

## Properties多环境配置

添加其他配置文件

```shell
# 开发环境
application-dev.properties
# 测试环境
application-test.properties
# 生产环境
application-pro.properties
# 默认
application.properties
```

之后在application.properties中进行激活

```properties
spring.profiles.active=dev
```

## YAML多环境配置

YAML使用---来区分不同环境

```yaml
spring:
	profiles:
		active: dev
---
spring:
	profiles: dev
---
spring:
	profiles: test
---
spring:
	profiles: pro
```

## 两种配置方式的比较

1. Properties配置多环境，需要添加多个配置文件，YAML只需要一个配件文件
2. 书写格式的差异，yaml相对比较简洁，优雅
3. YAML的缺点：不能通过@PropertySource注解加载。如果需要使用@PropertySource注解的方式加载值，那就要使用properties文件。

## 通过参数来选择环境

```shell
java -jar myapp.jar --spring.profiles.active=dev
```

***

# 日志配置-logback和log4j2

Spring Boot 支持Java Util Logging, Log4J2 and Logback等日志框架，默认使用的是logback

> 日志配置存在有两种配置方式：默认配置文件配置和引用外部配置文件配置

## 默认配置文件配置

> 默认配置文件配置通常不建议使用，因为不够灵活，对log4j2等不够友好)

```properties
# 日志文件名，比如：bookshop.log，或者是 /var/log/bookshop.log
logging.file=bookshop.log 
# 日志级别配置，比如： logging.level.org.springframework=DEBUG
logging.level.*=info
logging.level.org.springframework=DEBUG
```

## 引用外部配置文件

### logback配置方式

> spring boot默认会加载classpath:logback-spring.xml或者classpath:logback-spring.groovy

使用自定义配置文件，配置方式为

```properties
logging.config=classpath:logback-bookshop.xml
# 注意：不要使用logback这个来命名，否则spring boot将不能完全实例化
```

配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>

	<!-- 文件输出格式 -->
	<property name="PATTERN" value="%-12(%d{yyyy-MM-dd HH:mm:ss.SSS}) |-%-5level [%thread] %c [%L] -| %msg%n" />
	<!-- test文件路径 -->
	<property name="TEST_FILE_PATH" value="c:/opt/roncoo/logs" />
	<!-- pro文件路径 -->
	<property name="PRO_FILE_PATH" value="/opt/roncoo/logs" />

	<!-- 开发环境 -->
	<springProfile name="dev">
		<appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
			<encoder>
				<pattern>${PATTERN}</pattern>
			</encoder>
		</appender>
		
		<logger name="com.jiangyx.demo" level="debug"/>

		<root level="info">
			<appender-ref ref="CONSOLE" />
		</root>
	</springProfile>

	<!-- 测试环境 -->
	<springProfile name="test">
		<!-- 每天产生一个文件 -->
		<appender name="TEST-FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
			<!-- 文件路径 -->
			<file>${TEST_FILE_PATH}</file>
			<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
				<!-- 文件名称 -->
				<fileNamePattern>${TEST_FILE_PATH}/info.%d{yyyy-MM-dd}.log</fileNamePattern>
				<!-- 文件最大保存历史数量 -->
				<MaxHistory>100</MaxHistory>
			</rollingPolicy>
			
			<layout class="ch.qos.logback.classic.PatternLayout">
				<pattern>${PATTERN}</pattern>
			</layout>
		</appender>
		
		<root level="info">
			<appender-ref ref="TEST-FILE" />
		</root>
	</springProfile>

	<!-- 生产环境 -->
	<springProfile name="prod">
		<appender name="PROD_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
			<file>${PRO_FILE_PATH}</file>
			<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
				<fileNamePattern>${PRO_FILE_PATH}/warn.%d{yyyy-MM-dd}.log</fileNamePattern>
				<MaxHistory>100</MaxHistory>
			</rollingPolicy>
			<layout class="ch.qos.logback.classic.PatternLayout">
				<pattern>${PATTERN}</pattern>
			</layout>
		</appender>
		
		<root level="warn">
			<appender-ref ref="PROD_FILE" />
		</root>
	</springProfile>
</configuration>

```

### log4j配置

1、去除logback的依赖包，添加log4j2的依赖包

```xml
<exclusions>
  <exclusion>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-logging</artifactId>
  </exclusion>
</exclusions>

<!-- 使用log4j2 -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
```

2、在classpath添加log4j2.xml或者log4j2-spring.xml（spring boot 默认加载）

```properties
<?xml version="1.0" encoding="utf-8"?>
<configuration>
	<properties>
		<!-- 文件输出格式 -->
		<property name="PATTERN">%d{yyyy-MM-dd HH:mm:ss.SSS} |-%-5level [%thread] %c [%L] -| %msg%n</property>
	</properties>

	<appenders>
		<Console name="CONSOLE" target="system_out">
			<PatternLayout pattern="${PATTERN}" />
		</Console>
	</appenders>
	
	<loggers>
		<logger name="com.jiangyx.demo" level="debug" />
		<root level="info">
			<appenderref ref="CONSOLE" />
		</root>
	</loggers>

</configuration>
```

可以给不同环境配置不同的日志文件。只要在环境配置中加上

```properties
# 应用自定义配置
logging.config=classpath:log4j2-dev.xml
```

## Log4J2和Logback 比较

性能比较：Log4J2 和 Logback 都优于 log4j（不推荐使用）

配置方式：Logback最简洁，spring boot默认，推荐使用

