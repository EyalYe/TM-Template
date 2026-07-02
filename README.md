# TaskMaster-C3 — app project template

Build your own apps for a **TaskMaster-C3** device (1.3" OLED + rotary encoder + two
buttons). **Fork this repo** — it's your project. The OS (`taskmaster_core`) is a
**sealed dependency** you never fork or edit; you only add apps.

## The one file you edit: [`apps.yaml`](apps.yaml)

```yaml
core:                                   # the sealed OS — pin a version, never edit it
  git: git@github.com:EyalYe/TaskMaster.git
  path: components/taskmaster_core
  version: main
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

## Build it

Two ways — you don't need a toolchain if you use CI:

- **CI (no toolchain):** push. **GitHub Actions** builds the firmware `.bin` and
  attaches it to a **Release**. *(`.github/workflows/build.yml`)*
- **Local:** with ESP-IDF installed, `idf.py set-target esp32c3 && idf.py build`.

## Flash + update — local tools, nothing hosted

The maintainer hosts nothing. Two small Python tools live in [`tools/`](tools):

- **First flash (USB):** `python tools/flash.py --port <PORT> --bin <merged.bin>`
  — flashes over USB via `esptool`. *(coming in this template)*
- **Updates (OTA over your LAN):** `python tools/ota_serve.py --bin <app.bin>` serves
  the image from your machine; point the device's **`fw_url`** (Settings → Web config)
  at `http://<your-machine>:8000/firmware.bin` and it self-updates over Wi-Fi.
  *(coming in this template)*

## What the device does

The OS gives you: a Launcher, provisioning (Wi-Fi + your app's config from a phone),
a Settings hub, a status bar (time · weather · connectivity), offline caching, and OTA
— all provided by `taskmaster_core`. Your app just draws and reacts. See the core repo
for the full picture.
