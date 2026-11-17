# Herdr

Herdr is a desktop app for terminals and coding agents. Keep work in spaces, see which agents need you, and leave terminals running when you close the window.

## Install

### Linux and macOS

```sh
curl -fsSL https://releases.herdr.dev/app/install.sh | sh
```

Linux needs a desktop session and a working Vulkan driver. macOS builds are available for Apple Silicon; Intel Mac users can [build from source](#build-from-source). Linux builds are x86-64 and ARM64.

The installer adds Herdr to your apps and puts the command in `~/.local/bin`.
Add that folder to your PATH if the installer asks.
On macOS the app lives in `~/Applications/Herdr.app`.
For drag-and-drop installation on Apple Silicon, download [Herdr.dmg](https://releases.herdr.dev/app/Herdr.dmg) and drag Herdr to Applications.

On macOS (Apple Silicon), you can also install with [Homebrew](https://brew.sh/):

```sh
brew install --cask herdr-app
```

### Windows

Run in PowerShell on x64 Windows with curl and tar installed (Windows 10 1803 or newer):

```powershell
irm https://releases.herdr.dev/app/install.ps1 | iex
```

Herdr appears in the Start menu. The installer adds `%LOCALAPPDATA%\Programs\herdr-app` to your user PATH. Open a new terminal to pick it up.

### Windows ARM64 (this fork)

This fork targets **native Windows ARM64** for the graphical desktop app,
not the older terminal-only [`herdr`](https://github.com/herdrdev/herdr).
Download `herdr-desktop-windows-aarch64.zip` from
[this fork's releases](https://github.com/atastrophic/herdr-app/releases).
These are unsigned experimental builds; read the release's validation notes
before using them. GUI compatibility is unverified unless those notes say otherwise.
It contains `herdr-app` and the matching ARM64
[`herdr-core`](https://github.com/atastrophic/herdr-core), including Microsoft's
ARM64 ConPTY host. Keep each folder's contents together.

Extract the archive, then copy `herdr-app` and `herdr-core` into
`%LOCALAPPDATA%\Programs`. Add both folders to your user PATH and open
`herdr-app\herdr-app.exe` directly, or pin it to Start. It is a native GUI:
Windows Terminal and PowerShell are not needed to display the app.
The upstream PowerShell installers above still download x64 builds only.
Use this fork's releases for manual ARM64 updates; upstream does not publish
the Windows ARM64 assets needed by the updater.

#### Multiple desktop windows

The default launch still opens or raises the default window. Open another
window with a persistent named profile:

```powershell
herdr-app --profile work
herdr-app --profile personal
herdr-app --profile work status
```

Different profiles can run simultaneously. Each has its own settings, mods
and window state under `<app home>\profiles\window-<name>`; they share the
same core and terminals. Closing one window does not close the others.
Names use 1-24 ASCII letters, digits, hyphens or underscores.
Run these commands from a regular terminal. Inside a profile's own terminal,
bare `herdr-app status` already addresses that profile through its inherited
`HERDR_APP_HOME`.

#### Building the ARM64 bundle

The local build uses Rust 1.100.0 for `aarch64-pc-windows-msvc`,
Zig 0.16.0 (the x64 distribution under Windows emulation), native ARM64 LLVM
23.1.5, and Visual Studio 2022 Build Tools with the ARM64 C++ tools and
Windows 11 SDK. Initialize submodules before building.

From this checkout in PowerShell, adjust the Zig path to your installation:

```powershell
$env:ZIG = 'C:\tools\zig-x86_64-windows-0.16.0\zig.exe'
$env:LIBCLANG_PATH = 'C:\Program Files\LLVM\bin'
$env:PATH += ";$env:USERPROFILE\.cargo\bin;C:\Program Files\LLVM\bin"
cmd /c 'call "C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\VC\Auxiliary\Build\vcvarsarm64.bat" >nul && cargo build --release --locked'
```

Build [the core fork](https://github.com/atastrophic/herdr-core/tree/windows-arm64)
with the same environment, then run its
`scripts\fetch-conpty.ps1 -Dest target\release -Arch arm64`.
The app's executable is `target\release\herdr-app.exe`; the core executable
and its `conpty` directory come from the core checkout's `target\release`.
Do not set `HERDR_APP_RELEASE`: this source build intentionally leaves
automatic upstream app/core updates disabled.

The shell and PowerShell installers download and run code. Read the [shell installer](distribution/install.sh) or [PowerShell installer](distribution/install.ps1) first if you prefer.
They also install [herdr-core](https://github.com/herdrdev/herdr-core) if it is missing.
The app can help install or update a core it cannot use.

## Find your work

A **space** holds work on one machine, usually in one repo. Each space has **tabs**. Split a tab into **panes** to put terminals or mod panels beside each other.

An unnamed space follows its first terminal's folder. Name a space or tab when you want a label that stays.
New tabs start in the focused terminal's folder.

The agents sidebar brings agents from your spaces into one list, including workers started by another agent.
Agents that report status show as:

- **Working:** still busy.
- **Blocked:** waiting for you.
- **Done:** finished while you were looking elsewhere.
- **Idle:** not working, or a finished agent you have looked at.

Click an agent to see it. A worker without a pane opens in **peek**, over your tab. Keep it as a tab if you want it beside your other work.
Ctrl-click a live terminal address in terminal text to peek at it; use Cmd-click on macOS.
An address such as `t7` means a terminal on the same machine as the terminal showing it. `vps/t7` names one on another machine.

The window arranges terminals. The core runs them.
Closing the window leaves them running; closing a pane or tab ends its terminals.
After a reboot or core crash, kept panes get fresh shells in their last folders, not their old programs.

## Work on another machine

Add an SSH machine from Settings → Machines or the sidebar's Add machine action.
Pick a host from your SSH config or type how SSH reaches it.
Spaces on that machine run their terminals there, not on your computer.

Each machine keeps its spaces, tabs and splits in its core. Connected screens share that structure and the live terminals, but keep their own focus and zoom.
Remote work keeps running when your local window closes or your laptop sleeps, as long as the remote machine and core stay up.

## Make it yours

Open the command palette to find an action by name, or open Settings to change fonts, colors, sounds and other preferences.
Settings > Look also adjusts terminal line height, cell width and scroll speed. All default to 100%; layouts can override them with `terminal_line_height` (0.8–2), `terminal_cell_width` (0.8–2) and `terminal_scroll_speed` (0.1–5).
The built-in layouts include Herdr, Terminals and Herdr 98.
A **layout** arranges the window. **Mods** are small JavaScript programs that draw its panels, menus and dialogs.

Duplicate a shipped layout or mod before changing its structure or code. Your copies stay yours; Herdr updates the shipped ones on start.
Save a personal layout, mod or key file and the window reloads it.
A broken edit keeps the last good version and reports the error.

Ctrl+Shift+P shows every command and its key (Cmd+Shift+P on macOS). Change any key in `keys.json`; see [the keys section](defaults/AGENTS.md#keys-keysjson) of the mod guide for the format, including JSON objects for named, typed command arguments.

On macOS, Cmd+A selects the terminal's whole screen and scrollback, ready for Cmd+C.
In vim and other alternate-screen apps, it selects only that screen.
Linux and Windows keep Ctrl+A for the shell; bind `terminal.select_all` in `keys.json` if desired.

[Layouts and mods](docs/mods.md) · [Commands](docs/commands.md)

## Updates and privacy

Installed releases check for updates in the background and download a new app beside the running one. It takes over when you quit and reopen, or choose the update action. Your terminals keep running in the core.
Settings → Updates shows what is available. What's New opens after an update; you can open it again from the command palette.
Builds from source do not update themselves.

Usage telemetry is on by default in releases. Turn it off in Settings → Privacy.
Mods are trusted code, not sandboxed extensions. Read [security and privacy](docs/security.md) before installing someone else's mod.

## For agents

```sh
herdr-app --skill
```

This prints the agent guide for driving the window. The herdr-core skill points agents here.
Use herdr-core for running and coordinating programs; use the app to change what you see.

```sh
herdr-app ls
herdr-app status
herdr-app commands
```

The first two inspect the open window. The command list also works without one.
See the [command reference](docs/commands.md) for arguments.

## Build from source

For bugs, mod API requests and code contributions, see [Contributing](CONTRIBUTING.md).

Clone this repo with its submodules. Use Rust 1.95.0 and Zig 0.16.0, the versions used in [CI](.github/workflows/ci.yml).
Linux needs the development packages listed in that workflow.

```sh
git clone --recurse-submodules https://github.com/herdrdev/herdr-app.git
cd herdr-app
just build
just test
just install
```

Install [just](https://github.com/casey/just) to use these recipes.
The platform installer builds a release and adds the app to your desktop.
On Linux it links to this checkout, so keep the checkout in place.
Quit and reopen Herdr to use the new build.

Documentation checks also need [Bun](https://bun.sh): `just docs-check` checks generated pages and CLI examples; `just docs-generate` refreshes the reference pages.

To compare drawing changes on Windows, Linux or macOS, run the same fixed workloads against each build:

```sh
python scripts/bench.py --bin <app executable> --set spinners --seconds 20
```

The runner requires `hc` on PATH and uses the Python executable running the script. It prints a unique artifact folder with the run settings, counters and results, then stops its private app, terminals and core. Every set uses Python fixture programs; the default set’s TUI workload replaces historical `btop` measurements. `--only spinners` measures just the quiet native spinner scenario. `--set titles` runs eight agents whose titles spin after a prefix (`π ⠹ task`) on the main layout. Keep the window visible and the display awake, and compare the same window activity and display settings; inactive windows may draw at a lower rate. Windows records visibility, foreground activity and display Hz at each measurement boundary; other platforms mark these observations unavailable. Visibility does not establish occlusion. Draws are window render calls. Render-to-paint elapsed time stops before scene finalization, backend drawing, uploads and presentation; it is separate from OS process CPU time. Neither measures physical presentation or the FPS HUD’s derived MAX FPS.

`python scripts/bench_compare.py ghostty --root <artifact folder>` reuses a run’s fake agent for the macOS comparison. Benchmark checks: `python -m unittest discover -s scripts -p test_bench.py`.
