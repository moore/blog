---
title: "Penkesu"
date: 2024-11-03T15:54:45-08:00
draft: true
---

I often find myself sittan waiting for one thing or another and
intherse intersitial moments succumbing to impulse to conume
"content". Instead I would much rather be creating, but find the
device at hand, my phone, to be designed only for consumption.

In an attempt to improve this situation I am building a series of
potable devices made for input. Penkesu is the first. It is a build of
anothers's design and I am using it as a path finder device that will
teach me more about what I want.

## Missing Build Instrctuions

If you want to try to build one yourself fallow the instructions on
the Penkesu github. My notes on the build are:

 1. The bill of materials omits the needed USB cables.
 2. It can be very hard to fit a usb connector for the keyboard in the case.
 3. A switch is needed for power on/off. 
 4. The screen is very heavy and screen will not stand on it's own for me.
 5. It is not clear what orientation the hinges should be installed in.
 
### Missing USB Cables

#### Display Power

To assemble the computer you will need a power only cable with a usb
mirico connector to supply the display. I would suggest making this
yourself to minimise the diamater of the cable as it has to get
through the hinge along with the DVI ribben cable.

I assembled mine out of these connectors and sicilcone wire. The other
end of the cable attaches to the power output on the board.

#### Keyboard Cable

A cable is needed between the mircocontoler on the keyboard and the
miroco USB conector on the rpi. I made this cable using the same
connector I used to power the display. I found due to the limited
space in the case I needed to make my own connector agine.

### USB Connector on Keyboard

I started with the Ardino Pro Mrico that the original project
used. Mine unlike theirs came with a USB-C connector. I could not find
a male connector or cable that would fit in the case. It might have
worked if I had a Pro Mrico with a mrico usb connector, but in the end
I switched to a Adafruit KB2040 which has the D+ and D- pins broken
out. This let me skip the connector completely and solve the space
issue.

The KB2040 brought issues of it's own. The QMK ferimware for the
keyboard has not been maintained and would not compile. I have never
worked with QMK and chose to write a quick driver with KMK instead of
trying to modernize it. You can find that here.

### Power Switch

A switch is needed for power. I used one of these.

### The Heavy Display

The display is clearly designed for non mobile applacations. It has
heavy glass, metial back plate with standoffs. The contoler is also
mounted to the back. I think the vendor expects you to mount your rpi
to the display. This means that the display is about 1cm thick and
most of the wait of the device. My screen will not sit at a useful
height unless there is something proping up the hinge. This might also
be becouse I did not correctly install the hinges.

### Hinge Orentation
 
The DS hinges used have detents in them but it is not clear what the
correct rotation for the display is. I glued mine in so the detent was
in the "latched" state when closed. This might have been wrong there
were no notes on the correct way to do this.

## Other Notes

### Cable managment and  JST Connectors

I connectorized a coupple of the cables with JST connectors. This did
help he when working out the design but I have found them to make it
harder to fit everything in the 1cm space under he keyboard.

There are many cables that need to cross over each other. This
combined with the reality that my bratty had to move right as it would
not fit under the keyboard controler meant that managing cables was
fiddely. I should probbly go back in and shorten some of them and
remove the JST connectors.

### Power Managment

There are two issues with power managment:

 1. The rip zero has not sleep mode.
 2. There is no access to the battrie state. 
 
The lowest poser draw from the rpi zero 2 is around 100mw which is
significant. There is also no lid switch to turn off the display or
other devices when closed. The hardware also dose not have a way to
turn off the display thou its possible the back light could be turned
off.

The power control board dose have a low battery pin that is pulled
low. It would be worth adding logic to check that with a rpi GPIO.

There are lights for power, charging, and low bat, on the board but no
pins for them. I might try to bring then to the side with light
pipes. One could also try and move the LEDs to the out side of the
case by desoldering them and adding cables.

### Touch Screen

The display has a touch sensor but the design dose not use it. To
enable it a usb hub like this one would be needed as there is a singe
USB port on the rpi zero. Adding the USB cord from the display and the
USB hub could complacate cable routing further.

### Display Rotation

To rotate the console the fallowing must be added to the kernel command line  in */boot/firmware/cmdline.txt*: 

> video=HDMI-A-1:400x1280M@60,rotate=270

## Final thoughts

I do like the build and have learned lots of things to consider in my
first original design if I get to that.

The device is already working in the role of a portable hack box.

I can say I am glad I built it and would recommend others that are
interestead give it a try if the expincive screen is in there budget.

P.S. I have started collecting notes on my next build hear.