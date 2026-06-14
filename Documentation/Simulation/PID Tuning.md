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
- Goal is to make it stay 

## Pitch
p = 6.0

## Roll
- p = 6, i = 0.01, d = 0.001