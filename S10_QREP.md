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

## Workflows

