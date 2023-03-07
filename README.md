# Overview
**What is IBM Spectrum Fusion?**

IBM Spectrum Fusion is a container-native hybrid a cloud data platform that offers simplified deployment and data management for Kubernetes applications on Red Hat® OpenShift® Container Platform. 

IBM Spectrum Fusion provides a streamlined way for organizations to discover, secure, protect and manage data from the edge, to the core data center, to the public cloud.

IBM Spectrum Fusion comprises of Container Native Storge (IBM Spectrum Scale) and Backup & Restore software (IBM Spectrum Protect Plus).

IBM Spectrum Fusion is available as two deployments, namely IBM Spectrum Fusion and IBM Spectrum Fusion HCI.

**IBM Spectrum Fusion**

It is a software-defined storage management software with protection, replication, backup and caching elements and can be run on existing hardware resources. 

**IBM Spectrum Fusion HCI**

It is purpose-built, hyper-converged architecture designed to deploy bare metal Red Hat OpenShift container management and deployment software alongside IBM Spectrum Fusion software.

IBM Spectrum Fusion Operator uses IBM SPP & OADP - Storage Plug –In for backing up the container Application.

**What is OADP/Velero?**

OADP (OpenShift APIs for Data Protection) is an operator that Red Hat has created to create backup and restore APIs in the OpenShift cluster.

OADP provides the following APIs:
- Backup
- Restore
- Schedule
- BackupStorageLocation
- VolumeSnapshotLocation

The OADP operator will install Velero, and OpenShift plugins for Velero to use, for backup and restore operations.

OADP uses Velero backup controller for Kubernetes resources discovery & backup using the Kubernetes API and Velero CSI Plug-In the Persistent Volume backup.

The backup and restore configuration using OADP is quite complex compared to IBM Spectrum Fusion as it requires OpenShift development skills whereas IBM Spectrum simplifies the container backup and restore process using the user-friendly web portal.

In this tutorial, we will walk through the steps of setting up the container backup services using the IBM Spectrum Fusion 2.3 Operator for Red Hat® OpenShift® Kubernetes Service (ROKS) & OpenShift Data Foundation on IBM Cloud.

# Pre-Requisites:

- Obtain your [entitlement key](https://www.ibm.com/docs/en/spectrum-fusion/2.3?topic=prerequisites-obtaining-entitlement-key) to pull the required container images
- Create the image pull [secret](https://www.ibm.com/docs/en/spectrum-fusion/2.3?topic=prerequisites-creating-image-pull-secret) 
- IBM Cloud Object Storge  - [Create Instance](https://cloud.ibm.com/login?redirect=%2Fobjectstorage)
- Install [OpenShift CLI](https://docs.openshift.com/container-platform/4.10/cli_reference/openshift_cli/getting-started-cli.html) 
- The system requirements for installing IBM Spectrum Fusion and OpenShift data foundation on ROKS:

| Minimum Nodes/Hosts | CPU | RAM | Remarks |
| --------------- | --------------- | --------------- | --------------- |
| 3 Nodes (amd64: 64-bit Intel or AMD x86 hardware architecture) | 16 | 64GB | i.e., Flavor type: bx2.16x64 |

# High-Level Architecture Diagram:

![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/8abcf9f9ba15eacbfdd502d65ab46e448effc1a0/images/figure%201.png)

![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%202.png)

# Installation Steps:
1.  **Setup 3 Node ROKS Cluster**

    Logon to IBM Cloud Console – https://cloud.ibm.com

    Navigate to ROKS Clusters: https://cloud.ibm.com/kubernetes/clusters?platformType=openshift

    Click – **Create Cluster**

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%203.png)

    Select **Manual Setup** > **VPC** > Choose Your **VPC** from dropdown > Choose **Object Storage instance** from Drop Down

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%204.png)

    Select OpenShift Version **4.10.36**

    Select OCP Entitlement - **“Apply my Cloud Pack OCP Entitlement to this worker pool”**x if you have Cloud Pack OCP entitlement license

    Select Your **Resource Group**

    Select **Worker Zones (One Zone is sufficient for POC & test environment)**

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%205.png)

    Click on **Change flavor** from **Worker pool** and select **16 vCPU 64GB RAM** instance **(bx2.16x64)**, 
    **RHEL 8** Operating System

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%206.png)

    Type meaningful **Cluster name** and **Tag**

    click **Create**

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%207.png)

