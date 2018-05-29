---
title: Flask之请求扩展
copyright: true
date: 2018-05-29 19:29:17
tags: [Python, Flask]
categories: Flask深入学习
---
# 简介

Flask中没有直接提供中间件，但是我们可以使用Flask的请求的扩展来达到中间件的效果。请求扩展指的是在一次请求之前和之后执行的操作，有点像Java中Filter过滤器，它也可以设置一个或多个，并按照链条依次执行。Flask提供给我们`before_request`、`after_request`和`before_first_request`三个请求扩展。

# before_request

`before_request`装饰的函数会在请求进入视图函数前执行 ，与`Django`类似，如果该函数返回一个`None`，那么就会进入下一个被`@before_request`装饰的函数或者视图函数；如果该函数返回值不为`None`而是返回了一个响应对象，那么该函数之后就不会再进入视图函数，而是进入对应的`@after_request`装饰的函数，该函数执行完之后会进入更外层的一个被`@after_request`装饰的函数，最终响应经由`Web服务器(wsgi)`发送给客户端 。

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

`before_first_request`只会在第一个请求到来时触发其装饰的函数，内部实际上是利用一个`flag`状态实现，最初的时候，`flag=Flase`，这个条件下会从执行该函数，并在函数执行结束将`flag`的值修改成`True`。详细的源码分析会在下面展开 

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

***

# 源码分析

## 本质分析

在实例化Flask对象时，在其`_init__`方法中会为`before_request`和`after_request`分别创建一个字典，为`before_first_request`创建一个列表。

``` python
class Flask: 
    def __init__():
        ....
        self.before_request_funcs = {}
 
        self.before_first_request_funcs = []
 
        self.after_request_funcs = {}
        ...
```

对于`before_request`和`after_request`，当使用这两个装饰器装饰函数的时候，会将被装饰的函数添加到以`None`为键对应的列表中，这里使用`None`表示这是装饰`app`中的函数。在使用`蓝图（blueprint）`时，则就会以蓝图中的`name`作为键，将被装饰的函数存放如对应的列表中。

```python
class Flask: 
    ....
    @setupmethod
    def before_request(self, f): 
        self.before_request_funcs.setdefault(None, []).append(f)
        return f

    @setupmethod
    def before_first_request(self, f): 
        self.before_first_request_funcs.append(f)
        return f

    @setupmethod
    def after_request(self, f): 
        self.after_request_funcs.setdefault(None, []).append(f)
        return f
```

而在蓝图中

```python
class Blueprint:
    def before_request(self, f): 
        self.record_once(lambda s: s.app.before_request_funcs
            .setdefault(self.name, []).append(f))
        return f
    def after_request(self, f): 
        self.record_once(lambda s: s.app.after_request_funcs
            .setdefault(self.name, []).append(f))
        return f
```



> 在执行视图函数之前或之后，会分别遍历上述字典或列表，并执行其中的方法，下面源码中会详细介绍



## 详细流程分析

为了完整的弄清楚请求扩展的原理，我将从一个请求到响应来做详细的介绍。

当一个请求到达时，首先会执行`Flask`对象的`__call__()`方法 ，我们先从`app.__call__()`方法开始

`__call__`返回值中调用了`Flask`对象的`wsgi_app`方法，其内执行了`full_dispatch_request`方法

### 执行before_first_request

``` python
class Flask:
    def __init__(self, import_name, static_path=None, static_url_path=None,
                 static_folder='static', template_folder='templates',
                 instance_path=None, instance_relative_config=False,
                 root_path=None):
        # 当初始化时，设置是否已有第一次请求的Flag为False
        self._got_first_request = False
        
    def __call__(self, environ, start_response):
        """Shortcut for :attr:`wsgi_app`."""
        return self.wsgi_app(environ, start_response)
    def wsgi_app(self, environ, start_response):
        ctx = self.request_context(environ)
        ctx.push()
        error = None
        try:
            try:
                # 处理请求并生成响应返回
                response = self.full_dispatch_request()
            except Exception as e:
                error = e
                response = self.handle_exception(e)
            except:
                error = sys.exc_info()[1]
                raise
            return response(environ, start_response)
        finally:
            if self.should_ignore_error(error):
                error = None
            ctx.auto_pop(error)
            
    def full_dispatch_request(self):
        # 尝试执行@app.before_first_request
        self.try_trigger_before_first_request_functions()
        try:
            request_started.send(self)
            rv = self.preprocess_request()
            if rv is None:
                rv = self.dispatch_request()
        except Exception as e:
            rv = self.handle_user_exception(e)
        return self.finalize_request(rv)

    def try_trigger_before_first_request_functions(self):
        # 为True时，不执行func()，直接跳过
        if self._got_first_request:
            return
        # 将Flag锁住，防止再多线程情况下执行多次
        with self._before_request_lock:
            if self._got_first_request:
                return
            # 循环函数列表，并执行各个函数
            for func in self.before_first_request_funcs:
                func()
            # 执行完之后，设置为True，以后将不再执行
            self._got_first_request = True
```

