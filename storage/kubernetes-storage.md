# Kubernetes Storage

The whole point of Kubernetes Volumes is keeping data separate from pods, since pods are ephemeral and are constantly in flux. Volumes are therefore treated as actual volumes residing on the cluster, and are intended to have a lifetime longer than the scope of any pod.

* If a pod wants to use a volume, it needs to `claim` the volume for it's own use, and then `mount` the volume. This is where the `Persistent Volume` and `Persistent Volume Claim` terminology come from.
* You can share a volume between multiple pods (with some restrictions)
* File and Block storage types have standards and pluggable backends with rich APIs
* Kubernetes `leaves the storage specifics to the storage provider` - it just provides an interface for pods to consume it
* In other words, your enterprise storage system (Array, SAN, LUNs, other network storage device, etc) plugs into Kubernetes. Kubernetes then allows you to interact with the storage system via it's own abstraction layer, called the `PV (Persistent Volume) Subsystem`
    - The thing the enterprise storage system actually connects with is the `Container Storage Interface (CNI)`
    - The CNI allows a storage provider to write their own plugins that work with Kubernetes
    - As a practical example, assume you had a managed, enterprise-grade SAN from Dell. This would plug into Kubernetes via a CNI plugin, which is developed and maintained by Dell. Since Dell owns the storage device, they know best how to tell Kubernetes how to interact with their own hardware.

## Main Storage Kubernetes Primitives

1. Persistent Volumes (pv) - you can imagine this as a representation of the storage resource, in a specific slice or amount. For example, you could have a 20GB pv which is provisioned from the overall 50TB SAN

2. Persistent Volume Claims (pvc) - this could be equated to a ticket or a piece of paper which says 'I am allowed to use this particular Persistent Volume (pv) object'. `Each app needs to have a pvc to be able to use a particular pv.`

3. StorageClass (sc) - this is just a way to implement the PV + PVC architecture in a more scalable fashion.

## Container Storage Interface (CSI)

The history behind CSI was that originally, the code to handle storage providers was housed in `tree` (main line of Kubernetes open source code). This proved to be inflexible, and required vendor code dependencies in an open source project.

The move to CSI meant that storage providers could `research their own plugins for their storage devices on their own schedules, and didn't have to make their plugin code open source`

The CSI is considered a `standard`, so in theory, if you write a plugin conforming to CSI standards, it `should` work across any container orchaestrator, including Kubernetes, Docker Swarm, Mesos, etc

## Persistent Volumes

You create these in the cluster, but they exist outside of the scope of the pods. They are abstract slices of specific cloud and/or storage provider infrastrucure. 

For example, on Google Cloud Platform, you could make either a regular disk or SSD disk for storage, each providing a different amount of IOPS and raw data storage. In the k8s persistent volume, `you don't care about these specifics as they relate to GCP`, but you work with k8s attributes of a persistent volume, and the translation between the k8s version of IOPS and GCP is handled by the GCP CNI plugin.

A persistent volume is matched with its corresponding `claim` by matching the `spec:` definition between the claim and the volume.

## Pod Access Modes

There are three types of access modes that a pod can use to interact with a pv:

1. ReadWriteOnce
2. ReadWriteMany
3. ReadOnlyMany

The Once vs. Many is referring to the number of pods which can mount and access it. `ReadWriteOnce` only allows one pod to read and write to it at a time. `ReadWriteMany` allows multiple pods to read and write at the same time, and `ReadOnlyMany` only allows read operations.

Note: Not all volumes support all of these modes, and a `PV can only have one active PVC/AccessMode engaged at a time`, so it cannot be both a ReadWriteMany AND a ReadOnlyMany to different pods.

## Reclaim Policy

This is the behavior that a pv does when a claim is released (i.e. the pod was destroyed, or no longer needs the claim to access the volume).

1. Delete (deletes the claim, and can optionally delete the actual storage resource it represents)
2. Retain (keeps the volume, even if the pod(s) accessing it via pvc claims are all destroyed)