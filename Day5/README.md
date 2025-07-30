# Sky130 Day 5 – Final steps for RTL2GDS using TritonRoute and OpenSTA

## Routing and Design rule check (DRC)

- `Maze Routing - Lee's Algorithm`

![Maze Routing Grid](screenshots/Maze_Routing_Grid.png)

![Lee’s Algorithm Visualization](screenshots/Lee’s_Algorithm_Visualization.png)

- DRC Check

![DRC Error Report](screenshots/DRC.png)

![Magic DRC Screen](screenshots/Magic.png)

- Parasitics Extraction

![Parasitic Extraction Output](screenshots/Parasitic.png)


# Power Distribution Network and Routing

To generate the power distribution network and route the design, follow these steps.

## Step 1: Generate Power Distribution Network (PDN)

Run the following commands up to CTS:

```bash
prep -design picorv32a -tag 20-03_18-30 -overwrite
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs

set ::env(SYNTH_STRATEGY) "DELAY 0"
set ::env(SYNTH_SIZING) 1

run_synthesis

init_floorplan
place_io
global_placement_or
detailed_placement
tap_decap_or
detailed_placement

run_cts
```

Now, run the command below to generate the PDN:

```bash
gen_pdn
```

![PDN Generation Output 1](screenshots/PDN_Generation1.png) 

![PDN Generation Output 2](screenshots/PDN_Generation2.png)

To view the PDN output in Magic, navigate to:

```bash
cd openlane_working_dir/openlane/designs/picorv32a/runs/20-03_18-30/tmp/floorplan/
```

Use this command to open the PDN DEF file:

```bash
magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech \
lef read ../../tmp/merged.lef \
def read 12-pdn.def &
```

![PDN in Magic 1](screenshots/PDN_in_Magic.png)
 
![PDN in Magic 2](screenshots/PDN_in_Magic1.png) 

![PDN in Magic 3](screenshots/PDN_in_Magic2.png)

![PDN in Magic 3](screenshots/PDN_in_Magic3.png)

![PDN in Magic 3](screenshots/PDN_in_Magic4.png)

## Step 2: Perform Routing

Run the following command:

```bash
run_routing
```

This will invoke global and detailed routing.

![Routing Output 1](screenshots/Routing.png)

Routing began with 26,890 violations in the 1st optimization and reached 0 violations in the 57th.

To view the final routed DEF file in Magic, navigate to:

```bash
cd Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/20-03_18-30/results/routing/
```

Use the following command:

```bash
magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech \
lef read ../../tmp/merged.lef \
def read picorv32a.def &
```

![Final Routing in Magic 1](screenshots/Magic_1.png) 
![Final Routing in Magic 2](screenshots/Magic_2.png) 
![Final Routing in Magic 3](screenshots/Magic_3.png)
![Final Routing in Magic 3](screenshots/Magic_4.png)
![Final Routing in Magic 3](screenshots/Magic_5.png)
![Final Routing in Magic 3](screenshots/Magic_6.png)

Final DEF snapshot:

![Final DEF Snapshot](screenshotss/picorv32a_def.png)


# TritonRoute Features

TritonRoute is the detailed router used in OpenLane. It performs:

- Global routing refinement
- Detailed routing
- DRC (Design Rule Check) fixing
- Via generation
- Wirelength optimization
- Pin access checking

Below are the various logs and visual outputs from TritonRoute:

![TritonRoute Log 1](screenshots/TritonRoute1.png) 
![TritonRoute Log 2](screenshots/TritonRoute2.png) 
![TritonRoute Log 3](screenshots/TritonRoute3.png) 
![TritonRoute Log 4](screenshots/TritonRoute4.png) 
![TritonRoute Log 5](screenshots/TritonRoute5.png) 
![TritonRoute Log 6](screenshots/TritonRoute6.png) 
![TritonRoute Log 7](screenshots/TritonRoute7.png) 
![TritonRoute Log 8](screenshots/TritonRoute8.png)


