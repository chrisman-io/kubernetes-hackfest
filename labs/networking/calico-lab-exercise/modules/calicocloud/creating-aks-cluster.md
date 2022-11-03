# Module 0: Creating AKS cluster

---

**Goal:** Create AKS cluster.

> This workshop uses AKS cluster with Linux containers. To create a Windows Server container on an AKS cluster, consider exploring [AKS documents](https://docs.microsoft.com/en-us/azure/aks/windows-container-cli). This cluster deployment utilizes Azure CLI v2.x from your local terminal or via Azure Cloud Shell. Instructions for installing Azure CLI can be found [here](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli).

If you already have AKS cluster created make sure to verify it by running the Instructions number 12 and 13 below. If it comes out ok, then you can skip this module and go to [module 1](/modules/joining-aks-to-calico-cloud.md)

## Prerequisite Tasks

- Azure Account

## Instructions

1. Login to Azure Portal at http://portal.azure.com.

2. Open the Azure Cloud Shell and choose Bash Shell (do not choose Powershell)

   ![img-cloud-shell](https://user-images.githubusercontent.com/104035488/199605731-86d9f1c5-d3a6-40fb-8e95-3a9bf837c84b.png)

3. The first time Cloud Shell is started will require you to create a storage account.

4. Once your cloud shell is started, clone the workshop repo into the cloud shell environment

   ```bash
   git clone https://github.com/Azure/kubernetes-hackfest
   ```

   > Note: In the cloud shell, you are automatically logged into your Azure subscription.

5. Ensure you are using the correct Azure subscription you want to deploy AKS to.
    
	```bash
	# View subscriptions
	az account list --out table
 
    # Verify selected subscription
    az account show --out table
    ```
    
    ```bash
    # Set correct subscription (if needed)
    az account set --subscription <subscription_id>
  
    # Verify correct subscription is now set
    az account show --out table
    ```

6.  Create a unique identifier suffix for resources to be created in this lab.
    
    > *NOTE:* In the following sections we'll be generating and setting some environment variables. If you're terminal session restarts you may need to reset these variables. You can use that via the following command:
    >
    >source ~/workshopvars-calicloud.env

	```bash
    echo "# Start AKS Hackfest Calico OSS Lab Params" >> ~/workshopvars-calicloud.env
    UNIQUE_SUFFIX=$USER$RANDOM
    # Remove Underscores and Dashes (Not Allowed in AKS and ACR Names)
    UNIQUE_SUFFIX="${UNIQUE_SUFFIX//_}"
    UNIQUE_SUFFIX="${UNIQUE_SUFFIX//-}"
    # Check Unique Suffix Value (Should be No Underscores or Dashes)
    echo $UNIQUE_SUFFIX
    # Persist for Later Sessions in Case of Timeout
    echo export UNIQUE_SUFFIX=$UNIQUE_SUFFIX >> ~/workshopvars-calicloud.env
	```

7. Create an Azure Resource Group in your chosen region. We will use East US in this example.

   ```bash
   # Set Resource Group Name using the unique suffix
   RGNAME=aks-rg-$UNIQUE_SUFFIX
   # Persist for Later Sessions in Case of Timeout
   echo export RGNAME=$RGNAME >> ~/workshopvars-calicloud.env
   # Set Region (Location)
   LOCATION=eastus
   # Persist for Later Sessions in Case of Timeout
   echo export LOCATION=eastus >> ~/workshopvars-calicloud.env
   # Create Resource Group
   az group create -n $RGNAME -l $LOCATION --out table
   ```
    
8.  Create your AKS cluster in the resource group created in step 2 with 3 nodes. We will check for a recent version of kubnernetes before proceeding. You will use the Service Principal information from the prerequisite tasks.
    
    Use Unique CLUSTERNAME
    
    ```bash
    # Set AKS Cluster Name
    CLUSTERNAME=aks-${UNIQUE_SUFFIX}
    # Look at AKS Cluster Name for Future Reference
    echo $CLUSTERNAME
    # Persist for Later Sessions in Case of Timeout
    echo export CLUSTERNAME=aks-${UNIQUE_SUFFIX} >> ~/workshopvars-calicloud.env
    ```
    
    Get available kubernetes versions for the region. You will likely see more recent versions in your lab.
    
    ```bash
    az aks get-versions -l $LOCATION --output table
    ```
    
    Output is:
    ```bash
    KubernetesVersion    Upgrades
    -------------------  ------------------------
    1.24.6               None available
    1.24.3               1.24.6
    1.23.12              1.24.3, 1.24.6
    1.23.8               1.23.12, 1.24.3, 1.24.6
    1.22.15              1.23.8, 1.23.12
    1.22.11              1.22.15, 1.23.8, 1.23.12
    ```
    
    For this lab we will use 1.23.12 kubernetes version.
    
    ```bash
    K8SVERSION=1.23.12
    echo export K8SVERSION=1.23.12 >> ~/workshopvars-calicloud.env
    ```
    
    > The below command can take 10-20 minutes to run as it is creating the AKS cluster. Please be PATIENT and grab a coffee/tea/kombucha...
    
    ```bash
    # Create AKS Cluster - it is important to set the network-plugin as azure in order to connec to Calico Cloud
    az aks create -n $CLUSTERNAME -g $RGNAME \
    --kubernetes-version $K8SVERSION \
    --enable-managed-identity \
    --node-count 3 \
    --network-plugin azure \
    --no-wait \
    --generate-ssh-keys
    ```
    
9.  Verify your cluster status. The `ProvisioningState` should be `Succeeded`
    
    ```bash
    watch az aks list -o table -g $RGNAME --out table
    ```

    Wait until the `ProvisioningState` value becames `Succeeded`:
    ```bash
    Name           Location    ResourceGroup      KubernetesVersion    ProvisioningState    Fqdn
    -------------  ----------  -----------------  -------------------  -------------------  -----------------------------------------------------------------
    aksjessie2081  eastus      aks-rg-jessie2081  1.22.4               Succeeded             aksjessie2-aks-rg-jessie208-03cfb8-9713ae4f.hcp.eastus.azmk8s.io
    
    ```
    
    
10.  Get the Kubernetes config files for your new AKS cluster
    
    ```bash
    az aks get-credentials -n $CLUSTERNAME -g $RGNAME
    ```
    
6.  Verify you have API access to your new AKS cluster
    
    > Note: It can take 5 minutes for your nodes to appear and be in READY state. You can run `watch kubectl get nodes` to monitor status. Otherwise, the cluster is ready when the output is similar to the following:
    
	```bash
	kubectl get nodes
	```
	```
	NAME                                STATUS   ROLES   AGE    VERSION
	aks-nodepool1-29374799-vmss000000   Ready    agent   118s   v1.22.4
	aks-nodepool1-29374799-vmss000001   Ready    agent   2m3s   v1.22.4
	aks-nodepool1-29374799-vmss000002   Ready    agent   2m     v1.22.4
	```

	To see more details about your cluster:
	```bash
	kubectl cluster-info
	```
	
7. *[Optional]*  Install `calicoctl` CLI for use in later labs

    a) CloudShell

    ```bash
    # download and configure calicoctl
    curl -o calicoctl -O -L https://downloads.tigera.io/ee/binaries/v3.14.2/calicoctl

    chmod +x calicoctl
    
    # verify calicoctl is running 
    ./calicoctl version
    ```

    b) Linux
    >Tip: Consider navigating to a location that’s in your PATH. For example, /usr/local/bin/

    ```bash
    # download and configure calicoctl
    curl -o calicoctl -O -L https://downloads.tigera.io/ee/binaries/v3.14.2/calicoctl
    chmod +x calicoctl
    
    # verify calicoctl is running 
    calicoctl version
    ```

    c) MacOS
    >Tip: Consider navigating to a location that’s in your PATH. For example, /usr/local/bin/

    ```bash
    # download and configure calicoctl
    curl -o calicoctl -O -L  https://downloads.tigera.io/ee/binaries/v3.14.2/calicoctl-darwin-amd64

    chmod +x calicoctl
    
    # verify calicoctl is running 
    calicoctl version
    ```

    Note: If you are faced with `cannot be opened because the developer cannot be verified` error when using `calicoctl` for the first time. go to `Applicaitons` \> `System Prefences` \> `Security & Privacy` in the `General` tab at the bottom of the window click `Allow anyway`.  
    Note: If the location of calicoctl is not already in your PATH, move the file to one that is or add its location to your PATH. This will allow you to invoke it without having to prepend its location.

    d) Windows - using powershell command to download the calicoctl binary  
    >Tip: Consider runing powershell as administraor and navigating to a location that’s in your PATH. For example, C:\Windows.

    ```pwsh
    Invoke-WebRequest -Uri "https://downloads.tigera.io/ee/binaries/v3.14.2/calicoctl-windows-amd64.exe" -OutFile "calicocttl.exe"
    ```


--- 
## Next steps

You should now have a Kubernetes cluster running with 3 nodes. You do not see the master servers for the cluster because these are managed by Microsoft. The Control Plane services which manage the Kubernetes cluster such as scheduling, API access, configuration data store and object controllers are all provided as services to the nodes.
<br>    

    
[Next -> Module 1](../calicocloud/joining-aks-to-calico-cloud.md)
