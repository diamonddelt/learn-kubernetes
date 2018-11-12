# Kubernetes RBAC (Role-based Admission Control)

The security options available to you in Kubernetes are all centered around the API server. You as the administrator using `kubectl` on your client machine, as well as the cluster itself, both communicate with the API server to do everything, because all Kubernetes actions are modeled via CRUD REST-style request/response objects. And all of this communication back and forth to the API server takes place over HTTPS. 

The certificates to facilitate this HTTPS communication are taken care of when you install Kubernetes (it uses self-signed certificates by default, but you could use your own Certificate Authority if you wanted)

RBAC has been enabled since Kubernetes 1.6 (GA since 1.8), and is `deny-by-default` security

## Security Request Flow

The following flow assumes the communication is taking place over the secure HTTPS port

```text
Client makes request --> API server receives request --> authN --> authZ --> admission control --> schema validation --> API action completed
```

## authN (Authentication)

Is the client (user) making this request actually an authorized user?

## authZ (Authorization)

Does this authorized user actually have permission to do this particular API action?

## Admission Control

Mutations and validations are applied at this step, such as forcing all create actions to have a specific label, or ensuring that only certain images are created in pods, etc

## Schema Validation

This is the last `gate` of security before an API request is actually performed.

## Insecure Local Port Communication

Some clusters allow you to bypass the authN and authZ steps by logging into the master node and making API requests directly to the Admission Control step. `This is not advised for production use.`

```text
Client makes request directly on master--> API server receives request --> admission control --> schema validation --> API action completed
```