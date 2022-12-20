# Classic CDC for IMS Customisation

After the PSI installation, the next job is to create a CDCI instance and configure it and IMS to work together.

1. Create Classic CDC for IMS Instance
2. Configure z/OS Environment
3. Configure IMS Environment
4. Integrate into wider CDC Network


## Task 1: Create Classic CDC for IMS Instance

HLQ is CDCI.CAC

Edit The job to generate the instance libraries. ```CDCI.CAC.SCACSAMP(CECCUSC1)``` so that the HLQ of the instance libraries will be CDCI.CAC.I1 
```
//* ------------------------------------------------------------------   
//*                                                                      
//GENSAMP PROC CAC='CDCI.CAC.SCACSAMP',    INSTALLED HIGH LEVEL QUALIFIER
//             DISKU='SYSALLDA',    DASD UNIT                            
//             DISKVOL='EAV001',  DASD VOLSER                            
//             ISPHLQ='ISP',    ISPF HLQ                                 
//             ISPLANG='ENU'   ISPF LANGUAGE                             
//*                                                                      
//* ------------------------------------------------------------------   

//GENSAMPE EXEC GENSAMP                                           
//CRTSAMP.SYSTSIN  DD *                                           
 PROFILE NOPREFIX  MSGID                                          
                                                                  
 ISPSTART CMD(%CACCUSX1                                         + 
      CACDUNIT=SYSALLDA                                         + 
  /*  CACDVOLM=<CACDVOLM>         UNCOMMENT IF NEEDED  */       + 
  /*  CACMGTCL=<CACMGTCL>         UNCOMMENT IF NEEDED  */       + 
  /*  CACSTGCL=<CACSTGCL>         UNCOMMENT IF NEEDED  */       + 
      CACINHLQ=CDCI.CAC                                         + 
      CACUSHLQ=CDCI.CAC.I1                                      + 
      ISPFHLQ=ISP                                               + 
      ISPFLANG=ENU                                              + 
      SERVERROLE=(CDC_IMS_SRC)                                  + 
 )                                                                
/*                                                                
```

This will generate two PDS datasets

```
CDCI.CAC.I1.USERCONF
CDCI.CAC.I1.USERSAMP
```


### Possible Error (until ShopZ download is repaired).
As of late 2022 the REXX modeules CACCUSX1 and CACCUSX2 are broken.
Temporary solution is to copy the CACCUSX1,2,3,4 source modules 
from https://github.ibm.com/data-replication/Classic/tree/master/samplib and
copy CACCUSX4 into the botton of CACCUSX1 and CACCUSX2 to replace the INCLUDE CACCUSX4 reference.


Specify the Parameters to be used for the Instance

Edit CDCI.CAC.I1.USERSAMP(CECCUSP2) to specify the parameters for this instance

Available in /source/cdci path of this repository

### Run Instance Setup Jobs

Define Logstreams

Optional - delete the line with STG_DATACLAS if you don't have a storage class

CDCI.CAC.I1.USERSAMP(CECCDSLS)

```
//*********************************************
//* ALLOCATE THE LOG STREAM FOR THE EVENT LOG *
//*********************************************
//ALLOCEV    EXEC  PGM=IXCMIAPU                
//SYSPRINT DD SYSOUT=*                         
//SYSOUT   DD SYSOUT=*                         
//SYSIN    DD *                                
   DATA TYPE(LOGR) REPORT(NO)                  
   DEFINE LOGSTREAM NAME(CDCSRC.EVENTS)        
          DASDONLY(YES)                        
          MAXBUFSIZE(4096)                     
          LS_SIZE(1024)                        
          STG_SIZE(512)                        
          RETPD(14) AUTODELETE(YES)            
/*                                             

//********************************************************************
//* ALLOCATE THE LOG STREAM FOR THE DIAGNOSTIC LOG                   *
//********************************************************************
//ALLOCDG    EXEC  PGM=IXCMIAPU                                       
//SYSPRINT DD SYSOUT=*                                                
//SYSOUT   DD SYSOUT=*                                                
//SYSIN    DD *                                                       
   DATA TYPE(LOGR) REPORT(NO)                                         
   DEFINE LOGSTREAM NAME(CDCSRC.DIAGLOG)                              
          DASDONLY(YES)                                               
          LS_SIZE(1024)                                               
          STG_SIZE(512)                                               
          RETPD(7) AUTODELETE(YES)                                    
/*                                                                    
```