2.  **Install OpenShift Data Foundation on ROKS Cluster**
   
    Navigate to ROKS Clusters: https://cloud.ibm.com/kubernetes/clusters?platformType=openshift

    Open ROKS Cluster that you had created in previous steps
    Click **Install** button from **OpenShift Data Foundation** . This will create the ODF

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%208.png)

    Enter total disk storage requirement size in **“OSD pod size (Gi)”** and click **Install**

    For an example, if you need 1TB usable disk storage, enter 1024Gb.

    This will provision the additional disks to OCP cluster nodes in IBM Cloud and that will be converted in to ODF Storage pool which can be consumed by the POD/Container by Persistent Volume claim.

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%209.png)

    Verify OpenShift Data Foundation has installed successfully by login into **OpenShift Console** > Navigate to **Storage** > **Data Foundation** should visible underneath now & its status should be Green

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2010.png)

    Verify OCS storge class are created

    - ocs-storagecluster-cephfs
    - ocs-storagecluster-ceph-rbd
    - ocs-storagecluster-ceph-rgw
    - openshift-storage.noobaa.io

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2011.png)

3.  **Set ODF Storage Class to default**

    ODF StorageClass is required to set default for deploying the container application with dynamic persistent volume on ODF storage:

    In order to make ODF StorageClass to default , Navigate to **OpenShift Console** > Go to **StorageClasses** > Click on StorageClass **“ibm-vpc-block-10iops-tier”** which would be showing default

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2012.png)

    Click Pencil icon underneath **Annotations** > change the value to False for the  key **“storageclass.kubernetes.io/is-default-class”** > **Save**

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2013.png)

    Go back to **StorgeClasses** > Click on ODF StorageClass **“ocs-storagecluster-ceph-rbd”** from the list 

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2014.png)

    Click + Add More > Enter the key name **“storageclass.kubernetes.io/is-default-class”** and value **“true”** > **Save**

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2015.png)

4.  **Enable IBM Operator catalog on OCP if already not available**

    Connect to OCP cluster remotely from any of the local server via oc login. Login command can be found on OCP console right corner: 

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2016.png)

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2017.png)

    To run oc command locally, you need to have oc client installed on local server. Preferably use Linux server to run these commands.

    Once login is done , create the catalog_source.yaml  as described in link below and run oc apply command.

    https://github.com/IBM/cloud-pak/blob/master/reference/operator-catalog-enablement.md#command-line-enablement

    **The catalog can be enabled by applying the following YAML file to the OpenShift cluster:**

            apiVersion: operators.coreos.com/v1alpha1
            kind: CatalogSource
            metadata:
            name: ibm-operator-catalog
            namespace: openshift-marketplace
            annotations:
            olm.catalogImageTemplate: "icr.io/cpopen/ibm-operator-catalog:v{kube_major_version}.{kube_minor_version}"
            spec:
            displayName: IBM Operator Catalog
            publisher: IBM
            sourceType: grpc
            image: icr.io/cpopen/ibm-operator-catalog:latest
            updateStrategy:
                registryPoll:
                interval: 45m

    Once login is done , create the catalog_source.yaml  as described in link and run oc apply command.

            oc apply -f catalog_source.yaml -n openshift-marketplace

    On success you should get response for command oc get like :

            $ oc get CatalogSources ibm-operator-catalog -n openshift-marketplace
            NAME                   DISPLAY                TYPE   PUBLISHER   AGE
            ibm-operator-catalog   IBM Operator Catalog   grpc   IBM         28s
5.  **Obtain a production entitlement key for Spectrum fusion production images:** 
   
    Get a production entitlement key from: 
    https://myibm.ibm.com/products-services/containerlibrary 

    Detailed steps are mentioned here: 

    https://github.ibm.com/alchemy-registry/image-iam/blob/master/obtaining_entitlement.md#obtaining-a-production-entitlement-key

    Generated key looks like: 

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2018.png)

    Copy this key as we will be using this key as a password with the username cp to login to the CloudPak registry cp.icr.io

6.  **Update OCP’s global pull-secret with Production entitlement key:**
   
    Run following commands from the server where you are remotely connected to OCP cluster. Make sure you have jq library installed on the server. 

    First command will return base64 format of the username/password (entitlement key)

                echo -n "cp:REPLACE_WITH_GENERATED_ENTITLEMENT_KEY" | base64 -w0
    
    i.  Create auth.json which will have base64 encoded string of entitlement key :

                {
                "auth": "REPLACE_WITH_BASE64_ENCODED_ENTITLEMENT_KEY_FROM_PREVIOUS_COMMAND",
                "username":"cp",
                "password":"REPLACE_WITH_GENERATED_ENTITLEMENT_KEY"  
                }
  
    ii.  Run following command to create temp_config.json  
    
                oc get secret/pull-secret -n openshift-config -ojson | \
                jq -r '.data[".dockerconfigjson"]' | \
                base64 -d - | \
                jq '.[]."cp.icr.io" += input' - auth.json > temp_config.json

    iii.  Append and set the secret with new config

                oc set data secret/pull-secret -n openshift-config --from-file=.dockerconfigjson=temp_config.json

    iv.  Verify the secret with oc get secret:

                oc get secret/pull-secret -n openshift-config -ojson | \
                jq -r '.data[".dockerconfigjson"]' | \
                base64 -d -


