# Marlin 3D Printer Firmware for Flying Bear 4S and 5

This is the configuration [official Marlin](https://github.com/MarlinFirmware/Marlin) for Flying Bear Ghost 4S and 5 printer. MKS Robin Nano 1.x, MKS Robin Nano v2, MKS Robin Nano-s v1.3, MKS Robin Nano v1.3 boards are currently supported

This repository has several branches:

* [FB4S_WIFI](https://github.com/Sergey1560/Marlin_FB4S/tree/FB4S_WIFI) - this is the master branch. This thread contains additional code to work with the [MKS WIFI](https://github.com/makerbase-mks/MKS-WIFI) module installed in FB4S and FB5. Uploading files through a standard plugin in Cura. Classic Color UI interface.
* [vanilla_fb_2.0.x](https://github.com/Sergey1560/Marlin_FB4S/tree/vanilla_fb_2.0.x) - branch based on Marlin 2.0.x branch. No code changes. All changes are only in configuration files, for robin nano boards and Flying Bear printers. Classic Color UI interface. WIFI module is not working.
* [MKS_UI](https://github.com/Sergey1560/Marlin_FB4S/tree/MKS_UI) - branch based on 2.0.x Marlin branch. There is a very small buffer size change in the code to build on STM32F1. There are no changes in the code on STM32F4. All changes are only in configuration files, for robin nano boards and Flying Bear printers. MKS UI interface. WIFI module is working.

If you have any questions about setting up the firmware or using it, you can ask your question in the [telegram group](https://t.me/Ghostbustersss).

## MKS WIFI module

### Works

* Temperature display in Cura
* View SD card content
* Deleting files from SD card
* Download files to SD card
* Automatically start printing when a file is loaded.
* WIFI module setting (network and password)

### Does not work

* **File names in Russian** Rename the file in Cura
* Only works with SD card v2.0 and later. These are all cards from 1GB and more.
* Printer status display (printing, not printing) in Cura

## How it works, how to set up

### Firmware options

If you are satisfied with the typical configuration option, you can take ready-made firmware files in the [Releases] section (https://github.com/Sergey1560/Marlin_FB4S/releases)

You can return the standard firmware at any time. Just write it to SD and turn on the printer. You can get the standard firmware for the required board [here] (https://sergey1560.github.io/fb4s_howto/mks_board/)

To customize to your needs, you need to assemble the firmware yourself.

The Robin Nano-s v1.3 and Robin Nano v1.3 board is made on a different microcontroller (stm32f407), so to build the firmware for this board, you need to change:

* In the Marlin/Configuration.h file, the MOTHERBOARD parameter:

```C
#ifndef MOTHERBOARD
  #define MOTHERBOARD BOARD_MKS_ROBIN_NANO_V1_3_F4
#endif
```

* In the platformio.ini file, specify mks_robin_nano_v1_3_f4 in the default_envs parameter

### The first thing to do after flashing

The first thing to do after the firmware is to initialize the EEPROM (memory inside the printer), resetting the default settings. After the firmware, there is garbage there, which can lead to completely inexplicable behavior.

This is done through the menu Configuration -> Advanced settings -> Initialize eeprom.

### How to build the firmware yourself

[Video](https://www.youtube.com/watch?v=HirIZk0rWOQ) by Dmitry Sorkin

The board, Robin Nano v1.1 (1.2), is already selected as the default board. For Robin Nano-s v1.3 and Robin Nano v1.3 boards, you need to change the build parameters (described above).

In the Platformio menu, you can not select a board, but use the keyboard shortcut Ctrl + Alt + B.

After compilation, the finished firmware is in .pio/build/mks_robin_nano35/Robin_nano35.bin for Robin Nano v1.1(1.2) boards and in .pio/build/mks_robin_nano_v1_3/Robin_nano35.bin for Robin Nano-s v1.3 and Robin boards Nano v1.3

It is Robin_nano35.bin that needs to be written to the SD card, not firmaware.bin

### What you need to configure if you assemble it yourself

You need to set the direction of movement along the axes to suit your drivers in the file [Configuration.h](./Marlin/Configuration.h) (parameters INVERT_?_DIR, line 1373).

For convenience, the [Configuration.h](./Marlin/Configuration.h) file already contains ready-made sets of settings for all typical configurations.

For Robin Nano v1.1(1.2) boards:

* ALL_DRV_2208 - 4 TMC 2208/2209 drivers
* FB_4S_STOCK - 4 A4988 drivers. This is the configuration for FB4S with standard drivers.
* FB_5_STOCK - 2 TMC 2208 (on the X, Y axes) and 2 A4988 (on the Z, E axes)

For Robin Nano v1.3 boards:

* FB_5_NANO_S_V1_3 - for Robin Nano-S v1.3 board
* FB_5_NANO_V1_3_4TMC - Robin Nano v1.3 with 4 TMC 2208/2209 drivers
* FB_5_NANO_V1_3 - Robin Nano v1.3 with 2 TMC 2208/2209 drivers and 2 A4988 drivers

On line 1322, you need to select only one of the options:

```C
#define ALL_DRV_2208
//#define FB_4S_STOCK
//#define FB_5_STOCK
//#define FB_5_NANO_S_V1_3
//#define FB_5_NANO_V1_3_4TMC
//#define FB_5_NANO_V1_3
```

### WIFI settings if you are using prebuilt firmware

The network settings are stored in the ESP module itself. There are several customization options:

* If the module has already been configured, then perhaps no u don't need a setting
* If the module was not configured, or for some reason could not connect to the network, then it will start in access point mode with the network name MKSWIFI??? (instead of ? there will be arbitrary characters). Connect to this network, open the page at 192.168.4.1 and set the desired network settings.
* If you compile the firmware yourself, it is possible to transfer settings to the module at startup. To do this, in the file [mks_wifi_settings.h](./Marlin/src/module/mks_wifi/mks_wifi_settings.h) you can set the WIFI network parameters.
In order for these settings to be applied at power on, "MKS_WIFI_ENABLED_WIFI_CONFIG" must be enabled.

### WIFI status

Upon successful connection to the network (or creating a network in access point mode), the standard UART, which is output to the USB connector of the printer, will display the IP address and network name, and the IP address will be displayed on the printer screen.

### How to understand that WIFI is working

When the printer is turned on, the screen will display the status "WIFI init"

If the ESP module succeeded in connecting to the network, the IP address will be displayed on the screen.

When the file transfer starts, "Upload file" is displayed, while the upload progress is displayed as a percentage.

If the file is successfully accepted, "Upload done" will be displayed and **one beep will sound**

If there were errors while uploading the file, "Upload Failed" will be displayed and **three beeps will sound**

###BLTouch

Bltouch is disabled. [About connecting Bltouch](https://sergey1560.github.io/fb4s_howto/bltouch/).

### Firmware retract

Without using the "firmware retract" option, the slicer makes retracts with G1 motion commands. In the place where you need to execute the retract, the commands are inserted:

```
G1 E-2 F2100 ; Roll back 2mm at 35mm/s (2100mm/min)
movement commands
G1 E2 F2100 ; Return back 2mm at 35mm/s (2100mm/min)
```
Special commands for retracts in Marlin are [G10](https://marlinfw.org/docs/gcode/G010.html) and [G11](https://marlinfw.org/docs/gcode/G011.html). In the slicer, you need to enable support for firmware retract and then the G10 command will be inserted in the place where you need to "roll back" the plastic, and G11 where you need to return it. If no additional parameters are set, the parameters from the firmware (2mm, 35mm/s) will be used.

You can set the parameters with the commands [M207](https://marlinfw.org/docs/gcode/M207.html) and [M208](https://marlinfw.org/docs/gcode/M208.html).

In order to be able to configure the retract in the slicer, you need to add M207 to the start code. Typically, slicers allow you to add macros as command parameters.

Firmware retract allows you to change the retract values ​​from the printer menu while printing.

Marlin has a function to automatically recognize retracts with G1 commands and replace them with G10/G11. This feature is disabled.

If your slicer does not have firmware retract enabled, everything will work as usual.


### Driver TMC2209

By default, the firmware is configured to work with stepper motor drivers without software control. If you use the TMC 2209 or TMC 2208 drivers, you can enable UART control. Learn more about [configuring and connecting](https://sergey1560.github.io/fb4s_howto/tmc_uart/).

### EEPROM

There are 2 flash memory chips installed on Robin Nano boards: AT24C16 (2kb, connected via I2C) and W25Q64 (connected via SPI).

The size of the data that is stored in the EEPROM depends on the enabled options. When saving settings with the M500 command, the response contains the size of the saved data.

There are several options available for EEPROM storage in Marlin:

* SD card
* I2C EEPROM. This option is not used, the driver is disabled.
* SPI_EEPROM. Storage in W25Q64BV connected via SPI. This option is used by default.
* FLASH_EEPROM_EMULATION. This is EEPROM storage in STM32 flash memory. This option doesn't work.
* SRAM_EEPROM_EMULATION. This option doesn't work.

To include in [Configuration.h](./Marlin/Configuration.h) in the EEPROM section, you need to specify the desired define. Possible options are indicated in the comments. Example:

```C
#if ENABLED(EEPROM_SETTINGS)
/*
MKS Robin EEPROM:
EEPROM_SD
EEPROM_W25Q
*/
#define EEPROM_W25Q

#if ENABLED(EEPROM_W25Q)
#undef SDCARD_EEPROM_EMULATION
#undef USE_REAL_EEPROM
#undef FLASH_EEPROM_EMULATION
#undef SRAM_EEPROM_EMULATION
#undef I2C_EEPROM_AT24C16
#define SPI_EEPROM_W25Q
#define SPI_EEPROM
#define SPI_EEPROM_OFFSET 0x700000
#define USE_WIRED_EEPROM 1
#define MARLIN_EEPROM_SIZE 2048
#endif

#if ENABLED(EEPROM_SD)
#define SDCARD_EEPROM_EMULATION
#undef USE_REAL_EEPROM
#undef FLASH_EEPROM_EMULATION
#undef SRAM_EEPROM_EMULATION
#undef I2C_EEPROM_AT24C16
#undef SPI_EEPROM_W25Q
#undef USE_WIRED_EEPROM
#define MARLIN_EEPROM_SIZE 4096
#endif

#define EEPROM_AUTO_INIT // Init EEPROM automatically on any errors.
#endif
```

To change the storage location of the EEPROM, you need to replace "#define EEPROM_W25Q" with another option.

### Download firmware via WIFI

It is possible to send the firmware to the printer via WIFI. To do this, in the file [platformio.ini] (./platformio.ini) in the [env:mks_robin_nano35] section, you need to specify the IP address of the printer in the upload_flags option.

The file is transferred using curl, so you must either add curl to $PATH or specify the full path in the file [mks_robin_nano35.py](./buildroot/share/PlatformIO/scripts/mks_robin_nano35.py) on line 43.

After configuration, to send the firmware to the printer, select Upload in the platformio menu or press Ctrl+Alt+U.

After a successful file transfer, the printer will restart automatically.

## WIFI module, sending commands and files

You don't have to use Cura to send commands and files to the printer. For sending, you can use simple tools - curl and netcat.

To send commands, use tcp socket on port 8080. An example with netcat:

```
nc 192.168.0.105 8080
```

You can use telnet instead of netcat.

You can send a command g-code, and receive a response.

You can use curl to send files:

```
curl -v -H "Content-Type:application/octet-stream" http://192.168.0.105/upload?X-Filename=sd_file.gcode --data-binary @local_file.gcode
```

* *sd_file.gcode* - the name of the file under which will be saved on the sd card
* *local_file.gcode* - file name to send

In this example, the local_file.gcode file will be sent to the printer with IP 192.168.0.105, which will be saved on the sd card under the name sd_file.gcode

## Load settings into EEPROM from file

When updating the firmware, it is recommended to reset the settings to the default value and install them again. In order not to do this manually with each update, you can create a file with the necessary commands on the sd card and simply print it. An example of a settings file:

```
M502 ;Reset settings
M500 ;Save settings (similar to Initialize eeprom)

M92 X80 Y80 Z400 E421 ;Axes Step/mm setting

M301 P19 I1 D64 ;PID nozzle
M304 P26 I4 D102 ;PID table

M851 X37 Y-20 Z-0.95 ;Probe offset

M906 X700 Y800 Z800 ; Stepper motor driver current
M906 T0 E450

M603 L150 U150 ;Filament loading/unloading length
M500 ;Save settings
```

## WIFI print status monitoring

During printing, data reception from the WIFI module is disabled. This is done so that no garbage from esp gets into the command queue. However, in the opposite direction, from MK to esp, the transmission works. Therefore, if you need to monitor the print status remotely, you need to add the command [M155](https://marlinfw.org/docs/gcode/M155.html) to the start code to display the temperature and [M27](https://marlinfw.org/ docs/gcode/M027.html) to show print progress in bytes. In this case, the MK itself, after the number of seconds specified in the parameters, will send reports. You can receive them by connecting to a socket on port 8080. The MKS WIFI module only supports one connection at a time, so Cura must be closed.

To get information about the current height, you need to add post-processing in the slicer. In Cura, this can be done in Extentions->Post processing->Modify G-code. Add script to "Insert at layer change" and command M114.
