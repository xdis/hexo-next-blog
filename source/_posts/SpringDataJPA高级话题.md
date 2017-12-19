---
title: SpringDataJPA高级话题
copyright: true
date: 2017-12-19 21:06:16
tags: [SpringBoot, JPA]
categories: Spring与Dubbo分布式REST服务开发
---

* 持久化上下文
* 抓取策略
* 继承策略
* 乐观锁
* 验证注解

<!--more-->

# 持久化上下文

* 持久化上下文的生命周期与系统事务一致
  * 在事务开始时会自动创建持久化上下文
  * 在事务提交时会自动销毁
* 持久化上下文提供自动脏检查
  * 在事务提交时，持久化上下文会检查是否和数据库中的数据不一致，如果不一致会根据持久化上下文中的状态更新
* 持久化上下文是一级缓存
  * 只有findOne才会有一级缓存



## 自动脏检查

```java
@Autowired
private PlatformTransactionManager transactionManager;

@Test
public void test1() {
  TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionDefinition());
  Book book = bookRepository.findOne(1L);
  book.setName("世纪");
  System.out.println("success");
  transactionManager.commit(status);
}
```

```sql
success
Hibernate: 
    update
        bs_book 
    set
        bs_create_time=?,
        bs_category_bs_id=?,
        bs_name=? 
    where
        bs_id=?
```

上述代码就讲述了持久化上下文的自动脏检查功能，在事务提交时，持久化上下文会检查是否和数据库中的数据不一致，如果不一致会根据持久化上下文中的状态更新。

```java

@Test
public void test2() {
  TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionDefinition());
  Book book = bookRepository.findOne(1L);
  book.setName("世纪1");
  // 表示立即同步，而不等到事务提交时
  bookRepository.saveAndFlush(book);
  System.out.println("success");
  transactionManager.commit(status);
}
```

```sql
Hibernate: 
    update
        bs_book 
    set
        bs_create_time=?,
        bs_category_bs_id=?,
        bs_name=? 
    where
        bs_id=?
success
```

也可以使用**saveAndFlush** 进行立刻同步而不等到事务提交时，但是此时如果事务进行回滚同步也会失效，应为此时并没有事务提交



## 一级缓存

```java
@Test
public void test3() {
  bookRepository.findOne(1L);
  bookRepository.findOne(1L);
}
```

```sql
Hibernate: 
    select
        book0_.bs_id as bs_id1_2_0_,
        book0_.bs_create_time as bs_creat2_2_0_,
        book0_.bs_category_bs_id as bs_categ4_2_0_,
        book0_.bs_name as bs_name3_2_0_,
        category1_.bs_id as bs_id1_4_1_,
        category1_.bs_create_time as bs_creat2_4_1_,
        category1_.bs_name as bs_name3_4_1_ 
    from
        bs_book book0_ 
    left outer join
        bs_category category1_ 
            on book0_.bs_category_bs_id=category1_.bs_id 
    where
        book0_.bs_id=?
```

执行两次findOne其实只生成1条SQL，持久化上下文会根据ID进行缓存



***



# 抓取策略

```java
@Test
public void test4() {
  Book book = bookRepository.findOne(1L);
  System.out.println(book.getCategory().getName());
}
```

```sql
Hibernate: 
    select
        book0_.bs_id as bs_id1_2_0_,
        book0_.bs_create_time as bs_creat2_2_0_,
        book0_.bs_category_bs_id as bs_categ4_2_0_,
        book0_.bs_name as bs_name3_2_0_,
        category1_.bs_id as bs_id1_4_1_,
        category1_.bs_create_time as bs_creat2_4_1_,
        category1_.bs_name as bs_name3_4_1_ 
    from
        bs_book book0_ 
    left outer join
        bs_category category1_ 
            on book0_.bs_category_bs_id=category1_.bs_id 
    where
        book0_.bs_id=?
爱情
```

在使用id查找书籍时会默认将图书分类一起查出来，应为category默认的抓取策略是fetch，因此只需要执行一个SQL即可

```java
public interface BookRepository extends JpaRepository<Book, Long>, JpaSpecificationExecutor<Book> {
	
	Book findByName(String name);
	
	@Query("from Book b where b.name = ?1")
	List<Book> findBook(String name);
}
```

```java
@Test
public void test5() {
  Book book = bookRepository.findByName("世纪1");
  System.out.println(book.getCategory().getName());
}
```

