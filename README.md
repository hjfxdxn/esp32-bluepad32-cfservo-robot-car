# ESP32 Bluepad32 CFServo Robot Car

Bluetooth gamepad control for a two-wheel CFServo robot car using an ESP32-based Waveshare Bus Servo Driver HAT (A), [Bluepad32](https://github.com/ricardoquesada/bluepad32), and CFServo torque/current mode.

This sketch reads an Xbox or PS5 controller over Bluetooth. The left stick sets target wheel speed, and each wheel is controlled with a simple P speed loop:

```cpp
FeedBack()
ReadSpeed()
torqueCmd = Kp * (refSpeed - actSpeed)
WriteEle()
```

## Hardware

- Waveshare Bus Servo Driver HAT (A), ESP32 based
- Two CFServo wheel servos
- Xbox Bluetooth controller or PS5 DualSense controller
- USB cable for programming and Serial Monitor

Default servo bus pins:

```cpp
#define S_RXD 18
#define S_TXD 19
```

Default wheel IDs:

```cpp
const byte LEFT_WHEEL_ID = 1;
const byte RIGHT_WHEEL_ID = 2;
```

Change these IDs in `eleTorquesegyo_torque_p.ino` if your servos use different IDs.

## Compatibility Notes

Bluepad32 supports many modern controllers, including Xbox Wireless controllers and Sony DualSense controllers.

For PS5 DualSense / PS4 DualShock controllers, use an original ESP32 chip such as ESP32-WROOM-32 or ESP32-WROVER. ESP32-S3, ESP32-C3, ESP32-C6, and ESP32-H2 do not support Bluetooth Classic, so many PlayStation controllers will not connect to those chips.

Xbox notes:

- Xbox 360 controllers are not supported over normal Bluetooth.
- Older Xbox One controllers, such as models 1537 / 1697, usually do not support standard Bluetooth.
- Xbox One S model 1708 and Xbox Series X/S model 1914 are expected to work with Bluepad32.

## Install Bluepad32

In Arduino IDE:

1. Open `File -> Preferences`.
2. Add these board manager URLs:

```text
https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
https://raw.githubusercontent.com/ricardoquesada/esp32-arduino-lib-builder/master/bluepad32_files/package_esp32_bluepad32_index.json
```

3. Open `Tools -> Board -> Boards Manager`.
4. Install:

```text
esp32
ESP32 + Bluepad32
```

5. Select a board under:

```text
Tools -> Board -> ESP32 + Bluepad32 Arduino
```

Use the Bluepad32 board package, not the normal ESP32 board package.

## Install CFServo / SCServo

This sketch uses:

```cpp
#include <SCServo.h>
```

Install the CFServo / SCServo Arduino library before compiling. If using the Waveshare / CFServo package folder, copy the library into your Arduino libraries directory and restart Arduino IDE.

Typical Windows library path:

```text
Documents/Arduino/libraries/
```

## Upload

1. Open `eleTorquesegyo_torque_p.ino`.
2. Select a board under `ESP32 + Bluepad32 Arduino`.
3. Select the serial port under `Tools -> Port`.
4. Upload the sketch.
5. Open Serial Monitor at `115200 baud`.

On startup:

```text
Bluepad32 Torque P Speed Control Ready
Pair Xbox / PS5 controller now
Left stick sets target wheel speed
Serial commands: f=forget pairing keys, e=enable pairing, s=stop
```

## Pair An Xbox Controller

1. Open Serial Monitor at `115200`.
2. If needed, send:

```text
f
```

3. Press `EN` to restart the ESP32.
4. After restart, send:

```text
e
```

5. Turn off the Xbox controller by holding the Xbox button for about 10 seconds.
6. Press the Xbox button to turn it on.
7. Hold the pairing button on top of the controller until the Xbox light flashes quickly.
8. Wait for:

```text
Controller connected: slot=0, model=...
```

## Pair A PS5 DualSense Controller

1. Use an original ESP32 board, not ESP32-S3 / C3 / C6.
2. Open Serial Monitor at `115200`.
3. If needed, send `f`, then press `EN` to restart.
4. After restart, send `e`.
5. Hold `PS + Create` until the light bar flashes quickly.
6. Wait for:

```text
Controller connected: slot=0, model=...
```

## Serial Commands

```text
f = forget stored Bluetooth pairing keys
e = enable new Bluetooth connections
s = stop the robot immediately
```

Runtime feedback example:

```text
LX= 120 LY=-300  Lref=  98 Lact=  85 Ltq=   65  Rref=  42 Ract=  38 Rtq=   20
```

## Control Mapping

Left stick:

- Up: forward
- Down: backward
- Left: turn left in place
- Right: turn right in place

Differential mixing:

```cpp
leftLogicalSpeed = forward + turn;
rightLogicalSpeed = forward - turn;
```

The right wheel command is inverted before sending to the servo because the right wheel is mounted in the opposite direction.

## Tuning

Start testing with the robot lifted off the ground.

Useful parameters:

```cpp
const int MAX_REF_SPEED = 120;
float Kp = 5.0;
const int MAX_TORQUE_CMD = 1000;
const float LEFT_SPEED_TRIM = 1.00;
const float RIGHT_SPEED_TRIM = 1.27;
```

- Increase `MAX_REF_SPEED` or `Kp` carefully if the robot is too slow.
- Decrease `Kp` if the robot oscillates or shakes.
- Adjust `LEFT_SPEED_TRIM` or `RIGHT_SPEED_TRIM` if the robot does not drive straight.
- Reduce `MAX_TORQUE_CMD` if the torque is too strong.

## Troubleshooting

### `No controller`

The sketch is running, but no gamepad is connected yet. Try:

```text
f
press EN
e
put controller in pairing mode
```

### `DOWNLOAD_BOOT` / `waiting for download`

The ESP32 entered bootloader mode because `BOOT` was held while resetting. Release `BOOT`, then press `EN` once.

### `Failed to inquiry name ... using a fake one`

Bluepad32 saw a Bluetooth device but could not read its name. Clear pairing keys with `f`, restart with `EN`, then pair again.

## Credits

This project is based on an original CFServo torque-control sketch named `eleTorquesegyo.ino`, written by another author, and modified for Bluetooth gamepad robot-car control.

Third-party projects and libraries:

- [Bluepad32](https://github.com/ricardoquesada/bluepad32)
- CFServo / SCServo Arduino library from the Waveshare / CFServo package

If you are the original author of `eleTorquesegyo.ino` and want attribution updated or the file removed, please open an issue.

## License Notice

No project-wide open-source license is declared yet because the original sketch and third-party library license terms should be checked first.

Please follow the licenses of Bluepad32, CFServo / SCServo, and any original source code this sketch is based on.
