[![Microsoft Store Badge](https://img.shields.io/badge/Microsoft%20Store-005FB8?logo=microsoftstore&logoColor=fff&style=flat)](https://apps.microsoft.com/detail/9MSX91WQCM2V?)
[![Release](https://img.shields.io/github/v/release/clementgre/ThreeFingerDragOnWindows?label=Download%20version)](https://github.com/clementgre/ThreeFingerDragOnWindows/releases/latest)
[![TotalDownloads](https://img.shields.io/github/downloads/clementgre/ThreeFingerDragOnWindows/total)](https://github.com/clementgre/ThreeFingerDragOnWindows/releases/latest)
[![LatestDownloads](https://img.shields.io/github/downloads/clementgre/ThreeFingerDragOnWindows/latest/total)](https://github.com/clementgre/ThreeFingerDragOnWindows/releases/latest)

## Overview

ThreeFingerDragOnWindows aims to bring the macOS-style three-finger dragging functionality to Windows Precision touchpads.

With a simple touchpad gesture, this app allows you to drag windows and select text (by emulating a cursor drag by holding down the left mouse button).

## Preview
<p align="center">
  <img src='https://raw.githubusercontent.com/ClementGre/ThreeFingerDragOnWindows/main/ThreeFingerDragOnWindows/Assets/Screenshot-1.png' alt="App screenshot: Touchpad tab" width='700'>
  <img src='https://raw.githubusercontent.com/ClementGre/ThreeFingerDragOnWindows/main/ThreeFingerDragOnWindows/Assets/Screenshot-2.png' alt="App screenshot: Three Finger Drag tab" width='700'>
  <img src='https://raw.githubusercontent.com/ClementGre/ThreeFingerDragOnWindows/main/ThreeFingerDragOnWindows/Assets/Screenshot-3.png' alt="App screenshot: Other Settings tab" width='700'>
</p>

## Installation

If the installation fails, your computer might need to have the Windows App SDK redistributable installed. You can download it from this page: [https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/downloads](https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/downloads).

## How to use

Make sure to disable the "Tap twice and drag to multi-select" behaviour and all of the default 3-finger swipe behaviour
via ``Touchpad settings`` in Windows preferences for the drag to work without interferences.

To open the configuration pane, click the ThreeFingerDragOnWindows tray icon on the Windows taskbar.

### Preventing accidental drags while typing (Maximum distance between fingers)

On large touchpads, a thumb or palm edge can rest on the pad while typing. Combined with two
or three fingers of the other hand, this falsely registers as a three-finger drag. The
**"Maximum distance between fingers"** setting (Three Finger Drag tab) uses a single threshold
to handle both cases:

- **3 fingers spread too far apart** (e.g. one thumb on the left + two fingers on the right):
  the drag is **not** started, since real three-finger drags use fingers close together.
- **4 fingers with an outlier** (e.g. one accidental thumb + a genuine three-finger drag): if
  three of the contacts stay within the threshold and one is farther away, the far contact is
  treated as an accidental touch and **ignored**, so the drag still works with the remaining
  three fingers.

To choose a value: enable **Record logs** in the Other Settings tab, reproduce the accidental
touch, then read the `maxPairDist` value printed in the log file and set the threshold just
below it. Set it to **0** to disable the feature (default).

Note: outlier rejection only handles the 4-finger (1+3) case; 5+ fingers keep the original
behaviour.

## Project Status

The main goal of this project has been achieved, and the app is stable and usable. However, there are still a lot of potential improvements and features that could be added in the future, and a lot of platform-specific issues that could be investigated and fixed.
I (Clément Grennerat) am not anymore using Windows as my main OS, but I will continue to maintain the project and am open to contributions from the community!

## Build and Execute

The app targets **.NET 10** and is a packaged WinUI 3 (MSIX) application. It can be built and
run in Microsoft Visual Studio or Jetbrains Rider, **or** fully from the command line.

### Prerequisites (command-line build)

1. **.NET 10 SDK** ¡ª download from <https://dotnet.microsoft.com/en-us/download/dotnet/10.0>
   (verify with `dotnet --list-sdks`).
2. **Windows App Runtime 1.8** ¡ª install via
   `winget install -e --id Microsoft.WindowsAppRuntime.1.8`
   (needed to run an unpackaged build for quick testing).

### Option A ¡ª Quick compile check

```bash
dotnet build -c Debug -p:Platform=x64
```

This produces an unpackaged `.exe` under
`ThreeFingerDragOnWindows\bin\x64\Debug\net10.0-windows10.0.22000\`. Note: the unpackaged
executable only works for basic smoke testing ¡ª the full app is designed to run as an MSIX
package (see Option B).

### Option B ¡ª Build an installable MSIX (no Visual Studio required)

The app must run as a signed MSIX package. From a plain shell, without an IDE:

**1. Create a self-signed code-signing certificate** whose subject matches the `Publisher` in
`Package.appxmanifest` (`CN=A5174EBF-789F-4CD5-BF8B-E0CB932DB9AD`):

```powershell
$cert = New-SelfSignedCertificate -Type CodeSigningCert `
    -Subject 'CN=A5174EBF-789F-4CD5-BF8B-E0CB932DB9AD' `
    -CertStoreLocation 'Cert:\CurrentUser\My' `
    -KeyUsage DigitalSignature -KeyExportPolicy Exportable `
    -NotAfter (Get-Date).AddYears(3)
# Note the Thumbprint printed here; use it in the next steps.
```

**2. Enable signing in `ThreeFingerDragOnWindows/ThreeFingerDragOnWindows.csproj`** ¡ª set
`AppxPackageSigningEnabled` to `True` and add
`<PackageCertificateThumbprint>YOUR_THUMBPRINT</PackageCertificateThumbprint>` in the main
`<PropertyGroup>`. Also change `<AppxBundle>Always</AppxBundle>` to `Never` for a single-arch
package. *(Revert these edits afterwards to keep the repo clean; they are only needed to pack.)*

**3. Generate the package** (the `_GenerateAppxPackage` target is what actually produces the
`.msix`):

```bash
dotnet msbuild -t:_GenerateAppxPackage -p:Configuration=Debug -p:Platform=x64 -p:AppxPackage=True
```

The output lands in
`ThreeFingerDragOnWindows\AppPackages\ThreeFingerDragOnWindows_2.0.7.0_x64_Debug_Test\`.

**4. Trust the certificate and install the package** (run from an elevated PowerShell):

```powershell
# Import the signing cert into the Trusted Root store
$cer = Get-ChildItem 'Cert:\CurrentUser\My' | Where-Object { $_.Thumbprint -eq 'YOUR_THUMBPRINT' }
$store = New-Object System.Security.Cryptography.X509Certificates.X509Store('Root','LocalMachine')
$store.Open('ReadWrite'); $store.Add($cer); $store.Close()

# Install the MSIX
Add-AppxPackage -Path '...\ThreeFingerDragOnWindows_2.0.7.0_x64_Debug.msix'
```

If a package with the same identity is already installed (e.g. the Store version), uninstall it
first: `Get-AppxPackage -Name '50931ClmentGrennerat.ThreeFingersDragOnWindows' | Remove-AppxPackage -AllUsers`.

**5. Launch the app:**

```powershell
Start-Process 'shell:AppsFolder\50931ClmentGrennerat.ThreeFingersDragOnWindows_cvkce2k9t2r60!App'
```

## Libraries used

The app is a WinUI 3 app, that uses the [Microsoft.UI.Xaml](https://docs.microsoft.com/en-us/windows/apps/winui/winui3/) library.

Other libraries used:
- [emoacht/RawInput.Touchpad](https://github.com/emoacht/RawInput.Touchpad) Allows to get the raw input of the touchpad (included in the source code as TouchpadHelper.cs).
- [HavenDV/H.NotifyIcon](https://github.com/HavenDV/H.NotifyIcon) API for Windows taskbar tray icon in a WinUI app.
- [dahall/TaskScheduler](https://github.com/dahall/TaskScheduler) API for Windows TaskScheduler (used for the skipUAC).


<a href="https://apps.microsoft.com/detail/9msx91wqcm2v?mode=direct">
	<img src="https://get.microsoft.com/images/fr%20dark.svg" width="200"/>
</a>
