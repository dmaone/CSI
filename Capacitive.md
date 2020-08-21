# Capacitive stuff, IBM or otherwise

## Only switch-specific things are here. Most of the doc is in [Common.md](Common.md)!

# Planning: pinouts

* It is STRONGLY RECOMMENDED to set matrix size to your physical matrix size.
* YOU MUST USE CONTIGUOUS ROWS/COLUMNS, STARTING FROM ZERO.
* Pins on the board are marked `0.0`, on envelope - `P0_0`, and in PSoC Creator - `P0[0]`. All 3 refer to the same pin. `2.3`-`2.7` means "2.3 to 2.7, including".
* Effort is made to make pins contiguous, but that's not always possible, so pay attention.

### Default pinout: Project "8x24"

![default pinout](docs/8x24_pinout.png)
**WARNING** to use column 24, **R5** (between `0.1` and `12.6`) MUST be removed.

If you don't - column will always read zeroes. Which is, for a NORMALLY_HIGH switch like Beamspring, means they will appear **PRESSED ALL THE TIME**.

* D0: `0.2`. Must be connected to the nearest `GND`.
* Guard: `0.3`. Must be connected to the nearest `GND`. Separate wire is **HIGHLY RECOMMENDED**
* ADC Vref: `0.4`. Must be connected to the nearest `VDD`.
* Rows: `2.0`, `2.3`-`2.7`, `12.7`, `12.6`
* Cols: `1.0`-`1.7`, `3.0`, `3.1`, `3.3`-`3.7`, `15.0`, `15.1`, `15.5`, `0.0`, `0.1`, `0.5`-`0.7`, `15.4`.

### If you're changing the pinout
* You'll need 35 pins. That's not much more than the kit has - so choose the layout wisely.
* rows are top(`Rows0[0]`) to bottom(`Rows0[7]`), columns are left(`Cols[0]`) to right(`Cols[23]`).
* Keep in mind that in FlightController everything is 1-based so normies can use it, so rows will be 1 to 8.
* P12(all pins of it) cannot be used for analog connections. That means you can only assign Rows to those pins.
* It is better to separate Rows and Cols by a pin electrically connected to the ground, but not necessary. If you use adjacent pins - one matrix cell will have higher readings, that's all.


## Additional Hardware configuration hints
* If key monitor only shows zeroes, no matter which keys you press - set ADC resolution higher.
* If even at 12 bits it still doesn't work - increase charge delay. Recommended setting is 18.
* If several keys fire at once - set longer discharge delay. I found that 180 (10 microseconds) works pretty well, but you can go even higher for increased EMI rejection.

## Configuring thresholds
**NOTE** for beamspring, invert direction, so max->min and adjustments are negative.

**!!!NOTE!!!** If you see double presses on some keys, set threshold higher - just under the steady pressed state. Some keys have physical bounce and there are two peaks. Controller is too fast and sees this as 2 keypresses.

Short version ():
* Click "Key Monitor" button. 
* Click "Start!". Get the idea of levels that should be there - press keys, observe readings going up and down. Small numbers below 7-segment indicators are min/avg/max.
* Click "Stop!". Select "Max" into dropdown near the "reset" button, click "Reset", "Start!". 
* Wait 15 seconds or longer, while readings stabilize.
* click "Stop!". Click "Set thresholds". Close window.
* Click "Thresholds" in the main window. put a small positive value (see below) into adjuster spinbox, click "Adjust".
* Click "Apply".
* Close threshold editor. Select "Config -> Upload" in menu. Test. Once you're satisfied with results, "Command -> Commit".

Thresholds should be set ~2x higher than most of the matrix settles on. For beamspring - probably 75% of the highest reading.
**TEST SETTINGS BEFORE COMMITTING**. If you get thresholds wrong - there will be a red "UNSAFE" light in the status bar and keyboard will refuse to produce output. It's supposed to retest settings after you upload. If it doesn't - click "Scan", it will try to restart scanning.

### Debugging the matrix
If something is wrong - you can still use matrix monitor using the following trick:
* open matrix monitor,
* make sure the mode is "Now"
* click "Start!"
* go to main window
* click "Scan"

The numbers under the "LCD" numbers are min/avg/max since last reset. Looking at those will tell you A LOT about what controller sees - but not everything, because in normal mode controller processes about 30 thousand rows per second and there is no way to transmit that amount of data over USB 2.0.

**WARNING** exit this mode in the reverse order (or by pulling out the USB plug), otherwise you'll likely have to reboot.

This is mostly useful for beamsprings - because all keys are down in beamspring by default. Model F is much easier - everything is up and you can just press a key to observe the effect. So, recommended debugging strategy - pull out the PCB, lay it down over grounded conductive surface, start matrix monitor, put one flipper (or a small coin) over the PCB and observe the changes. If several rows/columns change at the same time - you have a short between those, find it. If readings are normal but assembled keyboard is unstable - it's likely a ground problem.
