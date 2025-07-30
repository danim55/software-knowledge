## Table of Contents

1. [What Is a Pod?](#what-is-a-pod)  
1. [Pod Manifest Example](#pod-manifest-example)  
1. [Workload Resources & Controllers](#workload-resources--controllers)  
1. [Pod Templates](#pod-templates)  
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

Usually you don't need to create Pods directly, even singleton Pods. Instead, create them using workload resources such as Deployment or Job. If your Pods need to track state, consider the StatefulSet resource.

Each Pod is meant to run a single instance of a given application. If you want to scale your application horizontally (to provide more overall resources by running more instances), you should use multiple Pods, one for each instance. In Kubernetes, this is typically referred to as replication. Replicated Pods are usually created and managed as a group by a workload resource and its controller.

Controllers handle replication, updates, and self‑healing (e.g., recreating Pods on node failure).

You'll rarely create individual Pods directly in Kubernetes—even singleton Pods. This is because Pods are designed as relatively ephemeral, disposable entities. When a Pod gets created (directly by you, or indirectly by a controller), the new Pod is scheduled to run on a Node in your cluster. The Pod remains on that node until the Pod finishes execution, the Pod object is deleted, the Pod is evicted for lack of resources, or the node fails.

> **Note:**
Restarting a container in a Pod should not be confused with restarting a Pod. A Pod is not a process, but an environment for running container(s). A Pod persists until it is deleted.

The name of a Pod must be a valid DNS subdomain value, but this can produce unexpected results for the Pod hostname. For best compatibility, the name should follow the more restrictive rules for a DNS label.

## Pod templates

Controllers for workload resources create Pods from a pod template and manage those Pods on your behalf.

PodTemplates are specifications for creating Pods, and are included in workload resources such as Deployments, Jobs, and DaemonSets.

Each controller for a workload resource uses the PodTemplate inside the workload object to make actual Pods. The PodTemplate is part of the desired state of whatever workload resource you used to run your app.

When you create a Pod, you can include environment variables in the Pod template for the containers that run in the Pod.

The sample below is a manifest for a simple Job with a template that starts one container. The container in that Pod prints a message then pauses.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: hello
spec:
  template:
    # This is the pod template
    spec:
      containers:
      - name: hello
        image: busybox:1.28
        command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 3600']
      restartPolicy: OnFailure
    # The pod template ends here
```

Modifying the pod template or switching to a new pod template has no direct effect on the Pods that already exist. If you change the pod template for a workload resource, that resource needs to create replacement Pods that use the updated template.

For example, the StatefulSet controller ensures that the running Pods match the current pod template for each StatefulSet object. If you edit the StatefulSet to change its pod template, the StatefulSet starts to create new Pods based on the updated template. Eventually, all of the old Pods are replaced with new Pods, and the update is complete.

Each workload resource implements its own rules for handling changes to the Pod template.

On Nodes, the kubelet does not directly observe or manage any of the details around pod templates and updates; those details are abstracted away. That abstraction and separation of concerns simplifies system semantics, and makes it feasible to extend the cluster's behavior without changing existing code.


## Pod Lifecycle & Updates

As mentioned in the previous section, when the Pod template for a workload resource is changed, the controller creates new Pods based on the updated template instead of updating or patching the existing Pods.

Kubernetes doesn't prevent you from managing Pods directly. It is possible to update some fields of a running Pod, in place. However, Pod update operations like patch, and replace have some limitations:

- Most of the metadata about a Pod is immutable. For example, you cannot change the namespace, name, uid, or creationTimestamp fields.
    - The generation field is unique. It will be automatically set by the system such that new pods have a generation of 1, and every update to mutable fields in the pod's spec will increment the generation by 1. 

- If the metadata.deletionTimestamp is set, no new entry can be added to the metadata.finalizers list.

- Pod updates may not change fields other than `spec.containers[*].image`, `spec.initContainers[*].image`, `spec.activeDeadlineSeconds` or `spec.tolerations`. For `spec.tolerations`, you can only add new entries.

- When updating the `spec.activeDeadlineSeconds` field, two types of updates are allowed:

    1. setting the unassigned field to a positive number;
    1. updating the field from a positive number to a smaller, non-negative number.


## Pod Subresources

The above update rules apply to regular pod updates, but other pod fields can be updated through subresources.

- **Resize:** The resize subresource allows container resources (spec.containers[*].resources) to be updated. See Resize Container Resources for more details.
- **Ephemeral** Containers: The ephemeralContainers subresource allows ephemeral containers to be added to a Pod. See Ephemeral Containers for more details.
- **Status:** The status subresource allows the pod status to be updated. This is typically only used by the Kubelet and other system controllers.
- **Binding:** The binding subresource allows setting the pod's spec.nodeName via a Binding request. This is typically only used by the scheduler.

## Resource Sharing & Communication

Pods enable data sharing and communication among their constituent containers.

Storage in Pods
A Pod can specify a set of shared storage volumes. All containers in the Pod can access the shared volumes, allowing those containers to share data. Volumes also allow persistent data in a Pod to survive in case one of the containers within needs to be restarted.

Pod networking
Each Pod is assigned a unique IP address for each address family. Every container in a Pod shares the network namespace, including the IP address and network ports. Inside a Pod (and only then), the containers that belong to the Pod can communicate with one another using localhost. When containers in a Pod communicate with entities outside the Pod, they must coordinate how they use the shared network resources (such as ports). Within a Pod, containers share an IP address and port space, and can find each other via localhost. The containers in a Pod can also communicate with each other using standard inter-process communications like SystemV semaphores or POSIX shared memory. Containers in different Pods have distinct IP addresses and can not communicate by OS-level IPC without special configuration. Containers that want to interact with a container running in a different Pod can use IP networking to communicate.

Containers within the Pod see the system hostname as being the same as the configured name for the Pod. There's more about this in the networking section.


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
