# 中国被子植物的灭绝风险研究
我的第一篇SCI，希望可以发2区

[TOC]
## 数据准备
数据结构为物种名与其相应的各个环境和性状相匹配后组成的表格。
原始数据为栅格化的中国dem数据，gdp数据，人口密度数据，人类活动足迹数据，年均温数据，年降水数据，NDVI数据，经过ArcGIS栅格化处理后统一为100km*100km的grid。
被子植物生长性状数据包括生长型（木本0，草本1），是否攀援（1为自立，0为攀援），树高，叶片长宽及其比例。
而濒危被子植物则取自中国生物多样性红色目录，将濒危等级参数化，目录包含了近危（NT=1），易危（VU=2），濒危（EN=3），极危（CR=4），野外绝灭与野外绝灭（EW/EX=5），我们将数据的物种名由foc转为tpl格式，将转换后重复的物种取最低濒危程度进行删减，得到了tpl格式的中国被子植物红色目录。
```r
setwd("/Users/calice/desktop/data4paper")
tplredlist <- read.csv("tplredlist.csv", head = T)
str(tplredlist)
```
可以得到红色目录中共包含4932种被子植物，共有1198属，198科。
我们收集到的数据则如下：
```r
phy <- read.csv("phylogenic.csv", head = T)
str(phy)
```
我们在去除了沉水植物后收集到了182科，1136属，4025种，占科的91.92%，属的94.82%，种的81.61%。
## 模型与计算
原计划使用R语言*caper*包的pgls函数，该函数的系统发育是基于布朗运动的，但是由于研究表明使用基于OU模型的pgls会有更高的AIC值，因此本次计算使用nlme包中的GLS模型：
```r
setwd("/Users/calice/desktop/test1")
library(dplyr)
library(ape)
library(map)
library(phytools)
library(geiger)
library(nlme)
alltraits <- read.csv("ecodata.csv", header = TRUE, row.names = 1)
alltree <- read.tree("all.tre")
name.check(alltree, alltraits)
pglsoudem <- gls(iucn ~ dem, correlation = corMartins(1, phy = alltree, fixed = FALSE), data = alltraits, method = "ML")
#Generalized least squares fit by maximum likelihood
#  Model: iucn ~ dem 
#  Data: alltraits 
#       AIC      BIC    logLik
#  11446.06 11471.26 -5719.031
#
#Correlation Structure: corMartins
# Formula: ~1 
# Parameter estimate(s):
#   alpha 
#59.03706 
#
#Coefficients:
#                 Value   Std.Error  t-value p-value
#(Intercept)  2.1871446 0.027160891 80.52551       0
#dem         -0.0001181 0.000013662 -8.64372       0
#
# Correlation: 
#    (Intr)
#dem -0.813
#
#Standardized residuals:
#        Min          Q1         Med          Q3         Max 
#-1.18193103 -0.95634115 -0.08378499  0.88888216  3.22616493 
#
#Residual standard error: 1.002421 
#Degrees of freedom: 4025 total; 4023 residual
pglsoudem <- gls(iucn ~ prep*temp, correlation = corMartins(1, phy = alltree, fixed = FALSE), data = alltraits, method = "ML")
sum_ou <- summary(pglsoudem)
sum_aov <- anova(pglsoudem)
sum_aov
#Denom. DF: 4021 
#            numDF   F-value p-value
#(Intercept)     1 16086.814  <.0001
#prep            1    79.499  <.0001
#temp            1    35.531  <.0001
#prep:temp       1     0.246    0.62
sum_ou
#Generalized least squares fit by maximum likelihood
#  Model: iucn ~ prep * temp 
#  Data: alltraits 
#       AIC      BIC   logLik
#  11410.36 11448.16 -5699.18
#
# Formula: ~1 
# Parameter estimate(s):
#   alpha 
#58.73448 
#
#Coefficients:
#                 Value  Std.Error   t-value p-value
#(Intercept)  1.6263147 0.08009129 20.305762  0.0000
#prep         0.0000547 0.00011153  0.490343  0.6239
#temp         0.0271809 0.00647443  4.198188  0.0000
#prep:temp   -0.0000030 0.00000599 -0.495873  0.6200
#
# Correlation: 
#          (Intr) prep   temp  
#prep      -0.882              
#temp      -0.608  0.351       
#prep:temp  0.837 -0.835 -0.768
#
#Standardized residuals:
#       Min         Q1        Med         Q3        Max 
#-1.2631443 -0.9252217 -0.1095166  0.8683842  3.3095313 
#
#Residual standard error: 0.9974919 
#Degrees of freedom: 4025 total; 4021 residual
pglsou <- gls(iucn ~ dem+gdp+peop+prep+temp+hfp+NDVI+growth+height+snsupport+leaflen+leafwid+leafratio, correlation = corMartins(1, phy = alltree, fixed = FALSE), data = alltraits, method = "ML")
summary(pglsou)
#Generalized least squares fit by maximum likelihood
#  Model: iucn ~ dem + gdp + peop + prep + temp + hfp + NDVI + growth +      height + snsupport + leaflen + leafwid + leafratio 
#  Data: alltraits 
#       AIC      BIC   logLik
#  11355.64 11456.44 -5661.82
#
#Correlation Structure: corMartins
# Formula: ~1 
# Parameter estimate(s):
#   alpha 
#61.90221 
#
#Coefficients:
#                 Value  Std.Error   t-value p-value
#(Intercept)  1.7458949 0.16015036 10.901599  0.0000
#dem          0.0000322 0.00002739  1.175581  0.2398
#gdp          0.0000218 0.00004798  0.453503  0.6502
#peop        -0.0000356 0.00039418 -0.090232  0.9281
#prep        -0.0000190 0.00007046 -0.269291  0.7877
#temp         0.0185189 0.00632384  2.928427  0.0034
#hfp          0.0157206 0.00829963  1.894137  0.0583
#NDVI        -0.0586513 0.17034448 -0.344310  0.7306
#growth      -0.1612386 0.03986646 -4.044467  0.0001
#height       0.0000573 0.00002752  2.080074  0.0376
#snsupport   -0.1076343 0.06033211 -1.784031  0.0745
#leaflen     -0.0006503 0.00143672 -0.452638  0.6508
#leafwid      0.0050007 0.00327462  1.527096  0.1268
#leafratio   -0.0014043 0.00047831 -2.935933  0.0033
#
# Correlation: 
#          (Intr) dem    gdp    peop   prep   temp   hfp   
#dem       -0.668                                          
#gdp        0.013 -0.121                                   
#peop      -0.051  0.176 -0.941                            
#prep       0.105 -0.137 -0.305  0.360                     
#temp      -0.237  0.529  0.258 -0.278 -0.615              
#hfp       -0.288  0.253 -0.010 -0.147 -0.328  0.075       
#NDVI      -0.516 -0.003  0.244 -0.172 -0.173 -0.172 -0.178
#growth    -0.077 -0.046 -0.044  0.045  0.018  0.067 -0.070
#height     0.018 -0.033 -0.010  0.006  0.005 -0.078  0.031
#snsupport -0.353  0.037  0.043 -0.042 -0.037  0.125 -0.018
#leaflen    0.021  0.006 -0.064  0.048 -0.013 -0.110  0.079
#leafwid   -0.073 -0.017  0.039 -0.024 -0.001  0.021 -0.037
#leafratio -0.066  0.024  0.028 -0.023  0.004  0.069 -0.021
#          NDVI   growth height snsppr leafln leafwd
#dem                                                
#gdp                                                
#peop                                               
#prep                                               
#temp                                               
#hfp                                                
#NDVI                                               
#growth    -0.025                                   
#height    -0.010  0.533                            
#snsupport  0.013 -0.194 -0.308                     
#leaflen   -0.035 -0.116 -0.070 -0.066              
#leafwid   -0.012  0.029 -0.107  0.160 -0.483       
#leafratio  0.043 -0.015  0.024  0.024 -0.490  0.302
#
#Standardized residuals:
#       Min         Q1        Med         Q3        Max 
#-1.6389044 -0.8833131 -0.1509136  0.7939094  3.3694417 
#
#Residual standard error: 0.9882533 
#Degrees of freedom: 4025 total; 4011 residual
anova(pglsou)
#Denom. DF: 4011 
#            numDF   F-value p-value
#(Intercept)     1 16348.552  <.0001
#dem             1    76.646  <.0001
#gdp             1     1.656  0.1982
#peop            1     1.895  0.1687
#prep            1    21.523  <.0001
#temp            1    18.681  <.0001
#hfp             1     1.597  0.2064
#NDVI            1     0.072  0.7889
#growth          1    45.874  <.0001
#height          1     3.149  0.0760
#snsupport       1     4.596  0.0321
#leaflen         1     1.579  0.2089
#leafwid         1     6.421  0.0113
#leafratio       1     8.620  0.0033
library(rJava)
library(glmulti)
model <- glmulti(pglsou, level = 1, crit = "aicc") 
```
一些借助别的包的方法：
```r
library(ape)
library(nlme)
#从github上下载r包
library(devtools)
install_github("hafen/cardoonTools")
library(cardoonTools)
alltraits <- read.csv("ecodata.csv", header = TRUE, row.names = 1)
alltree <- read.tree("all.tre")
#下面开始不同
correlation<-"OU"
dep_variable<-"iucn"
if(correlation=="OU"){
    cor <- corMartins(1, phy=alltree, fixed=FALSE)
}
ind_variable<-"peop"
fmla <- as.formula(paste(as.character(dep_variable), "~", as.character(ind_variable),sep=""))
pgls <- gls(model=fmla, correlation=cor, data=alltraits, control=glsControl(opt="optim"))
sum_res <- summary(pgls)
sum_aov <- anova(pgls)
parameter <- coef(summary(pgls))
coefficients <- cbind(rownames(parameter), parameter)
colnames(coefficients)[1]<-"parameter"
if(correlation == "OU") {
  alpha<-pgls$modelStruct[[1]][[1]]
  coefficients <- rbind(coefficients, c("alpha", alpha, NA, NA, NA))
}
modelfit_summary <- data.frame("AIC"= sum_res$AIC, loglik=sum_res$logLik, residual_SE=sum_res$sigma, df_total=sum_res$dims$N, df_residual=sum_res$dims$N-sum_res$dims$p)
pgls_plot <- function() {
    plot(alltraits[,ind_variable], alltraits[,dep_variable], pch=21, bg="gray80", xlab=ind_variable, ylab=dep_variable)
    abline(pgls, lty=2, lwd=2)
}
pglsPlot = cardoonPlot(expression(pgls_plot()), width=1000, height=1000, pgls=100)
pglsPlot = pglsPlot$png
```
需要注意的是，[Arbor](https://github.com/arborworkflows)对此方法做了全面完善：
```r
if(correlation=="BM"){
  cor <- corBrownian(1, phy=tree)
}
if(correlation=="OU"){
  cor <- corMartins(1, phy=tree, fixed=FALSE)
}
if(correlation=="Pagel"){
  cor <- corPagel(1, phy=tree, fixed=FALSE)
  cor1 <- corPagel(1, phy=tree, fixed=TRUE)
  cor0 <- corPagel(0, phy=tree, fixed=TRUE)
}
if(correlation=="ACDC"){
  cor <- corBlomberg(1, phy=tree, fixed=FALSE)
}
# substitute to mimic what R does with column names that contain spaces
dep_variable<-gsub(" ", ".", dep_variable)
ind_variable<-gsub(" ", ".", ind_variable)
```