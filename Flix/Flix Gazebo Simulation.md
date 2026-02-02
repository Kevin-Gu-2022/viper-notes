# How the Gazebo Simulation for Flix Works

## Overview

The Gazebo simulation for Flix is a clever piece of software engineering that allows you to test the exact same flight controller code that runs on the real ESP32 microcontroller, but within a physics simulation environment. The key insight here is that instead of writing separate simulation code, the developers created a compatibility layer that lets the Arduino firmware run directly inside Gazebo.

## Architecture: Three Key Components

Think of the simulation as having three main layers that work together, like a sandwich:

### 1. The Physical Model (The Bottom Layer)

At the foundation is a detailed physics model of the drone defined in `models/flix/flix.sdf`. This SDF (Simulation Description Format) file tells Gazebo everything about the drone's physical properties. The model includes a 65 gram body with realistic moments of inertia, which means it captures how the drone resists rotation around different axes. This is crucial because a real tiny drone behaves very differently from a large quadcopter.

The model also includes a virtual IMU sensor that runs at 1000 Hz, just like the real MPU9250 sensor on the physical drone. Importantly, this virtual sensor includes realistic noise characteristics: the gyroscope has 0.1 degrees per second of random noise, and the accelerometer has 8 milligrams of noise. These noise levels match what you'd see from the actual sensor, which helps ensure your control algorithms that work in simulation will also work on real hardware.

### 2. The Arduino Compatibility Layer (The Middle Layer)

Here's where things get interesting. The file `Arduino.h` provides a partial implementation of the Arduino API that works within the Gazebo plugin environment. This is essentially creating a fake Arduino platform that the firmware thinks it's running on.

For example, when the firmware calls `Serial.print()`, this compatibility layer intercepts that call and routes it to standard output in the terminal where Gazebo is running. When the firmware calls `micros()` to get the current time in microseconds, the compatibility layer returns the simulation time from Gazebo's physics engine. The `delay()` function uses C++ standard library sleep functions instead of Arduino's timing system.

Some functions are completely stubbed out because they don't make sense in simulation. For instance, `ledcAttach()` and `ledcWrite()` normally control PWM outputs to the motor drivers, but in simulation we're applying forces directly to the physics model, so these just return true without doing anything.

### 3. The Gazebo Plugin (The Top Layer)

The `simulator.cpp` file implements a Gazebo model plugin called `ModelFlix` that ties everything together. This plugin is where the real magic happens.

When Gazebo loads the world, it creates the drone model and attaches this plugin to it. The plugin's `Load()` method is called once during initialization, and it performs the Arduino-style setup: it calls `setupParameters()` to load saved control gains from virtual flash storage, and it initializes various subsystems.

