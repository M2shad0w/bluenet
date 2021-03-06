# Indoor Localization with BLE

This project aims at a wireless network with BLE nodes that use their mutual signal strengths to build up a network with their relative locations. This can be used later by someone carrying a smartphone to establish their location indoors. Everybody say that they can do it, but very few solutions are actually out there. Let's hope we can change that.

Bluetooth LE (BLE) does not inherently fit a wireless network. We have technology in-house (at the Almende group) that can do this [Myrianed](https://en.wikipedia.org/wiki/MyriaNed), but it has not been accepted in the mainstream yet. By the way, it is my personal opinion that solutions such as ZigBee, Z-Wave, MyriaNed, and other mesh solutions, will remain marginal except if they get accepted in a common handheld.

That's why BLE is interesting. A lot of phones come with BLE, so a solution is automatically useful to a large variety of people. It is not the best technology for the job. The network topology is a Personal Area Network (PAN), not a Local Area Network (LAN). This means that you cannot have all nodes communicating with all other nodes at the same time. To get RSSI values we will have to set up connections to other nodes and tear them down again. Not very efficient. But it will do the job. 

However, through the Timeslot API, it is possible to run a totally different protocol parallel to BLE using the same radio. This means that we can have a mesh network at the same time as providing for BLE functionality. The new S130 SoftDevice adds to that even multiple BLE roles at the same time. So many possibilities arise!

Feel free to clone this repos.

The code base comes from [http://hg.cmason.com/nrf](http://hg.cmason.com/nrf). Thanks a lot Christopher!

## Installation

The installation should not be hard when you have the Nordic SDK. Get this from their website after buying a development kit. You also need a cross-compiler for ARM. You need the JLink utilities from Segger. And you need cmake for the build process.

* [Nordic nRF51822 SDK](https://www.nordicsemi.com/eng/Products/Bluetooth-R-low-energy/nRF51822)
* [Nordic S110 Softdevice](http://www.nordicsemi.com/eng/Products/S110-SoftDevice-v7.0)
* [JLink Software](http://www.segger.com/jlink-software.html), there is a [.deb file](https://www.segger.com/jlink-software.html?step=1&file=JLinkLinuxDEB64_4.96.4)
* sudo aptitude install cmake

A cross-compiler for ARM is the `GCC` cross-compiler which is maintained by the ARM folks on [Launchpad](https://launchpad.net/gcc-arm-embedded).

    curl -v -O https://launchpad.net/gcc-arm-embedded/4.8/4.8-2014-q3-update/+download/gcc-arm-none-eabi-4_8-2014q3-20140805-linux.tar.bz2
    tar -xvf gcc-arm-none-eabi-4_8-2014q3-20140805-linux.tar.bz2 -C /opt    

This is a 32-bit application, so you will need some dependencies:

    sudo apt-get install libstdc++6:i386 libncurses5:i386

If the cross-compiler does not work, make sure you check if all its dependencies are met:

    ldd /opt/gcc-arm-none-eabi-4_8-2014q3/bin/arm-none-eabi-gcc
    
Unpack the Nordic files to for example the `/opt/softdevices` and `/opt/nrf51_sdk` directories. 

It is a `cmake` build system, so you will need it:

    sudo apt-get install cmake

You can now just type `make` in the main directory. Or you can build using the scripts (see below). Before that you'll have to adjust the default configuration settings (see below as well).

### Bugs

There is a bug in one of the SDK files, namely `nrf_svc.h` (different location depending on the SDK version):

    /opt/nrf51_sdk/v6/nrf51822/Include/s110/nrf_svc.h
    /opt/nrf51_sdk/v4/nrf51822/Include/ble/softdevice/nrf_svc.h

Change the assembly line:

        "bx r14" : : "I" (number) : "r0" \

Into:

        "bx r14" : : "I" ((uint16_t)number) : "r0" \

## Usage

You will have to attach a programmer/debugger somehow. Towards that you only need four pins. On the RFduino this is `GND`, `3V`, `RESET`, and `FACTORY` and they are subsequent pins on that side of the RFduino where there are most pins (the other side has the antenna stealing a bit of space for eventual pins). The pin layout of the JLink connector is written out on the [DoBots blog](http://dobots.nl/2014/03/05/rfduino-without-rfduino-code/).


### Configuration

Fork the code by clicking on:

* Fork [https://github.com/dobots/bluenet/fork](https://github.com/dobots/bluenet/fork).
* `git clone https://github.com/${YOUR_GITHUB_USERNAME}/bluenet`
* let us call this directory $BLUENET

Now you will have to set all fields in the configuration file:

* cp CMakeBuild.config.default CMakeBuild.config
* adjust the `NRF51822_DIR` to wherever you installed the Nordic SDK (it should have `/Include` and `/Source` subdirectories
* adjust the `SOFTDEVICE_DIR` to wherever you unzipped the latest SoftDevice from Nordic
* adjust the `SOFTDEVICE_SERIES` to for example `110` or `130` (SoftDevice name starts with it, without the `s`)
* adjust major accordingly `SOFTDEVICE_MAJOR=7`
* adjust minor accordingly `SOFTDEVICE_MINOR=0`
* adjust the `SOFTDEVICE_DIR_API` to the directory with the SoftDevice include files
* set `SOFTDEVICE_NO_SEPARATE_UICR_SECTION=1` if you use one of the earlier s110 softdevices with a separate UICR section
* adjust the type `SOFTDEVICE` accordingly (basename of file without `_softdevice.hex`)
* set the `APPLICATION_START_ADDRESS` to start of application in FLASH (called `CODE_R1_BASE` in Nordic documentation)
* set the `APPLICATION_LENGTH` to what remains of FLASH 
* set `RAM_R1_BASE` to the start of RAM that is available (SoftDevice S130 v0.5 uses a staggering 10kB from the 16kB!)
* set `RAM_APPLICATION_AMOUNT` to what remains for the application in RAM
* adjust the `COMPILER_PATH` and `COMPILER_TYPE` to your compiler (it will be used as `$COMPILER_PATH\bin\$COMPILER_TYPE-gcc`)
* adjust `JLINK` to the full name of the JLink utility (JLinkExe on Linux)
* adjust `JLINK_GDB_SERVER` to the full name of the JLink utility that supports gdb (JLinkGDBServer on Linux)
* set `BLUETOOTH_NAME` to something you like, but make sure it's short.
* adjust `INDOOR_SERVICE` to `1` if you want to enable it, the same is true for the other services
* adjust `CHAR_MESHING` to `1` if you want to enable meshing functionality, the same is true for other characteristics
* adjust `BOARD` to the correct number for your board. This determines the pin layout.
* adjust `HARDWARE_VERSION` to the correct version of the NRF51 chip you have. Use script/hardware_version.sh to check your version.
* adjust `SERIAL_VERBOSITY` to the value you prefer. Set it to None to disable all logging over serial. The default is 1 (info).
* adjust `MASTER_BUFFER_SIZE` if you want to have a larger buffer (this buffer is reused by all characteristics that use more than a single byte or string)

Let us now install the SoftDevice on the nRF51822:

    cd scripts
    ./softdevice.sh build
    ./softdevice.sh upload

Now we can build our own software:

    cd $BLUENET
    make

Alternatively, you can build it using our script:

    cd scripts
    ./firmware.sh build crownstone

There is also an upload and debug script. The first uses `JLink` to upload, the second uses `gdb` to attach itself
to the running process:

    ./firmware.sh upload crownstone
    ./firmware.sh debug crownstone

You can also run everything in sequence:

    ./firmware.sh all crownstone

And there you go. There are some more utility scripts, such as `reboot.sh`. Use as you wish. 

## Flashing with the ST-Link

The above assumes you have the J-Link programmer from Nordic. If you do not have that device, you can still program something like the RFduino or the Crownstone, by using an ST-Link. A full explanation can be found on <https://dobots.nl/2015/01/23/programming-the-nrf51822-with-the-st-link/>. 

### Combine softdevice and firmware

First of all you should combine all the required binaries into one big binary. This is done by the script combine.sh. Before you use it, you will need to install srec_cat on your system.

    sudo apt-get install srecord

If you call the script it basically just runs srec_cat:

    ./combine.sh

And you will see that it runs something like this:

    srec_cat /opt/softdevices/s110_nrf51822_7.0.0_softdevice.hex -intel crownstone.bin -binary -offset 0x00016000 -o combined.hex -intel

You have to adjust that file on the moment manually to switch between softdevices or to add/remove the bootloader, sorry! Note that the result is a `.hex` file. Such a file does haveinformation across multiple memory sections. If you upload a `.bin` file often configuration bits/bytes will not be set! 

### Upload with OpenOCD

Rather than downloading `openocd` from the Ubuntu repositories, it is recommended to get the newest software from the source:

    cd /opt
    git clone https://github.com/ntfreak/openocd
    sudo aptitude install libtool automake libusb-1.0-0-dev expect
    cd openocd
    ./bootstrap 
    ./configure --enable-stlink
    make
    sudo make install

Also, make sure you can use the USB ST-Link device without sudo rights:

    sudo cp scripts/openocd/49-stlinkv2.rules /etc/udev/rules.d
    sudo restart udev

You can now use the `flash_openocd.sh` script in the `scripts` directory:

    ./flash_openocd.sh init

And in another console:

    ./flash_openocd.sh upload combined.bin

Here the binary `combined.bin` is the softdevice and application combined.

## Meshing

The meshion functionality is the one we are currently integrating on the moment. So, this is a moving target. Set
`CHAR_MESHING` to `0` if you don't need it.

For the meshing functionality we use https://github.com/NordicSemiconductor/nRF51-ble-bcast-mesh written by a 
Trond Einar Snekvik, department of Engineering Cybernetics at Norwegian University of Science and Technology (and 
Nordic Semiconductors). This code makes use of the Timeslot API which 
is not supported yet in the alpha versions of the `S130`. Hence, if you want to use the meshing functionality, you will
have to use the `S110`.

This means that if you want to use a bootloader, you will also need the `S110` version of it, and the same is true
for the upload script:

* https://github.com/dobots/nrf51-dfu-bootloader-for-gcc-compiler/tree/s110
* https://github.com/dobots/nrf51_dfu_linux

## UART

Currently UART for debugging. In case you happen to have the nRFgo Motherboard (nRF6310, strongly recommended) you can 
easily connect the pints at P2.0 and P2.1 to respectively the pins RXD and TXD on P15 on the board. Do not forget to
switch on the RS232 switch. Subsequently you will need some RS232 to USB cable if you haven't an extremely old laptop.
The current set baudrate you can find in `src/serial.cpp` and is `38400` baud. To read from serial, my personal 
favorite application is  `minicom`, but feel free to use any other program.

    (sudo) minicom -c on -s -D /dev/ttyUSB0

The sudo rights are only necessary if your `udev` rights are not properly set. The above flags just set colors to `on`
and define the right usb port to use (if you've multiple).

## Bootloader

To upload a new program when the Crownstone is embedded in a wall socket is cumbersome. For that reason for deployment
we recommend to add a bootloader. The default bootloader from Nordic does not work with the `S130` devices. You will
need our fork:

    git clone https://github.com/dobots/nrf51-dfu-bootloader-for-gcc-compiler
    cd scripts
    ./all.sh

Note, that if you want to use meshing you will need the `S110` version! This can be found in the `s110` tag (see also
above).

You will have to set some fields such that the bootloader is loaded rather than the application directly. If you use the `J-Link` this is the sequence of commands you will need:

    ./softdevice.sh all
    ./writebyte.sh 0x10001014 0x00034000
    ./firmware.sh all bootloader 0x00034000

Here we place the bootloader at position `0x00034000`. If the bootloader becomes larger in size, you will need to go
down and adjust the code in the `dfu_types.h` file in the bootloader code.

And you should be good to upload binaries, for example with the following python script:

    git clone https://github.com/dobots/nrf51_dfu_linux
    python dfu.py -f crownstone.hex -a CD:E3:4A:47:1C:E4

Currently the upload script needs to be changed depending on the SoftDevice used, for the `S130`:

    ctrlpt_handle = 0x10
    ctrlpt_cccd_handle = 0x11
    data_handle = 0x0E

And for the `S110`:

    ctrlpt_handle = 0x0D
    ctrlpt_cccd_handle = 0x0E
    data_handle = 0x0B

Of course, this is too cumbersome. We will soon implement something that figures out the right handles automatically.

### Debugging bootloader

Make sure the bootloader is actually loaded and the proper address for the application is set. If you use the `J-Link` you can use `./general_command.sh` to read individual memory locations:

    mem 0x10001014 4

This should be `0x34000` if you use the bootloader. If it is `0xFFFF` the application will be loaded from the application start address.

If the bootloader does not find a valid app, there might indeed not be an app available, o its configuration field that tells it that the app is correct isn't set properly:

    mem 3FC00 10

This should return `0000 0001 0000 0000 0000 00FE 0000 0000`. If it isn't set correctly, make sure you have uploaded the `bootloader.hex` file (and not only the `bootloader.bin` file).

## iBeacon

There is iBeacon functionality, if you want to try that out. For that you will need to set a define in the main file.

    #define IBEACON

This is not the normal operation mode of the Crownstones, and it is not guaranteed to stay.

## Todo list

* Clean up code
* Obtain RSSI values
* Set up management for establishing connections, getting RSSI values, and tear down connections again
* Create an algorithm to come up with all locations of the nodes. These positions are relative, not absolute. The network will be known up to rotation and scale.
* Implement this algorithm in a distributed fashion. The type of algorithm I have in mind is belief propagation / message passing.

## Memory use

Due to the fact that the SoftDevice S130 uses 10kB out of 16kB, we have to be really careful with the 6kB we have left. In the `util/memory` directory you can see a tool from Eliot Stock to visualize the foot print of functions. We removed for example exception handling.

<p align="center">
<img src="docs/text.png?raw=true" alt="Memory .text section" height="500px"/>
</p>

<p align="center">
<img src="docs/rodata.png?raw=true" alt="Memory .rodata section" height="500px"/>
</p>

<p align="center">
<img src="docs/bss.png?raw=true" alt="Memory .bss section" height="500px"/>
</p>

Tips to reduce memory usage are really welcome!

## Commercial use

This code is used in a commercial product at DoBots, the [Crownstone](http://dobots.nl/products/crownstone). Our intellectual property exists on two levels. First, you can license our technology to create these extremely cheap BLE building blocks yourself. Second, we build services around BLE-enabled devices. This ranges from smartphones to [gadgets such as the "virtual memo"](http://dobots.nl/2014/07/15/ble-dobeacon-a-virtual-memo/). What this means for you as a developer is that we can be transparent about the software on the Crownstone, which is why this repository exists. Feel free to build your own services on top of it, and benefit from our software development as much as you want. 

It would be much appreciated to state "DoBots inside" in which case we will be happy to provide support to your organisation.

If you want to buy our hardware, please navigate to our [website](http://dobots.nl/products/crownstone). We offer a
hardware development kit for professional use.

## See also

The different other software tools and online resources have been mentioned above, but here is a short list:

* [Bluenet (this repos)](https://github.com/dobots/bluenet)
* [Bluenet documentation](http://dobots.github.io/bluenet/)
* [Bluenet documentation branch](https://github.com/dobots/bluenet/tree/gh-pages)
* [Bootloader](https://github.com/dobots/nrf51-dfu-bootloader-for-gcc-compiler/tree/s110)
* [DFU upload tool](https://github.com/dobots/nrf51_dfu_linux)
* [Crownstone website](http://dobots.nl/products/crownstone)
* [Crownstone Android app](https://play.google.com/store/apps/details?id=nl.dobots.CrownStone)
* [Crownstone app source](https://github.com/dobots/crownstone-app)
* [Crownstone USB image](https://github.com/dobots/crownstone-image)
* [Crownstone SDK manual](https://docs.google.com/a/almende.org/document/d/1uJ83c0rLC_swX-cVFDl0UyIFc9_81ccfRLK2vJJF1VE/edit)

If you have any problems that are not solved by documentation, please ask them as a GitHub [issue](https://github.com/dobots/bluenet/issues).
In that case everybody can profit from things that are currently incorrectly formulated, not yet formulated, or need
some other attention from us developers.

## Copyrights

Obviously, the copyrights of the code written by Christopher, belong to him.

The copyrights (2014-2015) for the rest of the code belongs to the team of Distributed Organisms B.V. and are provided under an noncontagious open-source license:

* Authors: Anne van Rossum, Dominik Egger, Bart van Vliet
* Date: 27 Jan. 2014
* License: LGPL v3+, Apache, or MIT, your choice
* Almende B.V., http://www.almende.com and DoBots B.V., http://www.dobots.nl
* Rotterdam, The Netherlands

Note, that we do not use any header files from Nordic. The header files are rewritten from scratch by Christopher 
especially for that purpose. This means you are not finding a Nordic license text in this repository. 
Of course, this means that you will have to get those files from Nordic via different means (we recommend to buy the
development kit). We just didn't want to "contaminate" this repository with files that we don't understand the license
implications of.

The only exception is the code for the meshing functionality that is put online by Nordic itself on 
https://github.com/NordicSemiconductor/nRF51-ble-bcast-mesh. This functionality can be found in `src/protocol` and
`include/protocol/` and falls of course under the Nordic license. You can disable the meshing functionality with
`CHAR_MESHING=0` if you do want to exclude that code from becoming part of the binary. You can still use the services
for individual nodes, but they won't be able to communicate with each other in that case.

