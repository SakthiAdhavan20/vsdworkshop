# 🗓️ Sky130 Day 1 – Inception of Open-Source EDA, OpenLANE and Sky130 PDK

---

## 🎯 Objective

- Understand how software applications become hardware using RTL to GDSII flow  
- Learn about SoC components: chips, dies, cores, pads, and IPs  
- Introduction to RISC-V and open-source EDA tools  
- Prepare and run synthesis in OpenLANE using Sky130  
- Characterize synthesized design using flop ratio  

---

## 📘 Theory Section

### 🔸 0. Introduction to QFN-48 Package, Chip, Pads, Core, Die, and IPs

- **Chip**: The actual silicon die that holds your digital logic  
- **Package (QFN-48)**: Holds and protects the chip; has 48 pins for connection  
- **Pads**: Metal terminals on the die used for bonding with package pins  
- **Core**: The active region of the die (excluding pads and IOs)  
- **IP Blocks**: Pre-designed logic like PLLs, memories, etc., reused across chips  

📷 Screenshot:  
![Processor/SoC](Screenshots/Processor-SoC.png)

📷 Screenshot:  
![ATMega32u4-RISC-based-microcontroller](Screenshots/ATMega32u4%20RISC-based%20microcontroller.png)


📷 Screenshot:  
![chip-package-core](Screenshots/chip-package-core.png)

---

### 🔸 1. Introduction to RISC-V

- **RISC-V** is an open-source Instruction Set Architecture (ISA)  
- Used widely in open-source and academic processor designs  
- Base ISA is simple, modular, and extensible  

📷 Screenshot:  
![riscv-diagram](Screenshots/riscv-diagram.png)

---

### 🔸 2. From Software Applications to Hardware

- Software is written in C, compiled to assembly (e.g., RISC-V), executed on hardware  
- Hardware implements ISA logic using digital circuits  
- VLSI engineers design the underlying digital hardware that executes software  

📷 Screenshot:  
![software-to-hardware](Screenshots/software-to-hardware.png)

---

### 🔸 3. Open-Source Digital ASIC Design Components

| Component | Tool Used       | Role                        |
|-----------|------------------|-----------------------------|
| Synthesis | Yosys            | RTL to gate-level netlist  |
| Floorplan | OpenLANE scripts | Macro and IO placement     |
| Layout    | Magic            | View/edit physical layout  |
| Checks    | Netgen/Klayout   | DRC, LVS, PEX              |

📷 Screenshot:  
![open-source-eda-stack](Screenshots/open-source-eda-stack.png)

---

### 🔸 4. Simplified RTL2GDSII Flow

1. Write RTL (Verilog)  
2. Synthesis → Netlist  
3. Floorplan → Placement → CTS → Routing  
4. DRC + LVS  
5. Export GDSII for tapeout  

📷 Screenshot:  
![rtl2gdsii-simplified](Screenshots/rtl2gdsii-simplified.png)

---

### 🔸 5. OpenLANE Detailed ASIC Design Flow

- Each OpenLANE step has configurable TCL scripts  
- Fully open-source RTL to GDSII automation  
- Output logs for every stage are stored inside `/runs/<design-name>/`  

📷 Screenshot:  
![openlane-detailed-flow](Screenshots/openlane-detailed-flow.png)

---

### 🔸 6. OpenLANE Directory Structure in Detail

Run this from the `openlane/` folder:

```bash
cd ~/Desktop/work/tools/openlane_working_dir/openlane/
tree vsdstdcelldesign
```

Sample Output:
```
vsdstdcelldesign/
├── config.tcl
├── runs/
│   └── vsdstdcelldesign/
│       ├── logs/
│       ├── results/
│       ├── tmp/
```

📷 Screenshot:  
![openlane-dir-structure](Screenshots/openlane-dir-structure.png)

---

## 🧪 Practical Section – Lab Steps for Synthesis

### 🔸 7. Design Preparation Step

```bash
cd ~/Desktop/work/tools/openlane_working_dir/openlane/
./flow.tcl -interactive
```

Then in the TCL prompt:

```tcl
prep -design vsdstdcelldesign
```

📤 Output:
```
[INFO]: Creating directory ./runs/vsdstdcelldesign
[INFO]: Copying source files...
[INFO]: Preparation complete.
```

📷 Screenshot:  
![design-prep-output](Screenshots/design-prep-output.png)

---

### 🔸 8. Run Synthesis + Review Files

```tcl
run_synthesis
```

📤 Output:
```
[INFO]: Number of cells: 14876
[INFO]: Number of DFF_X1 cells: 1613
[INFO]: Synthesis complete!
```

📷 Screenshot:  
![synthesis-log-output](Screenshots/synthesis-log-output.png)

Check the log file manually:

```bash
runs/vsdstdcelldesign/logs/synthesis/1-yosys_0.log
```

📷 Screenshot:  
![synthesis-log-file](Screenshots/synthesis-log-file.png)

---

### 🔸 9. Steps to Characterize Synthesis Results

Formula:

```
Flop Ratio = (Number of DFFs / Total Cells) * 100
           = (1613 / 14876) * 100 ≈ 10.84%
```

📷 Screenshot:  
![flop-ratio-calculation](Screenshots/flop-ratio-calculation.png)  
📷 Screenshot:  
![highlight-yosys-log](screenshots/highlight-yosys-log.png)

---

## ✅ Summary

| Step                       | Status | Notes                            |
|----------------------------|--------|----------------------------------|
| Theory + Chip Basics       | ✅     | QFN, RISC-V, RTL2GDSII explained |
| Tool Setup + Directory     | ✅     | Screenshot and structure shown   |
| Design Prep                | ✅     | `prep -design` complete          |
| Synthesis Run + Output     | ✅     | Logs + DFF count visible         |
| Flop Ratio Characterization| ✅     | 10.84% ratio calculated          |

---

