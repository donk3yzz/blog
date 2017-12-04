---
title: 翻译：R-Views Survival analysis with R
tags: [Survival Analysis,R Views]
---

　　原文来自[R Views-Survival analysis with R](https://rviews.rstudio.com/2017/09/25/survival-analysis-with-r/)，侵..侵删？

　　Survival analysis的起源可以追溯到1662年，一位名为John Graunt的伦敦商人基于手中的死亡记录数据发布了一系列的推断，这使得生存分析成为统计学最古老的分枝之一。在十八世纪以前，基本生命表方法，包括处理删失数据的技巧被人们发现；十八世纪早期，那些大师们的工作（de Moivre对于养老金的研究、Daniel Bernoulli对于接种天花风险的研究）奠定了生存分析现代方法的基础。如今，生存分析模型在工程、保险、市场营销、医药以及许多应用领域发挥着重要作用，所以，R中拥有丰富的生存分析函数也就不足为奇了。 [CRAN’s Survival Analysis Task View](https://cran.r-project.org/web/views/Survival.html)包含了几乎所有R中生存分析相关的函数和包，实在是无比的强大(indeed formidable)，我们都应该对于此Task View的维护者，Arthur Allignol和Aurielien Latouche充满感激之情。

（注：一股浓浓的的四级翻译味...）

　　但是如果当我们打开这个网页，就会发现里面的内容实在是太过于丰富，这座“大厦”让我们了解到R的开发人员为生存分析的理论和实践做出了多么大的贡献。尽管Task View的页面组织的很好，但是一个生存分析学者如果想要在里面找到他自己需要的方法可能也需要花上一些时间，更不用说R或者生存分析的新用户了。所以，我将提供一个比较狭窄的路径去供新人了解此Task View，依赖于`survival`,`ggplot`,`ggfortify`,`ranger`四个包。

　　`survival`包是R语言“生存分析大厦”的基石，不仅是因为`survival`包本身富有特点，更是因为其中的`Surv()`函数，建立了包含删失信息和失败时间的生存分析对象(Survival analysis object)，成为了生存分析的基本数据结构。此包由Terry Therneau博士于1986年开始编写，它的初次发布是在1989年末，应用于CMU主持的Statlib service项目中。随后这个包被编入[Splus](http://www.mayo.edu/research/documents/tr53pdf/doc-10027379)中，后来被编入了R。

　　`ggfortify`使得我们能够通过`ggplot::autoplot()`能够绘制漂亮的、单线生存图形。

　　在我使用的生存分析包中，`ranger`大概是一个惊喜，`ranger()`是众所周知的，在随机森林算法中集合分类与回归树的快速实现的函数，而[Benchmarks指出](https://arxiv.org/pdf/1508.04409.pdf)，`ranger()`适合于构建大量、高维数据集下的time-to-event model，对于网络营销应用具有重要意义。因为`ranger()`本身使用的就是`Surv()`的标准生存数据对象，所以在这个机器学习大热的时代中，`ranger()`同样成为一个了解生存分析的理想工具。

载入数据
--------

　　第一个代码块是载入需要的包和数据集，`veteran`是一个基于两种肺癌治疗方法的随机对比临床实验的数据集，来源于`survival`包。

``` r
library(survival)
library(ranger)
library(ggplot2)
library(dplyr)
library(ggfortify)

data(veteran)
head(veteran)
```

    ##   trt celltype time status karno diagtime age prior
    ## 1   1 squamous   72      1    60        7  69     0
    ## 2   1 squamous  411      1    70        5  64    10
    ## 3   1 squamous  228      1    60        3  38     0
    ## 4   1 squamous  126      1    60        9  63    10
    ## 5   1 squamous  118      1    70       11  65    10
    ## 6   1 squamous   10      1    20        5  49     0

　　`veteran`中的变量: - veteran 老兵 - trt: 1=standard 2=test - celltype: 1=squamous（鳞状的）, 2=small cell, 3=adeno（腺病毒）, 4=large - time: survival time in days - status: censoring status - karno: 卡氏方法评分 (100=good) - diagtime: months from diagnosis to randomization - age: in years - prior: 先前是否接受过治疗 0=no, 10=yes 注：[karnofsky](http://www.hospicepatients.org/karnofsky.html)

Kaplan Meier估计
----------------

　　首先是用`Surv()`函数建立标准的生存对象，`time`记录了生存时间；`status`表示患者死亡(status=1)或者生存时间删失(status=0)；下面`km`时间中的"+"表示时间是删失的。

``` r
# Kaplan Meier Survival Curve
km <- with(veteran, Surv(time, status))
head(km,80)
```

    ##  [1]  72  411  228  126  118   10   82  110  314  100+  42    8  144   25+
    ## [15]  11   30  384    4   54   13  123+  97+ 153   59  117   16  151   22 
    ## [29]  56   21   18  139   20   31   52  287   18   51  122   27   54    7 
    ## [43]  63  392   10    8   92   35  117  132   12  162    3   95  177  162 
    ## [57] 216  553  278   12  260  200  156  182+ 143  105  103  250  100  999 
    ## [71] 112   87+ 231+ 242  991  111    1  587  389   33

　　开始我们的分析，我们用formula`Surv(time,status)~1`和`survfit()`函数进行Kaplan-Meier估计的生存概率函数，`times`函数用于控制`summary()`函数的展示的时间。这里我们选择第1,30,60,90和之后每90天的生存概率估计。这是一个非常简单的模型，我们只需要三行代码进行拟合、摘要和绘图。

``` r
km_fit <- survfit(Surv(time, status) ~ 1, data=veteran)
summary(km_fit, times = c(1,30,60,90*(1:10)))
```

    ## Call: survfit(formula = Surv(time, status) ~ 1, data = veteran)
    ## 
    ##  time n.risk n.event survival std.err lower 95% CI upper 95% CI
    ##     1    137       2    0.985  0.0102      0.96552       1.0000
    ##    30     97      39    0.700  0.0392      0.62774       0.7816
    ##    60     73      22    0.538  0.0427      0.46070       0.6288
    ##    90     62      10    0.464  0.0428      0.38731       0.5560
    ##   180     27      30    0.222  0.0369      0.16066       0.3079
    ##   270     16       9    0.144  0.0319      0.09338       0.2223
    ##   360     10       6    0.090  0.0265      0.05061       0.1602
    ##   450      5       5    0.045  0.0194      0.01931       0.1049
    ##   540      4       1    0.036  0.0175      0.01389       0.0934
    ##   630      2       2    0.018  0.0126      0.00459       0.0707
    ##   720      2       0    0.018  0.0126      0.00459       0.0707
    ##   810      2       0    0.018  0.0126      0.00459       0.0707
    ##   900      2       0    0.018  0.0126      0.00459       0.0707

``` r
#plot(km_fit, xlab="Days", main = 'Kaplan Meyer Plot') #基础图形同样可以使用
autoplot(km_fit)
```

![](/img/翻译：R中的生存分析_files/figure-markdown_github/unnamed-chunk-3-1.png)

　　然后我们绘制按照治疗分组的生存曲线

``` r
km_trt_fit <- survfit(Surv(time, status) ~ trt, data=veteran)
#plot(km_trt_fit)
autoplot(km_trt_fit)
```

![](/img/翻译：R中的生存分析_files/figure-markdown_github/unnamed-chunk-4-1.png)

　　为了绘制一张探索性图，我们需要做一些数据整理去判断不同年龄的生存情况。我们创建一个新的数据框包含新的分类变量`AG`，其中`GT60`代表老兵年龄大于60岁，`LT60`代表小于60；然后我顺手把`trt`和`prior`变量重新定义为了因子变量，当然`survfit()`和`npsurv()`不需要这样的重新定义也可以正常工作。

``` r
#注：mutate()函数可以在数据框中添加新变量或改变变量定义，来源于dplyr包
vet <- mutate(veteran, AG = ifelse((age < 60), "LT60", "OV60"),
              AG = factor(AG),
              trt = factor(trt,labels=c("standard","test")),
              prior = factor(prior,labels=c("N0","Yes")))

km_AG_fit <- survfit(Surv(time, status) ~ AG, data=vet)
autoplot(km_AG_fit)
```

![](/img/翻译：R中的生存分析_files/figure-markdown_github/unnamed-chunk-5-1.png) 
　　虽然两条曲线在50天以前具有一定的重合性，不过在一年以后，年轻的患者还是具有更高的生存概率。

cox比例风险模型
---------------

　　接下来我将使用所有的协变量拟合一个cox比例风险模型

``` r
# Fit Cox Model
cox <- coxph(Surv(time, status) ~ trt + celltype + karno
             + diagtime + age + prior, data = vet)
summary(cox)
```

    ## Call:
    ## coxph(formula = Surv(time, status) ~ trt + celltype + karno + 
    ##     diagtime + age + prior, data = vet)
    ## 
    ##   n= 137, number of events= 128 
    ## 
    ##                         coef  exp(coef)   se(coef)      z Pr(>|z|)    
    ## trttest            2.946e-01  1.343e+00  2.075e-01  1.419  0.15577    
    ## celltypesmallcell  8.616e-01  2.367e+00  2.753e-01  3.130  0.00175 ** 
    ## celltypeadeno      1.196e+00  3.307e+00  3.009e-01  3.975 7.05e-05 ***
    ## celltypelarge      4.013e-01  1.494e+00  2.827e-01  1.420  0.15574    
    ## karno             -3.282e-02  9.677e-01  5.508e-03 -5.958 2.55e-09 ***
    ## diagtime           8.132e-05  1.000e+00  9.136e-03  0.009  0.99290    
    ## age               -8.706e-03  9.913e-01  9.300e-03 -0.936  0.34920    
    ## priorYes           7.159e-02  1.074e+00  2.323e-01  0.308  0.75794    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ##                   exp(coef) exp(-coef) lower .95 upper .95
    ## trttest              1.3426     0.7448    0.8939    2.0166
    ## celltypesmallcell    2.3669     0.4225    1.3799    4.0597
    ## celltypeadeno        3.3071     0.3024    1.8336    5.9647
    ## celltypelarge        1.4938     0.6695    0.8583    2.5996
    ## karno                0.9677     1.0334    0.9573    0.9782
    ## diagtime             1.0001     0.9999    0.9823    1.0182
    ## age                  0.9913     1.0087    0.9734    1.0096
    ## priorYes             1.0742     0.9309    0.6813    1.6937
    ## 
    ## Concordance= 0.736  (se = 0.03 )
    ## Rsquare= 0.364   (max possible= 0.999 )
    ## Likelihood ratio test= 62.1  on 8 df,   p=1.799e-10
    ## Wald test            = 62.37  on 8 df,   p=1.596e-10
    ## Score (logrank) test = 66.74  on 8 df,   p=2.186e-11

``` r
cox_fit <- survfit(cox)
#plot(cox_fit, main = "cph model", xlab="Days")
autoplot(cox_fit)
```

![](img/翻译：R中的生存分析_files/figure-markdown_github/unnamed-chunk-6-1.png) 
　　注意，这个模型把small cell type,adeno cell type和karno标记为显著的，然而在解释模型的过程中，有一些地方是值得注意的。如果cox比例风险模型被认为是“稳健的”，一些谨慎的分析者就会会检查模型下的假定。比如，cox比例风险模型假定所有协变量是不随时间变化的。但是在`survival`包的[一个梗概](https://cran.r-project.org/web/packages/survival/vignettes/timedep.pdf)中，Therneau, Crowson和Atkinson证明了Karnofsky score(karno)是依赖于时间的，所以这个假定在上面的模型中是被违背的。然后作者在这个梗概中，介绍了一种处理依赖于时间的协变量的方法。

　　那些习惯于计算ROC曲线（注：机器学习中对模型的一种评价标准）来评估模型表现的数据科学家们，应该对于一致性统计量（Concordence statistic）很有兴趣。在`survival`包里`survConcordance()`函数的文档中，将一致性定义为（翻译可能有问题建议读原文..）： 任意的两个观测量之间达到一致(agreement)的概率，而这里的一致(agreement)是指这两个观测量中拥有较短生存时间的那一个同样拥有更高的风险评分(risk score)，风险评分常来源于Cox模型或其他回归。 
>the probability of agreement for any two randomly chosen observations, where in this case agreement means that the observation with the shorter survival time of the two also has the larger risk score. The predictor (or risk score) will often be the result of a Cox model or other regression

　　他还强调，对于连续型协变量来说，一致性统计量等价于Kendall’s tau统计量（注：[Kendall tau distance](https://en.wikipedia.org/wiki/Kendall_tau_distance)），对于logistic回归，一致性统计量等价于ROC曲线下面的面积。 
>For continuous covariates concordance is equivalent to Kendall’s tau, and for logistic regression is is equivalent to the area under the ROC curve.

　　为了证明这一观点，我将使用`survival`包和` ggplot2`,`ggfortify `,`ranger`，对`veteran`数据集中的删失数据拟合Aalen’s additive regression model（注：关于[Aalen’s additive regression model](http://www.medicine.mcgill.ca/epidemiology/hanley/bios601/Encyclopedia%20of%20Biostatistics2ndEd2005.pdf)）,文档中声明：Aalen model假设累积风险模型*H*(*t*)=*a*(*t*)+*XB*(*T*),其中*a*(*t*)是依赖于时间的截距项，*X*是该对象协变量的向量（可能依赖于时间），*B*(*t*)是一个依赖于时间系数的矩阵。

　　下面的图形将展示协变量的效应是如何随着时间而变化的，请注意`karno`斜率(slope)中的陡坡(steep slope)和突变(abrupt change)。

``` r
aa_fit <-aareg(Surv(time, status) ~ trt + celltype +
                 karno + diagtime + age + prior , 
                 data = vet)
autoplot(aa_fit)
```

![](/img/翻译：R中的生存分析_files/figure-markdown_github/unnamed-chunk-7-1.png) 
## 随机生存森林模型
　　最后一个例子，我们将展示一个“数据科学式”的方法来构建时间-事件模型，我将使用`ranger()`函数来拟合随机森林集成模型，不过要注意，对于生存数据构建树型模型并没有什么新发现。Terry Therneau和 Brian Ripley一起同样写了[`rpart`](https://cran.r-project.org/web/packages/rpart/index.html)包，可以在[rpart简介](https://cran.r-project.org/web/packages/rpart/vignettes/longintro.pdf)的section8.4中找到生存分析案例。

　　`ranger()`为数据集中的每一个观测都建立了模型，下面的代码块建立的模型同上面的cox模型使用了同样的协变量并随机绘制了二十条曲线和一条代表所有患者平均水平的曲线，下面将使用R的基础绘图。

``` r
#ranger model
r_fit <- ranger(Surv(time, status) ~ trt + celltype + 
                     karno + diagtime + age + prior,
                     data = vet,
                     mtry = 4,
                     importance = "permutation",
                     splitrule = "extratrees",
                     verbose = TRUE)
```

``` r
# Average the survival models
death_times <- r_fit$unique.death.times 
surv_prob <- data.frame(r_fit$survival)
avg_prob <- sapply(surv_prob,mean)

# Plot the survival models for each patient
plot(r_fit$unique.death.times,r_fit$survival[1,], 
     type = "l", 
     ylim = c(0,1),
     col = "red",
     xlab = "Days",
     ylab = "survival",
     main = "Patient Survival Curves")

cols <- colors()
for (n in sample(c(2:dim(vet)[1]), 20)){
  lines(r_fit$unique.death.times, r_fit$survival[n,], type = "l", col = cols[n])
}
lines(death_times, avg_prob, lwd = 2)
legend(500, 0.7, legend = c('Average = black'))
```

![](/img/翻译：R中的生存分析_files/figure-markdown_github/unnamed-chunk-9-1.png)

　　下面的代码块将展示`ranger()`是如何对变量的重要性进行排序的

``` r
vi <- data.frame(sort(round(r_fit$variable.importance, 4), decreasing = TRUE))
names(vi) <- "importance"
head(vi)
```

    ##          importance
    ## karno        0.0766
    ## celltype     0.0319
    ## trt          0.0003
    ## diagtime     0.0003
    ## prior       -0.0015
    ## age         -0.0031

　　注意到`ranger()`把`karno`和`celltype`当做两个最重要的变量，这两个变量在上面的Cox模型中同样拥有最小的p值；同样注意，重要性的结果只给出了变量名而没有给出具体水平，这是因为`ranger()`或者其他树模型不经常创建虚拟变量(dummy varaiable)。

但是`ranger`计算了[Harrell’s c-index](https://pdfs.semanticscholar.org/7705/392f1068c76669de750c6d0da8144da3304d.pdf)（在370页有定义），和上面的一致性统计量(Corcondence statistic)相似，这是ROC曲线的一般化，它又降低了二元变量的 Wilcoxon-Mann-Whitney统计量，这又相当于计算ROC曲线下的面积 
>But `ranger()` does compute [Harrell’s c-index](https://pdfs.semanticscholar.org/7705/392f1068c76669de750c6d0da8144da3304d.pdf) (See \[8\] p. 370 for the definition), which is similar to the Concordance statistic described above. This is a generalization of the ROC curve, which reduces to the Wilcoxon-Mann-Whitney statistic for binary variables, which in turn, is equivalent to computing the area under the ROC curve.

``` r
cat("Prediction Error = 1 - Harrell's c-index = ", r_fit$prediction.error)
```

    ## Prediction Error = 1 - Harrell's c-index =  0.3013403

　　在第一次尝试中ROC值等0.68，在通常情况下这已经是一个不错的数了。但是注意，`ranger()`函数在建模时并没有对那些随时间变化的系数做任何处理，这显然是一个挑战。在一篇2011年的文章中，Hamad观察到： 
>However, in the context of survival trees, a further difficulty arises when time–varying effects are included. Hence, we feel that the interpretation of covariate effects with tree ensembles in general is still mainly unsolved and should attract future research.

　　然而在生存树的背景下，模型中包含随时间变化的效应成为了一个更大的问题。因此我们认为，对于树集模型中协变量的更一般的解释仍然是没有解决的，而且应该吸引后来者的更多研究。

　　我相信，树型模型在生存分析中的主要应用应该是用于处理大数据集。 最后，我将把三种生存曲线绘制在一幅图形中，下面的代码将把三个模型对象中的生存数据取出并把他们置于同一个数据框中以便于`ggplot`绘图。

``` r
# Set up for ggplot
kmi <- rep("KM",length(km_fit$time))
km_df <- data.frame(km_fit$time,km_fit$surv,kmi)
names(km_df) <- c("Time","Surv","Model")

coxi <- rep("Cox",length(cox_fit$time))
cox_df <- data.frame(cox_fit$time,cox_fit$surv,coxi)
names(cox_df) <- c("Time","Surv","Model")

rfi <- rep("RF",length(r_fit$unique.death.times))
rf_df <- data.frame(r_fit$unique.death.times,avg_prob,rfi)
names(rf_df) <- c("Time","Surv","Model")

plot_df <- rbind(km_df,cox_df,rf_df)

p <- ggplot(plot_df, aes(x = Time, y = Surv, color = Model))
p + geom_line()
```

![](/img/翻译：R中的生存分析_files/figure-markdown_github/unnamed-chunk-12-1.png)
　　在这个数据集中，我将更相信一个精心构建的、考虑时间变化系数的Cox模型，我怀疑数据量和数据维度的不足限制了`ranger()`的发挥。

　　以上四个包只是说明了R在生存分析中的工具作用，它们也确实展示了，在近20年的不断发展和改进中，R平台的丰富性。`ranger`包推荐了`survival`包，`ggfortify`包也是基于`ggplot2`构建的，展示了开源社区是如何帮助开发人员在前任的基础上继续工作的。`survival`包的随同文档，众多的在线资源，还有Concordance和Harrell’s c-index等被编入模型拟合结果的统计量，他们给出了R的几乎所有的统计深度。 
>...the statistics such as concordance and Harrell’s c-index packed into the objects produced by fitting the models gives some idea of the statistical depth that underlies almost everything R.

一些指导手册和论文
------------------

　　非常精美的基础生存分析教程，请看[Survival analysis in R](https://www.openintro.org/download.php?file=survival_analysis_in_R&referrer=/stat/surv.php)，或者由[OpenIntro](https://www.openintro.org/download.php?file=survival_analysis_in_R&referrer=/stat/surv.php)编写的[`OISurv`](https://cran.r-project.org/web/packages/OIsurv/index.html)包（注：我们老师也给了..）。

　　[这里](http://www.medicine.mcgill.ca/epidemiology/hanley/bios601/Encyclopedia%20of%20Biostatistics2ndEd2005.pdf)是对Aalen’s Additive Regression Model的介绍。

　　一些可以用`ranger()`处理的预测性的生存分析模型的例子，一定要看一下Manuel Amunategui的[文章](http://amunategui.github.io/survival-ensembles/)和[视频](https://www.youtube.com/watch?v=6q-UFJUZK0g&list=UUq4pm1i_VZqxKVVOz5qRBIA&index=1)（注：视频需要翻墙）。

　　感谢阅读。
