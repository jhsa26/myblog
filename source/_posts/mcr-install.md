---
title: 利用MCR脱离了matlab环境
date: 2017-07-30 08:35:57
tags: MCR
categories: Matlab
---

## **MCR在Ubuntu16.04.2-LTS安装**
假设你有一大串matlab代码，想把这些代码移植到某个软件上，将matlab改写成C/C\++无疑是最好的，但是很需要时间，比较简单的办法将m文件编译成动态库供C/C\++调用，但这需要在本地安装MCR。这样就可以避免安装matlab了。

我的脑上本身就安装了matlab，MCR压缩包位于:

`/usr/local/R2012a/toolbox/compiler/deploy/glnxa64/MCRInstaller.zip`，
```bash
sudo  mkdir  /usr/local/R2012a/MATLAB_Compiler_Runtime
sudo cp  /usr/local/R2012a/toolbox/compiler/deploy/glnxa64/MCRInstaller.zip   /usr/local/R2012a/MATLAB_Compiler_Runtime
sudo chmod 777 /usr/local/R2012a/MATLAB_Compiler_Runtime
unzip MCRInstaller.zip
./install
```
将下面提示的动态路径添加到 ~/.bashrc 中

![matlab](/images/img/1.png)
``` bash
echo "export  LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:/usr/local/MATLAB/R2012a/MATLAB_Compiler_Runtime/v717/runtime/glnxa64:/usr/local/MATLAB/R2012a/MATLAB_Compiler_Runtime/v717/bin/glnxa64:/usr/local/MATLAB/R2012a/MATLAB_Compiler_Runtime/v717/sys/os/glnxa64:/usr/local/MATLAB/R2012a/MATLAB_Compiler_Runtime/v717/sys/java/jre/glnxa64/jre/lib/amd64/native_threads:/usr/local/MATLAB/R2012a/MATLAB_Compiler_Runtime/v717/sys/java/jre/glnxa64/jre/lib/amd64/server:/usr/local/MATLAB/R2012a/MATLAB_Compiler_Runtime/v717/sys/java/jre/glnxa64/jre/lib/amd64" >>  ~/.bashrc
source  ~/.bashrc
```
- - -
## **将.m文件编译成函数库（cpp）**

我有一个Cal_moment_tensor.m的matlab文件，用matlab打开，并在matlab终端运行这个命令

```
mcc -W cpplib:libCal_moment_tensor -T link:lib Cal_moment_tensor
```
通过以上命令产生了如下5个文件

![a](/images/img/2.png)

>将`main.cpp`与以上5个文件放在一块，然后用以下命令编译得到可执行的`main`

```
g++ -o main -I. -I/usr/local/MATLAB/R2012a/MATLAB_Compiler_Runtime/v717/extern/include -L. -L/usr/local/MATLAB/R2012a/MATLAB_Compiler_Runtime/v717/runtime/glnxa64 -L/usr/local/MATLAB/R2012a/MATLAB_Compiler_Runtime/v717/bin/glnxa64 main.cpp -lCal_moment_tensor -lm -lmwmclmcrrt -lmwmclmcr
```
``` cpp
#include "mclmcrrt.h"
#include "mclmcr.h"
#include "mclcppclass.h"
#include "matrix.h"
#include "libCal_moment_tensor.h"
int main(void)
{
// initialize lib
if( !libCal_moment_tensorInitialize())
{
std::cout << "Could not initialize libMyAdd!" << std::endl;
return -1;
}
// using my function
Cal_moment_tensor();

// terminate the lib
libCal_moment_tensorTerminate();
// terminate MCR
mclTerminateApplication();
return 0;
}
```

其实对于参考[2]中C\++参数说明很清楚，但对于C\++很弱的人学习这个接口成本略高，最简单大办法把所有参数传递都用文件来实现。
以上例子请点击百度云[链接](https://pan.baidu.com/s/1slG0fwh).**密码h3u9**,自行下载。

**另外注明:**

>1. Ubuntu16.04的GCC 版本是 5.4.0的，因为在matlab上编译动态库的时候采用本地环境的GCC编译器，**GCC版本很重要**。

>2. 如果本地采用GCC 5.4.0编译的动态库，放到另外一台电脑，其GCC 为更低版本的如RedHat5.4系统自带的GCC 4.1.2 就会出错。

>3. 但低版本的GCC 编译出来的动态库可以放到高版本的GCC环境中用。
>
>4. 千万别去尝试在Ubuntu 16.04中自己编译安装GCC 4.1.2，会有很多问题的。

## 报错集锦

```
报错1:移植到红帽子5.4,GCC4.1.2上，原因是libmwmclmcr.so依赖的libstdc++.so.6并不是用MCR编译环境中的，而是先用系统中的库。 
```
![c](/images/img/cmp1.png)
```
报错2： 降低matlab版本，期待与GCC版本耦合，其实不然，2010版本支持GCC4.2, 见matlab安装目录下的sys/os/glnxa64/Readmelibstdc++，里面写了GCC版本;降低matlab版本; 这里出现的SSL，经过大神提醒，是少链接了一个ssl,在链接库时，加上一个 -lssl
```
![d](/images/img/cmp4.png)

```
报错4：消除了 -lssl导致的错误，然后是libmwmclmcr.so这个库一直不听话，报错，尽管是用的matlab2009,红帽子平台。最后放弃治疗。
```
![e](/images/img/cmp5.png)

## 总结
博主研究发现，其实是可以在不修改GCC版本的情况下，使得该编译在较低版本的GCC环境中编译成功。在这里我用到了以下几个查看库的命令

```
ldd  libmwmclmcr.so 					#查看libmwmclmcr.so所依赖哪些库
 
strings libstdc++.so.6 | grep "GLIB"    #查看C++标准库支持的版本，对于GCC4.1.2,它自带的libstdc++.so是不支持>=GLIB3.4.9。

nm -D libmwmclmcr.so					#列出.o .a .so中的符号信息，包括诸如符号的值，符号类型及符号名称等。所谓符号，通常指定义出的函数，全局变量。
```
![f](/images/img/cmp2.png)

下图是在红帽子的 `~/.bashrc`的配置以及`编译命令`

![g](/images/img/cmp3.png)
有了以上命令，我们可以大概确定哪些库没有链接上，这就很容易改正错误。我们把MCR装上，不就是因为担心其他平台上没有程序运行的环境吗？根据这个逻辑，我就没有折腾如何去安装这些库，直接在MCR中把这些库重名为系统库对应的名字，以及在~/.bashrc中的LD_LIBRARY_PATH中增加这些动态库的路径。最后运行成功！一开始若能定位到第一步出错在哪，就不会考虑替换GCC版本，替换matlab版本等思路。不然安装MCR又有何用？


**碰到这么多问题，总结一句吃了没文化的亏。:),不做怎么知道行与行不通，并没什么是一蹴而就。撸起袖子干！还有就是不让哥远程操作对方的系统，只能用虚拟机弄个相应的系统来做尝试**

- - - -
## **参考**
[1] http://blog.csdn.net/arackethis/article/details/43375943
[2] http://developer.51cto.com/art/200909/150944.htm
[3] http://www.cnblogs.com/itech/archive/2012/09/16/2687423.html
