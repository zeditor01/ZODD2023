# Header


Mount Points

```
/usr/lpp/IBM/db2zaiv1r5	DB2ZAI.OMVS.SCOYROOT
```


ADCD.Z25B.PARMLIB(BPXPRMDB)

```
/* ----------------------------------------------------------------- */ 
/*                                                                   */ 
/* DB2ZAI                                                            */ 
/*                                                                   */ 
/* ----------------------------------------------------------------- */ 
                                                                        
MOUNT    FILESYSTEM('DB2ZAI.OMVS.SCOYROOT')                             
         TYPE(ZFS)                                                      
         MODE(RDWR)                                                     
         MOUNTPOINT('/usr/lpp/IBM/db2zaiv1r5')                          
                                                                        
/* ----------------------------------------------------------------- */ 
/*                                                                   */ 
/* WMLZ V2.4                                                         */ 
/*                                                                   */ 
/* ----------------------------------------------------------------- */ 
                                                                        
MOUNT    FILESYSTEM('WMLZ.OMVS.SALNROOT')                               
         TYPE(ZFS)                                                      
         MODE(RDWR)                                                     
         MOUNTPOINT('/usr/lpp/IBM/aln/v2r4')                            
                                                                        
MOUNT    FILESYSTEM('WMLZ.OMVS.SANBZFS')                                
         TYPE(ZFS)                                                      
         MODE(RDWR)                                                     
         MOUNTPOINT('/usr/lpp/IBM/izoda/anaconda')                      
                                                                        
MOUNT    FILESYSTEM('WMLZ.OMVS.SAZKROOT')                               
         TYPE(ZFS)                                                      
         MODE(RDWR)                                                     
         MOUNTPOINT('/usr/lpp/IBM/izoda/spark')                         
                                                                        
```
