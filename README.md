# Obstacle Avoidance: URC 2026 Rover

Obstacle avoidance and GPS/IMU navigation for the Wisconsin Robotics 2026
University Rover Challenge (URC) rover. Fork of
[WisconsinRobotics/WRoverSoftware_25-26](https://github.com/WisconsinRobotics/WRoverSoftware_25-26);
this README covers the parts I wrote.

## Demo
<drag SAR video here>

Rover detects and avoids an obstacle during the SAR run. The avoidance
worked, but motion was jerky and took ~5 s (see Limitations).

## My contributions
| Package | Role |
|---|---|
| `navigation` | <algorithm, e.g. how it picks a heading and outputs drive commands> |
| `gps_tools` | <what it does, e.g. coordinate/bearing/distance utilities> |
| `wr_imu_compass` | <heading estimation from IMU> |

Co-designed with the team (implemented by others): master finder, state machine.

## How it works
<3 to 5 lines: inputs (GPS, IMU, <obstacle source>) -> navigation -> drive commands>

## Validation
- VFH+ (Vector Field Histogram+: bins obstacle density by direction and steers
  into the best free gap) worked in testing on a kart.
- Did not carry over to the rover. Suspected cause: poor localization
  (not verified).

## Limitations / future work
- Heading controller was P-only (proportional gain, no integral or derivative
  term), likely contributing to jerky motion.
- Next: add a D term using IMU gyro yaw rate (not differentiated heading error,
  which amplifies noise); log pose vs. ground truth to test the localization
  hypothesis; re-run VFH+ on the rover.

## Setup
Upstream GNSS/RTK setup from the team repo.

### Installation
Install the `ublox` package with:
```bash
sudo apt update
sudo apt install ros-[distro]-ublox-dgnss
```

### Running the Node
After setting up RTK on the base with ublox, make each base start receiving RTK and publishing the corrected topics:
```bash
ros2 run ublox_dgnss_node ublox_dgnss_node --ros-args -r __ns:=/rover1 -p device_serial_string:="'1'"
```
- Replace `/rover1` with your desired output topic namespace.
- Replace `"'1'"` with your device's specific id.

### Output Topics
For this example, the output topic is `/rover1/ubx_nav_pvt`, of type `UBXNavPVT`, importable from the installed `ublox-gps` package:
```python
from ublox_ubx_msgs.msg import UBXNavPVT
```

### Running base station launch scripts
Configure RTK on the base station:
```bash
./run_rtk_base_station.sh POLE_LAT POLE_LON POLE_ALT DISTANCE BEARING
# Example
./run_rtk_base_station.sh 38.221704 -110.421456 1309.99 0 0
```

Run the path planning, spiral planning, and base station controller:
```bash
./run_state_machine_base_station.sh LIDAR_PATH
# Example
./run_state_machine_base_station.sh ~/motel.laz
```

