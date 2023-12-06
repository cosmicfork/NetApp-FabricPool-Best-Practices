# Configuring FabricPool
## Introducing the Steps & Processes Necessary to Configure FabricPool
Configuring FabricPool refers to the steps to set up and connect a cloud tier to a local tier in ONTAP. There are four main steps to configure FabricPool: 
1. Creating a bucket/container on the object store 
2. Adding a cloud tier using the bucket to ONTAP 
3. Attaching the cloud tier to a local tier 
4. Setting up volume tiering policies 

## Understanding FabricPool’s Architecture 
To understand how FabricPool functions, you must learn a bit about FabricPool’s architecture. FabricPool operates by associating a cloud tier (an external bucket/object store) with a local tier (storage aggregate) in ONTAP, which creates a mixed collection of discs, a “FabricPool”. Any volumes inside the FabricPool can then use the tiering to keep active (hot) data on the high-performance storage (the local tier) while tiering inactive (cold) data to the external object store (the cloud tier). To learn more about FabricPool’s architecture, navigate to Appendix A of this documentation. 

## Understanding FabricPool’s Security Policies 
FabricPool sustains AES-256-GCM encryption on the local tier, on the cloud tier, and over the wire when moving data between tiers. Read more about FabricPool’s security protocols and features in Appendix B of this documentation. 

## Completing Key Tasks to Configure FabricPool 	
### 1. How to Create Containers/Buckets in a Local Tier
Buckets are containers that hold data. Once created, buckets can be added to local tiers as cloud tiers. You can create buckets to attach to FabricPool in a few ways: 

**How to Create a Bucket/Cloud Tier with NetApp StorageGRID** 
1. Open StorageGRID Tenant Manager 
2. Open the Admin Node in a web browser 
3. Log in with your tenant account ID, username, and password 
4. Select S3 
5. Select Buckets 
6. Click Create Bucket 
7. Provide a DNS compliant name 
8. Click Save 

**How to Create a Bucket/Cloud Tier with ONTAP S3**

