# Scaling Kubernetes

## When to scale

- Usually when demand goes up, you usually add more pods. However, if all of the CPU/RAM/resources on the existing nodes are being used, then adding more pods `doesn't work`. In that case, it is required to add more nodes to the cluster.

There are a few types of autoscaling that come with Kubernetes out of the box:

1. Horizonal Pod Autoscaling - this automatically increases the pod replica count in a deployment object
    - if there is no capacity (cpu/ram/etc), the additional pods are put in a `pending` state, and every 10 seconds, a watch determines if more nodes should be added to resolve `pending pods`
2. Vertical Pod Autoscaling

## Horizontal Pod Autoscaler (HPA)

- This is considered another type of Kubernetes object, of `kind: HorizontalPodAutoscaler`. This sets thresholds which control the overall triggering of pod creation.
- An example being a `targetCPUUtilizationPercentage: 50` means that the autoscaler will monitor the cluster, and spin up a new pod if it sees that any pod is consuming over 50% of a single CPU on the cluster.
- Load and redundancy is already considered, so the HPA doesn't worry about load balancing or making sure that pods/nodes are evenly balanced.