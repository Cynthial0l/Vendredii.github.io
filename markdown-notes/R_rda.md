# 基于R语言的冗余分析（RDA）
本教程使用了在Evolutionary Applications[(Jenkins et al.，2019)](https://onlinelibrary.wiley.com/doi/full/10.1111/eva.12849)上发表的欧洲龙虾（Homarus gammarus）种群遗传学研究中的双等位基因SNP基因型数据，数据可通过以下链接[下载](https://doi.org/10.5061/dryad.2v1kr38)
原版教程来自[Tom-Jenkins](https://github.com/Tom-Jenkins/seascape_rda_tutorial)
本教程使用的相关R环境的下载可以前往[我的github](https://github.com/Vendredii/Rstats/rda)
[TOC]
### 准备遗传学数据
加载相关R包与环境
```r
#adegenet包加载失败就把所有包更新一下，再重启一下
library(adegenet)
library(poppr)
library(tidyverse)
#setwd('/Users/calice/desktop/Rstats/rda')
load("lobster_1278ind_79snps_40pop.RData")
```
探索相关数据
```r
data_filt
nLoc(data_filt) # number of loci
nPop(data_filt) # number of sites
nInd(data_filt) # number of individuals
summary(data_filt$pop) # sample size
```
整理数据
```r
#结合撒丁岛的时间样本
popNames(data_filt) = gsub("Sar13", "Sar", popNames(data_filt))
popNames(data_filt) = gsub("Sar17", "Sar", popNames(data_filt))
#合并拉齐奥的样本
popNames(data_filt) = gsub("Tar", "Laz", popNames(data_filt))
#合并Idr的时间样本
popNames(data_filt) = gsub("Idr16", "Idr", popNames(data_filt))
popNames(data_filt) = gsub("Idr17", "Idr", popNames(data_filt))
#新数据
data_filt
nPop(data_filt) # number of sites
nInd(data_filt) # number of individuals
summary(data_filt$pop) # sample size
```
计算变量
```r
#计算等位基因频率
allele_freqs = data.frame(rraf(
#为每个地点计算等位基因频率
allele_freqs = data.frame(rraf(data_filt, by_pop=TRUE, correction = FALSE), check.names = FALSE)
#每个SNP只保留两个等位基因中的第一个（p = 1-q）
allele_freqs = allele_freqs[, seq(1, dim(allele_freqs)[2], 2)]
#导出
write.csv(allele_freqs, file = "allele_freqs.csv", row.names = TRUE)
#计算次等位基因频率
#按地点分离genind对象
site_list = seppop(data_filt)
names(site_list)
#为每个地点计算次等位基因频率
maf_list = lapply(site_list, FUN = minorAllele)
#将它们加入数据框
maf = as.data.frame(maf_list) %>% t() %>% as.data.frame()
head(maf)
#导出
write.csv(maf, file = "minor_allele_freqs.csv", row.names = TRUE)
```
可视化等位基因频率
```r
#添加地点记号
allele_freqs$site = rownames(allele_freqs)
#在数据框中添加区域
addregion = function(x){
  # If pop label is present function will output the region
  if(x=="Ale"|x=="The"|x=="Tor"|x=="Sky") y = " Aegean Sea "
  if(x=="Sar"|x=="Laz") y = " Central Mediterranean "
  if(x=="Vig"|x=="Brd"|x=="Cro"|x=="Eye"|x=="Heb"|x=="Iom"|x=="Ios"|x=="Loo"|x=="Lyn"|x=="Ork"|x=="Pad"|x=="Pem"|x=="She"|x=="Sbs"|x=="Sul") y = " Atlantic "
  if(x=="Jer"|x=="Idr"|x=="Cor"|x=="Hoo"|x=="Kil"|x=="Mul"|x=="Ven") y = " Atlantic "
  if(x=="Hel"|x=="Oos"|x=="Tro"|x=="Ber"|x=="Flo"|x=="Sin"|x=="Gul"|x=="Kav"|x=="Lys") y = " Scandinavia "
  return(y)
}
#增加区域记号
allele_freqs$region = sapply(rownames(allele_freqs), addregion)
#将数据帧转换为长格式
allele_freqs.long = allele_freqs %>%
  pivot_longer(cols = 1:79, names_to = "allele", values_to = "frequency")
allele_freqs.long
#使用factor中的levels参数定义地点的顺序
unique(allele_freqs.long$site)
site_order =  c("Tro","Ber","Flo","Gul","Kav","Lys","Sin","Hel","Oos",
                "Cro","Brd","Eye",
                "She","Ork","Heb","Sul","Cor","Hoo","Iom","Ios","Jer","Kil",
                "Loo","Lyn","Mul","Pad","Pem","Sbs","Ven",
                "Idr","Vig",
                "Sar","Laz","Ale","Sky","The","Tor")
allele_freqs.long$site_ord = factor(allele_freqs.long$site, levels = site_order)
#定义区域顺序
region_order = c(" Scandinavia "," Atlantic "," Central Mediterranean ", " Aegean Sea ")
allele_freqs.long$region = factor(allele_freqs.long$region, levels = region_order)
#创建配色方案
# blue=#377EB8, green=#7FC97F, orange=#FDB462, red=#E31A1C
col_scheme = c("#7FC97F","#377EB8","#FDB462","#E31A1C")
#SNP位点到子集的向量
desired_loci = c("7502","25608","31462","35584","42395","53314","58053","65064","65576")
desired_loci_ID = sapply(paste(desired_loci, "..", sep = ""),
                         grep,
                         unique(allele_freqs.long$allele),
                         value = TRUE) %>% as.vector()
#绘制所需SNP位点的子集数据集
allele_freqs.sub = allele_freqs.long %>% filter(allele %in% desired_loci_ID)
#设置ggplot2主题
ggtheme = theme(
  axis.text.x = element_blank(),
  axis.text.y = element_text(colour="black", size=6),
  axis.title = element_text(colour="black", size=15),
  panel.background = element_rect(fill="white"),
  panel.grid.minor = element_blank(),
  panel.grid.major = element_blank(),
  panel.border = element_rect(colour="black", fill=NA, size=0.5),
  plot.title = element_text(hjust = 0.5, size=18),
  legend.title = element_blank(),
  legend.text = element_text(size=15),
  legend.position = "top",
  legend.justification = "centre",
  # facet labels
  strip.text = element_text(colour="black", size=14)
)
#绘制柱状图
ggplot(data = allele_freqs.sub, aes(x = site_ord, y = frequency, fill = region))+
  geom_bar(stat = "identity", colour = "black", size = 0.3)+
  facet_wrap(~allele, scales = "free")+
  scale_y_continuous(limits = c(0,1), expand = c(0,0))+
  scale_fill_manual(values = col_scheme)+
  ylab("Allele frequency")+
  xlab("Site")+
  ggtheme
#ggsave("allele_freq.png", width=10, height=8, dpi=600)
#ggsave("allele_freq.pdf", width=10, height=8)
```
![barplot](Rmodel/Rplot12.jpeg)

