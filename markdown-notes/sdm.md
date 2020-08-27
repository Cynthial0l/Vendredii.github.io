# R语言与物种分布模型
## 前言
物种分布模型，已经烂大街了，但是还挺适合摸鱼的，而且似乎还有点实际作用。
[TOC]
我们需要安装相关程序包和程序：
```r
#安装程序包
install.packages("keras")
#安装相关组件
#for mac
keras::install_keras(method = "conda")
#for win/win
keras::install_keras()
#看看安装是否成功，ture才行
keras::is_keras_available()
#如果不成功再试试安装这些
reticulate::py_config()
tensorflow::tf_config()
```
![plot7](Rmodel/Rplot7.jpeg)
