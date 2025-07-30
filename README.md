# Digital VLSI SoC Design and Planning Workshop – 2 Weeks

Welcome to the fascinating universe of Physical Design (PnR – Place and Route), where the heart of every integrated circuit (IC) design beats. In this 2-week hands-on workshop, we journey through the complete RTL to GDSII flow using **OpenLANE** and **Sky130 PDK**, culminating in tape-out-ready layouts and deep insights into timing, layout, and routing.

This repository documents the entire workshop with step-by-step lab procedures, annotated screenshots, simulation results, and clear educational explanations—ideal for students, researchers, and enthusiasts aiming to build their own SoC.

---

## Workshop Overview

**Duration:** 2 Weeks  
**Focus:** Full RTL to GDSII flow using open-source tools  
**Tools Used:** OpenLANE, Magic, ngspice, OpenSTA, TritonCTS, TritonRoute  
**Goal:** To design, simulate, floorplan, characterize, and route a RISC-V-based SoC from scratch.

---

## Day-wise Breakdown

### Day 1 – Inception of Open-Source EDA and OpenLANE Introduction
- Overview of QFN-48 Package, chip architecture, and IPs
- RISC-V introduction and SoC design fundamentals
- OpenLANE ASIC design flow overview
- OpenLANE directory structure walkthrough
- Design preparation and synthesis flow
- GitHub structure and synthesis result analysis

**Folder:** `Day1_Inception_OpenLANE`

---

### Day 2 – Floorplanning and Library Cells

#### Chip Floor Planning
- Utilization factor, aspect ratio, and floorplan constraints
- Pre-placed cells, decoupling capacitors, and power planning
- Pin placement and logical blockage concepts
- Floorplan generation and Magic layout viewing

#### Library Binding and Placement
- Netlist binding and placement estimation
- Placement optimization with congestion awareness (RePlAce)
- Understanding the role of libraries and characterization

#### Cell Design and Characterization
- Designing a standard cell (inverter) from circuit to layout
- Timing metrics: thresholds, propagation delay, and transition time

**Folder:** `Day2_Floorplanning_Placement`

---

### Day 3 – Custom Cell Design with Magic and ngspice

- SPICE deck generation and inverter simulation with ngspice
- Static vs dynamic inverter behavior analysis
- Working with `vsdstdcelldesign` repository
- CMOS fabrication steps visualized in layout
- Inverter layout, DRC, and LEF extraction
- Debugging Sky130 tech DRC rules in Magic

**Folder:** `Day3_Cell_Design_Characterization`

---

### Day 4 – Pre-layout Timing and Clock Tree Optimization

#### Timing Modelling and Setup
- Creating delay tables for characterization
- Synthesis with the custom standard cell

#### Ideal Clock Timing Analysis
- Setup time and clock uncertainty analysis with OpenSTA
- Slack optimization and ECO-based timing fixes

#### Clock Tree Synthesis
- Routing clock trees using TritonCTS (H-tree method)
- Signal integrity considerations: crosstalk and shielding

#### Real Clock Timing Analysis
- Setup and hold analysis with post-CTS real clock trees
- Using accurate timing libraries for realistic results

**Folder:** `Day4_Timing_ClockTree`

---

### Day 5 – Final RTL2GDS Flow, Routing, and Signoff

#### Routing and DRC
- Maze routing using Lee’s algorithm
- DRC validation and fixing routing violations

#### Power Distribution and TritonRoute Features
- Creating power straps and connecting standard cells
- Guide-based routing and inter-layer connectivity
- Final GDSII generation and post-route file review

**Folder:** `Day5_RTL2GDS_Routing_DRC`

---

## Tools and Technology Stack

- **Sky130 PDK** – 130nm open-source process by Google & SkyWater
- **OpenLANE** – Automated RTL to GDSII physical design flow
- **Magic VLSI** – Layout editing and design rule checking
- **ngspice** – SPICE-level transistor simulation
- **OpenSTA** – Static timing analysis tool
- **TritonCTS** – Clock tree synthesis tool
- **TritonRoute** – Detailed routing engine with DRC support

---

## Documentation Format

Each day's folder includes:
- Step-by-step lab procedures
- High-resolution annotated screenshots
- Key observations and output summaries
- Shell commands and tool outputs
- Organized folder structure with reports

---

## Acknowledgements

This workshop is based on open-source platforms and learning initiatives by:
- [SkyWater & Google](https://skywater-pdk.readthedocs.io/) – Sky130 Process Design Kit
- [OpenLANE by efabless](https://www.efabless.com/openlane) – RTL2GDSII flow
- [VSDOpen](https://www.vlsisystemdesign.com/) – Open EDA learning and simulation labs

---

## Get Started

Browse each day’s folder to begin your RTL2GDS journey.

Start from Day 1 and build your own standard cell or SoC—fully open-source.

