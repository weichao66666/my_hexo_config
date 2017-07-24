---
layout: article
title: Hexo + GitHub Pages 搭建 blog
date: 2017-07-24 12:14:46
tags: 
categories: 
copyright: true
---

# <center>**Hexo + GitHub Pages 搭建 blog**</center>

---

# **Reference**

* [Hexo Documentation](https://hexo.io/docs/ "https://hexo.io/docs/")
* [node.js](https://nodejs.org/en/ "https://nodejs.org/en/")
* [git](https://git-scm.com/ "https://git-scm.com/")
* [手把手教你用Hexo+Github 搭建属于自己的博客](http://blog.csdn.net/gdutxiaoxu/article/details/53576018 "http://blog.csdn.net/gdutxiaoxu/article/details/53576018")
* [window下配置SSH连接GitHub、GitHub配置ssh key](https://jingyan.baidu.com/article/a65957f4e91ccf24e77f9b11.html "https://jingyan.baidu.com/article/a65957f4e91ccf24e77f9b11.html")
* [NexT](http://theme-next.iissnan.com/getting-started.html "http://theme-next.iissnan.com/getting-started.html")
* [hexo的next主题个性化教程:打造炫酷网站](http://www.jianshu.com/p/f054333ac9e6 "http://www.jianshu.com/p/f054333ac9e6")
* [Hexo文章图片存储选七牛（当然支持MD都可以）](http://www.jianshu.com/p/ec2c8acf63cd "http://www.jianshu.com/p/ec2c8acf63cd")

---

# **环境搭建**

## **安装 Node.js**

## **安装 Git**

## **安装 Hexo**

在本地创建文件夹，比如 `D:\weichao\hexo`

windows + r 调出运行，输入 `cmd`，进入刚才创建的目录
![](http://otkw6sse5.bkt.clouddn.com/Hexo-GitHub-Pages-%E6%90%AD%E5%BB%BA-blog_files5972c794ab644125a10008ec.png)

输入`npm install -g hexo-cli`
![](http://otkw6sse5.bkt.clouddn.com/Hexo-GitHub-Pages-%E6%90%AD%E5%BB%BA-blog_files5972c794ab644125a10008ed.png)
warn 不会影响正常使用

输入`npm install hexo --save`，等待安装完成

输入`hexo -v`，显示
![](http://otkw6sse5.bkt.clouddn.com/Hexo-GitHub-Pages-%E6%90%AD%E5%BB%BA-blog_files5972c794ab644125a10008e6.png)
说明 Hexo 安装完成

---

# **clone Hexo 项目**

在本地创建文件夹，比如 `D:\weichao\hexo\config`

windows + r 调出运行，输入 `cmd`，进入刚才创建的目录

输入`hexo init`

输入`npm install`

输入`hexo g`

输入`hexo s`，会提示浏览器访问地址：
![](http://otkw6sse5.bkt.clouddn.com/Hexo-GitHub-Pages-%E6%90%AD%E5%BB%BA-blog_files5972c794ab644125a10008ea.png)

在任意浏览器中输入该地址：
![](http://otkw6sse5.bkt.clouddn.com/Hexo-GitHub-Pages-%E6%90%AD%E5%BB%BA-blog_files5972c794ab644125a10008eb.png)

---

# **关联 Hexo 和 GitHub Pages**

windows + r 调出运行，输入 `cmd`，进入 `D:\weichao\hexo\config`

## **配置 Git**

输入用户名、邮箱

    git config --global user.name "xx"
    git config --global user.email "xx@yy.com"

![](http://otkw6sse5.bkt.clouddn.com/Hexo-GitHub-Pages-%E6%90%AD%E5%BB%BA-blog_files5972c794ab644125a10008e5.png)

修改 `D:\weichao\hexo\config\_config.yml`

    deploy:
      type:

为

    deploy:
      type: git
      repo: git@github.com:yourname/yourname.github.io.git
      branch: master

## **安装 git 扩展**

    npm install hexo-deployer-git --save

## **配置 SSH**

### **生成密钥**

进入 `C:\Users\itpiz\.ssh`

右键 -> Git Bash Here

    ssh-keygen -t rsa -C "xx@yy.com"

![](http://otkw6sse5.bkt.clouddn.com/Hexo-GitHub-Pages-%E6%90%AD%E5%BB%BA-blog_files5972c794ab644125a10008e7.png)
`id_rsa` 是密钥，`id_rsa.pub` 是公钥

### **添加密钥到 ssh-agent**

    eval "$(ssh-agent -s)"
    ssh-add ~/.ssh/id_rsa

![](http://otkw6sse5.bkt.clouddn.com/Hexo-GitHub-Pages-%E6%90%AD%E5%BB%BA-blog_files5972ce32ab644125a100096c.png)

### **添加密钥到 GitHub**

复制 `id_rsa.pub` 中的内容

![](http://otkw6sse5.bkt.clouddn.com/Hexo-GitHub-Pages-%E6%90%AD%E5%BB%BA-blog_files5972c794ab644125a10008e9.png)

### **测试 ssh keys 是否设置成功**

输入 `ssh -T git@github.com`

![](http://otkw6sse5.bkt.clouddn.com/Hexo-GitHub-Pages-%E6%90%AD%E5%BB%BA-blog_files5972c794ab644125a10008e8.png)

---

# **设置主题**

windows + r 调出运行，输入 `cmd`

进入 `D:\weichao\hexo\config`

## **clone 主题**

    git clone https://github.com/iissnan/hexo-theme-next themes/next

## **修改配置文件**

修改 `D:\weichao\hexo\config\_config.yml`

    theme: landscape

为

    theme: next

## **清除缓存**

    hexo clean

## **验证主题**

    hexo s -debug

---

# **创建文章并上传**

windows + r 调出运行，输入 `cmd`

进入 `D:\weichao\hexo\config`

新建一篇博客：

    hexo new post "article title"

生成并部署

    hexo d -g

![](http://otkw6sse5.bkt.clouddn.com/Hexo-GitHub-Pages-%E6%90%AD%E5%BB%BA-blog_files5972c794ab644125a10008e4.png)