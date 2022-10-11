@[TOC](目录)
#  一、实验准备
##  1、KEIL5
安装资源及教程可查看以往博客
## 2、STM32F103C8T6的最小核心板
##  3、STM32CubeMX
（1） 官网下载STM32CubeMX（需要绑定邮箱或者注册）
**网址：**[https://www.st.com/en/development-tools/stm32cubemx.html#get-software](https://www.st.com/en/development-tools/stm32cubemx.html#get-software)


（2）管理员身份运行安装程序，点击next：
![在这里插入图片描述](https://img-blog.csdnimg.cn/5cff3457f08c48838cca1d0adc8a5927.png)
（3）点击I accept，接着选择Next:
![在这里插入图片描述](https://img-blog.csdnimg.cn/2e624100e4d14261b4264cbf02274895.png)
(4)勾选第一个，第二个选项是是否同意ST公司收集你的个人使用信息等:
![在这里插入图片描述](https://img-blog.csdnimg.cn/8816483d225f4491acb46ec5277a4edc.png)
(5)选择安装路径（注意：安装位置不要出现中文）：
![在这里插入图片描述](https://img-blog.csdnimg.cn/7564a6ee8eb94060b85785670037a3cd.png)
（6）直接点NEXT，其他不用设置 之后开始安装：
![在这里插入图片描述](https://img-blog.csdnimg.cn/dcc6d12e68e34acd87ce4517be4e1dd0.png)
（7）完成安装：
![在这里插入图片描述](https://img-blog.csdnimg.cn/ea70c3a270924510bf9118c1c9df65c1.png)
##  4、安装HAL库

> STM32 HAL固件库是Hardware Abstraction Layer的缩写，中文名称是：硬件抽象层。HAL库是ST公司为STM32的MCU最新推出的抽象层嵌入式软件，为更方便的实现跨STM32产品的最大可移植性。HAL库的推出，可以说ST也慢慢的抛弃了原来的标准固件库，这也使得很多老用户不满。但是HAL库推出的同时，也加入了很多第三方的中间件，有RTOS，USB，TCP / IP和图形等等。
和标准库对比起来，STM32的HAL库更加的抽象，ST最终的目的是要实现在STM32系列MCU之间无缝移植，甚至在其他MCU也能实现快速移植。
并且从16年开始，ST公司就逐渐停止了对标准固件库的更新，转而倾向于HAL固件库和 Low-layer底层库的更新，停止标准库更新，也就表示了以后使用STM32CubeMX配置HAL/LL库是主流配置环境

**（1）打开STMCubeMX**
![在这里插入图片描述](https://img-blog.csdnimg.cn/80d6348804d14794a193b38da7e309af.png)

**（2）点击HELP->Manage embedded software packages :**
![在这里插入图片描述](https://img-blog.csdnimg.cn/ab9cdd93999b4ef184f881e79e4e562f.png)
**（3） 勾选上你要安装的HAL库， 点击“Install Now” 直到安装成功：**
![在这里插入图片描述](https://img-blog.csdnimg.cn/7dd5970d3f974274999becea3c616c94.png)

#  二、用HAL库点亮流水灯
##  1、新建项目
（1）在STMCubeMX的主界面，创建新项目：
![在这里插入图片描述](https://img-blog.csdnimg.cn/57d162b5a1334173b6c9ce5b35bac02d.png)
(2)在**part name**里选择自己的芯片，点击信息栏中的具体芯片信息选中，点击**start project**：

![在这里插入图片描述](https://img-blog.csdnimg.cn/a09379b8fddc4d238f0f724ba0d41b00.png)
（3）点击**system core**，进入**SYS**，在**debug**下选择**serial wire**：
![在这里插入图片描述](https://img-blog.csdnimg.cn/c83eb2f6d629496bba286632e01e032b.png)
（4）配置时钟，进入**rcc**，有两个时钟，一个是**hse**和**lse**，我们要用是GPIO接口，而这些接口都在APB2里：
![在这里插入图片描述](https://img-blog.csdnimg.cn/807b238e42284556872d7cbdfdccad0d.png)
观察时钟架构，APB2总线的时钟由hse控制，同时在这个界面得把PLLCLK右边选上：
![在这里插入图片描述](https://img-blog.csdnimg.cn/53617be89273413ea847c0af75293a3c.png)
（5）将**hse**那里设为**Crystal/Ceramic Resonator**：
![在这里插入图片描述](https://img-blog.csdnimg.cn/25c5cb36bd944233bb6679ae6cfd136d.png)
（6）相应的引脚设置输出寄存器，就是output一共三项是PA4，PB9，PC15：
![在这里插入图片描述](https://img-blog.csdnimg.cn/54ee9331bf8040d98b57c2e409d8e8fa.png)
（7）点击**project manager**，配置好自己的路径和项目名，然后**IDE**那项改为**MDK-ARM**：
![在这里插入图片描述](https://img-blog.csdnimg.cn/64f72e0ef60142df81d5d5d07c8f75d9.png)
（8）进入 **code generate**界面，选择生成初始化.c/.h文件，后面点击**generate code**，选择**open project**，然后就到KEIL5了：
![在这里插入图片描述](https://img-blog.csdnimg.cn/a50b5fcededb42ec83e245771dd9ed9b.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2b4a49b4c7f44c0492ab9ffd2d1aff54.png)
##   2、KEIL测试
（1）打开**main.c**文件，查看主函数
![在这里插入图片描述](https://img-blog.csdnimg.cn/3a51b0ee634141d2a4cb833580bdb216.png)
（2）将以下代码代替主函数内容：

```c
SystemClock_Config();//系统时钟初始化
  MX_GPIO_Init();//gpio初始化
  while (1)
  {		
		HAL_GPIO_WritePin(GPIOA,GPIO_PIN_4,GPIO_PIN_RESET);//PA4亮灯
		HAL_GPIO_WritePin(GPIOB,GPIO_PIN_9,GPIO_PIN_SET);//PB9熄灯
		HAL_GPIO_WritePin(GPIOC,GPIO_PIN_15,GPIO_PIN_SET);//PC15熄灯
		HAL_Delay(1000);//延时1s
		HAL_GPIO_WritePin(GPIOA,GPIO_PIN_4,GPIO_PIN_SET);//PA4熄灯
		HAL_GPIO_WritePin(GPIOB,GPIO_PIN_9,GPIO_PIN_RESET);//PB9亮灯
		HAL_GPIO_WritePin(GPIOC,GPIO_PIN_15,GPIO_PIN_SET);//PC15熄灯
		HAL_Delay(1000);//延时1s		
		HAL_GPIO_WritePin(GPIOA,GPIO_PIN_4,GPIO_PIN_SET);//PA4熄灯
		HAL_GPIO_WritePin(GPIOB,GPIO_PIN_9,GPIO_PIN_SET);//PB9熄灯
		HAL_GPIO_WritePin(GPIOC,GPIO_PIN_15,GPIO_PIN_RESET);//PC15亮灯
		HAL_Delay(1000);//延时1s
	}
```
（3）进行电路连接，以如下方式连接：
USB转TTL模块和stm32f103c8t6连接：
GND — GND
3v3 — 3v3
TXD — A10
RXD — A9
灯泡引脚分别对应：A4、B9、C15
![在这里插入图片描述](https://img-blog.csdnimg.cn/bea1e75f6c654919aa8a34e152b30a72.png)

（4）烧录并运行：

> 注意烧录时boot0置1，运行时boot0置0

（具体烧录及运行操作流程查看以往博客）
![在这里插入图片描述](https://img-blog.csdnimg.cn/617db5bf311d4f87aaf07c67dddbd7d4.gif#pic_center)

（5）观察GPIO端口的输出波形：
Target界面中，选择跟正确的晶振大小，此处选择8MHz的外部晶振：

> 这个选项在软件仿真中起到很重要的作用，如果选择错误，那么波形一定是错误的，因为时间不准确

![在这里插入图片描述](https://img-blog.csdnimg.cn/fce6108eaa5743d5aa12e22195e0ebf8.png)

Debug页的设置：

```c
两处Parameter需要写自己的芯片名称
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/041a4d623d8b4604bcdf2088d8f5d630.png)
进行Debug调试：
![在这里插入图片描述](https://img-blog.csdnimg.cn/a158ea82beb143f7b71a3a51c539846d.png)
选择逻辑分析仪：
![在这里插入图片描述](https://img-blog.csdnimg.cn/f12b0da836be43b3b1968c9dc3cc8580.png)
选择需观察的引脚：
①点击Setup Logic Analyzer
![在这里插入图片描述](https://img-blog.csdnimg.cn/0c5ab313a2854bd7b45dc8620121b086.png)
②添加要观察的引脚
![在这里插入图片描述](https://img-blog.csdnimg.cn/138f58979f5a4a29827a2ac944570910.png)
进行相关设置并运行
![在这里插入图片描述](https://img-blog.csdnimg.cn/d97393c783d4414288fa86027f70b475.png
![在这里插入图片描述](https://img-blog.csdnimg.cn/0c8a4c956217464592b6671d5a38b1ed.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/bdb255527a78486ca5eb77a0ab711f25.png)
观察相关波形：
![在这里插入图片描述](https://img-blog.csdnimg.cn/32ff9c9f9ff74b79b066b9b3a1efb477.png)

引脚为低电平的灯亮，高电平的灯不亮，高低电平转换周期（LED闪烁周期）为1s左右。
