---
title: Matlab实现简单的语音信号处理
date: 2021-04-12 13:06:02
tags: Matlab
categories: Matlab
---
学习数字信号处理课程时做过的一个实验。
### 一、实验目的
  1.学习并掌握数字信号处理的基本方法。
  2.学会使用MATLAB对语音信号进行处理。
  3.学习并掌握MATLAB设计滤波器的方法。
### 二、实验内容
   本次实验基于MATLAB R2019a完成，利用MATLAB对录制的语音信号进行了读取，播放，时域分析，频域分析以及滤波处理等操作。
   本次实验主要包括以下几个步骤：
1.音频的录制与导入
    使用手机自带的录音软件录制了格式为wav的一段语音信号。录音完成后，使用MATLAB的audioread函数对语音信号进行采样。
2.语音信号的时域分析和频域分析
    使用plot函数绘制语音信号的时域波形，对语音信号进行快速傅里叶变换，并绘制出语音信号的频域波形。
    <img src="https://cdn.jsdelivr.net/gh/yeyuwenxi/images.github.io/20210412_1.jpg" width="60%" height="60%">
3.设计滤波器对语音信号进行处理
（1）低通滤波器的设计
利用buttord和butter函数设计一个模拟巴特沃斯低通滤波器，并利用bilinear函数采用双线性变换法将该滤波器转换为数字低通滤波器。
<img src="https://cdn.jsdelivr.net/gh/yeyuwenxi/images.github.io/20210412_2.jpg" width="60%" height="60%">
低通滤波器处理后的信号
<img src="https://cdn.jsdelivr.net/gh/yeyuwenxi/images.github.io/20210412_3.jpg" width="60%" height="60%">
（2）高通滤波器的设计
利用buttord和butter函数设计一个模拟巴特沃斯高通滤波器，并利用bilinear函数采用双线性变换法将该滤波器转换为数字高通滤波器。
<img src="https://cdn.jsdelivr.net/gh/yeyuwenxi/images.github.io/20210412_4.jpg" width="60%" height="60%">
高通滤波器处理后的信号
<img src="https://cdn.jsdelivr.net/gh/yeyuwenxi/images.github.io/20210412_5.jpg" width="60%" height="60%">
（3）陷波器的设计
根据陷波器的表达式配置参数，设计陷波器如下图所示
<img src="https://cdn.jsdelivr.net/gh/yeyuwenxi/images.github.io/20210412_6.jpg" width="60%" height="60%">
使用陷波器处理后的频谱
<img src="https://cdn.jsdelivr.net/gh/yeyuwenxi/images.github.io/20210412_7.jpg" width="60%" height="60%">
（4）对语音信号加噪
<img src="https://cdn.jsdelivr.net/gh/yeyuwenxi/images.github.io/20210412_8.jpg" width="60%" height="60%">

### 三、实验代码
[x,fs]=audioread('yuyin.wav');%录入语音信号
n=length(x);%获取语音信号的长度
x_p=fft(x,n);%利用FFT算法对语音信号进行离散傅里叶变换
f=fs*(0:n-1)/n;%频率的归一化
figure(1);%绘制时域和频域波形
subplot(2,1,1);
plot(x);
title('原始语音信号采样后的时域波形');
xlabel('时间轴')
ylabel('幅值A')
subplot(2,1,2);
plot(f,abs(x_p));
title('原始语音信号采样后的频谱图');
xlabel('频率Hz');
ylabel('频率幅值');
%噪声
% noise=0.2*randn(1,n);
% x_z=x+noise';
% sound(x_z,fs);
% n=length(x);
% x_zp=fft(x_z,n);
% f=fs*(0:n/2-1)/n;
% figure(5);
% subplot(2,1,1);
% plot(x_z);
% title('加噪语音信号时域波形');
% xlabel('时间轴')
% ylabel('幅值A')
% subplot(2,1,2);
% plot(f,abs(x_zp(1:n/2)));
% title('加噪语音信号频谱图');
% xlabel('频率Hz');
% ylabel('频率幅值');
% %低通滤波器
fp=800;fs=1300;rs=35;rp=0.5;Fs=44100;
wp=2*Fs*tan(2*pi*fp/(2*Fs));%预畸
ws=2*Fs*tan(2*pi*fs/(2*Fs));
[n,wn]=buttord(wp,ws,rp,rs,'s');
[b,a]=butter(n,wn,'s');
[num,den]=bilinear(b,a,Fs);%双线性变换法  模拟滤波器转数字滤波器
[h,w]=freqz(num,den,512,Fs);
% 512为点数 Fs为频率范围
figure(2)
plot(w,abs(h));
xlabel('频率/Hz');ylabel('幅值');
title('巴特沃斯低通滤波器幅度特性');
axis([0,5000,0,1.2]); 
grid on;%网格线
%高通滤波器
% fp=3800;fs=3300;rs=3;rp=0.5;Fs=44100;
% wp=2*Fs*tan(2*pi*fp/(2*Fs));%预畸
% ws=2*Fs*tan(2*pi*fs/(2*Fs));
% [n,wn]=buttord(wp,ws,rp,rs,'s');
% [b,a]=butter(n,wn,'high','s');
% [num,den]=bilinear(b,a,Fs);%双线性变换法  模拟滤波器转数字滤波器
% [h,w]=freqz(num,den,512,Fs);
% % 512为点数 Fs为频率范围
% figure(2)
% plot(w,abs(h));
% xlabel('频率/Hz');ylabel('幅值');
% title('巴特沃斯高通滤波器幅度特性');
% axis([0,5000,0,1.2]); 
% grid on;%网格线
%陷波器
% f0=3000;Fs=44100;r=0.9;
% w0=2*pi*f0/Fs;
% num=[1 -2*cos(w0) 1];
% den=[1 -2*r*cos(w0) r*r];
% N=1024;
% [H,w]=freqz(num,den,N);
% plot(w/pi/2*Fs,abs(H));
% grid on;
% title('陷波器的幅频响应');
% [s1,Fs]=audioread('yuyin.wav'); 
% %x1=s1(:,1);%选取一个声道的数据
% x1=s1; 
% %sound(x1,Fs);
% N1=length(x1);
% Y1=fft(x1,N1);
% f1=Fs*(0:N1-1)/N1; 
% %t1=(0:N1-1)/Fs;
% figure(3)
% plot(f1,abs(Y1))
% xlabel('频率/Hz');ylabel('幅度');
% title('原始信号频谱');
% grid on;axis([0 50000 0 600])
y=filter(num,den,x);
%sound(y,Fs);
N2=length(y);
Y2=fft(y,N2);
f2=Fs*(0:N2-1)/N2;
%t2=(0:N2-1)/Fs;
figure(4)
plot(f2,abs(Y2))
xlabel('频率/Hz');ylabel('幅度');
title('过滤后信号的频谱'); 
grid on;
axis([0 50000 0 600])

### 四、实验总结
分析语音信号的频谱图可以看出，有效的语音信号主要集中在0到4000Hz之间，根据奈奎斯特定理，最低的采样频率为8000Hz.观察滤波前后语音信号频谱的变化，可以看出滤波器按要求处理了语音信号，本次实验顺利完成。
总的来说，通过这次实验我收获了很多，对matlab有了更加深入的认识，了解到了如何利用matlab的各种函数对语音信号进行处理，这次实验也加深了我对离散傅里叶变换和滤波器设计的理解，巩固了课堂上所学到的知识，也让我看到了自己所学的知识在生活中的应用。