You can find instructions for creating buckets with ONTAP S3 in the following technical documentation: [Provisioning Object Storage with System Manager](https://docs.netapp.com/us-en/ontap/s3-config/create-bucket-task.html) 

**How to Create a Bucket/Cloud Tier with Other Object Store Providers** 
<p>You can find instructions for creating cloud tiers on other object store providers on their respective sites.</p>

- [Alibaba Cloud Object Storage Service](https://www.alibabacloud.com/help/en/oss/getting-started/console-quick-start) 
- [Amazon S3](https://docs.aws.amazon.com/AmazonS3/latest/gsg/CreatingABucket.html)
- [Google Cloud Storage](https://cloud.google.com/storage/docs/creating-buckets) 
- [IBM Cloud Object Storage](https://cloud.ibm.com/docs/services/cloud-object-storage?topic=cloud-object-storage-getting-started)
- [Microsoft Azure Blob Storage](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-quickstart-blobs-portal)

### 2. How to Attach Cloud Tiers 
FabricPool supports the attachment of two buckets (cloud tiers) per local tier using FabricPool Mirror though attaching a single cloud tier is more common. A single cloud tier can be attached to a single local tier, and a single cloud tier can be attached to multiple local tiers. Attaching a single cloud tiger to multiple local tiers in a cluster is the general best practice. NetApp does not recommend attaching a single cloud tier to local tiers in multiple clusters. Make sure to review your public object store provider policies to ensure that your storage architectures comply. If they do not comply, this can cause performance drops across multiple local tiers that tier to the same cloud tier. 

### 3. How to Add Cloud Tiers with ONTAP System Manager
Before you can attach a cloud tier to a local tier and create a FabricPool, the cloud tier must be added to and identified by ONTAP. This can be done through the ONTAP System Manager. In order to add the cloud tier using the ONTAP System Manager, or the CLI, you need to have the following: 
- Server name (FDQDN). For example, s3.amazonaws.com. (Note: Azure might require the account prefix; accountprefix.blob.core.windows.net)
- Access key ID
- Secret key
- Bucket name (container name) 

**To add a cloud tier using the ONTAP System manager, do the following:** 
1. Launch ONTAP System Manager 
2. Click on Storage 
3. Click Tiers 
4. Click Add Cloud Tier 
5. Select and object store provider 
6. Complete the text fields as required by your object store provider 
**Note:** Make sure to enter the object store’s bucket name in the Container Name field 
7. Optional: cloud tiers can be attached to local tiers later if desired
8. Add the cloud tier to local tiers as a primary cloud or as a FabricPool Mirror. 
**Note:** Attaching a cloud tier to a local tier is a permanent action. The tiers cannot be unattached. You can use FabricPool Mirror to attach a different cloud tier. 
9. Click Save 

### 4. How to add tiers using ONTAP CLI
**To add tiers using ONTAP CLI, use the following commands:** 
```
object-store config create
-object-store-name <name>
-provider-type <AliCloud/AWS/Azure_Cloud/CAP/GoogleCloud/IBM_COS/ONTAP_S3/S3_Compatible/SGWS>
-port <443/8082> (public clouds/SGWS)
-server <name>
-container-name <bucket-name>
-access-key <string>
-secret-password <string>
-ssl-enabled true
-ipspace default
-is-certificate-validation-enabled true
-use-http-proxy false
-url-style <path-style/virtual-hosted-style>
```

### 5. How to use ONTAP to ONTAP tiering policies 
Starting with ONTAP 9.8, tiering to buckets created using ONTAP S3 is supported. This allows for ONTAP to ONTAP tiering. Any buckets in the local cluster are known to ONTAP automatically and should be available as an option when attaching a cloud tier to a local tier.

### 6. How to make sure the CA certification process is completed correctly 
CA certificates associated with private cloud object stores, like StorageGRID and some IBM Cloud Object Storage environments should be installed to ONTAP before attaching them to local tiers. The steps for that process are found here. Using CA certificates creates a trust relationship between ONTAP and the object store and helps secure access to management interfaces, gateway nodes, and storage. 

**Note:** Failure to install a CA certificate will result in an error unless certificate validation is turned off. Turning this off is not recommended, but is possible with ONTAP 9.4 and newer. You can find the process to turn off CA certificate validation in Appendix B of this documentation. 

### 7. How to attach a local tier to a cloud tier to create a FabricPool 
After an object store/bucket has been added to and identified by ONTAP as a cloud tier, it can be attached to a local tier to create a FabricPool. You can complete this process by using either ONTAP System Manager or ONTAP CLI. There are volume policies that can affect cloud-tier attachments, you can read about these in Appendix B of this documentation. 

**How to attach a cloud tier to a local tier with ONTAP System Manager** 
1. Launch ONTAP System Manager 
2. Click STORAGE 
3. Click the name of a local tier 
4. Click More 
5. Click Attach Cloud Tiers 
6. Select the primary cloud tier to attach 
7. Select volumes to set tiering policies 
8. Click Save (**Note:** by clicking save, you complete the permanent action of attaching a cloud tier to a local tier). 

**How to attach a cloud tier to a local tier with ONTAP CLI** 
To attach a cloud tier to a local tier with ONTAP CLI, run the following commands: 
```
storage aggregate object-store attach
-aggregate <name>
-object-store-name <name>
```
*Example:*
```
storage aggregate object-store attach -aggregate aggr1 -object-store-name aws_fabricpool_bucket
```
You can also attach a cloud tier to local tiers used by a FlexGroup volume. **To do this, run the following commands (for more information on FlexGroup Volumes, navigate to Appendix B):** 
```
volume show -volume <name> -fields aggr-list
```
 
**Then, run the following commands:** 
```
storage aggregate object-store attach
-aggregate <name>
-object-store-name <name>
-allow-flexgroup true
```

### 8. How to attach a local bucket to a local tier with ONTAP System Manager 
1. Launch ONTAP System Manager 
2. Click Storage 
3. Click the name of a local tier 
4. Click More 
5. Click Tier to Local Bucket
6. Select Existing or New (NOTE: if New, a new SVM and bucket is created. If available ONTAP System Manager selects low-cost media (FAS HDD) for the bucket) 
7. Select bucket capacity
8. Click Save (**Note:** when a new bucket is created, its secret key is displayed. Save/download this key because it will not be displayed again) 

### 9. How to implement cloud retrieval 
When using the default volume tiering policy (Auto volume tiering), if cold data blocks are read sequentially, they remain cold and on the cloud tier. This tends to be desirable behavior for most client needs and keeps deep file scans common to antivirus and analytics applications from writing cold data back to the local tier. However, starting with ONTAP 9.8, volumes can set cloud retrieval policies that override this behavior. 

**FabricPool provides 4 cloud retrieval policies to choose from:** 
**Default:** When cold blocks in a volume are read, they use the default behavior of their volume tiering policy 

**Never:** Cold blocks on a volume will remain cold when read and on the cloud tier. They will not be written to the local tier. This policy continues to use the volume’s tiering-minimum-cooling-days setting rather than being tiered as soon as possible. 

**On-Read:** When cold blocks in a volume with this cloud tiering policy set are read, randomly OR sequentially, they are made hot and written to the local tier. 

**Promote:** this cloud tiering policy immediately quees tiered data to return to the local tier-provided the tiering policy allows it: 

You can change a volume’s tiering cloud retrieval policy using ONTAP CLI. **To do so, run the following command:**

### 10. How to use FabricPool Mirror 
Beginning with ONTAP 9.7, FabricPool Mirror allows the attachment of two cloud tiers to a single local tier which creates more options for keeping data available and flexibly organizing it. 

FabricPool Mirror reflects data across two buckets. While buckets synchronize, data is read from the pre-existing primary bucket and written to the secondary bucket. This creates a mirrored state between the two buckets. 

Once the buckets are mirrored, any newly tiered data is simultaneously tiered to both buckets. Since data is being tiered to two buckets, the effective throughput is equal to half the rate of standard single-bucket tiering operations. All GET operations will take place from the primary bucket. If connectivity is interrupted to the primary bucket, then GET operations will take place from the secondary bucket. If connectivity to both buckets is blocked, then tiering operations are temporarily suspended until the connection is re-established. 

When adding a FabricPool Mirror to an existing FabricPool, any data previously tiered to the original cloud tier read from the primary bucket is written to the newly attached secondary bucket. 
