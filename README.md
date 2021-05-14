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

<h1 align="center">Marlin 3D Printer Firmware</h1>

<p align="center">
    <a href="/LICENSE"><img alt="GPL-V3.0 License" src="https://img.shields.io/github/license/marlinfirmware/marlin.svg"></a>
    <a href="https://github.com/MarlinFirmware/Marlin/graphs/contributors"><img alt="Contributors" src="https://img.shields.io/github/contributors/marlinfirmware/marlin.svg"></a>
    <a href="https://github.com/MarlinFirmware/Marlin/releases"><img alt="Last Release Date" src="https://img.shields.io/github/release-date/MarlinFirmware/Marlin"></a>
    <a href="https://github.com/MarlinFirmware/Marlin/actions/workflows/ci-build-tests.yml"><img alt="CI Status" src="https://github.com/MarlinFirmware/Marlin/actions/workflows/ci-build-tests.yml/badge.svg"></a>
    <a href="https://github.com/sponsors/thinkyhead"><img alt="GitHub Sponsors" src="https://img.shields.io/github/sponsors/thinkyhead?color=db61a2"></a>
    <br />
    <a href="https://fosstodon.org/@marlinfirmware"><img alt="Follow MarlinFirmware on Mastodon" src="https://img.shields.io/mastodon/follow/109450200866020466?domain=https%3A%2F%2Ffosstodon.org&logoColor=%2300B&style=social"></a>
</p>

