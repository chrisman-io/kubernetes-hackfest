# Module 2: Configuring demo applications

**Goal:** Deploy and configure demo applications.

## Steps

1. Change the directory to this lab environment:

    ```bash
    cd kubernetes-hackfest/labs/networking/calico-lab-exercise/
    ```

2. Deploy demo applications.
    This is app we will use for next modules.
      ![demo-diagram](../img/demo-diagram.png)

    ```bash
    # deploy dev app stack
    kubectl apply -f demo/dev/app.manifests.yaml

    # deploy boutiqueshop app stack.
    kubectl apply -f https://raw.githubusercontent.com/googlecloudplatform/microservices-demo/v0.3.8/release/kubernetes-manifests.yaml

    ```
      
    Confirm the pod/deployments are running. Note the loadgenerator pod waits for the frontend pod to respond to http calls before coming up and can take a few minutes. Eventually, the status of the pods in the default namespace will look as follows: 
    
    ```bash
    kubectl get pods -A -w | grep -e 'default\|dev'
    ```

    Output will be similar as below:
    ```bash
    default                      adservice-77d5cd745d-59rhv                        1/1     Running   0          90s
    default                      cartservice-74f56fd4b-pfh6j                       1/1     Running   0          91s
    default                      checkoutservice-69c8ff664b-dfwjn                  1/1     Running   0          91s
    default                      currencyservice-77654bbbdd-jzk82                  1/1     Running   0          91s
    default                      emailservice-54c7c5d9d-bjlh9                      1/1     Running   0          92s
    default                      frontend-99684f7f8-4sfnx                          1/1     Running   0          91s
    default                      loadgenerator-555fbdc87d-4nwv6                    1/1     Running   0          91s
    default                      paymentservice-bbcbdc6b6-d5xlr                    1/1     Running   0          91s
    default                      productcatalogservice-68765d49b6-mpwlj            1/1     Running   0          91s
    default                      recommendationservice-5f8c456796-zvk45            1/1     Running   0          91s
    default                      redis-cart-78746d49dc-4kppj                       1/1     Running   0          91s
    default                      shippingservice-5bd985c46d-nfqnf                  1/1     Running   0          91s
    dev                          centos                                            1/1     Running   0          92s
    dev                          dev-nginx-76c7dcb7b-d7f8d                         1/1     Running   0          92s
    dev                          dev-nginx-76c7dcb7b-r7667                         1/1     Running   0          92s
    dev                          netshoot                                          1/1     Running   0          92s
    ```


3. Install curl on loadgenerator pod
 
    > Before we implement network secruity rules we need to install curl on the loadgenerator pod for testing purposes later in the workshop. Note the installation will not survive a reboot so repeat this installation as necessary

    ```bash
    kubectl exec -it $(kubectl get po -l app=loadgenerator -ojsonpath='{.items[0].metadata.name}') -- sh -c 'apt-get update && sleep 10;apt install curl -y'
    ```

4. Deploy basic DNS policy as global allow.

    In order to explicitly allow workloads to connect to the Kubernetes DNS component, we are going to implement a policy that controls such traffic.
    We are going to deploy a few policies into policy tier to take advantage of hierarcical policy management.
    For more on tiers: https://docs.calicocloud.io/policy-design/overview


    This will add tiers `security` and `platform` to the aks cluster. 

    ```bash
    kubectl apply -f demo/tiers/tiers.yaml
    ```
    
    
    This will add `allow-kube-dns` policy to your `platform` tier. 
    ```bash
    kubectl apply -f demo/10-security-controls/allow-kube-dns.yaml
    ```
    

    This will add network policy to control the connection between different micro service of boutique app and `default-deny` policy to your `default` tier. 
    ```bash
    kubectl apply -f demo/boutiqueshop/policies.yaml
    kubectl apply -f demo/10-security-controls/default-deny.yaml
    ```

    


5. Deploy compliance reports.

    >The reports will be needed for a later lab.

    ```bash
    kubectl apply -f demo/40-compliance-reports
    ```
6. Deploy global alerts.

    >The alerts will be explored in a later lab. Ignore any warning messages - these do not affect the deployment of resources.

    ```bash
    kubectl apply -f demo/50-alerts/globalnetworkset.changed.yaml
    kubectl apply -f demo/50-alerts/unsanctioned.dns.access.yaml
    kubectl apply -f demo/50-alerts/unsanctioned.lateral.access.yaml
    ```


       
[Next -> Module 3](../calicocloud/using-observability-tools.md)
