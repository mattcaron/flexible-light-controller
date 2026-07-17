# The Flexible Light Controller (FLC)

## Objective

The purpose of this project is to develop firmware for a user-configurable lighting controller (that is, no needing to change or write code) so one can do some soldering, power on the board, access it over WiFi, mess with some settings, hit "Save & Apply" and have some nice [blinkenlights](https://en.wikipedia.org/wiki/Blinkenlights).

## Hardware

This project is based around the [WeMos D1 Mini](https://www.wemos.cc/en/latest/d1/d1_mini.html), or one of its many pin-compatible clones. Many of these are based around the ESP-12 module and are often referred to as "D1 Modules" or "NodeMCU modules". In quantities of 10 or so, they can be had for sub $3 each. You'll want a few.

## Software

### Arduino

Get the appropriate [Arduino IDE](https://www.arduino.cc/en/software/) for your platform. At time of writing, I'm using version 2.3.10.

**Note:** for my platform (Ubuntu 24.04), I downloaded the "Linux ZIP file" option and additional tweaking for the sandbox was required. If you get an error message that states:

    [252708:0429/091203.327243:FATAL:setuid_sandbox_host.cc(158)] The SUID sandbox helper binary was found, but is not configured correctly. Rather than run without sandboxing I'm aborting now. You need to make sure that /home/matt/Arduino/arduino-ide_X.Y.ZZZ_Linux_64bit/chrome-sandbox is owned by root and has mode 4755.
    Trace/breakpoint trap (core dumped)

To fix:

    cd wherever you unzipped it
    sudo chown root chrome-sandbox
    sudo chmod 4755 chrome-sandbox

Then you can just run the `arduino-ide` executable and it should work.

It will likely want to update libraries - I let it do so.

### ESP8266 Core for Arduino

[Follow these instructions](https://github.com/esp8266/Arduino#installing-with-boards-manager)

### Board selection

Once the IDE is launched and updated, Select `Tools -> Board -> ESP8266 Boards -> LOLIN(WEMOS) D1 R2 & Mini`.

### ESP8266 LittleFS Plugin

Follow [this tutorial](https://randomnerdtutorials.com/arduino-ide-2-install-esp8266-littlefs/).

At time of writing, I am using version 1.6.3.

For Linux, the instructions are basically:

1. Download the `arduino-littlefs-upload-1.6.3.vsix` file.
2. `mkdir ~/.arduinoIDE/plugins`
3. `mv arduino-littlefs-upload-1.6.3.vsix ~/.arduinoIDE/plugins`

And then restart the IDE.

You can check that it was successfully installed by doing `CTRL + SHIFT + P` to open the command palette and then typing "Upload". There should now be an entry for `Upload LittleFS to Pico/ESP8266/ESP32`. If not, double check that the file is in the correct place.

### ESP Async Webserver

In Arduino IDE, go to `Tools->Manage Libraries`, and search for "ESPAsyncWebserver". Install the "ESP Async WebServer by ESP32Async". At time of writing, the version is 3.11.2.

### Parcel (to compile the materialize source)

**Note:** This requires `npm`. e.g. `sudo apt install npm`.

Follow [these instructions](https://parceljs.org/getting-started/webapp/#installation) and **install it into the gui directory** which is usually just:

    cd gui
    npm install --save-dev parcel

If you want to do some development and test the UI locally, do:

    cd gui
    npm run dev

To build the UI, do:

    cd gui
    npm run build

the resulting file(s) will be put in `data/` for upload by the LittleFS file uploader.

## Reference documentation

* [Arduino IDE](https://docs.arduino.cc/software/ide/#ide-v2)
* [D1 Mini](https://www.wemos.cc/en/latest/d1/d1_mini.html)
* [ESP8266 Arduino Core](https://arduino-esp8266.readthedocs.io/en/latest/reference.html)
* [ESPAsyncWebserver](https://esp32async.github.io/ESPAsyncWebServer/)

## FAQ

### Why the D1 board?

Pragmatism. After the power consumption issues with the first generation build of [my thermostat project](https://github.com/mattcaron/thermostat) I did some investigation and determined that the LVDO regulator used on the clones had really bad quiescent current draw - measured in mA. The genuine D1 Mini used a really nice regulator which drew current in the uA range. For a wall-powered device, or one only used for a short time, this difference was irrelevant, but for a device which is to hang on the wall for months at a time, it became significant. So, I bought some of the genuine D1 boards and life was good - except that I ended up with a baggie of these clone boards. When the inspiration for this project started, they seemed the right choice. I already had some, they're cheap, and they will either be permanently wired into a model railroad layout, or used for a few hours for wargame terrain, powered either from a USB power bank or some sort of power bank hidden inside a building. As long as one starts with a fresh battery bank, even at the max 500mA current draw, a 10000mA bank will last for 10+ hours and 3 2000mA AA cells will last for 6+ hours, conservatively speaking. Even a 3 800mA AAA pack should get you 3+ hours. And this is worst case, so it should be suitable for our purposes.

### Why Arduino?

I'm going to be honest here - it's not my favorite. As someone pretty familiar with FreeRTOS, Arduino seems kind of limiting, and the 1.x IDEs kind of sucked relative to something like Visual Studio Code. But, it is very approachable and that has value. So, I'm going to try to give the new IDE a fair shake. (Note: I'm writing this at the start of the project, so we'll see how this viewpoint evolves.) Also, the single threaded nature of the application bugs me. Some years ago, [I did some experimentation with a multithreaded ESP32 runtime](https://github.com/espressif/arduino-esp32/compare/master...mattcaron:arduino-esp32:parallel_tasks), but never used it for anything real.

Anyway, for a single-core CPU like the ESP8266, that objection is irrelevant (and, honestly, you could argue that, if you pin the Arduino runtime to 1 core and have the other core do all the WiFi stuff, it's irrelevant for the ESP32 as well, as that is a perfectly fine design).

Still, there are a lot of things that FreeRTOS makes really easy (multiple independent time sensitive tasks, for example) that I expect to be slightly more difficult under the Arduino runtime. As mentioned above, we'll see how that evolves.

## Credits

Much of the project's basic structure, layout, and configuration ideas were cribbed from Benjamin von Deschwanden's excellent project [esp32-signal-generator](https://github.com/vdeschwb/esp32-signal-generator) (which I used to bench test things on my [radio dimmer adapter project](https://github.com/mattcaron/daisy_dimmer).) After all, if you think about it, what is an LED controller besides a signal generator?
