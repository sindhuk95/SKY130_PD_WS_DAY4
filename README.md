
## Timing Analysis and Clock Tree Synthesis (CTS)

During Placement, entire mag information is not necessary. Only the PR boundary, I/O, Power and ground rails of the cell is required. This information is is defined in LEF file.
The main objective is extract lef from the mag file and plug into our design flow.

Before moving any further lets you know about tracks. 
### Track 
 A line or path on which metal laters are drawn for routing. Track is used to define the height of the standard cell. 

As we are trying to implement our own stdcell, there are few guidelines to be followed
 -I/O ports must lie on the intersection on Horizontal and vertical tracks
 - Width and Height of standard cell 

This information is defined in ``tracks.info``. Lets see  info, lets say the I/O ports are in ``li1 layer`` and check whethere they are at intersection or not by using ``grid 0.46um 0.34um 0.23um 0.17um``

