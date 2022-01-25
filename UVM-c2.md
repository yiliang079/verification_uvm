# <u>**Just a driver**</u>

## **DUT **

------



```verilog
module dut(clk,
					rst_n,	
					rxd,
					rx_dv,
					txd,
					tx_en);
	input clk;
	input rst_n;
	input  [7:0] rxd;
	input rx_dv;
	output [7:0] txd;
	output tx_en;

	reg[7:0] txd;
	reg tx_en;

		always @(posedge clk) begin
						if(!rst_n) begin
						txd 	<= 8'b0;
						tx_en <= 1'b0;
						end
		else begin
			txd <= rxd;
      tx_en <= rx_dv;
		end
end
endmodule
```

## **Driver**

------



```c++
class my_driver extends uvm_driver;
  
  function new(string name = "my_driver", uvm_component parent = null);
    super.new(name, parent);
  endfunction
  
  extern virtual task main_phase(uvm_phase phase);
	//extern	告诉编译器该函数main_phase在类外定义
    //virtual 虚函数
    	//task
endclass
  
task my_driver::main_phase(uvm_phase phase);

    	top_tb.rxd	 <= 8'b0;
      top_tb.rx_dv <= 1'b0;

			while(!top_tb.rst_n);
					@(posedge top_tb.clk);

			for(int i = 0; i <256; i++)
        	begin
        			@(posedge top_tb.clk);
							top_tb.rxd <= $urandom_range(0, 255);
							top_tb.rx_dv <= 1'b1;
							'uvm_info("my_driver", "data is drived", UVM_LOW)
           end
           //uvm_info宏，主要用来display，冗余级别LOW？MEDIUM？HIGH
                //还有error和warning宏
      @(posedge top_tb.clk);
			top_tb.rx_dv <= 1'b0;

endtask			
  //get_full_name()获得当前节点的路径索引
```

