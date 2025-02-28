This is my fork of the Greenjay firmware. [Greenjay (link to parent repo)](https://github.com/bird-sanctuary/greenjay/) firmware is a derivative of [BLHeli_S](https://github.com/bitdump/BLHeli) and [Bluejay](https://github.com/bird-sanctuary/bluejay/) firmware. It runs on small ESCs to drive electric motors.

Brushless motor ESCs (electronic speed controllers) that are running BLHeli_S firmware can be converted to run brushed motors. This is great because these ESCs are small, are cheap, they can handle a lot of current, and they can be easily replaced if they break. Here are some examples size comparisons, along with cost and capability:

![](doc/imgs/greenjay_sizecompare.jpg)

I need something like this to drive the latest and greatest brushed motors in combat robotics, such as the [JCR Dartbox Viper](https://justcuzrobotics.com/products/dartbox-squared-drive), which is a 22mm gearboxed nerf gun motor with a hilariously high stall current rating of 12A. The next upgrade is the [JCR Dartbox Dragon](https://justcuzrobotics.com/products/dartbox-squared-drive?variant=45099914887477) that can suck down 18A.

[![](doc/imgs/jcr_dartbox_viper.jpg)](https://justcuzrobotics.com/products/dartbox-squared-drive)

Caveats
===

The current rating specified for a brushless ESC won't be true when it is used in brushed mode. Brushless motors have three winding phases, and so brushless ESCs have three half-bridges (half-bridges are a pair of MOSFETs). All of the half-bridges are used for brief moments while the motor rotates, sharing the load. But in brushed mode, two of those MOSFETs will be used 100% of the time, not sharing the load. So the current rating will be lower when in brushed mode. **The current rating is reduced to about 43% of the original stated rating.**

The highest claimed current rating of a BLHeli_S ESC I've found is 45A, which means in brushed mode, it can handle about 19A. They are still quite cheap and small.

If you need more current, BLHeli32 ESCs can also be converted to run in brushed mode, but it would require [AM32 firmware](https://github.com/am32-firmware/AM32/), not Greenjay.

Other disadvantages: no BEC, no current limiting

Since this is a bit of an unsupported hack, there's a ton of steps involved, and you will need some tools.

INSTRUCTIONS
===

Tools Needed
===

 * RC radio system (a radio transmitter and a receiver), or a servo tester of some sort
 * USB linker for ESCs
   * example Amazon product: [USB Linker Compatible with BL32 BLS AM32 Brushless ESC Open Source Speed Control Programming](https://www.amazon.com/Linker-Compatible-Brushless-Control-Programming/dp/B0CCXGFSB3/ref=sr_1_2) (I personally use this one) ![](doc/imgs/usblinker.png)
   * [ESC Programmer USB-C (AM32 / BLHeli_S / BLHeli_32)](https://shop.pearlgrey.io/product/esc-programmer-1-2-usb-c-blheli_s-blheli_32) from Pearl Grey, a fellow robotics hobbyist. This product is based on an Arduino Nano. It is a bit more versatile and can handle 4 ESCs at once. You need to select the `Preassembled with Arduino Nano` option.
   * [USB Programmer for AM32 BlHeli ESCs](https://justcuzrobotics.com/products/usb-programmer-for-am32-blheli-blheli32-escs) sold by Just Cuz Robotics (JCR), who created the Dartbox motors I mentioned earlier.
   * example Amazon product: [BL32 USB Linker Brushless ESC Open Source Speed Control Programming](https://www.amazon.com/FLASH-HOBBY-Brushless-Control-Programming/dp/B0B6V274JB/ref=sr_1_1) made by Flash Hobby, using CPxxx SiLabs chipset
   * example Amazon product: [ESC PC Software Adapter USB Linker Programmer Update for BLHeli Firmware](https://www.amazon.com/ZHIPAIJI-Software-Programmer-Firmware-Multicopter/dp/B09TPFLGBJ/ref=sr_1_3)
   * If you don't have any of these, then run BLHeliSuite, and it has a `Make Interface` tab that shows you how to use an Arduino to act as a USB linker. You still have to buy an Arduino.
   * For generalized instructions about BLHeli_S, see [Oscar Liang's guide](https://oscarliang.com/connect-flash-blheli-s-esc/)
 * Some way of powering up the ESC
   * this could be just whatever battery that you wanted to use to power your project
 * A brushed motor to test with (or a multimeter), and misc items (wire, etc)

Step 0 - Prerequisites
===

Downloads:

 * [BLHeliSuite](https://github.com/4712/BLHeliSuite/releases/tag/16714903)

Install BLHeliSuite, and install the USB driver for whatever USB linker you have.

For your convenience:

 * [FTDI drivers https://ftdichip.com/drivers/](https://ftdichip.com/drivers/)
 * [SiLabs drivers https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers](https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers?tab=downloads)
 * [CH340 drivers https://learn.sparkfun.com/tutorials/how-to-install-ch340-drivers/all](https://learn.sparkfun.com/tutorials/how-to-install-ch340-drivers/all)

Step 1 - Reading Original Settings
===

Plug in the USB linker to your computer via the USB cable. Do **not** power on your ESC, but plug the ESC into the USB linker.

![](doc/imgs/usblinkerconnection.jpg)

Run BLHeliSuite.

Select the correct interface type, which should be `SiLabs BLHeli Bootloader (USB/Com)`.

![](doc/imgs/blhelisuite_selectinterface.png)

Select the correct COM port number that's associated with your USB linker, and click connect.

![](doc/imgs/blhelisuite_connect.png)

Then the software will wait for you to power on your ESC.

![](doc/imgs/blhelisuite_connectpwr.png)

At this point, power on your ESC. The wait dialog will disappear. Click on the "Read Setup" button.

![](doc/imgs/blhelisuite_clickreadsetup.png)

The screen will populate with data and a confirmation dialog will appear. The most important thing right now is to remember the identification code of this particular ESC.

![](doc/imgs/blhelisuite_rememberidcode.png)

![](doc/imgs/blhelisuite_idcodeexplained.png)

IMPORTANT: You must remember this identification! It is also useful for when you need to restore brushless mode.

Step 2 - Find Correct HEX File
===

In [the hex directory](hex), you will see a bunch of files:

![](doc/imgs/bunchofhexfiles.png)

Here's the file name being explained:

![](doc/imgs/hexfilenamesexplained.png)

Find the files that matches your ESC's identification. Do not worry about the phase pin combination for now. **Download all three**. Use the "download raw file" option to make sure it gets saved as a `*.HEX` file.

![](doc/imgs/downloadrawfile.png)

Step 3 - Flash
===

Inside BLHeliSuite, click on "Flash Other".

![](doc/imgs/blhelisuite_flashother.png)

Select the file you want to flash, in this case, pick the one that matches your ESC's identification and use `_1_` for the phase-pin-combination. (the other two combination numbers might be useful in step 6)

![](doc/imgs/blhelisuite_pickfile.png)

When you confirm your file selection, you will see a final warning dialog, click `YES` to continue.

![](doc/imgs/blhelisuite_clickyes.png)

The flashing process will begin and you will see the progress, and then a confirmation will appear when it is done.

![](doc/imgs/blhelisuite_flashprogress.png)

![](doc/imgs/blhelisuite_flashdone.png)

It may ask you to write settings, select `no`. We will need to change the settings before writing them.

Step 4 - Change Settings
===

Edit all the setting items to match the following screenshot, and then click `Write Setup`.

![](doc/imgs/blhelisuite_editallitems.png)

![](doc/imgs/blhelisuite_writeok.png)

Step 5 - Test
===

Now it is time to test it. Unplug the ESC from the USB linker, and plug the ESC into your RC receiver. Turn on your RC radio system.

![](doc/imgs/testingconnection.png)

Through trial and error, figure out which two out of the three solder-tabs are the correct solder-tabs to control a brushed motor. Wire up a brushed motor with two wires and experimentally connect them to solder-tabs.

![](doc/imgs/try3combos.png)

NOTE: you can do this test with a multimeter, getting both a strong positive and a strong negative voltage means you have found the correct combination

Step 6 - Change the Pins
===

If you want to change which two out of the three solder-tabs are being used as the output, then change the file you flash. The phase-pin-combination number of the file name determines which solder-tabs are being used as output. But exactly which one is what combination is a mystery that can only be discovered by experimenting.

![](doc/imgs/mysterycombos.png)

(personally I prefer to use the outer-most tabs, and not use the center tab, because there's less risk of a short circuit)

Other Notes
===

As I said before, this is a fork of the Greenjay firmware, meaning I did not write it originally, and this repo is a copy of it so that I can modify it.

The only change I've made:

 * make sure all 3 combinations are automatically built
 * appended `GJ_` to the file names
 * placed all files in the `src` directory
 * add this instructional readme

I am unlikely to ever update the source code, and it is unlikely that BLHeli_S or the upstream Greenjay repository get updated.

If there are bugs, I will not be able to assist.
