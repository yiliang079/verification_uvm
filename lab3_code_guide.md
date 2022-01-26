# **lab3_code_guide**

[TOC]



## testbench

### testbench topology

------

<img src="https://github.com/yiliang079/pic/blob/main/IMG_4720.jpg?raw=true" alt="IMG_4720.jpg" style="zoom:75%;" />

### **test.sv**

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

### **test_collection.sv**

------

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



### **router_env.sv**

------

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

------

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



### driver.sv

------

##### class driver

```c++
class driver extends uvm_driver #(packet);
  virtual router_io vif;           // DUT virtual interface
  int               port_id = -1;  // Driver's designated port
    `uvm_component_utils_begin(driver)
    `uvm_field_int(port_id, UVM_ALL_ON | UVM_DEC)
  `uvm_component_utils_end
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
  function void build_phase(uvm_phase phase);
    super.build_phase(phase);
    `uvm_info("TRACE", $sformatf("%m"), UVM_HIGH);
    
    uvm_config_db#(int)::get(this, "", "port_id", port_id);
    uvm_config_db#(virtual router_io)::get(this, "", "vif", vif);

  endfunction: build_phase
```

###### end_of_elaboration_phase()

```c++
  function void end_of_elaboration_phase(uvm_phase phase);
    super.end_of_elaboration_phase(phase);
    `uvm_info("TRACE", $sformatf("%m"), UVM_HIGH);

    if (!(port_id inside {-1, [0:15]})) begin
      `uvm_fatal("CFGERR", $sformatf("port_id must be {-1, [0:15]}, not %0d!", port_id));
    end
    if (vif == null) begin
      `uvm_fatal("CFGERR", "Interface for Driver not set");
    end

  endfunction: end_of_elaboration_phase
```

###### start_of_simulation_phase()

```c++
  virtual function void start_of_simulation_phase(uvm_phase phase);
    super.start_of_simulation_phase(phase);
    `uvm_info("TRACE", $sformatf("%m"), UVM_HIGH);
    `uvm_info("DRV_CFG", $sformatf("port_id is: %0d", port_id), UVM_MEDIUM);
  endfunction: start_of_simulation_phase
```

###### run_phase()

```c++
    `uvm_info("TRACE", $sformatf("%m"), UVM_HIGH);

    forever begin
      seq_item_port.get_next_item(req);

      if (port_id inside { -1, req.sa }) begin
        send(req);
        `uvm_info("DRV_RUN", {"\n", req.sprint()}, UVM_MEDIUM);
      end

      seq_item_port.item_done();
    end
  endtask: run_phase
```

###### send()

```c++
  virtual task send(packet tr);
    `uvm_info("TRACE", $sformatf("%m"), UVM_HIGH);
    send_address(tr);
    send_pad(tr);
    send_payload(tr);
  endtask: send
```

###### send_addredd()

```c++
  virtual task send_address(packet tr);
    `uvm_info("TRACE", $sformatf("%m"), UVM_HIGH);
    vif.drvClk.frame_n[tr.sa] <= 1'b0;
    for(int i=0; i<4; i++) begin
      vif.drvClk.din[tr.sa] <= tr.da[i];
      @(vif.drvClk);
    end
  endtask: send_address
```

###### send_pad()

```c++
  virtual task send_pad(packet tr);
    `uvm_info("TRACE", $sformatf("%m"), UVM_HIGH);
    vif.drvClk.din[tr.sa] <= 1'b1;
    vif.drvClk.valid_n[tr.sa] <= 1'b1;
    repeat(5) @(vif.drvClk);
  endtask: send_pad
```

###### send_payload()

```c++
  virtual task send_payload(packet tr);
    logic [7:0] datum;
    `uvm_info("TRACE", $sformatf("%m"), UVM_HIGH);
    while(!vif.drvClk.busy_n[tr.sa]) @(vif.drvClk);
    foreach(tr.payload[index]) begin
      datum = tr.payload[index];
      for(int i=0; i<$size(tr.payload, 2); i++) begin
        vif.drvClk.din[tr.sa] <= datum[i];
        vif.drvClk.valid_n[tr.sa] <= 1'b0;
        vif.drvClk.frame_n[tr.sa] <= ((tr.payload.size()-1) == index) && (i==7);
        @(vif.drvClk);
      end
    end
    vif.drvClk.valid_n[tr.sa] <= 1'b1;
  endtask: send_payload
  //////////end
  endclass: driver
```



### uvm_sequencer.sv

------

###### Just typedef

省略，直接`typedef uvm_sequencer #(packet) packet_sequencer;`



### packet_sequence.sv

------

##### class packet_sequence_base 

```c++
class packet_sequence_base extends uvm_sequence #(packet);
  `uvm_object_utils(packet_sequence_base)
```

###### new()

```c++
  function new(string name = "packet_sequence_base");
    super.new(name);
    `uvm_info("TRACE", $sformatf("%m"), UVM_HIGH);

    `ifndef UVM_VERSION_1_1
     set_automatic_phase_objection(1);
    `endif
  endfunction: new
```

###### if UVM_VERSION_1_1

```c++
  `ifdef UVM_VERSION_1_1
  virtual task pre_start();
    if ((get_parent_sequence() == null) && (starting_phase != null)) begin
      starting_phase.raise_objection(this);
    end
  endtask: pre_start

  virtual task post_start();
    if ((get_parent_sequence() == null) && (starting_phase != null)) begin
      starting_phase.drop_objection(this);
    end
  endtask: post_start
  `endif
endclass: packet_sequence_base
```

##### class packet_sequencec

```c++
class packet_sequence extends packet_sequence_base;
  int       item_count = 10;
  int       port_id    = -1;
  int       valid_da[$];

  typedef int int_q_t[$];

  `uvm_object_utils_begin(packet_sequence)
    `uvm_field_int(item_count, UVM_ALL_ON)
    `uvm_field_queue_int(valid_da, UVM_ALL_ON)
    `uvm_field_int(port_id, UVM_ALL_ON)
  `uvm_object_utils_end
