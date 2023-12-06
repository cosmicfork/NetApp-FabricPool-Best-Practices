# Appendix B - Troubleshooting 
This section covers commonly asked-about features of NetApp FabricPool and details various policies and procedures. Further, in Section 9: Finding More FabricPool Documentation, you will find resources that will lead you to further FabricPool documentation beyond these documents, as well as contact information for NetApp’s software team to aid with troubleshooting. 

## 1. Understanding FabricPool’s Security Protocols 
FabricPool follows AES-256-GCM encryption on the local tier, on the cloud tier, and over the wire while moving data between tiers. 

### FabricPool’s Local Tier Security Capabilities 
On the local tier, FabricPool supports NetApp Storage Encryption (NSE), NetApp Volume Encryption (NVE), and NetApp Aggregate Encryption (NAE). 

### FabricPool’s Over-the-Wire Security Capabilities
Data moving between local and cloud tiers is encrypted by using TLS 1.2 using AES-256-GCM. Other encryption modes like CCM are not supported. Although communicating without TLS encryption is supported and could combat latency, it is not recommended.

### FabricPool’s Cloud Tier Security Capabilities 
All objects encrypted by NVE/NAE will remain encrypted when moved to the cloud tier. Client-side encryption keys are owned by ONTAP. Any objects not encrypted using NVE/NAE are automatically encrypted server-side using AES-256-GCM encryption. These encryption keys are owned by the respective object store, and no additional encryption is necessary. 

**Note:** FabricPool requires the use of the AES-256-GCM authenticated encryption. Other encryption modes, such as CCM, are not supported. 

## 2. Disabling FabricPool’s Cloud Tier Encryption
Beginning with ONTAP 9.7, encrypting cold data at rest is no longer required. Using FabricPool without encrypting data at rest is not recommended but could be required by low performance S3 compatible object store providers that can’t provide server-side encryption and low latency simultaneously. 

### To disable cloud tier encryption in these cases, run the following command: 

```
storage aggregate object-store config modify -serverside-encyption false
```

**Note:** You will need an advanced privilege level to complete this command. 

## 3. Using Other S3 Compatible Providers 
Any customers who want to use object stores that are not officially supported as a cloud tier in FabricPool can do so using the command below: 

```
-provider-type S3_Compatible
```

Customers must test and confirm that the object store meets their requirements, and Netapp does not support, nor is liable for any issues resulting from any third-party Object Store Service, specifically where it does not have an agreed support arrangement with the third party with whom the product originated. It is acknowledged and agreed that NetApp shall not be liable for any associated damage or otherwise be required to provide support on that 3rd party product. 

## 4. Managing CA certification validation processes 
CA certificate validation can be turned off when adding a StorageGRID cloud tier using ONTAP System Manager. 

### Follow the steps below: 
1. Launch ONTAP System Manager
2. Click Storage  
3. Click Tiers
4. Click Add Cloud Tier 
5. Select an object store provider 
6. Complete the text fields as required for your object store provider 
Uncheck the Object Store Certificate button to turn it off
7. Click Save 

### You can also turn off CA certificate validation when adding a private cloud tier by using the ONTAP CLI. Run the following commands: 

```
object-store config create
-object-store-name <name>
-provider-type <IBM_COS/ONTAP_S3/S3_Compatible/SGWS>
-port <443/8082> (other providers/SGWS)
-server <name>
-container-name <bucket-name>
-access-key <string>
-secret-password <string>
-ssl-enabled true
-ipspace default
-is-certificate-validation-enabled false
-use-http-proxy false
-url-stle <path-style/virtual-hosted-stle>
```

