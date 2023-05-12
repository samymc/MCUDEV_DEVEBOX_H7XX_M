# MCUDEV_DEVEBOX_H7XX_M
MCUDev DevEBox STM32H7XX_M configuration for using Micropython 1.6.

MCUDev DevEBox is a development board that comes in two versions:
- STM32H750VBT6 - 128K flash
- STM32H743VIT6 - 2048K flash

![image](https://github.com/samymc/MCUDEV_DEVEBOX_H7XX_M/assets/132628432/1c5317af-1c86-4487-823f-42240fb7e5c2)

It can be obtained on Aliexpress [here](https://www.aliexpress.us/item/3256801381147898.html?spm=a2g0o.order_list.order_list_main.66.21ef1802bEAxVj&gatewayAdapt=glo2usa4itemAdapt).
According to the documentation found, the corresponding procedure is explained [here](https://github.com/mcauser/MCUDEV_DEVEBOX_H7XX_M).
Unfortunately, in the v1.20.0 version of Micropython, the previous procedure no longer works. Therefore, I will explain an alternative way to install Micropython on the board.
In the "micropython-bin" folder of this repository, there is a compiled version of Micropython, which you can also find [here](https://github.com/mcauser/MCUDEV_DEVEBOX_H7XX_M/issues/11).

## Linux:
### Flashing via DFU
This board can be flashed using DFU. To put the board in DFU mode, disconnect the USB. Set BOOT0 to ON by connecting the BT0 pin to 3V3, and then reconnect the USB.
First, make sure you have the ["dfu-util"](https://dfu-util.sourceforge.net/dfuse.html) (Device Firmware Upgrade Utilities) tool installed:

<code>$ sudo apt-get install dfu-util</code>

Then, with the command <code>dfu-util --list</code>, you can view the MCUDev DevEBox board:

<code>dfu-util 0.9
Copyright 2005-2009 Weston Schmidt, Harald Welte and OpenMoko Inc.
Copyright 2010-2016 Tormod Volden and Stefan Schmidt
This program is Free Software and has ABSOLUTELY NO WARRANTY
Please report bugs to http://sourceforge.net/p/dfu-util/tickets/
<br>
Found DFU: [0483:df11] ver=0200, devnum=2, cfg=1, intf=0, path="1-1", alt=1, name="@Option Bytes   /0x5200201C/01*128 e", serial="200364500000"
Found DFU: [0483:df11] ver=0200, devnum=2, cfg=1, intf=0, path="1-1", alt=0, name="@Internal Flash   /0x08000000/16*128Kg", serial="200364500000"
</code> 

Observe the "alt" attribute, which will be used for the configuration of dfu-util.
Then, navigate to the "micropython-bin" folder:

<code>$ cd micropython-bin</code>

Since we have the "firmware.dfu" file in the "micropython-bin" folder, we will use it to upload the firmware. For this case, alt=0 indicates the memory address, and "firmware.dfu" is the file to upload. Use the following command:

<code>$ dfu-util -a 0 -D firmware.dfu
</code>

If all the procedures were executed correctly, the firmware should have been uploaded on the board. Then disconnect the USB, set BOOT0 to OFF by connecting the BT0 pin to ground, and then reconnect the USB.
Verify the serial port to connect to the board:

<code>$ cd /dev
</code>
  
Then, you can use the <code>screen</code> command:

<code>$ screen /dev/ttyACM0 115200
</code>

However, I prefer to use the Python Package ["ampy"](https://pypi.org/project/adafruit-ampy/).
Using ampy should work as follows:

<code>$ ampy --port ttyACM0 ls
/flash
</code>

As an example, we can upload the file <code>/example/main.py</code> to the <code>/flash</code> folder:

<code>$ cd /example
$ ampy --port ttyACM0 put main.py
</code>
<br>

## Windows
The easiest way I found to [flash the firmware](https://blog.golioth.io/program-mcu-from-wsl2-with-usb-support/) to the board is by using WSL (Windows Subsystem for Linux) and [usbipd-win](https://github.com/dorssel/usbipd-win). This means that the Linux procedure is applied within WSL. Review previous Linux configuration for usbipd-win [here](https://github.com/dorssel/usbipd-win/wiki/WSL-support).

### Flashing via DFU:
After installing usbipd-win and WSL(Ubuntu), we proceed to share the USB device and list the devices as follows:
(In PowerShell as admin)

<code>PS \> usbipd.exe wsl list
<br>
BUSID  VID:PID    DEVICE                                                        STATE
3-1    1a2c:9ef4  USB Input Device                                             Not attached
3-3    093a:2510  USB Input Device                                             Not attached
4-4    0483:df11  STM Device in DFU Mode                                        Not attached
</code>

For this case, the address of our board in DFU mode is "4-4". Let's proceed to share the USB:
(The WSL terminal (Linux) must be open at the same time as PowerShell)

<code>PS \> usbipd wsl attach --busid=4-4
</code>

Done. From here, the procedure is the same as in Linux using WSL (Linux). Once the firmware has been uploaded, set BOOT0 to OFF and check which serial communication port was activated. Finally, use ampy in PowerShell to perform the same procedure explained for Linux.
