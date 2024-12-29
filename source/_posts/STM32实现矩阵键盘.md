---
title: STM32实现矩阵键盘
date: 2021-05-30 18:45:21
tags: 
- 矩阵键盘
- 单片机
categories: STM32单片机
---
最近打了块矩阵按键的PCB板，焊完之后，打算写段代码，用STM32的单片机试试。
虽然没用32的单片机写过矩阵键盘的代码，但感觉不就是线反转法分别扫描行和列吗，也没啥难度，写的过程中才发现遇到了点问题。
相比于51单片机，32的io口是要考虑io方向和io模式的。
自己参考网上的一个例子写的代码，没想到被误导了，将4个io口设为上拉输入，另外4个io口设为下拉输入，按键按下的时候先检测行，再检测列，没想到遇到了点莫名奇妙的问题，不管怎么接线，总有两行按键检测不到。
于是我怀疑是不是io模式的问题，上拉输入和下拉输入接在一起，产生的情况可能是无法预测的。
我修改io模式为4个下拉输入，另外4个推挽输出高电平之后，果然解决了这一问题。
对于某些细节方面的东西，有时候还是不能想当然地认为会怎么样啊。

### 下附代码
```
//低四位输出高，高四位下拉输入
void KEY_Init1(void)
{
	
	GPIO_InitTypeDef  GPIO_InitStructure1;
GPIO_InitTypeDef  GPIO_InitStructure2;

  	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);	 
	GPIO_InitStructure1.GPIO_Pin = GPIO_Pin_0|GPIO_Pin_1|GPIO_Pin_2|GPIO_Pin_3;	 
 	GPIO_InitStructure1.GPIO_Mode = GPIO_Mode_Out_PP; 		 
	GPIO_InitStructure1.GPIO_Speed = GPIO_Speed_50MHz;
 	GPIO_Init(GPIOA, &GPIO_InitStructure1);	  
 		GPIO_SetBits(GPIOA,GPIO_Pin_0|GPIO_Pin_1|GPIO_Pin_2|GPIO_Pin_3);


  	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);	 
	GPIO_InitStructure2.GPIO_Pin = GPIO_Pin_4|GPIO_Pin_5|GPIO_Pin_6|GPIO_Pin_7;	 
 	GPIO_InitStructure2.GPIO_Mode = GPIO_Mode_IPD; 		 
	GPIO_InitStructure2.GPIO_Speed = GPIO_Speed_50MHz;
 	GPIO_Init(GPIOA, &GPIO_InitStructure2);	
} 
//低四位下拉输入，高四位输出高
void KEY_Init2(void)
{
	
	GPIO_InitTypeDef  GPIO_InitStructure1;

 		GPIO_InitTypeDef  GPIO_InitStructure2;

  	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);	 
	GPIO_InitStructure1.GPIO_Pin = GPIO_Pin_0|GPIO_Pin_1|GPIO_Pin_2|GPIO_Pin_3;	 
 	GPIO_InitStructure1.GPIO_Mode = GPIO_Mode_IPD; 		 
	GPIO_InitStructure1.GPIO_Speed = GPIO_Speed_50MHz;
 	GPIO_Init(GPIOA, &GPIO_InitStructure1);	
  
  	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);	 
	GPIO_InitStructure2.GPIO_Pin = GPIO_Pin_4|GPIO_Pin_5|GPIO_Pin_6|GPIO_Pin_7;	 
 	GPIO_InitStructure2.GPIO_Mode = GPIO_Mode_Out_PP; 		 
	GPIO_InitStructure2.GPIO_Speed = GPIO_Speed_50MHz;
 	GPIO_Init(GPIOA, &GPIO_InitStructure2);	
	GPIO_SetBits(GPIOA,GPIO_Pin_4|GPIO_Pin_5|GPIO_Pin_6|GPIO_Pin_7);
} 
//函数名：扫描函数
//返回值：有效键值或-1
//功能：矩阵按键扫描，返回一个值
short KeyPad_Scan(void)
{
	short num = -1; //保持按键值返回
	
	u16 readvalue = 0;
	u16 re=0;
	u16 re1=0;
	u16 re2=0;
	KEY_Init1();					//低4位引脚输出高，高4位引脚下拉输入
	readvalue = GPIO_ReadInputData(GPIOA);	//读GPIOA引脚电平
	readvalue &= 0x00ff;				//保留低8位的值（PA7-PA0）
	if(readvalue != 0x000f) 				//高4位引脚有一个被按下
	{
		delay_ms(10);//消抖10ms
		readvalue = GPIO_ReadInputData(GPIOA);	//读GPIOA引脚电平
		readvalue &= 0x00ff;
		if(readvalue != 0x000f)
		{
			re1 = GPIO_ReadInputData(GPIOA);	//再次读取状态
			re1 &= 0x00f0;  				//保留PA4-PA7的值

			KEY_Init2();  				//低4位引脚下拉输入，高4位输出高
			delay_ms(10);
			re2 = GPIO_ReadInputData(GPIOA);	//再次读取状态
			re2 &= 0x000f;				//保留PA0-PA3的值
			
			while((GPIO_ReadInputData(GPIOA)&0x00ff)!=0x00f0);
			//等待按键松开
			
			re=re1|re2;					//取或，就知道哪一行哪一列被按下啦
			
			switch(re)
			{
			case 0x0011: num = 12;break;  
			case 0x0012: num = 8;break;  
			case 0x0014: num = 4;break; 
			case 0x0018: num = 0;break;  
			case 0x0021: num = 13;break;  
			case 0x0022: num = 9;break; 
			case 0x0024: num = 5;break;  
			case 0x0028: num = 1;break;  
			case 0x0041: num = 14;break;  
			case 0x0042: num = 10;break;  
			case 0x0044: num = 6;break;  
			case 0x0048: num = 2;break;  
			case 0x0081: num = 15;break;  
			case 0x0082: num = 11;break;  
			case 0x0084: num = 7;break;  
			case 0x0088: num = 3;break;  
			}
			return num;
		}
	}
	return -1;
}


int main(){
	
	
	  delay_init();	    	 //延时函数初始化	  
		NVIC_Configuration(); 	 //设置NVIC中断分组2:2位抢占优先级，2位响应优先级 	LED_Init();			     //LED端口初始化
	  
	//	OLED_Init();			//初始化OLED  
		//OLED_Clear()  	; 
	 uart_init(115200);	 				//串口初始化为115200
	while(1){
		short key=0;
	key=KeyPad_Scan();
		if(key!=-1){
		printf("key=");
	printf("%d\n",key);}
		
	}
	
}
```
### 参考文章
[STM32的矩阵键盘扫描及处理](https://blog.csdn.net/Daniel__Lai/article/details/108916185?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522162237210816780271566581%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=162237210816780271566581&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_v2~rank_v29-8-108916185.first_rank_v2_pc_rank_v29&utm_term=STM32%E5%AE%9E%E7%8E%B0%E7%9F%A9%E9%98%B5%E6%8C%89%E9%94%AE&spm=1018.2226.3001.4187)