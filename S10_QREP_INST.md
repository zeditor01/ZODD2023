# QREP for Db2 z/OS

## Server XML

20221216
```
=== Order Size and File System Size Information ========================
                                                                        
The size of your order is 545 MB                                        
                                                                        
You need space in the file system used by z/OSMF Software Management    
Add Portable Software Instance for approximately twice the size of your 
order. To convert to 3390 cylinders, multiply the number of MB by 1.25  
and then multiply by 2.                                                 
                                                                        
For example, for a size of 5000 MB, then:                               
( (5,000 MB) * (1.25 CYL/MB) ) * 2 = 12,500 cylinders                   
                                                                        
== Server XML for Add Portable Software Instance From Download Server ==
You can copy the below statements into the z/OSMF Software Management   
Server XML box.                                                         
                                                                        
<SERVER                                                                 
  host="deliverycb-mul.dhe.ibm.com"                                     
  user="P8r98221"                                                       
  pw="c176C974767829r"                                                  
  >                                                                     
  <PACKAGE                                                              
      file="2022121500010/PROD/content/GIMPAF.XML"                      
      hash="DC03015920F951919A8F7741D33A5AA2E3948232"                   
      id="ST251943.content"                                             
   >                                                                    
  </PACKAGE>                                                            
</SERVER>    
```



## Download

Process
1. Launch z/OSMF at ```https://192.168.1.191:10443/zosmf/```
2. Open "Software Management"
3. Select "Portable Software Instances"
4. Actions - "Add - From Download Server"
5. Step 1 : Name = QREP ; Paste Server XML ; System = S0W1 ; UNIX Directory = /u/ibmuser/smpework/QREP
6. Step 2 : Paste in Client XML and Job Card
7. Step 3 : Action - Submit Job 
8. Step 4 : Complete the Download

See screenshots of the process in the S07_CDCV_INST notes.

## Workflows

## Deployment Workflows

Deployment Process
1. Launch z/OSMF at ```https://192.168.1.191:10443/zosmf/```
2. Open "Software Management"
3. Open "Deployments"
4. Select "New" and "Portable Software Instance"
5. Specify Name of Deployment (QREP)
6. Select "QREP" or whicher PSI you want to deploye
7. Objective = New CSI on target System S0W1
8. Optional - Check for missing SYSMODS
9. Configure Deployment (Accept Source Model, HLQs, Storage Classes, ZFS Mount Points )
10. Specifically - mount point for supplied MQ - allow deafult because we will use MQ9 from ADCD
11. Define Job Settings (location of PDS dataset for Jobs)
12. Submit Deployment Jobs ( Unzip ; Rename ; Update CSI )
13. Perform Workflows (Your Order & PostDeploy )

The Your Order workflow is nothing. Just override complete.

The PostDeploy Workflow contains important steps. JFDI

See screenshots of the process in the S07_CDCV_INST notes.



Mount Point
/usr/lpp/db2repl_11_04
QREP.OMVS.SASNROOT


## ERRORS

Note that JOB00703 (IBMUSERJ) failed OC4.
Execute ASNLKMSG EXEC to create symbolic links to the 18 message catalog files

```
//IBMUSERJ JOB  (NPA),'SMPE PSI JOB',CLASS=A,MSGCLASS=H,                JOB00703                           
//             NOTIFY=&SYSUID,MSGLEVEL=(1,1),REGION=0M                                                     
//*                                                                                                        
//*                                                                                                        
IEFC653I SUBSTITUTION JCL - (NPA),'SMPE PSI JOB',CLASS=A,MSGCLASS=H,NOTIFY=IBMUSER,MSGLEVEL=(1,1),REGION=0M
/*JOBPARM SYSAFF=S0W1                                                                                      
//********************************************************************                                     
//* JOB: HAAWB33N  (product-supplied job ASNISLKM in library SASNBASE)                                     
//*                                                                                                        
//* GDE: Product ServerPac INSTALLING YOUR ORDER                                                           
//*                                                                                                        
//* DOC: This job executes the ASNLKMSG EXEC to create                                                     
//*      symbolic links from the NLSPATH to the eighteen                                                   
//*      message catalog files for the Q and SQL Replication                                               
//*      feature of IBM InfoSphere Data Replication for DB2 for z/OS                                       
//*      V11.4.0.                                                                                          
//*                                                                                                        
//* NOTES:                                                                                                 
//*                                                                                                        
//*  1. Check and change, if required, the install directory.                                              
//*                                                                                                        
//*  2. Change the -NlsPath- string to a path name that                                                    
//*     includes all components of the NLSPATH directory                                                   
//*     name up to the %L component.  If the NLSPATH is                                                    
//*     /usr/lib/nls/msg/%L/%N, the path name is                                                           
//*     /usr/lib/nls/msg/.  Note that the replacement                                                      
//*     string is case sensitive.  Ensure the -NlsPath- is                                                 
//*     an absolute path name which begins and ends with a                                                 
//*     slash (/).  If you do not specify -NlsPath-,                                                       
//*     the job will create symbolic links from the default                                                
//*     NLSPATH, /usr/lib/nls/msg/, to the message catalog                                                 
//*     files.                                                                                             
//*                                                                                                        
//*  3. Ensure you execute this job from a userid that is UID=0 or                                         
//*     is the owner of the NLSPATH directory -NlsPath- specified.                                         
//*                                                                                                        
//*  4. This job should end with RC=0.  If not, then check the                                             
//*     output, consult the UNIX System Services Messages and                                              
//*     Codes book to correct the problem, and resubmit this job.                                          
//*                                                                                                        
//* MRC: The maximum expected return code is 0.                                                            
//*                                                                                                        
//*******************************************************************                                      
//REXX     EXEC  PGM=IKJEFT1A                                                                              
//SYSEXEC  DD DSN=QREP.ASN.SASNBASE,                                                                       
//            DISP=SHR                                                                                     
//SYSTSPRT  DD SYSOUT=*                                                                                    
//SYSTSIN  DD   *                                                                                          
//                                                                                                         
```
