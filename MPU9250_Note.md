# MPU9250资料

[英文数据手册](https://invensense.tdk.com/wp-content/uploads/2015/02/PS-MPU-9250A-01-v1.1.pdf)

gitee上的某大神中文资料: [MPU-9250: 首个中文九轴芯片资料 (gitee.com)](https://gitee.com/unmannedsupz/MPU-9250)

[寄存器映射手册](../Customize_the_remote_control/61813-GY-MPU9250九轴姿态传感器模块资料/相关芯片数据手册/RM-MPU-9250A-00.pdf)

[MPU9250中文说明书.pdf](file:///E:/File/RoboMaster/ACE/24/Customize_the_remote_control/MPU-9250/MPU9250中文说明书.pdf)



# 教程

[获取MPU9250九轴数据--以四轴飞行器姿态解算为例_小高Ggggg的博客-CSDN博客](https://blog.csdn.net/xgbg_/article/details/126910381?spm=1001.2014.3001.5502)

[MPU6050姿态解算2-欧拉角&旋转矩阵 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/195683958)

https://miaowlabs.com/book/lite/B07.html5

+ LSB

手册中提到的类似于 LSB/°/s 表示的就是灵敏度, 

例如 设置 ACCEL_FS_SEL 寄存器值 为 2, 此时量程为 ±8g 而灵敏度 为4096 LSB/g

如果此时 加速度计读到数据 为 1024, 则实际数值为 1024 / 4096 = 0.25g 

具体的LSB参数如下

![image-20230824151259674](https://raw.githubusercontent.com/wuage2335/My_MD_Online_Graph_Bed/main/img/image-20230824151259674.png)

![image-20230824164510301](https://raw.githubusercontent.com/wuage2335/My_MD_Online_Graph_Bed/main/img/image-20230824164510301.png)





![image-20230824151221523](https://raw.githubusercontent.com/wuage2335/My_MD_Online_Graph_Bed/main/img/image-20230824151221523.png)

![image-20230824165150736](https://raw.githubusercontent.com/wuage2335/My_MD_Online_Graph_Bed/main/img/image-20230824165150736.png)





+ brea姿态旋转

三维的姿态旋转可以通过二维的旋转解算进而推导

如下图，XY坐标系中，向量OP旋转β角度到了OP'的位置：

![img](https://raw.githubusercontent.com/wuage2335/My_MD_Online_Graph_Bed/main/img/v2-38f253e4128e2809f85fb42121f5eff2_r.jpg)

根据三角函数关系，可以列出向量OP与OP'的坐标表示形式：

对比上面个两个式子，将第2个式子展开：

![img](https://pic3.zhimg.com/v2-cdf8b5fa36af46cdd4986cdbc3ec8d2a_r.jpg)

用矩阵形式重新表示为：

![img](https://raw.githubusercontent.com/wuage2335/My_MD_Online_Graph_Bed/main/img/20230820150227.png)

这就是二维旋转的基本形式，中间的矩阵即二维旋转的**旋转矩阵**，坐标中的某一向量左乘该矩阵后，即得到这个向量旋转β角后的坐标。

**三维旋转**

**注意需要按照右手系**

三维旋转可视为绕某个轴旋转得来

此时环绕的轴对应的坐标值不变 

例如下图所示的绕 z 轴旋转:

![img](https://raw.githubusercontent.com/wuage2335/My_MD_Online_Graph_Bed/main/img/20230820152040.png)



+ 滤波算法

数据滤波是去除噪声还原真实数据的一种数据处理技术。任何的数据融合都是为了将多种途径的数据中的噪声滤波，以达到尽可能接近真实值的目的。从这个角度理解，将多种数据融合只是表象，滤除了信号中的噪声才是本质。

互补滤波和卡尔曼滤波都是能去除加速度传感器和陀螺仪的噪声，取得精准的姿态数据。其中，互补滤波简单易懂，适合萌新们初学入门使用。

**互补滤波**

2007年，MIT 大神 Shane Colton 发表了经典论文《The Balance Filter》，里面提出了一种对加速度传感器与陀螺仪进行数据融合的有效方法——互补滤波。互补滤波包括低通滤波（滤除或衰减短期加速度传感器波动），以及高通滤波（消除漂移对陀螺仪的影响）。

在加速度传感器部分，我们通过反正切函数求出了角度，这里用 Acc 表示；在陀螺仪部分，我们得到了角速度，这里用 Gyro表示。

互补滤波的核心公式：

> Angel = 0.98∗(*Angle*+*Gyro* *  *dt* )+0.02∗ *Acc*

其中，

> - Angle 为经过互补滤波后得到的角度
> - Gyro 为陀螺仪部分得到的角速度
> - Acc 为加速度传感器部分通过反正切函数 atan() 再转换单位后的角度
> - dt 为滤波器的运行周期(自己设定的)
> - 0.98 和 0.02 为加权系数 *α* 和 (1−*α*)

```C
// a = tau / (tau + dt)  
// acc = 加速度传感器数据 
// gyro = 陀螺仪数据 
// dt = 运行周期       
float angle;
float a;
float ComplementaryFliter(float acc, float gyro, float dt) 
{
    a = 0.98;  
    angle = a * (angle + gyro * dt) + (1 - a) * (acc);  
    return angle;  
}
```



































































































