# ShopZ PSI Process

## Disk Geometry

3390 Disk Geometry[https://ibmmainframes.com/references/disk.html]

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

