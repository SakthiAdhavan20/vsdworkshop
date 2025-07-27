# Day 4 – Pre-layout Timing Analysis and Clock Tree Optimization

---

## 1. Convert Grid Info to Track Info and Verify Magic Layout

Before integrating a custom standard cell into the PnR flow, we need to ensure it aligns with the design track grid.

**Conditions to Meet:**

- Ports must lie on intersections of horizontal and vertical tracks.  
- Width should be an odd multiple of horizontal pitch.  
- Height should be an even multiple of vertical pitch.

### Grid and Track Info:

![Track Info 1](../images/103.png)  
![Track Info 2](../images/104.png)

### Initial grid view in Magic:

![Initial Grid View](../images/105.png)

### Resize Grid in Magic:

```tcl
grid 0.46um 0.34um 0.23um 0.17um
```

![After Resize](../images/106.png)  
![Final Grid](../images/107.png)

---

## 2. Convert Magic Layout to LEF

In the Magic layout window:

```tcl
lef write sky130_inv.lef
```

This generates a standard cell LEF file that will be used in the OpenLANE flow.

![LEF Generation](../images/108.png)

---

## 3. Add Custom Cell to OpenLANE Setup

Open `config.tcl` of the design inside OpenLANE:

```tcl
set ::env(EXTRA_LEFS) "$::env(DESIGN_DIR)/src/sky130_inv.lef"
set ::env(EXTRA_GDS_FILES) "$::env(DESIGN_DIR)/src/sky130_inv.gds"
```

![Config Edit](../images/109.png)

Make sure the custom cell Verilog and Liberty timing model (`sky130_inv.v` and `sky130_inv.lib`) are also copied into the design `src/` folder.

---

## 4. Run Synthesis in OpenLANE

```bash
./flow.tcl -interactive
```

```tcl
package require openlane 0.9
prep -design picorv32a -tag 16-03_17-49 -overwrite

set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs

run_synthesis
```

![Synthesis 1](../images/115.png)  
![Synthesis 2](../images/116.png)

---

## 5. Inverter Cell in Netlist

Check the synthesized netlist for the inverter cell:

```bash
cd runs/picorv32a/results/synthesis/
gedit picorv32a.synthesis.v
```

Search for `sky130_inv`.

![Netlist Search](../images/117.png)

---

## 6. Include Inverter Cell in Liberty

Make sure `sky130_inv.lib` is added in config.tcl:

```tcl
set ::env(LIB_SYNTH) "$::env(DESIGN_DIR)/src/sky130_inv.lib"
```

![Liberty Include](../images/118.png)

---

## 7. Slack Violation and Delay Analysis

You can now simulate scenarios where slack violations occur, and analyze how `sky130_inv` can be inserted to improve delay.

![Slack Info](../images/119.png)

---

## 8. Timing Analysis with Ideal Clock

```tcl
run_floorplan
run_placement
run_cts
run_sta
```

Observe slack and critical paths.

![Timing STA 1](../images/137.png)  
![Timing STA 2](../images/138.png)  
![Timing STA 3](../images/139.png)

---

## 9. Delay Table Overview

Delay tables capture delay across combinations of input transition and output load.

| Input Slew | Output Load | Delay (ns) |
|------------|-------------|------------|
| 0.1ns      | 5fF         | 0.22       |
| 0.2ns      | 10fF        | 0.27       |

These are extracted using SPICE-level simulation and used in `.lib` characterization.

![Delay Table](../images/140.png)

---

## 10. OpenSTA Analysis with Custom Inverter

You can now re-run OpenSTA with a scenario where `sky130_inv` is placed between critical nodes.

```tcl
read_verilog sky130_inv.v
link_design sky130_inv
```

Analyze the new path and compare slack.

![STA with Custom INV](../images/141.png)

---

## 11. Run TritonCTS for Real Clock Network

```tcl
run_cts
```

This performs H-Tree based clock distribution and inserts clock buffers.

![CTS Result 1](../images/142.png)  
![CTS Result 2](../images/143.png)

---

## 12. Setup and Hold Analysis with Real Clock

After CTS:

```tcl
run_sta
```

![STA Real Clock 1](../images/144.png)  
![STA Real Clock 2](../images/145.png)

---

## 13. Slack Optimization using Inverter

Analyze where to insert the inverter to adjust delays and improve slack.

![Slack Fix Inverter](../images/146.png)

---

## 14. Crosstalk and Shielding

High fanout and long clock nets may suffer from crosstalk.

Solution: Shield them with power or ground.

![Crosstalk Fix](../images/147.png)

---

## 15. Summary

- We created a custom inverter cell and extracted LEF/lib.
- Integrated it into OpenLANE synthesis and STA.
- Used it to fix setup slack violations.
- Performed CTS and analyzed signal integrity.

---

