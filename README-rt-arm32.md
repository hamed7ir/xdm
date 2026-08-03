# Unofficial fork — Windows RT (ARM32) support

**This is an unofficial fork of [subhra74/xdm](https://github.com/subhra74/xdm).**
All credit for XDM belongs to its author, Subhra Das Gupta. This fork exists only to add
Windows RT / ARM32 support and an updated icon. It is not affiliated with or endorsed by
upstream.

> **Please report RT-specific issues HERE, not upstream.**
> The original author cannot reproduce them — Windows RT is an ARM32 platform he has no reason
> to own, and bug reports he cannot act on only cost him time. Anything that reproduces on
> ordinary Windows x86/x64 belongs upstream, and I would rather it went there.

---

## What this fork changes

Exactly one thing, in three files:

```
app/XDM/XDM.Win.Installer/icon.ico                    | Bin
app/XDM/XDM.WinForms.IntegrationUI/Icon/xdm-logo.ico  | Bin
app/XDM/XDM.Wpf.UI/xdm-logo.ico                       | Bin
3 files changed, 0 insertions(+), 0 deletions(-)
```

**An updated application icon.** No source line is modified. `<ApplicationIcon>` already
pointed at `xdm-logo.ico`, so replacing the asset covers the embedded executable icon, the
tray icon and the installer icon without touching the build.

Branched from **tag `8.0.29`** (`1ca5a25a`). At the time of forking that commit is also
upstream's `master` head — the tag is used as the anchor because it is immutable, so this fork
stays pinned to a known release even if `master` moves later.

See [`CHANGES-rt-arm32.md`](CHANGES-rt-arm32.md) for the dated change record, and
[`BUILD.md`](BUILD.md) for how to build (the repository README describes the old Java/Maven
build and is stale for XDM 8).

## What this fork does NOT change

**Nothing else.** XDM 8.0.29 runs on Windows RT ARM32 *unmodified* — HTTPS, multi-segment
downloading and resume all work with the stock binaries. No compatibility patch was needed.
The fork exists to make the build reproducible and to carry the icon, not to fix a defect.

`LICENSE` and every upstream copyright notice are byte-identical to upstream.

## Running XDM on Windows RT

Two prerequisites, neither of which is part of this repository:

1. **WPF for ARM32.** Windows RT 8.1 does not ship WPF, so a WPF application cannot start at
   all. An ARM32 build was published on XDA in 2018. Those are Microsoft's binaries and are not
   redistributed here.
2. **`SQLite.Interop.dll` for ARM32.** The only bundled native DLL XDM's managed code
   P/Invokes, and it is needed at startup. Not built from this repository.

Everything else — all of XDM's own assemblies — is AnyCPU and runs as-is.

**Known limitation:** the video-download path cannot work on ARM32. `yt-dlp_x86.exe` and
`ffmpeg-x86.exe` are x86 *processes*, and Windows RT has no x86 emulation. XDM on RT is a
plain and segmented HTTP/HTTPS downloader.

## Licence

GPL-2.0, unchanged from upstream. See [`LICENSE`](LICENSE).
