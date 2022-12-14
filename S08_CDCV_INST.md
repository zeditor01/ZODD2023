# Classic CDC for VSAM

## Server XML

20221216
```
=== Order Size and File System Size Information ========================
                                                                        
The size of your order is 549 MB                                        
                                                                        
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
  user="P4461x72"                                                       
  pw="a159793212p487C"                                                  
  >                                                                     
  <PACKAGE                                                              
      file="2022121500014/PROD/content/GIMPAF.XML"                      
      hash="FEB15638CDC2233CCC850C15C60F7D230C99910C"                   
      id="ST251940.content"                                             
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
5. Step 1 : Name = CDCV ; Paste Server XML ; System = S0W1 ; UNIX Directory = /u/ibmuser/smpework/CDCV
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
5. Specify Name of Deployment (CDCV)
6. Select "CDCV" or whicher PSI you want to deploye
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
