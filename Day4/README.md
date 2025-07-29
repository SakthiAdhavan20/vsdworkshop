# Sky130 Day 4 – Pre-layout Timing Analysis and Importance of a Good Clock Tree

---

## Timing Modelling using Delay Tables

---

## Lab Steps to Convert Grid Info to Track Info

We only require the `.lef` file, which contains the cell physical information; we don't need all the information in the `.mag` file.

So we have to extract the `.lef` file from the `.mag` file.

This is the inverter we have seen from the previous labs.

![Track Info](screenshots/inv.png)

### Inverter Layout

There are some conditions that need to be satisfied before we place standard cells into the PnR flow:

- Input and output ports of the standard cell should lie on the intersection of the vertical and horizontal tracks.
- Width of the standard cell should be odd multiples of the horizontal track pitch.
- Height of the standard cell should be even multiples of the vertical track pitch.

In the below location, we can find the `tracks.info` for `sky130_fd_sc_hd`.

- Track Info 1 

  ![Track Info](screenshots/Track_Info_1.png)

- Track Info 2 

  ![Track Info Zoom](screenshots/Track_Info_2.png)

Initially the grid size looks like this:

- **Original Grid**
 
  ![Original Grid](screenshots/Initial_Grid_View.png)

Now resize the grid as per the dimensions in `tracks.info`:

```
grid 0.46um 0.34um 0.23um 0.17um
```

- **Resized Grid 1** 

![Resized Grid 1](screenshots/After_Resize.png)


Here, the input and output ports (A and Y) are lying on the intersection of the vertical and horizontal pitch ✅

- Horizontal track pitch = 0.46um → width = 3 × 0.46 = 1.38um ✅ 
  
 
- Vertical track pitch = 0.34um → height = 8 × 0.34 = 2.72um ✅

 ![Horizontal Track](screenshots/Final_Grid.png)

### Save Layout

```tcl
save sky130_vsdinv.mag
magic -T sky130A.tech sky130_vsdinv.mag &
```
Generate `.lef` file:

```tcl
lef write
```

- **LEF Write** 

![LEF Write Output](screenshots/LEF_Generation.png)
  
![LEF Write Output](screenshots/LEF_Generation1.png)

![LEF Write Output](screenshots/LEF_Generation2.png)

![LEF Write Output](screenshots/LEF_Generation3.png)

---

## Copy `.lef` and `.lib` Files to OpenLANE src Directory

```bash
cp libs/sky130_fd_sc_hd__* ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/
cp sky130_vsdinv.lef ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/
```

- **Copied Files Result** 
 
![Copied LEF and LIB](screenshots/Copied_LEF_and_LIB.png)
  
![Copied LEF and LIB](screenshots/Copied_LEF_and_LIB_1.png)

---

## Update `config.tcl` with New File Paths

```tcl
set ::env(LIB_SYNTH) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib"
set ::env(LIB_SLOWEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__slow.lib"
set ::env(LIB_FASTEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__fast.lib"
set ::env(LIB_TYPICAL) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib"

set ::env(EXTRA_LEFS) [glob $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/src/*.lef]
```

- **Updated Config**

  ![Config.tcl Update](screenshots/Config_Edit.png)

- **sky130_fd_sc_hd__typical.lib**
  
  ![Config.tcl Update](screenshots/Config_Edit1.png)

- **sky130_fd_sc_hd__slow.lib**

  ![Config.tcl Update](screenshots/Config_Edit2.png)

- **sky130_fd_sc_hd__fast.lib**

  ![Config.tcl Update](screenshots/Config_Edit3.png)

---

## Start OpenLANE Flow

```tcl
./flow.tcl -interactive
package require openlane 0.9
prep -design picorv32a -tag 26-07_06-22 -overwrite
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs
run_synthesis
```

- **Run Synthesis 1** 

  ![Synthesis 1](screenshots/Synthesis_1.png)

- **Run Synthesis 2** 

  ![Synthesis 2](screenshots/Synthesis_2.png)

