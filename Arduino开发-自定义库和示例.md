title: Arduino开发-自定义库和示例
author: tr
tags:
  - Arduino
categories:
  - Arduino
date: 2021-11-17 02:48:00
---
# Arduino 自定义库

<!--more-->

> 买了一个PS2的手柄，希望写个库文件方便以后控制手柄，所以这里以PS2手柄库文件开发为例，研究如何自定义库,首先新建工程文件夹：`PS2_ESP8266`

## 库格式

> 文件夹的组织类型如下：


![upload successful](/images/pasted-186.png)


## keywords.txt

> 这个文件内容可以为空



## library.properties

> 这里定义一些库的属性

```shell
name=PS2Ctrl
version=1.0.0
author=Tang rui
maintainer=Tang rui
sentence=PS2 library for Arduino
paragraph=An easy way to control PS2 handle
category=Device Control
url=https://github.com/Moipa
architectures=*
includes=PS2Ctrl.h
```

## src目录下库文件编写

> 这里主要写一个头文件，和对头文件的实现方法，用于实例调用

> 头文件： 定义了一个名为`PS2Ctrl_H`的头文件，其内声明了一个`PS2Ctrl`类，编写构造方法和使用方法等。
>
> ```cpp
> #ifndef PS2Ctrl_H
> #define PS2Ctrl_H
> typedef unsigned short u16;
> typedef unsigned char u8;
> class PS2Ctrl
> {
> public:
>     // 构造函数
>     PS2Ctrl(int di_pin, int do_pin, int cs_pin, int clk_pin);
>     // 用于示例的一些函数
>     void PS2_Cmd(unsigned char CMD); // 发送命令到手柄
>     void PS2_ReadData(); // 接收所有PS2控制数据
>     void PS2_ClearData();
>     void PS2_ShortPoll();
>     void PS2_EnterConfig();
>     void PS2_TurnOnAnalogMode(); // 打开模拟模式
>     void PS2_VibrationMode();
>     void PS2_ExitConfig();
>     void PS2_SetInit();
>     void PS2();
>     // 定义手柄接收器的四个针脚 传输的时候需要CS为低电平，CLK由高变低
>     int di_pin; //接收器发送给单片机的信号
>     int do_pin; //单片机发送给接收器的信号
>     int cs_pin; // 选择信号
>     int clk_pin; // 时钟信号
>     unsigned char SELECT, L3, R3, START, UP, RIGHT, DOWN, LEFT, L2, R2, L1, R1, TRIANGLE, CIRCLE, FORK, SQUARE;
>     unsigned char PS2_LX, PS2_LY, PS2_RX, PS2_RY;
> };
> 
> #endif
> ```
>
> 头文件的实现文件：
>
> ```cpp
> #include "PS2Ctrl.h"
> #include "Arduino.h"
> 
> PS2Ctrl::PS2Ctrl(int di_pin,int do_pin,int cs_pin,int clk_pin){
>     this->di_pin = di_pin;
>     this->do_pin = do_pin;
>     this->cs_pin = cs_pin;
>     this->clk_pin = clk_pin;
> }
> 
> #define DI_pin this->di_pin  // D6  DATA
> #define DO_pin this->do_pin  // D7  CMD
> #define CS_pin this->cs_pin  // D5
> #define CLK_pin this->clk_pin // D8
> 
> #define DO_H digitalWrite(DO_pin, HIGH)
> #define DO_L digitalWrite(DO_pin, LOW)
> #define CLK_H digitalWrite(CLK_pin, HIGH)
> #define CLK_L digitalWrite(CLK_pin, LOW)
> #define CS_H digitalWrite(CS_pin, HIGH)
> #define CS_L digitalWrite(CS_pin, LOW)
> typedef unsigned short u16;
> typedef unsigned char u8;
> 
> u8 Data[9];
> u8 SELECT, L3, R3, START, UP, RIGHT, DOWN, LEFT, L2, R2, L1, R1, TRIANGLE, CIRCLE, FORK, SQUARE;
> u8 PS2_LX, PS2_LY, PS2_RX, PS2_RY;
> 
> void PS2Ctrl::PS2_Cmd(u8 CMD)
> {
>   volatile u16 ref = 0x01;
>   Data[1] = 0;
>   for (ref = 0x01; ref < 0x0100; ref <<= 1)
>   {
>     if (ref & CMD)
>     {
>       DO_H;
>     }
>     else
>       DO_L;
>     CLK_H;
>     delayMicroseconds(10);
>     CLK_L;
>     delayMicroseconds(10);
>     CLK_H;
>     if (digitalRead(DI_pin))
>     {
>       Data[1] = ref | Data[1];
>     }
>   }
>   delayMicroseconds(16);
> }
> 
> void PS2Ctrl::PS2_ReadData(void)
> {
>   volatile u8 byt = 0;
>   volatile u16 ref = 0x01;
>   CS_L;
>   PS2_Cmd(0x01);
>   PS2_Cmd(0x42);
>   for (byt = 2; byt < 9; byt++)
>   {
>     for (ref = 0x01; ref < 0x100; ref <<= 1)
>     {
>       CLK_H;
>       delayMicroseconds(10);
>       CLK_L;
>       delayMicroseconds(10);
>       CLK_H;
>       if (digitalRead(DI_pin))
>       {
>         Data[byt] = ref | Data[byt];
>       }
>     }
>     delayMicroseconds(16);
>   }
>   CS_H;
> }
> 
> void PS2Ctrl::PS2_ClearData()
> {
>   u8 a;
>   for (a = 0; a < 9; a++)
>   {
>     Data[a] = 0x00;
>   }
> }
> 
> void PS2Ctrl::PS2_ShortPoll(void)
> {
>   CS_L;
>   delayMicroseconds(16);
> 
>   PS2_Cmd(0x01);
>   PS2_Cmd(0x42);
>   PS2_Cmd(0x00);
>   PS2_Cmd(0x00);
>   PS2_Cmd(0x00);
> 
>   CS_H;
>   delayMicroseconds(16);
> }
> 
> void PS2Ctrl::PS2_EnterConfig(void)
> {
>   CS_L;
>   delayMicroseconds(16);
> 
>   PS2_Cmd(0x01);
>   PS2_Cmd(0x43);
>   PS2_Cmd(0x00);
>   PS2_Cmd(0x01);
>   PS2_Cmd(0x00);
>   PS2_Cmd(0x00);
>   PS2_Cmd(0x00);
>   PS2_Cmd(0x00);
>   PS2_Cmd(0x00);
> 
>   CS_H;
>   delayMicroseconds(16);
> }
> 
> void PS2Ctrl::PS2_TurnOnAnalogMode(void)
> {
>   CS_L;
>   delayMicroseconds(16);
> 
>   PS2_Cmd(0x01);
>   PS2_Cmd(0x44);
>   PS2_Cmd(0x00);
>   PS2_Cmd(0x01);
>   PS2_Cmd(0xEE);
>   PS2_Cmd(0x00);
>   PS2_Cmd(0x00);
>   PS2_Cmd(0x00);
>   PS2_Cmd(0x00);
> 
>   CS_H;
>   delayMicroseconds(16);
> }
> 
> void PS2Ctrl::PS2_VibrationMode(void)
> {
>   CS_L;
>   delayMicroseconds(16);
> 
>   PS2_Cmd(0x01);
>   PS2_Cmd(0x4D);
>   PS2_Cmd(0x00);
>   PS2_Cmd(0x00);
>   PS2_Cmd(0x01);
> 
>   CS_H;
>   delayMicroseconds(16);
> }
> 
> void PS2Ctrl::PS2_ExitConfig(void)
> {
>   CS_L;
>   delayMicroseconds(16);
> 
>   PS2_Cmd(0x01);
>   PS2_Cmd(0x43);
>   PS2_Cmd(0x00);
>   PS2_Cmd(0x00);
>   PS2_Cmd(0x5A);
>   PS2_Cmd(0x5A);
>   PS2_Cmd(0x5A);
>   PS2_Cmd(0x5A);
>   PS2_Cmd(0x5A);
> 
>   CS_H;
>   delayMicroseconds(16);
> }
> 
> void PS2Ctrl::PS2_SetInit(void)
> {
>   PS2_ShortPoll();
>   PS2_ShortPoll();
>   PS2_ShortPoll();
>   PS2_EnterConfig();
>   PS2_TurnOnAnalogMode();
>   PS2_VibrationMode();
>   PS2_ExitConfig();
> }
> 
> void PS2Ctrl::PS2(void)
> {
>   PS2_ClearData();
>   PS2_ReadData();
> 
>   LEFT = !(0x80 & Data[3]);
>   DOWN = !(0x40 & Data[3]);
>   RIGHT = !(0x20 & Data[3]);
>   UP = !(0x10 & Data[3]);
>   START = !(0x08 & Data[3]);
>   R3 = !(0x04 & Data[3]);
>   L3 = !(0x02 & Data[3]);
>   SELECT = !(0x01 & Data[3]);
> 
>   SQUARE = !(0x80 & Data[4]);
>   FORK = !(0x40 & Data[4]);
>   CIRCLE = !(0x20 & Data[4]);
>   TRIANGLE = !(0x10 & Data[4]);
>   R1 = !(0x08 & Data[4]);
>   L1 = !(0x04 & Data[4]);
>   R2 = !(0x02 & Data[4]);
>   L2 = !(0x01 & Data[4]);
> 
>   PS2_RX = Data[5];
>   PS2_RY = Data[6];
>   PS2_LX = Data[7];
>   PS2_LY = Data[8];
> 
>   delay(100);
> }
> ```
>
> 

