---
title: "Tables"
author: "冯盛世"
date: "2018年3月26日"
output: 
  html_document: 
    highlight: tango
    theme: null
---
highlight = tango






好吧又蹭了多元统计一次热度..

多元统计的实验报告是一份以Rmarkdown为基础，以latex数学公式为基础，以矩阵操作和矩阵计算为实现目标的文档，所以不免涉及到矩阵在文档中展示的问题；如果希望在文档中以latex数学公式展示矩阵，总不能寄希望于手动输入


```r
library(knitr)
```

```
## Warning: 程辑包'knitr'是用R版本3.4.4 来建造的
```

```r
library(huxtable)
```

```
## Warning: 程辑包'huxtable'是用R版本3.4.4 来建造的
```

```
## 
## 载入程辑包：'huxtable'
```

```
## The following object is masked from 'package:stats':
## 
##     filter
```


```r
set.seed(0125)
matrix_A <- matrix(data = rnorm(16, 2, 1), nrow = 4)
```

想象有一天你在写实践报告：

“建立矩阵$\mathbf A_{4\times4}$如下：

```r
matrix_A
```

```
##       [,1]    [,2]  [,3]  [,4]
## [1,] 2.933  2.3957 1.796 2.196
## [2,] 1.475 -0.1937 2.446 2.715
## [3,] 3.814  1.6397 1.678 1.040
## [4,] 2.083  2.1429 2.478 2.671
```
则矩阵$\mathbf A$...”

这这这，是不是有点太丑了hhh

那么R中有没有包能够直接把矩阵直接用latex公式输出的呢？

## 使用`xtable`展示latex矩阵

