# factory

[TOC]

## 管理测试案例的要求

------

测试案例需要能够修改类的变量

1. 修改约束信息
2. 插入错误条件
3. driver在发送数据前计算数据的冗余值

组件的实例可以根据变量/全局信息进行配置

为全局配置/特殊的对象，控制对象的位置

构造通用的功能

<u>**问题：如何在不改变原始代码的条件下，为transaction or组件的实例增加新的信息？**</u>

## **解决方案：UVM factory**

------



1. 工厂基础结构/寄存机制

   1. ``uvm_object_utils(transaction_type)`
   2. ``uvm_component_utils(component_type)`

2. 静态代理类程序创建对象

   1. `Class_name obj = class_name::type_id::create(...)`

3. 类的覆盖

   1. `set_type_override_by_type()`
   2. `Set_inst_override_by_type()`

   eg: `set_type_override_by_type(transaction::get_type(), transaction_da_3::get_type())`

   用处：tr修改约束；drv注入错误；修改时序

   注：tr_da_3 必须由tr继承而来



## 创建create

------

![pic_ 2022-01-26 15.50.44.jpg](https://github.com/yiliang079/pic/blob/main/pic_%202022-01-26%2015.50.44.jpg?raw=true)



## 覆盖override

------

<img src="https://github.com/yiliang079/pic/blob/main/pic_%202022-01-26%2015.52.29.jpg?raw=true" alt="pic_ 2022-01-26 15.52.29.jpg" style="zoom:33%;" />

类的覆盖

1. `set_type_override_by_type()`
2. `Set_inst_override_by_type()`

eg: `set_type_override_by_type(transaction::get_type(), transaction_da_3::get_type())`

用处：tr修改约束；drv注入错误；修改时序

注：tr_da_3 必须由tr继承而来

替换可以在makefile中完成：

​	`+uvm_set_type_override=transaction,new_transaction`

​	`+uvm_set_inst_override=driver,new_driver`(多一个路径

<img src="https://github.com/yiliang079/pic/blob/main/pic_%202022-01-26%2015.54.25.jpg?raw=true" alt="pic_ 2022-01-26 15.54.25.jpg" style="zoom:33%;" />



## 检查拓扑结构正确性

------

**uvm_top.print_topology()**

​	可以放在final_phase中

​	树形格式 uvm_top.print_topology( uvm_default_tree_printer )	

**检查factory的替换**

​	Factory.print()



## 参数化的组件

------

**必须使用不同的宏定义来注册**

```c++
class new_driver #(width=9) extends uvm_driver #(transaction);
	rand bit [width-1:0] data;
	`uvm_component_param_utils_begin(new_driver #(width))
    `uvm_field_int(data, UVM_DEFAULT)
  `uvm_component_utils_end
endclass
```

可以使用typedef来简化代码的编写

typedef new_driver #() new_driver_type;

<img src="https://github.com/yiliang079/pic/blob/main/pic_%202022-01-26%2016.29.42.jpg?raw=true" alt="pic_ 2022-01-26 16.29.42.jpg" style="zoom:33%;" />

## 组件函数的最佳使用方法

------

​	利用派生的方式拓展组件的功能，使用小的虚方法实现类的操作，可以用多态来重载

<img src="https://github.com/yiliang079/pic/blob/main/pic_%202022-01-26%2016.30.49.jpg?raw=true" alt="pic_ 2022-01-26 16.30.49.jpg" style="zoom:33%;" />