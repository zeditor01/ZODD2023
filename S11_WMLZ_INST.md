# Watson Machine Learning for z/OS

## Server XML

20221216
```
=== Order Size and File System Size Information ========================
                                                                        
The size of your order is 20721 MB                                      
                                                                        
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
  user="P918004i"                                                       
  pw="C224390892q735p"                                                  
  >                                                                     
  <PACKAGE                                                              
      file="2022121500015/PROD/content/GIMPAF.XML"                      
      hash="F63AD654BF2F3BAB5603460969FAFF37560B56FE"                   
      id="ST251944.content"                                             
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
5. Step 1 : Name = WMLZ ; Paste Server XML ; System = S0W1 ; UNIX Directory = /u/ibmuser/smpework/WMLZ
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
5. Specify Name of Deployment (WMLZ)
6. Select "WMLZ" or whicher PSI you want to deploye
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

Mountpoints for WLMZ

```
/usr/lpp/IBM/aln/v2r4	WMLZ.OMVS.SALNROOT	

/usr/lpp/IBM/izoda/anaconda	WMLZ.OMVS.SANBZFS	

/usr/lpp/IBM/izoda/spark	WMLZ.OMVS.SAZKROOT	
```

Permenant Mounts specified in ```ADCD.Z25B.PARMLIB(BPXPRMDB)```

```
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

Contents of ZFS filesystems

```
IBMUSER:/Z25B/usr/lpp/IBM/aln/v2r4: >ls
IBM            bin            extra          iml-onnx       iml-utilities  node_modules   usr
README         cics-scoring   iml-db2ads     iml-portal     iml-zostools   nodejs         wlp
alnsamp        configuration  iml-library    iml-services   imlpython      sparkaas

IBMUSER:/Z25B/usr/lpp/IBM/izoda/anaconda: >ls
2017_ga_pkgs.txt       2019_q3_pkgs36.txt     2020_q4_pkgs_p2.txt    2021_q2_pkgsn.txt      README_PYTHON37.md     install-march-ptf
2018_q1_pkgs.txt       2019_q3_r_links        2021_alivy_sec.txt     2021_q4_links          README_R.md            install_functions
2018_q3_links          2019_q4_links          2021_alivy_sec2.txt    2021_q4_pkgsn.txt      apar_notes.txt         install_functions_r
2018_q3_pkgs.txt       2019_q4_pkgs.txt       2021_alivy_sec2_links  2022_alivy_sec1.txt    bin                    lib
2018_sec_links         2019_q4_pkgsn.txt      2021_alivy_sec3.txt    2022_alivy_sec1_links  boot_pkgs.txt          man
2018_sec_pkgs.txt      2020_q1_pkgs_py.txt    2021_alivy_sec3_links  2022_alivy_sec2.txt    change-prefix          march_ptf_pkgs.txt
2018_sec_pkgsn.txt     2020_q1_pkgs_r.txt     2021_alivy_sec_links   2022_alivy_sec2_links  conda-meta             pkgs
2019_q1_links          2020_q1_py_links       2021_amaven_sec.txt    2022_alivy_sec3.txt    configure-anaconda     py37_pkgs.txt
2019_q1_pe0_links      2020_q1_r_links        2021_amaven_sec_links  2022_alivy_sec3_links  configure-anaconda-r   r_pkgs.txt
2019_q1_pkgs.txt       2020_q2_links          2021_q1_links          2022_alivy_sec4.txt    dsdbc                  reinstall
2019_q1_pkgsn.txt      2020_q2_pkgs.txt       2021_q1_pkgs.txt       2022_alivy_sec4_links  envs                   share
2019_q2_links          2020_q2_pkgsn.txt      2021_q2_p1_links       CHANGES.md             etc                    ssl
2019_q2_pkgsn.txt      2020_q4_p1_links       2021_q2_p2_links       IBM                    ga_pkgs.txt            var
2019_q3_links          2020_q4_p2_links       2021_q2_pkgs_p1.txt    README.md              include
2019_q3_pkgs.txt       2020_q4_pkgs_p1.txt    2021_q2_pkgs_p2.txt    README_CONFIGURE.md    initial-install

IBMUSER:/Z25B/usr/lpp/IBM/izoda/spark: >ls
IBM       spark24x
```
