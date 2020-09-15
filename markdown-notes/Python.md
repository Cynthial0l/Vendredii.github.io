# Python与其他

**写在前面**

Python好啊
本博写作希望实现的目标有二：
1. 实现Python能力的从无到有
2. 通过Python学习DeepLearning和MachineLearning
3. 靠python吃饭?

先从python的学习开始

[TOC]

# 关于编译与语言——一些杂谈
## 选择一个编译工具
最早是用Atom，但是说实话不是很适合萌新，虽然这种东西也没什么适合不适合的，毕竟我的第一个脚本就是用写字板加Terminal，但是一个好的编译器能让事情变得更加简单，对于Python的学习，使用Rstudio是可以的，使用PyCharm是多数人推荐的，使用Xcode是...令人厌恶的，而使用VScode，则会让人意外的舒服，可能这是巨硬的魔力吧，VScode可以装一个Python插件这样就能直接在右上角一键运行程序了，而不用点开来调出控制台，还挺适合我这种从来记不住快捷键的人的。
在VScode中，使用Command+/可以给代码快速添加注释（#）符号
## 在Mac（OSX）上使用Github
### 关于git 
git是一个开源的分布式的版本控制（Revision Control）系统，通过git我们可以实现跨区域的多人协同开发。而github就是一个基于git的面向开源软件项目的托管平台。  
在git中，**仓库**（Repository）是一个重要的概念，它是一个受版本控制的有所有文件修订历史的共享数据库，一般而言一个Repository对应一个Project。而**工作空间**（Workspace）则是用户在本地编辑的副本，工作空间中仓库的各个文件则称**工作树**（Working Tree）。而**暂存区**（Staging Area）则是用于暂存工作区的变化以便向版本库（Repository）提交**更改**（commit）的区域。  
简单来说就是用户在本地的工作区对项目进行改变，通过git add进入暂存区，然后提交（git commit）进入版本库，使得在线的版本库获得更新。  
### 名词介绍
*索引*（Index），是暂存区的另一种说法；  
*签入*（Check in），将新版本从工作空间复制回仓库；  
*签出*（checkout），将新版本从仓库复制到工作空间；  
*冲突*（Conflict），多人对同一文件的工作副本进行了修改并提交；  
*分支*（Branch），从主线上分离开的副本，默认分支为master；  
*合并*（Merge），将某分支上的更改联接到主干或另一分支；  
*标记*（Tags），某个分支某个特定时间点的状态，通过标记可以将仓库切换到标记时的状态。
### git安装与更新  
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

### 安装GitHub Desktop
首先使用搜索引擎搜索并安装
我们可以通过在github上你个人的仓库界面的右上角绿色的clone or download->clone in desktop打开，然后浏览器会自动转到github desktop，贴心的帮你创建一个本地的工作空间。当然你也可以通过在本地创建然后同步到你的仓库中来创建一个新的项目。
接着我们可以通过Atom或VScode等极为先进的编辑器去修改你的项目（VScode太好用了
更新完以后暂时无法将其更新到我们的在线库中，我们应当通过新建一个Branch去安放我们的更新，如果想要回滚，我们只需要去原来的Branch中在history里revert就行了。通过新建Branch，并在左下角完善Summary和Description，就可以Commit to提交了，接着点击右上角的Publish branch就可以把你的更改同步到在线库中了！

