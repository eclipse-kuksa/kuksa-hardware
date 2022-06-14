# How to flash eMMC Compute Modules

Compute modules witout eMMC ("Lite" versions), boot from the SD card in the CANOPi socket.

Compute Modules with integrated eMMC flash _can not_ boot from an SD card (this is a CM4 hardware limitation), therefore you need to programm the integrated eMMC flash.

Basically the official documentation applies, which you can find [here](https://www.raspberrypi.com/documentation/computers/compute-module.html#flashing-the-compute-module-emmc), or in [Jeff Gerlings nice tutorial](https://www.jeffgeerling.com/blog/2020/how-flash-raspberry-pi-os-compute-module-4-emmc-usbboot
).

These tutorials assume you are using the official [Compute Module 4 IO Board](https://www.raspberrypi.com/products/compute-module-4-io-board/).
In a nutshell, you need to jumper the `nRPI_BOOT` / `disable eMMC boot` jumper connect your board to our computer where you ran a small piece of software, which basiclly turns you compute module into a (slow...) mass storage device exposing the eMMC flash.

So you are wondering, where to find the `nRPI_BOOT` signal to repeat this trickery directly on a CANOPi? 

Glad you asked:

You just need to put a 1.27mm Jumper over  the marked `nPIREBO` pins.

![nRPIREBO jumper](../hw_doc/img/nRPIREBO_jumper.png)

Do all the flashing as per the linked documentation above, possibly not forgetting to do the [neccessary Raspberry OS modifications](configure_raspberryos.md), remove the jumper and reboot.

Word of warning/friendly advice: Putting the jumper can be tricky and requires a steady hand. Using pincers is recommended. Obviously, only try to set/remove the jumper when the CANOPi is powered off.
