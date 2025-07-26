# Day 2: Good Floorplan vs Bad Floorplan and Introduction to Library Cells

## Chip Floor Planning Considerations

### 1. Utilization Factor and Aspect Ratio

```sh
Utilisation Factor =  Area occupied by netlist
                     --------------------------
                        Total area of core

Aspect Ratio = Height / Width
```

- If Utilization = 1, there's no space left for routing or buffers. Typically, we keep it around 0.5–0.6.
- Lower utilization allows space for routing wires and filler cells.

📷 Screenshot:  
![utilization-ratio](screenshots/utilization-ratio.png)

---

### 2. Concept of Pre-placed Cells

- Pre-placed cells (e.g., macros) must be fixed before automated placement.
- These cannot be moved by place and route tools.

📷 Screenshot:  
![preplaced-cells](screenshots/preplaced-cells.png)

---

### 3. Decoupling Capacitors

- High switching blocks may suffer from voltage drop due to wire resistance and inductance.
- Decaps near those blocks ensure proper voltage levels by quickly supplying charge.

📷 Screenshot:  
![decaps-example](screenshots/decaps-example.png)

---

### 4. Power Planning

- Large simultaneous switching causes IR drop and ground bounce.
- Power mesh ensures stable power distribution via multiple VDD/VSS paths.

📷 Screenshot:  
![power-mesh](screenshots/power-mesh.png)

---

### 5. Pin Placement and Logical Cell Placement Blockage

- Pins are placed in the IO ring between the core and die.
- Pin locations depend on cell placement.
- Block cell placement near IO pins to prevent congestion.

📷 Screenshot:  
![pin-placement](screenshots/pin-placement.png)

📷 Screenshot:  
![placement-blockage](screenshots/placement-blockage.png)

---

## LAB: Floorplan using OpenLANE

### 6. Set Floorplan Configuration

- Config variables can be defined in:
  - `floorplan.tcl`
  - `config.tcl`
  - `sky130A_sky130_fd_sc_hd_config.tcl` (highest priority)

📷 Screenshot:  
![floorplan-config-sources](screenshots/floorplan-config-sources.png)

---

### 7. Run Floorplan

```tcl
% run_floorplan
```

- Output location:  
  `openlane/designs/picorv32a/runs/<date>/results/floorplan/picorv32a.floorplan.def`

📷 Screenshot:  
![floorplan-run-output](screenshots/floorplan-run-output.png)

---

### 8. Check Die Area from DEF File

```sh
DIEAREA (0 0) (554570 565290)
Unit: 1000 microns

Die Area = (554570/1000) * (565290/1000)
         = 311829.1653 µm²
```

📷 Screenshot:  
![die-vs-core](screenshots/die-vs-core.png)

---

### 9. Open Floorplan in Magic

```sh
magic -T sky130.tech \
      lef read ../../tmp/merged.lef \
      def read picorv32a.floorplan.def &
```

📷 Screenshot:  
![magic-floorplan](screenshots/magic-floorplan.png)

---

### 10. Analyze Magic Layout

- Use the `what` command in Magic’s tkcon to identify cells.
- Equidistant input pins visible if `FP_IO_MODE = 1`.
- Standard cells located at bottom left.

📷 Screenshot:  
![magic-what](screenshots/magic-what.png)

📷 Screenshot:  
![magic-cells](screenshots/magic-cells.png)

---

### 11. Floorplan Config Review

- Configs saved to:  
  `openlane/designs/picorv32a/runs/<date>/config.tcl`
- Overrides shown clearly, e.g.,:
  - `FP_IO_VMETAL`, `FP_CORE_UTIL`, etc.

📷 Screenshot:  
![floorplan-config-review](screenshots/floorplan-config-review.png)

---

### 12. Floorplan Logs

- Logs available at:  
  `openlane/designs/picorv32a/runs/<date>/logs/floorplan/`

📷 Screenshot:  
![floorplan-logs](screenshots/floorplan-logs.png)

---

### 13. ioPlacer.log

- Contains pin placement logs and IO warnings.

📷 Screenshot:  
![ioplacer-log](screenshots/ioplacer-log.png)

# Library Binding and Placement

### 📌 Placement Stage

Placement is performed in two stages:

- **Global Placement** – Initial placement without legalization. It uses **HPWL (Half Perimeter Wire Length)** as the optimization metric.
- **Detailed Placement** – Legalizes the placement by aligning cells to legal rows and eliminating overlaps.

![Placement stages](../images/35.png)

---

###  LAB: Running Placement in OpenLANE

To run placement, use the command:

```bash
run_placement
```

This command executes:

- **Global placement** using the `RePlAce` engine
- **Detailed placement** using the `OpenDP` tool (Open-Source Detailed Placer)

✅ **Objective**: Minimize the **overflow value**  
A successful placement run shows overflow values progressively decreasing with each iteration, indicating that the design is converging.

![Placement overflow log](../images/36.png)

---

###  Placement Output Directory

After placement is completed, the output files will be generated in:

```
openlane/designs/picorv32a/runs/<date>/results/placement/
```

Important file:
- `picorv32a.placement.def` – contains final standard cell positions in DEF format.

![Placement DEF file](../images/37.png)

---

### Visualizing the Placement in Magic

To visualize the placed layout in Magic, use this command:

```bash
magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech \
lef read ../../tmp/merged.lef \
def read picorv32a.placement.def &
```

---

###  Final Layout View

![Zoomed placement layout](../images/38.png)


