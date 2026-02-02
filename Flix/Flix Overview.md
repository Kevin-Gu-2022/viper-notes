[Flix](https://github.com/okalachev/flix) is an ESP32-based quadcopter build from scratch

More info in [blog](https://habr.com/ru/articles/814127/)
## PID Control
- Example [implementation](https://github.com/okalachev/flix/blob/master/flix/control.ino)
## Euler Angles
- Roll ($x$), Pitch ($y$), Yaw ($z$) are rotations about their respective axes
- Order of rotation MATTERS!!
- In robotics, convention is **ZYX** (Roll -> Pitch -> Yaw) 
- Robotics uses **extrinsic**, meaning rotations about a fixed axis, as opposed to intrinsic, which are rotations that move with the object
- Positive and negatives on axes follow right hand rule (direction in which hands curl around axis is the positive)
- Nose up: positive pitch, right side down: positive roll

- MAVLINK uses FRD (front-right-down) whereas Flix uses FLU (front-left-up)
- Flix handles this when sending and receiving messages from Mavlink protocol
- In FLU, a positive pitch means it's going down while a negative pitch (apply right hand rule to y-axis to see this)

### Gimbal Lock
- Gimbal lock is when a degree of freedom is lost due to the alignment of two of the axes. Happens when the middle axis aligns with either of the other two. For example, consider a 3D camera that has pitch at 90$\degree$: the pan and tilt will both produce the same effect. Quaternions eliminate this issue with the addition of a 4th axis

## Frames of Reference
- **Inertial frame**: Earth-fixed coordinate system with origin at defined home location
	- **Inertial**: No acceleration (either stationary or constant velocity)
- **Vehicle Frame:** A translation to the frame of the vehicle. Direction of axes same as the inertial frame
- **Vehicle-1 Frame**: Origin at centre of mass of vehicle. This frame rotated in right-handed direction by $\psi$ about the $\mathbf{k}$ axis (z-axis). Essentially, the vehicle frame rotated in yaw
- **Vehicle-2 Frame**: Origin at centre of mass. Rotates vehicle-1 frame by pitch angle $\theta$ about the $\mathbf{j}$ axis (y-axis). $\mathbf{i}$ axis of this frame pointing towards nose.
- **Body Frame**: Rotates the vehicle-2 frame about the roll angle $\phi$. Basically, the frame if you were sitting in the aircraft
- Together, the $\psi$, $\theta$ and $\phi$ angles are the Euler angles of the vehicle. In practise, quaternions are used as they are not susceptible to gimbal lock 

## Sensors
- **Accelerometers** measure the acceleration
- **Gyroscopes** measure the angular velocity, $\Omega$ 

## Single Cell Voltage Chart 

```
4.20V ███████████████████████████████████████ 100%✅ FULL CHARGE
4.10V ████████████████████████████████████░░░ 90% ✅ Nearly Full
4.00V ████████████████████████████████░░░░░░░ 80% ✅ High
3.90V ███████████████████████████░░░░░░░░░░░░ 70% ✅ Good
3.85V ██████████████████████████░░░░░░░░░░░░░ 65% ✅ STORAGE VOLTAGE
3.80V █████████████████████████░░░░░░░░░░░░░░ 55% ✅ Medium-High
3.70V ████████████████████░░░░░░░░░░░░░░░░░░░ 45% ✅ NOMINAL (Label)
3.60V ███████████████░░░░░░░░░░░░░░░░░░░░░░░░ 30% ⚠️ Medium
3.50V ██████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ 20% ⚠️ LAND NOW
3.40V █████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ 10% 🚨 Land Immediately
3.30V ██░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ 5%  🚨 MINIMUM SAFE
3.20V ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ <5% ❌ DAMAGED
3.00V ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ 0%  ❌ DEAD/DANGEROUS
```
