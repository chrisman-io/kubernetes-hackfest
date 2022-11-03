# Workshop cleanup

1. Delete AKS cluster.

    ```bash
    az aks delete --name $OSSCLUSTERNAME --resource-group $RGNAME
    ```

2. Delete the azure resource group. 

    ```bash
    az group delete --resource-group $RGNAME
    ```
