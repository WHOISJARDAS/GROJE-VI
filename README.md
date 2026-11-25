# GROJE-VI

A fork of [Belobug/VIctory-Belobug-AMD](https://github.com/Belobug/VIctory-Belobug-AMD) (itself a fork of [Marvisak/VIctory](https://github.com/Marvisak/VIctory)) that runs the Xbox Series build of the title on a Windows PC with an AMD Radeon card, from files you supply yourself. It follows Belobug's `main` and adds fixes for machines where the other trees do not start or do not last.

**This repository contains no game files, code or assets.** You need your own copy of the title. Not affiliated with or endorsed by its developer or publisher.

## State (2026-11-25)

Measured on one machine: AMD Radeon RX 9070 XT 16 GB (driver 32.0.32029.2010), Ryzen 7 9800X3D, 32 GB RAM, Windows 11 26200, Series X profile, diagnostics build. The fixed test scene is the first mission's autosave: boot, the opening, then the first room with the player standing still, frames 240-360 s after start.

| | GROJE-VI | DexNom/VIctory `2e01ab4` | Belobug `main` |
|---|---|---|---|
| Starts on a machine with Bottom-up ASLR off | yes | no (`int3` at `+0x19f4f58`) | no (same) |
| AMD compiler crash (`amdxc64.dll+0x861b9f`) | none | ends the run in 27-128 s (with the 4 GiB fix, so that it starts) | none |
| Frame rate in the test scene (median) | **8.3 fps** (4.6-10.3), 120 ms per frame | — (does not start; with our fixes: up to 7.9 fps, then `DEVICE_HUNG` at 171 s) | — (does not start) |
| Long runs | 3 six-minute runs: 0 exceptions, no device removal | — | — |

## What GROJE-VI adds to Belobug's `main`

- **The space below 4 GiB is kept for the title's 32-bit memory** (a port of Marvisak/VIctory#1009). Without it the title stops at its own assertion about 1 s in whenever host memory (here 2.3 GB of the AMD driver's mapped shader cache) fills the low 4 GiB, which is every start when Windows' Bottom-up ASLR is off. Sent as Belobug#39.
- **DualSense device paths end in a NUL**: `CreateFileW` read past each path; with no DualSense connected the run ended minutes in (`ntdll!wcslen`). Sent as Belobug#40, confirmed on two other machines.
- **A present waits for its copy before reading the probe and capture**, so `VICTORY_ASYNC_SUBMIT=1` no longer removes the device in diagnostics builds. Sent as Belobug#197.
- **`VICTORY_KEYS_FILE` lines can hold keys** (`W 1500`, `W+Z 2000`), to drive the title without its window in the foreground (diagnostics).

## Build and run

Rust (the version in `rust-toolchain.toml`) and Visual Studio Build Tools with the Windows SDK. The BCPack tables (`crates/victory-formats/src/bcpack-tables.bin`) as described in `docs/asset-policy.md`.

```powershell
cargo build --profile diag -p victory --features diagnostics
$env:VICTORY_PRESENT_TARGET = '1x1:1'
$env:VICTORY_HAIR = '1'
.\target\diag\victory.exe run "<your game folder>\game_scarlett_master.exe" --execute --console series-x
```

The first launch after a build translates every shader and builds about 2,500 pipelines: the loading screen can stay frozen, with sound, for minutes.

## Credits

- Marvisak and the VIctory contributors: the runtime everything here builds on (GPL-3.0-or-later).
- Belobug and the contributors whose pull requests Belobug merged: the Radeon work this fork follows.
- NuraadGit and MichaelSmithAI: the low 4 GiB guards (Marvisak/VIctory#1009).
- Developed with [Claude Code](https://claude.com/claude-code); every number above comes from runs on the hardware named.

Belobug's own README (the settings, the stand-in light, the known problems) is at [Belobug/VIctory-Belobug-AMD](https://github.com/Belobug/VIctory-Belobug-AMD#readme).
