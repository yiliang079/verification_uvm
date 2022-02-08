## 寄存器模型RAL

[TOC]

------

#### 主要内容

1. 创建代表寄存器的模型文件register model file
2. 使用RAL generator来创建UVM寄存器类
3. 在序列中使用UVM寄存器
4. 使用适配器adapter为驱动器正确传递UVM寄存器对象
5. 运行UVM内建的寄存器测试案列

------

##### register & memory

- 每一个待测设计都有寄存器和存储器
- 第一个要验证的项目就是寄存器
    - 复位值 reset value
    - 每一个比特的行为 bits behavior
- 可维护性
    - 修改测试案例
    - 修改固件模型 firmware model 

------

##### UVM寄存器概述

步骤：

1. 对每个寄存器进行定义
2. 将寄存器放入对应的address map
3. 创建register adapter
4. 顶层reg block对象的创建和使用
5. 将address map连接到bus sequencer和adapter
6. 在sequence/其他component中使用寄存器模型

UVM的寄存器模型是一组高级抽象的类，用来对DUT中具有地址映射的寄存器和存储器进行建模。它非常贴切的反映DUT中寄存器的各种特性，可以产生激励作用于DUT并进行寄存器功能检查。通过UVM的寄存器模型，可以简单高效的实现对DUT的寄存器进行前门或后门操作。它本身也提供了一些寄存器测试的sequence，方便用户直接使用。

UVM的寄存器模型是高度抽象化的，不依赖具体DUT而独立存在的。它使用一个中间变量uvm_reg_bus_op描述register的访问信息，用户必须创建一个继承自uvm_reg_adapter的类，实现uvm_reg_bus_op与真正作用到具体dut上的transaction的互相转换。
RAL: Register Abstraction Layer

