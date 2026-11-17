# Phased implementation plan

The sequence prioritizes independently testable capabilities. Acceptance thresholds that depend on sky conditions or telescope feel must be measured and written down before claiming field readiness.

## Milestone 0: Mac replay and coordinate service

**Deliverables**

- Minimal Python package and typed configuration.
- Deterministic replay/simulated source for timestamped solved pointings.
- Explicit coordinate units/frame representation and telescope alignment placeholder/identity transform.
- Localhost LX200 TCP server that publishes the current pointing and reports invalid/stale state safely.
- Structured diagnostics for source time, pointing age, and protocol activity.

**Dependencies:** uv, Ruff, pytest; standard library sockets/config/logging; Astropy only once transformations are required.

**Acceptance criteria**

- On macOS, replay a checked-in or generated fixture from start through stop with no Pi hardware.
- A protocol client can query the published coordinates and receives correctly formatted values for documented test cases.
- Tests exercise RA wrap, declination signs/limits, stale data, disconnected/reconnected clients, malformed requests, and clean shutdown.
- The service never issues a motor movement command; no movement-control code exists.

**Risks/questions:** Exact LX200 command subset and SkySafari expectations; frame and epoch semantics; validity timeout; whether TCP clients expect a particular handshake.

## Milestone 1: SkySafari protocol validation

**Deliverables:** Client compatibility notes and configuration for host/AP network mode, coordinate formatting behavior, status/error behavior, and connection lifecycle.

**Dependencies:** Milestone 0; a phone/tablet with SkySafari, or a reliable protocol emulator.

**Acceptance criteria:** SkySafari connects over local Wi-Fi and displays the simulated/replayed position; repeated start/stop and network loss recover predictably; manual telescope movement remains user-driven.

**Risks/questions:** SkySafari version and platform differences; choice of Pi access point versus joining existing Wi-Fi; local-network firewall/isolation settings.

## Milestone 2: Camera capture and image characterization

**Deliverables:** Raspberry Pi camera adapter using supported camera software; image acquisition diagnostics; captured sky-image dataset with metadata and provenance.

**Dependencies:** Pi 4, supported CSI camera, lens, stable bench power, focusable rigid mount.

**Acceptance criteria:** Repeatable frames at documented resolution/exposure/focus; no frame corruption under sustained capture; field of view and star counts characterized on multiple sky regions; thermal/current usage recorded.

**Risks/questions:** Wide-angle lens aberration, focus drift, exposure/sensitivity under light pollution, compatible sensor drivers for non-official camera.

## Milestone 3: Offline plate solver evaluation

**Deliverables:** Solver adapter and benchmark runner; compare at least a compact star-tracker solver (Tetra3 lineage) and Astrometry.net local solver on the same fixture set; selected indexes/data approach.

**Dependencies:** Milestone 2 images; local solver packages/databases; macOS and Pi 4.

**Acceptance criteria:** Benchmark reports success rate, solve-time percentiles, resource use, and failure diagnostics on Mac and Pi 4. Solves work without network access. Chosen backend reaches a documented acquisition/update target on representative images.

**Risks/questions:** Plate-solving latency, index footprint, difficult fields/conditions, image distortion, dependency licensing and ARM packaging.

## Milestone 4: Absolute camera pointing into service

**Deliverables:** Production solve source feeding pointing state and LX200 service; stale-solve policy and diagnostics; repeatable initial camera/telescope alignment workflow.

**Dependencies:** Milestones 1–3; telescope mounting bracket; observing site/time configuration.

**Acceptance criteria:** On the target Dobsonian, Nighthelm publishes stable coordinates consistent with independent reference pointings. After shifting the telescope, a fresh camera solve updates position; client behavior is reliable throughout a session.

**Risks/questions:** Camera-to-eyepiece calibration, mirror/tube flex, target coordinate reference, plate-solve quality requirements, weather and sky access.

## Milestone 5: IMU motion estimation

**Deliverables:** Selected IMU interface, timestamped sample acquisition, calibration/mounting procedure, independent rotation estimator, fusion with absolute solve anchors.

**Dependencies:** Milestone 4; candidate sensor evaluation; rigid known camera-to-IMU mount; rotation fixtures.

**Acceptance criteria:** Estimator predicts controlled movements between camera solves within an agreed angular error and time horizon; corrects to absolute solution when a solve arrives; degrades clearly when IMU data is invalid or stale.

**Risks/questions:** Gyro drift, compass interference, axis conventions, sensor timestamp jitter, motion/vibration during manual slews, benefit relative to continuous solves.

## Milestone 6: Field reliability and enclosure

**Deliverables:** Battery selection, stable Wi-Fi behavior, robust startup/recovery, user diagnostics, enclosure and finder-shoe mount, hardware assembly guide.

**Dependencies:** Earlier milestones with selected parts and measured load/thermal/alignment data.

**Acceptance criteria:** A complete observing session operates on battery; camera calibration survives mounting/removal within tolerance; focus and cabling remain stable; service reconnects after expected network and solve failures.

**Risks/questions:** Cold battery capacity, dew/condensation, thermal throttling, balance, ingress protection, power conversion noise, mechanical repeatability.

## Process and repository conventions

- Use semantic branches such as `docs/`, `feat/`, `fix/`; use Conventional Commit messages.
- Run Ruff lint and formatting checks and pytest in CI once the first executable capability and tests exist. Add the GitHub Actions workflow with Milestone 0 rather than creating a green-but-empty application test workflow during research.
- Keep test fixtures synthetic or openly licensed, and record image provenance. Do not commit third-party repository copies, generated indexes, or binaries without a clear need and license review.
- Milestones can be reviewed independently; defer later stages when an earlier measurement changes the design.
