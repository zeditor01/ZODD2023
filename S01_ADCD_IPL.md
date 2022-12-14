# ADCD z/OS V2.5 Setup.

Contents
1. P52 Paths
2. Unpack ADCD
3. Devmap
4. IPL Script 
5. Test IPLs

## P52 Paths

* Download ADCD to /home/neale/Downloads 
* Z25B Runtime folder is /home/ibmsys1/Z25B001
* Z25A Saved Folder is /home/ibmsys1/ADATA/Z25A001 

## Unpack ADCD

Unpac the core zVolumes
/home/ibmsys1/Z25B001/unzip2.sh

```
gunzip -c /home/neale/Downloads/B5BLZ1.gz > B5BLZ1 -q
gunzip -c /home/neale/Downloads/B5C551.gz > B5C551 -q
gunzip -c /home/neale/Downloads/B5C560.gz > B5C560 -q
gunzip -c /home/neale/Downloads/B5CFG1.gz > B5CFG1 -q
gunzip -c /home/neale/Downloads/B5DBAR.gz > B5DBAR -q
gunzip -c /home/neale/Downloads/B5DBC1.gz > B5DBC1 -q
gunzip -c /home/neale/Downloads/B5DBC2.gz > B5DBC2 -q
gunzip -c /home/neale/Downloads/B5DIS1.gz > B5DIS1 -q
gunzip -c /home/neale/Downloads/B5DIS2.gz > B5DIS2 -q
gunzip -c /home/neale/Downloads/B5DIS3.gz > B5DIS3 -q
gunzip -c /home/neale/Downloads/B5IMF1.gz > B5IMF1 -q
gunzip -c /home/neale/Downloads/B5INM1.gz > B5INM1 -q
gunzip -c /home/neale/Downloads/B5KAN1.gz > B5KAN1 -q
gunzip -c /home/neale/Downloads/B5PAGA.gz > B5PAGA -q
gunzip -c /home/neale/Downloads/B5PAGB.gz > B5PAGB -q
gunzip -c /home/neale/Downloads/B5PAGC.gz > B5PAGC -q
gunzip -c /home/neale/Downloads/B5PRD1.gz > B5PRD1 -q
gunzip -c /home/neale/Downloads/B5PRD2.gz > B5PRD2 -q
gunzip -c /home/neale/Downloads/B5PRD3.gz > B5PRD3 -q
gunzip -c /home/neale/Downloads/B5PRD4.gz > B5PRD4 -q
gunzip -c /home/neale/Downloads/B5PRD5.gz > B5PRD5 -q
gunzip -c /home/neale/Downloads/B5RES2.gz > B5RES2 -q
gunzip -c /home/neale/Downloads/B5SYS1.gz > B5SYS1 -q
gunzip -c /home/neale/Downloads/B5USR1.gz > B5USR1 -q
gunzip -c /home/neale/Downloads/B5USS1.gz > B5USS1 -q
gunzip -c /home/neale/Downloads/B5USS2.gz > B5USS2 -q
gunzip -c /home/neale/Downloads/B5USS3.gz > B5USS3 -q
gunzip -c /home/neale/Downloads/B5W901.gz > B5W901 -q
gunzip -c /home/neale/Downloads/B5W902.gz > B5W902 -q
gunzip -c /home/neale/Downloads/B5ZWE1.gz > B5ZWE1 -q

```

Unzip the resvol as ibmsys1

```
/usr/z1090/bin/Z1091_ADCD_install /home/ibmsys1/Z25B001/B5RES1.ZPD /home/ibmsys1/Z25B001/B5RES1 
```


## Devmap

devmapsydney.txt 

