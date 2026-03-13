# Steps to reprocude serial recovery error

This HowTo documents the steps to reproduce the error when executing serial update via mcuboot on stm32n6570_dk board

## Environment
 `west.yml` is updated in order to point to [my mcuboot fork](https://github.com/StefanRB/mcuboot/tree/n6_serial_recov) which contains `stm32n6570_dk_stm32n657xx_fsbl.conf` and 
`stm32n6570_dk_stm32n657xx_fsbl.overlay`

## Build flash shell example

```sh
west build -p always -b stm32n6570_dk -d build_mcuboot_flashshell --sysbuild zephyr/samples/drivers/flash_shell/
```
```sh
west flash -d build_mcuboot_flashshell/
```

## Get "Division by zero error"
open serial console and reset board
```
---- Opened the serial port /dev/ttyACM0 ----
Flash shell sample
*** Booting Zephyr OS build a5d4626b8c16 ***
uart:~$ 
```
list partitions
```
uart:~$ flash partitions 
partition@0                      mcuboot         0x00000000 64 KiB
partition@10000                  image-0         0x00010000 1536 KiB
partition@210000                 image-1         0x00210000 1536 KiB
partition@410000                 storage         0x00410000 64 KiB
uart:~$ 
```
get page info from image (ends up in "division by zero" error, which halts system)
```
uart:~$ flash page_info ospi-nor-flash@0 0x10000

[00:04:13.227,000] <err> os: ***** USAGE FAULT *****
[00:04:13.228,000] <err> os:   Division by zero
[00:04:13.228,000] <err> os: r0/a1:  0x340000e0  r1/a2:  0x00000000  r2/a3:  0x00000000
[00:04:13.228,000] <err> os: r3/a4:  0x00010000 r12/ip:  0x00000000 r14/lr:  0x70020781
[00:04:13.228,000] <err> os:  xpsr:  0x21000000
[00:04:13.228,000] <err> os: Faulting instruction address (r15/pc): 0x700207c0
[00:04:13.228,000] <err> os: >>> ZEPHYR FATAL ERROR 30: Unknown error on CPU 0
[00:04:13.228,000] <err> os: Current thread: 0x34000738 (shell_uart)
[00:04:13.250,000] <err> os: Halting system
```

## 5. serial recovery

Enter mcuboot:
* Press and hold "User1" Button
* Press and release "Reset" Button
* Release "User1" Button

use [mcumgrclient](https://github.com/vouch-opensource/mcumgr-client) to list images:
```sh
./mcumgr-client -d /dev/ttyACM0 list
mcumgr-client 0.0.7, Copyright © 2024 Vouch.io LLC

23:17:17 [INFO] send image list request
response: {
  "images": [
    {
      "image": 0,
      "slot": 0,
      "version": "0.0.0",
      "hash": "162831f72523a9f3e37569b473789fd11546e2c79160dcdf2d29e1727b02205e",
      "bootable": false,
      "pending": false,
      "confirmed": false,
      "active": false,
      "permanent": false
    }
  ]
```
do serial update (with same application as flashed before)
```sh
./mcumgr-client -d /dev/ttyACM0 upload ~/zephyr_sk/build_flashell/zephyr/zephyr.signed.bin
mcumgr-client 0.0.7, Copyright © 2024 Vouch.io LLC

23:21:42 [INFO] upload file: /home/user/zephyr_sk/build_flashshell/zephyr/zephyr.signed.bin
23:21:42 [INFO] flashing to slot 1
23:21:42 [INFO] 92704 bytes to transfer
  [00:00:23] [=======================================================================================================================] 90.53 KiB/90.53 KiB (0s)23:22:06 [INFO] upload took 23s
  ```
Press "Reset" Switch to reboot 
  * shell is not launched when opening serial port

Enter mcuboot:
* Press and hold "User1" Button
* Press and release "Reset" Button
* Release "User1" Button

use [mcumgrclient](https://github.com/vouch-opensource/mcumgr-client) to list images:
```sh
./mcumgr-client -d /dev/ttyACM0 list
mcumgr-client 0.0.7, Copyright © 2024 Vouch.io LLC

23:26:03 [INFO] send image list request
response: {
  "images": []
```
  