# Kubernetes Deployments

Kubernetes does not let the user run 'native' or 'naked' containers - they need to be packaged in a Pod. And manually creating, deleting, or scaling pods can instantly become an impossible challenge when you intend to rapidly scale an microservice architecture up and down. That is why a `deployment` exists - it declaritively handles the scaling and pod management within the yaml file parameter spec.

You could think of a deployment as a higher level `wrapper` for a pod, similar to how a pod is a higher level wrapper for a container image. Behind the scenes, the `replica set` object in Kubernetes is actually handling the logic operations of scaling, but since you as the user rarely interact with a replica set directly, you should just know that they sit `in between` a pod and a deployment, with the deployment object leveraging the capabilities of a replica set.

## Deployment Concepts

* A deployment only manages a single type of pod. For example, you would need two deployment objects to 'deploy' a fleet of Redis pods, and a fleet of Logstash pods to the same cluster.
* All changes to a deployment manifest should be made in the actual manifest yaml file, and promptly checked into version control. That way, the deployment manifest represents the `desired state configuration` of the cluster's deployment of pods.
* RollingUpdate strategy is the most common strategy for updating container versions in the pods of a deployment. They are used for 0 downtime deployments by specifying minimum and maximum thresholds of pods as the update happens
* A rolling update strategy actually creates a second replica set to hold the new version of the containers, while the deployment uses the old replica set to slowly decrement the old version of containers
* `labels` are the heart of the Kubernetes locator strategy; so every object must have a unique label. A deployment finds the pods to manage by the `matchLabels:` property in the yaml manifest. The deployment, once created, also adds its own labels to the pods it deployed, and this is tracked by the `template metadata label` in the yaml manifest.
* The `template` section of a deployment manifest is responsible for what triggers an update in the deployment. If you change a value in that section, the deployment will update itself.
* The other sections of the deployment manifest are used to specify information about what the deployment will do (i.e. it's strategy or labels) when the next update occurs