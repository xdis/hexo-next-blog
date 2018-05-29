---
title: Flask之请求扩展
copyright: true
date: 2018-05-29 19:29:17
tags: [Python, Flask]
categories: Flask深入学习
---
# 简介

Flask中请求的扩展指的是在一次请求之前和之后执行的操作，有点像Java中Filter过滤器，它也可以设置一个或多个，并按照链条依次执行。

# before_request

`before_request`表示在所有的请求之前进行拦截，如果该装饰器装饰的函数有返回值，则会直接返回响应，不会往下继续执行。可以使用它来进行权限管理而不使用装饰器。

**基于before_request做用户登录认证**

``` python
@app.before_request
def check_login():
    if request.path != '/login':
        if not session.get('user'):
            return redirect('/login')
```

# after_request

`after_request`表示在响应返回之后执行的函数，Flask在执行该函数时会自动注入response参数，而且必须返回该response

``` python
a = 0

@app.before_request
def process_request():
    global a
    a += 1
    print(f'start {a}')


@app.after_request
def process_response(response):
    global a
    a += 1
    print(f'end {a}')
    return response


@app.route('/')
def index():
    global a
    a += 1
    return f"index {a}"

"""
可以看到结果为
start 1
index 2
end 3
从此也可以看出执行的顺序
before_request
视图函数
after_request
"""
```

# before_first_request

`before_first_request`表示在应用启动后第一个请求之前所进行的处理。一般进行一些全局的初始化。

``` python
@app.before_request
def process():
    print('before')


@app.before_first_request
def first():
    print('first')


@app.route('/', methods=['GET'])
def index():
    print('index函数')
    return "Index"

"""
输出的结果为
first
before
index函数
"""
```



# Flask请求扩展的执行顺序

对于`before_request` 的执行顺序链是先进先执行，而对于`after_request`的执行顺序链则是先进后执行，详细看下面的例子：

```python
@app.before_request
def process_request1(*args,**kwargs):
	print('process_request1 进来了')

@app.before_request
def process_request2(*args,**kwargs):
	print('process_request2 进来了')


@app.after_request
def process_response1(response):
	print('process_response1 走了')
	return response

@app.after_request
def process_response2(response):
	print('process_response2 走了')
	return response


@app.route('/index',methods=['GET'])
def index():
	print('index函数')
	return "Index"

"""
执行的结果为
process_request1 进来了
process_request2 进来了
index函数
process_response2 走了
process_response1 走了
"""
```

**需要注意的是** ：当在`before_request`进行拦截的时候，`after_request`仍然会全部执行完毕。

``` python
@app.before_request
def process_request1(*args,**kwargs):
	print('process_request1 进来了')
    return "拦截"

@app.before_request
def process_request2(*args,**kwargs):
	print('process_request2 进来了')


@app.after_request
def process_response1(response):
	print('process_response1 走了')
	return response

@app.after_request
def process_response2(response):
	print('process_response2 走了')
	return response


@app.route('/index',methods=['GET'])
def index():
	print('index函数')
	return "Index"

"""
执行的结果为
process_request1 进来了
process_response2 走了
process_response1 走了
"""
```

# 定制错误信息

使用Flask的请求扩展可以让我们自定义错误信息的处理，例如自定义404、500页面。

``` python
@app.errorhandler(404)
def handler_404(arg):
    # 错误详情，可以进行页面跳转和写入错误日志
    print(arg)
    return "404"
```

# 定制模板中的方法

Flask请求扩展除了以上几种用法，还可以创建模板中的函数和过滤器

``` python
@app.template_global()
def add(a1, a2):
	return a1 + a2

"""
定义模板中的方法
在模板中使用
{{add(1, 2)}}
"""
```

```python
@app.template_filter()
def db(a1, a2, a3):
    return a1 + a2 + a3

"""
定义模板中的过滤器
模板中的使用

{{ 1|db(2,3)}}

其中 
1 = a1
2 = a2
3 = a3
"""
```

