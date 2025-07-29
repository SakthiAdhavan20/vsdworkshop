# Sky130 Day 4 – Pre-layout Timing Analysis and Clock Tree Optimization

---

# Sky130 Day 4 – Pre-layout Timing Analysis and Clock Tree Optimization

---

## 1. Convert Grid Info to Track Info and Verify Magic Layout

Before integrating a custom standard cell into the PnR flow, we need to ensure it aligns with the design track grid.

**Conditions to Meet:**
- Ports must lie on intersections of horizontal and vertical tracks.  
- Width should be an odd multiple of horizontal pitch.  
- Height should be an even multiple of vertical pitch.

```tcl
grid 0.46um 0.34um 2 2
```

![Track Info 1](screenshots/137.png)

---

## 2. Convert Magic Layout to Standard Cell LEF

```bash
magic -T sky130A.tech sky130_inv.mag
```

```magic
lef write
```

This will generate a `.lef` file for your inverter.

![LEF Export](screenshots/138.png)

---

## 3. Insert Custom Cell in `config.tcl`

Edit your `openlane/designs/picorv32a/config.tcl`:

```tcl
set ::env(CELL_LEFS) "$::env(CELL_LEFS) /path/to/sky130_inv.lef"
```

Also add to:

```tcl
set ::env(LIB_SYNTH) "$::env(LIB_SYNTH) /path/to/sky130_inv.lib"
set ::env(LIB_TYPICAL) "$::env(LIB_TYPICAL) /path/to/sky130_inv.lib"
```

![Config TCL Edit](screenshots/139.png)

---

## 4. Run Synthesis with Custom Inverter Cell

```bash
cd openlane
flow.tcl -design picorv32a -init_design_config
flow.tcl -design picorv32a -tag vsdinv_synth -overwrite
```

This will include the custom inverter in the netlist.

![Synthesis Run](screenshots/140.png)

---

## 5. Netlist and Cell Verification

Open the synthesized Verilog and confirm that `sky130_inv` is used.

```bash
cat runs/vsdinv_synth/results/synthesis/picorv32a.v | grep sky130_inv
```

![Netlist Verification](screenshots/141.png)

---

## 6. Delay Table (lib) Setup and Format

Your custom `.lib` file should have a delay table section like:

```liberty
cell (sky130_inv) {
  ...
  pin(A) {
    direction : input;
    ...
  }
  pin(Y) {
    direction : output;
    function : "A'";
    ...
    timing () {
      related_pin : "A";
      timing_sense : negative_unate;
      cell_rise(delay_table) {...}
      cell_fall(delay_table) {...}
    }
  }
}
```

![Delay Table](screenshots/142.png)

---

## 7. Setup OpenSTA for Ideal Clock Timing Analysis

```bash
cd openlane/designs/picorv32a
opensta
```

```tcl
read_liberty /path/to/sky130_inv.lib
read_verilog runs/vsdinv_synth/results/synthesis/picorv32a.v
link_design picorv32a
read_sdc runs/vsdinv_synth/results/synthesis/picorv32a.sdc
report_checks -path_delay min_max
```

![OpenSTA Ideal Clocks](screenshots/143.png)

---

## 8. Run TritonCTS (Clock Tree Synthesis)

```bash
flow.tcl -design picorv32a -tag vsdinv_cts -from cts
```

This step inserts and buffers clock tree using your given CTS buffer.

![CTS Run](screenshots/144.png)

---

## 9. Post-CTS Setup & Hold Timing Analysis

```bash
cd runs/vsdinv_cts/results/cts
opensta
```

```tcl
read_liberty /path/to/sky130_inv.lib
read_verilog picorv32a.cts.v
link_design picorv32a
read_sdc picorv32a.sdc
report_checks -path_delay min_max
```

Check for any setup or hold violations after CTS.

![Post-CTS Timing](screenshots/145.png)

---

## 10. Slack Optimization using Synthesis Configuration

You can optimize slack by adjusting synthesis constraints in `config.tcl`:

```tcl
set ::env(SYNTH_MAX_TRAN) 0.75
set ::env(SYNTH_MAX_FANOUT) 4
set ::env(SYNTH_STRATEGY) 2
```

Re-run synthesis and STA again.

![Slack Optimization](screenshots/146.png)

---

## 11. Crosstalk and Shielding

To minimize signal interference, ensure critical nets are shielded.

Use:

```tcl
set ::env(PL_TARGET_DENSITY) 0.50
set ::env(FP_PDN_VOFFSET) 10
```

Then re-run `flow.tcl`.

![Crosstalk Control](screenshots/147.png)

---

## Final Result Summary

```tcl
report_checks -path_delay min_max
```

Expected result:

- **Setup Slack**: MET
- **Hold Slack**: MET

![Final STA](screenshots/148.png)


### Lab Steps to configure OpenSTA for post-synth timing analysis

