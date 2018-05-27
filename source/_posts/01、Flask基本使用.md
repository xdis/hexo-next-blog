---
title: Flask基本使用
copyright: true
date: 2018-05-20 20:16:17
tags: [Python, Flask]
categories: Flask深入学习
---
# 简介

Flask是一个基于Python开发并且依赖jinja2模板和Werkzeug WSGI服务的一个微型框架。对于Werkzeug本质是Socket服务端，其用于接收http请求并对请求进行预处理，然后触发Flask框架，开发人员基于Flask框架提供的功能对请求进行相应的处理，并返回给用户，如果要返回给用户复杂的内容时，需要借助jinja2模板来实现对模板的处理，即：将模板和数据进行渲染，将渲染后的字符串返回给用户浏览器。

“微”(micro) 并不表示你需要把整个 Web 应用塞进单个 Python 文件（虽然确实可以 ），也不意味着 Flask 在功能上有所欠缺。微框架中的“微”意味着 Flask 旨在保持核心简单而易于扩展。Flask 不会替你做出太多决策——比如使用何种数据库。而那些 Flask 所选择的——比如使用何种模板引擎——则很容易替换。除此之外的一切都由可由你掌握。如此，Flask 可以与您珠联璧合。

默认情况下，Flask 不包含数据库抽象层、表单验证，或是其它任何已有多种库可以胜任的功能。然而，Flask 支持用扩展来给应用添加这些功能，如同是 Flask 本身实现的一样。众多的扩展提供了数据库集成、表单验证、上传处理、各种各样的开放认证技术等功能。Flask 也许是“微小”的，但它已准备好在需求繁杂的生产环境中投入使用。 

# 快速入门

## 安装Flask

在Python中项目中，为了防止项目的包管理冲突，通常采用的是一个项目一个环境的形式。因此Flask的安装步骤可以为：

``` shell
# 在项目根目录
python -m venv flask-demo
# 激活环境变量之后
pip install flask
```

## Flask最简案例

``` python
# app.py
from flask import Flask

app = Flask(__name__)


@app.route('/')
def index():
    return "Hello Flask"


if __name__ == '__main__':
    app.run()
```

在命令行中运行

``` shell
python app.py
```

访问[http://127.0.0.1:5000/](http://127.0.0.1:5000/) , 就能在网页上看到Hello Flask。这样一个最简的Flask案例就完成了。

# app.run()探索

在上面的例子中可以看到，当调用app对象的run方法后，Flask就会帮我们起一个开发专用的服务器，其源码如下

``` python
def run(self, host=None, port=None, debug=None, **options):
    from werkzeug.serving import run_simple
    if host is None:
        host = '127.0.0.1'
    if port is None:
        server_name = self.config['SERVER_NAME']
        if server_name and ':' in server_name:
            port = int(server_name.rsplit(':', 1)[1])
        else:
            port = 5000
    if debug is not None:
        self.debug = bool(debug)
    options.setdefault('use_reloader', self.debug)
    options.setdefault('use_debugger', self.debug)
    try:
        run_simple(host, port, self, **options)
    finally:
        self._got_first_request = False
```

可以看出在调用时，Flask会依次对host、port、debug等进行赋值。对于options参数，Flask会进行```use_reloader``` 和 ```use_debugger``` 的设置，这会影响到请求日志的打印和热加载。最后的run_simple方法其实就是使用werkzeug提供的方法启动的一个服务器，我们不使用其实也可以利用werkzeug写一个小型自定义web框架。

```python
from werkzeug.wrappers import Request, Response


@Request.application
def index(request):
    if request.path == '/':
        return Response('Hello World', 200)


if __name__ == '__main__':
    from werkzeug.serving import run_simple
    run_simple('127.0.0.1', 4000, index)
```

