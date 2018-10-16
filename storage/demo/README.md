# Storage Demo

The files within the demo subdirectory contain a Persistent Volume and Persistent Volume claim definition.

## Persistent Volume

The pv-example.yml defines a persistent volume to be created on Google Cloud, including specifics such as the volume being 20GB in size, and being an SSD volume. It also specifies the name of an actual GCP disk resource that must _already_ exist on the account, called `main-disk`. To follow along this example, you would need to create a new GCP disk and name it "main-disk", as well as ensure that it is of type SSD and has at least 20GB of storage space to allocate.

## Persistent Volume Claim

The pvc-example.yml is the corresponding persistent volume claim file which a pod can use to lay claim to the persistent volume declared above. You can see the matching spec details, including the `accessModes`, `storageClassName`, and `storage` values.

## Pod Referencing a PVC

The podpvcmount.yml file shows what a Pod manifest file would look like which actually implements the Persistent Volume Claim, which claims the Persistent Volume configured for a Google Cloud disk