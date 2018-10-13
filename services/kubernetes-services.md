# Kubernetes Services

Services (svc) are responsible for keeping ensuring that microservice A which needs to talk to Pods in microservice B can always communicate, even when microservice B pods are constantly being created or destroyed (and thus, losing, gaining, or changing IP addresses). In other words, `services keep track of pods constantly churning IP addresses`, and can be considered an abstraction for a network around pods. Services can usually be thought of as front-end loadbalancers or proxies to the individual pods and containers

## Services requirements

* Each service is assigned a name and an IP address, and these are guaranteed to be stable i.e. kubernetes guarantees the name and the IP address will never change during the svc's lifetime. The IP address is assigned by the cluster's built-in DNS provider (i.e. CoreDNS for example)
* When you give a service a `label selector`, such as `app=search`, you are saying to load balance all traffic to pods with the label `app=search`. You can assign multiple labels to a service
* When you create a `svc` object on the cluster, Kubernetes also creates an `ep (Endpoint)` object at the same time. This ep is a list pod IPs and ports that match the label selector on the service's configuration/YAML
* As pods are created/updated/deleted over the course of the service's lifetime, it is constantly updating the endpoint's list to reflect the current state of the pod IPs and ports

## Types of services

1. LoadBalancer
2. NodePort
3. ClusterIP

Keep in mind that all of these types of services do roughly the same thing - they balance traffic across nodes and pods.

### ClusterIP

This is the default type of service - if a YAML file describes a service, but does not specify a type, it is assumed to be `ClusterIP`. ClusterIP gets its own IP address and is only accessible within the cluster.

### NodePort

This gets it's own IP address like ClusterIP, but is `also` accessible from outside of the cluster, and gets a cluster-wide port. This allows the service to be reachable from outside of the cluster by combining the individual node IP address with the cluster-wide port appended to it. This allows you to actually connect to a service without knowing what the service's own IP address is. You can access from inside or outside of the cluster.

The port range can only be between 30000 - 32767.

### LoadBalancer

This integrates with a public cloud platform.

## The Service Network

All newly created services are given a long-lived IP address, which is on a different class of network than the pods it load balances traffic for. It sits on a third network segment called the `service network`. For reference, here are the following three networks in Kubernetes and their associated class IP ranges and CIDR blocks:

```yaml
Pod network:
  - 10.0.0.0/16
Node network:
  - 192.168.0.0/16
Service network:
  - 172.11.11.0/24
```

The service network is not actually routable from any pod. Traffic reaches it through a daemon set running on all nodes called the `kube-proxy`. There is always `one kube-proxy on every k8s node`. Essentially, this works by creating a set of IP lookup table rules on each node, so that when a pod in a node tries to resolve another pod, the lookup table proxies the traffic through the service, because only this lookup table (controlled via kube-proxy) knows what the services IP address is on the `service network`.

Since Kubernetes 1.2, the default kube-proxy runs in `IPTABLES mode`. However this wasn't really designed for scaling for load balancing, so now the default mode is `IPVS mode`, which uses the Linux kernel IP virtual server. It supports more routing algorithms besides round-robin too.