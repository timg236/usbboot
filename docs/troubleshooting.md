# Troubleshooting

This section describes how to diagnose common `rpiboot` failures for Compute Modules. 

## Product Information Portal
The [Product Information Portal](https://pip.raspberrypi.com/) contains the official documentation for hardware revision changes for Raspberry Pi computers.
Please check this first to ensure that the software is up to date.

## Hardware
* Inspect the Compute Module pins and connector for signs of damage and verify that the socket is free from debris.
* Check that the Compute Module is fully inserted.
* Check that `nRPIBOOT` / EMMC disable is pulled low BEFORE powering on the device.
   * On CM4 and CM5, if the USB cable is disconnected and the nRPIBOOT jumper is fitted then the green LED should be OFF. If the LED is on then the ROM is detecting that the GPIO for nRPIBOOT is high.
* Remove any non-essential peripherals from both the device being programmed.
* Verify that the red power LED IO board switches on when the board is powered.
* Use a short, high-quality USB cable.

### Raspberry Pi 5, CM5, Pi500 and Pi500+
* Always use a powered USB-C hub between the `rpiboot` host and the device being flashed. The `rpiboot` host computer's USB port might not be able to provide enough current to power the device being programmed.
* If the `rpiboot` host is a Pi 5 then you should also use a 5A capable USB-C power supply.

### Raspberry Pi 5, Pi 500 and Pi 500+
* Press, and hold the power button before supplying power to the device.
* Release the power button immediately after supplying power to the device.

## Software
The recommended host setup is Raspberry Pi with Raspberry Pi OS. Alternatively, most Linux X86 builds are also suitable but you could encounter hardware differences with USB ports on the host computer.

* Update to the latest software release using `apt update rpiboot` or download and rebuild this repository from Github.
* Run `rpiboot -v | tee log` to capture verbose log output. 

### Boot flow
The `rpiboot` system runs in multiple stages. The ROM, bootcode.bin, the VPU firmware (start.elf) and for the `mass-storage-gadget64` or `rpi-imager` a Linux initramfs. Each stage disconnects the USB device and presents a different USB descriptor. Each stage will appears as a new USB device connect in the `dmesg` log.

See also: [EEPROM boot flow](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#eeprom-boot-flow)

### bootcode.bin
Be careful not to overwrite `bootcode.bin` or `bootcode4.bin` with the executable from a different subdirectory. The `rpiboot` process simply looks for a file called `bootcode.bin` (or `bootcode4.bin` on BCM2711). However, the file in `recovery`/`secure-boot-recovery` directories is actually the `recovery.bin` EEPROM flashing tool.

## Diagnostics
* Monitor the Linux `dmesg` output and verify that a BCM boot device is detected immediately after powering on the device. If not, please check the `hardware` section.
* Check the green activity LED. On CM4 and CM5 this is activated by the software bootloader and should remain on. If not, then it's likely that the initial USB transfer to the ROM failed.
* On Compute Module 4 connect a HDMI monitor for additional debug output. Flashing the EEPROM using `recovery.bin` will show a green screen and the `mass-storage-gadget64` enables a console on the HDMI display.
* If `rpiboot` starts to download `bootcode4.bin` but the transfer fails then can indicate a cable issue OR a corrupted file. Check the hash of `bootcode.bin` file against this repository and check `dmesg` for USB error.
* If `bootcode.bin` or the `start.elf` detects an error then [error-code](https://www.raspberrypi.com/documentation/computers/configuration.html#led-warning-flash-codes) will be indicated by flashing the green activity LED.
* Add `uart_2ndstage=1` to the `config.txt` file in `msd/` or `recovery/` directories to enable UART debug output.
* After booting, the `mass-storage-gadget` provides login consoles on HDMI, boot UART and a virtual USB-CDC UART. Try connecting to this console (user `root` empty-password) to see if the device has booted and examine the storage devices.
