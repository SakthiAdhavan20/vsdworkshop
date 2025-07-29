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

![STA  File 1](screenshots/142.png)

![STA  File 2](screenshots/142.png)

![STA  File 3](screenshots/142.png)

![STA  File 4](screenshots/142.png)

![STA  File 5](screenshots/142.png)

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

![STA Configuration File](screenshots/142.png)

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

![SDC Configuration File](screenshots/143.png)

---

## Step 4: Run Timing Analysis using OpenSTA

```bash
sta pre_sta.conf
```

Sample output:

![STA Report 1](screenshots/144.png) 
![STA Report 2](screenshots/145.png) 
![STA Report 3](screenshots/146.png)

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

Fanout-modified results:

![Fanout Output 1](screenshots/147.png) 
![Fanout Output 2](screenshots/148.png) 
![Fanout Output 3](screenshots/149.png) 
![Fanout Output 4](screenshots/150.png) 
![Fanout Output 5](screenshots/151.png)

---


