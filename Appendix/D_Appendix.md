# Appendix D
## NetApp Private Storage for AWS
NetApp Private Storage (NPS) for AWS is fully compatible with FabricPool, ensuring performance and security. This cloud-connected storage architecture allows you to create a dynamic cloud infrastructure that seamlessly blends the scalability of AWS with the effiency of NetApp storage.

Typically deployed within AWS-approved Direct Connect partner colocation data centers such as Equinix, NPS for AWS leverages AWS Direct Connect to establish low-latency, highly accessible, dedicated connections between NetApp storage and the AWS cloud.

## Data tiering within Cloud Volumes ONTAP 
If you want to learn more details about data tiering, you can find all the documentation on the [NetApp Cloud Docs site](https://docs.netapp.com/us-en/cloud/).
## Data Tiering with Cloud Volumes ONTAP: Data Tiering Overview
By pairing a high-performance SSD or HDD tier for active data with a cost-effective object storage capacity tier for less frequently accessed data, you can achieve efficient and economical data management. This resource is for if you want to optimize your Cloud Volumes ONTAP storage costs by leveraging FabricPool technology:
[Data Tiering with Cloud Volumes ONTAP: Data Tiering Overview](https://docs.netapp.com/us-en/bluexp-cloud-volumes-ontap/concept-data-tiering.html)
### Tiering inactive data to low-cost object storage
By teaming up a SSD or HDD tier for your frequently used data and a budget-friendly object storage capacity tier for less active data, you can significantly reduce storage costs. This resource is if you want a more in depth overview of data tiering: 
[Tiering inactive data to low-cost object storage](https://docs.netapp.com/us-en/bluexp-cloud-volumes-ontap/task-tiering.html)
### Volume data tiering
When you enable automatic data tiering, the Amazon FSx file system automatically organizes your data based on your volume's tiering policy, making sure everything is in its right place for optimal performance and cost-effectiveness. This resource is if you want to learn more about third party NetApp filing systems: 
[Volume data tiering](https://docs.netapp.com/us-en/bluexp-cloud-volumes-ontap/task-tiering.html)
### Additional Resources
To learn more about the information that is described in this document, you can check out the following documents and websites:

- [ONTAP 9 Documentation](https://docs.netapp.com/us-en/cloud/) <br>
- [ONTAP Reference](https://docs.netapp.com/us-en/ontap/concepts/manual-pages.html)<br>
- [StorageGRID 11.7 Documentation](https://docs.netapp.com/us-en/storagegrid-117/)<br>
- [ONTAP S3 configuration overview with System Manager](https://docs.netapp.com/us-en/ontap/s3-config/index.html)<br>
- [Setup licensing for BlueXP Tiering](https://docs.netapp.com/us-en/bluexp-tiering/task-licensing-cloud-tiering.html)<br>
- [ONTAP FabricPool Licensing Overview](https://kb.netapp.com/onprem/ontap/dm/FabricPool/ONTAP_FabricPool_(FP)_Licensing_Overview)<br>
- [TR-4015: SnapMirror Configuration and Best Practices Guide](https://www.netapp.com/pdf.html?item=/media/17229-tr4015pdf.pdf)<br>
- [TR-4375: NetApp MetroCluster FC](https://www.netapp.com/pdf.html?item=/media/13482-tr4375pdf.pdf)<br>
- [TR-4571: FlexGroup Volume Best Practices](https://www.netapp.com/us/media/tr-4571.pdf)<br>
- [TR-4626: StorageGRID load balancer](https://www.netapp.com/us/media/tr-4626.pdf)<br>
- [TR-4689: NetApp MetroCluster IP](https://www.netapp.com/us/media/tr-4689.pdf)<br>
- [TR-4695: Database Storage Tiering with FabricPool](https://www.netapp.com/us/media/tr-4695.pdf)<br>
- [TR-4814: ONTAP S3 Best Practices](https://www.netapp.com/us/media/tr-4814.pdf)<br>
- [TR-4826: FabricPool with StorageGRID](https://www.netapp.com/us/media/tr-4826.pdf)


