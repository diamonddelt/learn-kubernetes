# Networking Demo

The demo files in this subdirectory aim to achieve the following:

- Test that a sample fleet of 3 pods are successfully deployed into a k8s cluster
- verify that the networking of the k8s cluster is working properly

Note: if you are deploying this onto a cluster with only one node (like I am testing it with on Docker/Kubernetes for Desktop on OS X), then all three pods will be put on the same node. But if the test is intended to be on a cluster with multiple nodes, such as 5, I would make the following modification to `connectivity-deployment.yaml`...

```yaml
spec:
  replicas: 5
```

to ensure that one pod was assigned to one node.

To apply the k8s deployment, I ran `kubectl apply -f connectivity-deployment.yml` in the current directory.

## Notes while writing/executing the demo deployment

1. After applying deployment, I noticed that no pods were available or in the `Ready` status. I ran `kubectl get pods -o wide` and saw that the status of the pods was in `CrashLoopBackOff`. Researching this, I realized that the container was encountering an error preventing it from running (it was the result of trying to run a bash shell inside of a container that didn't have bash installed by default). I got further debugging information by looking at the event process log by using `kubectl describe pod` while the pods were in an error state.
    - Because the restart policy was set to `Always`, the deployment kept trying to instantiate pods at higher and higher time intervals, despite them failing on the same error.
    - To resolve the problem, I used a different container base image which contained the bash shell by default (`ubuntu:latest, as opposed to alpine:latest`)
    - Re-applying the deployment and re-running `kubectl get pods -o wide` now shows all pods in a `READY` state and `Running` status.
2. After applying the deployment, the IP output of the pods was the following:

    ```text
    NAME                               READY     STATUS    RESTARTS   AGE       IP          NODE
    networking-test-6474876f79-4hwjh   1/1       Running   0          1m        10.1.0.56   docker-for-desktop
    networking-test-6474876f79-d9j7z   1/1       Running   0          1m        10.1.0.54   docker-for-desktop
    networking-test-6474876f79-xkcxq   1/1       Running   0          1m        10.1.0.55   docker-for-desktop
    ```

    Each node in the cluster would have a pod inside of its own subnet in the 10.X.X.X/24 CIDR block, but since I'm using docker-for-desktop, I only have one node, and subsequently one subnet associated with all three pods
3. I chose one pod at random, and ran `kubectl exec -it <pod identifier> bash` to enter a bash shell in the currently sleeping pod
4. The base image (ubuntu:latest) of the containers in the pods do not come with networking utilities by default, so it required an `apt-get update -y` and `apt-get install -y iputils-ping curl dnsutils iproute2`
5. Once the networking utilities are installed, I successfully pinged outside of the cluster with `ping 8.8.8.8`
6. I found out the individual pods IP address with `ip a | grep eth0`, because the pod is assigned the eth0 interface
7. I connected to a second pod at random, and used the same commands used in steps 3 and 4 to identify that pods IP address and install networking utilities
8. Once those were installed, I pinged the first pod's IP address and verified that two pods could communicate with each other `by default`