## 记录你的学习：使用markdown完善你的博客
### markdown语言
markdown是一种轻量级标记语言，它可以使你的文本拥有各自格式，同时markdown支持插入Latex公式，去完成你的投稿论文与毕业论文，如我们插入
```
$$ c = \sqrt{a^{2}+b_{xy}^{2}+e^{x}}$$
``` 
可以得到
$$ c = \sqrt{a^{2}+b_{xy}^{2}+e^{x}}$$
关于markdown的**极其简单**的语法，建议使用搜索引擎进行学习
### 创建博客
github pages是GitHub提供的一个个人静态主页网站托管服务，**开源**高效免费实时，而且可用空间高达1G，无敌。我们可以在github上创建一个域名为“你的名字.github.io”并选择相应的Jekyll主题就可以去美滋滋的拥有了一个个人博客了。
我原本打算照搬CSDN用HEXO＋Node.js通过nvm（Node Version Manager）去搭建我的博客，后来嫌烦就咕咕咕了。原生的就足够好了。
我们可以提交一个index.html作为博客的主页，也作为每个人开始github的第一段代码：
```html {class=line-numbers}
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
# Python
## 工作环境
Python不多介绍，牛逼
为了更好地利用python，我们需要利用虚拟空间来运行我们的python项目，以免各个项目与程序包之间发生干扰。
我们通过在Terminal里创建虚拟环境来完成这些工作：
```
#在当前目录下新建一个叫venv的文件，以后的所有操作都在那里
python3 -m venv venv
#激活虚拟环境
source venv/bin/activate
#退出虚拟环境
deactivate
#打开现有的虚拟环境，首先需要cd到相关文件夹
source bin/activate
```
库和r语言的包一样，是别人讲一些算法与功能打包后方便大家使用，可以利用Terminal/控制台安装库
```
pip install 库名
```
## 运行与储存你的代码
我们可以使用基于网页的jupyter notebook来储存、调试、运行你所有的代码，可以理解为网页版的VScode？
安装并运行jupyter
```
#更新pip
pip install --upgrade pip
#安装jupyer
pip3 install jupyter
#运行
jupyter notebook
#关闭
control+C
```
关于其他相关指南，可以看[简书](jianshu.com/p/91365f343585)
## 数据处理
### 数据预处理:pandas_profiling
这个是神器，可以快速预览数据。输出与数据有关的相关性分析和各类描述统计等等，还可以指出数据有哪些确实等：
安装pandas等依赖的库
```
pip3 install pandas
pip install pandas-profiling
```
运行：
```py
#导入程序包
import pandas as pd
import pandas_profiling
#导入数据
# df = pd.read_excel("data.xlsx")
df = pd.read_csv("data.csv", header = 0)
df
#启动pandas_profiling
profile = pandas_profiling.ProfileReport(df)
profile
#导出为网页
with open("report.html", "w") as f:
    f.write(profile.to_html())
```
### 绘制折线图
```py
import matplotlib.pyplot as plt 
y2 = [2,6,11,20,40]
L1, =plt.plot(x,y,color='blue',\
  linewidth=2,linestyle='--',\
  marker='*',markersize=10)
L2, =plt.plot(x,y2,color='blue',\
  linewidth=2,linestyle='-',\
  marker='s',markersize=10)
plt.title('My test')
plt.xlabel('Value')
plt.ylabel('result')
plt.xlim([1,5])
plt.ylim([1,40])
plt.legend([L1,L2,],['Line1','Line2'])
#xy(3,11)指箭头指向的位置，xytext是箭头起始的标注位置
plt.annotate('local max',xy=(3,11),xytext=(3.5,26),\
    xycoords='data',arrowprops=dict(facecolor='c',width=1))
plt.show()
```
### 绘制饼图
```py
import matplotlib.pyplot as plt
labels = 'frogs','hogs','dogs','logs'
size = 15,30,45,10
colors = 'yellowgreen','gold','lightskyblue','lightcoral'
explode = 0,0.1,0,0
plt.pie(size,explode=explode,labels=labels,colors=colors,shadow=True)
plt.show()
```
### 绘制箱型图
```py
import matplotlib.pyplot as plt
x = [1,2,3,4,5]
y = [1,4,9,16,25]
plt.bar(x,y,facecolor='g')
for x,y in zip(x,y):
    plt.text(x,y,'{f}'.format(f=y),ha='center',va='bottom')
plt.show()
```
### 生成随机数等
```py
import numpy as np
value = np.random.randint(1,100,10)
print(value)
```
计算两点距离
```py
import numpy as np
import matplotlib.pyplot as plt
value = np.random.randint(1,100,10)
x = np.random.randint(1,100,10)
y = np.random.randint(1,100,10)
plt.scatter(x,y,s=200,c=value,alpha=0.7)
i,j =0,0
while i <=8:
    j = 0
    while j<=8:
        d = (x[i]-x[j+1])**2 + (y[i]-y[j+1])**2
        if d <= 2500:
            plt.plot([x[1],x[j+1]],[y[i],y[j+1]])
        j = j+1
    i = i+1
