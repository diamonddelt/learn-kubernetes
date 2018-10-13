# Loadbalancing Demo

The demo files in this subdirectory aim to achieve the following:

- Test that a sample service is deployed correctly into a k8s cluster
- Test that a sample web app is deployed correctly into a k8s cluster with a service proxying traffic for it

## Demo files

1. `webserver.yaml` - declares a `service` for an application called _diamonddelt_ which uses the load balancing type of `NodePort` (which if you recall behaves like ClusterIP, but also allows the service to be reachable from outside of the cluster, by pointing to a specific node IP address and appending the NodePort port onto it). This file also declares a `deployment`, which is specifying a fleet size of 10 pods on the cluster, with rolling updates, and using the `nigelpoulton/acg-web:0.1 `container image for the pods (this is a CentOS/RedHat distro). The `---` block within the yaml file allows declaring more than one Kubernetes resource within the same file
2. `loadbalancer.yaml` - declares a `service` by itself, but this is intended to be run on a cloud platform provider, hence the type is `LoadBalancer` instead of ClusterIP or NodePort. It contains the same configuration as the service defined in webserver.yaml otherwise.


## Notes while writing/executing the demo deployment

1. When applying the webserver.yaml file, kubectl notes that two kinds of k8s objects are created when it is applied correctly:
    ```text
    λ kubectl apply -f webserver.yaml
    service/diamonddelt-svc created
    deployment.apps/diamonddelt-deploy created
    ```
2. You can see that the service of type NodePort (with NodePort value of 30001), and the 10 pods are created as expected:
    ```text
    λ kubectl get svc
    NAME              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
    diamonddelt-svc   NodePort    10.110.86.205   <none>        8080:30001/TCP   5m
    kubernetes        ClusterIP   10.96.0.1       <none>        443/TCP          6d

    λ kubectl get pods
    NAME                                  READY     STATUS    RESTARTS   AGE
    diamonddelt-deploy-564d784874-2sw54   1/1       Running   0          4m
    diamonddelt-deploy-564d784874-bzdtb   1/1       Running   0          4m
    diamonddelt-deploy-564d784874-crp9w   1/1       Running   0          4m
    diamonddelt-deploy-564d784874-f776q   1/1       Running   0          4m
    diamonddelt-deploy-564d784874-hjf9d   1/1       Running   0          4m
    diamonddelt-deploy-564d784874-ndq52   1/1       Running   0          4m
    diamonddelt-deploy-564d784874-rhsf2   1/1       Running   0          4m
    diamonddelt-deploy-564d784874-ss4z5   1/1       Running   0          4m
    diamonddelt-deploy-564d784874-v2fk9   1/1       Running   0          4m
    diamonddelt-deploy-564d784874-wh2q9   1/1       Running   0          4m
    ```
3. To verify that the NodePort service is working, we want to validate two conditions:
    1. That the nodes are reachable by the CLUSTER-IP address of the service with the internal port 8080 appended to it
    2. That the nodes are reachable by the NODE-IP address of any individual node, with the NodePort 30001 appended to it

    3. I connected to one of the pods at random with `kubectl exec -it diamonddelt-deploy-564d784874-2sw54`, and installed networking utilities on it, as before in the networking demo
    4. I was able to curl the service by its DNS name with port 8080 appended to it (to verify the cluster DNS and cluster IP works): 
        ```text
        [root@diamonddelt-deploy-564d784874-2sw54 /]# curl diamonddelt-svc:8080
        <html><head><title>ACG loves K8S</title><link rel="stylesheet" href="http://netdna.bootstrapcdn.com/bootstrap/3.1.1/css/bootstrap.min.css"/></head><body><div class="container"><div class="jumbotron"><h1>A Cloud Guru loves Kubernetes!!!</h1><p></p><p> <a class="btn btn-primary" href="https://www.amazon.com/Kubernetes-Book-Nigel-Poulton/dp/1521823634/ref=sr_1_3?ie=UTF8&amp;qid=1531240306&amp;sr=8-3&amp;keywords=nigel+poulton">The Kubernetes Book</a></p><p></p></div></div></body></html>
        ```
    5. I was also able to curl the service by a specific Node's IP address with NodePort 30001 appended to it (to verify the NodePort works):
    ```text
    curl 10.166.0.2:30001
    <Returns the same HTML output as above>
    ```
4. Deploying the service defined in `loadbalancer.yaml` while the previous pods, containers, and deployment is still active in the cluster will overwrite the service defined in webserver.yaml, while keeping existing cluster infrastructure as is. It is important to ensure that `selector: diamonddelt` is added to the newer loadbalancer.yaml definition, so it knows to control the same app.
    1. NOTE: Because I ran this command locally in a Minikube cluster, instead of hooking into an actual cloud provider, I expected this update to fail in an unexpected way:

    ```text
    λ kubectl get svc --watch
    NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
    diamonddelt-svc        NodePort       10.110.86.205   <none>        8080:30001/TCP   38m
    kubernetes             ClusterIP      10.96.0.1       <none>        443/TCP          6d
    loadbalancer-service   LoadBalancer   10.102.196.17   <pending>     8080:31620/TCP   <invalid>
    ```

    It appears that the age will always be <invalid> when the cloud provider is not available.