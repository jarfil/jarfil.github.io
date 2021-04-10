---
title: Ender 3v2 BLTouch leveling fix
permalink: new-page.html
description: 
date: 2021-04-10 22:23:27 +02:00
tags: "3dprinter ender3v2 bltouch 3dtouch"
---

# BLTouch doesn't fix it all

So you got a BLTouch or some 3DTouch working, and you're wondering why things aren't printing like they should. It was supposed to automatically compensate any bed irregularities, wasn't it?

Chances are you did nothing wrong, but your printer is. Don't worry, there is a way to work around it.

## How BLTouch works

Automated bed levelling correction works by finding the difference between two data points:

1. Z axis home position
1. Z height that triggers the probe

By probing the Z height of several points around the bed, it's possible to calculate a rough estimate of its shape, and compensate for it during printing.

## Material

* Ender 3 v2
    * Board v4.2.2 with silent drivers
* Ender 3 V2 3DTouch/BLTouch Mount (with wire duct), on [Thingiverse](https://www.thingiverse.com/thing:4657059)
* Makerbase 3DTouch (more on [GitHub](https://github.com/makerbase-mks/Ender3-3DTOUCH))
* [Jyers Marlin](https://github.com/Jyers/Marlin) Firmware
    *  v1.2.1 (E3V2-UBL-BLTouch-10x10-v4.2.2)

Recommended:

* [OctoPrint](https://octoprint.org/) (in [Docker](https://hub.docker.com/r/octoprint/octoprint))
    * plugin: [Bed Visualizer](https://plugins.octoprint.org/plugins/bedlevelvisualizer/)
* [Marlin GCODE](https://marlinfw.org/meta/gcode/)
    * [G29 UBL](https://marlinfw.org/docs/gcode/G029-ubl.html)

## Repeatability

The key to a good correction, is repeatability, or how to get the same measurement for the same thing every time, over and over, hundreds of times, thousands of times.

Every time the probe touches the bed in the same spot, the firmware needs to think it's at the same Z position. Otherwise, you're getting Z axis creep.

Not that the measure doesn't matter if it's reproducible, your bed and sensor can be as misaligned as you like, it just needs to give the same value every time.

## Check for Z axis creep

The simplest check is:
    
    home Z
    for 1 to 100
     probe point
    if last point != 0 then
     alert("Z axis creep!")

In Marling GCODE, that would be:

    G28 ; home X Y Z
    G29 P0 ; zero mesh data
    ; for 1 to 100
    G29 I1 ; invalidate closest point to the nozzle
    G29 P1 C ; probe invalidated point
    ; end for
    G29 T ; output topology
    
You should expect a repeatability error of:

* BLTouch: 0.005~0.015 mm
* 3DTouch: 0.010~0.050 mm

(BLtouch VS 3Dtouch 3D Printer bed leveling sensor match up, on [YouTube](https://www.youtube.com/watch?v=BPH9btHPcbc))

Anything above 0.010 mm will cause some trouble, but an error of 0.500 mm is even worse than having no correction at all.


### Examples of Z axis creep measures

![Automatic bed leveling through LCD](/assets/202104/bed-20210316-error.png)

![G29 P1 bed leveling](/assets/202104/bed-20210316a-graph.png)

![Maximum error per point after leveling 4 times in a row](/assets/202104/bed-20210317-max_error.png)

