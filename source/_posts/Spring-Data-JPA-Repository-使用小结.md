---
title: Spring-Data-JPA-Repository-使用小结
copyright: true
date: 2017-12-15 21:31:32
tags: [SpringBoot, JPA]
categories: Spring与Dubbo分布式REST服务开发
---
---
* Repository之基本增删改查
* Repository-分页和排序
* Repository-静态查询
* Repository-动态查询
* Repository-自定义Repository

<!--more-->

# Repository之基本增删改查

SpringDataJPA提供了Repository接口来提供ORM操作，例如其中的CrudRepository就提供了基本的增删改查的方法

## 修改配置文件

```properties
# 打印SQL
spring.jpa.show-sql=true
# 格式化打印的SQL
spring.jpa.properties.hibernate.format_sql=true
```

## 创建接口继承CrudRepository

```java
/**
 * @author Jhacker
 *
 */
public interface BookRepository extends CrudRepository<Book, Long> {
	
	List<Book> findByName(String name);
}

```

## 进行测试

### 新增

```java
@Autowired
private BookRepository bookRepository;

@Test
public void testInsert() {
  Book book = new Book();
  book.setName("简爱");
  bookRepository.save(book);
}
```

```shell
Hibernate: 
    insert 
    into
        bs_book
        (bs_create_time, bs_category_bs_id, bs_name) 
    values
        (?, ?, ?)
```

### 修改

```java
/**
* 测试更新
*/
@Test
public void testUpdate() {
  Book book = bookRepository.findOne(1L);
  book.setName("战争与和平");
  bookRepository.save(book);
}
```

如果实体存在ID，JPA会认为是更新，在更新前，JPA会先查询一次，如果对象存在则会更新，否则会进行新增

### 删除

```java
@Test
public void testDelete() {
  bookRepository.delete(1L);
}
```

在删除前，JPA会进行查询

### 查找

```java
@Test
public void testFindALL2() {
  List<Long> ids = new ArrayList<>();
  ids.add(1L);
  ids.add(2L);
  ids.add(3L);
  bookRepository.findAll(ids);
}

@Test
public void testFindALL() {
  bookRepository.findAll();
}

@Test
public void testFindOne() {
  bookRepository.findOne(1L);
}
```

***

# Repository-分页和排序

CrudRepository只能支持最简单的增删改查， 如果需要进行分页则需要将CrudRepository换成PagingAndSortingRepository，PagingAndSortingRepository是继承CrudRepository的，对CrudRepository进行了加强

## 排序

单个排序

```java
@Test
public void testSort() {
  bookRepository.findAll(new Sort(Direction.DESC, "name"));
}
```

```sql
    select
        book0_.bs_id as bs_id1_2_,
        book0_.bs_create_time as bs_creat2_2_,
        book0_.bs_category_bs_id as bs_categ4_2_,
        book0_.bs_name as bs_name3_2_ 
    from
        bs_book book0_ 
    order by
        book0_.bs_name desc
```

多个排序

```java
@Test
public void testSortMulty() {
  bookRepository.findAll(new Sort(new Order(Direction.DESC, "name"), 
                                  new Order(Direction.DESC, "id")));
}
```

```sql
select
        book0_.bs_id as bs_id1_2_,
        book0_.bs_create_time as bs_creat2_2_,
        book0_.bs_category_bs_id as bs_categ4_2_,
        book0_.bs_name as bs_name3_2_ 
    from
        bs_book book0_ 
    order by
        book0_.bs_name desc,
        book0_.bs_id desc
```

## 分页

```java
@Test
public void testPage() {
  Pageable pageable = new PageRequest(1, 10);
  bookRepository.findAll(pageable);
}
```

```sql
 select
        book0_.bs_id as bs_id1_2_,
        book0_.bs_create_time as bs_creat2_2_,
        book0_.bs_category_bs_id as bs_categ4_2_,
        book0_.bs_name as bs_name3_2_ 
    from
        bs_book book0_ limit ?,
        ?
```

```java
@Test
public void testPageAndSort() {
  Pageable pageable = new PageRequest(1, 10, new Sort(Direction.DESC, "name"));
  bookRepository.findAll(pageable);
}
```

```sql
    select
        book0_.bs_id as bs_id1_2_,
        book0_.bs_create_time as bs_creat2_2_,
        book0_.bs_category_bs_id as bs_categ4_2_,
        book0_.bs_name as bs_name3_2_ 
    from
        bs_book book0_ 
    order by
        book0_.bs_name desc limit ?,
        ?
```

## 带条件的分页

PagingAndSortingRepository只能支持最简单的分页操作，无法再分页时进行条件筛选，因此需要继承JpaRepository，JpaRepository继承PagingAndSortingRepository

```java
@Test
public void testPageWithExample() {
  Pageable pageable = new PageRequest(1, 10, new Sort(Direction.DESC, "name"));
  Book book = new Book();
  book.setName("简爱");
  Example<Book> example = Example.of(book);
  bookRepository.findAll(pageable);
}
```

```sql
 select
        book0_.bs_id as bs_id1_2_,
        book0_.bs_create_time as bs_creat2_2_,
        book0_.bs_category_bs_id as bs_categ4_2_,
        book0_.bs_name as bs_name3_2_ 
    from
        bs_book book0_ 
    where
        book0_.bs_name=? 
        and book0_.bs_id=0 
    order by
        book0_.bs_name desc limit ?,
        ?
```

