# Reference project comparison

Checked on 2026-11-16. The GitHub repository URLs below resolve to public repositories. Feature and implementation summaries reflect their repository READMEs and linked documentation; they are not independent performance evaluations.

| Project | Verified focus | Fit for Nighthelm | Main lessons / limits |
|---|---|---|---|
| [PiFinder](https://github.com/brickbots/PiFinder) | Raspberry Pi-based self-contained telescope finder with camera solving, IMU motion tracking, GPS, UI, and SkySafari integration | Closest full-system reference | Mature subsystem and calibration concepts. Broader than the headless MVP, and licensing differs among bundled components. Details: [review](pifinder.md). |
| [StepSolve](https://github.com/tomotvos/stepsolve) | Headless Pi plate-solving service, LX200 TCP for SkySafari, optional OnStepX integration | Very close to the coordinate-publishing scope | Useful reference for boot-to-service packaging and protocol focus. README describes .NET/C# and rpicam-still capture with optional `solve-field`; investigate exact source/dependency licenses and test behavior before borrowing ideas beyond published interfaces. |
| [PushNav](https://github.com/meridianfield/pushnav) | Cross-platform laptop-based live camera plate-solving for manual telescopes, with SkySafari and other clients | Relevant alternative deployment/solve cadence | Challenges the assumption that processing must live on a Pi. Described USB camera + lens approach suggests lower hardware cost if a phone/laptop were acceptable, but Nighthelm's intended self-contained Pi remains a different product target. |

## Architectural trade-offs surfaced

### Continuous solving versus IMU interpolation

PushNav presents continuous camera solving; PiFinder combines camera solves and IMU dead reckoning; StepSolve focuses on repeated plate solves and LX200 position publishing. Nighthelm should avoid deciding between these on paper. First benchmark solve latency and update rate with the selected camera and a range of real sky conditions. If solves are frequent enough, an IMU may be unnecessary for MVP. If responsiveness drops between solves, add a replaceable estimator and compare its drift to the benefit.

### Pi appliance versus host computer

PushNav demonstrates that a laptop can handle camera solving without telescope encoders. Nighthelm targets a self-contained, battery-powered Pi attached to the telescope, so host-computer architecture is a useful development and performance comparison rather than a change in product goal.

### Local solver options

- [Tetra3](https://github.com/esa/tetra3) targets star-tracker lost-in-space solving and provides a Python API plus databases. PiFinder documents using its lineage. It is a candidate for compact, constrained-field operation.
- [Astrometry.net](https://astrometry.net/doc/readme.html) supports local `solve-field` with local index files and constraints such as image scale. Its breadth is useful; index size and Pi 4 latency should be measured.
- PiFinder's Cedar-Solve and Cedar-Detect implementation is informative, but separate license terms for its components and binaries make direct reuse or bundling a distinct decision.

No solver is selected. Build a small interface around the selected backend so fixture replay, benchmarks, and the protocol server do not need to know solver internals.

## Recommendation

Borrow ideas from all three projects, not their code. Start with the StepSolve-style narrow output boundary, PiFinder-style absolute solve plus optional estimator separation, and PushNav's reminder to measure actual solve cadence. Keep Mac replay as an explicit supported development mode. Revisit source-level reuse only for a clearly scoped component after its license, API, and maintenance costs are understood.
