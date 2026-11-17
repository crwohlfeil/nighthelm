# Nighthelm initial investigation

**Status:** Phase 1 research and architecture planning; reviewed 2026-11-16.

## Executive summary

Nighthelm should be an independent, Python-first, headless PushTo tool. Its first complete software path should turn a known or recorded sky image into an absolute sky pointing and provide that position over an LX200-compatible TCP server. The observer moves the telescope manually; no motors, encoders, or GoTo controller are required. GPS is optional. IMU motion tracking is valuable for responsive updates but should follow a working camera solve and coordinate-serving path.

PiFinder is the closest mature reference for a self-contained Pi camera, plate solver, IMU estimator, GPS, and SkySafari system. Its architecture and engineering lessons are useful, but source reuse has meaningful constraints: the repository is GPL-3.0 and the Cedar detect binary has separately documented licensing terms. Nighthelm should initially reuse ideas and interfaces, not source. StepSolve is a particularly relevant headless/LX200 comparator, while PushNav shows that plate solving can be hosted on a general-purpose computer and that continuous camera solving is a plausible alternative to an IMU-first design.

For a first hardware prototype, prefer a Raspberry Pi 4 Model B with 2 GB or 4 GB RAM, a CSI camera with a fixed-focus wide-angle lens selected for the camera sensor, and a replaceable IMU breakout. Do not buy the BNO055 by default: Bosch says it is discontinued and not recommended for new designs. GPS, custom PCBs, batteries, and mechanical enclosure should wait until software and pointing requirements are better measured.

The first development milestone should run entirely on macOS: replay a timestamped fixture containing image-derived or simulated solved coordinates, convert to the shared pointing model, and serve that position over a localhost LX200 TCP endpoint. This isolates coordinate semantics and protocol behavior before hardware is introduced.

## Key architectural findings

- Absolute plate solves should be the source of truth. An IMU can propagate orientation between solves, but its drift and camera-to-telescope alignment require periodic camera correction.
- Optical camera direction and eyepiece/telescope direction may differ. Model this as a calibration transform rather than assuming the camera is coaxial with the eyepiece.
- Solve output, estimated current pointing, and telescope-protocol position should be separate concepts. Include timestamps, validity/freshness, and source/quality diagnostics.
- Keep sensor and camera acquisition replaceable. Fakes and replay inputs make platform-independent development practical.
- SkySafari integration should initially be outbound coordinate reporting using the LX200 dialect it expects. Whether to support target-setting commands can be resolved later; it is not necessary for manual PushTo MVP.
- Camera choice is mostly an optical and software support decision. Sensor pixel count alone does not determine astrometric usefulness; lens focal length, field of view, focus stability, exposure, star centroid quality, and solved index/catalog are central.

## Recommended implementation approach

1. Define coordinate, timestamp, status, and pointing-validity data structures independent of hardware.
2. Make a deterministic macOS replay/simulation loop and validate coordinate and protocol semantics with fixtures.
3. Implement a narrow LX200 TCP server and exercise it with a client or protocol fixture.
4. Add a camera adapter using Raspberry Pi's supported camera stack and collect raw or minimally processed star images.
5. Benchmark candidate plate solvers with representative night-sky frames on both Mac and Pi 4; compare solve success, latency, RAM, field coverage, and packaging burden.
6. Add IMU sampling and a separately testable orientation estimator only after mounting axes, camera-to-eyepiece alignment, and acceptable drift have been measured.

Astropy is a reasonable library for astronomy coordinates and time/location calculations. For solving, benchmark Tetra3/Cedar-Solve style lost-in-space solvers against Astrometry.net's local `solve-field` using constrained index files. Keep the solver behind a small adapter. Do not make the first architecture depend on a heavyweight UI framework or cloud service.

## Hardware recommendations

The practical first build is a Pi 4 Model B (2 GB minimum for a headless appliance; 4 GB provides comfortable development margin), official CSI-compatible camera or a supported third-party module, and a manually focused wide-angle lens. A camera module with an M12/CS lens mount gives optical flexibility; lock focus after nighttime calibration. Use microSD for initial work and a stable 5 V USB-C supply for bench testing. Power bank and enclosure selection should follow a measured load and thermal test.

Prototype with an IMU breakout selected after checking current stock, kernel/userspace support, and mounting calibration workflow. Treat BNO055 as legacy stock rather than a new design recommendation; evaluate BNO085/BNO086 and raw gyro/accelerometer alternatives on integration and noise, not advertised absolute-heading accuracy alone. Leave GPS optional; configure location and clock manually for first tests.

See [hardware notes](../hardware.md) for interfaces, mounting, and open trade-offs.

## Reference project assessment

Repository existence and current descriptions were checked on 2026-11-16:

- [PiFinder](https://github.com/brickbots/PiFinder) is a public, active, Pi-centric open-source hardware/software telescope finder, with GPS, camera solving, IMU tracking, self-contained UI, and Wi-Fi SkySafari support. Its documented solver path uses centroid detection and Tetra3-derived Cedar-Solve. See [PiFinder research](pifinder.md).
- [StepSolve](https://github.com/tomotvos/stepsolve) is a public Pi-based headless plate-solving service publishing coordinates over LX200 TCP, with optional OnStepX integration. Its README describes a compact C#/.NET implementation and runtime dependency on `rpicam-still`, with optional local Astrometry.net solver. It is a useful functional and deployment comparison, but not a reason to choose C# over the project's Python preference.
- [PushNav](https://github.com/meridianfield/pushnav) is a public cross-platform plate-solving PushTo project. Its README describes laptop-hosted live camera solving and reporting to multiple planetarium clients, including SkySafari, without encoders or motors. It demonstrates a contrasting solve-continuously strategy; detailed timing/robustness claims require hands-on evaluation.

## Unresolved questions

- What pointing accuracy and update rate make SkySafari feel useful at the eyepiece?
- What camera/lens and image-processing pipeline gives reliable stars across light pollution and common sky conditions?
- Can the selected solver meet latency and offline operation targets on Pi 4 with local indexes?
- Does IMU dead reckoning materially improve perceived responsiveness once camera field of view and solve cadence are known?
- What is the desired SkySafari LX200 behavior and exact network topology: Pi access point or join an existing Wi-Fi network?
- How should users calibrate and repeatably mount the camera relative to the telescope optical axis?
- Which IMU is available and supportable long-term, and is magnetometer heading useful or a source of local magnetic error?
- Which battery can provide a full observing session safely while powering the Pi and camera?
- Is Apache-2.0 acceptable for the project owner? This report recommends it for a permissive license with an explicit patent grant; it does not change the obligations of third-party dependencies.

## Proposed next steps

1. Review this research set and confirm project licensing preference.
2. Build the Mac-only replay, coordinate-model, and LX200 protocol milestone.
3. Test the protocol against the intended SkySafari version and verify connection settings.
4. Select camera/lens and solver using actual image samples; defer nonessential purchases.
5. Measure Pi 4 solve performance before choosing the final solver packaging strategy.
6. Add hardware and IMU in stages, recording calibration procedure and observing results.

Detailed material: [PiFinder review](pifinder.md), [alternatives](alternatives.md), [architecture](../architecture.md), [hardware](../hardware.md), and [implementation plan](../implementation-plan.md).
