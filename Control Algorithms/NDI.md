## The Fundamental Equations

The dynamics of a nonlinear control system are described by two coupled equations. The state equation **ẋ = f(x) + G(x)u** tells you how your system's complete internal state evolves over time, while the output equation **y = h(x)** tells you what your sensors can actually measure about that state.

## Understanding the Components

Your state vector **x** contains everything needed to completely describe your system at any moment. For a quadcopter, this includes position, velocity, orientation, and angular rates—twelve numbers that capture the full picture of where the drone is and how it's moving.

The derivative **ẋ** tells you how fast each component of the state is changing right now. This is your velocity vector in state space, showing which direction and how fast the system is evolving.

The drift term **f(x)** describes how your system naturally evolves without any control input. For your drone, this includes gravity pulling it down, air resistance slowing it, and gyroscopic effects from spinning propellers. If you cut power to all motors, f(x) governs what happens next.

The control term **G(x)u** represents how your actuators affect the state. The matrix G(x) encodes the control effectiveness, showing how motor thrusts translate into forces and torques. The dependence on x captures the nonlinearity—the same motor command has different effects depending on your current state, like how thrust pushes you down when inverted instead of up.

## Measurements and Their Derivatives

Your measurement vector **y = h(x)** contains only what your sensors can observe, which is often a limited view of the full state. An IMU gives you accelerations and angular rates, but not position or velocity directly. The function h maps from the complete state to this observable subset.

The measurement derivative **ẏ** tells you how your sensor readings are changing. Using the chain rule, this connects to the state derivative through **ẏ = (∂h/∂x)ẋ**, which says that measurement changes depend on how sensitive the measurements are to state changes, multiplied by how the state itself is changing.

## Lie Derivatives

When you substitute the dynamics equation into the expression for ẏ, you get **ẏ = (∂h/∂x)f(x) + (∂h/∂x)G(x)u**. The two terms appear so frequently in nonlinear control that they get special notation. We write **L_f h(x) = (∂h/∂x)f(x)** as the Lie derivative of h with respect to f, which captures how measurements naturally evolve due to drift. Similarly, **L_G h(x) = (∂h/∂x)G(x)** captures how measurements respond to control inputs. The complete equation becomes **ẏ = L_f h(x) + L_G h(x)u**, cleanly separating natural evolution from controlled changes.