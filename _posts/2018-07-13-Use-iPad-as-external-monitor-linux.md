---
title: Use iPad as External Monitor in Linux
date: 2018-07-13
layout: post
author: andrew
categories: [ linux ] 
---

Sometimes you just need a bit of extra space on your desktop. Perhaps you're debugging some CSS and think it would be useful to have your developer tools open on the side.  Instead of digging out a second monitor, hooking it up, and having to sit at a desk, why not use your iPad as a second display?  In this tutorial I will show you just how to set that up.

## Overview

We will be setting up an X11 VNC Server to "broadcast" a virtual desktop

We will then connect to the virtual desktop using a VNC client for the iPad.

## Getting Started

First, we need to install the necessary programs that we'll be using. From the host (linux) computer:

```
sudo apt install x11vnc x2x
```

This will install the X11 vnc server and `x2x`, a tool used for keyboard and mouse sharing.

Next, we need to "create" a new virtual display. To do that, we'll use the built-in linux function, `xrandr`

Create a new Modeline. We will be using a screen that is 1920x1080 at 60Hz

```
$ cvt 1920 1080 60

# 1920x1080 59.96 Hz (CVT 2.07M9) hsync: 67.16 kHz; pclk: 173.00 MHz
Modeline "1920x1080_60.00"  173.00  1920 2048 2248 2576  1080 1083 1088 1120 -hsync +vsync
```

Next, make a new mode in xrandr with the above Modeline:

```
xrandr --newmode "1920x1080_60.00"  173.00  1920 2048 2248 2576  1080 1083 1088 1120 -hsync +vsync
```

Now let's find an empty display:

```
$ xrandr | grep disconnected

DP-1 disconnected (normal left inverted right x axis y axis)
DP-2 disconnected (normal left inverted right x axis y axis)
DP-1-2 disconnected (normal left inverted right x axis y axis)
DP-1-3 disconnected (normal left inverted right x axis y axis)
```

We can use any of the above `DP-` connections to make the new virtual desktop. Add the new mode to an empty display:

```
xrandr --addmode DP-1 1920x1080_60.00
```

Now comes the fun part!  Start the new virtual monitor:

```
xrandr --output DP-1 --mode 1920x1080_60.00 --right-of DP-1-1
```

In the above example, we are adding the new virtual output to the right of my current desktop (`DP-1-1`).  You can get the name of the monitor by running `xrandr` without any flags.

Lastly, we'll start the X11 VNC server to show only this extended virtual desktop.

```
$ x11vnc -clip 1920x1080+2560+0 -usepw
```

We need to add 2560 pixels to the clip since that is the width of our current desktop. The `-usepw` flag enables, as you may have guessed, the use of a password to connect to the VNC instance.

This completes the hard part. The X11 VNC server is set up and running. Now we just have to connect to it from our iPad using a VNC Viewer.

Simply download a VNC viewer from the App store and enter your IP address. You should now see your virtual desktop!

## Shutdown

To stop and remove the virtual desktop, first stop the X11 VNC server by pressing `CTRL` + `c` in the running terminal window. To turn off the virtual desktop, run the command:

```
$ xrandr --output DP-1 --off
```