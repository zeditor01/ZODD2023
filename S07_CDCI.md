# Classic CDC for IMS

## SMPE Server XML

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
  user="P355749c"                                                       
  pw="k9896491426p26b"                                                  
  >                                                                     
  <PACKAGE                                                              
      file="2022121500012/PROD/content/GIMPAF.XML"                      
      hash="341FB67364B0DB2C86A3C6E2CDA790936D7F0BB4"                   
      id="ST251941.content"                                             
   >                                                                    
  </PACKAGE>                                                            
</SERVER> 
```

Client XML
```
<CLIENT
    downloadmethod="https"
    downloadkeyring="javatruststore"
       >
</CLIENT>
```

Job Statement
```
//IBMUSERJ JOB  (NPA),'SMPE PSI JOB',CLASS=A,MSGCLASS=H,
//             NOTIFY=&SYSUID,MSGLEVEL=(1,1),REGION=0M    
```


## Download

Process
1. Launch z/OSMF at ```https://192.168.1.191:10443/zosmf/```
2. Open "Software Management"
3. Select "Portable Software Instances"
4. Actions - "Add - From Download Server"
5. Step 1 : Name = CDCI ; Paste Server XML ; System = S0W1 ; UNIX Directory = /u/ibmuser/smpework/CDCI
6. Step 2 : Paste in Client XML and Job Card
7. Step 3 : Action - Submit Job 
8. Step 4 : 

PSI Download Screenshot 1
![psidown01](images/psidown01.JPG)

PSI Download Screenshot 2
![psidown02](images/psidown02.JPG)

PSI Download Screenshot 3
![psidown03](images/psidown03.JPG)

PSI Download Screenshot 4
![psidown04](images/psidown04.JPG)

PSI Download Screenshot 5
![psidown05](images/psidown05.JPG)

PSI Download Screenshot 6
![psidown06](images/psidown06.JPG)



## Workflows

