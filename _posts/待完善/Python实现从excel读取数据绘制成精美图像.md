# **Python实现从excel读取数据绘制成精美图像**

## 一、实验介绍

### 1.1 实验内容

这个世界从古至今一直是一个看颜值的世界。对于我们作报告，写文章时使用的图片，也是一样的。一图胜千言，一张制作精美的图片，不仅能展示大量的信息，更能体现绘图者的水平，审美，与态度。我的老板，国内外多家`SCI`，`EI`文章的审稿人，甚至跟我说，一篇文章拿到手里，一眼扫过去，看看数据和图片，就知道这篇文章值不值得发表，水平如何。由此观之，制作一张精美图片的意义，实在重大。本课程实现使用`python`从`excel`读取数据，并使用`matplotlib`绘制成二维图像。这一过程中，将通过一系列操作来美化图像，最终得到一个可以出版级别的图像。本课程对于需要书写实验报告，学位论文，发表文章，做`PPT`报告的学员具有较大价值。本课程的数据和图像，来源于我的一篇`SCI`文章，是一真实案例。

### 1.2 实验知识点

- 使用`xlrd`扩展包读取`excel`数据
- 使用`matplotlib`绘制二维图像
- 美化图像，添加标注，注释，显示`Latex`风格公式，坐标点处透明化处理等技巧

### 1.3 实验环境

- `python2.7`
- `Xfce`终端

### 1.4 适合人群

本课程难度为中等，适合具有`Python`基础的用户，对于需要书写实验报告，学位论文，发表文章，做`PPT`报告的学员具有较大价值。

### 1.5 代码获取

你可以通过下面命令将数据和代码下载到实验楼环境中，作为参照对比进行学习。

```bash
$ wget http://labfile.oss.aliyuncs.com/courses/791/finally.py
$ wget http://labfile.oss.aliyuncs.com/courses/791/my_data.xlsx
$ wget http://labfile.oss.aliyuncs.com/courses/791/phase_detector.xlsx
$ wget http://labfile.oss.aliyuncs.com/courses/791/phase_detector2.xlsx
```

## 二、开发准备

打开`Xfce`终端，下载并安装的相关依赖 。

```bash
$ sudo apt-get update
$ sudo apt-get install python-dev
$ sudo pip install numpy
$ sudo apt-get install python-matplotlib
$ sudo pip install xlrd
$ sudo apt-get install python-sip
$ sudo apt-get install libqt4-dev
$ sudo apt-get install python-qt4 python-qt4-dev pyqt4-dev-tools qt4-dev-tools
```

遇到是否安装的询问时，输入y，按回车键继续安装。

## 三、实验步骤

### 3.1 绘制一个简单图像，测试扩展包安装是否正常

安装完成`matplotlib`后，运行一个小程序测试其是否正常。我们来绘制一个非常简单的正弦函数。

```Python
import numpy as np
import matplotlib.pyplot as plt


x = np.linspace(0, 10, 500)
dashes = [10, 5, 100, 5]  # 10 points on, 5 off, 100 on, 5 off

fig, ax = plt.subplots()
line1, = ax.plot(x, np.sin(x), '--', linewidth=2,
                 label='Dashes set retroactively')
line1.set_dashes(dashes)

line2, = ax.plot(x, -1 * np.sin(x), dashes=[30, 5, 10, 5],
                 label='Dashes set proactively')

ax.legend(loc='lower right')
plt.show()
```

如果一切正常，应该得到如下显示的图片：

![此处输入图片的描述](D:\Blog\LyricYang.github.io\img\Python\0.9134969067050864.png)

这段程序来自官方的例程，只作为检验安装包之用。这个图片过于简单，也算不上精美。

### 3.2 测试`xlrd`扩展包

`xlrd`顾名思义，就是`excel`文件的后缀名`.xl`文件`read`的扩展包。这个包只能读取文件，不能写入。写入需要使用另外一个包。但是这个包，其实也能读取`.xlsx`文件。

从`excel`中读取数据的过程比较简单，首先从`xlrd`包导入`open_workbook`，然后打开`excel`文件，把每个`sheet`里的每一行每一列数据都读取出来即可。很明显，这是个循环过程。

