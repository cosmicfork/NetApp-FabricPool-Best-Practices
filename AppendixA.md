Understanding FabricPool's Architecture

FabricPool Overview:
--------------------

-   FabricPool is a feature in ONTAP that connects a local storage tier with a cloud tier. The cloud tier is an external object storage. Both of these tiers make a FabricPool, letting volumes within the tiers manage data efficiently and promptly. Hot(active) data stays in the local tier for high performance, while cold(inactive) data is moved to the cloud tier.

-   Understanding FabricPool is essential to building effective storage solutions with ONTAP.

Block Temperature:
------------------

-   Blocks are assigned a temperature when written to the local tier(hot).

-   A background cooling scan cools blocks over time.

-   NOTE - The All volume tiering policy is an exception to this. Blocks in volumes using the all tiering policy are immediately identified as cold and marked for tiering.

Object Creation:
----------------

-   FabricPool works at the WAFL block level, making 4MB objects from cooled blocks and sending them to the cloud tier.

-   Each object is made up of 1,024 4KB blocks.

-   The fixed object size is 4MB, but actual sizes may be less due to ONTAP storage efficiencies.

Understanding Data Movement:
----------------------------

-   Cold blocks marked for tiering are collected into 4MB objects and moved to the cloud tier.

-   The tiering fullness threshold decides when tiering happens, with the default set at >50% local tier capacity. (In ONTAP 9.5, the 50% tiering fullness threshold can be changed).

-   Write-back prevention occurs if the local tier is >90% full. This affirms that the local tier is used for active data.

