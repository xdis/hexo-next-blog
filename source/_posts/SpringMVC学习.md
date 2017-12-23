---
title: SpringMVC学习
copyright: true
date: 2017-12-23 22:23:34
tags: [SpringBoot, SpringMVC]
categories: Spring与Dubbo分布式REST服务开发
---

常用的SpringMVC的使用方式

<!--more-->

# Mock测试的基本写法

```java
public class BookControllerTest extends BookshopAdminApplicationTests {
	
	@Autowired
	private WebApplicationContext wac;
	
	private MockMvc mockMvc;
	
	@Before
	public void setup() {
		// 进行初始化，将上下文传入
		mockMvc = MockMvcBuilders.webAppContextSetup(wac).build();
	}
	
	@Test
	public void testWhenQuerySuccess() throws Exception {
		mockMvc.perform(MockMvcRequestBuilders.get("/book")	// 请求网址
				.param("name", "战争与和平")					// 请求参数
				.param("categoryId", "2")					
				.accept(MediaType.APPLICATION_JSON_UTF8))	// 验证返回类型
				.andExpect(MockMvcResultMatchers.status().isOk())	// 验证返回状态
				.andExpect(MockMvcResultMatchers.jsonPath("$.length()").value(3));	// 验证返回的JSON数据
		
	}
}
```

***

# SpringMVC中简单的参数映射

```java
@RestController
@RequestMapping(value="/book")
public class BookController {

//	可以通过指定参数来查询
//	@GetMapping
//	public List<BookInfo> query(@RequestParam(name="name") String bookName) {
//		System.out.println(bookName);
//		List<BookInfo> books = new ArrayList<BookInfo>();
//		books.add(new BookInfo());
//		books.add(new BookInfo());
//		books.add(new BookInfo());
//		return books;
//	}
	
	// 通过将参数封装成一个类来查
	@GetMapping
	public List<BookInfo> query(BookCondition condition) {
		System.out.println(condition.getName());
		System.out.println(condition.getCategoryId());
		List<BookInfo> books = new ArrayList<BookInfo>();
		books.add(new BookInfo());
		books.add(new BookInfo());
		books.add(new BookInfo());
		return books;
	}

}

```

***

# SpringMVC中使用分页

首先在pom文件中加上相关依赖

```xml
<dependency>
  <groupId>org.springframework.data</groupId>
  <artifactId>spring-data-commons</artifactId>
</dependency>
```

然后在控制器中的方法上就可以使用

```java
// 通过将参数封装成一个类来查
@GetMapping
public List<BookInfo> query(BookCondition condition, @PageableDefault(size=10, page=1) Pageable pageable) {
  // 使用@PageableDefault(size=10, page=1)可以设置一些分页的默认参数
  // 获取当前页
  System.out.println(pageable.getPageNumber());
  // 获取每页的条数
  System.out.println(pageable.getPageSize());
  // 获取排序规则
  System.out.println(pageable.getSort());
  System.out.println(condition.getName());
  System.out.println(condition.getCategoryId());
  List<BookInfo> books = new ArrayList<BookInfo>();
  books.add(new BookInfo());
  books.add(new BookInfo());
  books.add(new BookInfo());
  return books;
}
```

在测试代码里如下

```java
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;
**
	 * 测试查询列表成功
	 * @throws Exception
	 */
	@Test
	public void testWhenQuerySuccess() throws Exception {
		mockMvc.perform(get("/book")
				.param("name", "战争与和平")
				.param("categoryId", "2")
				.param("page", "2")
				.param("size", "15")
				.param("sort", "name,desc", "id,asc")
				.accept(MediaType.APPLICATION_JSON_UTF8))
				.andExpect(status().isOk())
				.andExpect(jsonPath("$.length()").value(3));
	}
```

# SpringMVC中路由参数绑定

```java
/**
* 测试查询详情成功
* @throws Exception
*/
@Test
public void testWhenGetInfoSuccess() throws Exception {
  mockMvc.perform(get("/book/1")
                  .accept(MediaType.APPLICATION_JSON_UTF8))
    .andExpect(status().isOk())
    .andExpect(jsonPath("$.name").value("战争与和平"));
}
```

```java
@GetMapping("/{id}")
public BookInfo queryInfo(@PathVariable int id) {
  // PathVariable可以用来获取路由中的参数
  System.out.println(id);
  BookInfo bookInfo = new BookInfo();
  bookInfo.setId(1L);
  bookInfo.setName("战争与和平");
  return bookInfo;
}

```

路由参数不仅可以为{id}这样，还可以使用正则

