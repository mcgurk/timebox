# mcgurk notes in 25.5.2020:

## Packet structure:
0x01 | SL SH | Payload | CL CH | 0x02
--- | --- | --- | --- | ---
Fixed | Packet length low/high | Command+Parameters | Checksum low/high | Fixed

If there is 0x01, 0x02 or 0x03 between start and end byte, they are masked like this:
| byte | masked version |
| --- | --- |
| 0x01 | 0x03 0x04 |
| 0x02 | 0x03 0x05 | 
| 0x03 | 0x03 0x06 |

### Example:
01 | 08 00 | 45 | 00 01 f0 00 00 | 3e 03 04 | 02
--- | --- | --- | --- | --- | ---
||| Switch view | clock, 24h, R, G, B | (0x01 masked to 0x03 0x04) ||

```
./timebox.py --address 11:75:58:C5:2D:6F --debug raw --payload 450001f00000
-> ['01', '08', '00', '45', '00', '03', '04', 'f0', '00', '00', '3e', '03', '04', '02']
```

## Commands I have got so far:
| Command | Parameters | Optional parameters | What it does | Example from real communication with Divoom Android App |
| --- | --- | --- | --- | --- |
| 0x18 | YL YH MM DD HH MM SS 00(?)| | Set date and time | 010b001814140518103a1000c20002 |
| 0x44 | 00(?) 0a(?) 0a(?) 04(?) +182 bytes | | Show image 
| 0x45 | 0x00 | MM (12h=0x00/24h=0x01) R0 G0 B0 | Clock view | 010900450003044b00ff0099030402 |
| 0x45 | 0x01 | MM (C=0x00/F=0x01) R0 G0 B0 | Weather view | 010900450304004b000304009b0002 |
| 0x45 | 0x02 | R0 G0 B0 BB (brightness) 00 (?) | Light view | 010900450305f030006400d4030402 |
| 0x5f | TT WW | | Set temperature and weather | 0105005f0703056d0002 |
| 0x71 | 0x00 | | Stopwatch view | 0104007100750002 |
| 0x71 | 0x01 | | Scoreboard view | 010400710304760002 |
| 0x72 | 0x00 | 00(stop)/01(start)/02(reset) | Stopwatch functions | 01050072000304780002 (start stopwatch) |
| 0x72 | 0x01 0x01(?) BL BH RL RH | | Scoreboard functions | 01090072030403040305000a00890002 (10-2, blue top, red under) |
| 0xab | ML MH | | Sleep after (parameter in minutes, low/high) | 010500ab1e00ce0002 (0xce = 30 minutes) |

### Image dataformat
12bits/pixel -> 1452bits, last nibble 0 -> 1456bits = 182 bytes. Bytes inside 16bit word are swapped. 2 pixels as 3 bytes: GR RB BG

### Weather animations:
| Code | Animation |
| --- | --- |
| 0x01 | Sunshine, clear sky |
| 0x02 | Sun + one cloud |
| 0x03 | Sun + two clouds |
| 0x04 | Sun + three clouds |
| 0x05 | Raining |
| 0x06 | Raining, but sun is peeking behind cloud |
| 0x07 | Rain and thunder |
| 0x08 | Snowing |
| 0x09 | Fog |
| 0x0a | Night, clear night (moon+stars) |
| 0x0b | Night, two clouds |
| 0x0c | Night, three clouds |
| 0x0d | Night, raining |
| 0x0e | Night and raining, but sun is peeking behind cloud (sun in night?) |
| 0x0f | Night, rain and thunder |
| 0x10 | Night, snowing |
| 0x11 | Night, fog |
| 0x12 | Clock(?) |
| 0x13 | Color noise(?)|
| 0x14 | Counts from 2 to 9 with graphical numbers(?) |
| 0x15 | (and 0x00, 0x16...) "demo"/mixed animations |

------------------------------------------------------------------------------------------------------

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
