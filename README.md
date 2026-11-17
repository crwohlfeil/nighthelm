# Nighthelm

Nighthelm is an open-source, headless PushTo navigation system for manually aimed amateur telescopes. A camera looks at the sky, plate solving determines where the telescope points, and a telescope-facing network interface shares coordinates with planetarium software such as SkySafari.

The first hardware target is a Sky-Watcher Classic 200P Dobsonian (203 mm aperture, 1200 mm focal length, f/5.9). The software is intended to remain independent of any particular telescope or mount. Nighthelm is strictly PushTo: the observer moves the telescope by hand; Nighthelm does not drive motors or perform GoTo movement.

## Intended functionality

- Determine pointing from camera images and plate solving.
- Add responsive motion estimates between camera solutions using an IMU, after the camera-only path is established.
- Serve telescope coordinates over an LX200-compatible TCP interface for SkySafari and compatible clients.
- Run headlessly on a Raspberry Pi, with a development and simulation path on macOS.
- Treat GPS as optional; users can configure location and time when GPS is absent.

The intended hardware includes a Raspberry Pi 4, a camera with a wide-field lens, an IMU, optional GPS, Wi-Fi, and battery power. Enclosure and finder-shoe mounting are later mechanical work.

## Development status

**Research and architecture planning.** This repository does not yet contain application functionality or selected hardware. The first development milestone is designed to run on a Mac using recorded or simulated inputs. See the [investigation summary](docs/research/initial-investigation.md), [architecture](docs/architecture.md), [hardware notes](docs/hardware.md), and [implementation plan](docs/implementation-plan.md).

## Relationship to existing projects

Nighthelm learns from the documented design of [PiFinder](https://github.com/brickbots/PiFinder), and compares it with [StepSolve](https://github.com/tomotvos/stepsolve) and [PushNav](https://github.com/meridianfield/pushnav). The current plan is an independent implementation, not a permanent fork. No third-party source code or binaries have been copied into this repository. Licensing and reuse considerations are recorded in the [reference project review](docs/research/pifinder.md).

## Development setup

Python is the planned implementation language. Use [uv](https://docs.astral.sh/uv/), Ruff, and pytest once the first executable package milestone begins. The `pyproject.toml` establishes only these basic development tools; application dependencies will be chosen alongside the capability that needs them.

```sh
uv sync --all-groups
uv run ruff check .
uv run ruff format --check .
uv run pytest
```

There are no application tests yet. The commands are the intended baseline for the first implementation phase.

## License

Nighthelm's original work is licensed under MIT. The project intends to implement its own code and use upstream libraries through their published interfaces. Review dependency licenses before adoption. PiFinder is a design and behavior reference only; Nighthelm does not use, copy, or adapt PiFinder source code. See the [PiFinder licensing analysis](docs/research/pifinder.md).
