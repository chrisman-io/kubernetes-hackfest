# Module 1: Configuring demo applications

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

    # deploy boutiqueshop app stack
    kubectl apply -f https://raw.githubusercontent.com/googlecloudplatform/microservices-demo/v0.3.8/release/kubernetes-manifests.yaml
    ```
    
    ```bash
    #confirm the pod/deployments are running. Note the loadgenerator pod waits for the frontend pod to respond to http calls before coming up and can take a few minutes. Eventually, the status of the pods will look as follows: 
    kubectl get pods -A -w | grep -e 'default\|dev'
    ```

    Output will be similar as below:
    ```bash
    default           adservice-77d5cd745d-8d5f6                1/1     Running   0          7m38s
    default           cartservice-74f56fd4b-nttdn               1/1     Running   0          7m39s
    default           checkoutservice-69c8ff664b-hd8nt          1/1     Running   0          7m39s
    default           currencyservice-77654bbbdd-m96s8          1/1     Running   0          7m39s
    default           emailservice-54c7c5d9d-r9hb4              1/1     Running   0          7m39s
    default           frontend-99684f7f8-v5q4h                  1/1     Running   0          7m39s
    default           loadgenerator-555fbdc87d-mj4qx            1/1     Running   0          7m39s
    default           paymentservice-bbcbdc6b6-vcx2w            1/1     Running   0          7m39s
    default           productcatalogservice-68765d49b6-gghbr    1/1     Running   0          7m39s
    default           recommendationservice-5f8c456796-b9v7v    1/1     Running   0          7m39s
    default           redis-cart-78746d49dc-j62kq               1/1     Running   0          7m38s
    default           shippingservice-5bd985c46d-f5b8n          1/1     Running   0          7m39s
    dev               centos                                    1/1     Running   0          7m40s
    dev               dev-nginx-76c7dcb7b-h79hs                 1/1     Running   0          7m40s
    dev               dev-nginx-76c7dcb7b-ttk89                 1/1     Running   0          7m40s
    dev               netshoot                                  1/1     Running   0          7m40s
    ```


3. Install curl on loadgenerator pod
 
    > Before we implement network secruity rules we need to install curl on the loadgenerator pod for testing purposes later in the workshop. Note the installation will not survive a reboot so repeat this installation as necessary

    ```bash
    kubectl exec -it $(kubectl get po -l app=loadgenerator -ojsonpath='{.items[0].metadata.name}') -- sh -c 'apt-get update && apt install curl -y'
    ```

[Next -> Module 2](../calicooss/using-security-controls.md)
