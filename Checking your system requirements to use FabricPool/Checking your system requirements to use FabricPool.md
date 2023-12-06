# Checking your system requirements to use FabricPool
This section covers all of the absolute requirements for setting up and using FabricPool. This includes tiering requirements, licensing requirements, Transmission Control Protocol (TCP) connection requirements, volume requirements, quality of service minimums requirements, and cluster high-availability pair requirements. This section does not include system recommendations, best practices are covered in the following section. 

## Compatible tiering requirements
FabricPool requires ONTAP 9.2 or later. In releases earlier than ONTAP 9.8, FabricPool is only supported on SSD local tiers. Additional FabricPool requirements depend on the version of ONTAP  and the intended cloud tier attachment. 

### Compatible platforms
* AFF
* FAS

FabricPool is supported on all FAS platforms except for the following:
* FAS8020
* FAS2554
* FAS2552
* FAS2520

### Compatible cloud tiers
*  Alibaba Cloud Object Storage Service (Standard, Infrequent Access)
* Amazon S3 (Standard, Standard-IA, One Zone-IA, Intelligent-Tiering, Glacier Instant Retrieval)
* Amazon Commercial Cloud Services (C2S)
* Google Cloud Storage (Multi-Regional, Regional, Nearline, Coldline, Archive)
* IBM Cloud Object Storage (Standard, Vault, Cold Vault, Flex)
* Microsoft Azure Blob Storage (Hot and Cool)
* NetApp StorageGRID 10.3 and later

**Note:** Glacier Flexible Retrieval and Glacier Deep Archive are not supported.

Compatible data tiers
• Amazon FSx for NetApp ONTAP 
• Azure NetApp Files 
• NetApp Cloud Volumes for Google Cloud 
• NetApp Cloud Volumes ONTAP for AWS 
• NetApp Cloud Volumes ONTAP for Azure

## Licensing Requirements 
FabricPool requires a capacity-based license when attaching third-party object storage providers (such as Amazon S3) as cloud tiers for AFF and FAS systems. 

### Cloud tiers that do NOT require licensing
* StorageGRID 
* ONTAP S3 
* Amazon S3 for Cloud Volumes ONTAP
* Google Cloud Storage for Cloud Volumes ONTAP
* Microsoft Azure Blob Storage for Cloud Volumes ONTAP

### Purchasing and activating licenses 
To activate a new Cloud Tiering license (including add-on or extensions to preexisting FabricPool licenses), go to the Cloud Manager Digital Wallet. Here, you can set up and configure tiering by using the Cloud Tiering service. Cloud Tiering licenses are available in pay-as-you-go subscriptions from cloud provider marketplaces, or 2-, 12-, 24-, and 36-month term-based licenses. Cloud Tiering licenses (including additional capacity for existing licenses) are available for purchase in 1TB increments.

**Note:** Cloud Tiering licenses are attached to a customer’s account. Total tiering capacity can be used across multiple clusters.

### Capacity-based licenses for third-party object storage
Tiering to the cloud tier stops when the amount of data (used capacity) stored on the cloud tier reaches the licensed capacity. License capacity must increase for additional data (including SnapMirror copies to volumes using the All Tiering policy) for tiering to continue. Although tiering stops, your data remains accessible from the cloud tier. Additional cold data remains on the local tier until the licensed capacity is increased.

### Additional Licensing 
ONTAP clusters tiering to endpoints other than Amazon S3, Google Cloud Storage, and Microsoft Azure Blob Storage can use Cloud Tiering licenses, but the license must be applied in a different manner than typical single-node and HA-configured ONTAP clusters.