```python
from xlrd import open_workbook
x_data1=[]
y_data1=[]
wb = open_workbook('phase_detector.xlsx')
for s in wb.sheets():
    print 'Sheet:',s.name
    for row in range(s.nrows):
        print 'the row is:',row
        values = []
        for col in range(s.ncols):
            values.append(s.cell(row,col).value)
        print values
        x_data1.append(values[0])
        y_data1.append(values[1])
```

如果安装包没有问题，这段代码应该能打印出`excel`表中的数据内容。解释一下 这段代码：打开一个`excel`文件后，首先对文件内的`sheet`进行循环，这是最外层循环；在每个`sheet`内，进行第二次循环，行循环；在每行内，进行列循环，这是第三层循环。在最内层列循环内，取出行列值，复制到新建的`values`列表内，很明显，源数据有几列，`values`列表就有几个元素。我们例子中的`excel`文件有两列，分别对应“角度”和`DC`值。所以在列循环结束后，我们将取得的数据保存到`x_data1`和`y_data1`这两个列表中。

### 3.3 绘制图像V1.0

第一个版本的功能很简单，从`excel`中读取数据，然后绘制成图像。具体程序如下：

```python
#!/usr/bin/python
#-*- coding: utf-8 -*-

import matplotlib.pyplot as plt
import xlrd
from xlrd import open_workbook

x_data=[]
y_data=[]
x_volte=[]
temp=[]
wb = open_workbook('my_data.xlsx')

for s in wb.sheets():
    print 'Sheet:',s.name
    for row in range(s.nrows):
        print 'the row is:',row
        values = []
        for col in range(s.ncols):
            values.append(s.cell(row,col).value)
        print values
        x_data.append(values[0])
        y_data.append(values[1])    
plt.plot(x_data, y_data, 'bo-',label=u"Phase curve",linewidth=1)
plt.title(u"TR14 phase detector")
plt.legend()

plt.xlabel(u"input-deg")
plt.ylabel(u"output-V")


plt.show()
print 'over!'
```

程序简单，显示的效果也是丑到哭：

![此处输入图片的描述](D:\Blog\LyricYang.github.io\img\Python\0.5123937605502709.png)

从`excel`中读取数据的程序，上面已经解释过了。这段代码后面的函数是`matplotlib`绘图的基本格式，此处的输入格式为：

```python
plt.plot(x轴数据, y轴数据, 曲线类型,图例说明,曲线线宽)
```

图片顶部的名称，由这行语句定义：

```python
plt.title(u"TR14 phase detector")
```

最后，使用这一语句使能显示：

```python
plt.legend()
```

### 3.4 绘制图像V1.1

这个图只绘制了一个表格的数据，我们一共有三个表格。但是就这个一个已经够丑了。我们先来美化一下。首先，坐标轴的问题：横轴的0点对应着纵轴的8，这个明显不行。我们来移动一下坐标轴，使之0点重合：

```python
#!/usr/bin/python
#-*- coding: utf-8 -*-

import matplotlib.pyplot as plt
from pylab import *
import xlrd
from xlrd import open_workbook

x_data=[]
y_data=[]
x_volte=[]
temp=[]
wb = open_workbook('my_data.xlsx')

for s in wb.sheets():
    print 'Sheet:',s.name
    for row in range(s.nrows):
        print 'the row is:',row
        values = []
        for col in range(s.ncols):
            values.append(s.cell(row,col).value)
        print values
        x_data.append(values[0])
        y_data.append(values[1])

plt.plot(x_data, y_data, 'bo-',label=u"Phase curve",linewidth=1)


plt.title(u"TR14 phase detector")
plt.legend()

ax = gca()
ax.spines['right'].set_color('none')
ax.spines['top'].set_color('none')
ax.xaxis.set_ticks_position('bottom')
ax.spines['bottom'].set_position(('data',0))
ax.yaxis.set_ticks_position('left')
ax.spines['left'].set_position(('data',0))

plt.xlabel(u"input-deg")
plt.ylabel(u"output-V")

plt.show()
print 'over!'
```

好的，移动坐标轴后，图片稍微顺眼了一点，我们也能明显的看出来，图像与横轴的交点大约在`180`度附近：

![此处输入图片的描述](D:\Blog\LyricYang.github.io\img\Python\0.055218130905020146.png)

解释一下移动坐标轴的代码：我们要移动坐标轴，首先要把旧的坐标拆了。怎么拆呢？原图是上下左右四面都有边界刻度的图像，我们首先把右边界拆了不要了，使用语句：

