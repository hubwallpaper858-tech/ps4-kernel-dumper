Kernel Dumper — FW 14.02
===

Dumps your PlayStation 4's kernel from memory to a USB drive.

This is a fork of the [Scene Collective kernel dumper] that adds support for firmware
**14.02** (build `J02700521`).

## What's new in this fork
- **FW 14.02 support** with anti-freeze guards in `source/main.c`:
  - Validates that `kexec` returned a real kernel base (`base >> 32 == 0xFFFFFFFF`)
    before dumping, instead of running on a bogus base.
  - Caps the dump size at 48 MB, so a corrupt in-memory ELF header can no longer make
    the dump loop spin the console for hours (the original "freeze" seen on 14.02).
- The kernel is written to `<USB>/PS4/<firmware>/kernel.bin`, with a `kernel.complete`
  marker so a finished dump is skipped on re-run.

## Notes
- Built with the [Scene Collective Payload SDK]. For FW 14.02, build against the
  [14.02 SDK fork], which already defines the offsets
  (`K1402_XFAST_SYSCALL = 0x1C0`, `K1402_COPYOUT = 0x2BDC50`, relative to kernel base
  `0xffffffff82200000`) and adds `caseentry(1402)` to the firmware switch.
- Supports any firmware the SDK supports; this fork is tested on 14.02.
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
