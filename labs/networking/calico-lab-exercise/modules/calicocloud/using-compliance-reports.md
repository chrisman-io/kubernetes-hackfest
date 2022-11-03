# Module 8: Using compliance reports

**Goal:** Use global reports to satisfy compliance requirementes.

## Steps

1. Use `Compliance Reports` view to see all generated reports.

    >We have deployed a few compliance reports in one of the first labs and by this time a few reports should have been already generated. 
    ```bash
    kubectl get globalreport        
    ```

    ```text                                                   
    NAME                          CREATED AT
    boutiqueshop-inventory        2022-11-03T12:44:06Z
    boutiqueshop-network-access   2022-11-03T12:44:06Z
    cluster-inventory             2022-11-03T12:44:06Z
    cluster-network-access        2022-11-03T12:44:06Z
    daily-cis-results             2022-11-03T12:44:06Z
    ```


    Calico provides `GlobalReport` resource to offer [Compliance reports](https://docs.tigera.io/compliance/overview) capability. There are several types of reports that you can configure:

    - CIS benchmarks
    - Inventory
    - Network access
    - Policy audit


    A compliance report could be configured to include only specific endpoints leveraging endpoint labels and selectors. Each report has the `schedule` field that determines how often the report is going to be generated and sets the time frame for the data to be included into the report.

    Compliance reports organize data in a CSV format which can be downloaded and moved to a long term data storage to meet compliance requirements.

    ![compliance report](../img/compliance-report.png)

2. Reports are generated 30 minutes after the end of the report as [documented](https://docs.tigera.io/compliance/overview#change-the-default-report-generation-time). As the compliance reports deployed in the [manifests](https://github.com/Azure/kubernetes-hackfest/tree/master/labs/networking/calico-lab-exercise/demo/40-compliance-reports) are scheduled to run every 15 minutes the generation of reports will take between 30-60 mins depending when the manifests were deloyed.
<br>

[Next -> Module 9](../calicocloud/enable-l7-visibility.md)