```python
ax.spines['right'].set_color('none')
```

把右边界的颜色设置为不可见，右边界就拆掉了。同理，再把上边界拆掉：

```python
ax.spines['top'].set_color('none')
```

拆完之后，就只剩下我们关心的左边界和下边界了，这俩就是`x`轴和`y`轴。然后我们移动这两个轴，使他们的零点对应起来：

```python
ax.xaxis.set_ticks_position('bottom')
ax.spines['bottom'].set_position(('data',0))
ax.yaxis.set_ticks_position('left')
ax.spines['left'].set_position(('data',0))
```

这样，就完成了坐标轴的移动。

### 3.5 绘制图像V1.2

我们能不能给图像过零点加个标记呢？显示的告诉看图者，过零点在哪，就免去看完图还得猜，要么就要问作报告的人。

```python
#!/usr/bin/python
#-*- coding: utf-8 -*-

import matplotlib.pyplot as plt
from pylab import *

import xlrd

from xlrd import open_workbook

x_data=[]
y_data=[]
x_volte=[]
temp=[]
wb = open_workbook('my_data.xlsx')

for s in wb.sheets():
    print 'Sheet:',s.name
    for row in range(s.nrows):
        print 'the row is:',row
        values = []
        for col in range(s.ncols):
            values.append(s.cell(row,col).value)
        print values
        x_data.append(values[0])
        y_data.append(values[1])

plt.plot(x_data, y_data, 'bo-',label=u"Phase curve",linewidth=1)

plt.annotate('zero point', xy=(180,0), xytext=(60,3), arrowprops=dict(facecolor='black', shrink=0.05),)

plt.title(u"TR14 phase detector")
plt.legend()

ax = gca()
ax.spines['right'].set_color('none')
ax.spines['top'].set_color('none')
ax.xaxis.set_ticks_position('bottom')
ax.spines['bottom'].set_position(('data',0))
ax.yaxis.set_ticks_position('left')
ax.spines['left'].set_position(('data',0))

plt.xlabel(u"input-deg")
plt.ylabel(u"output-V")

plt.show()
print 'over!'
```

好的，加上标注的图片，显示效果是这样的：

![此处输入图片的描述](D:\Blog\LyricYang.github.io\img\Python\0.9844868559737445.png)

标注的添加，使用这句语句：

```python
plt.annotate(标注文字, 标注的数据点, 标注文字坐标, 箭头形状)
```

这其中，标注的数据点是我们感兴趣的，需要说明的数据，而标注文字坐标，需要我们根据效果进行调节，既不能遮挡原曲线，又要醒目。

### 3.6 绘制图像V1.3

我们把三组数据都画在这幅图上，方便对比，此外，再加上一组理想数据进行对照。这一次我们再做些改进，把横坐标的单位用`Latex`引擎显示；不光标记零点，把两边的非线性区也标记出来；

