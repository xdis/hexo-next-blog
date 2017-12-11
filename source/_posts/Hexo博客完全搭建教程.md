---
title: Hexo博客完全搭建教程
copyright: true
date: 2017-12-11 20:16:17
tags: [教程]
categories: 博客
---
# 配置仓库

## 准备工作

* 有一个github账号
* 安装了node.js、npm
* 安装了git for windows

## 创建仓库

新建一个名为 你的```用户名.github.io```的仓库。
比如说，如果github用户名是test，那么就新建 ```test.github.io``` 的仓库（必须是用户名，其它名称无效），将来网站访问地址就是 http://test.github.io 

几个注意的地方：

> 1. 注册的邮箱一定要验证，否则不会成功；
> 2. 仓库名字必须是： ```username.github.io ```，其中 ```username``` 是你的用户名；
> 3. 仓库创建成功不会立即生效，需要过一段时间；

## 配置SSH KEY

> 使用ssh key来解决本地和服务器的连接问题。

检查本机已存在的ssh密钥
```git
$ cd ~/. ssh
```
如果提示：No such file or directory 说明你是第一次使用git。
```git
ssh-keygen -t rsa -C "邮件地址"
```
然后连续3次回车，最终会生成一个文件在用户目录下，打开用户目录，找到 .ssh\id_rsa.pub 文件，记事本打开并复制里面的内容，打开你的github主页，进入个人设置 -> SSH and GPG keys -> New SSH key：

```git
$ ssh -T git@github.com # 测试配置是否成功, 注意邮箱地址不用改
```
如果提示 ```Are you sure you want to continue connecting (yes/no)?``` ，输入yes，然后会看到：

> Hi liuxianan! You’ve successfully authenticated, but GitHub does not provide shell access.

看到这个信息说明SSH已配置成功！

此时你还需要配置：

```git
$ git config --global user.name "username"// 你的github用户名，非昵称
$ git config --global user.email  "email"// 填写你的github注册邮箱
```

# 配置HEXO

## 全局安装

```bash
npm install -g hexo
```

## 初始化

在电脑的某个地方新建一个名为hexo的文件夹（名字可以随便取），比如我的是 ```F:\Workspaces\hexo ```，由于这个文件夹将来就作为你存放代码的地方，所以最好不要随便放。

```bash
$ cd /f/Workspaces/hexo/
$ hexo init
```
在初始化后可以开启本地预览服务浏览内容

```shell
hexo s # 开启本地预览服务
```