7.  **OCP Node resize from IBM Cloud console:**

    The pull-secret updated on OCP cluster with new entitlement keys, those changes does not get reflect on all the nodes. To do that resize the worker pool size.

    i.      In IBM Cloud console, navigate to OpenShift cluster and open cluster details page.
    
    ii.     Under Worker pools section you will find Resize option in overflow menu.

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2019.png)

    iii.    Increase the worker pool size by 1 and wait until all the nodes are in ready state. 

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2020.png)

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2021.png)

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2022.png)

    iv.		Decrease by 1 and wait for all the nodes to be in ready state.

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2023.png)

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2024.png)

    v.      Repeat the steps iii (Increase the storage pool size by 1) & iv (decrease the storage pool by 1) from IBM Cloud console until all previously provisioned nodes are replaced by new ones.  
    
    vi.     Verify pull secrete is updated on all nodes by running the following command 

            $ for node in `oc get no |awk -F " " '/Ready/ {print $1}'`; do oc debug node/$node -- chroot /host cat /var/lib/kubelet/config.json;done

        Note: Steps 5 and 6 are needed because IBM Cloud on OpenShift requires manual trigger of resize and reload of nodes. 
8.  **Install IBM Spectrum Fusion 2.3.0 Operator from OperatorHub**

    Navigate to **Operators** from OpenShift Console > Click on **OperatorHub** > Search for **IBM Spectrum Fusion** under All Items

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2026.png)

    Click on IBM Spectrum Fusion Operator catalog > Click **Install**

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2027.png)

    Click **Install** from the pop-up window 

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2028.png)

    Verify IBM Spectrum Fusion Operator is installed successfully.

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2029.png)

9.  **Create SpectrumFusion CR**

    Create an instance of "SpectrumFusion" CRD, with the following details,
    -   Name - Give a name to the SpectrumFusion, for example "spectrumfusion"
    -   Labels - optional field
    -   **License** – Accept the license
    -   **Data protection** - Enable **“Data Protection”**
    -   **Storage Class** – there are some storage classes come with default installation of OCP on IBM Cloud. Select           **“ocs-storagecluster-ceph-rbd”** storage class.

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2030.png)

    This will install below four Operators
    -   IBM Spectrum Protect Plus Operator
    -   IBM Spectrum Protect Plus Container Backup Support
    -   OADP Operator
    -	Red Hat Integration – AMQ Streams

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2031.png)

10. **Fusion UI console config changes:**

    **IBM Spectrum Fusion 2.3 is having a bug with ROKS. After installation, OAuthClient “isf-oauth” doesn’t get created with redirectURIs.** This bug will be resolved in upcoming version 2.4.

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2032.png)

    As a workaround for version 2.3, Navigate to **Deployments** > Switch the name space to **ibm-spectrum-fusion-ns** > Search **ibm-ui-dep**

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2033.png)

    Click on deployment “**ibm-ui-dep**” > Go to **Environment** tab > Copy the value from **REDIRECTURL**

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2034.png)

    Update the REDIRECRURL in OAuthClient “**isf-oauth**”.
    Navigate to Home > API Explorer > Search **OauthClient**

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2035.png)

    Open **OAuthClient** > Go to **Instances** Tab > Open **isf-oauth** > Click **YAML** Tab > Update the **redirectURIs** value that you have copied previously > Click **SAVE**

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2036.png)

    Access the **IBM Spectrum Fusion** URL by Clicking on Application icon in the title bar of OpenShift console

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2037.png)

    This will launch the console of IBM Spectrum Fusion application but will get error message “**Application not available**”. This is a bug of version 2.3 and will be fixed in version 2.4

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2038.png)

    As a workaround, remove the text “**. apps**” from URL and hit press
    For an example,
    **Not working URL**: https://console-ibm-spectrum-fusion-ns.apps.spfcl03-2bef1f4b4097001da9502000c44fc2b2-0000.us-south.containers.appdomain.cloud
    **Working URL**: https://console-ibm-spectrum-fusion-ns.spfcl03-2bef1f4b4097001da9502000c44fc2b2-0000.us-south.containers.appdomain.cloud

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2039.png)

