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
- See [Telega docs](https://telega.zubax.com/commands/drive.html) on controlling Myxa ESCs with Cyphal
	- To control velocity of each motor, do:
	```bash
y pub -T 0.1 113:zubax.primitive.real16.Vector4 "[10, 10, 10, 30]"
	```
	- Index of each motor should be set up beforehand using the register `mns.setpoint_index` register. Easiest via Yukon
	- Note that ESCs currently are set to use ramp spinup
		- Below 5 rad/s, it considers it as stalled, set by this parameter: `drive.runner.0_ramp.velocity_stall_spinup: [5.0, 20.0]`
		- In practise, still might stall at 10 rad/s due to slight overshoot during deceleration that causes the velocity to momentarily drop below 5 rad/s

## Motor Control
### Run Strategies
- Use ramp spin-up
- Passive spin-up not relevant to Viper
#### Ramp Spinup
Can change the default of:
```YAML
drive.runner.0_ramp.spinup_current_pu: [0.5, 0.5]
drive.runner.0_ramp.spinup_duration: [0.20000000298023224, 0.4000000059604645, 1.5, 0.0020000000949949026, 0.014999999664723873, 0.6000000238418579]
```

to this:
```yaml
drive.runner.0_ramp.spinup_current_pu: [0.3, 0.3]
drive.runner.0_ramp.spinup_duration:   [0.1, 0.0, 0.4, 0.002, 0.015, 0.6]
```

Apparently more suitable for direct-drive propellers for faster spin-up, though I haven't found a considerable difference.
### Control Modes
- Velocity control mode: Uses INDI controller to control velocity in radians/second


> [!Tip]
> Make sure the power supply's current is not capped otherwise you may see the ESCs constantly restarting


> [!Config Dump]
> Dump config of motors into a `yaml` file which can later be imported:
> ```bash
> y rl 125 | y rb 123-126 > myxa_config.yaml
> ```