```tcl
./flow.tcl -interactive
package require openlane 0.9
prep -design picorv32a -tag 16-03_17-49 -overwrite
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs
set ::env(SYNTH_SIZING) 1
run_synthesis
```

> Before moving to post-synth analysis, complete the above steps. 
> To perform post-synthesis timing analysis, first we need to add config files into our flow.

Create a config file at:

```
/home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/pre_sta.conf
```

Add the following content:

![pre_sta.conf](screenshots/142.png)

```tcl
set_cmd_units -time ns -capacitance pF -current mA -voltage V -resistance kOhm -distance um
read_liberty -min /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/sky130_fd_sc_hd__fast.lib
read_liberty -max /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/sky130_fd_sc_hd__slow.lib
read_verilog /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/16-03_17-49/results/synthesis/picorv32a.synthesis.v
link_design picorv32a
read_sdc /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/my_base.sdc
report_checks -path_delay min_max -fields {slew trans net cap input_pin}
report_tns
report_wns
```

Create the SDC file at:

```
/home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/my_base.sdc
```

Add the following content:

![my_base.sdc](screenshots/143.png)

```tcl
set ::env(CLOCK_PORT) clk
set ::env(CLOCK_PERIOD) 24.73
set ::env(SYNTH_DRIVING_CELL) sky130_fd_sc_hd__inv_8
set ::env(SYNTH_DRIVING_CELL_PIN) Y
set ::env(SYNTH_CAP_LOAD) 17.65
create_clock [get_ports $::env(CLOCK_PORT)]  -name $::env(CLOCK_PORT)  -period $::env(CLOCK_PERIOD)
set IO_PCT  0.2
set input_delay_value [expr $::env(CLOCK_PERIOD) * $IO_PCT]
set output_delay_value [expr $::env(CLOCK_PERIOD) * $IO_PCT]
puts "\[INFO\]: Setting output delay to: $output_delay_value"
puts "\[INFO\]: Setting input delay to: $input_delay_value"

set clk_indx [lsearch [all_inputs] [get_port $::env(CLOCK_PORT)]]
#set rst_indx [lsearch [all_inputs] [get_port resetn]]
set all_inputs_wo_clk [lreplace [all_inputs] $clk_indx $clk_indx]
#set all_inputs_wo_clk_rst [lreplace $all_inputs_wo_clk $rst_indx $rst_indx]
set all_inputs_wo_clk_rst $all_inputs_wo_clk

# correct resetn
set_input_delay $input_delay_value  -clock [get_clocks $::env(CLOCK_PORT)] $all_inputs_wo_clk_rst
#set_input_delay 0.0 -clock [get_clocks $::env(CLOCK_PORT)] {resetn}
set_output_delay $output_delay_value  -clock [get_clocks $::env(CLOCK_PORT)] [all_outputs]

# TODO set this as parameter
set_driving_cell -lib_cell $::env(SYNTH_DRIVING_CELL) -pin $::env(SYNTH_DRIVING_CELL_PIN) [all_inputs]
set cap_load [expr $::env(SYNTH_CAP_LOAD) / 1000.0]
puts "\[INFO\]: Setting load to: $cap_load"
set_load  $cap_load [all_outputs]
```

Now go to the OpenLane directory and run:

```bash
sta pre_sta.conf
```

This performs post-synthesis timing analysis.

![STA Output 1](screenshots/144.png) 
![STA Output 2](screenshots/145.png) 
![STA Output 3](screenshots/146.png)

---

### Changing MAX_FANOUT to 4 and Re-running Synthesis

```tcl
./flow.tcl -interactive
package require openlane 0.9
prep -design picorv32a -tag 16-03_17-49 -overwrite
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs
set ::env(SYNTH_SIZING) 1
set ::env(SYNTH_MAX_FANOUT) 4
run_synthesis
```

---

![Fanout Update 1](screenshots/147.png) 
![Fanout Update 2](screenshots/148.png)
![Fanout Update 3](screenshots/149.png) 
![Fanout Update 4](screenshots/150.png) 
![Fanout Update 5](screenshots/151.png)

# Clock Tree Synthesis (CTS) using TritonCTS and Signal Integrity

## Introduction to CTS and Signal Integrity

![CTS Intro](../images/152.png)

![CTS Flow](../images/153.png)

- So we go for `H-Tree` method

![H-Tree Method](../images/154.png)

![H-Tree Buffers](../images/155.png)

- After adding buffers 👇

![After Buffering](../images/156.png)

## Clock Net Shielding 👇

![Shielding Step 1](../images/158.png)

![Shielding Step 2](../images/159.png)

- therefore the clock net is shieled 

![Shielded Clock Net](../images/157.png)

## Lab Steps to run CTS using TritonCTS

- From the previous labs, run until the placement stage using the commands:

