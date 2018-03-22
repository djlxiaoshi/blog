---
title: hexo初探---让写作飞起来
date: 2017-01-13 14:49:57
tags: 
    - hexo
    - github
categories: hexo博客搭建
---
这两天一直在捣鼓hexo，虽然不是很难，但是在搭建过程中，还是遇到了一些问题，导致花了两天。网上有很多相关教程，但是很多都含糊不清，其实让我豁然开朗的还是这篇文章，在此特别谢谢这位博主[小白独立搭建博客--Github Pages和Hexo简明教程](https://my.oschina.net/ryaneLee/blog/638440)。好了下面我们直接开始吧
<!-- more -->

## 所需工具
[Git](https://git-scm.com/download)：版本控制器   

[Node](https://nodejs.org/zh-cn/download/)：提供服务器端JavaScript开发环境

[GitHub](https://github.com)：需注册一个账号

在这里只提供下载链接，安装过程基本就是下一步了。


## 开始安装
###  1 安装hexo
安装好上述工具后，我们开始安装hexo，首先打开git,鼠标右键-->git bash here

![Git_bash.png](http://upload-images.jianshu.io/upload_images/1948637-970cfcfc5c4d9e07.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

打开效果如下图：

![打开Git.png](http://upload-images.jianshu.io/upload_images/1948637-f6d8b39d88552ff3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后执行下面的命令
`$ npm install -g hexo-cli`，全局安装hexo的脚手架，安装以后我们就可以全局使用hexo这个命令了。安装成功后执行下面命令`hexo -v`，就可以看见如下效果。
![hexo安装成功.png](http://upload-images.jianshu.io/upload_images/1948637-f6c1ceeafe9de1d8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

接着执行下面命令`npm install hexo --save`

然后新建一个文件夹比如hexo，在该文件夹下打开Git(即Git bash here)，执行下面命令

```
$ hexo init 
$ npm install
```

不出问题的话，该文件夹下面会新增这些文件。

![hexo_init.png](http://upload-images.jianshu.io/upload_images/1948637-a9604272a538c58f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 3 安装hexo服务器
Hexo 3.0 把服务器独立成了个别模块，您必须先安装 [hexo-server](https://github.com/hexojs/hexo-server) 才能使用。执行命令`$ npm install hexo-server --save`。这样安装好hexo服务器以后，我们就可以启动服务器了。执行命令`$ hexo server`，然后在浏览器输入网址**http://127.0.0.1:4000/**,就可以正常访问了。效果图如下：

![本地效果.png](http://upload-images.jianshu.io/upload_images/1948637-762b47f41c85b29e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)。

OK！现在本地已经没问题了，接下来就是要把它部署到远程服务器上了。

### 3  建立GitHub pages

新建一个github仓库

![new_repository.png](http://upload-images.jianshu.io/upload_images/1948637-e9ad42417d0f33d4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![GitHub_pages建立.png](http://upload-images.jianshu.io/upload_images/1948637-ac753802c8884e86.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这个仓库比较特殊，它要按照如下格式进行命名：你的GitHub用户昵称.github.io。新建好这个仓库后，见如下效果图


![djlxs.github.io.png](http://upload-images.jianshu.io/upload_images/1948637-82a49dd72ededad5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后点击settings选项，向下翻到Git Pages选项

![导入主题前Git_pages.png](http://upload-images.jianshu.io/upload_images/1948637-84fbadde099f3129.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上面有一个提示信息
>GitHub Pages is currently disabled. You must first add content to your repository before you can publish a GitHub Pages site

我们此时访问https://djlxs.github.io/ 是会报404的，因为没有内容，我们可以点击下面的Choose a theme按钮，选择一个主题后，在访问https://djlxs.github.io/ ，就可以了。此时看看GitHub Pages选项已经变成

![选择主题后.png](http://upload-images.jianshu.io/upload_images/1948637-ef566fc26d0cedcb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
上面有个提示信息
> Your GitHub Pages site is currently being built from the master
 branch

OK啦

### 4 配置SSH公钥

配置Github的SSH密钥可以让本地git项目与远程的github建立联系，让我们在本地写了代码之后直接通过git操作就可以实现本地代码库与Github代码库同步。操作如下：

第一步、看看是否存在SSH密钥(keys)
首先，我们需要看看是否看看本机是否存在SSH keys,打开Git Bash,并运行:

```
$ cd ~/. ssh
```

检查你本机用户home目录下是否存在.ssh目录
如果，不存在此目录，则进行第二步操作，否则，你本机已经存在ssh公钥和私钥，可以略过第二步，直接进入第三步操作。

第二步、创建一对新的SSH密钥(keys)

```
$ssh-keygen -t rsa -C "your_email@example.com" #这将按照你提供的邮箱地址，创建一对密钥
```

直接回车，则将密钥按默认文件进行存储。此时也可以输入特定的文件名，比如`/c/Users/you/.ssh/github_rsa`
接着，根据提示，你需要输入密码和确认密码,在这里我们直接enter，不用输入密码。

```
Enter passphrase (empty for no passphrase): [Type a passphrase]
Enter same passphrase again: [Type passphrase again]
```

输入完成之后，屏幕会显示如下信息：

```
Your identification has been saved in /c/Users/you/.ssh/id_rsa.Your public key 
has been saved in /c/Users/you/.ssh/id_rsa.pub.The key fingerprint 
is:01:0f:f4:3b:ca:85:d6:17:a1:7d:f0:68:9d:f0:a2:db your_email@example.com
```

第三步、在GitHub账户中添加你的公钥
运行如下命令，将公钥的内容复制到系统粘贴板(clipboard)中。

```
clip < ~/.ssh/id_rsa.pub
```

接着：
登陆GitHub,进入你的Account Settings.

![add_SSH_key.png](http://upload-images.jianshu.io/upload_images/1948637-2a9b043895299590.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

第四步、测试
可以输入下面的命令，看看设置是否成功，git@github.com的部分不要修改：

```
$ ssh -T git@github.com
```

成功后你会看到

```
Hi djlxs! You've successfully authenticated, but GitHub does not provide shell access.
```

第五步、设置用户信息
现在你已经可以通过SSH链接到GitHub了，还有一些个人信息需要完善的。 Git会根据用户的名字和邮箱来记录提交。GitHub也是用这些信息来做权限的处理，输入下面的代码进行个人信息的设置，把名称和邮箱替换成你自己的。

```
$ git config --global user.name "djlxs"//用户名
$ git config --global user.email "djlxs@gmail.com"//填写自己的邮箱
```

### 5 将本地文件同步至GitHub

![clone_https.png](http://upload-images.jianshu.io/upload_images/1948637-badcf3287146813f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

到新建的仓库下面复制HTTPS地址，然后打开本地hexo文件夹下的

![_config.jpg](http://upload-images.jianshu.io/upload_images/1948637-53f92f31dd7c739c.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

填入以下信息：

![add_config.jpg](http://upload-images.jianshu.io/upload_images/1948637-d7c3768113ba4bc5.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后在命令窗口执行下面命令,安装相关部署插件

```
$ npm install hexo-deployer-git --save
```

安装完成后，执行

```
hexo g
hexo d

```

或者

```
hexo d -g
```

此时会让你输入GitHub的账号和密码，输入后在浏览器中输入 https://djlxs.github.io/ 应该就可以访问了。

## 更换主题

首先去[hexo皮肤网站](https://hexo.io/themes/),选择一款进入它的GitHub地址然后将clone的文件移动到**themes**文件夹下。然后修改**_config.yml**文件中的theme为你所选择的主题的名称即可。然后执行

```
hexo g
hexo s
```

现在本地看看主题皮肤是否已经更换过来，然后执行

```
hexo d -g
```

同步至GitHub，刷新即可

## 绑定域名
首先在万网上购买自己的域名，然后进入管理界面


![域名解析.jpg](http://upload-images.jianshu.io/upload_images/1948637-b4d8e0c05fc2fe50.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![域名解析内容.jpg](http://upload-images.jianshu.io/upload_images/1948637-6ffad7b818b55492.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后在source文件夹下建立一个CNAME文件(没有后缀名)

![CNAME文件.jpg](http://upload-images.jianshu.io/upload_images/1948637-db064d9b5ef4c125.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
然后执行

CNAME:是提供别名解析，就是说可以从一个域名解析绑定到另一个域名。 例如：`http://djlxs.github.io`  解析绑定到 `http://djl.pub`。
其中主机记录***www***代表一个二级域名此时输入`http://www.djl.pub`是可以正常访问的。如果主机记录是***@***代表直接输入`http://djl.pub`是可以访问的
一般情况下这两种方式我们都会添加，所以你在浏览器地址栏中输入`http://djl.pub`和`http://www.djl.pub`跳到是同一个页面。

A:提供的是一种IP地址解析到你所买的域名的解析方式。例如：`151.101.24.133`  解析到 `djl.pub`

在这里我用的是阿里云自带的DNS服务器，当然你也可以使用其他第三方DNS服务器。例如：[dnspod](https://www.dnspod.cn/),添加解析记录的方式一样
但是这里要记得在你购买域名的网站上更改默认的DNS服务商，以下以万网（域名提供商）和dnspod（域名解析服务商）为例

首先进入控制台进入域名管理，找到相应域名点击**管理**

![域名管理控制台.jpg](http://upload-images.jianshu.io/upload_images/1948637-1dbf77201fec45d2.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击修**改DNS**

![修改DNS服务器.jpg](http://upload-images.jianshu.io/upload_images/1948637-ea78ba366c068ee3.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

将以下内容复制粘贴进下图的内容框中保存，即可
```
f1g1ns1.dnspod.net
f1g1ns2.dnspod.net
```

![更改DNS服务器.jpg](http://upload-images.jianshu.io/upload_images/1948637-baad0040dbc4dae9.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
hexo d -g
```

同步至GitHub即可，至此hexo博客搭建基本就大功告成。下面推荐一个工具[hexo-hey](https://github.com/nihgwu/hexo-hey)

[小白独立搭建博客--Github Pages和Hexo简明教程](https://my.oschina.net/ryaneLee/blog/638440)

[hexo中文文档](https://hexo.io/zh-cn/docs/index.html)

## 遇到问题

![问题01.jpg](http://upload-images.jianshu.io/upload_images/1948637-f354427a1d5932cf.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

解决办法:[参考链接](https://segmentfault.com/q/1010000003734223)、[参考链接](https://github.com/hexojs/hexo/issues/1503)

## 写在最后

如果你只是想搭建一个自己的博客，绑定自己的域名写写文章，不想自己动手添加其他功能（评论、分享、搜索），那么选择
一款好的皮肤就很重要，因为有的皮肤早已经帮你集成好了，只需要动手简单的改改配置文件就行，在这里给大家推荐一款皮肤
可能样式有些过于花哨，不过这些都可以自己调，功能很全，文档也比较全。[github地址](https://github.com/MOxFIVE/hexo-theme-yelee)

这是我在新的博客上面的第一篇文章，欢迎大家访问我的博客。[DJL箫氏](http://djl.pub/)