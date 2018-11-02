# Horizontal Pod Autoscaler Demo

The demo file `sample-hpa.yml` declares a minimal Horizontal Pod Autoscaler, which defines minimum and maximum pod thresholds and the scale parameter (in this case, each time a pod's CPU utilization is over 50%).

- `scaleTargetRef` refers to the resource to scale. In this example, it is a deployment, but it could be a replication controller or a replica set.