## examples文件夹下

> 编写示例文件：新增一个文件`PS2Ctrl.ino`
>
> 内容如下：
>
> ```c
> #include "PS2Ctrl.h"
> 
> PS2Ctrl ps2Ctrl(12,13,14,15);
> 
> void setup()
> {
>   Serial.begin(9600);
> 
>   // 设置给PS2控制器的这几个接口模式
>   pinMode(ps2Ctrl.di_pin, INPUT);
>   pinMode(ps2Ctrl.do_pin, OUTPUT);
>   pinMode(ps2Ctrl.cs_pin, OUTPUT);
>   pinMode(ps2Ctrl.clk_pin, OUTPUT);
> }
> 
> 
> void loop()
> {
>   // 读取PS2手柄数据
>   ps2Ctrl.PS2();
>   // 打印按键操作
>   SerialPrintKey();
> }
> 
> 
> void SerialPrintKey()
> {
>   Serial.println("*********key list********");
>   Serial.print("L3:");
>   Serial.print(ps2Ctrl.L3);
>   Serial.println();
>   ;
>   Serial.print("R3:");
>   Serial.print(ps2Ctrl.R3);
>   Serial.println();
>   ;
>   Serial.print("START:");
>   Serial.print(ps2Ctrl.START);
>   Serial.println();
>   ;
>   Serial.print("UP:");
>   Serial.print(ps2Ctrl.UP);
>   Serial.println();
>   ;
>   Serial.print("RIGHT:");
>   Serial.print(ps2Ctrl.RIGHT);
>   Serial.println();
>   ;
>   Serial.print("DOWN:");
>   Serial.print(ps2Ctrl.DOWN);
>   Serial.println();
>   ;
>   Serial.print("LEFT:");
>   Serial.print(ps2Ctrl.LEFT);
>   Serial.println();
>   ;
>   Serial.print("L2:");
>   Serial.print(ps2Ctrl.L2);
>   Serial.println();
>   ;
>   Serial.print("R2:");
>   Serial.print(ps2Ctrl.R2);
>   Serial.println();
>   ;
>   Serial.print("L1:");
>   Serial.print(ps2Ctrl.L1);
>   Serial.println();
>   ;
>   Serial.print("R1:");
>   Serial.print(ps2Ctrl.R1);
>   Serial.println();
>   ;
>   Serial.print("TRIANGLE:");
>   Serial.print(ps2Ctrl.TRIANGLE);
>   Serial.println();
>   ;
>   Serial.print("CIRCLE:");
>   Serial.print(ps2Ctrl.CIRCLE);
>   Serial.println();
>   ;
>   Serial.print("FORK:");
>   Serial.print(ps2Ctrl.FORK);
>   Serial.println();
>   ;
>   Serial.print("SQUARE:");
>   Serial.print(ps2Ctrl.SQUARE);
>   Serial.println();
>   ;
>   Serial.print("PS2_RX:");
>   Serial.print(ps2Ctrl.PS2_RX);
>   Serial.println();
>   ;
>   Serial.print("PS2_RY:");
>   Serial.print(ps2Ctrl.PS2_RY);
>   Serial.println();
>   ;
>   Serial.print("PS2_LX:");
>   Serial.print(ps2Ctrl.PS2_LX);
>   Serial.println();
>   ;
>   Serial.print("PS2_LY:");
>   Serial.print(ps2Ctrl.PS2_LY);
>   Serial.println();
>   ;
> }
> ```
>
> 

## 使用

> 将整个工程文件`PSE_ESP8266`放入`Arduino`的`libraries`内，现在可以在第三方示例库中看到我们的示例了


![upload successful](/images/pasted-185.png)