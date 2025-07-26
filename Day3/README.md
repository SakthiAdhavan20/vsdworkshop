# Day 3 - Design Library Cell using Magic Layout and ngspice Characterization

## 1. Introduction

In this session, we explore the physical layout of a CMOS inverter using the Magic VLSI tool and perform SPICE-level characterization using `ngspice` based on the Sky130 PDK. We also dive into DRC (Design Rule Check) debugging and LEF extraction.

---

## 2. SPICE Deck and ngspice Simulation

### 2.1 SPICE Deck Creation for CMOS Inverter

To simulate the inverter's behavior at the transistor level, we create a SPICE netlist that includes transistor definitions, node connectivity, load capacitance, and simulation parameters. This netlist is executed using `ngspice`, which allows us to observe the transient switching characteristics of the circuit.

The SPICE deck is typically saved as `sky130_inv.spice` in the same directory as the layout file created in Magic.

---

#### A typical SPICE deck includes:

- Node definitions (`vdd`, `in`, `0`)
- PMOS and NMOS transistor descriptions with connectivity (Drain-Gate-Source-Bulk)
- Netlist describing circuit structure
- Load capacitance at the output node
- Input signal source (`PULSE` waveform)
- Power rail setup (`vdd` and `gnd`)
- Simulation control directives (e.g., `.tran`, `.dc`)
- Library models for Sky130 PMOS and NMOS devices

---

📷 Screenshot:  
![spice1](screenshots/spice1.png)

📷 Screenshot:  
![spice2](screenshots/spice2.png)

📷 Screenshot:  
![spice3](screenshots/spice3.png)

📷 Screenshot:  
![spice4](screenshots/spice4.png)

---

## 3. CMOS Fabrication Process – Layout Inception

We visualize each fabrication step layer-by-layer using Magic.

### 3.1 Active Region  
📷 Screenshot:  
![active](screenshots/active.png)

### 3.2 N-well and P-well  
📷 Screenshot:  
![well](screenshots/well.png)

### 3.3 Gate Terminal Formation  
📷 Screenshot:  
![gate](screenshots/gate.png)

### 3.4 LDD Formation  
📷 Screenshot:  
![ldd](screenshots/ldd.png)

### 3.5 Source/Drain Formation  
📷 Screenshot:  
![source_drain](screenshots/source_drain.png)

### 3.6 Local Interconnect Formation  
📷 Screenshot:  
![local_interconnect](screenshots/local_interconnect.png)

### 3.7 Higher-Level Metal Formation  
📷 Screenshot:  
![metal](screenshots/metal.png)

### 3.8 Final Complete Layout  
📷 Screenshot:  
![layout_final](screenshots/layout_final.png)

📷 Screenshot:  
![layout_combined](screenshots/layout_combined.png)

---

## 4. Layout and LEF Extraction

### 4.1 Clone Inverter Layout Design

```bash
git clone https://github.com/nickson-jose/vsdstdcelldesign.git
```

📷 Screenshot:  
![git_clone](screenshots/66.png)

### 4.2 Open Inverter Layout in Magic

```bash
magic -T sky130A.tech sky130_inv.mag
```

📷 Screenshot:  
![magic_open](screenshots/67.png)

📷 Screenshot:  
![magic_layers1](screenshots/68.png)

📷 Screenshot:  
![magic_layers2](screenshots/69.png)

📷 Screenshot:  
![magic_layers3](screenshots/70.png)

📷 Screenshot:  
![magic_layers4](screenshots/71.png)

📷 Screenshot:  
![magic_layers5](screenshots/72.png)

📷 Screenshot:  
![magic_layers6](screenshots/73.png)

---

### 4.3 Extract LEF and SPICE Netlist

```bash
extract all
ext2spice sky130_inv.ext
ext2spice
```

📷 Screenshot:  
![ext2spice](screenshots/74.png)

📷 Screenshot:  
![ext_files](screenshots/75.png)

📷 Screenshot:  
![ext_content](screenshots/76.png)

---

## 5. Final SPICE Deck with Sky130 Tech

### 5.1 Modify `.option scale`  
📷 Screenshot:  
![scale_option](screenshots/77.png)

### 5.2 Use Correct Models  
- `pshort_model.0` for PMOS  
- `nshort_model.0` for NMOS  

📷 Screenshot:  
![lib_model](screenshots/79.png)

📷 Screenshot:  
![final_spice](screenshots/78.png)

---

## 6. Simulation and Characterization

### 6.1 Run ngspice

```bash
ngspice sky130_inv.spice
plot v(a) v(y)
```

📷 Screenshot:  
![ngspice_run](screenshots/80.png)

📷 Screenshot:  
![ngspice_plot](screenshots/81.png)

📷 Screenshot:  
![initial_waveform](screenshots/82.png)

### 6.2 Add Capacitance to Reduce Noise

📷 Screenshot:  
![noisy_wave](screenshots/83.png)

📷 Screenshot:  
![clean_wave](screenshots/84.png)

---

## 7. Delay and Transition Analysis

### 7.1 Rise Transition

- 20% of Vmax = 0.66V  
- 80% of Vmax = 2.64V  

📷 Screenshot:  
![rise_zoom1](screenshots/85.png)

📷 Screenshot:  
![rise_zoom2](screenshots/86.png)

📷 Screenshot:  
![rise_terminal](screenshots/87.png)

```
Rise Transition = 2.239 - 2.180 = 0.059ns
```

### 7.2 Rise Delay

📷 Screenshot:  
![rise_delay](screenshots/88.png)

📷 Screenshot:  
![rise_delay_terminal](screenshots/89.png)

```
Rise Delay = 2.207 - 2.150 = 0.057ns
```

### 7.3 Fall Transition

📷 Screenshot:  
![fall_transition_terminal](screenshots/91.png)

```
Fall Transition = 4.093 - 4.050 = 0.0428ns
```

### 7.4 Fall Delay

📷 Screenshot:  
![fall_delay_terminal](screenshots/90.png)

```
Fall Delay = 4.075 - 4.050 = 0.025ns
```

---

## 8. Sky130 Tech DRC Rules and Debugging

### 8.1 Download and Open DRC Labs

```bash
wget http://opencircuitdesign.com/open_pdks/archive/drc_tests.tgz
tar xfz drc_tests.tgz
magic -d XR
```

📷 Screenshot:  
![drc_download](screenshots/92.png)

📷 Screenshot:  
![magic_xr](screenshots/93.png)

### 8.2 Inspect and Fix DRC

📷 Screenshot:  
![met3_open](screenshots/94.png)

📷 Screenshot:  
![met3_inspect](screenshots/95.png)

📷 Screenshot:  
![met3_zoom](screenshots/96.png)

📷 Screenshot:  
![poly_open](screenshots/97.png)

📷 Screenshot:  
![poly_fix1](screenshots/98.png)

📷 Screenshot:  
![poly_fix2](screenshots/99.png)

📷 Screenshot:  
![poly_fix3](screenshots/100.png)

### 8.3 Reload Tech File and Recheck DRC

```bash
tech load sky130A.tech
drc check
```

📷 Screenshot:  
![load_tech](screenshots/101.png)

📷 Screenshot:  
![drc_pass](screenshots/102.png)

---