---

## Delay Tables

To avoid large skew between clock endpoints:

- Buffers on the same level must have same load → same delay
- Buffers must be identical across levels

- **Delay Table 1** 

  ![Delay 1](screenshots/Delay_1.png)

- **Delay Table 2** 

  ![Delay 2](screenshots/Delay_2.png)

- **Delay Table 3** 

  ![Delay 3](screenshots/Delay_3.png)

- **Delay Table 4** 

  ![Delay 4](screenshots/Delay_4.png)

**Skew = 0** because delay on both paths = x9' + y15

---

## Steps to Configure Synthesis to Fix Slack

View slack-related synthesis parameters in `README.md`

- **Synthesis Params** 

  ![Synthesis Param](screenshots/Synthesis_Param.png)

Update synthesis settings:

```tcl
prep -design picorv32a -tag 26-07_06-22 -overwrite
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs

echo $::env(SYNTH_STRATEGY)
set ::env(SYNTH_STRATEGY) "DELAY 0"

echo $::env(SYNTH_BUFFERING)
echo $::env(SYNTH_SIZING)
set ::env(SYNTH_SIZING) 1

echo $::env(SYNTH_DRIVING_CELL)

run_synthesis
```

- **Updated Synth Output 1** 

  ![Synthesis Output 1](screenshots/Synthesis_Output_1.png)

- **Updated Synth Output 2** 

  ![Synthesis Output 2](screenshots/Synthesis_Output_2.png)

**Before:**

```
Chip area: 147712.918400
tns: -711.59
wns: -23.89
```

**After:**

```
Chip area: 196832.528000
tns: 0
wns: 0
```
 ![Synthesis Output 2](screenshots/Synthesis_Output_3.png)

To verify inverter inclusion, search `merged.lef` for `vsdinv`

- **LEF verification 1** 

  ![LEF Search](screenshots/LEF_Search1.png)

- **LEF verification 2** 

  ![LEF Search 2](screenshots/LEF_Search2.png)

---

## Floorplanning and Placement

```tcl
run_floorplan
```

If error appears, refer to OpenLANE commands documentation

- **OpenLANE Error** 

  ![Error Msg](screenshots/Error_Msg.png)

- **Commands Ref** 

  ![Command Help](screenshots/Command_Help.png)

Then run:

```tcl
init_floorplan
place_io
global_placement_or
detailed_placement
tap_decap_or
detailed_placement
```

- **init_floorplan** 

  ![Floorplan](screenshots/Floorplan.png)

- **place_io** 

  ![place_io](screenshots/place_io.png)

- **global_placement_or** 

  ![global_placement_or](screenshots/global_placement_or.png)

- **detailed_placement** 

  ![Decap and Re-Placement](screenshots/Decap_and_Re-Placement.png)

- **tap_decap_or**
  
  ![tap_decap_or](screenshots/tap_decap_or.png)

- **detailed_placement**

  ![Decap and Re-Placement](screenshots/Decap_and_Re-Placemen1.png)

---

## View Placement in Magic

```tcl
magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech \
lef read ../../tmp/merged.lef \
def read picorv32a.placement.def &
```
 ![Magic View 1](screenshots/Magic_View.png)

- **Magic View**

  ![Magic View 1](screenshots/Magic_View_1.png)

 
  ![Magic View 2](screenshots/Magic_View_2.png)


  ![Magic View 3](screenshots/Magic_View_3.png)


  ![Magic View 4](screenshots/Magic_View_4.png)


  ![Magic View 5](screenshots/Magic_View_5.png)


  ![Magic View 6](screenshots/Magic_View_6.png)


  ![Magic View 7](screenshots/Magic_View_7.png)

# Timing Analysis with Ideal Clocks using OpenSTA

![STA  File 1](screenshots/STA1.png)

![STA  File 2](screenshots/STA2.png)

![STA  File 3](screenshots/STA3.png)