UVM寄存器模型基本结构如下图所示。uvm_reg_block是UVM register layer的类，其层次化的结构反映了DUT的Register状况。uvm_reg_adapter是必不可少的，它实现了bus driver需要的transaction和中间变量uvm_reg_bus_op直间的相互转换。寄存器前门访问是依靠寄存器模型自动产生sequence，并发送给bus driver来完成的。
![UVM Register Model](https://img-blog.csdnimg.cn/20190613153923341.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dvbmRlcl9jb29sZQ==,size_10,color_FFFFFF,t_70)

------

##### 层次结构

- `uvm_reg_field`是寄存器模型的最小单位，和DUT的每个register里的bit filed对应。
- `uvm_reg` 和dut中每个register对应，其宽度一般和总线位宽一致，里面可以包含多个`uvm_reg_field`。
- `uvm_reg_block`里包含`uvm_reg`，一般一个最底层模块级的DUT的所有寄存器，具有相同的基地址，会放在一个`reg_block`中。`uvm_reg_block`内也可包含其他低层次的`reg_block`。
- `reg_block`里有含有`uvm_reg_map`类的对象`default_map`,进行地址映射，以及用来完成寄存器前后门访问操作。
- 一个寄存器模型必须包含一个`reg_block`。
- 一个`reg_block`可以包含多个`reg_map`, 从而实现一个`reg_block`应用到不同总线，或不同地址段上。
- `uvm_mem`是对dut中memory进行建模使用的。
- 这些类均是继承自`uvm_object`类。一个完整的register model, 均有这些层次化的register元素构成，放到顶层的reg block中。一个reg block可以包含子block，register，register file和memories，如下图所示。

![???](https://img-blog.csdnimg.cn/20190613172046504.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dvbmRlcl9jb29sZQ==,size_10,color_FFFFFF,t_70)

------

##### 如何构建寄存器模型？

- 自己写。。
- 用eda
    - **Synopsy VCS中自带的ralgen工具**
    - Candance和Mentor均有自己的工具。
    - Paradigm Works公司开源的 RegWorks Spec2Reg 。
    - 此外还有agnisys公司的IDesignSpec

------

#### 1、寄存器定义

```c++
class cfs_dut_reg_ctrl extends uvm_reg;

   rand uvm_reg_field reserved;  //reserved
   rand uvm_reg_field enable; //control for enabling the DUT

   `uvm_object_utils(cfs_dut_reg_config)

   function new(string name = "cfs_dut_reg_config");
      //specify the name of the register, its width in bits and if it has coverage
      super.new(name, 32, 1);
   endfunction

   virtual function void build();
      reserved = uvm_reg_field::type_id::create("reserved");
      //specify parent, width, lsb position, rights, volatility,
      //reset value, has reset, is_rand, individually_accessible
      reserved.configure(this, 31, 1, "RO", 0, 0, 1, 1, 1);

      enable = uvm_reg_field::type_id::create("enable");
      enable.configure(this, 1, 0, "RW", 0, 0, 1, 1, 1);
   endfunction
endclass

```

- new()函数
- 要将寄存器的宽度传入super.new()的第二个参数，super.new()的第三个参数是`uvm_coverage_model_e`类型,用以设置寄存器是否参与加入覆盖率：

|              value | description                            |
| -----------------: | -------------------------------------- |
|    UVM_NO_COVERAGE | None                                   |
|   UVM_CVR_REG_BITS | individual register bits               |
|   UVM_CVR_ADDR_MAP | Individual register and memory address |
| UVM_CVR_FIELD_VALS | Field values                           |
|        UVM_CVR_ALL | all coverage models                    |

-  **build函数**，这个build和UVM_component的bulid_phase并不一样，并不会自动执行，**需要手动调用**。

使用uvm_field的configure函数对field配置

```c++
enable.configure( .parent              ( this ),
               .size                   ( 3    ),
               .lsb_pos                ( 0    ),
               .access                 ( "RW" ),
               .volatile               ( 0    ),
               .reset                  ( 0    ),
               .has_reset              ( 1    ),
               .is_rand                ( 1    ),
               .individually_accessible( 0    ) );
/*
 参数一是此域的父辈，也就是此域位于哪个寄存器中，即是this；
 参数二是此域的宽度；
 参数三是此域的最低位在整个寄存器的位置，从0开始计数；
 参数四表示此字段的存取方式；
 参数五表示是否是易失的（volatile），这个参数一般不会使用；
 参数六表示此域上电复位后的默认值；
 参数七表示此域时都有复位；
 参数八表示这个域是否可以随机化；
 参数九表示这个域是否可以单独存取。
*/
```

------

#### 2、将寄存器放入register block中，并加入对应的address map

- 使用`uvm_reg_block`
- reg_block中也有**build函数**，在其中要做如下事情：

1. 调用`create_map`函数完成`default_map`的实例化,

    `default_map = create_map(“default_map”,0,2,UVM_BIG_ENDINA,0);`

    create_map的**第一个参数是名字**，**第二个参数是该reg block的基地址**，**第三个参数是寄存器所映射到的总线的宽度**(单位是byte，不是bit)，**第四个参数是大小端**，**第五个参数表示该寄存器能否按byte寻址**。

2. 完成每个寄存器的 build 及 configure 操作。`uvm_reg`的configure函数原型： 

    `function void configure ( uvm_reg_block blk_parent, uvm_reg_file regfile_parent = null, string hdl_path = ""` 

    其第一个参数是所在reg block的指针，第二个参数是reg_file指针，第三个是寄存器后面访问路径—string类型。

3. 把每个寄存器加入到default_map中。uvm_reg_map存有各个寄存器的地址信息。`default_map.add_reg(ctrl, `h10,"RW")`  第一个参数是要添加的寄存器名，第二个是地址，第三个是寄存器的读写属性。

4. 如果一个寄存器可以通过两个物理总线访问，则需要将其添加到多个address map中。

```c++
class cfs_dut_reg_block extends uvm_reg_block;
   `uvm_object_utils(cfs_dut_reg_block)

   //Control register
   rand cfs_dut_reg_ctrl ctrl;
   //Status register
   rand cfs_dut_reg_status status;

   function new(string name = "cfs_dut_reg_block");
      super.new(name, UVM_CVR_ALL);
      ctrl = cfs_dut_reg_ctrl::type_id::create("ctrl");
      status = cfs_dut_reg_status::type_id::create("status");
   endfunction

   virtual function void build();
      default_map = create_map(“default_map”,0,4,UVM_BIG_ENDINA,0);
      
      ctrl.configure(this, null, "");
      ctrl.build();
      default_map.add_reg(ctrl,`h10,"RW")

      status.configure(this, null, "");
      status.build();
      default_map.add_reg(status,`h14,"RO")

   endfunction
endclass

```

示例代码

------

#### 3、创建regsiter adapter

before：

前门访问和后门访问：

- **前门访问**是通过物理总线向dut发起寄存器访问操作，消耗仿真时间
- **后面操作**不通过物理总线，不消耗仿真时间

**前门访问过程：**write

1. 当调用寄存器的write()任务后，产生uvm_reg_item类型的transaction:rw，之后调用uvm_reg::do_write()。
2. 在uvm_reg_map中，调用reg_adapter.reg2bus将rw转换成bus driver对应的transaction。
3. 把transaction交给sequencer，最终由bus driver驱动到对应的bus interface上。
4. bus monitor在bus interface上检测到bus transaction 。
5. reg_predictor会调用reg_adapter.bus2reg将该bus transaction转换成uvm_reg_item。
6. 从driver中返回的req会转换成uvm_reg_item类型，如防止sequencer的response队列溢出，需要在adapter中设置provides_reponses.
7. 寄存器模型根据返回的uvm_reg_item来更新寄存器的value，m_mirrored和m_desired三个值

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190614184540451.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dvbmRlcl9jb29sZQ==,size_16,color_FFFFFF,t_70)

back：

寄存器模型的前门操作都会通过sequence产生一个`uvm_reg_bus_op`类型的变量，他不能直接被bus sequencer和driver接受。需要定义一个继承自`uvm_reg_adpater`的adapter，来完成与bus transaction之间的转换，之后才能交给交给bus_sequencer和bus_driver，实现前门访问。 而从bus_driver返回的rsp,也需要由adapter转换成`uvm_reg_bus_op`类型变量，返回给寄存器模型，用来更新内部值。
在adapter中，要实现:

1. `reg2bus`: 其作用是将`uvm_reg_bus_op`类型变量转换成`bus_sequencer`能够接受的transaction。
  `virtual function uvm_sequence_item reg2bus(const ref uvm_reg_bus_op rw);`

  

2. `bus2reg`: 其将收集到的transaction转换成寄存器模型使用的uvm_reg_bus_op类型变量，用以更新寄存器模型中相应寄存器的值。
virtual function void bus2reg(uvm_sequence_item bus_item, ref uvm_reg_bus_op rw);

```c++
class cfs_dut_reg_adapter extends uvm_reg_adapter;
   `uvm_object_utils(cfs_dut_reg_adapter)

   function new(string name = "cfs_dut_reg_adapter");
      super.new(name);
   endfunction
        
   virtual function uvm_sequence_item reg2bus(const ref uvm_reg_bus_op rw);
      acme_apb_drv_transfer transfer = acme_apb_drv_transfer::type_id::create("transfer");

      if(rw.kind == UVM_WRITE) begin
         transfer.direction = APB_WRITE;
      end
      else begin
         transfer.direction = APB_READ;
      end 
            
      transfer.data = rw.data;
      transfer.address = rw.addr;
      
      return transfer;
   endfunction
        
   virtual function void bus2reg(uvm_sequence_item bus_item, ref uvm_reg_bus_op rw);
      acme_apb_mon_transfer transfer;
            
      if($cast(transfer, bus_item)) begin
         if(transfer.direction == APB_WRITE) begin
            rw.kind = UVM_WRITE;
         end
         else begin
            rw.kind = UVM_READ;
         end

         rw.addr = transfer.address;
         rw.data = transfer.data;
         rw.status = UVM_IS_OK;
      end
      else begin
         `uvm_fatal(get_name(), $sformatf("Could not cast to acme_apb_mon_transfer: %s", 
            bus_item.get_type_name()))
      end
   endfunction
endclass

```

------

#### 4、顶层reg_block对象的创建和使用

**整个仿真平台只创建一个reg model对象，在其他地方使用指针调用。一般在test中创建顶层reg_block，及adapt和predictor.**

reg_block传完要调用其configure函数，配置后面访问路径。

```c++
   // in  base test class
   ...
   irtual function void build_phase(uvm_phase phase);
      super.build_phase(phase);
      
      //create all the elements of the environment
      reg_block = cfs_dut_reg_block::type_id::create("reg_block",this);
      apb_agent = acme_apb_agent::type_id::create("apb_agent", this);
      adapter = cfs_dut_reg_adapter::type_id::create("adapter");
      predictor = uvm_reg_predictor#(acme_apb_mon_transfer)::type_id::create("predictor", this);
      reg_block.configure(null,"");
      reg_block.build();
      reg_block.lock_model();
      reg_block.reset("HARD");
   endfunction
   ...

```

#### 5、将address map连接到bus sequencer和adapter

在test或env的connect phase中,调用default_map或其他用户自定义的address map对象中的set_sequencer方法, 并把前门操作的bus sequencer及adaptor作为参数传入。

```c++
virtual function void connect_phase(uvm_phase phase);
      super.connect_phase(phase);
      
      //required to start physical register accesses using the registers
      reg_block.default_map.set_sequencer(apb_agent.sequencer, adapter);
      
      predictor.map = reg_block.default_map;
      predictor.adapter = adapter;
      apb_agent.monitor.output_port.connect(predictor.bus_in);
   endfunction
```

#### 6、在sequence或其他component中使用寄存器模型

在sequence中使用：
要先在对应的sequencer中定义一个顶层reg_block的指针，并指向base_test的reg_block对应，之后再sequence中调用p_sequencer访问，如：

- `p_sequencer.p_reg_block.enable.wirte(status,1,UVM_FRONTDOOR);`

     // 前门访问 front-door

- `p_sequencer.p_reg_block.enable.re'a'd(status,value,UVM_BACKDOOR);`

    // 后门访问 back-door

    