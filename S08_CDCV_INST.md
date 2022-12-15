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


## Workflows


