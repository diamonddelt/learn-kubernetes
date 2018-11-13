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

These are different ways in which the API can prove the client executing an API action is `who they say they are`.

There are a few systems in place which support authentication:

1. Bearer tokens
2. Client certs
3. Bootstrap tokens
4. External systems

However, there is `no concept of users or user accounts` in Kubernetes. Kubernetes relies on external systems to implement users/user groups, such as Active Directory, IAM providers, or other federation systems.

The Kubernetes API expects a request to come with embedded authentication credentials in it, which it hands off to the 3rd party IAM solution. It expects the IAM solution to return a yes or no that the user is authenticated.

There is a type of account that `IS` stored in Kubernetes, but it is called a `service account`, and these are only used by the Kubernetes `system` components to do common tasks that client users do not worry about (examples being polling the cluster, monitoring pods, scaling operations, etc)

## authZ (Authorization)

Does this authorized user actually have permission to do this particular API action?

In other words, `who` can perform what `actions` to which `resources`?

RBAC operates on `deny by default`, so even if the user/client making the API request passes the _authentication_ stage, that user is `still not authorized` to do anything unless that is explicitly configured.

The difference when you create a new cluster is, the first user is given a superuser level account by default. This is not suitable for production, so dealing with RBAC and user accounts is really only a production-cluster issue. These are configured with:

- `Role` = this defines that verbs and actions can be done on various Kubernetes resources

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: demo-rbac-user
  namespace: demo-namespace
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

- `Role Bindings` = this links the role to the authenticated user, to enable that user to have authorization according to the `Role` document

```yaml
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: demo-rbac-binding
  namespace: demo-namespace
subjects:
- kind: User
  name: rrrasti@yahoo.com
  apiGroup: ""
roleRef:
  kind: Role
  name: demo-rbac-user
  apiGroup: ""
```

You can also create `ClusterRole` and `ClusterRoleBinding`, which look identical to the yaml manifests for Roles and RoleBindings, only these also grant authorization to the cluster and it's actions. In other words, `they are not restricted to a namespace.`

You can output the current yaml definitions of the current cluster role bindings with `$ kubectl get clusterrolebindings -o yaml` and grep through to find what you need

## Admission Control

Mutations and validations are applied at this step, such as forcing all create actions to have a specific label, or ensuring that only certain images are created in pods, etc

## Schema Validation

This is the last `gate` of security before an API request is actually performed.

## Insecure Local Port Communication

Some clusters allow you to bypass the authN and authZ steps by logging into the master node and making API requests directly to the Admission Control step. `This is not advised for production use.`

```text
Client makes request directly on master--> API server receives request --> admission control --> schema validation --> API action completed
```