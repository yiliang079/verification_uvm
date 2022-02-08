```c++
class ms_scoreboard extends scoreboard;

  `uvm_analysis_imp_decl(_before)
  `uvm_analysis_imp_decl(_after)

  uvm_analysis_imp_before #(packet, ms_scoreboard) ms_before_export;
  uvm_analysis_imp_after  #(packet, ms_scoreboard) ms_after_export;
  
      ms_before_export = new("ms_before_export", this);
    ms_after_export  = new("ms_after_export", this);
    
        before_export.connect(ms_before_export);
    after_export.connect(ms_after_export);
```

```c++
class router_env extends uvm_env;

  input_agent i_agt[16];

    foreach (o_agt[i]) begin
      o_agt[i].analysis_port.connect(sb.after_export);
    end
```

```c++
class output_agent extends uvm_agent;

  virtual router_io vif;           
  int               port_id = -1;  
  oMonitor          mon;
  uvm_analysis_port #(packet) analysis_port;

  analysis_port = new("analysis_port", this);
    mon.analysis_port.connect(this.analysis_port);
```

```c++
class oMonitor extends uvm_monitor;
  
  virtual router_io vif;
  int               port_id = -1;
  uvm_analysis_port #(packet) analysis_port;
      analysis_port = new("analysis_port", this);
      
            analysis_port.write(tr);
```



