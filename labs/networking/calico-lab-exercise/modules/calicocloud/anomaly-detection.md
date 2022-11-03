# Module 7: Anomaly Detection

**Goal:** Configure Anomaly Detection to alert upon abnormal/suspicious traffic
---

Calico offers [Anomaly Detection](https://docs.tigera.io/threat/security-anomalies) (AD) as a part of its [threat defense](https://docs.tigera.io/threat/) capabilities. Calico's Machine Learning software is able to baseline "normal" traffic patterns and subsequently detect abnormal or suspicious behavior. This may resemble an Indicator of Compromise and will generate an Alert in the UI.

## Steps

1. Configure the Anomaly Detection alerts.

    Instructions below are for a Managed cluster of version 3.14+. Follow the [Anomaly Detection doc](https://docs.calicocloud.io/threat/security-anomalies) to configure AD jobs in management and standalone clusters.

    Navigate to `Activity -> Anomaly Detection` view in the Calico Cloud UI and enable `Port Scan detection` alert.

    ![Anomaly Detection configuration](../img/anomaly-detection-config.png)

    Or use CLI command below to enable this alert.

    ```bash
    # example to enable AD alert via CLI command
    kubectl apply -f demo/90-anomaly-detection/ad-alerts.yaml
    ```

2. Confirm if the anomaly detector is running before simulating an abnomal behavior.

    ```bash
    kubectl get globalalerts | grep -i tigera.io.detector 
    ```

    The output should look similar to this:

    ```bash
    tigera.io.detector.port-scan   2022-11-03T13:05:05Z
    ```		

3. Enable the AD job for the anomaly detection analysis.

    ```bash
    sed -i 's,CLUSTERNAME,'"$CLUSTERNAME"',' demo/90-anomaly-detection/ad-jobs-deployment-managed.yaml
    kubectl apply -f demo/90-anomaly-detection/ad-jobs-deployment-managed.yaml
    ```

4. Confirm the ad job is running before simulating anomaly behaviour
   ```bash
   kubectl get pods -n tigera-intrusion-detection
   ```
   Output will be like:
   ```bash
   NAME                                              READY   STATUS    RESTARTS   AGE
   ad-jobs-deployment-86684f644c-xjht8               1/1     Running   0          19m
   intrusion-detection-controller-6f5986ff6f-tg2zq   1/1     Running   0          47m
   ```

5. Simulate a port scan anomaly by using an NMAP utility.

    ```bash
    # simulate port scan
    POD_IP=$(kubectl -n dev get po --selector app=nginx -o jsonpath='{.items[0].status.podIP}')
    kubectl -n dev exec netshoot -- nmap -Pn -r -p 1-600 $POD_IP
    ```

    ```text
    # expected output
    Starting Nmap 7.92 ( https://nmap.org ) at 2022-11-03 13:06 UTC
    Nmap scan report for 10.224.0.85
    Host is up.
    All 600 scanned ports on 10.224.0.85 are in ignored states.
    Not shown: 600 filtered tcp ports (no-response)
    
    Nmap done: 1 IP address (1 host up) scanned in 121.25 seconds
    ```

6. After a few minutes we can see the Alert generated in the Web UI

<img src="../img/anomaly-detection-alert.png" alt="Anomaly Detection Alert" width="100%"/>

[Next -> Module 8](../calicocloud/using-compliance-reports.md)

