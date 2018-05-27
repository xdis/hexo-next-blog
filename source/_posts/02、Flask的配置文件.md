---
title: Flask配置文件
copyright: true
date: 2018-05-25 20:16:17
tags: [Python, Flask]
categories: Flask深入学习
---
# 为什么需要配置

一般来说应用需要一个配置文件来记载相关配置，例如数据库的配置，各种第三方插件库的配置，分页信息等等。Flask被设计成在应用启动时会读取配置文件。你可以硬编码在代码中进行配置，这种方式在一些小型应用上是可以使用的，但是有更好的方式。

# Flask的基本配置

我们可以在实例化Flask后加载Flask的配置，最简单的方式如下：

```python
app = Flask(__name__)
app.config['TESTING'] = True
```

在配置加载后，你可以在应用的任何地方读取或修改Flask的配置

```python
# 读取Flask的配置
print(app.testing)

# 设置Flask的配置
app.testing = False

# 同时更新多个Flask配置
app.config.update(
	TESTING=True,
    SECRET_KEY='Flask密匙'
)
```

# Flask默认配置加载的流程

首先在Flask初始化时，Flask会进行配置的加载

```python
def __init__(self, import_name, static_path=None, static_url_path=None,
             static_folder='static', template_folder='templates',
             instance_path=None, instance_relative_config=False,
             root_path=None):
	.......
    # Flask的配置实际上是Config类。
    # 它的行为和普通的字段完全相同
    # 但是它支持其他方法来从文件中加载配置
    self.config = self.make_config(instance_relative_config)
```

从这可以看出Flask是利用make_config方法来加载配置文件并赋给Flask实例对象的config属性

接下来我们来看看make_config方法

```python
def make_config(self, instance_relative=False):
    root_path = self.root_path
    if instance_relative:
        root_path = self.instance_path
    return self.config_class(root_path, self.default_config)
```

在这个方法中由两个地方的路径需要考虑`self.root_path` 和`self.instance_path`。

## self.root_path的加载 

接下来我们先看```self.root_path``` ，看源码可知Flask类中并没有定义root_path的属性，那么root_path是如何获取的呢？我们来看下Flask类的部分定义

```python
class Flask(_PackageBoundObject):
    def __init__(self, import_name, static_path=None, static_url_path=None,
             static_folder='static', template_folder='templates',
             instance_path=None, instance_relative_config=False,
             root_path=None):
    _PackageBoundObject.__init__(self, import_name,
                                 template_folder=template_folder,
                                 root_path=root_path)
```

从上述代码中可以看出，Flask类是继承`_PackageBoundObject`类的，并在初始化时调用了父类的初始化方法，父类中的`__init__`代码如下：

```python
class _PackageBoundObject(object):

    def __init__(self, import_name, template_folder=None, root_path=None):
        #: The name of the package or module.  Do not change this once
        #: it was set by the constructor.
        self.import_name = import_name

        #: location of the templates.  ``None`` if templates should not be
        #: exposed.
        self.template_folder = template_folder

        if root_path is None:
            root_path = get_root_path(self.import_name)

        #: Where is the app root located?
        self.root_path = root_path

        self._static_folder = None
        self._static_url_path = None
```

从上可知Flask中的root_path属性是继承于其父类`_PackageBoundObject`，`get_root_path(self.import_name)` 就是获取root_path的方法

```python
def get_root_path(import_name):
    mod = sys.modules.get(import_name)
    if mod is not None and hasattr(mod, '__file__'):
        return os.path.dirname(os.path.abspath(mod.__file__))
    ......
```

从上面的整个流程可以看出root_path其实就是我们Flask项目启动文件存放的父文件夹路径。

## self.instance_path的加载

```python
if instance_relative:
    root_path = self.instance_path
```

从代码可知如果传入的参数instance_relative为True，那么root_path会被替换成self.instance_path，而instance_relative是由实例化Flask对象时传入的`instance_relative_config` 决定的。那么这个参数有何作用？

```python
# Flask.__init__
if instance_path is None:
    instance_path = self.auto_find_instance_path()
```

```python
def auto_find_instance_path(self):
    """Tries to locate the instance path if it was not provided to the
    constructor of the application class.  It will basically calculate
    the path to a folder named ``instance`` next to your main file or
    the package.

    .. versionadded:: 0.8
    """
    # 找到起始核心文件所处的文件夹路径
    prefix, package_path = find_package(self.import_name)
    if prefix is None:
        return os.path.join(package_path, 'instance')
    return os.path.join(prefix, 'var', self.name + '-instance')
```

