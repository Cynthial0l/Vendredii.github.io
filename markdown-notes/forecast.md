# 预测：原理与实践
## 前言
来自澳大利亚莫纳什大学的教材[Forecast：Principles and Practice](https://otexts.com/fpp2/index.html)
所有计算与建模均基于R语言，可以通过在CRAN上安装fpp2软件包以便进行下面所有的操作。
***
Hyndman, R.J., & Athanasopoulos, G. (2018) Forecasting: principles and practice, 2nd edition, OTexts: Melbourne, Australia. OTexts.com/fpp2.
[TOC]
## 入门
### 关于预测
有些预测很简单，而有些预测很难，事件的预测取决于以下三个因素：
1. 我们对造成这种情况的因素的了解程度（对变量的了解
2. 有多少数据可供预测
3. 预测是否会影响我们要预测的事物

一个好的预测模型可以捕捉事物变化的方式，我们通常假定环境变化的方式将持续到未来，即高度波动的环境将继续高度波动...等。
如果没有可用数据，或可用数据与预测不相关，则必须使用**定性预测**的方法，而当有关于过去的数字信息且有理由假设过去模式的某些方面将持续到将来时，则可以采用**定量预测**的方式，定量预测可以通过时间序列数据进行预测。
在预测时，我们通常不直接指出预测的值，而是给出预测间隔，该间隔给出了随机变量可以相对较高的概率获取的一系列值。例如，一个95％的预测间隔包含一系列值，其中应包括概率为95％的实际未来值。其中给了一条线表示了各个预测结果的平均值，这种预测方法称**点预测**。

预测任务可以分解为以下5个步骤：
1. 问题定义：怎么预测
2. 搜集信息：相关专业知识与数据
3. 初步分析：绘制图表，探讨趋势
4. 选择模型：就是选择模型啦
5. 使用模型：并评估其效果

## 时间序列图像的绘制
### ts对象
时间可以作为`ts`对象存储与r中，如2012年的数据为10，2013年为11，2014年为15：
```r
y <- ts(c(10,11,15), start = 2012)
```
对于每年进行一次以上的观察，我们需要添加`fraquency`参数，如每月数据我们储存为了`z`，则：
```r
y <- ts(z, start = 2012, frequency = 12)
```
备注：一年有52周

### 时间图（折线）
对于时间序列数据，首先应当绘制相关的时间图。即将观察值相对于观察时间绘制，连续观察由直线连接。下图显示了澳大利亚两个最大城市之间的安捷航空每周的经济客运量。
![plot1](Forecast/Rplot1.jpeg)
```r
#autoplot可以自动生成合适的图表
autoplot(melsyd[,"Economy.Class"]) +
  ggtitle("Economy class passengers: Melbourne-Sydney") +
  xlab("Year") +
  ylab("Thousands")
```
由图片可以知道：
1. 1989年有一段时期没有乘客载运-这是由于劳资纠纷造成的。
2. 在1992年有一段时间，载客量有所减少。这是由于试验将某些经济舱座位替换为商务舱座位。
3. 1991年下半年客运量大大增加。
4. 每年年初，负载都有一些大的下降。这些是由于假期的影响。
5. 该系列的水平存在长期波动，该波动在1987年增加，在1989年减少，并在1990年和1991年再次增加。
6. 有些时期缺少观察结果。

也有一些简单的时间序列图：
```r
autoplot(a10) +
  ggtitle("Antidiabetic drug sales") +
  ylab("$ million") +
  xlab("Year")
```
![plot2](Forecast/Rplot2.jpeg)
这是澳大利亚抗糖尿病药物的月销售额。显然这里的趋势是不断增长，每年年初的下降则与政府年初的补贴有关。

对于那些有季节性变化趋势的数据，我们可以将不同年份的数据进行对比，如上面的澳大利亚糖尿病药物数据：
```r
ggseasonplot(a10, year.labels=TRUE, year.labels.left=TRUE) +
  ylab("$ million") +
  ggtitle("Seasonal plot: antidiabetic drug sales")
```
![plot3](Forecast/Rplot3.jpeg)
我们可以直观的观察到，在这种情况下，很明显，每年一月份的销售额都有很大的增长。实际上，这些可能是12月下旬的销售，因为客户在年末之前有库存，但是直到一两周后才向政府注册销售。该图还显示，2008年3月的销售额异常少（大多数其他年份显示2月至3月之间有所增长）。2008年6月的销售量很少，可能是由于在收集数据时对销售的计数不完整。

也许我们还可以用极坐标来显示：
```r
ggseasonplot(a10, polar=TRUE) +
  ylab("$ million") +
  ggtitle("Polar seasonal plot: antidiabetic drug sales")
```
![plot4](Forecast/Rplot4.jpeg)
也可以把每个季节的数据搜集在一起来显示：
```r
ggsubseriesplot(a10) +
  ylab("$ million") +
  ggtitle("Seasonal subseries plot: antidiabetic drug sales")
```
![plot5](Forecast/Rplot5.jpeg)
### 散点图
散点图可以探索不同因素的时间序列图像之间的关系。
下图显示了两个时间序列：澳大利亚维多利亚州2014年的半小时用电需求（千兆瓦）和温度（摄氏度）。温度是墨尔本（维多利亚州最大的城市）的温度，而需求值是整个州的温度。
```r
autoplot(elecdemand[,c("Demand","Temperature")], facets=TRUE) +
  xlab("Year: 2014") + ylab("") +
  ggtitle("Half-hourly electricity demand: Victoria, Australia")
```
![plot6](Forecast/Rplot6.jpeg)
我们可以通过绘制一个序列与另一个序列的散点图来研究用电需求与温度之间的关系。
```r
qplot(Temperature, Demand, data=as.data.frame(elecdemand)) +
  ylab("Demand (GW)") + xlab("Temperature (Celsius)")
```
![plot7](Forecast/Rplot7.jpeg)
此散点图有助于我们可视化变量之间的关系。显然，由于空调的影响，当温度高时会出现高需求。但是相反的，对于非常低的温度，需求增加。
相关系数就是衡量两个变量之间是否是线性关系的一种手段，相关系数仅测量线性关系的强度，有时会产生误导，因此还需要查看数据图来获得更多详细结论。

散点图矩阵
当存在多个潜在的预测变量时，将每个变量相对于另一个变量作图很有用。下图显示了五个时间序列，该序列显示了澳大利亚新南威尔士州五个地区的季度访客人数。
```r
autoplot(visnights[,1:5], facets=TRUE) +
  ylab("Number of visitor nights each quarter (millions)")
```
![plot8](Forecast/Rplot8.jpeg)
显示它们的散点图矩阵（需要GGally包）
```r
GGally::ggpairs(as.data.frame(visnights[,1:5]))
```
![plot9](Forecast/Rplot9.jpeg)
散点图矩阵的值是可以快速查看所有变量对之间的关​​系。在此示例中，曲线的第二列显示，新南威尔士州北部海岸的游客和新南威尔士州南部海岸的游客之间存在很强的正相关关系，但新南威尔士州北部海岸的游客和新南威尔士州南部内陆的游客之间没有可检测的关系。异常值也可以看到。新南威尔士州都会区有一个异常高的季度，与2000年悉尼奥运会相对应。
### 滞后图
滞后嘛，就是滞后，比如5月数据一阶滞后后就算到4月里去了，那种意思。下图显示了澳大利亚啤酒季度产量的散点图，其中横轴显示了时间序列的滞后值。
```r
beer2 <- window(ausbeer, start=1992)
gglagplot(beer2)
```
![plot10](Forecast/Rplot10.jpeg)
我们可以发现啤酒产量有4个月的滞后期，即生产4个月后可能才进入销售渠道。
### 自相关
自相关性度量时间序列的滞后值之间的线性关系，即数据是否自己有规律地变动。
对于上图中的九个散点图分别求相关系数可得其自相关情况。
```r
ggAcf(beer2)
```
![plot11](Forecast/Rplot11.jpeg)
蓝色虚线外表示相关性是否显着不为零。

当数据具有趋势时，小的滞后的自相关往往会很大并且是正的，因为及时附近的观测值的大小也很近。因此，趋势时间序列的ACF倾向于具有正值，而正值随着滞后的增加而逐渐降低。

当数据是季节性的时，季节性滞后（以季节性频率的倍数）的自相关将大于其他滞后。

如澳大利亚每月电力需求及其自相关：

```r
aelec <- window(elec, start=1980)
autoplot(aelec) + xlab("Year") + ylab("GWh")
```
![plot12](Forecast/Rplot12.jpeg)
```r
ggAcf(aelec, lag=48)
```
![plot13](Forecast/Rplot13.jpeg)
### 白噪声
没有自相关的时间序列称为白噪声（R6里的white noise）
如：
```r
set.seed(30)
y <- ts(rnorm(50))
autoplot(y) + ggtitle("White noise")
```
![plot14](Forecast/Rplot14.jpeg)
```r
ggAcf(y)
```
![plot15](Forecast/Rplot15.jpeg)
对于白噪声系列，我们希望每个自相关接近于零。当然，由于存在一些随机变化，它们将不完全等于零。对于白噪声系列，我们预计ACF中95％的尖峰位于$\pm 2/\sqrt{\check{\textup{T}}}$内,其中$\check{\textup{T}}$是时间序列的长度。