![STA  File 4](screenshots/STA4.png)

![STA  File 5](screenshots/STA5.png)

---

## Step 1: Initial Setup and Synthesis

```bash
./flow.tcl -interactive
package require openlane 0.9
prep -design picorv32a -tag 26-07_06-22 -overwrite
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs
set ::env(SYNTH_SIZING) 1
run_synthesis
```

![Synthesis](screenshots/Synthesis.png)

---

## Step 2: Create the STA Configuration File

Path:
```
/home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/pre_sta.conf
```

```tcl
set_cmd_units -time ns -capacitance pF -current mA -voltage V -resistance kOhm -distance um
read_liberty -min /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/sky130_fd_sc_hd__fast.lib
read_liberty -max /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/sky130_fd_sc_hd__slow.lib
read_verilog /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/26-07_06-22/results/synthesis/picorv32a.synthesis.v
link_design picorv32a
read_sdc /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/my_base.sdc
report_checks -path_delay min_max -fields {slew trans net cap input_pin}
report_tns
report_wns
```

![STA Configuration File](screenshots/STA_Configuration_File.png)

---

## Step 3: Create the SDC File

Path:
```
/home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/my_base.sdc
```

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
set all_inputs_wo_clk [lreplace [all_inputs] $clk_indx $clk_indx]
set all_inputs_wo_clk_rst $all_inputs_wo_clk

set_input_delay $input_delay_value  -clock [get_clocks $::env(CLOCK_PORT)] $all_inputs_wo_clk_rst
set_output_delay $output_delay_value  -clock [get_clocks $::env(CLOCK_PORT)] [all_outputs]

set_driving_cell -lib_cell $::env(SYNTH_DRIVING_CELL) -pin $::env(SYNTH_DRIVING_CELL_PIN) [all_inputs]
set cap_load [expr $::env(SYNTH_CAP_LOAD) / 1000.0]
puts "\[INFO\]: Setting load to: $cap_load"
set_load  $cap_load [all_outputs]
```

![SDC Configuration File](screenshots/SDC_Configuration_File.png)

---

## Step 4: Run Timing Analysis using OpenSTA

```bash
sta pre_sta.conf
```

Sample output:

![STA Report 1](screenshots/STA_Report_1.png) 
![STA Report 2](screenshots/STA_Report_2.png) 
![STA Report 3](screenshots/STA_Report_3.png)
![STA Report 4](screenshots/STA_Report_4.png)
---

## Step 5: Modify Max Fanout and Re-run Synthesis

```bash
./flow.tcl -interactive
package require openlane 0.9
prep -design picorv32a -tag 26-07_06-22 -overwrite
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs
set ::env(SYNTH_SIZING) 1
set ::env(SYNTH_MAX_FANOUT) 4
run_synthesis
```
![Re-run Synthesis](screenshots/Re-run_Synthesis.png)

![Re-run Synthesis](screenshots/Re-run_Synthesis1.png)

Fanout-modified results:

![Fanout Output 1](screenshots/Fanout_Output_1.png) 
![Fanout Output 2](screenshots/Fanout_Output_2.png) 
![Fanout Output 3](screenshots/Fanout_Output_3.png) 
![Fanout Output 4](screenshots/Fanout_Output_4.png) 

---

# Clock Tree Synthesis TritonCTS and Signal Integrity

![Clock Tree Buffer Insertion 1](screenshots/Clock_Tree_Buffer_Insertion_1.png)

![Clock Tree Buffer Insertion 2](screenshots/Clock_Tree_Buffer_Insertion_2.png)

- So we go for `H-Tree` method:

![H-Tree Clock Distribution](screenshots/H-Tree_Clock_Distribution.png)

![Clock H-Tree Visualization](screenshots/Clock_H-Tree_Visualization.png)

- After adding buffers 

![After Buffer Insertion](screenshots/After_Buffer_Insertion.png)

---

### Clock Net Shielding 👇

![Clock Net Shielding Visual 1](screenshots/Clock_Net_Shielding_Visual1.png)

![Clock Net Shielding Visual 2](screenshots/Clock_Net_Shielding_Visual2.png)

- Therefore, the clock net is shielded:

![Final Clock Shielding Result](screenshots/Final_Clock_Shielding_Result.png)

---

### Lab Steps to Run CTS Using TritonCTS

From the previous labs, run until the placement stage using the following commands:

```bash
prep -design picorv32a -tag 26-07_06-22 -overwrite
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

