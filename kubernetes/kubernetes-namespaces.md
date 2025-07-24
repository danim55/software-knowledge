## Table of Contents

- [What Are Namespaces?](#what-are-namespaces)  
- [When to Use Namespaces](#when-to-use-namespaces)  
- [Initial Namespaces](#initial-namespaces)  
- [Working with Namespaces](#working-with-namespaces)  
- [Namespaces and DNS](#namespaces-and-dns)  


## What Are Namespaces?

Namespaces provide a mechanism to isolate groups of Kubernetes resources within a single cluster. Resource names must be unique within each namespace, but the same name can be used across different namespaces. Namespaces apply only to **namespaced objects** (e.g., Deployments, Services), not to cluster-wide objects like Nodes, PersistentVolumes, or StorageClasses.  


## When to Use Namespaces

Namespaces are most useful in large or multi-team environments to:

- Isolate resource names and avoid naming collisions  
- Apply **resource quotas** per team or project  
- Scope **RBAC** rules and policies  
- Separate environments (e.g., dev, staging, prod) without creating separate clusters  
For small clusters (few users), namespaces are typically unnecessary.  


## Initial Namespaces

By default, Kubernetes includes these four namespaces:

`default`
- Kubernetes includes this namespace so that you can start using your new cluster without first creating a namespace.

`kube-node-lease`
- This namespace holds Lease objects associated with each node. Node leases allow the kubelet to send heartbeats so that the control plane can detect node failure.

`kube-public`
- This namespace is readable by all clients (including those not authenticated). This namespace is mostly reserved for cluster usage, in case that some resources should be visible and readable publicly throughout the whole cluster. The public aspect of this namespace is only a convention, not a requirement.

`kube-system`
- The namespace for objects created by the Kubernetes system.

**Note:** Avoid using the `default` namespace in production—create and use custom namespaces instead.


## Working with Namespaces

**Viewing existing namespaces:**

```bash
kubectl get namespace
````

Sample output:

```
NAME              STATUS   AGE
default           Active   1d
kube-node-lease   Active   1d
kube-public       Active   1d
kube-system       Active   1d
```

**Creating a new namespace:**

```bash
kubectl create namespace my-team
```

**Using a namespace in commands:**

* **Temporary scope:**

  ```bash
  kubectl get pods --namespace=my-team
  ```

* **Persistent default:**

  ```bash
  kubectl config set-context --current --namespace=my-team
  ```


## Namespaces and DNS

When you create a Service, it creates a corresponding DNS entry. This entry is of the form `<service-name>.<namespace-name>.svc.cluster.local`,
which means that if a container only uses `<service-name>`, it will resolve to the service which is local to a namespace. This is useful for using the same configuration across multiple namespaces such as Development, Staging and Production. If you want to reach across namespaces, you need to use the fully qualified domain name (FQDN).

As a result, all namespace names must be valid RFC 1123 DNS labels.


## Not All Objects Are Namespaced

* Namespaces apply only to namespaced objects (Pods, Services, Deployments, etc.)
* Cluster-wide objects like Nodes, PersistentVolumes, StorageClasses have no namespace
  To verify which resources are namespaced:

```bash
kubectl api-resources --namespaced=true
kubectl api-resources --namespaced=false
```