```python
#!/usr/bin/python
#-*- coding: utf-8 -*-
import numpy as np
import matplotlib.pyplot as plt
from xlrd import open_workbook
from pylab import *

x_data=[]
y_data=[]
x_data1=[]
y_data1=[]
x_data2=[]
y_data2=[]
x_data3=[]
y_data3=[]
x_volte=[]
temp=[]


plt.annotate('Close loop point',size=18, xy=(180, 0.1), xycoords='data',
                xytext=(-100, 40), textcoords='offset points',
                arrowprops=dict(arrowstyle="->",connectionstyle="arc3,rad=.2")
                )
plt.annotate(' ', xy=(0, -0.1), xycoords='data',
                xytext=(200, -90), textcoords='offset points',
                arrowprops=dict(arrowstyle="->",connectionstyle="arc3,rad=-.2")
                )
plt.annotate('Zero point in non-monotonic region', size=18,xy=(360, 0), xycoords='data',
                xytext=(-290, -110), textcoords='offset points',
                arrowprops=dict(arrowstyle="->",connectionstyle="arc3,rad=.2")
                )



wb = open_workbook('phase_detector.xlsx')
for s in wb.sheets():
    print 'Sheet:',s.name
    for row in range(s.nrows):
        print 'the row is:',row
        values = []
        for col in range(s.ncols):
            values.append(s.cell(row,col).value)
        print values
        x_data1.append(values[0])
        y_data1.append(values[1])
plt.plot(x_data1, y_data1, 'g',label=u"Original",linewidth=2)        

wb = open_workbook('phase_detector2.xlsx')
for s in wb.sheets():
    print 'Sheet:',s.name
    for row in range(s.nrows):
        print 'the row is:',row
        values = []
        for col in range(s.ncols):
            values.append(s.cell(row,col).value)
        print values
        x_data2.append(values[0])
        y_data2.append(values[1])
plt.plot(x_data2, y_data2, 'r',label=u"Move the pullup resistor",linewidth=2)




wb = open_workbook('my_data.xlsx')
for s in wb.sheets():
    print 'Sheet:',s.name
    for row in range(s.nrows):
        print 'the row is:',row
        values = []
        for col in range(s.ncols):
            values.append(s.cell(row,col).value)
        print values
        x_data.append(values[0])
        y_data.append(values[1])
plt.plot(x_data, y_data, 'b',label=u"Faster D latch and XOR",linewidth=2)

for i in range(360):
    x_data3.append(i)
    y_data3.append((i-180)*0.052-0.092)
plt.plot(x_data3, y_data3, 'c',label=u"The Ideal Curve",linewidth=2)

#plt.title(u"2 \pi phase detector", fontproperties=font)
plt.title(u"$2\pi$ phase detector",size=20)
plt.legend(loc=0)#显示label
#移动坐标轴代码
ax = gca()
ax.spines['right'].set_color('none')
ax.spines['top'].set_color('none')
ax.xaxis.set_ticks_position('bottom')
ax.spines['bottom'].set_position(('data',0))
ax.yaxis.set_ticks_position('left')
ax.spines['left'].set_position(('data',0))



plt.xlabel(u"$\phi/deg$",size=20)
plt.ylabel(u"$DC/V$",size=20)

plt.show()
print 'over!'
```

看一下显示的效果：

![此处输入图片的描述](D:\Blog\LyricYang.github.io\img\Python\0.6808566766672772.png)

`Latex`表示数学公式，使用`$$`表示两个符号之间的内容是数学符号。圆周率就可以简单表示为`$\pi$`，简单到哭，显示效果却很好看。同样的，`$\phi$`表示角度符号，书写和读音相近，很好记。

对于圆周率，角度公式这类数学符号，使用`Latex`来表示，是非常方便的。这张图比起上面的要好看得多了。但是，依然觉得还是有些丑。好像用平滑线画出来的图像，并不如用点线画出来的好看。而且点线更能反映实际的数据点。此外，我们的图像跟坐标轴重叠的地方，把坐标和数字都挡住了，看着不太美。

图中的理想曲线的数据，是根据电路原理纯计算出来的，要讲清楚需要较大篇幅，这里就不展开了，只是为了配合比较而用，这部分代码，大家知道即可：

```python
for i in range(360):
    x_data3.append(i)
    y_data3.append((i-180)*0.052-0.092)
plt.plot(x_data3, y_data3, 'c',label=u"The Ideal Curve",linewidth=2)
```

### 3.7 绘制图像V1.4

我们再就上述问题，进行优化。优化的过程包括：改变横坐标的显示，使用弧度显示；优化图像与横坐标相交的部分，透明显示；增加网络标度。`Talk is cheap, show me the code`：

