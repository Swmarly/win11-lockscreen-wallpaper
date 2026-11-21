# Lock Screen Wallpaper for Windows 11 (multi-monitor)

In Extend mode on Windows 10 or 11, the lock screen normally shows its
wallpaper on the primary monitor and leaves secondary monitors black. This
tray utility fills those secondary monitors with the Windows lock-screen
wallpaper or images you choose. It uses the signed overlay technique from
[VoidVolker/LockScreen](https://github.com/VoidVolker/LockScreen).

<img width="228" height="178" alt="image" src="https://github.com/user-attachments/assets/18679ede-af2e-482b-b522-dcfcdb5cd819" />


## Install

```powershell
git clone https://github.com/Swmarly/win11-lockscreen-wallpaper.git
```

Install the [.NET 8 SDK](https://dotnet.microsoft.com/download/dotnet/8.0)
on Windows 10 or 11. Open the cloned `win11-lockscreen-wallpaper` folder in
File Explorer and double-click `install.cmd`. It creates or reuses a local
code-signing certificate, builds and signs the app, and prompts for
administrator approval to install it. After installation, look for the app
in the system tray. You can also relaunch it from the Start menu by
searching for **LockScreenWallpaper**.

**Updating.** Run `git pull` from the cloned folder, then run `install.cmd` again.

**Uninstalling.** If you enabled **Start with Windows**, turn it off from
the tray menu before running `uninstall.cmd`. The uninstaller leaves your
settings and event log in `%AppData%\LockScreenWallpaper`.

## Usage

Right-click the tray icon:

| Option | What it does |
|---|---|
| **Sync with Windows lock screen** | On by default. Reads the current Windows lock-screen image each time you lock or preview the overlay. Turn it off to use your custom images. |
| **Set default wallpaper...** | Image shown on every secondary monitor that doesn't have its own assigned image |
| **Set wallpaper for monitor** | Assign a distinct image to a specific secondary monitor |
| **Preview overlay (5s)** | Shows the overlay for 5 seconds without locking, to check placement/scaling |
| **Open event log** | Opens a timestamped log of lock/unlock events, handy if something isn't showing up as expected |
| **Start with Windows** | Adds/removes a startup entry (recommended, since the app has to already be running before you lock) |
| **Exit** | Quits the app |

With sync on, the Windows image takes precedence on all secondary monitors.
If Windows cannot provide it, the app uses each monitor's custom image or
the default image if that monitor has no assignment.

Settings live at `%AppData%\LockScreenWallpaper\settings.json`.

## How it works

When Windows reports a session lock through `SystemEvents.SessionSwitch`,
the app creates a borderless, non-activating overlay on each secondary
monitor. Each image is scaled and cropped to fill its monitor. The app
closes the overlays on unlock. The signed app uses `uiAccess` so the
overlays can appear above `LockApp.exe` on the same desktop.

The secondary wallpapers stay visible during PIN entry when Windows keeps
the sign-in flow on the same desktop. Windows does not expose a supported
signal for the start of PIN entry to this tray app, so the secondary
wallpapers keep their existing appearance. The overlays are unavailable
while Windows shows a secure desktop for credential entry.

## Why signing is required

The app requests `uiAccess="true"`, a Windows feature also used by
assistive software. With the default Windows security policy, a UIAccess
app needs a trusted digital signature and must run from a protected
location such as Program Files.

The installer builds and signs the executable with a local self-signed
certificate, adds that certificate to this machine's trust stores, and
installs the app under `%ProgramFiles%\LockScreenWallpaper`.

## Security considerations

`install.cmd` makes two real, system-wide changes. Read this before
clicking through the UAC prompt:

1. **It adds a self-signed certificate to your machine's Trusted Root
   Certification Authorities and Trusted Publishers stores**, the same
   trust store used for real public CAs. Anything signed by that
   certificate's private key is trusted machine-wide for code-signing
   purposes afterward. It's scoped to a `CodeSigningCert` (can't be misused
   for TLS or email), and the private key is generated **non-exportable**,
   so it can't be copied off the machine by something running as you. But
   the risk isn't zero: anything that can act as you can still *ask that
   key to sign things*, and a signed-and-trusted binary can also request
   `uiAccess` itself. Reasonable for a personal machine you administer
   yourself; not something to do on a shared, managed, or high-security
   machine.
2. **It installs the app to `%ProgramFiles%\LockScreenWallpaper`**,
   a protected directory that satisfies the default `uiAccess` location policy.

If you'd rather not do either of these, see the alternative below.

## Alternative approach (no signing required)

If full-duration coverage matters more to you than avoiding a system trust
change, the other proven technique is **mirroring the display topology**
instead of drawing an overlay: switch to "Duplicate" mode on lock (every
monitor mirrors the primary) and back to "Extend" on unlock, via
`SetDisplayConfig(SDC_APPLY | SDC_TOPOLOGY_CLONE)`. That's what
[Wintermelon](https://github.com/arun-goud/Wintermelon) does. No signing or
trust changes at all, and it survives real `LogonUI` credential entry, but
every monitor shows the *same* mirrored image, there's a brief resolution
change on lock/unlock, and open windows get reshuffled. This repo
intentionally took the other trade-off: distinct per-monitor images, at the
cost of the signing setup above.

## Credits

- [VoidVolker/LockScreen](https://github.com/VoidVolker/LockScreen): the
  signed-overlay-window technique this project is based on.
- [arun-goud/Wintermelon](https://github.com/arun-goud/Wintermelon): the
  topology-mirroring alternative described above.

## License

[MIT](LICENSE)
