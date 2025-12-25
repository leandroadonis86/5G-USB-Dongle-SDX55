# INFO

Firmware steps and upgrade file `FOTA_326-72-6_to_326-73-0_0R19.zip` or `FOTA_326-71_to_326-73_0R19.zip` can be found in the link https://www.tricascadeinc.com/vos-5g-qa .


# EXTRACT FOTA FIRMWARE

The module TRITOM SG500M2-X M.2 use the memory IC MT "JZ080". The IC JZ080 is a marking code (FBGA code) for the Micron MT29GZ5A5BPGGA-53IT.87J memory component, which is an e.MMC (managed NAND memory) device with integrated LPDDR4 memory.

Technical Specifications:

```
Manufacturer: Micron
Component Type: e.MMC Memory (Managed NAND Flash) with LPDDR4 DRAM
NAND Density: 4Gb (512M x 8)
DRAM Density: 4Gb (128M x 32)
Supply Voltage: 1.8V
Clock Frequency: 1866 MHz
Interface: Parallel, with x8 (NAND) and x32 (DRAM) bus width
Package: 149-ball WFBGA (8x9.5)
Operating Temperature: -40°C to +85°C
```

To extract the firmware for ex. `FOTA_326-72-6_to_326-73-0_0R19.zip`:

1) Unzip the file `FOTA_326-72-6_to_326-73-0_0R19.zip`.
2) Inside the "firmware-update" folder the `sdxprairie-oemfs.ubi` is the file with the content that will change the system. In the system it's called `/oem/` folder.
3) Two ways possible to be extracted: `binwalk` or `ubireader_extract_files`.

Binwalk method execute:

`$ binwalk -v --signature -e sdxprairie-oemfs.ubi --directory=oemfs_extracted`

Ubireader_extract_files method execute:

`$ ubireader_extract_files sdxprairie-oemfs.ubi`


# MODIFYING FOTA FIRMWARE

For modifying FOTA firmware the process requires a NAND simulation into UBIFS type partition.

1) Simulate a NAND MT-device with nandsim
   
```
$ sudo modprobe nandsim \
    first_id_byte=0x2c \
    second_id_byte=0xac \
    third_id_byte=0x80 \
    fourth_id_byte=0x26 \
    page_size=4096 \
    oob_size=256 \
    erasesize=262144
```

2) Find out the MT-device id (make sure it's "mtd0: 20000000 00040000")

```
$ cat /proc/mtd | grep -i "NAND Simulator"
mtd0: 20000000 00040000 "NAND simulator partition 0"
```

3) Load UBI kernel module

`$ sudo modprobe ubi`

4) Erase MT-device (should be 256Kibyte)

```
$ sudo flash_erase /dev/mtd0 0 0
Erasing 256 Kibyte @ 1ffc0000 -- 100 % complete 
```

5) Check MT-device configuration

```
$ mtdinfo /dev/mtd0
mtd0
Name:                           NAND simulator partition 0
Type:                           nand
Eraseblock size:                262144 bytes, 256.0 KiB
Amount of eraseblocks:          2048 (536870912 bytes, 512.0 MiB)
Minimum input/output unit size: 4096 bytes
Sub-page size:                  1024 bytes
OOB size:                       128 bytes
Character device major/minor:   90:0
Bad blocks are allowed:         true
Device is writable:             true
```

6) Flash the UBI firmware image with ubiformat

```
$ sudo ubiformat /dev/mtd0 -f sdxprairie-oemfs.ubi -s 4096 -y
ubiformat: mtd0 (nand), size 536870912 bytes (512.0 MiB), 2048 eraseblocks of 262144 bytes (256.0 KiB), min. I/O size 4096 bytes
libscan: scanning eraseblock 2047 -- 100 % complete  
ubiformat: 2048 eraseblocks are supposedly empty
ubiformat: flashing eraseblock 31 -- 100 % complete  
ubiformat: formatting eraseblock 2047 -- 100 % complete  
```

7) Attach MT-device to UBI with ubiattach and note the UBI device number

```
$ sudo ubiattach -p /dev/mtd0 -O 4096
UBI device number 0, total 2048 LEBs (520093696 bytes, 496.0 MiB), available 0 LEBs (0 bytes), LEB size 253952 bytes (248.0 KiB)
```

8) Check if Ubi is correct to mount

```
$ ubinfo /dev/ubi0
ubi0
Volumes count:                           1
Logical eraseblock size:                 253952 bytes, 248.0 KiB
Total amount of logical eraseblocks:     2048 (520093696 bytes, 496.0 MiB)
Amount of available logical eraseblocks: 0 (0 bytes)
Maximum count of volumes                 128
Count of bad physical eraseblocks:       0
Count of reserved physical eraseblocks:  40
Current maximum erase counter value:     1
Minimum input/output unit size:          4096 bytes
Character device major/minor:            236:0
Present volumes:                         0
```

9) Create a folder "ubifs-root" to mount the image

`$ sudo mkdir -p /mnt/ubifs-root`

10) Check if module "ubi sile system type" is active in the system to be able to mount

```
$ cat /proc/filesystems
...
...
...
nodev	ubifs
```

11) Mount it

`sudo mount --types ubifs ubi0 /mnt/ubifs-root/`

12) If it's correct and no errors found, check ubi info for further configurations

```
$ ubinfo -a
UBI version:                    1
Count of UBI devices:           1
UBI control device major/minor: 10:56
Present UBI devices:            ubi0
```

```
ubi0
Volumes count:                           1
Logical eraseblock size:                 253952 bytes, 248.0 KiB
Total amount of logical eraseblocks:     2048 (520093696 bytes, 496.0 MiB)
Amount of available logical eraseblocks: 0 (0 bytes)
Maximum count of volumes                 128
Count of bad physical eraseblocks:       0
Count of reserved physical eraseblocks:  40
Current maximum erase counter value:     1
Minimum input/output unit size:          4096 bytes
Character device major/minor:            236:0
Present volumes:                         0
```

```
Volume ID:   0 (on ubi0)
Type:        dynamic
Alignment:   1
Size:        2002 LEBs (508411904 bytes, 484.8 MiB)
State:       OK
Name:        oemfs
Character device major/minor: 236:1
```


# Q&A

> What system was used to extract the FOTA firmware?

Ubuntu Release 20.04.6 LTS (Focal Fossa) 64-bit.

> What pakages need to install?

binwalk, mtd-utils.

> What means this hex byte codes on nandsim? "first_id_byte=0x2c..."?

First_id_byte=0x2c 'manufactor code', second_id_byte=0xac 'device id: 4Gbit, x8, 3.3V/1.8V', third_id_byte=0x80 'size of page', fourth_id_byte=0x26 'size of block'. This information can be calculated and found in Micron MT29GZ5A5BPGGA-53IT.87J datasheet. The bytes must be compactible with your 'JZ080' NAND so it can be able to attach the image file without errors.

> ...

...
