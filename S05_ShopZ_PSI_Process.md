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

Create 6 Model 9s
```
alcckd /home/ibmsys1/Z25B001/EAV001 -d3390-27
alcckd /home/ibmsys1/Z25B001/EAV002 -d3390-27
alcckd /home/ibmsys1/Z25B001/EAV003 -d3390-27
alcckd /home/ibmsys1/Z25B001/EAV004 -d3390-27
alcckd /home/ibmsys1/Z25B001/EAV005 -d3390-27
alcckd /home/ibmsys1/Z25B001/EAV006 -d3390-27
```

Add to devmap ; initvol ; convert to sms (?)

Now run this job

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
             NAME(BIG6.ZFS) -                                         
             VOLUMES(EAV00A EAV00B EAV00C -                           
                     EAV00D EAV00E EAV00F) -                          
             DATACLASS(DCEXTEAV) -                                    
             LINEAR CYL(3336 3336) -                                  
             SHAREOPTIONS(3) -                                        
         )                                                            
/*                                                                    
//*                                                                   
// SET ZFSDSN='BIG6.ZFS'                                              
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
    FILESYSTEM('BIG6.ZFS')                                            
/*                                                                    
```