```java
@GetMapping("/{id:\\d}")
public BookInfo queryInfo(@PathVariable int id) {
  // PathVariable可以用来获取路由中的参数
  System.out.println(id);
  BookInfo bookInfo = new BookInfo();
  bookInfo.setId(1L);
  bookInfo.setName("战争与和平");
  return bookInfo;
}

```

```java
@Test
public void testWhenGetInfoSuccess() throws Exception {
  mockMvc.perform(get("/book/10")
                  .accept(MediaType.APPLICATION_JSON_UTF8))
    .andExpect(status().isOk())
    .andExpect(jsonPath("$.name").value("战争与和平"));
}
```

上述结果测试会不通过，因为路由中的id只能为1个数字

***

# JsonView的使用

有这样的场景，对于同样的一个类，在列表的接口中，可能不需要某一些字段，但这些字段可以在详情页出现，这时我们可以为每个场景创建一个类，不过这样的代码会十分的冗余，因此使用JsonView可以解决此问题

```java
/**
 * 
 */
package com.jiangyx.bookshop.dto.book;

import com.fasterxml.jackson.annotation.JsonView;

/**
 * 
 * @author Jhacker
 *
 */
public class BookInfo {
	
  	// 根据场景定义接口
	public interface BookListView {};
	public interface BookDetailView extends BookListView {};
	
	private Long id;
	private String name;
	private String content;
	
  	// 在各个属性的GET方法上定义字段显示的场景
	@JsonView(BookListView.class)
	public Long getId() {
		return id;
	}
	
	public void setId(Long id) {
		this.id = id;
	}
	
	@JsonView(BookListView.class)
	public String getName() {
		return name;
	}
	
	public void setName(String name) {
		this.name = name;
	}
	
	@JsonView(BookDetailView.class)
	public String getContent() {
		return content;
	}
	
	public void setContent(String content) {
		this.content = content;
	}
	
	@Override
	public String toString() {
		return "BookInfo [id=" + id + ", name=" + name + ", content=" + content + "]";
	}
}

```

```java
@GetMapping
@JsonView(BookListView.class)
public List<BookInfo> query(BookCondition condition, @PageableDefault(size=10, page=1) Pageable pageable) {
  List<BookInfo> books = new ArrayList<BookInfo>();
  books.add(new BookInfo());
  books.add(new BookInfo());
  books.add(new BookInfo());
  return books;
}

@GetMapping("/{id}")
@JsonView(BookDetailView.class)
public BookInfo queryInfo(@PathVariable Long id) {
  BookInfo bookInfo = new BookInfo();
  bookInfo.setId(1L);
  bookInfo.setName("战争与和平");
  return bookInfo;
}
```

这样在BookListView场景下只会显示id和name字段，在BookDetailView场景下会额外显示content字段

```java
/**
* 测试查询详情成功
* @throws Exception
*/
@Test
public void testWhenGetInfoSuccess() throws Exception {
  String result = mockMvc.perform(get("/book/1")
                                  .accept(MediaType.APPLICATION_JSON_UTF8))
    .andExpect(status().isOk())
    .andExpect(jsonPath("$.name").value("战争与和平"))
    .andReturn()
    .getResponse()
    .getContentAsString();
  System.out.println(result);
}

```

***

# SpringMVC中的参数校验

参数校验可以使用hibernate.validator中验证注解

```java
public class BookInfo {
	
	public interface BookListView {};
	public interface BookDetailView extends BookListView {};
	
	private Long id;
	private String name;
  
  	// 使用非空注解
	@NotBlank
	private String content;
	private Date publishDate;
}

```

```java
@PostMapping
public BookInfo create(@Validated @RequestBody BookInfo bookInfo, BindingResult result) {
  if (result.hasErrors()) {
    result.getAllErrors().stream()
      .forEach(error -> System.out.println(error.getObjectName() + error.getDefaultMessage()));
    System.out.println("返回错误信息");
  }
  System.out.println(bookInfo);
  System.out.println(bookInfo.getPublishDate());
  bookInfo.setId(1L);
  bookInfo.setContent("内容");
  return bookInfo;
}
```

```java
@Test
public void testWhenCreateBook() throws Exception {
  String content = "{\"id\": null, \"name\": \"战争与和平\", \"content\": null, \"publishDate\": \"2017-12-21\"}";
  String result = mockMvc.perform(post("/book")
                                  .content(content)
                                  .contentType(MediaType.APPLICATION_JSON_UTF8)
                                  .accept(MediaType.APPLICATION_JSON_UTF8))
    .andExpect(status().isOk())
    .andExpect(jsonPath("$.id").value("1"))
    .andReturn()
    .getResponse()
    .getContentAsString();
  System.out.println(result);
}
```

@Validated说明要使用校验

***

# SpringMVC中Date的处理

在使用Jackson进行参数绑定时，时区是默认为美国时区，因此应该在配置文件中设置Jackso使用的时区

