---
title: 随时随地用markdown写作
date: 2019-10-20 12:16:09
thumbnail: /gallery/thumbnail/markdown.png
categories:
    - 工具介绍
tags:
    - 推荐
    - tools
---
&emsp;&emsp;前些天做了一个类似知乎的web应用，使用react + ant design pro，由于需要一个富文本编辑插件，寻找了好久，找到了一个相对合适的react富文本编辑插件： react-draft-wysiwyg，最后勉强实现了效果， 但是这个插件在react 16 以上的版本有部分兼容性问题， 查看github貌似这个项目也没有人继续维护了。于是就在想，是否富文本编辑器是体验最好的，于是我看到了markdown。 

<!-- more -->

先说说我觉得markdown编辑的好处：
1. 可以以一种比较简单的方式处理格式，个人觉得比较简化，可能是程序员的福音吧；
2. 现在绝大多数的博客网站或社交网站都支持markdown转换文章；
3. 使用markdown编辑可以在没有网络或不使用编辑后台的方式离线编辑后上传；  

其次是个人发现markdown的不足：
1. 需要一个入门的过程，不像富文本那样大家都会，毕竟word应该没人没用过；
2. 没有富文本那样的上传文件（图片、视频）那样方便，要么引用静态文件，要么使用类似oss这样的文件服务；
3. 排版的方面要多注意，样式什么的需要写一些符号来支持，总之需要一点门槛吧；

## 快速开始  

### 编辑器推荐（排名不分先后）

``` bash
1. Visual studio Code （安装preview插件： Markdown Preview Enhanced）
2. MarkdownPad
3. BookPad(window10)
4. Typora
5. Atom
7. ...其他一些编辑器都可以，但是最好可以preview，毕竟可以看见自己写的是不是自己所想的那样
8. 还有许多在线markdown编辑的网站和chrome的插件，但我觉得下一个编辑器会方便的多
```

### 3分钟入门语法  

#### 标题分级
``` bash
# 一级标题
## 二级标题
### 三级标题
#### 四级标题
##### 五级标题
###### 六级标题
```

#### 段落格式
``` bash
首行缩进： &emsp; 一个空文字 &nbsp; 一个空字符 中文使用&emsp;&emsp;
段落换行： 最好段尾最后加上2空格再回车
文本居中： <center>我要居中</center>
文本居右： <p align="right">我要居右</p> 其实就是html语法
斜体文本： *斜体文本* / _斜体文本_
粗体文本： **粗体文本** / __粗体文本__
粗斜文本： ***粗斜文本*** / ___粗斜文本___
分隔线：   *** / --- (三个以上就行)
删除线：   ~~删除线~~
下划线：   <u>下划线</u>
标注：     xxx[^我是标注] 需要声明标注的解释： [^我是标注]: 我是标注的声明
超链接：   [百度](http://baidu.com)
图片：     ![alt text](url)  这里的url可以是网页上的图片url，也可以是自己oss中的图片链接，还可以是静态资源的引用（比如此文档统计目录下的图片 img.jpg 可以使用 ![alt](img.png) 的方式引入）
区块：     > 我是区块 （每次嵌套换行多一个 > ）
```

#### 列表格式
``` bash
无序列表： */+/- 开头
有序列表： 1. 内容 （注意. 之后的空格）
嵌套列表： 一层列表之后换行以 - 开头
          1. 内容
            - 二级 （同样要注意- 后的空格）
```

#### 代码片段
推荐方式：
 
> `` ``` 语言(java) ``
> public static void main(String[] args) {
> &emsp;System.out.println("Hello world!");
> }
> `` ``` ``   

#### 表格
``` bash 
后台接口开发者一般都有过用markdown写接口文档的经历

|名称  | 类型 | 长度 | 非空 | 示例 |
|:--- |---:  |:---: |-----|------|
|left |right |center |true |text |
|参数1|String| 64   | true | xxxx |

其中的 : 存在的方向用来标识字段在table中的位置
```

#### 数学公式（如果用不到可以先跳过）

