---
title: Flask之request和response
copyright: true
date: 2018-06-06 20:16:17
tags: [Python, Flask]
categories: Flask深入学习
---
# request对象

## request对象剖析

在Flask中，每一次请求的请求对象都是不同的。并且和Django不同，如果你想在视图函数中操作当前url的请求，你需要导入`from flask import request` ，而不是像Django那样作为参数传入。

既然Flask中的request是从外部导入的，那么Flask怎么实现请求的隔离呢？Flask底层是通过堆栈和Local对象实现的，Flask会单独为该线程/协程开辟一块空间，其中会包含一个列表，每当请求进来，就会将请求数据放入到列表中，然后request对象会将队列最前面的请求数据取出。

## 在视图函数中使用request对象

### request.method

获取当前请求的类型。通常在视图函数中对于不同请求类型进行不同的业务逻辑操作。

### request.cookies

返回一个存放了`cookie` 键值对的 `ImmutableTypeConversionDict ` 对象。该类继承`dict` ，因此可以像操作字典那样操作`cookie` 。

```python
{'csrftoken': 'InkKIBWF7qJicP87jhGogOMeRq9ahiLWGn2smeP6MuafECKUqQNTld1OMsGQXtUN', 'session': 'eyJ1c2VyaW5mbyI6InppYXdhbmcifQ.DS9RNg.Vuuq0DDy1tlORml8IYgTAThecRA'} 
```

### request.headers

获取当前请求头信息，返回值为一个`EnvironHeaders`对象，该对象支持操作请求头的各种方法，比如`get, getlist, add, pop等` 

```python
Host: 127.0.0.1:5000
Connection: keep-alive
Content-Length: 19
Pragma: no-cache
Cache-Control: no-cache
Origin: http://127.0.0.1:5000
Upgrade-Insecure-Requests: 1
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.84 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
Referer: http://127.0.0.1:5000/login/
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9
Cookie: csrftoken=InkKIBWF7qJicP87jhGogOMeRq9ahiLWGn2smeP6MuafECKUqQNTld1OMsGQXtUN; session=eyJ1c2VyaW5mbyI6InppYXdhbmcifQ.DS9RNg.Vuuq0DDy1tlORml8IYgTAThecRA

 # Post 请求
```

```python
Host: 127.0.0.1:5000
Connection: keep-alive
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.84 Safari/537.36
Upgrade-Insecure-Requests: 1
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9
Cookie: csrftoken=InkKIBWF7qJicP87jhGogOMeRq9ahiLWGn2smeP6MuafECKUqQNTld1OMsGQXtUN; session=eyJ1c2VyaW5mbyI6InppYXdhbmcifQ.DS9RNg.Vuuq0DDy1tlORml8IYgTAThecRA

# Get请求
```

### request.path

获取当前请求路径，不包含查询字符串 `/login/`

### request.full_path

获取当前请求路径以及查询字符串`/login/?name=ziawang&age=12` 

### request.url

当前请求的完整url地址`http://127.0.0.1:5000/login/ ` 

### request.base_url

完整url地址不带查询字符串`http://127.0.0.1:5000/login/ ` 

### request.url_root

返回根路径 `http://127.0.0.1:5000/` 

### request.host_url

返回主机的IP地址以及端口号组成的url地址`http://127.0.0.1:5000/ ` 

### request.host

返回主机的IP地址以及端口号组成的字符串`127.0.0.1:5000` 

### request.files

获取前端`form`表单提交的文件（对于POST请求，需要设置form表单的`enctype`参数）。

返回的对象直接就可以使用其`save`方法保存文件

### request.url_role

返回当前请求匹配到的路由映射关系中`role`

```python
@app.route('/login/<username>/', methods=['GET', 'POST'])
def login(username):
    if request.method == 'GET':
        print(request.url_rule)
        return render_template('login.html')

# ------------------------结果
/login/<username>/
```

### request.view_args

返回一个存放了视图函数参数与对应value组成的字典

