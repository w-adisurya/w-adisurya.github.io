---
layout: post
title: "SAN Switch ?"
date: 2020-05-13 21:35:00 +0700
---

Need to note down this thing cause seems like you'll meet this creature as long as you're in server storage world.  
Todays menu : How to add zoning to SAN Switch for mapping new volume from storage to server ..  World Wide Name zoning

## TL;DR  
This SAN Switch seems like Brocade 300 Switch, read the reference doc at [Brocade 300 Switch](https://www.dataswitchworks.com/datasheets/switches/300-ds-03.pdf){:target="_blank"}   
Great Introduction into [Zoning Topology](http://www.sanadmin.net/2015/11/san-switches.html){:target="_blank"}  

<img src="\images\SAN\SANTopology.png" alt="1vyrain" title="SANTopology" /> 
<head>						<!--- Memberikan border line pada table --->
<style>
table 

td, th {
  border: 1px solid #dddddd;
  text-align: left;
  padding: 8px;
}

tr:nth-child(even) {
  background-color: #dddddd;
}
</style>
</head>
<br/>							<!--- Memberikan blank line --->
**Notes**:
+	<em>Plug in serial cable to switch, UTP side to switch and USB side to your laptop</em>
+	<em>Open putty, connection type is serial, speed is 96xx and serial line using whatever COM port that appear in your device manager, example: COM1 </em>
+	<em>For safety purpose, backup zoning config using putty session logging, save it as .txt</em>
+	<em>No_Light means no FC cables connected to that port, Online means FC cables connected and WWN is listed ready to configure, No_Module simply no FC module is plugged</em>
+	<em>Zoning is one part of the main goal, after zoning, make new volume to attach to server and make server list that connected to SAN Storage, this part is usually done in Storage Manager</em>

# Check Existing Switch Config
	SW300:admin> switchshow
	switchName:	SW300
	switchType:	71.2
	switchState:	Online   
	switchMode:	Native
	switchRole:	Principal
	switchDomain:	1
	switchId:	fffc01
	switchWwn:	10:xx:c4:f5:7c:d5:b7:b4
	zoning:		ON (ServerDC)
	switchBeacon:	OFF
	HIF Mode:	OFF

	Index Port Address Media Speed       State   Proto
	==================================================
	   0   0   01xxxx   id    N8	   No_Light    FC  
	   1   1   0101xx   id    N8	   Online      FC  F-Port  21:xx:xx:24:ff:19:ee:xx 
	   2   2   0102xx   id    N8	   Online      FC  F-Port  21:xx:xx:24:ff:1e:40:68 
	   3   3   0103xx   id    N8	   Online      FC  F-Port  1 N Port + 1 NPIV public 
	   4   4   0104xx   id    N8	   Online      FC  F-Port  1 N Port + 1 NPIV public 
	   5   5   0105xx   id    N8	   Online      FC  F-Port  21:xx:f4:ee:d4:50:yy:ae 
	   6   6   0106xx   id    N8	   Online      FC  F-Port  21:xx:xx:24:ff:1e:02:c4 
	   7   7   0107xx   id    N8	   Online      FC  F-Port  21:xx:f4:ee:d4:50:yy:8e 
	   8   8   0108xx   id    N8	   Online      FC  F-Port  21:xx:xx:24:ff:1e:03:0c 
	   9   9   0109xx   id    N8	   Online      FC  F-Port  21:xx:f4:ee:d4:50:yy:ca 
	  10  10   010axx   id    N8	   Online      FC  F-Port  21:xx:f4:ee:d4:50:yy:c8 
	  11  11   010bxx   id    N8	   Online      FC  F-Port  21:xx:f4:ee:d4:50:87:fc 
	  12  12   010cxx   id    N8	   Online      FC  F-Port  21:xx:f4:ee:d4:50:88:04 
	  13  13   010dxx   id    N8	   Online      FC  F-Port  21:xx:34:80:0d:6c:ad:b6 
	  14  14   010exx   id    N8	   Online      FC  F-Port  21:xx:34:80:0d:6c:ad:c0 
	  15  15   010fxx   id    N8	   No_Light    FC  
	  16  16   011xx0   --    N8	   No_Module   FC  (No POD License) Disabled
	  17  17   0111xx   --    N8	   No_Module   FC  (No POD License) Disabled
	  18  18   0112xx   --    N8	   No_Module   FC  (No POD License) Disabled
	  19  19   0113xx   --    N8	   No_Module   FC  (No POD License) Disabled
	  20  20   0114xx   --    N8	   No_Module   FC  (No POD License) Disabled
	  21  21   0115xx   --    N8	   No_Module   FC  (No POD License) Disabled
	  22  22   0116xx   --    N8	   No_Module   FC  (No POD License) Disabled
	  23  23   0117xx   --    N8	   No_Module   FC  (No POD License) Disabled
	SW300:admin> 

# Check Existing Zoning Config
	SW300:admin>  
	SW300:admin> alishow  
	Defined configuration:  
	 cfg:	ServerDC	  
		Zone_P; Zone_V; Zone_LabA; Zone_LabB;  
	 zone:	Zone_P	P_CTRL1_PORT1; P_CTRL2_PORT1  
	 zone:	Zone_V	V_CTRL1_PORT1; V_CTRL2_PORT2  
	 zone:	Zone_LabA	  
		LabA_P1; V_CTRL1_PORT1; V_CTRL2_PORT2  
	 zone:	Zone_LabB  
		LabB_P1; V_CTRL1_PORT1; V_CTRL2_PORT2  
	 alias:	P_CTRL1_PORT1	  
			50:xx:d3:10:05:5b:9c:08  
	 alias:	P_CTRL2_PORT1	  
			50:xx:d3:10:05:5b:9c:21  
	 alias:	V_CTRL1_PORT1	  
			50:xx:d3:10:05:5b:9c:3a  
	 alias:	V_CTRL2_PORT2	  
			50:xx:d3:10:05:5b:9c:3b  
	 alias:	LabA_P1	  
			21:xx:f4:ee:d4:50:yy:ae  
	 alias:	LabA_P2	  
			21:xx:f4:ee:d4:50:yy:af  
	 alias:	LabB_P1	  
			21:xx:f4:ee:d4:50:yy:8e  
	 alias:	LabB_P2	  
			21:xx:f4:ee:d4:50:yy:8f  

	Effective configuration:
	 cfg:	ServerDC	
	 zone:	Zone_P	50:xx:d3:10:05:5b:9c:08
			50:xx:d3:10:05:5b:9c:21
	 zone:	Zone_V	50:xx:d3:10:05:5b:9c:3a
			50:xx:d3:10:05:5b:9c:3b
	 zone:	Zone_LabA	
			21:xx:f4:ee:d4:50:yy:ae
			50:xx:d3:10:05:5b:9c:3a
			50:xx:d3:10:05:5b:9c:3b
	 zone:	Zone_LabB	
			21:xx:f4:ee:d4:50:yy:8e
			50:xx:d3:10:05:5b:9c:3a
			50:xx:d3:10:05:5b:9c:3b
	SW300:admin> 

# Delete Zoning Config
	SW300:admin> 
	SW300:admin> cfgremove "ServerDC", "Zone_Temp_Zoning"
	SW300:admin> zonedelete "Zone_Temp_Zoning"
	SW300:admin> alidelete "Temp_Zoning"
	SW300:admin> cfgsave
	WARNING!!!
	The changes you are attempting to save will render the
	Effective configuration and the Defined configuration
	inconsistent. The inconsistency will result in different
	Effective Zoning configurations for switches in the fabric if
	a zone merge or HA failover happens. To avoid inconsistency
	it is recommended to commit the configurations using the
	'cfgenable' command.
	Do you want to proceed with saving the Defined
	zoning configuration only?  (yes, y, no, n): [no] y
	sw0 Updating flash ...
	2020/05/13-01:15:39, [ZONE-1024], 150, FID 128, INFO, SW300, cfgSave completes successfully.
	.
	Updating flash ...
	2020/05/13-01:15:39, [ZONE-1062], 151, FID 128, WARNING, SW300, Defined and Effective zone configurations are inconsistent.
	SW300:admin> cfgenable ServerDC
	You are about to enable a new zoning configuration.
	This action will replace the old zoning configuration with the
	current configuration selected. If the update includes changes 
	to one or more traffic isolation zones, the update may result in  
	localized disruption to traffic on ports associated with
	the traffic isolation zone changes.
	Do you want to enable 'ServerDC' configuration  (yes, y, no, n): [no] y
	sw0 Updating flash ...
	2020/05/13-01:15:51, [ZONE-1022], 152, FID 128, INFO, SW300, The effective configuration has changed to ServerDC.   
	zone config "ServerDC" is in effect
	Updating flash ...
	SW300:admin>

# Add Zoning Config
	SW300:admin> 
	SW300:admin> alicreate "Something1", "21:xx:34:80:0d:6c:ad:b6"
	SW300:admin> alicreate "Something2", "21:xx:34:80:0d:6c:ad:c0"
	SW300:admin> zonecreate "Zone_Something1", "Something1; V_CTRL1_PORT1; V_CTRL2_PORT2"
	SW300:admin> zonecreate "Zone_Something2", "Something2; V_CTRL1_PORT1; V_CTRL2_PORT2"
	SW300:admin> cfgadd "ServerDC", "Zone_Something1; Zone_Something2"
	SW300:admin> cfgenable ServerDC
	You are about to enable a new zoning configuration.
	This action will replace the old zoning configuration with the
	current configuration selected. If the update includes changes 
	to one or more traffic isolation zones, the update may result in  
	localized disruption to traffic on ports associated with
	the traffic isolation zone changes.
	Do you want to enable 'ServerDC' configuration  (yes, y, no, n): [no] y
	sw0 Updating flash ...
	2020/05/13-01:22:06, [ZONE-1022], 153, FID 128, INFO, SW300, The effective configuration has changed to ServerDC.   
	zone config "ServerDC" is in effect
	Updating flash ...
	SW300:admin> cfgsave
	You are about to save the Defined zoning configuration. This
	action will only save the changes on Defined configuration.
	If the update includes changes to one or more traffic isolation
	zones, you must issue the 'cfgenable' command for the changes
	to take effect.
	Do you want to save the Defined zoning configuration only?  (yes, y, no, n): [no] y
	Nothing changed: nothing to save, returning ...
	SW300:admin> 