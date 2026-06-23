# Inner Loop Tuning
- Goal is to make it stay at a certain angle
- Moving the joystick should have no effect, because I am not bypassing the outer loop, unlike Flix
	- The joystick inputs are angle targets, not rate targets

## Pitch
p = 0.35, i = 0.4, d = 0.005
3.429 - 3.142 = 0.287 overshoot
basically no oscillations
much quicker response
## Roll
Overshoot of only 3.982 - 3.142 = 0.84
Basically zero oscillations: 0.012 
p = 0.05, i = 0.2, d = 0.001
## Yaw
p = 0.5
i = 0.2
d = 0.001
ok response. not terribly important


# Outer Loop Tuning
- Goal is to make it stay at whatever orientation is given by joystick
- Have found it not very effective when thrust increased...
- Two targets: tracking (how close it tracks to joystick), as well as disturbance rejection (e.g. mass off centre, wind, prods)

## Pitch
p = 6.0
i & d are 0 in test results, but have since made them non-zero because disturbance causes drift

## Roll
- p = 6, i = 0.01, d = 0.001

## Yaw
- Test results are with 0 yaw
- Though I changed it to use non-zero otherwise, yaw drifts when disturbance