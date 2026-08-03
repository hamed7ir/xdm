# Change record — rt-arm32 fork

Unofficial fork of [subhra74/xdm](https://github.com/subhra74/xdm).
Branched from tag **`8.0.29`**, commit `1ca5a25aae007826c859c81bea494e7c102e1242`.

## Why this file exists

GPL-2.0 §2(a) requires modified files to "carry prominent notices stating that you changed the
files and the date of any change."

Every file this fork modifies is a **binary `.ico`**. A binary cannot carry a text notice, and
**no source file is modified at all** — so there is nothing for a per-file notice to attach to.
This file is the notice instead, covering every change with its date, alongside the commit
history.

If a future change touches source, that file gets its own in-file notice in the usual way.

---

## 2026-08-02 — application icon

Replaced the application icon. No source line changed.

| file | from | to |
|---|---|---|
| `app/XDM/XDM.Wpf.UI/xdm-logo.ico` | 4,904 bytes | 99,678 bytes |
| `app/XDM/XDM.WinForms.IntegrationUI/Icon/xdm-logo.ico` | 4,904 bytes | 99,678 bytes |
| `app/XDM/XDM.Win.Installer/icon.ico` | 4,904 bytes | 99,678 bytes |

`XDM.Wpf.UI/xdm-logo.ico` is dual-purpose and needs no project change: `<ApplicationIcon>`
(line 14) embeds it in the executable, and `<None Update="xdm-logo.ico">` (line 78) copies it
to the output directory, where `AppTrayIcon.cs` loads it at runtime for the tray. So the one
asset covers the window, taskbar, Alt-Tab, Explorer and tray icons.

The new icon contains 16/32/48/64/128 px entries. The 256×256 entry present in the source
artwork was dropped — it accounted for 270 KB of 370 KB on its own, and 128 px was judged ample
for every surface Windows renders here. **That judgement was wrong; see the 2026-08-03 entry.**
The entries are **not** PNG-compressed: `AppTrayIcon.cs` loads the file with
`System.Drawing.Icon`, which has a long history of rejecting PNG-compressed `.ico` entries, and
a throwing tray icon would be a visible regression.

`app/XDM/xdm-logo.ico` (repository root copy) was deliberately **left unchanged**: no `.csproj`
or `.wxs` in the tree references it, and it may serve the Linux build. Changing it would widen
the diff for no benefit on Windows.

---

## 2026-08-03 — application icon, full size ladder

Replaced the same three `.ico` files again. No source line changed.

| file | from | to |
|---|---|---|
| `app/XDM/XDM.Wpf.UI/xdm-logo.ico` | 99,678 bytes | 419,110 bytes |
| `app/XDM/XDM.WinForms.IntegrationUI/Icon/xdm-logo.ico` | 99,678 bytes | 419,110 bytes |
| `app/XDM/XDM.Win.Installer/icon.ico` | 99,678 bytes | 419,110 bytes |

**Corrects the 2026-08-02 decision to drop the 256×256 entry.** On the device, with the desktop's
icon size scrolled up, XDM's icon rendered visibly smaller and softer than a neighbouring
application whose icon carries a 256. Windows uses 256×256 for "Extra large icons" and for large
thumbnail views; without it the shell upscales the 128 and the result looks exactly like a
lower-quality icon. 270 KB was the wrong thing to economise on.

The file now carries **16, 20, 24, 32, 40, 48, 64, 96, 128 and 256 px**. The six sizes that were
already there are copied through byte-for-byte — that artwork was reviewed on the device and
there was no reason to re-encode it. The four new ones (20, 24, 40, 96) are the DPI-scaled shell
metrics, generated from the 256 with Lanczos; Windows would otherwise synthesise them by scaling
the nearest frame at draw time.

Every entry remains an uncompressed 32bpp BMP, for the `System.Drawing.Icon` reason above.

Built with `tools/build_icon.py` in the packaging repository.

---

## Not changed

`LICENSE` and all upstream copyright notices are byte-identical to upstream. No behavioural,
functional or build change has been made — XDM 8.0.29 runs on Windows RT ARM32 unmodified.