Then, every physics simulation step (which happens 1000 times per second based on the world file's max_step_size of 0.001), the `OnUpdate()` method is called. This is the heartbeat of the simulation.

## The Update Loop: Where Everything Comes Together

Let's walk through what happens during each simulation step, because this is where you can really see how the real firmware runs in the virtual environment:

First, the plugin reads the current simulation time from Gazebo and makes it available to the firmware code through the global variable `t`. This ensures the firmware's timing calculations use the simulation clock rather than real-world time.

Next, it reads the virtual IMU sensor. The plugin gets the angular velocity (gyro) and linear acceleration from Gazebo's IMU sensor object. Notice that the accelerometer data passes through a low-pass filter with alpha=0.1 before being stored in the global `acc` variable. This is intentional - it smooths out high-frequency noise that might not be present in Gazebo's physics but would be in the real sensor.

The firmware then runs through its normal control loop: `readRC()` checks for remote control input, `estimate()` runs the attitude estimation algorithm using the gyro and accelerometer data, and `control()` executes the PID control loops to calculate motor commands.

Here's a subtle but important detail: after the attitude estimator runs, the plugin forces the yaw angle to match Gazebo's actual yaw using `attitude.setYaw(this->model->WorldPose().Yaw())`. This is done because small errors in yaw estimation can accumulate over time without any absolute reference (this is called yaw drift), and in the simulation we have perfect knowledge of the true orientation, so we might as well use it. On the real drone, you'd need a magnetometer or GPS to correct yaw drift.

After the control loop calculates motor thrust values (stored in the `motors[]` array), the plugin needs to translate these into actual forces and torques in the physics simulation. The `applyMotorForces()` method does this conversion.

## Motor Force Translation: From PWM Values to Physics

In the real world, motor thrust values (ranging from 0 to 1) get converted to PWM signals that drive the motors. In simulation, we need to convert these thrust values into Newtonian forces and torques that Gazebo's physics engine understands.

The plugin uses a measured maximum thrust of about 30 grams-force (0.03 times Earth's gravity) per motor. This value was determined empirically by the developers through testing. Each motor's thrust is multiplied by this maximum and applied as an upward force at the motor's position relative to the drone's center of mass.

The motors are positioned at a distance of 0.035355 meters (about 35mm) from the center, arranged in an X configuration. When you apply thrust at these offset positions, you naturally create torques around the center of mass, which is exactly how the real drone works.

There's also a clever touch for realism: each motor has a slightly different scale factor (1.0, 1.1, 0.9, 1.05). This simulates the natural asymmetry you find in real hardware where motors aren't perfectly matched. This asymmetry forces the control system to actually work to maintain stability, just like on a real drone.

Additionally, the motors create yaw torque because of propeller drag. Each motor applies a torque of about 24 gram-centimeters in the direction determined by its rotation. This is why drones typically use counter-rotating propeller pairs - to allow yaw control by varying the balance between clockwise and counterclockwise spinning motors.

## Control Input: Remote Control and MAVLink

The simulation supports two ways to control the drone, mirroring what's possible with the real hardware.

For USB remote control, there's a joystick interface (implemented in `joystick.h` that we haven't examined in detail, but it reads from Linux's joystick device files). The `readRC()` function in the firmware reads these joystick values and maps them to control stick positions.

For smartphone control, the simulation includes a full MAVLink communication stack. MAVLink is a lightweight messaging protocol designed for communicating with small unmanned vehicles. The `wifi.h` and `mavlink.ino` files implement UDP broadcasting of telemetry data and reception of control commands.

When you open QGroundControl on your phone, it discovers the simulated drone on the local network through MAVLink heartbeat messages, connects to it, and can send control inputs through the MAVLink manual control message. The firmware receives these messages and converts them to the same control stick positions that would come from a physical RC receiver.

## Why This Design is Powerful

The beauty of this architecture is that you're testing the exact same control algorithms, the same PID loops, the same attitude estimation code that will run on the real hardware. There's no translation layer or simulation-specific control code. If you tune your PID gains in simulation and they work well, there's a very high probability they'll work similarly on the real drone (though you'll still want to be conservative when first testing in the real world).

The Arduino compatibility layer is minimal and focused. It doesn't try to simulate every Arduino function - only the ones the Flix firmware actually uses. This keeps the complexity manageable while still providing high fidelity where it matters: the IMU data, the timing, and the control loop execution.

The parameter system works identically in simulation and reality, using the same storage format. When you adjust control gains through the command line interface or via MAVLink parameters, those changes get saved to a virtual flash storage system that persists between simulation runs, just like they would on the real drone's ESP32 flash memory.

## Building and Running

When you run `make simulator`, a few things happen behind the scenes. The Makefile compiles `simulator.cpp` along with all the firmware `.ino` files into a shared library called `libflix.so`. This shared library is what Gazebo loads as a plugin.

The compilation is a bit unusual because `.ino` files are Arduino sketches, not standard C++ files. But since the compatibility layer provides Arduino's API, they compile just fine with a regular C++ compiler. The `#include` directives in `simulator.cpp` literally include the entire `.ino` files, treating them as if they were `.cpp` files.

Gazebo then loads the `flix.world` file, which describes the simulation environment. This world file includes the floor model for visual reference, a sun for lighting, and most importantly, the flix model at 30cm altitude (so it starts hovering rather than on the ground). The world file references the flix model, which in turn references the `libflix.so` plugin.

When the simulation starts running, you get a 3D visualization of the tiny drone with semi-transparent propeller disks. The white disks indicate front motors, and the red ones indicate rear motors, which helps you understand the drone's orientation.

## Practical Implications

This simulation environment lets you do several valuable things safely:

You can crash repeatedly without any risk or cost. Want to test an aggressive control algorithm? Try it in simulation first. If the virtual drone flips and crashes, you just hit the reset button and try again.

You can log extensive data. The logging system records attitude, rates, control targets, and motor outputs at 100 Hz. In simulation, you can let this run for as long as you want without worrying about memory limits, then dump all the data and analyze it in detail.

You can test edge cases. What happens if you lose RC signal? The failsafe code will engage and the drone should descend gradually. You can verify this behavior works correctly before relying on it with real hardware.

You can iterate on control gains quickly. Adjusting PID parameters on a real drone is stressful because bad gains can cause crashes. In simulation, you can rapidly test different values and see their effect on stability and responsiveness without any risk.

The simulation isn't perfect - no simulation is. The aerodynamic effects are simplified (real propellers create complex airflow patterns), there's no ground effect modeled, and the sensor noise might not capture all the characteristics of real sensors. But for developing and testing the core control algorithms, this simulation provides an remarkably accurate testbed that will save you time, money, and frustration compared to testing exclusively on real hardware.