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
mount -f CDCI.CAC.I1.ZFS /opt/IBM/isclassic113/catalog

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

Create the Classic Catalog 
Edit & Run
CDCI.CAC.I1.USERSAMP(CECCDCAT)

```
//* ------------------------------------------------------------------
//* Reset the USS file system resident catalog files to an            
//* empty state.                                                      
//* ------------------------------------------------------------------
//*                                                                   
//RESETCTD  EXEC  PGM=IEBGENER,REGION=0M                              
//SYSPRINT  DD SYSOUT=&SOUT                                           
//SYSIN     DD DUMMY                                                  
//SYSUT1    DD DUMMY,                                                 
//             DCB=(RECFM=FBS,LRECL=1,BLKSIZE=5120)                   
//SYSUT2    DD PATHDISP=(KEEP,KEEP),                                  
//             PATHMODE=(SIRWXU,SIRWXG),                              
//             PATHOPTS=(ORDWR,OCREAT,OTRUNC),                        
//             PATH='/opt/IBM/isclassic113/catalog/caccat'            
//*                                                                   
//RESETCTX  EXEC  PGM=IEBGENER,REGION=0M                              
//SYSPRINT  DD SYSOUT=&SOUT                                           
//SYSIN     DD DUMMY                                                  
//SYSUT1    DD DUMMY,                                                 
//             DCB=(RECFM=FBS,LRECL=1,BLKSIZE=5120)                   
//SYSUT2    DD PATHDISP=(KEEP,KEEP),                                  
//             PATHMODE=(SIRWXU,SIRWXG),                              
//             PATHOPTS=(ORDWR,OCREAT,OTRUNC),                        
//             PATH='/opt/IBM/isclassic113/catalog/cacindx'           
//*                                                                   
//* ----------------------------------------------------------------- 
//*                                                                   
//* Initialize the catalog files                                      
//*                                                                   
//* ----------------------------------------------------------------- 
//*                                                                   
//INIT     EXEC PGM=CACCATUT,PARM='INIT',REGION=&RGN                  
//STEPLIB  DD  DISP=SHR,DSN=&CAC..SCACLOAD                            
//SYSOUT   DD  SYSOUT=&SOUT                                           
//SYSPRINT DD  SYSOUT=&SOUT                                           
//MSGCAT   DD  DISP=SHR,DSN=&CAC..SCACMSGS                            
//CACCAT   DD  PATHDISP=(KEEP,KEEP),                                  
//             PATH='/opt/IBM/isclassic113/catalog/caccat'            
//CACINDX  DD  PATHDISP=(KEEP,KEEP),                                  
//             PATH='/opt/IBM/isclassic113/catalog/cacindx'           
```

Create Replication Mapping Datasets

 EDIT & Run
CDCI.CAC.I1.USERSAMP(CECCDSUB)

```
// EXPORT SYMLIST=(CPHLQ)             
// SET CPHLQ='CDCI.CAC.I1.CDCSRC'     
//*                                   
//ALLOC001 EXEC PGM=IDCAMS            
//SYSPRINT DD   SYSOUT=*              
//SYSIN    DD   *,SYMBOLS=JCLONLY     
 DELETE &CPHLQ..SUB PURGE             
 IF LASTCC=8 THEN SET MAXCC=0         
 /**/                                 
 DELETE &CPHLQ..RM PURGE              
 IF LASTCC=8 THEN SET MAXCC=0         
 /**/                                 
 DEFINE CLUSTER                  -    
    (NAME(&CPHLQ..SUB)        -       
     RECSZ(1024 2048)            -    
     KEY(80 12)                  -    
     CYL(2 1)                    -    
     SPEED REUSE)                -    
  DATA                           -    
    (NAME(&CPHLQ..SUB.DATA)   -       
     CISZ(8192)                  -    
     FSPC(20 5))                 -    
  INDEX                          -    
    (NAME(&CPHLQ..SUB.INDEX)  -       
     CISZ(6144))                      
 /**/                                 
 DEFINE CLUSTER                  -    
    (NAME(&CPHLQ..RM)        -        
     RECSZ(512 2432)             -    
     KEY(20 12)                  -    
     CYL(15 5)                   -    
     SPEED REUSE)                -    
   DATA                          -    
     (NAME(&CPHLQ..RM.DATA)   -       
      CISZ(8192)                 -    
      FSPC(20 5))                -    
   INDEX                         -    
     (NAME(&CPHLQ..RM.INDEX)  -       
      CISZ(2048))                     
/*                                    
```


Create Replication Bookmark Queue (Optional)

Edit & Run
CDCI.CAC.I1.USERSAMP(CECPBKLQ)

```
//DEFINE EXEC PGM=CSQUTIL,PARM='CSQ9'                          
//STEPLIB   DD DISP=SHR,DSN=CSQ911.SCSQANLE                    
//          DD DISP=SHR,DSN=CSQ911.SCSQAUTH                    
//SYSPRINT DD  SYSOUT=*                                        
//SYSIN    DD  *                                               
  COMMAND DDNAME(DEFQ)                                         
/*                                                             
//DEFQ DD *                                                    
  DEFINE QL(CDCI.BOOKMARK) +                                   
    DEFPSIST(YES) GET(ENABLED) INDXTYPE(MSGID) PUT(ENABLED) +  
    DEFSOPT(SHARED) SHARE                                      
/*                                                             
//                                                             
```

Fails until you define the QM stuff to RACF
```
 SDSF OUTPUT DISPLAY CECPBKLQ JOB01248  DSID   103 LINE 0       COLUMNS 02- 161 
 COMMAND INPUT ===>                                            SCROLL ===> PAGE 
********************************* TOP OF DATA **********************************
 CSQU000I CSQUTIL IBM MQ for z/OS V9.2.5                                        
 CSQU001I CSQUTIL Queue Manager Utility - 2022-12-20 14:51:31                   
  COMMAND DDNAME(DEFQ)                                                          
 CSQU127I  Executing COMMAND using input from DEFQ data set                     
 CSQU120I  Connecting to CSQ9                                                   
 CSQU121I  Connected to queue manager CSQ9                                      
 CSQU055I  Target queue manager is CSQ9                                         
  DEFINE QL(CDCI.BOOKMARK) +                                                    
    DEFPSIST(YES) GET(ENABLED) INDXTYPE(MSGID) PUT(ENABLED) +                   
    DEFSOPT(SHARED) SHARE                                                       
CSQN205I   COUNT=       3, RETURN=00000020, REASON=FFFFFFFF                     
CSQ9016E %CSQ9 ' DEFINE' command request not authorized                         
CSQ9023E %CSQ9 CSQ9SCND 'DEFINE QL' ABNORMAL COMPLETION                         
 CSQU057I  1 commands read                                                      
 CSQU058I  1 commands issued and responses received, 1 failed                   
 CSQU143I  1 COMMAND statements attempted                                       
 CSQU144I  1  statements executed successfully                                  
 CSQU148I CSQUTIL Utility completed, return code=0                              
******************************** BOTTOM OF DATA ********************************
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



