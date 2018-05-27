---
title: Flask路由系统
copyright: true
date: 2018-05-27 11:29:17
tags: [Python, Flask]
categories: Flask深入学习
---
# 简介

Flask中原生提供路由系统方便我们使用，在第一个例子中

```python
@app.route('/')
def index():
    return "hello world"
```

这样简短的代码，Flask就能当我们访问/时候帮我们匹配到index函数，那么问题就来了，Flask帮我们做了什么？先看以下route的定义：

```python
def route(self, rule, **options):
    def decorator(f):
        # 从字典中获取endpoint的值，如果不存在则赋值为None
        endpoint = options.pop('endpoint', None)
        # 调用Flask实例对象app的add_url_rule方法添加映射
        self.add_url_rule(rule, endpoint, f, **options)
        return f
    return decorator
```

有上面代码可看出，Flask提供了route装饰器简便我们添加路由映射的操作，但本质是调用了add_url_rule方法

```python
@setupmethod
def add_url_rule(self, rule, endpoint=None, view_func=None, **options):
    # 如果endpoint没有赋值的话，Flask会读取我们试图函数的名称作为endpoint
    if endpoint is None:
        endpoint = _endpoint_from_view_func(view_func)
    options['endpoint'] = endpoint
    # 读取methods参数
    methods = options.pop('methods', None)
	
    # 如果view_func的methods属性为None，则默认为有个仅包含GET的元祖
    # 否则取view_func的methods属性
    if methods is None:
        methods = getattr(view_func, 'methods', None) or ('GET',)
        
    # 如果methods是字符串类型的则抛出异常
    # methods必须为由字符串组成的可迭代对象
    if isinstance(methods, string_types):
        raise TypeError('Allowed methods have to be iterables of strings, '
                        'for example: @app.route(..., methods=["POST"])')
    
    # 将所有的method转为大写
    methods = set(item.upper() for item in methods)
	.....
    # 实例化一个Rule类
    rule = self.url_rule_class(rule, methods=methods, **options)
    rule.provide_automatic_options = provide_automatic_options
	
    # 将规则实例添加到路由映射中
    self.url_map.add(rule)
    if view_func is not None:
        old_func = self.view_functions.get(endpoint)
        if old_func is not None and old_func != view_func:
            raise AssertionError('View function mapping is overwriting an '
                                 'existing endpoint function: %s' % endpoint)
        # 将endpoint和view_func进行映射，方便使用url_for生成路由
        self.view_functions[endpoint] = view_func
```

# 添加路由的两种方法

从上述代码可以知道`app.route`本质上是一个带参数的装饰器，在其中使用add_url_rule方法进行路由的添加，因此添加一个路由的方式有

## 使用装饰器添加

``` python
@app.route('/')
def index():
    pass
```

## 使用函数添加

``` python
app.add_url_rule('/', 'index', index)
```

> 注意如果我们另外自定义了装饰器来装饰视图函数时，**一定要注意装饰器的位置**，**多个装饰器对同一个函数进行装饰时是有执行先后的，顺序为从下到上** 

``` python
def check_session(func):
    """
    对用户登录进行校验
    """
    def inner(*args, **kwargs):
        if session.get('user') is None:
            return redirect(location='/login/')
        return func(*args, **kwargs)
    return inner

@app.route('/user', methods=['GET'])
@check_session
def user_index():
    pass
```

上述代码check_session必须放在@app.route下方。

其原因是当多个装饰器修饰同一个函数时，上层的装饰器会将下层装饰器以及被装饰的函数作为`参数func`传递到自己的参数列表中。这意味着调用被装饰的函数`index`时，本质上是调用的经过了层层封装之后的一个装饰器函数，外层函数执行时会将其内封装的其他函数层层由内往外执行。

如果将check_session放在上方，则在访问/user路径时，根据装饰器的顺序先执行了经@app.route装饰的方法，其中没有做校验，并直接将响应返回，因此不会执行check_session中的方法。

下面的例子可以体现装饰器的顺序：

```python
def one(func):
    def inner(*args, **kwargs):
        print('1111111 ,before')
        ret = func(*args, **kwargs)
        print('111111, after')
        return ret
    return inner


def two(func):
    def inner(*args, **kwargs):
        print('2222222 ,before')
        ret = func(*args, **kwargs)
        print('22222222, after')
        return ret
    return inner


@one
@two
def index():
    print(index.__name__, 'running')
# 相当于index = one(two(index))

"""
执行结果
1111111 ,before
2222222 ,before
inner running
22222222, after
111111, after
"""

index()
```

# route参数介绍

```python
route(rule, endpoint, methods=['GET'], alias=None, defaults=None, strict_slashes=None, redirect_to=None, subdomain=None)
```

* rule：url路径
* endpoint：视图函数别名
  * 设置此值可以使用`url_for()` 来生成rule对应的url路径
  * 可以在`app.view_functions` 这个字典中通过此别名获取视图函数对象
* method：指定视图函数处理请求的类型。默认为GET，常用的还有POST、PUT等
* alias：URL路径的别名
* defaults：当url路径中没有传递参数，但是对应的视图函数有需要参数的时候，可以使用defaults创建一个字典来传递给视图函数
  * **注意，当为视图函数使用defaults指定了参数，并且url中也提供了改参数时，视图函数会使用defaults指定的参数而忽略掉url中传递的参数** 
