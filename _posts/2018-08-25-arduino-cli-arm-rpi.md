---
title:  "Arduino CLI with ARM host (RPi)"
date:   2018-08-25 00:00:00 -0600
header:
    teaser: "{{ '/assets/img/posts/2018-08-25-arduino-cli-arm-rpi/header.png' | absolute_url }}"
---

Some hours ago, [Arduino.cc] release Arduino Command Line Interface (a.k.a CLI).

> For more details, check this video: [https://youtu.be/3vtIisvxewc]

Basically this the Arduino toolchains. With this tools you avoid write the Makefile every you can need it. The development is more simple and faster than with other solution (avr-gcc and avr-dude). You can edit your Arduino Sketch (*.ino) with any text editor, vim, nano or whatever you use, and later compiler and deploy it. All over from your terminal!

By this example, I show you how to download and configure Arduino CLI on an Raspberry 2 B with Raspbian OS. Theoretically, this guide can used for install in any Linux architecture (with the correct build), in this case ARM.

> IMPORTANT: this version is under development (alpha version), maybe you have a experience a unstable behavior!

First steps
------

First to all, you need download the latest version of Arduino CLI from the [Arduino-CLI repo]:

```bash
cyan0xff@raspberrypi:~ $ wget http://downloads.arduino.cc/arduino-cli/arduino-cli-0.2.0-alpha.preview-linuxarm.tar.bz2
```

After that, you need uncompressed the execute file and rename to arduino-cli:

```bash
cyan0xff@raspberrypi:~ $ tar -xvf arduino-cli-0.2.0-alpha.preview-linuxarm.tar.bz2
cyan0xff@raspberrypi:~ $ mv arduino-cli-0.2.0-alpha.preview-linuxarm.tar.bz2 arduino-cli
```

Now, move arduino-cli to /usr/local/bin and create your default workspace for Arduino (under /home/ user dir). Also you need create a dir (.arduino15/) and a file (.json) inside it to manage the Arduino packages, both under /root/ directory:

```bash
cyan0xff@raspberrypi:~ $ sudo mv arduino-cli /usr/local/bin
cyan0xff@raspberrypi:~ $ mkdir ~/Arduino/
cyan0xff@raspberrypi:~ $ sudo mkdir /root/.arduino15
cyan0xff@raspberrypi:~ $ sudo touch /root/.arduino15/package_index.json
```

Now, check if Arduino CLI is installed correctly:

```bash
cyan0xff@raspberrypi:~ $ arduino-cli version
arduino-cli version 0.2.0-alpha.preview
```

After that, you need download the boards index, type:

```bash
cyan0xff@raspberrypi: $ sudo arduino-cli core update-index
```

After, install the core avr (or whatever you need it):

```bash
cyan0xff@raspberrypi: $ sudo arduino-cli core install arduino:avr
...
Installing arduino:avr-gcc@4.9.2-atmel3.5.4-arduino2
arduino:avr-gcc@4.9.2-atmel3.5.4-arduino2 installed
Installing arduino:avrdude@6.3.0-arduino9
arduino:avrdude@6.3.0-arduino9 installed
Installing arduino:arduinoOTA@1.1.1
arduino:arduinoOTA@1.1.1 installed
Installing arduino:avr@1.6.21...
arduino:avr@1.6.21 installed
```

Arduino CLI is ready to compile and deploy your projects!

Connect your Arduino Board and verified if Raspbian recognized it:

```bash
cyan0xff@raspberrypi: $ sudo arduino-cli board list
Discovering...

FQBN    Port            ID              Board Name
        /dev/ttyUSB0    1A86:7523       unknown
```

> NOTE: on mi case, the Board Name is unknown because is an clone board of UNO

First project
------

Create a new project (remember, you need create the Arduino Workspace previously):

```bash
cyan0xff@raspberrypi:~ $ arduino-cli sketch new BlinkSerial
Sketch created in: /home/cyan0xff/Arduino/BlinkSerial
```

> NOTE: It's very important that skecth.ino is inside in a folder with the same name. This if you want create the project manually (mkdir sketch/ and touch sketch/sketch.ino).

Edit the sketch. This example is a simple Blink LED with an serial message (baud 115200).

```bash
cyan0xff@raspberrypi:~ $ vim ~/Arduino/BlinkSerial/BlinkSerial.ino

void setup() {
  Serial.begin(115200);

  pinMode(LED_BUILTIN, OUTPUT);
}

void loop() {
  digitalWrite(LED_BUILTIN, HIGH);
  Serial.prinln("Hello from Arduino!");
  delay(750);

  digitalWrite(LED_BUILTIN, LOW);
  delay(150);
}
~
~
~
```

Once time you save the sketch, you need compile it:

```bash
cyan0xff@raspberrypi:~ $ sudo arduino-cli compile -b arduino:avr:uno ~/Arduino/BlinkSerial/
...
Sketch uses 1990 bytes (6%) of program storage space. Maximum is 32256 bytes.
Global variables use 208 bytes (10%) of dynamic memory, leaving 1840 bytes for local variables. Maximum is 2048 bytes.
```

> NOTE: The parameter on arduino:avr:<\board> needs to be replaced for your Arduino board version!

Attach Arduino board and upload your code:

```bash
cyan0xff@raspberrypi:~ $ sudo arduino-cli board attach serial:///dev/ttyUSB0 ~/Arduino/BlinkSerial/
cyan0xff@raspberrypi:~ $ sudo arduino-cli upload -p /dev/ttyUSB0 --fqbn arduino:avr:uno ~/Arduino/BlinkSerial/
```

It's all! Your Arduino is ready and execute your code!

![Blink UNO]({{"/assets/img/posts/2018-08-25-arduino-cli-arm-rpi/blink_uno.gif" | absolute_url }})

Open minicom and listen the serial port with the baud 115200:

```bash
cyan0xff@raspberrypi:~ $ sudo minicom -D /dev/ttyUSB0 -b 115200
...
...
Port /dev/ttyUSB0, 00:08:20

Press CTRL-A Z for help on special keys

Hello from Arduino!
Hello from Arduino!
Hello from Arduino!
Hello from Arduino!
Hello from Arduino!
Hello from Arduino!
Hello from Arduino!
Hello from Arduino!
...
```

How to search and install an lib?
------

Arduino CLI provide a tool for search and download the same libraries of Arduino IDE from official repos. With this tool you can download and install any lib if you need it!

> You can use grep for filter the info. Example:

```bash
cyan0xff@raspberrypi:~ $ sudo arduino-cli lib search lcd | grep -i "Name:\|Paragraph:"
...
Name: "LcdEffects"
  Paragraph:  Underlining! Bold! Italics! This library lets you print all these and more on character LCDs.
Name: "LcdProgressBarDouble"
  Paragraph:  Depends on LiquidChrystal library.
Name: "FaBo 212 LCD PCF8574"
  Paragraph:  16x2 LCD I2C module.
Name: "LcdBarGraph"
  Paragraph:  Using the bouned LiquidChrystal library, bar-graph can be displayed in the screen. See demo: http://youtu.be/noXtsvPRwQk
Name: "Parallax LCD"
  Paragraph:  It is known to work with Parallax LCD\'s (27976, 27977, 27979).
cyan0xff@raspberrypi:~ $
```

> The same search, but only show the name lib.

```bash
cyan0xff@raspberrypi:~ $ sudo arduino-cli lib search lcd | grep -i "Name:"
...
Name: "LcdProgressBar"
Name: "Grove - LCD RGB Backlight"
Name: "LcdBarGraph"
Name: "LCDMenuLib"
Name: "SparkFun Color LCD Shield"
Name: "CheapLCD"
Name: "DatavisionLCD"
cyan0xff@raspberrypi:~ $
```

> Arduino CLI commands:

```bash
cyan0xff@raspberrypi:~ $ sudo arduino-cli
Arduino Command Line Interface (arduino-cli).

Usage:
  arduino-cli [command]

Examples:
  arduino-cli <command> [flags...]

Available Commands:
  board         Arduino board commands.
  compile       Compiles Arduino sketches.
  config        Arduino Configuration Commands.
  core          Arduino Core operations.
  help          Help about any command
  lib           Arduino commands about libraries.
  sketch        Arduino CLI Sketch Commands.
  upload        Upload Arduino sketches.
  version       Shows version number of arduino CLI.

Flags:
      --config-file string   The custom config file (if not specified ./.cli-config.yml will be used).
      --debug                Enables debug output (super verbose, used to debug the CLI).
      --format string        The output format, can be [text|json]. (default "text")
  -h, --help                 help for arduino-cli

Use "arduino-cli [command] --help" for more information about a command.
```

[https://youtu.be/3vtIisvxewc]: https://youtu.be/3vtIisvxewc
[Arduino.cc]: https://www.arduino.cc/
[Arduino-CLI repo]: https://github.com/arduino/arduino-cli