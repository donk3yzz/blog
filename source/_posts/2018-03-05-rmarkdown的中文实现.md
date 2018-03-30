---
title: Rmarkdown的中文实现
author: ''
date: '2018-03-05'
slug: rmarkdown的中文实现
categories: []
tags:
  - R studio
  - little tips
---

我这个行为准确来讲叫蹭热度...

## Rmarkdown的中文输出

Rmarkdown因为结合了markdown标记语言的特性和R代码块而被广泛使用，不过使用Rmarkdown输出中文PDF却是一件令人头疼的事情，不经过配置的Rstudio直接knit输出pdf文件，中文部分会是一片空白。

虽然这个问题已经找到了可靠的解决办法，不过随着`tinytex`的诞生，这个实现又多了一种轻量级的方法。

<!-- more -->

## 中文输出空白简单原因

在Rstudio中，Rmarkdown的输出是基于`knitr`实现（这也就不奇怪为什么输出按钮名为“knit”），而`knitr`的实现rmd输出为pdf机理大概是：

- 首先运行代码块，将rmd文件转换为md文件
- 调用pandoc，将md转换为pdf文件

所以问题大概就出在第二步，pandoc实现从md到pdf的转换需要使用Latex引擎，而后者对于中文的支持...有目共睹，所以需要一个比较可靠的解决办法。

而友链里已经躺着了一篇[Rmarkdown中文实现
](https://www.jianshu.com/p/3a1d95f9965a)给出了一种比较普遍的解决方法，不过需要修正两点：

- 第二步中，不需要手动安装pandoc
- 第三步中，需要安装完整版的MikTex

关于第二条，Rstudio在安装时已经附带了pandoc，所以无需手动安装；第三条安装完整的MikTex是谢大推荐，而完整版的MikTex的体型巨大，占用系统资源不说，下载与安装过程都十分耗费时间。

## 比较轻量的中文实现过程

更轻量的Rmarkdown中文实现过程基于谢大新开发的`tinytex`([中文文档](https://yihui.name/tinytex/cn/))，目前已经在cran上发布。

中文文档中的描述如下：

>TinyTeX 是一个瘦身版的TeX Live。TeX Live 的庞大体型问题困扰我多年，在 2018 年之前我终于抽出一周时间来解决这个问题，其实方案很简单：把对普通用户毫无用处的源代码和文档去掉即可。具体技术细节参见常见问题，总体而言就是利用了TeX Live 的自动化安装方式，配置文件中设置不要安装源文件和文档。
TinyTeX 假设你不惧怕或反感使用命令行，但其实需要的命令行指令并不复杂，常见任务都可以通过一行命令搞定。如果不清楚如何打开系统命令行窗口，请参见常见问题。

不过在下载与使用前需要注意，谢大不推荐在一个电脑上同时配置多个Tex软件，如果使用tinytex，需要将电脑上原有的MikTex/TexLive/Ctex卸载。这样也意味着，如果今后有Latex的使用需求，需要完全基于tinytex实现，可能需要对tinytex的使用有一定的掌握（使用Rmarkdown来实现latex编写论文暂时是不易实现的）。

在`tinytex`的使用中出现问题可以在[其github](https://github.com/yihui/tinytex)的issue中提问。

好了下面是正题，基于`tinytex`的Rmarkdown中文实现方式和原理与前面文章中大致相同：

### 下载Rstudio和`rmarkdown`

直接利用cran安装`rmarkdown`即可

```
install.packages('rmarkdown')
```

如果安装受阻可以尝试更新R（推荐使用`installr::updater()`），或使用devtools下载。

### 下载安装`tinytex`

运行下面代码，然后等待

```
install.packages("tinytex")
tinytex::install_tinytex()
```

Windows系统需要10分钟左右的下载安装时间，取决于网速，下载完成后重启Rstudio（或重启电脑..）然后运行

```
tinytex:::is_tinytex()
```
如果返回`TRUE`则证明安装完成，即可进行下一步。

### 使用Rmarkdown模版包`rticles`

首先需要下载Rmarkdown模版包`rticles`

```
install.packages("rticles")
```

在上面步骤全部完成的前提下，点击左上角新建，选择Rmarkdown

![](/img/Rmarkdown中文实现/img_1.png)

在弹出的对话框左侧选择From Template，然后选择Ctex Document，点击OK

![](/img/Rmarkdown中文实现/img_2.png)


然后我们就得到了一个Rmarkdown的中文模版，其中有一些Ctex document介绍和简单的Rmarkdown介绍。将author, title和正文编辑为我们需要的内容，点击Knit按钮，如果语法没有问题，一篇基于Ctex的R语言pdf中文文档将完成。

![](/img/Rmarkdown中文实现/img_3.png)


### 首次使用

如果是第一次使用基于`tinytex`的Rmarkdown，点击Knit时，我们将看到类似于`tinytex`安装时的情景，这是`tinytex`开始工作，它会下载运行Ctex模版所需的latex文件，下载与安装完成后，将会得到所需pdf

## 结尾

注：谢大，即谢益辉，`knitr`,`tinytex`等包作者，`rmarkdown`,`rticles`等包重要贡献者，本人偶像。

应该是受思维和语言表达能力的限制，感觉自己废话是真的多...

感谢观看
