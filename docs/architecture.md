# Proposed architecture

Nighthelm is a headless service with a small number of replaceable components. The first implementation should use ordinary Python modules and protocols/interfaces; avoid a general plugin framework or distributed process graph.

## Data path

```text
camera source ──> image frame ──> plate-solver adapter ──> absolute camera pointing
                                         │                         │
replay/simulation ───────────────────────┘                         v
                                                       pointing estimator
                                                        (IMU later)
                                                                 │
location/time config or optional GPS ─────────────────────────────┤
                                                                 v
                                                       telescope-axis pointing
                                                                 │
                                                    LX200 TCP server ──> SkySafari
```

Every boundary should carry timestamps and units explicitly. A result should include validity, source (`replay`, `camera`, `IMU-propagated`), freshness, and solver quality diagnostics. The protocol layer must not mistake a stale estimate for a current one.

## Proposed module responsibilities

| Concern | Responsibility | Mac development path |
|---|---|---|
| Hardware interfaces and sensor acquisition | Narrow camera, IMU, and optional GPS interfaces; concrete Raspberry Pi adapters only when hardware is selected | Fake providers and timestamped fixture/replay providers |
| Image capture and plate solving | Capture configuration, image metadata, preprocessing, solver adapter, structured solve result | Recorded image frames; separate solve backends behind an interface |
| Orientation estimation and sensor fusion | Combine absolute camera fixes with movement since exposure; own drift/confidence/freshness policy | Deterministic quaternion/rotation fixtures; estimator independent of device I/O |
| Astronomical coordinate transformations | Convert between solver coordinates, chosen sky frame, site/time-aware coordinates, and protocol representation | Astropy with explicit frame/time/site; golden reference fixtures |
| Telescope positioning and calibration | Camera-axis to eyepiece/telescope-axis transform, user calibration and persistence | Simulated known transform and calibration fixture |
| LX200 protocol integration | TCP lifecycle, command parsing, coordinate serialization, error/status semantics | Localhost socket client and protocol fixtures |
| Configuration, diagnostics, and logging | Typed configuration, safe defaults, connection state, solve timing/failures, structured logs | File/CLI configuration and captured logs |

These are responsibility boundaries, not a mandate for one file per row. Begin with a compact package and split as evidence justifies it.

## Data model concepts

- `Site`: latitude, longitude, optional elevation, source, and validity.
- `ObservationTime`: UTC timestamp and source; GPS time is optional.
- `ImageFrame`: pixels or path/reference, exposure start/end, camera metadata, and associated IMU sample if available.
- `Pointing`: unit-defined coordinates plus frame/epoch, roll where required, timestamp, validity, source, and quality.
- `Alignment`: transform between camera optical axis and telescope/eyepiece axis, with calibration metadata.
- `PointingState`: most recent absolute solve, current estimate, age since solve, and status.

Keep RA/Dec, Alt/Az, and raw orientation distinctions explicit. Use quaternions or rotation matrices consistently internally for 3D rotations; avoid Euler-angle interpolation. Confirm coordinate-frame conventions against the chosen solver and LX200 client before freezing serialized fields.

## Library choices to evaluate

- **Astropy**: coordinate frames, time, locations, units, and transformations. Use it instead of writing custom celestial transforms. Keep its use at defined boundaries because it is a substantial dependency.
- **Tetra3**: compact lost-in-space solver candidate with Python-facing APIs and configurable database. Verify supported Python versions, license, and Pi 4 performance.
- **Astrometry.net**: mature local `solve-field` alternative. Evaluate indexes and installation footprint as well as accuracy and latency.
- **Picamera2/libcamera**: supported Raspberry Pi camera software stack. Keep it out of the Mac replay path and isolate it in a camera adapter.
- **pyserial / GPS packages**: defer until a specific GPS module and device path are selected. Linux serial access may be sufficient; don't add a GPS framework for configuration-only MVP.
- **Ruff, pytest, uv**: planned development tools, not runtime dependencies.

Do not introduce NumPy, SciPy, or an IMU abstraction library until selected solver/math APIs demonstrate the need. Likely transitively required numerical packages may be accepted as dependencies when their exact roles and platform availability are known.

## Mac development and diagnostics

The first runnable service should accept a `PointingSource` supplied by dependency injection. A replay provider should step through timestamped JSON/CSV and image fixtures at controllable speed. A local TCP server can publish the current replay pointing to a client. This can validate frame semantics, RA/Dec formatting, socket lifecycle, stale-data behavior, and error recovery with no Pi or telescope.

For image path development, keep a small set of representative sky-image fixtures with provenance and license documented. Allow solver adapters to run locally; production solving must not require a network. Add diagnostic output for image dimensions/exposure, centroid count, solve duration, match count/residual where available, time since last solve, last IMU sample, and published pointing age.

## Reliability and open choices

- Publish a coordinate only when valid. Decide a stale cutoff and client-visible error behavior after SkySafari tests.
- The initial service should bind only to an explicitly configured interface and document its Wi-Fi security expectations.
- Start one process or a small async/threaded service. Separate processes and shared-memory frames only if capture and solving contention is measured.
- Define graceful shutdown, reconnect behavior, and diagnostic logs before adding a user-facing dashboard.
- Preserve manual movement as a product invariant. No command may move the telescope.

Protocol references for implementation include the [Meade LX200 command set](https://www.astro.louisville.edu/software/xmtel/archive/xmtel-indi-6.0/xmtel-6.0l/support/lx200/CommandSet.html) and [SkySafari/SkyFi setup guidance](https://support.simulationcurriculum.com/hc/en-us/articles/231541087-SkyFi-Quick-Start-Guide-and-Video-Demonstration). These establish useful starting points; actual SkySafari behavior should be checked against the intended app version and setup.
