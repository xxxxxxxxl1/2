# Go1导盲犬使用文档（Python）

另外一份资料：https://www.yuque.com/ironfatty/nly1un/fscioc

额外的注意事项如下：1、不要去动内置的python版本，可以改软连接，切勿删除旧版本的python，了解清楚软连接原理及过程代码再进行操作
2、权限不够尽量都进入root用户进行操作，能进root用户操作不使用sudo临时赋权，能sudo临时赋权不chmod改文件权限
3、注意系统配置文件修改之前存档，如配置环境变量时如果敲错系统桌面会崩溃，这个时候需要进入命令模式进行恢复，比较麻烦
4、如果下载了很多文件注意使用du命令看下内存使用情况，如果使用率达到100%同样桌面会被禁用，这个时候进入命令模式恢复更加麻烦，所以需要避免这种情况的发生
## 0 基础知识

### 0.1 材料准备

* Go1一台
* USB hub一个
* HDMI视频线
* 显示屏
* 键盘鼠标一套
* 免驱的无线网卡一个

### 0.2 网络连接

* 将Go1上电，将USB hub插到主控nano的USB口上，在USB hub上连接免驱的无线网卡，插上键盘鼠标，插上HDMI视频线，并连接屏幕。如果需要登录，登录密码为``123``
  
