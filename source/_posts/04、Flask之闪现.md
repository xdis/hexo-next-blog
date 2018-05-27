---
title: Flask之闪现
copyright: true
date: 2018-05-27 19:29:17
tags: [Python, Flask]
categories: Flask深入学习
---
# 简介

Flask 提供了一个非常简单的方法来使用闪现系统向用户反馈信息。闪现系统使得在一个请求结束的时候记录一个信息，然后在且仅仅在下一个请求中访问这个数据。 

# 基本使用

使用Flask中的闪现非常简单，只要通过`flash()` ，进行设置以及`get_flashed_messages`方法进行读取即可

## 存入

``` python
from flask import Flask, flash, get_flashed_messages

app = Flask(__name__)
app.config['SECRET_KEY'] = '123456'


@app.route(rule='/<int:nid>')
def index(nid):
    # 可以多次使用推入多个值
    flash(str(nid))
    return "index"
```

由于flash实际是基于session来实现的，因此不会出现不同用户之间的闪现有相互的冲突。因为要使用到session，因此必须配置SECRET_KEY

``` python
def flash(message, category='message'):
    # 可以看出flash中存入的消息其实是一个数组，因此可以推入多个值
    flashes = session.get('_flashes', [])
    flashes.append((category, message))
    session['_flashes'] = flashes
    message_flashed.send(current_app._get_current_object(),
                         message=message, category=category)
```

## 读取

``` python
@app.route('/get')
def get():
    a = get_flashed_messages()
    print(a)
    return "true"
```

一旦使用了`get_flashed_message`方法，则之前设置的所有flash都会被清空，其源码为

```python
def get_flashed_messages(with_categories=False, category_filter=[]):

    flashes = _request_ctx_stack.top.flashes
    if flashes is None:
        # 从session中弹出并清空
        _request_ctx_stack.top.flashes = flashes = session.pop('_flashes') \
            if '_flashes' in session else []
    if category_filter:
        flashes = list(filter(lambda f: f[0] in category_filter, flashes))
    if not with_categories:
        return [x[1] for x in flashes]
    return flashes

```

# 分类闪现

当闪现一个消息时，是可以提供一个分类的。未指定分类时默认的分类为 `message` 。 可以使用分类来提供给用户更好的反馈。

``` python
@app.route(rule='/<int:nid>')
def index(nid):
    # 写入不同种类的闪现
    flash('test', 'type')
    flash('test2', 'type2')
    flash(str(nid), 'ids')
    return "true"

@app.route('/get')
def geto():
    a = get_flashed_messages(with_categories=False)
    # ['test', 'test2', '11', 'test', 'test2', '22']
    print(a)
    a = get_flashed_messages(with_categories=True)
    print(a)
    # [('type', 'test'), ('type2', 'test2'), ('ids', '11'), ('type', 'test'), ('type2', 'test2'), ('ids', '22')]
    return "true"
```

# 过滤闪现

可以将一个分类的列表传入到 [`get_flashed_messages()`](http://docs.jinkan.org/docs/flask/api.html#flask.get_flashed_messages) 中， 以过滤函数返回的结果 

```python
@app.route(rule='/<int:nid>')
def index(nid):
    # 写入不同种类的闪现
    flash('test', 'type')
    flash('test2', 'type2')
    flash(str(nid), 'ids')
    return "true"

@app.route('/get')
def geto():
    a = get_flashed_messages(category_filter=['ids'])
    print(a)
    # ['11', '22']
    a = get_flashed_messages(category_filter=['ids', 'type'])
    # ['test', '11', 'test', '22']
    print(a)
    return "true"
```

# 在模板中的使用

flash在模板中的使用和在Python代码中使用一致

``` jinja2
# 普通使用
{% with messages = get_flashed_messages() %}
  {% if messages %}
    <ul class=flashes>
    {% for message in messages %}
      <li>{{ message }}</li>
    {% endfor %}
    </ul>
  {% endif %}
{% endwith %}

# 使用分类闪现
{% with messages = get_flashed_messages(with_categories=true) %}
  {% if messages %}
    <ul class=flashes>
    {% for category, message in messages %}
      <li class="{{ category }}">{{ message }}</li>
    {% endfor %}
    </ul>
  {% endif %}
{% endwith %}

# 使用过滤闪现
{% with errors = get_flashed_messages(category_filter=["error"]) %}
{% if errors %}
<div class="alert-message block-message error">
  <a class="close" href="#">×</a>
  <ul>
    {%- for msg in errors %}
    <li>{{ msg }}</li>
    {% endfor -%}
  </ul>
</div>
{% endif %}
{% endwith %}
```

