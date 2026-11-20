# RpiGuiV2

Hardware accelerated **gui-v2** on a **Raspberry Pi 2, 3 or Zero 2 W** running
Venus OS, as a [SetupHelper](https://github.com/kwindrem/SetupHelper) package.

> **Status: beta.** Tested on a Raspberry Pi 2 Model B & Pi 3 (see issue report) with a 1080p HDMI display
> on Venus OS v3.85: gui-v2 runs on the GPU (`VC4 V3D 2.1`). Not yet tested on a
> RPI, a Zero 2 W or with the official 7" touchscreen. Keep a copy of your SD
> card or a spare card, and report results in
> [venus issue 1305](https://github.com/victronenergy/venus/issues/1305).

## Why

On these Raspberry Pis Venus OS renders gui-v2 in software, which keeps the CPU
busy. Three things are missing for the VideoCore IV GPU to be used:

1. `/u-boot/config.txt` does not load the vc4 KMS display driver
   (`dtoverlay=vc4-kms-v3d`), and that overlay isn't on the boot partition.
2. The Venus OS mesa build has no `vc4` GPU driver.
3. Qt is configured for the plain `linuxfb` framebuffer.

## What the package does

| Change | Details |
|---|---|
| `/u-boot/config.txt` | adds `dtoverlay=vc4-kms-v3d`, `display_auto_detect=1` and `disable_overscan=1` between `#### RpiGuiV2 begin/end` markers |
| `/u-boot/overlays/` | adds `vc4-kms-v3d.dtbo` and `vc4-kms-dsi-7inch.dtbo` (official 7" touchscreen) |
| `/usr/lib/dri/vc4_dri.so` | mesa 24.0.7 vc4 driver, built for Venus OS (see `build/`) |
| `/etc/profile.d/qt6.sh`, `/etc/qt-kms.conf` | Qt `eglfs` on the vc4 KMS device, falling back to `linuxfb` when it isn't there |
| `/lib/udev/display-hotplug` | patch: handle DRM devices with more than one connector (vc4 has HDMI, DSI and Writeback) |
| `/u-boot/cmdline.txt` | only with the `rotate180` option: adds `video=<connector>:<mode>,rotate=180` for HDMI and the 7" display, using the mode the firmware passes |
| gui-v2 | installed from the **Victron package feed** matching your firmware, in the background by the `RpiGuiV2Installer` service |
| `start-gui` service | replaces the gui-v1 only `gui` service, so gui-v1 or gui-v2 can be selected in the settings (like on the Raspberry Pi 5) |

Uninstalling restores all of it. gui-v2 is not included in this package: it is
downloaded from Victron's feed, so the first install needs an internet
connection. The download refuses to run if the feed would upgrade any package of
your firmware (the feed only holds the latest release).

## Requirements

- Raspberry Pi 2, 3 (B, B+, A+) or Zero 2 W: Venus OS machine `raspberrypi2`.
  The Raspberry Pi 4 and 5 need a different GPU driver and are refused.
- Venus OS v3.80 or later with mesa 24.0.7 (checked during install).
- [SetupHelper](https://github.com/kwindrem/SetupHelper) installed.

## Install

In **Settings → PackageManager → Inactive packages → new**, enter package name
`RpiGuiV2`, GitHub user `Born2Ski`, branch `main`, then download and install it.

Or from the command line:

```sh
wget -qO - https://github.com/Born2Ski/RpiGuiV2/archive/main.tar.gz | tar -xzf - -C /data
rm -rf /data/RpiGuiV2 && mv /data/RpiGuiV2-main /data/RpiGuiV2
/data/RpiGuiV2/setup
```

Reboot when asked. gui-v2 then downloads in the background; follow it with
`tail -F /var/log/RpiGuiV2Installer/current`. When it's done the display
switches to gui-v2. A firmware update removes the changes on the root file
system; PackageManager reinstalls the package and gui-v2 is downloaded again.

## Choosing gui-v1 or gui-v2

The package selects gui-v2 when installed (the command line setup asks). Switch
with the existing GUI setting, or:

```sh
dbus -y com.victronenergy.settings /Settings/Gui/RunningVersion SetValue 2   # 1 = gui-v1
svc -t /service/start-gui
```

gui-v1 is used whenever gui-v2 isn't installed yet or no display is connected
through the vc4 KMS driver.

## Rotating the display

With the KMS driver the firmware rotation settings (`display_rotate`,
`display_hdmi_rotate`, `lcd_rotate`) no longer apply. The package can rotate the
display by **180 degrees** through the kernel command line instead (the
command line setup asks), with touch input rotated to match:

```sh
touch /data/setupOptions/RpiGuiV2/rotate180     # rm to turn it off again
/data/RpiGuiV2/setup install
reboot
```

The mode of the display is written into the entry, so after changing to a
display with another resolution run `/data/RpiGuiV2/setup install` again.

90 and 270 degrees are not possible: the Pi's display hardware can only flip
the picture, and gui-v2 has no rotation setting of its own.

Tested with an HDMI display (no touch). Touch rotation is untested.

## Check it uses the GPU

```sh
ls /dev/dri/by-path/          # should list platform-soc:gpu-card
svc -d /service/start-gui
. /etc/profile.d/qt6.sh
QSG_INFO=1 /opt/victronenergy/gui-v2/venus-gui-v2 2>&1 | grep -i renderer   # VC4 V3D 2.1
svc -u /service/start-gui
```

## Things to know

- **Display settings:** with the KMS driver, firmware display options such as
  `hdmi_cvt`, `hdmi_mode` and `lcd_rotate` (e.g. from RpiDisplaySetup) no longer
  apply. HDMI displays need a valid EDID; the official 7" touchscreen is detected
  automatically and its touch input moves to the `edt-ft5x06` driver.
- **Overscan:** `disable_overscan=1` removes the black border the firmware
  otherwise adds under KMS. On a TV that cuts off the edges, keep the overscan
  with `touch /data/setupOptions/RpiGuiV2/keepOverscan`, run
  `/data/RpiGuiV2/setup` again (install) and reboot.
- **SPI displays** that rely on `/dev/fb0` may end up on `/dev/fb1`.
- **Memory:** the vc4 overlay reserves 256 MB of CMA memory (still usable by
  Linux when the GPU doesn't need it).
- **License:** gui-v2 is Victron Energy software and may only be used in systems
  built around Victron Energy products.

## Files

- `setup`: SetupHelper setup script
- `installGuiV2`: background gui-v2 install, run by the `RpiGuiV2Installer` service
- `startGuiService/`: the `start-gui` service
- `FileSets/`: files installed or patched by SetupHelper
- `build/`: how `vc4_dri.so` was built
- `THIRD-PARTY-NOTICES.md`: licenses of the included binaries
