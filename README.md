<div align=center>

<img src="extras/banner.png" alt="Banner" width="60%">

</div>
<h1 align=center>Geometry Dash · Switch Port</h1>

A wrapper/port of the Android releases of **Geometry Dash**, **Meltdown**,
**SubZero**, **World**, and **Lite**. It loads the original arm64 game binaries —
`libcocos2dcpp.so` (RobTop's cocos2d-x 2.2 fork + the game) and the stock
`libfmod.so` — patches them and runs them inside a minimal Android-like
environment natively on the Switch.

### Switch 2 (kaiHEN)

This fork builds for **Switch 2 running [kaiHEN](https://github.com/ouncebrew/kaihen)**
by default, using the [ounce-devkit](https://github.com/ouncebrew/ounce-devkit)
(libounce in place of libnx, OunceVK — NVK + Zink — in place of Switch 1 Mesa).
The install layout below is the same; launch hbmenu by holding **R** while
opening a game, then start the port from there.

What differs from the Switch 1 build:

* **Code mapping goes through kaiHEN's `ldr:ro`.** A kaiHEN game process may not
  call `svcMapProcessCodeMemory` / `svcSetProcessMemoryPermission`, so each
  Android `.so` is described to RO as an NRO in place (text RX, ro R, data RW)
  and RO maps it. See the comment above `so_place_all()` in `source/so_util.c`.
* **SaltyNX** hooks are no-ops (it's a Switch 1 Atmosphere sysmodule).
* Thread pinning uses whatever cores the process core mask grants instead of
  assuming Switch 1's four.
* `debug.log` (loader layout, module addresses, fatal errors) and `stderr.log`
  (OunceVK/Mesa output) are written next to the `.nro`. Grab these if
  something breaks on hardware.

### Download

Grab `gdash_switch2.nro` from the [Releases](../../releases) page (Switch 2 /
kaiHEN). Every push also builds Switch 1 and Switch 2 `.nro`s under
[Actions](../../actions); open a run and download its artifacts.

### How to install

Your saves and settings folder from the Switch 1 port works as-is: copy it
over and drop the Switch 2 `.nro` in.

You need a **2.2.14x arm64** Android release; extract the files from your own
legally-owned copy of the game.

Compatibility has been checked against the supplied **full game 2.2.145**
arm64 package (code 42), **2.2.148** Meltdown (code 15), SubZero (code 13),
World (code 15), and Lite (code 44), alongside the older full-game library.
The port handles both the older GNU and the newer Android libc++ string layouts
used by these releases.

Create a separate folder for each game you install and fill it:

```
/switch/<anything>/
  gdash_switch2.nro       # or whatever the .nro is called
  libcocos2dcpp.so        # from the APK's lib/arm64-v8a/
  libfmod.so              # from the APK's lib/arm64-v8a/
  assets/                 # the entire contents of the APK's assets/ folder
  config.txt              # created on first run
  prefs.txt               # created on first run; includes anonymous network ID
  cacert.pem               # generated from the Switch firmware trust store
  save/                    # created on first run; game saves
```

### Notes

This will not run in applet/album mode — it needs the full memory of a game
override. Launch hbmenu by holding **R** while opening an installed title
(on kaiHEN, the first R-launch after starting HEN may open the game itself;
close it and do it again), then start the port. On Switch 1 a forwarder also
works.

`config.txt` is created on first run:

* `screen_width` / `screen_height` — render resolution; `-1` auto-picks
  1280x720 handheld / 1920x1080 docked.
* `cursor_speed` — pointer speed of the stick-driven cursor, px/s at 720p.
* `menu_autosave_interval_seconds` — menu save cooldown, **30** seconds by
  default; `0` disables menu autosaves. Old `autosave_interval_seconds` entries
  are ignored so older test configurations use the new menu-only default.

### How to build

Install the devkitPro Switch toolchain and portlibs:

```sh
pacman -S devkitA64 switch-tools libnx switch-sdl2 switch-mesa switch-libdrm_nouveau
```

For Switch 2, also install the ounce devkit (`libounce`, `ouncevk`,
`libkaihen`) by running `sh install.sh` from
[ounce-devkit](https://github.com/ouncebrew/ounce-devkit).

Then `make` for Switch 2, or `make SWITCH1=1` for the original Switch 1 build.

No local toolchain? Use Docker the same way CI does: the "Build the devkit
image" step in `.github/workflows/build.yml` makes an `ounce-devkit` image
from devkitPro's image plus the three [ounce] packages, then:

```sh
docker run --rm -v "$PWD:/build/gdash" -w /build/gdash ounce-devkit make
```

(The devkit's own `docker build` works too, but it syncs pacman with
pkg.devkitpro.org, which sometimes answers 403.)

### Credits

* naga (NaGaa95) for the original Geometry Dash Switch port this fork builds on.
* The kaiHEN / ounce-devkit contributors for the Switch 2 homebrew runtime and SDK.
* TheOfficialFloW for the original Android so-loader.
* hatoving for the Geometry Dash PS Vita port this one is based on.
* fgsfds for max_nx / the Switch so-loader groundwork reused here.

### Support

Support naga, the original port's author:

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/D1D1P2MOG)

### Legal

This project has no affiliation with RobTop Games. "Geometry Dash" and
related marks belong to their respective owners. No assets or program code
from the original game or its Android release are included here. We do not
condone piracy; users must own a legal copy of the game.

Unless noted otherwise, the source in this repository is under the MIT
License, as stated in the original port and in the headers of its source files.