从上述代码可以看出如果在Flask对象初始化时传入instance_relative_config=True的话，那么配置文件的路径会从instance_path中读取，如果instance_path为Node则会默认配置文件夹名称为`instance`

## 生成Config类的实例

从上述代码可看出，make_config方法根据root_path和default_config生成Config类的实例，self.config_class 其实指代的是Config类的引用

```python
# 定义引用
config_class = Config

# 实例化一个Config类
config_class(root_path, default_config)
```

# Flask加载自定义配置

上述流程介绍的仅仅是Flask加载默认配置的流程。现在的Flask实例对象中就已经存在config属性了，接下来我们来看看Flask如何更新我们自定义的配置。

## 通过字典的设置方式设置

```python
app.config['DEBUG'] = True
```

由于Config对象本质上是字典，因此可以直接使用此方式，还可以使用app.config.update(...)来更新多个配置

## 通过from_pyfile

```python
 def from_pyfile(self, filename, silent=False):
    # 根据先前传入的root_path获取到配置文件的全路径
    filename = os.path.join(self.root_path, filename)
    d = types.ModuleType('config')
    d.__file__ = filename
    try:
        # 读取配置文件，并将其转为Python代码并执行
        with open(filename, mode='rb') as config_file:
            exec(compile(config_file.read(), filename, 'exec'), d.__dict__)
    except IOError as e:
        if silent and e.errno in (errno.ENOENT, errno.EISDIR):
            return False
        e.strerror = 'Unable to load configuration file (%s)' % e.strerror
        raise
    self.from_object(d)
    return True

 def from_object(self, obj):
    if isinstance(obj, string_types):
        obj = import_string(obj)
    for key in dir(obj):
        # 可以看出，配置项必须为大写，不然Flask是无法识别去更新的
        if key.isupper():
            self[key] = getattr(obj, key)

```

from_pyfile更新的逻辑大致就是

1. 读取配置文件的文本内容。
2. 将文本内容编译成字节码并执行，并将结果赋值给`types.ModuleType('config')`实例化出来的属性。
3. 遍历其属性，如果属性名是大写，就进行添加或覆盖。

## 通过from_json

```python
def from_json(self, filename, silent=False):
    filename = os.path.join(self.root_path, filename)

    try:
        with open(filename) as json_file:
            obj = json.loads(json_file.read())
    except IOError as e:
        if silent and e.errno in (errno.ENOENT, errno.EISDIR):
            return False
        e.strerror = 'Unable to load configuration file (%s)' % e.strerror
        raise
    return self.from_mapping(obj)

def from_mapping(self, *mapping, **kwargs):
    mappings = []
    if len(mapping) == 1:
        if hasattr(mapping[0], 'items'):
            mappings.append(mapping[0].items())
        else:
            mappings.append(mapping[0])
    elif len(mapping) > 1:
        raise TypeError(
            'expected at most 1 positional argument, got %d' % len(mapping)
        )
    mappings.append(kwargs.items())
    for mapping in mappings:
        for (key, value) in mapping:
            if key.isupper():
                self[key] = value
    return True
```

其主要逻辑和from_pyfile差不多，只是from_json做了json.loads的操作。

## 通过from_object

这个方式是最常用并且Flask是推荐使用的，使用方式如下

```python
# config.py

class Config(object):
    """
    基础配置
    """
    DEBUG = False
    TESTING = False
    DATABASE_URI = 'sqlite://memory'


class ProductionConfig(Config):
    """
    生产环境下的配置文件
    """
    DATABASE_URI = 'mysql://user@localhost/foo'


class DevelopmentConfig(Config):
    """
    开发环境的配置文件
    """
    DEBUG = True


class TestingConfig(Config):
    """
    测试环境下的配置文件
    """
    TESTING = True

```

可以定义不同环境下的配置，方便互相切换而不用大量更改代码。

```python
# 使用方式为app.config.from_object("文件名.类名")
app.config.from_object("config.DevelopmentConfig")
```

# 总结

1. 配置文件通常使用from_object的方式，方便在不同环境切换不同的配置文件。
2. settings.py文件默认路径要放在程序root_path目录
3. 如果instance_relative_config为True，则就是instance_path目录