- After placement is done, move to the CTS stage and run:

```bash
run_cts
```

(This uses **TritonCTS** in **OpenROAD**)

---

### CTS Output

![CTS Output 1](screenshots/CTS_Output_1.png)

![CTS Output 2](screenshots/CTS_Output_2.png)

![CTS Output 3](screenshots/CTS_Output_3.png)

![CTS Output 4](screenshots/CTS_Output_4.png)

![CTS Output 5](screenshots/CTS_Output_5.png)

![CTS Output 6](screenshots/CTS_Output_6.png)

---

### Verifying CTS Run

![CTS Verify 1](screenshots/CTS_Verify_1.png)

![CTS Verify 2](screenshots/CTS_Verify_2.png)

![CTS Verify 3](screenshots/CTS_Verify_3.png)


# Timing Analysis with Real Clocks

---

## Setup Time Analysis

![Setup Analysis 1](screenshots/Setup_Analysis_1.png) 
![Setup Analysis 2](screenshots/Setup_Analysis_2.png) 
![Setup Analysis 3](screenshots/Setup_Analysis_3.png)

---

## Hold Time Analysis

![Hold Analysis 1](screenshots/Hold_Analysis_1.png) 
![Hold Analysis 2](screenshots/Hold_Analysis_2.png) 
![Hold Analysis 3](screenshots/Hold_Analysis_3.png) 
![Hold Analysis 4](screenshots/Hold_Analysis_4.png) 
![Hold Analysis 5](screenshots/Hold_Analysis_5.png)

---

## LAB: Analyze Timing with Real Clocks using OpenSTA

We now run `run_cts` and verify CTS outputs.

```bash
run_cts
```

### CTS Output Files

![CTS run output 1](screenshots/CTS_run_output.png) 

![CTS run output 1](screenshots/CTS_run_output1.png)

---

## Post-CTS Timing Analysis with OpenROAD + OpenSTA

Launch OpenROAD and load DEF/LEF for post-CTS timing analysis:

```tcl
openroad

read_lef /openLANE_flow/designs/picorv32a/runs/26-07_06-22/tmp/merged.lef
read_def /openLANE_flow/designs/picorv32a/runs/26-07_06-22/results/cts/picorv32a.cts.def
write_db pico_cts.db
read_db pico_cts.db
read_verilog /openLANE_flow/designs/picorv32a/runs/26-07_06-22/results/synthesis/picorv32a.synthesis_cts.v
read_liberty $::env(LIB_TYPICAL)
link_design picorv32a
read_sdc /openLANE_flow/designs/picorv32a/src/my_base.sdc
set_propagated_clock [all_clocks]

report_checks -path_delay min_max -format full_clock_expanded -digits 4
```

### Report Output:

![Report check 1](screenshots/Report_check_1.png) 
![Report check 2](screenshots/Report_check_2.png) 
![Report check 3](screenshots/Report_check_3.png)

---

## LAB: Observe Impact of Bigger CTS Buffers

Temporarily remove smallest buffer:

```tcl
echo $::env(CTS_CLK_BUFFER_LIST)
set ::env(CTS_CLK_BUFFER_LIST) [lreplace $::env(CTS_CLK_BUFFER_LIST) 0 0]
```

Reset DEF:

```tcl
set ::env(CURRENT_DEF) /openLANE_flow/designs/picorv32a/runs/26-07_06-22/results/placement/picorv32a.placement.def
run_cts
echo $::env(CTS_CLK_BUFFER_LIST)
```

