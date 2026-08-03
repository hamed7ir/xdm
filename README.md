<h1 align="center">Xtreme Download Manager — Windows RT (ARM32)</h1>

<p align="center">
  An <b>unofficial</b> fork of <a href="https://github.com/subhra74/xdm">subhra74/xdm</a>
  that runs on Windows RT 8.1 on ARM32.<br>
  All credit for XDM belongs to its author, <b>Subhra Das Gupta</b>.<br>
  Not affiliated with, endorsed by, or supported by the XDM project.
</p>

<p align="center">
  <a href="https://github.com/hamed7ir/xdm/releases/latest"><b>Download 8.0.29-rt1</b></a>
  &nbsp;·&nbsp;
  <a href="BUILD.md">Build it yourself</a>
  &nbsp;·&nbsp;
  <a href="README-rt-arm32.md">What this fork changes</a>
</p>

---

<p align="center">
  <img width="1366" height="768" alt="xdmPNG" src="https://github.com/user-attachments/assets/3cfbdd2f-a47e-4161-b7f3-62267be9100e" />

       alt="XDM 8.0.29-rt1 running on a Surface RT (screenshot to be added)"
       width="720">
</p>
<p align="center"><i>XDM running on a Surface RT — Windows RT 8.1, Tegra 3, ARM32.</i></p>

---

## Why this exists

Windows RT was left with almost no desktop software, and a download manager is one of the things
a slow, resumable connection actually needs.

The useful finding is that **there was nothing to port**. XDM 8.0.29 runs on Windows RT ARM32
*unmodified* — HTTPS, multi-segment downloading and resume all work with the stock assemblies,
because they are AnyCPU and the CLR simply JITs them for ARM. No compatibility patch was needed,
and this fork contains no behavioural change.

Two things were missing, and neither is XDM's fault:

- **WPF.** RT's desktop .NET Framework is a trimmed build with the WPF assemblies removed, so a
  WPF application cannot start at all — it fails at assembly load, before any of its own code
  runs. An ARM32 build of WPF fixes it. See [Prerequisites](#prerequisites).
- **`SQLite.Interop.dll` for ARM32.** The one native DLL XDM's managed code P/Invokes, and it is
  needed at startup. Built from source — recipe, gates and binary at
  [hamed7ir/sqlite-interop-arm32](https://github.com/hamed7ir/sqlite-interop-arm32).

## Install

Two packages, both in the [latest release](https://github.com/hamed7ir/xdm/releases/latest).
They contain the same application, byte for byte.

| | |
|---|---|
| **Setup** (`…-Setup-AnyCPU.zip`) | Per-user install, no administrator. Start-menu shortcut, Add/Remove Programs entry, uninstaller. |
| **Portable** (`…-portable.zip`) | Extract and run. No registry writes by the package, no shortcuts. |

Neither is an MSI, and that is not a stylistic choice — **Windows RT refuses MSIs**, and an Inno
Setup installer is worse still, because its loader is native x86 and cannot start on ARM at all.
The setup package's `Setup.exe` is compiled **AnyCPU/MSIL**, and is Windows Forms rather than WPF
so that it can still run on the machine where WPF is the missing piece.

Check where you stand first — this changes nothing:

```
Setup.exe --check
```

## Prerequisites

XDM needs exactly two things:

- **.NET Framework 4.5 or later** — `xdm-app.exe` targets `net4.5`
- **WPF**

**It does not need .NET 4.8.** Windows RT 8.1 already ships 4.5.1, and on Windows 10 ARM32, where
4.7 is as high as the Framework goes, that is fine too.

What Windows RT does not ship is WPF. The route verified on the device:

1. **KB2919355** — the Windows RT 8.1 Update
2. a **.NET Framework** (for RT: <https://go.microsoft.com/fwlink/?linkid=2088632>)
3. **`wpf-prerequisite\wpf-setup.cmd`**, shipped in both packages — extract the WPF archive next
   to it and run it
4. XDM

**The WPF binaries are not in these packages and never will be.** They are Microsoft's files,
they are not covered by this project's licence, and they are not ours to redistribute. What ships
is a script and a manifest of names, sizes and SHA256 hashes for all 70 files, so a copy you
obtained yourself can be verified before you run it. A list of hashes is not the content.

The ARM32 WPF build was published on XDA in March 2018 by *never_released*:
<https://xdaforums.com/t/wpf-for-windows-rt-devices.3763345/>

## Browsers

XDM's own extension is a Chrome extension, and there is no Chrome for ARM32 Windows RT.

For **Pale Moon**, **Basilisk** and other UXP browsers there is a separate add-on that hands
downloads to XDM and forwards the cookies the browser would actually have sent — including
`httpOnly` session cookies, so downloads behind a login work:

> **[hamed7ir/xdm-bridge](https://github.com/hamed7ir/xdm-bridge)**

It works with stock, unmodified XDM on any platform, and is linked rather than bundled so the two
can be updated independently.

## Known limitations on Windows RT

**The video-download feature does not work.** `yt-dlp_x86.exe` and `ffmpeg-x86.exe` are x86
*processes*, and Windows RT has no x86 emulation, so they are omitted rather than shipped as
14 MB that cannot execute. XDM degrades cleanly — you get a message, not a crash. If you accept
its offer to download the missing helper it fetches the **x86** build, because upstream selects
the asset by operating system rather than by processor; that download succeeds and still cannot
run.

**The browser-integration guide (`xdm-guide.exe`) is omitted** — it walks you through installing
XDM's Chrome extension into Chrome, Edge, Brave, Opera or Vivaldi, none of which exist for ARM32
RT. Those five buttons in Settings show a "could not launch" message.

XDM on Windows RT is a plain and segmented HTTP/HTTPS downloader. That part is device-tested:
HTTPS, multi-segment downloading, resume across a dropped connection, and downloads behind a
login.

## Reporting problems

> **Please report RT-specific issues HERE, not upstream.**
> The original author cannot reproduce them — Windows RT is an ARM32 platform he has no reason to
> own, and bug reports he cannot act on only cost him time. Anything that reproduces on ordinary
> Windows x86/x64 belongs upstream, and I would rather it went there.

## Building

See **[BUILD.md](BUILD.md)**. The repository's original README described a Java/Maven build; that
is stale for XDM 8, which is C# and built with the .NET SDK.

Nothing here is ARM-specific — the output is AnyCPU and you do not need an ARM machine. Two build
switches are load-bearing and neither is the default; `BUILD.md` explains why, and carries the
gate every build should pass before it ships.

## Licence

**GPL-2.0**, unchanged from upstream — see [`LICENSE`](LICENSE), which is byte-identical to
upstream's.

This fork replaces the application icon and this README, and adds `BUILD.md`,
`README-rt-arm32.md` and `CHANGES-rt-arm32.md`. **No source line is changed.**
[`CHANGES-rt-arm32.md`](CHANGES-rt-arm32.md) is the GPL-2.0 §2(a) change record.

---

*This README was rewritten for this fork on 2026-08-03; it previously described upstream's
Java/Maven build of XDM 7. Upstream's own README is at
<https://github.com/subhra74/xdm#readme>.*
