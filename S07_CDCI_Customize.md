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



## Task 2: Configure z/OS Environment

## Task 3: Configure IMS Environment


## Task 4: Integrate into wider CDC Network



