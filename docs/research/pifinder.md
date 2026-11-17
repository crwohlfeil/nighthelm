# PiFinder reference review

**Repository reviewed:** [brickbots/PiFinder](https://github.com/brickbots/PiFinder). **Documentation reviewed:** PiFinder developer architecture, contributor guide, positioning notes, build guide, parts list, and license notes, checked 2026-11-16.

## Relevance and scope

PiFinder is the closest reference to Nighthelm's intended end-to-end function: a Raspberry Pi mounted to a manual telescope, camera-based plate-solving, IMU motion estimates, GPS location/time, and SkySafari over Wi-Fi. The public repository presents itself as an open-source hardware and software project. It is broader than Nighthelm's initial scope: it includes a built-in display, keypad, catalogs, charts, object selection, and a web UI. Nighthelm can use it to understand failure modes and subsystem boundaries without reproducing its full product.

## Verified architecture

The developer architecture describes separately launched units/processes for camera, IMU, GPS, plate solver, position integrator, user interface, web server, and SkySafari position server. They share state and use queues for events/messages. The newer positioning documentation explicitly separates the solver's per-attempt result from the integrator-owned long-lived pointing estimate.

This is a useful distinction for Nighthelm. A solver should report an optical-axis absolute solution plus metadata; a position estimator can combine that measurement with movement data; the protocol layer should publish only a valid, appropriately fresh telescope-axis position. Nighthelm can keep these as ordinary Python objects and small interfaces initially, without importing PiFinder's multiprocessing setup or UI.

## Camera capture and image processing

PiFinder's documented architecture uses a camera process that captures frames into shared image state. The solver consumes the newest frame, extracts star centroids, and invokes Tetra3-derived Cedar-Solve. `cedar-detect-server` performs centroid detection in Rust as a separate process and exposes a gRPC interface; when unavailable, PiFinder documents fallback to Tetra3's own centroid extraction. The solver is isolated from IMU reads: the image metadata includes the IMU quaternion sampled at exposure end, which anchors the solve for later integration.

Takeaway: isolate image capture, centroid extraction, and solve behind clear boundaries. Avoid coupling the telescope protocol to image processing. Nighthelm need not adopt PiFinder's shared memory, multiprocessing manager, gRPC server, or Rust helper until measurements show that simpler Python paths miss latency targets.

## Plate-solving options

PiFinder documents a reduced star database tuned to compatible camera/lens fields of view to reduce solve time. It uses Tetra3 lineage with Cedar-Solve and optional accelerated centroiding. Tetra3 is an embedded/lost-in-space star-tracker solver and is worth evaluating for a constrained instrument. Astrometry.net's `solve-field` is a mature local solver with configurable index files and image scale constraints, but its index footprint and solve latency on the Pi need measurement. StepSolve identifies it as an optional local solver; that is a project claim, not a comparative benchmark.

The recommended action is a solver adapter and a benchmark over representative recorded frames, including difficult frames. Record success rate, solve-time distribution, memory, index storage, image preparation, and installation effort. Do not rely on an internet solver in an observing field.

## IMU integration and alignment

PiFinder's positioning notes describe camera and aligned pointing separately, with RA/Dec/roll, a quaternion IMU anchor captured at exposure, and an inertial dead-reckoner that advances the last camera solve. They also document a per-build camera-to-IMU rotation constant and a static camera-to-aligned-axis rotation. This demonstrates that an IMU's local sensor axes, mounting orientation, image rotation, and camera-to-eyepiece offset all need calibration.

This model is worth adapting conceptually. The calibration math and estimator should be implemented and tested independently from any UI and sensor API. PiFinder's specific quaternion conventions, mounting constants, or code are not assumed to fit Nighthelm's camera mount or Dobsonian geometry.

## GPS, telescope alignment, coordinates

PiFinder has a GPS subsystem that publishes location and time into shared state. Its integrator derives topocentric altitude/azimuth from pointing, site location, and time. Camera solve is therefore the absolute sky anchor, and GPS contributes context for coordinate conversion and time rather than a substitute for pointing.

Nighthelm should represent site/time as configuration inputs first. GPS is an optional provider of those values. Use Astropy for standard celestial coordinate/time/location transformations instead of hand-rolling astronomy transformations. Be explicit about coordinate frame, epoch, units, timestamp, and apparent versus catalog coordinate expectations at the LX200 boundary.

## SkySafari and LX200 integration

PiFinder documents a position server for SkySafari, including Wi-Fi access point operation and published position. Its project has an internal server and more functionality than Nighthelm requires. StepSolve is a more tightly scoped comparison because its README specifically describes LX200 TCP publishing for SkySafari. Nighthelm should independently implement only the required LX200-compatible command subset, initially returning current coordinates and status; protocol details and SkySafari client behavior need explicit integration tests.

## Headless operation, simulation, and Pi 4 performance

PiFinder has fake/none hardware-specific implementations for non-Pi development and documents running without hardware. Its contributor guide notes that a non-Pi development environment can run code while lacking IMU, GPS, and other physical features. This is direct precedent for replay and fake adapters.

PiFinder also splits work into processes and uses a specialized centroid process. This indicates that the full product balances camera, UI, sensor, and solving workloads, but does not prove that Nighthelm needs the same complexity. Since Nighthelm is headless and has a narrow published output, benchmark on Pi 4 before selecting process count or native acceleration. Optimize measured bottlenecks only. A 2 GB board is a plausible initial target, but RAM and thermal margins must be measured with actual image buffers and solver indexes.

## What Nighthelm can learn or reuse

| Area | Recommendation | Reason |
|---|---|---|
| State boundaries and solve/integrator split | Adapt as an architectural idea | Helps separate absolute correction from short-term prediction |
| Camera/IMU alignment model | Adapt conceptually, derive Nighthelm-specific calibration | Mounting, image rotation, and optical-axis offsets are system-specific |
| Hardware fakes and replay | Adapt | Supports macOS work and deterministic diagnostics |
| Tetra3/Cedar-Solve or Astrometry.net | Evaluate as dependencies/tools | Both avoid re-inventing star matching; package/license/performance suitability remain to be checked |
| PiFinder UI, catalog, charting, and full process graph | Do not port for MVP | Outside Nighthelm's headless positioning scope |
| PiFinder IMU and camera driver code | Do not copy initially | Avoid tight hardware coupling and license obligations; implement adapters after choosing actual hardware |
| Cedar detect binary | Do not redistribute or depend on it without licensing review | It is not covered by the root repo license alone |

## Licensing and reuse implications

The repository's top-level `LICENSE` identifies GPL-3.0. Its `bin/README.md` says Cedar-Solve is Apache-2.0, while Cedar Detect is distributed under Functional Source License FSL-1.1-MIT, with the PiFinder binaries distributed under a separate explicit license from the copyright holder because PiFinder is also sold commercially. These terms must not be collapsed into the assumption that every file or binary is GPL-3.0 or permissively reusable.

Nighthelm's initial plan is to use no PiFinder source code. If substantial GPL-3.0 code is later copied or adapted into a combined/distributed program, source distribution and downstream licensing obligations may apply; obtain legal review for the intended distribution and compatibility. FSL restrictions are distinct and may be incompatible with the project's open-source distribution goals for a bundled component. Apache-2.0 is selected for Nighthelm's original work because it is permissive and includes an express patent license; this does not relicense upstream code or waive third-party license obligations. Dependency license notices and attribution will be handled when dependencies are selected.

## Sources

- [PiFinder repository](https://github.com/brickbots/PiFinder)
- [Developer architecture](https://pifinder.readthedocs.io/en/main/dev_arch.html)
- [Positioning model and solver/integrator description](https://github.com/brickbots/PiFinder/blob/release/docs/ax/positioning.md)
- [Contributor guide](https://pifinder.readthedocs.io/en/v3_hardware/dev_guide.html)
- [Parts list](https://pifinder.readthedocs.io/en/release/BOM.html)
- [PiFinder binary licensing notes](https://github.com/brickbots/PiFinder/blob/release/bin/README.md)
- [Tetra3](https://github.com/esa/tetra3)
- [Astrometry.net documentation](https://astrometry.net/doc/readme.html)
