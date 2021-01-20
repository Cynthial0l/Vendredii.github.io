# 结构方程模型
基于Yves Rosseel的论文，doi: [10.18637/jss.v048.i02](https://www.jstatsoft.org/article/view/v048i02)
据说有本书叫Latent Variable Modeling Using R，没看过，先mark

该结构方程模型（SEM）基于R语言的lavaan函数包运行。lavaan是潜在变量分析的缩写，它的名字揭示了长期目标：提供一系列工具，可用于探索、估计和理解各种潜在变量模型，包括因子分析、结构方程、纵向、多级、潜在类、项目反应和缺失数据模型。
[TOC]
```r
#为了获得与商业软件相似的输出，lavaan开发了下面的功能
#lavan将努力产生与Mplus的输出相似的输出，无论是在数字上还是视觉上
mimic = “Mplus”
#lavan产生的输出接近EQS的输出，至少在数字上（不是视觉上），
simic= “EQS”
```
## SEM之前
首先了解测量模型（measurement model）和结构模型（structural model），测量模型关注的是因子的载荷和结构（EFA和CFA），结构模型则关注的是跨因子间的预测和解释（回归，相关等等）。
CFA是结构强度（Confirmatory Factor Analysis），是确定当前设定的变量之间是有关系的，如果CFA通过，那可以把模型继续下去了。
那么所有的SEM模型都是这几个步骤：
1. CFA通过，就是保证结构
2. 路径分析（ANOVA & Correlation），显示相关分析，**零阶相关**矩阵要显著相关才能进入下一步，然后就是共同方法偏差检验，如测量方式相同就要做这个，就是对所有变量做单因素的CFA，如果没有拟合，说明不怎么出现共同方法偏差，就比较好了。接着是路径分析，就是画一个饱和模型然后删到仅存的路径的效应量和sig都比较好了。然后可以多做几个算是竞争模型看看情况。
3. 中介及调节检验，做bootstrap什么的，然后就好了

## 关于SEM
路径图通常是研究人员寻求拟合SEM模型的起点。非正式地说，路径图是一种示意图，它代表了研究人员要拟合的模型的简明概述。它包括所有相关的观察变量（通常用方框表示）和潜在变量（用圆圈表示），并用箭头说明这些变量之间的（假设的）关系。一个变量对另一个变量的直接影响用单箭头表示，而变量之间（未解释的）相关性用双头箭头表示。研究者的主要问题通常是将此图转换为SEM程序所期望的适当输入。此外，研究人员必须格外小心，以确保模型是可识别和可估计的。
在lavan软件包中，模型是通过一种功能强大、易于使用的基于文本的语法（称为“lavan模型语法”）来指定的。考虑一个简单的回归模型，其中有一个连续的因变量$y$，以及四个自变量$x_1$、$x_2$、$x_3$和$x_4$。通常的回归模型可以写如下：
$$y_{i}=\beta _{0}+\beta _{1}x_{1i}+\beta _{2}x_{2i}+\beta _{3}x_{3i}+\beta _{4}x_{4i}+\varepsilon _{i}$$
其中$β_0$被称为截距，$β_1$到$β_4$是四个变量中每一个的回归系数，$ε_i$是观测值i的残差。R环境的一个吸引人的特点是我们可以用紧凑的方式来表达一个类似于上述的回归公式：
```r
y ~ x1 + x2 + x3 + x4
```
在这个公式中，`~`是回归运算符。在运算符的左侧，我们有因变量`y`，在右侧，我们有自变量，用`+`分隔。注意，公式中没有明确包含截距与残差项。但是当这个模型被拟合时（例如使用`lm()`函数），残差的截距和方差都将被估计。当然，其基本逻辑是截距和残差项（几乎）总是（线性）回归模型的一部分，而且在回归公式中不需要提及它们。只需要指定结构部分（因变量和自变量），其余部分由`lm()`函数负责。
看待SEM模型的一种方法是，它们只是线性回归的扩展。第一个扩展是可以同时拥有多个回归方程。第二个扩展是，一个方程中的自变量（外生变量）可以是另一个方程中的因变量（内生变量）。使用与R中单个方程相同的语法来指定这些回归方程似乎很自然；我们只有一个以上的回归方程。例如，我们可以有一组三个回归方程：
```r
y1 ~ x1 + x2 + x3 + x4
y2 ~ x5 + x6 + x7 + x8
y3 ~ y1 + y2
```
SEM模型的第三个扩展是它们包含连续的潜在变量。在lavaan中，任何回归公式都可以包含作为因变量或自变量的潜在变量。例如，在下面显示的语法中，以`f`开头的变量是潜在变量：
```r
y1 ~ f1 + f2 + x1 + x2
f1 ~ x1 + x2
```
模型语法的这一部分与SEM模型的“结构部分”相对应。为了描述模型的“测量部分”，我们需要为每个潜在变量指定（观察到的或潜在的）指标。在lavaan中，这是用特殊运算符'=~'来完成的，这可以从中看出。此公式的左侧包含潜在变量的名称。右侧包含此潜在变量的指示符，用“+”运算符分隔。例如：
```r
f1 =~ item1 + item2 + item3
f2 =~ item4 + item5 + item6 + item7
f3 =~ f1 + f2
```
在本例中，变量`item1`到`item7`是观察变量。因此，潜在变量f1和f2是一阶因子。潜在变量f3是一个二阶因子，因为它的所有指标本身都是潜在变量。
为了在模型语法中指定（残差）方差和协方差，lavaan提供了`~~`运算符。如果左右两侧的变量名相同，则为方差(var)。如果名字不同，那就是协方差(covar)。残差(协)方差和非残差(协)方差之间的区别是自动进行的。例如：
```r
item1 ~~ item1 # variance 
item1 ~~ item2 # covariance
```
最后，观察变量和潜在变量的截距是简单的回归公式（使用“~”运算符），只有一个截距（用数字“1”明确表示）作为唯一的预测因子：
```r
item1 ~ 1 # intercept of an observed variable 
f1 ~ 1 # intercept of a latent variable
```
描述SEM模型的典型模型语法将包含多个公式类型。在lavaan中，要将它们粘在一起，必须将它们指定为文本字符串。环境中可以用单引号括起来。例如:
```r
myModel <- '# regressions 
            y ~ f1 + f2 
            y ~ x1 + x2
            f1 ~ x1 + x2
            # latent variables
            f1 =~ item1 + item2 + item3
            f2 =~ item4 + item5 + item6 + item7
            f3 =~ f1 + f2
            # (residual) variances and covariances item1 ~~ item1
            item1 ~~ item2
            # intercepts 
            item1 ~ 1 
            f1 ~ 1'
```
这段代码将生成一个名为myModel的模型语法对象，稍后在调用一个函数时可以使用该对象来估计给定数据集的\模型，它说明了lavaan模型语法的几个特性。公式可以拆分为多行，您可以在单引号内使用注释（以“#”字符开头）和空行，以提高模型语法的可读性。公式的指定顺序无关紧要。因此，即使在使用“=~”运算符定义回归公式之前，也可以使用它们。最后，由于这个模型语法只不过是一个文本字符串，所以您可以在单独的文本文件中键入语法，然后使用readLines()之类的函数来读入它。或者，R的文本处理基础设施可以用来为各种模型生成语法。
## 应用lavaan进行CFA
lavaan包包含一个名为HolzingerSwineford1939的内置数据集。1939年Holzinger&Swineford数据集是一个“经典”数据集，已在许多关于结构方程建模的论文和书籍中使用，包括一些商业SEM软件包的手册。数据包括来自两所不同学校（巴斯德和格兰特怀特）的七年级和八年级儿童的智力测试分数。在我们的数据集版本中，最初26个测试中只有9个包含在内。通常针对这9个变量提出的CFA模型由三个相关的潜在变量（或因子）组成，每个变量都有三个指标：
1. 视觉相关的因素由3个变量决定: x1, x2, x3,
2. 文本相关的因素由3个变量决定: x4, x5, x6,
3. 速度相关的因素由3个变量决定: x7, x8, x9.
因此，我们从装载Lavaan包与数据开始：
```r
library("lavaan")
```
在下面的内容中，我们将把这个三因素模型称为“H&S模型”，图1以图形方式表示。注意，图中的路径图是简化的：它不表示观测变量的残差方差或外生潜在变量的方差。不过，它抓住了模型的本质。在讨论该模型的lavan模型语法之前，首先需要确定该模型中的自由参数。在这个模型中有三个潜在变量（因子），每个变量有三个指标，因此需要估计九个因子的负荷。潜在变量之间还有三个协方差-另外三个参数。这些参数分别用双头箭头和双头箭头表示。此外，我们还需要估计9个观测变量的残差方差和潜在变量的方差，从而得到12个额外的自由参数。我们总共有24个参数。但是模型还没有确定，因为我们需要设置潜在变量的度量。通常有两种方法可以做到这一点：（1）对于每个潜在变量，将其中一个指标（通常是第一个）的因子负荷固定为常数（通常为1.0），或（2）标准化潜在变量的方差。不管怎样，我们修复了其中的三个参数，并且有21个参数仍然是自由的。由parTable（）方法生成的表2包含了该模型所有相关参数的概述，包括三个固定因子荷载。表中的每一行对应于一个参数。“rhs”、“op”和“lhs”列唯一地定义了模型的参数。所有带有“=~”运算符的参数都是因子加载，而带有“~~”运算符的所有参数都是方差或协方差。“free”列中的非零元素是模型的自由参数。“free”列中的零元素对应于固定参数，其值在“ustart”列中找到。“用户”栏的含义将在下面解释。
Lavaan有三种方法来指定模型。在第一种方法中，用户对模型的最小描述由程序自动添加其余元素。这种“用户友好”的方法在fitting函数`cfa()`和`sem()`中实现。在第二种方法中，所有模型参数的完整说明必须由用户提供，不会自动添加任何内容。这是“超级用户”方法，在函数`lavaan()`中实现。最后，在第三种方法中，通过在模型语法中提供对模型的不完整描述，但使用lavan函数的`auto.*`参数添加选定的参数组，从而混合了最简方法和完整方法。我们依次说明和讨论这些方法。
### cfa()和sem()的方法
在第一种方法中，用户提供的模型语法应该尽可能简洁易懂。为了实现这一点，模型语法中通常只包含潜在变量（使用“=~”运算符）和回归（使用“~”运算符）。其他模型参数（对于该模型：观测变量的残差方差、因子的方差和因子之间的协方差）是自动添加的。由于H&S示例包含三个潜在变量，但没有回归，因此最简语法非常简短：
```r
HS.model <- 'visual =~ x1 + x2 + x3
             textual =~ x4 + x5 + x6
             speed =~ x7 + x8 + x9'
```
我们导入数据：
```r
fit <- cfa(HS.model, data = HolzingerSwineford1939)
```
函数cfa()是用于拟合验证性因子分析(cfa)模型的专用函数。第一个参数是包含lavaan模型语法的对象。第二个参数是包含观察到的变量的数据集。下表中的“user”列显示哪些参数显式包含在用户指定的模型语法（=1）中，哪些参数是由cfa()函数（=0）添加的。如果已安装模型，则始终可以（且信息量很大）使用以下命令检查此参数表：
```r
parTable(fit)
```
```r
   id     lhs op     rhs user block group free ustart exo label plabel start   est    se
1   1  visual =~      x1    1     1     1    0      1   0         .p1. 1.000 1.000 0.000
2   2  visual =~      x2    1     1     1    1     NA   0         .p2. 0.778 0.554 0.100
3   3  visual =~      x3    1     1     1    2     NA   0         .p3. 1.107 0.729 0.109
4   4 textual =~      x4    1     1     1    0      1   0         .p4. 1.000 1.000 0.000
5   5 textual =~      x5    1     1     1    3     NA   0         .p5. 1.133 1.113 0.065
6   6 textual =~      x6    1     1     1    4     NA   0         .p6. 0.924 0.926 0.055
7   7   speed =~      x7    1     1     1    0      1   0         .p7. 1.000 1.000 0.000
8   8   speed =~      x8    1     1     1    5     NA   0         .p8. 1.225 1.180 0.165
9   9   speed =~      x9    1     1     1    6     NA   0         .p9. 0.854 1.082 0.151
10 10      x1 ~~      x1    0     1     1    7     NA   0        .p10. 0.679 0.549 0.114
11 11      x2 ~~      x2    0     1     1    8     NA   0        .p11. 0.691 1.134 0.102
12 12      x3 ~~      x3    0     1     1    9     NA   0        .p12. 0.637 0.844 0.091
13 13      x4 ~~      x4    0     1     1   10     NA   0        .p13. 0.675 0.371 0.048
14 14      x5 ~~      x5    0     1     1   11     NA   0        .p14. 0.830 0.446 0.058
15 15      x6 ~~      x6    0     1     1   12     NA   0        .p15. 0.598 0.356 0.043
16 16      x7 ~~      x7    0     1     1   13     NA   0        .p16. 0.592 0.799 0.081
17 17      x8 ~~      x8    0     1     1   14     NA   0        .p17. 0.511 0.488 0.074
18 18      x9 ~~      x9    0     1     1   15     NA   0        .p18. 0.508 0.566 0.071
19 19  visual ~~  visual    0     1     1   16     NA   0        .p19. 0.050 0.809 0.145
20 20 textual ~~ textual    0     1     1   17     NA   0        .p20. 0.050 0.979 0.112
21 21   speed ~~   speed    0     1     1   18     NA   0        .p21. 0.050 0.384 0.086
22 22  visual ~~ textual    0     1     1   19     NA   0        .p22. 0.000 0.408 0.074
23 23  visual ~~   speed    0     1     1   20     NA   0        .p23. 0.000 0.262 0.056
24 24 textual ~~   speed    0     1     1   21     NA   0        .p24. 0.000 0.173 0.049
```
当使用cfa()或sem()函数时，default会包含多组参数。这些参数集的完整列表为：
1. `auto.fix.first`默认true，将第一个指标的系数荷载固定为1
2. `auto.fix.single`默认true，将单个指标的残差方差固定为0
3. `int.ov.free`默认true，自由估计观测变量的截距
（仅当包含平均结构时）
4. `int.lv.free`默认false，自由估计潜在变量的截获（仅
如果包括平均结构）
在我们的例子中，只使用了第一个函数（固定第一个指标的因子负荷）。第二个仅当模型包含由单个指标表示的潜在变量时才需要。第三和第四个只有在向模型中添加平均结构时才需要。
在我们继续下一个方法之前，必须强调所有这些“自动”操作都可以被覆盖。模型语法始终优先于自动生成的操作。例如，如果不希望固定第一个指标的因子负荷，而是要固定潜在方差的方差，则模型语法将调整如下：
```r
HS.model.bis <- 'visual =~ NA*x1 + x2 + x3
                 textual =~ NA*x4 + x5 + x6
                 speed =~ NA*x7 + x8 + x9
                 visual ~~ 1*visual
                 textual ~~ 1*textual
                 speed ~~ 1*speed'
```
如上所示，通过将模型参数与数值相乘来固定模型参数，否则通过将固定参数与`NA`相乘来释放固定参数。上面的模型语法覆盖了固定第一个因子加载和估计因子方差的默认行为。然而，在实践中，使用此参数化的一个更方便的方法是保留原始语法，通过添加`std.lv = TRUE`到`cfa()`函数中即可：
```r
fit <- cfa(HS.model, data = HolzingerSwineford1939, std.lv = TRUE)
```
### lavaan()的方法
在许多情况下，将简洁的模型语法与cfa()和sem()函数结合使用非常方便，特别是对于许多传统模型。但有时，这些自动操作可能会妨碍工作，特别是当需要指定非标准模型时。对于这些情况，用户可能更喜欢使用lavaan()数。lavaan()函数的“特性”是，默认情况下它不会向模型添加任何额外的参数，也不会尝试使模型可识别。如果在不使用auto.*参数的情况下调用lavaan()函数，则用户有责任指定正确的模型语法。这可能导致更长的型号规格，但用户可以完全控制。对于H&S模型，完整的Lavan模型语法为：
```r
HS.model.full <- '# latent variables
                    latent variables visual=~1*x1+x2+x3
                    textual =~ 1*x4 + x5 + x6
                    speed =~ 1*x7 + x8 + x9
                  # residual variances observed variables
                    x1~~x1
                    x2~~x2
                    x3~~x3
                    x4~~x4
                    x5~~x5
                    x6~~x6
                    x7~~x7
                    x8~~x8
                    x9~~x9
                  # factor variances
                    visual ~~ visual
                    textual ~~ textual
                    speed ~~ speed
                    factor covariances
                    visual ~~ textual + speed
                    textual ~~ speed'
fit <- lavaan(HS.model.full, data = HolzingerSwineford1939)
```
也可以结合auto.*参数一起使用：
当使用lavan()函数时，用户可以完全控制，但模型语法可能很长，并且包含许多可以轻松自动添加的公式。为了在使用Lavan语法的完整模型规范和自动添加某些参数之间进行比较，lavan()函数提供了几个可选参数，这些参数可用于向模型中添加一组特定参数，或固定一组特定参数。例如，在下面的模型语法中，第一个因子的加载显式地固定为1，并且因子之间的协方差是手动添加的。然而，在模型语法中省略残差方差和因子方差会更方便和简洁。以下模型语法和对lavan()的调用实现了这一点：
```r
HS.model.mixed <- '# latent variables
                     visual =~1*x1+x2+x3
                     textual =~ 1*x4 + x5 + x6
                     speed =~ 1*x7 + x8 + x9
                   # factor covariances
                     visual ~~ textual + speed
                     textual ~~ speed'
fit <- lavaan(HS.model.mixed, data = HolzingerSwineford1939, auto.var = TRUE)
```
### 检查结果
上述三种方法都适用于同一模型。cfa()、sem()和lavan()拟合函数都返回一个`lavan`类的对象，对于这个对象，有几种方法可用于检查模型拟合统计信息和参数估计值:
1. `summary()`可以通过`fit.measures`, `standardized`和`rsquare`进行进一步设定，这个会输出一个关于模型的超长总结
2. `show()`输出一个短的总结
3. `coef()`以命名的数值向量的形式返回模型中自由参数的估计值
4. `fitted()`返回模型的隐含矩（协方差矩阵和平均向量）
5. `resid()`返回原始的、标准化的或标准化的残差（隐含和观察到的力矩之间的差异）
6. `vcov()`返回估计参数的协方差矩阵
7. `predict()`计算因子得分
8. `logLik()`返回拟合模型的对数似然（如果使用了最大似然估计）
9. `AIC()``BIC()`计算信息准则（如果使用最大似然估计）
10. `update()`更新为合适的的Lavaan对象
11. `inspect()`窥视模型的内部；默认情况下，它返回计算模型中自由参数的模型矩阵列表；还可用于提取起始值、渐变值等
如果其中一个或多个设置为TRUE，输出将分别使用因变量的附加拟合测量、标准化估计和$R^{2}$值来丰富。在下面的示例中，我们只请求附加的拟合度:
```r
HS.model <- 'visual =~ x1 + x2 + x3
             textual =~ x4 + x5 + x6
             speed  =~x7+x8+x9'
fit <- cfa(HS.model, data = HolzingerSwineford1939)
summary(fit, fit.measures = TRUE)
```
输出包括三个部分。第一部分（前6行）包含包版本号、模型是否收敛（以及迭代次数）以及分析中使用的有效观察数。接下来，打印模型$χ^2$检验统计量、自由度和$p$值。如果fit.measures = TRUE，则导出第二部分，其中包含基线模型的测试统计数据（假设所有观测变量都不相关）和几个常用拟合指数。如果使用最大似然估计，本节还将包含关于对数似然、AIC和BIC的信息。第三部分概述了参数估计，包括使用的标准误差类型，以及是否使用观测或预期信息矩阵来计算标准误差。然后，对于每个模型参数，显示估计值和标准误差，如果合适，还显示基于Wald检验的z值和相应的双侧p值。为了便于参数估计值的读取，它们被分为三个部分：（1）因子负荷，（2）因子协方差，（3）观测变量和因子的残差方差。

我们也可以使用`parameterEstimates()`函数。
尽管summary()方法提供了一个很好的模型结果摘要，但它只对数据的可视化有用。另一种方法是parameterEstimates()方法，它将参数估计值作为数据帧，使信息易于访问以进行进一步处理。默认情况下，parameterEstimates()方法包括所有模型参数的估计值、标准误差、z值、p值和95%置信区间。
```r
parameterEstimates(fit)
```
通过设置level参数可以更改置信级别。设置`ci=FALSE`会抑制置信区间。此函数的另一个用途是通过设置`standardized=TRUE`来获得估算的几个标准版本：
```r
Est <- parameterEstimates(fit, ci = FALSE, standardized = TRUE)
subset(Est, op == "=~")
```
这里只显示系数荷载。相对于先前的输出，添加了三列具有标准化值。在第一列(std.lv)，只有潜在变量被标准化；在第二列(std.all)，潜在变量和观察变量均已标准化；在第三列(std.nox)中，除外生观测变量外，潜变量和观测变量均已标准化。如果外生观测变量的标准化意义不大（例如，二元协变量），那么最后一个选项可能会有用。由于此模型中没有外部协变量，因此最后两列在输出中是相同的。

还可以使用`modificationIndices()`函数。
如果模型拟合度不高，检查修正指数（MIs）及其相应的期望参数变化（EPCs）可能是有益的。本质上，修正指数提供了一个粗略的估计，如果一个特定的参数是无约束的，模型的χ2检验统计量将如何改善。预期的参数更改是此参数作为自由参数包含时的值。modificationIndices()方法（或具有较短名称的别名modifications()）将打印出一长串参数作为`data.frame`. 在下面的输出中，我们只显示修改指数为10或更高的那些参数:
```r
MI <- modificationIndices(fit)
subset(MI, mi >10)
```
最后三列包含标准化的epc，使用与普通参数估计相同的标准化约定。
## 应用lavaan进行SEM建模
在我们的第二个例子中，我们将探讨“工业化和政治民主”数据集，该数据集以前由Bollen在1989年出版的《结构方程建模》（Bollen 1989）一书中使用，并包含在Lavaan中.  数据集包含了发展中国家政治民主和工业化的各种衡量标准。在模型中，定义了三个潜在变量。分析的重点是模型的结构部分（即潜在变量之间的回归）。
```r
model <- '
    # measurement model
      ind60 =~ x1 + x2 + x3
      dem60 =~ y1 + y2 + y3 + y4
      dem65 =~ y5 + y6 + y7 + y8
    # regressions
      dem60 ~ ind60
      dem65 ~ ind60 + dem60
    # residual covariances
      y1~~y5
      y2~~y4 +y6
      y3~~y7
      y4 ~~ y8
      y6 ~~ y8'
fit <- sem(model, data = PoliticalDemocracy) 
summary(fit, standardized = TRUE)
```
可以给定参数标签和进行简单的等式约束：
在lavaan中，每个参数都有一个名称，称为“参数标签(parameter label)”。命名方案是自动的，遵循一组简单的规则。每个标签由三个组件组成，它们描述了定义参数的相关公式。第一部分是显示在公式运算符左侧的变量名。第二部分是公式的运算符类型，第三部分是运算符右侧与参数对应的变量。要查看实际的命名机制，我们可以使用coef()函数，该函数返回自由参数的（估计）值及其相应的参数标签。
```r
coef(fit)
```
用户可以在模型语法中通过预先将变量名与该标签相乘来提供自定义标签。例如，考虑以下回归公式：
```r
y ~ b1*x1 + b2*x2 + b3*x3 + b4*x4
```
这里我们将四个回归系数命名为b1、b2、b3和b4。自定义标签很方便，因为您可以在模型语法的其他地方引用它们。特别是，标签可用于对某些参数施加相等约束。如果两个参数具有相同的名称，那么它们将被视为相同的，并且只为它们计算一个值（即，一个简单的等式约束）。为了说明这一点，我们将重新指定政治民主数据的模型语法。在博伦书中的原始示例中，dem60因子的因子载荷被约束为等于dem65因子的因子载荷。这是有意义的，因为这是在两个时间点上测量的同一个结构。为了执行这些等式约束，我们将dem60因子的因子加载（任意）标记为d1、d2和d3。注意，我们没有标记第一个因子加载，因为它是一个固定参数（等于1.0）。接下来，我们对dem65因子的因子加载使用相同的标签，有效地施加了三个等式约束:
```r
model.equal <- '# measurement model
                     ind60 =~ x1 + x2+x3
                     dem60 =~ y1 + d1*y2 + d2*y3 + d3*y4
                     dem65 =~ y5 + d1*y6 + d2*y7 + d3*y8
                   # regressions
                     dem60 ~ ind60
                     dem65 ~ ind60 + dem60
                   # residual covariances
                     y1~~y5
                     y2~~y4 +y6
                     y3~~y7
                     y4~~y8
                     y6 ~~ y8'
fit.equal <- sem(model.equal, data = PoliticalDemocracy) 
summary(fit.equal)
```
与无约束模型相比，约束模型的拟合稍差。但情况是否明显更糟？为了比较两个嵌套模型，我们可以使用`anova()`函数，该函数将计算$χ^2$差异检验：
```r
anova(fit, fit.equal)
```
接下来可以提取拟合测度：
带有参数的`summary()`方法合适的措施=TRUE将输出多个拟合度量值。如果进一步处理需要fit统计信息，则首选`fitMeasures()`方法。`fitMeasures()`的第一个参数是fitted对象，第二个参数是包含要提取的fit度量值名称的字符向量。例如，如果我们只需要CFI和RMSEA值，我们可以使用：
```r
fitMeasures(fit, c("cfi", "rmsea"))
```
为了完成我们的SEM示例，我们将简要介绍`inspect()`方法，该方法允许用户从lavaan对象的引擎盖下窥视。默认情况下，对已安装的Lavan对象调用`inspect()`将返回一个模型矩阵列表，这些矩阵在内部用于表示模型。自由参数是非零整数。
```r
inspect(fit)
```
输出显示，lavaan目前使用的是LISREL矩阵表示法，尽管没有区分内生变量和外生变量。这就是所谓的“all-y”表示法。在将来的版本中，我计划考虑替代矩阵表示，包括Bentler-Weeks和网状作用模型（RAM）方法（Bollen 1989，第9章）。要查看每个模型矩阵中参数的起始值，请键入
```r
inspect(fit, what = "start")
```
Lavaan软件包完全支持多组SEM。要请求多组分析，可以将数据集中定义组成员身份的变量传递给`cfa()`、`sem()`或`lavaan()`函数调用的group参数。默认情况下，在所有组中都会拟合同一个模型，而对模型参数没有任何相等约束。在下面的例子中，我们将H&S模型应用于这两所学校（Pasteur和Grant-White）:
```r
HS.model <- 'visual =~ x1 + x2 + x3
             textual =~ x4 + x5 + x6
            speed =~ x7 + x8 + x9'
fit <- cfa(HS.model, data = HolzingerSwineford1939, group = "school")
```
`summary()`输出相当长，此处未显示。基本上，它显示了巴斯德群的一组参数估计值，然后是格兰特怀特群的另一组参数估计值。如果我们希望跨组对模型参数施加相等约束，可以使用组。相等争论。例如，`group.equal = c(“loading”, “intercepts”)`将约束系数荷载和观测变量截距在各组之间相等。可以包含在组。相等参数在拟合函数的帮助页中进行了描述。作为一个简单的例子，我们将拟合两个学派的H&S模型，但约束因子载荷和截距相等。方差分析函数可以用来比较两种模型的拟合:
```r
fit.metric <- cfa(HS.model, data = HolzingerSwineford1939,
+ group = "school", group.equal = c("loadings", "intercepts"))
anova(fit, fit.metric)
```
如果group.equal参数用于约束组之间的因子加载，所有因子加载都会受到影响。如果需要一些异常，可以使用`group.partial`参数，它接受一个参数标签向量，指定哪些参数将在组之间重新自由使用。因此，结合`group.equal`以及`group.partial`参数为用户提供了一种灵活的机制来指定跨组相等约束。
## 其他功能
### 渐进无分布估计（ADF）
在lavaan中，可以通过在一个拟合函数中使用估计器自变量来设置估计器。默认为最大似然估计，或`estimator=“ML”`。要切换到ADF estimator，可以设置`estimator=“WLS”`。

### Satorra-Bentler标度检验统计量与稳健标准差
另一种策略是使用最大似然（ML）来估计模型参数，即使已知数据是非正态的。在这种情况下，参数估计仍然是一致的（如果模型被识别并正确指定），但是标准误差往往太小（高达25–50%），这意味着我们可能会过多地拒绝零假设（参数为零）。另外，模型（$χ^2$）检验统计量往往过大，这意味着我们可能会经常拒绝模型。
在SEM文献中，一些作者扩展了ML方法来产生标准误差，这些标准误差对于任意分布（具有有限的四阶矩）是渐近正确的，并且其中重新缩放的检验统计量用于整体模型评估。
在lavaan中，test参数可用于在不同的测试统计之间切换。设置`test=“Satorra-Bentler"`用标度版补充标准$χ^2$模型试验。在`summary()`方法生成的输出中，缩放和未缩放的模型测试（以及相应的拟合指数）都显示在相邻的列中。因为人们通常需要稳健的标准误差和标度检验统计量，所以指定`estimator=“MLM”`可以使用标准最大似然来估计模型参数，但要使用稳健的标准误差和Satorra-Bentler标度检验统计量来拟合模型。
```r
fit <- cfa(HS.model, data = HolzingerSwineford1939, missing = "listwise", + estimator = "MLM", mimic = "Mplus")
summary(fit, estimates = FALSE, fit.measures = TRUE)
```
在这个例子中，`simic=“Mplus”`参数被用来模拟Mplus程序计算Satorra Bentler标度测试统计的方式。默认情况下（即，当省略模拟参数时），Lavan将使用EQS程序使用的方法。为了模拟由EQS程序报告的Satorra-Bentler标度检验统计量的准确值，可以使用:
```r
fit <- cfa(HS.model, data = HolzingerSwineford1939, estimator = "MLM", mimic = "EQS")
fit
```
### Bootstrapping：naıve Bootstrap和Bollen-Stine Bootstrap
在lavaan中，通过设置`se=“bootstrap”`可以获得引导标准误差。在这种情况下，`parameterEstimates()`方法生成的置信区间将是基于引导的置信区间。如果`test=“bootstrap”`或`test=“bollen.stine"`，首先转换数据以执行基于模型的“Bollen-Stine”引导。bootstrap标准误差也基于这些基于模型的bootstrap绘图，并用bootstrap概率值来补充χ2检验统计量的标准p值，该值是通过计算引导样本中的检验统计量超过原始（父）样本的检验统计量值的比例得到的。
默认情况下，lavaan生成$R=1000$的引导绘制，但是这个数字可以通过设置bootstrap参数来更改。设置`verbose=TRUE`以监视引导过程可能会提供信息。
### 缺失值
如果数据包含缺失值，lavan中的默认行为是列表删除。如果缺失机制是MCAR（随机完全缺失）或MAR（随机缺失），则Lavan软件包提供了case-wise（或“full info”）最大似然（FIML）估计。在调用fitting函数时，可以通过指定参数`missing=“ml”`（或其别名`missing=“FIML”`）来启用FIML估计。一个非限制（h1）模型将被自动估计，以便所有常用拟合指数都可用。稳健的标准误差也可用，如果数据是不完整的和非正态的，则是标度检验统计量。
### 线性和非线性等式和不等式约束
在许多应用中，需要对一些模型参数施加约束。例如，可以强制要求方差参数严格为正。对于某些模型，重要的是指定一个参数等于其他参数的某个（线性或非线性）函数。lavaan包的目的是使用lavaan模型语法使这些约束易于指定。一个简短的例子将说明lavaan中的约束语法。考虑以下回归：
```r
y ~ b1*x1 + b2*x2 + b3*x3
```
其中我们明确地将回归系数标记为$b_1$、$b_2$和$b_3$。我们创建一个包含这四个变量的演示数据集，并拟合回归模型：
```r
set.seed(1234)
Data <- data.frame(y = rnorm(100), x1 = rnorm(100), x2 = rnorm(100), + x3 = rnorm(100))
model <- 'y ~ b1*x1 + b2*x2 + b3*x3'
fit <- sem(model, data = Data)
coef(fit)
```
假设我们希望对b1施加两个（非线性）约束：$b_1=(b_2+b_3)^2$和$b_1≥exp(b_2+b_3)$。第一个约束是等式约束，而第二个约束是不等式约束。这两个约束都是非线性的。在lavaan，这是通过以下方式实现的：
```r
model.constr <- '# model with labeled parameters
                   y ~ b1*x1 + b2*x2 + b3*x3
                 # constraints
                  b1 == (b2 + b3)^2
                  b1 > exp(b2 + b3)' 
fit <- sem(model.constr, data = Data) 
summary(fit)
```
### 间接效应与中介分析
一旦对模型参数进行了拟合，我们就可以对模型的原始函数值进行估计。一个例子是两个（或更多）回归系数的乘积的间接效应。考虑一个具有三个变量的经典中介设置：Y是因变量，X是预测因子，M是中介变量。为了说明这一点，我们再次创建一个包含这三个变量的演示数据集，并拟合一个包含X对Y的直接作用和X通过M对Y的间接作用:
```r
set.seed(1234)
X <- rnorm(100)
M <- 0.5 * X + rnorm(100)
Y <- 0.7 * M + rnorm(100)
Data <- data.frame(X = X, Y = Y, M = M) R> model <- '# direct effect
            Y ~ c*X
          # mediator
            M ~ a*X
            Y ~ b*M
          # indirect effect (a*b)
            ab := a*b
          # total effect
            total := c + (a*b)' 
fit <- sem(model, data = Data) 
summary(fit)
```
该示例说明了lavaan模型语法中`:=`运算符的用法。此运算符“定义”新参数，这些参数采用原始模型参数的任意函数。但是，必须根据模型语法中明确提到的参数标签来指定函数。默认情况下，这些定义参数的标准误是使用delta方法计算的。与其他模型一样，只需在fitting函数中指定`se = “bootstrap”`，就可以请求引导标准误。
## 词云分析
也就是关键词分析（TF-IDF），需要下面这两个包：
```r
#加载包
library(jiebaR)
library(wordcloud2)
```
### jiebaR包
其主体为worker函数：
```r
worker(type = "mix", #type就是选择模型，其中mix是混合模型，mp是最大概率法，hmm是隐式马尔科夫模型，query是索引模型，tag是标记模型，simhash是simhash模型，keywords是关键词模型，根据IDF搞
       dict = DICTPATH, #系统词典
       hmm = HMMPATH, 
       user = USERPATH, #用户词典
       idf = IDFPATH, #IDF词典
       stop_word = STOPPATH, #关键词用停止词库
       write = T, #是否将文件分词结果写入文件，默认FALSE
       qmax = 20, 
       topn = 5, #关键词数,默认5个
       encoding = "UTF-8", detect = T, symbol = F, lines = 1e+05,
       output = NULL, bylines = F, user_weight = "max" #用户权重
       )
```
使用方法：
```r
wk = worker()
#wk["这里放文字，或者txt"]
segment("这里放文字，或者txt",wk)
```
该函数可以自动****计算频数：
```r
wk = worker()
words = "数模竞赛不会真出这么难的题目吧，不会吧不会吧"
freq(segment(words,wk))
```
处理文件的词频需要加行代码：
```r
out=file("like_segment.txt")
freq <- freq(strsplit(readLines(out,encoding="UTF-8")," ")[[1]])
```
也可以**提取关键词**：
```r
wk = worker(type = "keywords", topn = 2)
wk <= words
```
### wordcloud2包
这个包可以绘制词云分析的结果：
```r
#去除数字
freq <- freq[!grepl('[0-9]+',names(freq))]
#去除字母
freq <- freq[!grepl('a-zA-Z',names(freq))]
#查看处理完后剩余的词数
length(freq)
#降序排序，并提取出现次数最多的前100个词语
freq <- sort(freq, decreasing = TRUE)[1:100]
#查看100个词频最高的
freq
#提取前150个结果
freq <- f[1:150,]
#形状设置为一颗五角星等等
wordcloud2(f2, size = 0.8, shape = "star") #或者"cardioid"或"diamond"
```
可以基于JavaScript定义词云的颜色：
```r
js_color_fun = "function (word, weight) {
  return (weight > 2000) ? '#f02222' : '#c09292';
}"
wordcloud2(demoFreqC, color = htmlwidgets::JS(js_color_fun), 
           backgroundColor = 'black')
#也可以这样：
wordcloud2(demoFreqC, color = ifelse(demoFreqC[, 2] > 2000, '#f02222', '#c09292')
```
一个小小的实现：基于淘宝搜索的词云分析
关于tf-idf：
```
TF-IDF = TF(词频) * 逆文档频率(IDF)
```
```r
library(jiebaR)
readChineseWords <- function (path) {
  # 读取网页或文件 去除标点和英文
  rawstring = readLines(path)
  rawstring = paste0(rawstring, collapse = ' ')
  s = gsub('\\w', '', rawstring, perl=TRUE)
  s = gsub('[[:punct:]]', ' ', s)
  return(s)
}
 
# 淘宝首页搜索'男'和'女'对应的网页链接
male_link = 'https://s.taobao.com/search?q=%E7%94%B7&search_type=item&sourceId=tb.index'
female_link = 'https://s.taobao.com/search?q=%E5%A5%B3&search_type=item&sourceId=tb.index'

male_str = readChineseWords(male_link)
female_str = readChineseWords(female_link)
# 分词
cc = worker()
new_user_word(cc,'打底裤','ddk')
male_words = cc[male_str]
female_words = cc[female_str]
# 计算tf-idf 
idf = get_idf(list(male_words, female_words))
get_tf_idf <- function(words){
  words_freq = table(words)
  df = data.frame(name=names(words_freq), freq=as.numeric(words_freq))
  df = merge(df, idf, all.x = TRUE)
  wc_df = data.frame(words=df$name, freq=ceiling(df$count * df$freq * 10))
  # 缺失和0值替换成1 
  wc_df$freq[wc_df$freq == 0 | is.na(wc_df$freq)] = 1
  return(wc_df)
}

# 绘制词云
male_df = get_tf_idf(male_words)
female_df = get_tf_idf(female_words)
wordcloud2(male_df,
           backgroundColor = 'black', color = 'random-light')
wordcloud2(female_df, 
           backgroundColor = 'black', color = 'random-light')
```