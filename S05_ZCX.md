# z/OS Container Extensions

Priority to make this work because of...
1. ZD&T License Prevention Threat
2. Nested SIE (which stops ZCX on ZVDT)
3. Possibly old ZD&T Driver with wrong emulated hardware feature codes


## Container Hosting Foundations
Appear to be installed, based on ADCD Z25B Docco


ADCD_Z25B_Docco [http://dtsc.dfw.ibm.com/MVSDS/%27HTTPD2.ADCD.GLOBAL.SHTML(A25BREAD)%27)]

And the License is enabled in ```ADCD.Z25B.PARMLIB(IFAPRD00)```

```
PRODUCT OWNER('IBM CORP')                
        NAME('Z/OS CHF')                 
        ID(5655-HZ1)                     
        VERSION(*)                       
        RELEASE(*)                       
        MOD(*)                           
        FEATURENAME('CONTAINERHOSTFND')  
        STATE(ENABLED)                   
```

So we should be able to deploy a ZCX Instance and find out if it works. If it fails we may need to upgrade to zdt 14.0


        