**For more information, see:** [ Applying Cloud Tiering licenses to clusters in special configurations.](https://docs.netapp.com/us-en/bluexp-tiering/task-licensing-cloud-tiering.html#apply-cloud-tiering-licenses-to-clusters-in-special-configurations)

**Note:** Legacy 12-month FabricPool licenses are required for NetApp MetroCluster, FabricPool Mirror, and dark site or other air-gapped environments that are not yet supported by Cloud Tiering.

## Transport Layer Security configuration
When FabricPool uses StorageGRID or other private clouds such as some IBM Cloud Object Storage environments as a cloud tier, it must use a Transport Layer Security (TLS) connection. If you have Certificate Authority (CA) certificates associated with private cloud object stores, you should install them on ONTAP before attaching them to local tiers. Using CA certificates creates a trusted relationship between ONTAP and the object store and helps to secure access to management interfaces, gateway nodes, and storage.

**Note:** Although beginning in ONTAP 9.4, CA certificates are no longer required, Failure to install a CA certificate results in an error unless certificate validation is turned off. See [CA certificate recommendations](#ca-certificate-recommendations) 

## Transmission Control Protocol (TCP) connection requirements
Transmission Control Protocol (TCP) is a connection between a client and a server, which is established prior to sharing data on an IP (Internet Protocol) network. Object store infrastructure must be capable of supporting at least 700 TCP connections. FabricPool can use 1600-3800 TCP connections per node, per object store endpoint. Server-side load balancers, firewalls, and proxies must be sized to appropriately handle FabricPool traffic. FabricPool Mirror doubles this number because it connects to two different endpoints. 

**Note:** To view your Object store information, go to the Clusters page, then click the menu icon for a cluster and select Object Store Info.

## Volume Requirements 
In order to attach a cloud tier to a local tier that contains volumes, the space guarantee must be set to None. Setting the “space-guarantee none” parameter assures thin provisioning of the volume. This means the amount of space consumed by volumes with this guaranteed type grows as data is added. 

**To set the Volume space guarantee to None, run the following command in ONTAP CLI:**

volume modify -space-guarantee none

**Note:** The amount of space consumed by volumes is determined by the initial volume size if you fail to set the “space-guarantee none” parameter. Setting the “space-guarantee none” parameter is essential for FabricPool because the volume must support cloud tier data that becomes hot and is brought back to the local tier.

## Quality of service minimums requirements
Quality of service minimums (QoS Min) provides performance minimums, while FabricPool sends blocks to an object store and decreases performance. QoS Min must be turned off on volumes in FabricPool local tiers. Alternatively, tiering must be turned off on volumes that require QoS Min.

**To turn off tiering on volumes requiring QoS Min, run the following command in ONTAP CLI:**

-tiering-policy none

## Cluster high-availability pair requirements
Cluster high-availability (HA) pairs that use FabricPool require two intercluster logical interfaces (LIFs) to communicate with the cloud tier. Creating intercluster LIFs enables the cluster network to communicate with a node.

**Note:** Disabling or deleting an intercluster LIF interrupts communication to the cloud tier.

# Checking FabricPool System Recommendations

## CA certificate recommendations
A CA certificate is a digital certificate issued by a Trusted Third Party (TTP). Beginning in ONTAP 9.4, CA certificates are no longer required. However, using signed certificates from a third-party CA remains the recommended best practice. 

### Potential errors
Failure to install a CA certificate results in an error unless certificate validation is turned off.

FabricPool requires that CA certificates use the same fully qualified domain name (FQDN) as the cloudtier server with which they are associated. In releases earlier than StorageGRID 11.3, the default CA certificates use a common name (CN) that is not based on the server’s FQDN. To successfully attach StorageGRID 11.2 or earlier releases as a cloud tier, you must replace the certificates in the grid with certificates that use the correct FQDN. Although you can use self-signed certificates, using signed certificates from a third-party certificate authority is the recommended best practice. Using the common name causes certificate-based errors that prohibit StorageGRID from attatching to ONTAP local tiers, for example:

* Unable to add a cloud tier. Cannot verify the certificate provided by the object store server. The certificates might not be installed on the cluster. Do you want to add the certificate now?
* Cannot verify the certificate provided by the object store server.

### Installing CA certificates
To install CA certificates in ONTAP, complete the following steps:

1. Retrieve the CA certificates.
2. Install the certificates into ONTAP.

### Retrieving CA certificates
Retrieve the Root CA certificate and, if they exist, any intermediate CA certificates in Base-64 encoded format (sometimes also called PEM format) from the Certification Authority who created the certificate. If you followed the procedure for StorageGRID SSL Certificate Configuration these are the certificates in the chain.pem file.

**To retrieve the certificate for a StorageGRID endpoint, complete the following steps:**
1. Open the StorageGRID Administration console.

2. Select Configuration > Load Balancer Endpoints.

3. Select your endpoint and click Edit Endpoint.

4. Copy the certificate PEM, including:
-----BEGIN CERTIFICATE----- and -----END CERTIFICATE-----

To retrieve the certificate when using a third-party load balancer, complete the following steps:
1. Run the following command using ONTAP CLI:
openssl s_client -connect <FQDN> -showcerts

2. Copy the certificate, including:
-----BEGIN CERTIFICATE----- and -----END CERTIFICATE-----

## Installing certificates to ONTAP
When adding a new Cloud Tier of type StorageGRID in your ONTAP System Manager, you can paste the CA certificate. If there is an intermediate CA which issued the StorageGRID certificate, then this must be the intermediate CA certificate. If the StorageGRID certificate was issued directly by the Root CA, then you must use the Root CA certificate. 

**To install the Root certificates (and any intermediate certificates), run the following command in ONTAP CLI:**
security certificate install -vserver <name> -type server-ca

## IPspace recommendations 
If you are using more than one intercluster LIF on a node with different routing, NetApp recommends placing them in different IPspaces. During configuration, FabricPool can select from multiple IPspaces, but it is unable to select specific intercluster LIFs within an IPspace.

**Note:** Disabling or deleting an intercluster LIF interrupts communication to the cloud tier.

### Internet Protocol version
Beginning in ONTAP 9.9.1, FabricPool supports IPv6. Prior to ONTAP 9.9.1 FabricPool only supports IPv4.

## FlexGroup volume recommendations 
When provisioning FlexGroup volumes on FabricPool local tiers (storage aggregates), ONTAP System Manager requires that the FlexGroup volume uses FabricPool local tiers on every cluster node. This is a recommended best practice but is not a requirement when manually provisioning FlexGroup volumes. Provisioning FlexGroup constituent volumes on heterogeneous local tiers (some using FabricPool, some not using FabricPool) is not recommended and will result in unpredictable tiering and performance.

## Cluster high-availability pair recommendations
If you are using more than one intercluster LIF on a node with different routing, NetApp recommends placing them in different IPspaces. During configuration, FabricPool can select from multiple IPspaces, but it is unable to select specific intercluster LIFs within an IPspace. NetApp recommends creating an intercluster LIF on additional HA pairs to seamlessly attach cloud tiers to local tiers on those nodes as well. If you are using more than one intercluster LIF on a node with different routing, NetApp recommends placing them in different IPspaces. During configuration, FabricPool can select from multiple IPspaces, but it is unable to select specific intercluster LIFs within an IPspace.

