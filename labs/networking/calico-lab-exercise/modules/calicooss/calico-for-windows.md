# Module 3: Adding windows node in your AKS and protect them with Calico policy
>Calico for Windows is a hybrid implementation that requires a Linux cluster for Calico components and Linux workloads, and Windows nodes for Windows workloads.


**Goal:** Create client and server pods on Linux and Windows nodes, verify connectivity between the pods, and then isolate pod traffic.

**Docs:** https://docs.projectcalico.org/getting-started/windows-calico/quickstart

## Steps

1. Enable AKS windows Calico feature and register the service in your cluster. **Note:** This may take several minutes to complete, so it could be a good time for a coffee or lunch break.

    
    ```bash
    az feature register --namespace "Microsoft.ContainerService" --name "EnableAKSWindowsCalico"
    ```

    Output will be like this:
    ```text
    {
      "id": "/subscriptions/03cfb895-358d-4ad4-8aba-aeede8dbfc30/providers/Microsoft.Features/providers/Microsoft.ContainerService/features/EnableAKSWindowsCalico",
      "name": "Microsoft.ContainerService/EnableAKSWindowsCalico",
      "properties": {
        "state": "Registered"
      },
      "type": "Microsoft.Features/providers/features"
    }
    ```


   Verify that the feature is registered. 

    ```bash
    az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/EnableAKSWindowsCalico')].{Name:name,State:properties.state}"
    ```
    
    Output will be
    ```bash
    Name                                               State
    -------------------------------------------------  ----------
    Microsoft.ContainerService/EnableAKSWindowsCalico  Registered
    ```


2. Refresh the registration of the Microsoft.ContainerService resource provider.

   ```bash
   az provider register --namespace Microsoft.ContainerService
   ```


3. Add a Windows node in your pool, and confirm the result.
   ```bash
   az aks nodepool add \
   --resource-group $RGNAME \
   --cluster-name $OSSCLUSTERNAME \
   --os-type Windows \
   --name npwin  \
   --node-count 1 \
   --kubernetes-version $K8SVERSION \
   --node-vm-size Standard_D2s_v3
   ```
   
   ```bash
   kubectl get nodes
   ```

   ```bash
   ### The output is like:
   NAME                                STATUS   ROLES   AGE   VERSION
   aks-nodepool1-17497551-vmss000000   Ready    agent   80m   v1.23.12
   aks-nodepool1-17497551-vmss000001   Ready    agent   80m   v1.23.12
   aks-nodepool1-17497551-vmss000002   Ready    agent   80m   v1.23.12
   aksnpwin000000                      Ready    agent   25s   v1.23.12
   ```

4. Create demo pods in Linux and Windows nodes. Expected Outcome:
   - Create a client (busybox) and server (nginx) pod on the Linux nodes:
   - Create a client pod (powershell) and a server (porter) pod on the Windows nodes

    ```bash
    kubectl apply -f demo/win-demo/
    ```
    
    ```bash
    # Windows images are large and can take some time to start, run a watch and wait for the pods to be in running state
    watch kubectl get pods -n calico-demo
    ```

    ```bash
    ### The output is like when ready:
    NAME      READY   STATUS    RESTARTS   AGE
    busybox   1/1     Running   0          58s
    nginx     1/1     Running   0          58s
    porter    1/1     Running   0          58s
    pwsh      1/1     Running   0          58s
    ```

5. Check the connectivities between pods. Expected Outcome:  

   - The traffic between `busybox` in Linux node and `porter` in Windows node is allowed. 
   - The traffic between `powershell` in Windows node and `nginx` in Linux node is allowed. 
   - The traffic between `busybox` in Linux node and `nginx` in Linux node is allowed. 
   - The traffic between `powershell` in Windows node and `porter` in Windows node is allowed. 

   ```bash
   kubectl exec -n calico-demo busybox -- nc -vz $(kubectl get po porter -n calico-demo -o 'jsonpath={.status.podIP}') 80
   kubectl exec -n calico-demo pwsh -- powershell Invoke-WebRequest -Uri http://$(kubectl get po nginx -n calico-demo -o 'jsonpath={.status.podIP}') -UseBasicParsing -TimeoutSec 5 | grep Status
   kubectl exec -n calico-demo busybox -- nc -vz $(kubectl get po nginx -n calico-demo -o 'jsonpath={.status.podIP}') 80
   kubectl exec -n calico-demo pwsh -- powershell Invoke-WebRequest -Uri http://$(kubectl get po porter -n calico-demo -o 'jsonpath={.status.podIP}') -UseBasicParsing -TimeoutSec 5 | grep Status
   ```
  
   The output will be like:
   ```bash
   10.224.0.121 (10.224.0.121:80) open
   StatusCode        : 200
   StatusDescription : OK
   10.224.0.65 (10.224.0.65:80) open
   StatusCode        : 200
   StatusDescription : OK
   ```

6. Create policy to explicitly allow the `busybox` pod in Linux node to reach the `porter` pod in Windows node, and deny the `powershell` pod in Windows node to reach the `nginx` pod in Linux node
   ```bash
   calicoctl --allow-version-mismatch apply -f demo/20-egress-access-controls/allow-busybox.yaml
   calicoctl --allow-version-mismatch apply -f demo/20-egress-access-controls/deny-nginx.yaml
   ```

7. Check the connectivities between pods. Expected Outcome:  
   - The traffic between `busybox` in Linux node and `porter` in Windows node is allowed. 
   - The traffic between `powershell` in Windows node and `nginx` in Linux node is denied. 

      ![demo-diagram](../img/windows-demo.png)

   ```bash
   kubectl exec -n calico-demo busybox -- nc -vz $(kubectl get po porter -n calico-demo -o 'jsonpath={.status.podIP}') 80

   kubectl exec -n calico-demo pwsh -- powershell Invoke-WebRequest -Uri http://$(kubectl get po nginx -n calico-demo -o 'jsonpath={.status.podIP}') -UseBasicParsing -TimeoutSec 5
   ```
   
   The output will be like:
   ```bash
   ##nc command output
   10.224.0.121 (10.224.0.121:80) open
 
   ##powershell command output 
   Invoke-WebRequest : The operation has timed out.
   At line:1 char:1
   + Invoke-WebRequest -Uri http://10.224.0.65 -UseBasicParsing -TimeoutSe ...
   + ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
       + CategoryInfo          : InvalidOperation: (System.Net.HttpWebRequest:Htt 
      pWebRequest) [Invoke-WebRequest], WebException
       + FullyQualifiedErrorId : WebCmdletWebResponseException,Microsoft.PowerShe 
      ll.Commands.InvokeWebRequestCommand
    
   command terminated with exit code 1


[Next -> Module 4](../calicooss/wireguard-encryption.md)
