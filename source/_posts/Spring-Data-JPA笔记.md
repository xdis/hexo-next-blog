---
title: Spring Data JPA笔记
copyright: true
date: 2017-12-13 21:31:32
tags: [SpringBoot, JPA]
categories: Spring与Dubbo分布式REST服务开发
---

Spring与Dubbo分布式REST服务开发之Spring-Data-JPA
<!--more-->

# Spring Data JPA简介

Spring Data JPA是Spring基于Hibernate开发的一个JPA框架。如果用过Hibernate或者MyBatis的话，就会知道对象关系映射（ORM）框架有多么方便。但是Spring Data JPA框架功能更进一步，为我们做了 一个数据持久层框架几乎能做的任何事情。



# SpringBoot中集成Spring Data JPA

## 在pom文件中加入相关依赖以及数据库驱动

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <scope>runtime</scope>
</dependency>
```

## 配置数据库连接

在*application.properties*中添加下述配置

```properties
# 配置数据库连接地址
spring.datasource.url=jdbc:mysql://localhost:3306/bookshop?useUnicode=yes&charaterEncoding=UTF-8&useSSL=false
# 配置连接驱动
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
# 配置数据库用户
spring.datasource.username=root
# 配置数据库密码
spring.datasource.password=root

# 设置JPA自动建表
spring.jpa.generate-ddl=true
```



# 对象映射-基本属性映射

```java
@Entity				
// 	表明该类是需要映射的对象。
//	当程序运行时会扫描带有该注解的类，检查数据库中是否已存该表，不存在则自动创建，如果字段增加也会自动增加字段，修改则不会进行操作
public class Category {
	
	@Id				// 设置为主键
	@GeneratedValue // 主键生成策略
	private int id;
	
	// 所有不同属性都会默认存在一个@Basic注解
	@Column(length=20, nullable=false, unique=true) //	列字段的配置
	private String name;
	
	@Transient		//加上此注解，JPA会忽略此字段的创建
	private String extra;
}
```

```java

/**
 * @author Jhacker
 *
 */
@Entity
public class Author {
	
	@Id
	@GeneratedValue
	private int id;
	
	private String name;
	
  	// columnDefinition 可以自定义DDL创建时字段的设置
	@Column(columnDefinition="INT(3)")
	private int age;
	
	@Temporal(TemporalType.DATE)	// 设置存入数据库的时间类型
	private Date birthday;
	
	@Enumerated(EnumType.STRING)	// 设置枚举存入至数据库的类型
	private Sex sex;
	
	@Embedded		// 嵌入内嵌对象
	private Address address;
}
```

```java
/**
 * @author Jhacker
 * 地址对象内嵌对象
 */
@Embeddable		// 声明对象为可内嵌对象
public class Address {
	
	// 省
	private String province;
	
	// 市
	private String city;
	
	// 地区
	private String area;
	
	// 地址
	private String address;
	
	// 邮政编码
	private String zipcode;	
}
```

当主对象和被嵌入对象的生命周期一致时才应该用内嵌对象。

内嵌对象的使用场景：当某一些字段在多个实体类中都可能会被用到，例如省、市、区等等地址相关的信息，可以单独抽出一张表进行维护，也可以使用内嵌对象，这样可以统一的维护所有包含内嵌对象Address的对象

*@ElementCollection* 映射集合对象时在该属性上加上此注解(只能映射基本类型和内嵌对象类型)，此时JPA会另外生成一张表进行维护，例如

```java
@ElementCollection
private List<String> hobbies;
// 此时会生成一张 auth_hoobies表，其中有字段 authod_id, hobby
// 在查询author对象时，jpa会自动查询该表，将结果映射到查询出的author对象上
```

***

# 对象映射-全局命名策略

在JPA创建表时，默认是按照属性和类名进行创建的，如果想要加上的前缀可以通过

```java
@Table(name="表名")
@Column(name="列名")
```

但是如果每个都像上述一样的设置就会变得很繁琐，因此可以设置统一的命名策略来进行前缀的统一添加

## 创建一个命名策略类

```java
package com.jiangyx.bookshop.support;

import org.hibernate.boot.model.naming.Identifier;
import org.hibernate.boot.model.naming.ImplicitNamingStrategyJpaCompliantImpl;
import org.hibernate.boot.spi.MetadataBuildingContext;

public class BookShopNamingStrategy extends ImplicitNamingStrategyJpaCompliantImpl {

	/**
	 * 
	 */
	private static final long serialVersionUID = 5656681986835349319L;
	
	@Override
	protected Identifier toIdentifier(String stringForm, MetadataBuildingContext buildingContext) {
		// 设置全局命名策略，所有的表名和列名都会加上前缀bs_
		return super.toIdentifier("bs_" + stringForm, buildingContext);
	}
}

```

## 在配置文件中配置

```properties
spring.jpa.hibernate.naming.implicit-strategy=com.jiangyx.bookshop.support.BookShopNamingStrategy
```

这样所有创建的类都会拥有统一的前缀了

# 对象映射-双向一对多关系映射

在项目中一对多和多对一关系是十分常见，而在JPA中使用@ManyToOne和@OneToMany注解来标明

## @ManyToOne

```java
/**
 * 
 */
