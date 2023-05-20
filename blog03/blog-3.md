This article records how I solved the compatibility problems of USBasp programmer.

I bought this cheap USBasp from an online store, hoping to use it to program AVR/Arduino. But there are some compatibility issues. I finally solved them, if you have a similar problem, I hope this article can help you.

# USBasp Programmer Compatibility Problems

## Background

The USBasp programmer is an essential tool/accessory for embedded system engineers/firmware developers. It is a USB ICSP (In-Circuit Serial Programmer) that allows developers to easily upload firmware/bootloader on AVR microcontrollers. Unlike conventional serial programmers such as USB-TTL converters, the USBasp programmer stands out by utilizing an atmega88 (or atmega8) microcontroller without the need for a dedicated chip. It relies solely on a firmware-based USB driver, eliminating the requirement for a specialized USB controller.

![avatar](https://raw.githubusercontent.com/WaylonMao/blogImages/main/blog03/usbasp.JPG)

This USBasp programmer was purchased from an online store. It is a cheap USBasp programmer built with Atmega 8A, and it works fine. However, there are two compatibility issues with the Arduino IDE and avrdude.

### Problem 1

- Driver problem: Arduino IDE cannot recognize the USBasp programmer. You can choose the programmer in the Arduino IDE, but this USBasp programmer does not work.

![avatar](https://raw.githubusercontent.com/WaylonMao/blogImages/main/blog03/arduino-usbasp.png)

### Problem 2

- Warning message: There are always annoying warning messages after programming in the Arduino IDE:

```
avrdude: warning: cannot set sck period. please check for usbasp firmware update.
```

Following screenshot shows the warning messages after buring bootloader. It did upload the bootloader successfully, but the warning messages are annoying.

![avatar](https://raw.githubusercontent.com/WaylonMao/blogImages/main/blog03/warning.png)

## Research

Driver problem is tricky. First, the driver provided by the online store is old. It is not compatible with the latest version of avrdude. Second, some users may find that Windows can recognize the USBasp programmer, but avrdude and Arduino IDE cannot work with it.

I googled the driver problem, and found Windows may use another driver file for the USBasp programmer, but this driver is not compatible with avrdude. The solution is to replace the driver file with the correct one.

About the waning issue, as it said, this problem was caused by the firmware of the USBasp programmer. The firmware of the USBasp programmer maybe outdated, or not built by [Thomas Fischl](https://www.fischl.de/contact.html), the original author of the USBasp programmer. The solution is to update the firmware of the USBasp programmer.

## Solution for Driver Problem

### Download the Zadig

[Zadig](https://zadig.akeo.ie/) is a Windows application that installs generic USB drivers, such as WinUSB, libusb-win32/libusb0.sys or libusbK, to help you access USB devices.

You may click [here](https://zadig.akeo.ie/downloads/) and choose the latest version of Zadig to download.

### Replace the Driver

1. You can put `zadig.exe` in any folder you want, and then run it.

2. Connect the USBasp programmer to your computer, and then select the USBasp programmer from dropdown list in Zadig.

3. Choose `libusbK` in the driver box, and then click `Reinstall Driver`.

![avatar](https://raw.githubusercontent.com/WaylonMao/blogImages/main/blog03/usbasp-driver.png)

### Reason

It seems that Windows and the driver provided by seller is based on `libusb-win32`. Its certificate has expired and the library has now been superseded by libusbK.

Others say it's a 64-bit Windows issue. I haven't looked into these reasons too closely. In particular, there are many similar products on the market, although the appearances are very similar.

I believe replacing the driver with Zadig will solve most of the driver issues. Also a reminder that when using **a different USB port**, it may be necessary to **re-use the Zadig**.

## Solution for Warning Message

### Tools

To program the USBasp programmer, we need another programmer. There is no resone to buy another programmer, so I decided to use the Arduino UNO board as the programmer.

This is not my first time to use Arduino-ISP to program other AVR/Arduino boards. This is a very useful feature of Arduino. Save money, and save time.

You can find more information about this feature on the [Arduino ISP](https://docs.arduino.cc/built-in-examples/arduino-isp/ArduinoISP). This ISP feature is not only for burning bootloader, but also for programming other AVR/Arduino boards.

And also, you should also set up the environment for avrdude. You can find more information about avrdude on the [avrdude](https://www.nongnu.org/avrdude/) website. You can download it, unzip it, and then add the path of the `avrdude.exe` to the environment variable `PATH`.

### Upload Arduino ISP Sketch

1. Open the Arduino IDE, and then open the `ArduinoISP` sketch from `File > Examples > 11.ArduinoISP > ArduinoISP`.

![avatar](https://raw.githubusercontent.com/WaylonMao/blogImages/main/blog03/arduino-isp.png)

2. Select the board from `Tools > Board`.

3. Select the port from `Tools > Port`.

4. Upload the sketch to the Arduino board.

5. Disconnect the Arduino board from the computer.

### Wiring

The wiring is very simple. Connect the Arduino board to the USBasp programmer as follows:

| Arduino UNO | USBasp |
| ----------- | ------ |
| PIN 10      | RESET  |
| PIN 11      | MOSI   |
| PIN 12      | MISO   |
| PIN 13      | SCK    |
| 5V          | VCC    |
| GND         | GND    |

![avatar](https://raw.githubusercontent.com/WaylonMao/blogImages/main/blog03/arduino-usbasp.jpeg)

### Upadae Firmware

1. Prepare the firmware file. You can download the latest firmware from [here](https://www.fischl.de/usbasp/).

> In my case, I downloaded the `usbasp.atmega8.2011-05-28.tar.gz` file. And the path of the firmware file is `usbasp.atmega8.2011-05-28\bin\firmware\usbasp.atmega8.2011-05-28.hex`.

> [nerdralph](https://github.com/nerdralph) also developed an improved version of the firmware, you can try it if you want. You can find it [here](https://github.com/nerdralph/usbasp). I tested 1.06, and it works as well as the Thomas's firmware.

2. **Important!!!** Change the jumper on the USBasp programmer to `Update` mode. (In my case, it is JP1.)

3. Connect the Arduino board to the computer.

4. Check the `PORT` or `COM` number of the Arduino board. You can find it from `Device Manager`.

![avatar](https://raw.githubusercontent.com/WaylonMao/blogImages/main/blog03/check-port.png)

5. Open the command line or PowerShell, and then enter the following command to backup the firmware of the USBasp programmer. (It is recommended to backup the firmware before updating the firmware.)

```bash
# Change the COM number to yours, here is COM4.
avrdude -c arduino -p m8 -P COM4 -b 19200 -U flash:r:flash.hex:i
```

> This will generate a `flash.hex` file, which contains the current firmware of the USBasp programmer. You can use it to restore the firmware if you want.

6. Enter the following command to update the firmware of the USBasp programmer.

```bash
# Change the COM number to yours, here is COM4.
avrdude -c arduino -p m8 -P COM4 -b 19200 -U flash:w:usbasp.atmega8.2011-05-28.hex:i
```

> If the firmware file is in another folder, you should add the path of the firmware file to the command.

The following screenshot shows the output of the command line. It includes the backup and update process.

![avatar](https://raw.githubusercontent.com/WaylonMao/blogImages/main/blog03/backup-update.png)

7. **Important!!!** Change the jumper to 3.3V or 5V. You also can remove the jumper for no power supply.

8. Disconnect the Arduino board from the computer. And remove the wiring.

### Testing

After updating the firmware, the warning message is gone. And the USBasp programmer works fine with Arduino IDE and avrdude.

The following screenshot shows the result of burning bootloader by USBasp without warning message.

![avatar](https://raw.githubusercontent.com/WaylonMao/blogImages/main/blog03/test.png)

## Summary

After updating the firmware, you maybe need run Zadig again to replace the driver. And also, you should also set up the environment for avrdude. I wrote the driver issue first, because without fixing it, you won't see the warning message.

Through this troubleshooting, I learned more hardware and software knowledge. Also learned a lot about the history of AVR tools. When I was in junior school, I needed to program an AVR chip for my science project. At that time, I searched the circuit diagram online, bought an RS-232 plug and some components, and DIYed my own programming line. Because programmer was expensive at that time. Right now, it is very convenient to buy a cheap USBasp programmer online. It is a good tool to learn AVR/Arduino. Especially you want to learn bare metal programming, like me.

I hope this article can help you. If you have any questions, please email me or connect me on LinkedIn: [https://www.linkedin.com/in/weilong-mao/](https://www.linkedin.com/in/weilong-mao/)

## References

Thomas Fischl, USBasp - USB programmer for Atmel AVR controllers, [https://www.fischl.de/usbasp/](https://www.fischl.de/usbasp/)

Arduino, Arduino ISP, [https://docs.arduino.cc/built-in-examples/arduino-isp/ArduinoISP](https://docs.arduino.cc/built-in-examples/arduino-isp/ArduinoISP)

AVRDUDE - User Manual, [https://www.nongnu.org/avrdude/user-manual/avrdude.html](https://www.nongnu.org/avrdude/user-manual/avrdude.html)

USBasp Firmware Update, [http://www.bitbanging.space/posts/usbasp-firmware-update](http://www.bitbanging.space/posts/usbasp-firmware-update)

## Like this article?

Please follow my Github: [William Mao](https://github.com/WaylonMao)

Connect with me on my LinkedIn: [Weilong Mao](https://www.linkedin.com/in/weilong-mao/)
