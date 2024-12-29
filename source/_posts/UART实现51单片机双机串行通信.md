---
title: UART实现51单片机双机串行通信
date: 2021-04-11 21:39:02
tags:
- 单片机
- 汇编
categories: 51单片机
---
### 第一章 系统整体概述
本次单片机双机通讯系统设计实验是通过单片机间的串行通讯控制两个单片机进行通信的设计与制作。一个单片机系统做为发送端，从键盘输入自己的班级和姓名同时显示在 LCD 显示屏上并发送给另外一个单片机，另一个单片机系统作为接收端，把接收到的信息显示在 LCD 显示屏上。通过判断两个单片机系统中LCD屏幕显示的信息是否相同即可判断双机系统之间的串行通讯是否正常进行。
本次双机通讯系统采用了串行口方式1进行通讯，一帧信息为10位，其中有一个起始位，8个数据位和1个停止位。通讯的波特率位2400波特，T1工作在定时器方式2，震荡频率选用11.0592MH，通过计算可得TH1=TL1=0F4H,PCON寄存器的SMOD位为0。
整个系统的硬件部分是由两个单片机最小系统，两个 LCD 显示屏以及一个矩阵键盘构成。
### 第二章 硬件设计与实现
1.准备电路元器件及焊接工具：STC89C51RC芯片2个、IC插座、40p排针2个、11.0592MHz晶振2个、30pF瓷片电容4个、10uF电解电容2个、排阻一个、电阻若干、LCD1602A显示屏2个、10*15cm洞洞板一个、电源线、杜邦线、下载器、电烙铁、焊锡丝等。
2.焊接电路板：按照设计好的原理图进行焊接。
3.调试：下载程序，将两个单片机系统进行连接，通过按键测试双机通讯能否正常进行，若通讯正常，则两个单片机西铜所连接的lcd屏幕将随着按键按下而显示相同的字符，若不显示或字符紊乱，则双机通讯存在故障。
* 焊接成果图
<img src="https://cdn.jsdelivr.net/gh/yeyuwenxi/images.github.io/20210411_10.jpg" width="60%" height="60%">

### 第三章 软件设计与编程
1.编写代码：利用keil软件进行编程，本次设计使用了汇编语言来进行编写 
2.设计仿真电路图：将电源、时钟电路、复位电路、51单片机、LED数码管及LCD1602显示屏正确连接。电源选用+5v，晶振选用11.0592MHz，时钟电路选用30pF电容，复位电路选用10kΩ电阻、10uF电容，LED数码管串联470Ω电阻接在P2引脚，电阻的作用为保护电路,防止led灯被击穿，将键盘矩阵接在P0口，LCD1602A的D0-D7接口接在P1口。
3.仿真调试：用proteus进行仿真测试，将事先编好的程序导入51单片机中，以测试程序是否能达到目标要求。

* 编程
* 发送端程序
ORG 0000H
LJMP MAIN
MAIN:
RS EQU P3.7
RW EQU P3.6
E EQU P3.5
MOV P1,#00000001B	;清屏
ACALL ENABLE
MOV P1,#00111100B	;  功能设定
ACALL ENABLE
MOV P1,#00001100B	 ; 开关控制
ACALL ENABLE
MOV P1,#00000110B	 ;模式设置
ACALL ENABLE
MOV P1,#80H
ACALL ENABLE
MOV R2,#00H
MOV DPTR,#TABLE3
MOV R1,#00H
;OR0:MOV A,R1
;MOVC A,@A+DPTR
;MOV P2,A
;ACALL WRITE
;INC R1
;CJNE A,#00H,OR0
MOV P1,#80H
ACALL ENABLE


ORI:MOV P0,#0FH 
MOV A,P0
CJNE A,#0FH,ORII
LJMP ORI
ORII:
ACALL KEYSCAN
MOV A,30H

ASTART:CLR EA
MOV TMOD,#20H
MOV TH1,#0F4H
MOV TL1,#0F4H
MOV PCON,#00H
SETB TR1
MOV SCON,#50H
ALOOP1:MOV SBUF,#0E1H
JNB TI,$
CLR TI
JNB RI,$
CLR RI
MOV A,SBUF
XRL A,#0E2H
JNZ ALOOP1


