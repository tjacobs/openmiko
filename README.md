# OpenMiko

OpenMiko is custom open source firmware for cameras that use the Ingenic T20 chip.

These cameras include the Wyzecam V2 and Xiaomi Xiaofang.

## Features

- Support for RTSP and MJPEG
- SSH server
- ffmpeg
- mjpg-streamer

## Overview

This repository contains kernel and rootfs for cameras using Inegnic T20 SOC:

Ingenic T20X (128Mb DDR) | &nbsp;
:-- | --:
![Wyze Cam v2](doc/wyzecam_v2/img/wyzecam_v2.jpg) Wyze Cam V2 | 
![Xiaomi Dafang](doc/xiaomi_dafang/img/xiaomi_dafang.jpg) Xiaomi Dafang | ![Wyze Cam Pan](doc/WYZECP1/img/wyzecam_pan.jpg) Wyze Cam Pan

## Installation and Usage

Rename the firmware to `demo.bin` and put it in on the root of the microSD card.

[![Assets](doc/img/assets.png)](https://github.com/openmiko/openmiko/releases)

Power off the camera and insert the microSD card.

Hold the setup button, plug in your USB cable, keep holding the setup button for 1-2 seconds until the light is solid blue, then release the button.

After about 30 seconds you should get a flashing yellow LED which indicates the camera is working.

## Setting up the wifi

Wifi configuration is done via the sdcard.

On the sdcard create the directories `/config/overlay/etc`:

![Overlay Filesystem](doc/img/overlay_filesystem.png)

In the `etc` directory copy the file [`wpa_supplicant.conf`](https://github.com/openmiko/openmiko/blob/master/overlay_minimal/etc/wpa_supplicant.conf). Edit this file and plug in your wifi name and password.

Insert the sdcard into the camera and reboot. OpenMiko will copy this directory over to the `/config` partition (which is persistent flash storage). This method can also be used to overwrite other files. For example:

## Usage

The configuration by default provides 3 output streams:

- 1920x1080 H264
- 1920x1080 JPG over HTTP (MJPEG)
- 640x360 H264

The streams can be accessed using the following URLs:

- rtsp://IP:8554/video3_unicast
- rtsp://IP:8554/video5_unicast

- http://IP:8080/?action=stream


## Settings

Settings can be changed by editing /etc/videocapture_settings.json. However the changes will not persist unless you write them to the flash. To ease saving these settings copy the file to `/config/overlay/etc/videocapture_settings.json`. The files in /config are mounted to the flash chip and will survive reboots.

```
"general_settings": {
	"flip_vertical": 0,			// 1 flips image along vertical axis, 0 disables
	"flip_horizontal": 0,		// 1 flips image along horizontal axis, 0 disables
	"show_timestamp": 1 		// 1 enables timestamp, 0 disables
},
```

### Writing Config Files

On boot it is possible to put files on the sdcard and have them copied permanently to the configuration storage area of the camera.

You should create a directory called `config` on the sdcard.

Inside this directory create more directories. In particular create `/config/overlay/etc` and any files you want to write to the camera. For example to change the wifi module that is loaded create a file `<SDCARD>/config/overlay/etc/openmiko.conf`.

In `openmiko.conf` copy the default one from https://github.com/openmiko/openmiko/blob/master/overlay_minimal/etc/openmiko.conf and change the line that says `WIFI_MODULE=8189fs` to `WIFI_MODULE=8189es`.


### Profile

```
0 - profile Constrained Baseline
1 - profile Main
2 - profile High
```

### Resetting the configuration

While the camera is started hold down reset button for at least 6 seconds. After 6 seconds the blue LED should turn on and pulse 3 heartbeats. The `/config` partition (which is mounted to the persistent flash memory itself) will be removed.

## Troubleshooting

If you can't seem to get up and running here are some things to check:

- Make sure you are using unix style line endings in wpa_supplicant.conf (this was fixed in a later release)
- Inside wpa_supplicant.conf the `psk` and `ssid` settings need to have double quotes around the string. For example `psk="password_keep_double_quotes"`
- Make sure the file does not have a `.txt` extension. The file should show up as `wpa_supplicant.conf`. `Not wpa_supplicant.conf.txt`
- The `config/overlay/etc` directory is *cAsE sEnsItiVe*. Make sure it is all lowercase.
- The MAC address does change when flashing from the one on the sticker. Check your router to see the new DHCP address.
- There are some reports that assigning a static IP / DHCP reservation does help the WyzeCam connect to the network
- Logs should appear on the sdcard if the system is properly booting
- Some WyzeCams have a buggy bootloader from the factory and won't flash anything. The only way around this is to flash a new bootloader.
- If you flashed the custom HD Dafang bootloader you will need to revert to the original one. A copy of the old WyzeCam V2 original bootloader is here: https://github.com/openmiko/openmiko/blob/master/stock_firmware/wyzecam_v2/wyzecam_v2_stock_bootloader.bin

