# Velocity Tuning

:::warning
For this mode to work properly [Stabilized mode](#stabilized-mode) must already be configured!
:::

[Position mode](../flight_modes_rover/manual.md#position-mode) is the most advanced manual mode, utilizing closed loop yaw rate and speed control and leveraging position estimates.

To configure set the following parameters:

1. [RO_SPEED_LIM](#RO_SPEED_LIM) [m/s]: This is the maximum speed you want to allow for your rover.
   This will define the stick-to-speed mapping for position mode and set an upper limit for the speed setpoint for all [auto modes](#auto-modes).
1. [RO_MAX_THR_SPEED](#RO_MAX_THR_SPEED) [m/s]: This parameter is used to calculate the feed-forward term of the closed loop speed control which linearly maps desired speeds to normalized motor commands.
   As mentioned in the [Manual mode](../flight_modes_rover/manual.md#manual-mode) configuration , a good starting point is the observed ground speed when the rover drives at maximum throttle in [Manual mode](../flight_modes_rover/manual.md#manual-mode).

   <a id="RA_SPEED_TUNING"></a>

   ::: tip
   To further tune this parameter:

   1. Set [RO_SPEED_P](#RO_SPEED_P) and [RO_SPEED_I](#RO_SPEED_I) to zero.
      This way the speed is only controlled by the feed-forward term, which makes it easier to tune.
   1. Put the rover in [Position mode](../flight_modes_rover/manual.md#position-mode) and then move the left stick of your controller up and/or down and hold it at a few different levels for a couple of seconds each.
   1. Disarm the rover and from the flight log plot the `adjusted_speed_body_x_setpoint` and the `measured_speed_body_x` from the [RoverVelocityStatus](../msg_docs/RoverVelocityStatus.md) message over each other.
   1. If the actual speed of the rover is higher than the speed setpoint, increase [RO_MAX_THR_SPEED](#RO_MAX_THR_SPEED).
      If it is the other way around decrease the parameter and repeat until you are satisfied with the setpoint tracking.

   :::

   ::: info
   If your rover oscillates when driving a straight line in [Position mode](../flight_modes_rover/manual.md#position-mode), set this parameter to the observed ground speed at maximum throttle in [Manual mode](../flight_modes_rover/manual.md#manual-mode) and complete steps 5-7 first before continuing the tuning of the closed loop speed control (Steps 2-4).
   :::

1. [RO_SPEED_P](#RO_SPEED_P) [-]: Proportional gain of the closed loop speed controller.

   ::: tip
   This parameter can be tuned the same way as [RO_MAX_THR_SPEED](#RA_SPEED_TUNING).
   If you tuned [RO_MAX_THR_SPEED](#RO_MAX_THR_SPEED) well, you might only need a very small value.
   :::

1. [RO_SPEED_I](#RO_SPEED_I) [-]: Integral gain for the closed loop speed controller.

   ::: tip
   For the closed loop speed control an integrator gain is useful because this setpoint is often constant for a while and an integrator eliminates steady state errors that can cause the rover to never reach the setpoint.
   :::

1. [PP_LOOKAHD_GAIN](#PP_LOOKAHD_GAIN): When driving in a straight line (right stick centered) position mode leverages the same path following algorithm used in [auto modes](#auto-modes) called [pure pursuit](#pure-pursuit-guidance-logic) to achieve the best possible straight line driving behaviour ([Illustration of control architecture](#pure_pursuit_controller)).
   This parameter determines how aggressive the controller will steer towards the path.

   ::: tip
   Decreasing the parameter makes it more aggressive but can lead to oscillations.

   To tune this:

   1. Start with a value of 1 for [PP_LOOKAHD_GAIN](#PP_LOOKAHD_GAIN)
   1. Put the rover in [Position mode](../flight_modes_rover/manual.md#position-mode) and while driving a straight line at approximately half the maximum speed observe its behaviour.
   1. If the rover does not drive in a straight line, reduce the value of the parameter, if it oscillates around the path increase the value.
   1. Repeat until you are satisfied with the behaviour.

   :::

1. [PP_LOOKAHD_MIN](#PP_LOOKAHD_MIN): Minimum threshold for the lookahead distance used by the [pure pursuit algorithm](#pure-pursuit-guidance-logic).

   ::: tip
   Put the rover in [Position mode](../flight_modes_rover/manual.md#position-mode) and drive at very low speeds, if the rover starts to oscillate even though the tuning of [PP_LOOKAHD_GAIN](#PP_LOOKAHD_GAIN) was good for medium speeds, then increase the value of [PP_LOOKAHD_MIN](#PP_LOOKAHD_MIN).
   :::

1. [PP_LOOKAHD_MAX](#PP_LOOKAHD_MAX): Maximum threshold for the lookahead distance used by [pure pursuit](#pure-pursuit-guidance-logic).

   ::: tip
   Put the rover in [Position mode](../flight_modes_rover/manual.md#position-mode) and drive at very high speeds, if the rover does not drive in a straight line even though the tuning of [PP_LOOKAHD_GAIN](#PP_LOOKAHD_GAIN) was good for medium speeds, then decrease the value of [PP_LOOKAHD_MAX](#PP_LOOKAHD_MAX).
   :::

1. (Advanced) [RO_SPEED_TH](#RO_SPEED_TH) [m/s]: The minimum threshold for the speed measurement not to be interpreted as zero.
   This can be used to cut off measurement noise when the rover is standing still.

The rover is now ready to drive in [Position mode](../flight_modes_rover/manual.md#position-mode).