```sql
Hibernate: 
    select
        book0_.bs_id as bs_id1_2_,
        book0_.bs_create_time as bs_creat2_2_,
        book0_.bs_category_bs_id as bs_categ4_2_,
        book0_.bs_name as bs_name3_2_ 
    from
        bs_book book0_ 
    where
        book0_.bs_name=?
Hibernate: 
    select
        category0_.bs_id as bs_id1_4_0_,
        category0_.bs_create_time as bs_creat2_4_0_,
        category0_.bs_name as bs_name3_4_0_ 
    from
        bs_category category0_ 
    where
        category0_.bs_id=?
爱情
```

这一部分的代码使用动态查询，但是SpringDataJPA没有进行关联查询，而且数据量一大，上面的代码效率就会很低，因此需要进行手动设置抓取策略

```java
/**
 * @author Jhacker
 *
 */
public interface BookRepository extends JpaRepository<Book, Long>, JpaSpecificationExecutor<Book> {
	
	// 表示在执行此方法时，将book中的category属性一并查出来
	@EntityGraph(attributePaths = {"category"})
	Book findByName(String name);
	
	@Query("from Book b where b.name = ?1")
	List<Book> findBook(String name);
}
```

此时在执行上述的测试用例，就能看到和findOne一样的效果



## 设置统一的抓取策略

像上述代码所示，我们可以给每个方法进行抓取策略的设置，但是如果有十几二十几个方法都用相同的策略，以后如果需要修改或删除会十分麻烦，因此可以在实体类上设置统一的抓去策略

```java
@Entity
@NamedEntityGraph(name="Book.fetch.category",
		attributeNodes = {
				@NamedAttributeNode("category")
		})
public class Book extends DomainImpl {
}
```

```java
// 指明需要的策略名
@EntityGraph(value="Book.fetch.category")
Book findByName(String name);
```



***



# 继承策略

## SINGLE_TABLE

```java
@Entity
@Inheritance(strategy=InheritanceType.SINGLE_TABLE)
class Book {
  private long id;
  private String name;
}

@Entity
class PrintBook extends Book {
  private Date printDate;
}

@Entity
class EBook extends Book {
  private Date publishDate;
}
```

此策略是默认策略，他并不会生成另外两张表，只会将两个子类中额外的属性添加到Book表中，此外再加上一个type属性进行区分

## JOINED

```java
@Entity
@Inheritance(strategy=InheritanceType.)
class Book {
  private long id;
  private String name;
}

@Entity
class PrintBook extends Book {
  private Date printDate;
}

@Entity
class EBook extends Book {
  private Date publishDate;
}
```

在此策略下，会生成额外的两张表，每张表有各个子类的属性和外键，做插入时会在Book表中插入一条，然后在子类表中插入，查询时会自动进行外连接

## TABLE_PRE_CLASS

> 使用较少



***

# 乐观锁

在SpringDataJPA中使用乐观锁比较简单只需要在实体类中新增

```java
@Version
private int version;
```



***



# Hibernate Valodator

SpringDataJPA提供了很多种验证的注解，这种验证注解不会影响到数据表的生成，只会在数据写入数据库之前进行验证。

| 注解                              | 描述                                       |
| ------------------------------- | ---------------------------------------- |
| @AssertFalse                    | 值必须为False                                |
| @AssertTrue                     | 值必须为True                                 |
| @DecimalMax(value=, inclusive=) | 值必须小于等于(inclusive=true)/小于(inclusive=false)value属性指定的值，可以注解在字符串类型的属性上 |
| @DecimalMin(value=, inclusive=) | 值必须大于等于(inclusive=true)/大于(inclusive=false)value属性指定的值，可以注解在字符串类型的属性上 |
| @Digits(integer=,fraction=)     | 数字格式检查，integer指定整数部分的最大长度，fraction指定小数部分的最大长度 |
| @Future                         | 值必须是未来的日期                                |
| @Past                           | 值必须是过去的日期                                |
| @Max(value=)                    | 值必须小于等于value的值。不能注释在字符串类型的属性上            |
| @Min(value=)                    | 值必须大于等于value的值。不能注释在字符串类型的属性上            |
| @NotNull                        | 值不能为空                                    |
| @Null                           | 值必须为空                                    |
| @Pattern(regex)                 | 字符串必须匹配正则表达式                             |
| @Size(min=, max=)               | 集合的元素数量必须在min和max之间                      |
| @Email                          | 字符串必须是Email地址                            |
| @Lenght(min=, max=)             | 检查字符串长度                                  |
| @NotBlank                       | 字符串必须有字符                                 |
| @NotEmpty                       | 字符串不为null，集合有元素                          |
| @Range(min=, max=)              | 数字必须大于等于min, 小于等于max                     |
| @SafeHtml                       | 字符串为安全的html                              |
| @URL                            | 字符串是合法的URL                               |