``` bash

1. 行内与独行
  行内公式：将公式插入到本行内，符号：$公式内容$，如：$xyz$
  独行公式：将公式插入到新的一行内，并且居中，符号：$$公式内容$$，如：$$xyz$$

2. 上标、下标与组合
  上标符号，符号：^，如：$x^4$
  下标符号，符号：_，如：$x_1$
  组合符号，符号：{}，如：${16}_{8}O{2+}_{2}$

3. 汉字、字体与格式
  汉字形式，符号：mbox{}，如：$V_{mbox{初始}}$
  字体控制，符号：displaystyle，如：$\displaystyle \frac{x+y}{y+z}$
  下划线符号，符号：underline，如：$\underline{x+y}$
  标签，符号\tag{数字}，如：$tag{11}$
  上大括号，符号：overbrace{算式}，如：$\overbrace{a+b+c+d}^{2.0}$
  下大括号，符号：underbrace{算式}，如：$a+\underbrace{b+c}_{1.0}+d$
  上位符号，符号：stacrel{上位符号}{基位符号}，如：$\vec{x}stackrel{mathrm{def}}{=}{x_1,\dots,x_n}$

4. 占位符
  两个quad空格，符号：\qquad，如：$x \qquad y$
  quad空格，符号：\quad，如：$x \quad y$
  大空格，符号\，如：$x \ y$
  中空格，符号\:，如：$x : y$
  小空格，符号\,，如：$x , y$
  没有空格，符号``，如：$xy$
  紧贴，符号\!，如：$x ! y$

5. 定界符与组合
  括号，符号：（）\big(\big) \Big(\Big) \bigg(\bigg) \Bigg(\Bigg)，如：$（）\big(\big) \Big(\Big) \bigg(\bigg) \Bigg(\Bigg)$
  中括号，符号：[]，如：$[x+y]$
  大括号，符号：\{ \}，如：${x+y}$
  自适应括号，符号：\left \right，如：$\left(x\right)$，$\left(x{yz}\right)$
  组合公式，符号：{上位公式 \choose 下位公式}，如：${n+1 \choose k}={n \choose k}+{n \choose k-1}$
  组合公式，符号：{上位公式 \atop 下位公式}，如：$\sum_{k_0,k_1,\ldots>0 \atop k_0+k_1+\cdots=n}A_{k_0}A_{k_1}\cdots$

6. 四则运算
  加法运算，符号：+，如：$x+y=z$
  减法运算，符号：-，如：$x-y=z$
  加减运算，符号：\pm，如：$x \pm y=z$
  减甲运算，符号：\mp，如：$x \mp y=z$
  乘法运算，符号：\times，如：$x \times y=z$
  点乘运算，符号：\cdot，如：$x \cdot y=z$
  星乘运算，符号：\ast，如：$x \ast y=z$
  除法运算，符号：\div，如：$x \div y=z$
  斜法运算，符号：/，如：$x/y=z$
  分式表示，符号：\frac{分子}{分母}，如：$\frac{x+y}{y+z}$
  分式表示，符号：{分子} \voer {分母}，如：${x+y} \over {y+z}$
  绝对值表示，符号：||，如：$|x+y|$

7. 高级运算
  平均数运算，符号：\overline{算式}，如：$\overline{xyz}$
  开二次方运算，符号：\sqrt，如：$\sqrt x$
  开方运算，符号：\sqrt[开方数]{被开方数}，如：$\sqrt[3]{x+y}$
  对数运算，符号：\log，如：$\log(x)$
  极限运算，符号：\lim，如：$\lim^{x \to \infty}_{y \to 0}{\frac{x}{y}}$
  极限运算，符号：\displaystyle \lim，如：$\displaystyle \lim^{x \to \infty}_{y \to 0}{\frac{x}{y}}$
  求和运算，符号：\sum，如：$\sum^{x \to \infty}_{y \to 0}{\frac{x}{y}}$
  求和运算，符号：\displaystyle \sum，如：$\displaystyle \sum^{x \to \infty}_{y \to 0}{\frac{x}{y}}$
  积分运算，符号：\int，如：$\int^{\infty}_{0}{xdx}$
  积分运算，符号：\displaystyle \int，如：$\displaystyle \int^{\infty}_{0}{xdx}$
  微分运算，符号：\partial，如：$\frac{\partial x}{\partial y}$
  矩阵表示，符号：\begin{matrix} \end{matrix}，如：$$
  \begin{bmatrix}
    1 & 2 & 3 \\
    4 & 5 & 6 \\
    7 & 8 & 9
    \end{bmatrix} \tag{1}
$$

8. 逻辑运算
  等于运算，符号：=，如：$x+y=z$
  大于运算，符号：>，如：$x+y>z$
  小于运算，符号：<，如：$x+y<z$
  大于等于运算，符号：\geq，如：$x+y \geq z$
  小于等于运算，符号：\leq，如：$x+y \leq z$
  不等于运算，符号：\neq，如：$x+y \neq z$
  不大于等于运算，符号：\ngeq，如：$x+y \ngeq z$
  不大于等于运算，符号：\not\geq，如：$x+y \not\geq z$
  不小于等于运算，符号：\nleq，如：$x+y \nleq z$
  不小于等于运算，符号：\not\leq，如：$x+y \not\leq z$
  约等于运算，符号：\approx，如：$x+y \approx z$
  恒定等于运算，符号：\equiv，如：$x+y \equiv z$

9. 集合运算
  属于运算，符号：\in，如：$x \in y$
  不属于运算，符号：\notin，如：$x \notin y$
  不属于运算，符号：\not\in，如：$x \not\in y$
  子集运算，符号：\subset，如：$x \subset y$
  子集运算，符号：\supset，如：$x \supset y$
  真子集运算，符号：\subseteq，如：$x \subseteq y$
  非真子集运算，符号：\subsetneq，如：$x \subsetneq y$
  真子集运算，符号：\supseteq，如：$x \supseteq y$
  非真子集运算，符号：\supsetneq，如：$x \supsetneq y$
  非子集运算，符号：\not\subset，如：$x \not\subset y$
  非子集运算，符号：\not\supset，如：$x \not\supset y$
  并集运算，符号：\cup，如：$x \cup y$
  交集运算，符号：\cap，如：$x \cap y$
  差集运算，符号：\setminus，如：$x \setminus y$
  同或运算，符号：\bigodot，如：$x \bigodot y$
  同与运算，符号：\bigotimes，如：$x \bigotimes y$
  实数集合，符号：\mathbb{R}，如：\mathbb{R}
  自然数集合，符号：\mathbb{Z}，如：\mathbb{Z}
  空集，符号：\emptyset，如：$\emptyset$ 

10. 数学符号
  无穷，符号：\infty，如：$\infty$
  虚数，符号：\imath，如：$\imath$
  虚数，符号：\jmath，如：$\jmath$
  数学符号，符号\hat{a}，如：$\hat{a}$
  数学符号，符号\check{a}，如：$\check{a}$
  数学符号，符号\breve{a}，如：$\breve{a}$
  数学符号，符号\tilde{a}，如：$\tilde{a}$
  数学符号，符号\bar{a}，如：$\bar{a}$
  矢量符号，符号\vec{a}，如：$\vec{a}$
  数学符号，符号\acute{a}，如：$\acute{a}$
  数学符号，符号\grave{a}，如：$\grave{a}$
  数学符号，符号\mathring{a}，如：$\mathring{a}$
  一阶导数符号，符号\dot{a}，如：$\dot{a}$
  二阶导数符号，符号\ddot{a}，如：$\ddot{a}$
  上箭头，符号：\uparrow，如：$\uparrow$
  上箭头，符号：\Uparrow，如：$\Uparrow$
  下箭头，符号：\downarrow，如：$\downarrow$
  下箭头，符号：\Downarrow，如：$\Downarrow$
  左箭头，符号：\leftarrow，如：$\leftarrow$
  左箭头，符号：\Leftarrow，如：$\Leftarrow$
  右箭头，符号：\rightarrow，如：$\rightarrow$
  右箭头，符号：\Rightarrow，如：$\Rightarrow$
  底端对齐的省略号，符号：\ldots，如：$1,2,\ldots,n$
  中线对齐的省略号，符号：\cdots，如：$x_1^2 + x_2^2 + \cdots + x_n^2$
  竖直对齐的省略号，符号：\vdots，如：$\vdots$
  斜对齐的省略号，符号：\ddots，如：$\ddots$
```

#### 参考链接

[数学公式: https://www.jianshu.com/p/e74eb43960a1](https://www.jianshu.com/p/e74eb43960a1)
[菜鸟教程: https://www.runoob.com/markdown/md-lists.html](https://www.runoob.com/markdown/md-lists.html)
[画矩阵: https://blog.csdn.net/qq_38228254/article/details/79469727](https://blog.csdn.net/qq_38228254/article/details/79469727)