```python
@app.route('/login/<username>/', methods=['GET', 'POST'])
def login(username):
    if request.method == 'GET':
        print(request.view_args, 'get')
        return render_template('login.html')
    else:
        print(request.view_args, 'post')
        username = request.form.get('username')
        password = request.form.get('password')
        if username == 'a' and password == 'aaa':
            session['userinfo'] = 'jiangyx'
            return 'hello'
        return '用户名或错误'

# -----------------------结果
{'username': 'jiangyx'} get
{'username': 'jiangyx'} post
```

### request.json

如果当前请求的`MIME`类型是json相关的，那么就会将json数据反序列化。源码中提示我们应该直接使用`request.get_json()`来代替`reqyest.json` 。

```python
@property
def json(self):
    """If the mimetype is :mimetype:`application/json` this will contain the
    parsed JSON data.  Otherwise this will be ``None``.

    The :meth:`get_json` method should be used instead.
    """
    from warnings import warn
    warn(DeprecationWarning('json is deprecated.  '
                            'Use get_json() instead.'), stacklevel=2)
    return self.get_json()
```

### request.is_json

判断当前请求对应的`MIME`类型是否是`application/json`或者以`application/`开头，以`+json`结尾的`MIME`类型 

```python
def is_json(self):
    """Indicates if this request is JSON or not.  By default a request
    is considered to include JSON data if the mimetype is
    :mimetype:`application/json` or :mimetype:`application/*+json`.
    """
    mt = self.mimetype
    if mt == 'application/json':
        return True
    if mt.startswith('application/') and mt.endswith('+json'):
        return True
    return False
```

### request.form

获取前端`form`表单`POST`提交的数据。返回一个`ImmutableMultiDict`对象，可以使用该对象的`get(key)`获取`value`

### request.querystring

获取前端`GET`请求发送的数据，但是返回的是一个字符串

### request.args

获取前端`GET`请求发送的数据，是一个`ImmutableMultiDict`对象

### request.values

获取前端`GET 和 POST`发送的数据，是一个`ImmutableMultiDict`对象



## request常用方法

### request.get_json()

作用同`request.json`，如果提交的数据是`json`格式的，就会对数据反序列化并返回 ，在其源码中，本质上就是调用了`json`模块的`loads`方法 

```python
def get_json(self, force=False, silent=False, cache=True):
    """Parses the incoming JSON request data and returns it.  By default
    this function will return ``None`` if the mimetype is not
    :mimetype:`application/json` but this can be overridden by the
    ``force`` parameter. If parsing fails the
    :meth:`on_json_loading_failed` method on the request object will be
    invoked.

    :param force: if set to ``True`` the mimetype is ignored.
    :param silent: if set to ``True`` this method will fail silently
                   and return ``None``.
    :param cache: if set to ``True`` the parsed JSON data is remembered
                  on the request.
    """
    rv = getattr(self, '_cached_json', _missing)
    # We return cached JSON only when the cache is enabled.
    if cache and rv is not _missing:
        return rv

    if not (force or self.is_json):
        return None

    request_charset = self.mimetype_params.get('charset')
    try:
        data = _get_data(self, cache)
        if request_charset is not None:
            rv = json.loads(data, encoding=request_charset)
        else:
            rv = json.loads(data)
    except ValueError as e:
        if silent:
            rv = None
        else:
            rv = self.on_json_loading_failed(e)
    if cache:
        self._cached_json = rv
    return rv
```



***

# response对象

## Flask中返回响应的几种方式

### render_template 

`render_template(template_name_or_list, **context) ` ,这个方法会将参数中指定的页面渲染成`html`，然后将页面数据封装进响应体中返回 。

```python
def render_template(template_name_or_list, **context):
    """Renders a template from the template folder with the given
    context.

    :param template_name_or_list: the name of the template to be
                                  rendered, or an iterable with template names
                                  the first one existing will be rendered
    # 一个模板的名称或者存放了多个模板名称的可迭代对象，当提供了多个模板的时候，该方法会遍历该可迭代对	     # 象，并将第一个有效的(存在的)模板渲染
    :param context: the variables that should be available in the
                    context of the template.
    """
    ctx = _app_ctx_stack.top
    ctx.app.update_template_context(context)
    return _render(ctx.app.jinja_env.get_or_select_template(template_name_or_list),
                   context, ctx.app)
```

### redirect

`redirect(location, code=302, Response=None) ` 这个方法会返回一个`response响应对象`，本质上就是一个`wsgi application（即我们的app）`，当调用该方法的时候，会重定向客户端到指定的ur路径 。

code可以支持`302 303 305 307`，但是不支持`300 304`，因为对于300来说，它不是真正意义上的重定向，此外这个方法除了给客户端返回了一个`302`重定向状态码之外，他还在响应头中添加了location。

```python
def redirect(location, code=302, Response=None):
 
    if Response is None:
        # 如果我们没有指定resposne参数，flask会自动为我们创建一个from werkzeug.wrappers import 		    # Response响应对象
        from werkzeug.wrappers import Response

    display_location = escape(location)
    if isinstance(location, text_type):
        # Safe conversion is necessary here as we might redirect
        # to a broken URI scheme (for instance itms-services).
        from werkzeug.urls import iri_to_uri
        location = iri_to_uri(location, safe_conversion=True)
    response = Response(
        '<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">\n'
        '<title>Redirecting...</title>\n'
        '<h1>Redirecting...</h1>\n'
        '<p>You should be redirected automatically to target URL: '
        '<a href="%s">%s</a>.  If not click the link.' %
        (escape(location), display_location), code, mimetype='text/html')
    response.headers['Location'] = location
    return response
```

### 直接使用字符串

如果在视图函数中直接返回一个字符串，Flas会自动的将这个字符转换成响应对象

### make_response 

有时候我们需要对响应对象添加一些必要响应头信息，并且有时候视图函数并不是都直接返回一个响应对象，也有可能返回一个值（比字符串），如果我们想要在视图函数中给响应添加一些响应头信息，我们需要通过`make_response`将字符串或其他对象使用`make_response`方法生成响应，为该响应对象添加了响应头信息之后再作为返回值返回 。

```python
def make_response(*args):
    if not args:
        return current_app.response_class()
    if len(args) == 1:
        args = args[0]
    return current_app.make_response(args)
```

### jsonfy 

返回一个响应体内容为`json`格式的响应。 

jsonf本质上其实是指向了json.jsonify()，这个方法返回的是一个`ClientResponse(默认)`对象

```python
jsonify(*args, **kwargs)
该函数的参数将被转换成json格式放在响应的响应体中
返回响应头信息中的MIME类型将会被设置成application/json
如果提供了单个参数，那么就会使用json.dumps直接对该对象进行处理
如果提供了多个对象，就会将该对象先组装，再使用json.dumps处理
	当多个对象是键值对的形式时，将会被组装成字典
    当多个对象是单独的value形式时，将会把多个value组装成一个列表
    比如jsonfy(1, 2, 3)和jsonfy([1, 2, 3])将会被组装成相同的"[1, 2, 3]"
```

```python
def jsonify(*args, **kwargs):

    indent = None
    separators = (',', ':')

    if current_app.config['JSONIFY_PRETTYPRINT_REGULAR'] and not request.is_xhr:
        indent = 2
        separators = (', ', ': ')

    if args and kwargs:
        raise TypeError('jsonify() behavior undefined when passed both args and kwargs')
    elif len(args) == 1:  # single args are passed directly to dumps()
        data = args[0]
    else:
        data = args or kwargs

    return current_app.response_class(                                          # 看这里！返回的是一个响应对象
        (dumps(data, indent=indent, separators=separators), '\n'),
        mimetype=current_app.config['JSONIFY_MIMETYPE']
    )


# 进入返回值对应的response_class继续看，你会发现
class ClientRequest:
    # 省略部分
    self.response_class = response_class or ClientResponse      
    
```

