---
title: Rstudio的良苦用心之Addins
tags: [R studio, little tips]
---

　　评论区正式开放

　　昨天逛[Rweekly](https://rweekly.org/)的时候发现了Rstudio社区里的这样一个帖子[What are your favorite Rstudio addins?](https://community.rstudio.com/t/what-are-your-favorite-rstudio-addins/1771)，然后就是一个恍然大明白，原来这个简单的图标里藏的还是有一些玄机的..

<!-- more -->

![Rstudio Addins](\img\r-studio-addins_files\addins.png)

　　[Rstudio Addins](https://www.rstudio.com/resources/webinars/understanding-add-ins/)，按照官方的说法就是一个为了Rstudio用户们方便工作、定制他们专用的Rstudio界面的小插件，即  
>This powerful feature enables anyone who can write R code to customize RStudio for their own work.

　　至于Addins的列表，有人在GitHub上做了总结而且做成了一个Addins，[`addinslist`](https://github.com/daattali/addinslist)（好像有点绕，就是"an addin of addins"23333），我们可以按照各个Addins的说明各取所需，当然我们使用的第一个Addins就可以从这个入手

Browse Rstudio Addins
---------------------

　　首先需要安装`addinslist`的包：

```r
install.packages("addinslist")
```

　　安装成功之后点击R脚本框上面的Addins，就可以发现我们的第一个Addins了 

![](\img\r-studio-addins_files\browse rstudio addins.png)

　　点击就会弹出一个窗口，里面就是各种各样的Rstudio Addins了，第一列是Addins名称，第二列是描述，第三列是包的名字，第四列中✔表示该包通过了CRAN的认可，然后就是作者和关于这个Addin的视频/截图/gif介绍了，其中绿色的是我们已经安装的，找到感兴趣的Addin单击即会提示安装。

![](\img\r-studio-addins_files\browse rstudio addins2.png)

　　然后我们就可以通过搜索得到`styler`和`ggThemeAssist`并安装，下面的Addins演示将以这两个插件为例。

styler
------

　　[`styler`](https://github.com/r-lib/styler)是由GitHub organization [R infrastructure](https://github.com/r-lib)编写的（真·大佬organization），一个...治代码习惯的插件。不得不说，不好的代码习惯在编程（尤其是修改代码的时候）还是有点容易搞到自己的，也让读者或者合作者看起来特别头疼..就比如说我们的非参课本的程序，那一大坨看着简直不要太费劲.. 关于R语言代码格式，可以参考一下这篇[是时候讨论一下代码规范性了](http://mp.weixin.qq.com/s/kNEK2398OZ1D9PIY37iQMw)，louwill大佬毕竟这是我在某不怎么喜欢的公众号里最喜欢的一位作者。

　　当然，通过上面`addinslist`的安装，我们现在有了`styler`，可以帮我们安排一下代码格式了 比如，现在在我们手里的是全国统计教材编审委员会“十二五”规划教材之《非参数统计学》第四版的符号检验函数

```r
sign.test <- function(x, pi, q0)
{s1=sum(x<q0);s2=sum(x>q0);n=s1+s2
p1=pbinom(s1,n,p);p2=1-pbinom(s1-1,n,p)
if (p1>p2)m1="One tail test: H1:Q>q0"
  else m1="One tail test: H1:Q<q0"
p.value=min(p1,p2);m2="Two tail test";p.value2=2*p.value
if (q0==median(x)){p.value=0.5,p.value2=1}
list(sign.test=m1, p.values.of.one.tail.test=p.value,
p.value.of.two.tail.test=p.value2)}
```

　　这是我按照P25页符号检验函数原封不动码下来的..视觉效果如何大家心里应该都有数反正我是心挺累的，如果不是强行挤印刷版面节约纸张保护环境的话，那只能说编者的不把大学生当人看了.. 当然我们现在有`styler`了，选中上面的符号函数，然后点击Addins里面的`Style Selection`，结果如下

```r
sign.test <- function(x, pi, q0) {
  s1 <- sum(x < q0)
  s2 <- sum(x > q0)
  n <- s1 + s2
  p1 <- pbinom(s1, n, p)
  p2 <- 1 - pbinom(s1 - 1, n, p)
  if (p1 > p2) {
    m1 <- "One tail test: H1:Q>q0"
  } else {
    m1 <- "One tail test: H1:Q<q0"
  }
  p.value <- min(p1, p2)
  m2 <- "Two tail test"
  p.value2 <- 2 * p.value
  if (q0 == median(x)) {
    p.value <- 0.5
    p.value2 <- 1
  }
  list(
    sign.test = m1, p.values.of.one.tail.test = p.value,
    p.value.of.two.tail.test = p.value2
  )
}
```

　　是不是舒服了很多，下一个。

ggThemeAssist
-------------

　　下一个是`ggplot`系的辅助Addins[`ggThemeAssist`](https://github.com/calligross/ggthemeassist)，`ggplot`是啥我就不讲了毕竟我真不是开公众号的..

　　就以上篇文章里面的生存分析图型为例

``` r
library(survival)
library(ggplot2)
library(ggfortify)
data(veteran)

km_trt_fit <- survfit(Surv(time, status) ~ trt, data=veteran)
autoplot(km_trt_fit)
```

![](/img/r-studio-addins_files/figure-markdown_github/unnamed-chunk-4-1.png) 这张图虽然很好看，但是我想把坐标轴、图例都换成中文的，把底色换一下，再加上标题和支持怎么办？

　　用`ggThemeAssist` 选中上面的`autoplot(km_trt_fit)`，然后点击Addins里面的ggplot Theme Assistant，就可以在里面调啊调了，记得最后点一下右上角的`Done`就好。

``` r
autoplot(km_trt_fit) + theme(axis.text = element_text(face = "bold"), 
    legend.text = element_text(face = "italic", 
        family = "CNS1"), legend.title = element_text(face = "bold.italic", 
        family = "Times"), legend.position = "left") +labs(title = "keplan-meier生存估计", x = "时间", 
    y = "生存概率", colour = "治疗", fill = "治疗", 
    caption = "powered by ggThemeAssist") + theme(panel.grid.major = element_line(linetype = "dashed"), 
    panel.background = element_rect(fill = "khaki"))
```

![](/img/r-studio-addins_files/figure-markdown_github/unnamed-chunk-5-1.png) 
(其实也不怎么好看..主要是这个用法..)

　　工具的使用建立在用户对`ggplot`各种参数的了解之上，不过我觉得如果用这个Addins反向学习`ggplot`的各个参数应该也是可行的

　　更多Addins的用法和更详细介绍可参考这些Addins中的GitHub主页，[addinslist](https://github.com/daattali/addinslist),[ggThemeAssist](https://github.com/calligross/ggthemeassist),[styler](https://github.com/r-lib/styler)。

　　抛砖引玉，感谢阅读。