# Connector From DC-DC to Pika Spark
- Pika Spark has the SM02B-GHS-TB connector onboard

|**Code**|**Meaning**|**Description**|
|---|---|---|
|**SM**|**Mounting Type**|Stands for **Surface Mount**. These are designed to be soldered to pads on the top of a PCB, rather than through holes.|
|**02**|**Circuit Count**|Indicates this is a **2-pin** (2-circuit) header.|
|**B**|**Entry Direction**|Stands for **Side Entry** (Right Angle). If this were an "S", it would be a Top Entry (Straight up) header.|
|**GH**|**Series Name**|Refers to the **JST GH Family** (1.25mm pitch).|
|**S**|**Internal Code**|Refers to the **Shape/Form** of the shroud. "S" is the standard shrouded version.|
|**TB**|**Packaging**|Stands for **Tape & Reel**. This tells the factory the parts come on a large plastic wheel for automated assembly machines.|
- The corresponding connector is [GHR-02V-S](https://au.rs-online.com/web/p/wire-housings-plugs/7521734), but this Aliexpress one will also work
	- GH1.25 Female: 5 Pcs, Connector Type: Single head, Pins: 2P, 28 AWG, pre-crimped wires on header side
	  https://www.aliexpress.com/item/1005009390067334.html
- 26 AWG for 1A
- Max power of Arduino Portenta X8 is [4000 mA](https://core-electronics.com.au/attachments/localcontent/ABX00049-datasheet_4359766d976.pdf), and this [blog](https://oscarliang.com/wires-connectors/#26AWG-28AWG) suggests 26AWG should be sufficient for just powering electronics 

