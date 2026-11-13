# VOkit

An independent native Linux companion for compatible vivo phones, built with Go, GTK4 and libadwaita. GStreamer performs video and audio decoding in native libraries. It does not require Wine or USB debugging. This project is not affiliated with or endorsed by vivo.

## Demo



https://github.com/user-attachments/assets/e7aef66c-e769-4877-8e59-e77b9f24ef67



VOkit running on Linux with X300 FE:

| File manager and phone mirroring | Live camera preview |
| --- | --- |
| ![VOkit file manager with thumbnails and a separate phone mirror window](rimages/screenshot-2026-10-21_17.20.12.png) | ![Live rear-camera preview in VOkit](rimages/screenshot-2026-10-21_17.21.17.png) |
| **Phone storage** | **QR pairing** |
| ![Browsing folders in the connected phone's storage](rimages/screenshot-2026-10-21_17.22.04.png) | ![Pairing by scanning the QR code with Office Kit on the phone](rimages/screenshot-2026-10-21_17.23.14.png) |

## Features

Connect the laptop and phone to the same network, open the desktop app, click **Connect phone**, and scan its QR with the phone's Office Kit connection feature. No phone IP entry or vivo-account sign-in is required: the fresh QR scan supplies the phone's private network address through vivo's rendezvous service. This QR bootstrap needs internet access. An optional IP override is available under Settings → Advanced connection. Approve the requested features on the phone. Closing the main window hides it while the connection and incoming transfers continue. The system tray icon stays available: green means connected, amber means connecting, and red means disconnected. Click it to reopen VOkit, or use its menu to open phone mirroring alone, camera and microphone sharing, phone files, Settings, or connection controls. Closing the main window keeps those features running; choose **Quit VOkit** to stop the session. A desktop panel with StatusNotifier support is required to show the tray icon; GNOME Shell may need a tray extension. Reopening VOkit from the application launcher also reveals the existing window.

Implemented features include file browsing and upload/download, image/video thumbnails and viewing, text clipboard sync, phone screen mirroring with touch and typing, and front/rear camera sharing. The X300 FE with OriginOS 6 has been tested; 720p screen mirroring was reported smooth. Camera latency varies with network and encoder settings. Compatibility with every phone, Linux distribution and laptop is not established.

The desktop shows the connected phone’s name and automatically leaves Settings after pairing. Settings has a close button. It uses a dark File Manager layout with the phone's global Recent, Images, Videos, Audio and Documents collections. More results load as you scroll; search applies to loaded results. Image previews fit the whole image as the window resizes, and videos open in a resizable player with full-screen support. The separate phone mirror keeps touch and typing enabled while streaming; quality and start/stop controls are in its three-dot menu. Its frame follows the stream’s aspect ratio in portrait and landscape. Corner checkboxes select multiple files; the toolbar checkbox selects all loaded files. Save multiple files to a folder in one operation; existing files are preserved with numbered filenames.

Phone Share transfers are received automatically while the paired app runs, including while the main window is hidden. Multi-file and repeated shares were verified on the X300 FE. The default destination is `~/Downloads/Vivo Office Kit`; change it in Settings before the next connection. Existing files are preserved, and repeated names receive a suffix. The receiver accepts regular files, with limits of 1 GiB per file, 200 files / 4 GiB per batch, and four queued batches. No additional receive prompt is required on the PC. Each file uses its own authenticated, pinned data channel to the paired phone; it never refreshes certificate trust or retries a failed request within that channel. Opening media uses the phone’s direct media endpoint through a capability-protected loopback stream, without first downloading a complete file or keeping a disk cache. Images are decoded in memory (up to 32 MiB); videos stream progressively and support seeking (up to 1 GiB). Format support depends on installed decoders. Phones that ignore byte ranges require extra network reads when seeking. Explicit Save and phone Share transfers still write to the chosen download directory.

Camera sharing publishes to a v4l2loopback device, normally `/dev/video10` named **Vivo Phone**. Select that camera in the video-call application. Local preview can be hidden independently of the webcam stream; this passed a live 90-frame Linux consumer check on the X300 FE. Audio sharing uses a PipeWire virtual source when the phone actually provides audio; the tested X300 FE advertised audio but did not send microphone packets in this run, so phone microphone sharing is not yet confirmed. Linux virtual microphones are selected by name in meeting applications, rather than through a separate `/dev/video` node. Enable “Phone and connection notifications” before pairing to forward new phone notifications to the desktop. The phone must allow notification handoff. Notification text is bounded and escaped, and is excluded from diagnostic logs. Vivo-account login is not implemented. The vendor client uses region-specific web sign-in and a login callback, then authenticated device registration/discovery and cloud connection messages; those need a separate implementation and a successful interactive sign-in test. Notes/cloud sync and Remote PC are also not complete.

## Interface language

The desktop supports English, Simplified Chinese, Spanish, French, German and Japanese. It follows the computer's message language by default (`LC_ALL`, `LC_MESSAGES`, `LANG`, and the `LANGUAGE` preference list); unsupported languages fall back to English. Chinese regional variants currently use Simplified Chinese. Settings → Appearance → **Language** lets you choose a language or restore **Follow system**. Restart VOkit after changing it. Display language and phone pairing region are independent. Translation catalogs are bundled with source builds, user installs and Arch packages, with no network translation service required.

Translation files live in `desktop/gtk/vivo_desktop/locales/*.json`. English source messages are the keys; named formatting fields such as `{value0}` must be preserved. `make test-desktop` checks message coverage, formatting fields, locale selection and all supported languages' windows.

## Pairing region

In Settings → **Pairing region / 配对地区**, select **中国大陆（vivo 互传）** for mainland phones, or **Overseas / India (Office Kit)** for the original overseas edition (the default). The choice is saved and applies to the next QR and connection, including both LAN and Wi-Fi Direct. Mainland mode follows vivo Connect's domestic flow: `https://psuite.vivo.com.cn/vbusiness`, an HTTPS QR, and `isOversea=false` in the phone handshake. Overseas mode uses the India gateway, a `pcsuite:` QR and `isOversea=true`. Changing regions requires a fresh scan.

For the CLI, add `--region china` or `--region overseas` to `pair`:

```sh
./build/vivolinux pair --region china --trust-scanned-phone --persistent
./build/vivolinux pair --region china --wifi-direct --trust-scanned-phone --persistent
```

Scan the mainland QR using vivo 互传's computer connection scanner; use Office Kit for the overseas edition. No vivo-account login is required for this anonymous QR pairing flow. Regional protocol tests use simulated phones; full mainland-phone feature compatibility still needs a real-device scan and phone approval.

## Wi-Fi Direct with the existing vivo Connect service

Settings → **Wi-Fi Direct** selects direct QR pairing instead of LAN rendezvous. This reuses the **same installed `vivo-connect-p2p.service`**, private supplicant, DHCP process, and persistent group created by vivo Connect. It does not install or launch a second wireless helper. Enable the persistent service in vivo Connect first, for the same desktop user.

Quit vivo Connect **and its Node backend**, leaving the system service running, then create a QR in VOkit and scan it on the phone. The service is configured for one backend port, so the two frontends cannot own it simultaneously. VOkit reads `/etc/vivo-connect-p2p/service.json` and the existing private API credential, receives the service's registration/heartbeats, and listens for the phone on the group's gateway at port 9199. Registration can take up to 45 seconds. Pairing expires after three minutes. This direct QR path does not use the internet rendezvous service.

Disconnecting or quitting VOkit closes its pairing window and local listeners; the system service and persistent group remain available for the next connection. Each VOkit connection currently requires a new scan; vivo-account/Bluetooth automatic reconnect is not implemented. Leave **Phone IP override** empty unless you know the phone's current address on the direct group. Turn off **Wi-Fi Direct** to return to ordinary LAN pairing.

The CLI uses the same flow:

```sh
./build/vivolinux pair --wifi-direct --trust-scanned-phone --persistent
```

## Install

Arch Linux / CachyOS and Debian / Ubuntu:

```sh
./install.sh --check --with-webcam
./install.sh --with-webcam
vokit
```

Thunar users can right-click selected files and choose **Send To → Send to phone with VOkit**. This accepts multiple files and queues them until the phone connects. You can also run `vokit /path/to/file1 /path/to/file2` or choose multiple files with the app’s Import button. Files go to the phone’s Download folder with their original names; name collisions request a renamed copy.

Run the installer as your regular desktop user. It uses sudo for dependencies and installation, builds as your user, and installs the application under `/usr/local`. Webcam setup is enabled by default; use `--without-webcam` if you do not need a virtual camera. Debian/Ubuntu needs a release with Go 1.23+, GTK 4.16+ and libadwaita 1.6+ packages. Dependency installation needs internet access; no distro repository is added.

Webcam installation requests v4l2loopback DKMS and headers matching the running kernel, preserves existing configuration, and grants the active local session access only to the **Vivo Phone** virtual camera through a udev `uaccess` rule. It attempts to load the module without stopping existing cameras. A reboot may be needed after kernel/header changes. Secure Boot may require your distribution's module-signing procedure. Phone consent and Wayland portal prompts require your interactive approval. The app never runs as root, changes audio defaults or edits the firewall.

For an Arch package built from this local source snapshot:

```sh
./scripts/build-arch-package.sh
cd build/arch-package
makepkg -si
```

The generated PKGBUILD has a checksum for the generated source archive; it does not download vendor installers. Install `v4l2loopback-dkms` and the matching kernel headers for webcam output. The package installs camera configuration and a scoped udev rule; reboot or load `v4l2loopback` after installation. Existing loaded drivers are not reconfigured automatically.

## Build

Build dependencies: Go 1.23+ and Make. Runtime dependencies: Python 3 with PyGObject, GTK 4.16+, libadwaita 1.6+, GStreamer with base/good/bad/libav plugins, `qrencode`, `wl-clipboard` on Wayland, and PipeWire tools for optional audio sharing. Install the GStreamer GTK4 sink (`gst-plugin-gtk4` on Arch, `gstreamer1.0-gtk4` on Debian/Ubuntu) for the media viewer. Screen and camera preview also include a bounded RGB fallback. Tests use Xvfb and xauth. ADB/scrcpy remain optional command-line fallbacks.

```sh
make build desktop
./build/vokit
make test
make test-desktop
make check-packaging
```

To stage an already built installation without privileges or host changes:

```sh
./install.sh --destdir /tmp/vivo-package-stage
```

`make install-app PREFIX="$HOME/.local"` installs application files for one user; it does not configure the kernel camera module. The GTK launchers and Go CLI live in `bin`; Python frontend modules live in `lib/vivo-office-kit`. Explicit Quit stops sessions owned by the desktop; hiding the window does not.

Both the installer and `make install-app` register **VOkit** (Chinese desktops: **VOkit 手机互联**) in the applications menu with the green computer-and-phone icon. Search for `VOkit`, `vivo`, `互传`, or `投屏`, then pin it to your dock if desired. The launcher uses the installed executable's absolute path, so a user installation also works without adding `~/.local/bin` to the desktop's PATH. Nine transparent icon sizes cover menus, docks and high-DPI displays; the launcher ID matches the GTK application for window grouping. Installation refreshes desktop/icon caches when available. Staged package installs skip host cache updates; Arch packages use the system's desktop/icon cache hooks.

The initial Go build downloads the pinned `github.com/coder/websocket` dependency. GTK, libadwaita, PyGObject and GStreamer are system dependencies; no pip downloads are required. See `THIRD_PARTY_NOTICES` for dependency notices. No project redistribution license has been selected yet; select one before distributing a public release. Vendor binaries, installers, captured sessions and research evidence are excluded from the application source package.