ALOOP2:MOV R0,#30H
;MOV R7,#01H
MOV R6,#00H
ALOOP3:MOV SBUF,@R0
MOV A,R6
ADD A,@R0
MOV R6,A
;INC R0
JNB TI,$
CLR TI
;DJNZ R7,ALOOP3
MOV SBUF,R6
JNB TI,$
CLR TI
JNB RI,$
CLR RI
MOV A,SBUF
JNZ ALOOP2
;RET

;MOV P2,#0C0H
;ACALL ENABLE
MOV A,30H
MOV DPTR,#TABLE1
MOVC A,@A+DPTR
MOV P1,A	  ;lcd的显示
ACALL WRITE
INC R1;
CJNE R1,#04H,JISHU
MOV P1,#0C0H
ACALL ENABLE
MOV R1,#05H
JISHU:
ACALL DELAY

SJMP ORI

AJMP $
WRITE:SETB RS
CLR RW
CLR E
ACALL DELAY
SETB E
RET

ENABLE:CLR RS
CLR RW
CLR E
ACALL DELAY
SETB E
RET
KEYSCAN:MOV P0,#0FH
        MOV A,P0
ANL A,#0FH
MOV R3,A
MOV P0,#0F0H
MOV A,P0
ANL A,#0F0H
ORL A,R3
CJNE A,#0FFH,KEYPRO
RET
KEYPRO:MOV R3,A	  ;R3放键值
       MOV DPTR,#KEYVALUE
   MOV R4,#0FFH
KEY1:INC R4	   ;从1到16查找键值
     MOV A,R4	;R4放偏移量
 MOVC A,@A+DPTR
 MOV 31H,R3
 CJNE A,31H,KEY1  ;判断是否与键值相等 
 MOV A,R4
 MOV 30H,A
 RET
DELAY:
MOV R4,#60
D1:MOV R5,#60
D2:MOV R6,#30
DJNZ R6,$
DJNZ R5,D2   
DJNZ R4,D1   
RET
KEYVALUE: DB 77H,7BH,7DH,7EH,0B7H,0BBH,0BDH,0BEH,0D7H,0DBH,0DDH,0DEH,0E7H,0EBH,0EDH,0EEH
TABLE: DB  30H, 31H, 32H, 33H ,34H ,35H ,36H, 37H, 38H, 39H ,41H, 42H, 43H, 44H, 45H, 46H 
TABLE1:DB '1','8','0','3','P','E','N','G','C','H','E','N','L','I','A','G'
TABLE2:  DB 0C0H,0F9H,0A4H,0B0H,99H,92H,82H,0F8H,80H,90H,88H,83H,0C6H,0A1H,86H,8EH
TABLE3: DB " PENG CHENLIANG ",00H 
END

* 接收端程序
ORG 0000H
RS EQU P3.7
RW EQU P3.6
E EQU P3.5
MOV P1,#00000001B	;清屏
ACALL ENABLE
MOV P1,#00111100B	;  功能设定
ACALL ENABLE
MOV P1,#00001100B	 ; 开关控制
ACALL ENABLE
MOV P1,#00000110B	 ;模式设置
ACALL ENABLE
MOV P1,#80H
ACALL ENABLE
MOV R1,#00H
BSTART:CLR EA
MOV TMOD,#20H
MOV TH1,#0F4H
MOV TL1,#0F4H
MOV PCON,#00H
SETB TR1
MOV SCON,#50H
BLOOP1:	 JNB RI,$
CLR RI
MOV A,SBUF
XRL A,#0E1H
JNZ BLOOP1
MOV SBUF,#0E2H
JNB TI,$
CLR TI
MOV R0,#30H
;MOV R7,#01H
MOV R6,#00H

BLOOP2:JNB RI,$
CLR RI
MOV A,SBUF
MOV @R0,A
;INC R0
ADD A,R6
MOV R6,A

;DJNZ R7,BLOOP2

JNB RI,$
CLR RI
MOV A,SBUF
XRL A,R6

JZ END1
MOV SBUF,#0FFH
JNB TI,$
CLR TI
SJMP BLOOP1
END1:MOV SBUF,#00H	

;MOV P2,#0C0H
;ACALL ENABLE
MOV A,30H
MOV DPTR,#TABLE1
MOVC A,@A+DPTR
MOV P1,A	  ;lcd的显示
ACALL WRITE
INC R1;
CJNE R1,#04H,JISHU
MOV P1,#0C0H
ACALL ENABLE
MOV R1,#05H

