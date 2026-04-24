- The target is the amount of power needed to be delivered to the motors. A thrust target and torque target are combined in different ways to each motor to control overall vehicle position
- The torque here is the overall torque needed in the whole body to reach set point defined by user input (i.e. controller)
- This mixer performs a linear superposition: 
	- Motor output = thrust + pitch contribution + roll contribution + yaw contribution
```cpp
motors[MOTOR_FRONT_LEFT]  = thrustTarget + torqueTarget.y + torqueTarget.x - torqueTarget.z;
motors[MOTOR_FRONT_RIGHT] = thrustTarget + torqueTarget.y - torqueTarget.x + torqueTarget.z;
motors[MOTOR_REAR_LEFT]   = thrustTarget - torqueTarget.y + torqueTarget.x + torqueTarget.z;
motors[MOTOR_REAR_RIGHT]  = thrustTarget - torqueTarget.y - torqueTarget.x - torqueTarget.z;
```

>[!Note]
>x and y for torque are switched around

### Thrust

```
thrustTarget
```

Added to **every motor**, so increasing thrust raises the drone without rotating it.

---
### Pitch (`torqueTarget.y`)

```
+ torqueTarget.y  // front motors
- torqueTarget.y  // rear motors
```

- Positive pitch torque:
    - Front motors increase
    - Rear motors decrease
- Result: drone pitches **forward**
---
### Roll (`torqueTarget.x`)

```
+ torqueTarget.x  // left motors
- torqueTarget.x  // right motors
```

- Positive roll torque:
    - Left motors increase
    - Right motors decrease
- Result: drone rolls **right**

---

### Yaw (`torqueTarget.z`)

![[Pasted image 20260122214247.png|300]]

```
+ torqueTarget.z  // motors spinning CCW
- torqueTarget.z  // motors spinning CW
```
Yaw is controlled by **motor reaction torque**, not tilt:
- Increasing speed on one diagonal
- Decreasing on the other
- Net vertical thrust stays roughly constant
- Drone rotates about **Z axis**