# Day 2 – Chip Floorplanning & Floorplan in OpenLANE

## Floorplanning Concepts

### 1. Utilization Factor & Aspect Ratio

```bash
Utilisation Factor =  Area occupied by netlist
                     --------------------------
                        Total area of core

Aspect Ratio = Height / Width
```

- Utilization Factor ≈ 0.5–0.6 in practice.  
- 100% utilization leaves no space for routing or buffers.  
- Aspect Ratio is used to control layout shape.

---

### 2. Fix Location of Pre-Placed Cells

- Pre-placed cells (like macros) are positioned manually before automated placement begins.
- Their location must be set in advance and will not be adjusted by tools.

---

### 3. Add Decoupling Capacitors

- Preplaced cells demand large current during switching.
- Due to resistance/inductance in wires, voltage can drop.
- Decaps help supply fast-switching current locally to maintain voltage levels within the noise margin.

---

### 4. Power Rails (Mesh)

- Decaps can’t be placed everywhere.
- A power mesh ensures:
  - Nearby VDD/VSS availability
  - Reduced voltage dips and ground bounces during switching

---

### 5. Placement of Pins

- I/O Pins are placed in the area between core and die boundary.
- Their position is chosen based on connectivity with cells.

---

### 6. Block Placement in Pin Region

- Block placement in areas where I/O pins are located.
- Ensures no cells are placed near the boundary to avoid congestion.

📷 Screenshot:  
![logical-blockage](screenshots/20.png)

📷 Screenshot:  
![pin-boundaries](screenshots/19.png)

---

## LAB – Floorplanning with OpenLANE

### Step 1: Understand Configuration Hierarchy

- Floorplan settings are defined in OpenLANE config files.
- Priority order (low to high):
  1. `floorplan.tcl`
  2. `config.tcl`
  3. `sky130A_sky130_fd_sc_hd_config.tcl`

📷 Screenshot:  
![config-folder](screenshots/21.png)

📷 Screenshot:  
![config-order-1](screenshots/24.png)

📷 Screenshot:  
![config-order-2](screenshots/23.png)

📷 Screenshot:  
![floorplan-tcl](screenshots/22.png)

---

### Step 2: Run Floorplan

```bash
run_floorplan
```

- Output DEF file generated at:
  ```
  openlane/designs/picorv32a/runs/<date>/results/floorplan/picorv32a.floorplan.def
  ```

📷 Screenshot:  
![def-terminal](screenshots/diearea.png)

### Calculate Die Area

```bash
DIEAREA (0 0) (554570 565290)
Unit = 1000 microns

Area = (554570 / 1000) * (565290 / 1000)
     = 311829.1653 microns²
```

---

### Step 3: View DEF in Magic

```bash
magic -T /path/to/sky130A.tech \
  lef read ../../tmp/merged.lef \
  def read picorv32a.floorplan.def &
```

📷 Screenshot:  
![layout-in-magic](screenshots/25.png)

---

### Step 4: Check I/O Pins

- Input pins are placed equally along the boundary.
- Controlled by `FP_IO_MODE = 1`.

📷 Screenshot:  
![io-pins](screenshots/31.png)

---

### Step 5: Inspect Components in Magic

Use tkcon window:
1. Select a component
2. Type:

```bash
what
```

📷 Screenshot:  
![component-1](screenshots/26.png)  
📷 Screenshot:  
![component-2](screenshots/27.png)  
📷 Screenshot:  
![component-3](screenshots/28.png)  
📷 Screenshot:  
![component-4](screenshots/29.png)

---

### Step 6: Check Standard Cells

- Standard cells will appear at the bottom-left area of the layout.

📷 Screenshot:  
![std-cells](screenshots/30.png)

---

## Config Parameters Used

Inside:

```bash
openlane/designs/picorv32a/runs/<date>/
```

Important files/folders:
- `config.tcl`
- `results/`
- `reports/`
- `logs/`
- `cmds.log`
- `tmp/`

📷 Screenshot:  
![output-config](screenshots/32.png)

- `FP_IO_HMETAL` and `FP_IO_VMETAL` values were overridden from defaults.
- `FP_CORE_UTIL` set to 65 in one file, but overwritten to 50 due to higher-priority config.

---

## Debug Floorplan Logs

Logs are available here:

```bash
openlane/designs/picorv32a/runs/<date>/logs/floorplan/
```

📷 Screenshot:  
![log-dir](screenshots/33.png)

📷 Screenshot:  
![ioplacer-log](screenshots/34.png)

---

## References

- [DEF Format Overview](https://ivlsi.com/design-exchange-format-def-in-vlsi-physical-design/)
- [DEF 5.8 Language Reference](https://coriolis.lip6.fr/doc/lefdef/lefdefref/LEFSyntax.html)

