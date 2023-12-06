# Appendix C
## CLI commands
### All-access commands

**Volumes & Tiering**

#### To set the Volume space guarantee to None, run the following command in ONTAP CLI:

volume modify -space-guarantee none

#### To view FabricPool space utilization details using the ONTAP CLI, run the following command:
storage aggregate object-store show-space


#### To turn off tiering on volumes requiring QoS Min, run the following command in ONTAP CLI:

-tiering-policy none

#### To perform a volume move using the ONTAP CLI, run the following command:

vol move start -vserver <name> -volume <name> -destination-aggregate <name> -tiering-policy
<policy>

#### To add a cloud tier by using the ONTAP CLI, run the following commands:
object-store config create -object-store-name -provider-type -port <443/8082> (public clouds/SGWS) -server -container-name -access-key -secret-password -ssl-enabled true -ipspace default -is-certificate-validation-enabled true -use-http-proxy false -url-style 

#### To attach a cloud tier to a local tier (storage aggregate) by using the ONTAP CLI, run the following commands:
storage aggregate object-store attach
-aggregate <name>
-object-store-name <name>

#### To list the local tiers used by a FlexGroup volume, and attach a cloud tier to those local tiers by using the ONTAP CLI, run the following commands:
volume show -volume <name> -fields aggr-list

#### To change a volume’s tiering policy by using the ONTAP CLI, run the following command:

volume modify -vserver <svm_name> -volume <volume_name> -tiering-policy <auto|snapshotonly|all|none>

#### To attach an additional cloud tier, a FabricPool Mirror, to a local tier (storage aggregate) using the ONTAP CLI, run the following commands:
storage aggregate object-store mirror -aggregate <aggregate name> -name <object-store-name-2>

#### To swap cloud tiers so the primary and secondary cloud tier relationships change in a FabricPool Mirror using the ONTAP CLI, run the following commands:
storage aggregate object-store modify -aggregate <aggregate name> -name <object-store-name-2> -
mirror-type primary

#### To add tags to a volume using the ONTAP CLI, run the following commands:
volume modify <name> -tiering-object-tags <key1=value1>,<key2=value2>, <key3=value3>, etc.

**Licensing**
#### To view the capacity status of the FabricPool license using the ONTAP CLI, run the following command:
system license show-status

**Certificates**
#### You can turn off CA certificate validation when adding a private cloud tier by using the ONTAP CLI. To do so, run the following commands:
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

#### To view certification when using a third-party load balancer using the ONTAP CLI, run the following command:
openssl s_client -connect <FQDN> -showcerts

#### To install the Root certificates (and any intermediate certificates), run the following command in ONTAP CLI:
security certificate install -vserver <name> -type server-ca

**Inactive data reporting (IDR)**
#### To enable IDR on a non-FabricPool local tier, run the following command:
storage aggregate modify -aggregate <name> -is-inactive-data-reporting-enabled true
#### To display IDR by using the ONTAP CLI, run the following command:
storage aggregate show-space -fields performance-tier-inactive-user-data, performance-tierinactive-user-data-percent
#### To display IDR on a single volume by using the ONTAP CLI, run the following command:
Volume show -fields performance-tier-inactive-user-data, performance-tier-inactive-user-datapercent

**Object Store Profiler** 
#### To remove the FabricPool Mirror by using the ONTAP CLI, run the following commands:
storage aggregate object-store unmirror -aggregate <aggregate name> -name <object-store-name-1>

**Error messages**
#### To view the MetroCluster error messages, run the following command: 
metrocluster check show
### Advanced privilege commands

**Volumes & Tiering**
#### To view the status of the tiering scan, run the following command:
volume object-store tiering show

#### To change the tiering fullness threshold, run the following command:
storage aggregate object-store modify –aggregate <name> –tiering-fullness-threshold <#> (0%-99%)
-object-store-name <name>

#### To change the default unreclaimed space threshold, run the following command:
storage aggregate object-store modify –aggregate <name> -object-store-name <name> –unreclaimedspace-threshold <%> (0%-99%)

#### To disable cloud tier encryption, run the following command:
storage aggregate object-store config modify -serverside-encyption false

#### To change a volume’s cloud retrieval policy using the ONTAP CLI, run the following command:
volume modify -vserver <svm_name> -volume <volume_name> -cloud-retrieval-policy
<default|never|on-read|promote>

#### To change a volume’s tiering minimum cooling days setting using the ONTAP CLI, run the following command:
volume modify -vserver <svm_name> -volume <volume_name> -tiering-minimum-cooling-days <2-183> 

**Object store profiler**
#### To start the object store profiler using the ONTAP CLI, run the following command:
storage aggregate object-store profiler start -object-store-name <name> -node <name>

#### To viewing the results of the object store profiler using the ONTAP CLI, run the following command:
storage aggregate object-store profiler show

**PUT throttling**
#### To throttle FabricPool PUT operations by using the ONTAP CLI, run the following command:

storage aggregate object-store put-rate-limit modify -node <name> -default <true|false> -putrate-bytes-limit <integer>[KB|MB|GB|TB|PB]