```

###### pre_start()

```c++
  virtual task pre_start();
    super.pre_start();

    uvm_config_db#(int)::get(get_sequencer(), get_type_name(), "item_count", item_count);
    uvm_config_db#(int_q_t)::get(get_sequencer(), get_type_name(), "valid_da", valid_da);
    uvm_config_db#(int)::get(get_sequencer().get_parent(), "", "port_id",port_id);
    if (!(port_id inside {-1, [0:15]})) begin
      `uvm_fatal("CFGERR", $sformatf("Illegal port_id value of %0d", port_id));
    end
  endtask: pre_start
```

###### new()

```c++
  function new(string name = "packet_sequence");
    super.new(name);
    `uvm_info("TRACE", $sformatf("%m"), UVM_HIGH);
    for (int i=0; i<16; i++) begin
      valid_da.push_back(i);    // Set the default to enable all ports.
    end
  endfunction: new
```

###### body()

```c++
  virtual task body();
    `uvm_info("TRACE", $sformatf("%m"), UVM_HIGH);

    repeat(item_count) begin

      `ifndef UVM_VERSION
        `uvm_do_with(req, {if (port_id == -1) sa inside {[0:15]}; else sa == port_id; da inside valid_da;});
      // For IEEE UVM
      `else
        `uvm_do(req,,, {if (port_id == -1) sa inside {[0:15]}; else sa == port_id; da inside valid_da;});
      `endif

    end
 endtask: body
```

```c++
endclass: packet_sequence
```



### packet.sv

------

##### class packet

```c++
class packet extends uvm_sequence_item;

  rand bit [3:0] sa, da;
  rand bit[7:0] payload[$];

  `uvm_object_utils_begin(packet)
    `uvm_field_int(sa, UVM_ALL_ON | UVM_NOCOMPARE)
    `uvm_field_int(da, UVM_ALL_ON)
    `uvm_field_queue_int(payload, UVM_ALL_ON)
  `uvm_object_utils_end
```

###### constraint{}

```c++
  constraint valid 
  {
    payload.size inside {[1:10]};
  }
```

###### new()

```c++
  function new(string name = "packet");
    super.new(name);
    `uvm_info("TRACE", $sformatf("%m"), UVM_HIGH);
  endfunction: new
///////////end
endclass: packet
```



## reset

------

#### reset_agent.sv 

```c++
typedef class reset_driver;
typedef class reset_monitor;
```

##### class reset_agent

```c++
class reset_agent extends uvm_agent;
  typedef uvm_sequencer#(reset_tr) reset_sequencer;

  virtual reset_io vif;          // DUT virtual interface
  reset_sequencer  sqr;
  reset_driver     drv;
  reset_monitor    mon;
  
  `uvm_component_utils(reset_agent)
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
    `uvm_info("RSTCFG", $sformatf("Reset agent %s setting for is_active is: %p", this.get_name(), is_active), UVM_MEDIUM);

    uvm_config_db#(virtual reset_io)::get(this, "", "vif", vif);
    uvm_config_db#(virtual reset_io)::set(this, "*", "vif", vif);

    if (is_active == UVM_ACTIVE) begin
      sqr = reset_sequencer::type_id::create("sqr", this);
      drv = reset_driver::type_id::create("drv", this);
    end
    mon = reset_monitor::type_id::create("mon", this);
  endfunction: build_phase
```

###### connect_phase()

```c++
  virtual function void connect_phase(uvm_phase phase);
    super.connect_phase(phase);
    `uvm_info("TRACE", $sformatf("%m"), UVM_HIGH);
    if (is_active == UVM_ACTIVE) begin
      drv.seq_item_port.connect(sqr.seq_item_export);
    end
  endfunction: connect_phase
```

###### end_of_elaboration_phase()

```c++
  virtual function void end_of_elaboration_phase(uvm_phase phase);
    super.end_of_elaboration_phase(phase);
    `uvm_info("TRACE", $sformatf("%m"), UVM_HIGH);
    if (vif == null) begin
      `uvm_fatal("CFGERR", "Interface for reset agent not set");
    end
  endfunction: end_of_elaboration_phase
////////////end
endclass
```

##### class resset_driver

```c++
class reset_driver extends uvm_driver #(reset_tr);
  virtual reset_io vif;          // DUT virtual interface
  `uvm_component_utils(reset_driver)
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

    uvm_config_db#(virtual reset_io)::get(this, "", "vif", vif);
  endfunction: build_phase
```

###### end_of_elaboration_phase()

```c++
  virtual function void end_of_elaboration_phase(uvm_phase phase);
    super.end_of_elaboration_phase(phase);
    `uvm_info("TRACE", $sformatf("%m"), UVM_HIGH);
    if (vif == null) begin
      `uvm_fatal("CFGERR", "Interface for reset driver not set");
    end
  endfunction: end_of_elaboration_phase
```

###### run_phase()

```c++
  virtual task run_phase(uvm_phase phase);
    `uvm_info("TRACE", $sformatf("%m"), UVM_HIGH);

    forever begin
      seq_item_port.get_next_item(req);
      drive(req);
      seq_item_port.item_done();
    end
  endtask: run_phase
```

###### task driver()

```c++
  virtual task drive(reset_tr tr);
    `uvm_info("TRACE", $sformatf("%m"), UVM_HIGH);
    if (tr.kind == reset_tr::ASSERT) begin
      vif.reset_n = 1'b0;
      repeat(tr.cycles) @(vif.mst);
    end else begin
      vif.reset_n <= '1;
      repeat(tr.cycles) @(vif.mst);
    end
  endtask: drive