-   ![Image1](https://lh7-us.googleusercontent.com/xvrsjJ2nv8H5lIityxmlEt5nVIBLLunCyy1ZIgW94vSo4dB_nSVgbJa1DPtdNizgTpcyxYmzq9F88xguWbrTAUikMM9aED5FJ1dbkukjglMX7IjkTFi7U3mSloHKJ64rsltdNxnY8D-Dz8VAnvFyHow)

-   ![Image2](https://lh7-us.googleusercontent.com/hJUTscLa0N3DuXEAET0bFgFn4yP5vzs8kx35gbE3zN-EN5BRSaX-d7eTZ6SrT4IoJJBt8vryiZ3alNQV82T1zgov-ZvGn9wkW1QLbKyvrwKEUzQ1i-KYrA5l2wzjSQaBeMdZG3Fx7FqVVdSD_UkKabY)

SnapMirror Behavior:
--------------------

-   SnapMirror controls the movement of data between the local and cloud tiers based on tiering policies.

-   ![Image3](https://lh7-us.googleusercontent.com/4rGaep_cs3zguaLuxHTCTnXebGNYXd7INCph7p4dc7G5tSEcWH6fsgsBxztb8kf0Hex8iH-z8XkOS3IFH7_K85D10Zzx9zd8bcV3XLEavrlFTArmL94HBy9TsmMf34hvrUWRpuHS-bTFwQ-cBACJiZk)

Volume Move:
------------

-   Volume move(vol move) is a non-disruptive function that moves a volume from one local tier(source) to another destination. 

### Destination Local Tier:

-   If the volumes destination local tier does not have a connected cloud tier, then data on the source volume's cloud tier moves to the local tier on the destination.

-   Starting with ONTAP 9.6, data in the same bucket doesn't move back which promotes network efficiency. 

-   NOTE - Some configurations are incompatible with optimized volume moves:

-   Changing tiering policy during volume move

-   Source and destination aggregates use different encryption keys

-   FlexClone volumes

-   FlexClone parent volumes

-   MetroCluster

-   Unsynchronized FabricPool Mirror buckets

-   Different Amazon S3 naming formats for source and destination aggregates

### Destination Local tier with a  Cloud tier:

-   If the destination local tier has an attached cloud tier, Cloud-tiered data is written to the local tier first, then to the cloud tier.

-   This enhances volume move performances and significantly reduces cutover time.

Volume Tiering Policy:
----------------------

-   If no tiering policy is indicated, the destination volume uses the source volume's policy.

-   Different policies can be specified during volume move, impacting destination volume creation.

-   Source and destination volumes in SVM DR are required to use the same tiering policy.

### Minimum Cooling Days:

-   In Pre-ONTAP 9.8, moving a volume resets block inactivity period on the local tier. For example, a volume using the auto volume tiering policy with data on the local tier that has been inactive for 20 days has data inactivity reset to 0 days after a volume move.

-   In ONTAP 9.8+, blocks inactive longer than destination volume's minimum cooling days are tiered. Blocks with less inactivity reset to 0 days.

### How to configure Tiering Policies:

-   The Auto tiering policy moves all data to the local tier  first:

-   ![Image4](https://lh7-us.googleusercontent.com/gF5c2zxYkgIGAn1q08U0BUSzBdqHjFr8po1rmPs5dJ3R-8VK5nQ0umYKVpeV4GamU7WMlEx714qiFRVsDBtb5lbKiYCAgQFBR8K8c6OP1jUXBeIQUIBCRMwpjKHZca3lZMqUY99Gz6c4k3_I5Sg5G5o)

-   The Snapshot-Only tiering policy moves data variably, prioritizing the local tier. COld Snapshot blocks move to the cloud tier:

-   ![Image5](https://lh7-us.googleusercontent.com/QM5Z8Vak7yeOz0-CKtvUSQy33zQf54jkYQNKFGYZck49gf0igsC3bjABuujaxkmMIeL3Dzxd2cQULrme1ULb77rVXATTzOwhcdYF-YPjzU3A2U0d4LrqXJSwR1olo0u2a86LowZTFE2VEo19vtUScO8)

-   The All tiering policy immediately identifies cold data, writing to the cloud tier: 

-   ![Image6](https://lh7-us.googleusercontent.com/2at8bYZ2rTDqQ1ueZR9FXODHJFhtMusDiGi6xw3J22_7_S_oGGaYjNUJb-htudEK6aYNGcOmPwX5hF3aYYqnWxQGl9vGsY0jqH2zgakOBvER9KQQLA6-SlC-GY54z2BPTul826Nthc6g0ZCioXn7GoY)

-   The None tiering policy writes data to the local tier:

-   ![Image7](https://lh7-us.googleusercontent.com/lxXnx-ndGAEIkM9KFEiczrTcP3Vpdrk-j1ghhYHPJP3R_to6lE5wh36Vb8oYEaSG8ljRjSfEh57Kaft3FmVN7OubWDAiMC683JVG45Kaj1Ma0bDn1QkzTRSC5va9DL89HkNKLuTEio0XQeNAjIkJQgI)

### How to perform a volume move with ONTAP System Manager:

1.  ![Image8](https://lh7-us.googleusercontent.com/aIrww7QyHh-Wjj8ZVIa6qCFQnB9itNCOR-AQ4kxOOTJCIhElIN6FyNYkLlLmBiqQlymvAEQlLV2zep8X4xcaocODcf4V98uR7-yLz1AmaI75__etm0q-IzPPN0FOF5EQUUkcMdX728SooXVhRkvYK00)

2.  To perform a volume move with ONTAP system manager, follow these steps:

3.  Click STORAGE  

4.  Click Volumes 

5.  Select the volume you want to move 

6.  Click More 

7.  Click Move 

8.  Select a destination local tier 

9.  Selecting a tiering policy 

10. Click Move.

### How to perform a volume move using ONTAP CLI:

-   To perform a volume moving using the ONTAP CLI, run the following command:

-   ![Image9](https://lh7-us.googleusercontent.com/aPHHdgA1PPtKKAPR9c_31Il_4uAwlpL0vAmdAEtyCiox7S5TZnTps48ErgmlrJqz_J6JpRze-ZEH8GWDMqnUZ0tlaZqOkCLkjBtSJY7_lCNnSQlrGQvg2TPtKhc2fAMmxLbnwijpgoKn0OWup7rgaRU)

### FlexClone volumes:

-   FlexClone volumes are copies of a parent FlexVol volume. These copies inherit the volume tiering policy and the tiering-minimum-cooling days settings of the parent FlexVol volume.

-   NetApp recommends using tiering settings on the parent FlexVol that are either equal to or less aggressive than any of the clones. As a best practice, this keeps more data owned by the parent volume on the local tier, increasing the performance of the clone volumes.

-   If a FlexClone volume is split (volume clone split) from its parent volume, the copy operation writes the FlexClone volume's blocks to the local tier.

### FlexGroup volumes:

-   A FlexGroup volume is a single namespace that is made up of multiple constituent member volumes but is managed as a single volume. Individual files in a FlexGroup volume are allocated to individual member volumes and are not striped across volumes or nodes.

-   FlexGroup volumes are not constrained by the 100TB and two-billion file limitations of FlexVol volumes. Instead, FlexGroup volumes are only limited by the physical maximums of the underlying hardware and have been tested to 20PB and 400 billion files. Architectural maximums could be higher.

-   Volume tiering policies apply at the FlexGroup volume level.

-   FabricPool Provisioning is recommended for each cluster node but not mandatory.

Object Storage:
---------------

-   Object storage manages data as objects in containers or buckets.

-   Objects are put inside a container and are not nested as files inside a directory.

-   Object storage offers less performance than file or block storage.

-   This offers scalability beyond traditional file or block storage.

### FabricPool Cloud Tiers:

-   FabricPool supports various cloud providers for object storage like Amazon, Google, IBM, etc for cloud tiers.

-   More than one type of cloud tier can be used in a cluster.

-   In ONTAP 9.7+, FabricPool mirror enables the attachment of two cloud tiers to one single local tier.

ONTAP S3:
---------

-   ONTAP 9.8+ supports tiering to buckets created using ONTAP S3.

-   When tiering more than 300TB of inactive data, NetApp recommends using StorageGRID.

### Object Deletion and Defragmentation:

-   FabricPool deletes entire objects based on unreferenced blocks.

-   For example, there are 1,024 4KB blocks in a 4MB object tiered to Amazon S3. Defragmentation and deletion do not occur until less than 205 4KB blocks (20% of 1,024) are being referenced by ONTAP. When enough (1,024) blocks have zero references, their original 4MB objects are deleted, and a new object is created.

-   You can change the percentage.

-   ![Image10](https://lh7-us.googleusercontent.com/AHARACRRF2tyRaklqgM_8E06R2tZPL_UCTOuNOx8WoaxzbxRo-Q74RBZLBwi-YXmFR6L9HtfYQZ5rfuBtQiAUZH58QWLYFwSeVwjlZMo1k8z-tYg7fDOLHFJGk5MXfwPRMJrwShOFj8dnO21sHK0o1A)

-   Alternatively, consider increasing unreclaimed space thresholds if object fragmentation causes significantly more object store capacity to be used than necessary for the data being referenced by ONTAP. For example, using an unreclaimed space threshold of 20% in a worst-case scenario where all objects are equally fragmented to the maximum allowable extent means that it is possible for 80% of total capacity in the cloud tier to be unreferenced by ONTAP. For example:

-   2TB referenced by ONTAP + 8TB unreferenced by ONTAP = 10TB total capacity used by the cloud tier.

-   ![Image11](https://lh7-us.googleusercontent.com/PJnphBec0zqoPO8pgoZEmTEP7fRBcqGy4RgWMBs44er2ZifO9OQBL44hUB0mhOki5EhHWDOQ-oIyKmZsAFpCsK45V1vlbkdwUa8QdV-NfTEJJDy8MgDNyDcM8OwTpDcODZ9er3yw4RaXrxnUYtmp5-E)

### ONTAP storage efficiencies:

-   Compression, deduplication, and compaction are preserved when moving data to the cloud.

-   NOTE - Third-party deduplication has not been qualified by NetApp.

### Temperature-Sensitive Storage Efficiency(TSSE):

-   Beginning with ONTAP 9.8+, compression adjustments are based on data temperature.

-   Supported on FabricPool-enabled local tiers with ONTAP 9.10.1+

-   TSSE scans the temperature of the data and compresses larger or smaller blocks of data accordingly.

-   TSSE makes data storage more efficient.