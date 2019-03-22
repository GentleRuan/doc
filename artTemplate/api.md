## 一、为什么要这么做？

   纯前后端分离，更多的关注交互的实现， 模块化管理，易维护，重用，少写代码



## 二、什么是模板引擎

​    模板引擎（这里特指用于Web开发的模板引擎）是为了使用户界面与业务数据（内容）分离而产生的，它可以生成特定格式的文档，用于网站的模板引擎就会生成一个标准的HTML文档



## 三、如何使用？

​    artTemplate模板，这个插件拥有接近 JavaScript 渲染极限的的性能，友好的调试语法、运行时错误日志精确到模板所在行；支持在模板文件上打断点（Webpack Loader）art-template 以优雅的方式解决你 web 开发中 dom 字符串拼接与数据绑定的工具
   方式有两种：a、浏览器方法；b、node环境预编译；



## 四、artTemplate语法

######   1、直接加载template.js，使用一个type="text/html"的script标签存放模板，使用template(id,data)来调用渲染页面。

```html
<!DOCTYPE HTML>
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
    <title>浏览器使用方法</title>
    <meta name="keywords" content="浏览器使用方法">
    <meta name="description" content="浏览器使用方法">
    <link type="text/css" rel="stylesheet" href="http://uid.wangxiaobao.com/jsdoc/style.css"/>
</head>
<body>
<h1><a>浏览器使用方法</a></h1>
<div class="wrapper clearfix">
    <div class="f-right render-box"></div>
</div>
<script type="text/javascript" src="http://uid.wangxiaobao.com/wcb.js" data-main="jsdoc/base.js?version=201509081238"></script>
    // json数据
var data = {
    title: '基本例子',
    isAdmin: true,
    list: ['文艺', '博客', '摄影', '电影', '民谣', '旅行', '吉他'],
    value: '<span style="color:#F00">hello world!</span>'
};
<!--使用一个type="text/html"的script标签存放模板-->
<script id="test" type="text/html">
<ul>
        {{each list as value i}}
            <li>索引 {{i + 1}} ：{{value}}</li>
        {{/each}}
</ul>
</script>
<!--支持include方法直接引入-->
<!--#include file="tpl/index.html"-->
</body>
</html>

```

###### 2、使用预编译方法，基于node环境依赖于art-tempalter模板引擎的情况下配合 gulp 自动化预编译器tmod，它能把前端模板编译成不依赖模板引擎运行的 JS 文件，让前端模板可以突破浏览器的限制，模板间的调用按文件与目录的方式组织，通过构建工具将整个目录的模板（html文件）压缩打包到一个名为 template.js 的脚本文件，template(__dirname + '/xxx/xxx',data)来调用渲染页面，模板中同样支持引入子模板。

```
 <!DOCTYPE HTML>
        <html>
        <head>
            <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
            <title>预编译</title>
            <meta name="keywords" content="预编译">
            <meta name="description" content="预编译">
            <link type="text/css" rel="stylesheet" href="http://uid.wangxiaobao.com/jsdoc/style.css"/>
        </head>
        <body>
        <h1><a>预编译</a></h1>
        <div class="wrapper clearfix">
            <div class="f-right render-box"></div>
        </div>
        <script type="text/javascript" src="http://uid.wangxiaobao.com/wcb.js" data-main="jsdoc/base.js?version=201509081238"></script>

// json数据
var data = {
    title: '基本例子',
    isAdmin: true,
    list: ['文艺', '博客', '摄影', '电影', '民谣', '旅行', '吉他'],
    value: '<span style="color:#F00">hello world!</span>'
};
<!--按文件与目录组织模板，比如index.html-->
<ul>
    {{each list as value i}}
    <li>索引 {{i + 1}} ：{{value}}</li>
    {{/each}}
</ul>
<!--支持include方法直接引入-->
{{include file="tpl/index.html"}}
</body>
</html>
```

###### 3、常用语句

》》**if语句**

》》**If...else 语句**

》》**each **

》》**模板中默认支持的js方法有，字符串、数组、对象的方法,以下列举出项目中最常用的一些方法。

   比如大小写转换{{temp.toUpperCase()}}**

