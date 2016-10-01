# v-rep python
Simple python binding for
[Coppelia Robotics V-REP simulator](http://www.coppeliarobotics.com/) ([remote API](http://www.coppeliarobotics.com/helpFiles/en/remoteApiOverview.htm))

## Getting started
0. Requirements: CPython >= 3.5.2 version, pip
1. Install library from PyPI by entering this command:
    `pip install vrep-python`
2. Create environment variable with path to platform-specific native library (remoteApi):
    `VREP_LIBRARY="vrep_folder/programming/remoteApiBindings/lib/lib/64Bit/"`
    * For windows users:
        *NOT TESTED*
    * For linux users:
        Add `export VREP_LIBRARY="vrep_folder/programming/remoteApiBindings/lib/lib/64Bit/"`
        to `.bashrc`
    
3. Find the socket port number in `V-REP/remoteApiConnections.txt` and use
    it to connect to the server simulator

## Currently implemented things
In the current version is not implemented features such as remote management GUI,
additional configuration properties of objects and shapes, etc.
Basically implemented those components that are required to control the robot:
* Joint
* Proximity sensor
* Vision sensor
* Force sensor
* Position sensor (used for that dummy or shape object)
* ~~Remote function calls~~

## Example
```python
from vrepapi import VRepApi
import time

class PioneerP3DX:

    def __init__(self, api: VRepApi):
        self._api = api
        self._left_motor = api.joint.with_velocity_control("left_motor")
        self._right_motor = api.joint.with_velocity_control("right_motor")
        self._left_sensor = api.sensor.proximity("left_sensor")
        self._right_sensor = api.sensor.proximity("right_sensor")

    def rotate_right(self, speed=2.0):
        self._set_two_motor(speed, -speed)

    def rotate_left(self, speed=2.0):
        self._set_two_motor(-speed, speed)

    def move_forward(self, speed=2.0):
        self._set_two_motor(speed, speed)

    def move_backward(self, speed=2.0):
        self._set_two_motor(-speed, -speed)

    def _set_two_motor(self, left: float, right: float):
        self._left_motor.set_target_velocity(left)
        self._right_motor.set_target_velocity(right)

    def right_length(self):
        return self._right_sensor.read()[1].distance()

    def left_length(self):
        return self._left_sensor.read()[1].distance()

api = VRepApi.connect("127.0.0.1", 19997)

api.simulation.start()

r = PioneerP3DX(api)
for _ in range(100):
    rl = r.right_length()
    ll = r.left_length()
    if rl != 0 and rl < 10:
        r.rotate_left()
    elif ll != 0 and ll < 10:
        r.rotate_right()
    else:
        r.move_forward()
    time.sleep(0.1)

api.simulation.pause()
```


