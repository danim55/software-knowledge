````markdown
## Table of Contents

1. [What Is a Pod?](#what-is-a-pod)  
2. [Pod Context & Isolation](#pod-context--isolation)  
3. [Pod Components](#pod-components)  
4. [Pod Use Cases](#pod-use-cases)  
5. [Pod Manifest Example](#pod-manifest-example)  
6. [Workload Resources & Controllers](#workload-resources--controllers)  
7. [Pod Lifecycle & Updates](#pod-lifecycle--updates)  
8. [Pod Subresources](#pod-subresources)  
9. [Resource Sharing & Communication](#resource-sharing--communication)  
10. [Pod Security Settings](#pod-security-settings)  
11. [Static Pods](#static-pods)  
12. [Container Probes](#container-probes)  

---

## What Is a Pod?

Pods are the smallest deployable units in Kubernetes. A Pod is a group of one or more containers with shared storage, network resources, and runtime context. Pods model an application‑specific "logical host": containers in the same Pod are co‑located, co‑scheduled, and run in a shared environment.

> **Note:** You need a container runtime on each node (e.g., containerd) for Pods to run there.

---

## Pod Context & Isolation

Within a Pod, containers share:

- **Linux namespaces** (PID, network, IPC)  
- **cgroups** for resource limits  
- **Volumes** for shared storage  

This context is analogous to a light VM; containers may have further sub‑isolations if needed.

---

## Pod Components

- **Application containers:** The main workloads.  
- **Init containers:** Run sequentially before app containers on startup.  
- **Ephemeral containers:** Injected at runtime for debugging.

---

## Pod Use Cases

1. **Single‑container Pods**  
   - Most common pattern: Pod wraps one container.  
   - Scaling is done by creating multiple Pods via a controller.

2. **Multi‑container Pods**  
   - Sidecar, ambassador, adapter patterns.  
   - Containers share network/IP (`localhost`) and volumes.  
   - Use only when containers are tightly coupled.

> You don’t use multiple containers for replication—use workload controllers instead.

---

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
````

Create it with:

```bash
kubectl apply -f https://k8s.io/examples/pods/simple-pod.yaml
```

---

## Workload Resources & Controllers

Rather than creating Pods directly, use controllers to manage them:

* **Deployment**
* **StatefulSet**
* **DaemonSet**
* **Job / CronJob**

Controllers handle replication, updates, and self‑healing (e.g., recreating Pods on node failure).

---

## Pod Lifecycle & Updates

* Pods are **ephemeral**: they run until completion, deletion, eviction, or node failure.
* A Pod’s **name** must be a valid DNS subdomain; hostnames follow DNS label rules.
* Most fields are **immutable** (e.g., name, namespace).
* To update spec fields, controllers create **replacement Pods** rather than patching existing ones.

---

## Pod Subresources

Special update paths:

* **Resize:** update `spec.containers[*].resources`
* **EphemeralContainers:** add debugging containers
* **Status:** update Pod status (used by kubelet)
* **Binding:** set `spec.nodeName` via binding API

---

## Resource Sharing & Communication

* **Storage:** Pods declare `volumes`; all containers can mount and share data.
* **Networking:** Pods get a unique IP; containers share network namespace and port space.

  * Intra‑Pod: communicate via `localhost`
  * Inter‑Pod: use Pod IP or Services

---

## Pod Security Settings

Use `securityContext` at Pod or container level to:

* Drop Linux capabilities
* Enforce non‑root user/group IDs
* Apply Seccomp or AppArmor profiles
* (Windows) set `hostProcess` or Windows security options

> **Caution:** Privileged mode overrides many security settings—use sparingly.

---

## Static Pods

* Defined on a node’s filesystem; managed directly by the kubelet.
* Used for self‑hosted control planes.
* Kubelet creates mirror Pods on the API server for visibility, but they cannot be controlled via the API.
* Cannot reference other API objects (ConfigMaps, Secrets, etc.).

---

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

```
::contentReference[oaicite:0]{index=0}
```
