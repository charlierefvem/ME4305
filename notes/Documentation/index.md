---
title: ME4305 MicroPython Firmware and API Documentation
type: reference
tags:
  - micropython
  - firmware
  - api
  - documentation
source:
  course: ME4305
  repository: https://github.com/charlierefvem/micropython
  commit: fd90403415b3f8962d778ab96c0ff5427c5e1773
  workflow: .github/workflows/build_firmware.yml
  run: https://github.com/charlierefvem/micropython/actions/runs/36178780466
status: draft
---

The ME4305 firmware for the NUCLEO-L476RG combines the MicroPython interpreter, the third-party `ulab` numerical module, and custom frozen modules used in the course. A frozen module is stored in the firmware image, so it can be imported without first copying its `.py` file to the board.

> [!note] Firmware download
> [Download the `custom-nucleo-l476rg-firmware` artifact from the successful build matched to these pages.](https://github.com/charlierefvem/micropython/actions/runs/36178780466#artifacts) GitHub Actions artifacts are retained for a limited time. If that artifact is no longer available, check the [Build Custom Mechatronics Firmware run history](https://github.com/charlierefvem/micropython/actions/workflows/build_firmware.yml).

## Choose the right documentation

Use the documentation source that matches the code you are trying to use:

1. **ME4305 course notes:** The custom frozen modules intended for student use are documented in the API pages below.
2. **`ulab` documentation:** The firmware currently includes the third-party [`ulab` numerical module](https://micropython-ulab.readthedocs.io/en/latest/), which is documented by its upstream project.
3. **MicroPython documentation:** The [official MicroPython documentation](https://docs.micropython.org/en/latest/) describes the interpreter and standard built-in modules. A feature in the general documentation may still depend on the port and firmware configuration.

## Custom frozen modules

> [!table]
> Student-facing API pages for the custom modules in the pinned source snapshot.
>
> | Module | Purpose |
> | --- | --- |
> | [`cotask`](cotask.md) | Run generator-based tasks with cooperative scheduling and optional timing profiles. |
> | [`fsm`](fsm.md) | Provide a small base class for finite-state-machine tasks. |
> | [`runningstats`](runningstats.md) | Accumulate a mean, sample variance, standard deviation, and maximum without storing every sample. |
> | [`taskgarbage`](taskgarbage.md) | Run garbage collection as a cooperative task. |
> | [`tracebuffer`](tracebuffer.md) | Store compact task and state events in a fixed-size trace buffer. |

## Reproducible firmware snapshot

These pages describe custom source commit [`fd90403415b3f8962d778ab96c0ff5427c5e1773`](https://github.com/charlierefvem/micropython/commit/fd90403415b3f8962d778ab96c0ff5427c5e1773). The matching [Build Custom Mechatronics Firmware run](https://github.com/charlierefvem/micropython/actions/runs/36178780466) completed successfully and produced the `custom-nucleo-l476rg-firmware` artifact.

That build resolved the `master` branches of the two upstream projects at build time. These are development revisions selected by the build, not claims about the latest tagged releases:

- MicroPython: [`09f5bb447504a058376c62fe991b3613531837e6`](https://github.com/micropython/micropython/commit/09f5bb447504a058376c62fe991b3613531837e6)
- `ulab`: [`b65c9dbb6c055888bb6fb9b386888a456a60dc88`](https://github.com/v923z/micropython-ulab/commit/b65c9dbb6c055888bb6fb9b386888a456a60dc88)

> [!note]
> The workflow also generates and freezes `registerinterface.py` during the build. The pinned commit does not contain a durable, browsable copy of that generated module, so its student-facing API is not documented here.

TODO (Instructor Review): Decide whether the generated `registerinterface.py` API is ready for publication and, if so, provide a durable source artifact that can be linked to an exact commit.