```java
@Test
public void testPageWithExampleMatching() {
  Pageable pageable = new PageRequest(1, 10, new Sort(Direction.DESC, "name"));
  Book book = new Book();
  book.setName("简");

  ExampleMatcher matcher = ExampleMatcher.matching().withStringMatcher(StringMatcher.CONTAINING);
  Example<Book> example = Example.of(book, matcher);
  bookRepository.findAll(example, pageable);
}
```

```sql
  select
        book0_.bs_id as bs_id1_2_,
        book0_.bs_create_time as bs_creat2_2_,
        book0_.bs_category_bs_id as bs_categ4_2_,
        book0_.bs_name as bs_name3_2_ 
    from
        bs_book book0_ 
    where
        (
            book0_.bs_name like ?
        ) 
        and book0_.bs_id=0 
    order by
        book0_.bs_name desc limit ?,
        ?
```

***

# Repository-静态查询

Repository可以通过类似于findByName方式来查找条件为name时候的数据。操作如下

```java
/**
 * @author Jhacker
 *
 */
public interface BookRepository extends JpaRepository<Book, Long> {
	
	List<Book> findByName(String name);
}
```

```java
@Test
public void testFindByName() {
  bookRepository.findByName("简爱");
}
```

```sql
 select
        book0_.bs_id as bs_id1_2_,
        book0_.bs_create_time as bs_creat2_2_,
        book0_.bs_category_bs_id as bs_categ4_2_,
        book0_.bs_name as bs_name3_2_ 
    from
        bs_book book0_ 
    where
        book0_.bs_name=?
```

还可以通过@Query注解来进行查询

```java
/**
 * @author Jhacker
 *
 */
public interface BookRepository extends JpaRepository<Book, Long> {
	
	List<Book> findByName(String name);
	
	@Query("from Book b where b.name = ?1")
	List<Book> findBook(String name);
}
```

```java
@Test
public void testFindBook() {
  bookRepository.findBook("简爱");
}
```

```sql
    select
        book0_.bs_id as bs_id1_2_,
        book0_.bs_create_time as bs_creat2_2_,
        book0_.bs_category_bs_id as bs_categ4_2_,
        book0_.bs_name as bs_name3_2_ 
    from
        bs_book book0_ 
    where
        book0_.bs_name=?
```

@Query可以进行任何操作，如果需要执行增删改操作则需要在方法上加上@Modifying表示进行表修改

***

# Repository-动态查询

一般的JpaRepository无法实现动态查询，因此SpringDataJPA提供了JpaSpecificationExecutor接口，只需要继承就能拥有动态查询的功能

```java
public interface BookRepository extends JpaRepository<Book, Long>, JpaSpecificationExecutor<Book> {
	
	List<Book> findByName(String name);
	
	@Query("from Book b where b.name = ?1")
	List<Book> findBook(String name);
}
```

```java
// 测试多条件查询
@Test 
public void testSpecification() {
  Specification<Book> spec = new Specification<Book>() {

    @Override
    public Predicate toPredicate(Root<Book> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
      // TODO Auto-generated method stub
      Predicate p1 = cb.equal(root.get("name"), "简爱");
      Predicate p2 = cb.equal(root.get("category").get("name"), "战争");
      Predicate p3 = cb.and(p1, p2);
      return p3;
    }
  };
  bookRepository.findAll(spec);
}
```

```sql
  select
        book0_.bs_id as bs_id1_2_,
        book0_.bs_create_time as bs_creat2_2_,
        book0_.bs_category_bs_id as bs_categ4_2_,
        book0_.bs_name as bs_name3_2_ 
    from
        bs_book book0_ cross 
    join
        bs_category category1_ 
    where
        book0_.bs_category_bs_id=category1_.bs_id 
        and book0_.bs_name=? 
        and category1_.bs_name=?
```

***

# Repository-自定义Repository实现

如果我们需要在每次新增操作前打印日志，那么默认的Repository就无法实现此功能，因此需要自定义Repository

新建一个自定义Repository

```java
package com.jiangyx.bookshop.support;

import javax.persistence.EntityManager;

import org.springframework.data.jpa.repository.support.JpaEntityInformation;
import org.springframework.data.jpa.repository.support.SimpleJpaRepository;

public class BookShopRepositoryImpl<T> extends SimpleJpaRepository<T, Long> {

	public BookShopRepositoryImpl(JpaEntityInformation<T, ?> entityInformation, EntityManager entityManager) {
		super(entityInformation, entityManager);
	}
	
	@Override
	public <S extends T> S save(S entity) {
		System.out.println("打印日志");
		return super.save(entity);
	}

}

```

在启动文件上添加注解

```java
package com.jiangyx.bookshop;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;

import com.jiangyx.bookshop.support.BookShopRepositoryImpl;

@SpringBootApplication
@EnableJpaRepositories(repositoryBaseClass=BookShopRepositoryImpl.class)
public class BookshopApplication {

	public static void main(String[] args) {
		SpringApplication.run(BookshopApplication.class, args);
	}
}

```

进行测试

```java
/**
* 测试新增
*/
@Test
public void testInsert() {
  Book book = new Book();
  book.setName("简爱");
  bookRepository.save(book);
}

```

```sql
打印日志
Hibernate: 
    insert 
    into
        bs_book
        (bs_create_time, bs_category_bs_id, bs_name) 
    values
        (?, ?, ?)
```

