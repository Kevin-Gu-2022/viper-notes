# Overview
- [AD0505A](https://shop.zubax.com/collections/electric-drives/products/zubax-myxa) are high-end Permanent Magnet Synchronous Motor (PMSM) Field-Oriented Control (FOC) motor controllers (FOC (Electronic Speed Controller) ESC) for light unmanned aircraft and watercraft.
- Provides up to 1200 W of continuous power output and supports a wide range of operating voltages from 13 to 51 V (Li-ion 4–12 S).
- Runs its own control firmware called [Telega](https://telega.zubax.com/) with tutorial [here](https://telega.zubax.com/tutorials/index.html)
- Hardware [datasheet](https://files.zubax.com/products/com.zubax.telega/Zubax_Myxa_Datasheet.pdf)

# CAN

> [!important]
> The CAN-like connector on the very left is not CAN! It is just a AUX I/O interface

- Two CAN buses, but only CAN1 is actually in use. CAN2 connectors (the ones on bottom of PCB) are not soldered on
- The 2 CAN buses of CAN1 are connected in parallel 

## Cyphal
- Save and restart same
- Run Motor ID command in Yukon to obtain motor parameters (will undergo a series of tests on motor automatically). Will be in maintenance mode during this time
	- Relevant registers:
		- `motor.mechanical_ratio`
		- `motor.current_max`
		- `motor.resistance`
		- `motor.inductance_dq`
		- `motor.flux_linkage`

## Motor Control
### Run Strategies
- Use ramp spin-up
- Passive spin-up not relevant to Viper

### Control Modes
- Velocity control mode: Uses INDI controller to control velocity in radians/second