[`xtable`](http://xtable.r-forge.r-project.org/)可以将R对象转换为'xtable'对象，以便于将其打印为LaTex或HTML表格。可以在cran上直接安装。

其中我们需要的函数为`xtable::xtableMatharray()`，可以将我们的$\mathbf A$转换为LaTex的array形式输出：


```r
library(xtable)
```

```
## 
## 载入程辑包：'xtable'
```

```
## The following objects are masked from 'package:huxtable':
## 
##     align, align<-, caption, caption<-, label, label<-,
##     sanitize
```

```r
mtb <- xtableMatharray(matrix_A, digits = 5)


class(mtb)
```

```
## [1] "xtableMatharray" "xtable"          "data.frame"
```

```r
str(mtb)
```

```
## Classes 'xtableMatharray', 'xtable' and 'data.frame':	4 obs. of  4 variables:
##  $ 1: num  2.93 1.47 3.81 2.08
##  $ 2: num  2.396 -0.194 1.64 2.143
##  $ 3: num  1.8 2.45 1.68 2.48
##  $ 4: num  2.2 2.71 1.04 2.67
##  - attr(*, "align")= chr  "r" "r" "r" "r" ...
##  - attr(*, "digits")= num  5 5 5 5 5
##  - attr(*, "display")= chr  "s" "f" "f" "f" ...
```

```r
mtb
```

```
## \begin{array}{rrrr}
##   2.93333 & 2.39572 & 1.79627 & 2.19617 \\ 
##   1.47497 & -0.19367 & 2.44562 & 2.71483 \\ 
##   3.81444 & 1.63968 & 1.67841 & 1.03986 \\ 
##   2.08305 & 2.14285 & 2.47848 & 2.67072 \\ 
##   \end{array}
```

可以看到`mtb`是一个`"xtableMatharray""xtable"  "data.frame"`的类型，如果在Rstudio中用`view(mtb)`调用则会得到一个数据框，而如果直接请求`mtb`则会返回$\mathbf A$的LaTex array编码。

然后我们调用`print()`

```
```r, results='asis'
print(mtb)
```
```


```r
print(mtb)
```

\begin{array}{rrrr}
  2.93333 & 2.39572 & 1.79627 & 2.19617 \\ 
  1.47497 & -0.19367 & 2.44562 & 2.71483 \\ 
  3.81444 & 1.63968 & 1.67841 & 1.03986 \\ 
  2.08305 & 2.14285 & 2.47848 & 2.67072 \\ 
  \end{array}
就可以展示LaTex公式下的$\mathbf A$了。

## 原理简述

我在前一篇文章里简单说过Rmarkdown输出pdf文档时的工作原理，这里也类似，基于基于pandoc的`knitr`的文档输出模式，可以让我们在从Rmarkdown文档到html文件的转换过程中，同时使用R代码、markdown语法、html语法和html-mathjax支持下的LaTex数学符号。

（我们将在后面关于`kable()`的章节中复习此原理）

所以可以推测，在`xtableMatharray()`已经将$\mathbf A$的LaTex公式转换完成的情况下，欲将该公式在最终html文档中输出，有两种途径：

* 将`xtablMatharray()`的结果复制粘贴在Rmarkdown文件中的某对$$$$中

* 将在`xtablMatharray()`的输出结果与`knitr`的转换之间建立关联

前者简单易懂，但是不符合我们可重复性报告的原则；后者也不难实现，只需要在代码块中调整参数`results = 'asis'`

```r results='asis'




## 其他tables们

事实上在Rmarkdown中输出tables的需求并不只有LaTex矩阵一种，大部分用户的输出需求还是对于`data.frame`的需求，而输出文档也一些是涵盖了markdown, pdf, word, html种种。下面是对一些其他的tables输出函数的简单介绍。

### xtable()

`xtable::xtableMatharray()`是专门用来输出LaTex array制式的函数，而`xtable()`可以用于输出LaTex或html格式。


```r
xt_iris <- xtable(head(iris))

print(xt_iris)
```

```
## % latex table generated in R 3.4.3 by xtable 1.8-2 package
## % Fri Mar 30 10:14:22 2018
## \begin{table}[ht]
## \centering
## \begin{tabular}{rrrrrl}
##   \hline
##  & Sepal.Length & Sepal.Width & Petal.Length & Petal.Width & Species \\ 
##   \hline
## 1 & 5.10 & 3.50 & 1.40 & 0.20 & setosa \\ 
##   2 & 4.90 & 3.00 & 1.40 & 0.20 & setosa \\ 
##   3 & 4.70 & 3.20 & 1.30 & 0.20 & setosa \\ 
##   4 & 4.60 & 3.10 & 1.50 & 0.20 & setosa \\ 
##   5 & 5.00 & 3.60 & 1.40 & 0.20 & setosa \\ 
##   6 & 5.40 & 3.90 & 1.70 & 0.40 & setosa \\ 
##    \hline
## \end{tabular}
## \end{table}
```

```r
print(xt_iris, type = 'html')
```

```
## <!-- html table generated in R 3.4.3 by xtable 1.8-2 package -->
## <!-- Fri Mar 30 10:14:22 2018 -->
## <table border=1>
## <tr> <th>  </th> <th> Sepal.Length </th> <th> Sepal.Width </th> <th> Petal.Length </th> <th> Petal.Width </th> <th> Species </th>  </tr>
##   <tr> <td align="right"> 1 </td> <td align="right"> 5.10 </td> <td align="right"> 3.50 </td> <td align="right"> 1.40 </td> <td align="right"> 0.20 </td> <td> setosa </td> </tr>
##   <tr> <td align="right"> 2 </td> <td align="right"> 4.90 </td> <td align="right"> 3.00 </td> <td align="right"> 1.40 </td> <td align="right"> 0.20 </td> <td> setosa </td> </tr>
##   <tr> <td align="right"> 3 </td> <td align="right"> 4.70 </td> <td align="right"> 3.20 </td> <td align="right"> 1.30 </td> <td align="right"> 0.20 </td> <td> setosa </td> </tr>
##   <tr> <td align="right"> 4 </td> <td align="right"> 4.60 </td> <td align="right"> 3.10 </td> <td align="right"> 1.50 </td> <td align="right"> 0.20 </td> <td> setosa </td> </tr>
##   <tr> <td align="right"> 5 </td> <td align="right"> 5.00 </td> <td align="right"> 3.60 </td> <td align="right"> 1.40 </td> <td align="right"> 0.20 </td> <td> setosa </td> </tr>
##   <tr> <td align="right"> 6 </td> <td align="right"> 5.40 </td> <td align="right"> 3.90 </td> <td align="right"> 1.70 </td> <td align="right"> 0.40 </td> <td> setosa </td> </tr>
##    </table>
```


```r
print(xt_iris)
```

% latex table generated in R 3.4.3 by xtable 1.8-2 package
% Fri Mar 30 10:14:22 2018
\begin{table}[ht]
\centering
\begin{tabular}{rrrrrl}
  \hline
 & Sepal.Length & Sepal.Width & Petal.Length & Petal.Width & Species \\ 
  \hline
1 & 5.10 & 3.50 & 1.40 & 0.20 & setosa \\ 
  2 & 4.90 & 3.00 & 1.40 & 0.20 & setosa \\ 
  3 & 4.70 & 3.20 & 1.30 & 0.20 & setosa \\ 
  4 & 4.60 & 3.10 & 1.50 & 0.20 & setosa \\ 
  5 & 5.00 & 3.60 & 1.40 & 0.20 & setosa \\ 
  6 & 5.40 & 3.90 & 1.70 & 0.40 & setosa \\ 
   \hline
\end{tabular}
\end{table}

```r
print(xt_iris, type = 'html')
```

<!-- html table generated in R 3.4.3 by xtable 1.8-2 package -->
<!-- Fri Mar 30 10:14:22 2018 -->
<table border=1>
<tr> <th>  </th> <th> Sepal.Length </th> <th> Sepal.Width </th> <th> Petal.Length </th> <th> Petal.Width </th> <th> Species </th>  </tr>
  <tr> <td align="right"> 1 </td> <td align="right"> 5.10 </td> <td align="right"> 3.50 </td> <td align="right"> 1.40 </td> <td align="right"> 0.20 </td> <td> setosa </td> </tr>
  <tr> <td align="right"> 2 </td> <td align="right"> 4.90 </td> <td align="right"> 3.00 </td> <td align="right"> 1.40 </td> <td align="right"> 0.20 </td> <td> setosa </td> </tr>
  <tr> <td align="right"> 3 </td> <td align="right"> 4.70 </td> <td align="right"> 3.20 </td> <td align="right"> 1.30 </td> <td align="right"> 0.20 </td> <td> setosa </td> </tr>
  <tr> <td align="right"> 4 </td> <td align="right"> 4.60 </td> <td align="right"> 3.10 </td> <td align="right"> 1.50 </td> <td align="right"> 0.20 </td> <td> setosa </td> </tr>
  <tr> <td align="right"> 5 </td> <td align="right"> 5.00 </td> <td align="right"> 3.60 </td> <td align="right"> 1.40 </td> <td align="right"> 0.20 </td> <td> setosa </td> </tr>
  <tr> <td align="right"> 6 </td> <td align="right"> 5.40 </td> <td align="right"> 3.90 </td> <td align="right"> 1.70 </td> <td align="right"> 0.40 </td> <td> setosa </td> </tr>
   </table>

可以看到，`xtable()`可以输出LaTex和html制式的表格，事实上`xtable()`中还有很多参数可以调整，以满足定制需求（排版函数一般皆是如此，下同）。

### 代表了更先进生产力的`huxtable`

[`huxtable`](https://hughjonesd.github.io/huxtable/)是一个用于创建html和LaTex table的R包，与`xtable`提供的功能相似，不过其具有更简洁的使用方法。除去基本的表格效果控制以外，它的主要功能包括：

* 可以输出为LaTex, Word, HTML和Markdown
* 易与`knitr`和Rmarkdown进行结合
* 支持R中标准的subsetting操作，支持`dplyr`函数，支持管道操作
* 具有快速table主题
* 使用`huxreg()`函数自动创建回归输出表

建立`hux`对象，支持`dplyr`操作，支持主题……这这这…这不是table界的`ggplot2`吗？（就是似乎函数形式有点缺乏美感…）

在基于`knitr`的Rmarkdown文档中，使用`huxtable`在Rmarkdown中输出表格不需要多余的设置，直接调用该hux对象本身即可：


```r
library(huxtable)
library(dplyr)

data(mtcars)
car_ht <- as_hux(mtcars) %>%
  huxtable::add_rownames(colname = "Car") %>%
  ## dplyr操作
  slice(1:10) %>%
  select(Car, mpg, cyl, hp) %>%
  arrange(hp) %>%
  filter(cyl > 4) %>%
  rename(MPG = mpg, Cylinders = cyl, Horsepower = hp) %>%
  mutate(kml = MPG / 2.82) %>%
  ## huxtable操作
  set_number_format(1:7, "kml", 2) %>%
  set_col_width(c(.35, .15, .15, .15, .2)) %>%
  set_width(.6) %>%
  add_colnames() %>%
  theme_striped(stripe = grDevices::grey(0.9), 
                header_row = T, header_col = F)
car_ht
```

```
## Error in yaml::yaml.load(string, ...): Scanner error: while scanning a simple key at line 2, column 1 could not find expected ':' at line 3, column 1
```
关于`huxtable`的更多细节请参考[Introduction to Huxtable
](https://hughjonesd.github.io/huxtable/huxtable.html)


### 亲儿子`kable()`和基友`kableExtra`

`kable()`是`knitr`中自带的制表函数，为了支持knitr的多种文档的需求，`kable()`支持latex, html, markdown, pandoc和rst制式的表格。

当然了，作为`knitr`的亲儿子，制表是不需要多余操作的，只需要简单的调用`kable()`就可以了，


```r
library(knitr)
kable(head(iris))
```



| Sepal.Length| Sepal.Width| Petal.Length| Petal.Width|Species |
|------------:|-----------:|------------:|-----------:|:-------|
|          5.1|         3.5|          1.4|         0.2|setosa  |
|          4.9|         3.0|          1.4|         0.2|setosa  |
|          4.7|         3.2|          1.3|         0.2|setosa  |
|          4.6|         3.1|          1.5|         0.2|setosa  |
|          5.0|         3.6|          1.4|         0.2|setosa  |
|          5.4|         3.9|          1.7|         0.4|setosa  |

相当于调用：
```r
kable(head(iris), format = 'html')
```

#### 插曲

我们同样可以在Rmarkdown中指定`kable()`的输出形式。不过，由于我们使用的是HTML文档，参数`format="latex"`的调用会没有结果显示；指定`format='markdown'`或`format='pandoc'`会输出markdown语法和pandoc语法的表格，不过它们会在下面的转义过程中被转义为HTML语法。

```r
> kable(head(iris), format = "latex")

\begin{tabular}{r|r|r|r|l}
\hline
Sepal.Length & Sepal.Width & Petal.Length & Petal.Width & Species\\
\hline
5.1 & 3.5 & 1.4 & 0.2 & setosa\\
\hline
4.9 & 3.0 & 1.4 & 0.2 & setosa\\
\hline
4.7 & 3.2 & 1.3 & 0.2 & setosa\\
\hline
4.6 & 3.1 & 1.5 & 0.2 & setosa\\
\hline
5.0 & 3.6 & 1.4 & 0.2 & setosa\\
\hline
5.4 & 3.9 & 1.7 & 0.4 & setosa\\
\hline
\end{tabular}
```
```r
> kable(head(iris), format = "markdown")

| Sepal.Length| Sepal.Width| Petal.Length| Petal.Width|Species |
|------------:|-----------:|------------:|-----------:|:-------|
|          5.1|         3.5|          1.4|         0.2|setosa  |
|          4.9|         3.0|          1.4|         0.2|setosa  |
|          4.7|         3.2|          1.3|         0.2|setosa  |
|          4.6|         3.1|          1.5|         0.2|setosa  |
|          5.0|         3.6|          1.4|         0.2|setosa  |
|          5.4|         3.9|          1.7|         0.4|setosa  |
```

```r
kable(head(iris), format = "latex")
```


\begin{tabular}{r|r|r|r|l}
\hline
Sepal.Length & Sepal.Width & Petal.Length & Petal.Width & Species\\
\hline
5.1 & 3.5 & 1.4 & 0.2 & setosa\\
\hline
4.9 & 3.0 & 1.4 & 0.2 & setosa\\
\hline
4.7 & 3.2 & 1.3 & 0.2 & setosa\\
\hline
4.6 & 3.1 & 1.5 & 0.2 & setosa\\
\hline
5.0 & 3.6 & 1.4 & 0.2 & setosa\\
\hline
5.4 & 3.9 & 1.7 & 0.4 & setosa\\
\hline
\end{tabular}

```r
kable(head(iris), format = "markdown")
```



| Sepal.Length| Sepal.Width| Petal.Length| Petal.Width|Species |
|------------:|-----------:|------------:|-----------:|:-------|
|          5.1|         3.5|          1.4|         0.2|setosa  |
|          4.9|         3.0|          1.4|         0.2|setosa  |
|          4.7|         3.2|          1.3|         0.2|setosa  |
|          4.6|         3.1|          1.5|         0.2|setosa  |
|          5.0|         3.6|          1.4|         0.2|setosa  |
|          5.4|         3.9|          1.7|         0.4|setosa  |

### `kableExtra`
`kable()`因为其原生于`knitr`、调用简单方便而广受欢迎，而`kableExtra`则满足了`kable()`的更多定制性需求。


```r
library(kableExtra)
```

```
## Warning: 程辑包'kableExtra'是用R版本3.4.4 来建造的
```

```
## 
## 载入程辑包：'kableExtra'
```

```
## The following object is masked from 'package:huxtable':
## 
##     add_footnote
```

```r
dt <- mtcars[1:5, 1:6]
kable(dt)
```



|                  |  mpg| cyl| disp|  hp| drat|    wt|
|:-----------------|----:|---:|----:|---:|----:|-----:|
|Mazda RX4         | 21.0|   6|  160| 110| 3.90| 2.620|
|Mazda RX4 Wag     | 21.0|   6|  160| 110| 3.90| 2.875|
|Datsun 710        | 22.8|   4|  108|  93| 3.85| 2.320|
|Hornet 4 Drive    | 21.4|   6|  258| 110| 3.08| 3.215|
|Hornet Sportabout | 18.7|   8|  360| 175| 3.15| 3.440|

```r
kable(dt, "html") %>%
  kable_styling(bootstrap_options = c("striped", "hover"))
```

<table class="table table-striped table-hover" style="margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;">   </th>
   <th style="text-align:right;"> mpg </th>
   <th style="text-align:right;"> cyl </th>
   <th style="text-align:right;"> disp </th>
   <th style="text-align:right;"> hp </th>
   <th style="text-align:right;"> drat </th>
   <th style="text-align:right;"> wt </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> Mazda RX4 </td>
   <td style="text-align:right;"> 21.0 </td>
   <td style="text-align:right;"> 6 </td>
   <td style="text-align:right;"> 160 </td>
   <td style="text-align:right;"> 110 </td>
   <td style="text-align:right;"> 3.90 </td>
   <td style="text-align:right;"> 2.620 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Mazda RX4 Wag </td>
   <td style="text-align:right;"> 21.0 </td>
   <td style="text-align:right;"> 6 </td>
   <td style="text-align:right;"> 160 </td>
   <td style="text-align:right;"> 110 </td>
   <td style="text-align:right;"> 3.90 </td>
   <td style="text-align:right;"> 2.875 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Datsun 710 </td>
   <td style="text-align:right;"> 22.8 </td>
   <td style="text-align:right;"> 4 </td>
   <td style="text-align:right;"> 108 </td>
   <td style="text-align:right;"> 93 </td>
   <td style="text-align:right;"> 3.85 </td>
   <td style="text-align:right;"> 2.320 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Hornet 4 Drive </td>
   <td style="text-align:right;"> 21.4 </td>
   <td style="text-align:right;"> 6 </td>
   <td style="text-align:right;"> 258 </td>
   <td style="text-align:right;"> 110 </td>
   <td style="text-align:right;"> 3.08 </td>
   <td style="text-align:right;"> 3.215 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Hornet Sportabout </td>
   <td style="text-align:right;"> 18.7 </td>
   <td style="text-align:right;"> 8 </td>
   <td style="text-align:right;"> 360 </td>
   <td style="text-align:right;"> 175 </td>
   <td style="text-align:right;"> 3.15 </td>
   <td style="text-align:right;"> 3.440 </td>
  </tr>
</tbody>
</table>
  

```r
DT::datatable(dt)
```

```
## PhantomJS not found. You can install it with webshot::install_phantomjs(). If it is installed, please make sure the phantomjs executable can be found via the PATH variable.
```

```
## Warning in normalizePath(path.expand(path), winslash, mustWork):
## path[1]=".\webshot331039227cc2.png": 系统找不到指定的文件。
```

```
## Warning in file(con, "rb"): 无法打开文件'C:\Users\ASUS\AppData\Local
## \Temp\Rtmp2RuwnG\file33102a34399e\webshot331039227cc2.png': No such
## file or directory
```

```
## Error in file(con, "rb"): 无法打开链结
```

```r
popover_dt <- data.frame(
  position = c("top", "bottom", "right", "left"),
  stringsAsFactors = FALSE
)
popover_dt$`Hover over these items` <- cell_spec(
  paste("Message on", popover_dt$position), # Cell texts
  popover = spec_popover(
    content = popover_dt$position,
    title = NULL,                           # title will add a Title Panel on top
    position = popover_dt$position
  ))
```

```
## Setting cell_spec format as html
```

```r
kable(popover_dt, "html", escape = FALSE) %>%
  kable_styling("striped", full_width = FALSE)
```

<table class="table table-striped" style="width: auto !important; margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;"> position </th>
   <th style="text-align:left;"> Hover over these items </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> top </td>
   <td style="text-align:left;"> <span style="     " data-toggle="popover" data-trigger="hover" data-placement="top" data-content="top">Message on top</span> </td>
  </tr>
  <tr>
   <td style="text-align:left;"> bottom </td>
   <td style="text-align:left;"> <span style="     " data-toggle="popover" data-trigger="hover" data-placement="bottom" data-content="bottom">Message on bottom</span> </td>
  </tr>
  <tr>
   <td style="text-align:left;"> right </td>
   <td style="text-align:left;"> <span style="     " data-toggle="popover" data-trigger="hover" data-placement="right" data-content="right">Message on right</span> </td>
  </tr>
  <tr>
   <td style="text-align:left;"> left </td>
   <td style="text-align:left;"> <span style="     " data-toggle="popover" data-trigger="hover" data-placement="left" data-content="left">Message on left</span> </td>
  </tr>
</tbody>
</table>