JISHU:LJMP BSTART



WRITE:SETB RS
CLR RW
CLR E
ACALL DELAY
SETB E
RET

ENABLE:CLR RS
CLR RW
CLR E
ACALL DELAY
SETB E
RET

DELAY:
MOV R4,#60
D1:MOV R5,#60
D2:MOV R6,#30
DJNZ R6,$
DJNZ R5,D2   
DJNZ R4,D1   
RET
KEYVALUE: DB 77H,7BH,7DH,7EH,0B7H,0BBH,0BDH,0BEH,0D7H,0DBH,0DDH,0DEH,0E7H,0EBH,0EDH,0EEH
TABLE: DB  30H, 31H, 32H, 33H ,34H ,35H ,36H, 37H, 38H, 39H ,41H, 42H, 43H, 44H, 45H, 46H
TABLE1:DB '1','8','0','3','P','E','N','G','C','H','E','N','L','I','A','G' 
TABLE2:  DB 0C0H,0F9H,0A4H,0B0H,99H,92H,82H,0F8H,80H,90H,88H,83H,0C6H,0A1H,86H,8EH
TABLE3: DB " PENG CHENLIANG ",00H 
END
* Proteus 仿真电路图
<img src="https://cdn.jsdelivr.net/gh/yeyuwenxi/images.github.io/20210411_11.jpg">

### 第四章 作品调试与分析
本次制作的双机通讯系统是在键盘显示模块之后的又一个更加复杂的模块，不得不说双机通讯系统的设计确实比之前的两个模块难了很多难了很多。从发送端与接收端通讯协议的构建，到键盘扫描，键码获取和键码的发送与传输，每一步都存在很多问题，更加麻烦的是，如何把LCD的显示和通讯系统结合起来。
虽然硬件部分并没有遇到什么困难，但软件部分通讯协议的构建和通讯系统与LCD显示如何相结合的问题确实很让人困扰。经过反复思考后，在发送端的程序中，我把通讯信息发送的部分放在了键盘扫描的循环中，每按下一次键盘，系统就把扫描到的键码转换为键值，发送给接收端，并在发送端的LCD上把键值显示出来。而在接收端，我把LCD显示的程序放在了通讯接受的循环里，系统每接收到一个字符，就把字符显示到接收端的LCD上。由于每按下一次键盘只显示一个字符，故每次只发送一个字符即可。
在LCD显示时我还发现，两个LCDv0口需要的电压居然不一样，起初我还以为是两个LCD颜色不同，所以特性存在些许的差距，后来仔细检查后才发现，原来是两个单片机系统的GND没有接到一起，所以接在两个LCD的v0口接在同一个单片机系统的电位器上并不能正常运行。之后我把两个单片机系统的GND连到了一起，采用同一个电源供电，解决了这一问题。
### 第五章 制作感受
 本次实验模块的设计与制作确实比之前的复杂了很多，但我自己在这个过程中也得到了很大的提高与成长。
从通讯系统的构建，两个系统之间的应答，校验和的检验，到如何把键盘输入的字符正确地发送并正确地显示在LCD屏幕上，每一步充满了各种挫折和困顿。虽然硬件上利用了之前的系统，但软件上协议部分确实很让人头疼。写代码的过程中虽然遇到过很多挫败，也有过很多心情沮丧的时候，但更多的是知识的成长以及心中不断燃烧的好奇与热情，在这样的热情和兴趣下，自己一步步克服障碍，完成了双机通讯系统的设计制作。 虽然每一次看到bug是都很失落，但当最后看到自己的名字成功地显示到两个LCD的屏幕上时，就觉得所有的努力和付出都是值得的。
通过双机通讯系统的设计与制作，自己的的确确学到了很多。从知识方面来看，自己对通讯协议的构建有了初步的认识，了解到了如何在单片机之间进行串行通讯。另外，自己也对汇编语言有了更加深入而细致的认识，看到了汇编语言相对其他语言在单片机操作上的优势，看到了汇编语言的形象化与底层化。除此之外，自己通过这个过程，也收获了更多新的技能，对单片机有了更加深入而全面的认识，为以后的理论学习和课程实践打下了一个稳健的基础。自己也十分期待下一次的实验模块制作，期待自己可以更好地运用所学的知识，制作出更好的作品。
