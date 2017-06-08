---
title:Android模拟服务器数据
tags: Android
grammar_cjkRuby: true
---

确保电脑和手机在同一个WIFI环境
# Charles 设置

1. 在电脑端安装证书，如图 1-2
![enter description here][1]

# 手机端配置

![](https://raw.githubusercontent.com/Shinelw/Android/master/BlogPicture/charles/Screen%20Shot%202016-04-21%20at%203.51.36%20PM.png)


 ![enter description here][2]


此图中的ip是电脑的ip地址

然后在手机自带浏览器输入 chls.pro/ssl 下载ssl证书，安装；

# Charles 再设置

选择你需要更改数据的网址，右键选择** Map Remote**

配置如下：

Map To 的网址可以选择 http://www.mocky.io/ 生成你需要的数据



![enter description here][4]

# 遇见的问题 

1. 手机设置后无法下载证书

可以通过电脑下载好之后在手机安装证书 

2. 安装证书之后还是无法链接

关掉电脑的防火墙

  [1]: https://wx4.sinaimg.cn/large/534fc2d6ly1ffwit6dg0yj20je07uwf1.jpg
  [2]: https://raw.githubusercontent.com/Shinelw/Android/master/BlogPicture/charles/Screen%20Shot%202016-04-21%20at%203.53.23%20PM.png
  [3]: http://markdown.xiaoshujiang.com/img/spinner.gif "[[[1495614080350]]]"
  [4]: https://ws2.sinaimg.cn/large/534fc2d6ly1ffwitnl42tj20bo0eet94.jpg