```python
#!/usr/bin/python
#-*- coding: utf-8 -*-
import numpy as np
import matplotlib.pyplot as plt
from xlrd import open_workbook
from pylab import *

x_data=[]
y_data=[]
x_data1=[]
y_data1=[]
x_data2=[]
y_data2=[]
x_data3=[]
y_data3=[]
x_volte=[]
temp=[]



plt.annotate('The favorite close loop point',size=16, xy=(1, 0.1), xycoords='data',
                xytext=(-180, 40), textcoords='offset points',
                arrowprops=dict(arrowstyle="->",connectionstyle="arc3,rad=.2")
                )
plt.annotate(' ', xy=(0.02, -0.2), xycoords='data',
                xytext=(200, -90), textcoords='offset points',
                arrowprops=dict(arrowstyle="->",connectionstyle="arc3,rad=-.2")
                )
plt.annotate('Zero point in non-monotonic region', size=16,xy=(1.97, -0.3), xycoords='data',
                xytext=(-290, -110), textcoords='offset points',
                arrowprops=dict(arrowstyle="->",connectionstyle="arc3,rad=.2")
                )



wb = open_workbook('phase_detector.xlsx')
for s in wb.sheets():
    print 'Sheet:',s.name
    for row in range(s.nrows):
        print 'the row is:',row
        values = []
        for col in range(s.ncols):
            values.append(s.cell(row,col).value)
        print values
        #x_data1.append(values[0])
        x_data1.append(values[0]/180.0)
        y_data1.append(values[1])
plt.plot(x_data1, y_data1, 'g--',label=u"Original",linewidth=2)        

wb = open_workbook('phase_detector2.xlsx')
for s in wb.sheets():
    print 'Sheet:',s.name
    for row in range(s.nrows):
        print 'the row is:',row
        values = []
        for col in range(s.ncols):
            values.append(s.cell(row,col).value)
        print values
        #x_data2.append(values[0])
        x_data2.append(values[0]/180.0)
        y_data2.append(values[1])
plt.plot(x_data2, y_data2, 'r-.',label=u"Move the pullup resistor",linewidth=2)




wb = open_workbook('my_data.xlsx')
for s in wb.sheets():
    print 'Sheet:',s.name
    for row in range(s.nrows):
        print 'the row is:',row
        values = []
        for col in range(s.ncols):
            values.append(s.cell(row,col).value)
        print values
        #x_data.append(values[0])
        x_data.append(values[0]/180.0)
        y_data.append(values[1])
plt.plot(x_data, y_data, 'bo--',label=u"Faster D latch and XOR",linewidth=2)

for i in range(360):
    #x_data3.append(i)
    x_data3.append(i/180.0)
    y_data3.append((i-180)*0.052-0.092)
plt.plot(x_data3, y_data3, 'c',label=u"The Ideal Curve",linewidth=2)


plt.title(u"$2\pi$ phase detector",size=20)
plt.legend(loc=0)#显示label
#移动坐标轴代码
ax = gca()
ax.spines['right'].set_color('none')
ax.spines['top'].set_color('none')
ax.xaxis.set_ticks_position('bottom')
ax.spines['bottom'].set_position(('data',0))
ax.yaxis.set_ticks_position('left')
ax.spines['left'].set_position(('data',0))



plt.xlabel(u"$\phi/rad$",size=20)#角度单位为pi
plt.ylabel(u"$DC/V$",size=20)

plt.xticks([0, 0.5, 1, 1.5, 2],[r'$0$', r'$\pi/2$', r'$\pi$', r'$1.5\pi$', r'$2\pi$'],size=16)

for label in ax.get_xticklabels() + ax.get_yticklabels():
    #label.set_fontsize(16)
    label.set_bbox(dict(facecolor='white', edgecolor='None', alpha=0.65 ))

plt.grid(True)

plt.show()
print 'over!'
```

最终，这张图像的显示效果如下：

![此处输入图片的描述](D:\Blog\LyricYang.github.io\img\Python\0.004688679622091962.png)

与我们最开始那张图比起来，是不是有种脱胎换骨的感觉？这其中，对图像与坐标轴相交的部分，做了透明化处理，代码为：

```python
for label in ax.get_xticklabels() + ax.get_yticklabels():
    #label.set_fontsize(16)
    label.set_bbox(dict(facecolor='white', edgecolor='None', alpha=0.65 ))
```

透明度由其中的参数`alpha=0.65`控制，如果想更透明，就把这个数改到更小，`0`代表完全透明，`1`代表不透明。

而改变横轴坐标显示方式的代码为：

```python
plt.xticks([0, 0.5, 1, 1.5, 2],[r'$0$', r'$\pi/2$', r'$\pi$', r'$1.5\pi$', r'$2\pi$'],size=16)
```

这里直接手动指定`x`轴的标度。依然是使用`Latex`引擎来表示数学公式。

## 四、实验总结

这节课使用`python`的绘图包`matplotlib`绘制了一副图像。图像的数据来源于`excel`数据表。与使用数据表画图相比，通过程序控制绘图，得到了更加灵活和精细的控制，最终绘制除了一幅精美的图像。

## 五、课后习题

对比每个版本程序的不同，找出优化部分的程序；修改数据来源，从`txt`或者`word`中读取数据，绘制图像；将以前使用`excel`绘制的图像，改用`python`重新绘制一遍。

来源： https://www.shiyanlou.com/courses/running