package com.jiangyx.bookshop.domain;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.ManyToOne;

/**
 * @author Jhacker
 *
 */
@Entity
public class Book {
	
	@Id
	@GeneratedValue
	private long id;
	
	private String name;
	
	@ManyToOne
	private Category category;
}

```

@ManyToOne注解是来描述多对一关系的，使用此注解会在多的一方产生一个外键与一的一方进行关联

## @OneToMany

```java
package com.jiangyx.bookshop.domain;

import java.util.List;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.OneToMany;
import javax.persistence.Transient;

/**
 * @author Jhacker
 *
 */
@Entity
public class Category {
	
	@Id				// 设置为主键
	@GeneratedValue // 主键生成策略
	private int id;
	
	// 默认存在一个@Basic注解
	@Column(length=20, nullable=false, unique=true)
	private String name;
	
	@Transient		//加上此注解，JPA会忽略此字段的创建
	private String extra;
	
	@OneToMany
	private List<Book> books;
}

```

@OneToMany是来描述一对多关系的，JPA采用生成另一张表来维护一对多的关系。

## 双向一对多关系映射

在程序中双向关系是十分常见，例如上述例子，我们可以通过图书来获取图书的分类，同样也可以通过分类来获取分类下所有的图书，因此需要设置双向一对多关系映射

设置双向一对多关系映射只需要在一对多的地方进行额外设置

```java
package com.jiangyx.bookshop.domain;

import java.util.List;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.OneToMany;
import javax.persistence.Transient;

/**
 * @author Jhacker
 *
 */
@Entity
public class Category {
	
	@Id				// 设置为主键
	@GeneratedValue // 主键生成策略
	private int id;
	
	// 默认存在一个@Basic注解
	@Column(length=20, nullable=false, unique=true)
	private String name;
	
	@Transient		//加上此注解，JPA会忽略此字段的创建
	private String extra;
	
	@OneToMany(mappedBy="category")		// 放弃维护一对多关系(不建新表)，交给Book来维护关系(产生外检)
	private List<Book> books;
}

```

通过mappedBy来放弃维护一对多关系(不建新表)，交给多的一方来维护关系(产生外键)

***

# 对象映射-多对多

在项目中的多对多通常不使用@ManyToMany注解因为会没有中间对象。而是采用新建一张表，通过两个多对一关系进行设置

例如书和作者的关系，一本书可能属于多个作者，一个作者可能写了多本书

## 新建一个中间对象(中间表)

```java
/**
 * @author Jhacker
 *
 */
@Entity
public class BookAuthor {
	
	@Id
	@GeneratedValue
	private long id;
	
	@ManyToOne
	private Book book;
	
	@ManyToOne
	private Author author;
	
}

```

## 在作者处加入多对一

```java
@OneToMany(mappedBy="author")
@OrderBy("book.name ASC")
private List<BookAuthor> books;
```

## 在图书处加入多对一

```java
@OneToMany(mappedBy="book")
private List<BookAuthor> authors;
```

***

# 对象映射-一对一

在项目中可能出现一些字段不常用，但每次全部查询可能会耗费性能，因此这时可以采用一对一的方式

## 新建一个额外表

```java
/**
 * 
 */
package com.jiangyx.bookshop.domain;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.OneToOne;

/**
 * @author Jhacker
 *
 */
@Entity
public class AuthorInfo {
	
	@Id
	@GeneratedValue
	private long id;
	
	private String school;
	
	@OneToOne(mappedBy="authorInfo")
  	// 表示放弃管理关联，将管理权交给目标对象Author中的authorInfo属性
	private Author author;
}

```

## 在作者表中加入一对一

```java
@OneToOne
private AuthorInfo authorInfo;
```

***

# 对象映射-继承关系映射

有一些字段是公用的，而在每个类中加就会产生冗余，因此可以使用继承关系映射来解决

抽取公共字段

```java
/**
 * @author Jhacker
 *
 */
@MappedSuperclass
public class DomainImpl {
	@Id
	@GeneratedValue
	private long id;
	
	@Temporal(TemporalType.TIMESTAMP)
	private Date create_time;

	public long getId() {
		return id;
	}

	public void setId(long id) {
		this.id = id;
	}
}

```

然后每个类继承此类即可

***

# 小结

主要写了JPA的基本使用以及常用注解的使用

常用注解有

```java
@Table			表定义
@Entity			声明实体
@Id				主键
@GeneratedValue	主键生成策略
@Temporal		时间格式设置
@Column			列定义
@Transient		忽略此字段
@OntToMany		一对多关系(产生额外表)
@ManyToOne		多对一关系(产生外键)
@OneToOne		一对一关系
@Order			指明排序方式
@Embedded		映射内嵌对象
@Embeddable		声明内嵌对象
@Enumerated		映射枚举
@ElementCollection	映射和对象生命周期一致的集合
```

