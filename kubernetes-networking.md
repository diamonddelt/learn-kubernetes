# Kubernetes Networking

As each microservice is a discrete unit of work, requiring it's own unique set of networking requirements, networking in general becomes extremely important to Kubernetes.

Traditional monoliths are typically composed of a gigantic app, with a static IP address, which has its own DNS.

In microservice architecture, however, each microservice is it's own endpoint, and thus must `have it's own IP address` AND `be resolvable by every other endpoint (microservice)`. Scaling endpoints means there is a new normal for constant creation and deletion of private IP addresses in a subnet, meaning there has to be a way to reliably discover and route to each of these endpoints quickly whenever there is a change (scale up or down). 

This has led to the creation and need of `scalable DNS`, and there are a lot of new providers of this service.

In Kubernetes, the `service` / `svc` allows us to create a load balancer. In a k8s YAML file, this is described as `kind: Service`.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service-for-app-name
  labels:
    app: my-app-name
spec:
  ports:
    - port: 80
    - port: 443
  selector:
    app: my-app-name
    tier: frontend
  type: LoadBalancer
```

In the metadata above, the `label` tags this service as part of the app's frontend `tier`, while the `name` specifies what the service is referred to by the cluster's DNS (CoreDNS is an example of cluster DNS). In other words, once the kubernetes API creates this desired state, you could ping or reach the service by it's metadata `name`

You would choose `type: LoadBalancer` if your k8s cluster is on a major cloud platform (AWS/Azure/GCP) which can create a load balancer via API calls. This allows Kubernetes to provision one on your behalf with a public IP address, without you specifically knowing how to manipulate the cloud providers load balancer API.

The `selector:` section will proxy traffic to pods with labels matching the `app:` name. In an example case, with the above configuration, if an end user hits the cluster IP, and consequently hits this service/load balancer over port 443, it will have its traffic routed/proxied to any pod behind the service which has a container exposing a `containerPort: 443` and a label matching `app:`. This containerPort definition and the label live in the deployment object YAML when creating a k8s deployment.