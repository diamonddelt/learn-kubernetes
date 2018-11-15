# Other Kubernetes Features

- `Daemon Sets (ds)` - These work like a deployment, but their main concern is ensuring that one pod is running on each node in the cluster. You can think of the pod that the daemon set ensures is running as the `daemon` on a linux system.
- `Stateful Sets (sts)` - These also work like a deployment, but they manage pods with 'state' or stateful applications
- `Job` - manages pods, but it's main concern is making sure a specific set of steps controlling a pod's lifecycle gets executed properly
- `CronJob` - also in the `batch API` group, but these control when pods run i.e. setting up a cron schedule to run pods, as opposed to them continuously running until their container's command completes
- `PodSecurityPolicy (psp)` - specific security policies controlling what pods can do and what resources those pods can affect; this is important to consider in a production cluster's workload
- `ResourceQuota (quota)` - these behave like resource requests, but they operate at the namespace level; i.e. you can specify a limit that any particular namespace can use for any resource. this is good for distributed teams using a shared cluster with many namespaces, to avoid any one namespace consuming all of the resources.
- `CustomResourceDefinition (crd)` - allows you to define a custom resource. you could create a new resource, like a pod, deployment, etc, and manage it through the Kubernetes lifecycle. this can add a great deal of power to a cluster, but will also introduce challenges to ensure the custom resource conforms with the best practices and security governing the standard resources

As stated in the `scaling` section, you should always specify resource requests and limits in your Kubernetes objects, even if the deployment can handle scaling them or your cluster has sufficient resources to supply them. It's best practice and avoid any future, potential resource bottlenecks if you consider it from the start.