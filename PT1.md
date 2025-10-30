# 第一次笔记
## Verilog基础语法
### 注释
Verilog的注释规则和C语言相同
### 标识符和关键字
```verilog
reg [3:0] counter;  //reg为关键字,counter为标识符
input clk;          //input为关键字,clk为标志符
input CLK;          //CLK和clk是两个不同的标志符
```
## Verilog数值表示
### 数值种类
Verilog HDL有四种基本的值来表示硬件电路中的电平逻辑

    0:逻辑0或者“假”
    1:逻辑1或者“真”
    x或X:未知(可能为1可能为0)
    z或Z:高阻
Z意味着信号处于高阻状态,常见于信号(input reg)没有驱动时的结果,和信号受到的上下拉有很大关系,  
上拉逻辑值为1,下拉则为0
### 整数数值的表示方法
合法的基数格式有四种

    十进制      'd 'D
    十六进制    'h 'H 
    二进制      'b 'B
    八进制      'o 'O
数值可以指明位宽,也可以不指明位宽  
#### 指明位宽
```verilog
4'b1011         //4bit的二进制数1011
32'h3022_c0de   //32bit的四进制数3022c0de(加下划线是为了增强其可读性)
```
#### 不指明位宽
直接写数字时,默认为十进制表示,以下三种表达方式效果是一样的
```verilog
counter = 'd100;//一般会根据编译器自动分配位宽,常见的为32bit
counter = 100;
counter = 32'h64;
```
#### 负数表示
通常在表示位宽的数字前面加减号来表示负数,例如
```verilog
-6'd15//也就是-15,如果是二进制表示的话,相当于直接去后面数字原码的补码来存储

```
**需要注意的是**,例如  4'd-2  这样,将减号放在基数和数字之间是非法的
#### 科学计数法(待补充)
#### 字符串表示法(待补充)
## Verilog数据类型
Verilog最常用的两种数据类型就是**wire(线网)**和**reg(寄存器)**,其余类型可以理解为对这两种类型的扩展或辅助
### wire 线网
wire类型用来表示器件之间的物理连线,如果没有元件连接到wire上,wire的值一般为“Z”
```verilog
wire interrupt;
wire flag1,flag2;
wire gnd=1'b0;
```
### reg 寄存器
reg用来表示存储单元,会保持原来的值直到被改写
```verilog
reg clk_temp;
reg flag1,flag2;
```
### 向量
**当位宽大于1时**,wire和reg即可声名为向量的形式,例如
```verilog
reg [3:0]       counter;    //声名4bit位宽的寄存器counter
wire[32-1:0]    gpio_data;  //声名32bit位宽的线网型变量gpio_data
wire[8:2]       addr;       //声名7bit位宽的线网型变量addr,范围为8:2
reg[0:31]       data;       //声名32bit位宽的寄存器变量data,最高有效位为0
```
可以指明向量中的某一些位直接进行运算
```verilog
wire [9:0] data_low=data[0:9];
addr_temp[3:2]=addr[8:7]+1'b1;
```
Verilog还支持bit位后固定位宽的向量域选择访问

**[bit+:width]从起始位bit开始递增,位宽为width**
**[bit-:width]从起始位bit开始递减,位宽为width**

```verilog
//下面两种赋值是等效的
A=data1[31-:8];
A=data1[31:24];

//下面两种赋值是等效的
B=data1[0+:8];
B=data1[0:7];
```
对信号重新进行组合合成新的向量时,需要借助大括号
```verilog
wire [32:0] temp1,temp2;
assign temp1 = {byte[0][7:0],data1[31:8]};//数据拼接
assign temp2 = {32{1'b0}};                //赋值32位的数值0  
```
### 整数,实数,时间寄存器变量
整数,实数,时间等数据类型也属于寄存器类型
#### integer 整数
整数类型用关键字integer来声明,声名时不用指明位宽,位宽和编译器有关,一般为32bit  
reg型变量为无符号数,而integer型变量为有符号数  
例如
```verilog
reg[31:0]   data1;
reg[7:0]    byte[3:0];  //数组变量
integer j;              //整型变量,用来辅助生成数字电路
always@* begin
    for(j=0;j<=3;j=j+1)
    byte1[j]=data1[(j+1)*8-1:j*8];
    //把data1[7:0]...data1[31:24]依次赋值给byte[0][7:0]...byte[3][7:0]
    end
end
```
j的信号在物理意义上并不存在,只是作为一个辅助变量存在
#### real 实数
用关键字real声明,将一个实数值赋给整数,只有整数部分被保存
#### time 时间
特殊的时间寄存器time型变量对仿真时间进行保存,宽度一般为64bit,通过调整系统参数$time获取当前仿真时间
```verilog
time current_time
initial begin
    #100
    current_time = $time;//current_time的大小为100
end

```
#### 数组
在Verilog中允许声名reg,wire,integer,real及其向量类型的数组  
对于多维数组,用户需要说明其每一维的索引
```verilog
integer         flag[7:0];
reg [3:0]       counter[3:0];
wire[7:0]       addr_bus[3:0];
wire            data_bit[7:0][5:0];
reg[31:0]       data_4d[11:0][3:0][3:0][255:0];
```
对数组元素的赋值操作
```verilog
flag[1]=32'd0;                  //将flag数组中的第二个元素赋值为32bit的0值
counter[3] =4'hf;               //将数组counter中的第4个元素赋值赋为4bit 十六进制数f,等效于counter[3][3:0]=4'hf,即可省略宽度
assign addr_bus[0]=8'b0;        //将数组addr_bus中第一个元素的值赋为0
assign data_bit[0][1]=1'b1;     //将数组data_bit的第一行第二列的元素赋值为1,这里不能省略第二个访问标号,即assign data_bit[0]=1'b1;是非法的
data_4d[0][0][0][0][15:0]=15'd3;//将数组data_4d中标号为[0][0][0][0]的寄存器单元15~0bit赋值为3
```
**不要将向量和数组混淆！**向量是一个单独的元件,位宽为n;数组由多个元件组成,其中每个元件的位宽为n或1
#### 存储器(待补充)
#### 参数(待补充)
#### 字符串(待补充)