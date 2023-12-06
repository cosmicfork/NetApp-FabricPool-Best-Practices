How to set up FabricPool for use with existing ONTAP system

Cloud Tiering:
--------------

1.  First, you need to confirm your cloud tiering license/third-party certificate. FabricPool requires a capacity-based license when attaching third-party object storage providers.

2.  These certificates should be installed into ONTAP before attaching them for use with local tiers.

3.  NOTE - Beginning with ONTAP 9.4, CA Certificates are not required. Regardless, using signed certificates from a third-party authority remains the best practice. 

4.  Be aware of the different versions you are using because some might require specific permissions

5.  FDQN - FabricPool stipulates that CA Certificates must have a fully qualified domain name as the cloud tier server with which they are connected

6.  In releases prior to StorageGRID 11.3, the CA certificates use a common name that is not based on the FQDN. This might cause errors that prohibit StorageGRID from being attached to ONTAP local tiers.

7.  Potential Errors - "Unable to add a cloud tier. Cannot verify the certificated provided by the object store server. The certificates might not be installed on the cluster."

8.  In order to avoid these potential errors and successfully attach StorageGRID 11.2 or earlier releases as a cloud tier, you have to replace the certificates in the grid with certificates that use an accurate FQDN.

Installation:
-------------

-   Installation requires CA certificates. To install, first retrieve the CA Certificates and then install the certificates into ONTAP.

### How to Retrieve CA Certificates:

1.  Open the StorageGRID admin console > Select Configuration > Load Balancer Endpoints > Select your endpoint and click edit endpoint > copy the certificate PEM:

2.  ![Image1](https://lh7-us.googleusercontent.com/mXjAaiU1p3C_mUfvU-cEejgq_UnfTZfGzxk4_OO_zphh-XYNw2E-QpBvI9TA-lvjN5LRIRdx6x5RjaApu4C7ba412sXgnUT5if2Legj54dbrh75-RnCwHB0K76tZ3cx-GkP1_tGLtGq3Q2UkYqYR8FA)

3.  Retrieving the certificate when using a third-party load balancer:

4.  ![Image2](https://lh7-us.googleusercontent.com/Wt39-P2sa_jZFfM2-vK2gRlqe-9oIZsouDWCOCEcGiRcA612W6j68NMDt5dVZO4xlxoOU1akuS7SiSOFXDMrNsYoXoyUWfRkWlYtrxNtpplDdL7k4uB8UvRg18HaWxXHB9kJo7UA4ezZUyVS5BkWL58)

5.  ![](https://lh7-us.googleusercontent.com/rgcapwufmYWELNdktb-XX4Cwk98z8425UIITeR63Y9OMDwjdZ2ulS19UDofC3I-8JoKQ8aUAYtDLctfuTGMsxhbT9slchzV_eBnKTrFzfYEcoWTN8DxzMIkNBxBu8wpfaOWh-xmTKWKHqNEwJv-Hteg)

### How to Install Certificates to ONTAP:

1.  To install the root certificates to ONTAP, run the following command:

      ![Image3](https://lh7-us.googleusercontent.com/4kysufy5hKmrCjNxpBoFrOlPkqc8SqfS_jVmsAJEKgvhVdx6QFwpybQHUHkrM2_1CFneH7y6zTAXOfSQq-VZ6qcl8fagAFSWvy6R-ZWU2X5aHfl7vmLy871nSieesywwECzFbs2bcSw-KJ481l5Xtho)