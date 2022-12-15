# ShopZ PSI Process

## Disk Geometry

3390 Disk Geometry [https://ibmmainframes.com/references/disk.html]

3390-009 (8.5GB)
* 10017 CYL (849,960 bytes/CYL)
* 15 TRACKS per CYL (56,664 bytes/Track)

2000 CYL = 1.6GB

## ZFS for smpework

Not a permenant mount.
Mount on demand at /u/ibmuser/smpework 

## Smaller Downloads in SGEXTEAV

IBMUSER.CNTL(CRTZFS01)

```
//IBMUSERJ JOB  (NPA),'INIT 3380 DASD',CLASS=A,MSGCLASS=H,            
//             NOTIFY=&SYSUID,MSGLEVEL=(1,1),REGION=0M                
//********************************************************************
//*                                                                  *
//* PURPOSE: CREATE ZFS DATASET ON SGEXTEAV                          *
//* and MOUNT IT AT /u/ibmuser/smpework                              *
//*                                                                  *
//********************************************************************
//CREATE   EXEC PGM=IDCAMS,REGION=0M                                  
//SYSPRINT DD SYSOUT=*                                                
//SYSIN    DD *                                                       
  DEFINE -                                                            
       CLUSTER -                                                      
         ( -                                                          
             NAME(SMPE.WORK.ZFS) -                                    
             LINEAR -                                                 
             CYL(2000 1000) VOLUME(EAV001) -                          
             DATACLASS(DCEXT) -                                       
             SHAREOPTIONS(3) -                                        
         )                                                            
/*                                                                    
//*                                                                   
// SET ZFSDSN='SMPE.WORK.ZFS'                                         
//FORMAT   EXEC PGM=IOEAGFMT,REGION=0M,COND=(0,LT),                   
// PARM='-aggregate &ZFSDSN -compat'                                  
//SYSPRINT DD SYSOUT=*                                                
//STDOUT   DD SYSOUT=*                                                
//STDERR   DD SYSOUT=*                                                
//SYSUDUMP DD SYSOUT=*                                                
//CEEDUMP  DD SYSOUT=*                                                
//*                                                                   
//*                                                                   
//* MOUNT THE DATASET AT THE MOUNTPOINT DIRECTORY                     
//*                                                                   
//MOUNT    EXEC PGM=IKJEFT01,REGION=0M,DYNAMNBR=99,COND=(0,LT)        
//SYSTSPRT  DD SYSOUT=*                                               
//SYSTSIN   DD *                                                      
  PROFILE MSGID WTPMSG                                                
  MOUNT TYPE(ZFS) +                                                   
    MODE(RDWR) +                                                      
    MOUNTPOINT('/u/ibmuser/smpework') +                               
    FILESYSTEM('SMPE.WORK.ZFS')                                       
/*                                                                    
```

## Larger Downloads over Six 3390s

Create a ZFS spanning a pack of six 3390-009 volumes

```
//DEFINE   EXEC   PGM=IDCAMS                      
//SYSPRINT DD     SYSOUT=H                        
//SYSUDUMP DD     SYSOUT=H                        
//AMSDUMP  DD     SYSOUT=H                        
//SYSIN    DD     *                               
     DELETE 'SQLDI.ZFS'                            
     DEFINE CLUSTER (NAME(SQLDI.ZFS) -             
            VOLUMES(EAV00A EAV00B EAV00C -        
                    EAV00D EAV00E EAV00F) -       
            DATACLASS(DCEXTEAV) -                 
            LINEAR CYL(3336 3336) SHAREOPTIONS(3))
/*                                                
//FORMAT   EXEC   PGM=IOEAGFMT,REGION=0M,         
// PARM=('-aggregate SQLDI.ZFS -compat')           
//SYSPRINT DD     SYSOUT=H                        
//STDOUT   DD     SYSOUT=H                        
//STDERR   DD     SYSOUT=H                        
//SYSUDUMP DD     SYSOUT=H                        
//CEEDUMP  DD     SYSOUT=H                        
//*

```