#   Container Application Backup Steps

1.  **Deploy Sample application in OpenShift and verify it creates the persistent volume in ODF storage**

    Navigate to OpenShift Console > In **Administrator** view > **Projects** > **Create Project**

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2040.png)

    Enter **Name** “demoapps” and click **Create**

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2041.png)

    Switch to **Developer view** from OpenShift console > change the project to “**demoapps**” > Click **+Add**  

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2042.png) 
  
    Click **All services** from Developer Catalog > Search for catalog “**Node.js + PostgreSQL**” templates

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2043.png)

    Click catalog “**Node.js + PostgreSQL**” templates > Click **Instantiate Template** > Verify namespace is “**demoapps**” > Click **Create**

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2044.png)

    Wait until pods are showing in running state

    Switch back to **Administrator view** from Console > Verify **nodejs-postgresql-persistent-XXX** & **postgesqlXX** pods are in running state

    Pods are in running state means it has created with PerstistentVolume successfully using ODF default StorageClass. 

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2045.png)

    In order to verify persistent volume created, Navigate to **Storage** > **PersistentVolumeClaims** > Verify “**postgresql**” is in **bound** state and its StorgeClass is “**ocs-storagecluster-ceph-rbd**”which is a defaut StorageClass that you setup in previous steps.

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2046.png)

2.	**Add Backup Storge Location**

    To save the backup copy to IBM Cloud Object Storage, Add IBM Cloud Object Storage as a backup location in IBM Spectrum Fusion

    -  Create a Bucket in IBM Cloud Object Storage

        Login to **IBM Cloud** > Navigate to **Resource list** > Find the **Cloud Object Storage instance** that you have created before under the **Storage** > Access the **IBM Cloud Object Storage Instance** > Click **Buckets** > Create **Bucket +**

        ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2047.png)

        Click **Customize your bucket** > Enter **unique bucket name** > Select **Resiliency** > Select **Location** > Select **Storage Class** > Click **Create bucket**

        ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2048.png)

    -  Set permission on Bucket 

        Click **Bucket Name > Permissions Tab** > Select **Service ID** from Access policies > Select Role **“Writer”** and Click **Create access policy**

        ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2049.png)

    -  Copy Endpoint name

        Bucket End point name is required to connect to Object Storage

        Click **Configuration** Tab > Click **Endpoints**

        ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2050.png)

        Scroll down and **copy** the **Direct Endpoint** name from **Endpoints**

        ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2051.png)

    -  Copy Access Key and Secrete Key from Service Credential

        To authenticate with IBM Cloud Object storage, Access Key and Secrete key of Service credential are required

        Navigate to **IBM Cloud Object Storage** > Click **Service Credentials** > **Expand Service Credential** > Copy Access key from “**access_key_id**” and Secret key from “**secrete_access_key**”

    -  Add backup location

        Login to **IBM Spectrum Console** > Navigate to **Backup Policies** > Click **Locations** Tab > Click **Add Location +**

        ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2052.png)

        Enter **Name** i.e., “ibmcos” > Select a type of storage backup location “**IBM Cloud (IBM. Object Storage)**” > Enter Endpoint **Name** > Enter **Bucket Name** > Enter **Access Key and Secret Key**

        ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2053.png)

        Verify the storage location is added and showing as connected

        ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2054.png)

3.  **Create Backup Policy**

    Backup policies define parameters that are applied to backup jobs. These parameters specify the storage location, frequency of backup jobs, the retention period for backed-up data, and other directives for backup operations.

	Navigate to **Backup Policies > Policies Tab > Click Add Policy +**

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2055.png)

    Enter **Name > Set the frequency** in which the associated backup jobs must run > Select Object Storage from Backup Locations 

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2056.png)

    **Search** and then select the **backup location** > Define **Retention period**  and click **Create policy**

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2057.png)

    Validate Backup Policy is created

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2058.png)

4.  **Assign Backup Policy to container applications**

    Container application runs under the namespace (Or project) in OpenShift. You can’t assign the backup policy to individual resources within the namespace, so you must assign the backup policy to namespace (Or Project) instead.

    To assign the backup policy, Navigate to **Application Page** > Click the **Assign policy** for the namespace (Or Project) where application is running in 

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2059.png)

    Choose Backup policy and Click **Assign Policy**

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2060.png)

    This will immediately trigger the one-time backup of container application and subsequent backup will trigger based on the schedule defined in the back up policy.

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2061.png)

    Click on application namespace > Go to **Backups Tab**> Monitor the backup job until it completes

