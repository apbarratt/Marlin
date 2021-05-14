# Notes on our Marlin build

The official marlin read me starts a bit further now with the title "Marlin 3D Printer Firmware" but in the meantime, these are my notes on what I've configured here.

## Our machine

Our machine is an original Ender 3 with various upgrades:

* BigTreeTech (BTT) SKR Mini E3 V2.0 board
  * [Amazon link](https://amzn.to/33HmLQz)
    * I think I actually purchased this directly from BTT's official store, but Amazon certainly would have shipped it faster!
  * This is a "silent" board.  I was dubious of what silent really meant but I'm pretty impressed so far!  Basically all the weird squealing noises that the stepper motors made just vanish due to different stepper drivers and how they're configured.  Supposedly.  I actually suspect there's some amount of witchcraft involved.
  * Bad news is the hot end fan is just as noisy as ever.
  * I've found my print quality is vastly improved too and I never have weird slow down moments anymore when printing from octoprint (rare, but they happened).  I guess it being able to think and process faster just results in less mechanical mistakes, which makes sense.
  * As an aside, when I use my laser module with this printer using the hotend fan port, I get far more reliable control over it with this board than I did with the old creality one.
    * An example of this would be that setting the fan speed to speed 1 (of 255) should turn on the 500mw laser with enough power to shine but not burn.  This is the case on this board but on the original Creality burn, this level of control didn't seem to be there and I got a slowly burning laser anyway.
* OctoPrint on Raspberry Pi 4.  Admitedly I don't think this affects the firmware a huge amount, but I was keeping it in mind in terms of enabling functions.
  * The upgrade from Raspberry Pi 3 to 4 was very recent.  So far I've noticed a slight improvement in terms of web interface speed but not as much as I might have expected.  It was still very reliable already, I just had a spare 4 lying around.
* BLTouch mounted using official Ender 3 mounting kit.
  * [Amazon link](https://amzn.to/3uZvt93)
    * This is the listing I purchased from.
  * I believe I have a version 3.0 BLTouch, but I've not enabled some of the V3 specific features for this just yet as I'm a bit nervous of using higher voltages, especially if it seems to be working without them.
* BMG Extruder (clone)
  * [Amazon link](https://amzn.to/3w9V7YP)
    * This is the listing I purchased from.
  * The *official* BMG extruder is extortionately priced at $80 and I don't believe for a second that such a basic concept can be any different to the one I've purchased here for £16 (around $22).
  * It was recommended to me by someone on Facebook when I was having trouble with my metal extruder not gripping.  It seemed like overkill but for the price I figured I might as well try it and am extremely pleased that I did.
  * This extruder will push anything down that bowden tube with strength enough to prevent the jams that just seemed a fact of life before hand.
  * I found that before this, that I couldn't really use nozzle sizes below or much above 0.4mm with any reliable results from print to print.  That issue vanished with this extruder.
  * I don't use direct drive so I haven't attempted this with anything like a flexible material.  It's my understanding I'd have to be ditching the bowden tube set up entirely for such things.
* BigTreeTech (BTT) Smart Filament Sensor
  * [Amazon link](https://amzn.to/3ya8ppM)
    * I have to confess, I can't remember where I bought mine from.  Probably directly from BTT when I got the board though.
  * Okay.  This firmware has partly been put together to try and get this thing working.  I have never been able to get this to work correctly and have had it removed completely.  Frustratingly I lost the original firmware I configured the last time I flashed my printer so I've never been able to check to see what might have been wrong.  Hopefully this start from scratch and keeping it version controlled will allow some more experimentation.
  * As I'm a bit mistrusting of this thing, it is enabled in this firmware but turned off by default for the time being.
    * This actually means you can use this firmware even if you don't have this sensor plugged in at all.  It won't even notice.
* Blue Laser Engraving moudle
  * [Creality Store link](https://www.creality3dofficial.com/collections/spare-parts-acc/products/creality-blue-violet-light)
    * I don't know why, but it seems you can't get the official Creality recommended kit on Amazon.
    * We run a laser engraving business and this little kit is what got us started.  These days we don't really do any laser engraving on our Printer, indeed I've stuck this kit on a DIY CNC machine now, but I may use it on this machine again from time to time for unusual jobs :)
    * It's not a strong laser, basic burning only, no cutting, but it's a fun bit of kit to play with.
    * Just stay safe! Lasers are dangerous, even lower power ones like these.

## Our firmware config (the highlights)

So I've based our config on a number of different tutorials regarding the SKR Mini E3 V2 board as well as tutorials for BLTouch configuration and of course, the dreaded BTT Smart Filament Sensor.  And of course I took into account a few of my own personal preferences and tweaks to the printer I've made.

* BLTouch
  * This has been configured to work using BiLinear probing on a 5x5 grid.
    * The 5x5 grid does unfortunately take a wee while to probe before each print, but overall I've found that this has been a valuable addition to an already well leveled bed.
      * If you plan to, like me and as seems to be recommended, probe before every print, you will need to add a G29 code just after the normal G28 code in the starting Gcode within your slicing software.
  * I've set the BLTouch delay to 200.  Apparently this is okay on the V3, but may need to go back to 500 on some others.
  * I've turned on High Speed mode.  I don't know if I had it on before, but high speed does sound good!
    * Apparently it's good on the V3 but you might want to turn this off for other versions.
  * I've fitted my BLTouch using the official ender 3 mounting kit.
    * I've positioned this plate to sit as far up as possible before tightening the screws.  This just makes the chances of the probe scraping an ongoing print that little less likely.
      * The result of this positioning is that I've found my Z-offset recently has been around 1.6mm.  Compare this to not paying much attention to the plate's vertical positioning and it was as high as 2.75mm.  It may not sound like a big difference but it really did seem to make a huge difference in practice.
    * I've taken a few measurements and found that my Z-probes X Y offset to be 43.5mm to the left and 5mm to the front and have set this in this firmware.
      * See NOZZLE_TO_PROBE_OFFSET in Configuration.h
      * Take note that this seems to be fairly different to several other people's experience with this exact same mounting plate. So do find time to take your own measurements.
    * Whilst you may already know your normal Z-offset, I'm following the advice of others and keeping the firmware default at 0 to prevent accidents.  After all, your Z-offset can change from such things as changing the bed surface or switching out nozzles, etc.
  * I've set the probing margin to 15mm.  It's normally 10mm and despite my bed clips barely entering the bed, the probe getting so close to them has always made me nervouse so I figured an extra 5mm couldn't hurt.
* E-Steps default value
  * The default E-steps value is usually 93 on the Ender 3.  This is fine, you'd be expected to do some basic E-steps calibration anyway to find a reasonable value.  But with the BMG extruder, 93 isn't really enough to do anything with at all.  I could have set the default to the reference value of 385, but at the time of writing I know my calibrated E-steps are set to 411.76 so I figure I might as well just plonk this value in there :)
    * See DEFAULT_AXIS_STEPS_PER_UNIT in Configuration.h
    * If you are not using a BMG extruder and hope to base your own firmware on mine, you will definitely want to change this, probably back to 93, before you flash your machine.
* PID Autotune for both hotend and bed
  * Autotuning the hot end temperature using Marlin's PID Autotune command really does seem to make a nice difference in avoiding weird heating errors and fluctuations. So I've ensured this is enabled.
  * What I'm doing new this time around though is also enabling the PID Autotune functionality for the heated bed.  I've not really had many issues with my bed to be honest but I figured having it well calibrated using the same functionality I do for the hotend couldn't hurt :)
  * I've plonked the latest results from my most recent PID Autotunes as the default values.  Obviously your mileage may vary so run your own autotunes.
* ARC support
  * I've often spotted the Arc Welder extensions for octoprint and Cura and wondered what kind of a difference they'd make.  Basically arc welder converts the many little move G0/G1 commands in a curve into single G2/G3 arc commands instead.  Supposedly this will improve stuttering on the printer.
  * I figured that I seem to have the space available to include this ARC support so I might as well add it in there :)
* Custom Bootscreen
  * Because why not? =D
    * The wee laser company my wife and I run has a cartoon logo that makes me smile, so I figured it'd be fun to see it flash up when I turn on the printer ;)
* Smart Filament Runout Sensor
  * So I've enabled support for this and used some recommended settings based on a couple of tutorials, but I did set the default value for it to be turned off.  Basically I'll be able to turn it on in the menu should I want to try it out.
* LCD Bed Leveling
  * BLTouch is not a replacement for normal bed leveling and the LCD bed leveling menus are rather useful in speeding this up I've found, especially the one for just leveling the corners.
* Set up a couple of my most used materials.  Both PLA but my glow in the dark stuff needs a stupid hot bed to work.
* Nozzle Park Feature
  * Yeah, I don't know why this would ever be turned off, it's so nifty!  But it's a neccessity if I get this filament sensor working.
* Speaker
  * I hate it, but I've enabled the speaker.
  * I think I've configured it correctly though to not beep on every single button press though!
    * Just enough to make sure I hear alarms when they sound.
* Power Loss Recovery
  * Honestly, I never use this because it only works when printing from an SD card.  I only use Octoprint, but I figured it wouldn't hurt to have it turned on if that ever changes.
* Block buffer sizes
  * I've increased the block buffer sizes to 32 and 64 in their respective places.
  * I don't know if this is going to make much of a difference but I saw it recommended on multiple tutorials as something worth while doing to improve performance on this board as it can spare the space.
* Pin debugging
  * I'm not really sure if this is something I'll use, but one tutorial suggested it and it sounded like a good idea on the whole.

## A note on flashing

I don't understand what the reasoning is, but whilst you can build this firmware using PlatformIO in VSCode, I have found that it cannot be uploaded directly to the board over USB.

As such, you are required to place the generated bin file (found at .pio/build/STM32F103RC_btt_512K_USB/firmware.bin) directly onto an SD card and turn on the printer with it inserted... like a caveman!  I know!

## That's it from me

I would very much welcome feedback, I am by no means an expert on this stuff :)

Now, on to the original Marlin Readme text...

***

<p align="center"><img src="buildroot/share/pixmaps/logo/marlin-outrun-nf-500.png" height="250" alt="MarlinFirmware's logo" /></p>

# Marlin 3D Printer Firmware

<h1 align="center">Marlin 3D Printer Firmware</h1>

<p align="center">
    <a href="/LICENSE"><img alt="GPL-V3.0 License" src="https://img.shields.io/github/license/marlinfirmware/marlin.svg"></a>
    <a href="https://github.com/MarlinFirmware/Marlin/graphs/contributors"><img alt="Contributors" src="https://img.shields.io/github/contributors/marlinfirmware/marlin.svg"></a>
    <a href="https://github.com/MarlinFirmware/Marlin/releases"><img alt="Last Release Date" src="https://img.shields.io/github/release-date/MarlinFirmware/Marlin"></a>
    <a href="https://github.com/MarlinFirmware/Marlin/actions"><img alt="CI Status" src="https://github.com/MarlinFirmware/Marlin/actions/workflows/test-builds.yml/badge.svg"></a>
    <a href="https://github.com/sponsors/thinkyhead"><img alt="GitHub Sponsors" src="https://img.shields.io/github/sponsors/thinkyhead?color=db61a2"></a>
    <br />
    <a href="https://twitter.com/MarlinFirmware"><img alt="Follow MarlinFirmware on Twitter" src="https://img.shields.io/twitter/follow/MarlinFirmware?style=social&logo=twitter"></a>
</p>

Additional documentation can be found at the [Marlin Home Page](https://marlinfw.org/).
Please test this firmware and let us know if it misbehaves in any way. Volunteers are standing by!

## Marlin 2.0

Marlin 2.0 takes this popular RepRap firmware to the next level by adding support for much faster 32-bit and ARM-based boards while improving support for 8-bit AVR boards. Read about Marlin's decision to use a "Hardware Abstraction Layer" below.

Download earlier versions of Marlin on the [Releases page](https://github.com/MarlinFirmware/Marlin/releases).

## Example Configurations

Before building Marlin you'll need to configure it for your specific hardware. Your vendor should have already provided source code with configurations for the installed firmware, but if you ever decide to upgrade you'll need updated configuration files. Marlin users have contributed dozens of tested example configurations to get you started. Visit the [MarlinFirmware/Configurations](https://github.com/MarlinFirmware/Configurations) repository to find the right configuration for your hardware.

## Building Marlin 2.0

To build Marlin 2.0 you'll need [Arduino IDE 1.8.8 or newer](https://www.arduino.cc/en/main/software) or [PlatformIO](http://docs.platformio.org/en/latest/ide.html#platformio-ide). Detailed build and install instructions are posted at:

  - [Installing Marlin (Arduino)](http://marlinfw.org/docs/basics/install_arduino.html)
  - [Installing Marlin (VSCode)](http://marlinfw.org/docs/basics/install_platformio_vscode.html).

### Supported Platforms

  Platform|MCU|Example Boards
  --------|---|-------
  [Arduino AVR](https://www.arduino.cc/)|ATmega|RAMPS, Melzi, RAMBo
  [Teensy++ 2.0](https://www.microchip.com/en-us/product/AT90USB1286)|AT90USB1286|Printrboard
  [Arduino Due](https://www.arduino.cc/en/Guide/ArduinoDue)|SAM3X8E|RAMPS-FD, RADDS, RAMPS4DUE
  [ESP32](https://github.com/espressif/arduino-esp32)|ESP32|FYSETC E4, E4d@BOX, MRR
  [LPC1768](https://www.nxp.com/products/processors-and-microcontrollers/arm-microcontrollers/general-purpose-mcus/lpc1700-cortex-m3/512-kb-flash-64-kb-sram-ethernet-usb-lqfp100-package:LPC1768FBD100)|ARM® Cortex-M3|MKS SBASE, Re-ARM, Selena Compact
  [LPC1769](https://www.nxp.com/products/processors-and-microcontrollers/arm-microcontrollers/general-purpose-mcus/lpc1700-cortex-m3/512-kb-flash-64-kb-sram-ethernet-usb-lqfp100-package:LPC1769FBD100)|ARM® Cortex-M3|Smoothieboard, Azteeg X5 mini, TH3D EZBoard
  [STM32F103](https://www.st.com/en/microcontrollers-microprocessors/stm32f103.html)|ARM® Cortex-M3|Malyan M200, GTM32 Pro, MKS Robin, BTT SKR Mini
  [STM32F401](https://www.st.com/en/microcontrollers-microprocessors/stm32f401.html)|ARM® Cortex-M4|ARMED, Rumba32, SKR Pro, Lerdge, FYSETC S6, Artillery Ruby
  [STM32F7x6](https://www.st.com/en/microcontrollers-microprocessors/stm32f7x6.html)|ARM® Cortex-M7|The Borg, RemRam V1
  [SAMD51P20A](https://www.adafruit.com/product/4064)|ARM® Cortex-M4|Adafruit Grand Central M4
  [Teensy 3.5](https://www.pjrc.com/store/teensy35.html)|ARM® Cortex-M4|
  [Teensy 3.6](https://www.pjrc.com/store/teensy36.html)|ARM® Cortex-M4|
  [Teensy 4.0](https://www.pjrc.com/store/teensy40.html)|ARM® Cortex-M7|
  [Teensy 4.1](https://www.pjrc.com/store/teensy41.html)|ARM® Cortex-M7|
  Linux Native|x86/ARM/etc.|Raspberry Pi

## Submitting Changes

- Submit **Bug Fixes** as Pull Requests to the ([bugfix-2.0.x](https://github.com/MarlinFirmware/Marlin/tree/bugfix-2.0.x)) branch.
- Follow the [Coding Standards](http://marlinfw.org/docs/development/coding_standards.html) to gain points with the maintainers.
- Please submit your questions and concerns to the [Issue Queue](https://github.com/MarlinFirmware/Marlin/issues).

## Marlin Support

The Issue Queue is reserved for Bug Reports and Feature Requests. To get help with configuration and troubleshooting, please use the following resources:

- [Marlin Documentation](https://marlinfw.org) - Official Marlin documentation
- [Marlin Discord](https://discord.gg/n5NJ59y) - Discuss issues with Marlin users and developers
- Facebook Group ["Marlin Firmware"](https://www.facebook.com/groups/1049718498464482/)
- RepRap.org [Marlin Forum](https://forums.reprap.org/list.php?415)
- Facebook Group ["Marlin Firmware for 3D Printers"](https://www.facebook.com/groups/3Dtechtalk/)
- [Marlin Configuration](https://www.youtube.com/results?search_query=marlin+configuration) on YouTube

## Contributors

Marlin is constantly improving thanks to a huge number of contributors from all over the world bringing their specialties and talents. Huge thanks are due to [all the contributors](https://github.com/MarlinFirmware/Marlin/graphs/contributors) who regularly patch up bugs, help direct traffic, and basically keep Marlin from falling apart. Marlin's continued existence would not be possible without them.

## Administration

Regular users can open and close their own issues, but only the administrators can do project-related things like add labels, merge changes, set milestones, and kick trolls. The current Marlin admin team consists of:

 - Scott Lahteine [[@thinkyhead](https://github.com/thinkyhead)] - USA - Project Maintainer &nbsp; [💸 Donate](https://www.thinkyhead.com/donate-to-marlin)
 - Roxanne Neufeld [[@Roxy-3D](https://github.com/Roxy-3D)] - USA
 - Keith Bennett [[@thisiskeithb](https://github.com/thisiskeithb)] - USA &nbsp; [💸 Donate](https://github.com/sponsors/thisiskeithb)
 - Peter Ellens [[@ellensp](https://github.com/ellensp)] - New Zealand
 - Victor Oliveira [[@rhapsodyv](https://github.com/rhapsodyv)] - Brazil
 - Chris Pepper [[@p3p](https://github.com/p3p)] - UK
 - Jason Smith [[@sjasonsmith](https://github.com/sjasonsmith)] - USA
 - Luu Lac [[@shitcreek](https://github.com/shitcreek)] - USA
 - Bob Kuhn [[@Bob-the-Kuhn](https://github.com/Bob-the-Kuhn)] - USA
 - Erik van der Zalm [[@ErikZalm](https://github.com/ErikZalm)] - Netherlands &nbsp; [💸 Donate](https://flattr.com/submit/auto?user_id=ErikZalm&url=https://github.com/MarlinFirmware/Marlin&title=Marlin&language=&tags=github&category=software)

## License

Marlin is published under the [GPL license](/LICENSE) because we believe in open development. The GPL comes with both rights and obligations. Whether you use Marlin firmware as the driver for your open or closed-source product, you must keep Marlin open, and you must provide your compatible Marlin source code to end users upon request. The most straightforward way to comply with the Marlin license is to make a fork of Marlin on Github, perform your modifications, and direct users to your modified fork.

While we can't prevent the use of this code in products (3D printers, CNC, etc.) that are closed source or crippled by a patent, we would prefer that you choose another firmware or, better yet, make your own.
