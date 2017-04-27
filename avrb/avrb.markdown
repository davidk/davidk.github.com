---
title: "Avrrrinator B"
date: 2014-05-17
categories:
  - pcb
  - buspirate
aliases:
  - /avrb
---

## Silkscreen text (bottom): "Squirrel Packing A Ruby Laser Rod"

![avrrrinator rev b](/avrb/images/avrrrinator_b_top.png)

For the prior revision A release (single output, no logic level voltage translation), [click here](http://open.konspyre.org/blog/2013/01/23/the-avrrrinator-revision-a/).

This is revision B, adding two outputs behind buffers (which also offers logic level voltage shifting).

![avrrrinator rev b programmin](/avrb/images/avrrrinator_b_early.png)

With revision B, you can flash two attached projects (not at the same time!) without moving your cables around.

This could be mis-construed to be more awesome than it is, so just to be clear: This isn't a jig for mass-manufacturing, but for projects where it is nice to compare and contrast among two devices.

Being an adapter for the Bus Pirate means getting features tailored for hacking too, like the ability to provide a small amount of power to connected projects (around 150mA):

```bash
# Only available on some firmware revisions
# it seems, YMMV
$ screen /dev/ttyUSB0 115200
HiZ> m 9
DIO> W
```

Of course, this thing does actually program AVR chips!

## B-side: "What are the avrdude commands? I can't remember the avrdude commands!"

Revision B includes a cheat sheet. Just flip the board over.

![reverse side avrb](/avrb/images/avrrrinator_b_rear.png)

Here are the general commands listed if you want to cut and paste:

```bash
$ avrdude -v -p attiny4313 -c buspirate
-P /dev/ttyUSB0 -U flash:w:blinky313.hex -x reset=cs

$ avrdude -v -p attiny85 -c buspirate
-P /dev/ttyUSB0 -U flash:w:blinky85.hex -x reset=aux
```

* Change `blinky313.hex` or `blinky85.hex` to the hex file for flashing/upload.

* `-x reset=cs` will target the AVR attached to the "ISP CS" header, and `-x reset=aux` will target the AVR attached to "ISP AUX".

* `/dev/ttyUSB0` will need to be changed to reflect the location of your Bus Pirate.

See below for test runs of Avrrrinator Rev. B against an ATTiny4313 and ATTiny85.

## Board Files

Grab the board file here: [https://github.com/davidk/avrrrinator/tree/master/hardware/revB](https://github.com/davidk/avrrrinator/tree/master/hardware/revB)

Keen observers (read: nobody) will notice that i've switched to KiCad for some recent projects. Revision B has been in the works for quite some time! I
may get around to porting this project over to KiCad one day, but it'll likely involve a redo of all the routing and placement.

### Something to keep in mind: ***This board has small parts. They're probably a sanity hazard.***

0603 (imperial) is considered by some to be just at the edge of being too difficult to do by hand/without solder
paste and vision assistance (microscope).

_You can probably get by on good leaded solder, some solder wick, and drag soldering (for the fine pitch ICs)._

The board can be sent off for fabrication in several ways:

1. Take the board file, load it into Eagle and generate gerbers appropriate for your fabricator.

2. [Click through to the repository](https://github.com/davidk/avrrrinator/tree/master/hardware/revB) for the shared projects link at OSHPark. Order from there. (You can also upload the .brd file to OSHPark for fabrication.)

While you wait, buy the parts on the [Bill of Materials list](https://github.com/davidk/avrrrinator/tree/master/hardware/revB#bill-of-materials-for-revision-b).

## Soldering

For accurate placement information, see the [Bill of Materials](https://github.com/davidk/avrrrinator/tree/master/hardware/revB#bill-of-materials-for-revision-b).

_The two surface mount switches are quite delicate. If using lead-free solder paste, be aware that the plastic switch may melt slightly before the solder reflows. Careful hot air application/hand soldering is suggested._

It is best to start off with soldering the larger ICs, and end with the smaller 0603 resistors and capacitors. Only solder in the through-hole components when you are satisfied with the surface mount placement.

Capacitors are all 0603 (imperial) 0.1µF, so just solder them to everything C.
All resistors are 0603 (imperial), and 10k ohms.

## Caveats and future improvements

There are some things you should be aware of:

* This is intended for programming AVR microcontrollers through an ISP interface, using avrdude. Boards like the Arduino have a 6-pin AVR ISP connector. Nothing else unless you use hacks.

* There is very little in the way of protection. This board will happily hang around and get knocked in the head by some really bad things. Physically and electrically.

* While revision B has been tested in a limited capacity, this breakout is fairly new. A crazy combination of conditions $X $Y $Z will cause problems. So don't pack this as your only programmer to Makerfaire, etc.

* Routing needs improvement. Everything works, it just isn't pretty/nice enough.

* The 10k resistors exist to set the state of the AVR side during the high impedance/tri-state stages. Further revisions (if done) will suppress the cross-coupling present more elegantly.

* This requires a version 3 Bus Pirate. Version 4 won't work unless connections are jumpered across to the breakout (untested since I do not have a v4 Bus Pirate).

* This also requires a supported Bus Pirate 3 firmware revision, as well as a recent avrdude version with Bus Pirate support. This board has been tested with avrdude version: 5.11.1, and Bus Pirate firmware version: 5.10 (r559). ***If you have avrdude version 6, there is a _bug_ that prevents you from using more than one output (requires a separate posting).***

The following change to `buspirate.c` makes things work again.
```diff
Index: buspirate.c
===================================================================
--- buspirate.c (revision 1285)
+++ buspirate.c (working copy)
@@ -573,7 +573,7 @@

        /* 0b0100wxyz - Configure peripherals w=power, x=pull-ups/aux2, y=AUX, z=CS
         * we want power (0x48) and all reset pins high. */
-       PDATA(pgm)->current_peripherals_config  = 0x48 | PDATA(pgm)->reset;
+       PDATA(pgm)->current_peripherals_config  = 0x48 | BP_RESET_CS | BP_RESET_AUX;
        buspirate_expect_bin_byte(pgm, PDATA(pgm)->current_peripherals_config, 0x01);
        usleep(50000);  // sleep for 50ms after power up  
```

## Licensing

This project is Open Source Hardware. Feel free to open, examine, and make. Officially, revision B is [Creative Commons Zero](http://creativecommons.org/publicdomain/zero/1.0/). This is a public domain dedication with an added warranty disclaimer.

I can't stop you from not doing this (and I really don't want to), but if you plan to make these and sell them, please spin off a cleverly named derivative. Just change the name, url, and add/remove fancy flourishes as desired. I'll be happy to add you to a list of (hopefully improved) derivatives too. This is mostly because I lack the bandwidth to support projects.

## Acknowledgments

The following deserve thanks for various reasons:

* [***OSHPark***](http://www.oshpark.com): Purple all the things! It was very useful to forgo breadboarding at one point and just spin prototypes without abandon.

* [***The BB313 board***](http://www.johngineer.com/projects/bb313/): Many things are borrowed from this board. Parts: ISP connector, and pull jumper. The silkscreen for the ISP connector is borrowed from the BB313 -- it is more user-friendly than the standard "honeycomb" connector footprint. More importantly, these boards were used as test targets throughout the development of the Avrrrinator (both revision A and B).

* [***Dangerous Prototypes' Bus Pirate***](http://dangerousprototypes.com/docs/Bus_Pirate), as well as its [***part list***](http://dangerousprototypes.com/docs/Partlist) and [***footprints***](http://dangerousprototypes.com/docs/Dangerous_Prototypes_Cadsoft_Eagle_parts_library) library. In particular, the power selection switch is borrowed directly from Dangerous Protoypes' list and footprint. The Bus Pirate connector is as well.

* [***avrdude***](http://www.nongnu.org/avrdude/) support for the Bus Pirate and documentation in its man page.

## Past

Many spins ago, I decided to showcase what I was working on. Here is an earlier prototype of this revision, targeting two ATTiny MCUs for 24 hours. It was an entry into the Adafruit 6 second film festival contest:

<iframe width="500" height="480" src="//www.youtube.com/embed/1nvUlqp4ISU" frameborder="0" allowfullscreen></iframe>

## Test output

```bash
avrdude -v -p attiny4313 -c buspirate -P /dev/ttyUSB0 -U flash:w:blinky313.hex -x reset=cs speed=7

avrdude: Version 5.11.1, compiled on Jun 18 2013 at 10:33:15
         Copyright (c) 2000-2005 Brian Dean, http://www.bdmicro.com/
         Copyright (c) 2007-2009 Joerg Wunsch

         System wide configuration file is "/etc/avrdude/avrdude.conf"
         User configuration file is "/home/davidk/.avrduderc"
         User configuration file does not exist or is not a regular file, skipping

         Using Port                    : /dev/ttyUSB0
         Using Programmer              : buspirate
         AVR Part                      : ATtiny4313
         Chip Erase delay              : 9000 us
         PAGEL                         : PD4
         BS2                           : PD6
         RESET disposition             : possible i/o
         RETRY pulse                   : SCK
         serial program mode           : yes
         parallel program mode         : yes
         Timeout                       : 200
         StabDelay                     : 100
         CmdexeDelay                   : 25
         SyncLoops                     : 32
         ByteDelay                     : 0
         PollIndex                     : 3
         PollValue                     : 0x53
         Memory Detail                 :

                                  Block Poll               Page                       Polled
           Memory Type Mode Delay Size  Indx Paged  Size   Size #Pages MinW  MaxW   ReadBack
           ----------- ---- ----- ----- ---- ------ ------ ---- ------ ----- ----- ---------
           eeprom        65     6     4    0 no        256    4      0  4000  4500 0xff 0xff
           flash         65     6    32    0 yes      4096   64     64  4500  4500 0xff 0xff
           signature      0     0     0    0 no          3    0      0     0     0 0x00 0x00
           lock           0     0     0    0 no          1    0      0  9000  9000 0x00 0x00
           lfuse          0     0     0    0 no          1    0      0  9000  9000 0x00 0x00
           hfuse          0     0     0    0 no          1    0      0  9000  9000 0x00 0x00
           efuse          0     0     0    0 no          1    0      0  9000  9000 0x00 0x00
           calibration    0     0     0    0 no          2    0      0     0     0 0x00 0x00

         Programmer Type : BusPirate
         Description     : The Bus Pirate

Detecting BusPirate...
avrdude: buspirate_readline(): #
avrdude: buspirate_readline(): RE
avrdude: buspirate_readline(): Bus Pirate v3.5
avrdude: buspirate_readline(): Firmware v6.1 r1676  Bootloader v4.4
avrdude: buspirate_readline(): DEVID:0x0447 REVID:0x3046 (24FJ64GA002 B8)
avrdude: buspirate_readline(): http://dangerousprototypes.com
avrdude: buspirate_readline(): HiZ>
**
BusPirate: using BINARY mode
BusPirate binmode version: 1
BusPirate SPI version: 1
avrdude: AVR device initialized and ready to accept instructions

Reading | ################################################## | 100% 0.01s

avrdude: Device signature = 0x1e920d
avrdude: safemode: lfuse reads as 64
avrdude: safemode: hfuse reads as DF
avrdude: safemode: efuse reads as FF
avrdude: NOTE: FLASH memory has been specified, an erase cycle will be performed
         To disable this feature, specify the -D option.
avrdude: erasing chip
avrdude: reading input file "blinky313.hex"
avrdude: input file blinky313.hex auto detected as Intel Hex
avrdude: writing flash (60 bytes):

Writing | ################################################## | 100% 0.19s

avrdude: 60 bytes of flash written
avrdude: verifying flash memory against blinky313.hex:
avrdude: load data flash data from input file blinky313.hex:
avrdude: input file blinky313.hex auto detected as Intel Hex
avrdude: input file blinky313.hex contains 60 bytes
avrdude: reading on-chip flash data:

Reading | ################################################## | 100% 0.18s

avrdude: verifying ...
avrdude: 60 bytes of flash verified

avrdude: safemode: lfuse reads as 64
avrdude: safemode: hfuse reads as DF
avrdude: safemode: efuse reads as FF
avrdude: safemode: Fuses OK
BusPirate is back in the text mode

avrdude done.  Thank you.

avrdude -v -p attiny85 -c buspirate -P /dev/ttyUSB0 -U flash:w:blinky45.hex -x reset=aux speed=7

avrdude: Version 5.11.1, compiled on Jun 18 2013 at 10:33:15
         Copyright (c) 2000-2005 Brian Dean, http://www.bdmicro.com/
         Copyright (c) 2007-2009 Joerg Wunsch

         System wide configuration file is "/etc/avrdude/avrdude.conf"
         User configuration file is "/home/davidk/.avrduderc"
         User configuration file does not exist or is not a regular file, skipping

         Using Port                    : /dev/ttyUSB0
         Using Programmer              : buspirate
         AVR Part                      : ATtiny85
         Chip Erase delay              : 4500 us
         PAGEL                         : P00
         BS2                           : P00
         RESET disposition             : possible i/o
         RETRY pulse                   : SCK
         serial program mode           : yes
         parallel program mode         : yes
         Timeout                       : 200
         StabDelay                     : 100
         CmdexeDelay                   : 25
         SyncLoops                     : 32
         ByteDelay                     : 0
         PollIndex                     : 3
         PollValue                     : 0x53
         Memory Detail                 :

                                  Block Poll               Page                       Polled
           Memory Type Mode Delay Size  Indx Paged  Size   Size #Pages MinW  MaxW   ReadBack
           ----------- ---- ----- ----- ---- ------ ------ ---- ------ ----- ----- ---------
           eeprom        65     6     4    0 no        512    4      0  4000  4500 0xff 0xff
           flash         65     6    32    0 yes      8192   64    128  4500  4500 0xff 0xff
           signature      0     0     0    0 no          3    0      0     0     0 0x00 0x00
           lock           0     0     0    0 no          1    0      0  9000  9000 0x00 0x00
           lfuse          0     0     0    0 no          1    0      0  9000  9000 0x00 0x00
           hfuse          0     0     0    0 no          1    0      0  9000  9000 0x00 0x00
           efuse          0     0     0    0 no          1    0      0  9000  9000 0x00 0x00
           calibration    0     0     0    0 no          2    0      0     0     0 0x00 0x00

         Programmer Type : BusPirate
         Description     : The Bus Pirate

Detecting BusPirate...
avrdude: buspirate_readline(): #
avrdude: buspirate_readline(): RE
avrdude: buspirate_readline(): Bus Pirate v3.5
avrdude: buspirate_readline(): Firmware v6.1 r1676  Bootloader v4.4
avrdude: buspirate_readline(): DEVID:0x0447 REVID:0x3046 (24FJ64GA002 B8)
avrdude: buspirate_readline(): http://dangerousprototypes.com
avrdude: buspirate_readline(): HiZ>
**
BusPirate: using BINARY mode
BusPirate binmode version: 1
BusPirate SPI version: 1
avrdude: AVR device initialized and ready to accept instructions

Reading | ################################################## | 100% 0.01s

avrdude: Device signature = 0x1e930b
avrdude: safemode: lfuse reads as 62
avrdude: safemode: hfuse reads as DF
avrdude: safemode: efuse reads as FF
avrdude: NOTE: FLASH memory has been specified, an erase cycle will be performed
         To disable this feature, specify the -D option.
avrdude: erasing chip
avrdude: reading input file "blinky45.hex"
avrdude: input file blinky45.hex auto detected as Intel Hex
avrdude: writing flash (18 bytes):

Writing | ################################################## | 100% 0.06s

avrdude: 18 bytes of flash written
avrdude: verifying flash memory against blinky45.hex:
avrdude: load data flash data from input file blinky45.hex:
avrdude: input file blinky45.hex auto detected as Intel Hex
avrdude: input file blinky45.hex contains 18 bytes
avrdude: reading on-chip flash data:

Reading | ################################################## | 100% 0.05s

avrdude: verifying ...
avrdude: 18 bytes of flash verified

avrdude: safemode: lfuse reads as 62
avrdude: safemode: hfuse reads as DF
avrdude: safemode: efuse reads as FF
avrdude: safemode: Fuses OK
BusPirate is back in the text mode

avrdude done.  Thank you.
```

cheers
