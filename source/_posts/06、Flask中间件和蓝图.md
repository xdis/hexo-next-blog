---
title: Flask中间件和蓝图
copyright: true
date: 2018-05-31 20:16:17
tags: [Python, Flask]
categories: Flask深入学习
---
# 中间件

Flask中除了请求扩展，还可以自己实现中间件来对所有请求做一层拦截。从源码中可知请求的完整执行流程是由Flask对象中的`wsgi_app`方法中完成的

```python
def wsgi_app(self, environ, start_response):
    ctx = self.request_context(environ)
    ctx.push()
    error = None
    try:
        try:
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
```

因此我们可以通过以下方式为Flask添加中间件

```python
class MyMiddleWare:
    def __init__(self, old_wsgi_app):
        self.old_wsgi_app = old_wsgi_app
        
    def __call__(self, environ, start_response):
        print('请求开始之前')
        # 通过调用原始的wsgi_app方法
        ret = self.old_wsgi_app(environ, start_response)
        print('响应结束之后')
        return ret
```

```python
app = Flask(__name__)
# 通过覆盖原来的wsgi_app的方法，来进行中间的添加
app.wsgi_app = MyMiddleWare(app.wsgi_app)
app.run()
```

***

# 蓝图

我们为什么要使用蓝图？一个项目是由很多个功能组成的，比如对于微信来说，聊天是他的一个功能，朋友圈也是他的一个功能，以及支付等，如果要把一个项目的所有功能全部写在一个`.py`文件中，即使实现了这些功能，在后期维护的时候也会非常麻烦，我们应该将项目中的功能进行分类，不同应用的功能放在不同的文件中。

 对于Flask，由于他的路由系统本质上是装饰器的形式，而调用装饰器的时候需要使用`当前应用`来调用，即`app.route` 。

## 不使用蓝图进行功能拆分

按上面所述，蓝图作用其实是一个帮助我们合理拆分项目的结构，那么不使用蓝图来进行拆分是否可以呢？答案是肯定的，我们可以按以下操作。

1、创建一个包，在其`__init__.py`文件中实例化Flask对象

``` python
# __init__.py
from flask import Flask

app = Flask(__name__)

```

2、在包中根据功能模块创建相应的`py`文件

```python
# user.py

from . import app

@app.route('/user')
def user():
    return 'user'
```

3、回到`__init__.py`文件中注册`user`

```python
# __init__.py
from flask import Flask

app = Flask(__name__)

from views import user
```

4、在项目根目录创建项目启动文件

```python
# app.py
from views import app

if __name__ == '__main__':
    app.run()
```

这样就完成了项目的拆分，拆分后的目录结构为

```python
views
	__init__.py
    user.py
    
app.py
```

### 循环导入问题

在第一次做模块拆分时，新手很容易将项目拆分成以下情况，这是很常见的循环导入问题

```python
# app.py

from flask import Flask

app = Flask(__name__)
print(id(app))

from views import user


if __name__ == '__main__':
    print(id(app))
    app.run()
```

```python
# user.py

from app import app

@app.route('/user')
def user():
    return 'user'
```

上述代码乍看没有问题，但是当启动后访问却会报错，接下来看下流程图

