# Connect to Raspberry Pi using Serial/UART

Access your Raspberry Pi from your laptop or computer without needing to hook it up to a screen, find an extra keyboard and mouse, or configure networking.

Sources:

* [Attaching to a Raspberry Pi's Serial Console (UART) for debugging by Jeff Geerling](https://www.jeffgeerling.com/blog/2021/attaching-raspberry-pis-serial-console-uart-debugging)
* [USB Ethernet Gadget: A Beginner's Guide by thagrol](https://github.com/thagrol/Guides/blob/main/ethernetgadget.pdf)

<br />

## Prerequisites

* USB-to-serial adaptor, such as the [Raspberry Pi Debug Probe](https://thepihut.com/products/raspberry-pi-debug-probe) or (cheaper) alternatives e.g.

  * [USB to UART serial console cable from Pimoroni](https://shop.pimoroni.com/products/usb-to-uart-serial-console-cable?variant=288389664)
  * [USB to TTL Serial Cable from PiHut](https://thepihut.com/products/usb-to-ttl-serial-cable-debug-console-cable-for-raspberry-pi)
  * [Adafruit USB to TTL Serial Cable](https://www.adafruit.com/product/954)
  * [USB to RS232 TTL Serial Interface Cable from Hobby Components](https://hobbycomponents.com/usb-interface/618-usb-to-rs232-ttl-serial-interface-cable)
  * [Silicon Labs CP2102 3.3V USB to UART Serial Interface Mmodule Adaptor](https://hobbycomponents.com/usb-interface/396-usb-to-uart-serial-interface-module-adaptor)

<br />

## How to setup and connect

1. Open your Raspberry Pi's boot config file with your preferred text editor e.g.

   ```bash
   nano /boot/firmware/config.txt
   
   # Or if you have bullseye or an older version of Raspberry Pi OS installed:
   nano /boot/config.txt
   ```

   The boot config file can be accessed in any of the following ways:

   **If you are able to log into your Raspberry Pi:**

   1. Log into your Raspberry Pi.
   2. Open the boot config file located at `/boot/config.txt` (bullseye and older) or  `/boot/firmware/config.txt` (bookworm and newer).

   **If you are unable to log into your Raspberry Pi** e.g. no display, keyboard and mouse attached, or SSH is not enabled, or not connected to your network:

   1. Power down your Raspberry Pi and unplug it from the power supply.
   2. Remove the drive on which Raspberry Pi OS is installed i.e. SD card or USB drive or SSD drive.
   3. Plug the drive into another computer - Mac, Windows or Linux - and this should mount the drive with the name `bootfs`.
   4. Open the file located at `/bootfs/config.txt`.

2. Enable serial/UART connection by adding the following lines at the bottom of the boot config file.

   ```bash
   [all]
   enable_uart=1         # Enable shell messages on the serial connection
   ```

   See https://www.raspberrypi.com/documentation/computers/config_txt.html#enable_uart

   **If you are using Raspberry Pi 5 AND connecting your USB to serial adaptor to your Raspberry Pi's GPIO pins**, you will need to also append the following lines.

   ```bash
   [pi5]
   dtparam=uart0=on      # Enable UART0/ttyAMA0 on GPIO 14 & 15
   dtparam=uart0_console # Enable UART0/ttyAMA0 on GPIO 14 & 15 and make it the console UART
   ```

   Raspberry Pi 5 models have built-in UART 3-pin connector and the access to serial console defaults to those pins. To use UART via GPIO instead, the default UART pins need to be diverted to GPIO 14 and 15 (pins 8 and 10 on the 40-pin header) with the above additional settings.

3. Save and close the boot config file. If you edited the file on another computer, eject your SD card/USB drive/SSD drive and put it back into your Raspberry Pi.

4. Make sure your Raspberry Pi is shutdown and unplugged from the power supply. From here onwards, Raspberry Pi should not be switched on or plugged to the power supply until the very last step.

5. Connect the USB to serial adaptor to your Raspberry Pi via pins as follows (for visual guide, see the Fritzing on [Jeff Greeling's notes](https://www.jeffgeerling.com/blog/2021/attaching-raspberry-pis-serial-console-uart-debugging)):

   | USB to serial adaptor | Raspberry Pi        | Raspberry Pi GPIO | Raspberry Pi Pin |
   | --------------------- | ------------------- | ----------------- | ---------------- |
   | GND (Ground)          | GND (Ground)        |                   | 6                |
   | RxD (Receive Data)    | TxD (Transmit Data) | 14                | 8                |
   | TxD (Transmit Data)   | RxD (Receive Data)  | 15                | 10               |

   **If you are using Raspberry Pi 5 AND Raspberry Pi Debug Probe**: You can use the 3-pin to 3pin cable and attach one end to the probe's UART connector and the other end to your Raspberry Pi's UART connector.

6. Connect the USB to serial adaptor to your computer via USB. This is the computer from which to access your Raspberry Pi's serial console.

7. Open a terminal window on your computer, run the following command to obtain the device id of your USB to serial adaptor.

   ```bash 
   ls /dev | grep usb
   
   # Expected output
   # Note that the device number following cu.usbserial- and tty.usbserial- may be 
   # different for you.
   cu.usbserial-0001
   tty.usbserial-0001
   ```

8. Staying in the terminal window on your computer, start a serial console session using the Linux `screen` command or [CoolTerm](https://freeware.the-meiers.org/) or [PuTTY](https://www.putty.org/) (for the latter two, please refer to the respective documentation on 'Connecting to Serial Port') e.g.

   ```bash
   screen /dev/tty.usbserial-<YOUR DEVICE NUMBER> 115200
   
   # Example
   screen /dev/tty.usbserial-0001 115200
   ```

   You may only see a cursor blinking and no other activity in your terminal/CoolTerm/PuTTY window, but don't worry, keep your session alive and do not close the terminal/CoolTerm/PuTTY window.

   **A tip from [marty](https://github.com/martysteer): "I also removed the 'quiet' keyword/line from the cmdline.txt\*, which I think is what makes the boot up messages appear on console."**
   \* /boot/firmware/cmdline.txt (bookworm and newer) or /boot/cmdline.txt (bullseye or older) on Raspberry Pi OS; or /bootfs/cmdline.txt on SD card/USB drive/SSD drive.

9. Finally, connect your Raspberry Pi to the power supply and boot it up. Within a minute or so, you should see a login prompt in your terminal/CoolTerm/PuTTY window. Enter your username and press the enter key, then your password and press the enter key... you should now be logged into our Raspberry Pi and be able to run commands like `sudo raspi-config`.

