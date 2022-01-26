# UVM lab `router.sv` 

[TOC]

base

![img](https://img-blog.csdnimg.cn/e00f5e1b5ac24d01942dc01669797b4a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBASSBDIGUgcg==,size_20,color_FFFFFF,t_70,g_se,x_16)

### 管脚图

------

该`router`有5个输入，4个输出，除了时钟和复位端口外其它信号端口均为16bits，如上左图；

需要注意的是，该模块中每个信号的相同bit位算为“一起的”，即操作的时候是`din[0]`, `frame_n[0]`, `valid_n[0]`一起操作，而不能 `din[0]`, `frame_n[1]`, `valid_n[2]`跨位操作.另外该模块儿可以选择从哪路进，从哪路出，相应的路为地址。

### 信号说明

------

**input:** 

`reset_n`     - 低电平有效
`clk`	      - 主输入时钟
`frame_n`     - 在整个输入`packet`中必须保持有效/低电平
`valid_n`     - 数据输入有效位
`din`	      - 数据输入

**output:**

`dout`	      - 数据输出
`valido_n`   - tells output device that "dout" contain valid data
`frameo_n`   - 在整个输出`packet`中必须保持有效/低电平

note1: frame_n must deasserted at least one cycle between packets.
note2: frame_n must be deasserted with the last valid din bit in the frame.

### 设计说明

------

1. 时钟上升沿触发和采样
2. 输入输出均为串行，即1bit / 1clk
3. `packet`包括`header`和`payload`
4. `packet`可以通过任何一个输入端口输入并从对应输出端口输出
5. 输入和输出之间没有内部延迟

### 复位协议

------

![img](https://img-blog.csdnimg.cn/e61ea76c65d041fbbe94bc52ed2be4a9.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBASSBDIGUgcg==,size_20,color_FFFFFF,t_70,g_se,x_16)

1. 复位时，`reset_n`为低电平,`frame_n`和`valid_n`为高电平
2. 有效复位**至少保持1个clk**
3. 复位后**至少等待15个时钟**周期后才可以发送数据

### 输入信号协议

------

![img](https://img-blog.csdnimg.cn/9681fbb3ba804c67bd20c4d3f819e15a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBASSBDIGUgcg==,size_20,color_FFFFFF,t_70,g_se,x_16)

**din信号：**

1. `din[i]`中的i表示从哪路输入，din中的第一段4bit的数据表示输出地址（低位开始），从哪路输出
2. 地址传输完毕后拉高进入隔离段
3. 隔离段结束后开始传输数据（低位开始）

**frame_n信号：**

1. 下降沿指示`packet`的第一位数据
2. 上升沿指示`packet`的最后一位数据

**valid_n信号：**

1. 其在`din`的地址输入时间段可为任意值x
2. 在隔离段`pad`拉高
3. 其拉低时表示数据有效，因此在`payload`段若其拉高，则`din`数据无效
4. 数据输入完毕后拉高

### 输出信号协议

------

![img](https://img-blog.csdnimg.cn/d7240bb7ebe54d6f851e509dc881c39f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBASSBDIGUgcg==,size_20,color_FFFFFF,t_70,g_se,x_16)

当`valido_n`和`frameo_n`均为低时数据有效，除了`packet`最后一位输出数据时`frameo_n`为高 。





ref：

[1]: https://blog.csdn.net/qq_41337361/article/details/122201031	"Synopsys SV Lab Guide—router简介"