```properties
spring.jackson.time-zone=GMT+8
```

***

# 获取请求中的Cookie和Header值

```java
@Test
public void testHeaderOrCookiesExists() throws Exception {
  mockMvc.perform(get("/book/cookies_or_header")
                  .cookie(new Cookie("name", "jiangyx"))
                  .header("auth", "tolensss")
                  .accept(MediaType.APPLICATION_JSON_UTF8))
    .andExpect(status().isOk());
}
```

```java
@GetMapping("/cookies_or_header")
public void cookies_or_header(@CookieValue String name, @RequestHeader String auth) {
  System.out.println(name);
  System.out.println(auth);
}
```

***

# 异常处理

## 显示自定义404页和500页面

在resources文件夹下的static目录下创建error/404.html和error/500.html，在请求返回400和500时会自动转向此页面

## 全局异常处理

```java
@RestControllerAdvice
public class ExceptionHandlerController {
	
	@ExceptionHandler(RuntimeException.class)
	@ResponseStatus(code= HttpStatus.INTERNAL_SERVER_ERROR)
	public Map<String, Object> handlerException(RuntimeException runtimeException) {
		Map<String, Object> map = new HashMap<>();
		map.put("code", "500");
		map.put("message", runtimeException.getMessage());
		return map;
	}
}
```



```java
@GetMapping("/exception")
public void exception() {
  throw new RuntimeException("收到抛出");
}
```

如上述代码所示，编写一个可以捕获全局RuntimeException的异常处理类，同样可以指定自定义异常的处理类，从而进行不同的返回

***

# 拦截器

创建我们自己的拦截器类并实现 HandlerInterceptor 接口

```java
/**
 * @author Jhacker
 *
 */
@Component
public class TimeInterceptor implements HandlerInterceptor {

	@Override
	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {
		// 在方法调用之前
		System.out.println("preHandle");
		System.out.println(((HandlerMethod)handler).getBean().getClass().getName());
		System.out.println(((HandlerMethod)handler).getMethod().getName());
		request.setAttribute("time", new Date().getTime());
		return true;
	}

	@Override
	public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
			ModelAndView modelAndView) throws Exception {
		// 在方法调用之后，如果抛出异常则不会调用
		System.out.println("postHandle");
		System.out.println("方法执行了" + (new Date().getTime() - (Long)request.getAttribute("time")));
	}

	@Override
	public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)
			throws Exception {
		// 在方法调用之后，如果抛出异常还会调用
		System.out.println("afterCompletion");
		System.out.println("方法执行了" + (new Date().getTime() - (Long)request.getAttribute("time")));
		System.out.println("ex:" + ex);
	}
}

```

要注意在afterCompletion中，如果在全局处理时有捕获异常，则会先将异常处理后再执行afterCompletion方法，因此其中的Exception其实一直为null，如果没有处理，则会有值



创建一个Java类继承WebMvcConfigurerAdapter，并重写 addInterceptors 方法。

```java
/**
 * @author Jhacker
 *
 */
@Configuration
public class WebConfig extends WebMvcConfigurerAdapter {
	
	@Autowired
	private TimeInterceptor timeInteceptor;
	
	@Override
	public void addInterceptors(InterceptorRegistry registry) {
      	// 实例化我们自定义的拦截器，然后将对像手动添加到拦截器链中 
		// 针对某些URL注册对应的拦截器
		registry.addInterceptor(timeInteceptor).addPathPatterns("/web/**", "/admin/**");
	}
}

```

***

# 过滤器

```java
/**
 * @author Jhacker
 *
 */
@Configuration
public class WebConfig extends WebMvcConfigurerAdapter {

	@Bean
	public FilterRegistrationBean characterEncodingFilterRegister() {
		FilterRegistrationBean registrationBean = new FilterRegistrationBean();
		CharacterEncodingFilter characterEncodingFilter = new CharacterEncodingFilter("utf-8");
		characterEncodingFilter.setForceEncoding(true);
		// 注册过滤器
		registrationBean.setFilter(characterEncodingFilter);
		List<String> list = new ArrayList<>();
		list.add("/admin/**");
		// 设置哪些URL需要进行过滤
		registrationBean.setUrlPatterns(list);
		return registrationBean;
	}
}

```

如上，注册了一个编码转换的过滤器，并且指定URL进行过滤。Listener和Servelet也是用类似的方法进行注册



***

# 文件上传

在配置文件中配置需要上传的文件最大值(默认为1MB)

```properties
# 设置最大上传大小
spring.http.multipart.max-file-size= 10MB
```

在控制器中写上传代码

