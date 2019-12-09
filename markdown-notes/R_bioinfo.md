# 生物信息学
[TOC]
## 系统发育数据
原作者：Jesús N. Pinto-Ledezma and Jeannine Cavender-Bares
了解数据对于研究生物多样性非常重要，而现在使用的一个常见数据是描述谱系之间以及谱系之间的进化关系的系统发育树。从这里到本简短教程的结尾，我们将尝试解释如何导入/导出和处理系统发育信息的基础知识。
## 系统发育广义最小二乘法（PGLS）
来源：R course in Ilhabela, Brazil, June 2015
首先，我们需要安装一些程序包
```r
library(ape)
library(geiger)
library(nlme)
library(maps)
library(phytools)
```
然后我们需要一些数据进行处理，这里可以使用安乐蜥属（*Anolis*）的数据和系统发育树进行尝试。相关的文件在[我的github](https://github.com/Vendredii/Rstats)。
```r
anoleData <- read.csv('/Users/desktop/r/anolisDataAppended.csv',row.name = 1)
anoleTree <- read.tree('/Users/desktop/r/anolis.phy')
#显示树
plot(anoleTree)
#Geiger有一个函数可以查看树内的物种是否和数据内的物种名匹配
name.check(anoleTree, anoleData)
#[1] "OK"
#我们可以查看数据awesomeness和hostility是否关联
plot(anoleData[,c("awesomeness", "hostility")])
#如果关联，那么我们可以检测它们的PIC（多态信息量）
#提取列
host<-anoleData[,"hostility"]
awe<-anoleData[,"awesomeness"]
#赋名
names(host)<-names(awe)<-rownames(anoleData)
#计算PIC
hPic<-pic(host, anoleTree)
aPic<-pic(awe, anoleTree)
#建模（回归）
picModel<-lm(hPic~aPic-1)
#结果
summary(picModel)
#lm(formula = hPic ~ aPic - 1)
#
#Residuals:
#    Min      1Q  Median      3Q     Max 
#-2.1051 -0.4188  0.0103  0.3137  4.9991 
#
#Coefficients:
#     Estimate Std. Error t value Pr(>|t|)    
#aPic -0.97758    0.04516  -21.65   <2e-16 ***
#---
#Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ #0.1 ‘ ’ 1
#
#Residual standard error: 0.8967 on 98 degrees of freedom
#Multiple R-squared:  0.827,	Adjusted R-squared:  0.8253 
#F-statistic: 468.6 on 1 and 98 DF,  p-value: < 2.2e-16
#绘图
plot(hPic~aPic)
abline(a=0, b=coef(picModel))
```
以上全部的过程就是PGLS的流程，我们可以用PGLS将它们很简单地完成：
```r
pglsModel<-gls(hostility~awesomeness, correlation=corBrownian(phy=anoleTree), data=anoleData, method="ML")
summary(pglsModel)
#Generalized least squares fit by maximum likelihood
#  Model: hostility ~ awesomeness 
#  Data: anoleData 
#       AIC     BIC    logLik
#  190.9775 198.793 -92.48875
#
#Correlation Structure: corBrownian
# Formula: ~1 
# Parameter estimate(s):
#numeric(0)
#
#Coefficients:
#                 Value  Std.Error    t-value p-value
#(Intercept)  0.1505613 0.26262709   0.573289  0.5678
#awesomeness -0.9775847 0.04515861 -21.647804  0.0000
#
# Correlation: 
#            (Intr)
#awesomeness -0.042
#
#Standardized residuals:
#        Min          Q1         Med          Q3         Max 
#-0.76019997 -0.39056977 -0.04941607  0.19596725  1.07373699 
#
#Residual standard error: 0.8877372 
#Degrees of freedom: 100 total; 98 residual
#读取之前的回归系数
coef(pglsModel)
plot(host~awe)
abline(a=coef(pglsModel)[1], b=coef(pglsModel)[2])
```
但是PGLS的功能更多，我们还可以给数据增加anova分析来判断离散程度,也可以同时模拟多个变量：
```r
pglsModel2<-gls(hostility~ecomorph, correlation=corBrownian(phy=anoleTree), data=anoleData, method="ML")
anova(pglsModel2)
#Denom. DF: 93 
#            numDF    F-value p-value
#(Intercept)     1 0.01986379  0.8882
#ecomorph        6 0.23482069  0.9641
coef(pglsModel2)
#(Intercept)  ecomorphGB   ecomorphT  ecomorphTC  ecomorphTG  ecomorphTW 
#  0.4843515  -0.6315992  -1.0585278  -0.8558138  -0.4085610  -0.4039460 
#  ecomorphU 
# -0.7021719
#同时拟合多个变量
pglsModel3<-gls(hostility~ecomorph*awesomeness, correlation=corBrownian(phy=anoleTree), data=anoleData, method="ML")
anova(pglsModel3)
#Denom. DF: 86 
#                     numDF  F-value p-value
#(Intercept)              1   0.1416  0.7076
#ecomorph                 6   1.6740  0.1371
#awesomeness              1 549.8314  <.0001
#ecomorph:awesomeness     6   4.5226  0.0005
```
我们还可以假设错误结构遵循OU模型而不是布朗运动：
```r
#不要收敛，不然难以固定
pglsModelLambda<-gls(hostility~awesomeness, correlation=corPagel(1, phy=anoleTree, fixed=FALSE), data=anoleData, method="ML")
#这是规模问题。我们可以通过增加分支长度来快速固定参数。除了重新调整讨厌的参数外，这不会影响分析。
tempTree<-anoleTree
tempTree$edge.length<-tempTree$edge.length * 100
pglsModelLambda<-gls(hostility~awesomeness, correlation=corPagel(1, phy=tempTree, fixed=FALSE), data=anoleData, method="ML")
summary(pglsModelLambda)
#Generalized least squares fit by maximum likelihood
#  Model: hostility ~ awesomeness 
#  Data: anoleData 
#       AIC      BIC    logLik
#  72.56056 82.98124 -32.28028
#
#Correlation Structure: corPagel
# Formula: ~1 
# Parameter estimate(s):
#    lambda 
#-0.1585633 
#
#Coefficients:
#                 Value  Std.Error    t-value p-value
#(Intercept)  0.0612470 0.01581847   3.871868   2e-04
#awesomeness -0.8776519 0.03104246 -28.272628   0e+00
#
# Correlation: 
#            (Intr)
#awesomeness -1    
#
#Standardized residuals:
#        Min          Q1         Med          Q3         Max 
#-1.78946302 -0.71477505  0.00309539  0.78509306  2.23215144 
#
#Residual standard error: 0.3709858 
#Degrees of freedom: 100 total; 98 residual
pglsModelOU<-gls(hostility~awesomeness, correlation=corMartins(1, phy=tempTree), data=anoleData, method="ML")
summary(pglsModelOU)
#Generalized least squares fit by maximum likelihood
#  Model: hostility ~ awesomeness 
#  Data: anoleData 
#       AIC      BIC    logLik
#  96.63478 107.0555 -44.31739
#
#Correlation Structure: corMartins
# Formula: ~1 
# Parameter estimate(s):
#   alpha 
#4.441625 
#
#Coefficients:
#                 Value  Std.Error    t-value p-value
#(Intercept)  0.1084258 0.03952884   2.742954  0.0072
#awesomeness -0.8811632 0.03657646 -24.090988  0.0000
#
# Correlation: 
#            (Intr)
#awesomeness -0.269
#
#Standardized residuals:
#       Min         Q1        Med         Q3        Max 
#-1.8664557 -0.8132899 -0.1103815  0.6474918  2.0919152 
#
#Residual standard error: 0.376904 
#Degrees of freedom: 100 total; 98 residual
```