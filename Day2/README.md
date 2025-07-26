# 🗓️ Sky130 Day 2 – Floorplanning in OpenLANE (Core Area, DIE Area, IO Placement)

---

## 🎯 Objective

- Understand DIE area vs Core area
- Modify floorplan parameters in `floorplan.tcl`
- Re-run OpenLANE to observe changes in floorplan size
- Visualize IO placement, Core utilization, and Cell density
- Understand the role of PDN (Power Delivery Network)

---

## 📘 Theory Section

### 🔸 0. DIE Area vs Core Area

- **DIE Area** = Total area of the silicon chip
- **Core Area** = Portion of DIE used to place standard cells
- **Margins** = Distance from DIE edge to core edge (left, right, top, bottom)

📷 Screenshot:  
![die-vs-core](screenshots/die-vs-core.png)

---

### 🔸 1. IO Placement

- IO pads are placed around the core area
- Defined via configuration files in OpenLANE
- Even distribution helps signal integrity

📷 Screenshot:  
![io-placement](screenshots/io-placement.png)

---

### 🔸 2. PDN Grid and Power Planning

- PDN delivers power (VDD, VSS) to all parts of chip
- Implemented in **metal layers**
- Configured using `pdn.tcl` script in OpenLANE

📷 Screenshot:  
![pdn-grid](screenshots/pdn-grid.png)

---

## 🧪 Practical Section – Floorplanning with OpenLANE

### 🔸 3. Modify Floorplan Parameters

Inside OpenLANE interactive shell:

```tcl
set ::env(FP_CORE_UTIL) 0.3
set ::env(FP_ASPECT_RATIO) 1.0
set ::env(FP_CORE_MARGIN) 2

