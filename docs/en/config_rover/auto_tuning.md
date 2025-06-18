# Auto Tuning

::: warning
For auto modes to work properly [Manual Mode](#manual-mode), [Acro mode](#acro-mode)and [Position mode](#position-mode) must already be configured!
:::

<a id="pure_pursuit_controller"></a>

In [auto modes](../flight_modes_rover/auto.md#mission-mode) the autopilot takes over navigation tasks.

The required parameter configuration is discussed in the following sections.

### Speed

1. (Optional) [RO_SPEED_RED](#RO_SPEED_RED): Tuning parameter for the speed reduction based on the course error.
   This can be used to limit the maximum allowed speed based on the difference between the current course and the bearing setpoint:
   $v_{max} = v_{full throttle} \cdot (1 - \theta_{normalized} \cdot k) $

   with

   - $v_{max}:$ Maximum speed
   - $v_{full throttle}:$ Speed at maximum throttle [RO_MAX_THR_SPEED](#RO_MAX_THR_SPEED)
   - $\theta_{normalized}:$ Course error (Course - bearing setpoint) normalized from  $[0\degree, 180\degree]$ to $[0, 1]$
   - $k:$ Tuning parameter [RO_SPEED_RED](#RO_MAX_THR_SPEED)

   ::: note
   This parameter is used to calculate the speed at which the vehicle arrives at a waypoint based on the upcoming corner.
   Set to -1 to disable course error based speed reduction.
   :::

2. (Optional) [RO_DECEL_LIM](#RO_DECEL_LIM) [m/s^2] and [RO_JERK_LIM](#RO_JERK_LIM) [m/s^3] are used to calculate a speed trajectory such that the rover reaches the next waypoint with the calculated cornering speed.

   ::: tip
   Plan a mission for the rover to drive a square and observe how it slows down when approaching a waypoint:

   - If the rover decelerates too quickly decrease the [RO_DECEL_LIM](#RO_DECEL_LIM) parameter, if it starts slowing down too early increase the parameter.
   - If you observe a jerking motion as the rover slows down, decrease the [RO_JERK_LIM](#RO_JERK_LIM) parameter otherwise increase it as much as possible as it can interfere with the tuning of [RO_DECEL_LIM](#RO_DECEL_LIM).

   These two parameters have to be tuned as a pair, repeat until you are satisfied with the behaviour.
   :::

3. Plot the `adjusted_speed_body_x_setpoint` and `measured_speed_body_x` from the [RoverVelocityStatus](../msg_docs/RoverVelocityStatus.md) message over each other.
   If the tracking of these setpoints is not satisfactory adjust the values for [RO_SPEED_P](#RO_SPEED_P) and [RO_SPEED_I](#RO_SPEED_I).

### Corner Cutting

The module employs a special cornering logic causing the rover to "cut corners" to achieve a smooth trajectory.
This is done by scaling the acceptance radius based on the corner the rover has to drive (for geometric explanation see [Cornering logic](#corner-cutting-logic-info-only)).

![Cornering Logic](../../assets/airframes/rover/rover_ackermann/cornering_comparison.png)

The degree to which corner cutting is allowed can be tuned, or disabled, with the following parameters:

::: info
The corner cutting effect is a tradeoff between how close you get to the waypoint and the smoothness of the trajectory.
:::

1. [NAV_ACC_RAD](../advanced_config/parameter_reference.md#NAV_ACC_RAD) [m]: Default acceptance radius
   This is also used as a lower bound for the acceptance radius scaling.
2. [RA_ACC_RAD_MAX](#RA_ACC_RAD_MAX) [m]: The maximum the acceptance radius can be scaled to.
   Set equal to [NAV_ACC_RAD](../advanced_config/parameter_reference.md#NAV_ACC_RAD) to disable the corner cutting effect.
3. [RA_ACC_RAD_GAIN](#RA_ACC_RAD_GAIN) [-]: This tuning parameter is a multiplicand on the [calculated ideal acceptance radius](#corner-cutting-logic) to account for dynamic effects.

   :::tip
   Initially set this parameter to `1`.
   If you observe the rover overshooting the corner, increase this parameter until you are satisfied with the behaviour.
   Note that the scaling of the acceptance radius is limited by [RA_ACC_RAD_MAX](#RA_ACC_RAD_MAX).
   :::

### Path Following

The [pure pursuit](#pure-pursuit-guidance-logic) algorithm is used to calculate a yaw rate setpoint for the vehicle that is then close loop controlled.
The close loop yaw rate was tuned in the configuration of the [Acro mode](#acro-mode), and the pure pursuit was tuned when setting up the [Position mode](#position-mode).
During any auto navigation task observe the behaviour of the rover.

If you are unsatisfied with the path following, there are 2 steps to take:

1. Plot the `adjusted_yaw_rate_setpoint` and the `measured_yaw_rate` from the [RoverRateStatus](../msg_docs/RoverRateStatus.md) over each other.
   If the tracking of these setpoints is not satisfactory adjust the values for [RO_YAW_RATE_P](#RO_YAW_RATE_P) and [RO_YAW_RATE_I](#RO_YAW_RATE_I).
2. Plot the `adjusted_yaw_setpoint` and the `measured_yaw` from the [RoverAttitudeStatus](../msg_docs/RoverAttitudeStatus.md) over each other.
   If the tracking of these setpoints is not satisfactory adjust the value for [RO_YAW_P](#RO_YAW_RATE_P) and potentially further tune [RO_YAW_RATE_I](#RO_YAW_RATE_I).
3. Steps 1 and 2 ensures accurate setpoint tracking, if the path following is still unsatisfactory you need to further tune the [pure pursuit](#pure-pursuit-guidance-logic) parameters.

## Pure Pursuit Guidance Logic

The desired yaw setpoints are generated using a pure pursuit algorithm.

The controller takes the intersection point between a circle around the vehicle and a line segment.
In mission mode this line is usually constructed by connecting the previous and current waypoint.

![Pure Pursuit Algorithm](../../assets/airframes/rover/flight_modes/pure_pursuit_algorithm.png)

The radius of the circle around the vehicle is used to tune the controller and is often referred to as look-ahead distance.

The look-ahead distance sets how aggressive the controller behaves and is defined as $l_d = v \cdot k$.
It depends on the velocity $v$ of the rover and a tuning parameter $k$ that can be set with the parameter [PP_LOOKAHD_GAIN](#PP_LOOKAHD_GAIN).

::: info
A lower value of [PP_LOOKAHD_GAIN](#PP_LOOKAHD_GAIN) makes the controller more aggressive but can lead to oscillations!
:::

The lookahead is constrained between [PP_LOOKAHD_MAX](#PP_LOOKAHD_MAX) and [PP_LOOKAHD_MIN](#PP_LOOKAHD_MIN).

If the distance from the path to the rover is bigger than the lookahead distance, the rover will target the point on the path that is closest to the rover.

To summarize, the following parameters can be used to tune the controller:

| Parameter                                                                                                | Description                             | Unit |
| -------------------------------------------------------------------------------------------------------- | --------------------------------------- | ---- |
| <a id="PP_LOOKAHD_GAIN"></a>[PP_LOOKAHD_GAIN](../advanced_config/parameter_reference.md#PP_LOOKAHD_GAIN) | Main tuning parameter                   | -    |
| <a id="PP_LOOKAHD_MAX"></a>[PP_LOOKAHD_MAX](../advanced_config/parameter_reference.md#PP_LOOKAHD_MAX)    | Maximum value for the look ahead radius | m    |
| <a id="PP_LOOKAHD_MIN"></a>[PP_LOOKAHD_MIN](../advanced_config/parameter_reference.md#PP_LOOKAHD_MIN)    | Minimum value for the look ahead radius | m    |

## Corner Cutting Logic (Info only)

To enable a smooth trajectory, the acceptance radius of waypoints is scaled based on the angle between a line segment from the current-to-previous and current-to-next waypoints.
The ideal trajectory would be to arrive at the next line segment with the heading pointing towards the next waypoint.
For this purpose the minimum turning circle of the rover is inscribed tangentially to both line segments.

![Cornering Logic](../../assets/airframes/rover/rover_ackermann/cornering_logic.png)

The acceptance radius of the waypoint is set to the distance from the waypoint to the tangential points between the circle and the line segments:

$$
\begin{align*}
r_{min} &= \frac{L}{\sin\left( \delta_{max}\right) } \\
\theta  &= \frac{1}{2}\arccos\left( \frac{\vec{a}*\vec{b}}{|\vec{a}||\vec{b}|}\right) \\
r_{acc} &= \frac{r_{min}}{\tan\left( \theta\right) }
\end{align*}
$$

| Symbol         | Description                        | Unit |
| -------------- | ---------------------------------- | ---- |
| $\vec{a}$      | Vector from current to previous WP | m    |
| $\vec{b}$      | Vector from current to next WP     | m    |
| $r_{min}$      | Minimum turn radius                | m    |
| $\delta_{max}$ | Maximum steer angle                | m    |
| $r_{acc}$      | Acceptance radius                  | m    |
