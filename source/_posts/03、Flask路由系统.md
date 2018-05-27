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

在`add_url_rule`内部，将传入的url路径和其他参数进一步封装成了一个Rule对象，这一步骤是通过实例化`app` 对象的`self.url_rule_class` 实现的，然后将封装后的Role对象保存到了`app`对象的`url_map`中。最后将`endpoint`和视图函数`view_func` 最为一组键值对存放在`app`对象的`view_functions`字典中。而`app.view_functions`将被作用来执行视图函数。

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

# CBV和FBV

Flask提供了CBV和FBV让我们注册视图函数

## FBV

FBV就是上述讲到的使用装饰器绑定视图函数的方法，也是最常用的一种方法，和Django中的视图函数几乎相同，但是有几个不同的地方 

* 在flask中， 视图函数不需要在参数列表中传递`request`对象，而是通过导入的方式获取
* flask中的视图函数直接可以返回一个`字符串`，作用相当于Django的`HttpResponse`

``` python
@app.route('/login')
def login():
    return "login"
```

## CBV

CBV是类似Django中常用的一种注册路由的方式，它是基于类来实现的，实现CBV有以下步骤

### 导入MethodView，并创建其派生类

```python
from flask.views import MethodView
```

### 创建请求对应的方法

在`MethodView`的派生类中，方法名要与请求的方式相对应，比如GET请求，需要定义`get`方法

``` python
class IndexView(MethodView):
    def get(self):
        pass
```

### 添加配置

在`MethodView`的派生类中，还可以添加一些配置，就像在装饰器中那样

#### method

一个列表，存放能够处理的请求类型，如果`method`指定为`['GET']`，即使派生类中定义了处理POST请求的`post`方法，当有POST请求匹配到该路由时，也无法执行`post`方法。

``` python
class IndexView(MethodView):
    methods = ['GET']
    
    def get(self):
        pass
```

#### decorators

一个列表，存放装饰器函数的函数引用，在函数列表中的顺序决定装饰器的执行顺序

```python
class IndexView(MethodView):
    methods = ['GET']
    decorators = [one, two]

    def get(self):
        pass

"""
相当于
@one
@two
"""
```

### 路由配置

最后一步就是注册路由了，CBV的路由注册只能使用`app.add_url_rule()`

```python
app.add_url_rule('/index', view_func=IndexView.as_view('index'))
```

### as_view

`as_view`方法会将`MethodView`的派生类转换成一个Flask路由系统能够使用的视图函数，当派生类中指定了装饰器时，会遍历装饰器列表，并将装饰器装饰到视图函数上。

``` python
class View(object):
    @classmethod
    def as_view(cls, name, *class_args, **class_kwargs):
        
        def view(*args, **kwargs):
            self = view.view_class(*class_args, **class_kwargs)
            return self.dispatch_request(*args, **kwargs)

        if cls.decorators:
            view.__name__ = name                
            view.__module__ = cls.__module__
            # 遍历列表，循环对视图函数使用了装饰器
            for decorator in cls.decorators:
                view = decorator(view)              
        view.view_class = cls
        view.__name__ = name
        view.__doc__ = cls.__doc__
        view.__module__ = cls.__module__
        view.methods = cls.methods
        return view
```

``` python
class MethodView(with_metaclass(MethodViewType, View)):
     
    def dispatch_request(self, *args, **kwargs):
        meth = getattr(self, request.method.lower(), None)
        if meth is None and request.method == 'HEAD':
            meth = getattr(self, 'get', None)
        assert meth is not None, 'Unimplemented method %r' % request.method
        return meth(*args, **kwargs)
```

# converter（转换器）

在我们使用`@app.route`的时候，其中的`rule`可以使用类似于`rule='/<int:nid>'`，这样使用flask就会将nid转为int类型，其本质是`Converter`转换器类。接下来，我们通过源码来找一下Flask提供的几个内置转换器类。

首先进入`route`装饰器内部，我们可以看见该装饰器本质上是调用的`add_url_rule`方法，并将装饰器中的参数传入

```python
Class Flask:
    def route(self, rule, **options):
        def decorator(f):
            endpoint = options.pop('endpoint', None)
            self.add_url_rule(rule, endpoint, f, **options)
            return f
        return decorator
```