* strict_slashes：是否对url最后的**/**严格要求，当设置为True时，程序在处理url的时候不会再url后自动添加/

``` python
@app.route('/index', strict_slashes=False)
# 访问/index和/index/均可

@app.route('/index', strict_slashes=True)
# 只能访问/index
```

* redirect_to：当请求的url匹配到了设置了redirect_to的路由，该请求会自动被重定向到redirect_to指定的url路径
  * 当项目迁移的时候可能会使用，例如当旧版url和新版url不同时，访问旧版url就会自动重定向到新版上面。
  * 如果url中提供了参数，并且改参数在新版的视图函数中也需要用到时，就需要将该参数作为参数传递给新的url中

``` python
@app.route('/index/<int:id>', redirect_to='/home/<id>')

# 或者可以如下
def old(adapter, id):
    return f'/home/{id}'
@app.route('/index/<int:id>', redirect_to=old)
```

* subdomain：定义二级域名时使用，但是必须添加SERVER_NAME配置

``` python
from flask import Flask

app = Flask(__name__)
app.config['SERVER_NAME'] = 'jiangyx.com:5000'

# 请求http://blog.jiangyx.com:5000
@app.route('/', subdomain='blog')
def index():
    return 'blog.jiangyx.com'

# 动态匹配子域名
# 访问http://jiang.jiangyx.com:5000/blog  => jiang.jiangyx.com
# 访问http://yx.jiangyx.com:5000/blog  => yx.jiangyx.com
@app.route('/blog', subdomain="<var>")
def login(var):
    return f"{var}.jiangyx.com"

"""
还需要在hosts文件中加入子域名的配置
127.0.0.1 blog.jiangyx.com
127.0.0.1 jiang.jiangyx.com
127.0.0.1 yx.jiangyx.com
"""
```

# 流程分析

在起初我们大致分析了route的装载原理，下面我们在按照流程分析下

## route装饰器

实际上在route装饰器的内部主要就是调用了`add_url_rule` 方法，只不过使用了装饰器后，就不需要指定`view_func` 和`endpoint` 了(如果不需要额外命名)

``` python
def route(self, rule, **options):
    def decorator(f):
        endpoint = options.pop('endpoint', None)
        self.add_url_rule(rule, endpoint, f, **options)
        return f
    return decorator
```

## add_url_rule

在add_url_rule内部，将传入的url路径和其他参数进一步封装成了一个Rule对象，这一步骤是通过实例化`app` 对象的`self.url_rule_class` 实现的，然后将封装后的Role对象保存到了`app`对象的`url_map`中。最后将`endpoint`和视图函数`view_func` 最为一组键值对存放在`app`对象的`view_functions`字典中。而`app.view_functions`将被作用来执行视图函数。

``` python
class Flask(_PackageBoundObject):
    # url_rule_class指向了Role类
    url_rule_class = Rule
    
    @setupmethod
    def add_url_rule(self, rule, endpoint=None, view_func=None, **options):
        ....
        # 传入的url路径和其他参数进一步封装成了一个Rule对象
        rule = self.url_rule_class(rule, methods=methods, **options)        
        rule.provide_automatic_options = provide_automatic_options  
        # 保存到url_map中
        self.url_map.add(rule)
        if view_func is not None:
            old_func = self.view_functions.get(endpoint)
            if old_func is not None and old_func != view_func:
                raise AssertionError('View function mapping is overwriting an '
                                     'existing endpoint function: %s' % endpoint)
            # 将视图函数和endpoint做映射保存进字典中
            self.view_functions[endpoint] = view_func               
```

## url_map

而`url_map`本质上是一个`Map`对象

```python
url_map = Map([
    Rule('/foo/<slug>', endpoint='foo'),
    Rule('/some/old/url/<slug>', redirect_to='foo/<slug>'),
    Rule('/other/old/url/<int:id>', redirect_to=foo_with_slug)
]
```

## Roule类

```python
class Rule(RuleFactory):
    def __init__(self, string, defaults=None, subdomain=None, methods=None,
                 build_only=False, endpoint=None, strict_slashes=None,
                 redirect_to=None, alias=False, host=None):
        if not string.startswith('/'):
            raise ValueError('urls must start with a leading slash')
        self.rule = string
        self.is_leaf = not string.endswith('/')

        self.map = None
        self.strict_slashes = strict_slashes
        self.subdomain = subdomain
        self.host = host
        self.defaults = defaults
        self.build_only = build_only
        self.alias = alias
        if methods is None:
            self.methods = None
        else:
            if isinstance(methods, str):
                raise TypeError('param `methods` should be `Iterable[str]`, not `str`')
            self.methods = set([x.upper() for x in methods])
            if 'HEAD' not in self.methods and 'GET' in self.methods:
                self.methods.add('HEAD')
        self.endpoint = endpoint
        self.redirect_to = redirect_to

        if defaults:
            self.arguments = set(map(str, defaults))
        else:
            self.arguments = set()
        self._trace = self._converters = self._regex = self._argument_weights = None
```

可以看出Route类封装了大部分app.route装饰器所传入的参数，Flask路由本质就是从url_map中找出和当前路径匹配的Rule实例，并在执行一系列的操作。