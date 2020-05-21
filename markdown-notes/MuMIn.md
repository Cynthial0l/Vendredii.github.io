## MuMIn函数包使用教程

翻译自[谷歌](https://sites.google.com/site/rforfishandwildlifegrads/home/mumin_usage_examples)，似乎是MuMIn包作者本人写的教程，侵删。

MuMIn是一个相当灵活的R包，用于对各种线性模型（包括普通线性回归和广义混合模型）进行模型选择和模型平均。如果您不知道后者是什么，请不必担心，本教程仍然有用。我将假定你对R有点熟悉，并且知道以下内容：`＃`用于向R代码添加注释，R脚本的使用与安装，工作目录的指定，以及如何读写数据文件。如果不是这样，你可能需要再学点基础的。在继续本教程之前，请确保已安装MuMIn和R软件包lme4。您可能还需要下载两个用逗号分隔的示例数据文件：[example data.csv](https://sites.google.com/site/rforfishandwildlifegrads/home/mumin_usage_examples/Example%20data.csv?attredirects=0&d=1)和[Westslope.csv](https://sites.google.com/site/rforfishandwildlifegrads/home/week-8/Westslope.csv?attredirects=0&d=1)。包含以下所有代码的脚本位于[此处](https://sites.google.com/site/rforfishandwildlifegrads/home/mumin_usage_examples/Example%20code%20for%20MuMIn%20package.R?attredirects=0&d=1)，并且此网站的pdf副本位于[此处](https://sites.google.com/site/rforfishandwildlifegrads/home/mumin_usage_examples/Using%20R%20package%20MuMIn.pdf?attredirects=0&d=1)。

为了演示MuMIn中的许多有用功能，让我们使用示例数据。
```r
#直接从网上下载数据
dater <- read.csv("https://sites.google.com/site/rforfishandwildlifegrads/home/mumin_usage_examples/Example%20data.csv?attredirects=0&d=1")
#或者下载后从本地读取
#dater<-read.csv("Example data.csv")
#简单看看数据
head(dater)
```
数据包括3个响应变量：数量，密度和存在度。以及5个解释变量：elev（海拔），slope（坡度），面积，距离（至最近的种群的）和覆盖率（％）。一个好的做法是使用summary来查看关于数据的摘要，以确保没有任何丢失的数据。
```r
#确保所有数值都没有丢失
summary(dater)
```
使用“require”或“library”功能加载程序包。
```r
#加载“MuMIn包”
require(MuMIn)
```
接下来应该做的一件事是更改R函数处理丢失数据的方式的全局选项。通过进行此更改，如果缺少数据，该功能将不起作用。如果你使用“dredge”功能进行探索性数据分析，则这是必需的。
```r
#更改“na. action”函数，使得有NA时会报错
options(na.action = "na.fail")
```
好，我们准备出发了。让我们拟合四个解释动物密度变化的候选模型。理想情况下，这些模型将代表假设。根据响应的性质，我们将普通线性回归与“lm”函数结合使用。
```r
#首先，拟合4个候选线性模型以解释密度变化
mod1<-lm(density~distance+elev, data = dater)
mod2<-lm(density~slope+pct.cover, data = dater)
mod3<-lm(density~slope+distance, data = dater)
mod4<-lm(density~slope+distance+elev, data = dater)
```
现在，我们可以使用mod.sel或model.sel函数（相同）进行模型选择。默认的模型选择标准是Akaike的信息标准（AIC），并带有较小的样本偏差调整AICc。 在这里，我们将创建一个包含所有模型选择信息的对象“out.put”。
```r
#使用mod.sel函数进行模型选择
#并将输出放入对象out.put
out.put<-mod.sel(mod1,mod2,mod3,mod4)
#看看结果如何，这会是带有小样本偏差调整AICc和AIC
out.put
> Model selection table
  (Int) dst elv pct.cvr slp df logLik AICc delta
  mod1 -0.013280 5.195e-05 1.832e-05 4 863.907 -1719.7 0.00
  mod4 -0.013080 5.181e-05 2.002e-05 -0.01096 5 863.944 -1717.6 2.01
  mod3 -0.012920 5.207e-05 0.08634 4 860.130 -1712.1 7.55
  mod2 0.003614 2.777e-05 0.05374 4 819.492 -1630.8 88.83
  weight
  mod1 0.720
  mod4 0.264
  mod3 0.016
  mod2 0.000
  Models ranked by AICc(x)
```
结果显示在上方`>`后，模型从最佳（顶部）到最差（底部）分类。 看起来像mod1最好,包含距离（dst），仰角（elev）两个预测变量，截距权重为0.72。 密度变化的最好解释（假设）是0.72 / 0.264 = 2.72次。请注意，该函数不使用全名作为模型参数，而是创建缩写。 我们经常需要通过使用某些规则指定模型的置信度集来表达模型选择的不确定性。在这里，我们可以使用子集函数来选择符合条件的模型。请注意，**权重针对所选模型进行了重新标准化**。即，将它们调整为加在一起。
```r
#使用子集函数创建模型的置信度集
#选择deltaAICc小于5的模型
#重要提示：权重已重新归一化！
subset(out.put, delta <5)
> Model selection table
  (Int) dst elv slp df logLik AICc delta weight
  mod1 -0.01328 5.195e-05 1.832e-05 4 863.907 -1719.7 0.00 0.732
  mod4 -0.01308 5.181e-05 2.002e-05 -0.01096 5 863.944 -1717.6 2.01 0.268
  Models ranked by AICc(x)
#使用Royall的1/8规则选择模型以获取证据强度
#重要提示：权重已重新归一化！
subset(out.put, 1/8 < weight/max(out.put$weight))
> Model selection table
  (Int) dst elv slp df logLik AICc delta weight
  mod1 -0.01328 5.195e-05 1.832e-05 4 863.907 -1719.7 0.00 0.732
  mod4 -0.01308 5.181e-05 2.002e-05 -0.01096 5 863.944 -1717.6 2.01 0.268
  Models ranked by AICc(x)
```
与上述delta < 5并无太大差异。让我们尝试基于模型权重的总和的另一个条件。
```r
#选择95％累积权重的模型
#重要提示：权重已重新归一化！
> Model selection table
  (Int) dst elv df logLik AICc delta weight
  mod1 -0.01328 5.195e-05 1.832e-05 4 863.907 -1719.7 0 1
  Models ranked by AICc(x)
```
在大多数情况下，你希望将模型选择结果包括在报告，出版物或论文的表格中。在这里，我们需要将mod.sel函数的输出强制转换为data.frame。该数据帧的前c个元素包含我们想要的内容。我怎么知道这些的呢？我首先创建了数据框，然后使用“str”函数查看数据框中的元素。
```r
#将对象强制输出到数据帧中
#out.put中的6-10行的元素中有我们想要的内容
sel.table<-as.data.frame(out.put)[6:10]
sel.table
>      df logLik AICc delta weight
  mod1 4 863.9067 -1719.653 0.000000 7.196856e-01
  mod4 5 863.9437 -1717.646 2.006937 2.638408e-01
  mod3 4 860.1296 -1712.099 7.554120 1.647353e-02
  mod2 4 819.4916 -1630.823 88.830152 3.697604e-20
```
这有点混乱，无法准备任何报告。让我们先整理一下。
```r
#稍微清理一下，用round函数四舍五入
sel.table[,2:3]<- round(sel.table[,2:3],2)
sel.table[,4:5]<- round(sel.table[,4:5],3)
#这样就更好看了
> sel.table df logLik AICc delta weight
  mod1 4 863.91 -1719.65 0.000 0.720
  mod4 5 863.94 -1717.65 2.007 0.264
  mod3 4 860.13 -1712.10 7.554 0.016
  mod2 4 819.49 -1630.82 88.830 0.000
#如何稍微重命名列以适合适当的约定
# number of parameters (df) should be K
names(sel.table)[1] = "K"
##确保将模型名称放在一列中
sel.table$Model<-rownames(sel.table)
#用公式替换模型名称有点棘手，所以要小心
for(i in 1:nrow(sel.table)) sel.table$Model[i]<- as.character(formula(paste(sel.table$Model[i])))[3]
#结果
sel.table
>      df logLik AICc delta weight Model
  mod1 4 863.91 -1719.65 0.000 0.720 distance + elev
  mod4 5 863.94 -1717.65 2.007 0.264 slope + distance + elev
  mod3 4 860.13 -1712.10 7.554 0.016 slope + distance
  mod2 4 819.49 -1630.82 88.830 0.000 slope + pct.cover
#列的重新排序很少
sel.table<-sel.table[,c(6,1,2,3,4,5)]
sel.table
> Model                        K logLik AICc delta weight
  mod1 distance + elev         4 863.91 -1719.65 0.000 0.720
  mod4 slope + distance + elev 5 863.94 -1717.65 2.007 0.264
  mod3 slope + distance        4 860.13 -1712.10 7.554 0.016
  mod2 slope + pct.cover       4 819.49 -1630.82 88.830 0.000
```
现在各个模型从最佳拟合到最差拟合排序。你通常可以删除AICc，因为它位于表中的Log Likelihood（logLik），但我们将其保留在此处。我们已准备好生成报告，因此可以将其写入以逗号分隔的文件中。
```r
#写入文件，此处为逗号分隔的值格式
#确保正确指定了您的工作目录
write.csv(sel.table,"My model selection table.csv", row.names = F)
```
所有MuMIn功能的默认模型选择标准是AICc。如果你不喜欢或没有其他收藏夹，则可以使用mod.sel中的“排名”选项指定该方法。以下是使用贝叶斯或Schwartz信息标准（BIC）一致的AIC，Fishers信息矩阵（CAICF）和准AIC（QAIC，下面将详细介绍）选择模型的代码。
```r
#用标准BIC进行模型选择
mod.sel(mod1,mod2,mod3,mod4, rank = BIC) Model selection table
>         (Int)     dst      elv   pct.cvr slp df logLik BIC delta
  mod1 -0.013280 5.195e-05 1.832e-05 4 863.907 -1705.6 0.00
  mod4 -0.013080 5.181e-05 2.002e-05 -0.01096 5 863.944 -1700.2 5.47
  mod3 -0.012920 5.207e-05 0.08634 4 860.130 -1698.1 7.55
  mod2  0.003614 2.777e-05 0.05374 4 819.492 -1616.8 88.83
  weight
  mod1 0.919
  mod4 0.060
  mod3 0.021
  mod2 0.000
  Models ranked by BIC(x)
#AIC与Fishers信息矩阵一致
mod.sel(mod1,mod2,mod3,mod4, rank = CAICF) Model selection table
> (Int) dst elv pct.cvr slp df logLik CAICF delta
  mod3 -0.012920 5.207e-05 0.08634 4 860.130 -1642.9 0.00
  mod1 -0.013280 5.195e-05 1.832e-05 4 863.907 -1633.1 9.80
  mod4 -0.013080 5.181e-05 2.002e-05 -0.01096 5 863.944 -1619.2 23.66
  mod2 0.003614 2.777e-05 0.05374 4 819.492 -1565.5 77.37
  weight
  mod3 0.993
  mod1 0.007
  mod4 0.000
  mod2 0.000
  Models ranked by CAICF(x)
```
还有一些MuMin函数可用于计算模型选择标准，例如AIC，AICc，BIC和Mallows Cp（不建议使用的临时模型选择标准）。
```r
#Mallows Cp
Cp(mod4)
> [1] 0.01757519

#AIC
AIC(mod1,mod2)
> df AIC
  mod1 4 -1719.813
  mod2 4 -1630.983
```
请注意，在df上方实际上是模型参数的数量，通常定义为K。
```r
# CAICF
CAICF(mod1, mod2)
> CAICF
  mod1 -1633.082
  mod2 -1565.503
```
各个参数的相对重要性也可以使用模型权重进行检查。在此，对包含感兴趣参数的每个模型的Akaike权重进行求和。这些已定义为重要性权重，您可以使用“importance”函数从mod.sel对象中获取它们。
```r
#各个预测变量的重要性权重
#使用重要性函数计算
importance(out.put)
> importance(out.put) distance elev slope pct.cover
           Importance:    1    0.98  0.28  <0.01
  N containing models:    3      2     3     1
```
查看上面的输出，有足够的证据证明距离和高度（权重接近1）是影响密度的因子，而pct.cover则少得多。参数出现的候选模型的数量可能对重要性权重产生很大影响。例如，截距包含在所有模型中，因此重要性权重为1（因此从未显示）。在以上输出中，pct.cover仅在一种模型中，因此请谨慎解释权重。
模型平均是合并模型选择不确定性的一种手段。 在此，使用每个候选模型的相应模型权重对参数估计值进行加权并求和。 Burnham和Anderson定义了两种用于模型平均的方法，其中，对发生预测变量xj的所有模型进行参数估计，并对所有模型进行预测，而不只是对发生预测变量xj的模型进行参数估计。 MuMIn函数model.avg进行两种类型的模型平均，并将第一种类型的模型平均报告为“子集”，将第二种类型的模型报告为“完整”。
```r
#使用所有候选模型的模型平均值，请始终使用modified.var = TRUE
MA.ests <- model.avg(out.put, revised.var = TRUE)
> Call:
  model.avg.model.selection(object = out.put, revised.var = TRUE)
  Component models:
  ‘12’ ‘124’ ‘14’ ‘34’
  Coefficients:
  (Intercept) distance elev slope pct.cover
  subset -0.01322053 5.191484e-05 1.877402e-05 -0.005241959 2.777260e-05
  full -0.01322053 5.191484e-05 1.846475e-05 -0.001469396 1.026921e-24
```
上面是两种类型的模型平均系数。
这是你的估计值，无条件标准误差，你需要调整后的SE和上下CL.
```r
MA.ests$avg.model
> Estimate Std. Error Adjusted SE Lower CI Upper CI
  (Intercept) -1.322053e-02 2.196457e-03 2.207049e-03 -1.754627e-02 -8.894795e-03
  distance 5.191484e-05 5.221077e-06 5.246295e-06 4.163229e-05 6.219739e-05
  elev 1.877402e-05 4.910574e-06 4.933771e-06 9.104006e-06 2.844403e-05
  slope -5.241959e-03 4.582284e-02 4.598957e-02 -9.537986e-02 8.489594e-02
  pct.cover 2.777260e-05 2.716872e-05 2.729983e-05 -2.573408e-05 8.127928e-05
#这是Beta tilda bar MA的估算值
MA.ests$coef.shrinkage
> (Intercept) distance elev slope pct.cover
  -1.322053e-02 5.191484e-05 1.846475e-05 -1.469396e-03 1.026921e-24
#您还可以获取各个参数的重要性权重
MA.ests$importance
>                     distance elev slope pct.cover
           Importance:    1    0.98  0.28  <0.01
  N containing models:    3     2     3     1
```
我们可以使用其他选项来选择并创建仅根据我们的信任集中的那些模型创建的复合模型。 例如
```r
#仅使用子集命令为模型的置信度集中的参数创建模型的平均估计值
MA.ests<-model.avg(out.put, subset= delta < 5, revised.var = TRUE)
MA.ests$avg.model
> Estimate Std. Error Adjusted SE Lower CI Upper CI
  (Intercept) -1.322554e-02 2.194158e-03 2.204742e-03 -1.754676e-02 -8.904326e-03
  distance 5.191232e-05 5.219487e-06 5.244699e-06 4.163290e-05 6.219174e-05
  elev 1.877402e-05 4.910574e-06 4.933771e-06 9.104006e-06 2.844403e-05
  slope -1.096031e-02 4.060037e-02 4.079708e-02 -9.092112e-02 6.900050e-02
#让我们整理一点并将表写入文件
MA.est.table<-round(MA.ests$avg.model[,c(1,3:5)],6)
MA.est.table
>  Estimate Adjusted SE Lower CI Upper CI
  (Intercept) -0.013226 0.002205 -0.017547 -0.008904
  distance 0.000052 0.000005 0.000042 0.000062
  elev 0.000019 0.000005 0.000009 0.000028
  slope -0.010960 0.040797 -0.090921 0.069001
#输出为csv
write.csv(MA.est.table, "My model averaged estimates.csv")
```
对于正常的线性模型，可以使用模型平均参数估计来预测自变量（也称为预测变量）的各种值的响应，此处为密度。 这等效于预测每个候选模型的响应，并使用相应的模型权重平均预测值。请注意，这不适用于非正常模型，例如逻辑回归或泊松回归。我们可以使用MuMIn函数的输出对预测值进行模型平均。在这里，我们在MuMIn中使用一些功能，在基本R函数中使用一些功能。例如，以下是用于计算模型平均预测的代码。
```r
#extract parameters and weights from confidence model set
#使用get.models函数
pred.parms<-get.models(out.put, subset= delta < 5)
#使用每个模型预测值，这里仅使用示例数据集，您可以使用新的数据集
model.preds = sapply(pred.parms, predict, newdata = dater)
> model.preds mod1 mod4 mod3 mod2
  1 1.572387e-02 1.561711e-02 1.647889e-02 0.009264373
  2 8.710879e-03 8.812682e-03 8.035629e-03 0.007606412
  3 4.596332e-03 4.510560e-03 3.989456e-03 0.011867896
  4 1.125713e-02 1.133978e-02 8.993757e-03 0.011941525
  (rest of output not shown)
```
上面是对每种候选模型所做的预测（将其切掉一点）。现在我们需要通过模型权重和总和对权重进行加权（就像上面的模型平均系数一样）。最简单的方法是使用矩阵乘法。
```r
#通过其AIC权重对每个模型的预测进行加权
#“Weights”功能提取权重
#我们还使用矩阵乘法％*％
mod.ave.preds<-model.preds %*% Weights(out.put)
mod.ave.preds
> [,1]
  1 1.570814e-02
  2 8.726615e-03
  3 4.563705e-03
  4 1.124165e-02
  5 1.518054e-02
  6 7.068765e-03`
  (rest of output not shown)
```
模型平均用于绘图的一个更有趣的应用是创建一个数据集，其中将除单个预测变量（下面的海拔高度）以外的所有值都设置为其平均值。
```r
#海拔范围从观察到的最小值到最大值
elev=c(min(dater$elev):max(dater$elev))
#用平均值创建plotdata数据框
plotdata<-as.data.frame(lapply(lapply(dater[5:8],mean),rep,length(elev))))
plotdata<-cbind(elev,plotdata)
#现在可以预测每个模型的绘图数据的密度
model.preds = sapply(pred.parms, predict, newdata = plotdata)
#通过其AIC权重和和（矩阵乘法）对每个模型的预测进行加权
mod.ave4plot<-model.preds %*% Weights(out.put)
#绘制模型平均预测密度与海拔的关系
plot(mod.ave4plot~ elev, type = 'l', xlab="Elevation (m)", ylab="Model averaged predicted density")
```
MuMIn的另一个有用功能是**dredge**。但是，你仅应将其用于探索目的。强烈不鼓励进行数据挖掘，并且可能导致虚假（无关紧要的或更糟糕的是，错误的）结果和推断。因此，请阅读以下消息，请用户当心。
```r
#仅供探索用途！！！别整活
#使用所有参数拟合模型
all.parms<-lm(density~slope+distance+elev+ pct.cover, data = dater)
#dredge功能适合所有上面的all.parms模型中的变量的拟合
results<-dredge(all.parms)
results
> Model selection table
  (Int) dst elv pct.cvr slp df logLik AICc delta
  4 -0.013280 5.195e-05 1.832e-05 4 863.907 -1719.7 0.00
  8 -0.014560 5.177e-05 1.860e-05 2.362e-05 5 864.448 -1718.7 1.00
  12 -0.013080 5.181e-05 2.002e-05 -0.01096 5 863.944 -1717.6 2.01
  16 -0.014360 5.162e-05 2.044e-05 2.377e-05 -0.01187 6 864.491 -1716.6 3.01
  (rest not shown)
#获取最佳支持的模型
subset(results, delta <5)
> Model selection table
  (Int) dst elv pct.cvr slp df logLik AICc delta weight
  4 -0.01328 5.195e-05 1.832e-05 4 863.907 -1719.7 0.00 0.455
  8 -0.01456 5.177e-05 1.860e-05 2.362e-05 5 864.448 -1718.7 1.00 0.276
  12 -0.01308 5.181e-05 2.002e-05 -0.01096 5 863.944 -1717.6 2.01 0.167
  16 -0.01436 5.162e-05 2.044e-05 2.377e-05 -0.01187 6 864.491 -1716.6 3.01 0.101
  Models ranked by AICc(x)
#获取最好的模型
subset(results, delta == 0)
> Global model call: lm(formula = density ~ slope + distance + elev + pct.cover, data = dater)
  ---
  Model selection table
  (Int) dst elv df logLik AICc delta weight
  4 -0.01328 5.195e-05 1.832e-05 4 863.907 -1719.7 0 1
  Models ranked by AICc(x)
#计算可变重要性权重
> importance(results)
                      distance elev pct.cover slope
           Importance:   1.00   0.98  0.38    0.28
  N containing models:     8     8     8       8
```
上面请注意，每个参数都具有相同数量的模型。
```r
#使用其他模型选择标准
results<-dredge(all.parms, rank = BIC)
results
> Global model call: lm(formula = density ~ slope + distance + elev + pct.cover, data = dater)
  ---
  Model selection table
  (Int) dst elv pct.cvr slp df logLik BIC delta
  4 -0.013280 5.195e-05 1.832e-05 4 863.907 -1705.6 0.00
  8 -0.014560 5.177e-05 1.860e-05 2.362e-05 5 864.448 -1701.2 4.46
  12 -0.013080 5.181e-05 2.002e-05 -0.01096 5 863.944 -1700.2 5.47
  10 -0.012920 5.207e-05 0.08634 4 860.130 -1698.1 7.55
  (rest not shown)
#每个模型最多只允许3个参数，最小1个参数
results<-dredge(all.parms,m.max =3, m.min = 1)
results
> Model selection table
  (Int) dst elv pct.cvr slp df logLik AICc delta
  4 -0.013280 5.195e-05 1.832e-05 4 863.907 -1719.7 0.00
  8 -0.014560 5.177e-05 1.860e-05 2.362e-05 5 864.448 -1718.7 1.00
  12 -0.013080 5.181e-05 2.002e-05 -0.01096 5 863.944 -1717.6 2.01
  10 -0.012920 5.207e-05 0.08634 4 860.130 -1712.1 7.55
  (rest not shown)
#适合所有模型，但不包括同时具有坡度和高程的模型
results<-dredge(all.parms, subset= !(slope && elev))
results
> Model selection table
  (Int) dst elv pct.cvr slp df logLik AICc delta weight
  4 -0.013280 5.195e-05 1.832e-05 4 863.907 -1719.7 0.00 0.609
  8 -0.014560 5.177e-05 1.860e-05 2.362e-05 5 864.448 -1718.7 1.00 0.370
  10 -0.012920 5.207e-05 0.08634 4 860.130 -1712.1 7.55 0.014
  (rest not shown)
#在所有模型中都包括高程
results<-dredge(all.parms,fixed =c("elev"))
results
> Model selection table
  (Int) dst elv pct.cvr slp df logLik AICc delta weight
  2 -0.013280 5.195e-05 1.832e-05 4 863.907 -1719.7 0.00 0.455
  4 -0.014560 5.177e-05 1.860e-05 2.362e-05 5 864.448 -1718.7 1.00 0.276
  6 -0.013080 5.181e-05 2.002e-05 -0.01096 5 863.944 -1717.6 2.01 0.167
  (rest not shown) 
#使用dredge创建的对象也可以用于创建模型平均参数
MA.ests<-model.avg(results, subset= delta < 2, revised.var = TRUE)
MA.ests$avg.model 
> Estimate Std. Error Adjusted SE Lower CI Upper CI
  (Intercept) -1.376386e-02 2.373991e-03 2.384677e-03 -1.843774e-02 -9.089980e-03
  distance 5.188373e-05 5.210960e-06 5.236138e-06 4.162108e-05 6.214637e-05
  elev 1.842395e-05 3.598801e-06 3.616170e-06 1.133638e-05 2.551151e-05
  pct.cover 2.362281e-05 2.286638e-05 2.297717e-05 -2.141161e-05 6.865722e-05
```
上面的所有功能都可以与通过使用glm函数拟合线性模型而创建的对象一起使用。 以上所有内容均适用于这些glm对象。但是，你应该知道一个重要的区别。由于存在额外的差异，GLM（例如Poisson回归和Logistic回归）通常无法满足统计假设。通常将其定义为过度分散（对于正常的线性回归而言不是问题！），并且需要使用准AIC进行模型选择。这是使用泊松回归和glm函数拟合计数数据时过度分散的示例。
```r
#拟合全局泊松回归模型
global.mod<-glm(count~area+distance+elev+ slope, data = dater, family = poisson)
summary(global.mod)
> Call:
  glm(formula = count ~ area + distance + elev + slope, family = poisson,
  data = dater)
  Deviance Residuals:
  Min 1Q Median 3Q Max
  -5.6618 -2.0085 -0.9933 0.9147 3.6812
  Coefficients:
  Estimate Std. Error z value Pr(>|z|)
  (Intercept) -2.972e+00 1.113e-01 -26.709 < 2e-16 ***
  area 2.654e-03 5.533e-05 47.972 < 2e-16 ***
  distance 5.317e-03 1.552e-04 34.248 < 2e-16 ***
  elev 2.095e-03 2.765e-04 7.576 3.57e-14 ***
  slope -4.664e+00 1.589e+00 -2.936 0.00333 **
  ---
  Signif. codes: 0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1
  (Dispersion parameter for poisson family taken to be 1)
  Null deviance: 4460.39 on 254 degrees of freedom
  Residual deviance: 898.82 on 250 degrees of freedom
  AIC: 1603.1
#计算chat以评估模型假设，chat>1表示过度分散
chat<-sum(residuals(glob.mod,"pearson")^2)/glob.mod$df.residual
chat
> [1] 2.898076
```
为了说明过度分散，我们使用拟泊松回归并在glm函数中指定“拟泊松”。
```r
#使全局拟泊松回归模型适合全局模型
global.mod<-glm(count~area+distance+elev+ slope, data = dater, family = quasipoisson)
summary(global.mod)
> Call:
  glm(formula = count ~ area + distance + elev + slope, family = quasipoisson,
  data = dater)
  Deviance Residuals:
  Min 1Q Median 3Q Max
  -5.6618 -2.0085 -0.9933 0.9147 3.6812
  Coefficients:
  Estimate Std. Error t value Pr(>|t|)
  (Intercept) -2.9715045 0.1898075 -15.655 < 2e-16 ***
  area 0.0026545 0.0000944 28.118 < 2e-16 ***
  distance 0.0053167 0.0002649 20.074 < 2e-16 ***
  elev 0.0020946 0.0004717 4.440 1.35e-05 ***
  slope -4.6641697 2.7104978 -1.721 0.0865 .
  ---
  Signif. codes: 0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1
  (Dispersion parameter for quasipoisson family taken to be 2.910718)
  Null deviance: 4460.39 on 254 degrees of freedom
  Residual deviance: 898.82 on 250 degrees of freedom
  AIC: NA
```
然后，就像创建lm函数和密度响应变量一样，我们创建候选模型集。
```r
#候选集
modl2<-glm(count~area+slope, data = dater, family = quasipoisson)
modl3<-glm(count~area+distance, data = dater, family = quasipoisson)
modl4<-glm(count~area+elev, data = dater, family = quasipoisson)
#尝试使用准AICc，QAICc选择模型，请确保使用model.set函数中的rank.args提供全局模型的聊天记录
quasi.MS<-model.sel(global.mod,modl2,modl3,modl4, rank = QAICc, rank.args = alist(chat = chat))
#让我们看看我们得到了什么
as.data.frame(quasi.MS)
>            (Intercept)   area      distance     elev       slope   df logLik
  global.mod -2.9715045 0.002654473 0.005316652 0.002094616 -4.664170 5 NA
  modl2      -0.7917247 0.002441674     NA         NA        3.750308 3 NA
  modl3      -2.5867633 0.002665374 0.005324297    NA            NA   3 NA
  modl4      -0.8972105 0.002437204     NA      0.001056615      NA   3 NA
              QAICc delta weight
  global.mod    NA    NA   NA
  modl2         NA    NA   NA
  modl3         NA    NA   NA
  modl4         NA    NA   NA
```
休斯顿，我们有问题！查看所有这些出现NA的地方：QAIC，增量和权重-出问题了。这是因为glm不为拟回归（和拟二项式）回归提供对数似然性。我们需要创建一个函数来欺骗glm放弃对数可能性。
```r
#这是获得可能性并计算QAICc所必需的
x.quasipoisson <- function(...) {
res <- quasipoisson(...)
res$aic <- poisson(...)$aic
res
}
```
现在，我们使用功能和“updata”功能来设置模型选择。 这应该在每个候选模型上完成。
```r
#更新模型，以便获得对数似然
global.mod<-update(global.mod,family = "x.quasipoisson")
modl2<-update(modl2,family = "x.quasipoisson")
modl3<-update(modl3,family = "x.quasipoisson")
modl4<-update(modl4,family = "x.quasipoisson")
```
现在，我们很高兴使用QAICc选择模型。
```r
#现在进行模型选择
quasi.MS<-model.sel(global.mod,modl2,modl3,modl4, rank = QAICc, rank.args = alist(chat = chat))
as.data.frame(quasi.MS)
>             (Intercept) area distance elev slope df logLik
  global.mod -2.9715045 0.002654473 0.005316652 0.002094616 -4.664170 5 -796.5328
  modl3 -2.5867633 0.002665374 0.005324297 NA NA 3 -856.1141
  modl4 -0.8972105 0.002437204 NA 0.001056615 NA 3 -1325.0153
  modl2 -0.7917247 0.002441674 NA NA 3.750308 3 -1345.5074
  QAICc delta weight
  global.mod 562.0363 0.00000 1.000000e+00
  modl3 598.9754 36.93913 9.522895e-09
  modl4 922.5702 360.53391 5.141100e-79
  modl2 936.7121 374.67577 4.367061e-82
```
现在，我们可以像上面一样使用其他任何MuMIn函数。例如，我们可以使用子集来选择最佳模型。
```r
#获得最佳模型
subset(quasi.MS, delta == 0)
> Model selection table
                     (Intrc) area dstnc elev slope df logLik QAICc delta
  global.mod -2.972 0.002654 0.005317 0.002095 -4.664 5 -796.533 562 0
           weight
  global.mod 1
  Models ranked by QAICc(x, chat = chat)
#是的，dredge有效，但仅适用于更新的模型
dredge(global.mod, rank = "QAICc", chat = chat)
> Global model call: glm(formula = count ~ area + distance + elev + slope, family = "x.quasipoisson", data = dater)
  ---
  Model selection table
   (Intrc) area dstnc elev slope df logLik QAICc delta weight
  16 -2.9720 0.002654 0.005317 0.002095 -4.664 5 -796.533 562.0 0.00 0.612
   8 -3.0120 0.002636 0.005336 0.001364 4 -800.893 562.9 0.91 0.388
  (rest not shown)
```
MuMIn函数还可用于广义线性混合模型的模型选择。在这里，我们将使用lmer4软件包，并使用分层逻辑回归模型进行一些模型选择。确保已安装lme4。让我们加载示例数据集。
```r
#直接从网站读取数据
trout<-read.csv("http://sites.google.com/site/rforfishandwildlifegrads/home/week-8/Westslope.csv?attredirects=0&d=1")
#或读取下载的文件
#trout<-read.csv("Westslope.csv")
head(trout)
```
该数据包含内陆哥伦比亚河流域内流域内溪流的威斯洛普特（Westslope）凶猛鳟鱼的有无数据。 该文件包含以下数据：
1. 存在-物种存在（1）或不存在（0）
2. WSHD-分水岭ID
3. SOIL_PROD-生产性土壤流域的百分比
4. 梯度-河流的梯度（％）
5. WIDTH-河流的平均宽度，以米为单位
与所有模型选择练习一样，您应该首先拟合全局模型并评估模型假设，例如残差的分布，独立性等。在下面，我们拟合全局模型（使用带glmer函数的westslope数据的model1和4个候选模型 Logistic回归如此“类型（family） = 二项式（binomial）”。
```r
##拟合logistic回归并将随机效应输出到model1
model1 <-glmer(PRESENCE ~ SOIL_PROD + GRADIENT + WIDTH + (1|WSHD),data = trout, family = binomial)
summary(model1)
> Generalized linear mixed model fit by maximum likelihood (Laplace Approximation) [glmerMod]
  Family: binomial ( logit )
  Formula: PRESENCE ~ SOIL_PROD + GRADIENT + WIDTH + (1 | WSHD)
  Data: trout
  AIC BIC logLik deviance df.resid
  423.0 448.1 -206.5 413.0 1115
  Scaled residuals:
  Min 1Q Median 3Q Max
  -3.2280 -0.1067 -0.0278 -0.0053 13.9827
  Random effects:
  Groups Name Variance Std.Dev.
  WSHD (Intercept) 19.41 4.406
  Number of obs: 1120, groups: WSHD, 56
  Fixed effects:
  Estimate Std. Error z value Pr(>|z|)
  (Intercept) -6.40275 1.83118 -3.497 0.000471 ***
  SOIL_PROD 0.07282 0.03178 2.291 0.021964 *
  GRADIENT 0.32147 0.04009 8.019 1.06e-15 ***
  WIDTH -0.60649 0.07601 -7.979 1.47e-15 ***
  ---
  Signif. codes: 0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1
  Correlation of Fixed Effects:
  (Intr) SOIL_P GRADIE
  SOIL_PROD -0.808
  GRADIENT -0.362 0.156
  WIDTH 0.019 -0.140 -0.485
#拟合剩余的候选模型
model2 <-glmer(PRESENCE ~ GRADIENT + WIDTH + (1|WSHD),data = trout, family = binomial)
model3 <-glmer(PRESENCE ~ SOIL_PROD + (1|WSHD),data = trout, family = binomial)
model4 <-glmer(PRESENCE ~ SOIL_PROD + WIDTH + (1 + SOIL_PROD|WSHD),data = trout, family = binomial)
model5 <-glmer(PRESENCE ~ SOIL_PROD + WIDTH + (1 |WSHD),data = trout, family = binomial)
#使用BIC进行模型选择
my.models<-model.sel(model1,model2,model3,model4,model5,rank=BIC)
my.models
> Model selection table
          (Int) GRA SOI_PRO WID random df logLik BIC delta weight
  model2 -3.410 0.3186 -0.6040 W 4 -209.270 446.6 0.00 0.681
  model1 -6.403 0.3215 0.07282 -0.6065 W 5 -206.520 448.1 1.52 0.319
  model4 -6.667 0.12800 -0.4401 1+S_P|W 6 -257.123 556.4 109.75 0.000
  model5 -1.910 0.04778 -0.4397 W 4 -264.977 558.0 111.41 0.000
  model3 -5.233 0.04074 W 3 -309.658 640.4 193.75 0.000
  Models ranked by NULL
  Random terms:
  W = ‘1 | WSHD’
  1+S_P|W = ‘1 + SOIL_PROD | WSHD’
```
最好的近似模型是模型2，其中包含梯度，宽度和随机变化的截距。 我们还可以使用任何其他MuMIn函数，例如
```r
#计算重要性权重
importance(my.models)
>                    GRADIENT WIDTH SOIL_PROD
           Importance:  1.00   1.00    0.32
  N containing models:   2      4       4
#是的，dredge也可以在这里工作，谢谢！
dredge(model1, rank = BIC)
> Model selection table
  (Int) GRA SOI_PRO WID df logLik BIC delta weight
  6 -3.41000 0.3186 -0.6040 4 -209.270 446.6 0.00 0.681
  8 -6.40300 0.3215 0.07282 -0.6065 5 -206.520 448.1 1.52 0.319
  2 -6.80900 0.2375 3 -262.364 545.8 99.17 0.000
  4 -9.05000 0.2390 0.05337 4 -259.790 547.7 101.04 0.000
  5 0.07927 -0.4396 3 -267.178 555.4 108.79 0.000
  7 -1.91000 0.04778 -0.4397 4 -264.977 558.0 111.41 0.000
  1 -3.54100 2 -311.858 637.8 191.13 0.000
  3 -5.23300 0.04074 3 -309.658 640.4 193.75 0.000
  Models ranked by BIC(x)
  Random terms (all models):
  ‘1 | WSHD’
```
一句话警告，不要为混合模型估算模型平均参数！ 但是，您可以对GLMM的预测进行平均模型化。