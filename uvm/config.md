# config_db

##### overview

<u>全局对象之间的数据共享</u>

入门：[SystemVerilog | UVM | Config_db机制基础 (qq.com)](https://mp.weixin.qq.com/s/dWyGKVE1RS8ztbnXvH0l5Q)

进阶：[SystemVerilog | UVM | 十分钟深入Config_db机制 (qq.com)](https://mp.weixin.qq.com/s/As_OZFMNCZqGD02yvQCQKA)

`Config_db`的全称是Configuration Database，顾名思义，它就是一个数据库。数据库存在的意义，就是提供数据的存储、管理和访问控制功能，`config_db`也不外如此。

`Config_db`是UVM内部的一个以多维数组作为存储结构，并提供了便捷访问接口的全局数据库。UVM有了这个数据库，类似于数字系统中基于共享存储的数据交互，不同环境组件甚至对象之间就可以非常方便地进行数据和句柄的共享，特别是在传递配置对象的时候，我想这也是该数据库被命名为Configuration的原因吧。

`Config_db`所用到的方法定义在`uvm_config_db`这个类中，该类继承自`uvm_resource_db`，可以看成是对其进行了必要的封装，以提供给用户简单易用的接口。这里所说的”简单易用的接口“指的是set(...)函数和get(...)函数，用于向数据库存放数据对象和读取数据对象。此外，`uvm_config_db`提供的主要函数还有exists(...)函数，用于检查某个数据对象是否在库；wait_modified(...)函数，用于耗时等待某个数据对象的更新。所有定义在`uvm_config_db`中的方法都是静态方法，即可以直接通过类名来调用。

当我们要将某个变量、接口句柄或者其他数据对象存储到该数据库的时候，实际上`config_db`会将该数据对象“打包”成一个资源（resource）对象，并打上各种属性标签，比如资源名称（name）、类型（type）、可见范围（scope）和值（value），并以此作为数据库的检索条件。当然，这些属性标签，也是我们在添加数据对象的时候需要给予set(...)函数的参数。

**从函数原型应该可以看到，get函数是返回值的，该返回值可以用来判断是否成功获取到想要的资源对象，这一点将非常有用。**

------

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

### 优先级

------

当不得已需要在环境中的多个地方，对相同数据对象，通过config_db进行配置的时候，就需要关心到这些资源的优先级，即在get时实际上会get到哪一个值或句柄。在config_db这一层（指的是不深入到uvm_resource_db这一层实现上），大概有以下几点需要关注。

第一点是get的优先级。get的机制比较简单，根据cntxt，inst_name，filed_name来找到全局资源池中所有匹配到的资源，并构成资源数组，然后返回该资源数组中优先级最高的资源。这里的优先级（precedence）跟上面提到的name、type、scope和value一样，是每个资源对象拥有的属性。如果优先级一样，就返回排在资源数组最前面的资源！

第二点是非build_phase时的set。不在build_phase函数中set的资源，优先级是一样的（default_precedence），但后set的资源会排在资源数组的最前面！

第三点是build_phase时的set。在build_phase函数中set的资源，优先级会根据cntxt的级数降低（default_precedence - cntxt.get_depth()）。也就是理想使用情况下（这里的理想使用情况指的是在顶层时cntxt参数用null，在其他层cntxt参数用this的情况），越靠近顶层set的资源的优先级更高。

### UVM案例解析

------

![图片](https://mmbiz.qpic.cn/mmbiz_png/YJOEW8ib9oGPNSkTvTYIpibIqBiabVLWqqibref3Pmgickk3VNHmiarB3V71E7Tj9CibMrOYyLfSpS6PMVB7pNKoNqZ7Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



![图片](https://mmbiz.qpic.cn/mmbiz_png/YJOEW8ib9oGPNSkTvTYIpibIqBiabVLWqqib2AJLzItkvqnoAyziagesNfpOkT3fdia8BtRiaEZYYYeI3cMib2cmuZ9fuA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



