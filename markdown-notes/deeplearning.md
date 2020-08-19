# 深度学习
## 前言
施工中
R语言上与深度学习相关的原生程序包很少，大多其实是基于Python的二道贩子包。
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
[TOC]
## 简单的神经网络预测
在运行一个简单的神经网络前需要加载这些包与数据
```r
library(cowplot)
library(keras)
library(dplyr)
library(tensorflow)
library(ggplot2)
#从美国国家标准与技术研究院数据库（NMIST）获取手写数字图像数据
mnist = dataset_mnist()
#看看这些数据的结构
str(mnist)
#设置测试集与训练集
x_train = mnist$train$x
y_train = mnist$train$y
x_test = mnist$test$x
y_test = mnist$test$y
#并查看他们的结构
```
`array_reshape`函数允许我们将三维数组（如在mnist数据集中找到的三维数组）整形为矩阵。我们的28x28像素图像将变成具有长度的数组/向量28*28=784。
```r
#L表示整数
height = 28L
width = 28L
#转为矩阵
x_train = array_reshape(x_train, c(nrow(x_train), height * width))
x_test = array_reshape(x_test, c(nrow(x_test), height * width))
#我们看看结构，可以发现这些已经不是二维数据了
str(x_train)
str(x_test)
summary(x_train[, 500:550])
#查看数据，可以注意到每个像素变成了介于0（色谱的黑端）到255（色谱的白端）之间的像素值
x_train[1, ]
#将数据缩放至0-1之间
x_train = x_train / 255
x_test = x_test / 255
summary(x_train[, 500:520])
#通过找到每个像素列的最大值，然后取该向量的最大值来明确地确认它。
max(apply(x_train, MARGIN = 2, max))
#最大值都是1
```
现在我们可以定义模型了。我们要建立一个顺序的层堆栈。该`units`参数定义了我们在每个层中应该有多少个节点（神经元）。`input_shape`允许我们在初始输入层中定义图像尺寸。该`activation`参数允许我们传入激活函数的名称作为参数。
```r
model = keras_model_sequential() 
model %>%  
#输入层加一层隐藏层的结构
#layer_dense可以增加隐藏层.
#input_shape参数实际指定输入层；“units=”and“activation=”定义第一个隐藏层。
#或者我们在layer_dense()前有一个单独的layer_input().
layer_dense(units = 64, activation = 'relu', input_shape = 784) %>% 
  layer_dropout(rate = 0.4) %>% 
# 第二层隐藏层
layer_dense(units = 16, activation = 'relu') %>%
  layer_dropout(rate = 0.3) %>%
#输出层
layer_dense(units = 10, activation = 'softmax')
summary(model)
```
我们使用`sparse_categorical_crossentropy`作为损失函数，因为我们要处理多个分类（即分类变量），而使用`optimizer_rmsprop()`作为优化器，因为它的性能可能比带动量的梯度下降要好一些。什么是`lr`参数呢？我们还选择“accuracy”作为我们的指标，以便为结果产生简单的分类率。
```r
model %>% compile(
  loss = 'sparse_categorical_crossentropy',
  # loss = "mean_squared_error",
  optimizer = optimizer_rmsprop(lr = 0.001),
  metrics = c('accuracy')
)
```
### 训练与评估
现在我们可以使用训练模型了`fit`，在这里，我们只需传递X和Y变量以及其他超参数即可。
观看模型构建时代。一个时期（epoch）是所有训练数据的一次迭代，此处通过批处理128个观测值来完成。
```r
(history = model %>% fit(
  x_train, y_train, 
  epochs = 45, batch_size = 128, 
  validation_split = 0.2
))
```
![plot1](deeplearning/Rplot01.jpeg)
如何解释这些参数呢：
loss：损失是每批训练数据中平均损失的平均值。我们希望早期批次的损失高于晚期批次的损失，因为模型应该随着时间的推移而不断学习。我们希望以后的数据损失更少。
acc：训练的准确性
val_loss和val_acc是测试数据的损失和准确性。
可以通过ggplot绘制训练历史：
```r
plot(history) + theme_minimal()
#ggsave()来保存
```
![plot1](deeplearning/Rplot02.jpeg)
评估测试数据的性能：
```r
model %>% evaluate(x_test, y_test)
#结果非常好
#313/313 [==============================] - 0s 1ms/step - loss: 0.1895 - accuracy: 0.9634
#     loss  accuracy 
#0.1894975 0.9634000 
#对测试数据生成预测，无需显式评估。
preds = model %>% predict(x_test)
dim(preds)
head(round(preds, 4))
glimpse(preds)
```