5.  **Run On-Demand Backup**

    You can run the On-Demand backup by going into running the “**Backup now**” option from **Action**

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2062.png)

6.  **Application Consistent Backup**

    IBM Spectrum Fusion backs up application data consistently across all the Persistent Volumes (PVs). It ensures that the recovered data is error-free, reliable, and usable.

    In Application Consistency approach, application I/O must be paused before a snapshot is taken by using hooks. Though it achieves the highest level of consistency, it comes at the cost of application downtime that can be disruptive. The pause can be a long time depending on the application. For instructions about specifying these application hooks, see [Backup Hooks](https://www.ibm.com/docs/en/spp/10.1.12?topic=data-backing-up-application-consistent).

    -  Backup Hooks

       -   When performing a backup, you can specify one or more commands to execute in a container in a pod when that pod  is being backed up. The commands can be configured to run before any custom action processing (“pre” hooks), or after all custom actions have been completed and any additional items specified by custom action have been backed up (“post” hooks). Note that hooks are not executed within a shell on the containers.

            The backup Hooks can be specified via annotations on the pod itself.

    -   Annotations

        -   You can use the following annotations on a pod to make Velero execute a hook when backing up the pod

            **Pre hooks**

            -   pre.hook.backup.velero.io/container    

                -   The container where the command should be executed. Defaults to the first container in the pod. Optional.

            -   pre.hook.backup.velero.io/command

                -   The command to execute. If you need multiple arguments, specify the command as a JSON array, such as ["/usr/bin/uname", "-a"]

            -   pre.hook.backup.velero.io/on-error

                -   What to do if the command returns a non-zero exit code. Defaults to Fail. Valid values are Fail and Continue. Optional.
  
            -   pre.hook.backup.velero.io/timeout

                -   How long to wait for the command to execute. The hook is considered in error if the command exceeds the timeout. Defaults to 30s. Optional.

            **Post Hooks**

            -	post.hook.backup.velero.io/container

                - 	The container where the command should be executed. Defaults to the first container in the pod. Optional.

            -   post.hook.backup.velero.io/command

                -	The command to execute. If you need multiple arguments, specify the command as a JSON array, such as ["/usr/bin/uname", "-a"]

            -	post.hook.backup.velero.io/on-error

                -	What to do if the command returns a non-zero exit code. Defaults to Fail. Valid values are Fail and Continue. Optional.

            -	post.hook.backup.velero.io/timeout

                -	How long to wait for the command to execute. The hook is considered in error if the command exceeds the timeout. **Defaults to 30s**. Optional.

    	    Apply the pre and post backup hooks annotations directly to declarative deployment.   Below is an example of what updating an object in place might look like.

            ```
            oc annotate pod -n demoapps -l name=postgresql \
            pre.hook.backup.velero.io/command='["/bin/bash", "-c", 'psql  -c "\"select pg_start_backup('app_cons');\""']' \ pre.hook.backup.velero.io/container=postgresql \
            post.hook.backup.velero.io/command='["/bin/bash", "-c", 'psql -c "\"select pg_stop_backup( );\"""]'\ 
            post.hook.backup.velero.io/container=postgresql

            ```


 

#   Restore steps

There are below important points to consider while restoring the backup

-	Backups that are taken using an Object Storage can be restored to same project as well as new or different project whereas Inplace snapshot can be restored only to same project, not to any new or different project.

-	If you try to restore a project that is deleted from OpenShift®, then the restore to the same project with Inplace snapshot does not include PVCs.  

Restoring Backup from Object Storage:

In the menu, click **Applications** > **Search and Open** the application that you want to restore > Click **Restore**

![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2063.png)

In the **Restore type** section, you can choose to restore to a different project or restore to same project

1.	Restore Application to New or Different Project

    If you select Restore to a different project, Select  New Project from OpenShift Project section > Enter Name of the project >  Select a successful backup from the list and then Click Restore

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2064.png)

2.	Restore Application to same Project

    If you want to  Restore application’s missing  resource and PVC > Select  Restore PVC to the same project>  Select a successful backup from the list > Select PVC from PVCs  and then Click Restore

    ![alt text](https://github.com/mdhamat/IBM-Spectrum-fusion/blob/00b9b706b92cfa8fa564eb4b4ca824ad713d9987/images/Figure%2065.png)


#   Conclusion

This tutorial walked you through the steps of setting up 3 node ROKS cluster, Installation of OpenShift Data Foundation Software-Defined-Storage for ROKS, Installation and configuration of IBM Spectrum Fusion Operator in OpenShift cluster.  This tutorial also showed you how to backup the container application to IBM Cloud Object Storage and its restore steps.
    
        

        














   


  

