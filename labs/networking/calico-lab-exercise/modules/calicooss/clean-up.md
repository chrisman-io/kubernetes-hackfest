# Chapter one: Calico OSS cleanup

1. Delete AKS cluster.

    ```bash
    az aks delete --name $OSSCLUSTERNAME --resource-group $RGNAME
    ```

2. Delete the azure resource group. 

    ```bash
    az group delete --resource-group $RGNAME
    ```

3. Delete the workshopvars-calioss.env file.

   ```bash
   rm workshopvars-calioss.env
   ```

[Next -> Chapter 2 - Module 0](../calicocloud/creating-aks-cluster.md)