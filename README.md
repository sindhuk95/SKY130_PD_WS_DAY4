

## Timing Analysis and Clock Tree Synthesis (CTS)

### Standard Cell LEF generation

During Placement, entire mag information is not necessary. Only the PR boundary, I/O ports, Power and ground rails of the cell is required. This information is is defined in LEF file.
The main objective is extract lef from the mag file and plug into our design flow.

Before moving any further lets know about tracks. 
### Track 
 A line or path on which metal layers are drawn for routing. Track is used to define the height of the standard cell. 

As we are trying to implement our own stdcell, there are few guidelines to be followed
 -I/O ports must lie on the intersection on Horizontal and vertical tracks
 - Width and Height of standard cell are odd mutliples of Horizontal track pitch and Vertical track pitch

This information is defined in ``tracks.info``. The syntax is `` metallayer direction offset spacing``
``li1 X 0.23 0.46
  li1 Y 0.17 0.34`` 
  
  ![4-1](https://github.com/sindhuk95/SKY130_PD_WS_DAY4/assets/135046169/0b1a5578-e3a4-41b4-b8c5-6ce8c4414228)

lets say the I/O ports are in ``li1 layer`` and check whethere they are at intersection or not by using ``grid 0.46um 0.34um 0.23um 0.17um`` on tkcon windowand and to activate the grid, press G on the magic console.
 
 Now, check whether the width and height of std cell met by measuring the tracks the cells has occupied.
 
 ![4-3 grid](https://github.com/sindhuk95/SKY130_PD_WS_DAY4/assets/135046169/886c310e-0701-4bf1-ba06-0d6cc4c0a3b6)

### create Port Definition: 

However, certain properties and definitions need to be set to the pins of the cell. For LEF files, a cell that contains ports is written as a macro cell, and the ports are the declared as PINs of the macro. Our objective is to extract LEF from a mag file. 

The way to define a port is through Magic console and following are the steps:
- In Magic Layout window, first source the .mag file for the design (here inverter). Then Edit >> Text which opens up a dialogue box.

![4-6 port declaration](https://github.com/sindhuk95/SKY130_PD_WS_DAY4/assets/135046169/df06c1ee-65a7-455f-9604-607b556d1ca4)

- When you double press S at the I/O lables, the text automatically takes the string name and size. Ensure the Port enable checkbox is checked and default checkbox is unchecked as shown in the figure:
-
![image](https://github.com/sindhuk95/SKY130_PD_WS_DAY4/assets/135046169/0b28cc75-8648-4211-a865-60128a0a47bf)

- In the above two figures, The number in the textarea near enable checkbox defines the order in which the ports will be written in LEF file (0 being the first).

-  For power and ground layers, the definition could be same or different than the signal layer. Here, ground and power connectivity are taken from metal1 (Check by using
what command on tkcon once you make box on VPWR and VGND on magic window)

![image](https://github.com/sindhuk95/SKY130_PD_WS_DAY4/assets/135046169/e14054c8-114e-46b2-a14d-7a6ed17763c5)

### Set port class and port use attributes for layout 
After defining ports, the next step is setting port class and port use attributes.

Select port A in magic:
```
port class input
port use signal
```
Select Y area
```
port class output
port class signal
```
Select VPWR area
```
port class inout
port use power
```
Select VGND area
```
port class inout
port use ground

```

# CUSTOM CELL NAMING

Name the custom cell through tkcon window as ``sky130_vsdinv.mag``.

![4-8 our cell](https://github.com/sindhuk95/SKY130_PD_WS_DAY4/assets/135046169/68fc0578-dd29-45c5-a6ee-d64cbc02a60b)

this gets created in ``openlane/vsdstdcelldesign`` directory.

![4-7 custon name for cell](https://github.com/sindhuk95/SKY130_PD_WS_DAY4/assets/135046169/7c2aebaf-5e2a-4e26-800f-b2e04d340e3f)


LEF extraction can be carried out in tkcon as follows:
```
lef write

```
![lef write](https://github.com/sindhuk95/SKY130_PD_WS_DAY4/assets/135046169/9cc3b2a1-c0c1-4190-a220-fcfd13ad6723)

This generates ``sky130_vsdinv.lef`` file.

![4-9 lef file created](https://github.com/sindhuk95/SKY130_PD_WS_DAY4/assets/135046169/fabbb30e-fb9f-4a9a-bb98-c7de461cac54)

### Integrating custom cell in OpenLANE

In order to include our custom standard cell to ur design and run the first step synthesis, copy the sky130_vsdinv.lef file to the designs/picorv32a/src directory.
Since abc maps the standard cell to a library, there must be a library that defines the CMOS inverter. The sky130_fd_sc_hd_typical.lib as well as fast and slow libs files from vsdstdcelldesign/libs directory needs to be copied to the designs/picorv32a/src directory. If you are interested to check our cell in lib and lef, go to lib and lef folders

![image](https://github.com/sindhuk95/SKY130_PD_WS_DAY4/assets/135046169/77f8b542-d996-4060-956f-c02283e2f5b0)


![4-10 lef and lib file to picrov32a](https://github.com/sindhuk95/SKY130_PD_WS_DAY4/assets/135046169/685c25b1-c3f1-43f2-b7e0-1985cd341531)


Next, modifiy the ``config.tcl`` 

```
set ::env(LIB_SYNTH) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130/sky130_fd_sc_hd__typical.lib"
set ::env(LIB_SLOWEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130/sky130_fd_sc_hd__slow.lib"
set ::env(LIB_FASTEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130/sky130_fd_sc_hd__fast.lib"
set ::env(LIB_TYPICAL) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130/sky130_fd_sc_hd__typical.lib"
set ::env(EXTRA_LEFS) [glob $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/src/*.lef]
```

![image](https://github.com/sindhuk95/SKY130_PD_WS_DAY4/assets/135046169/d6414049-bbc3-4c80-b516-7b616afa973d)


In order to integrate the standard cell in the OpenLANE flow, invoke openLANE as usual and carry out following steps:
``` 
prep -design picorv32a -tag 03-06-08-35 -overwrite 
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs
run_synthesis
```
After synthesis , my total neagtive and worst negative lack values are 

``
tns : -711.59
wns: -23.89
``
### Delay

Basically, Delay is a parameter that has huge impact on our cells in the design. Delay decides each and every other factor in timing. 
For a cell with different size, threshold voltages, delay model table is created where we can it as timing table.
``Delay of a cell depends on input transition and out load``. 
Lets say two scenarios, 
we have long wire and the cell(X1) is sitting at the end of the wire : the delay of this cell will be different because of the bad transition that caused due to the resistance and capcitances on the long wire.
we have the same cell sitting at the end of the short wire: the delay of this will be different since the tarn is not that bad comapred to the earlier scenario.
Eventhough both are same cells, depending upon the input tran, the delay got chaned. Same goes with o/p load also.

Just for better values for slack, I checked few synthesis environment variables like synthesis strategy, synthesis buffering and synthesis sizing , maximum fanout of cells and made changes if neccessary.

![image](https://github.com/sindhuk95/SKY130_PD_WS_DAY4/assets/135046169/0296f08a-fdab-4c22-9ec4-d4b04ee78d2c)

run synthesis again for better slack

Next floorplan is run, followed by placement:
```
run_floorplan
run_placement
```
To check the layout invoke magic from the ```results/placement``` directory:

```
magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.def &
```

Since the custom standard cell has been plugged into the design,in the openlane flow, it would be visible in the layout.

# paste the cell image once u do placement and expand it by S on ouyr cell and exapnd typing in tkco


### Post-synthesis timing analysis

Timing analysis is carried out outside the openLANE flow using OpenSTA tool. For this, a new file ```pre_sta.conf``` is created. This file would be reqiured to carry out the STA analysis. Invoke OpenSTA outside the openLANE flow as follows:
```
sta pre_sta.conf
```

Since clock is propagated only once we do CTS, In placement stage clock is considered to be ideal. So only setup slack is taken into consideration before CTS.

`` Setup time: minimum time required for the data to be stable before the active edge of the clock to get properly captured.
setup slack : data required time - data arrival time``

clock is generated from PLL which has inbuilt circuit which cells and some logic. There might variations in the clock generation depending upon the ckt. These variations are collectivity known as clock uncertainity. In that jitter is one of the parameter. 

``Clock Jitter : deviation of clock edge from its original location.``

From the timing report, we can improve slack by upsizing the cells i.e., by replacing the cells with high drive strength and we can see significant changes in the slack.

###  CLOCK TREE SYNTHESIS

  In this stage clock is propagated and make sure that clock reaches each and every clock pin from clock source with mininimum skew and insertion delay. Inorder to do this, we implement H-tree using mid point strategy. For balancing the skews, we use clock invteres or bufferes in the clock path. 
 Before attempting to run CTS in TritonCTS tool, if the slack was attempted to be reduced in previous run, the netlist may have gotten modified by cell replacement techniques. Therefore, the verilog file needs to be modified using the ```write_verilog``` command. Then, the synthesis, floorplan and placement is run again. To run CTS use the below command:
```
run_cts
```

Since, clock is propagated, from this stage, we do timing analysis with real clocks. From now post cts analysis is performed by operoad within the openlane flow

```
openroad
write_db pico_cts.db
read_db pico_cts.db
read_verilog /openLANE_flow/designs/picorv32a/runs/03-07_11-25/results/synthesis/picorv32a.synthesis_cts.v
read_liberty $::env(LIB_SYNTH_COMPLETE)
link_design picorv32a
read_sdc /openLANE_flow/designs/picorv32a/src/my_base.sdc
set_propagated_clock (all_clocks)
report_checks -path_delay min_max -format full_clock_expanded -digits 4
```

## Final steps in RTL2GDS
### Power Distribution Network generation
Unlike the general ASIC flow, Power Distribution Network generation is not a part of floorplan run in OpenLANE. PDN must be generated after CTS and post-CTS STA analyses:

```
gen_pdn
```
we can check whether PDN has been created or no by check the current def environment variable: ``` echo S::env(CURRENT_DEF)```.
  - Once the command is given, power distribution netwrok is generated.
  - The power distribution network has to take the `design_cts.def` as the input def file.
  - Power rings,strapes and rails are created by PDN.
  - From VDD and VSS pads, power is drawn to power rings.
  - Next, the horizontal and vertical strapes connected to rings draw the power from strapes. 
  - Stapes are connected to rings and these rings are connected to std cells. So, standard cells get power from rails.
  -  The standard cells are designed such that it's height is multiples of the vertical tracks /track pitch. Here, the pitch is `2.72`. Only if the above conditions are adhered it is possible to power the standard cells.
  -  There are definitions for the straps and the rails. In this design, straps are at metal layer 4 and 5 and the standard cell rails are at the metal layer 1. Vias connect accross the layers as required.

![image](https://github.com/sindhuk95/SKY130_PD_WS_DAY4/assets/135046169/5d1cf948-7ba7-4f17-a8e8-5ceebf0a13d1)

# Routing

The two types of engines that perform two types of routing are
 - Global routing - routing is divided into grids and 3D routing graph used during this by ``FASTE ROUTE``. 
 - Detailed routing - Finer grids and routing guides used to implement physical wiring ``tritonRoute`` 
- Fast Route generates the routing guides, whereas Triton Route uses the Global Route and then completes the routing with some strategies and optimisations for finding the best possible path connect the pins. 

Features of TritonRoute:
1. Performs Initial detail route
2. Honouring pre-processed route guides
   - Initial route guide
   - splitting
   - merging
   - bridging
3. Assumes route guide for each net satisfy inter guide connectivity
4. Works on MILP based panel routing scheme with Intra-layer parallel and inter-layer sequential routing framework

This routing stage must have the ``CURRENT_DEF`` set to ``pdn.def``

Start routing by using 

``run_routing``

- The options for routing can be set in the `config.tcl` file. 
- The optimisations in routing can also be done by specifying the routing strategy to use different version of `TritonRoute Engine`. There is a trade0ff between the optimised route and the runtime for routing.
- For the default setting picorv32a takes approximately 30 minutesaccording to the current version of TritonRoute.

Parasitic extraction is done separately using spef extractor oustide the openlane.
Invoke the engine using the command in SPEF_EXTRACTOR directory:
``python3 main.py /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/03-06_15-23/tmp/merged.lef /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/03-06_15-23/results/routing/picorv32a.def/``










