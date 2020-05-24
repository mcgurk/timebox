Packet structure:
0x01 | SL SH | CC | PP... | CL CH | 0x02
--- | --- | --- | --- | --- | ---
Fixed | Packet length low/high | Command | Parameters | Checksum low/high | Fixed

Example:
01 | 08 00 | 45 | 00 01 f0 00 00 | 3e 03 04 | 02
--- | --- | --- | --- | --- | ---
 |  | Switch view | clock, 24h, R, G, B |  | 

# Divoom Timebox CLI
Control the divoom timebox using your terminal.
Thanks to derHeinz and his [divoom-aurabox code](https://github.com/derHeinz/divoom-adapter) for giving some hints on how to interpret the protocol.

## Project status
This project is WIP, so not every feature might work as expected yet. Also this tools is communicating with the timebox unidirectional.
This means, that no answers from the box will be processed at the moment.

## What works
* Switching "screens" :  "clock","temp","anim","graph","image","stopwatch","scoreboard
* display images, most formats should be supported thanks to [pillow](https://github.com/python-pillow/Pillow). The images will be scaled to fit the 11x11 matrix
* display animations, either load a series of images from a directory OR a GIF animation. The loaded frames will also be scaled
* display clock and set 12h/24h format as well as color. There are a dozen of possebilities to describe a color, take a look at [colour](https://github.com/vaab/colour)
* display temperature set °C/°F as well as the color
* switch radio on/off
* setting time

## What does not work (or needs improvements)
* Setting the radios frequency does not yet work
* animations with a shorter framelength "as usual" (I am about to investigate this) might be "glued together" resulting in two or more animations shown after each other
* Error handling is bad at the moment, so be careful what you type. The CLI might not inform you about what went wrong yet.
* Python3 support
* ... Documentation ;) ...
* ... No Tests, only tested on my Linux boxes ...

## Windows integration
To use this library on Windows, you need to install a version of the [pybluez](https://github.com/pybluez/pybluez)
library that contains a fix for [getpeername support](https://github.com/pybluez/pybluez/pull/201).
A pre-built binary for the library supporting Python 2.7 and AMD64 can be downloaded [here](https://1drv.ms/u/s!AtLn8ELpA_G9frRzQuMKJ1QixKw).

## Future plans
* split CLI and library
* support text "rendering" and marquees
* support everything that could be done using the "original" android app
* web server providing a timebox api