plt.show()
```
### 数据筛选
数据处理
数据源是csv格式，
```py
import pandas as pd
df = pd.read_csv('H:/x.csv',usecols=['subdate','temperature','humidity'])
#查看数据
print(df)
print(df.dtypes)
print(df.colums)
print(df.index)
print(df.values)
#1指pandas的自动标序缩影的序号，先列后行
print(df['tempetature'][1])
#先行后列，-1表示最后一行
print(df,iloc[1:5,1])
#转为日期类型
df['subdate'] = pd.to_datetime(df['subdate'])
#绘图
df.plot(x='subdate',y='temperature',rot='45')
plot.show()
#处理缺失，日期缺失
#生成连续日期
dateindex = pd.date_range(start='20140101',end='20140110')
#将系统的缩影改为日期
df.index = df['subdate']
#重新设置缩影
#重设缩影后就可以发现缺失的日期了
df = df.reindex(dateindex)
#以0行为索引寻找缺失值
print(np.where(pd.isna(df))[0])
#寻找缺失行的行标
print(df.index(np.where(pd.isna(df))[0])
#用0填充缺失值
df = df.fillna(0)
#使用后面的一个值填充，'pad'为前一个值
df = df.fillna(method='bfill')
#填充平均值
for i in np.where(pd.isna(df))[0]:
	df.iloc[i,1] = (df.iloc[i-1,1] + df.iloc[i+1,1])/2
print(df)
#筛选异常数据
error_temperature = df['temperature']
error_temperature_list = error_temperature[(12 < error_temperature) | (error_temperature < 6)]
print(error_temperature_list)
print(error_temperature_list.index)
#数据缺失与异常的位置全部设置为’NaN‘
#对’NaN‘处进行数据填充
for i in error_temperature_list.index:
	df['temperature'][i] = 'NaN'
print(df)
#删除列
df = df.drop(['subdate'],axis = 1)
```
## 机器学习
### 回归模型
线性回归模型
```py
import pandas as pd
import numpy as np
import matplotlib.pylab as plt
x = np.arange(1,10,1)
y = x*0.9+np.sin(x)
plt.plot(x,y,'o')
plt.show()
#deg=1是一阶，一次回归
model = np.polyfit(x,y,deg=1)
print(model)
#0.5为步长
x2 = np.arange(-2,12,0.5)
y2 = np.polyval(model,x2)
plt.plot(x,y,'o',x2,y2,'*')
plt.show()
```
分析世卫组织疫情数据
```py
df = pd.read_csv('H:/python/yiqing.csv',usecols=['编号‘,'累计‘],encoding='GBK')
plt.plot(df.iloc[:,1],df.iloc[:,0],'o')
plt.show()
x = df.iloc[:75,1]
y = df.iloc[:75,0]
model = np.polyfit(x,y,deg=2)
x2 = np.range(1,100,1)
y2 = np.polyval(model,x2)
print(y2)
plt.plot(x,y,'o',x2,y2,'*')
```
### KNN（k-近邻）算法
监督学习中的分类算法
1-语文高-文科
2-综合
3-数学高-理科
前80训练，预测后20个
```py
df = pd.read_csv('H:/python/fenlei.csv')
print(df,head())
train_x=df.iloc[0:80,2:4]
train_y=df.iloc[0:80,1]
from sklearn import neighbors
model = neighbors.KNeighborsClassifier()
model.fit(train_x,train_y)
#看k值
print(model.fit(train_x,train_y))
#预测
test_x = df.iloc[80:100,2:4]
test_p = model.predict(test_x)
print(test_p)
#对比一下
test_y = df.iloc[80:100,1].values
print(test_y)
#打分
p = model.score(test_x,test_y)
print(p)
```
### 聚类算法（k-means）
无监督学习-聚类算法k-means
```py
df = pd.read_csv('H:/python/fenlei.csv')
train_x = df.iloc[0:80,2:4]
train_x2 = np.array(train_x[['yuwen','shuxue']])
#看看形式
print(train_x2.shape)
from sklearn.cluster import KMeans
model = KMeans(n_clusters=3)
print(model)
print(model.labels_)
train_y = df.iloc[0:80,1]
#聚类形成的中心
print(model.cluster_centers_)
plt.plot(df.iloc[0:80,2],df.iloc[0:80,3],'o')
plt.show()
#绘制好看的彩图
def showCluster(dataSet, k, centroids, clusterAssment):
    numSamples, dim = dataSet.shape
    if dim != 2:
        print("Sorry! I can not draw because the dimension of your data is not 2!")
        return 1
    mark = ['or','ob','og','ok','^r','+r','sr','dr','<r','pr']
    if k > len(mark):
        print("Sorry! Your k is too large!")
        return 1
    for i in range(numSamples):
        markIndex = int(clusterAssment[i,0])
        plt.plot(dataSet[i,0],dataSet[i,1],mark[markIndex])
    mark = ['Dr','Db','Dg','Dk','^b','+b','sb','db','<b','pb']
    for i in range(k):
        plt.plot(centroids[i,0],centroids[i,1],mark[i],markeredgecolor='k',markersize=15)
    plt.show()
train_x_array = np.array(train_x[['yuwen','shuxue']])
showCluster(train_x_array,3,model.cluster_centers_,model.lebels_.reshape(-1,1))
```
### 决策树
监督模型：决策树
```py
df = pd.read_csv('H:/python/fenlei.csv')
train_x = df.iloc[0:80,2:4]
train_y=df.iloc[0:80,1]
from sklearn import tree
#用信息熵
model = tree.DecisionTreeClassifier(criterion='entropy')
model.fit(train_x,train_y)
test_x = df.iloc[80:100,2:4]
test_p = model.predict(test_x)
print(test_p)
```
## 图像处理
### 基于OpenCV的图像拼接
原理是基于OpenCV提供的SIFT算法去寻找两张图片的相似点并对一张图片进行矩阵变换后进行拼接：
```python {class=line-numbers}
import numpy as np
import cv2 as cv
from matplotlib import pyplot as plt

if __name__ == '__main__':
#重叠部分的边界像素值：
    top, bot, left, right = 100, 100, 0, 500
    img1 = cv.imread('test1.jpg')
    img2 = cv.imread('test2.jpg')
    srcImg = cv.copyMakeBorder(img1, top, bot, left, right, cv.BORDER_CONSTANT, value=(0, 0, 0))
    testImg = cv.copyMakeBorder(img2, top, bot, left, right, cv.BORDER_CONSTANT, value=(0, 0, 0))
    img1gray = cv.cvtColor(srcImg, cv.COLOR_BGR2GRAY)
    img2gray = cv.cvtColor(testImg, cv.COLOR_BGR2GRAY)
    sift = cv.xfeatures2d_SIFT().create()
    # find the keypoints and descriptors with SIFT
    kp1, des1 = sift.detectAndCompute(img1gray, None)
    kp2, des2 = sift.detectAndCompute(img2gray, None)
    # FLANN parameters
    FLANN_INDEX_KDTREE = 1
    index_params = dict(algorithm=FLANN_INDEX_KDTREE, trees=5)
    search_params = dict(checks=50)
    flann = cv.FlannBasedMatcher(index_params, search_params)
    matches = flann.knnMatch(des1, des2, k=2)

    # Need to draw only good matches, so create a mask
    matchesMask = [[0, 0] for i in range(len(matches))]

    good = []
    pts1 = []
    pts2 = []
    # ratio test as per Lowe's paper
    for i, (m, n) in enumerate(matches):
        if m.distance < 0.7*n.distance:
            good.append(m)
            pts2.append(kp2[m.trainIdx].pt)
            pts1.append(kp1[m.queryIdx].pt)
            matchesMask[i] = [1, 0]

    draw_params = dict(matchColor=(0, 255, 0),
                       singlePointColor=(255, 0, 0),
                       matchesMask=matchesMask,
                       flags=0)
    img3 = cv.drawMatchesKnn(img1gray, kp1, img2gray, kp2, matches, None, **draw_params)
    plt.imshow(img3, ), plt.show()

    rows, cols = srcImg.shape[:2]
    MIN_MATCH_COUNT = 10
    if len(good) > MIN_MATCH_COUNT:
        src_pts = np.float32([kp1[m.queryIdx].pt for m in good]).reshape(-1, 1, 2)
        dst_pts = np.float32([kp2[m.trainIdx].pt for m in good]).reshape(-1, 1, 2)
        M, mask = cv.findHomography(src_pts, dst_pts, cv.RANSAC, 5.0)
        warpImg = cv.warpPerspective(testImg, np.array(M), (testImg.shape[1], testImg.shape[0]), flags=cv.WARP_INVERSE_MAP)

        for col in range(0, cols):
            if srcImg[:, col].any() and warpImg[:, col].any():
                left = col
                break
        for col in range(cols-1, 0, -1):
            if srcImg[:, col].any() and warpImg[:, col].any():
                right = col
                break

        res = np.zeros([rows, cols, 3], np.uint8)
        for row in range(0, rows):
            for col in range(0, cols):
                if not srcImg[row, col].any():
                    res[row, col] = warpImg[row, col]
                elif not warpImg[row, col].any():
                    res[row, col] = srcImg[row, col]
                else:
                    srcImgLen = float(abs(col - left))
                    testImgLen = float(abs(col - right))
                    alpha = srcImgLen / (srcImgLen + testImgLen)
                    res[row, col] = np.clip(srcImg[row, col] * (1-alpha) + warpImg[row, col] * alpha, 0, 255)

        # opencv is bgr, matplotlib is rgb
        res = cv.cvtColor(res, cv.COLOR_BGR2RGB)
        # show the result
        plt.figure()
        plt.imshow(res)
        plt.show()
    else:
        print("Not enough matches are found - {}/{}".format(len(good), MIN_MATCH_COUNT))
        matchesMask = None
```
更精确的拼接也许需要OpenPano算法，以后再说