### 执行before_request

在执行完成`before_first_request`之后，就会执行`before_request`装饰的函数，对应的在`full_dispatch_request`中会执行`preprocess_request`方法，在该方法内部会从`before_request_funcs`获取`before_request`装饰的所有函数，并执行

``` python
def full_dispatch_request(self):
    # 尝试执行@app.before_first_request
    self.try_trigger_before_first_request_functions()
    try:
        # 首先会通过信号的方式告诉Flask开始处理当前请求
        request_started.send(self)
        # 执行before_request装饰的所有函数
        rv = self.preprocess_request()
        # 如果返回不为None则不会执行视图函数
        if rv is None:
            # 当这些函数均没有返回值的时候，就会执行dispatch_request方法，
            # 该方法内部就会执行请求url匹配到的视图函数
            rv = self.dispatch_request()
    except Exception as e:
        rv = self.handle_user_exception(e)
    return self.finalize_request(rv)

def preprocess_request(self):
    bp = _request_ctx_stack.top.request.blueprint

    funcs = self.url_value_preprocessors.get(None, ())
    if bp is not None and bp in self.url_value_preprocessors:
        funcs = chain(funcs, self.url_value_preprocessors[bp])
    for func in funcs:
        func(request.endpoint, request.view_args)
    # 获取before_request装饰的所有函数
    funcs = self.before_request_funcs.get(None, ())
    if bp is not None and bp in self.before_request_funcs:
        funcs = chain(funcs, self.before_request_funcs[bp])
    for func in funcs:
        rv = func()
        # 表示拦截
        if rv is not None:
            return rv
        
def dispatch_request(self):
    req = _request_ctx_stack.top.request
    if req.routing_exception is not None:
        self.raise_routing_exception(req)

    rule = req.url_rule
    if getattr(rule, 'provide_automatic_options', False) \
       and req.method == 'OPTIONS':
        return self.make_default_options_response()
    # 执行视图函数
    return self.view_functions[rule.endpoint](**req.view_args)
```

### 执行after_request

执行完url匹配的视图函数之后，如果不出错，就会执行最后一个函数`finalize_request()`，在这个函数内部会使用`make_response`将上述函数的返回值封装成一个响应对象，然后将该响应对象传递给`process_response(response)`方法

```python
def finalize_request(self, rv, from_error_handler=False):
    response = self.make_response(rv)
    try:
        # 处理after_request_funcs中的函数
        response = self.process_response(response)
        # 信号通知flask，请求处理完毕，可以返回响应了
        request_finished.send(self, response=response)
    except Exception:
        if not from_error_handler:
            raise
        self.logger.exception('Request finalizing failed with an '
                              'error while handling an error')
    return response

def process_response(self, response):
    ctx = _request_ctx_stack.top
    bp = ctx.request.blueprint
    funcs = ctx._after_request_functions
    # 获取所有蓝图中定义的after_request
    if bp is not None and bp in self.after_request_funcs:
        funcs = chain(funcs, reversed(self.after_request_funcs[bp]))
    # 获取所有app中定义的after_request  
    if None in self.after_request_funcs:
        funcs = chain(funcs, reversed(self.after_request_funcs[None]))
    # 将所有的函数合成一个列表
    # 遍历并执行after_request_funcs中的函数
    for handler in funcs:
        response = handler(response)
    if not self.session_interface.is_null_session(ctx.session):
        self.save_session(ctx.session, response)
    return response
```