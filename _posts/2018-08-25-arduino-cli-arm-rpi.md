---
title:  "Arduino CLI with ARM host (RPi)"
date:   2018-08-25 00:00:00 -0600
header:
    teaser: "https://raw.githubusercontent.com/martinsgmx/martinsgmx.github.io/master/assets/img/posts/2018-08-25-arduino-cli-arm-rpi/header.png"
---

Basically this the Arduino toolchains. With this tools you avoid write the Makefile and build with Arduino IDE. With this solution you can edit your Arduino Sketch (*.ino) with any text editor, vim, nano or whatever you use, and later compiler and deploy it. All over from your terminal!

> Protip: If you're using vim, you can install [Arduino syntax highlighting]!

On this example, I show you how to download and configure Arduino CLI on an Raspberry 2 B with Raspbian OS. Theorically, this guide can used for install in any Linux architecture (with the correct build: ARM64, x86, x64).

> IMPORTANT: this version is under development (alpha version), maybe you have a experience a unstable behavior!

First steps
------

First to all, you need download the latest version of Arduino CLI from the [Arduino-CLI repo]:

```bash
hst@rpi:~ $ wget https://downloads.arduino.cc/arduino-cli/arduino-cli_latest_Linux_ARMv7.tar.gz
```

After that, uncompressed the file:

```bash
hst@rpi:~ $ tar -xvf arduino-cli_latest_Linux_ARMv7.tar.gz
hst@rpi:~ $ mv arduino-cli_latest_Linux_ARMv7
```

Now, dowload the board index. The package_index.json is a list with all supported by Arduino core.

```bash
hst@rpi:~ $ sudo mv arduino-cli /usr/local/bin
hst@rpi:~ $ sudo mkdir /root/.arduino15
hst@rpi:~ $ sudo touch /root/.arduino15/package_index.json
hst@rpi:~ $ sudo arduino-cli config init
```

After that, you need download the boards index, type:

```bash
hst@rpi: $ sudo arduino-cli core update-index
```

> This step if only you want install arduino:avr -> atmel mcu:m328p:m2560~!

Install the arduino:avr core (or whatever you need):

```bash
hst@rpi: $ sudo arduino-cli core install arduino:avr
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
hst@rpi: $ sudo arduino-cli board list
Discovering...

FQBN    Port            ID              Board Name
        /dev/ttyUSB0    1A86:7523       unknown
```

> NOTE: on this case, the Board Name is unknown because is an clone board.

First project
------

Create a new project:

```bash
hst@rpi:~ $ arduino-cli sketch new BlinkSerial
Sketch created in: /home/cyan0xff/Arduino/BlinkSerial
```

> NOTE: It's very important that sketch.ino is inside in a folder with the same name. This if you want create the project manually (mkdir sketch/ and touch sketch/sketch.ino).

Example sketch.

```bash
hst@rpi:~ $ vim ~/Arduino/BlinkSerial/BlinkSerial.ino

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

Once time save your sketch, you need compile it:

```bash
hst@rpi:~ $ sudo arduino-cli compile -b arduino:avr:uno ~/Arduino/BlinkSerial/
...
Sketch uses 1990 bytes (6%) of program storage space. Maximum is 32256 bytes.
Global variables use 208 bytes (10%) of dynamic memory, leaving 1840 bytes for local variables. Maximum is 2048 bytes.
```

> NOTE: The parameter: arduino:avr:<\board> needs to be replaced by your Arduino board version!

Attach Arduino board and upload your code:

```bash
hst@rpi:~ $ sudo arduino-cli board attach serial:///dev/ttyUSB0 ~/Arduino/BlinkSerial/
hst@rpi:~ $ sudo arduino-cli upload -p /dev/ttyUSB0 --fqbn arduino:avr:uno ~/Arduino/BlinkSerial/
```

It's all! Your Arduino is ready and execute your code!

![Blink UNO]({{"/assets/img/posts/2018-08-25-arduino-cli-arm-rpi/blink_uno.gif" | absolute_url }})

Open minicom and listen the serial port with the baud 115200:

```bash
hst@rpi:~ $ sudo minicom -D /dev/ttyUSB0 -b 115200
...
...
Port /dev/ttyUSB0, 00:00:20

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

Arduino CLI provide a tool for search and install the same libraries of Arduino IDE (official repos).

> Tip: You can use grep for filter the info.

```bash
hst@rpi:~ $ sudo arduino-cli lib search lcd | grep -i "Name:\|Paragraph:"
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
hst@rpi:~ $
```

> The same search, but only show the name lib.

```bash
hst@rpi:~ $ sudo arduino-cli lib search lcd | grep -i "Name:"
...
Name: "LcdProgressBar"
Name: "Grove - LCD RGB Backlight"
Name: "LcdBarGraph"
Name: "LCDMenuLib"
Name: "SparkFun Color LCD Shield"
Name: "CheapLCD"
Name: "DatavisionLCD"
hst@rpi:~ $
```

> Installing a lib, LcdProgressBar:

```bash
hst@rpi:~ $ sudo arduino-cli lib install LcdProgressBar
 0 / 536477 [-----------------------------------------------------------------------------------------------------]  0.00%
 LcdProgressBar@1.0.1 0 B / 523.90 KiB [--------------------------------------------------------------------------] 0.00%
 LcdProgressBar@1.0.1 75.20 KiB / 523.90 KiB [=========>----------------------------------------------------------]  14.35% 
 LcdProgressBar@1.0.1 334.96 KiB / 523.90 KiB [==========================================>------------------------] 63.94%
 LcdProgressBar@1.0.1 downloaded

Installed LcdProgressBar@1.0.1
hst@rpi:~ $
```

And that's it, now you can add the lib in your .ino file!

> Arduino CLI commands:

```bash
hst@rpi:~ $ sudo arduino-cli
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

Use "arduino-cli [command] --help" for more information about any command.
```

[Arduino syntax highlighting]:https://github.com/sudar/vim-arduino-syntax
[Arduino.cc]: https://www.arduino.cc/
[Arduino-CLI repo]: https://github.com/arduino/arduino-cli
