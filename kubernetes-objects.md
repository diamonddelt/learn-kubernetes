# Kubernetes Objects

Every one of the API primitives below gets deployed onto a Kubernetes cluster as an `object`

* Pod - a logical wrapper around one or more containers (typically Docker, but could be LXC or anything else)
* Deployment - a logical construct around one or more pods. The concept of a 'deployment' exists to quickly and reliably scale groups of pods up and down at the same time, when those pods are supposed to scale together. This ensures high availability and seamless rolling updates, etc
* ds (Daemon Sets) - ensures one and only pod of a specific type will run on any given Worker node on the cluster. This is to ensure high availability in case one of the Worker nodes goes down.
* sts (Stateful Sets) - these coordinate application components that require state i.e. an app that uses sticky sessions
* secrets
* pv (Persistent Volumes)
* svc (Services)
* ing (Ingress)