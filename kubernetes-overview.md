# Kubernetes Overview

## Control Plane

The brains of the cluster; contains the scheduling and control mechanisms for coordination. Includes:

- the Master nodes
- etcd (the persistent store. This is a distributed database, and consequently, the hardest part to scale if you roll your own k8s cluster)
- c-m
- sched.
- api (this is what kubectl talks to via POST requests)

The control plan *should* always be HA - so 3/5/7 nodes, etc - in other words, you need at least 2 replicas and a decider/arbiter node

## Workers

These are the nodes where the apps/containers/pods/etc run on. These contain your code/applications. The master nodes are controlling these.

## The Kubernetes API

Everything that you interact with or otherwise write or code in Kubernetes is contained within the API. The API contains all of the interactive bits of Kubernetes.

The API supports REST and CRUD operations:

- GET
- POST
- PATCH
- UPDATE
- PUT
- DELETE

Kubernetes works in `desired state configuration` - in other words, you define the YAML on how it is 'supposed' to be, and it does the work to make it so.

The API is split into major API groups:

- core
- apps
- authorization
- storage

All of these API groups are managed by `SIGs` (Special Interest Groups). These are people who are responsible for development of these API groups and their services. There is a SIG list which you can research on the k8s github page, which lists who those people are.

Kubernetes development is in three stages:

- Alpha (dangerous to use; bleeding edge)
- Beta (needs user acceptance testing but relatively useable)
- GA (ready for production use)