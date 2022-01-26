# config_db

组件显示和查询程序(不怎么用

<img src="https://github.com/yiliang079/pic/blob/main/pic_%202022-01-26%2014.00.28.jpg?raw=true" alt="pic_ 2022-01-26 14.00.28.jpg" style="zoom: 33%;" />

<img src="https://github.com/yiliang079/pic/blob/main/pic_%202022-01-26%2014.03.38.jpg?raw=true" alt="pic_ 2022-01-26 14.03.38.jpg" style="zoom:33%;" />

------

<img src="https://github.com/yiliang079/pic/blob/main/pic_%202022-01-26%2014.07.01.jpg?raw=true" alt="pic_ 2022-01-26 14.07.01.jpg" style="zoom: 33%;" />

**<u>重点记住！！！</u>**

注：用于传递参数必须成对出现

Eg1:

<img src="https://github.com/yiliang079/pic/blob/main/pic_%202022-01-26%2014.10.32.jpg?raw=true" alt="pic_ 2022-01-26 14.10.32.jpg" style="zoom: 33%;" />

以上例子中，

**environment**中的

`uvm_config_db #(int)::set(this, "agnt", "port_id", 10);`

对应**agent**中的

`uvm_config_db #(int)::get(this, "", "port_id", port_id);`

而**agent**中的

`uvm_config_db #(int)::set(this, "*", "port_id", port_id);`

将传递`port_id = 10` 给下属的所有组件，例如`driver` / `sequencer`

另一种设置方法：

直接在**environment**中设置 `agnt.*` 直接设置下属 `port_id` 都为10，但agnt的并不会改变

### 物理接口配置

------

面试题：UVM的interface是怎么连接tb和DUT的？

进行接口的物理连接时不能将virtual interface作为构造函数的参数，必须使用config_db方式，在uvm_test中set，在driver中get。**<u>！必须是virtual！</u>**

**配置组件的DUT接口**

![pic_ 2022-01-26 14.49.08.jpg](https://github.com/yiliang079/pic/blob/main/pic_%202022-01-26%2014.49.08.jpg?raw=true)

driver必须配置好interface，否则仿真将无意义，要加一个判断是否get到吗，否则直接fatal停止仿真。

### 全局UVM资源

------

![pic_ 2022-01-26 14.58.11.jpg](https://github.com/yiliang079/pic/blob/main/pic_%202022-01-26%2014.58.11.jpg?raw=true)

![pic_ 2022-01-26 15.01.39.jpg](https://github.com/yiliang079/pic/blob/main/pic_%202022-01-26%2015.01.39.jpg?raw=true)

`uvm_config_db`配置组件

`uvm_resource_db`全局使用

传递的数据类型必须一样

### 调试

------

1. 在语句前加上``uvm_info`
2. 使用仿真命令选项 （makefile中使用很方便
   1. `+UVM_CONFIG_DB_TRACE`
   2. `+UVM_RESOURCE_DB_TRACE`
3. 在testcase中使用跟踪控制 (很少用
   1. `uvm_config_db_options::turn_tracing_on()`
   2. `uvm_config_db_options::turn_tracing_off()`
   3. `uvm_config_db_options::is_tracing()`
4. 选用 dump()













