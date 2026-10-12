# KingpinQ3: status of the build, issues and Android port

This file records what has been verified, what was changed, and what is still
open. It is updated as work lands on branch `arena/971e7b42-kingpinq3-source-code`.

## 1. Can the game be played end to end?

The repository contains engine and game **code only**. The original Kingpin
assets (`basekpq3/pak0.pk3`, see `misc/setup/setup.xml.in`) are not included, so a
full playthrough cannot be verified from this repository alone. Everything below
is about the code: it builds, and the issues listed were checked against it.

## 2. Build verification (GitHub Actions)

Workflow: `.github/workflows/build.yml`

| Job | Toolchain | Builds | Status |
|-----|-----------|--------|--------|
| `linux` | Ubuntu 22.04, gcc 11 | client, dedicated server, game module | passing |
| `linux-gcc9` | `gcc:9.3.0` container | client, dedicated server, game module | passing |

Compiler and linker diagnostics are reported as annotations and in the step summary
when a build fails.

Bugs found and fixed by the CI work:

* `code/renderer/tr_fog.cc` did not compile in `COMPAT_KPQ3` builds, because
  `trGlobals_t::glfogsettings` was only declared for `COMPAT_ET`. The declaration
  now also covers `COMPAT_KPQ3`. The ET-only consumers are still guarded.
* The client link rules did not depend on the internal libjpeg archive
  (`$(JPEGSLIB)`), so a `make -j` build could link before the archive existed.

## 3. Issue status (upstream: 0xA5EA/KingpinQ3-Source-Code)

| # | Title | Status | Notes |
|---|-------|--------|-------|
| 1 | fix linux build with gcc 9.3.0 | builds on gcc 9.3 and 11 in CI | Can be closed once the maintainer confirms. |
| 2 | Fix build on windows 10 with msvc19 | not verified | Needs a Windows/MSVC runner. |
| 3 | water/heathaze render issue | open | Needs a map and a visual check. |
| 4 | Rocket Launcher missing impact effects | fixed in code | `CG_RocketExplosion_Init` now adds the same fireball sprite and ember splash as the grenade launcher. Needs in-game confirmation. |
| 5 | fix flame damage | open | Balance change; needs playtesting to pick values. |
| 6 | kaas does not compile .aas on 64-bit | open | The error comes from the BSP splitter (`CheckPlaneAgainstParents` in `code/kaas/brushbsp.c`). It needs a reproducing map. |
| 7 | missing death message Grenade launcher | open | The obituary path looks complete; the drop is probably an event-overflow or ordering issue. Needs runtime logs. |
| 8 | Enhance the vote message ingame | open | UI work. |
| 9 | Use freetype to render fonts ingame | open (feature) | FreeType rasterisation exists in `tr_font.cc`, but the game loads pre-rendered `.dat` glyph sets and the runtime TTF path is commented out. |
| 10 | botlib error with kpq3dm2 (no entity with model 1) | open | Needs the map. |
| 12 | deluxmap light falloff | open | Lives in the `kmap2` compiler. |
| 13 | pk3 files for .bsp distribution | open (feature) | Tooling work in `kmap2`. |
| 14 | mega health missing | open | `Pickup_Health` clamps mega health to max. The "over max" branch is commented out with a FIXME. Needs a decision on intended behaviour. |

## 4. Android port

Not started in code yet. The current code has no Android support, and the blockers
are larger than the touch UI itself:

* **Backend.** The client uses SDL 1.2 (`code/sdl/*`, `code/SDL12/*`). Android needs SDL2
  or a native EGL/GLES window layer.
* **Renderer.** `code/renderer` is desktop OpenGL: about 987 GL calls, 25 immediate-mode
  `glBegin`/`glEnd` blocks, fixed-function matrix calls, and GLSL `#version 120/130`
  shaders. Porting it to GLES 3.0 is the largest piece of work.
* **Input.** Touch controls, a layout editor (move, resize, show or hide buttons),
  a switch that turns touch off for gamepad or keyboard play, and gamepad mapping.
* **Launcher.** A Kotlin/Java front end for choosing the data folder, settings and
  starting the engine.
* **Build.** Needs the Android NDK and SDK. This sandbox cannot reach
  `dl.google.com` or Google Maven, so an APK cannot be built here. The plan is to
  build it in GitHub Actions with the Android SDK/NDK installed on the runner.

Work is planned in phases: (1) SDL2 backend on desktop; (2) GLES renderer, checked
on desktop with an ES profile; (3) Android project with NDK build and SDL2 glue;
(4) touch and gamepad input, then the layout editor; (5) launcher, settings and
device testing.
