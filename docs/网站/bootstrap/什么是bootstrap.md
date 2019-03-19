# 什么是bootstrap
bootstrap是一个前端开发框架。对于开发前端的人来说，bootstrap就是一整套已经写好的样式表和动画。
使用的时候先将它的.css文件和.js文件导入，然后只需要给标签附上相对于的class名字就能够使用bootstrap准备好的样式。

>## 举个例子

```
<link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/4.1.0/css/bootstrap.min.css">
  <script src="https://cdn.staticfile.org/jquery/3.2.1/jquery.min.js"></script>
  <script src="https://cdn.staticfile.org/popper.js/1.12.5/umd/popper.min.js"></script>
  <script src="https://cdn.staticfile.org/twitter-bootstrap/4.1.0/js/bootstrap.min.js"></script>
```
这个是官方提供的一个在线的引用样式表，不需要你下载就可以直接用。

然后随意编辑一个按钮
```
<button class="btn btn-primary">按钮</button>
```
然后你会得到
<html>
<head>
<link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/4.1.0/css/bootstrap.min.css">
  <script src="https://cdn.staticfile.org/jquery/3.2.1/jquery.min.js"></script>
  <script src="https://cdn.staticfile.org/popper.js/1.12.5/umd/popper.min.js"></script>
  <script src="https://cdn.staticfile.org/twitter-bootstrap/4.1.0/js/bootstrap.min.js"></script>
</head>
<body>
<button class="btn btn-primary">按钮</button>
</body>
</html>

>注意！我什么样式都没有给它添加

我所做的只是给它的class加上了btn、btn-primary两个名字，它便自动应用了bootstrap中的样式。

如何，是不是很方便？