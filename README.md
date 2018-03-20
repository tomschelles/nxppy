nxppy
=====
nxppy is a *very* simple Python wrapper for interfacing with the excellent [NXP EXPLORE-NFC shield](https://www.newark.com/nxp/explore-nfc-ww/nfc-add-on-board-raspberry-pi/dp/45X6356) for the [Raspberry Pi](http://www.raspberrypi.org/).  It takes NXP's NFC Reader Library and provides a thin layer for detecting a Mifare NFC tag, reading its UID (unique identifier), and reading/writing data from/to the user area.

License
=====
All files in this repository are distributed under the MIT license.

####External components
This work was based very heavily on NXP's MifareClassic example code. The example code was only reorganized to be more conducive as an interface. NXP still retains full copyright and ownership of the example code and the NFC Reader Library. The license conditions that need to be accepted in order to use this project in conjunction with the NFC Reader Library can be found in the document [NXP_NFC_Reader_Library_licencefile.pdf](https://github.com/Schoberm/nxppy/blob/master/NXP_NFC_Reader_Library_licencefile.pdf)

Compatibility
=====
Tested with both Python 2.7 and 3.4 as installed on Raspian Jessie.

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
sudo setup.py install
```

Installation will take some time source zip is included in install files.

Usage
=====
Currently, the module supports ISO14443-3A/4A cards only:

```python
import nxppy

mifare = nxppy.Mifare()

# Select the first available tag and return the UID
uid = mifare.select()

# Read a single block of 4 bytes from block 10 
block10_bytes = mifare.read_block(10)

# Write a single block of 4 bytes
mifare.write_block(10, 'abcd')

# Read the entire user data area (this relies on byte 2 of the capability container being set to the correct tag size)
user_data = mifare.read()
```

Example polling for tags:

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

NDEF records can be read as bytes, and then passed through [ndeflib](https://github.com/nfcpy/ndeflib) for parsing

```python
import nxppy
import ndef

# Instantiate reader
mifare = nxppy.Mifare()

# Select tag
uid = mifare.select()

# Read NDEF data
ndef_data = mifare.read_ndef()

# Parse NDEF data
ndef_records = list(ndef.message_decoder(ndef_data))
```

**Authentication example:**<br />
This example uses the address layout of a NTAG216 card. Please refer to the specific card manual for the address layout.<br />
NTAG cards remain authenticated until removed from field or an error occurs. Reauthenticate to gain access again.

NTAG216 configuration page layout:

| Address  | Byte 0 | Byte 1 |    Byte 2    | Byte 3 |
| -------- | ------- | ----- | ------------ | ------ |
|   0xE3   | Mirror  | RFUI  | Mirror_Page  | AUTH0  |
|   0xE4   | Access  | RFUI  |    RFUI      | RFUI   |
|   0xE5   |  PWD    |  PWD  |     PWD      |  PWD   |

Relevant parts:<br />
**0xE3 Byte3** Defines the address from which the password verification is required<br />
**0xE4 Byte0 Bit7** Defines access protection: 0 for write protection (read only), 1 for read and write protection<br />
**0xE5** All 4 bytes are used to store the password

Values not mentioned here are the cards default values and not altered by this script.


```python
import nxppy

#initilize and select tag
mifare = nxppy.Mifare()
uid = mifare.select()


#Enable protection
####################

#write password 1234
mifare.write_block(0xE5, '1234')

#password protection starting at address D1
mifare.write_block(0xE3, b'\x04\x00\x00\xD1')

#enable readprotection (default is write protection only)
mifare.write_block(0xE4, b'\x80\x05\x00\x00')


#authenticate
####################
mifare.pwd_auth('1234')
#read and write from/to protected address space


#disable protection
####################

#reset password
mifare.write_block(0xE5, b'\xFF\xFF\xFF\xFF')

#set protection addr beyond address space
mifare.write_block(0xE3, b'\x04\x00\x00\xE7')

#reset readprotection 
mifare.write_block(0xE4, b'\x00\x05\x00\x00')

```

