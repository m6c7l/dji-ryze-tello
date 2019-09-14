#### What is telemetry?

* Telemetry is a way to collect measurements and other data, e.g. state, from a remotely controlled device like a drone.
* The Tello drone sends telemetry data via UDP to destination port 88xx about every 100 ms.

#### What data is provided?

* inertial measurements in three spatial and (mostly) orthogonal directions
  * linear acceleration from the accelerometer (to infer pose relative to earth's gravity pulling the drone down)
  * angular velocity from the gyroscope (provides fast response of changes in pose since accelerometers are inherently slow)
* measurements relative to earth's magnetic field in three spatial and (mostly) orthogonal directions
  * magnetic flux density of the magnetometer (to infer orientation)
* ambient data
  * temperatures from two temperature sensors for overheat protection
  * atmospheric pressure from the barometer (to infer height)
* important states
  * battery level
  * time being airborne
* estimations processed on the drone
  * Euler angles of the absolute pose
  * distance to the drone estimated by the time difference due to the propagation delay (time of flight)

#### How to collect telemetry data?

* Just connect your computer via Wifi to the Tello drone and start the script.