```
# Neale Devmap for ZDT V13.0 created 20210303
# Volume directory: /home/ibmsys1/Z25A001

[system]
memory      26000m          # define storage size for virtual host
processors  8               # number of processors
3270port    3270            # port number for TN3270 connections

[manager]
name aws3274 0002           # define a few 3270 terminals
device 0700 3279 3274 mstcon
device 0701 3279 3274 tso1
device 0702 3279 3274 tso2
device 0703 3279 3274 tso3
device 0704 3279 3274 tso4

# [manager]
# name awsrdr 010C           # define a card reader for job submission
# device 00C 2540 2821 /home/ibmsys1/ZDT1204/../cards/*

[manager]  # tap define network adapter (OSA) for communication with Linux
name awsosa 0024 --path=A1 --pathtype=OSD --tunnel_intf=y   # QDIO mode
device 400 osa osa --unitadd=0
device 401 osa osa --unitadd=1
device 402 osa osa --unitadd=2

[manager]  # OSA define OSA for general network communication
name awsosa 0022 --path=F0 --pathtype=OSD 
device 404 osa osa  
device 405 osa osa  
device 406 osa osa  

[manager]
name awsckd 0001
device 0A80 3390 3390 /home/ibmsys1/Z25B001/B5RES1
device 0A81 3390 3390 /home/ibmsys1/Z25B001/B5BLZ1
device 0A82 3390 3390 /home/ibmsys1/Z25B001/B5SYS1
device 0A83 3390 3390 /home/ibmsys1/Z25B001/B5C560
device 0A84 3390 3390 /home/ibmsys1/Z25B001/B5C551
device 0A85 3390 3390 /home/ibmsys1/Z25B001/B5CFG1

device 0A86 3390 3390 /home/ibmsys1/Z25B001/B5DBAR
device 0A87 3390 3390 /home/ibmsys1/Z25B001/B5DBC1
device 0A88 3390 3390 /home/ibmsys1/Z25B001/B5DBC2

device 0A89 3390 3390 /home/ibmsys1/Z25B001/B5DIS1
device 0A8A 3390 3390 /home/ibmsys1/Z25B001/B5DIS2
device 0A8B 3390 3390 /home/ibmsys1/Z25B001/B5DIS3

device 0A8C 3390 3390 /home/ibmsys1/Z25B001/B5IMF1
device 0A8D 3390 3390 /home/ibmsys1/Z25B001/B5INM1
device 0A8E 3390 3390 /home/ibmsys1/Z25B001/B5KAN1

device 0A90 3390 3390 /home/ibmsys1/Z25B001/B5PAGA
device 0A91 3390 3390 /home/ibmsys1/Z25B001/B5PAGB
device 0A92 3390 3390 /home/ibmsys1/Z25B001/B5PAGC

device 0A93 3390 3390 /home/ibmsys1/Z25B001/B5PRD1
device 0A94 3390 3390 /home/ibmsys1/Z25B001/B5PRD2
device 0A95 3390 3390 /home/ibmsys1/Z25B001/B5PRD3
device 0A96 3390 3390 /home/ibmsys1/Z25B001/B5PRD4 
device 0A97 3390 3390 /home/ibmsys1/Z25B001/B5PRD5

device 0A99 3390 3390 /home/ibmsys1/Z25B001/B5RES2
device 0A9A 3390 3390 /home/ibmsys1/Z25B001/B5USR1
device 0A9B 3390 3390 /home/ibmsys1/Z25B001/B5USS1
device 0A9C 3390 3390 /home/ibmsys1/Z25B001/B5USS2
device 0A9D 3390 3390 /home/ibmsys1/Z25B001/B5USS3
# device 0A9E 3390 3390 /home/ibmsys1/Z25B001/B5W901
# device 0A9F 3390 3390 /home/ibmsys1/Z25B001/B5W902
device 0AA0 3390 3390 /home/ibmsys1/Z25B001/B5ZWE1 

# device 0AA1 3390 3390 /home/ibmsys1/Z25A001/EAV00A
# device 0AA2 3390 3390 /home/ibmsys1/Z25A001/EAV00B
# device 0AA3 3390 3390 /home/ibmsys1/Z25A001/EAV00C
# device 0AA4 3390 3390 /home/ibmsys1/Z25A001/EAV00D
# device 0AA5 3390 3390 /home/ibmsys1/Z25A001/EAV00E
# device 0AA6 3390 3390 /home/ibmsys1/Z25A001/EAV00F
# device 0AA7 3390 3390 /home/ibmsys1/Z25A001/USER0A
# device 0AA8 3390 3390 /home/ibmsys1/Z25A001/USER0B
# device 0AA9 3390 3390 /home/ibmsys1/Z25A001/USER0C
# device 0AAA 3390 3390 /home/ibmsys1/Z25A001/USER0D
# device 0AAB 3390 3390 /home/ibmsys1/Z25B001/B5ZCX1
# device 0AAC 3390 3390 /home/ibmsys1/Z25A001/ZCX00B
# device 0AAD 3390 3390 /home/ibmsys1/Z25A001/ZCX00C
# device 0AAE 3390 3390 /home/ibmsys1/Z25A001/ZCX00D

# device 0AAF 3390 3390 /home/ibmsys1/Z25A001/EAV001
# device 0AB0 3390 3390 /home/ibmsys1/Z25A001/EAV002

# device 0AB1 3390 3390 /home/ibmsys1/Z25A001/DB200A
# device 0AB2 3390 3390 /home/ibmsys1/Z25A001/DB200B

# device 0AB3 3390 3390 /home/ibmsys1/Z25A001/USER0E
# device 0AB4 3390 3390 /home/ibmsys1/Z25A001/USER0F

# device 0600 3390 3390 /home/ibmsys1/Z25A001/WORK01


```


## IPL Scripts

coldipl.sh 

```
awsckmap devmapsydney.txt
awsstart --map devmapsydney.txt
x3270 -port 3270 mstcon@localhost &
x3270 -port 3270 tso@localhost &
ipl 0a80 parm 0A82CS
```

ipl_sydney.sh

```
awsckmap devmapsydney.txt
awsstart --map devmapsydney.txt
x3270 -port 3270 mstcon@localhost &
x3270 -port 3270 tso@localhost &
ipl 0a80 parm 0A82DB
```



## Test IPLs 

cold IPL

IBMUSER / IBMUSER
Change PWD to SYS1

warm IPL.

Base system operational