Re-run analysis:

```tcl
openroad

read_lef /openLANE_flow/designs/picorv32a/runs/26-07_06-22/tmp/merged.lef
read_def /openLANE_flow/designs/picorv32a/runs/26-07_06-22/results/cts/picorv32a.cts.def
write_db pico_cts1.db
read_db pico_cts1.db
read_verilog /openLANE_flow/designs/picorv32a/runs/26-07_06-22/results/synthesis/picorv32a.synthesis_cts.v
read_liberty $::env(LIB_TYPICAL)
link_design picorv32a
read_sdc /openLANE_flow/designs/picorv32a/src/my_base.sdc
set_propagated_clock [all_clocks]

report_checks -path_delay min_max -fields {slew trans net cap input_pins} -format full_clock_expanded -digits 4

report_clock_skew -hold
report_clock_skew -setup
```

### Output:

![Impact buffer 1](screenshots/Impact_buffer_1.png) 
![Impact buffer 2](screenshots/Impact_buffer_2.png) 
![Impact buffer 3](screenshots/Impact_buffer_3.png) 
![Impact buffer 4](screenshots/Impact_buffer_4.png)

---

## LAB: Insert Specific Buffer (e.g., clkbuf_1)

Insert a specific CTS buffer to manually control buffer impact:

```tcl
echo $::env(CTS_CLK_BUFFER_LIST)
set ::env(CTS_CLK_BUFFER_LIST) [linsert $::env(CTS_CLK_BUFFER_LIST) 0 sky130_fd_sc_hd__clkbuf_1]
```

![Buffer insert](screenshots/Buffer_insert.png)

---

## Buffer Timing Comparison

```
For clkbuf_1
    Hold slack  =   0.1127  (MET)
    Setup slack =  13.8266  (MET)

For clkbuf_2
    Hold slack  =   0.2892  (MET)
    Setup slack =  13.8266  (MET)
```

- **Conclusion**: Using larger CTS buffers (like `clkbuf_2`) improves hold slack.

---

##  Summary

|  Topic                              | ✅ Description                                                                 |
|--------------------------------------|--------------------------------------------------------------------------------|
| Grid to Track Alignment              | Aligned Magic layout to routing tracks by ensuring port placement and dimensions. |
| Standard Cell LEF Generation         | Converted `.mag` layout of `vsdinv` to `.lef` using `magic` and `lef` command.   |
| Delay Table Modeling                 | Learned delay behavior vs. input slope & load; validated delay symmetry.        |
| Synthesis Configuration              | Used `SYNTH_SIZING`, `SYNTH_STRATEGY`, and `SYNTH_MAX_FANOUT` to improve slack. |
| STA with Ideal Clocks                | Performed timing analysis post-synthesis with `OpenSTA` using ideal clocks.     |
| CTS with TritonCTS                   | Routed clock tree with H-tree buffering and measured skew improvement.          |
| Real Clock Timing Analysis           | Configured clock propagation and buffers; analyzed setup/hold after CTS.        |



## Observations

| Test Condition                     | TNS (ns) | WNS (ns) | Hold Slack (ns) | Result                 | Notes                           |
|--------------------------------------|----------|----------|------------------|-------------------------|---------------------------------|
| Initial synthesis                    | -711.59  | -23.89   | —                | ❌ Slack Violation       | No sizing/strategy applied      |
| After sizing & strategy              | 0        | 0        | —                | ✅ Slack Fixed           | Applied `SYNTH_STRATEGY=2`      |
| Fanout optimization (`fanout=4`)     | 0        | 0        | —                | ✅ Stable Slack          | Reduced output load per gate    |
| Post-CTS with `clkbuf_1`             | —        | —        | 0.1127           | ✅ Acceptable Hold       | Balanced tree, default buffer   |
| Post-CTS with stronger `clkbuf_2`    | —        | —        | 0.2892           | ✅ Improved Hold         | Better delay with larger buffer |




