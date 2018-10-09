# Kubernetes Objects

Every one of the API primitives below gets deployed onto a Kubernetes cluster as an `object`

* Pod - a logical wrapper around one or more containers (typically Docker, but could be LXC or anything else)
* Deployment - a logical construct around one or more pods. The concept of a 'deployment' exists to quickly and reliably scale groups of pods up and down at the same time, when those pods are supposed to scale together. This ensures high availability and seamless rolling updates, etc
* ds (Daemon Sets) - ensures one and only pod of a specific type will run on any given Worker node on the cluster. This is to ensure high availability in case one of the Worker nodes goes down.
* sts (Stateful Sets) - these coordinate application components that require state i.e. an app that uses sticky sessions
* secrets - these are objects which are intended to facilitate secure communication between other k8s objects, such as deployments, persistent-volumes, or services
* pv (Persistent Volumes) - this is a primitive which can hook into a cloud or other raw infrastructure and store data persistently, that a fluid, scaling microservices architecture can read from consistently. This can be thought of as your traditional `SAN`. These are mapped with `pvc`s (Persistent Volume Claims)
* svc (Services) - these can be thought of as code, applications, scripts, or other components which do not necessarily belong to kubernetes, but that help facilitate traffic getting into the cluster (AKA `network abstraction`). An example of an `svc` would be an implementation of a load balancer
* ing (Ingress)