进入`add_url_role`中看看是怎么处理`rule`的，因为转换器就存在于`rule`中，发现该方法中对`rule`进行了一次封装，并返回新的`Rule类`的对象

```python
class Flask:
    url_rule_class = Rule

    @setupmethod
    def add_url_rule(self, rule, endpoint=None, view_func=None, **options):
        #   中间省略部分
        methods |= required_methods
        # 封装rule
        rule = self.url_rule_class(rule, methods=methods, **options)   
```

在`Role`类中，有一个`get_converter`方法，这个方法就是用来获取`rule`中我们指定的转换器名称的转换器

```python
def get_converter(self, variable_name, converter_name, args, kwargs):
    if converter_name not in self.map.converters:
        raise LookupError('the converter %r does not exist' % converter_name)
    # 调用的是Map类中的converters属性
    return self.map.converters[converter_name](self.map, *args, **kwargs)
```

查看Map类的定义，可以看见其默认设置的转换器为`DEFAULT_CONVERTERS`

``` python
class Map:
    self.converters = self.default_converters.copy()
    default_converters = ImmutableDict(DEFAULT_CONVERTERS)
```

在routing.py中就会找到Flask支持的默认转换器`DEFAULT_CONVERTERS`

```python
DEFAULT_CONVERTERS = {
    'default':          UnicodeConverter,
    'string':           UnicodeConverter,
    'any':              AnyConverter,
    'path':             PathConverter,
    'int':              IntegerConverter,
    'float':            FloatConverter,
    'uuid':             UUIDConverter,
}
```

从上面的代码可以看出，每一个转换器都对应一个自己的Converter类，而查看Converter的基类BaseConverter中可以看到，每个Converter类都需要定义`to_python`和`to_url`方法。

```python
class BaseConverter(object):

    """Base class for all converters."""
    regex = '[^/]+'
    weight = 100

    def __init__(self, map):
        self.map = map

    def to_python(self, value):
        return value

    def to_url(self, value):
        return url_quote(value, charset=self.map.charset)
```

## to_python

to_python(self, value) 用于视图函数，参数`value`是从请求路径中匹配到的字符串value，该函数返回值将传给视图函数来作为视图函数的参数。例如

``` python
@app.route('/user/<int:id>')
def index(id):
    pass

"""
to_python(self, value) 中的value对应的是/user/<int:id>中匹配到的字符串id
def index(id) 视图函数中的id则是经过to_python处理过后返回的值
"""
```

## to_url

`to_url(self, value)`用于`url_for`反向生成url，该参数vlue作为传递给`url_for`的参数，返回的值用于生成URL中的参数

## 自定义正则匹配转换器

### 定义转换器类

```python
from werkzeug.routing import BaseConverter
 
class RegexConverter(BaseConverter):
    """
    自定义URL匹配正则表达式
    """
    def __init__(self, map, regex):
        super(RegexConverter, self).__init__(map)
        self.regex = regex

    def to_python(self, value):
        """
        路由匹配时，匹配成功后传递给视图函数中参数的值
        :param value: 
        :return: 
        """
        return int(value)

    def to_url(self, value):
        """
        使用url_for反向生成URL时，传递的参数经过该方法处理，返回的值用于生成URL中的参数
        :param value: 
        :return: 
        """
        val = super(RegexConverter, self).to_url(value)
        return val
```

### 添加配置

``` python
# 添加到Flask中
app.url_map.converters['regex'] = RegexConverter
```

该操作其实是在DEFAULT_CONVERTERS中添加一组键值对`regex:RegexConverter `

``` python
DEFAULT_CONVERTERS = {
    'default':          UnicodeConverter,
    'string':           UnicodeConverter,
    'any':              AnyConverter,
    'path':             PathConverter,
    'int':              IntegerConverter,
    'float':            FloatConverter,
    'uuid':             UUIDConverter,
    'regex':            RegexConverter,
}
# 这样，以后在调用该转换器时，在rule中只需要使用regex就可以了
```

### 使用

```python
@app.route('/index/<regex("\d+"):nid>')
def index(nid):
    print(url_for('index', nid='888'))
    return 'Index'
```

