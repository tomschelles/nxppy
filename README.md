nxppy
=====
I was having lots of issues installing this library and found a fix so I decided to make it aviable for all of you. 

nxppy is a *very* simple Python wrapper for interfacing with the excellent [NXP EXPLORE-NFC shield](https://www.newark.com/nxp/explore-nfc-ww/nfc-add-on-board-raspberry-pi/dp/45X6356) for the [Raspberry Pi](http://www.raspberrypi.org/).  It takes NXP's NFC Reader Library and provides a thin layer for detecting a Mifare NFC tag, reading its UID (unique identifier), and reading/writing data from/to the user area.

License
=====
All files in this repository are distributed under the MIT license.

####External components
This work was based very heavily on NXP's MifareClassic example code. The example code was only reorganized to be more conducive as an interface. NXP still retains full copyright and ownership of the example code and the NFC Reader Library. The license conditions that need to be accepted in order to use this project in conjunction with the NFC Reader Library can be found in the document [NXP_NFC_Reader_Library_licencefile.pdf](https://github.com/Schoberm/nxppy/blob/master/NXP_NFC_Reader_Library_licencefile.pdf)


Requirements
=====
The EXPLORE-NFC card relies on SPI being enabled. Please enable SPI using raspi-config prior to installing nxppy.

Installation
=====

Copy all files to the Raspberry. 

```
sudo apt-get update
sudo apt-get install build-essential cmake python3-dev unzip wget
```

go to the correct directory and simply run:

```
sudo python setup.py install
```

Installation will take some time source zip is included in install files.

Usage
=====
Currently, the module supports ISO14443-3A/4A cards only:

```python
import nxppy
import time

mifare = nxppy.Mifare()

# Print card UIDs as they are detected
while True:
    try:
        uid = mifare.select()
        print(uid)
    except nxppy.SelectError:
        # SelectError is raised if no card is in the field.
        pass

    time.sleep(1)
```
