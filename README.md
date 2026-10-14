[README.md](https://github.com/user-attachments/files/33446104/README.md)
# Robot Arm

A 5-DOF pick-and-place robot arm controlled by a Raspberry Pi, with inverse
kinematics and a web-based GUI.

## Hardware

- Raspberry Pi (any model with I2C)
- PCA9685 16-channel PWM servo driver
- 5 x TowerPro MG996R (or similar) 180° servo motors
- 3D-printed / kit arm frame

### Wiring (servo → PCA9685 channel)

| Joint    | Channel |
| -------- | ------- |
| Gripper  | 0       |
| Wrist    | 1       |
| Elbow    | 2       |
| *(unused)* | 3     |
| Shoulder | 4       |
| Base     | 5       |

## Software architecture

```
arm_config.py    Joint calibration, link lengths, servo parameters
hardware.py      PCA9685 driver with MockPWM laptop fallback
servo.py         Single-servo control: clamping, smooth ramps
kinematics.py    Forward + inverse kinematics (base yaw + 2-link planar + wrist)
robot_arm.py     High-level controller: named joints, IK move, poses, sequences
app.py           Flask web GUI for all of the above
calibrate.py     Low-level servo test / calibration tool
```

## Setup

### On the Raspberry Pi

```bash
git clone https://github.com/anhle2403/robot-arm.git
cd robot-arm

pip install -r requirements.txt
pip install adafruit-circuitpython-pca9685 adafruit-blinka
```

### On a laptop (no hardware)

```bash
pip install -r requirements.txt
```

The code auto-detects missing I2C and falls back to `MockPWM`, which logs
servo commands to the console so you can develop without the arm.

## Quick start

### 1. Calibrate (important!)

Measure your arm's link lengths and calibrate each servo — see
[CALIBRATION.md](CALIBRATION.md).

### 2. Run the web GUI

```bash
python app.py
```

Open `http://<pi-ip>:5000` in a browser. You get:

- **Joint sliders** — jog each servo in real time
- **Cartesian move** — enter (x, y, z, pitch) and the arm uses IK
- **Gripper** — open / close buttons
- **Poses** — save and recall named positions
- **Sequences** — record a series of poses and play them back
- **Persistence** — save/load everything to `arm_data.json`

### 3. Python API

```python
from robot_arm import RobotArm

arm = RobotArm()
arm.home()

arm.move_joint("base", 45)
arm.move_to(150, 0, 100, pitch=0)  # IK move

arm.gripper_close()
arm.save_pose("pick")
arm.recall_pose("pick")
arm.gripper_open()

arm.shutdown()
```

## Project status

- [x] Modular servo control with calibration
- [x] Forward + inverse kinematics (verified over 5000 random poses)
- [x] Web GUI with joint jog, IK, gripper, poses, sequences
- [x] MockPWM fallback for laptop development
- [ ] Calibrate with real arm measurements
- [ ] Trajectory smoothing with acceleration limits
- [ ] Camera + object detection (pick-by-sight)
- [ ] Gamepad teleop
- [ ] 3D arm visualiser in the web GUI

## License

MIT
