
Stratasys
---------

This is a software to read and write data on a Stratasys cartridge memory.

You may want to use it so you can refill or build your own cartridge.

## Installation

### Dependencies

- python 2.7
- [pycrypto](https://www.dlitz.net/software/pycrypto/)

## Usage

### Print information about a cartridge

You have to provide the machine type (fox, prodigy or quantum) and the EEPROM uid,
in hexadecimal form without the '0x' prefix. Note that the EEPROM uid use to end
with "23".

    $ ./stratasys-cli.py eeprom -t fox -e 4343434343434343 -i random_file.bin

The EEPROM uid use to end with '23'. You may have to reverse it. Say you have
"233a38b1020000c0", you should reverse it to be "c0000002b1383a23".

If you provide the '-r' option, arguments to pass to stratasys-cli will be printed
to help you recreate the cartridge.

The input file must be a binary file.

### Create your own cartridge

By providing all required information, the software will provide a new valid eeprom
that you can write on a cartridge.

    $ ./stratasys-cli.py eeprom --machine-type fox --eeprom-uid 4343434343434343 --serial-number 1234.0 --material-name ABS --manufacturing-lot 1234 --manufacturing-date "2001-01-01 01:01:01" --use-date "2002-02-02 02:02:02" --initial-material 11.1 --current-material 22.2 --key-fragment 4141414141414141 --version 1 --signature STRATASYS -o random_file.bin

All the dates are in international format: yyyy-mm-dd hh:mm:ss

You have to provide the correct machine-type and the valid eeprom uid. You can
customize all the rest.

The EEPROM uid use to end with '23'. You may have to reverse it. Say you have
"233a38b1020000c0", you should reverse it to be "c0000002b1383a23".

The generated file will be 113 bytes in size. You can complete the file with zeroes
if you want to make it 512 bytes long, the usual EEPROM size.

### List supported material

If you want a list of all known material, simply:
    $ ./stratasys-cli.py material --list
    0       ABS
    1       ABS_RED
    2       ABS_GRN
    [...]

Use those names when creating a new cartridge.

## Interesting fork / rewrite

* [slaytonrd/CartridgeWriter](https://github.com/slaytonrnd/CartridgeWriter) - rewritten in C# by slaytonrd

## Interfacing with the cartridge

### Bus-pirate

- Use the MISO wire (orange) for the data
- Use the GROUND wire (black) on the ground
- Connect the 5V (grey) on the pull-up voltage input (blue)

Use the following schematic as a reference:

    Bus pirate

    grey    >---+
                |
    blue    >---+
                 eeprom
                +------+
    orange  >---| Data |
                |      |
    black   >---| Gnd  |
                +------+

Use the scripts available in the `helper` directory.

To read an eeprom:

    $ ./bp_read.py /dev/ttyUSB0 eeprom.bin

To write an eeprom:

    $ ./bp_write.py /dev/ttyUSB0 eeprom_new.bin

### Raspberry Pi

- Use the GPIO 4 (pin 7) for the data
- Use any GROUND (pin 6,9,14,20 or 25) on the ground
- Use the 5V Power (pin 2) to pull-up the data line using a ~4.7k resistor

Use the following schematic as a reference:

    Raspberry pi

     5V     >---+
                |
           4.7k Z    eeprom
                |   +------+
    GPIO4   >---+---| Data |
                    |      |
    GROUND  >-------| Gnd  |
                    +------+

Then you'll need to probe 2 kernel modules:

    $ sudo modprobe w1-wire w1-ds2433

You should now see your eeprom appearing:

    $ ls -l /sys/bus/w1/devices/w1_bus_master1
    23-xxxxxxxxxxxx/
    (...)

To print the eeprom uid:

    $ xxd -p /sys/bus/w1/devices/w1_bus_master1/23-xxxxxxxxxxxx/id
    23xxxxxxxxxxxx

To read an eeprom:

    $ cp /sys/bus/w1/devices/w1_bus_master1/23-xxxxxxxxxxxx/eeprom ~/eeprom.bin

To write an eeprom:

    $ cp ~/eeprom_new.bin /sys/bus/w1/devices/w1_bus_master1/23-xxxxxxxxxxxx/eeprom

## Setup code

Not supported yet.

## Acknowledgement

Special thanks to the Stratahackers group. Without them, nothing like this could
be possible. They provided moral and technical support!