![2018-05-31_212446.jpg](https://i.loli.net/2018/05/31/5b0ff7e64c674.jpg)

从上述流程图中可以看出实际上在`app.py`中的`from views import user`这段代码其实又执行了一遍Flask对象的实例化，因此路由是注册到了新的app对象上，但是因为第二次执行发生在模块导入的时候，因此`__name__ != __main__`的，总的来说，启动和注册路由的app实例不是同一个。

***

## 使用蓝图进行功能拆分

1、创建项目文件夹

首先创建一个项目文件夹，一般为项目的名称，这里命名为`demo`

2、创建启动文件

在`demo`目录下创建一个项目启动文件，这里命名为`manage.py`

3、创建模块文件夹

在项目目录创建一个模块文件夹 `apps`，在其中创建一个`__init__.py`，在这个文件内部实例化Flask对象，实例化过程中，Flask的父类`_PackageBoundObject `会调用`get_root_path() `来获取`root_path`，在本例中`root_path = demo/apps`

```python
from flask import Flask

app = Flask(__name__)
```

4、创建模块文件，引入蓝图

在`demo/apps`下创建模块文件，文件内倒入蓝图对象并实例化，接着创建视图函数。在配置路由时，需要使用实例化的`blueprint`对象调用`route`装饰器。

``` python
# user.py

from flask.blueprints import Blueprint
from flask import render_template

user = Blueprint('user', url_prefix='/user', import_name=__name__)


@user.route('/')
def user_index():
    return render_template('user/index.html')
```

``` python
# order.py

from flask.blueprints import Blueprint
from flask import render_template

order = Blueprint('order',  url_prefix='/order', import_name=__name__, template_folder='templates/order/')


@order.route('/')
def order_index():
    return render_template('index.html')
```

5、注册蓝图

在`__init__.py`中注册蓝图，要注意，**你在蓝图所在的文件中定义的视图函数名称不要和蓝图的名称重复，因为两者在全局命名空间中，这样做会覆盖蓝图的变量名的指向 ，推荐使用  蓝图名_函数名 来进行命名**

```python
from flask import Flask
from .user import user
from .order import order

app = Flask(__name__)

app.register_blueprint(user)
app.register_blueprint(order)
```

6、启动项目

```python

```

7、项目目录

```python
├─apps
│  ├─templates
│  │  ├─order
│  │  └─user
│  ├─ order.py
│  ├─ user.py
├─manage.py
```

## 关于静态文件和模板文件的路径配置 

- `templates`和`static`时flask默认用来存放`模板文件`和`静态文件`的位置，实际上，flask在寻找`templates`和`static`目录是根据实例化时提供的`root_path`寻找的。
- 在实例化蓝图时，你可以不指定`template_folder`和`static_folder`，但是必须明确指定你的静态文件或模板在`static/templates`中的确切位置 。比如在上例中，在`templates`中分别为`account`和`order`蓝图创建了各自存放模板的路径，如果不指定`template_folder`，在视图函数中，模板的位置就得这样写`templates/account/ziawang.html`，如果你指定了`templates_folder="templates/order/"`，那么在`order`蓝图的视图函数中，对于`templates/order`下的模板文件，你只需要写`order.html`即可

## 蓝图项目目录结构介绍

### 中小型项目

```
E:\PRO_FLASK
│  run.py
│
└─pro_flask
    │  __init__.py
    │
    ├─statics
    │      code.png
    │
    ├─templates
    │      login.html
    │
    ├─views
       │  account.py
       │  blog.py
       │  user.py
  
# 这里你也可以将静态文件和模板文件向上面实例一样划分结构
```

### 大型项目

```
E:\PRO_FLASK
│  run.py                   # app.run()
│
└─pro_flask
    │  __init__.py          # Flask()
    │
    ├─admin                 # admin 应用
    │  │  views.py
    │  │  __init__.py
    │  │
    │  ├─static
    │  ├─templates
    │
    ├─web                   # web应用
       │  views.py
       │  __init__.py
       │
       ├─static
       ├─templates
```

[大型项目目录实例](http://p0lgavykh.bkt.clouddn.com/pro_flask_%E5%A4%A7%E5%9E%8B%E5%BA%94%E7%94%A8%E7%9B%AE%E5%BD%95%E7%A4%BA%E4%BE%8B.zip)

[中小型项目目录实例](http://p0lgavykh.bkt.clouddn.com/pro_flask_%E7%AE%80%E5%8D%95%E5%BA%94%E7%94%A8%E7%A8%8B%E5%BA%8F%E7%9B%AE%E5%BD%95%E7%A4%BA%E4%BE%8B.zip)