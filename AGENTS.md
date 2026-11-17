# Nighthelm project instructions

- Keep the project a manual PushTo system. Do not add motor control or GoTo behavior.
- Do not commit application functionality until the research phase has been reviewed.
- Keep hardware access behind small interfaces so core logic can run on macOS with simulation or recorded data.
- Treat PiFinder only as a design and behavior reference. Do not use, copy, or adapt PiFinder source code in Nighthelm.
- For any other reference project, do not copy substantial code; check the project's and dependency's license before reuse.
- Treat GPS as optional and IMU fusion as a planned later capability.
- Use `uv` for Python environments, Ruff for linting and formatting, and pytest for tests.
- Use Conventional Commits (for example, `docs: document initial architecture`) and semantic branch prefixes (`docs/`, `feat/`, `fix/`).
- Prefer small modules, explicit data structures, and dependency injection over framework-heavy abstractions.
- Document unresolved decisions and assumptions. Do not invent hardware drivers or add dependencies without an immediate use.