endclass
```

##### class reset_monitor

```c++
class reset_monitor extends uvm_monitor;
  virtual reset_io vif;          // DUT virtual interface
  uvm_analysis_port #(reset_tr) analysis_port;
  uvm_event reset_event = uvm_event_pool::get_global("reset");
  `uvm_component_utils(reset_monitor)
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

    uvm_config_db#(virtual reset_io)::get(this, "", "vif", vif);
    analysis_port = new("analysis_port", this);
  endfunction: build_phase
```

###### end_of_elaboration_phase()

```c++
  virtual function void end_of_elaboration_phase(uvm_phase phase);
    super.end_of_elaboration_phase(phase);
    `uvm_info("TRACE", $sformatf("%m"), UVM_HIGH);
    if (vif == null) begin
      `uvm_fatal("CFGERR", "Interface for reset monitor not set");
    end
  endfunction: end_of_elaboration_phase
```

###### run_phase()

```c++
  virtual task run_phase(uvm_phase phase);
    `uvm_info("TRACE", $sformatf("%m"), UVM_HIGH);

    forever begin
      reset_tr tr = reset_tr::type_id::create("tr", this);
      detect(tr);
      analysis_port.write(tr);
    end
  endtask: run_phase
```

###### task detect()

```c++
  virtual task detect(reset_tr tr);
    `uvm_info("TRACE", $sformatf("%m"), UVM_HIGH);
    @(vif.reset_n);
    assert(!$isunknown(vif.reset_n));
    if (vif.reset_n == 1'b0) begin
      tr.kind = reset_tr::ASSERT;
      reset_event.trigger();
    end else begin
      tr.kind = reset_tr::DEASSERT;
      reset_event.reset(.wakeup(1));
    end
  endtask: detect
      /////////////////end
endclass
```



#### reset_sequence.sv

------

##### class reset_sequence

```c++
class reset_sequence extends uvm_sequence #(reset_tr);
  `uvm_object_utils(reset_sequence)
```

###### new()

```c++
  function new(string name = "reset_sequence");
    super.new(name);
    `uvm_info("TRACE", $sformatf("%m"), UVM_HIGH);

    `ifndef UVM_VERSION_1_1
     set_automatic_phase_objection(1);
    `endif
  endfunction: new
```

###### body()

```c++
  virtual task body();
    `uvm_info("TRACE", $sformatf("%m"), UVM_HIGH);
        `ifndef UVM_VERSION
      `uvm_do_with(req, {kind == DEASSERT; cycles == 2;});
      `uvm_do_with(req, {kind == ASSERT; cycles == 1;});
      `uvm_do_with(req, {kind == DEASSERT; cycles == 15;});
    
    // For IEEE UVM
    
    `else
      `uvm_do(req,,, {kind == DEASSERT; cycles == 2;});
      `uvm_do(req,,, {kind == ASSERT; cycles == 1;});
      `uvm_do(req,,, {kind == DEASSERT; cycles == 15;});
    `endif

  endtask: body
```

###### **if UVM_VERSION_1_1**

------

###### pre_start()

```c++
  virtual task pre_start();
    `uvm_info("TRACE", $sformatf("%m"), UVM_HIGH);
    if ((get_parent_sequence() == null) && (starting_phase != null)) begin
      starting_phase.raise_objection(this);
    end
  endtask: pre_start
```

###### post_start()

```c++
  virtual task post_start();
    `uvm_info("TRACE", $sformatf("%m"), UVM_HIGH);
    if ((get_parent_sequence() == null) && (starting_phase != null)) begin
      starting_phase.drop_objection(this);
    end
  endtask: post_start
  `endif

endclass: reset_sequence
```



#### reset_tr.sv

------

##### class reset_tr

```c++
class reset_tr extends uvm_sequence_item;
  typedef enum {ASSERT, DEASSERT} kind_e;
  rand kind_e kind;
  rand int unsigned cycles = 1;

  `uvm_object_utils_begin(reset_tr)
    `uvm_field_enum(kind_e, kind, UVM_ALL_ON)
    `uvm_field_int(cycles, UVM_ALL_ON)
  `uvm_object_utils_end
```

###### new()

```c++
  function new(string name = "reset_tr");
    super.new(name);
    `uvm_info("TRACE", $sformatf("%m"), UVM_HIGH);
  endfunction: new
endclass
```



## RTL

------

###### router_test_top.sv

```verilog
module router_test_top;
  parameter simulation_cycle = 100 ;
  bit  SystemClock;

  reset_io  reset_if(SystemClock);
  router_io router_if(SystemClock);
  host_io   host_if(SystemClock);
  router    dut(.clk(SystemClock), .reset_n(reset_if.reset_n), .io(router_if), .host(host_if));

  initial begin
    $fsdbDumpvars;
    forever #(simulation_cycle/2) SystemClock = ~SystemClock ;
  end
endmodule  
```

###### reset_io.sv

```verilog
`ifndef RESET_IO__SV
`define RESET_IO__SV

interface reset_io(input logic clk);
  logic        reset_n;

  clocking mst @(posedge clk);
    output reset_n;
  endclocking

  clocking mon @(posedge clk);
    input reset_n;
  endclocking

  modport dut(input reset_n);
endinterface: reset_io
`endif
```

###### router_io.sv

```verilog
`ifndef ROUTER_IO__SV
`define ROUTER_IO__SV

interface router_io(input bit clk);

   logic [15:0] frame_n ;
   logic [15:0] valid_n ;
   logic [15:0] din ;
   logic [15:0] dout ;
   logic [15:0] busy_n ;
   logic [15:0] valido_n ;
   logic [15:0] frameo_n ;

   clocking drvClk @(posedge clk);
      output  frame_n;
      output  valid_n;
      output  din;
      input   busy_n;
   endclocking: drvClk

   clocking iMonClk @(posedge clk);
      input  frame_n;
      input  valid_n;
      input  din;
      input  busy_n;
   endclocking: iMonClk

   clocking oMonClk @(posedge clk);
      input  dout;
      input  valido_n;
      input  frameo_n;
   endclocking: oMonClk

   modport dut(input clk, frame_n, valid_n, din, output dout, busy_n, valido_n, frameo_n);