```java
@PostMapping("/upload")
public FileInfo upload(MultipartFile file) throws IllegalStateException, IOException {

  String path = "D:\\upload";
  String extName = StringUtils.substringAfterLast(file.getOriginalFilename(), ".");
  File localFile = new File(path, new Date().getTime() + "." + extName);
  file.transferTo(localFile);

  return new FileInfo(localFile.getAbsolutePath());
}
```

使用MockMvc进行测试

```java
@Test
public void whenUploadSuccess() throws Exception {
  String result = mockMvc.perform(fileUpload("/file/upload")
                                  .file(new MockMultipartFile("file", "1.txt", "multipart/form-data", "hello upload".getBytes())))
    .andExpect(status().isOk())
    .andReturn()
    .getResponse().getContentAsString();
  System.out.println(result);
}
```

因为使用了StringUtils，则需要加入相关依赖

```xml
<dependency>
  <groupId>commons-lang</groupId>
  <artifactId>commons-lang</artifactId>
  <version>2.4</version>
</dependency>
```



***

# 文件下载

```java
@GetMapping("/download")
public void download(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse) throws FileNotFoundException, IOException {
  String filePath = "D:\\\\upload\\1514017586300.txt";
  try(InputStream inputStream = new FileInputStream(filePath);
      OutputStream outputStream = httpServletResponse.getOutputStream();){
    httpServletResponse.setContentType("application/x-download");
    httpServletResponse.addHeader("Content-Disposition", "attachment;filename=test.txt;");
    IOUtils.copy(inputStream, outputStream);
    outputStream.flush();
  }
}
```

使用了IOUtils,因此需要导入commons-io

```xml
<dependency>
  <groupId>commons-io</groupId>
  <artifactId>commons-io</artifactId>
  <version>2.2</version>
</dependency>
```

***

# 异步处理Http请求

## Callable

```java
/**
	 * 查询详情
	 * @param id
	 * @return
	 */
@GetMapping("/{id:\\d}")
@JsonView(BookDetailView.class)
public Callable<BookInfo> queryInfo(@PathVariable int id) {
  Long startTime = new Date().getTime();
  System.out.println("请求开始" + Thread.currentThread().getName());
  Callable<BookInfo> result = () -> {
    System.out.println("线程开始" + Thread.currentThread().getName());
    Thread.sleep(1000);
    BookInfo bookInfo = new BookInfo();
    bookInfo.setId(1L);
    bookInfo.setName("战争与和平");
    System.out.println("线程结束" + Thread.currentThread().getName() + ",返回耗时" + (new Date().getTime() - startTime) + "ms");
    return bookInfo;
  };
  System.out.println("请求结束" + Thread.currentThread().getName() + ",返回耗时" + (new Date().getTime() - startTime) + "ms");
  return result;

}
```

将Callable返回就能异步的处理Http请求，当一个请求进来时，会新开一个线程来处理请求数据的操作，主线程则会直接返回，开始处理下一个请求，当线程返回结果后，客户端才会收到响应。

Callable并不能加快访问速度，只是提高了并发量而已。

## DeferredResult

Callable只能处理在一个线程中能获取到结果数据的情景，如果我的数据是通过消息队列的方式，监听获取到的则就不能使用上述方法了。这时可以使用DeferredResult

```java
private ConcurrentMap<Long, DeferredResult<BookInfo>> map = new ConcurrentHashMap<>();
/**
	 * 查询详情
	 * @param id
	 * @return
	 */
@GetMapping("/{id:\\d}")
@JsonView(BookDetailView.class)
public DeferredResult<BookInfo> queryInfo(@PathVariable int id) {
  DeferredResult<BookInfo> result = new DeferredResult<>();
  map.put((long) id, result);
  return result;

}

private void listener(BookInfo info) {
  map.get(info.getId()).setResult(info);
}
```

使用DeferredResult就能在任意线程中返回获取的对象



***

# 使用Swagger生成文档

当我们开发前后端分离项目的时候，接口文档的维护是非常麻烦的事情，使用Swagger可以自动根据接口生成相应的文档



1、添加依赖

```xml
<dependency>
  <groupId>io.springfox</groupId>
  <artifactId>springfox-swagger2</artifactId>
  <version>2.6.1</version>
</dependency>
<dependency>
  <groupId>io.springfox</groupId>
  <artifactId>springfox-swagger-ui</artifactId>
  <version>2.6.1</version>
</dependency>
```

2、开启使用Swagger

在SpringBoot启动文件上添加注解**@EnableSwagger2** 

```java
@SpringBootApplication
@EnableSwagger2
public class BookshopAdminApplication {

	public static void main(String[] args) {
		SpringApplication.run(BookshopAdminApplication.class, args);
	}
}
```

3、重启服务并访问域名/swagger-ui.html#/

***

# WireMock伪造服务

http://wiremock.org/docs/getting-started/