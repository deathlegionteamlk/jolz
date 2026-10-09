# jolz

> A Linux terminal forged in darkness.

jolz is a native Android terminal emulator and Linux environment with a `pkg` package manager and plugin system. Built for those who dwell in the command line.

**Developed by:** `lucifer x jela` | **Team:** `deathlegion` | **Site:** https://jolz.deathlegion.site

---

## Download

**Latest APK (v1.3.0, 4.0 MB):** [jolz-1.3.0.apk](https://github.com/deathlegionteamlk/jolz/releases/download/v1.3.0/jolz-1.3.0.apk)

Or from the [Releases page](https://github.com/deathlegionteamlk/jolz/releases).

**Mirror (CDN):** https://jolz.vercel.app/jolz-1.3.0.apk

---

## Install

1. Download the APK to your Android device.
2. Open it (enable "Install from unknown sources" if prompted).
3. Tap **Install**.
4. Open **jolz** — you'll see the welcome screen.

> Works on **Android 5.0 (API 21) and newer**. Signed with v1 + v2 + v3 signature schemes.
> **All ABIs:** arm64-v8a, armeabi-v7a, x86, x86_64

---

## What's new in v1.3.0

### Fixed (black screen / crash)
- **Environment not cleared** — the shell process now inherits the system environment instead of clearing it (was removing critical vars like BOOTCLASSPATH, ANDROID_DATA)
- **Layout inflation** — replaced `?attr/actionBarSize` with fixed `56dp` to avoid theme attribute resolution failures
- **ShellEnvironment simplified** — removed references to non-existent paths (usr/lib, etc/inputrc, etc/bash.bashrc) that could cause shell startup issues
- All previous fixes from v1.2.0 retained (theme, AppBarLayout removal, sharedUserId removal, null safety)

### Features
- Native VT/ANSI terminal emulator (pure Java)
- `pkg` package manager (install bash, python3, ruby, node, vim, git, ...)
- 9 plugin types (theme, font, boot, device API, command, scheduler, ...)
- 11 color schemes (Dracula, Nord, Solarized, Monokai, One Dark, Gruvbox, Tokyo Night, Catppuccin, Rose Pine, Gotham)
- Home screen widget
- App shortcuts
- File Explorer
- Boot scripts
- 24+ built-in commands
- Foreground service
- Crash reports with logcat
- 155+ features
- 8 activities

---

## Source code

This repository is **private**. Only the APK release and this README are public.

---

## License

Proprietary. Developed by **lucifer x jela** for the **deathlegion team**.

(c) 2026 deathlegion team. All rights reserved.
