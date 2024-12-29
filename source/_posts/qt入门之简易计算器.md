---
title: qt入门之简易计算器
date: 2021-06-28 14:54:32
tags: qt
categories: qt
---
最近考完试比较闲了，简单学了一下qt,做了个最基本的计算器。
### 界面设计
通过可视化的方法，拖动实现界面
<img src="https://cdn.jsdelivr.net/gh/yeyuwenxi/images.github.io/20210628_1.png" >

### 逻辑实现
在头文件中实现类的定义和类内成员的声明
```

class MainWindow : public QMainWindow
{
    Q_OBJECT

public:
    explicit MainWindow(QWidget *parent = 0);
    ~MainWindow();
    int num1;
    int num2;
    double result;
    int op;

private slots:
    void on_button_1_clicked();

    void on_button_2_clicked();

    void on_button_3_clicked();

    void on_button_add_clicked();

    void on_button_4_clicked();

    void on_button_5_clicked();

    void on_button_6_clicked();

    void on_button_sub_clicked();

    void on_button_7_clicked();

    void on_button_8_clicked();

    void on_button_9_clicked();

    void on_button_mult_clicked();

    void on_button_clear_clicked();

    void on_button_0_clicked();

    void on_button_equal_clicked();

    void on_button_div_clicked();

private:
    Ui::MainWindow *ui;
};
```
在cpp文件中实现类内成员的定义
```
#include "mainwindow.h"
#include "ui_mainwindow.h"
#include <iostream>
using namespace std;
MainWindow::MainWindow(QWidget *parent) :
    QMainWindow(parent),
    ui(new Ui::MainWindow)
{
    ui->setupUi(this);
    this->setWindowTitle("计算器1.0");
}

MainWindow::~MainWindow()
{
    delete ui;
}

void MainWindow::on_button_1_clicked()
{
    ui->textEdit->moveCursor(QTextCursor::End);
       ui->textEdit->insertPlainText("1");
}

void MainWindow::on_button_2_clicked()
{
    ui->textEdit->moveCursor(QTextCursor::End);
       ui->textEdit->insertPlainText("2");
}

void MainWindow::on_button_3_clicked()
{
    ui->textEdit->moveCursor(QTextCursor::End);
       ui->textEdit->insertPlainText("3");
}

void MainWindow::on_button_add_clicked()
{
    ui->textEdit->moveCursor(QTextCursor::End);
       ui->textEdit->insertPlainText("+");
}

void MainWindow::on_button_4_clicked()
{
    ui->textEdit->moveCursor(QTextCursor::End);
       ui->textEdit->insertPlainText("4");
}

void MainWindow::on_button_5_clicked()
{
    ui->textEdit->moveCursor(QTextCursor::End);
       ui->textEdit->insertPlainText("5");
}

void MainWindow::on_button_6_clicked()
{
    ui->textEdit->moveCursor(QTextCursor::End);
       ui->textEdit->insertPlainText("6");
}

void MainWindow::on_button_sub_clicked()
{
    ui->textEdit->moveCursor(QTextCursor::End);
       ui->textEdit->insertPlainText("-");
}

void MainWindow::on_button_7_clicked()
{
    ui->textEdit->moveCursor(QTextCursor::End);
       ui->textEdit->insertPlainText("7");
}

void MainWindow::on_button_8_clicked()
{
    ui->textEdit->moveCursor(QTextCursor::End);
       ui->textEdit->insertPlainText("8");
}

void MainWindow::on_button_9_clicked()
{
    ui->textEdit->moveCursor(QTextCursor::End);
       ui->textEdit->insertPlainText("9");
}

void MainWindow::on_button_mult_clicked()
{
    ui->textEdit->moveCursor(QTextCursor::End);
       ui->textEdit->insertPlainText("*");
}

void MainWindow::on_button_clear_clicked()
{
 ui->textEdit->setText("");
}

void MainWindow::on_button_0_clicked()
{
    ui->textEdit->moveCursor(QTextCursor::End);
       ui->textEdit->insertPlainText("0");
}

void MainWindow::on_button_equal_clicked()
{
    ui->textEdit->moveCursor(QTextCursor::End);
       ui->textEdit->insertPlainText("=");
       QString text = ui->textEdit->toPlainText();
       string text1=text.toStdString();
       //cout<<text1<<endl;
       //cout<<text1[1]<<endl;
       int len=text1.size();
       const char * text2=text1.c_str();
       cout<<text2;
       int flag=0;
       num1=0;
       num2=0;
       op=0;
       result=0;
       for(int i=0;i<len;i++){
       if(text2[i]>='0'&&text2[i]<='9'){
       if(flag==0){
           num1=num1*10+text2[i]-'0';
       }
       if(flag==1){
           num2=num2*10+text2[i]-'0';
       }
       }else if(text2[i]=='+'){
           flag=1;
           op=1;
           cout<<"+++"<<endl;
       }
       else if(text2[i]=='-'){
           flag=1;
           op=2;
       }
       else if(text2[i]=='*'){
           flag=1;
           op=3;
       }
       else if(text2[i]=='/'){
           flag=1;
           op=4;
       }
       else if(text2[i]=='='){
           if(op==1){
               result=num1+num2;
               cout<<num1<<endl;
               cout<<num2<<endl;
           }
           if(op==2){
               result=num1-num2;
           }
           if(op==3){
               result=num1*num2;
           }
           if(op==4){
               result=(num1*1.00)/num2;
           }

              ui->textEdit->moveCursor(QTextCursor::End);
              ui->textEdit->insertPlainText(QString::number(result));
       }
       }

}

void MainWindow::on_button_div_clicked()
{
    ui->textEdit->moveCursor(QTextCursor::End);
       ui->textEdit->insertPlainText("/");
}

```

### 原理分析
qt整体的架构还是比较清晰的，界面和逻辑部分互相分离，又通过信号与槽进行必要的通信。
头文件中完成了类的定义，而对类的成员函数仅作声明，具体实现在cpp文件中完成，整个过程实现了类内声明，类外定义。
计算器的基本原理也比较简单，按下按钮时，字符附加到编辑框对应的字符串后，最后按下等号时读取整个字符串，分离两个操作数和一个运算符，进行计算，并将最后的计算结果显示到编辑框中。
### 测试结果
生成exe文件并通过命令行配置好注册表文件后，点击exe文件就可以正常运行了，运行结果如图所示。
<img src="https://cdn.jsdelivr.net/gh/yeyuwenxi/images.github.io/20210628_2.png" >