》》模板里方法的使用，通过template.helper()定义更多方法

###### 4、友好的dev-debug



###### 5、API指南

》》artTemplate使用API：http://docs.source3g.com:8989/api/#tpl-index



## 五、相关文档

》》artTemplate文档参考：https://www.awesomes.cn/repo/aui/arttemplate

》》TmodJS文档参考：https://github.com/aui/tmodjs/#%E5%AE%89%E8%A3%85

》》sass扩展语言使用文档参考：https://www.sass.hk/



## 六、前端源文件结构

```
├─ui-source/             存放线上和测试环境的文件目录
│   ├─develop/           测试环境的img,js,css,data,html等文件(用不上)
│   ├─doc/               前端API指南文档的img,js,css,data,html等文件
│   ├─preview/           正式环境的img,js,css,data,html等文件（用不上）
│   ├─ui/                正式环境的img,js,css文件夹
│   │  ├─ui/
│   │  │  ├─project/      项目名称（参考项目中的househ5）
│   │  │  │  ├─img/       图片管理
│   │  │  │  ├─base.js    js入口模块，需要带上每次迭代的版本号
│   │  │  │  └─style.css  css文件，需要带上每次迭代的版本号
│   ├─uid/                测试环境的img,js,css文件夹
│   │  ├─uid/
│   │  │  ├─project/      项目名称（参考项目中的househ5）
│   │  │  │  ├─img/       图片管理
│   │  │  │  ├─base.js    js入口模块，需要带上每次迭代的版本号
│   │  │  │  └─style.css  css文件，需要带上每次迭代的版本号
│   ├─view/
│   │  ├─view/            正式环境用来管理html格式的文件
│   │  │  ├─project/
│   │  │  │  ├─data/
│   │  │  │  ├─img/
│   │  │  │  └─index.html 入口文件
│   ├─view2/              测试环境用来管理html格式的文件
│   │  ├─view2/
│   │  │  ├─project/      项目名称（参考项目中的househ5）
│   │  │  │  ├─data/
│   │  │  │  ├─img/
│   │  │  │  └─index.html 入口文件
│   ├─50x.shtml
│   ├─index.shtml
│   ├─index.html
│   │  │
│   │  │
└─v1/                    此文件目录是采用artTemplate模板编译的项目
    │  gulpfile.js       gulp构建文件(此文件不用理会)
    │  │
    ├─doc/               前端文档
    │  │
    ├─build/             用来存放gulp构建好的发布文件，功能模块和页面样式块文件,如img,js,css
    │  ├─ project/       项目名（参考项目中的househ5）
    │  │  │─img/         gulp打包出来图片
    │  │  │─base.js      gulp打包出来的js文件，需要带上每次迭代的版本号
    │  │  └─style.css    gulp打包出来的css文件，需要带上每次迭代的版本号
    │  ......
    │  │
    │  ├─util/
    │  │  │  *js         各种插件
    │  ├─  *js           各种插件
    ├─demo/              用来存放gulp构建好的发布文件,如.html等模板文件
    │  ├─ project/       项目名称（参考项目中的househ5）
    │  │  │─data/        模拟的假数据
    │  │  │─img/         相对于当前文件存放的图片
    │  │  │─tpl/         模板文件
    │  │  └─index.html   入口文件
    │  │
    └─src/                         本地项目开发用源文件
        └─project/                 项目名（参考项目中的househ5）
           ├─demo/                 静态文件目录
           │  ├─ data/             数据模块
           │  │  └─ data.json      模拟数据
           │  ├─ img/              图片
           │  └─ tpl/              模板文件目录
           │  │  └─ index/         node环境下的使用是以文件目录路径的方式来调用模板文件
           │  │  │  └─ index.html  模板文件
           │  └─ index.html        入口文件
           ├─ui/                   img,js,css文件目录
           │  ├─ css/
           │  │  ├─ img/
           │  │  └─ scss           采用sass预处理编译
           │  └─ js/
           │     ├─base.js
           │     └─template_helpers.js  template辅助方法
           └─ gulp.js                   项目所在的gulp构建文件
```

