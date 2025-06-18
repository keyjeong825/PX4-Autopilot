# Attitude Tuning

::: warning
For this mode to work properly [Acro mode](#acro-mode) must've already been configured!
:::

For [Stabilized mode](../flight_modes_rover/differential.md#stabilized-mode) the controller utilizes a closed loop yaw controller, which creates a yaw rate setpoint to control the yaw when it is active:

![Cascaded PID for yaw control](../../assets/airframes/rover/rover_differential/cascaded_pid_for_yaw.png)

Unlike the closed loop yaw rate, this controller has no feed-forward term.
Therefore you only need to tune the closed loop gains:

1. [RO_YAW_P](#RO_YAW_P) [-]: Proportional gain for the closed loop yaw controller.

   ::: tip
   In stabilized mode the closed loop yaw control is only active when driving a straight line (no yaw rate input).
   To tune it start with a value of 1 for [RO_YAW_P](#RO_YAW_P).
   Put the rover into stabilized mode and move the left stick of your controller up to drive forwards.
   Disarm the rover and from the flight log plot the `measured_yaw` and the `adjusted_yaw_setpoint` from the [RoverAttitudeStatus](../msg_docs/RoverAttitudeStatus.md) message over each other.
   Increase/Decrease the parameter until you are satisfied with the setpoint tracking.
   :::

::: info
For the closed loop yaw control an integrator gain is useful because this setpoint is often constant for a while and an integrator eliminates steady state errors that can cause the rover to never reach the setpoint.
Since the yaw and yaw rate controllers are cascaded, there only needs to be one integrator which is in the yaw rate controller. If you observe a steady state error in the yaw setpoint increase the [RO_YAW_RATE_I](#RO_YAW_RATE_I) parameter.
:::

The rover is now ready to drive in [Stabilized mode](../flight_modes_rover/manual.md#stabilized-mode).
