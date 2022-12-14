# ShopZ maintenance procedure


# Process

Maintenance Process is
1. Check if PTF installed in Target Zone
2. Run a Report
3. ShopZ Order PTF ( and upload the report )
4. Edit JCL to perform secure download to ZFS
5. Run an SMPE Apply
6. Read the Hold Data ( eg: restart z/OSMF )


## 1. Check if PTF installed in Target Zone
Identify the Global Zone for zOSMF = MVS.GLOBAL.CSI 

Run an SMPE Query

![csiquery01](images/csiquery01.JPG)

Check Results : PTF UI79663 not installed in the target zone

![csiquery02](images/csiquery02.JPG)




