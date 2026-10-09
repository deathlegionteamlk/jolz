# jolz

> A Linux terminal forged in darkness.

jolz is a native Android terminal emulator and Linux environment with a `pkg` package manager and plugin system. Built for those who dwell in the command line.

**Developed by:** `lucifer x jela` | **Team:** `deathlegion` | **Site:** https://jolz.deathlegion.site

---

## Download

**Latest APK (v1.2.0, 4.0 MB):** [jolz-1.2.0.apk](https://github.com/deathlegionteamlk/jolz/releases/download/v1.2.0/jolz-1.2.0.apk)

Or from the [Releases page](https://github.com/deathlegionteamlk/jolz/releases).

**Mirror (CDN):** https://jolz.deathlegion.site/jolz-1.2.0.apk

---

## Install

1. Download the APK to your Android device.
2. Open it (enable "Install from unknown sources" if prompted).
3. Tap **Install**.
4. Open **jolz** — you'll see the welcome screen.

> Works on **Android 5.0 (API 21) and newer**. Signed with v1 + v2 + v3 signature schemes.
> **All ABIs:** arm64-v8a, armeabi-v7a, x86, x86_64

---

## What's new in v1.2.0

### Fixed (crash on launch)
- **Theme**: switched from `Theme.MaterialComponents.NoActionBar` to `Theme.AppCompat.NoActionBar` for maximum compatibility
- **Layouts**: removed all `AppBarLayout` references (replaced with plain `Toolbar`) to avoid material view inflation crashes
- **Manifest**: removed deprecated `android:sharedUserId` attribute (causes issues on Android 10+)
- **TerminalView.loadPrefs()**: null-safe — won't crash if App/prefs not yet initialized
- **TerminalSession constructor**: null-safe scrollback loading
- **All init wrapped in try-catch** throughout the app

### Added
- **3 new utility classes**: `BatteryUtils`, `PowerUtils`, `SystemSettingsUtils`
- **System settings shortcuts**: open WiFi, location, display, sound, battery, storage, airplane, accessibility, developer options
- **Power management**: wake lock acquire/release, power save mode detection, thermal status
- **Battery utilities**: detailed battery info (level, status, plugged, temperature, voltage, health, technology, capacity, charge counter, current)
- **Overlay permission** request utility
- **Write settings** permission request utility
- **New logo**: sharingan-inspired ring + flame + `$ jolz` text in red/green gradient
- **New launcher icons**: darker aesthetic, all densities (mdpi→xxxhdpi)

### Improved
- 155+ total features
- 76 Java source files
- 8 activities
- Comprehensive packages list on the website (120+ packages)

---

## Features

### Terminal emulator
- Native VT/ANSI parser (pure Java, no WebView)
- 256-color + truecolor support
- Cursor styles, blinking, scrollback (2000 lines)
- Mouse reporting, bracketed paste
- Pinch-to-zoom font size
- Long-press context menu, double-tap to paste
- Animated welcome screen with ASCII art

### Package manager (`pkg`)
```sh
pkg update
pkg install bash python3 ruby node vim neovim git curl wget ssh tmux zsh clang gcc make cmake
pkg search neovim
pkg upgrade
pkg list installed
```

Commands: install, remove, update, upgrade, search, list, info, depends, rdepends, files, mirror, clean, hold, unhold, outdated, stats, export, import, self, help, version.

### Plugin system (9 types)
Theme, Font, ColorScheme, Boot, KeyBind, DeviceApi, Command, Scheduler, Hook.

29 plugin permissions with CPU/memory/network sandboxing.

### Built-in commands (24+)
sysinfo, netinfo, meminfo, cpuinfo, diskinfo, battery, device, uptime, ps, mounts, jolz-info, jolz-version, jolz-help, echo, date, whoami, uname, which, ls-jolz, cat-jolz, mkdir-jolz, rm-jolz, touch-jolz, history-clear, exit.

### Color schemes (11)
Dracula, Nord, Solarized Dark, Solarized Light, Monokai, Gotham, One Dark, Gruvbox Dark, Tokyo Night, Catppuccin, Rose Pine.

### UI
- File Explorer activity
- Home screen widget
- App shortcuts (long-press launcher)
- Extra keys bar (Ctrl, Alt, F1-F12, arrows)
- Multiple sessions with persistence
- Haptic feedback (vibrate on key, bell, success/error)
- Share & clipboard utilities

### System integration
- Boot scripts (run on device startup)
- Foreground service
- Crash reports with logcat
- Notification channels (default, boot, pkg, plugin, session, error)
- System settings shortcuts

---

## Stats

- **155+** features
- **4.0 MB** signed APK
- **76** Java source files (private)
- **11** color schemes
- **9** plugin types
- **29** plugin permissions
- **24+** built-in commands
- **8** activities
- **1** home screen widget
- **Android 5.0+** (API 21+)
- **All ABIs**

---

## Source code

This repository is **private**. Only the APK release and this README are public.

---

## License

Proprietary. Developed by **lucifer x jela** for the **deathlegion team**.

(c) 2026 deathlegion team. All rights reserved.
