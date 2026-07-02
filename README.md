# TaskMaster-C3 — app project template

Build your own apps for a **TaskMaster-C3** device (1.3" OLED + rotary encoder + two
buttons). **Fork this repo** — it's your project. The OS (`taskmaster_core`) is a
**sealed dependency** you never fork or edit; you only add apps.

## The one file you edit: [`apps.yaml`](apps.yaml)

```yaml
core:                                   # the sealed OS — pin a release tag, never edit it
  git: https://github.com/EyalYe/TaskMaster.git
  path: components/taskmaster_core
  version: v1.2.1
apps:                                   # YOUR apps
  - name: app_skeleton
    path: apps/app_skeleton
```

A build hook compiles `apps.yaml` into `main/idf_component.yml` (generated, git-ignored)
— you never touch that, `main/`, or the core.

## Write an app

Copy [`apps/app_skeleton`](apps/app_skeleton) to a new folder, rename it, and add an
entry in `apps.yaml`. An app fills in a `device_app_t` (`init` / `on_event` / `render`
/ `exit`) and self-registers. Full contract: **[`docs/APP_API.md`](docs/APP_API.md)**.

## Example apps

Real, working apps built exactly this way — read them, or use one as a starting point:

- **[TM-Pomodoro](https://github.com/EyalYe/TM-Pomodoro)** — a **full-screen graphical**
  work/break timer (two shrinking circles, no hint bar). Shows raw LVGL drawing, the
  live `tick_ms`, and long-press Select. *(pins core `v1.2.0`)*
- **[TM-ToDo](https://github.com/EyalYe/TM-ToDo)** — **two apps sharing one library**,
  pick one: Todoist (Cloud) or a LAN server (Local), both over a shared `todo_common`
  (offline cache + write-replay). Shows a shared component + app selection in `apps.yaml`.
- **`apps/app_skeleton`** (in this repo) — the minimal starting point.

## Build it

Two ways — you don't need a toolchain if you use CI:

- **CI (no toolchain):** push. **GitHub Actions** builds the firmware and attaches the
  two `.bin` files (below) to the workflow **artifact / Release**. *(`.github/workflows/build.yml`)*
- **Local:** with ESP-IDF installed, `idf.py set-target esp32c3 && idf.py build`.

## The two firmware files (which is which)

Every build — CI or local — produces **two** files. Use the right one for the job:

| File | Use it for | Command |
|---|---|---|
| **`firmware-merged.bin`** | **First flash over USB** — the *whole* image (bootloader + partitions + app) | `python tools/flash.py --bin firmware-merged.bin` |
| **`taskmaster_c3_app.bin`** | **OTA update over Wi-Fi** — the *app image only* | `python tools/ota_serve.py --bin taskmaster_c3_app.bin` |

- **From CI:** download the artifact/Release — both files are in it.
- **From a local build:** the files are under `build/`. You can skip `--bin` entirely — run
  `python tools/flash.py` (auto-flashes the full `build/` output) or `python tools/ota_serve.py`
  (auto-serves the app image). Only a *downloaded* file needs `--bin`.

## Flash + update — local tools, nothing hosted

Nothing is hosted in the cloud. The two Python tools in [`tools/`](tools) only need
`pip install esptool` (no ESP-IDF):

- **First flash (USB):** `python tools/flash.py --bin firmware-merged.bin` (or bare
  `python tools/flash.py` for a local build). Auto-detects the port; if it won't connect,
  hold **BOOT**, tap **RESET**, release **BOOT** (download mode), then retry.
- **Updates (OTA over your LAN):** `python tools/ota_serve.py --bin taskmaster_c3_app.bin`
  (or bare for a local build). It prints a URL; point the device's **`fw_url`** at it
  (Settings → Web config), then Settings → **Check update** — the device downloads over
  Wi-Fi and reboots into it. You host nothing.

## What the device does

The OS gives you: a Launcher, provisioning (Wi-Fi + your app's config from a phone),
a Settings hub, a status bar (time · weather · connectivity), offline caching, and OTA
— all provided by `taskmaster_core`. Your app just draws and reacts. See the core repo
for the full picture.