## 5. Using Thin Provisioning 
FabricPool cannot attach a cloud tier to a local tier that contains volumes using a space guarantee other than none (for example, volume). For additional information on this policy, see [FabricPool’s Requirements](https://www.netapp.com/pdf.html?item=/media/17239-tr-4598.pdf). 

## 6. Using FlexGroup Volumes on FabricPool Local Tiers 
When provisioning FlexGroup volumes on FabricPool local tiers (storage aggregates), automatic processes in ONTAP System Manager require that the FlexGroup Volume uses FabricPool local tiers on every cluster node. This is a recommended practice, but not a requirement when manually provisioning FlexGroup volumes. 

Provisioning FlexGroup constituent volumes on heterogeneous local tiers (some that use FabricPool, some not using FabricPool) is not recommended and will result in unpredictable tiering and performance. 

## 7. Learning FabricPool’s Volume Tiering Policies 
By default, volumes use the None volume tiering policy. The exception is newly created FlexVol volumes on FabricPool aggregates which use the **Snapshot-Only** volume tiering policy. 

Once you create volumes, the volume tiering policy can be changed using ONTAP System Manager or the ONTAP CLI. 

### An Overview of FabricPool’s four provided volume tiering policies 
**Auto:** 
- All cold blocks in the volume are moved to the cloud tier; assuming the local tier is greater than 50% utilized, it will take approximately 31 days for inactive blocks to become cold. The auto-cooling period is then adjustable between 2 days and 183 days by using the tiering-minimum-cooling-days command. 
- When cold blocks are read randomly, they are made hot and written to the local tier.
- When cold blocks are read sequentially, they stay cold and remain on the cloud tier. They are not written to the local tier. 
- Object storage is not transactional like file or block storage. Making changes to files being stored as objects in volumes with overly aggressive minimum cooling days can result in the creation of new objects, fragmentation of existing objects, decreased read performance, and the addition of storage inefficiencies. 

**Snapshot-only:** 
- Cold blocks that are not shared with the active file system are moved to the cloud tier. Assuming the local tier is greater than 50% utilized, it takes about two days for inactive blocks to become cold. The cooling period is adjustable from 2 to 183 days using the tiering-minimum-cooling-days command. 
- When read, cold blocks associated with Snapshot-only copies stay cold and are not written back to the local tier. 

**All:**
- All data blocks placed in the volume are immediately marked as cold and moved to the cloud tier as soon as possible. There is no need to wait 48 hours for new blocks to cool. 
- When cold blocks are read, they remain cold and stay on the cloud tier. They are not written to the local tier. 
- Object storage is not transactional like file or block storage. Making changes to files being stored as objects in volumes using the All tiering policy can result in the creation of new objects, fragmentation of existing objects, decreased read performance, and the addition of storage inefficiencies. 
- In releases earlier than ONTAP 9.6, the Backup volume tiering policy functions the same as the All policy with the exception that the Backup policy can only be set on data protection volumes (destination targets). 

**None (default):** 
- Volumes do not tier cold data to the cloud tier.
- Setting the tiering policy to none prevents new tiering. Any Volume data that has previously been moved to the cloud tier will remain in the cloud tier until it becomes hot and is automatically moved back to the local tier. 
- When cold blocks are read, they are made hot and written to the local tier. 

### How to Change a Volume’s Tiering Policy
To change a volume’s tiering policy, you can use either the ONTAP System Manager, or the ONTAP CLI: 

**A. ONTAP System Manager**
1. Launch ONTAP System manager 
2. Click Storage 
3. Click Volumes
4. Select a volume 
5. Click Edit 
6. Select the tiering policy you want to apply to the volume: 
7. Click Save (Note: Beginning in ONTAP 9.8, changing the tiering policy to All, Auto, or Snapshot-Only triggers the background tiering scan to run immediately). 

**B.  ONTAP CLI** 

To change a volume’s tiering policy in ONTAP CLI, run the following command:
```
volume modify -vserver <svm_name> -volume <volume_name> -tiering-policy <auto|snapshotonly|all|none> 
```
## 8. Understanding FabricPool’s Minimum Cooling Days Volume Tiering Policies
FabricPool is not an ILM policy that permanently archives data after a set period of time. Since instead, FabricPool is a high-performance tiering solution that makes data immediately accessible, it can dynamically move data to and from the cloud tier based on client activity. 

The ```tiering-minimum-cooling-days``` setting determines how many days must pass before inactive data in a volume using the Auto or Snapshot-Only policy is considered cold and eligible for tiering. 

**Note:** Increasing ```tiering-minimum-cooling-days``` increases the footprint of inactive data on the local tier (data takes longer before it’s marked inactive and ready for tiering to the cloud tier). Also, if data is read from the cloud tier, made hot, and written back to the local tier, it takes longer to become inactive again and is tiered back to the cloud tier. 

### An Overview of the Different Cooling Settings Based on Volume Tiering Policy 
**Auto**

The default ```tiering-minimum-cooling-days``` setting for the Auto tiering policy is 31 days. Because reads keep block temperatures hot, this value might reduce the amount of data that is eligible to be tiered and increase the amount of data kept on the local tier. If you would like to reduce this value from the default 31 days, be aware that the data should be inactive before being marked as cold. ```Tiering-minimum-cooling-days``` should be set an extra day longer to accommodate a preceding day of heavy reads. 

**Snapshot-Only**
The default ```tiering-minimum-cooling-days``` setting for the Snapshot-Only tiering policy is two days. This minimum allows additional time for background processes to provide maximum storage efficiency and prevents daily data-protection processes from needing to read data in the cloud tier. 

### How to Change a Volume’s Tiering Minimum Cooling Days Setting 
To change a volume’s ```tiering-minimum-cooling-days``` setting using ONTAP CLI, run the following command: 

```
volume modify -vserver <svm_name> -volume <volume_name> -tiering-minimum-cooling-days <2-183>
```

**Note:** Changing the tiering policy between Auto and Snapshot-Only (or the other way around) resets the tiering-minimum-cooling-days parameter to its default setting for the target policy. 

## 9. Using MetroCluster with FabricPool 

### How MetroCluster Works with FabricPool Mirror 
Starting with ONTAP 9.7, MetroCluster’s continuous availability feature extends to FabricPool through FabricPool Mirror. NetApp’s MetroCluster enables continuous data availability and mobility even across geographically separate data centers. MetroCluster’s continuous availability and disaster recovery software runs through the ONTAP data management software. 

While using FabricPool Mirror, data is mirrored across two buckets so Cluster A’s local tiers can be connected to cloud tier A1 (the primary cloud tier) and cloud tier A2 (the mirror cloud tier). Meanwhile, Cluster B’s local tiers can be connected to cloud tier B1 (the primary B cloud tier) and cloud tier B2 (the mirror cloud tier).


**Note:** To successfully create a FabricPool local tier using MetroCluster, you must ensure that primary and mirror buckets are accessible from both clusters. 

MetroCluster expects mirroring across all local tiers, both traditional aggregates, and those using FabricPool. Any unmirrored aggregates in MetroCluster environments do not need to use FabricPool Mirror, but this will prompt error messages warning that the aggregate is not mirrored and that they are missing a FabricPool Mirror. 

## 10. Finding more FabricPool documentation
### a. Below is a list of further documentation and resources for NetApp FabricPool: 
- [TR-4598: FabricPool Best Practices in ONTAP 9.13.1](https://www.netapp.com/pdf.html?item=/media/17239-tr-4598.pdf) (primary source for this documentation)
- [FabricPool Overview](https://docs.netapp.com/us-en/ontap/concepts/fabricpool-concept.html#:~:text=FabricPool%20is%20an%20ONTAP%20feature,object%20storage%20in%20the%20cloud.)
- [FabricPool Tier Management Overview](https://docs.netapp.com/us-en/ontap/fabricpool/tiering-policies-concept.html#:~:text=FabricPool%20tiering%20policies%20determine%20when,decreases%20when%20it%20is%20not.)

### b. You can call support to ask more questions about FabricPool’s functionality at the number listed below, and visit the support website below for more troubleshooting resources: 
- Support Center Number: 888-463-8277
- Support Website: https://mysupport.netapp.com/site/
