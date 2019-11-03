# Python学习
## 前言
Python好啊

本博写作希望实现的目标有二：
1. 实现Python能力的从无到有
2. 通过Python学习DeepLearning和MachineLearning

先从python的学习开始

[TOC]

## 引入：在Mac（OSX）上使用Github
### 1.1  注册  
不多说

### 1.2  关于git
#### 1.2.1  git介绍  
git是一个开源的分布式的版本控制（Revision Control）系统，通过git我们可以实现跨区域的多人协同开发。而github就是一个基于git的面向开源软件项目的托管平台。  
在git中，**仓库**（Repository）是一个重要的概念，它是一个受版本控制的有所有文件修订历史的共享数据库，一般而言一个Repository对应一个Project。而**工作空间**（Workspace）则是用户在本地编辑的副本，工作空间中仓库的各个文件则称**工作树**（Working Tree）。而**暂存区**（Staging Area）则是用于暂存工作区的变化以便向版本库（Repository）提交**更改**（commit）的区域。  
简单来说就是用户在本地的工作区对项目进行改变，通过git add进入暂存区，然后提交（git commit）进入版本库，使得在线的版本库获得更新。  
#### 1.2.2  其他名词  
*索引*（Index），是暂存区的另一种说法；  
*签入*（Check in），将新版本从工作空间复制回仓库；  
*签出*（checkout），将新版本从仓库复制到工作空间；  
*冲突*（Conflict），多人对同一文件的工作副本进行了修改并提交；  
*分支*（Branch），从主线上分离开的副本，默认分支为master；  
*合并*（Merge），将某分支上的更改联接到主干或另一分支；  
*标记*（Tags），某个分支某个特定时间点的状态，通过标记可以将仓库切换到标记时的状态。

### 1.3  git安装与更新  
在Terminal输入git可以查看是否安装了git，如果没有，可以通过Xcode->Preference->Downloads->Command Line Tools进行安装，也可以去git官网下载安装。  
通过  
```
git —version
```  
可以查看当前安装的git的版本  
而通过git clone https://github.com/git/git ，我们可以对git进行自动更新  
要保证github连通功能的实现，必须配置ssh（Secure Shell），这是一种为远程登录准备的安全协议。因此需要创建ssh key去配置git。  
1. 通过terminal设置username和e-mail：  
```
git config --global.user.name “你的名字”  
git config --global.email “你的邮箱“
```  
2. 创建ssh key  
```
ssh-keygen -t rsa -C “你的邮箱”
```  
3. 按一通“y”确定完了就好了，git会在~/下生成一个.ssh文件夹（在Finder里是看不见的），点进去打开id_rsa.pub完整地复制你的key即可获得秘钥。  
可以使用cat命令查看你的key  
```
cat .ssh/id_rsa.pub
```  
如果想在终端中查看这些文件可以输入  
```
open ~/.ssh
```  
4. 进入github官网，在Settings->SSH and GPG keys->New SSH Key中添加你的ssh key，添加完成后在终端输入：  
```
ssh -T git@github.com
```  
如果输出“You’ve successfully…”那么说明已经链接成功。

### 1.4  安装GitHub Desktop
首先使用搜索引擎搜索并安装
我们可以通过在github上你个人的仓库界面的右上角绿色的clone or download->clone in desktop打开，然后浏览器会自动转到github desktop，贴心的帮你创建一个本地的工作空间。当然你也可以通过在本地创建然后同步到你的仓库中来创建一个新的项目。
接着我们可以通过Atom或VScode等极为先进的编辑器去修改你的项目（VScode太好用了
更新完以后暂时无法将其更新到我们的在线库中，我们应当通过新建一个Branch去安放我们的更新，如果想要回滚，我们只需要去原来的Branch中在history里revert就行了。通过新建Branch，并在左下角完善Summary和Description，就可以Commit to提交了，接着点击右上角的Publish branch就可以把你的更改同步到在线库中了！

## 从笔记开始：使用markdown完善你的博客
### 2.1 markdown语言
markdown是一种轻量级标记语言，它可以使你的文本拥有各自格式，同时markdown支持插入Latex公式，去完成你的投稿论文与毕业论文，如我们插入
```
$$ c = \sqrt{a^{2}+b_{xy}^{2}+e^{x}}$$
``` 
可以得到
$$ c = \sqrt{a^{2}+b_{xy}^{2}+e^{x}}$$
关于markdown的**极其简单**的语法，建议使用搜索引擎进行学习
### 2.2 创建博客
github pages是GitHub提供的一个个人静态主页网站托管服务，**开源**高效免费实时，而且可用空间高达1G，无敌。我们可以在github上创建一个域名为“你的名字.github.io”并选择相应的Jekyll主题就可以去美滋滋的拥有了一个个人博客了。
我原本打算照搬CSDN用HEXO＋Node.js通过nvm（Node Version Manager）去搭建我的博客，后来嫌烦就咕咕咕了。原生的就足够好了。
我们可以提交一个index.html作为博客的主页，也作为每个人开始github的第一段代码：
```html{class=line-numbers}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
    <style type="text/css">
    h1{
        text-align:center
        font-size:50px
    }
    </style>
</head>
<body>
    <h1>Hello World</h1>
</body>
</html>
```
一个标准的使用Jekyll工具创建的网站，其目录结构一般为：
```jekyll{class=line-numbers}
#保存配置数据
    _config.yml
#未发布的文章
    _drafts
        nb.textile
        nb.markdown
#可以将这些东西加到你的文章和布局中
    _includes
        footer.html
        header.html
#文章外部的模板
    _layouts
        default.html
        post.html
#我们的文章在这里
    _posts
        2019-10-20-gck.textile
        2019-10-21-gck.textile
#一些配置文件
    _data
        members.yml
    _site
    index.html
```
那其实根据这个解释我们就可以无障碍解决博客问题了。
我们在写完markdown后必须将.md转为html文件后一同上传才能在博客中显示。对于忘了装VScode的用户（比如昨天的我），可以通过python中的markdown库进行markdown至html的转换。代码如下：
```python{class=line-numbers}
#安装markdown转换的玩意
pip3 install markdown
#加载相关库
import codecs, markdown
#读取markdown文本
input_file = codecs.open("name.md", mode="r",encoding="utf-8")
text = input_file.read()
#转为html文本
html = markdown.markdown(text)
#保存为文件
output_file = codecs.open("name.html", mode="w", encoding="utf-8")
output_file.write(html)
```
当然，我们使用极为先进的VScode的Markdown All in One和Markdown Preview Enhanced等极为先进的extensions来预览我们的markdown文档，通过f1(OSX里是反人类的command+shift+P）打开命令窗口并输入“mark”，我们可以将其转为html，也可以直接在预览界面右键选择html，并进而转为喜闻乐见的**pdf**！