Additional documentation can be found at the [Marlin Home Page](//marlinfw.org/).
Please test this firmware and let us know if it misbehaves in any way. Volunteers are standing by!

## Marlin 2.1

Marlin 2.1 continues to support both 32-bit ARM and 8-bit AVR boards while adding support for up to 9 coordinated axes and to up to 8 extruders.

Download earlier versions of Marlin on the [Releases page](//github.com/MarlinFirmware/Marlin/releases).

## Example Configurations

Before you can build Marlin for your machine you'll need a configuration for your specific hardware. Upon request, your vendor will be happy to provide you with the complete source code and configurations for your machine, but you'll need to get updated configuration files if you want to install a newer version of Marlin. Fortunately, Marlin users have contributed dozens of tested configurations to get you started. Visit the [MarlinFirmware/Configurations](//github.com/MarlinFirmware/Configurations) repository to find the right configuration for your hardware.

## Building Marlin 2.1

To build and upload Marlin you will use one of these tools:

- The free [Visual Studio Code](//code.visualstudio.com/download) using the [Auto Build Marlin](//marlinfw.org/docs/basics/auto_build_marlin.html) extension.
- The free [Arduino IDE](//www.arduino.cc/en/main/software) : See [Building Marlin with Arduino](//marlinfw.org/docs/basics/install_arduino.html)
- You can also use VSCode with devcontainer : See [Installing Marlin (VSCode devcontainer)](http://marlinfw.org/docs/basics/install_devcontainer_vscode.html).

Marlin is optimized to build with the **PlatformIO IDE** extension for **Visual Studio Code**. You can still build Marlin with **Arduino IDE**, and we hope to improve the Arduino build experience, but at this time PlatformIO is the better choice.

## 8-Bit AVR Boards

We intend to continue supporting 8-bit AVR boards in perpetuity, maintaining a single codebase that can apply to all machines. We want casual hobbyists and tinkerers and owners of older machines to benefit from the community's innovations just as much as those with fancier machines. Plus, those old AVR-based machines are often the best for your testing and feedback!

## Hardware Abstraction Layer (HAL)

Marlin includes an abstraction layer to provide a common API for all the platforms it targets. This allows Marlin code to address the details of motion and user interface tasks at the lowest and highest levels with no system overhead, tying all events directly to the hardware clock.

Every new HAL opens up a world of hardware. At this time we need HALs for RP2040 and the Duet3D family of boards. A HAL that wraps an RTOS is an interesting concept that could be explored. Did you know that Marlin includes a Simulator that can run on Windows, macOS, and Linux? Join the Discord to help move these sub-projects forward!

### Supported Platforms

  Platform|MCU|Example Boards
  --------|---|-------
  [Arduino AVR](//www.arduino.cc/)|ATmega|RAMPS, Melzi, RAMBo
  [Teensy++ 2.0](//www.microchip.com/en-us/product/AT90USB1286)|AT90USB1286|Printrboard
  [Arduino Due](//www.arduino.cc/en/Guide/ArduinoDue)|SAM3X8E|RAMPS-FD, RADDS, RAMPS4DUE
  [ESP32](//github.com/espressif/arduino-esp32)|ESP32|FYSETC E4, E4d@BOX, MRR
  [LPC1768](//www.nxp.com/products/processors-and-microcontrollers/arm-microcontrollers/general-purpose-mcus/lpc1700-cortex-m3/512-kb-flash-64-kb-sram-ethernet-usb-lqfp100-package:LPC1768FBD100)|ARM® Cortex-M3|MKS SBASE, Re-ARM, Selena Compact
  [LPC1769](//www.nxp.com/products/processors-and-microcontrollers/arm-microcontrollers/general-purpose-mcus/lpc1700-cortex-m3/512-kb-flash-64-kb-sram-ethernet-usb-lqfp100-package:LPC1769FBD100)|ARM® Cortex-M3|Smoothieboard, Azteeg X5 mini, TH3D EZBoard
  [STM32F103](//www.st.com/en/microcontrollers-microprocessors/stm32f103.html)|ARM® Cortex-M3|Malyan M200, GTM32 Pro, MKS Robin, BTT SKR Mini
  [STM32F401](//www.st.com/en/microcontrollers-microprocessors/stm32f401.html)|ARM® Cortex-M4|ARMED, Rumba32, SKR Pro, Lerdge, FYSETC S6, Artillery Ruby
  [STM32F7x6](//www.st.com/en/microcontrollers-microprocessors/stm32f7x6.html)|ARM® Cortex-M7|The Borg, RemRam V1
  [STM32G0B1RET6](//www.st.com/en/microcontrollers-microprocessors/stm32g0x1.html)|ARM® Cortex-M0+|BigTreeTech SKR mini E3 V3.0
  [STM32H743xIT6](//www.st.com/en/microcontrollers-microprocessors/stm32h743-753.html)|ARM® Cortex-M7|BigTreeTech SKR V3.0, SKR EZ V3.0, SKR SE BX V2.0/V3.0
  [SAMD21P20A](//www.adafruit.com/product/4064)|ARM® Cortex-M0+|Adafruit Grand Central M4
  [SAMD51P20A](//www.adafruit.com/product/4064)|ARM® Cortex-M4|Adafruit Grand Central M4
  [Teensy 3.2/3.1](//www.pjrc.com/teensy/teensy31.html)|MK20DX256VLH7 ARM® Cortex-M4|
  [Teensy 3.5](//www.pjrc.com/store/teensy35.html)|MK64FX512-VMD12 ARM® Cortex-M4|
  [Teensy 3.6](//www.pjrc.com/store/teensy36.html)|MK66FX1MB-VMD18 ARM® Cortex-M4|
  [Teensy 4.0](//www.pjrc.com/store/teensy40.html)|MIMXRT1062-DVL6B ARM® Cortex-M7|
  [Teensy 4.1](//www.pjrc.com/store/teensy41.html)|MIMXRT1062-DVJ6B ARM® Cortex-M7|
  Linux Native|x86 / ARM / RISC-V|Raspberry Pi GPIO
  Simulator|Windows, macOS, Linux|Desktop OS
  [All supported boards](//marlinfw.org/docs/hardware/boards.html#boards-list)|All platforms|All boards

## Marlin Support

The Issue Queue is reserved for Bug Reports and Feature Requests. Please use the following resources for help with configuration and troubleshooting:

- [Marlin Documentation](//marlinfw.org) - Official Marlin documentation
- [Marlin Discord](//discord.com/servers/marlin-firmware-461605380783472640) - Discuss issues with Marlin users and developers
- Facebook Group ["Marlin Firmware"](//www.facebook.com/groups/1049718498464482/)
- RepRap.org [Marlin Forum](//forums.reprap.org/list.php?415)
- Facebook Group ["Marlin Firmware for 3D Printers"](//www.facebook.com/groups/3Dtechtalk/)
- [Marlin Configuration](//www.youtube.com/results?search_query=marlin+configuration) on YouTube

## Contributing Patches

You can contribute patches by submitting a Pull Request to the ([bugfix-2.1.x](//github.com/MarlinFirmware/Marlin/tree/bugfix-2.1.x)) branch.

- We use branches named with a "bugfix" or "dev" prefix to fix bugs and integrate new features.
- Follow the [Coding Standards](//marlinfw.org/docs/development/coding_standards.html) to gain points with the maintainers.
- Please submit Feature Requests and Bug Reports to the [Issue Queue](//github.com/MarlinFirmware/Marlin/issues/new/choose). See above for user support.
- Whenever you add new features, be sure to add one or more build tests to `buildroot/tests`. Any tests added to a PR will be run within that PR on GitHub servers as soon as they are pushed. To minimize iteration be sure to run your new tests locally, if possible.
  - Local build tests:
    - All: `make tests-config-all-local`
    - Single: `make tests-config-single-local TEST_TARGET=...`
  - Local build tests in Docker:
    - All: `make tests-config-all-local-docker`
    - Single: `make tests-config-all-local-docker TEST_TARGET=...`
  - To run all unit test suites:
    - Using PIO: `platformio run -t test-marlin`
    - Using Make: `make unit-test-all-local`
    - Using Docker + make: `maker unit-test-all-local-docker`
  - To run a single unit test suite:
    - Using PIO: `platformio run -t marlin_<test-suite-name>`
    - Using make: `make unit-test-single-local TEST_TARGET=<test-suite-name>`
    - Using Docker + make: `maker unit-test-single-local-docker TEST_TARGET=<test-suite-name>`
- If your feature can be unit tested, add one or more unit tests. For more information see our documentation on [Unit Tests](test).

## Contributors

Marlin is constantly improving thanks to a huge number of contributors from all over the world bringing their specialties and talents. Huge thanks are due to [all the contributors](//github.com/MarlinFirmware/Marlin/graphs/contributors) who regularly patch up bugs, help direct traffic, and basically keep Marlin from falling apart. Marlin's continued existence would not be possible without them.

Marlin Firmware original logo design by Ahmet Cem TURAN [@ahmetcemturan](//github.com/ahmetcemturan).

## Project Leadership

Name|Role|Link|Donate
----|----|----|----
🇺🇸 Scott Lahteine|Project Lead|[[@thinkyhead](//github.com/thinkyhead)]|[💸 Donate](//marlinfw.org/docs/development/contributing.html#donate)
🇺🇸 Roxanne Neufeld|Admin|[[@Roxy-3D](//github.com/Roxy-3D)]|
🇺🇸 Keith Bennett|Admin|[[@thisiskeithb](//github.com/thisiskeithb)]|[💸 Donate](//github.com/sponsors/thisiskeithb)
🇺🇸 Jason Smith|Admin|[[@sjasonsmith](//github.com/sjasonsmith)]|
🇧🇷 Victor Oliveira|Admin|[[@rhapsodyv](//github.com/rhapsodyv)]|
🇬🇧 Chris Pepper|Admin|[[@p3p](//github.com/p3p)]|
🇳🇿 Peter Ellens|Admin|[[@ellensp](//github.com/ellensp)]|[💸 Donate](//ko-fi.com/ellensp)
🇺🇸 Bob Kuhn|Admin|[[@Bob-the-Kuhn](//github.com/Bob-the-Kuhn)]|
🇳🇱 Erik van der Zalm|Founder|[[@ErikZalm](//github.com/ErikZalm)]|

## Star History

<a id="starchart" href="https://star-history.com/#MarlinFirmware/Marlin&Date">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/svg?repos=MarlinFirmware/Marlin&type=Date&theme=dark" />
    <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/svg?repos=MarlinFirmware/Marlin&type=Date" />
    <img alt="Star History Chart" src="https://api.star-history.com/svg?repos=MarlinFirmware/Marlin&type=Date" />
  </picture>
</a>

## License

Marlin is published under the [GPL license](/LICENSE) because we believe in open development. The GPL comes with both rights and obligations. Whether you use Marlin firmware as the driver for your open or closed-source product, you must keep Marlin open, and you must provide your compatible Marlin source code to end users upon request. The most straightforward way to comply with the Marlin license is to make a fork of Marlin on Github, perform your modifications, and direct users to your modified fork.
