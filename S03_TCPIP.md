# TCPIP and Certificates Customisation.

Each time, my customisations get better.
This time

* I want to use the Tunnel for TCPIP because that's what ZVA does
* I want to check out all the certificates ( z/OSMF, ShopZ etc... )

## TCPIP High Level



## ZD&T Application in P52 Linux 

User ibmsys1 invoked find_io command

```
[ibmsys1@localhost Z25B001]$ find_io
   
 FIND_IO for "ibmsys1@localhost.localdomain" 
                                                                                                
         Interface         Current          MAC                IPv4              IPv6           
 Path    Name              State            Address            Address           Address        
------   ----------------  ---------------- -----------------  ----------------  -------------- 
  F0     enp0s31f6         UP, RUNNING      e8:6a:64:5d:2e:8a  192.168.1.188     fe80::8124:21fd:e178:6d63%enp0s31f6  
  F1     wlp0s20f3         DOWN             d6:a8:9b:eb:d2:e6  *                 *               
. 
  *      virbr0            UP, NOT-RUNNING  52:54:00:a4:96:d1  192.168.122.1     *               
  *      virbr0-nic        DOWN             52:54:00:a4:96:d1  *                 *               
. 
  A0     tap0              DOWN             02:a0:a0:a0:a0:a0  *                 *               
  A1     tap1              DOWN             02:a1:a1:a1:a1:a1  *                 *               
  A2     tap2              DOWN             02:a2:a2:a2:a2:a2  *                 *               
  A3     tap3              DOWN             02:a3:a3:a3:a3:a3  *                 *               
  A4     tap4              DOWN             02:a4:a4:a4:a4:a4  *                 *               
  A5     tap5              DOWN             02:a5:a5:a5:a5:a5  *                 *               
  A6     tap6              DOWN             02:a6:a6:a6:a6:a6  *                 *               
  A7     tap7              DOWN             02:a7:a7:a7:a7:a7  *                 *               
   
                                                                                                
         Interface                         Current Settings                                     
 Path    Name              RxChkSum      TSO     GSO     GRO     LRO    RX VLAN       MTU**     
------   ----------------  ---------------- -----------------  ----------------  -------------- 
  F0     enp0s31f6           Off         On*     Off     Off     Off      Off         1500 
  F1     wlp0s20f3           Off         Off     On*     On*     Off      Off         1500 
. 
  *      virbr0              Off         On*     On*     On*     Off      Off         1500 
  *      virbr0-nic          Off         Off     On*     On*     Off      Off         1500 

```

devmap configuration is

```
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
```