```tcl
prep -design picorv32a -tag 16-03_17-49 -overwrite
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs

set ::env(SYNTH_STRATEGY) "DELAY 0"
set ::env(SYNTH_SIZING) 1

run_synthesis
init_floorplan
place_io
global_placement_or
detailed_placement
tap_decap_or
detailed_placement
```

- After placement is done, we move on to the CTS stage and run it using the command:

```tcl
run_cts
```

- This uses `TritonCTS` inside OpenROAD to insert clock buffers and route the clock net.

## Output After CTS Stage

![CTS Output 1](../images/173.png)

![CTS Output 2](../images/174.png)

![CTS Output 3](../images/175.png)

## Verifying if CTS is Run Properly or Not

![CTS Verify 1](../images/176.png)

![CTS Verify 2](../images/177.png)

![CTS Verify 3](../images/178.png)

# Timing analysis with real clocks

---

## Setup time Analysis

![Setup timing 1](../images/160.png)  
![Setup timing 2](../images/162.png)  
![Setup timing 3](../images/161.png)  

---

## Hold time Analysis

![Hold timing 1](../images/164.png)  
![Hold timing 2](../images/163.png)  
![Hold timing 3](../images/165.png)  
![Hold timing 4](../images/166.png)  
![Hold timing 5](../images/167.png)  

---

## LAB - Steps to analyze timing with real clocks

### 1. Run CTS  
Use the command:
```bash
run_cts
```

#### Output:
![CTS run output 1](../images/182.png)  
![CTS run output 2](../images/183.png)  
![CTS run output 3](../images/184.png)  

---

### 2. Post-CTS Timing Analysis using OpenROAD

Launch OpenROAD and run the following commands:

```tcl
openroad
read_lef /openLANE_flow/designs/picorv32a/runs/16-03_17-49/tmp/merged.lef
read_def /openLANE_flow/designs/picorv32a/runs/16-03_17-49/results/cts/picorv32a.cts.def
write_db pico_cts.db
read_db pico_cts.db
read_verilog /openLANE_flow/designs/picorv32a/runs/16-03_17-49/results/synthesis/picorv32a.synthesis_cts.v
read_liberty $::env(LIB_TYPICAL)
link_design picorv32a
read_sdc /openLANE_flow/designs/picorv32a/src/my_base.sdc
set_propagated_clock [all_clocks]
report_checks -path_delay min_max -format full_clock_expanded -digits 4
```

#### Output:
![Timing analysis post-CTS 1](../images/185.png)  
![Timing analysis post-CTS 2](../images/186.png)  
![Timing analysis post-CTS 3](../images/187.png)  

---

## LAB - Observe impact of bigger CTS buffers on setup/hold

```tcl
echo $::env(CTS_CLK_BUFFER_LIST)
set ::env(CTS_CLK_BUFFER_LIST) [lreplace $::env(CTS_CLK_BUFFER_LIST) 0 0]

echo $::env(CURRENT_DEF)
set ::env(CURRENT_DEF) /openLANE_flow/designs/picorv32a/runs/16-03_17-49/results/placement/picorv32a.placement.def

run_cts
echo $::env(CTS_CLK_BUFFER_LIST)

openroad

read_lef /openLANE_flow/designs/picorv32a/runs/16-03_17-49/tmp/merged.lef
read_def /openLANE_flow/designs/picorv32a/runs/16-03_17-49/results/cts/picorv32a.cts.def
write_db pico_cts1.db
read_db pico_cts1.db
read_verilog /openLANE_flow/designs/picorv32a/runs/16-03_17-49/results/synthesis/picorv32a.synthesis_cts.v
read_liberty $::env(LIB_TYPICAL)
link_design picorv32a
read_sdc /openLANE_flow/designs/picorv32a/src/my_base.sdc
set_propagated_clock [all_clocks]
report_checks -path_delay min_max -fields {slew trans net cap input_pins} -format full_clock_expanded -digits 4

report_clock_skew -hold
report_clock_skew -setup
```

#### Output:
![CTS buffer comparison 1](../images/188.png)  
![CTS buffer comparison 2](../images/189.png)  
![CTS buffer comparison 3](../images/190.png)  
![CTS buffer comparison 4](../images/191.png)  

---

### Insert smaller buffer (sky130_fd_sc_hd__clkbuf_1)

```tcl
echo $::env(CTS_CLK_BUFFER_LIST)
set ::env(CTS_CLK_BUFFER_LIST) [linsert $::env(CTS_CLK_BUFFER_LIST) 0 sky130_fd_sc_hd__clkbuf_1]
```

![Insert clkbuf_1](../images/192.png)

---

### Timing Comparison Results

```text
For clkbuf_1
    hold slack -   0.1127   slack (MET)
    setup slack -  13.8266  slack (MET)

For clkbuf_2
    hold slack -   0.2892   slack (MET)
    setup slack -  13.8266  slack (MET)
```

 **Observation**: Hold slack improved when using `clkbuf_2`.



