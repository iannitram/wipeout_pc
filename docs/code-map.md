# wipeout_pc code map

- **Repo:** iannitram/wipeout_pc (public fork). **Kind:** C loader and runtime plus a C++ GPU library (CMake and Ninja, driven by `build.sh`), Python offline tools and a Windows launcher: a native Windows (and Linux AppImage) port of *WipEout Omega Collection* (PS4) in the style of bbport.
- **Upstream:** AbdelmoumenDeghbouche/wipeout_pc (`upstream/main`). **Integration branch:** `main`. This is the kept copy of a pair; `wipeout-omega` is the superseded one.
- **Describes commit:** `06f3019` (2026-10-27). Upstream's `main` is about 60 commits further on; the daily sync has not merged them, so they are not described here.

The game's original x86-64 code runs directly on the CPU, a runtime replaces the PS4 system libraries, and the graphics are translated to Vulkan by a vendored shadPS4 video core. There is no CPU emulator. One `eboot.bin` runs WipEout 2048, HD and Fury. No game files are in the repo. Many names (`gow3-probe`, `GOW3_*` variables, launcher files, `gow3:` comments) are left over from the God of War III port this project started from; upstream plans to rename them.

## Layout

| Path | What it owns |
|---|---|
| `src/` | Loader (`probe.c`) and HLE runtime, one `runtime_*.c` per area (kernel, memory, thread, file, audio, ajm, mp3, music, json, videodec, pad, savedata, services, and others); ISA rewrite and emulation (`isa_rewrite.c`, `isa_emulate.c`), red-zone handling (`redzone.cpp`), CPU check, Windows compat layer, per-game hook headers, `import_names.inc` |
| `scripts/` | Offline tools: `pkg_extract.py` and `pkg_keys.py` (fake PKG unpack), `prepare.py`, `link_libc.py`, `link_modules.py`, `game_profiles.py`, `game_update.py`, `gen_import_names.py`, `patches.py`, `mods.py`, `save_backup.py`, `steam_shortcut.py`, `windows_debug.py`, graphics options, Discord presence |
| `gpu/` | Renderer library (CMake project, `gpu/CMakeLists.txt`); see the next table |
| `launcher/` | Windows settings and start window (`gow3_launcher_win.py`, assets, language, Vulkan probe, button icons) |
| `packaging/` | `windows/` (zip build `package.sh`, `smoke-test.ps1`, DLSS build script, resource files, `README-Windows.txt`), `linux/` (AppImage: `build.sh`, `AppRun`, `ci-deps.sh`, smoke test), `default.nix`, `gow3-entry.c`, `appimage.sh` |
| `patches/` | `WipEout_Omega_Collection.xml` and `God_of_War_III_Remastered.xml`: game patches compiled at start by `scripts/patches.py` |
| `extras/mods/Ultrawide` | An optional ultrawide mod |
| `tools/` | Developer tools: A/B scripts, FSR 4 shader rewrites and capture (`fsr4cap/`), `bridge_helper/`, controller diagram drawing, frame-map and import tools |
| `tests/` | About 110 test files (C, C++ and Python) plus `data/` and `stubs/`: runtime pieces (MP3, videodec, services, save data, JSON, pad map, audio downmix), hooks, pipeline cache, camera motion, Windows debug tooling, patches |
| `docs/` | `windows-debug.md`, `windows-debug-wsl.md`, `MODS.md`, `upscaler.md`, `parallel_gpu.md`, `motion_vectors.md`, `ps4-memory.md`, `ps4-pro-video.md`, `ultrawide.md`, `DEPTH_ADAPTIVE_TAA.md` |
| `.github/` | Workflows `linux-appimage.yml`, `windows-zip.yml`, `windows-debug.yml`; issue forms |
| `third_party/LibAtrac9` | Submodule: ATRAC9 decoder |
| `build.sh`, `msys2-env.sh`, `run.py`, `run.sh` | Build driver (also `--test`), MSYS2 environment, run wrappers |
| `README.md`, `LICENSE` | Upstream's readme and license |
| `AGENTS.fork.md`, `CHANGELOG.fork.md`, `PIPELINE.md` | Our rules for this mirror, change history, stage record |

### Inside `gpu/`

| Path | What it owns |
|---|---|
| `shadps4/` | Vendored shadPS4 subset (`VENDOR.txt`: commit `b4e7ae7`, GPL-2.0-or-later, local changes marked `gow3:`, plus backported recompiler fixes): `video_core`, `shader_recompiler`, `common`, `core` |
| `shim/` | Glue between runtime and renderer: `gow3gpu.cpp`, overlay, settings, copy and write logging, actor and orb hooks, window and ImGui |
| `dlss_bridge/`, `tools/`, `patches/` | DLSS bridge; an FSR 4 benchmark; patches applied to the FSR-Vulkan submodule |
| `third_party/` | In-tree: `fonts`, `gcn`, `half`, `sirit`. Submodules: `fsr-vulkan`, `imgui`, `magic_enum`, `miniz`, `xbyak` |

## Where our changes live

Upstream owns everything. `git log upstream/main..HEAD` is one commit (the pipeline normalisation) and three files added: `AGENTS.fork.md`, `CHANGELOG.fork.md`, `PIPELINE.md`.

## Build and test

```sh
git clone --recursive <repo> && cd wipeout_pc
bash build.sh                  # in the MSYS2 CLANG64 shell on Windows
bash build.sh --test           # builds and runs the native C tests
python -m unittest discover -s tests -p "test_windows_debug*.py"   # as the debug CI does
```

- Windows build needs MSYS2 CLANG64 with the packages in `packaging/windows/README.md`, Python 3.10+ with Tk, and `pillow iced-x86==1.21.0 cryptography`. Set `core.autocrlf` to false before building or the FSR patches do not apply. Submodules: `gpu/third_party/{fsr-vulkan,imgui,magic_enum,miniz,xbyak}`, `third_party/LibAtrac9`.
- Individual C tests are `EXCLUDE_FROM_ALL` CMake targets such as `mp3-test`, `videodec-test`, `services-test`, `savedata-test`, `json-test`, `pad-map-test`, `audio-downmix-test`.
- CI: `windows-zip.yml` (build, package, smoke test; a tag publishes a pre-release), `linux-appimage.yml` (Ubuntu 24.04 build, AppImage smoke test on Debian, Fedora, Ubuntu, Arch and openSUSE), `windows-debug.yml` (debug-tool tests on Windows).

## Mac / PC / Linux

- Windows 10/11 is the primary target (PIPELINE.md: platforms windows), with a CPU with AVX and a Vulkan 1.3 GPU. A Linux AppImage is built in CI and the README says to build from source on Linux.
- Not targeted on macOS. A Mac session can read and edit files but cannot build or run anything here (R04).
