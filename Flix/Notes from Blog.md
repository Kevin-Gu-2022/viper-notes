## Blog
https://habr.com/ru/articles/814127/

## State Estimation
- Complementary filter in *state estimation* subsystem combines the gyro and acceleration readings
	- In `estimate.ino`
## Control Loop
- Typical quadcopter control algorithms have multiple layers of control loops:
```mermaid
flowchart TD
	S([User Input]) -->|Target Position| B
    B[Position Control] -->|Target Linear Velocity| C
    C[Linear Velocity Control] -->|Target Orientation| D
    D[Orientation Control] -->|Target Angular Velocity| F[Angular Velocity Control]
    F --> M([Control Signals to Motor])

```
- Uses a cascaded control loop that controls the attitude and the angular velocity of the quadcopter (lowest level of cascade) 
	- In `control.ino`
- The user's input and the current attitude are used as inputs to the attitude control level, which itself outputs the desired angular velocity for the next level
- The angular velocity level uses a PID controller to determine the necessary control signals for the motors
- The motor control signals go to the motor mixer that actually delivers the correct controls to individual motors. See details [[Motor Mixer|here]].
- Control loop constrained by IMU frequency, which is set at 1 kHz

## Control Algorithm Flowchart
![[d2.svg]]

## Attitude Controller
```c++
void interpretControls() {
	//...
	if (mode == STAB) {
		float yawTarget = attitudeTarget.getYaw();
		if (!armed || invalid(yawTarget) || controlYaw != 0) yawTarget = attitude.getYaw(); // reset yaw target
		attitudeTarget = Quaternion::fromEuler(Vector(controlRoll * tiltMax, controlPitch * tiltMax, yawTarget));
		ratesExtra = Vector(0, 0, -controlYaw * maxRate.z); // positive yaw stick means clockwise rotation in FLU
	//...
}



void controlAttitude() {
	if (!armed || attitudeTarget.invalid() || thrustTarget < 0.1) return; // skip attitude control

	const Vector up(0, 0, 1);
	Vector upActual = Quaternion::rotateVector(up, attitude);
	Vector upTarget = Quaternion::rotateVector(up, attitudeTarget);

	Vector error = Vector::rotationVectorBetween(upTarget, upActual);

	// This ratesExtra.x and .y are both 0
	ratesTarget.x = rollPID.update(error.x) + ratesExtra.x;
	ratesTarget.y = pitchPID.update(error.y) + ratesExtra.y;

	float yawError = wrapAngle(attitudeTarget.getYaw() - attitude.getYaw());
	// The feedforward term allows continuous yaw rotation
	ratesTarget.z = yawPID.update(yawError) + ratesExtra.z;
}
```