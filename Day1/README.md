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

📷 Screenshot:  
![isa-to-hardware](Screenshots/isa-to-hardware.png)       

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

### 🔸 6. sky130A PDK Directory Structure

Once OpenLANE is installed, the Platform Development Kit (PDK) for Sky130 is downloaded under the path:

`~/Desktop/work/tools/openlane_working_dir/pdks/sky130A/`

---

Run the following to explore the PDK structure:

```bash
cd ~/Desktop/work/tools/openlane_working_dir/pdks/sky130A/
ls -ltr
```

📂 Sample Output:
```
libs.tech/     # Technology-specific data: magic, ngspice, klayout, etc.
libs.ref/      # Reference libraries: standard cell libraries, IOs, macros
SOURCES        # Source info file for PDK installation
```

---

📘 Inside `libs.tech/` – Contains tools-related tech files

```bash
cd libs.tech
ls -ltr
```

📂 Output:
```
magic/       # For layout editing (used by Magic tool)
ngspice/     # For simulation models
netgen/      # For LVS checking
openlane/    # Tech data used by OpenLANE
klayout/     # For viewing layouts
irsim/       # For digital simulation
xschem/      # For schematic capture
xcircuit/    # Alternative schematic tool
qflow/       # Digital flow tool
```

---

📘 Inside `libs.ref/` – Contains standard cells, IOs, macros

```bash
cd ../libs.ref
ls -ltr
```

📂 Output:
```
sky130_fd_sc_hd/     # High Density standard cells
sky130_fd_sc_hs/     # High Speed standard cells
sky130_fd_sc_ms/     # Medium Speed standard cells
sky130_fd_sc_ls/     # Low Speed standard cells
sky130_fd_sc_lp/     # Low Power standard cells
sky130_fd_sc_hdll/   # High Density Low Leakage
sky130_fd_sc_hvl/    # High Voltage Libraries
sky130_fd_io/        # IO Cells
sky130_sram_macros/  # SRAM Macros
sky130_ml_xx_hd/     # Mixed-signal libraries
sky130_fd_pr/        # Primitive devices (transistors, resistors, etc.)
sky130_osu_sc_t18/   # OSU-based standard cells (for experimentation)
```

---

📷 Screenshot:  
![sky130A PDK Directory](Screenshots/sky130A-pdk-director.png)


---

## 🧪 Practical Section – Lab Steps for Synthesis

### 🔸 7.OpenLANE Flow – Picorv32a Design (Sky130 PDK)

This section documents running the `picorv32a` design using OpenLANE v0.21 and Sky130A PDK. The flow includes preparing the interactive session, running the `prep` step, and checking output directories.

---

#### ✅ Step 1 – Launch Docker and Start OpenLANE Flow

```bash
cd ~/Desktop/work/tools/openlane_working_dir/openlane/
docker
```

Inside Docker shell:

```tcl
./flow.tcl -interactive
```

Then at `%` prompt:

```tcl
package require openlane 0.9
```

📷 Screenshot:  
![Step 1 - Docker Interactive](Screenshots/step1_docker_interactive.png)

---

#### ✅ Step 2 – Prepare the Picorv32a Design

Inside the interactive OpenLANE shell:

```tcl
prep -design picorv32a
```

This step:

- Loads configuration from `config.tcl`
- Initializes PDK and Standard Cells
- Merges LEF/Liberty files
- Final message: **Preparation complete**

📷 Screenshot:  
![Step 2 - Prep Design check](Screenshots/step2_prep_design_check.png)

📷 Screenshot:  
![Step 2 - Prep Design](Screenshots/step2_prep_design.png)

---

#### ✅ Step 3 – Explore Design Run and Temporary Files

```bash
cd designs/picorv32a/runs/
cd 26-07_06-22/
cd tmp/
ls -ltr
```

You can now access all intermediate folders:

- `floorplan/`, `synthesis/`, `routing/`, etc.
- Files like `merged.lef`, `trimmed.lib`, `met_layers_list.txt`

📷 Screenshot:  
![Step 3 - Explore tmp Folder](Screenshots/tep3_explore_tmp_folder.png)

---

---

### 🔸 8. Run Synthesis + Review Files

```bash
run_synthesis
```

After running the above command, OpenLANE performs RTL synthesis using **Yosys** and generates a synthesized gate-level netlist for the design `picorv32a`.

---

#### 8.1 Synthesis Statistics – `picorv32a`

After synthesis is complete, the following statistics are printed:

```
=== picorv32a ===

Number of wires:              14596  
Number of wire bits:          14978  
Number of public wires:        1565  
Number of public wire bits:    1947  
Number of memories:               0  
Number of memory bits:            0  
Number of processes:              0  
Number of cells:              14876
```

Below is a partial breakdown of the most-used standard cells:

| Cell Name                  | Count | Description         |
|---------------------------|-------|---------------------|
| sky130_fd_sc_hd__dfxtp_2  | 1613  | D Flip-Flop         |
| sky130_fd_sc_hd__inv_2    | 1615  | Inverter            |
| sky130_fd_sc_hd__mux2_1   | 1224  | 2:1 Multiplexer     |
| sky130_fd_sc_hd__or2_2    | 1088  | 2-input OR gate     |
| sky130_fd_sc_hd__a2bb2o_2 | 1748  | Complex logic gate  |
| ...                       | ...   | ...                 |

---

##### D Flip-Flop Ratio

We calculate the ratio of sequential elements (flip-flops) to total cells:

- **Total cells** = 14876  
- **D Flip-Flops (dfxtp_2)** = 1613  
- **DFF Ratio** = 1613 / 14876 ≈ **10.84%**

This gives insight into the sequential vs. combinational logic density.

---

#### 8.2 Verilog Backend & Log Summary

After synthesis, OpenLANE triggers the **Verilog backend**, which:

- Optimizes logic via `abc`
- Simplifies expressions via `opt_expr`
- Maps cells to standard library
- Dumps synthesized Verilog for `picorv32a`

📝 **Yosys Info**:

- Version: 0.9+3621  
- Optimizations: `abc`, `opt_expr`, etc.  
- Backend Module: `\picorv32a`  
- Synthesized Netlist Output:  
  `/openLANE_flow/designs/picorv32a/runs/<run>/results/synthesis/picorv32a.synthesis.v`

---

#### 8.3 Static Timing Analysis (STA)

OpenLANE then runs **OpenSTA** for post-synthesis timing analysis.

```tcl
set input_delay  = 4.946 ns  
set output_delay = 4.946 ns  
set load         = 0.01765
```

⚠️ *STA Warnings (liberty files)*:

- Some operating conditions not found:  
  `ff_n40C_1v95`, `ss_100C_1v60`  
- These will be resolved in later steps when proper corners are set.

---

#### 8.4 Timing Summary

| Metric | Value       |
|--------|-------------|
| TNS    | -759.46 ns  |
| WNS    | -24.89 ns   |

 *Negative slack* indicates **timing violations** that will be addressed in later stages (placement, CTS, routing).


📷 Screenshot:
![run_synthesis_summary](screenshots/run_synthesis_summary1.png)

![run_synthesis_summary](screenshots/run_synthesis_summary2.png)

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

