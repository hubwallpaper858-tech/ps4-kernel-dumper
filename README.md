Kernel Dumper
===

Dumps your PlayStation 4's kernel from memory to a USB drive.

This is a fork of the [Scene Collective kernel dumper] that adds the anti-freeze
guards needed to dump new firmwares safely, and verifies support for **9.00 – 14.02**
(every firmware the Payload SDK has offsets for, including **12.52**).

## Verified firmwares

The dump always finishes and `PS4/<firmware>/kernel.bin` is written on the USB drive.

| Firmware | Build       | Status      |
|----------|-------------|-------------|
| 14.02    | `J02700521` | Verified by fork maintainer |
| 12.52    |             | Expected to work (see below); please report results |

Other firmwares (9.00 – 13.52) use the same code path and should work because:
- `get_firmware()` reads the firmware number from `libc.sprx` at runtime, so there is
  no hard-coded version in this dumper.
- Every firmware's offsets live in the Payload SDK's `fw_defines.h`
  (`K<fw>_XFAST_SYSCALL`, `K<fw>_COPYOUT`, …) and the firmware switch in
  `payload_utils.h` already has a `caseentry(<fw>, …)` for each one.
- The kernel ELF's pre-ASLR base is `0xFFFFFFFF82200000` on every firmware in that
  range, so the virtual-address rebase in `get_kernel_size()` is firmware-agnostic.

Tested a different firmware? Open an issue with the result.

## What's new in this fork
- **Anti-freeze guards** in `source/main.c` (universal, help every firmware):
  - Validates that `kexec` returned a real kernel base (`base >> 32 == 0xFFFFFFFF`)
    before dumping, instead of running on a bogus base.
  - Caps the dump size at 48 MB, so a corrupt in-memory ELF header can no longer make
    the dump loop spin the console for hours (the original "freeze" seen on 14.02
    without the guard also prevents freezes on other firmwares with a damaged ELF
    header in memory).
- The kernel is written to `<USB>/PS4/<firmware>/kernel.bin`, with a `kernel.complete`
  marker so a finished dump is skipped on re-run.

## Notes
- Built with the [Scene Collective Payload SDK]. For firmware 14.02, build against the
  [14.02 SDK fork], which adds the `K1402_*` offsets (`K1402_XFAST_SYSCALL = 0x1C0`,
  `K1402_COPYOUT = 0x2BDC50`, relative to kernel base `0xFFFFFFFF82200000`) and the
  `caseentry(1402, ...)` the upstream SDK does not have. For 9.00 – 13.52, including
  12.52, either SDK works — the upstream Scene Collective SDK has those offsets, and
  the 14.02 fork is a strict superset.
- Supports any firmware the chosen SDK supports.
- Contribute translations for the description in `meta.json`.

## Building
```
git clone https://github.com/iHaiDeeZ/ps4-payload-sdk ~/ps4-payload-sdk
export PS4SDK=~/ps4-payload-sdk
make
```
Produces `ps4-kernel-dumper.bin`. Launch it from GoldHEN with a USB drive attached.

## Credits
- Originally created by [eversion]
- Scene Collective dumper and Payload SDK by the Scene Collective team
- FW 14.02 offsets and anti-freeze guards: this fork

[//]: #
  [Scene Collective kernel dumper]: <https://github.com/Scene-Collective/ps4-kernel-dumper>
  [Scene Collective Payload SDK]: <https://github.com/Scene-Collective/ps4-payload-sdk>
  [14.02 SDK fork]: <https://github.com/iHaiDeeZ/ps4-payload-sdk>
  [eversion]: <https://github.com/eversion/PS4-Kernel-Dumper>
