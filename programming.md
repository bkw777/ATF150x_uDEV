# To program ATF150x

## Overview
References:  
https://github.com/peterzieba/5Vpld?tab=readme-ov-file#old-approach-wincupl-16v8-22v10-and-atf150x  
https://github.com/peterzieba/5Vpld/blob/main/PROGRAMMING.md#cpld-devices-atf1502-atf1504-atf1508  

There is no short answer, at least for the initial installation & setup.

Just for basic background reference, the normal process originally would have been:  
* Install [WinCUPL and ATMISP](https://www.microchip.com/en-us/products/fpgas-and-plds/spld-cplds/pld-design-resources)  
* In WinCUPL write a `file.pld` and compile it to create a `file.jed`  
* Use ATMISP to program `file.jed` to the device using a [ATDH1150USB](https://www.microchip.com/en-us/development-tool/atdh1150usb) programmer.

If you use Windows and actually have a real ATDH1150USB programmer then you can still do that.

But ATMISP only supports that one programmer and today that is over $90, even though it doesn't do anything special, it's barely any different than a $3 usb-blaster clone inside. There are no clones like for Altera USB-Blaster, Xilinx Platform Cable USB, ST-LInk etc.

And the WinCUPL gui is annoying and crashy, and doesn't do anything special. It's just a basic text editor and front-end to run `CUPL.EXE` which in turn runs `fit150\*.exe`. It's most useful feature is actually just the trivial device mnemonic lookup util to provide the special code name that corresponds to your part number to put on the `Device xxxx;` line in the header of your PLD source file.

So it's more convenient today to:  
* Write a [PLD file](CUPL/leds.pld) in whatever editor you like.  
* Compile it to a JED file using a [script](https://github.com/peterzieba/5Vpld?tab=readme-ov-file#5vcomp-the-cupl-compiler--your-favorite-text-editor-or-ide-16v8-22v10-and-atf150x) that runs cupl.exe without the gui.  
* Use ATMISP or [fuseconv.py](https://github.com/whitequark/prjbureau/blob/main/util/fuseconv.py) to write a SVF file instead of directly programming the device.  
* Use any commmon [FT232R](https://amazon.com/dp/B0CQVB6JFV) or [FT232H](https://www.adafruit.com/product/2264) module and [openocd](https://openocd.org/) to program the device from the SVF file.


## Software Installation & Setup

These 2 workflow recipes, one for linux and one for windows, do at least make it convenient to work once the annoying install & setup is done. Follow either the Linux or Windows workflow recipes from here:
https://github.com/peterzieba/5Vpld?tab=readme-ov-file#5vcomp-the-cupl-compiler--your-favorite-text-editor-or-ide-16v8-22v10-and-atf150x

A few small notes I'll add, following the linux recipe:
* I used a fresh dedicated wineprefix path just for this, not the default `~/.wine`  
* I made symlinks instead of copies of `fit150\*.exe`  
* The directions show an innoextract command that extracts 5 files from the Prochip installer, but then the following directions only explicitly mention copying the 3 `fit\*.exe` files. You need to copy & overwrite all 5 extracted files.

Here is a start to finish install on Ubuntu in 2024.
```
$ sudo dpkg --add-architecture i386
$ sudo apt install wine wine32:i386 winetricks playonlinux innoextract openocd telnet
$ mkdir ~/atf150x
$ cd ~/atf150x
$ wget https://ww1.microchip.com/downloads/en/DeviceDoc/awincupl.exe.zip
$ wget https://ww1.microchip.com/downloads/en/DeviceDoc/ATMISP7.zip 
$ wget https://ww1.microchip.com/downloads/en/DeviceDoc/ProChip5.0.1.zip
$ unzip awincupl.exe.zip
$ unzip ProChip5.0.1.zip
$ export WINEARCH=win32 WINEPREFIX=~/.wine_atmel
$ winetricks mfc40 mfc42
$ wine awincupl.exe
$ echo Supply serial number 60008009 then exit
$ wine "c:/wincupl/wincupl/wincupl.exe"
$ wine ATMISP7_setup.exe
$ innoextract \
 -I app/Prochip/pldfit/aprim.lib \
 -I app/Prochip/pldfit/atmel.std \
 -I app/Prochip/pldfit/fit1502.exe \
 -I app/Prochip/pldfit/fit1504.exe \
 -I app/Prochip/pldfit/fit1508.exe \
 ProChip5.0.1/ProChip5_setup.exe
$ cp -f app/Prochip/pldfit/* ${WINEPREFIX}/drive_c/Wincupl/WinCupl/Fitters
$ cd ${WINEPREFIX}/drive_c/Wincupl/WinCupl/Fitters
$ ln -sf fit1502.exe find1502.exe
$ ln -sf fit1504.exe find1504.exe
$ ln -sf fit1508.exe find1508.exe
$ cd ~/.local/bin && wget https://github.com/peterzieba/5Vpld/raw/main/linux-workflow/5vcomp && chmod 755 5vcomp
$ cd
$ rm -rf ~/atf150x
```

Now to use the stuff that was just installed...

## Hardware

If using an FT232R module  
amazon.com/dp/B0CQVB6JFV  
amazon.com/dp/B0BJKCSZZW  
adafruit.com/product/284  

Wiring:  
FT232R - JTAG  
-----------  
TXD - TCK  
RXD - TDI  
RTS - TDO  
CTS - TMS  
GND - GND  


If using an FT232H module  
adafruit.com/product/2264  
amazon.com/dp/B07T9CPMHT  
ftdichip.com/products/um232h

Wiring:  
FT232H - JTAG  
-------------  
AD0 - TCK  
AD1 - TDI  
AD2 - TDO  
AD3 - TMS  
GND - GND  


In either case, if the uDEV is NOT powered, then also connect VCC to VCC and set the 3V3/5V jumper on the FTDI module to match your chip.  
AS/ASL = 5V  
ASV/ASVL = 3V3  

##  Compile a PLD source to JED

```
$ mkdir leds
$ cd leds
$ wget https://github.com/bkw777/ATF150x_uDEV/raw/master/CUPL/leds.pld
$ 5vcomp leds.pld
```

You should now have a file `leds.jed`

## Convert the JED to SVF

Run ATMISP:
```
$ export WINEARCH=win32 WINEPREFIX=~/.wine_atmel
$ wine "c:/atmisp7/atmisp.exe"
```

File -> New  
Number of devices: 1  
Device Name: `ATF1502ASL`  
JTAG Instruction: `Program/Verify`  
JEDEC File: `leds.jed`  
OK

[x] Write SVF File  
SVF File Name: `leds.svf`

Run  
Exit  

This should produce `leds.svf`

## Program the device with the SVF

Use openocd to program the device with the SVF file.  

The long command below assumes a FT232R usb adapter and a 1502 device.  

If you have a FT232H module, change `-f interface/ft232r.cfg` to `-f interface/ftdi/um232h.cfg`  
(those cfg files are part of the openocd install in /usr/share/openocd)

If you have a 1504 chip, change `0x0150203f` to `0x0150403f`  
If you have a 1508 chip, change `0x0150203f` to `0x0150803f`  

```
$ openocd \
 -f interface/ft232r.cfg \
 -c "transport select jtag" \
 -c "jtag newtap tap0 tap -irlen 3 -expected-id 0x0150203f" \
 -c init \
 -c "svf leds.svf" \
 -c shutdown
```

--------------------------------------------------------------------------

The device is now programmed with the code from `leds.pld`

Install the uDEV board in a breadboard and supply power to VCC & GND (or let the usb module supply power via the jtag connection)

Insert a wire in the breadboard at pin 21.

Touch the other end of the wire to the vcc or gnd rails alternately.

Both LEDs change state depending on whether pin 21 is high or low.