* 将主控Nano连接到无线网
* 
* 注：每个端口对应着一个系统，总共有三个，分别的IP地址及系统如下：
* 192.168.123.161（树莓派，这个系统可以直接连接手机热点，主要功能是运动主板）
* 192.168.123.13(Ubuntu，这个系统是没有桌面，可以通过ssh进入系统，ssh相关知识自己学习）
* 192.168.123.14（Ubuntu）
* 192.168.123.15(Ubuntu，和14一样需要通过数据线共享热点的方式进行联网，怎么设置上网查找，设置的时候需要格外注意！！！）
* 

### 0.3 更换主Nano软件安装源

点击Nano桌面上``Terminal``的图标，输入以下代码：

```bash
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
sudo vim /etc/apt/sources.list # 连按两下d即可整行删除，然后输入i进行编辑
```

在`sources.list`中写入国内源，这里选择清华的镜像源：

vim的操作教程可以参考：https://www.runoob.com/linux/linux-vim.html

```bash
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic main multiverse restricted universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-security main multiverse restricted universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-updates main multiverse restricted universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-backports main multiverse restricted universe

输入完后，退出vim。
```
然后继续输入，更新源

```bash
sudo apt-get update
sudo apt-get upgrade
```

如果有选项，默认即可。

### 0.4 主Nano的pip换源

点击Nano桌面上``Terminal``的图标，输入以下代码：

```bash
mkdir ~/.pip
vim ~/.pip/pip.conf
```

写入

```bash
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
[install]
trusted-host = https://pypi.tuna.tsinghua.edu.cn
```
输入完后，保存退出``vim``。

### 0.5 在主Nnao上安装jtop工具

jtop工具可以显示Nano各硬件的使用情况

点击Nano桌面上``Terminal``的图标，输入以下代码：

```bash
sudo -H python3 -m pip install jetson-stats
```

运行（情况如下图所示）：

```bash
jtop
```

![](./image/Go1_Guidedog/jtop截图.png)

### 0.6 SSH连接头部Nano

点击主控Nano桌面上``Terminal``的图标，输入以下代码：

```bash
ssh unitree@192.168.123.13
password：123
```

![](./image/Go1_Guidedog/ssh连接成功.png)

### 0.7 SCP发送文件到头部Nano

把``UnitreeCameraSdk``拷到``主控Nano``的桌面上。

点击主控``Nano``桌面上``Terminal``的图标，输入以下代码：

```bash
scp -r /home/Unitree/Desktop/UnitreeCameraSdk unitree@192.168.123.13:/home/unitree/
```
完成传输。

### 0.8 头部Nano``192.168.123.13``开启摄像头数据发送

#### 0.8.1 SSH连接头部Nano``192.168.123.13``

点击主控Nano:``192.168.123.15``桌面上``Terminal``的图标，输入以下代码：

```bash
ssh unitree@192.168.123.13
password：123
```

#### 0.8.2 停止头部Nano相机调用进程

待成功连接后，继续在终端输入以下：

```bash
sudo su
password:123     
sudo ps -A | grep point | awk '{print $1}' | xargs kill -9
sudo ps -aux|grep mqttControlNode|grep -v grep|head -n 1|awk '{print $2}'|xargs kill -9
sudo ps -aux|grep live_human_pose|grep -v grep|head -n 1|awk '{print $2}'|xargs kill -9
```

#### 0.8.3 编译程序，打开Go1头部相机

待成功结束进程后，继续在终端输入：

```bash
cd /home/Unitree/UnitreeCameraSDK
mkdir build 
cd build
cmake ..
make -j8
```

等待``make``完成后

继续在终端输入：

```bash
cd ..
./bins/example_putimagetrans_0 # 打开前方相机
```

#### 0.8.4 打开Go1下巴部位的相机

在主控Nano``192.168.123.15``上打开一个新的终端，在终端输入：
```bash
ssh unitree@192.168.123.13
password:123
cd /home/Unitree/UnitreeCameraSDK
./bins/example_putimagetrans_1 # 打开下巴相机
```

#### 0.8.5 开机自启动（选用）

建议将相机启动脚本加入到头部Nano的开机启动项里：
开机自启动教程：https://blog.csdn.net/weixin_38369492/article/details/110631329

### 0.9 主Nano运动控制环境依赖

* lcm>=1.40
* python-opencv<=4.3
* 安装msgpack
* Paddle Inference环境：https://blog.csdn.net/qq_45779334/article/details/118611953
* 根据unitree_legged_sdk指导手册完成ARM64环境下的配置编译
  
## 1 程序说明

### 1.1 库函数

#### 1.1.0 宇树运动控制库Unitree_Lib

该库函数用于控制Go1的运动。

#### 1.1.1 盲道阈值库color_dist.py

该库函数保存了前方相机与下巴相机的HSV上下阈值，用于滤波。

#### 1.1.2 双目相机库StereoCamera_Lib.py

该库函数从UDP抓取图像，并输出画面。

#### 1.1.3 PaddleDetection_Inference_Lib.py

百度Paddle深度学习框架相关目标检测程序接口。

### 1.2 主函数

#### 1.2.1 盲道跟随程序 guideDogBlindPathFollow.py

该程序的大致实现思路如下：

* 输入下巴的相机图像数据
* 分析图像数据，检测Go1脚下的盲道状态
* 根据Go1脚下的盲道状态选择进入三个状态：1.跟随状态；2.转弯状态；3.搜索状态。
  

程序的实现流程图如下所示：

![](./image/Go1_Guidedog/程序流程1.png)

#### 1.2.2 目标检测数据样本采集程序 dataCollectDetection.py

该程序的大致实现思路如下：

* 输入下巴的相机图像数据
* 将读取到的数据写入文件夹，并命名
  

程序的实现流程图如下所示：

![](./image/Go1_Guidedog/dataCollectDetection程序流程图.png)

#### 1.2.3 实时目标检测脚本 realtimeDetectionDemo.py

* 输入下巴的相机图像数据
* 图像预处理
* 得到识别结果
* 将结果显示
  

程序的实现流程图如下所示：

 ![](./image/Go1_Guidedog/realtimeDetectionDemo程序流程图.png)

#### 1.2.4 多进程导盲犬巡线+识别脚本 guideDogMultiprocess.py

该程序的大致实现思路如下：

* 在1.2.1程序的基础上增加了目标检测功能
* 采用多线程加速

程序的实现流程图如下所示：

![](./image/Go1_Guidedog/程序流程2.jpg)

## 2 实例说明

### 2.1 盲道跟随的实现

编译完成宇树的``unitree_legged_sdk(version:3.4.1)``

运行位于``scripts``文件夹中的``guideDogBlindPathFollow.py``程序:在``scripts``目录下打开终端

```bash
python3 guideDogBlindPathFollow.py
```

### 2.2 PPYOLO训练目标检测模型

#### 2.2.1 相机数据采集

* 解除相机发送端Nano的相机占用，打开头部Nano的相机发送程序。
  
* 将程序包拷到主控nano的桌面上。

* 进入scripts文件夹，右键菜单，打开``open in terminal``,在终端中输入

```bash
python3 dataCollectDetection.py
```

![](./image/Go1_Guidedog/数据采集程序运行.png)

按``CTRL+C``停止采集，数据保存在``/home/unitree/Desktop/guide_dog_go1/data``文件夹中，如下图所示：
![](./image/Go1_Guidedog/图像采集示例.png)

在导盲的程序中，我们主要识别五类对象：

* 红灯
* 绿灯
* 盲道
* 障碍物
* 斑马线

因此，我们开启采集程序后，围绕这五类对象以不同角度进行图像数据采集。得到这些数据之后，将文件夹数据拷到自己的电脑上。

#### 2.2.2 Easydata智能标注

* 将得到的数据``data``，做成一个``.zip``文件。

* 这里我们用到的打标工具是``easydata``,登入``https://ai.baidu.com/easydata/``的网址，网址首页如下图所示。

![](./image/Go1_Guidedog/easydata.png)

* 标注的教程可参考

```bash
https://ai.baidu.com/ai-doc/EasyData/6k9jouf9h
```

五类标签分别为
* 红灯
* 绿灯
* 盲道
* 障碍物
* 斑马线
``注：数据集请以coco的格式导出，便于飞桨平台处理``

#### 2.2.3 PPyolo模型进行训练

* 将打标完成的数据集，利用百度飞桨的平台进行模型的在线训练
网址：``https://www.paddlepaddle.org.cn/``
模型训练参考网址：
 ``https://aistudio.baidu.com/aistudio/projectdetail/2502447?contributionType=1``
* 训练完成后得到的模型文件放置以下路径
``/home/unitree/Desktop/guide_dog_go1/model/``
* 修改配置文件如下
![](./image/Go1_Guidedog/模型配置文件修改.png)  

#### 2.2.4 进行识别Demo展示

为了后续识别程序的顺利执行，需要给nano增加虚拟内存。教程如下：
``https://blog.csdn.net/u012254599/article/details/102935135``

在``/home/unitree/Desktop/guide_dog_go1/scripts``下打开终端

输入以下代码：
```bash
python3 realtimeDetectionDemo.py -v
```
程序运行demo如下所示：

![](./image/Go1_Guidedog/识别demo.png)  

#### 2.3 多进程巡线例程

在``/home/unitree/Desktop/guide_dog_go1/scripts``下打开终端

输入以下代码：
```bash
python3 guideDogMulitprocess.py 
```

Tips：这个时候，可能会出现程序卡住的情况
打开终端输入
```bash
ps -aux | grep py
```
``kill``进程
```bash
sudo kill -s 9 进程号
```

注：1.在文件夹中的程序不建议使用，建议是自己架构出思路图，然后根据思路图进行编程实现（中间通过发送UDP指令进行控制）这样的工作量反而会小一点，模型在飞浆上标注生成，数据集可以让机器狗多跑几遍赛道采集，模型生成之后再传给机器狗，与程序引入模型的路径一致。
2.另外机器狗的相机和腿的sdk需要自己去查询一下运动程序版本再去下载对应的sdk发行版本，查询方式见语雀上官方的开发文档，网址：https://www.yuque.com/ironfatty/nly1un/fscioc
