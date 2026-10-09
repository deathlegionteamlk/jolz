# jolz

> A full-featured Linux terminal in your pocket.

jolz is a native Android terminal emulator and Linux environment. Run a real shell, install real packages with `pkg`, and extend it with plugins. Built for everyone who lives in the terminal.

**Developed by:** `lucifer x jela` | **Team:** `deathlegion` | **Site:** https://jolz.deathlegion.site

---

## Download

**Latest APK (v1.0.0, 3.9 MB):** [jolz-1.0.0.apk](https://github.com/deathlegionteamlk/jolz/releases/download/v1.0.0/jolz-1.0.0.apk)

Or grab it from the [Releases page](https://github.com/deathlegionteamlk/jolz/releases).

**Or download from the official site:** https://jolz.deathlegion.site/jolz-1.0.0.apk

---

## Install

1. Download the APK to your Android device.
2. Open it (enable "Install from unknown sources" if prompted).
3. Tap **Install**.
4. Open **jolz** and you're in a Linux shell.

> Works on **Android 5.0 (Lollipop, API 21) and newer**, including all versions of Android 7, 8, 9, 10, 11, 12, 13, 14, and 15. The APK is signed with v1 + v2 + v3 signature schemes so it installs cleanly on every Android version.

**Supported ABIs:** `arm64-v8a`, `armeabi-v7a`, `x86`, `x86_64`

---

## What's inside

### Terminal emulator core
- Native VT/ANSI parser (no JavaScript, no WebView — pure Java)
- 256-color + truecolor (24-bit) support
- Cursor styles, blinking, scrollback (default 2000 lines)
- Mouse reporting, bracketed paste
- Resizable on the fly, pinch-to-zoom font size
- Long-press context menu, double-tap to paste

### Package manager (`pkg`)
Install other tools and languages directly from the terminal:

```sh
pkg update
pkg install bash
pkg install python3 ruby node vim neovim git curl wget ssh tmux zsh clang gcc make cmake
pkg search neovim
pkg info python3
pkg upgrade
pkg list installed
```

Other commands: `remove`, `update`, `upgrade`, `search`, `list`, `info`, `depends`, `rdepends`, `files`, `mirror <list|add|remove|select>`, `clean`, `hold`, `unhold`, `outdated`, `stats`, `export`, `import`, `self`, `help`, `version`.

Features: dependency resolution with topological sort, multiple mirrors, package signatures verification (SHA-256), parallel downloads, backup & restore of installed packages.

### Plugin system (9 types)
- **Theme** — change terminal appearance (background, foreground, accent)
- **Font** — load custom .ttf/.otf fonts
- **ColorScheme** — ship your own 16-color palette
- **Boot** — run shell scripts on device startup
- **KeyBind** — register custom key bindings
- **DeviceApi** — call Android APIs (vibrate, location, flashlight, sensors, battery, clipboard, notifications)
- **Command** — register custom shell commands
- **Scheduler** — run tasks on interval/cron/once
- **Hook** — listen for app events

Plugins live in `$JOLZ_HOME/plugins/installed/<id>/plugin.json`. Each plugin declares its required permissions; the runtime enforces a 29-permission sandbox with CPU, memory, and network budgets.

### Built-in commands (24+)
Available without any package installed:

| Command | Description |
|---|---|
| `jolz-info` | Show app info |
| `jolz-version` | Show version |
| `jolz-help` | Show help |
| `sysinfo` | System info |
| `netinfo` | Network info |
| `meminfo` | Memory info |
| `cpuinfo` | CPU info |
| `diskinfo` | Disk usage |
| `battery` | Battery status |
| `device` | Device info |
| `uptime` | Device uptime |
| `ps` | Process list |
| `mounts` | Mount points |
| `echo` | Echo args |
| `date` | Show date/time |
| `whoami` | Print user |
| `uname [-a]` | System name |
| `which <cmd>` | Locate command |
| `ls-jolz <dir>` | List files in jolz home |
| `cat-jolz <file>` | Cat a file |
| `mkdir-jolz <dir>` | Make directory |
| `rm-jolz <path>` | Remove path |
| `touch-jolz <file>` | Touch a file |
| `history-clear` | Clear history |
| `exit` | Exit info |

### Color schemes (11)
Dracula, Nord, Solarized Dark, Solarized Light, Monokai, Gotham, One Dark, Gruvbox Dark, Tokyo Night, Catppuccin, Rose Pine.

### Extra keys bar
Toggleable bar with: `ESC`, `TAB`, `CTRL`, `ALT`, `↑`, `↓`, `←`, `→`, `HOME`, `END`, `PGUP`, `PGDN`, `F1`–`F12`, `-`, `/`, `*`, `DEL`, `ENTER`.

### Sessions
- Multiple concurrent terminal sessions
- Persist sessions across app restarts
- Session switcher activity

### System integration
- Home screen widget (tap to open terminal)
- Boot scripts (run automatically on device startup)
- Foreground service for long-running tasks
- Crash reports with logcat capture (saved to `crashes/`)
- Notification channels: default, boot, pkg, plugin, session, error

### Settings
Appearance (font size, font family, color scheme, theme mode, colors), behavior (wake lock, vibrate, sound, scrollback, shell path, initial command, extra keys, fullscreen, immersive, keep screen on, persistent sessions, bell, swap ctrl/alt, tab completion, boot scripts, history size, encoding), network (mirror, repositories, auto-update, signature verification, parallel downloads, timeout), storage (crash report, log level).

---

## Stats

- **124+** features implemented
- **3.9 MB** signed APK
- **69** Java source files (not included in this repo)
- **11** color schemes
- **9** plugin types
- **29** plugin permissions
- **24+** built-in commands
- **Android 5.0+** (API 21+)
- **All ABIs** (arm64, arm, x86, x86_64)

---

## Source code

This repository is **private**. Only the APK release and this README are public. The Java source code is not distributed.

For bug reports, feature requests, or licensing inquiries, contact the deathlegion team at https://jolz.deathlegion.site.

---

## Acknowledgements

Built with:
- AndroidX (AppCompat, RecyclerView, Preference, Lifecycle, Fragment)
- Material Components for Android
- A custom VT/ANSI terminal emulator written from scratch in Java

Inspired by Termux, but rewritten from scratch with a focus on the plugin architecture and the deathlegion team's design language.

---

## License

Proprietary. Developed by **lucifer x jela** for the **deathlegion team**.

© 2026 deathlegion team. All rights reserved.
