# **lab3_code_guide**

[TOC]



## testbench

### testbench topology

------

![IMG_4720.jpg](https://github.com/yiliang079/pic/blob/main/IMG_4720.jpg?raw=true)

### <u>**test.sv**</u>

```verilog
program automatic test;
import uvm_pkg::*;
import router_test_pkg::*;

initial begin
  uvm_resource_db#(virtual router_io)::set("router_vif", "", router_test_top.router_if);
  uvm_resource_db#(virtual reset_io)::set("reset_vif", "", router_test_top.reset_if);
  $timeformat(-9, 1, "ns", 10);
  run_test();
end

endprogram

```

### <u>**test_collection.sv**</u>

##### class test_base

```c++
class test_base extends uvm_test;
  `uvm_component_utils(test_base)

  router_env env;
  virtual router_io router_vif;
  virtual reset_io  reset_vif;
```

###### new()

```c++
  function new(string name, uvm_component parent);
    super.new(name, parent);
    `uvm_info("TRACE", $sformatf("%m"), UVM_HIGH);
  endfunction: new
```

###### build_phase()

```c++
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    `uvm_info("TRACE", $sformatf("%m"), UVM_HIGH);
    env = router_env::type_id::create("env", this);

    uvm_resource_db#(virtual router_io)::read_by_type("router_vif", router_vif, this);
    uvm_resource_db#(virtual reset_io)::read_by_type("reset_vif", reset_vif, this);

    uvm_config_db#(virtual router_io)::set(this, "env.i_agt", "vif", router_vif);
    uvm_config_db#(virtual reset_io)::set(this, "env.r_agt", "vif", reset_vif);

  endfunction: build_phase
```

###### final_phase()

```c++
  endfunction: build_phase

  virtual function void final_phase(uvm_phase phase);
    super.final_phase(phase);
    `uvm_info("TRACE", $sformatf("%m"), UVM_HIGH);

    if (uvm_report_enabled(UVM_MEDIUM, UVM_INFO, "TOPOLOGY")) begin
      uvm_root::get().print_topology();
    end

    if (uvm_report_enabled(UVM_MEDIUM, UVM_INFO, "FACTORY")) begin
      uvm_factory::get().print();
    end
  endfunction: final_phase
      //////end
endclass: test_base
```

##### class test_da_3_type

```c++
class test_da_3_inst extends test_base;
  `uvm_component_utils(test_da_3_inst)
```

###### new()

```c++
function new(string name, uvm_component parent);
    super.new(name, parent);
    `uvm_info("TRACE", $sformatf("%m"), UVM_HIGH);
  endfunction: new
```

###### build_phase()

```c++
virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    `uvm_info("TRACE", $sformatf("%m"), UVM_HIGH);
    set_inst_override_by_type("env.i_agt.sqr.*", packet::get_type(), packet_da_3::get_type());
  endfunction: build_phase
      ///////////end
endclass: test_da_3_inst
```

##### class test_da_3_seq

```c++
class test_da_3_seq extends test_base;
  `uvm_component_utils(test_da_3_seq)
```

###### new()

```c++
  function new(string name, uvm_component parent);
    super.new(name, parent);
    `uvm_info("TRACE", $sformatf("%m"), UVM_HIGH);
  endfunction: new
```

###### build_phase()

```c++
virtual function void build_phase(uvm_phase phase);
    packet_sequence::int_q_t valid_da = {3};
    super.build_phase(phase);
    `uvm_info("TRACE", $sformatf("%m"), UVM_HIGH);
    uvm_config_db#(packet_sequence::int_q_t)::set(this, "env.i_agt.sqr.packet_sequence", "valid_da", valid_da);
    uvm_config_db#(int)::set(this, "env.i_agt.sqr.packet_sequence", "item_count", 20);
  endfunction: build_phase
      ///////////////end
endclass: test_da_3_seq
```

### **<u>router_env.sv</u>**

##### class router_env

```c++
class router_env extends uvm_env;
  input_agent i_agt;
  reset_agent r_agt;

  `uvm_component_utils(router_env)
```

###### new()

```c++
function new(string name, uvm_component parent);
    super.new(name, parent);
    `uvm_info("TRACE", $sformatf("%m"), UVM_HIGH);
  endfunction: new
```

###### build_phase()

```c++
virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    `uvm_info("TRACE", $sformatf("%m"), UVM_HIGH);

    i_agt = input_agent::type_id::create("i_agt", this);
    uvm_config_db #(uvm_object_wrapper)::set(this, "i_agt.sqr.main_phase", "default_sequence", packet_sequence::get_type());
    
    r_agt = reset_agent::type_id::create("r_agt", this);
    uvm_config_db #(uvm_object_wrapper)::set(this, "r_agt.sqr.reset_phase", "default_sequence", reset_sequence::get_type());

  endfunction: build_phase
////////////////////end
endclass: router_env
```

### input_agent.sv

##### class input_agent

```c++
class input_agent extends uvm_agent;
  `uvm_component_utils(input_agent)

  typedef uvm_sequencer #(packet) packet_sequencer;

  virtual router_io vif;
  packet_sequencer  sqr;
  driver            drv;
  int               port_id = -1;
```

###### new()

```c++
function new(string name, uvm_component parent);
    super.new(name, parent);
    `uvm_info("TRACE", $sformatf("%m"), UVM_HIGH);
  endfunction: new
```

###### build_phase()

```c++
  virtual function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    `uvm_info("TRACE", $sformatf("%m"), UVM_HIGH);

    sqr = packet_sequencer::type_id::create("sqr", this);
    drv = driver::type_id::create("drv", this);

    uvm_config_db#(int)::get(this, "", "port_id", port_id);
    uvm_config_db#(virtual router_io)::get(this, "", "vif", vif);

    uvm_config_db#(int)::set(this, "*", "port_id", port_id);
    uvm_config_db#(virtual router_io)::set(this, "*", "vif", vif);
  endfunction: build_phase
```

###### connect_phase()

```c++
  virtual function void connect_phase(uvm_phase phase);
    super.connect_phase(phase);
    `uvm_info("TRACE", $sformatf("%m"), UVM_HIGH);

    drv.seq_item_port.connect(sqr.seq_item_export);
  endfunction: connect_phase
```

###### end_of_elaboration_phase()

```c++
  virtual function void end_of_elaboration_phase(uvm_phase phase);
    super.end_of_elaboration_phase(phase);
    `uvm_info("TRACE", $sformatf("%m"), UVM_HIGH);

    if (!(port_id inside {-1, [0:15]})) begin
      `uvm_fatal("CFGERR", $sformatf("port_id must be {-1, [0:15]}, not %0d!", port_id));
    end
    if (vif == null) begin
      `uvm_fatal("CFGERR", "Interface for input agent not set");
    end
  endfunction: end_of_elaboration_phase
      /////////////end
endclass: input_agent
```































## RTL

------