[(19条消息) C++ Virtual详解_悦峰原创博客-CSDN博客_c++ virtual](https://blog.csdn.net/ring0hx/article/details/1605254?spm=1001.2101.3001.6650.2&utm_medium=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~default-2.pc_relevant_paycolumn_v2&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~default-2.pc_relevant_paycolumn_v2&utm_relevant_index=5)

实例化driver类

```c++
my_driver t1;
t1 = new();
```

## **Top_tb**

------

**验证平台搭建**

```verilog
`timescale 1ns/1ps
`include "uvm_macros.svh"
	//主要是UVM中的一些宏定义
import uvm_pkg::*;
	//UVM库，导入才能编译my_driver中的uvm_driver
`include "my_driver.sv"
	//导入刚刚写的只有driver的验证平台

module top_tb;
			reg clk;
			reg rst_n;
			reg[7:0] rxd;
			reg rx_dv;
			wire[7:0] txd;
			wire tx_en;

dut my_dut(.clk(clk),
.rst_n(rst_n),
.rxd(rxd),
.rx_dv(rx_dv),
.txd(txd),
.tx_en(tx_en));

initial begin
  
	my_driver drv;
	drv = new("drv", null);
  	//实例化my_driver类，其中一般parent不为null
	drv.main_phase(null);
	$finish();
  
end

initial begin
	clk = 0;
	forever begin
		#100 clk = ~clk;
	end 
end

initial begin
	rst_n = 1'b0;
 			 #1000;
	rst_n = 1'b1;
end

endmodule
```

## **Factory**

------

factory宏将my_driver记录到UVM内部中的一张表 

*`uvm_compoent_utils(my_driver)*

```c++
class my_driver extends uvm_driver;
  
		`uvm_compoent_utils(my_driver)
     	//factory宏将my_driver记录到UVM内部中的一张表 
      
  function new(string name = "my_driver", uvm_component parent = null);
    super.new(name, parent);

		`uvm_info("my_driver", "new is called", UVM_LOW)
      //调用new构建类时输出提示
      
  endfunction
  
  extern virtual task main_phase(uvm_phase phase);
	//extern	告诉编译器该函数main_phase在类外定义
    //virtual 虚函数
    	//task
endclass
  
task my_driver::main_phase(uvm_phase phase);
			`uvm_info("my_driver", "main_phase is called", UVM_LOW)
        //调用main_phase时输出提示
    	top_tb.rxd	 <= 8'b0;
      top_tb.rx_dv <= 1'b0;

			while(!top_tb.rst_n);
					@(posedge top_tb.clk);

			for(int i = 0; i <256; i++)
        	begin
        			@(posedge top_tb.clk);
							top_tb.rxd <= $urandom_range(0, 255);
							top_tb.rx_dv <= 1'b1;
							'uvm_info("my_driver", "data is drived", UVM_LOW)
           end
           //uvm_info宏，主要用来display，冗余级别LOW？MEDIUM？HIGH
                //还有error和warning宏
      @(posedge top_tb.clk);
			top_tb.rx_dv <= 1'b0;

endtask			
  //get_full_name()获得当前节点的路径索引
```

**加入了factory机制后testbench需要改动：**

```verilog
module top_tb;
......
initial begin
	run_test("my_driver");
end
endmodule

//以上代码取代了以下部分的代码，改变了my_driver的实例化方式
initial begin
	my_driver drv;
	drv = new("drv", null);
	drv.main_phase(null);
	$finish();
end
```

性质：

1、使用类名来直接实例化一个类，`uvm_component_utils`宏带来的效果

2、`uvm_component_utils`宏自动调用`main_phase`

3、没有输出`driver`



## **Objection**

------

为何没有finish

```verilog
class my_driver extends uvm_driver;
  
		`uvm_compoent_utils(my_driver)
     	//factory宏将my_driver记录到UVM内部中的一张表 
      
  function new(string name = "my_driver", uvm_component parent = null);
    super.new(name, parent);

		`uvm_info("my_driver", "new is called", UVM_LOW)
      //调用new构建类时输出提示
      
  endfunction
  
  extern virtual task main_phase(uvm_phase phase);
	//extern	告诉编译器该函数main_phase在类外定义
    //virtual 虚函数
    	//task
endclass
  
task my_driver::main_phase(uvm_phase phase);
  
  		phase.raise_objection(this)
  			//与drop_objection成双出现，在仿真时间之前
  
			`uvm_info("my_driver", "main_phase is called", UVM_LOW)
        //调用main_phase时输出提示
    	top_tb.rxd	 <= 8'b0;
      top_tb.rx_dv <= 1'b0;

			while(!top_tb.rst_n);
					@(posedge top_tb.clk);

			for(int i = 0; i <256; i++)
        	begin
        			@(posedge top_tb.clk);
							top_tb.rxd <= $urandom_range(0, 255);
							top_tb.rx_dv <= 1'b1;
							'uvm_info("my_driver", "data is drived", UVM_LOW)
           end
           //uvm_info宏，主要用来display，冗余级别LOW？MEDIUM？HIGH
                //还有error和warning宏
      @(posedge top_tb.clk);
			top_tb.rx_dv <= 1'b0;

  phase.drop_objection(this);
  		//可以理解为 $finish 平替
endtask			
```



## **Virtual Interface** 

------

1、使用宏来避免绝对路径(没什么用

2、加入virtual interface

**interface的定义**

```verilog
//新文件 my_if.sv
interface my_if(input clk, input rst_n)
  logic [7:0] data;
  logic valid;
endinterface
```

```verilog
//top_tb中
my_if input_if(clk, rst_n);
my_if output_if(clk, rst_n);

dut my_dut(.clk(clk),
           .rst_n(rst_n),
           .rxd(input_if.data),
           .rx_dv(input_if.valid),
           .txd(output_if.data),
           .tx_en(output_if.valid));
//？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？
//原来版本
dut my_dut(.clk(clk),
					 .rst_n(rst_n),
           .rxd(rxd),
           .rx_dv(rx_dv),
           .txd(txd),
           .tx_en(tx_en));

```

```c++
//在类中使用虚声明来命名一个interface
class my_driver extends uvm_driver;
		virtual my_if vif;

`uvm_compoent_utils(my_driver)
     	//factory宏将my_driver记录到UVM内部中的一张表 
      
  function new(string name = "my_driver", uvm_component parent = null);
    super.new(name, parent);

		`uvm_info("my_driver", "new is called", UVM_LOW)
      //调用new构建类时输出提示
      
  endfunction
  
  extern virtual task main_phase(uvm_phase phase);
	//extern	告诉编译器该函数main_phase在类外定义
    //virtual 虚函数
    	//task
endclass
  
task my_driver::main_phase(uvm_phase phase);
  
  		phase.raise_objection(this)
  			//与drop_objection成双出现，在仿真时间之前
  
			`uvm_info("my_driver", "main_phase is called", UVM_LOW)
        //调用main_phase时输出提示
    	vif.data	 <= 8'b0;
      vif.valid  <= 1'b0;

			while(!vif.rst_n);
					@(posedge vif.clk);

			for(int i = 0; i <256; i++)
        	begin
        			@(posedge vif.clk);
							vif.data <= $urandom_range(0, 255);
							vif.valid <= 1'b1;
							'uvm_info("my_driver", "data is drived", UVM_LOW)
           end
           //uvm_info宏，主要用来display，冗余级别LOW？MEDIUM？HIGH
                //还有error和warning宏
      @(posedge vif.clk);
			vif.valid <= 1'b0;

  phase.drop_objection(this);
  		//可以理解为 $finish 平替
endtask			
```

*仍有问题：`driver`中定义的`vif`没有和`top_tb`中的 `input_if / output_if` 连起来*

注：UVM通过`run_test`实例化了一个脱离了 top_tb 层次化结构的实例。建立了一个新的层次结构。

## **Config_db**

------

Set/get收信寄信

```verilog
//Set in  top_tb
initial begin
  uvm_config_db#(virtual my_if)::set(null, "uvm_test_top", "vif", input_if);
  
end

//Get in my_driver
virtual function void build_phase(uvm_phase phase);
  super.build_phase(phase);//显性调用
  `uvm_info("my_driver", "build_phase is called", UVM_LOW);
  
  if(!uvm_config_db#(virtual my_if)::get(this, "", "vif", vif))
    `uvm_fatal("my_driver", "virtual interface must be set for vif")
    /*
    build_phase: UVM自带的phase
    UVM启动自动执行，在new()之后，main_phase之前执行
    用来通过config_db的set/get来传送数据和实例化变量
    不消耗访问时间
    */
endfunction
```

```c++
 uvm_config_db#(virtual my_if)::set(null, "uvm_test_top", "vif", input_if);
!uvm_config_db#(virtual my_if)::get(this, "", "vif", vif)
  /*第三个变量要一致，将哪个interface：input_if -> my_driver	
    							得到的      interface：vif -> my_driver	
  uvm_test_top: run_test创建的my-driver实例名字
  */
```



## **Summary**

------

`a whole driver`

`include：driver/ factory/ objection/ config_db/ interface/`

​				`build_phase/ main_phase`

```c++
//在类中使用虚声明来命名一个interface
class my_driver extends uvm_driver;
		virtual my_if vif;

`uvm_compoent_utils(my_driver)
     	//factory宏将my_driver记录到UVM内部中的一张表 
      
function new(string name = "my_driver", uvm_component parent = null);
    super.new(name, parent);

		`uvm_info("my_driver", "new is called", UVM_LOW)
      //调用new构建类时输出提示
  endfunction
      
virtual function void build_phase(uvm_phase phase);
  super.build_phase(phase);//显性调用
  `uvm_info("my_driver", "build_phase is called", UVM_LOW);
  if(!uvm_config_db#(virtual my_if)::get(this, "", "vif", vif))
    `uvm_fatal("my_driver", "virtual interface must be set for vif")
endfunction
    
  extern virtual task main_phase(uvm_phase phase);
	//extern	告诉编译器该函数main_phase在类外定义
    //virtual 虚函数
    	//task
endclass
  
task my_driver::main_phase(uvm_phase phase);
  
  		phase.raise_objection(this)
  			//与drop_objection成双出现，在仿真时间之前
  
			`uvm_info("my_driver", "main_phase is called", UVM_LOW)
        //调用main_phase时输出提示
    	vif.data	 <= 8'b0;
      vif.valid  <= 1'b0;

			while(!vif.rst_n);
					@(posedge vif.clk);

			for(int i = 0; i <256; i++)
        	begin
        			@(posedge vif.clk);
							vif.data <= $urandom_range(0, 255);
							vif.valid <= 1'b1;
							'uvm_info("my_driver", "data is drived", UVM_LOW)
           end
           //uvm_info宏，主要用来display，冗余级别LOW？MEDIUM？HIGH
                //还有error和warning宏
      @(posedge vif.clk);
			vif.valid <= 1'b0;

  phase.drop_objection(this);
  		//可以理解为 $finish 平替
endtask		
```

Top_tb

```verilog
`timescale 1ns/1ps
`include "uvm_macros.svh"
	//主要是UVM中的一些宏定义
import uvm_pkg::*;
	//UVM库，导入才能编译my_driver中的uvm_driver
`include "my_driver.sv"
	//导入刚刚写的只有driver的验证平台

module top_tb;
			reg clk;
			reg rst_n;
			reg[7:0] rxd;
			reg rx_dv;
			wire[7:0] txd;
			wire tx_en;

my_if input_if(clk, rst_n);
my_if output_if(clk, rst_n);

dut my_dut(.clk(clk),
           .rst_n(rst_n),
           .rxd(input_if.data),
           .rx_dv(input_if.valid),
           .txd(output_if.data),
           .tx_en(output_if.valid));

initial begin
	run_test("my_driver");
end

initial begin
	clk = 0;
	forever begin
		#100 clk = ~clk;
	end 
end

initial begin
	rst_n = 1'b0;
 			 #1000;
	rst_n = 1'b1;
end

endmodule
```

`my_if.sv`

```verilog
interface my_if(input clk, input rst_n)
  logic [7:0] data;
  logic valid;
endinterface
```



# <u>**More compoent**</u>

## Transaction

------

数据交换![image-20220121115055186](/Users/apple/Library/Application Support/typora-user-images/image-20220121115055186.png)

`Transaction`是整个验证平台中流动的信息单元。Sequence产生出transaction，通过sequencer把此transaction转交给driver，driver根据此transaction的信息驱动接口信号。Monitor监测接口数据，并把数据封装成transaction的形式传递给`reference model`或者`scoreboard`。

`my_transaction.sv`

``` c++
class my_transaction extends uvm_sequence_item;
	rand bit[47:0] dmac;
	rand bit[47:0] smac;
  rand bit[15:0] ether_type;
	rand byte			 pload[];
	rand bit[31:0] ccrc;

constraint pload_cons
{
  pload.size >= 46;
  pload.size <= 1500;
}

function bit[31:0] calc_crc();
	return 32'h0;
endfunction
  
function void post_randomize();
	crc = calc_crc;
endfunction
  
`uvm_object_utils(my_transaction)
function new(string name = "my_transaction");
	super.new(name);
endfunction
  
  endclass
```

在`my_driver`中实现基于`my_transaction`的驱动：

```verilog
//先不写。。。
```

## env

------

通过容器类env来实例化`driver、monitor、reference model、scoreboard`等

所有`env`都派生自`uvm_env`

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20220121150146469.png" alt="image-20220121150146469" style="zoom:25%;" />

```c++
class my_env extends uvm_env;

	my_driver drv;
function new(string name = "my_env", uvm_component parent);
	super.new(name, parent);
endfunction
  
virtual function void build_phase(uvm_phase phase);
	super.build_phase(phase);
	drv = my_driver::type_id::create("drv", this);
//driver的实例化方式 type_name::type_id::create
//this指针表示my_env
endfunction
  `uvm_component_utils(my_env)
endclass
```

`Top_tb`相应的改变

```verilog
initial begin
	run_test("my_env");
end

initial begin
	uvm_config_db#(virtual my_if)::set(null, "uvm_test_top.drv", "vif", input_if);
end

//or
//uvm_config_db#(virtual my_if)::set(null, "uvm_test_top.my_drv", "vif", input_if);
//?
```

## monitor

------

检测DUT行为

`driver`：把`transaction`级别的数据转变成DUT的端口级别，并驱动给DUT

`monitor`：收集DUT端口数据，转换成`transaction`交给后续组件，如：`reference model` / `scoreboard`

```c++
//太长了。。
class my_monitor extends uvm_monitor;
	virtual my_if vif;
	`uvm_component_utils(my_monitor)
  function new(string name = "my_monitor", uvm_component parent = null);
		super.new(name, parent);
	endfunction
    
	virtual function void build_phase(uvm_phase phase);
		super.build_phase(phase);
		if(!uvm_config_db#(virtual my_if)::get(this, "", "vif", vif))
 	  `uvm_fatal("my_monitor", "virtual interface must be set for vif!!!")
  endfunction

extern task main_phase(uvm_phase phase);
extern task collect_one_pkt(my_transaction tr);

endclass

task my_monitor::main_phase(uvm_phase phase);
	my_transaction tr;
	while(1) begin
	tr = new("tr");
	collect_one_pkt(tr);
	end
endtask
    
task my_monitor::collect_one_pkt(my_transaction tr);
	bit[7:0] data_q[$];
	int psize;
	while(1) begin
	@(posedge vif.clk);
	if(vif.valid) break;
	end

	`uvm_info("my_monitor", "begin to collect one pkt", UVM_LOW);
	while(vif.valid) begin
	data_q.push_back(vif.data);
	@(posedge vif.clk);
	end
//pop dmac
	for(int i = 0; i < 6; i++) begin
	tr.dmac = {tr.dmac[39:0], data_q.pop_front()};
	end
 //pop smac
 //pop ether_type
 //pop payload
 //pop crc
	for(int i = 0; i < 4; i++) begin
	tr.crc = {tr.crc[23:0], data_q.pop_front()};
	end
	`uvm_info("my_monitor", "end collect one pkt, print it:", UVM_LOW);
	tr.my_print();
endtask
```

`monitor`需要时刻收集数据， 永不停歇， 所以在`main_phase`中使用`while（ 1）` 循环来实现这一目的。

使用两个`monitor`分别检测DUT的输入和输出`i_mon` / `o_mon`

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20220121154014569.png" alt="image-20220121154014569" style="zoom:25%;" /><img src="/Users/apple/Library/Application Support/typora-user-images/image-20220121154046581.png" alt="image-20220121154046581" style="zoom:25%;" />

## agent

------

封装

所有的`agent`都要派生自`uvm_agent`类， 且其本身是一个`component`， 应该使用`uvm_component_utils`宏来实现`factory`注册。

只需实例化`agent`，不需要实例化里面的类，会自己实例化

封装后需要修改`interface`

**agent的定义**

```c++
class my_agent extends uvm_agent ;
	my_driver drv;
	my_monitor mon;

function new(string name, uvm_component parent);
	super.new(name, parent);
endfunction

extern virtual function void build_phase(uvm_phase phase);
extern virtual function void connect_phase(uvm_phase phase);

`uvm_component_utils(my_agent)
endclass

function void my_agent::build_phase(uvm_phase phase);
	super.build_phase(phase);
	if (is_active == UVM_ACTIVE) begin
		drv = my_driver::type_id::create("drv", this);
	end
	mon = my_monitor::type_id::create("mon", this);
endfunction

function void my_agent::connect_phase(uvm_phase phase);
	super.connect_phase(phase);
endfunction
```

`UVM_ACTIVE` / `UVM_PASSIVE`

`if (is_active == UVM_ACTIVE) begin`
		`drv = my_driver::type_id::create("drv", this);`

这个枚举变量仅有两个值： `UVM_PASSIVE`和`UVM_ACTIVE`。 在`uvm_agent`中， `is_active`的值默认为`UVM_ACTIVE`， 在这种式下， 是需要实例化`driver`的。 那么什么是`UVM_PASSIVE`模式呢？ 以本章的DUT为例， 如图2-5所示， 在输出端口上不需要驱动任何信号， 只需要监测信号。 在这种情况下， 端口上是只需要`monitor`的， 所以`driver`可以不用实例化。

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20220121160504813.png" alt="image-20220121160504813" style="zoom: 25%;" />

## **reference model

------

`reference model`用于完成和DUT相同的功能。 `reference model`的输出被`scoreboard`接收， 用于和DUT的输出相比较。 DUT如果很复杂， 那么`reference model`也会相当复杂。

```c++
class my_model extends uvm_component;
	uvm_blocking_get_port #(my_transaction) port;
	uvm_analysis_port #(my_transaction) ap;

	extern function new(string name, uvm_component parent);
	extern function void build_phase(uvm_phase phase);
	extern virtual task main_phase(uvm_phase phase);

	`uvm_component_utils(my_model)
endclass

function my_model::new(string name, uvm_component parent);
	super.new(name, parent);
endfunction

function void my_model::build_phase(uvm_phase phase);
	super.build_phase(phase);
	port = new("port", this);
	ap = new("ap", this);
endfunction

task my_model::main_phase(uvm_phase phase);
	my_transaction tr;
	my_transaction new_tr;
	super.main_phase(phase);
	while(1) begin
		port.get(tr);
		new_tr = new("new_tr");
		new_tr.my_copy(tr);
		`uvm_info("my_model", "get one transaction, copy and print it:", UVM_LOW)
		new_tr.my_print();
		ap.write(new_tr);
	end
endtask
```

~~数据相连问题先不看。。。太乱了~~

## *scoreboard

------

`my_scoreboard`要比较的数据一是来源于`reference model`， 二是来源于`o_agt`的`monitor`。 

前者通过`exp_port`获取， 而后者通过`act_port`获取。 

在`main_phase`中通过`fork`建立起了两个进程， 一个进程处理`exp_port`的数据， 当收到数据后， 把数据放入`expect_queue`中； 另外一个进程处理`act_port`的数据， 这是DUT的输出数据， 当收集到这些数据后， 从`expect_queue`中弹出之前从exp_port收到的数据， 并调用`my_transaction`的`my_compare`函数。

 采用这种比较处理方式的前提是`exp_port`要比`act_port`先收到数据。 由于DUT处理数据需要延时， 而`reference model`是基于高级语言的处理， 一般不需要延时， 因此可以保证`exp_port`的数据在`act_port`的数据之前到来。

```c++
class my_scoreboard extends uvm_scoreboard;
	my_transaction expect_queue[$];
	uvm_blocking_get_port #(my_transaction) exp_port;
	uvm_blocking_get_port #(my_transaction) act_port;
	
	`uvm_component_utils(my_scoreboard)

extern function new(string name, uvm_component parent = null);
extern virtual function void build_phase(uvm_phase phase);
extern virtual task main_phase(uvm_phase phase);
endclass
  
function my_scoreboard::new(string name, uvm_component parent = null);
	super.new(name, parent);
endfunction

function void my_scoreboard::build_phase(uvm_phase phase);
	super.build_phase(phase);
	exp_port = new("exp_port", this);
	act_port = new("act_port", this);
endfunction

task my_scoreboard::main_phase(uvm_phase phase);
	my_transaction get_expect, get_actual, tmp_tran;
	bit result;
	super.main_phase(phase);
	fork
	while (1) begin
		exp_port.get(get_expect);
		expect_queue.push_back(get_expect);
	end
    
	while (1) begin
		act_port.get(get_actual);
		if(expect_queue.size() > 0) begin
		tmp_tran = expect_queue.pop_front();
		result = get_actual.my_compare(tmp_tran);
		if(result) begin
			`uvm_info("my_scoreboard", "Compare SUCCESSFULLY", UVM_LOW);
		end
      
		else begin
			`uvm_error("my_scoreboard", "Compare FAILED");
			$display("the expect pkt is");
			tmp_tran.my_print();
			$display("the actual pkt is");
			get_actual.my_print();
		end
end
      
else begin
	`uvm_error("my_scoreboard", "Received from DUT, while Expect Que ue is empty");
	$display("the unexpected pkt is");
	get_actual.my_print();
	end
end
    
join
endtask
```



## *field_automation

------

<img src="https://images2017.cnblogs.com/blog/737711/201710/737711-20171020103354115-302556825.png" alt="img" style="zoom: 80%;" />

引入`my_mointor`时， 在`my_transaction`中加入了`my_print`函数； 

引入`reference` `model`时， 加入了`my_copy`函数； 

引入`scoreboard`时， 加入了`my_compare`函数；

上述三个函数虽然各自不同， 但是对于不同的`transaction`来说， 都是类似的： 它们都需要逐字段地对`transaction`进行某些操作。
通过定义`field_automation`自动实现这三个函数。使用`uvm_field`宏来实现

```c++
class my_transaction extends uvm_sequence_item;

rand bit[47:0] dmac;
rand bit[47:0] smac;
rand bit[15:0] ether_type;
rand byte pload[];
rand bit[31:0] crc;
…
`uvm_object_utils_begin(my_transaction)
`uvm_field_int(dmac, UVM_ALL_ON)
`uvm_field_int(smac, UVM_ALL_ON)
`uvm_field_int(ether_type, UVM_ALL_ON)
`uvm_field_array_int(pload, UVM_ALL_ON)
`uvm_field_int(crc, UVM_ALL_ON)
`uvm_object_utils_end
…
endclass
```

使用`uvm_object_utils_begin`和`uvm_object_utils_end`来实现`my_transaction`的`factory`注册， 在这两个宏中间， 使用`uvm_field`宏注册所有字段。 `uvm_field`系列宏随着`transaction`成员变量的不同而不同， 如上面的定义中出现了针对`bit`类型的`uvm_field_int`及针对`byte`类型动态数组的`uvm_field_array_int`。 

当使用上述宏注册之后， 可以直接调用`copy`、 `compare`、 `print`等函数， 而无需自己定义。 这极大地简化了验证平台的搭建， 提高了效率。

引入`field_automation`机制的另外一大好处是简化了`driver`和`monitor`。 在2.3.1节及2.3.3节中， `my_driver`的`drv_one_pkt`任务和`my_monitor`的`collect_one_pkt`任务代码很长， 但是几乎都是一些重复性的代码

```c++
//留着给具体模块的实现
```



## *sequence

------

1、**sequencer**

`sequence`机制用于产生激励， 它是UVM中最重要的机制之一。 在本书前面所有的例子中， 激励都是在`driver`中产生的， 但是在一个规范化的UVM验证平台中， `driver`只负责驱动`transaction`， 而不负责产生`transaction`。 `sequence`机制有两大组成部分， 一是`sequence`， 二是`sequencer`。 

my_squencer.sv

```c++
class my_sequencer extends uvm_sequencer #(my_transaction);
function new(string name, uvm_component parent);
	super.new(name, parent);
endfunction
	`uvm_component_utils(my_sequencer)
endclass
```

`sequencer`派生自`uvm_sequencer`， 并且使用`uvm_component_utils`宏来注册到`factory`中。 `uvm_sequencer`是一个参数化的类， 其参数是`my_transaction`， 即此`sequencer`产生的`transaction`的类型。

`sequencer`产生`transaction`， 而`driver`负责接收`transaction`。 

将`driver`/`monitor`/`sequencer`封装到一起

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20220121173021413.png" alt="image-20220121173021413" style="zoom: 33%;" />

2、**sequence**

<img src="/Users/apple/Library/Application Support/typora-user-images/image-20220121173634615.png" alt="image-20220121173634615" style="zoom: 33%;" />

并没有适合的`sequence`位置

`sequence`不属于验证平台的任何一部分， 但是它与`sequencer`之间有密切的联系

只有在`sequencer`的帮助下， `sequence`产生出的`transaction`才能最终送给driver； 

同样， `sequencer`只有在`sequence`出现的情况下才能体现其价值， 如果没有`sequence`， `sequencer`就几乎没有任何作用。 `sequence`就像是一个弹夹， 里面的子弹是`transaction`， 而`sequencer`是一把枪。 弹夹只有放入枪中才有意义， 枪只有在放入弹夹后才能发挥威力。

`sequence`与`sequencer`还有**显著的区别**。 从本质上来说， `sequencer`是一个`uvm_component`，而`sequence`是一个`uvm_object`。 

与`my_transaction`一样， `sequence`也有其生命周期。 它的生命周期比`my_transaction`要更长一些， 其内的`transaction`全部发送完毕后， 它的生命周期也就结束了。 这就好比一个弹夹， 其里面的子弹用完后就没有任何意义了。 因此， 一个`sequence`应该使用`uvm_object_utils`宏注册到`factory`中:

`my_sequence.sv`

```c++
class my_sequence extends uvm_sequence #(my_transaction);
my_transaction m_trans;

function new(string name= "my_sequence");
	super.new(name);
endfunction

virtual task body();
	repeat (10) begin
	`uvm_do(m_trans)
	end
	#1000;
endtask

`uvm_object_utils(my_sequence)
endclass
```



## default_sequence

------



## base_test
