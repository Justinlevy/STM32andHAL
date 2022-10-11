@[TOC](目录)
#  一、实验原理
##  1、STMF1简介
**STM32F1**系列是来自ARM公司具有突破性的以ARM Cortex-M3为内核的32为微处理器，内核为ARM公司为要求高性能，低功耗，低成本，性价比高的嵌入式应用专门设计的Cortex-M内核。
##  2、地址映射
**（1）M3存储器映射**

![在这里插入图片描述](https://img-blog.csdnimg.cn/d59e877c4b814815893faf14b90f540a.png)

**LED灯程序中，宏定义：**

```c
#define GPIOC_BASE (APB2PERIPH_BASE + 0x1000)
#define APB2PERIPH_BASE (PERIPH_BASE + 0x10000)
#define PERIPH_BASE ((uint32_t)0x40000000)
```

 - PERIPH_BASE 外设基地址：因为stm32是32位的，宏展开为0x40000000并转化为 uint32_t
 - APB2PERIPH_BASE 总线基地址：宏展开为PERIPH_BASE加上偏移地址 0x10000
**（2）寄存器寻址**
**GPIOB基址**
GPIOB相关的寄存器，都住在0x4001 0C00到0x4001 0FFF范围内。
![在这里插入图片描述](https://img-blog.csdnimg.cn/d97d349d205f4028b5ae116c1112b218.png)
**端口输入寄存器地址偏移**
存储数据的位置：`0X40010C00+0X0x =`
地址为： `GPIOC_BASE +0x0x`
![在这里插入图片描述](https://img-blog.csdnimg.cn/84dad9b0d1734a5589ce907c87ae81b6.png)
**数据**
![在这里插入图片描述](https://img-blog.csdnimg.cn/83ebeebd01b0478f851b28692f8fdf4f.png)
**（3）地址映射**

```c
GPIO_TypeDef * GPIOx; //定义一个 GPIO_TypeDef 型结构体指针 GPIOx
GPIOx = GPIOA; //把指针地址设置为宏 GPIOA 地址
GPIOx->CRL = 0xffffffff; //通过指针访问并修改 GPIOA_CRL 寄存器
```
##  3、寄存器映射
给已分配好地址(通过存储器映射实现)的有特定功能的内存单元取别名的过程就叫**寄存器映射**
会有**GPIOA->CRL=0x0000 0000**这种写法，表示将16进制数0赋值给GPIOA的CRL寄存器所在的存储单元

```c
#define PERIPH_BASE      ((uint32_t)0x40000000) 
```
这里属于存储器级别的映射，将外设基地址映射到0x40000000

```c
#define APB2PERIPH_BASE       (PERIPH_BASE + 0x10000)
```
这里对外设基地址进行偏移量为0x10000的地址偏移，偏移到APB2总线对应外设区。

```c
#define GPIOA_BASE            (APB2PERIPH_BASE + 0x0800)
```
这里对APB2外设基地址进行偏移量为0x0800的地址偏移，偏移到GPIOA对应区域
##  4、 GPIO端口初始化设置
**时钟配置**
本次实验采用GPIOA、B、C三个端口。该三个端口都属于APB2总线
（1）找到时钟使能寄存器映射基地址
![在这里插入图片描述](https://img-blog.csdnimg.cn/6da6e3495cbb4a7a9feaa365e559bc20.png)
（2）找到端口偏移地址以及对应端口所在位置
![在这里插入图片描述](https://img-blog.csdnimg.cn/2ac8750f530247298db42fa6fb8cb59b.png)
（3）使能对应端口时钟

```c
//----------------APB2使能时钟寄存器 ---------------------
#define RCC_APB2ENR		*((unsigned volatile int*)0x40021018)

	RCC_APB2ENR|=1<<2|1<<3|1<<4;			//APB2-GPIOA、GPIOB、GPIOC外设时钟使能	
```
##  5、输入输出模式和输出速率设置
本次实验采用通用推挽输出模式，最高输出时钟频率2Mhz。分别用到A4、B5、C14三个引脚。其中A4、B5属于端口配置低寄存器偏移地址为0x00，C13属于端口配置高寄存器偏移地址为0x04。
![在这里插入图片描述](https://img-blog.csdnimg.cn/d469670fe47b40c0bd723a106119f501.png)
（1）找到GPIOx端口基地址
![在这里插入图片描述](https://img-blog.csdnimg.cn/812bf5141f1f4ef19e6e4cf5725ca336.png)
（2）配置对应引脚寄存器，基地址+偏移量

```c
//----------------GPIOA配置寄存器 -----------------------
#define GPIOA_CRL		*((unsigned volatile int*)0x40010800)
//----------------GPIOB配置寄存器 -----------------------
#define GPIOB_CRL		*((unsigned volatile int*)0x40010C00)
//----------------GPIOC配置寄存器 -----------------------
#define GPIOC_CRH		*((unsigned volatile int*)0x40011004)

```
（3）设置输出模式为推挽输出，输出速度为2Mhz

```c
	GPIOA_CRL&=0xFFF0FFFF;		//设置位 清零	
	GPIOA_CRL|=0x00020000;		//PA4推挽输出，把第19、18、17、16位变为0010
	
	GPIOB_CRL&=0xFF0FFFFF;		//设置位 清零	
	GPIOB_CRL|=0x00200000;		//PB5推挽输出，把第23、22、21、20变为0010
	 
	GPIOC_CRH&=0xFF0FFFFF;		//设置位 清零	
	GPIOC_CRH|=0x00200000;		//PC14推挽输出，把第23、22、21、20变为0010
```
#  二、流水灯代码实现
##  1、原理
本次实验采用三个灯实现，亮灯状态用1表示,灭灯状态用0表示。
初始状态为0 0 0，
状态一为1 0 0
状态二为0 1 0
状态三为0 0 1
状态三结束后继续进入状态一，一直循环达到流水灯效果。
##  2、c语言实现
**（1）创建项目**
![在这里插入图片描述](https://img-blog.csdnimg.cn/954b3bf272304f599cf8d3a201d58eb1.png)
**（2）选择STM32F103C8开发板**
![在这里插入图片描述](https://img-blog.csdnimg.cn/fa5774544d804f518d8f9dac8a7b4447.png)

> 创建项目出现弹窗，不勾选setup项，只勾选core项

**（3）在output里选择create hex file**
![在这里插入图片描述](https://img-blog.csdnimg.cn/c8a20f82f7c84818a08499a96fa21248.png)
**（4）source group里创建led.c，并写入代码，注意项目结构，使用的引脚是PA7，PB9，PC15**
![在这里插入图片描述](https://img-blog.csdnimg.cn/ed39bc21b9b54bb8a10b38dee1fc0955.png)


```c
//--------------APB2???????------------------------
#define RCC_AP2ENR	*((unsigned volatile int*)0x40021018)
	//----------------GPIOA????? ------------------------
#define GPIOA_CRL	*((unsigned volatile int*)0x40010800)
#define	GPIOA_ORD	*((unsigned volatile int*)0x4001080C)
//----------------GPIOB????? ------------------------
#define GPIOB_CRH	*((unsigned volatile int*)0x40010C04)
#define	GPIOB_ORD	*((unsigned volatile int*)0x40010C0C)
//----------------GPIOC????? ------------------------
#define GPIOC_CRH	*((unsigned volatile int*)0x40011004)
#define	GPIOC_ORD	*((unsigned volatile int*)0x4001100C)
//-------------------???????-----------------------
void  Delay_wxc( volatile  unsigned  int  t)
{
     unsigned  int  i;
     while(t--)
         for (i=0;i<800;i++);
}
//------------------------???--------------------------
int main()
{
	int j=100;
	RCC_AP2ENR|=1<<2;			//APB2-GPIOA??????
	RCC_AP2ENR|=1<<3;			//APB2-GPIOB??????	
	RCC_AP2ENR|=1<<4;			//APB2-GPIOC??????
	//????????? RCC_APB2ENR|=1<<3|1<<4;
	GPIOA_CRL&=0x0FFFFFFF;		//??? ??	
	GPIOA_CRL|=0x20000000;		//PA7????
	GPIOA_ORD|=1<<7;			//???????
	
	GPIOB_CRH&=0xFFFFFF0F;		//??? ??	
	GPIOB_CRH|=0x00000020;		//PB9????
	GPIOB_ORD|=1<<9;			//???????
	
	GPIOC_CRH&=0x0FFFFFFF;		//??? ??
	GPIOC_CRH|=0x30000000;   	//PC15????
	GPIOC_ORD|=0x1<<15;			//???????	
	while(j)
	{	
		GPIOA_ORD=0x0<<7;		//PA7???	
		Delay_wxc(1000000);
		GPIOA_ORD=0x1<<7;		//PA7???
		Delay_wxc(3000000);
		
		GPIOB_ORD=0x0<<9;		//PB9???	
		Delay_wxc(1000000);
		GPIOB_ORD=0x1<<9;		//PB9???
		Delay_wxc(3000000);
		
		GPIOC_ORD=0x0<<15;		//PC15???	
		Delay_wxc(1000000);
		GPIOC_ORD=0x1<<15;		//PC15???
		Delay_wxc(3000000);
	}
}
```
**(5)线路对接**
**USB转TTL模块和stm32f103c8t6连接**：
![在这里插入图片描述](https://img-blog.csdnimg.cn/e8524e0e87d243cf96e1f8b9dff216e7.png)

**（6）烧录**
在build之后会在object文件夹下有对应的hex文件生成

 - 生成hex文件
 - 使用驱动进行烧录操作
 - 连接到电脑，打开mcuisp，上传HEX文件到stm32f103c8t6上：
![在这里插入图片描述](https://img-blog.csdnimg.cn/c07bed208cfe489f97f8eaacb0b76023.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/d7370b97603f4e2394aaf8af1bc2170a.png)
**（7）开始编译后的下载成功**![在这里插入图片描述](https://img-blog.csdnimg.cn/87f27054a3b04123b0abfaf542611740.png)
**（8）结果**
![在这里插入图片描述](https://img-blog.csdnimg.cn/7bf8fa029c9646a9a7f48fca7695beca.gif)

###  3、汇编语言实现

```c
RCC_APB2ENR EQU 0x40021018;配置RCC寄存器,时钟,0x40021018为时钟地址

GPIOB_BASE EQU 0x40010C00
GPIOC_BASE EQU 0x40011000
GPIOA_BASE EQU 0x40010800
	
GPIOB_CRL EQU 0x40010C00
GPIOC_CRH EQU 0x40011004
GPIOA_CRL EQU 0x40010800
	
GPIOB_ODR EQU 0x40010C0C
GPIOC_ODR EQU 0x4001100C
GPIOA_ODR EQU 0x4001080C

Stack_Size EQU  0x00000400;栈的大小
	
                AREA    STACK, NOINIT, READWRITE, ALIGN=3 ;NOINIT： = NO Init，不初始化。READWRITE : 可读，可写。ALIGN =3 : 2^3 对齐，即8字节对齐。
Stack_Mem       SPACE   Stack_Size
__initial_sp




                AREA    RESET, DATA, READONLY

__Vectors       DCD     __initial_sp               ; Top of Stack
                DCD     Reset_Handler              ; Reset Handler
                    
                    
                AREA    |.text|, CODE, READONLY
                    
                THUMB
                REQUIRE8
                PRESERVE8
                    
                ENTRY
Reset_Handler 
				bl LED_Init;bl：带链接的跳转指令。当使用该指令跳转时，当前地址(PC)会自动送入LR寄存器
MainLoop        BL LED_ON_C
                BL Delay
                BL LED_OFF_C
                BL Delay
				BL LED_ON_A
                BL Delay
                BL LED_OFF_A
                BL Delay
				BL LED_ON_B
                BL Delay
                BL LED_OFF_B
                BL Delay
                
                B MainLoop;B:无条件跳转。
LED_Init;LED初始化
                PUSH {R0,R1, LR};R0,R1,LR中的值放入堆栈
                ;控制时钟
                LDR R0,=RCC_APB2ENR;LDR是把地址装载到寄存器中(比如R0)。
                ORR R0,R0,#0x1c
                LDR R1,=RCC_APB2ENR
                STR R0,[R1]
				
				
                ;初始化GPIOA_CRL
                LDR R0,=GPIOA_CRL
                BIC R0,R0,#0x0fffffff;BIC 先把立即数取反，再按位与
                LDR R1,=GPIOA_CRL
                STR R0,[R1]
				
                LDR R0,=GPIOA_CRL
                ORR R0,#0x00000001
                LDR R1,=GPIOA_CRL
                STR R0,[R1]
                ;将PA0置1
                MOV R0,#0x01
                LDR R1,=GPIOA_ORD
                STR R0,[R1]
				
				
                ;初始化GPIOB_CRL
                LDR R0,=GPIOB_CRL
                BIC R0,R0,#0x0fffffff;BIC 先把立即数取反，再按位与
                LDR R1,=GPIOB_CRL
                STR R0,[R1]
				
                LDR R0,=GPIOB_CRL
                ORR R0,#0x00000001
                LDR R1,=GPIOB_CRL
                STR R0,[R1]
                ;将PB0置1
                MOV R0,#0x01
                LDR R1,=GPIOA_ORD
                STR R0,[R1]
				
				
				 ;初始化GPIOC
                LDR R0,=GPIOC_CRH
                BIC R0,R0,#0x0fffffff
                LDR R1,=GPIOC_CRH
                STR R0,[R1]
				
                LDR R0,=GPIOC_CRH
                ORR R0,#0x01000000
                LDR R1,=GPIOC_CRH
                STR R0,[R1]
                ;将PC15置1
                MOV R0,#0x8000
                LDR R1,=GPIOC_ORD
                STR R0,[R1]
             
                POP {R0,R1,PC};将栈中之前存的R0，R1，LR的值返还给R0，R1，PC
LED_ON_A
                PUSH {R0,R1, LR}    
                
                MOV R0,#0x00
                LDR R1,=GPIOA_ORD 
                STR R0,[R1]
             
                POP {R0,R1,PC}
             
LED_OFF_A
                PUSH {R0,R1, LR}    
                
                MOV R0,#0x01
                LDR R1,=GPIOA_ORD 
                STR R0,[R1]
             
                POP {R0,R1,PC}  
LED_ON_B;亮灯
                PUSH {R0,R1, LR}    
                
                MOV R0,#0x00
                LDR R1,=GPIOB_ORD
                STR R0,[R1]
             
                POP {R0,R1,PC}
             
LED_OFF_B;熄灯
                PUSH {R0,R1, LR}    
                
                MOV R0,#0x01
                LDR R1,=GPIOB_ORD
                STR R0,[R1]
             
                POP {R0,R1,PC}  
LED_ON_C;亮灯
                PUSH {R0,R1, LR}    
                
                MOV R0,#0x00
                LDR R1,=GPIOC_ORD
                STR R0,[R1]
             
                POP {R0,R1,PC}
             
LED_OFF_C;熄灯
                PUSH {R0,R1, LR}    
                
                MOV R0,#0x0100
                LDR R1,=GPIOC_ORD
                STR R0,[R1]
             
                POP {R0,R1,PC}             
             
Delay
                PUSH {R0,R1, LR}
                
                MOVS R0,#0
                MOVS R1,#0
                MOVS R2,#0
                
DelayLoop0        
                ADDS R0,R0,#1

                CMP R0,#330
                BCC DelayLoop0
                
                MOVS R0,#0
                ADDS R1,R1,#1
                CMP R1,#330
                BCC DelayLoop0

                MOVS R0,#0
                MOVS R1,#0
                ADDS R2,R2,#1
                CMP R2,#15
                BCC DelayLoop0
                
                
                POP {R0,R1,PC}    
                NOP
				END
```
#  三、总结
1、学习和理解STM32F103系列芯片的地址映射和寄存器映射原理，GPIO端口的初始化设置三步骤
2、了解到烧录过程的基本情况
3、注意烧录是线路链接、面包板是否插稳、烧录选项是否选对
#  四、参考
[https://blog.csdn.net/weixin_47554309/article/details/120810913](https://blog.csdn.net/weixin_47554309/article/details/120810913)
[https://blog.csdn.net/qq_47281915/article/details/120812867](https://blog.csdn.net/qq_47281915/article/details/120812867)
[https://blog.csdn.net/geek_monkey/article/details/86293880](https://blog.csdn.net/geek_monkey/article/details/86293880)
