# Basic Customisations to make the system healthy.

Each time, my customisations get better.
This time

* I want to get the ACS routines well managed
* I want to place all products etc... on EXTAEV volumes. One big pool.
* I want to plan my DASD diet for ZVA 

## PARMLIB and PROCLIB Concatenations

SYS1.IPLPARM(LOADxx)

All say
```
IODF     99 SYS1 
INITSQA  0000M 0008M 
SYSCAT   B5SYS1113CCATALOG.Z25B.MASTER 
SYSPARM  AL 
IEASYM   00 
NUCLST   00 
PARMLIB  USER.Z25B.PARMLIB                            B5CFG1
PARMLIB  FEU.Z25B.PARMLIB                             B5CFG1
PARMLIB  ADCD.Z25B.PARMLIB                            B5SYS1
PARMLIB  SYS1.PARMLIB                                 B5RES1
NUCLEUS  1 
SYSPLEX  ADCDPL 
```

ADCD.Z25B.PARMLIB(MSTJCL00)

Says

```
//MSTJCL00 JOB MSGLEVEL=(1,1),TIME=1440 
//         EXEC PGM=IEEMB860,DPRTY=(15,15) 
//STCINRDR DD SYSOUT=(A,INTRDR) 
//TSOINRDR DD SYSOUT=(A,INTRDR) 
//IEFPDSI  DD DSN=USER.&SYSVER..PROCLIB,DISP=SHR
//         DD DSN=FEU.&SYSVER..PROCLIB,DISP=SHR 
//         DD DSN=ADCD.&SYSVER..PROCLIB,DISP=SHR
//         DD DSN=SYS1.PROCLIB,DISP=SHR 
//SYSUADS  DD DSN=SYS1.UADS,DISP=SHR 
//SYSLBC   DD DSN=SYS1.BRODCAST,DISP=SHR 

```

## CLOCK.

Checout PARMLIB members
FEU.Z25C.PARMLIB
USER.Z25C.PARMLIB
ADCD.Z25C,PARMLIB
Members IEASYSAL, IEASYSDB, IEASYS00 exist in FEU ... All Say CLOCK=00 

