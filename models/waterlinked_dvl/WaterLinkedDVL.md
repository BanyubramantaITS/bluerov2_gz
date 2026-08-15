# Water Linked DVL A50

A standalone four-beam Doppler Velocity Log, modelled as its own Gazebo model so
it can be attached to a vehicle the way the real unit bolts onto a BlueROV2.

Requires **two** world plugins. `gz-sim-sensors-system` alone is not enough — the
DVL is serviced by its own system, and without it the sensor block is silently
ignored:

~~~xml
<plugin filename="gz-sim-sensors-system" name="gz::sim::systems::Sensors"/>
<plugin filename="gz-sim-dvl-system" name="gz::sim::systems::DopplerVelocityLogSystem"/>
~~~

## Usage

~~~bash
export GZ_SIM_RESOURCE_PATH=$GZ_SIM_RESOURCE_PATH:\
~/colcon_ws/src/bluerov2_gz/models
~~~

Include it in a world or a vehicle model:

~~~xml
<include>
  <uri>model://waterlinked_dvl</uri>
  <pose>0 0 -0.10 0 0 0</pose>
</include>
~~~

Watch the output:

~~~bash
gz topic -t /dvl/velocity -e
~~~

A message carrying `target { type: DVL_TARGET_BOTTOM }` and a populated
`velocity` block means bottom lock. Beams present but empty means no lock —
most often because the vehicle is below the 5 cm minimum altitude, i.e. sitting
on the bottom.

## Frames

The link origin is the **beam-axis intersection point**, 28 mm above the A50
back plate, because that is where Water Linked defines the sensor frame origin
and therefore what ArduSub's `VISO_POS_*` lever arm refers to. Keeping them
coincident means the mount pose copies straight into the params.

Beams point −Z at identity rotation, so a **downward mount needs no rotation** —
attach it with an identity pose and only the offset.

The *reported velocity* axes are **FRD** — x forward, y starboard, z down toward
the transducers — which is not the z-up body frame the BlueROV2 models use. That
conversion belongs to whatever consumes the velocity, not to the mount. See
<https://docs.waterlinked.com/dvl/axes/>.

## Fidelity

Modelled from Water Linked's published figures where they exist:

| Property | Value | Source |
|---|---|---|
| Minimum altitude | 0.05 m | [range mode](https://docs.waterlinked.com/dvl/range-mode/) |
| Update rate | 10 Hz (range mode 1, 0.3–3.0 m) | [range mode](https://docs.waterlinked.com/dvl/range-mode/) |
| Frame origin | 28 mm above back plate | [axes](https://docs.waterlinked.com/dvl/axes/) |

Two figures are **not** A50-verified and are carried over from Gazebo's shipped
`dvl_world.sdf` (a Teledyne Pathfinder): the **30° beam tilt** and the **0.002
m/s noise stddev**. Both are tuning knobs. The real A50 reports a per-ping
figure of merit rather than a fixed stddev, so a consumer should map that onto
the EKF's velocity error instead of trusting one constant.

The visual is an approximate envelope, not a dimensionally accurate A50.
