# factory

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

检查