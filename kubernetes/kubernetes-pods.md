## Table of Contents

1. [What Is a Pod?](#what-is-a-pod)  
1. [Pod Manifest Example](#pod-manifest-example)  
1. [Workload Resources & Controllers](#workload-resources--controllers)  
1. [Pod Lifecycle & Updates](#pod-lifecycle--updates)  
1. [Pod Subresources](#pod-subresources)  
1. [Resource Sharing & Communication](#resource-sharing--communication)  
1. [Pod Security Settings](#pod-security-settings)  
1. [Static Pods](#static-pods)  
1. [Container Probes](#container-probes)  

# Pods

Pods are the smallest deployable units of computing that you can create and manage in Kubernetes.

A Pod (as in a pod of whales or pea pod) is a group of one or more containers, with shared storage and network resources, and a specification for how to run the containers. A Pod's contents are always co-located and co-scheduled, and run in a shared context. A Pod models an application-specific "logical host": it contains one or more application containers which are relatively tightly coupled. In non-cloud contexts, applications executed on the same physical or virtual machine are analogous to cloud applications executed on the same logical host.

As well as application containers, a Pod can contain init containers that run during Pod startup. You can also inject ephemeral containers for debugging a running Pod.

## What Is a Pod?

> **Note:** You need a container runtime on each node (e.g., containerd) for Pods to run there.

Within a Pod, containers share:

- **Linux namespaces** (PID, network, IPC)  
- **cgroups** for resource limits  
- **Volumes** for shared storage  

This context is analogous to a light VM; containers may have further sub‑isolations if needed.

A Pod is similar to a set of containers with shared namespaces and shared filesystem volumes.

- **Application containers:** The main workloads.  
- **Init containers:** Run sequentially before app containers on startup.  
- **Ephemeral containers:** Injected at runtime for debugging.

Pods in a Kubernetes cluster are used in two main ways:

- **Pods that run a single container.** The "one-container-per-Pod" model is the most common Kubernetes use case; in this case, you can think of a Pod as a wrapper around a single container; Kubernetes manages Pods rather than managing the containers directly.

- **Pods that run multiple containers that need to work together.** A Pod can encapsulate an application composed of multiple co-located containers that are tightly coupled and need to share resources. These co-located containers form a single cohesive unit.

    Grouping multiple co-located and co-managed containers in a single Pod is a relatively advanced use case. You should use this pattern only in specific instances in which your containers are tightly coupled.


## Pod Manifest Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: nginx
      image: nginx:1.14.2
      ports:
        - containerPort: 80
```

Create it with:

```bash
kubectl apply -f https://k8s.io/examples/pods/simple-pod.yaml
```


## Workload Resources & Controllers

Rather than creating Pods directly, use controllers to manage them:

* **Deployment**
* **StatefulSet**
* **DaemonSet**
* **Job / CronJob**

Controllers handle replication, updates, and self‑healing (e.g., recreating Pods on node failure).


## Pod Lifecycle & Updates

* Pods are **ephemeral**: they run until completion, deletion, eviction, or node failure.
* A Pod's **name** must be a valid DNS subdomain; hostnames follow DNS label rules.
* Most fields are **immutable** (e.g., name, namespace).
* To update spec fields, controllers create **replacement Pods** rather than patching existing ones.


## Pod Subresources

Special update paths:

* **Resize:** update `spec.containers[*].resources`
* **EphemeralContainers:** add debugging containers
* **Status:** update Pod status (used by kubelet)
* **Binding:** set `spec.nodeName` via binding API


## Resource Sharing & Communication

* **Storage:** Pods declare `volumes`; all containers can mount and share data.
* **Networking:** Pods get a unique IP; containers share network namespace and port space.

  * Intra‑Pod: communicate via `localhost`
  * Inter‑Pod: use Pod IP or Services


## Pod Security Settings

Use `securityContext` at Pod or container level to:

* Drop Linux capabilities
* Enforce non‑root user/group IDs
* Apply Seccomp or AppArmor profiles
* (Windows) set `hostProcess` or Windows security options

> **Caution:** Privileged mode overrides many security settings—use sparingly.


## Static Pods

* Defined on a node's filesystem; managed directly by the kubelet.
* Used for self‑hosted control planes.
* Kubelet creates mirror Pods on the API server for visibility, but they cannot be controlled via the API.
* Cannot reference other API objects (ConfigMaps, Secrets, etc.).


## Container Probes

Diagnostics run by the kubelet:

* **ExecAction:** run a command in the container
* **TCPSocketAction:** check a TCP port
* **HTTPGetAction:** perform an HTTP GET

Configure probes in the Pod spec to monitor container health:

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
```