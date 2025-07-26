# 📦 Floorplanning in OpenLANE

## 📘 What is Floorplanning?

Floorplanning defines how your circuit is laid out inside the chip. It decides the size and shape of the core, where pins and power rails go, and where components can be placed.

Key concepts:
```shell
Utilization Factor = Area occupied by netlist / Total core area
Aspect Ratio      = Height / Width
```

👉 A Utilization Factor between **0.5–0.6** is ideal.  
👉 100% utilization = no space for routing = ❌ bad design.

---

### 🧠 Key Considerations:

1. **Utilization & Aspect Ratio**  
   - Keep space for routing and buffers.

2. **Pre-placed Cells**  
   - Fixed logic blocks manually placed before auto placement.

3. **Decoupling Capacitors**  
   - Placed near high-current logic blocks to maintain voltage stability.

4. **Power Rails**  
   - Mesh structure with multiple VDD/VSS pins to avoid noise issues.

5. **IO Pin Placement**  
   - Pins are placed between the core and die boundary.

6. **Placement Blockages**  
   - Prevent cells from being placed over pins or blocked areas.

<p align="center">
  <img width="750" height="400" src="../images/20.png">
</p>

<p align="center">
  <img width="1000" height="600" src="../images/19.png">
</p>

---

## 🧪 LAB: Floorplan in OpenLANE

---

### 📂 Step 1: Config Files

Go to OpenLANE's configuration directory:

```bash
cd openlane/configuration/
```

📁 Priority of config files:
1. `floorplan.tcl` – lowest
2. `config.tcl`
3. `sky130_fd_sc_hd_config.tcl` – highest

<p align="center">
  <img width="1000" height="250" src="../images/21.png">
</p>

<p align="center">
  <img width="800" height="400" src="../images/24.png">
</p>

<p align="center">
  <img width="800" height="350" src="../images/23.png">
</p>

<p align="center">
  <img width="800" height="250" src="../images/22.png">
</p>

---

### ▶️ Step 2: Run Floorplan

Enter OpenLANE shell:

```tcl
% run_floorplan
```

DEF file will be saved at:

```
openlane/designs/picorv32a/runs/<date>/results/floorplan/picorv32a.floorplan.def
```

<p align="center">
  <img width="900" height="450" src="../images/diearea.png">
</p>

---

### 🧮 Step 3: Die Area Calculation

```shell
DIEAREA (0 0) (554570 565290)
Unit distance = 1000
Area = (554.57 µm * 565.29 µm) ≈ 311829.16 µm²
```

📖 [Read about DEF](https://ivlsi.com/design-exchange-format-def-in-vlsi-physical-design/) | [Official DEF Spec](https://coriolis.lip6.fr/doc/lefdef/lefdefref/LEFSyntax.html)

---

### 🧰 Step 4: Open DEF in Magic

```bash
magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech \
      lef read ../../tmp/merged.lef \
      def read picorv32a.floorplan.def &
```

<p align="center">
  <img width="900" height="450" src="../images/25.png">
</p>

---

### 📌 Step 5: Inspect Layout

**IO Pins (equidistant)**

<p align="center">
  <img width="900" height="400" src="../images/31.png">
</p>

**Use `what` in tkcon to identify components**

<p align="center">
  <img width="900" height="450" src="../images/26.png">
</p>

<p align="center">
  <img width="900" height="450" src="../images/27.png">
</p>

<p align="center">
  <img width="900" height="450" src="../images/28.png">
</p>

<p align="center">
  <img width="900" height="450" src="../images/29.png">
</p>

**Standard Cells in Bottom-Left**

<p align="center">
  <img width="900" height="450" src="../images/30.png">
</p>

---

### 📋 Step 6: Review Config Parameters

Check output config:

```bash
openlane/designs/picorv32a/runs/<date>/config.tcl
```

<p align="center">
  <img width="900" height="450" src="../images/32.png">
</p>

🧠 Notes:
- `FP_CORE_UTIL` was overridden from 65 → 50 by highest-priority config file.
- `FP_IO_VMETAL`, `FP_IO_HMETAL` were also overridden.

---

### 🧾 Step 7: View Floorplan Logs

Check logs:

```bash
openlane/designs/picorv32a/runs/<date>/logs/floorplan/
```

<p align="center">
  <img width="1000" height="200" src="../images/33.png">
</p>

<p align="center">
  <img width="1000" height="350" src="../images/34.png">
</p>

---

✅ **Completed Floorplan Successfully!**

---

