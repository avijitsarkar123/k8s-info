## Managing Compute Resources for Containers

- When you specify a Pod, you can optionally specify how much CPU and memory (RAM) each Container needs. When Containers have resource requests specified, the scheduler can make better decisions about which nodes to place Pods on.

- CPU and memory are each a resource type. A resource type has a base unit. CPU is specified in units of cores, and memory is specified in units of bytes.

### Resource requests and limits of Pod and Container

Each Container of a Pod can specify one or more of the following:

- spec.containers[].resources.limits.cpu
- spec.containers[].resources.limits.memory
- spec.containers[].resources.requests.cpu
- spec.containers[].resources.requests.memory

**Example:**

- The following Pod has two Containers. Each Container has a request of 0.25 cpu and 64MiB (226 bytes) of memory.

- Each Container has a limit of 0.5 cpu and 128MiB of memory.

- You can say the Pod has a request of 0.5 cpu and 128 MiB of memory, and a limit of 1 cpu and 256MiB of memory.

```
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: db
    image: mysql
    env:
    - name: MYSQL_ROOT_PASSWORD
      value: "password"
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
  - name: wp
    image: wordpress
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

### How Pods with resource requests are scheduled

- When you create a Pod, the Kubernetes scheduler selects a node for the Pod to run on. Each node has a maximum capacity for each of the resource types: the amount of CPU and memory it can provide for Pods.

- The scheduler ensures that, for each resource type, the sum of the resource requests of the scheduled Containers is less than the capacity of the node.

- Note that although actual memory or CPU resource usage on nodes is very low, the scheduler still refuses to place a Pod on a node if the capacity check fails.

### How Pods with resource limits are run

- If a Container exceeds its memory limit, it might be terminated. If it is restartable, the kubelet will restart it, as with any other type of runtime failure.

- If a Container exceeds its memory request, it is likely that its Pod will be evicted whenever the node runs out of memory.

- A Container might or might not be allowed to exceed its CPU limit for extended periods of time. However, it will not be killed for excessive CPU usage

### Troubleshooting

- You can check node capacities and amounts allocated with the below command:

```
kubectl describe nodes
```

- By looking at the Pods section, you can see which Pods are taking up space on the node.

- The amount of resources available to Pods is less than the node capacity, because system daemons use a portion of the available resources.

- If the scheduler cannot find any node where a Pod can fit, the Pod remains unscheduled until a place can be found. An event is produced each time the scheduler fails to find a place for the Pod

- Container might get terminated because it is resource-starved. To check whether a Container is being killed because it is hitting a resource limit, call `kubectl describe pod` on the Pod of interest