endinterface: router_io

`endif
```

###### router.sv

```verilog
module router(input logic clk, reset_n, router_io.dut io, host_io.dut host);

parameter host_reg_base_address       = 'h1000;
parameter counter_base_address        = 'h2000;
parameter register_file_base_address  = 'h3000;
parameter ram_base_address            = 'h4000;

logic [15:0] din_reg, frame_reg, valid_reg;

wire [15:0] request0_n, request1_n, request2_n, request3_n, request4_n, request5_n, request6_n, request7_n, request8_n, request9_n, request10_n, request11_n, request12_n, request13_n, request14_n, request15_n;

wire [15:0] grant0_n, grant1_n, grant2_n, grant3_n, grant4_n, grant5_n, grant6_n, grant7_n, grant8_n, grant9_n, grant10_n, grant11_n, grant12_n, grant13_n, grant14_n, grant15_n;


wire [15:0] request_n[16], grant_n[16];

wire [3:0] ip_src_op[16];

wire [15:0] deassert, frameo_int;

logic        enable;
logic [15:0] data_out;
logic [15:0] lock;
logic [15:0] read2clear;
logic [15:0] host_reg['h100];
logic [15:0] counters['h20];
logic [15:0] ram['h1000];

logic [15:0] frame_n_previous, frameo_n_previous;
wire  [15:0] framei_complete, frameo_complete;

const logic [7:0] rev_id = 8'h03;
const logic [7:0] chip_id = 8'h5a;
const logic [15:0] host_id = { chip_id, rev_id };

assign framei_complete = ~frame_n_previous & io.frame_n;
assign frameo_complete = ~frameo_n_previous & io.frameo_n;

assign host.data = (!host.rd_n && enable)? data_out:'z;

//always_ff @(posedge io.clk) begin
always_ff @(posedge clk) begin
  frame_n_previous  <= io.frame_n;
  frameo_n_previous <= io.frameo_n;
end

//always_ff @(posedge io.clk or negedge reset_n) begin
always_ff @(posedge clk or negedge reset_n) begin
  if (!reset_n) begin
    for (int i=0; i<$size(counters); i++) begin
      counters[i] <= '0;
    end
  end
  else begin
    if (!host.rd_n && (host.address inside {['h2000:'h201f]})) begin
      counters[(host.address - counter_base_address) & 'h001f] <= 0;
    end else begin
      for (int i=0; i<16; i++) begin
        if (framei_complete[i]) counters[i] <= counters[i] + 1;
      end
      for (int i=0; i<16; i++) begin
        if (frameo_complete[i]) counters['h10 + i] <= counters['h10 + i] + 1;
      end
    end
  end
end

always_ff @(posedge clk or negedge reset_n) begin
  if (!reset_n) begin
// Need to be '1 for RAL lab
    lock       <= '0;
    read2clear <= '1;
    foreach (host_reg[i])
      host_reg[i] <= '0;
  end else begin
    if (!host.wr_n) begin
      casex(host.address)
        'h0100: lock <= ~host.data & lock;
        'h10??: host_reg[host.address - host_reg_base_address] <= host.data;
        'h4???: ram[host.address - ram_base_address] <= host.data;
      endcase
    end
    if (!host.rd_n & (host.address == 'h0101)) begin
      read2clear <= '0;
    end else begin
      read2clear <= read2clear + 1;
    end
  end
end

always_comb begin
  if (!host.rd_n) begin
    enable = 0;
    case(host.address) inside
      'h0000:         begin data_out = host_id; enable = 1; end
      'h0100:         begin data_out = lock; enable = 1; end
      'h0101:         begin data_out = read2clear; enable = 1; end
      'h10??:         begin data_out = host_reg[(host.address - host_reg_base_address) & 'h00ff]; enable = 1; end
      'h200?, 'h201?: begin data_out = counters[(host.address - counter_base_address) & 'h001f]; enable = 1; end
      'h4???:         begin data_out = ram[host.address - ram_base_address]; enable = 1; end
    endcase
  end
end



iport #(.port_number(0)) ip0(clk, reset_n, io.din, io.frame_n, lock, grant_n, request0_n, io.busy_n[0]);
iport #(.port_number(1)) ip1(clk, reset_n, io.din, io.frame_n, lock, grant_n, request1_n, io.busy_n[1]);
iport #(.port_number(2)) ip2(clk, reset_n, io.din, io.frame_n, lock, grant_n, request2_n, io.busy_n[2]);
iport #(.port_number(3)) ip3(clk, reset_n, io.din, io.frame_n, lock, grant_n, request3_n, io.busy_n[3]);
iport #(.port_number(4)) ip4(clk, reset_n, io.din, io.frame_n, lock, grant_n, request4_n, io.busy_n[4]);
iport #(.port_number(5)) ip5(clk, reset_n, io.din, io.frame_n, lock, grant_n, request5_n, io.busy_n[5]);
iport #(.port_number(6)) ip6(clk, reset_n, io.din, io.frame_n, lock, grant_n, request6_n, io.busy_n[6]);
iport #(.port_number(7)) ip7(clk, reset_n, io.din, io.frame_n, lock, grant_n, request7_n, io.busy_n[7]);
iport #(.port_number(8)) ip8(clk, reset_n, io.din, io.frame_n, lock, grant_n, request8_n, io.busy_n[8]);
iport #(.port_number(9)) ip9(clk, reset_n, io.din, io.frame_n, lock, grant_n, request9_n, io.busy_n[9]);
iport #(.port_number(10)) ip10(clk, reset_n, io.din, io.frame_n, lock, grant_n, request10_n, io.busy_n[10]);
iport #(.port_number(11)) ip11(clk, reset_n, io.din, io.frame_n, lock, grant_n, request11_n, io.busy_n[11]);
iport #(.port_number(12)) ip12(clk, reset_n, io.din, io.frame_n, lock, grant_n, request12_n, io.busy_n[12]);
iport #(.port_number(13)) ip13(clk, reset_n, io.din, io.frame_n, lock, grant_n, request13_n, io.busy_n[13]);
iport #(.port_number(14)) ip14(clk, reset_n, io.din, io.frame_n, lock, grant_n, request14_n, io.busy_n[14]);
iport #(.port_number(15)) ip15(clk, reset_n, io.din, io.frame_n, lock, grant_n, request15_n, io.busy_n[15]);

oport #(.port_number(0)) op0(clk, reset_n, request_n, lock, frameo_int, grant0_n, ip_src_op[0], deassert[0]);
oport #(.port_number(1)) op1(clk, reset_n, request_n, lock, frameo_int, grant1_n, ip_src_op[1], deassert[1]);
oport #(.port_number(2)) op2(clk, reset_n, request_n, lock, frameo_int, grant2_n, ip_src_op[2], deassert[2]);
oport #(.port_number(3)) op3(clk, reset_n, request_n, lock, frameo_int, grant3_n, ip_src_op[3], deassert[3]);
oport #(.port_number(4)) op4(clk, reset_n, request_n, lock, frameo_int, grant4_n, ip_src_op[4], deassert[4]);
oport #(.port_number(5)) op5(clk, reset_n, request_n, lock, frameo_int, grant5_n, ip_src_op[5], deassert[5]);
oport #(.port_number(6)) op6(clk, reset_n, request_n, lock, frameo_int, grant6_n, ip_src_op[6], deassert[6]);
oport #(.port_number(7)) op7(clk, reset_n, request_n, lock, frameo_int, grant7_n, ip_src_op[7], deassert[7]);
oport #(.port_number(8)) op8(clk, reset_n, request_n, lock, frameo_int, grant8_n, ip_src_op[8], deassert[8]);
oport #(.port_number(9)) op9(clk, reset_n, request_n, lock, frameo_int, grant9_n, ip_src_op[9], deassert[9]);
oport #(.port_number(10)) op10(clk, reset_n, request_n, lock, frameo_int, grant10_n, ip_src_op[10], deassert[10]);
oport #(.port_number(11)) op11(clk, reset_n, request_n, lock, frameo_int, grant11_n, ip_src_op[11], deassert[11]);
oport #(.port_number(12)) op12(clk, reset_n, request_n, lock, frameo_int, grant12_n, ip_src_op[12], deassert[12]);
oport #(.port_number(13)) op13(clk, reset_n, request_n, lock, frameo_int, grant13_n, ip_src_op[13], deassert[13]);
oport #(.port_number(14)) op14(clk, reset_n, request_n, lock, frameo_int, grant14_n, ip_src_op[14], deassert[14]);
oport #(.port_number(15)) op15(clk, reset_n, request_n, lock, frameo_int, grant15_n, ip_src_op[15], deassert[15]);

assign frameo_int[0] = frame_reg[ip_src_op[0]];
assign frameo_int[1] = frame_reg[ip_src_op[1]];
assign frameo_int[2] = frame_reg[ip_src_op[2]];
assign frameo_int[3] = frame_reg[ip_src_op[3]];
assign frameo_int[4] = frame_reg[ip_src_op[4]];
assign frameo_int[5] = frame_reg[ip_src_op[5]];
assign frameo_int[6] = frame_reg[ip_src_op[6]];
assign frameo_int[7] = frame_reg[ip_src_op[7]];
assign frameo_int[8] = frame_reg[ip_src_op[8]];
assign frameo_int[9] = frame_reg[ip_src_op[9]];
assign frameo_int[10] = frame_reg[ip_src_op[10]];
assign frameo_int[11] = frame_reg[ip_src_op[11]];
assign frameo_int[12] = frame_reg[ip_src_op[12]];
assign frameo_int[13] = frame_reg[ip_src_op[13]];
assign frameo_int[14] = frame_reg[ip_src_op[14]];
assign frameo_int[15] = frame_reg[ip_src_op[15]];

assign io.frameo_n = frameo_int | deassert;

assign io.valido_n[0] = valid_reg[ip_src_op[0]] | deassert[0];
assign io.valido_n[1] = valid_reg[ip_src_op[1]] | deassert[1];
assign io.valido_n[2] = valid_reg[ip_src_op[2]] | deassert[2];
assign io.valido_n[3] = valid_reg[ip_src_op[3]] | deassert[3];
assign io.valido_n[4] = valid_reg[ip_src_op[4]] | deassert[4];
assign io.valido_n[5] = valid_reg[ip_src_op[5]] | deassert[5];
assign io.valido_n[6] = valid_reg[ip_src_op[6]] | deassert[6];
assign io.valido_n[7] = valid_reg[ip_src_op[7]] | deassert[7];
assign io.valido_n[8] = valid_reg[ip_src_op[8]] | deassert[8];
assign io.valido_n[9] = valid_reg[ip_src_op[9]] | deassert[9];
assign io.valido_n[10] = valid_reg[ip_src_op[10]] | deassert[10];
assign io.valido_n[11] = valid_reg[ip_src_op[11]] | deassert[11];
assign io.valido_n[12] = valid_reg[ip_src_op[12]] | deassert[12];
assign io.valido_n[13] = valid_reg[ip_src_op[13]] | deassert[13];
assign io.valido_n[14] = valid_reg[ip_src_op[14]] | deassert[14];
assign io.valido_n[15] = valid_reg[ip_src_op[15]] | deassert[15];

assign io.dout[0] = (din_reg[ip_src_op[0]] | valid_reg[ip_src_op[0]] | deassert[0]);
assign io.dout[1] = (din_reg[ip_src_op[1]] | valid_reg[ip_src_op[1]] | deassert[1]);
assign io.dout[2] = (din_reg[ip_src_op[2]] | valid_reg[ip_src_op[2]] | deassert[2]);
assign io.dout[3] = (din_reg[ip_src_op[3]] | valid_reg[ip_src_op[3]] | deassert[3]);
assign io.dout[4] = (din_reg[ip_src_op[4]] | valid_reg[ip_src_op[4]] | deassert[4]);
assign io.dout[5] = (din_reg[ip_src_op[5]] | valid_reg[ip_src_op[5]] | deassert[5]);
assign io.dout[6] = (din_reg[ip_src_op[6]] | valid_reg[ip_src_op[6]] | deassert[6]);
assign io.dout[7] = (din_reg[ip_src_op[7]] | valid_reg[ip_src_op[7]] | deassert[7]);
assign io.dout[8] = (din_reg[ip_src_op[8]] | valid_reg[ip_src_op[8]] | deassert[8]);
assign io.dout[9] = (din_reg[ip_src_op[9]] | valid_reg[ip_src_op[9]] | deassert[9]);
assign io.dout[10] = (din_reg[ip_src_op[10]] | valid_reg[ip_src_op[10]] | deassert[10]);
assign io.dout[11] = (din_reg[ip_src_op[11]] | valid_reg[ip_src_op[11]] | deassert[11]);
assign io.dout[12] = (din_reg[ip_src_op[12]] | valid_reg[ip_src_op[12]] | deassert[12]);
assign io.dout[13] = (din_reg[ip_src_op[13]] | valid_reg[ip_src_op[13]] | deassert[13]);
assign io.dout[14] = (din_reg[ip_src_op[14]] | valid_reg[ip_src_op[14]] | deassert[14]);
assign io.dout[15] = (din_reg[ip_src_op[15]] | valid_reg[ip_src_op[15]] | deassert[15]);

assign grant_n[0] = {grant15_n[0], grant14_n[0], grant13_n[0], grant12_n[0], grant11_n[0], grant10_n[0], grant9_n[0], grant8_n[0], grant7_n[0], grant6_n[0], grant5_n[0], grant4_n[0], grant3_n[0], grant2_n[0], grant1_n[0], grant0_n[0]}; 
assign grant_n[1] = {grant15_n[1], grant14_n[1], grant13_n[1], grant12_n[1], grant11_n[1], grant10_n[1], grant9_n[1], grant8_n[1], grant7_n[1], grant6_n[1], grant5_n[1], grant4_n[1], grant3_n[1], grant2_n[1], grant1_n[1], grant0_n[1]}; 
assign grant_n[2] = {grant15_n[2], grant14_n[2], grant13_n[2], grant12_n[2], grant11_n[2], grant10_n[2], grant9_n[2], grant8_n[2], grant7_n[2], grant6_n[2], grant5_n[2], grant4_n[2], grant3_n[2], grant2_n[2], grant1_n[2], grant0_n[2]}; 
assign grant_n[3] = {grant15_n[3], grant14_n[3], grant13_n[3], grant12_n[3], grant11_n[3], grant10_n[3], grant9_n[3], grant8_n[3], grant7_n[3], grant6_n[3], grant5_n[3], grant4_n[3], grant3_n[3], grant2_n[3], grant1_n[3], grant0_n[3]}; 
assign grant_n[4] = {grant15_n[4], grant14_n[4], grant13_n[4], grant12_n[4], grant11_n[4], grant10_n[4], grant9_n[4], grant8_n[4], grant7_n[4], grant6_n[4], grant5_n[4], grant4_n[4], grant3_n[4], grant2_n[4], grant1_n[4], grant0_n[4]}; 
assign grant_n[5] = {grant15_n[5], grant14_n[5], grant13_n[5], grant12_n[5], grant11_n[5], grant10_n[5], grant9_n[5], grant8_n[5], grant7_n[5], grant6_n[5], grant5_n[5], grant4_n[5], grant3_n[5], grant2_n[5], grant1_n[5], grant0_n[5]}; 
assign grant_n[6] = {grant15_n[6], grant14_n[6], grant13_n[6], grant12_n[6], grant11_n[6], grant10_n[6], grant9_n[6], grant8_n[6], grant7_n[6], grant6_n[6], grant5_n[6], grant4_n[6], grant3_n[6], grant2_n[6], grant1_n[6], grant0_n[6]}; 
assign grant_n[7] = {grant15_n[7], grant14_n[7], grant13_n[7], grant12_n[7], grant11_n[7], grant10_n[7], grant9_n[7], grant8_n[7], grant7_n[7], grant6_n[7], grant5_n[7], grant4_n[7], grant3_n[7], grant2_n[7], grant1_n[7], grant0_n[7]}; 
assign grant_n[8] = {grant15_n[8], grant14_n[8], grant13_n[8], grant12_n[8], grant11_n[8], grant10_n[8], grant9_n[8], grant8_n[8], grant7_n[8], grant6_n[8], grant5_n[8], grant4_n[8], grant3_n[8], grant2_n[8], grant1_n[8], grant0_n[8]}; 
assign grant_n[9] = {grant15_n[9], grant14_n[9], grant13_n[9], grant12_n[9], grant11_n[9], grant10_n[9], grant9_n[9], grant8_n[9], grant7_n[9], grant6_n[9], grant5_n[9], grant4_n[9], grant3_n[9], grant2_n[9], grant1_n[9], grant0_n[9]}; 
assign grant_n[10] = {grant15_n[10], grant14_n[10], grant13_n[10], grant12_n[10], grant11_n[10], grant10_n[10], grant9_n[10], grant8_n[10], grant7_n[10], grant6_n[10], grant5_n[10], grant4_n[10], grant3_n[10], grant2_n[10], grant1_n[10], grant0_n[10]}; 
assign grant_n[11] = {grant15_n[11], grant14_n[11], grant13_n[11], grant12_n[11], grant11_n[11], grant10_n[11], grant9_n[11], grant8_n[11], grant7_n[11], grant6_n[11], grant5_n[11], grant4_n[11], grant3_n[11], grant2_n[11], grant1_n[11], grant0_n[11]}; 
assign grant_n[12] = {grant15_n[12], grant14_n[12], grant13_n[12], grant12_n[12], grant11_n[12], grant10_n[12], grant9_n[12], grant8_n[12], grant7_n[12], grant6_n[12], grant5_n[12], grant4_n[12], grant3_n[12], grant2_n[12], grant1_n[12], grant0_n[12]}; 
assign grant_n[13] = {grant15_n[13], grant14_n[13], grant13_n[13], grant12_n[13], grant11_n[13], grant10_n[13], grant9_n[13], grant8_n[13], grant7_n[13], grant6_n[13], grant5_n[13], grant4_n[13], grant3_n[13], grant2_n[13], grant1_n[13], grant0_n[13]}; 
assign grant_n[14] = {grant15_n[14], grant14_n[14], grant13_n[14], grant12_n[14], grant11_n[14], grant10_n[14], grant9_n[14], grant8_n[14], grant7_n[14], grant6_n[14], grant5_n[14], grant4_n[14], grant3_n[14], grant2_n[14], grant1_n[14], grant0_n[14]}; 
assign grant_n[15] = {grant15_n[15], grant14_n[15], grant13_n[15], grant12_n[15], grant11_n[15], grant10_n[15], grant9_n[15], grant8_n[15], grant7_n[15], grant6_n[15], grant5_n[15], grant4_n[15], grant3_n[15], grant2_n[15], grant1_n[15], grant0_n[15]}; 

assign request_n[0] = {request15_n[0], request14_n[0], request13_n[0], request12_n[0], request11_n[0], request10_n[0], request9_n[0], request8_n[0], request7_n[0], request6_n[0], request5_n[0], request4_n[0], request3_n[0], request2_n[0], request1_n[0], request0_n[0]}; 
assign request_n[1] = {request15_n[1], request14_n[1], request13_n[1], request12_n[1], request11_n[1], request10_n[1], request9_n[1], request8_n[1], request7_n[1], request6_n[1], request5_n[1], request4_n[1], request3_n[1], request2_n[1], request1_n[1], request0_n[1]}; 
assign request_n[2] = {request15_n[2], request14_n[2], request13_n[2], request12_n[2], request11_n[2], request10_n[2], request9_n[2], request8_n[2], request7_n[2], request6_n[2], request5_n[2], request4_n[2], request3_n[2], request2_n[2], request1_n[2], request0_n[2]}; 
assign request_n[3] = {request15_n[3], request14_n[3], request13_n[3], request12_n[3], request11_n[3], request10_n[3], request9_n[3], request8_n[3], request7_n[3], request6_n[3], request5_n[3], request4_n[3], request3_n[3], request2_n[3], request1_n[3], request0_n[3]}; 
assign request_n[4] = {request15_n[4], request14_n[4], request13_n[4], request12_n[4], request11_n[4], request10_n[4], request9_n[4], request8_n[4], request7_n[4], request6_n[4], request5_n[4], request4_n[4], request3_n[4], request2_n[4], request1_n[4], request0_n[4]}; 
assign request_n[5] = {request15_n[5], request14_n[5], request13_n[5], request12_n[5], request11_n[5], request10_n[5], request9_n[5], request8_n[5], request7_n[5], request6_n[5], request5_n[5], request4_n[5], request3_n[5], request2_n[5], request1_n[5], request0_n[5]}; 
assign request_n[6] = {request15_n[6], request14_n[6], request13_n[6], request12_n[6], request11_n[6], request10_n[6], request9_n[6], request8_n[6], request7_n[6], request6_n[6], request5_n[6], request4_n[6], request3_n[6], request2_n[6], request1_n[6], request0_n[6]}; 
assign request_n[7] = {request15_n[7], request14_n[7], request13_n[7], request12_n[7], request11_n[7], request10_n[7], request9_n[7], request8_n[7], request7_n[7], request6_n[7], request5_n[7], request4_n[7], request3_n[7], request2_n[7], request1_n[7], request0_n[7]}; 
assign request_n[8] = {request15_n[8], request14_n[8], request13_n[8], request12_n[8], request11_n[8], request10_n[8], request9_n[8], request8_n[8], request7_n[8], request6_n[8], request5_n[8], request4_n[8], request3_n[8], request2_n[8], request1_n[8], request0_n[8]}; 
assign request_n[9] = {request15_n[9], request14_n[9], request13_n[9], request12_n[9], request11_n[9], request10_n[9], request9_n[9], request8_n[9], request7_n[9], request6_n[9], request5_n[9], request4_n[9], request3_n[9], request2_n[9], request1_n[9], request0_n[9]}; 
assign request_n[10] = {request15_n[10], request14_n[10], request13_n[10], request12_n[10], request11_n[10], request10_n[10], request9_n[10], request8_n[10], request7_n[10], request6_n[10], request5_n[10], request4_n[10], request3_n[10], request2_n[10], request1_n[10], request0_n[10]}; 
assign request_n[11] = {request15_n[11], request14_n[11], request13_n[11], request12_n[11], request11_n[11], request10_n[11], request9_n[11], request8_n[11], request7_n[11], request6_n[11], request5_n[11], request4_n[11], request3_n[11], request2_n[11], request1_n[11], request0_n[11]}; 
assign request_n[12] = {request15_n[12], request14_n[12], request13_n[12], request12_n[12], request11_n[12], request10_n[12], request9_n[12], request8_n[12], request7_n[12], request6_n[12], request5_n[12], request4_n[12], request3_n[12], request2_n[12], request1_n[12], request0_n[12]}; 
assign request_n[13] = {request15_n[13], request14_n[13], request13_n[13], request12_n[13], request11_n[13], request10_n[13], request9_n[13], request8_n[13], request7_n[13], request6_n[13], request5_n[13], request4_n[13], request3_n[13], request2_n[13], request1_n[13], request0_n[13]}; 
assign request_n[14] = {request15_n[14], request14_n[14], request13_n[14], request12_n[14], request11_n[14], request10_n[14], request9_n[14], request8_n[14], request7_n[14], request6_n[14], request5_n[14], request4_n[14], request3_n[14], request2_n[14], request1_n[14], request0_n[14]}; 
assign request_n[15] = {request15_n[15], request14_n[15], request13_n[15], request12_n[15], request11_n[15], request10_n[15], request9_n[15], request8_n[15], request7_n[15], request6_n[15], request5_n[15], request4_n[15], request3_n[15], request2_n[15], request1_n[15], request0_n[15]}; 

always_ff @(posedge io.clk) begin
  din_reg <= io.din;
  frame_reg <= io.frame_n;
  valid_reg <= io.valid_n;
end

endmodule  //router

module iport(input logic clk, reset_n, input logic[15:0] din, frame_n, lock, grant_n[16], output logic[15:0] request_n, output logic busy_n);
parameter port_number = 0;

logic [3:0] state, nxt_state, oport_no, nxt_oport_no;
logic       busy_n_int;

always_ff @(posedge clk or negedge reset_n) begin
  if (!reset_n) begin
    state    <= 0;
    oport_no <= 0;
    busy_n   <= 1'b1;
  end
  else if (lock[port_number] == 0) begin
    state    <= nxt_state;
    oport_no <= nxt_oport_no;
    busy_n   <= busy_n_int;
  end
end

always_comb begin
  request_n = '1;
  nxt_state = state + 1;
  busy_n_int = 1'b1;
  nxt_oport_no = oport_no;
  unique case(state) inside
    [0:3]:   if (frame_n[port_number] == 1'b1) nxt_state = 0;
             else begin
               nxt_oport_no[state] = din[port_number];
             end
    [4:5]:   if ((frame_n[port_number] == 1'b1) || (din[port_number] == 1'b0)) nxt_state = 0;
    6:       if ((frame_n[port_number] == 1'b1) || (din[port_number] == 1'b0)) nxt_state = 0;
             else begin
               request_n[oport_no] = 1'b0;
             end
    7:       if ((frame_n[port_number] == 1'b1) || (din[port_number] == 1'b0)) nxt_state = 0;
             else if (grant_n[port_number][oport_no] == 1'b1) begin
               request_n[oport_no] = 1'b0;
               busy_n_int = 1'b0;
               nxt_state = state;
             end
    8:       begin
//               busy_n_int = 1'b0;
               if (frame_n[port_number] == 1'b1) begin
                 nxt_state = 0;
               end else begin
                 nxt_state = state;
               end
             end
    default: nxt_state = 0;
  endcase
end

endmodule  //iport

module oport(input logic clk, reset_n, input logic[15:0] request_n[16], input logic[15:0] lock, frame_n, output logic[15:0] grant_n, output logic[3:0] ip_src_op, output logic deassert);

parameter port_number = 0;

logic [3:0] port_last, nxt_port, port_no, nxt_ip_src_op;
logic [2:0] state, nxt_state;
logic       hit;

wire [15:0] request_int;

assign request_int = (request_n[port_number] >> (port_last+1)) | (request_n[port_number] << (16-(port_last+1)));

always_comb begin
  hit = 1'b1;
  port_no = 0;
  case(1'b0)
    request_int[0]:  port_no = 0;
    request_int[1]:  port_no = 1;
    request_int[2]:  port_no = 2;
    request_int[3]:  port_no = 3;
    request_int[4]:  port_no = 4;
    request_int[5]:  port_no = 5;
    request_int[6]:  port_no = 6;
    request_int[7]:  port_no = 7;
    request_int[8]:  port_no = 8;
    request_int[9]:  port_no = 9;
    request_int[10]:  port_no = 10;
    request_int[11]:  port_no = 11;
    request_int[12]:  port_no = 12;
    request_int[13]:  port_no = 13;
    request_int[14]:  port_no = 14;
    request_int[15]:  port_no = 15;
    default: hit = 1'b0;
  endcase
end

always_ff @(posedge clk or negedge reset_n) begin
  if (!reset_n) begin
    state     <= 1;
    port_last <= '1;
    ip_src_op <= 0;
  end
  else if (lock[port_number] == 0) begin
    state     <= nxt_state;
    port_last <= nxt_port;
    ip_src_op <= nxt_ip_src_op;
  end
end

always_comb begin
  nxt_state     = '0;
  nxt_port      = port_last;
  nxt_ip_src_op = ip_src_op;
  deassert      = 0;
  grant_n = '1;
    case(1'b1)
      state[0]: if (hit) begin
                  nxt_port        = port_no + port_last + 1;
                  nxt_state[1]    = 1;
                  nxt_ip_src_op   = nxt_port;
                  deassert        = 1;
                end
                else begin
                  nxt_state[0]    = 1;
                  deassert        = 1;
                end
      state[1]: begin
                  grant_n[port_last] = 1'b0;
                  nxt_state[2]       = 1;
                end
      state[2]: if (frame_n[port_number]) nxt_state[0] = 1;
                else nxt_state[2] = 1;
    endcase
end

endmodule  //oport

```











