## chrome浏览器：打包插件与安装打包插件详细教程

最新推荐文章于 2025-09-02 16:27:28 发布

原创 于 2020-07-25 09:08:29 发布 · 3.3w 阅读

· ![](https://csdnimg.cn/release/blogv2/dist/pc/img/newHeart2023Black.png) 28

· ![](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarCollect2.png) 66 ·

CC 4.0 BY-SA版权

本内容属个人原创，未经博主允许不得转载，转载需要申明出处，谢谢。

## 1 进入插件管理

在chrome浏览器地址栏输入：chrome://extensions/然后回车，即可进入插件管理。也可通过如下方式进入。

 点击chrome浏览器右上角的“![](https://i-blog.csdnimg.cn/blog_migrate/8f2fd89bbbea1862408cb50054e4d1d5.png)”，列表中选择“更多工具”，然后选择 “开发者工具”，进入插件管理。

![](https://i-blog.csdnimg.cn/blog_migrate/d3ab5b8531da9829418b0ac31e3ca369.png)

 打开开发者模式。

![](https://i-blog.csdnimg.cn/blog_migrate/7b16f5d3c6dde869c9034e55c77dabcb.png)

## 2 打包插件

### 2.1 查看插件ID

![](https://i-blog.csdnimg.cn/blog_migrate/671e48803cad39af58a2464387061311.png)

### 2.2 查找扩展程序路径

推荐使用Everything搜索工具，搜索插件ID，找到插件的位置。或者直接去C:\\Users\\xxx \\AppData\\Local\\Google\\Chrome\\User Data\\Default\\Extensions\\下查找对应ID的插件。

Everthing下载地址：[官方下载](https://www.voidtools.com/zh-cn/downloads/)，[CSDN下载](https://download.csdn.net/download/qq_34160841/12659907)

![](https://i-blog.csdnimg.cn/blog_migrate/0415fb3b89878c7e82d6af0d88bb79df.png) 一般插件会有对应的版本，需要进入版本文件夹，然后复制该完整路径。

![](https://i-blog.csdnimg.cn/blog_migrate/6935e2bb0c203d53c0753848672c0c7d.png)

### 2.3 打包扩展程序

点击左上角“打包扩展程序”，在可以输入上一步需要打包扩展程序的完整路径（也可以通过店家“浏览”，然后在打开的文件浏览器中选择扩展程序路径），然后点击“打包扩展程序”。

![](https://i-blog.csdnimg.cn/blog_migrate/938bc84e412c830c4c570fcf0aa518b1.png)

![](https://i-blog.csdnimg.cn/blog_migrate/8f639a86c23c0b5c0d06a5f502211078.png)

### 2.4 查看打包文件

打包文件在选择打包插件的上一级目录下，如下图所示。

![](https://i-blog.csdnimg.cn/blog_migrate/e49ced2f7352bf090049187d0453d145.png)

## 3 安装扩展程序 

### 3.1 方法一：安装解压扩展程序

#### 3.1.1 修改文件扩展名

将xxx.crx改名为xxx.rar。看不到文件扩展名请参考：[显示文件扩展名](https://blog.csdn.net/qq_34160841/article/details/104862230)。

![](https://i-blog.csdnimg.cn/blog_migrate/7c4ddfdbf7d56a746a94b06abfcb9290.png)

#### 3.1.2 解压文件

 将上一步修改文件扩展名后的文件进行解压。

![](https://i-blog.csdnimg.cn/blog_migrate/641e284768ed6d96468ad5c359c0f9bd.png)

#### 3.1.3 安装扩展程序 

在chrome浏览器扩展程序页面点击“加载已解压扩展程序”，然后选择解压扩展程序的路径，然后点击“选择文件夹”。

![](https://i-blog.csdnimg.cn/blog_migrate/f9734a8461cc2eda28b1419604f23345.png)

#### 3.1.4 完成安装

![](https://i-blog.csdnimg.cn/blog_migrate/c1209d0180c0fa699cacbe490e9651ef.png)

#### 3.1.5 去除错误

点击错误，打开界面点击“全部清除”

![](https://i-blog.csdnimg.cn/blog_migrate/51b5ddcacc327ea1333c7ed6493360d6.png)

![](https://i-blog.csdnimg.cn/blog_migrate/f0a5299bcac303a035d5b8a224e44daf.png)

###  3.2 拖拽安装

将打包的xxx.crx文件拖拽到chrome浏览器，然后点击“添加到扩展程序”即可。

![](https://i-blog.csdnimg.cn/blog_migrate/ba996c25435627111cf558c8c245a93d.png)

如果安装完成后出现如下所示，请采用方法1进行安装。

![](https://i-blog.csdnimg.cn/blog_migrate/f17ea8f1d3f0a979bfe45f92d97220ac.png)

##  4 插件地址

谷歌官方插件地址：[https://chrome.google.com/webstore/category/extensions](https://chrome.google.com/webstore/category/extensions)

国内插件地址：[http://www.cnplugins.com/](http://www.cnplugins.com/)

官方插件地址，插件很多，但是需要能访问谷歌才可以；国内插件地址插件较少，可以搜索可用插件。
