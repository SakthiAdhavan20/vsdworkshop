# Sky130 Day 3 - Design Library Cell using Magic Layout and ngspice Characterization

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

  
![spice1](screenshots/spice1.png)

  
![spice2](screenshots/spice2.png)

 
![spice3](screenshots/spice3.png)

  
![spice4](screenshots/spice4.png)

---

## 3. CMOS Fabrication Process – Layout Inception

We visualize each fabrication step layer-by-layer using Magic.

### 3.1 Active Region  
  
![active](screenshots/active.png)

### 3.2 N-well and P-well  
  
![well](screenshots/well.png)

### 3.3 Gate Terminal Formation  
  
![gate](screenshots/gate.png)

### 3.4 LDD Formation  
  
![ldd](screenshots/ldd.png)

### 3.5 Source/Drain Formation  
 
![source_drain](screenshots/source_drain.png)

### 3.6 Local Interconnect Formation  
 
![local_interconnect](screenshots/local_interconnect.png)

### 3.7 Higher-Level Metal Formation  
  
![metal](screenshots/metal.png)

### 3.8 Final Complete Layout  
 
![layout_final](screenshots/layout_final.png)

  
![layout_combined](screenshots/layout_combined.png)

---

## 4. Layout and LEF Extraction

### 4.1 Clone Inverter Layout Design

- to work in the futher labs clone(download) the below repository at the location `/Desktop/work/tools/openlane_working_dir/openlane`

```bash
git clone https://github.com/nickson-jose/vsdstdcelldesign.git
```
 
![git_clone](screenshots/git_clone.png)

- these are the files and folders are in the `vsdstdcelldesign` repo

### 4.2 Open Inverter Layout in Magic

- to open the `sky130_inv.mag` file in magic the command is

```bash
magic -T sky130A.tech sky130_inv.mag
```

- the image below is an inverter layout

![magic_open](screenshots/magic_open.png)

- the images below shows about different layers and interconnects in the inverter layout
 
![magic_layers1](screenshots/magic_layers1.png)

  
![magic_layers2](screenshots/magic_layers2.png)

  
![magic_layers3](screenshots/magic_layers3.png)

  
![magic_layers4](screenshots/magic_layers4.png)

  
![magic_layers5](screenshots/magic_layers5.png)

 
![magic_layers6](screenshots/magic_layers6.png)

---

### 4.3 Extract LEF and SPICE Netlist

- to extract lef file from layout using the below commands in the `tkon` window 

```bash
extract all
ext2spice sky130_inv.ext
ext2spice
```
![ext2spice](screenshots/ext2spice.png)

- two files `sky130_inv.ext` and `sky130_inv.spice` are created
  
![ext_files](screenshots/ext_files.png)

- the `sky130_inv.ext` file is containing the following contents 
 
![ext_content](screenshots/ext_content.png)

---

## 5. Final SPICE Deck with Sky130 Tech

### 5.1 Modify `.option scale`:  

- Editing `.option scale` – 1
![scale_option](screenshots/scale_option1.png)

- Editing `.option scale` – 2 
![scale_option](screenshots/scale_option2.png)

- Final `.option scale` line
![scale_option](screenshots/scale_option.png)


### 5.2 Use Correct Models  

- `pshort_model.0` for PMOS 

- `nshort_model.0` for NMOS

- PMOS and NMOS models added – 1 
![lib_model](screenshots/lib_model.png)

- PMOS and NMOS models added – 2
![lib_model](screenshots/lib_model1.png)

- Final SPICE deck with Sky130 models
  
![final_spice](screenshots/final_spice.png)

---

## 6. Simulation and Characterization

### 6.1 Run ngspice

```bash
ngspice sky130_inv.spice
plot v(a) v(y)
```

- Terminal view of ngspice run 
![ngspice_run](screenshots/ngspice_run.png)

- Output waveform from simulation
  
![ngspice_plot](screenshots/ngspice_plot.png)


### 6.2 Add Capacitance to Reduce Noise

- Final SPICE deck with added capacitance

![final_spice](screenshots/final_spice1.png)

- Noisy output waveform
 
![noisy_wave](screenshots/noisy_wave.png)

- Clean output waveform
 
![clean_wave](screenshots/clean_wave.png)

---

## 7. Delay and Transition Analysis

### 7.1 Rise Transition

- 20% of Vmax = 0.66V 

- 80% of Vmax = 2.64V 

- 20% rise voltage point
![rise_zoom1](screenshots/rise_zoom1.png)

- 80% rise voltage point 
![rise_zoom2](screenshots/rise_zoom2.png)

```
Rise Transition = 2.24 - 2.04 = 0.2ns
```

### 7.2 Rise Delay

  
![rise_delay](screenshots/rise_delay.png)

```
Rise Delay = 2.2466 - 2.21016 = 0.03644ns
```

### 7.3 Fall Transition

  
![fall_transition_terminal](screenshots/fall_transition_terminal.png)

```
Fall Transition = 4.05532 - 4.053 = 0.00232ns
```

### 7.4 Fall Delay

  
![fall_delay_terminal](screenshots/fall_delay_terminal.png)

```
Fall Delay = 4.055 - 4.038 = 0.017ns
```

---

## 8. Sky130 Tech DRC Rules and Debugging

### 8.1 Download and Open DRC Labs

```bash
wget http://opencircuitdesign.com/open_pdks/archive/drc_tests.tgz
tar xfz drc_tests.tgz
magic -d XR
```


![drc_download](screenshots/drc_download.png)


![magic_xr](screenshots/magic_xr.png)

### 8.2 Inspect and Fix DRC

 
![met3_open](screenshots/met3_open.png)

 
![met3_inspect](screenshots/met3_inspect.png)


![met3_zoom](screenshots/met3_zoom.png)


  
![poly_open](screenshots/poly_open.png)


![poly_fix1](screenshots/poly_fix1.png)

 
![poly_fix2](screenshots/poly_fix2.png)



### 8.3 Reload Tech File and Recheck DRC

```bash
tech load sky130A.tech
drc check
```

 
![load_tech](screenshots/load_tech.png)

![load_tech](screenshots/load_tech1.png)

  
![drc_pass](screenshots/drc_pass.png)

![drc_pass](screenshots/drc_pass1.png)


---


