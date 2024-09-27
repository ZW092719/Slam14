# 视觉slam十四讲

## Linux配置

安装的东西：

安装前先：

```
sudo apt update
```

g++

cmake：

[ubuntu安装cmake的三种方法（超方便！）-CSDN博客](https://blog.csdn.net/Man_1man/article/details/126467371)

git

vscode

ROS

htop

vim

terminator

mlocate







## 第一讲

### 引言

SLAM：Simultaneous Localization and Mapping（同时定位与地图构建）

搭载特定传感器的主体，在没有环境先验信息的情况下，于运动中建立环境的模型，同时估计自己的运动

**视觉slam**：以相机为主要传感器的SLAM，（还有激光雷达SLAM，区分他们主要根据传感器的不同）

**问题：**从图像中估计相机的**运动**以及**环境**的情况

**应用：**无人机、机器人、无人驾驶、AR/VR

**困难之处：**

1.三维空间的运动

2.受噪声影响

3.数据来源只有图像

4.人类看到的是图像，计算机得到的是数值矩阵

灰度？

5.how to make computer see

学习角度：

1.牵涉的理论广泛

机器人三维运动、计算机视觉、状态估计理论（通过有噪声的数据估计内在的变量）

2.从理论到实现的困难

3.资料缺乏

**相关书籍：**

![image-20240827173038516](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240827173038516.png)

第一本：内容丰富、推导严谨、没有专门介绍slam、缺少实践指导

第二本：机器人学的状态估计 

第三本：Probabilistic Robotics，二维激光方面

以上的书缺少实践的指导，缺少代码的编写



本书特点基础、必要的内容、大量的编程，重视工程实践！

整体的目录：

![image-20240827193009521](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240827193009521.png)

预备知识：

![image-20240827193633368](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240827193633368.png)

## 第二讲

### 引例

初识slam，

小萝卜机器人，可自主移动，（轮子，电机，通讯，主要传感器相机、相机就好比机器人的眼睛有了眼睛就有了一切活动）

**两大问题**：

1.我在什么地方-----定位

2.周围什么样------建图

两者相辅相成，**相互耦合**

**两类传感器：**

1.安装于环境中的：

二维码marker、gps、导轨磁条

2.携带于机器人本体上的

LMU、激光、相机

**基于传感器有两类slam：**

激光slam与视觉（相机）slam

激光研究目前较为透彻、视觉在准确精度还有进步空间

视觉slam的缺点：光要强且要有识别度不能全白或全黑的环境、可能受人干扰比如有人挡住镜头、处理视频流计算量大

目前的研究方向：让其受限条件少一些！

#### 关于相机

相机的本质**以二维投影的形式**记录了三维世界、但是丢掉了一个维度：距离

**相机的分类：**

1.单目 Monocular（相较于下面更难些）：**没有深度**，必须通过移动相机产生深度

例如在拍照过程中，用手去托起周围的建筑或者景观就是这个原因，必须在**移动相机**后才能得知场景的结构

2.双目 Stereo：通过视差计算深度Stereo

3.深度 RGBD：通过物理方法测量深度

4.其他：鱼眼 全景

 当相机运动起来时：

1.场景和成像有几何关系

2.近处的物体的像运动的快

3.远处的物体的像运动的慢

由此可以推断距离

**双目相机**：

基于基线的测距

左右眼的微小差异判断远近，同样远处物体变化小，近处物体变化大--由此推算出距离 计算量非常大

**RGBD相机**：

可以通过物理手段测量出深度、深度值较准确、量程小在室外容易受到太阳光等影响干扰

**三种相机的共同点：**

1）利用图像和场景的几何关系，计算相机运动和场景结构Motion & Structrue

2）三维空间的运动和结构

3）图像来自连续视频

**相机的缺点：**

1）要求光照相对稳定

2）不适用于高速运动

3）无法应对纹理缺失

比如两张白色的墙的照片是无法判断运动信息的

4）单目丢失深度信息

5）双目计算量大

**相机的标定：**

对象：

> 1.内参：
>
> ![image-20240918202546707](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240918202546707.png)
>
> 2.外参：
>
> ![image-20240918202833416](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240918202833416.png)

方法：

> 1.张正友标定法
>
> 2.opencv
>
> 3.kalibr（github）
>
> 4.wiki/csdn



### slam流程

视觉slam框架：

前端：**VO 视觉里程计**

> 视觉里程计：
>
> 相邻图像估计相机运动，建立局部地图
>
> 通过两张图象计算运动和结构
>
> 不可避免的**有漂移**
>
> 方法：
>
> 特征点法，直接法

后端优化：Optimization

> 后端优化的启动是人为设定的不是说当经过回环检测发现回到原来位置才触发，可以人为设定经过一段时间或当插入一个关键帧后触发
>
> 从带有噪声的数据中**优化**轨迹和地图**状态估计问题**
>
> 最大后验概率估计MAP
>
> 前期以EKF（滤波）为代表，现在以图优化为代表、十、十一讲

回环检测：Loop Closing

> 检测机器人是否回到早先位置
>
> 识别到达过的场景
>
> 计算图像间的相似性
>
> 方法：词袋模型
>
> ![image-20240918121012867](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240918121012867.png)

建图：Mapping

> 根据轨迹建立地图
>
> 用于导航、、规划、通讯、可视化、交互等
>
> 度量地图vs拓扑地图
>
> 稀疏地图（Sparse Mapping）vs稠密地图（Dense Mapping）

稀疏建图：是用少量的关键点来描绘整个环境。这些关键点是图像中的显著特征（比如角落或者特定的点）。

稠密建图：是记录整个环境中的**每一个小细节**。它创建的地图非常详细，几乎包含每个像素的信息。（但相应的计算和存储需求也比较高！）

[【即将开源】单目＋深度学习，实现实时稠密建图！CoRL 2021_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1Vh411b723/?spm_id_from=333.337.search-card.all.click&vd_source=32dc776c679a4e7c4f117be057a5d40c)

![image-20240828080315168](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240828080315168.png)

**slam整体效果如下：**

https://www.bilibili.com/video/BV1T64y1D7n2/?spm_id_from=333.337.search-card.all.click



### slam的数学描述

离散时间：t=1，2，...，k，机器人位置：x1、x2、...xk

需要把他们都看作随机变量，服从概率分布

机器人从上一个时刻运动到下一个时刻的**运动方程：**
$$
x_{k}=f(x_{k-1},u_{k},w_{k})
$$
$x_{k-1}$为上一时刻的位置，$u_{k}$为传感器的读数或输入，$w_{k}$为噪声

已知的是：

上一时刻位姿$x_{k-1}$，传感器输入$u_{k}$，和运动方程

需要计算的是：

**当前时刻位姿**$x_{k}$

提供对状态x的**先验**，正向推理

以上就是定位的过程！

------

路标（三维空间点）：$y_{1}$，$y_{2}$,.....$y_{n}$

**观测方程**：

不同的传感器观测方程不同

传感器在位置$x_{k}$探测到了路标$y_{j}$
$$
z_{k,j}=h(x_{k},y_{j},v_{k,j})
$$


$z_{k,j}$为观测数据，$v_{k,j}$为噪声，提供对状态x的后验，由果溯因

已知的是：

当前时刻对路标的观测值$z_{k,j}$，观测方程

需要求的是：

**路标的位置坐标$y_{j}$**

这就是建图的过程！

------

以上为两个基本方程，运动方程与观测方程。需要通过两个方程求解的就是机器人位姿$x_{k}$和路标的坐标$y_{j}$。这两个分别对应着定位和建图

关于机器人位姿描述：

> 在2D中：x，y坐标以及一个偏角θ来描述姿态
>
> 在3D中：就需要x，y，z以及描述姿态的变量----ch3,ch4

关于运动规律：

> 即对应运动方程f()
>
> 刚体运动
>
> 可以做出一些运动假设

关于观测原理：

> 即对应观测方程h()
>
> ---ch5

关于求解方法：

> ch6



### 实践部分

安装Ubuntu

理解一个程序由哪几部分构成

Hello SLAM

```
在ubuntu下打出hello slam
在命令行中编程（看不懂有点难）
```

头文件与库文件

使用ide

#### linux介绍

目录介绍：

> /    : 根目录
>
> /bin    :用户基础二进制文件
>
> /boot    :静态启动文件
>
> /cdrom   :光盘安装点
>
> /dev    :设备文件
>
> /etc   :配置文件
>
> /home    :主目录（用户）
>
> /lib    :存放着系统的最基本的动态连接共享库
>
> /root    :root主目录
>
> /usr    :很重要的目录，很多程序在这

基本命令：

> ls
>
> cd
>
> touch
>
> mkdir
>
> rm
>
> git
>
> dpkg   ：安装二进制文件
>
> ```
> dpkg -i <文件>
> ```
>
> apt   :安装东西
>
> ```
> sudo apt update
> sudo apt install <程序>
> ```

常用的应用：

> htop：  任务管理器类似，可以看到cpu等的活动
>
> vim
>
> terminator： 更方便的终端使用，
>
> git
>
> vscode
>
> ROS：一键安装！  
>
> g++
>
> cmake
>
> ```
> wget http://fishros.com/install -O fishros && . fishros
> ```

#### linux下运行c++

![image-20240919210755417](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240919210755417.png)

使用cmake，比g++更为方便

先建立一个build文件夹，在这里面使用

```
cmake ..   // 对上一级编译
make    // 接着使用make
```

在发布源代码时将build文件删除即可！

运行操作

```
./可执行文件名
```

关于如何调用库文件

```
首先要有库函数的源代码文件.cpp
接着要有其声明方式的文件.h   (.h文件用来调用库函数的文件中的函数)

主函数文件.cpp
在其中#include"库函数.h"
```

> CMakeList.txt中的设置：
>
> ```
> add_library(文件名 库函数.cpp)  // 静态库.a
> add_library(文件名 SHARED 库函数.cpp) // 共享库 .so
> 
> add_executable(主函数文件名 主函数.cpp) // 主函数可执行设置
> target_link_libraries(主函数文件名 库函数文件名) // 链接操作
> ```





## 第三讲：三维空间刚体运动

### 坐标系

**何为坐标系：**

三根不共面的轴

三根坐标轴的单位方向向量就是基，坐标系可以用它的基来表示
$$
[e_1,e_2,e_3]
$$
如下为正交矩阵：

![image-20240919220221614](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240919220221614.png)

> 正交矩阵有如下性质：
>
> A‘A = E
>
> A’ = A逆

**机器人中的坐标系：**

世界系/惯性系：

机体系：

传感器参考系：

不同坐标系之间存在着变换关系：

------

**向量内积的计算：**
$$
a\cdot b=a^\mathrm{T}b=\sum_{i=1}^3a_ib_i=\left|\boldsymbol{a}\right|\left|\boldsymbol{b}\right|\cos\left\langle\boldsymbol{a},\boldsymbol{b}\right\rangle.
$$
内积得到确定的数值

**向量外积的计算：**
$$
\boldsymbol{a}\times\boldsymbol{b}=\begin{Vmatrix}\boldsymbol{e}_1&\boldsymbol{e}_2&\boldsymbol{e}_3\\\\a_1&a_2&a_3\\\\b_1&b_2&b_3\end{Vmatrix}=\begin{bmatrix}a_2b_3-a_3b_2\\\\a_3b_1-a_1b_3\\\\a_1b_2-a_2b_1\end{bmatrix}=\begin{bmatrix}0&-a_3&a_2\\\\a_3&0&-a_1\\\\-a_2&a_1&0\end{bmatrix}\boldsymbol{b}\stackrel{\text{def}}{=}\boldsymbol{a}^{\wedge}\boldsymbol{b}.
$$
得到的计算结果是一个具体的向量，方向垂直于这两个向量，大小为$\left|\boldsymbol{a}\right|\left|\boldsymbol{b}\right|\sin\left\langle\boldsymbol{a},\boldsymbol{b}\right\rangle $。

对于外积运算，引入^符号，其为反对称矩阵（关于主对角线为相反数）**任意一个向量都对应着唯一一个反对称矩阵**

在slam中我们所认为的向量均是**列向量**！

------

**关于机器人传感器得到的坐标系：**

对于视觉slam得到的数据为相机坐标系下的数据，

求解过程如下：pixel-->camero-->world

camero--->world就是slam所需要估计的：机器人分析自己站在图中哪个位置，面向哪个方向才能看到图片中的东西。



### 位姿变换

**旋转变换**：
![image-20240919224141176](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240919224141176.png)

![image-20240919224333983](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240919224333983.png)

> 关于单位正交基，基中的元素均为一个**3x1的向量**，对于空间中的任意一个向量均可以被该坐标系下的单位正交基唯一表示
>
> 线代的内容具体看笔记复习！



接着对公式3.5等式两边同乘$\begin{bmatrix}e_1^\mathrm{T}\\e_2^\mathrm{T}\\e_3^\mathrm{T}\end{bmatrix}$，左边的系数变为了单位矩阵，因此得到：
$$
\begin{bmatrix}a_1\\\\a_2\\\\a_3\end{bmatrix}=\begin{bmatrix}e_1^\mathrm{T}e_1'&e_1^\mathrm{T}e_2'&e_1^\mathrm{T}e_3'\\\\e_2^\mathrm{T}e_1'&e_2^\mathrm{T}e_2'&e_2^\mathrm{T}e_3'\\\\e_3^\mathrm{T}e_1'&e_3^\mathrm{T}e_2'&e_3^\mathrm{T}e_3'\end{bmatrix}\begin{bmatrix}a_1'\\\\a_2'\\\\a_3'\end{bmatrix}\stackrel{\mathrm{def}}{=}Ra'.
$$
将中间的大矩阵拿出来，定义为$R$，这个矩阵由**两组基之间的内积**组成，刻画了旋转前后同一个向量的坐标变换关系。R被称为**旋转矩阵**，大小3x3（具体画出来计算一下就很容易理解是如何变换的了！）

> 关于旋转矩阵的性质:
>
> 1.**行列式为1**的正交矩阵（逆矩阵和自身的转置相同称为正交矩阵）
>
> 正交矩阵相乘还是正交矩阵
>
> 行列式为1的正交矩阵  <----> 旋转矩阵，两者互推

因此可将n维旋转矩阵定义如下：
$$
\mathrm{SO}(n)=\{\boldsymbol{R}\in\mathbb{R}^{n\times n}|\boldsymbol{RR}^\mathrm{T}=\boldsymbol{I},\det(\boldsymbol{R})=1\}.
$$
SO是特殊正交群。SO(3)就指三维空间的旋转。

由：
$$
a=Ra^{\prime}
$$
又可以推出：
$$
a^{\prime}=R^{-1}a=R^{\mathrm{T}}a.
$$
$R^{\mathrm{T}}$刻画了一个相反的旋转

------

**在旋转基础上的平移（欧式变换）：**

世界坐标系中的向量$a$，经过一次旋转和一次平移$t$后，得到了$a'$，将旋转和平移合到一起有：
$$
a^{\prime}=Ra+t                                 (3.9)
$$
![image-20240920110155034](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240920110155034.png)

关于$t_{12}$他表示把位置由**坐标系2对齐到坐标系1**的平移量，也是一个**3x1的列向量**

平移和旋转都是从右向左读!

------

### 变换矩阵与齐次坐标

**变换矩阵的引入：**

假设我们需要多次的变换，比如向量a要经过两次旋转和平移到向量c：
$$
c=R_2\left(R_1a+t_1\right)+t_2.
$$
这样的形式就如同套娃会很麻烦，因此引入齐次坐标和变换矩阵。

重写3.9式子：
$$
\begin{bmatrix}a'\\\\1\end{bmatrix}=\begin{bmatrix}R&t\\\\\mathbf{0}^\mathrm{T}&1\end{bmatrix}\begin{bmatrix}a\\\\1\end{bmatrix}\stackrel{\mathrm{def}}=\boldsymbol{T}\begin{bmatrix}a\\\\1\end{bmatrix}.
$$
由于矩阵相乘要求列数等于行数，因此需要添加一些东西使其满足相乘条件，即在原坐标向量下添加一个1变为4维向量，称为一个**齐次坐标**，矩阵$T$称为**变换矩阵**
$$
T=\begin{bmatrix}R&t\\\\\mathbf{0}^\mathrm{T}&1\end{bmatrix}
$$
是一个4x4的矩阵，上述过程很容易理解画图推算一下就会明白

该矩阵又被称为特殊欧式群：

记作：
$$
\mathrm{SE}(3)=\left\{\boldsymbol{T}=\begin{bmatrix}\boldsymbol{R}&\boldsymbol{t}\\\\\boldsymbol{0}^\mathrm{T}&1\end{bmatrix}\in\mathbb{R}^{4\times4}|\boldsymbol{R}\in\mathrm{SO}(3),\boldsymbol{t}\in\mathbb{R}^3\right\}.
$$
这时的连续变换会非常方便了：
$$
\tilde{\boldsymbol{b}}=\boldsymbol{T}_1\tilde{\boldsymbol{a}}, \tilde{\boldsymbol{c}}=\boldsymbol{T}_2\tilde{\boldsymbol{b}}\quad\Rightarrow\tilde{\boldsymbol{c}}=\boldsymbol{T}_2\boldsymbol{T}_1\tilde{\boldsymbol{a}}.
$$
变换矩阵$T$同样具有反变换，其反变换矩阵为：
$$
T^{-1}=\begin{bmatrix}R^\mathrm{T}&-R^\mathrm{T}t\\\\\mathbf{0}^\mathrm{T}&1\end{bmatrix}.
$$

> 注意事项：
>
> 平移并非是简单的取原平移的相反数，因为是先旋转后平移，所以平移量是受到旋转量影响的，知道就行！

### 实践1：Eigen



### 旋转向量和欧拉角

**关于自由度的知识补充：**

自由刚体的自由度为：$Dof=6$

包括在x，y，z轴上的移动，还有在x，y，z轴上的旋转

#### 旋转向量：

1.变换矩阵用4x4矩阵共16个变量描述了一个6自由度的变换，这显得表达方式是较为冗余的

2.旋转矩阵自身带有约束他必须为正交矩阵且行列式为1，当想要估计或优化一个旋转矩阵或变换矩阵时，这些约束会使得求解困难。

因此需要一种方式更为紧凑的描述旋转和平移。

任意旋转都可由一个**旋转轴和旋转角**来刻画，因此使用一个向量其方向与旋转轴一致，长度等于旋转角，这种向量称为**旋转向量**，因而只需要一个三维向量即可描述旋转。

最终对于**变换矩阵**，我们使用一个旋转向量和一个平移向量即可表达一次变换，这时变量的维数正好是**六维**

**旋转矩阵与旋转向量的关系：**

假设有一个旋转矩阵$R$，旋转轴为一个单位长度的向量$n$，旋转角度为θ，那么向量θn（数乘）也可用来描述旋转，他们关系如下：

*从旋转向量到旋转矩阵*：
$$
\boldsymbol{R}=\cos\theta\boldsymbol{I}+(1-\cos\theta)\boldsymbol{n}\boldsymbol{n}^\mathrm{T}+\sin\theta\boldsymbol{n}^\mathrm{\wedge}.
$$
罗德里格斯公式。^代表反对称矩阵

*从旋转矩阵到旋转向量*：
$$
\theta=\arccos\frac{\mathrm{tr}(\boldsymbol{R})-1}2.
$$
对于转轴n，在几何意义上理解就是旋转轴上的向量经过旋转不发生改变
$$
Rn=n.
$$
即转轴n为矩阵R特征值为1所对的特征向量，再归一化的结果

#### 欧拉角

旋转矩阵和旋转向量虽然能描述旋转，但对人类来说不直观，因此引入欧拉角。

> 旋转矩阵和旋转向量的劣势体现在：
>
> 旋转矩阵不方便找到且具有约束条件，旋转向量要求找到旋转轴这个旋转轴是不好找到的

欧拉角使用了**三个分离的转角**，把一个旋转分解成3次绕不同轴的旋转。例如：先绕X轴，再绕Y轴，最后绕Z轴就得到了一个XYZ轴的旋转

*注意旋转顺序的不同得到的最终结果是不一样的*

欧拉角中常用的一种为：偏航----俯仰----滚转 3个角度来描述一个旋转，其等价于ZYX轴的旋转

1.绕Z轴旋转得到偏航角yaw

2.绕旋转之后的Y轴旋转得到俯仰角pitch

3.绕旋转之后的X轴旋转得到滚转角roll

**ZYX**转角的定义如下：

![image-20240921181500162](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240921181500162.png)

ZYX（**从右向左读**）也被叫做rpy角

**动轴转动：**绕着旋转之后的轴旋转

> 交流好用
>
> 会产生万向锁现象（也叫做奇异性问题）（当俯仰角即绕**Y轴旋转的角度为+-90**°时，第一次的绕Z轴旋转和第三次的绕X轴旋转会使用同一个轴，这使得系统丢失了一个自由度）

**定轴转动：**绕着旋转前的轴来转（即最初始的轴）

> 编程使用
>
> 无万向锁现象

无论哪种旋转都要声明好旋转顺序！
**奇异性问题：**
凡是用**三个变量来表示三维空间的旋转**都是具有奇异性的（或者说不连续性）

旋转向量：存在0到2Π的跳变

欧拉角：

1.存在0到2Π跳变

2.万向锁

3.不适用于增量式的调整（无论动轴还是定轴转动）

> 何为增量式调整：
>
> 比如初始时rpy：10，30，50，我让他经过rpy：1，0，0得到rpy：11，30，50
>
> 这是明显不行的

**用三个变量描述旋转的最大问题：**

> 无论是旋转向量还是欧拉角这些描述形式很难用连续变化的参数描述连续变化的运动，这导致在优化时无法进行求导插值操作



### 四元数

引入：

旋转矩阵用9个量描述3自由度具有冗余性；欧拉角和旋转向量是紧凑的但具有奇异性。事实上找不到不带奇异性的三维向量描述方式。

复数的乘法表示复平面上的旋转，例如：乘上负数i相当于逆时针把一个复向量旋转90°。类似的在表达三维空间时也有一种类似于复数的代数**：四元数。**他是紧凑的且不具有奇异性

先来看在二维平面下：当我们想将复平面的向量旋转θ时，可以给这个复向量乘以$\mathrm{e}^{\mathrm{i}\theta}$，也可以使用欧拉公式写作普通形式：
$$
\mathrm{e}^{\mathrm{i}\theta}=\cos\theta+\mathrm{i}\sin\theta.
$$
在三维旋转下：使用单位四元数来描述
$$
\boldsymbol{q}=q_0+q_1\mathrm{i}+\mathrm{q}_2\mathrm{j}+\mathrm{q}_3\mathrm{k},
$$
也可写作：
$$
\boldsymbol{q}=[s,\boldsymbol{v}]^\mathrm{T},\quad s=q_0\in\mathbb{R},\quad\boldsymbol{v}=[q_1,q_2,q_3]^\mathrm{T}\in\mathbb{R}^3.
$$
分为实数部分，和虚部组合起来的三维向量

四元数$q$具有一个实部和三个虚部，i，j，k为三个虚部看作是三个坐标轴，三个虚部满足如下关系：
$$
\begin{cases}\mathrm{i^2=j^2=k^2=-1}\\\mathrm{ij=k,ji=-k}\\\mathrm{jk=i,kj=-i}\\\mathrm{ki=j,ik=-j}\end{cases}.
$$



#### 四元数的运算

与通常的负数运算一样，可以进行四则运算，共轭，求逆，数乘

1.加减运算：
$$
\boldsymbol{q}_a\pm\boldsymbol{q}_b=[s_a\pm s_b,\boldsymbol{v}_a\pm\boldsymbol{v}_b]^\mathrm{T}.
$$


 2.乘法：
$$
\boldsymbol{q}_a\boldsymbol{q}_b=\begin{bmatrix}s_as_b-\boldsymbol{v}_a^\mathrm{T}\boldsymbol{v}_b,s_a\boldsymbol{v}_b+s_b\boldsymbol{v}_a+\boldsymbol{v}_a\times\boldsymbol{v}_b\end{bmatrix}^\mathrm{T}.
$$
不具有交换律

3.模长：
$$
\|q_{a}\|=\sqrt{s_{a}^{2}+x_{a}^{2}+y_{a}^{2}+z_{a}^{2}}.
$$
两个四元数乘积的模即模的乘积：
$$
\|q_aq_b\|=\|q_a\|\|q_b\|.
$$
4.共轭：
$$
\boldsymbol{q}_a^*=s_a-x_a\mathrm{i}-\mathrm{y}_\mathrm{aj}-\mathrm{z}_\mathrm{a}\mathrm{k}=[\mathrm{s}_\mathrm{a},-\mathrm{v}_\mathrm{a}]^\mathrm{T}.
$$

$$
q^*q=qq^*=[s_a^2+v^\mathrm{T}v,\mathbf{0}]^\mathrm{T}.
$$

5.逆：
$$
q^{-1}=q^*/\|q\|^2.
$$
四元数与自己的逆相乘为实四元数1

若q为单位四元数其逆和共轭就是同一个量。同样：
$$
(\boldsymbol{q}_a\boldsymbol{q}_b)^{-1}=\boldsymbol{q}_b^{-1}\boldsymbol{q}_a^{-1}.
$$
6.数乘
$$
\mathrm{k}\boldsymbol{q}=\left[\mathrm{k}s,\mathrm{k}\boldsymbol{v}\right]^\mathrm{T}.
$$

#### 用四元数表示旋转

假设空间具有一个三位点$\boldsymbol{p} = [x,y,z] \in \mathbb{R}^3$，以及一个由单位四元数$q$指定的旋转，$p$经过旋转后变为$p'$，他们的关系该如何表达：

首先，把三维空间点用一个虚四元数来描述：
$$
\boldsymbol{p}=[0,x,y,z]^\mathrm{T}=[0,\boldsymbol{v}]^\mathrm{T}.
$$
相当于把四元数的三个虚部与空间中的3个轴相对应。那么旋转后的$p'$可表示为：
$$
p'=qpq^{-1}.
$$
最后得到的结果也是四元数，之后再把$p'$的虚部取出即得到旋转后的坐标

#### 四元数与其他旋转的变换

设$\boldsymbol{q}=[s,\boldsymbol{v}]^\mathrm{T}$，那么定义如下的符号：
$$
\boldsymbol{q}^+=\begin{bmatrix}s&-\boldsymbol{v}^\mathrm{T}\\\\\boldsymbol{v}&s\boldsymbol{I}+\boldsymbol{v}^\wedge\end{bmatrix},\quad\boldsymbol{q}^\oplus=\begin{bmatrix}s&-\boldsymbol{v}^\mathrm{T}\\\\\boldsymbol{v}&s\boldsymbol{I}-\boldsymbol{v}^\wedge\end{bmatrix}.
$$
于是四元数乘法可写作矩阵形式：
$$
q_1^+\boldsymbol{q}_2=\begin{bmatrix}s_1&-\boldsymbol{v}_1^\mathrm{T}\\\\\boldsymbol{v}_1&s_1\boldsymbol{I}+\boldsymbol{v}_1^\wedge\end{bmatrix}\begin{bmatrix}s_2\\\\\boldsymbol{v}_2\end{bmatrix}=\begin{bmatrix}-\boldsymbol{v}_1^\mathrm{T}\boldsymbol{v}_2+s_1s_2\\\\s_1\boldsymbol{v}_2+s_2\boldsymbol{v}_1+\boldsymbol{v}_1^\wedge\boldsymbol{v}_2\end{bmatrix}=\boldsymbol{q}_1\boldsymbol{q}_2.
$$
同样可证：
$$
q_1\boldsymbol{q}_2=\boldsymbol{q}_1^+\boldsymbol{q}_2=\boldsymbol{q}_2^\oplus\boldsymbol{q}_1.
$$
因此，
$$
\begin{aligned}\boldsymbol{p}^{\prime}&=\boldsymbol{q}\boldsymbol{p}\boldsymbol{q}^{-1}=\boldsymbol{q}^+\boldsymbol{p}^+\boldsymbol{q}^{-1}\\&=\boldsymbol{q}^+\boldsymbol{q}^{-1^\oplus}\boldsymbol{p}.\end{aligned}
$$
其中：
$$
\boldsymbol{q}^+\big(\boldsymbol{q}^{-1}\big)^\oplus=\begin{bmatrix}s&-\boldsymbol{v}^\mathrm{T}\\\\\boldsymbol{v}&s\boldsymbol{I}+\boldsymbol{v}^\mathrm{\wedge}\end{bmatrix}\begin{bmatrix}s&\boldsymbol{v}^\mathrm{T}\\\\-\boldsymbol{v}&s\boldsymbol{I}+\boldsymbol{v}^\mathrm{\wedge}\end{bmatrix}=\begin{bmatrix}1&\boldsymbol{0}\\\\\boldsymbol{0}^\mathrm{T}&\boldsymbol{v}\boldsymbol{v}^\mathrm{T}+s^2\boldsymbol{I}+2s\boldsymbol{v}^\mathrm{\wedge}+\left(\boldsymbol{v}^\mathrm{\wedge}\right)^2\end{bmatrix}
$$
从而**旋转矩阵与四元数的关系**如下：
$$
R=vv^\mathrm{T}+s^2\boldsymbol{I}+2s\boldsymbol{v}^\wedge+\left(\boldsymbol{v}^\wedge\right)^2.
$$
再经**推导四元数到旋转向量**的公式如下：
![image-20240921202103661](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240921202103661.png)

在编程中程序库会写好各种转换关系！

### 编程中的问题

经过上述学习实际上有四种旋转方式，旋转矩阵、旋转向量、欧拉角、四元数各有优缺点，但是对同一旋转/姿态的不同描述方式是等价的

1.c++的库对上面的所有运算均有对应的实现

2.四元数好用难理解，欧拉角难用好理解

3.旋转的左乘和右乘是有区别的，一定一定使用左乘！

**4.关于位姿和旋转：**

有时候会用以上描述旋转的量来描述一个刚体的姿态，有时又会用他们来描述刚体在一段时间的旋转变化，想要弄清不太容易



### 实践部分：

#### vscode配置

#### **1.基本插件**

c++、cmake、ROS、汉化

快速修复来配置.vcode文件

mlocate包可在终端查找文件位置!

**先sudo安装**

```
locate Core | grep Eigen   // 查找某个文件
```

之后复制地址添加到.json



**嵌入式终端：**

ctrl+esc下面的按键



还有其他的ssh，Debug、Git等等





#### **2.Eigen使用**

具有官方手册

纯头文件，没有.cpp原代码，不需要写target_link_libraries

##### **useEigen代码分析：**

首先看一下他的cmakelist.txt

```
cmake_minimum_required(VERSION 2.8)
project(useEigen)

set(CMAKE_BUILD_TYPE "Release")
set(CMAKE_CXX_FLAGS "-O3")

# 添加Eigen头文件
include_directories("/usr/include/eigen3")
add_executable(eigenMatrix eigenMatrix.cpp)
```

由于eigen是纯头文件所以没有link_libraries



之后Eigen下的各种模块，及其主要功能：

![img](https://i-blog.csdnimg.cn/blog_migrate/012e03c967f05dc0daf199b1ad0fd322.png)

一般情况下：直接include<Eigen/Dense>就足够

```
using namespace Eigen;
```

接下来不用再Eigen::这样子了方便

声明矩阵：

```
Matrix<float, 2, 3> matrix_23;
```

```
Vector3d v_3d;
Vector3d重构了一下,他就相当于Matrix<double,3,1>

Matrix3d matrix_33;
他就相当于Matrix<double,3,3>


MatrixXd // 动态大小矩阵


// 输入数据
矩阵名<<
matrix_23<<1,2,3,4,5,6;
他会先按行录入,行被填满后换下一行
// 打印
cout<<matrix_23<<endl;


// 访问矩阵中元素
使用(i,j),代表i行j列的数据


// 矩阵相乘
直接对两个矩阵使用*即可
但是,Eigen库中不同数据类型的矩阵无法直接相乘,而且需要注意维度问题
否则报错：error: static assertion   failed:YOU_MIXED_MATRICES_OF_DIFFERENT_SIZES


1.矩阵类型转换函数:   矩阵名.cast<要转换的数据类行>()
2.转置函数:    .transpose()
3.Random函数:   MatrixXd::Random(行数,列数) // 一种矩阵类型::Random()
范围从-1到1
4. 矩阵.sum():    所有元素的和
5.矩阵.trace():   矩阵的迹(对角线元素之和)
6.矩阵.inverse():   矩阵的逆
7.矩阵.determinant():  矩阵的行列式的值
```

**求特征值以及特征向量：**

![image-20240922112308722](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240922112308722.png)

```
SelfAdjointEigenSolver<Matrix3d> eigen_solver(matrix_33.transpose() * matrix_33) // 矩阵乘以自己的转置得到一个对称矩阵
cout << "Eigen values = \n" << eigen_solver.eigenvalues() << endl;
cout << "Eigen vectors = \n" << eigen_solver.eigenvectors() << endl;
// SelfAdjointEigenSolver模板计算自伴随矩阵的特征值和特征向量


求解特征值和特征向量均要用到EigenSolver这个模板
```

**(作业)求解方程组：**

```
// 1.方法一:直接求逆矩阵再乘以等式右侧的系数(缺点运行很慢)
// 2.方法二:利用矩阵分解如QR分解
x = matrix_NN.colPivHouseholderQr().solve(v_Nd);





// 完整代码
// 解方程
  // 我们求解 matrix_NN * x = v_Nd 这个方程
  // N的大小在前边的宏里定义，它由随机数生成
  // 直接求逆自然是最直接的，但是求逆运算量大

  Matrix<double, MATRIX_SIZE, MATRIX_SIZE> matrix_NN
      = MatrixXd::Random(MATRIX_SIZE, MATRIX_SIZE);
  matrix_NN = matrix_NN * matrix_NN.transpose();  // 保证半正定
  Matrix<double, MATRIX_SIZE, 1> v_Nd = MatrixXd::Random(MATRIX_SIZE, 1);

  clock_t time_stt = clock(); // 计时
  // 直接求逆
  Matrix<double, MATRIX_SIZE, 1> x = matrix_NN.inverse() * v_Nd;
  cout << "time of normal inverse is "
       << 1000 * (clock() - time_stt) / (double) CLOCKS_PER_SEC << "ms" << endl;
  cout << "x = " << x.transpose() << endl;

  // 通常用矩阵分解来求，例如QR分解，速度会快很多
  time_stt = clock();
  x = matrix_NN.colPivHouseholderQr().solve(v_Nd);
  cout << "time of Qr decomposition is "
       << 1000 * (clock() - time_stt) / (double) CLOCKS_PER_SEC << "ms" << endl;
  cout << "x = " << x.transpose() << endl;
```

其他的一些分解方法求方程组：

| 分解                                                         | 方法                                | 矩阵要求    | 速度 （中小） | 速度 （大） | 准确性 |
| :----------------------------------------------------------- | :---------------------------------- | :---------- | :------------ | :---------- | :----- |
| [PartialPivLU](http://eigen.tuxfamily.org/dox/classEigen_1_1PartialPivLU.html) | partialPivLu（）                    | 可逆的      | ++            | ++          | +      |
| [FullPivLU](http://eigen.tuxfamily.org/dox/classEigen_1_1FullPivLU.html) | fullPivLu（）                       | 没有        | --            | --          | +++    |
| [HouseholderQR](http://eigen.tuxfamily.org/dox/classEigen_1_1HouseholderQR.html) | HouseholderQr（）                   | 没有        | ++            | ++          | +      |
| [ColPivHouseholderQR](http://eigen.tuxfamily.org/dox/classEigen_1_1ColPivHouseholderQR.html) | colPivHouseholderQr（）             | 没有        | +             | --          | +++    |
| [FullPivHouseholderQR](http://eigen.tuxfamily.org/dox/classEigen_1_1FullPivHouseholderQR.html) | fullPivHouseholderQr（）            | 没有        | --            | --          | +++    |
| [CompleteOrthogonalDecomposition](http://eigen.tuxfamily.org/dox/classEigen_1_1CompleteOrthogonalDecomposition.html) | completeOrthogonalDecomposition（） | 没有        | +             | --          | +++    |
| [LLT](http://eigen.tuxfamily.org/dox/classEigen_1_1LLT.html) | llt（）                             | 正定        | +++           | +++         | +      |
| [LDLT](http://eigen.tuxfamily.org/dox/classEigen_1_1LDLT.html) | ldlt（）                            | 正或负 半定 | +++           | +           | ++     |
| [BDCSVD](http://eigen.tuxfamily.org/dox/classEigen_1_1BDCSVD.html) | bdcSvd（）                          | 没有        | --            | --          | +++    |
| [JacobiSVD](http://eigen.tuxfamily.org/dox/classEigen_1_1JacobiSVD.html) | jacobiSvd（）                       | 没有        | --            | ---         | +++    |

##### useGeometry解析

Eigen/Geometry 模块提供了各种旋转和平移的表示

关于使用问题：

[使用Eigen实现四元数、欧拉角、旋转矩阵、旋转向量之间的转换 Eigen::Affine3f和Eigen::Matrix4f的转换 以及float 和 double类型转换_eigen::affine转欧拉角-CSDN博客](https://blog.csdn.net/Enochzhu/article/details/125934638)

[Eigen中旋转矩阵、四元数与变换矩阵的使用 - 北极星！ - 博客园 (cnblogs.com)](https://www.cnblogs.com/zhjblogs/p/16840564.html)

```
旋转矩阵（3X3）:Eigen::Matrix3d
旋转向量（3X1）:Eigen::AngleAxisd
四元数（4X1）:Eigen::Quaterniond
平移向量（3X1）:Eigen::Vector3d
变换矩阵（4X4）:Eigen::Isometry3d
```

首先**旋转向量的使用：**

> 他与真实的旋转向量不太一样，真实的旋转向量是三维的，向量的方向是旋转轴，模长是旋转角度
>
> 而在Eigen库中：是一个四维向量组成的，第一个元素代表**旋转角度**，后面的一个**向量为旋转轴**
>
> ```
> AngleAxisd rotation_vector(M_PI / 4, Vector3d(0, 0, 1));     //沿 Z 轴旋转 45 度
> // 注意Π的使用
> // 还要注意旋转轴一定是一个单位向量，模长为1
> ```
>
> 旋转向量转换为旋转矩阵：
>
> ```
> rotation_vector.matrix()
> // 或者
> rotation_vector.toRotationMatrix()
> ```
>
> 使用旋转向量进行旋转：
>
> 直接左乘就好
>
> ```
> Vector3d v(1, 0, 0);
> Vector3d v_rotated = rotation_vector * v;
> ```
>
> 使用旋转矩阵进行旋转：
>
> ```
> v_rotated = rotation_matrix * v;
> ```

**欧拉角的使用：**

> 旋转矩阵转换为欧拉角：
>
> ```
> // 欧拉角: 可以将旋转矩阵直接转换成欧拉角
> Vector3d euler_angles = rotation_matrix.eulerAngles(2, 1, 0); // ZYX顺序，即yaw-pitch-roll顺序
> ```
>
> 欧拉角转换为旋转矩阵：
>
> 

**变换矩阵：**

> 由旋转外加一个平移组成：
>
> 函数声明：
>
> ```
> // 欧氏变换矩阵使用 Eigen::Isometry
> Isometry3d T = Isometry3d::Identity();                // 虽然称为3d，实质上是4＊4的矩阵
> T.rotate(rotation_vector);                                     // 按照rotation_vector进行旋转
> T.pretranslate(Vector3d(1, 3, 4));                     // 把平移向量设成(1,3,4)
> cout << "Transform matrix = \n" << T.matrix() << endl;
> ```
>
> 一定一定一定注意：在构造变换矩阵时，Isometry3d**一定声明成Identity单位矩阵**，之后再往里添加旋转和平移
>
> Isometry无法直接cout！必须.matrix()才可用cout
>
> （**作业如何取出变换矩阵的旋转（3x3）部分**）：
>
> ```
> // 取出矩阵左上角3x3部分
>     Matrix3d m_33;
>     m_33 =  T2w.matrix().block<3,3>(0, 0);   // block<3,3>代表3x3的区域,(0,0)代表起始位置 向下和向右扩展到3x3区域
>     cout << "取出后的矩阵为：\n" << m_33 << endl;
> ```
>
> 

**四元数的使用：**

使用前一定先**归一化**

```
Q.normalize();
```

归一化的意义：
四元数归一化：对四元数的单位化，单位化的四元数可以表示一个旋转.
规范化四元数作用：
1.表征旋转的四元数应该是规范化的四元数，但是由于计算误差等因素, 计算过程中四元数会逐渐失去规范化特性，因此必须对四元数做规范化处理
2.意义在于单位化四元数在空间旋转时是不会拉伸的,仅有旋转角度.这类似与线性代数里面的正交变换.
3.由于误差的引入，使得计算的变换四元数的模不再等于1，变换四元数失去规范性，因此需要再次更新四元数
————————————————

> eigen中的四元数顺序为：// 请注意coeffs的顺序是(x,y,z,w),w为实部，前三者为虚部
>
> ```
> 使用q.coeffts()即可提取
> ```
>
> 将旋转向量转换为四元数：
>
> ```
> // 可以直接把AngleAxis赋值给四元数，反之亦然
>   Quaterniond q = Quaterniond(rotation_vector);
> ```
>
> 也可将旋转矩阵转换为四元数：
>
> ```
> // 也可以把旋转矩阵赋给它
>   q = Quaterniond(rotation_matrix);
> ```
>
> 四元数进行旋转：
>
> ```
> // 使用四元数旋转一个向量，使用重载的乘法即可
>   v_rotated = q * v; // 注意数学上是qvq^{-1}
> ```
>



**小罗卜的例子：**

```
#include <iostream>
#include <cmath>

using namespace std;
#include <Eigen/Core>
#include <Eigen/Geometry>

using namespace Eigen;

int main (int argc, char** argv) {
    // 萝卜一号、二号的位姿信息
    Quaterniond q1(0.35, 0.2, 0.3, 0.1), q2(-0.5, 0.4, -0.1, 0.2); // 四元数
    // 四元必须归一化
    q1.normalize();
    q2.normalize();
    Vector3d t1(0.3, 0.1, 0.1), t2(-0.1, 0.5, 0.3);

    Vector3d Pr1(0.5, 0, 0.2); // 某个点在萝卜一号下的坐标
    Vector3d Prw = Vector3d :: Ones(); // 该点在世界坐标系下的坐标
    Vector3d Pr2 = Vector3d :: Ones(); // 最终求解结果

    Isometry3d T1w(q1); // 世界坐标系到1号坐标系
    T1w.pretranslate(t1);
    cout << "正确的结果：\n" << T1w.matrix() << endl;

    Isometry3d test = Isometry3d :: Identity(); // 一定声明成单位矩阵
    test.rotate(q1.toRotationMatrix());
    test.pretranslate(t1);
    cout << "错误的结果：\n" << test.matrix() << endl;

    Prw = T1w.inverse() * Pr1;
    cout << "该点在世界坐标系下的位置为：\n" << Prw << endl;

    Isometry3d T2w(q2); // 世界坐标系到二号坐标系
    T2w.pretranslate(t2);

    Pr2 = T2w * Prw;
    cout << "该点在二号坐标系下的位置为：\n" << Pr2 << endl;
    return 0;
}
```

**pangolin绘制相机轨迹：**

<img src="C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240923095924584.png" alt="image-20240923095924584" style="zoom: 67%;" />



#### **3.CMAKE**

具有官方手册

```
cmake_minimum_required(VERSION 2.8)
project(useEigen)

set(CMAKE_BUILD_TYPE "Release")
set(CMAKE_CXX_FLAGS "-O3")

# 添加Eigen头文件
include_directories("/usr/include/eigen3")
add_executable(eigenMatrix eigenMatrix.cpp)
```

大小写问题：变量区分大小写，指令不区分

**set使用：**

https://blog.csdn.net/Calvin_zhou/article/details/104060927

**message:**

```
(无) = 重要消息；
 STATUS = 非重要消息；
 WARNING = CMake 警告, 会继续执行；
 AUTHOR_WARNING = CMake 警告 (dev), 会继续执行；
 SEND_ERROR = CMake 错误, 继续执行，但是会跳过生成的步骤；
 FATAL_ERROR = CMake 错误, 终止所有处理过程；
```

![image-20240922164322726](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240922164322726.png)



**Eigen库添加：**

1.方法一：locate工具寻找具体细节见上面的vscode配置

2.找包的通用解决：

```
# 添加Pangolin依赖
find_package( Pangolin REQUIRED) 
include_directories( ${Pangolin_INCLUDE_DIRS} )

add_executable( visualizeGeometry visualizeGeometry.cpp )
target_link_libraries( visualizeGeometry ${Pangolin_LIBRARIES} )
```

find_package中的REQUIRED意味着如果找不到就直接报错不再往下执行

find_package会返回两个东西:

${某某某_INCLUDE_DIRS}，

${某某某_LIBRARIES}

那么该如何去确定这个“某某某”具体是什么如何拼写！？

下面的命令：

```
locate eigen | grep cmake
```

如下：

![image-20240922165549998](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240922165549998.png)

然后打开Config.cmake文件

![image-20240922170741186](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240922170741186.png)

你可以看到最为关键的信息他们就是要在cmakelists中写的

pangolin如下：

![image-20240922171056173](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240922171056173.png)

### 作业

https://github.com/cckaixin/Practical_Homework_for_slambook14

<img src="C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240923165813232.png" alt="image-20240923165813232" style="zoom: 67%;" />

//有两个右手系1和2,其中2系的x轴与1系的y轴方向相同，2系的y轴与1系z轴方向相反，2系的z轴与1系的x轴相反,两个坐标系原点重合
//求R12，求1系中(1,1,1)在2系中的坐标。请自己编写一个c++程序实现它，并用Cmake编译，得到能输出答案的可执行文件

首先整体代码如下：

```
#include<iostream>
using namespace std;
#include<eigen3/Eigen/Geometry>
#include<eigen3/Eigen/Core>
#include<cmath>
using namespace Eigen;
int main (int argc, char** argv) {
    AngleAxisd s1(-M_PI/2, Vector3d(1, 0, 0)); // 首先沿着X轴
    AngleAxisd s2(-M_PI/2, s1 * Vector3d(0, 1, 0)); // 再沿着y轴
    AngleAxisd s3(0, s2 * s1 * Vector3d(0, 0, 1)); // 再沿着z轴
    Matrix3d R12 = (s3 * s2 * s1).matrix();
    cout << "坐标系2到坐标系1的旋转矩阵为：\n" << R12 << endl; // 坐标系1位姿态到坐标系2位姿的旋转矩阵

    // 1系中（1,1,1）在2系中坐标
    Vector3d p1(1, 1, 1);
    Vector3d p2;
    p2 = R12.inverse() * p1;
    cout << "坐标为：\n" << p2 << endl;
    return 0;
}
```

**几个很重要的注意点！**

**1.针对坐标系的位姿变化与坐标点的变换**

```
// 你可能会疑惑，为什么把1系转到2系得到的旋转量是R12，按照我们说的从右往左读的习惯，我们应该写成R21啊？
// 我们所说的a1=R12a2+t12中，R12和t12确实是从右向左读没错，但这只是坐标变换时的较为合适的读法
// R12实际对应的是坐标系1姿态到坐标系2姿态的旋转，我们之所以说R12角标从右往左读，那是对于坐标变化来说的，对坐标轴的姿态变换要从左往右读才是真实旋转方向，这点很重要！
// 同理，t12实际对应的是坐标系1原点指向坐标系2原点的向量，我们之所以说t12角标从右往左读，那是对于坐标变换来说的，对坐标轴的位置变换要从左往右读才是真实平移向量方向，这点很重要！
```

即可认为R12代表着**坐标系1的位姿**转换为**坐标系2的位姿**的旋转；同时也代表**坐标系2下的具体的某个点**到**坐标系1下某个点的***旋转矩阵*！

> 理解这点很重要，因此在乘的时候要对R12取逆再乘以坐标系1下的一个坐标，这才是坐标系1下的点到坐标系2下的点的转换

从这里可以发现非常有趣的一点：**坐标之间变换与坐标系之间变换似乎是反过来的**。

[坐标变换与坐标系变换 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/661060377)

**2.具体细节**：

首先人们能直观感觉出来的旋转只有使用欧拉角或者旋转向量，因此首先定义角轴来描述这一旋转过程。

注意这一过程为**动轴旋转**，因此AngleAxisd的轴不是固定的而是**上一次旋转的叠加**！

并且AngleAxisd的第一个旋转α，是**绕轴逆时针旋转**，若为-Π/2才是顺时针

接着坐标系1到坐标系2的位姿变化矩阵就是：

```
Matrix3d R12 = (s3 * s2 * s1).matrix();
```

不要受s3，s2，s1的顺序影响，因为实际上先做s1旋转，之后s2，再s3，

**一个疑问？这是rpy角吗？rpy角的顺序都说是ZYX那这不就与上述冲突了吗**

一个推测：按照作者答案的写法应该是**XYZ的旋转顺序**！所以作者出错了

实际上应写作：

```
s3 * (s2 * (s1 * 原始坐标系))  // 这是先XYZ？？？？？？吗
```

这样就好理解了

> 我的理解：
>
> 实际上在计算机中，电脑不鸟你或者不明白不区分什么坐标系之类的，上述求出的R12是坐标系1的位姿到坐标系2的位姿的一个变化，但是计算机可不管那么多他就把他当作一种运算。
>
> 假设坐标系1下有一个点（1，1，1）他做了R12这样的运动，你可以在坐标系1中让他去做这样动轴旋转的运动得到的结果不会是在坐标系2下的坐标（1，-1，-1），而得到的是（-1，-1，1）

**再次强调，再进行坐标系的变换时R12代表的是坐标系1到坐标系2的变换！，然而在进行坐标点的变换时，R12代表的是坐标系2下的点转到坐标系1下！**

**3.关于左乘和右乘**

![image-20240924173352113](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240924173352113.png)

至此差不多就已经明白了，在上述的例子中我们得到了R12，

**倘若是左乘**那么就是对坐标系1中的点（1，1，1）做了R12旋转，坐标系是不变的，得到的结果也还是在坐标系1下的坐标，经验证完全正确的

**如果是右乘**，得到的便是在新坐标系下的坐标！但是要注意一点要对原来的坐标transpose一下变为行向量使其可以相乘！这也就是说

```
R12.inverse() * vector3d(1,1,1)得到的结果与vector3d(1,1,1).transpose() * R12得到的结果是一致的！！！
```

至于左乘与右乘哪个写法更常用有待商榷！

**4.关于位姿**

可拆解为两个一个是位置，另一个是姿态

其中位置：**位置**（Position）：通常用三维坐标表示（例如，(x,y,z)(x, y, z)(x,y,z)），表示物体在世界坐标系中的具体位置。

姿态：**从世界坐标系到物体坐标系**：当我们在SLAM中移动传感器或机器人时，物体坐标系会随着位置和姿态的变化而变化。姿态的变化意味着物体坐标系相对于世界坐标系的旋转，**那么其姿态的矩阵应该就是世界坐标系到物体坐标系的变换矩阵R**

> 比如A相对于B坐标系，那么其姿态应该就是B到A坐标系的变换。
>
> 这个姿态可以用多种方式，比如B到A的变换矩阵，四元数，欧拉角，旋转向量

最后附上gpt的问答，完美解决该问题！

![image-20240924175120005](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240924175120005.png)



最后再来一个网站，可以直观看出各变换方式间的转换：https://danceswithcode.net/engineeringnotes/quaternions/conversion_tool.html

关于方向锁：

[无伤理解欧拉角中的“万向死锁”现象_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1Nr4y1j7kn/?spm_id_from=333.337.search-card.all.click&vd_source=0da0b7e545e1a65e82836ac4eff73077)

另外还有关于矩阵calculus（矩阵求导很重要！）

https://en.wikipedia.org/wiki/Matrix_calculus

## 第四讲 ：李群与李代数

首先一些建议：
1.先把握大的逻辑，不要钻细节

2.如何使用数学工具解决slam问题很重要

3.数学工具相关理论不必深究

### **引入：**
在第三讲中介绍了旋转的各种表示方式，但在slam中除了表示之外我们还要对其进行估计和优化，因为在slam中位姿是未知的，而我们需要**解决什么样的相机位姿最符合当前观测数据这一问题**，一种典型方式就是把他构建为一个优化问题，求解最优的R、t使得误差最小。

这个优化问题即：误差是位姿的一个函数，而slam中**前端Localization**就是找使得误差最小的位姿！

> 可以认为：
>
> error = f(x)
>
> x = [R,t]

计算最小误差也就是要求最低点，该过程的计算方式也就是**梯度下降的算法**

然而倘若我们直接将旋转矩阵作为优化变量会引入额外的约束，在上一讲我们提到过旋转矩阵自身是带有约束的（正交且行列式为1），这使得优化变得困难。

**因此通过李群---李代数间的转换关系，我们希望把位姿估计变成无约束的优化问题，简化求解。**

另一种理解对于位移他是对加法封闭的即可以进行加法，故它可以进行求导优化

然而旋转矩阵和变换矩阵对加法不封闭无法像位移一样求导优化：

> ![image-20240925170715365](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240925170715365.png)

**但是SO(3)与SE(3)所对应的李代数他们是支持加法的因而我们将对李群的导数写成对李代数的导数，之后再转变回去！**

















### 李群与李代数基础

#### 群

在第三讲我们说三维旋转矩阵构成了特殊正交群SO(3)，而变换矩阵构成了特殊欧式群SE(3)，他们分别如下：
$$
\begin{aligned}
&\text{SO(3)} =\{\boldsymbol{R}\in\mathbb{R}^{3\times3}|\boldsymbol{RR}^{\mathrm{T}}=\boldsymbol{I},\det(\boldsymbol{R})=1\}, \\
&\mathrm{SE}(3) =\left\{\boldsymbol{T}=\begin{bmatrix}\boldsymbol{R}&\boldsymbol{t}\\\\\boldsymbol{0}^\mathrm{T}&1\end{bmatrix}\in\mathbb{R}^{4\times4}|\boldsymbol{R}\in\mathrm{SO}(3),\boldsymbol{t}\in\mathbb{R}^3\right\}. 
\end{aligned}
$$
一个很重要的地方，**这两个只对乘法封闭对加分不封闭**，也就是说：

对任意两个旋转矩阵R1,R2他们的和不再是一个旋转矩阵。变换矩阵同样如此
$$
R_1+R_2\notin\mathrm{SO}(3),\quad T_1+T_2\notin\mathrm{SE}(3).
$$
乘法对应着旋转或变换的复合，两个旋转矩阵相乘代表做了两次旋转。对于这种只有一个（良好的）运算的集合我们称之为**群**

------

进一步给出群的概念：

群是**一种集合**加上**一种运算**的代数结构。我们把集合记作A，运算记作.，那么群可记作

$G=(A,\cdot)$。群要求这个运算满足以下条件：

![image-20240924210557830](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240924210557830.png)

那么容易验证旋转矩阵集合和矩阵乘法这个运算构成群，变换矩阵集合和矩阵乘法也构成群。

进一步来看**李群**：

**李群是指具有连续（光滑）性质的群**。像整数群那样离散的群没有连续性质所以不是李群。而SO(N)和SE(n)在实数空间上是连续的。

下面我们将主要讨论对于相机姿态估计尤其重要的两个李群SO(3)和SE(3)。

### 李代数引出

对任意旋转矩阵R，我们知到：
$$
RR^{\mathrm{T}}=I.
$$
现在，我们说R是某个相机的旋转，它会随时间连续变化，即为时间的函数：$R(t)$

由于其仍为旋转矩阵，故：
$$
\boldsymbol{R}(t)\boldsymbol{R}(t)^\mathrm{T}=\boldsymbol{I}.
$$
在等式两边对时间求导，得到：
$$
\dot{\boldsymbol{R}}(t)\boldsymbol{R}(t)^\mathrm{T}+\boldsymbol{R}(t)\dot{\boldsymbol{R}}(t)^\mathrm{T}=0.
$$
整理得：
$$
\dot{\boldsymbol{R}}(t)\boldsymbol{R}(t)^\mathrm{T}=-\left(\dot{\boldsymbol{R}}(t)\boldsymbol{R}(t)^\mathrm{T}\right)^\mathrm{T}.
$$
可以看出$\dot{\boldsymbol{R}}(t)\boldsymbol{R}(t)^\mathrm{T}$是一个反对称矩阵，在上一节我们引入符号^将一个向量变成了一个反对称矩阵。同理，对于**任意反对称矩阵**我们也能找到**唯一与之对应的向量**把这个运算用符合v表示

> 对上一节的回顾：
> <img src="C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240924214107486.png" alt="image-20240924214107486" style="zoom:80%;" />

于是我们可以找到一个**三维向量**$\phi(t)\in\mathbb{R}^3$与之对应：
$$
\dot{\boldsymbol{R}}(t)\boldsymbol{R}(t)^{\mathrm{T}}=\boldsymbol{\phi}(t)^{\wedge}.
$$
我们对等式同时右乘$R(t)$，则：
$$
\dot{\boldsymbol{R}}(t)=\boldsymbol{\phi}(t)^\wedge\boldsymbol{R}(t)=\begin{bmatrix}0&-\phi_3&\phi_2\\\phi_3&0&-\phi_1\\\\-\phi_2&\phi_1&0\end{bmatrix}\boldsymbol{R}(t).
$$
可以看到每对旋转矩阵求一次导数，只需左乘一个$\phi^{\wedge}(t)$矩阵。考虑$t_0=0$时刻，设此时旋转矩阵R(0) = I。则根据导数定义将R(t)在t=0附近进行一阶泰勒展开：
$$
\begin{aligned}
R(t)& \approx\boldsymbol{R}\left(t_{0}\right)+\dot{\boldsymbol{R}}\left(t_{0}\right)\left(t-t_{0}\right) \\
&=I+\phi(t_{0})^{\wedge}(t).
\end{aligned}
$$
可以看到$\phi $反映了R的导数性质，故称他在SO(3)原点附近的正切空间上。同时在t0附近，设Φ保持为常数$\boldsymbol{\phi}(t_0)=\boldsymbol{\phi}_0$。那么根据上上一个式子：
$$
\dot{\boldsymbol{R}}(t)=\boldsymbol{\phi}(t_0)^\wedge\boldsymbol{R}(t)=\boldsymbol{\phi}_0^\wedge\boldsymbol{R}(t).
$$
上式是一个关于R的微分方程·，而且有初始值$\boldsymbol{R}(0)=I$，故可解得：
$$
\boldsymbol{R}(t)=\exp{(\boldsymbol{\phi}_0^{\wedge}t)}
$$
这说明在t=0附近，旋转矩阵可由以上公式计算出来。可以看到旋转矩阵R与另一个反对称矩阵$\phi_{0}^{\wedge}t$通过指数关系发生了联系。但是矩阵的指数是什么呢？我们还有以下问题：

1.给定某时刻的旋转矩阵R我们就能得到一个Φ，它描述了R在局部的导数关系。与R对应的Φ有什么含义？我们说**Φ正是对应到SO(3)上的李代数$\text{so(3)}$**

2.其次，在给定某个**向量Φ**时，矩阵指数exp(Φ^)如何计算？反之，给定R时能否具有相反的运算来计算Φ？事实上这正是李群与李代数间的指数/对数映射！

下面进一步解决这两个问题。

### 李代数定义

每个李群都有与之对应的李代数，其描述了李群的局部性质。

李代数由一个集合V，一个数域F和一个二元运算[,]组成。如果满足以下几条性质，则称$(\mathbb{V},\mathbb{F},[,])$为一个李代数，记作g

![image-20240924220906834](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240924220906834.png)

其中二元运算被称为李括号。

![image-20240924221054160](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240924221054160.png)

### 李代数so(3)

之前提到的**Φ**就是一种李代数。SO(3)对应的李代数是定义在$R^3$上的向量，每个Φ都可以生成一个反对称矩阵：
$$
\boldsymbol{\Phi}=\boldsymbol{\phi}^\wedge=\begin{bmatrix}0&-\phi_3&\phi_2\\\\\phi_3&0&-\phi_1\\\\-\phi_2&\phi_1&0\end{bmatrix}\in\mathbb{R}^{3\times3}
$$
在此定义下两个向量phi1,ph2的李括号为：
$$
[\phi_1,\phi_2]=(\boldsymbol{\Phi}_1\boldsymbol{\Phi}_2-\boldsymbol{\Phi}_2\boldsymbol{\Phi}_1)^\vee .
$$
由于向量Φ与反对称矩阵是一一对应的，那么我们就说**李代数so(3)的元素**是*三维向量*或者*三维反对称矩阵*，不加区别：
$$
\mathfrak{so}(3)=\left\{\phi\in\mathbb{R}^3,\boldsymbol{\Phi}=\phi^\wedge\in\mathbb{R}^{3\times3}\right\}.
$$
至此，**李代数so(3)**即为一个由三维向量组成的集合，每个向量对应一个**反对称矩阵**，可以用于表达旋转矩阵的导数。他与**SO(3)**的关系由指数映射给出：
$$
R=\exp(\phi^{\wedge}).
$$

### 李代数se(3)

对于SE(3)他也有对应的李代数se(3)。其位于$R^6$空间中
$$
\mathfrak{sc}(3)=\left\{\boldsymbol{\xi}=\begin{bmatrix}\rho\\\\\boldsymbol{\phi}\end{bmatrix}\in\mathbb{R}^6,\boldsymbol{\rho}\in\mathbb{R}^3,\boldsymbol{\phi}\in\mathfrak{so}(3),\boldsymbol{\xi}^\wedge=\begin{bmatrix}\boldsymbol{\phi}^\wedge&\boldsymbol{\rho}\\\\\boldsymbol{0}^\mathrm{T}&0\end{bmatrix}\in\mathbb{R}^{4\times4}\right\}.
$$
我们把每个se(3)元素记作$\xi $，它是一个**六维向量**，前三维为平移**（但含义与变换矩阵中的平移不同）**，记作ρ；后三维是旋转记作Φ，实质上是**so(3)元素**。

同时拓展^符号的定义：**将一个六维向量转换成四维矩阵**，但这里不再表示反对称：
$$
\xi^\wedge=\begin{bmatrix}\phi^\wedge&\rho\\0^\mathrm{T}&0\end{bmatrix}\in\mathbb{R}^{4\times4}.
$$
![image-20240924223209796](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240924223209796.png)

李代数se(3)的李括号如下：
$$
[\xi_1,\xi_2]=(\xi_1^\wedge\xi_2^\wedge-\xi_2^\wedge\xi_1^\wedge)^\vee .
$$

### 指数与对数映射

#### so(3)上的指数映射

$$
R=\exp(\phi^{\wedge}).
$$



此部分主要解决如何计算**exp(Φ^)**的问题。这涉及到指数映射的概念

最终可推导出：
$$
\exp(\theta\boldsymbol{a}^\wedge)=\cos\theta\boldsymbol{I}+(1-\cos\theta)\boldsymbol{a}\boldsymbol{a}^\mathrm{T}+\sin\theta\boldsymbol{a}^\wedge.
$$
其中*θ为三维向量Φ的模长*，*a是一个长度为1的方向向量*

这个指数映射**即为罗德里格斯公式**，**通过他我们把李代数so(3)中的任意一个向量对应到了一个位于SO(3)中的旋转矩阵**

同样也能把**SO(3)中的元素对应到so(3)中**：
$$
\phi=\ln\left(\boldsymbol{R}\right)^\vee=\left(\sum_{n=0}^\infty\frac{\left(-1\right)^n}{n+1}\left(\boldsymbol{R}-\boldsymbol{I}\right)^{n+1}\right)^\vee.
$$
但是真正求解**旋转角和转轴**采用如下方式：

> ![image-20240925095351151](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240925095351151.png)

**关于李群与李代数的对应关系的进一步说明**：

> ![image-20240925095526311](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240925095526311.png)

**这也说明李群与李代数的关系即可认为是旋转矩阵R与旋转向量的关系，旋转矩阵的导数可以由旋转向量来指定，指导着如何在旋转矩阵中进行微积分运算！**

#### se(3)上的指数映射

同样可推导得出：
![image-20240925114905514](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240925114905514.png)

其中J为：

![image-20240925114932169](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240925114932169.png)

### 总结：李群与李代数的关系

对于SO(3)和so(3)：

![image-20240925115104629](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240925115104629.png)

对于SE(3)和se(3)：

![image-20240925115158465](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240925115158465.png)



### 李代数求导与扰动模型

使用李代数的一大动机是进行优化，而在优化过程中导数是非常重要的信息。

下面考虑一个问题当在SO(3)中完成两个矩阵乘法时，李代数中so(3)发生了什么改变呢？

假定对某个旋转R，对应的李代数为Φ。我们给他左乘一个微小旋转ΔR，对应李代数为ΔΦ那么李群上的结果就是ΔRR，李代数上为：
$$
\exp\left(\Delta\phi^{\wedge}\right)\exp\left(\phi^{\wedge}\right)=\exp\left(\left(\phi+J_{l}^{-1}\left(\phi\right)\Delta\phi\right)^{\wedge}\right).
$$
另外在李代数上进行加法：
$$
\exp\left((\phi+\Delta\phi)^{\wedge}\right)=\exp\left((J_{l}\Delta\phi)^{\wedge}\right)\exp\left(\phi^{\wedge}\right)=\exp\left(\phi^{\wedge}\right)\exp\left((J_{r}\Delta\phi)^{\wedge}\right).
$$
具体里面的变量含义以及雅可比的展开内容请看书。

另外对于李代数se(3)也有类似的公式。请看课本

**只需记住这为之后在李代数上的微积分做了铺垫**

#### SO(3)上的李代数求导

下面来讨论一个带有李代数的函数，以及关于李代数求导问题。**在SLAM中我们要估计一个相机的位置和姿态，该位姿是由SO(3)上的旋转矩阵或SE(3)上的变换矩阵描述的**。

设某个时刻小罗卜的位姿为T，观察到一个世界坐标位于p的点，产生了一个观测数据z，那么：
$$
z=Tp+w.
$$

> 这里的变换矩阵T，也就是位姿：指的是**相机坐标系到世界坐标系的变换矩阵**
>
> 有两层含义：1.描述了从相机坐标系到世界坐标系的旋转过程
>
> ​                        2.可以用于将世界坐标系下的点转换到相机坐标系下

其中w为噪声。由于他的存在z往往不可能精确满足z = Tp。所以我们通常计算理想的观测与实际数据的误差：
$$
e=\boldsymbol{z}-\boldsymbol{T}\boldsymbol{p}.
$$
假设共有N个这样的路标点和观测，于是就有N个上式。那么，对小罗卜进行位姿估计相当于寻找一个最优的T使得整体误差最小：
$$
\min_TJ(\boldsymbol{T})=\sum_{i=1}^N\left\|\boldsymbol{z}_i-\boldsymbol{T}\boldsymbol{p}_i\right\|_2^2.
$$

>  就是各项平方和加在一起再开根号

求解此问题需要计算目标函数J与变换矩阵T的导数。**这里重要的是我们会经常构建与位姿有关的函数，然后讨论函数关于位姿的导数，以调整当前的估计值**。

**然而SO(3),SE(3)没有良好的定义加法他们是群，如果直接把他们当作不同矩阵来优化就必须加以约束，因此选择李代数求解优化**

使用李代数解决求导有两种思路

1.用李代数表示姿态，然后根据李代数加法对李代数求导

2.对李群左乘或右乘微笑扰动，然后对该扰动求导，称为左扰动和右扰动模型

##### 李代数求导

考虑SO(3)上的情况，假设对一个空间点p进行了旋转得到RP，那么现在要计算旋转之后点的坐标相对于旋转的导数，记作：
$$
\frac{\partial\left(Rp\right)}{\partial R}.
$$
经推导（看书）可最终得到：
$$
\frac{\partial\left(\boldsymbol{Rp}\right)}{\partial\boldsymbol{\phi}}=\left(-\boldsymbol{Rp}\right)^{\wedge}\boldsymbol{J}_l.
$$
含有较复杂的Jl

该方法得到的最终结果即为最佳位姿与下面的扰动模型进行区分

##### 扰动模型（左乘）

对R进行一次扰动ΔR，看结果相对于扰动的变化率。设左扰动对应的李代数为φ，对φ求导。

得到：
$$
\begin{aligned}
\frac{\partial\left(Rp\right)}{\partial\varphi}& =\lim_{\varphi\to0}\frac{\exp\left(\varphi^{\wedge}\right)\exp\left(\phi^{\wedge}\right)p-\exp\left(\phi^{\wedge}\right)p}\varphi  \\
&=\lim_{\varphi\to0}\frac{\left(\boldsymbol{I}+\boldsymbol{\varphi}^\wedge\right)\exp\left(\boldsymbol{\phi}^\wedge\right)\boldsymbol{p}-\exp\left(\boldsymbol{\phi}^\wedge\right)\boldsymbol{p}}\varphi \\
&=\lim_{\varphi\to0}\frac{\varphi^{\wedge}Rp}\varphi=\lim_{\varphi\to0}\frac{-(Rp)^{\wedge}\varphi}\varphi=-(Rp)^{\wedge}.
\end{aligned}
$$
**对于扰动模型求导之后得到的是在当前位姿基础上达到最佳位姿的最佳增量ΔR，故最终最佳的位姿是ΔRR**

#### SE(3)上的李代数求导

![image-20240925170137792](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240925170137792.png)

最终推导得到：

![image-20240925170254675](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240925170254675.png)

把最后的结果定义为一个运算符，它把一个齐次坐标的空间点变换成一个4x6的矩阵。



### 大总结

为什么引入李代数：因为在估计最优位姿时涉及到求导，而求导会有Δ+一个东西这种加法运算，然而SO3,SE3他们作为李群都不满足加法，所以引入李代数

借助李代数我们便可完成这一过程

李群转换为李代数需要ln（李群）

李代数转换为李群需要exp（李代数）

SO3,SE3对应的相互转换的公式可以见上述推导。

并且实际上李代数so3就可以理解为旋转向量（角轴）

### 实践部分

^，v符号的应用

SO3，so3的转换；SE3，se3转换关系

#### Sophus库

演示了一些操作比较简单

只需记住，

李群转李代数：变量名.log()

李代数转李群：exp(变量名)

然后还有^操作为：hat

v操作为：vee

**主要是sophus的配置问题：**

关于locate的搜索问题：

有时他会找不到，这是因为刚装进去的东西不会被laocate所查找到

使用sudo updatadb更新之后再找









#### CMAKE语法

target_link_libraries()







































## 第五讲

### 相机模型

[相机模型、参数和各个坐标系(世界坐标系、相机坐标系、归一化坐标系、图像坐标系、像素坐标系之间变换）_相机坐标系和归一化坐标系-CSDN博客](https://blog.csdn.net/qq_40918859/article/details/122271381)

照片记录了真实世界在成像平面上的投影，但丢失了深度信息

#### 小孔成像模型

##### 世界坐标系到相机坐标系的转换

对一个几何物体作旋转R，平移t的运动称为刚体变换，那么世界坐标系到相机坐标系的变换就是一个刚体变换，**R与t与相机无关又称为相机外参。**
$$
\begin{bmatrix}\mathrm X_\mathrm{c}\\\mathrm Y_\mathrm{c}\\\mathrm Z_\mathrm{c}\\1\end{bmatrix} = \begin{bmatrix}\mathrm R&&\mathrm t\\0^\mathrm{T}&&1\end{bmatrix}\begin{bmatrix}\mathrm X_\mathrm{w}\\\mathrm Y_\mathrm{w}\\\mathrm Z_\mathrm{w}\\1\end{bmatrix} , \mathrm T = \begin{bmatrix}\mathrm R&&\mathrm t\\0^\mathrm{T}&&1\end{bmatrix}
$$
外参也就是slam的估计目标



##### 相机坐标系到成像平面

**首先相机坐标系为：**

![image-20240831214634873](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240831214634873.png)

z轴为相机镜头拍照的方向！

那么根据针孔呈像，一个三维空间下的物体可被呈现到一个平面上，该物体在平面上的坐标如下：

![image-20240831215018999](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240831215018999.png)

整体如下：

![image-20240901101155480](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240901101155480.png)

根据这个相似三角型：
$$
\frac Zf=-\frac X{X^{\prime}}=-\frac Y{Y^{\prime}}.
$$
有负号说明是一个倒像，但是实际上相机会把他进行一个翻转使得最终呈现出**正像**
$$
\frac Zf=\frac X{X^{\prime}}=\frac Y{Y^{\prime}}.
$$
整理可得：得到了物体在相机坐标系到**呈像平面坐标系的变换**
$$
X^{\prime}=f\frac{X}{Z}\\Y^{\prime}=f\frac{Y}{Z}
$$
其中*Z为物体到相机的距离，f*为相机焦距

##### **呈像平面再转换为像素坐标：**

$$
\begin{cases}u=\alpha X'+c_x\\[2ex]v=\beta Y'+c_y\end{cases}.
$$

将X撇，Y撇代入，将αf写作fx，βf写作fy
$$
\begin{cases}u=f_x\frac{X}{Z}+c_x\\[2ex]v=f_y\frac{Y}{Z}+c_y\end{cases}.
$$
其中X,Y是以米为单位的。u代表像素坐标系下的横坐标，v代表像素坐标系下的纵坐标

cx，cy为像素平面的中心坐标

> 相机拍摄的数字图像在计算机中都是以二维矩阵储存的，它的坐标以**像素**为单位，而且坐标原点一般在图像的左上角，而实际物理图像的坐标原点在图像中心，坐标单位为**米**。

**像素坐标的矩阵形式：**
$$
\begin{pmatrix}u\\v\\1\end{pmatrix}=\frac{1}{Z}\begin{pmatrix}f_x&0&c_x\\\\0&f_y&c_y\\\\0&0&1\end{pmatrix}\begin{pmatrix}X\\\\Y\\\\Z\end{pmatrix}\stackrel{\Delta}{=}\frac{1}{Z}KP.
$$
传统的形式为：
$$
Z\begin{pmatrix}u\\\\v\\\\1\end{pmatrix}=\begin{pmatrix}f_x&0&c_x\\\\0&f_y&c_y\\\\0&0&1\end{pmatrix}\begin{pmatrix}X\\\\Y\\\\Z\end{pmatrix}\triangleq KP.
$$
左侧为齐次坐标，中间矩阵称为内参数（在相机生产后就已经固定），右侧为非齐次坐标，把Z乘过去，内参数矩阵记作K

![image-20240901105204766](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240901105204766.png)

也就是说无论Z取几物体的投影点都是确定的（Z是物体在相机坐标系下到镜头的距离）

**也就是说最终得到了物体在相机坐标系下的坐标与像素坐标系下坐标的关系**
$$
Z\begin{pmatrix}u\\\\v\\\\1\end{pmatrix}\triangleq KP.
$$


那么最后根据世界坐标系到相机坐标系的变换，我们便可完整得到世界坐标系到像素坐标系的变换
$$
ZP_{uv}=Z\left[\begin{array}{c}u\\\\v\\\\1\end{array}\right]=K\left(RP_w+t\right)=KTP_w.
$$
**注：右侧式子隐含了一次非齐次到齐次的变换**



其中在相机坐标转换为像素坐标时可以借助一个中间平面即归一化平面。

那么最终的整体转换公式如下：
![image-20240901115712236](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240901115712236.png)

对一个几何物体作**旋转R**，**平移t**的运动称为刚体变换（**R，T为相机外参**）其中**Zc为物体到相机的距离**，





**整体公式为：**
![image-20240901120508912](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240901120508912.png)

**整体的流程：**

![image-20240901161102867](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240901161102867.png)

![image-20240901161113757](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240901161113757.png)

整体效果如下：

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fb5335c93bb2ee6e5015e7d835780cac.png#pic_center)



#### 畸变问题

畸变发生在归一化平面上的坐标转为像素平面的过程中

针孔前的镜头会引起畸变

![image-20240901161444972](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240901161444972.png)

主要的畸变类型：径向和切向

径向畸变描述：
$$
x_{corrected}=x(1+k_1r^2+k_2r^4+k_3r^6)\\y_{corrected}=y(1+k_1r^2+k_2r^4+k_3r^6)
$$
切向畸变：
$$
\begin{gathered}
x_{corrected} =x+2p_1xy+p_2(r^2+2x^2) \\
y_{corrected} =y+p_{1}(r^{2}+2y^{2})+2p_{2}xy 
\end{gathered}
$$
那么畸变可被描述为：
$$
\begin{cases}x_{corrected}=x(1+k_1r^2+k_2r^4+k_3r^6)+2p_1xy+p_2(r^2+2x^2)\\y_{corrected}=y(1+k_1r^2+k_2r^4+k_3r^6)+p_1(r^2+2y^2)+2p_2xy\end{cases}.
$$
k1、k2、k3、p1、p2这五个参数不一定全部使用，可以根据实际情况合理选择，



### 双目相机模型

左右相机中心距离b称为基线



![image-20240901162501506](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240901162501506.png)

左右像素的几何关系：
$$
\frac{z-f}{z}=\frac{b-u_L+u_R}{b}.
$$
整理得：
$$
z=\frac{fb}{d},\quad d=u_L-u_R.
$$
d为视差描述同一个点在左右目上成像的距离，d最小为1个像素，因此数目**能测量的深度Z有最大值：fb**

虽然距离公式简单，但d不容易计算

### RGB-D相机

**物理手段测量**深度

1.ToF或结构光两种主要原理

2.通常能得到与RGB图对应的深度图

### 图像

相机成像后，生成了图像。

图像在计算机中以矩阵形式存储（二维数组）

需要对感光度量化成数值

![image-20240901164046807](C:\Users\zbw\AppData\Roaming\Typora\typora-user-images\image-20240901164046807.png)

灰度图：

> 灰度图是一种只包含亮度信息的图像，每个像素点的值表示图像在该位置的亮度强度。灰度图不包含颜色信息，因此通常是单通道的。

深度图：

> 深度图是一种特殊的图像，表示场景中每个像素到摄像机的距离。深度信息通常以灰度或伪彩色编码的形式显示。
>
> 深度图像中的每个像素值表示该像素与摄像机之间的距离。

彩色图：

> 彩色图是包含颜色信息的图像，通常由三个通道组成，分别代表红色（R）、绿色（G）和蓝色（B）。通过不同强度的红、绿、蓝组合可以表示各种颜色。



### 实践部分

