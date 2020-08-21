# Things that are applicable to firmware in general, not switch-specific

### ExpHdr (AKA "Solenoid connector")
Configured to blink the kit's LED on keypress. `12.4` is a CapsLock LED, `12.3` - NumLock.

**Leave it alone, unless you really know what you're doing. You can permanently damage GPIOs if you fuck up here.**

DO NOT POWER ANYTHING FROM GPIOs. Max GPIO source current is 4mA. FOUR. MILLIAMPERES.

So, `ExpHdr0`(`12.3` by default) would be "enable" line, and `ExpHdr2` (`2.1`) is the "fire" line. Delay settings are in "Hardware" section of FlightController. `ExpTgl` "USB scancode" toggles between normal operations and "all GPIOs are pulled to the ground".


# Soldering

I recommend flashing initial firmware plus some safe configuration first. It's just easier to plug the empty kit in - for initial flashing you'll have to plug the whole kit into standard USB socket, and that might be hard with a big hunk of metal attached to it.

### SOLDER THE KIT SO YOU CAN SEE THE CHIP.

It's in CAPS because I was stupid enough to solder it chip (and LED) facing backplate and blinking LED debugging without seeing the led is pain.

### Grounding considerations
Ground. Does. Not. Matter.

As long as you connected ALL the metal parts adjacent to PCB **and** PCB ground with ANY of the GND pins of the kit - you are fine. Stop worrying about it.

If your PCB doesn't have any vias - MAKE SURE YOU CONNECTED ALL GROUND TRACES ON BOTH SIDES OF THE PCB to controller ground. Floating traces are bad, mmkay.

Except if you see very unstable readings (matrix monitor over 15 seconds with no keypresses shows something like 1/50/255). Then you _probably_ have a ground quality issue.

When in doubt - use star ground. That is, connect all the things that must be grounded (basically, ALL THE THINGS except rows and columns) to a single point, using different wires.

## Hardware configuration hints
* If you're using Linux and can't connect - run as root OR fix your device permissions. No, I don't find fucking with your udev enjoyable, thank you very much.
* If you see the red UNSAFE in the bottom right corner and what to know why - see "Debugging the matrix" in switch-specific README
* If you see double actuations on keypress - increase debouncing steps.
* If you haven't touched ExpHdr pins - "Solenoid" mode and drive time set to 100 will make a LED on the kit blink with every keypress (not in setup mode). Smaller setting will decrease brightness.


# Flashing "default" firmware
This section is removed to stop causing stupid questions.
You MUST build. Deal with it.


# Building custom firmware
You'll need Windows machine. Mac with VirtualBox will do - but a couple of notes there.
You'll need to download [Virtualbox](https://www.virtualbox.org/wiki/Downloads) (v5.2.2 was used) - don't forget the extension pack - and a [windows image](https://developer.microsoft.com/en-us/microsoft-edge/tools/vms/) - I used "MSEdge on Win10". Download, unpack, click the .ovf, wait for import to complete. Then open settings (right-click, "Settings"), click "Ports", select "USB" tab, enable USB 2.0 controller, click the plug with blue dot. THIS WILL CONNECT ALL NEW USB DEVICES TO YOUR VM. Run the VM now. You have your windows development environment now. Download PSoC creator into VM, install (typical works just fine), and continue.
Linux with Virtualbox will probably work too - but not tested as I don't have any hardware running Linux and running virtualbox inside virtualbox is not something I plan to do anytime soon.

Download and install [PSoC Creator](http://www.cypress.com/products/psoc-creator-integrated-design-environment-ide) - you actually can download without suffering akamai download manager. Typical install works just fine. v4.3 was used, though newer should work too.

### Bootloader
You likely don't need to change bootloader - but building it will verify that your development environment is set up correctly.
* Open PSoC Creator
* File -> Open -> Project/Workspace, find "CommonSense.cywrk" and open it.
* Select "Project 'Bootloader'" in "Workspace Explorer" on the left, right-click, "Set as Active Project".
* **IMPORTANT:** Select "Release" in the combo box on the top toolbar that likely displays "Debug".
* Right-click again, "Device Selector", find and select "CY8C5888LTI-LP097". It's likely selected already.
* Press Shift-F6 to build bootloader.

#### Important troubleshooting notes
* "DTD prohibited for security reasons" means "you used ClownZip, and it couldn't decompress the .zip"
* "Unable to find component X" - right-click on top-left item in the left tree editor ("Workpace 'CommonSense' (7 Projects)"), click "Update Components". It should download the missing parts.
* Compilation errors: pay attention to the **IMPORTANT** warning. If it says "Release" - ask at DT, but see the top paragraph. You get support that you paid for (and you can't pay and there are no plans for that).

### Firmware
I will use "8x24" as a project name, but if you don't want to have merge conflicts in config.h every time you pull - you can copy+rename the project. You will only copy programmable hardware settings (which will not change - I run that for 1.5 years now and never had to) and config.h. Everything else will NOT be copied, so you will get all the updates.

Do a smoke test first: Repeat the "Bootloader" part for the 8x24 project.

When that succeeds - it's a good time to make your firmware customizations.

`CommonSense/8x24.cydsn/config.h` is a config file. You can set number of rows, columns and layers there, as well as switch type.
Again: **build without any changes first**!

For non-default pin mapping: Expand a project in the left pane, click "Pins" in "Design Wide Resources". You will see chip model and a table in the big window. Assign pins according to plan.

The Big Moment: plug the kit in. The "gold fingers" part goes into the USB socket.

Press "Ctrl-F5" (Debug -> Program). PSoC Programmer will likely demand to update KitProg - it will walk you thru. Refer to "Flashing "default" firmware" section for troubleshooting tips.

---
It's time now to disconnect the board, peel that polyimide film off the micro USB socket, find the micro-USB cable in the cable mess under your table, and plug it into "Target" part of the kit.

..I was able to plug _both_ ends into the same USB hub (thinkpad docking station) without frying anything - but your mileage may definitely vary.

You may optionally break the kitprog away in a symbolic gesture, but a) if I were you, I wouldn't - not until you set everything up successfully at least - and b) cut the laminate at the snap-off line first. Doesn't have to be all the way (though you certainly can if you're looking for excuse to dremel things), but enough so the board doesn't buckle so horribly and I2C/RS323 traces are not lifted from the substrate. Oh, and soldering probably have to wait until this moment at least.


# FlightController

FlightController is an utility which allows runtime configuration of keyboard parameters

The building process is quite involved (See Qt-build/README.md in CommonSense) - at least on Windows. OS X requires homebrew AND XCode, but is scriptable.

There are Windows and OS X version in the [Releases](releases) section of CommonSense repo.


## Configuring layouts
Pretty straightforward. If thresholds are configured, pressed keys will be highlighted by "yellow highlighter" color and focused on. If you have another keyboard - pressing a letter there will change combo box value to that letter, allowing for VERY quick layout generation.

"Layer mods" configures what Fn/LLck buttons do. Press F to ~~pay respects~~ scroll to those - several times. It will cycle thro all USB scancodes starting with "F".

"Import" and "Export" will load and save to file. Format is compatible with xwhatsit layout files.

IF you see keypresses in setup mode, but not in normal - check out global.h, #define NOT_A_KEYBOARD. If it's 1 - change back to zero.

## Macros and delays
Figure it out yourself, please. It's not that hard. Delays are the _library_ of delays which will be used in macros.
