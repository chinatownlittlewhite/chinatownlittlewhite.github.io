---
redirect_from: /_posts/2023-3-31-TDM-GCC MINGW配置与优化算法测试函数使用/
title: TDM-GCC MINGW配置与优化算法测试函数使用
tag:
  -插件配置
  
---
# 安装TDM-GCC MINGW 
点击[这里](https://sourceforge.net/projects/dcplusplus/files/Dev/MinGW/GCC%206.3.0%20rev2%20%28used%20for%20DC%2B%2B%200.866-0.868%29/x86_64-6.3.0-release-win32-seh-rt_v5-rev2.7z/download?use_mirror=phoenixnap&download=&failedmirror=jaist.dl.sourceforge.net)即可下载TDM-GCC MINGW,注意跳转到页面后等待几秒会自动开始下载，如果要配置到matlab下载的版本需要6.3.0版本，这个链接进入之后下载的就是6.3.0版本。 
下面是下载页面 
![](https://chinatownlittlewhite.github.io/images/2023-3-31-1.png) 
在下载完成之后，使用压缩软件将文件压缩至路径内，建议安装在D盘里，记录下下载的路径，将在配置环境时使用。 
# 配置环境变量 
打开电脑的高级系统设置并点击环境变量。 
![](https://chinatownlittlewhite.github.io/images/2023-3-31-2.png) 
点开下方系统变量的新建按钮创建一个环境变量

变量名设置为：MW_MINGW64_LOC

变量的值是压缩的文件的路径，将路径直接复制添加到值当中

![](https://chinatownlittlewhite.github.io/images/2023-3-31-3.png) 

点击path变量进行添加，目的是为了方便查询下载的版本是否符合要求 
![](https://chinatownlittlewhite.github.io/images/2023-3-31-4.png) 
添加内容为%MW_MINGW64_LOC%\bin 
![](https://chinatownlittlewhite.github.io/images/2023-3-31-5.png) 

配置完成后点击确定，然后进入cmd窗口（win+r键后在弹出的窗口输入cmd然后回车）

在cmp窗口输入文件安装位置所在的盘，比如D盘则输入：D: 然后回车如果在C盘则不用这一步操作

输入gcc -v 然后回车可以看到如下界面 
![](https://chinatownlittlewhite.github.io/images/2023-3-31-6.png) 
可以看到我们下载的版本是6.3.0

接下来进入matlab的命令行窗口进行操作

输入如下命令

setenv('MW_MINGW64_LOC','这里填所下载的文件的路径')

mex -setup C++,成功后可以看到如下显示 
![](https://chinatownlittlewhite.github.io/images/2023-3-31-7.png) 
 这样编译环境就配置成功了 
 # 测试函数的使用 
 这里以CEC2013的测试函数集来举例

首先在matlab界面打开文件夹并在命令行窗口输入mex cec13_func.cpp 注：cec13.func.cpp是所需编译的c++文件名 
![](https://chinatownlittlewhite.github.io/images/2023-3-31-8.png) 
这样我们就可以对这个测试集当中的函数进行调用了

对于举例的这个测试函数集调用方式是f = cec13_func(x,func_num);

输入的参数x是一个D*pop维的矩阵，即初始种群，注：这里求适应度的是对列向量操作，必要时可以将输入的x矩阵进行转置后输入即f = cec13_func(x’,func_num);

注：这里求得的适应度是一次性求一整个种群的所有个体，并非一个一个求。

还有另一种测试函数集的形式 
![](https://chinatownlittlewhite.github.io/images/2023-3-31-9.png) 
 这个就可以对函数句柄进行直接调用了，例如这个的调格式就是f=benchmark_func(x,func_num)

注：这个测试集是在github上下载的，但是我在实际使用时发现代码无法直接使用，进行了一些修改才能正常使用。
