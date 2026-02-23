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
- The corresponding connector, I think is [this](https://au.rs-online.com/web/p/wire-housings-plugs/7521734), but this Aliexpress one will also work
	- [5/10PCS GH 1.25 2p 3p 4p 5p 6p 7P 8P Female Connector With Wire 15cm Cable GH1.25 Single Double Connector 28AWG](https://www.aliexpress.com/item/1005009390067334.html)
- Would be ideal if wires are pre-crimped
- 26 AWG for 1A
- Actually, I don't think it is either of those :(

- On other side, add a TVS diode connector (1.5KE 75A). Extra margin to account for cheap component: https://www.aliexpress.com/item/32879274481.html
- Pair with 1000µF, 50V electrolytic capacitor to perform smoothing