Edit ADCD.Z25C.PARMLIB(CLOCK00) 
(Didn't exist in FEU or USER

```
OPERATOR NOPROMPT 
TIMEZONE E.10.00.00
ETRMODE  NO 
ETRZONE  NO 
ETRDELTA 10 
STPMODE  NO 
STPZONE  NO 
```


## Startup and Shutdown Automation

Edit ```FEU.Z25B.PARMLIB(VTAMALL)```

```
COMMANDPREFIX=NONE   /* THIS IS THE DEFAULT VALUE */ 
/*--------------------------------------------------------------------*
/*--------------------------------------------------------------------*
PAUSE 10      /* WAIT FOR SYS TO GET UP FIRST  */ 
S RRS,SUB=MSTR,GNAME=&SYSNAME 
S TSO         /* TIME SHARING OPTION */ 
S SDSF 
S EZAZSSI,P=S0W1 
PAUSE 10      /* WAIT FOR SYS TO GET UP FIRST  */ 
S TCPIP 
PAUSE 5 
S TN3270 
PAUSE 5 
/*S INETD */ 
-DBCG START DB2 
PAUSE 10 
S HTTPD1 
S NFSS 
S CSF 
S HZSPROC 
PAUSE 5 
S SSHD 
PAUSE 5 
S CICSTS56 
PAUSE 10 
S IMS15RL1 
PAUSE 3 
S IMS15CR1 
PAUSE 10 
%CSQ9 START QMGR 
PAUSE 30 
S IMSWARM 
%CSQ9 START CHINIT 
PAUSE 10 
S ASCH,SUB=MSTR 
PAUSE 5 
S JMON 
PAUSE 3 
S RSED 
PAUSE 3 
S CFZCIM 
PAUSE 10      /* WAIT FOR SYS TO GET UP FIRST  */ 
S BLZBFA 
PAUSE 10 
S BLZISPFD 
PAUSE 10 
S BUZAGNT 
/* S IZUANG1 */ 
/* S IZUSVR1 */ 
S ZOSCSRV 
D T                          /* DISPLAY ENDING TIME 
* COMMENT LINES MAY ALSO START WITH A SINGLE ASTERISK. 
* REMEMBER - BLANK LINES ARE A BIG NO-NO. 
```

Add
```
S FTPD
S INETD
```

Come back later for zOSMF

Edit ```FEU.Z25B.PARMLIB(SHUTALL)```

Add
```
P FTPD
P INETD
```


## ISMF Planning

Change ISMF Profile to provide Administrator View. ( m.2.0.0 )

Storage Groups

```
CXROOTSG POOL
SGAPPL   POOL
SGARCH   POOL
SGBASE   POOL
SGCICS   POOL
SGDB2    POOL
SGEXTEAV POOL
SGIMS    POOL
SGMQS    POOL
SGVIO    VIO 
```

CXROOTSG - will be used for ZCX ( which needs huge datasets - 20GB ) ```B5ZCX1```

SGBASE - will be used for small shit ```B5USR1 USER01 USER02 ...```

SGDB2 - for Db2 ```B5DBC2 DB2001 DB2002 ...```

SGEXTEAV will be used for Installing Products and ZFS datasets ```EAV001 EAV002 ... EAV00F```


## Esoteric Device

Create a Mod 27

```
alcckd /home/ibmsys1/Z25B001/WORK01 -d3390-27
```


Edit ADCD.Z24C.PARMLIB(VATLSTDB)
Edit ADCD.Z24C.PARMLIB(VATLST00)


WORK*,0,0,3390   ,Y


## EXTEAV devices

Create Six Mod 27s

```
alcckd /home/ibmsys1/Z25B001/EAV001 -d3390-27
alcckd /home/ibmsys1/Z25B001/EAV002 -d3390-27
alcckd /home/ibmsys1/Z25B001/EAV003 -d3390-27
alcckd /home/ibmsys1/Z25B001/EAV004 -d3390-27
alcckd /home/ibmsys1/Z25B001/EAV005 -d3390-27
alcckd /home/ibmsys1/Z25B001/EAV006 -d3390-27
```

## ZCX Device

Create a Mod 54

```
alcckd /home/ibmsys1/Z25B001/B5ZCX1 -d3390-54
```

## Update Devmap 

```
device 0AA1 3390 3390 /home/ibmsys1/Z25B001/EAV001
device 0AA2 3390 3390 /home/ibmsys1/Z25B001/EAV002
device 0AA3 3390 3390 /home/ibmsys1/Z25B001/EAV003
device 0AA4 3390 3390 /home/ibmsys1/Z25B001/EAV004
device 0AA5 3390 3390 /home/ibmsys1/Z25B001/EAV005
device 0AA6 3390 3390 /home/ibmsys1/Z25B001/EAV006

device 0AAA 3390 3390 /home/ibmsys1/Z25B001/B5ZCX1

device 0600 3390 3390 /home/ibmsys1/Z25B001/WORK01
```


## Verify devmap

chmod 777 *

awsckmap  devmapsydney.txt 


## Initialise Volumes and Convert to SMS

IPL with volumes in devmap.

Initialise voumes, one by one
```
//ICKDSF01 JOB (FB3),'INIT 3380 DASD',CLASS=A,MSGCLASS=H,
// NOTIFY=&SYSUID,MSGLEVEL=(1,1)
//STEP01 EXEC PGM=ICKDSF,REGION=0M
//SYSPRINT DD SYSOUT=*
//SYSIN DD *
INIT -
MAP /* MAP DEFECTS */ -
UNIT(****) /* ADDRESS */ -
NOCHECK -
CONTINUE -
NOVERIFY /* DO NOT CHECK VOLID */ -
VTOC(9,0,200) /* 200 TRK VTOC */ -
INDEX(0,1,44) /* 45 TRK INDEX */ -
OWNERID('IBMUSER') -
VOLID(******) /* NEW VOLID */
/*
```

Reply to Message ```R NN,U```

Vary unit online with ```VARY AB2,ONLINE```

If necessary, add volume to SMS storagegroup, activate new SMS dataset

Convert to SMS

```
//ICKDSF01 JOB (FB3),'SMS CONVERT',CLASS=A,MSGCLASS=H,
// NOTIFY=&SYSUID,MSGLEVEL=(1,1)
//STEP01 EXEC PGM=ADRDSSU,REGION=0M
//SYSPRINT DD SYSOUT=*
//INVOL1 DD VOL=SER=USER0A,UNIT=3390,DISP=SHR
//SYSIN DD *
CONVERTV -
DDNAME(INVOL1) -
SMS
/*
```

## ACS Routines

### ACS Step 1 - Usercats

Think ahead of Aliases the I want to use for product installs

* DVM (Data Virtualisation Manager)
* CDCI (Classic CDC for IMS)
* CDCV (Classic CDC for VSAM)
* CDCD (CDC for Db2 z/OS)
* QREP (Q Replication)
* ZCX (ZCX)
* ZCXPRV1, ZCXSTC1, ZCXADM1
* WMLZ

DEFINE ALIAS (NAME('DVM') RELATE('USERCAT.Z25B.USER'))

DEFINE ALIAS (NAME('CDCI') RELATE('USERCAT.Z25B.USER'))
DEFINE ALIAS (NAME('CDCV') RELATE('USERCAT.Z25B.USER'))
DEFINE ALIAS (NAME('CDCD') RELATE('USERCAT.Z25B.USER'))

DEFINE ALIAS (NAME('QREP') RELATE('USERCAT.Z25B.USER'))

DEFINE ALIAS (NAME('ZCX') RELATE('USERCAT.Z25B.USER'))
DEFINE ALIAS (NAME('ZCXPRV1') RELATE('USERCAT.Z25B.USER'))
DEFINE ALIAS (NAME('ZCXSTC1') RELATE('USERCAT.Z25B.USER'))
DEFINE ALIAS (NAME('ZCXADM1') RELATE('USERCAT.Z25B.USER')) 

DEFINE ALIAS (NAME('WMLZ') RELATE('USERCAT.Z25B.USER'))


### Edit ACS Routines

Use ISMF to Check Active ACS ```SYS1.S0W1.DFSMS.CNTL```

Edit DATACLAS. Move If &SIZE > 4052MB rule AFTER the ZCX rule. 

```
IF &DSN(1) = 'ZCX' THEN          
  DO                             
    IF &DSN(2) = 'VS' THEN       
      DO                         
            SET &DATACLAS='CXDC' 
          END                    
        END                      
IF &DSN(2) = 'ZCX' THEN          
          DO                     
        IF &DSN(3) = 'FS' THEN   
          DO                     
            SET &DATACLAS='CXDC' 
          END                    
      END   
```

Edit STORCLAS.

```
	  FILTLIST DVM_HLQ          INCLUDE(DVM.**) 
    
    FILTLIST CDC_HLQ          INCLUDE(CDCI.**,    
                                      CDCV.**,
                                      CDCD.**,
                               
	  FILTLIST QREP_HLQ          INCLUDE(QREP.**)       
 
 	  FILTLIST WMLZ_HLQ          INCLUDE(WMLZ.**)     
                                      
    FILTLIST ZCX_HLQ          INCLUDE(ZCX.**,    
                                  ZCXPRV1.**,
                                  ZCXSTC1.**,
                                  ZCXADM1.**) 
								  
								  

WHEN (&HLQ = &ZCX_HLQ)           
  DO                             
    SET &STORCLAS = 'CXROOTSC'   
    EXIT CODE(0)                 
  END   
  
  WHEN (&HLQ = &DVM_HLQ)           
  DO                             
    SET &STORCLAS = 'SCEXTEAV'   
    EXIT CODE(0)                 
  END   
  
  WHEN (&HLQ = &CDC_HLQ)           
  DO                             
    SET &STORCLAS = 'SCEXTEAV'    
    EXIT CODE(0)                 
  END   
  
  WHEN (&HLQ = &QREP_HLQ)           
  DO                             
    SET &STORCLAS = 'SCEXTEAV'    
    EXIT CODE(0)                 
  END   
  
  WHEN (&HLQ = &WMLZ_HLQ)           
  DO                             
    SET &STORCLAS = 'SCEXTEAV'   
    EXIT CODE(0)                 
  END   
```

Edit STORGRP

```
WRITE '&DSN = ' &DSN
WRITE '&STORCLAS = ' &STORCLAS
WRITE '&DATACLAS = ' &DATACLAS
WRITE '&DSTYPE = ' &DSTYPE

  WHEN (&STORCLAS= 'CXROOTSC') 
  DO                         
    SET &STORGRP = 'CXROOTSG'
    EXIT CODE(0)             
  END   
```

Activate SMS 

* ISMF Translate Each of the 3 members .... CDS = 'SYS1.S0W1.DFSMS.SCDS'  ; Source = SYS1.S0W1.DFSMS.CNTL  ; Member = 1,2,3
* ISMF Validate ...
* ISMF Activate ...'SYS1.S0W1.DFSMS.SCDS'

OK ... Datasets for all proposed installations should go to the correct volumes 



