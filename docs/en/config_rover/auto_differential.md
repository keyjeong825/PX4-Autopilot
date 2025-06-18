1. [RD_TRANS_DRV_TRN](#RD_TRANS_DRV_TRN) ($\degree$): If the angle is smaller than 180° - [RD_TRANS_DRV_TRN](#RD_TRANS_DRV_TRN) the rover will come to a full stop at the waypoint to then turn on the spot. For more information on the [RD_TRANS_DRV_TRN](#RD_TRANS_DRV_TRN) parameter see [State Machine](#state-machine).

### State Machine

The module employs the following state machine to make full use of a differential rovers ability to turn on the spot:

![Differential state machine](../../assets/airframes/rover/rover_differential/differential_state_machine.png)

These transition thresholds can be set with [RD_TRANS_DRV_TRN](#RD_TRANS_DRV_TRN) and [RD_TRANS_TRN_DRV](#RD_TRANS_TRN_DRV).
