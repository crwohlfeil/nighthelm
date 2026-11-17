# Preliminary hardware notes

This is a research-stage bill of materials, not a selected purchase list. Prices and stock vary; obtain current local quotes before ordering. Choose parts based on supported drivers, focus/mounting mechanics, field of view, power draw, and repeatable availability.

## Minimal staged bill of materials

| Item | Prototype recommendation | Notes |
|---|---|---|
| Compute | Raspberry Pi 4 Model B, 2 GB as a minimum candidate; 4 GB preferred for benchmark margin | Pi 4 is the stated target and includes CSI, Wi-Fi, 40-pin GPIO, I²C and UART. PiFinder's own BOM uses 2 GB and says more memory is fine. Measure actual solver/index and image-buffer footprint before finalizing. |
| Camera | CSI module verified against current Raspberry Pi camera software; start with Pi Camera Module 3 or HQ Camera | Camera Module 3 provides standard or wide fixed module lens choices; HQ camera supports external CS/M12 lenses. A global-shutter camera is unnecessary for stationary exposure but may help if tracking motion blur is observed. |
| Lens | Fixed-focus wide-angle lens with a mechanically lockable focus ring, appropriate to sensor size and mount | Exact focal length/field is unresolved. Too wide reduces pixels per star; too narrow reduces star count and acquisition tolerance. Evaluate field of view, distortion, aperture, aberration, and focus at infinity with solver trials. |
| IMU | Delay purchase until camera-only MVP; then compare BNO085/BNO086 breakout or raw gyro+accelerometer boards with maintained Linux support | BNO055 is a useful reference device but Bosch states it is discontinued and not recommended for new designs. Orientation fusion, mounting axes, vibration, and magnetic interference matter. |
| GPS | Optional USB GPS dongle or UART GNSS module with documented Linux/NMEA support | Manual site/time configuration is adequate for the first milestone. If UART is used, reserve GPIO UART and configure serial console; if USB, it consumes a port and may be mechanically less convenient. |
| Storage | 16–32 GB high-endurance microSD for first bench prototype | Solver indexes could affect storage needs. Prefer reliable storage and clean shutdown behavior before adding large catalogs. |
| Power | Bench supply first; later USB-C power bank with verified 5 V output/current and capacity | Official Pi 4 supply target is 5 V/3 A. Camera, peripherals, Wi-Fi, conversion losses, and cold-weather capacity change runtime; measure at load before selecting capacity. |
| Interconnect | Camera ribbon cable compatible with Pi 4 CSI connector; short I²C leads; optional USB serial | Pi 4 uses a standard Pi camera connector; verify cable width/orientation and the camera's adapter requirements. Keep I²C short and mechanically strain-relieved. |
| Mounting | Temporary rigid bracket on a finder shoe or dovetail adapter; later printed enclosure | Hold camera boresight stable relative to tube. Provide focus lock, lens cap, cable strain relief, airflow, and repeatable removal/reinstallation. Avoid blocking the Dobsonian altitude bearings or balance. |

## Pi 4 model and RAM

The Pi 4 Model B product brief now lists 1, 2, 3, 4, and 8 GB variants. Nighthelm's workload is still unknown, so RAM recommendation is preliminary. PiFinder documents a 2 GB BOM and uses substantial parallel camera/UI/sensor work; Nighthelm omits UI but plate-solver indexes can change memory needs. Start at 2 GB if cost and availability favor it; choose 4 GB if the premium is modest or the same unit will be used for development. Do not pay for 8 GB without a benchmark showing a benefit.

## Camera and optical notes

Raspberry Pi documents Camera Module 2, Camera Module 3 (standard and wide variants), High Quality Camera with CS/M12 mount, AI Camera, and Global Shutter Camera as official modules compatible with Pi computers that have CSI connectors. Camera software support should be checked for the selected OS/image: use the supported libcamera/rpicam stack and Picamera2 for Python access rather than building on the legacy camera stack.