接着可以访问[http://localhost:4000](http://localhost:4000)即可看到hexo初始化的一片Hello World文章，很多人会碰到浏览器一直在转圈但是就是加载不出来的问题，一般情况下是因为端口占用的缘故，因为4000这个端口太常见了，解决端口冲突问题即可。

## 常用hexo命令

### 常见命令

```shell
hexo new "postName" #新建文章
hexo new page "pageName" #新建页面
hexo generate #生成静态页面至public目录
hexo server #开启预览访问端口（默认端口4000，'ctrl + c'关闭server）
hexo deploy #部署到GitHub
hexo help  # 查看帮助
hexo version  #查看Hexo的版本
```

### 命令缩写

```shell
hexo n == hexo new
hexo g == hexo generate
hexo s == hexo server
hexo d == hexo deploy
```

### 组合命令

```shell
hexo s -g #生成并本地预览
hexo d -g #生成并上传
```



***

# 安装Next主题

## 下载主题

在终端窗口下，定位到 Hexo 站点目录下。使用 `Git` checkout 代码：

```shell
git clone https://github.com/iissnan/hexo-theme-next themes/next
```

## 启用主题

与所有 Hexo 主题启用的模式一样。 当 克隆/下载 完成后，打开 站点配置文件(_config.yml)， 找到 `theme` 字段，并将其值更改为 `next`。

 修改上述配置之后即可重启本地预览服务，即可看到主题更换成功。

# 上传博客

在上传之前需要确定以下配置

* ```ssh key```配置成功
* 配置站点文件中有关deploy部分

```shell
deploy:
  type: git
  repository: git@github.com:jiangyx3915/jiangyx3915.github.io.git
  branch: master
```

* 安装部署插件

```shell
npm install hexo-deployer-git --save
```

完成上述操作后执行命令

```shell
hexo g d #就会将本次有改动的代码全部提交，没有改动的不会：
```

***

# Hexo配置详解

## 网站配置

```shell
title           网站标题
subtitle        网站副标题
description     网站描述
author          您的名字
language        网站使用的语言
timezone        网站时区。Hexo 默认使用您电脑的时区。时区列表。比如说：America/New_York, Japan, 和 UTC 。
```

其中，description主要用于SEO，告诉搜索引擎一个关于您站点的简单描述，通常建议在其中包含您网站的关键词。author参数用于主题显示文章的作者。

## 网址配置

```shell
url                  网址     
root               网站根目录    
permalink         文章的永久链接格式  
permalink_defaults  永久链接中各部分的默认值    
```

## 目录配置

```shell
source_dir       资源文件夹，这个文件夹用来存放内容。     
public_dir       公共文件夹，这个文件夹用于存放生成的站点文件。 
tag_dir          标签文件夹  
archive_dir      归档文件夹  
category_dir     分类文件夹  
code_dir         Include code 文件夹   
i18n_dir         国际化（i18n）文件夹   
skip_render      跳过指定文件的渲染，您可使用 glob 表达式来匹配路径。
```

## 文章设置

```shell
new_post_name       新文章的文件名称    
default_layout      预设布局    
auto_spacing        在中文和英文之间加入空格    
titlecase           把标题转换为 title case   
external_link       在新标签中打开链接 
filename_case       把文件名称转换为 (1) 小写或 (2) 大写     
render_drafts       显示草稿    
post_asset_folder   启动 Asset 文件夹    
relative_link       把链接改为与根目录的相对位址  
future              显示未来的文章     
highlight           代码块的设置
```

## 分类标签设置

```shell
default_category    默认分类
category_map        分类别名    
tag_map             标签别名    
```

## 时间/日期格式设置

```shell
date_format     日期格式 
time_format     时间格式
```

## 分页设置

```shell
per_page    每页显示的文章量 (0 = 关闭分页功能)   10
pagination_dir  分页目录                        page
```

## 扩展设置

```shell
theme       当前主题名称。值为false时禁用主题
deploy      部署部分的设置
```

***

# Next主题个性化设置

## 在右上角或者左上角实现fork me on github

点击[这里](https://github.com/blog/273-github-ribbons)挑选自己喜欢的样式，并复制代码。 

然后粘贴刚才复制的代码到`themes/next/layout/_layout.swig`文件中(放在`<div class="headband"></div>`的下面)，并把`href`改为你的github地址 



## 添加RSS

切换到你的blog的根目录下 ，然后安装 Hexo 插件

```shell
npm install --save hexo-generator-feed
```

然后编辑站点配置文件在里面的末尾添加

```shell
# Extensions
## Plugins: http://hexo.io/plugins/
plugins: hexo-generate-feed
```

然后打开next主题文件夹里面的`_config.yml`,在里面配置为如下样子：(就是在`rss:`的后面加上`/atom.xml`,**注意**在冒号后面要加一个空格)

```shell
# Set rss to false to disable feed link.
# Leave rss as empty to use site's feed link.
# Set rss to specific value if you have burned your feed already.
rss: /atom.xml
```

配置完之后运行

```shell
hexo g
```

重新生成一次，你会在`./public` 文件夹中看到 `atom.xml` 文件。然后启动服务器查看是否有效，之后再部署到 Github 中。



## 添加动态背景

在主题配置文件中找到canvas_nest: false，把它改为canvas_nest: true



## 实现点击出现桃心效果

在网址输入如下```http://7u2ss1.com1.z0.glb.clouddn.com/love.js```

然后将里面的代码copy一下，新建`love.js`文件并且将代码复制进去，然后保存。将`love.js`文件放到路径`/themes/next/source/js/src`里面，然后打开`\themes\next\layout\_layout.swig`文件,在末尾（在前面引用会出现找不到的bug）添加以下代码：

```shell
<!-- 页面点击小红心 -->
<script type="text/javascript" src="/js/src/love.js"></script>
```



## 修改文章内链接文本样式

修改文件 `themes\next\source\css\_common\components\post\post.styl`，在末尾添加如下css样式

```shell
// 文章内链接文本样式
.post-body p a{
  color: #0593d3;
  border-bottom: none;
  border-bottom: 1px solid #0593d3;
  &:hover {
    color: #fc6423;
    border-bottom: none;
    border-bottom: 1px solid #fc6423;
  }
}
```

其中选择`.post-body` 是为了不影响标题，选择 `p` 是为了不影响首页“阅读全文”的显示样式,颜色可以自己定义。



## 修改文章底部的那个带#号的标签

修改模板`/themes/next/layout/_macro/post.swig`，搜索 `rel="tag">#`，将 # 换成`<i class="fa fa-tag"></i>`



## 在每篇文章末尾统一添加“本文结束”标记

在路径 `\themes\next\layout\_macro` 中新建 `passage-end-tag.swig` 文件,并添加以下内容

```shell
<div>
    {% if not is_index %}
        <div style="text-align:center;color: #ccc;font-size:14px;">-------------本文结束<i class="fa fa-paw"></i>感谢您的阅读-------------</div>
    {% endif %}
</div>
```

接着打开`\themes\next\layout\_macro\post.swig`文件，在`post-body` 之后， `post-footer` 之前添加如下画红色部分代码（post-footer之前两个div）

```shell
<div>
  {% if not is_index %}
    {% include 'passage-end-tag.swig' %}
  {% endif %}
</div>
```

然后打开主题配置文件（`_config.yml`),在末尾添加

```shell
# 文章末尾添加“本文结束”标记
passage_end_tag:
  enabled: true
```

完成以上设置之后，在每篇文章之后都会添加如上效果图的样子。



## 修改作者头像并旋转

打开`\themes\next\source\css\_common\components\sidebar\sidebar-author.styl`，在里面添加如下代码

```shell
.site-author-image {
  display: block;
  margin: 0 auto;
  padding: $site-author-image-padding;
  max-width: $site-author-image-width;
  height: $site-author-image-height;
  border: $site-author-image-border-width solid $site-author-image-border-color;

  /* 头像圆形 */
  border-radius: 80px;
  -webkit-border-radius: 80px;
  -moz-border-radius: 80px;
  box-shadow: inset 0 -1px 0 #333sf;

  /* 设置循环动画 [animation: (play)动画名称 (2s)动画播放时长单位秒或微秒 (ase-out)动画播放的速度曲线为以低速结束 
    (1s)等待1秒然后开始动画 (1)动画播放次数(infinite为循环播放) ]*/


  /* 鼠标经过头像旋转360度 */
  -webkit-transition: -webkit-transform 1.0s ease-out;
  -moz-transition: -moz-transform 1.0s ease-out;
  transition: transform 1.0s ease-out;
}

img:hover {
  /* 鼠标经过停止头像旋转 
  -webkit-animation-play-state:paused;
  animation-play-state:paused;*/

  /* 鼠标经过头像旋转360度 */
  -webkit-transform: rotateZ(360deg);
  -moz-transform: rotateZ(360deg);
  transform: rotateZ(360deg);
}

/* Z 轴旋转动画 */
@-webkit-keyframes play {
  0% {
    -webkit-transform: rotateZ(0deg);
  }
  100% {
    -webkit-transform: rotateZ(-360deg);
  }
}
@-moz-keyframes play {
  0% {
    -moz-transform: rotateZ(0deg);
  }
  100% {
    -moz-transform: rotateZ(-360deg);
  }
}
@keyframes play {
  0% {
    transform: rotateZ(0deg);
  }
  100% {
    transform: rotateZ(-360deg);
  }
}
```



## 博文压缩

在站点的根目录下执行以下命令

```shell
npm install gulp -g
npm install gulp-minify-css gulp-uglify gulp-htmlmin gulp-htmlclean gulp --save
```

在项目根目录新建`gulpfile.js` ，并填入以下内容

```shell
var gulp = require('gulp');
var minifycss = require('gulp-minify-css');
var uglify = require('gulp-uglify');
var htmlmin = require('gulp-htmlmin');
var htmlclean = require('gulp-htmlclean');
// 压缩 public 目录 css
gulp.task('minify-css', function() {
    return gulp.src('./public/**/*.css')
        .pipe(minifycss())
        .pipe(gulp.dest('./public'));
});
// 压缩 public 目录 html
gulp.task('minify-html', function() {
  return gulp.src('./public/**/*.html')
    .pipe(htmlclean())
    .pipe(htmlmin({
         removeComments: true,
         minifyJS: true,
         minifyCSS: true,
         minifyURLs: true,
    }))
    .pipe(gulp.dest('./public'))
});
// 压缩 public/js 目录 js
gulp.task('minify-js', function() {
    return gulp.src('./public/**/*.js')
        .pipe(uglify())
        .pipe(gulp.dest('./public'));
});
// 执行 gulp 命令时执行的任务
gulp.task('default', [
    'minify-html','minify-css','minify-js'
]);
```

生成博文是执行 `hexo g && gulp` 就会根据 `gulpfile.js` 中的配置，对 public 目录中的静态资源文件进行压缩。



## 修改代码块自定义样式

打开`\themes\next\source\css\_custom\custom.styl`,向里面加入：(颜色可以自己定义)

```shell
// Custom styles.
code {
    color: #ff7600;
    background: #fbf7f8;
    margin: 2px;
}
// 大代码块的自定义样式
.highlight, pre {
    margin: 5px 0;
    padding: 5px;
    border-radius: 3px;
}
.highlight, code, pre {
    border: 1px solid #d6d6d6;
}
```



## 侧边栏社交小图标设置

打开主题配置文件（`_config.yml`），搜索`social_icons:`,在[图标库](http://fontawesome.io/icons/)找自己喜欢的小图标，并将名字复制在如下位置，保存即可

![ICON](http://upload-images.jianshu.io/upload_images/5308475-21e22b05edc57b5b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



## 主页文章添加阴影效果

打开`\themes\next\source\css\_custom\custom.styl`,向里面加入

```shell
// 主页文章添加阴影效果
 .post {
   margin-top: 60px;
   margin-bottom: 60px;
   padding: 25px;
   -webkit-box-shadow: 0 0 5px rgba(202, 203, 203, .5);
   -moz-box-shadow: 0 0 5px rgba(202, 203, 204, .5);
  }
```



## 在网站底部加上访问量

打开`\themes\next\layout_partials\footer.swig`文件,在copyright前加上画红线这句话

```shell
<script async src="https://dn-lbstatics.qbox.me/busuanzi/2.3/busuanzi.pure.mini.js"></script>
```

然后再合适的位置添加显示统计的代码

```shell
<div class="powered-by">
<i class="fa fa-user-md"></i><span id="busuanzi_container_site_uv">
  本站访客数:<span id="busuanzi_value_site_uv"></span>
</span>
</div>
```

在这里有两中不同计算方式的统计代码： 

1. **pv**的方式，单个用户连续点击n篇文章，记录n次访问量
2. **uv**的方式，单个用户连续点击n篇文章，只记录1次访客数

添加之后再执行`hexo d -g`，然后再刷新页面就能看到效果



## 网站底部字数统计

切换到根目录下，然后运行如下代码

```shell
npm install hexo-wordcount --save
```

然后在`/themes/next/layout/_partials/footer.swig`文件尾部加上

```shell
<div class="theme-info">
  <div class="powered-by"></div>
  <span class="post-count">博客全站共{{ totalcount(site) }}字</span>
</div>
```



## 添加 README.md 文件

每个项目下一般都有一个 `README.md` 文件，但是使用 hexo 部署到仓库后，项目下是没有 `README.md` 文件的。

在 Hexo 目录下的 `source` 根目录下添加一个 `README.md` 文件，修改站点配置文件 _`config.yml`，将 `skip_render` 参数的值设置为

```shell
skip_render: README.md
```

保存退出即可。再次使用 `hexo d` 命令部署博客的时候就不会在渲染 README.md 这个文件了。



## 设置网站的图标

在[EasyIcon](http://www.easyicon.net/)中找一张（32*32）的`ico`图标,或者去别的网站下载或者制作，并将图标名称改为`favicon.ico`，然后把图标放在`/themes/next/source/images`里，并且修改主题配置文件

```shell
# Put your favicon.ico into `hexo-site/source/` directory.
favicon: /favicon.ico
```



## 实现统计功能

在根目录下安装 `hexo-wordcount`,运行

```shell
npm install hexo-wordcount --save
```

然后在主题的配置文件中，配置如下

```shell
# Post wordcount display settings
# Dependencies: https://github.com/willin/hexo-wordcount
post_wordcount:
  item_text: true
  wordcount: true
  min2read: true
```



## 添加顶部加载条

打开`/themes/next/layout/_partials/head.swig`文件，在meta后添加

```shell
<script src="//cdn.bootcss.com/pace/1.0.2/pace.min.js"></script>
<link href="//cdn.bootcss.com/pace/1.0.2/themes/pink/pace-theme-flash.css" rel="stylesheet">
```

但是，默认的是粉色的，要改变颜色可以在`/themes/next/layout/_partials/head.swig`文件中添加如下代码（接在刚才link的后面）

```shell
<style>
    .pace .pace-progress {
        background: #1E92FB; /*进度条颜色*/
        height: 3px;
    }
    .pace .pace-progress-inner {
         box-shadow: 0 0 10px #1E92FB, 0 0 5px     #1E92FB; /*阴影颜色*/
    }
    .pace .pace-activity {
        border-top-color: #1E92FB;    /*上边框颜色*/
        border-left-color: #1E92FB;    /*左边框颜色*/
    }
</style>
```

此外，如果是最新的next主题只需修改主题配置文件(_config.yml)将`pace: false`改为`pace: true`就行了，你还可以换不同样式的加载条



## 在文章底部增加版权信息

在目录 `next/layout/_macro/下`添加 `my-copyright.swig`

```shell
{% if page.copyright %}
<div class="my_post_copyright">
  <script src="//cdn.bootcss.com/clipboard.js/1.5.10/clipboard.min.js"></script>

  <!-- JS库 sweetalert 可修改路径 -->
  <script type="text/javascript" src="http://jslibs.wuxubj.cn/sweetalert_mini/jquery-1.7.1.min.js"></script>
  <script src="http://jslibs.wuxubj.cn/sweetalert_mini/sweetalert.min.js"></script>
  <link rel="stylesheet" type="text/css" href="http://jslibs.wuxubj.cn/sweetalert_mini/sweetalert.mini.css">
  <p><span>本文标题:</span><a href="{{ url_for(page.path) }}">{{ page.title }}</a></p>
  <p><span>文章作者:</span><a href="/" title="访问 {{ theme.author }} 的个人博客">{{ theme.author }}</a></p>
  <p><span>发布时间:</span>{{ page.date.format("YYYY年MM月DD日 - HH:MM") }}</p>
  <p><span>最后更新:</span>{{ page.updated.format("YYYY年MM月DD日 - HH:MM") }}</p>
  <p><span>原始链接:</span><a href="{{ url_for(page.path) }}" title="{{ page.title }}">{{ page.permalink }}</a>
    <span class="copy-path"  title="点击复制文章链接"><i class="fa fa-clipboard" data-clipboard-text="{{ page.permalink }}"  aria-label="复制成功！"></i></span>
  </p>
  <p><span>许可协议:</span><i class="fa fa-creative-commons"></i> <a rel="license" href="https://creativecommons.org/licenses/by-nc-nd/4.0/" target="_blank" title="Attribution-NonCommercial-NoDerivatives 4.0 International (CC BY-NC-ND 4.0)">署名-非商业性使用-禁止演绎 4.0 国际</a> 转载请保留原文链接及作者。</p>  
</div>
<script> 
    var clipboard = new Clipboard('.fa-clipboard');
    clipboard.on('success', $(function(){
      $(".fa-clipboard").click(function(){
        swal({   
          title: "",   
          text: '复制成功',   
          html: false,
          timer: 500,   
          showConfirmButton: false
        });
      });
    }));  
</script>
{% endif %}
```

在目录`next/source/css/_common/components/post/`下添加`my-post-copyright.styl`

```css
.my_post_copyright {
  width: 85%;
  max-width: 45em;
  margin: 2.8em auto 0;
  padding: 0.5em 1.0em;
  border: 1px solid #d3d3d3;
  font-size: 0.93rem;
  line-height: 1.6em;
  word-break: break-all;
  background: rgba(255,255,255,0.4);
}
.my_post_copyright p{margin:0;}
.my_post_copyright span {
  display: inline-block;
  width: 5.2em;
  color: #b5b5b5;
  font-weight: bold;
}
.my_post_copyright .raw {
  margin-left: 1em;
  width: 5em;
}
.my_post_copyright a {
  color: #808080;
  border-bottom:0;
}
.my_post_copyright a:hover {
  color: #a3d2a3;
  text-decoration: underline;
}
.my_post_copyright:hover .fa-clipboard {
  color: #000;
}
.my_post_copyright .post-url:hover {
  font-weight: normal;
}
.my_post_copyright .copy-path {
  margin-left: 1em;
  width: 1em;
  +mobile(){display:none;}
}
.my_post_copyright .copy-path:hover {
  color: #808080;
  cursor: pointer;
}
```

修改`next/layout/_macro/post.swig`，在代码

```html
<div>
      {% if not is_index %}
        {% include 'wechat-subscriber.swig' %}
      {% endif %}
</div>
```

之前添加增加如下代码

```html
<div>
      {% if not is_index %}
        {% include 'my-copyright.swig' %}
      {% endif %}
</div>
```

修改`next/source/css/_common/components/post/post.styl`文件，在最后一行增加代码：

```css
@import "my-post-copyright"
```

保存重新生成即可。 



## 添加来必力跟帖

在主题配置文件`_config.yml` 文件中添加如下配置

```shell
# Support for LiveRe comments system.
# You can get your uid from https://livere.com/insight/myCode (General web site)
livere_uid: your uid
```

然后在 `layout/_scripts/third-party/comments/` 目录中添加 livere.swig，文件内容如下

```html
{% if not (theme.duoshuo and theme.duoshuo.shortname) and not theme.duoshuo_shortname and not theme.disqus_shortname and not theme.hypercomments_id and not theme.gentie_productKey %}
  {% if theme.livere_uid %}
    <script type="text/javascript">
      (function(d, s) {
        var j, e = d.getElementsByTagName(s)[0];
        if (typeof LivereTower === 'function') { return; }
        j = d.createElement(s);
        j.src = 'https://cdn-city.livere.com/js/embed.dist.js';
        j.async = true;
        e.parentNode.insertBefore(j, e);
      })(document, 'script');
    </script>
  {% endif %}
{% endif %}
```

然后在 `layout/_scripts/third-party/comments.swig`文件中追加

```html
{% include './comments/livere.swig' %}
```

最后，在 `layout/_partials/comments.swig` 文件中条件最后追加 LiveRe 插件是否引用的判断逻辑

```shell
{% elseif theme.livere_uid %}
      <div id="lv-container" data-id="city" data-uid="{{ theme.livere_uid }}"></div>
{% endif %}
```



## 隐藏网页底部powered By Hexo / 强力驱动

打开`themes/next/layout/_partials/footer.swig`,使用””隐藏之间的代码即可，或者直接删除



## 文章加密访问

打开`themes->next->layout->_partials->head.swig`文件,在以下位置插入这样一段代码

```javascript
<script>
    (function(){
        if('{{ page.password }}'){
            if (prompt('请输入文章密码') !== '{{ page.password }}'){
                alert('密码错误！');
                history.back();
            }
        }
    })();
</script>
```

然后在文章上的头部出加上password: 密码



## 添加jiathis分享

在**主题配置文件**中，jiathis为true



## 侧边栏推荐阅读

打开主题配置文件修改

```shell
# Blogrolls
links_title: 推荐阅读
#links_layout: block
links_layout: inline
links:
  优设: http://www.uisdc.com/
  张鑫旭: http://www.zhangxinxu.com/
  Web前端导航: http://www.alloyteam.com/nav/
  前端书籍资料: http://www.36zhen.com/t?id=3448
  百度前端技术学院: http://ife.baidu.com/
  google前端开发基础: http://wf.uisdc.com/cn/
```



## 自定义鼠标样式

打开`themes/next/source/css/_custom/custom.styl`,在里面写下如下代码

```css
// 鼠标样式
  * {
      cursor: url("http://om8u46rmb.bkt.clouddn.com/sword2.ico"),auto!important
  }
  :active {
      cursor: url("http://om8u46rmb.bkt.clouddn.com/sword1.ico"),auto!important
  }
```

其中 url 里面必须是 ico 图片，ico 图片可以上传到网上（我是使用七牛云图床），然后获取外链，复制到 url 里就行了



## 博客自定义域名

在博客的source目录下创建一个CNAME文件，其中写入需要访问的域名

```shell
ping jiangyx3915.github.io
```

获取到具体的域名

在域名解析中进行解析设置即可