Define Classic Catalog ZFS

CDCI.CAC.I1.USERSAMP(CECCRZCT)

```
//CACVSDEF EXEC PGM=IDCAMS,REGION=0M                                   
//SYSPRINT  DD SYSOUT=&SOUT                                            
//SYSIN     DD *                                                       
 PARM GRAPHICS(CHAIN(TN))                                              
                                                                       
 /* ------------------------------------------------------------- */   
 /*                                                               */   
 /*   Ensure that the high level qualifier set which references   */   
 /*   the VSAM LDS to be used for the zFS aggregate is the same   */   
 /*   string specified as the value of the USRHLQ JCL symbolic    */   
 /*   parameter.                                                  */   
 /*                                                               */   
 /* ------------------------------------------------------------- */   
                                                                       
 DELETE (CDCI.CAC.I1.ZFS) -                                            
   PURGE CLUSTER                                                       
                                                                       
 SET MAXCC = 0                                                         
                                                                       
 DEFINE CLUSTER ( -                                                    
   NAME(CDCI.CAC.I1.ZFS) -                                             
   LINEAR -                                                            
   MEGABYTES(150 128) -                                                
   VOLUMES(*) -                                                        
   SHAREOPTIONS(3 3)  -                                                
   CISZ(4096))                                                         
/*                                                                     
//*                                                                    
//* ------------------------------------------------------------------ 
//*                                                                    
//*    . Format the VSAM LDS as a zFS aggregate.                       
//*                                                                    
//*      Note that all parameters for the zFS formatting utility       
//*      (IOEAGFMT) are Case Sensitive                                 
//*                                                                    
//* ------------------------------------------------------------------ 
//*                                                                    
//FORMAT EXEC PGM=IOEAGFMT,REGION=0M,                                  
//    COND=(0,LT),                                                     
//    PARM='-aggregate CDCI.CAC.I1.ZFS  -compat'                       
//SYSPRINT  DD SYSOUT=&SOUT                                            
//STDOUT    DD SYSOUT=&SOUT                                            
//STDERR    DD SYSOUT=&SOUT                                            
//CEEDUMP   DD SYSOUT=&SOUT                                            
//*                                                                    
```

Create USS directory where the Classic Catalog will be mounted

mkdir /opt/IBM/isclassic113/catalog

Temporary mount
mount -f CCDC.I1.ZFS /opt/IBM/isclassic113/catalog

Permanent mount
Edit ADCD.Z24B.PARMLIB(BPXPRMDB)

```
/* ----------------------------------------------------------------- */
/*                                                                   */
/* Classic CDC for IMS                                               */
/*                                                                   */
/* ----------------------------------------------------------------- */
                                                                       
MOUNT    FILESYSTEM('CDCI.CAC.I1.ZFS')                                 
         TYPE(ZFS)                                                     
         MODE(RDWR)                                                    
         MOUNTPOINT('/opt/IBM/isclassic113/catalog')                   
                                                                       
```


Allocate the Configuration files & Import the default source configuration
Edit & Run
CCDC.I1.USERSAMP(CECCDCFG)