Sky acquisition wants a wide field and enough star detections. The target telescope's 1200 mm focal length describes the main optical tube; the finder camera's lens determines the solver field of view. The camera needs a fixed focus near infinity and stable mounting. Lens suitability cannot be concluded from nominal focal length alone: sensor size, focus, edge quality, aperture, and solver database all matter. Compare the official Camera Module 3 wide module against a CSI board with interchangeable lens before purchase.

## BNO055 and alternatives

The BNO055 combines accelerometer, gyro, magnetometer, and on-chip fusion and appears in older PiFinder builds, making it a valuable software/reference target. However, Bosch Sensortec's support forum says the BNO055 has been discontinued and is not recommended for new designs. Treat breakout availability as uncertain and avoid hard-coding its Euler conventions or API.

Evaluate a BNO085/BNO086 board if a fused quaternion and available Linux interface are valuable, or a maintained raw IMU breakout if Nighthelm will own fusion. A magnetometer can be disturbed by telescope hardware, batteries, wiring, and local steel; gyro/accelerometer-only relative rotation may be more useful for short intervals, with periodic optical correction. Selection criteria: current stock, sample rates, timestamping, calibration retention, orientation-frame documentation, Python/library maintenance, Raspberry Pi 4 I²C behavior, and repeatability after power cycling.

## Interfaces and electrical planning

- **CSI:** camera ribbon attaches directly to the Pi 4 camera serial interface. Confirm connector style (15-pin Pi 4 versus 22-pin smaller Pi variants) and cable routing.
- **GPIO/I²C:** Pi 4 40-pin header exposes 3.3 V logic I²C, commonly SDA/SCL on GPIO2/GPIO3 (pins 3/5), plus 3.3 V and ground. Verify breakout voltage compatibility and pull-ups; never apply 5 V logic to GPIO.
- **UART:** GPIO UART pins are available for a GNSS receiver, but Linux console configuration and serial device naming must be handled. USB GPS is easier for early experiments.
- **Wi-Fi:** Pi 4 has dual-band 2.4/5 GHz Wi-Fi. The first prototype can join an existing local network; access point mode can be tested later if direct phone connectivity is preferred.
- **Power:** use a supply/power bank able to maintain Pi's required input under peak load. Record voltage/current with camera capture, solving, and Wi-Fi active; choose battery capacity from measured watts × desired hours with conversion and cold-weather margin.

## Mounting and test order

1. Bench-run Pi, camera, and solver on a stable tripod or sky-facing mount before choosing enclosure geometry.
2. Measure camera field, solve cadence, temperature, current, and focal stability.
3. Add a rigid removable bracket; repeat boresight calibration after reinstall.
4. Add IMU at a documented orientation relative to the camera and tube; repeat tests under altitude/azimuth rotations.
5. Add a battery after measuring average and peak load. Validate a full session and low-battery behavior.
6. Design the printed enclosure and finder-shoe adapter around selected boards, cable bend radii, heat dissipation, dew protection, and balance.

## Sources

- [Raspberry Pi 4 Model B specifications](https://www.raspberrypi.com/products/raspberry-pi-4-model-b/specifications/)
- [Raspberry Pi camera hardware](https://www.raspberrypi.com/documentation/accessories/camera.html)
- [Raspberry Pi camera software](https://www.raspberrypi.com/documentation/accessories/camera-software.html)
- [PiFinder parts list](https://pifinder.readthedocs.io/en/release/BOM.html)
- [Bosch BNO055 alternatives/replacement discussion](https://community.bosch-sensortec.com/mems-sensors-forum-jrmujtaw/post/bno055-alternatives-and-replacement-KNK3ARUUqZnW8oZ)
- [BNO055 datasheet](https://www.bosch-sensortec.com/media/boschsensortec/downloads/datasheets/bst-bno055-ds000.pdf)
