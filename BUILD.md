# Building XDM for Windows (this fork)

> The repository README documents a **Java/Maven** build. That is stale — it describes XDM 7.
> XDM 8 is C#, built with the .NET SDK. This file documents what actually works.

Nothing here is ARM-specific. See [Why there is no ARM32 build](#why-there-is-no-arm32-build).

---

## What you need

| | | |
|---|---|---|
| .NET SDK | 8.0 (or any current SDK) | only a *build* tool — nothing from it ships |
| .NET Framework targeting packs | 4.5 and/or 4.6 | usually already present with Visual Studio |
| Windows | any x64 machine | the output is AnyCPU; you do not need an ARM machine |

The SDK version is irrelevant to the output. `TargetFramework` decides what is produced, and
that is pinned below.

---

## Build the Windows app

```
dotnet restore app/XDM/XDM.Wpf.UI/XDM.Wpf.UI.csproj -p:TargetFramework=net4.5

dotnet build   app/XDM/XDM.Wpf.UI/XDM.Wpf.UI.csproj ^
    -c Release ^
    -p:TargetFramework=net4.5 ^
    -p:AssemblyVersion=8.0.29.0 ^
    --no-restore
```

Output: `app/XDM/XDM.Wpf.UI/bin/Release/net4.5/xdm-app.exe`

### Both switches are load-bearing

**`-p:TargetFramework=net4.5`** — the csproj's checked-in default is `net4.7.2`; the
multi-target line is commented out:

```xml
<!--<TargetFrameworks>net4.6;net4.5</TargetFrameworks>-->
<TargetFramework>net4.7.2</TargetFramework>
```

The released `-win7` MSI is built at **net4.5** and the `win8` one at **net4.6**. Neither is
what you get by default, so pin it explicitly or you will not reproduce a shipped build.

**`-p:AssemblyVersion=8.0.29.0`** — the csproj hardcodes `<AssemblyVersion>8.0.1</AssemblyVersion>`.
Every release build has to override it. (Upstream may want to fix that; it is not this fork's
business to change.)

### ⚠ Two projects produce `xdm-app.exe`

| project | AssemblyName | TFMs |
|---|---|---|
| `XDM.Wpf.UI` | `xdm-app` | net4.7.2 / **net4.6 / net4.5** ← the Windows app |
| `XDM.Gtk.UI` | `xdm-app` | **net6.0** ← the Linux app |

Building the wrong one yields a perfectly plausible `xdm-app.exe` that targets net6.0. It will
not run on Windows RT, and the failure looks like a mystery launch problem rather than a build
mistake. Check the output before shipping it — see the gate below.

`XDM.App.Host` produces `xdm-app-host.exe`, which is **not** part of the Windows payload at all.

---

## Parity gate

Run this on any build you intend to ship. Both properties are required for the binary to run on
ARM32 at all, and neither is obvious from the build log.

```powershell
Add-Type -AssemblyName System.Reflection
$a = [System.Reflection.AssemblyName]::GetAssemblyName("bin\Release\net4.5\xdm-app.exe")
$a.ProcessorArchitecture      # must be MSIL  (i.e. AnyCPU)
$a.Version                    # must be 8.0.29.0
```

Or with `corflags.exe` from the Windows SDK:

```
corflags bin\Release\net4.5\xdm-app.exe
```

Required:

```
ILONLY         : 1
32BITREQUIRED  : 0        <- if this is 1, it cannot run on ARM32
```

`32BITREQUIRED = 1` marks the assembly x86-only. `<Platforms>x86</Platforms>` in the csproj
makes this look likely, but a default `dotnet build` produces AnyCPU — verify rather than
assume.

Also confirm the `TargetFrameworkAttribute` on the output is **not** `net6.0` or later.

---

## Why there is no ARM32 build

There is nothing to build for ARM32. **XDM's managed assemblies are AnyCPU** — the same IL runs
on x86, x64 and ARM32, JIT-compiled by whichever CLR loads it. That is why a stock build works
on Windows RT unmodified.

The only native components are:

| file | what it is |
|---|---|
| `SQLite.Interop.dll` | System.Data.SQLite's native half — **needs an ARM32 build** |
| `yt-dlp_x86.exe`, `ffmpeg-x86.exe` | separate x86 **processes** — cannot run on ARM32 at all |

`SQLite.Interop.dll` is on the *startup* path (XDM opens `downloads.db` immediately), so an
ARM32 build of it is required. It is not built from this repository — see the project README
for where to get one.

`yt-dlp` and `ffmpeg` are launched as child processes and are only used for the video path.
Ordinary and segmented HTTP/HTTPS downloading does not touch them.

---

## Building the installer

The MSI is built with WiX 3.11 (`app/XDM/XDM.Win.Installer/`). There are separate `.wxs` files
per target framework — `product-net4.5.0.wxs`, `product-net4.6.0.wxs` — matching the two
released MSI variants. This fork does not change the installer.
