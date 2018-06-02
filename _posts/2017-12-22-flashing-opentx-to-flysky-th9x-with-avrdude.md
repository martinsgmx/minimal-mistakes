---
title: "Flashing OpenTX to FlySky TH9x with AVRDUDE"
date:   2017-12-22 00:15:00 -0600
header:
    teaser: "{{ '/assets/img/posts/2017-12-22-flashing-opentx-to-flysky-th9x-with-avrdude/header.png' | absolute_url }}"
---

DISCLAIMER:
------

I don't take any responsibility about your control. Flash under your own risk! If you're following this guide step by step, your don't have any problem later. If your main board (th9x) is brick, you need buy other to solved problem. Don't need more! Not, don't need sell you control by parts. TH9x main board is very generic and some little cheap, you can buy one on [Banggood] or AliExpress.

Before starts you need know something's...
<!-- ------ -->

You need a AVR programmer for flashing [OpenTX] hex (firmware). You can use a USBasp, or you can use your own Arduino as AVRisp. On my case, I used USBasp.

Let's starts...
------

First of all, we need open FlySky TH9x to access main board and connect the USBasp or AVRisp. Unscrew and open it. I recommend umount the main board for works more easily. Be careful with flex screen and cables! 

I use a AVR cable with 10 pins to connect directly the main board with USBasp. I cut a end and solder it to main board.

![AVR cable]({{"/assets/img/posts/2017-12-22-flashing-opentx-to-flysky-th9x-with-avrdude/000_10_pins_cable.png" | absolute_url }})

Main board with cables. I used some flux to weld easily.

![Wiring main board]({{"/assets/img/posts/2017-12-22-flashing-opentx-to-flysky-th9x-with-avrdude/001_wiring.png" | absolute_url }})

> The first cable (that with red line) is the PIN ONE.

| ISP cable with 10 pins | | Main board TH9x |
|:-----:|:-:|:---------:|
|   1   |<->|   MOSI    |
|   2   |<->|   +5V     |
|   3   |<->|   ~       |
|   4   |<->|   ~       |
|   5   |<->|   RESET   |
|   6   |<->|   ~       |
|   7   |<->|   SCK     |
|   8   |<->|   ~       |
|   9   |<->|   MISO    |
|   10  |<->|   GROUND  |

![AVR pinout]({{"/assets/img/posts/2017-12-22-flashing-opentx-to-flysky-th9x-with-avrdude/002_avr_pinout.png" | absolute_url }})

You can use double sided tape to stick the cable and avoid desolded.

After weld it and before screw it the main board, verified the screen doesn't move.

![Screen screw]({{"/assets/img/posts/2017-12-22-flashing-opentx-to-flysky-th9x-with-avrdude/004_before_screw.png" | absolute_url }})

Screw it!

Well, after complete this, you control is ready to AVRdude. 
IMPORTANT: This process you must do it without a battery!

I create a .zip specifically for this control, FlySky TH9x. If you download this .zip don't need install [OpenTX] Companion.

The firmware to flash is OpenTX 2.2 - stable release ENGLISH version.

| Windows                     | Linux                                |
|:---------------------------:|:------------------------------------:|
| Download: [Windows .zip]    | Download: [Linux .zip (only .hex)]   |

Windows .zip contains the .hex file (OpenTX 2.2) and avrdude utility.

Linux .zip only contains the .hex file! You need install avrdude via APT, pacman, or any packages manager. 

```bash
$: apt-get install avrdude -y
```

Unzip the files, and open your console on the same directory, type: avrdude.exe

```bash
Z:\th9x-opentx>avrdude.exe
...
  -?                         Display this usage.

avrdude version 5.10, URL: <http://savannah.nongnu.org/projects/avrdude/>
Z:\th9x-opentx>
```

If you see the previous lines on your console, AVRdude works fine!

Now, flash own hex...
------

Connect you own USBasp or AVRisp to main board, and later, connect USBasp to PC.

Type on the console:

```bash
Z:\th9x-opentx>avrdude.exe -c usbasp -P USB -p m64 -b 19200 -U flash:w:opentx-9x-templates-audio-gvars-battgraph-pgbar-en.hex:i
```

If you're using AVRisp, you need changed ...-c usbasp... by ...-c avrisp... and the port is using arduino, in this example use the COM4 port (in linux is ttyACM0 or any other):

```bash
Z:\th9x-opentx>avrdude.exe -c avrisp -P COM4 -p m64 -b 19200 -U flash:w:opentx-9x-templates-audio-gvars-battgraph-pgbar-en.hex:i
```

If you're using Linux, the command is the same, only need type avrdude, deleting dot exe (avrdude <<.exe>>)

```bash
$: avrdude -c usbasp -P USB -p m64 -b 19200 -U flash:w:opentx-9x-templates-audio-gvars-battgraph-pgbar-en.hex:i
```

After flash, the console show:

```bash
...
avrdude.exe: verifying ...
avrdude.exe: 56042 bytes of flash verified

avrdude.exe: safemode: Fuses OK

avrdude.exe done.  Thank you.
```

Now your FlySky TH9x have OpenTX 2.2!

Now disconnect USBasp. Connect the battery to your radio and power on. Consider have a good battery level (OpenTX need some little to flash the eeprom).

First power on!
------

After flashing OpenTX, and on the first power on, the radio send a error code <<EEPROM error>> says. DON'T worry about that! Press any key and wait OpenTX repair it! After that, you radio works fine!

> You don't know how to calibrate sticks? Don't worry! [Check this video!]

> If you need access to Setup menu, or any other? The way to access is Long press the + KEY. Try with other keys (UP, -, DOWN).

> If you need how to configure OpenTX and setup your control, check [OpenTX University.]

ENJOY! :D

[OpenTX]: http://www.open-tx.org/downloads.html
[Banggood]: https://www.banggood.com/Flysky-FS-TH9X-2_4G-9CH-Transmitter-Spare-Part-Motherboard-Mainboard-p-1057427.html
[Linux .zip (only .hex)]: https://github.com/martindevmx/opentx-flysky-th9x/blob/master/opentxt-2.2-th9x-linux.zip?raw=true
[Windows .zip]: https://github.com/martindevmx/opentx-flysky-th9x/blob/master/opentxt-2.2-th9x-windows.zip?raw=true
[OpenTX University.]: http://open-txu.org/home/undergraduate-courses/radio-setup/general-radio-settings/
[Check this video!]: https://youtu.be/BKiYsAQ2AMg
[WinAVR download page]: https://sourceforge.net/projects/winavr/