```
//*************************************************                   
//* ALLOCATE CONFIGURATION FILES                  *                   
//*************************************************                   
//CFGALLOC     EXEC PGM=IEFBR14                                       
//CACCFGD      DD  DSN=&CPHLQ..CACCFGD,                               
//             UNIT=&DISKU,                                           
//             STORCLAS=&STGCL,                                       
//             MGMTCLAS=&MGTCL,                                       
//             SPACE=(TRK,(20,10)),                                   
//             DCB=(RECFM=FBS,LRECL=64,BLKSIZE=27968),                
//             DISP=(NEW,CATLG,DELETE)                                
//CACCFGX  DD  DSN=&CPHLQ..CACCFGX,                                   
//             UNIT=&DISKU,                                           
//             STORCLAS=&STGCL,                                       
//             MGMTCLAS=&MGTCL,                                       
//             SPACE=(TRK,(10,5)),                                    
//             DCB=(RECFM=FBS,LRECL=64,BLKSIZE=27968),                
//             DISP=(NEW,CATLG,DELETE)                                
//*                                                                   
//SYSOUT   DD  SYSOUT=&SOUT                                           
//SYSPRINT DD  SYSOUT=&SOUT                                           
//********************************************************************
//* IMPORT SOURCE CONFIGURATION                                      *
//********************************************************************
//CFGINIT   EXEC PGM=CACCFGUT,REGION=&RGN                             
//STEPLIB  DD  DISP=SHR,DSN=&CAC..SCACLOAD                            
//MSGCAT   DD  DISP=SHR,DSN=&CAC..SCACMSGS                            
//*                                                                   
//* CONFIGURATION FILES TO POPULATE                                   
//CACCFGD  DD DISP=SHR,DSN=&CPHLQ..CACCFGD                            
//CACCFGX  DD DISP=SHR,DSN=&CPHLQ..CACCFGX                            
//*                                                                   
//* CONFIG DEFINITION TO IMPORT                                       
//IMPORTCF DD DSN=&USRHLQ..USERCONF,DISP=SHR                          
//SYSTERM  DD SYSOUT=&SOUT                                            
//SYSPRINT DD SYSOUT=&SOUT                                            
//SYSOUT   DD SYSOUT=&SOUT                                            
//SYSIN    DD DUMMY                                                   
//********************************************************************
//* ALLOCATE SYSMDUMP FILE                                           *
//********************************************************************
//DELETDMP  EXEC PGM=IEFBR14                                          
//DDUMP  DD  DSN=&CPHLQ..SYSMDUMP,                                    
//             DISP=(MOD,DELETE,DELETE),                              
//             UNIT=SYSALLDA,                                         
//             VOL=(,,,3),                                            
//             SPACE=(TRK,(2500,450)),                                
//             DCB=(LRECL=4160,RECFM=FBS,                             
//             DSORG=PS)                                              
//ALLOCDMP  EXEC PGM=IEFBR14                                          
//MDUMP     DD DSN=&CPHLQ..SYSMDUMP,                                  
//             DISP=(NEW,CATLG,DELETE),                               
//             UNIT=SYSALLDA,                                         
//             VOL=(,,,3),                                            
//             SPACE=(TRK,(2500,450)),                                
//             DCB=(LRECL=4160,RECFM=FBS,                             
//             DSORG=PS)                                              
//     PEND                                                           
//ALLOCATE EXEC ALLOCFG                                               
//CFGINIT.SYSIN DD *                                                  
IMPORT,CONFIG,FILENAME=DDN:IMPORTCF(CECCDXFG)                         
REPORT                                                                
QUIT                                                                  
/*                                                                    
```



## Task 2: Configure z/OS Environment

add CCDC.SCACLOAD to the APF Authorized list of libraries. Edit ```ADCD.Z25B.PARMLIB(PROGAD)```

```
/**********************************************/                      
/*  CDCI                                      */                      
/**********************************************/                      
APF ADD                                                               
    DSNAME(CDCI.CAC.SCACLOAD)                           VOLUME(EAV004)
```

TCPIP Ports

Check Port 9087 and 5003 are available and open for connection to Classic CDC Started Task from client tools.


## Task 3: Configure IMS Environment


## Task 4: Integrate into wider CDC Network



